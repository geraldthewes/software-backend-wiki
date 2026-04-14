# REST API Conventions

> **Tier 3** | Source: RFC 7807, REST architectural style | Enforces/Derives From: wiki/tier1-sources/owasp/a01-broken-access-control.md, wiki/tier1-sources/owasp/a03-injection.md

## Summary

REST conventions are the shared vocabulary that makes APIs intuitive. Deviating from them forces clients to read documentation for things that should be predictable. This page documents the rules that apply to every REST API an agent designs or reviews.

## Resource Naming

Resources are nouns, not verbs. Collections are plural. Sub-resources are hierarchical.

```
# Collections
GET /users             — list all users
GET /orders            — list all orders

# Single resource
GET    /users/{id}     — get user by id
PUT    /users/{id}     — replace user
PATCH  /users/{id}     — partial update
DELETE /users/{id}     — delete user

# Sub-resources (hierarchical)
GET    /users/{id}/orders           — orders belonging to a user
POST   /users/{id}/orders           — create an order for a user
GET    /users/{id}/orders/{oid}     — specific order for a user

# Actions that don't fit CRUD — use a sub-resource noun
POST /orders/{id}/cancellation      — cancel an order (not /cancelOrder)
POST /accounts/{id}/password-reset  — trigger password reset
```

Never use verbs in URLs:

| Wrong | Right |
|-------|-------|
| `GET /getUsers` | `GET /users` |
| `POST /createOrder` | `POST /orders` |
| `PUT /updateUser/1` | `PUT /users/1` |
| `DELETE /deleteUser/1` | `DELETE /users/1` |

## HTTP Methods

| Method | Semantics | Idempotent | Safe |
|--------|-----------|-----------|------|
| GET | Read | Yes | Yes |
| POST | Create | No | No |
| PUT | Replace / Upsert | Yes | No |
| PATCH | Partial update | Not necessarily | No |
| DELETE | Delete | Yes | No |

- **Safe**: does not change server state.
- **Idempotent**: calling the same request N times has the same effect as calling it once.

```
# Idempotent: calling PUT /users/1 multiple times with the same body is safe
PUT /users/1
{"name": "Alice", "email": "alice@example.com"}

# Not idempotent: each POST /orders creates a new order
POST /orders
{"product_id": 42, "quantity": 1}
```

## HTTP Status Codes

Use the right status code. Never return `200 OK` with an error body.

### Success Codes

| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 OK | Success | GET, PUT, PATCH — returns a body |
| 201 Created | Resource created | POST that creates a resource |
| 202 Accepted | Async processing started | When work is deferred to a queue |
| 204 No Content | Success, no body | DELETE, PUT when body not needed |

### Client Error Codes (4xx)

| Code | Meaning | When to Use |
|------|---------|-------------|
| 400 Bad Request | Malformed request or validation failure | Missing fields, wrong types |
| 401 Unauthorized | Missing or invalid authentication | No token, expired token |
| 403 Forbidden | Authenticated but not authorized | Valid token but insufficient permission |
| 404 Not Found | Resource does not exist | Invalid ID in path |
| 409 Conflict | State conflict | Duplicate email, optimistic locking failure |
| 422 Unprocessable Entity | Semantically invalid (valid JSON, invalid business logic) | Email format correct but already taken |
| 429 Too Many Requests | Rate limited | Include `Retry-After` header |

### Server Error Codes (5xx)

| Code | Meaning |
|------|---------|
| 500 Internal Server Error | Unexpected server failure |
| 502 Bad Gateway | Upstream service returned invalid response |
| 503 Service Unavailable | Server temporarily unable to handle requests (include `Retry-After`) |

## Error Response Format — RFC 7807 Problem Details

All error responses must use a consistent format. RFC 7807 is the standard:

```json
{
  "type": "https://api.example.com/errors/validation",
  "title": "Validation Failed",
  "status": 400,
  "detail": "The 'email' field is required and must be a valid email address.",
  "instance": "/users",
  "errors": [
    {"field": "email", "message": "required"},
    {"field": "name", "message": "must be between 1 and 100 characters"}
  ]
}
```

Fields:
- `type`: URI identifying the error type (stable, documentable).
- `title`: human-readable summary of the error type.
- `status`: HTTP status code.
- `detail`: human-readable explanation of the specific occurrence.
- `instance`: the URI of the request that triggered the error.

## Versioning Strategies

| Strategy | Example | Pros | Cons |
|----------|---------|------|------|
| **URL path** | `/v1/users` | Explicit, cacheable, easy to test | URL changes |
| Query param | `/users?version=1` | No URL change | Easy to omit, less explicit |
| Accept header | `Accept: application/vnd.api+json;version=1` | Semantically correct | Complex for clients |

**URL path versioning is the default.** It is explicit, cacheable by CDNs, and easy to test in a browser.

Rules:
- Only increment the version on breaking changes.
- A breaking change: removing a field, changing a field type, removing an endpoint, changing semantics of a status code.
- Additive changes (new optional fields, new endpoints) do not require a version bump.
- Maintain `v(n-1)` for at least 6 months after releasing `v(n)`.

## Pagination

### Page + Size (simple)

```
GET /users?page=2&page_size=20

Response:
{
  "data": [...],
  "pagination": {
    "page": 2,
    "page_size": 20,
    "total": 1500,
    "total_pages": 75
  }
}
```

### Cursor-Based (recommended for large datasets)

```
GET /users?limit=20
GET /users?limit=20&cursor=eyJpZCI6MTAwfQ==

Response:
{
  "data": [...],
  "pagination": {
    "limit": 20,
    "next_cursor": "eyJpZCI6MTIwfQ==",
    "has_more": true
  }
}
```

Cursor-based pagination is O(log N) per page (index seek) and does not suffer from the "page drift" problem where inserting new records shifts results across pages.

## Filtering and Sorting

```
# Filtering
GET /users?status=active&role=admin

# Sorting (field:direction)
GET /users?sort=created_at:desc

# Combining
GET /orders?status=pending&sort=created_at:asc&limit=20
```

## Request and Response Design

- **JSON field names**: `snake_case` (consistent with Python, readable).
- **Timestamps**: ISO 8601 UTC — `"2025-01-15T10:30:00Z"`. Never epoch integers in public APIs.
- **Null vs. absent**: use explicit `null` for absent optional fields; do not omit fields from the response. Consistent structure is easier to consume.
- **Collections**: always return an array under a named key, not as the root — allows adding metadata later without breaking clients.

```json
{"data": [...], "pagination": {...}}   // good — extensible
[...]                                  // bad — cannot add metadata
```

## Idempotency Keys for Safe POST Retries

```http
POST /orders
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json

{"product_id": 42, "quantity": 1}
```

The server stores the response keyed by the idempotency key. If the same key is sent again (retry after network failure), return the stored response instead of processing twice. Key should expire after 24 hours.

## Rate Limiting

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 60
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1735900800

{"type": "...", "title": "Rate Limit Exceeded", "status": 429, "detail": "Retry after 60 seconds."}
```

## See Also

- wiki/tier3-working/api-design/openapi.md
- wiki/tier3-working/api-design/grpc.md
- wiki/tier1-sources/owasp/a01-broken-access-control.md
- wiki/tier1-sources/owasp/a03-injection.md

## Source

RFC 7807 — Problem Details for HTTP APIs. "RESTful Web APIs" (Richardson & Amundsen, O'Reilly 2013). Fielding dissertation (2000). Stripe API design guide.
