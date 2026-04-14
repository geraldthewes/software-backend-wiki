# gRPC API Design

> **Tier 3** | Source: gRPC documentation, Protocol Buffers spec | Enforces/Derives From: wiki/tier2-core/distributed-systems/overview.md

## Summary

gRPC is a high-performance RPC framework using Protocol Buffers as the serialization format. It is the preferred protocol for internal service-to-service communication where performance, strong typing, and bidirectional streaming matter.

## When to Use gRPC Over REST

| Requirement | gRPC | REST |
|------------|------|------|
| Internal microservice communication | Preferred | Acceptable |
| High throughput, low latency | Yes (binary, HTTP/2) | Heavier (JSON, HTTP/1.1) |
| Bidirectional streaming | Yes (native) | No |
| Browser clients | No (needs grpc-web proxy) | Yes |
| Human-readable debugging | No | Yes |
| Strongly typed contract | Yes (protobuf) | Via OpenAPI |
| Code generation for all languages | Yes | Via openapi-generator |

Binary protobuf messages are typically 3–10x smaller than equivalent JSON, and HTTP/2 multiplexing reduces connection overhead.

## Protocol Buffers

The `.proto` file is the contract. Both client and server code are generated from it.

```protobuf
syntax = "proto3";

package users.v1;

option go_package = "github.com/myorg/myservice/gen/users/v1;usersv1";
option python_package = "gen.users.v1";

// User domain message
message User {
  int64 id = 1;
  string name = 2;
  string email = 3;
  string created_at = 4;
}

// Request/Response messages — one per RPC method
message GetUserRequest {
  int64 user_id = 1;
}

message CreateUserRequest {
  string name = 1;
  string email = 2;
}

message ListUsersRequest {
  int32 page_size = 1;
  string page_token = 2;
}

message ListUsersResponse {
  repeated User users = 1;
  string next_page_token = 2;
}

// Service definition
service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc CreateUser(CreateUserRequest) returns (User);
  rpc ListUsers(ListUsersRequest) returns (ListUsersResponse);
  rpc StreamUsers(ListUsersRequest) returns (stream User);      // server streaming
}
```

## Service Definition Patterns

### Unary RPC (most common)

```protobuf
rpc GetUser(GetUserRequest) returns (User);
```

Client sends one request, server returns one response.

### Server Streaming

```protobuf
rpc ListUsers(ListUsersRequest) returns (stream User);
```

Server streams multiple responses. Client reads until stream is closed. Use for large result sets that should not be buffered in memory.

### Client Streaming

```protobuf
rpc CreateUsers(stream CreateUserRequest) returns (CreateUsersResponse);
```

Client streams multiple requests. Server responds once. Use for bulk operations.

### Bidirectional Streaming

```protobuf
rpc Chat(stream ChatMessage) returns (stream ChatMessage);
```

Both sides stream independently. Use for real-time communication.

## Field Numbering Rules

Field numbers identify fields in the binary encoding. These rules are critical for backward compatibility:

- **Never change a field number** — old clients will misread the field.
- **Never reuse a field number** — even after deleting the old field.
- **Add new fields with new numbers** — existing clients ignore unknown fields.
- **Mark deleted fields as reserved** — prevents accidental reuse.

```protobuf
message User {
  int64 id = 1;
  string name = 2;
  // string old_field = 3;  -- deleted
  string email = 4;

  reserved 3;              // prevent reuse of old field number
  reserved "old_field";    // prevent reuse of old field name
}
```

## gRPC Error Model

gRPC status codes replace HTTP status codes:

