# Architecture Patterns with Python — Reference Summary

**Title**: Architecture Patterns with Python: Enabling Test-Driven Development, Domain-Driven Design, and Event-Driven Microservices
**Authors**: Harry J.W. Percival and Bob Gregory
**Publisher**: O'Reilly Media, 2020
**Print ISBN**: 9781492052203
**License**: Creative Commons CC-BY-ND (https://creativecommons.org/licenses/by-nd/4.0/)
**Free online**: https://www.cosmicpython.com/book/preface.html
**Source repository**: https://github.com/cosmicpython/book
**Companion code**: https://github.com/cosmicpython/code (per-chapter branches)

---

## About the Book

"Architecture Patterns with Python" (commonly called "Cosmic Python") is the canonical practitioner guide for applying Domain-Driven Design (DDD), ports-and-adapters (hexagonal) architecture, and event-driven patterns in Python services. The book grew from the authors' experience building MADE.com's furniture stock allocation platform and was reviewed by Martin Fowler.

The book's central thesis: keeping business logic in a rich domain model, insulated from infrastructure by well-placed abstractions, is achievable in Python without a heavyweight framework — and the result is code that is testable, replaceable, and evolvable. The primary implementing idea is Dependency Inversion (the D in SOLID): all dependency arrows point inward toward the domain, never outward toward infrastructure.

---

## Running Example Domain

All chapters use a single running domain: **MADE.com furniture stock-allocation**. The core objects are:

- `OrderLine`: a line on a customer order (SKU + quantity)
- `Batch`: a shipment of stock arriving from a supplier (SKU + quantity + ETA)
- `Product`: the aggregate root that owns a collection of Batches
- `allocate(line, batches)`: the core domain service that allocates an order line to the best available batch

This deliberate single-domain approach shows how each pattern accretes on the previous one rather than being an isolated technique.

---

## Chapter-to-Wiki-Page Map

| Chapter | Topic | Wiki Page |
|---------|-------|-----------|
| Preface + Introduction | Context and layering philosophy | `wiki/tier2-core/architecture-patterns/overview.md` |
| Ch 1 — Domain Modelling | Plain-Python domain objects | `wiki/tier2-core/architecture-patterns/domain-model.md` |
| Ch 2 — Repository Pattern | Persistence abstraction | `wiki/tier2-core/architecture-patterns/repository.md` |
| Ch 3 — Coupling and Abstractions | Interlude on abstraction design | Folded into `overview.md` and `ports-and-adapters.md` |
| Ch 4 — Service Layer | Orchestration layer | `wiki/tier2-core/architecture-patterns/service-layer.md` |
| Ch 5 — TDD in High Gear and Low Gear | Test layering strategy | Deferred to `wiki/tier2-core/testing-strategies/test-pyramid.md` |
| Ch 6 — Unit of Work | Transaction context manager | `wiki/tier2-core/architecture-patterns/unit-of-work.md` |
| Ch 7 — Aggregates and Consistency Boundaries | Aggregate root, invariants | `wiki/tier2-core/architecture-patterns/aggregates.md` |
| Ch 8 — Domain Events | Events emitted from aggregates | `wiki/tier2-core/architecture-patterns/domain-events-message-bus.md` |
| Ch 9 — Going to Town on the Message Bus | Bus as primary entrypoint | `wiki/tier2-core/architecture-patterns/domain-events-message-bus.md` |
| Ch 10 — Commands and Command Handler | Commands vs Events distinction | `wiki/tier2-core/architecture-patterns/commands-vs-events.md` |
| Ch 11 — Event-Driven Architecture | External integration | `wiki/tier2-core/architecture-patterns/event-driven-integration.md` |
| Ch 12 — CQRS | Read model separation | `wiki/tier2-core/architecture-patterns/cqrs.md` |
| Ch 13 — Dependency Injection and Bootstrapping | Composition root | `wiki/tier2-core/architecture-patterns/dependency-injection-bootstrap.md` |
| Preface, Intro, Ch 2, Ch 4 (cross-cutting) | Hexagonal architecture | `wiki/tier2-core/architecture-patterns/ports-and-adapters.md` |
| Appendix E — Validation | Where validation belongs | `wiki/tier2-core/architecture-patterns/validation.md` |
| Epilogue — How Do I Get There from Here? | Legacy migration | `wiki/tier2-core/architecture-patterns/legacy-migration.md` |
| Appendix B — Project Structure | Folder layout, src/ layout | Folded into `overview.md` |
| Appendix C — CSV Infrastructure Swap | Pattern payoff demo | Noted in `repository.md` |
| Appendix D — Django ORM | Django friction analysis | Noted in `repository.md` |

---

## Key Influences Cited

- Eric Evans, *Domain-Driven Design* (2003) — domain model, aggregate, bounded context
- Vaughn Vernon, *Implementing Domain-Driven Design* (2013)
- Martin Fowler, *Patterns of Enterprise Application Architecture* (2002) — repository, unit of work, service layer, identity map
- Alistair Cockburn — Hexagonal Architecture / Ports and Adapters
- Jeffrey Palermo — Onion Architecture
- Robert C. Martin — Clean Architecture
- Ian Cooper — TDD re-examined (test behavior not implementations)
- Greg Young — CQRS, event sourcing

---

## Licensing Note

The book is CC-BY-ND. Wiki pages are original synthesis in the wiki's own voice; they summarize and cite the book but do not reproduce chapter prose verbatim.
