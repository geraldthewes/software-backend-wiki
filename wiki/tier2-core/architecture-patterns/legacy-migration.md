# Migrating a Legacy Codebase (Tier 2)

> **Tier 2** | Source: "Architecture Patterns with Python" Epilogue (Percival & Gregory) | Authority: established | Derives From: SWEBOK KA07, Strangler Fig pattern (Fowler), Big Ball of Mud (Foote & Yoder)

## Summary

The "big ball of mud" is the most common real-world architecture: business logic scattered through Flask views, ORM models, and utility functions; no clear domain model; tests that hit the database for every assertion. The epilogue addresses the practical question: "I understand the patterns — how do I get from here to there without rewriting everything?"

The answer is incremental adoption using the **Strangler Fig** pattern. You do not rewrite; you grow a clean architecture around the existing code, gradually strangling the old with the new.

## Key Concepts

### The Big Ball of Mud

A codebase becomes a ball of mud through incremental hacking under time pressure:

- Business logic in Flask view functions (`@app.route` with 100-line functions)
- Validation mixed with persistence in the same method
- SQLAlchemy models used as both the ORM layer and the domain model
- Service logic duplicated in REST endpoints, Celery tasks, and management commands
- Tests that start a full Flask app and hit a test database for every test — slow and brittle

The symptoms: any change breaks something unexpected; adding a feature requires touching five files; the test suite takes minutes; nobody understands how it all fits together.

### The Strangler Fig

Martin Fowler's **Strangler Fig Application** pattern: a new system grows alongside the old, gradually taking over responsibility, until the old system is gone.

```
Phase 1: Identify one bounded context that is well-defined and testable
  → Build a new service layer + domain model for just that context
  → Keep the old Flask views calling the new service layer
  → Old views become thin adapters

Phase 2: Extract the repository layer
  → Replace direct SQLAlchemy queries in the old views with repository calls
  → Old test DB fixtures replaced by FakeRepository
  → Test speed improves immediately

Phase 3: Introduce the Unit of Work
  → Wrap multi-step operations in a UoW context manager
  → Transaction boundaries become explicit

Phase 4: Aggregate boundaries
  → Identify which entities form a consistency cluster
  → Introduce a root object; remove direct mutation of child objects

Phase 5 (optional): Event-driven
  → Extract side effects (emails, cache invalidation) into event handlers
  → Add a message bus if the service grows complex enough
```

Critically: **at each step the system still runs**. No long-lived refactoring branch. No "big bang" rewrite.

### The Architecture Tax

Applying these patterns has a cost — the "architecture tax": more files, more indirection, more concepts for a new team member to learn. The tax is justified when:

- The codebase is large enough that a ball-of-mud approach creates daily friction.
- Multiple teams need to work independently on the same codebase.
- The business logic is complex enough to warrant a rich domain model.
- The test suite is too slow to run on every commit.

The tax is *not* justified for small services with simple business rules. A Flask app with direct SQLAlchemy queries and no service layer is fine for a 500-line CRUD service. Apply the patterns proportionally to the complexity.

### Django-Specific Considerations (Appendix D)

Django's ORM is tightly integrated — the `Model` class is simultaneously the persistence mapping and the domain object. Decoupling them requires classical mapping, which Django does not natively support. The practical compromise:

- Keep Django models as ORM objects; build a separate domain model layer.
- The service layer accepts and returns domain objects; mappers translate to/from Django ORM models at the boundary.
- Django's `transaction.atomic()` context manager plays the role of the Unit of Work.
- Django's ORM query methods replace the `Session`-based SQLAlchemy repository implementation.

The cost is higher boilerplate; the benefit is the same testability and domain isolation.

### When to Stop

Not every project needs the full stack of patterns. Guidelines for stopping short:

| Pattern | Apply when |
|---------|-----------|
| Domain model | Business logic is non-trivial (more than CRUD) |
| Repository | You need to test the service layer without a database |
| Service layer | Multiple entrypoints (HTTP + CLI + background task) share the same logic |
| Unit of Work | You need transactional guarantees across multiple repository operations |
| Aggregates | Concurrent mutations to the same data cause correctness issues |
| Domain events | Side effects need to decouple from the triggering operation |
| Message bus | Multiple handlers respond to the same event |
| CQRS | ORM reads have N+1 issues or read/write shapes diverge |

## Agent Guidance

### Do

- Start incremental migration with the subdomain where tests are worst: no domain model, direct ORM in controllers.
- Introduce the repository layer first — it immediately improves test speed without changing the architecture otherwise.
- Run the existing test suite at every step; regressions surface immediately.
- Accept the architecture tax only where the complexity justifies it.

### Do Not

- Do not attempt a full rewrite; use the Strangler Fig and keep the system running throughout.
- Do not apply all 13 patterns to a new microservice "just in case" — start with domain model + repository + service layer; add more only as the complexity demands.
- Do not force Django to work identically to the SQLAlchemy-based approach; use `transaction.atomic()` and Django querysets where they fit.
- Do not migrate without tests in place first — write characterization tests for the existing behavior before refactoring.

## Checklist

- [ ] Characterization tests written for existing behavior before starting migration
- [ ] One bounded context identified as the first migration target
- [ ] Repository layer extracted (test speed should improve immediately)
- [ ] Service layer functions callable directly without HTTP stack
- [ ] Architecture tax evaluated: does complexity justify the patterns being added?

## See Also

- wiki/tier2-core/architecture-patterns/overview.md
- wiki/tier2-core/architecture-patterns/ports-and-adapters.md
- wiki/tier1-sources/swebok-v4/ka07-maintenance.md
- wiki/tier2-core/testing-strategies/test-pyramid.md
- wiki/tier1-sources/swebok-v4/ka02-architecture.md

## Source

Harry Percival and Bob Gregory, *Architecture Patterns with Python*, Epilogue (O'Reilly 2020). CC-BY-ND.
https://www.cosmicpython.com/book/epilogue_1_how_to_get_there_from_here.html
Martin Fowler, "Strangler Fig Application" (2004). https://martinfowler.com/bliki/StranglerFigApplication.html
Brian Foote and Joseph Yoder, "Big Ball of Mud" (1999).
