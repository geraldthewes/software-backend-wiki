# Architecture Patterns with Python — Overview (Tier 2)

> **Tier 2** | Source: "Architecture Patterns with Python" (Percival & Gregory, O'Reilly 2020) | Authority: established | Derives From: SWEBOK KA02, KA03, DDD

## Summary

"Architecture Patterns with Python" (Cosmic Python) is the canonical practitioner guide for applying Domain-Driven Design, ports-and-adapters (hexagonal) architecture, and event-driven patterns in Python services. Its central insight is that the Dependency Inversion Principle — dependency arrows always point inward toward the domain, never outward toward infrastructure — is achievable in plain Python without a framework, and the result is a codebase whose business logic is fully unit-testable in memory with no database required.

The book is structured in two parts. **Part 1** (Ch 1–7) builds a clean domain core: domain model → repository → service layer → unit of work → aggregates. Each layer insulates the next from infrastructure. **Part 2** (Ch 8–13) introduces event-driven evolution: domain events → message bus → commands → external integration → CQRS → dependency injection bootstrap. The Epilogue covers migration from an existing "big ball of mud".

## Key Concepts

### The Central Rule

```
Domain Layer        ←  business logic, rich model, Protocol interfaces
     ↑ depends on
Service Layer       ←  orchestration, use cases, commands
     ↑ depends on
Infrastructure      ←  Flask, SQLAlchemy, Redis, SMTP (all adapters)
```

Dependency arrows point **inward**. Infrastructure imports domain; domain never imports infrastructure. This is the Dependency Inversion Principle expressed as an architectural constraint — see `wiki/tier2-core/solid-principles/dip.md` for the principle; this book is its end-to-end application.

### Pattern Catalog

| Pattern | Source Chapter(s) | Wiki Page |
|---------|-------------------|-----------|
| Domain Model | Ch 1 | `wiki/tier2-core/architecture-patterns/domain-model.md` |
| Repository | Ch 2 | `wiki/tier2-core/architecture-patterns/repository.md` |
| Service Layer | Ch 4 | `wiki/tier2-core/architecture-patterns/service-layer.md` |
| Unit of Work | Ch 6 | `wiki/tier2-core/architecture-patterns/unit-of-work.md` |
| Aggregates | Ch 7 | `wiki/tier2-core/architecture-patterns/aggregates.md` |
| Domain Events + Message Bus | Ch 8–9 | `wiki/tier2-core/architecture-patterns/domain-events-message-bus.md` |
| Commands vs Events | Ch 10 | `wiki/tier2-core/architecture-patterns/commands-vs-events.md` |
| Event-Driven Integration | Ch 11 | `wiki/tier2-core/architecture-patterns/event-driven-integration.md` |
| CQRS | Ch 12 | `wiki/tier2-core/architecture-patterns/cqrs.md` |
| Dependency Injection + Bootstrap | Ch 13 | `wiki/tier2-core/architecture-patterns/dependency-injection-bootstrap.md` |
| Ports and Adapters | Cross-cutting | `wiki/tier2-core/architecture-patterns/ports-and-adapters.md` |
| Validation placement | Appendix E | `wiki/tier2-core/architecture-patterns/validation.md` |
| Legacy Migration | Epilogue | `wiki/tier2-core/architecture-patterns/legacy-migration.md` |

### On Coupling and Abstractions (Ch 3 Interlude)

A good abstraction is one that simplifies the calling code's concern — it hides a moving part that the caller does not need to know about. The chapter uses a file-sync kata to show that:

- A pure-function "generate instructions" layer that takes a pair of file-system snapshots and returns instructions is fully testable without touching the filesystem.
- An "apply instructions" layer that executes the instructions against real paths can be tested with a handful of integration tests.

The lesson: **identify what can be made pure (deterministic, no side effects) and push I/O to the boundary**. This is the Functional Core / Imperative Shell pattern — see `wiki/tier3-working/python/functional-core.md`.

### Recommended Project Structure (Appendix B)

```
project/
  src/
    myapp/
      domain/          # Pure Python; no infrastructure imports
        model.py
        events.py
      service_layer/
        handlers.py
        unit_of_work.py
        messagebus.py
      adapters/
        repository.py  # SQLAlchemy implementations
        orm.py         # Classical ORM mapping
        redis_eventpublisher.py
      entrypoints/
        flask_app.py
        redis_eventconsumer.py
      config.py
      bootstrap.py     # Composition root
  tests/
    unit/              # Domain-only tests; no DB
    integration/       # Repository + UoW tests against real DB
    e2e/               # Full stack via HTTP
```

The `src/` layout (PEP 517/518) prevents accidental imports of un-installed source. Tests mirror the `src/myapp/` tree.

### Infrastructure Swap: The Abstraction Payoff

Appendix C re-implements the full allocation service using CSV files instead of SQLAlchemy; the service layer is unchanged. Appendix D ports it to Django ORM with minimal service-layer edits. These demonstrate concretely that the patterns deliver on their promise: swap the infrastructure adapter without touching business logic.

## Agent Guidance

### Do

- Read `ports-and-adapters.md` first when starting a new domain-driven service — it provides the architectural frame for all other patterns.
- Use the sub-pages (domain-model, repository, service-layer, unit-of-work, aggregates) in Part 1 order when building a new service from scratch.
- Treat the message bus and UoW as a pair — neither is useful without the other for event-driven work.
- Use the `tests/unit/` → `tests/integration/` → `tests/e2e/` split from the recommended project structure; consult `wiki/tier2-core/testing-strategies/test-pyramid.md` for ratios.

### Do Not

- Do not import SQLAlchemy models, database sessions, or HTTP request/response objects from domain model or service layer modules.
- Do not assume CQRS is needed for all services — start with the simple read path (plain ORM query in the service layer) and only introduce a read model when the SELECT N+1 or data-shape mismatch forces it.
- Do not conflate Command with Event — the distinction matters for error handling and replay semantics; consult `commands-vs-events.md`.
- Do not treat these patterns as all-or-nothing; the book explicitly recommends incremental adoption — see `legacy-migration.md`.

## Checklist

- [ ] Dependency arrows verified: domain layer imports no infrastructure modules
- [ ] Repository interface defined as a `Protocol` in the domain layer
- [ ] Service layer functions are thin orchestration: fetch → domain operation → persist
- [ ] Unit-of-work wraps each service operation as an atomic transaction
- [ ] Aggregate root is the only entry point for mutations within its consistency boundary
- [ ] Pattern choices reviewed against test-pyramid.md for the right testing strategy
- [ ] Appropriate sub-page consulted rather than reasoning from this overview alone

## See Also

- wiki/tier2-core/architecture-patterns/ports-and-adapters.md
- wiki/tier2-core/architecture-patterns/domain-model.md
- wiki/tier2-core/solid-principles/dip.md
- wiki/tier1-sources/swebok-v4/ka02-architecture.md
- wiki/tier1-sources/swebok-v4/ka03-design.md
- wiki/tier2-core/testing-strategies/test-pyramid.md
- wiki/tier3-working/python/functional-core.md

## Source

Harry Percival and Bob Gregory, *Architecture Patterns with Python* (O'Reilly 2020). CC-BY-ND.
https://www.cosmicpython.com/ | https://github.com/cosmicpython/book
Reference summary: `references/architecture-patterns-with-python.md`