| gRPC Status | Equivalent HTTP | Use For |
|-------------|-----------------|---------|
| `OK` | 200 | Success |
| `NOT_FOUND` | 404 | Resource does not exist |
| `INVALID_ARGUMENT` | 400 | Client sent invalid data |
| `ALREADY_EXISTS` | 409 | Resource already exists |
| `PERMISSION_DENIED` | 403 | Authenticated but not authorized |
| `UNAUTHENTICATED` | 401 | Missing or invalid credentials |
| `RESOURCE_EXHAUSTED` | 429 | Rate limited or quota exceeded |
| `INTERNAL` | 500 | Server-side error |
| `UNAVAILABLE` | 503 | Service temporarily down |
| `DEADLINE_EXCEEDED` | 504 | Timeout exceeded |

For rich error details, use `google.rpc.Status` with error details:

```protobuf
import "google/rpc/status.proto";
import "google/rpc/error_details.proto";
```

## Python gRPC

```bash
pip install grpcio grpcio-tools

# Generate Python code from proto
python -m grpc_tools.protoc \
  -I protos \
  --python_out=gen/ \
  --grpc_python_out=gen/ \
  protos/users/v1/service.proto
```

```python
import grpc
from gen.users.v1 import service_pb2, service_pb2_grpc

# Server implementation
class UserServiceServicer(service_pb2_grpc.UserServiceServicer):
    def GetUser(self, request, context):
        user = self._users.get(request.user_id)
        if user is None:
            context.abort(grpc.StatusCode.NOT_FOUND, f"User {request.user_id} not found")
        return service_pb2.User(id=user.id, name=user.name, email=user.email)

# Client
channel = grpc.secure_channel("users-service:443", grpc.ssl_channel_credentials())
stub = service_pb2_grpc.UserServiceStub(channel)
user = stub.GetUser(service_pb2.GetUserRequest(user_id=1))
```

## Go gRPC

```bash
# Install plugins
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest

# Generate Go code
protoc --go_out=. --go-grpc_out=. protos/users/v1/service.proto
```

```go
// Server implementation
type userServer struct {
    usersv1.UnimplementedUserServiceServer
    repo UserRepository
}

func (s *userServer) GetUser(ctx context.Context, req *usersv1.GetUserRequest) (*usersv1.User, error) {
    user, err := s.repo.Get(ctx, req.UserId)
    if err != nil {
        return nil, status.Errorf(codes.NotFound, "user %d not found", req.UserId)
    }
    return &usersv1.User{Id: user.ID, Name: user.Name, Email: user.Email}, nil
}
```

## Security

- **Always use TLS in production**: `grpc.ssl_channel_credentials()` (Python), `credentials.NewTLS(tlsConfig)` (Go).
- **mTLS for service-to-service**: both client and server present certificates. Provides mutual authentication.
- **Interceptors for auth**: check JWTs or service credentials in a gRPC interceptor (equivalent to HTTP middleware).

```go
// Go unary interceptor
func authInterceptor(ctx context.Context, req any, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (any, error) {
    token, err := extractToken(ctx)
    if err != nil {
        return nil, status.Error(codes.Unauthenticated, "invalid token")
    }
    if !validateToken(token) {
        return nil, status.Error(codes.PermissionDenied, "insufficient permissions")
    }
    return handler(ctx, req)
}
```

## Health Checking

Implement the standard gRPC health checking protocol so load balancers and orchestrators can probe readiness:

```protobuf
import "grpc/health/v1/health.proto";
```

```go
import "google.golang.org/grpc/health/grpc_health_v1"

healthServer := health.NewServer()
grpc_health_v1.RegisterHealthServer(grpcServer, healthServer)
healthServer.SetServingStatus("users.v1.UserService", grpc_health_v1.HealthCheckResponse_SERVING)
```

## See Also

- wiki/tier3-working/api-design/rest-conventions.md
- wiki/tier3-working/api-design/openapi.md
- wiki/tier2-core/distributed-systems/overview.md

## Source

gRPC documentation (grpc.io). Protocol Buffers Language Guide (developers.google.com). Google API Design Guide (aip.dev). "gRPC: Up and Running" (Indrasiri & Kuruppu, O'Reilly 2020).
