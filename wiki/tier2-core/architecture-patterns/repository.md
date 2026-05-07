# Repository Pattern (Tier 2)

> **Tier 2** | Source: "Architecture Patterns with Python" Ch 2 (Percival & Gregory) | Authority: established | Derives From: SWEBOK KA03, DDD, Martin Fowler PoEAA

## Summary

The Repository pattern is an abstraction over data storage that presents a collection-like interface to the domain layer. Service layer code calls `repo.get(id)` and `repo.add(obj)` — it has no knowledge of SQL, session objects, or ORM internals. The repository translates between the domain model and the storage mechanism.

This page covers the pattern's conceptual design. For a full Python worked example with Protocol-based typing, in-memory fakes, and test patterns, see `wiki/tier3-working/database-patterns/repository-pattern.md`.

## Key Concepts

### The Repository as a Collection

From the domain's perspective, a repository behaves like an in-memory collection:

```
batch = repo.get("batch-001")    # retrieve by identity
repo.add(new_batch)              # persist a new object
all_batches = repo.list()        # enumerate all
```

There is no `repo.save()` that passes a changed object back — the pattern relies on the Unit of Work (see `unit-of-work.md`) to track and flush changes automatically. The repository manages identity mapping: retrieving the same object twice returns the same Python object.

### Classical (Imperative) ORM Mapping

The book uses SQLAlchemy's **classical mapping** (also called imperative mapping) to keep domain objects free of SQLAlchemy base class inheritance:

```python
# adapters/orm.py

from sqlalchemy import Table, Column, Integer, String, Date, ForeignKey, MetaData
from sqlalchemy.orm import registry

mapper_registry = registry()
metadata = mapper_registry.metadata

order_lines = Table(
    "order_lines",
    metadata,
    Column("id", Integer, primary_key=True, autoincrement=True),
    Column("sku", String(255)),
    Column("qty", Integer),
    Column("orderid", String(255)),
)

batches = Table(
    "batches",
    metadata,
    Column("id", Integer, primary_key=True, autoincrement=True),
    Column("reference", String(255)),
    Column("sku", String(255)),
    Column("_purchased_quantity", Integer),
    Column("eta", Date, nullable=True),
)

allocations = Table(
    "allocations",
    metadata,
    Column("id", Integer, primary_key=True, autoincrement=True),
    Column("orderline_id", Integer, ForeignKey("order_lines.id")),
    Column("batch_id", Integer, ForeignKey("batches.id")),
)


def start_mappers():
    order_lines_mapper = mapper_registry.map_imperatively(OrderLine, order_lines)
    mapper_registry.map_imperatively(
        Batch,
        batches,
        properties={
            "_allocations": relationship(order_lines_mapper, secondary=allocations),
        },
    )
```

The domain classes `OrderLine` and `Batch` remain plain Python — no `Base`, no `Column()` on attributes. SQLAlchemy attaches the mapping at startup via `start_mappers()`.

### Repository Protocol and Implementations

```python
# domain/ports.py (or domain/repository.py)
from typing import Protocol
from .model import Batch


class AbstractBatchRepository(Protocol):
    def add(self, batch: Batch) -> None: ...
    def get(self, reference: str) -> Batch | None: ...
    def list(self) -> list[Batch]: ...
```

```python
# adapters/repository.py
from sqlalchemy.orm import Session
from domain.model import Batch


class SqlAlchemyBatchRepository:
    def __init__(self, session: Session) -> None:
        self._session = session

    def add(self, batch: Batch) -> None:
        self._session.add(batch)

    def get(self, reference: str) -> Batch | None:
        return self._session.query(Batch).filter_by(reference=reference).first()

    def list(self) -> list[Batch]:
        return self._session.query(Batch).all()
```

```python
# For unit tests — no database needed
class FakeBatchRepository:
    def __init__(self, batches: list[Batch]) -> None:
        self._batches = set(batches)

    def add(self, batch: Batch) -> None:
        self._batches.add(batch)

    def get(self, reference: str) -> Batch | None:
        return next((b for b in self._batches if b.reference == reference), None)

    def list(self) -> list[Batch]:
        return list(self._batches)
```

### Infrastructure Swap Payoff

Appendix C of the book replaces `SqlAlchemyBatchRepository` with a `CsvBatchRepository` that reads from CSV files — the service layer is unchanged. Appendix D ports to Django ORM. The domain model and service functions require zero edits. This is the concrete proof that the repository abstraction delivers on its promise.

## Agent Guidance

### Do

- Define the repository `Protocol` in the domain layer; place concrete implementations in `adapters/repository.py`.
- Write a `Fake*Repository` (in-memory `set` or `dict`) for unit testing service layer functions.
- Keep repositories thin: only `add`, `get`, `list`, and optionally `remove`. No query logic beyond identity lookup.
- Use classical/imperative SQLAlchemy mapping to keep domain objects free of ORM base class pollution.

### Do Not

- Do not pass `Session` objects into the domain layer or service layer functions — the Unit of Work owns the session.
- Do not add business logic (filtering, sorting by domain rules) to repository methods; that belongs in the domain or service layer.
- Do not mix `SELECT` queries for multiple aggregate types into one repository; each aggregate type gets its own repository.
- Do not use SQLAlchemy's declarative `Base` on domain objects — it imports ORM state into the domain.

## Checklist

- [ ] Repository `Protocol` is defined in the domain or service layer (not in `adapters/`)
- [ ] `SqlAlchemy*Repository` lives in `adapters/repository.py`
- [ ] `Fake*Repository` exists for each repository used in unit tests
- [ ] No `Session` objects flow into service layer function signatures
- [ ] Classical mapping used — no `Base` subclass on domain objects

## See Also

- wiki/tier2-core/architecture-patterns/unit-of-work.md
- wiki/tier2-core/architecture-patterns/service-layer.md
- wiki/tier2-core/architecture-patterns/ports-and-adapters.md
- wiki/tier3-working/database-patterns/repository-pattern.md
- wiki/tier3-working/worked-examples/repository-pattern.md
- wiki/tier2-core/solid-principles/dip.md

## Source

Harry Percival and Bob Gregory, *Architecture Patterns with Python*, Chapter 2 (O'Reilly 2020). CC-BY-ND.
https://www.cosmicpython.com/book/chapter_02_repository.html
Martin Fowler, *Patterns of Enterprise Application Architecture* (Addison-Wesley 2002), Repository pattern.
