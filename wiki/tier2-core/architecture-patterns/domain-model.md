# Domain Model (Tier 2)

> **Tier 2** | Source: "Architecture Patterns with Python" Ch 1 (Percival & Gregory) | Authority: established | Derives From: SWEBOK KA03, DDD (Evans 2003)

## Summary

The domain model is the heart of the application: it encodes the business rules and vocabulary in plain Python objects with no infrastructure dependencies. When business logic lives in the domain model rather than scattered across controllers, services, or ORM models, it becomes possible to test every business rule in memory — no database, no HTTP, no I/O required.

A domain model built with plain Python dataclasses and functions is the foundation all other patterns in this book stack on top of. Infrastructure adapts to the domain; the domain does not adapt to infrastructure.

## Key Concepts

### Value Objects and Entities

| Concept | Definition | Python Idiom |
|---------|-----------|-------------|
| **Value Object** | Defined purely by its attributes; no identity beyond its values; should be immutable | `@dataclass(frozen=True)` |
| **Entity** | Has a unique identity that persists through attribute changes; equality is identity-based | `@dataclass(eq=False)` + `__eq__` comparing id |
| **Domain Service** | A stateless operation that involves multiple domain objects but belongs to no single one | Plain function or module-level function |

Use value objects by default; only introduce entities when the concept genuinely has an identity that survives mutations (e.g., a customer order has an identity — changing the quantity does not make it a different order; an order line's SKU+qty is a value — two lines with the same SKU+qty are interchangeable).

### Allocation Domain Example

```python
from __future__ import annotations
from dataclasses import dataclass
from datetime import date
from typing import Optional


@dataclass(frozen=True)
class OrderLine:
    """Value object: identity is (orderid, sku, qty)."""
    orderid: str
    sku: str
    qty: int


class Batch:
    """Entity: identity is the batch reference."""

    def __init__(self, ref: str, sku: str, qty: int, eta: Optional[date]) -> None:
        self.reference = ref
        self.sku = sku
        self.eta = eta
        self._purchased_quantity = qty
        self._allocations: set[OrderLine] = set()

    def allocate(self, line: OrderLine) -> None:
        if self.can_allocate(line):
            self._allocations.add(line)

    def deallocate(self, line: OrderLine) -> None:
        if line in self._allocations:
            self._allocations.remove(line)

    @property
    def allocated_quantity(self) -> int:
        return sum(line.qty for line in self._allocations)

    @property
    def available_quantity(self) -> int:
        return self._purchased_quantity - self.allocated_quantity

    def can_allocate(self, line: OrderLine) -> bool:
        return self.sku == line.sku and self.available_quantity >= line.qty

    def __eq__(self, other: object) -> bool:
        if not isinstance(other, Batch):
            return False
        return self.reference == other.reference

    def __hash__(self) -> int:
        return hash(self.reference)

    def __gt__(self, other: Batch) -> bool:
        if self.eta is None:
            return False
        if other.eta is None:
            return True
        return self.eta > other.eta


def allocate(line: OrderLine, batches: list[Batch]) -> str:
    """Domain service: allocates a line to the best available batch."""
    try:
        batch = next(b for b in sorted(batches) if b.can_allocate(line))
    except StopIteration:
        raise OutOfStock(f"Out of stock for sku {line.sku}")
    batch.allocate(line)
    return batch.reference


class OutOfStock(Exception):
    pass
```

### Testing the Domain Model

Domain model tests are fast, self-contained, and express business rules:

```python
from datetime import date


def test_prefers_warehouse_batches_to_shipments():
    in_stock = Batch("in-stock-batch", "RETRO-CLOCK", 100, eta=None)
    shipment = Batch("shipment-batch", "RETRO-CLOCK", 100, eta=date(2025, 6, 1))
    line = OrderLine("oref", "RETRO-CLOCK", 10)

    allocate(line, [in_stock, shipment])

    assert in_stock.available_quantity == 90
    assert shipment.available_quantity == 100


def test_raises_out_of_stock_if_cannot_allocate():
    batch = Batch("batch1", "SMALL-TABLE", 10, eta=date(2025, 6, 1))
    allocate(OrderLine("order1", "SMALL-TABLE", 10), [batch])

    with pytest.raises(OutOfStock, match="SMALL-TABLE"):
        allocate(OrderLine("order2", "SMALL-TABLE", 1), [batch])
```

No mocks. No database. No HTTP. Tests run in microseconds.

### Where Not to Put Business Logic

| Anti-Pattern | Problem |
|-------------|---------|
| Business logic in controllers / views | Untestable without HTTP stack; mixes concerns |
| Business logic in ORM models | Couples domain to database schema; ORM changes break rules |
| Business logic in transaction scripts | Procedural spaghetti; rules scatter; no single authoritative model |
| Anemic domain model (getters/setters only, logic in services) | Domain knowledge disappears into service layer; rules duplicated |

## Agent Guidance

### Do

- Use `@dataclass(frozen=True)` for value objects to enforce immutability.
- Keep domain objects free of any import that touches infrastructure (sqlalchemy, redis, requests, etc.).
- Express business invariants directly in domain methods — if the object cannot be put in an invalid state, the invariant is structurally enforced.
- Write one domain test per business rule; tests are the executable specification.

### Do Not

- Do not import `Session`, `Base`, or any ORM model from domain objects.
- Do not use inheritance to share behavior across entities that are not true subtypes — prefer composition and protocols.
- Do not put validation of external input (HTTP, CLI) in the domain model; see `wiki/tier2-core/architecture-patterns/validation.md`.
- Do not model an entity as a value object just because its attributes are stable — if it has an identity, use an entity.

## Checklist

- [ ] Value objects use `@dataclass(frozen=True)`; entities use explicit `__eq__` on identity field
- [ ] Domain module imports: only stdlib + other domain modules
- [ ] Every business rule has a unit test that runs without I/O
- [ ] `OutOfStock`, `InvalidSku`, and other business exceptions are defined in the domain layer
- [ ] No ORM models, session objects, or HTTP request objects in domain code

## See Also

- wiki/tier2-core/architecture-patterns/overview.md
- wiki/tier2-core/architecture-patterns/repository.md
- wiki/tier2-core/architecture-patterns/aggregates.md
- wiki/tier2-core/solid-principles/srp.md
- wiki/tier1-sources/swebok-v4/ka03-design.md

## Source

Harry Percival and Bob Gregory, *Architecture Patterns with Python*, Chapter 1 (O'Reilly 2020). CC-BY-ND.
https://www.cosmicpython.com/book/chapter_01_domain_model.html
