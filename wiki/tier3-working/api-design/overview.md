# API Design — Overview

> **Tier 3** | Source: REST, gRPC, OpenAPI specifications | Enforces/Derives From: wiki/tier1-sources/owasp/a01-broken-access-control.md, wiki/tier1-sources/owasp/a03-injection.md, wiki/tier1-sources/swebok-v4/ka02-architecture.md

## Summary

APIs are the contracts between services and between services and clients. A well-designed API is stable, secure, and intuitive. A poorly designed one breaks clients on every change and creates security vulnerabilities. This page guides the choice of API style and links to detailed sub-pages.

## API as a Product Mindset

An API is a product consumed by other engineers — internal teams, partners, or the public. Design it with the same care as a user-facing feature:

- **Contract-first**: write the specification before the implementation. The spec is the source of truth that aligns client and server teams.
- **Stability**: breaking changes have a cost that propagates to all consumers. Version the API explicitly.
- **Security by design**: access control and input validation are not afterthoughts. They are the first things specified (OWASP A01, A03).

## When to Use REST vs. gRPC vs. GraphQL

| Use Case | Recommended Style | Reason |
|----------|------------------|--------|
| Public-facing web/mobile API | REST + OpenAPI | Wide client support, human-readable, cacheable |
| Internal service-to-service | gRPC | Binary protocol (smaller payload), strongly typed, generated clients |
| Client-driven flexible queries | GraphQL | Clients specify exactly what data they need |
| Simple webhooks and callbacks | REST (POST) | Simplest option for event delivery |
| Streaming or bidirectional | gRPC | Native streaming support |
| High-throughput microservice mesh | gRPC | Efficient binary serialization |

## Contract-First vs. Code-First

| Approach | How | When to Use |
|----------|-----|-------------|
| **Contract-first** | Write OpenAPI spec or `.proto` file first; generate stubs | Multiple teams, public APIs, anywhere correctness matters |
| **Code-first** | Write code; generate spec from annotations | Solo projects, rapid prototypes, internal tools |

Prefer contract-first for any API that multiple teams will consume. The spec becomes a shared contract that prevents integration surprises.

## Quick Decision Table

| Requirement | REST | gRPC | GraphQL |
|-------------|------|------|---------|
| Browser clients | Yes | No (needs grpc-web) | Yes |
| Streaming | Limited (SSE) | Yes (native) | Yes (subscriptions) |
| Strong typing | Via OpenAPI | Yes (protobuf) | Yes (schema) |
| Caching | Yes (HTTP cache) | No (binary) | Harder |
| Human readability | Yes | No | Yes |
| Code generation | Yes (openapi-generator) | Yes (protoc) | Yes |
| Internal only | OK | Preferred | OK |

## Sub-Pages

| Page | What It Covers |
|------|----------------|
| wiki/tier3-working/api-design/rest-conventions.md | Resource naming, HTTP methods, status codes, versioning, error format |
| wiki/tier3-working/api-design/openapi.md | Contract-first design, OpenAPI 3.1, code generation, validation |
| wiki/tier3-working/api-design/grpc.md | Protobuf, service patterns, error model, security, health checking |

## See Also

- wiki/tier3-working/api-design/rest-conventions.md
- wiki/tier3-working/api-design/openapi.md
- wiki/tier3-working/api-design/grpc.md
- wiki/tier1-sources/owasp/a01-broken-access-control.md
- wiki/tier1-sources/owasp/a03-injection.md

## Source

REST architectural style (Fielding, 2000). gRPC documentation (grpc.io). OpenAPI Specification 3.1.0 (spec.openapis.org). OWASP API Security Top 10.
