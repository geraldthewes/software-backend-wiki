# OpenAPI / Contract-First API Design

> **Tier 3** | Source: OpenAPI Specification 3.1.0 | Enforces/Derives From: wiki/tier1-sources/swebok-v4/ka02-architecture.md

## Summary

Contract-first API design writes the OpenAPI specification before any implementation. The spec is the source of truth that aligns teams, drives code generation, and enables automated validation. This prevents the "spec drift" problem where documentation and implementation diverge over time.

## Contract-First Principle

Write the OpenAPI spec before writing server code. The workflow:

1. Define resources, operations, request/response schemas in YAML.
2. Review the spec with client team — agree on the contract before writing code.
3. Generate server stubs from the spec.
4. Implement business logic inside the stubs.
5. Validate responses against the spec in CI.

This prevents the most common API integration failure: client and server teams writing to different mental models of the same contract.

## OpenAPI 3.1 — Spec Structure

```yaml
openapi: "3.1.0"
info:
  title: User Service API
  version: "1.0.0"
  description: Manages user accounts and profiles

servers:
  - url: https://api.example.com/v1
    description: Production
  - url: http://localhost:8000/v1
    description: Local development

paths:
  /users:
    get:
      summary: List users
      operationId: listUsers
      parameters:
        - $ref: "#/components/parameters/PageLimit"
        - $ref: "#/components/parameters/PageCursor"
      responses:
        "200":
          description: Paginated list of users
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/UserList"
        "400":
          $ref: "#/components/responses/BadRequest"
      security:
        - BearerAuth: []

    post:
      summary: Create a user
      operationId: createUser
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/CreateUserRequest"
      responses:
        "201":
          description: User created
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/User"
        "400":
          $ref: "#/components/responses/BadRequest"
        "409":
          $ref: "#/components/responses/Conflict"
      security:
        - BearerAuth: []

  /users/{userId}:
    parameters:
      - name: userId
        in: path
        required: true
        schema:
          type: integer
    get:
      summary: Get a user by ID
      operationId: getUser
      responses:
        "200":
          description: User found
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/User"
        "404":
          $ref: "#/components/responses/NotFound"
      security:
        - BearerAuth: []

components:
  schemas:
    User:
      type: object
      required: [id, name, email, created_at]
      additionalProperties: false
      properties:
        id:
          type: integer
        name:
          type: string
          minLength: 1
          maxLength: 100
        email:
          type: string
          format: email
        created_at:
          type: string
          format: date-time

    CreateUserRequest:
      type: object
      required: [name, email]
      additionalProperties: false
      properties:
        name:
          type: string
          minLength: 1
          maxLength: 100
        email:
          type: string
          format: email

    UserList:
      type: object
      required: [data, pagination]
      properties:
        data:
          type: array
          items:
            $ref: "#/components/schemas/User"
        pagination:
          $ref: "#/components/schemas/Pagination"

    Pagination:
      type: object
      properties:
        next_cursor:
          type: [string, "null"]
        has_more:
          type: boolean

    Problem:
      type: object
      properties:
        type:
          type: string
        title:
          type: string
        status:
          type: integer
        detail:
          type: string

  parameters:
    PageLimit:
      name: limit
      in: query
      schema:
        type: integer
        minimum: 1
        maximum: 100
        default: 20
    PageCursor:
      name: cursor
      in: query
      schema:
        type: string

  responses:
    BadRequest:
      description: Validation error
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Problem"
    NotFound:
      description: Resource not found
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Problem"
    Conflict:
      description: Resource already exists
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/Problem"

  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

## Schema Design Rules

- Use `$ref` to share schemas. Define each model once in `components/schemas`; reference everywhere.
- Always list `required` fields explicitly. Omitted fields default to optional.
- Use `nullable: true` (OpenAPI 3.0) or `type: [string, "null"]` (OpenAPI 3.1) for optional nullable fields.
- Use `additionalProperties: false` on request schemas to reject unknown fields.
- Use `enum` for string constants:
  ```yaml
  status:
    type: string
    enum: [active, inactive, suspended]
  ```

## Code Generation

Generate server stubs and client SDKs from the spec. The spec drives the code — not the other way around.

```bash
# Install openapi-generator
npm install @openapitools/openapi-generator-cli -g

# Generate Python FastAPI server stub
openapi-generator-cli generate \
  -i api/openapi.yaml \
  -g python-fastapi \
  -o generated/server

# Generate TypeScript client
openapi-generator-cli generate \
  -i api/openapi.yaml \
  -g typescript-fetch \
  -o generated/client-ts
```

## Validation in CI

Validate the spec itself and validate that the implementation matches the spec.

```bash
# Lint the spec
npm install -g @stoplight/spectral-cli
spectral lint api/openapi.yaml

# Validate requests and responses against spec during tests
pip install schemathesis
schemathesis run api/openapi.yaml --base-url http://localhost:8000
```

## Mock Server During Development

Run Prism as a mock server from the spec so frontend teams can develop before the backend is built:

```bash
npm install -g @stoplight/prism-cli
prism mock api/openapi.yaml
# Prism now serves mock responses at http://localhost:4010
```

## Python: FastAPI

FastAPI generates OpenAPI from type annotations. Pydantic models become JSON schemas automatically.

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, EmailStr

app = FastAPI(title="User Service", version="1.0.0")

class CreateUserRequest(BaseModel):
    name: str
    email: EmailStr

    model_config = {"extra": "forbid"}   # additionalProperties: false

class UserResponse(BaseModel):
    id: int
    name: str
    email: str
    created_at: str

@app.post("/v1/users", response_model=UserResponse, status_code=201)
async def create_user(body: CreateUserRequest) -> UserResponse:
    # FastAPI validates the request body against CreateUserRequest automatically
    ...
```

Access the generated spec at `/openapi.json` and Swagger UI at `/docs`.

## See Also

- wiki/tier3-working/api-design/rest-conventions.md
- wiki/tier3-working/api-design/grpc.md
- wiki/tier1-sources/swebok-v4/ka02-architecture.md

## Source

OpenAPI Specification 3.1.0 (spec.openapis.org). FastAPI documentation (fastapi.tiangolo.com). Spectral (stoplight.io). Schemathesis (schemathesis.readthedocs.io).
