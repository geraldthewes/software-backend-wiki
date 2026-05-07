# Unit of Work Pattern (Tier 2)

> **Tier 2** | Source: "Architecture Patterns with Python" Ch 6 (Percival & Gregory) | Authority: established | Derives From: SWEBOK KA03, Martin Fowler PoEAA

## Summary

The Unit of Work (UoW) pattern encapsulates a database transaction as a context manager. It is the single object the service layer interacts with for persistence — the service layer never touches a database session directly. The UoW wraps one or more repositories, provides `commit()` and `rollback()`, and (in the event-driven chapters) collects domain events emitted by aggregates so the message bus can dispatch them after commit.

The UoW is the bridge between the service layer and the database. It replaces the raw SQLAlchemy `Session` as the service layer's persistence handle.

## Key Concepts

### Abstract Unit of Work

```python
# service_layer/unit_of_work.py

import abc
from domain import model


class AbstractUnitOfWork(abc.ABC):
    products: AbstractProductRepository

    def __enter__(self) -> "AbstractUnitOfWork":
        return self

    def __exit__(self, *args: object) -> None:
        self.rollback()

    @abc.abstractmethod
    def commit(self) -> None: ...

    @abc.abstractmethod
    def rollback(self) -> None: ...

    def collect_new_events(self):
        for product in self.products.seen:
            while product.events:
                yield product.events.pop(0)
```

The `__exit__` auto-rollbacks on any exception, which is the safe default. `commit()` must be called explicitly by the service layer.

### SQLAlchemy Implementation

```python
# service_layer/unit_of_work.py (continued)

from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, Session

DEFAULT_SESSION_FACTORY = sessionmaker(bind=create_engine(config.get_postgres_uri()))


class SqlAlchemyUnitOfWork(AbstractUnitOfWork):
    def __init__(self, session_factory=DEFAULT_SESSION_FACTORY) -> None:
        self.session_factory = session_factory

    def __enter__(self) -> "SqlAlchemyUnitOfWork":
        self.session: Session = self.session_factory()
        self.products = SqlAlchemyProductRepository(self.session)
        return super().__enter__()

    def __exit__(self, *args: object) -> None:
        super().__exit__(*args)
        self.session.close()

    def commit(self) -> None:
        self.session.commit()

    def rollback(self) -> None:
        self.session.rollback()
```

### Fake Unit of Work for Tests

```python
class FakeUnitOfWork(AbstractUnitOfWork):
    def __init__(self) -> None:
        self.products = FakeProductRepository([])
        self.committed = False

    def commit(self) -> None:
        self.committed = True

    def rollback(self) -> None:
        pass
```

`committed` lets tests assert that the service function called `commit()`:

```python
def test_add_batch_saves_on_commit():
    uow = FakeUnitOfWork()
    services.add_batch("b1", "CRUNCHY-ARMCHAIR", 100, None, uow)
    assert uow.committed
```

### Repository Tracking with `seen`

For event dispatch, repositories track every aggregate they hand out:

```python
class AbstractProductRepository(Protocol):
    seen: set[Product]

    def add(self, product: Product) -> None: ...
    def get(self, sku: str) -> Product | None: ...


class SqlAlchemyProductRepository:
    def __init__(self, session: Session) -> None:
        self._session = session
        self.seen: set[Product] = set()

    def add(self, product: Product) -> None:
        self._session.add(product)
        self.seen.add(product)

    def get(self, sku: str) -> Product | None:
        product = self._session.query(Product).filter_by(sku=sku).first()
        if product:
            self.seen.add(product)
        return product
```

After `commit()`, the message bus calls `uow.collect_new_events()` to drain events from all seen aggregates and dispatch them.

### Transaction Scope

One `with uow:` block = one database transaction = one unit of work:

```python
def allocate(orderid: str, sku: str, qty: int, uow: AbstractUnitOfWork) -> str:
    with uow:
        product = uow.products.get(sku)          # begins transaction implicitly
        if product is None:
            raise InvalidSku(f"Invalid sku {sku}")
        batchref = product.allocate(OrderLine(orderid, sku, qty))
        uow.commit()                              # commits and closes
    return batchref                               # outside context: no active session
```

If `product.allocate` raises a domain exception, `__exit__` rolls back automatically.

## Agent Guidance

### Do

- Use `with uow:` for every service operation — even read-only ones, so that session lifecycle is consistent.
- Call `uow.commit()` explicitly at the end of successful service operations, never deep inside domain methods.
- Use `FakeUnitOfWork` in all service-layer unit tests; never create a real `Session` in unit tests.
- Let `__exit__` handle rollback on exception — never write `except: uow.rollback()` manually.

### Do Not

- Do not pass `Session` objects to service layer functions — pass the UoW.
- Do not call `uow.commit()` more than once per `with` block; use a single commit per transaction boundary.
- Do not hold a UoW context open across HTTP requests or long-running background jobs.
- Do not access `uow.session` directly from service layer code — that breaks the abstraction.

## Checklist

- [ ] Service layer accepts `AbstractUnitOfWork`, not `Session`
- [ ] `FakeUnitOfWork` in unit tests; `SqlAlchemyUnitOfWork` in integration/E2E
- [ ] `uow.commit()` called explicitly before exiting `with` block on success
- [ ] `__exit__` auto-rollback verified — no explicit `except: rollback()` needed
- [ ] Repository `.seen` set populated for event collection

## See Also

- wiki/tier2-core/architecture-patterns/service-layer.md
- wiki/tier2-core/architecture-patterns/repository.md
- wiki/tier2-core/architecture-patterns/domain-events-message-bus.md
- wiki/tier2-core/architecture-patterns/aggregates.md
- wiki/tier3-working/worked-examples/repository-pattern.md

## Source

Harry Percival and Bob Gregory, *Architecture Patterns with Python*, Chapter 6 (O'Reilly 2020). CC-BY-ND.
https://www.cosmicpython.com/book/chapter_06_uow.html
Martin Fowler, *Patterns of Enterprise Application Architecture*, Unit of Work pattern (2002).
