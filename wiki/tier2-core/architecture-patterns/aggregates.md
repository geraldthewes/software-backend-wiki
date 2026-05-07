# Aggregates and Consistency Boundaries (Tier 2)

> **Tier 2** | Source: "Architecture Patterns with Python" Ch 7 (Percival & Gregory) | Authority: established | Derives From: SWEBOK KA03, DDD (Evans 2003)

## Summary

An aggregate is a cluster of domain objects that must change together to maintain a business invariant. One object in the cluster is the **aggregate root** — the only entry point for mutations within the aggregate. External code holds references only to the root; it never directly mutates internal objects.

Aggregates define the unit of strong consistency: all invariants within an aggregate are satisfied at the end of every transaction. Invariants across aggregates can only be eventually consistent. This distinction drives how you design transactions and — later — how you scale to event-driven microservices.

## Key Concepts

### The Problem: Concurrent Allocations

Without aggregates, multiple requests can simultaneously allocate the same stock, violating the "total allocations ≤ available quantity" invariant:

```
Request A: reads 10 available → allocates 10 → writes 0 remaining
Request B: reads 10 available → allocates 10 → writes 0 remaining (WRONG: should fail)
```

Fixing this with a database-row lock on every `Batch` row would require locking an unbounded number of rows. The aggregate pattern identifies the right scope for the lock.

### Aggregate Root: Product

Refactoring `Batch` allocation into `Product`:

```python
from __future__ import annotations
from dataclasses import dataclass, field
from typing import Optional
from datetime import date


class Product:
    """Aggregate root for all batches of a given SKU."""

    def __init__(self, sku: str, batches: list[Batch] | None = None) -> None:
        self.sku = sku
        self.batches: list[Batch] = batches or []
        self.version_number: int = 0          # optimistic concurrency
        self.events: list[object] = []        # domain events (Ch 8)

    def allocate(self, line: OrderLine) -> str:
        try:
            batch = next(b for b in sorted(self.batches) if b.can_allocate(line))
        except StopIteration:
            self.events.append(OutOfStock(line.sku))
            return None
        batch.allocate(line)
        self.version_number += 1
        return batch.reference

    def change_batch_quantity(self, ref: str, qty: int) -> None:
        batch = next(b for b in self.batches if b.reference == ref)
        batch._purchased_quantity = qty
        while batch.available_quantity < 0:
            line = batch.deallocate_one()
            self.events.append(AllocationRequired(line.orderid, line.sku, line.qty))
        self.version_number += 1
```

`Product` is the only way to call `allocate`. No code outside the aggregate touches `Batch._allocations` directly.

### Optimistic Concurrency with `version_number`

```sql
UPDATE products
   SET version_number = :new_version
 WHERE sku = :sku AND version_number = :expected_version;
```

If two concurrent transactions both read `version_number = 5` and try to write `version_number = 6`, only one will succeed — the other's `UPDATE` matches zero rows and must retry. This avoids row-level locks while still enforcing the invariant.

In SQLAlchemy:

```python
# adapters/orm.py
products = Table(
    "products",
    metadata,
    Column("id", Integer, primary_key=True, autoincrement=True),
    Column("sku", String(255)),
    Column("version_number", Integer, server_default="0"),
)

# adapters/repository.py
class SqlAlchemyProductRepository:
    def get(self, sku: str) -> Product | None:
        return (
            self._session.query(Product)
            .filter_by(sku=sku)
            .with_for_update()       # SELECT FOR UPDATE — pessimistic alternative
            .first()
        )
```

The book shows both pessimistic (`SELECT FOR UPDATE`) and optimistic (`version_number` CHECK) approaches. Optimistic is preferred for low-contention workloads.

### Bounded Context

A **bounded context** is a conceptual boundary within which a domain model is internally consistent and terms have a specific meaning. The same concept (e.g., "product") means different things in the Allocation bounded context versus the Purchasing bounded context. Each bounded context has its own model and should not share ORM models across contexts.

Communication across bounded context boundaries happens through events (Part 2 of the book), not direct object references.

| Concept | Inside Allocation BC | Inside Purchasing BC |
|---------|---------------------|---------------------|
| Product | SKU + list of Batches + allocation history | Supplier + purchase orders + cost |
| Batch | Shipment with ETA + available quantity | Line on a PO + lead time |

Aggregate boundaries ≠ bounded context boundaries, but both are tools for managing consistency scope.

### Rules of Thumb

| Rule | Reason |
|------|--------|
| Reference other aggregates by identity (ID), not by Python object reference | Prevents cross-aggregate load-on-demand |
| One aggregate per transaction | Each UoW commit touches exactly one aggregate |
| Keep aggregates small | Large aggregates = long transactions = high contention |
| Aggregates that must be consistent together should be one aggregate | If A's invariants depend on B's state, A and B are one aggregate |

## Agent Guidance

### Do

- Choose the aggregate root by asking: "what is the thing that enforces the invariant?" That is the root.
- Increment `version_number` on every mutation inside the aggregate; check it in the `UPDATE` to detect conflicts.
- Emit domain events from the aggregate (not from the service layer) when something business-significant happens.
- Keep aggregate boundaries small: if an aggregate rarely has concurrent conflicts, it may be too large.

### Do Not

- Do not expose internal objects (e.g., `Batch`) outside the aggregate for mutation — all changes go through the root.
- Do not load or mutate two aggregates in a single transaction; use domain events to coordinate across boundaries.
- Do not put business invariant logic in the repository or service layer — it belongs in the aggregate's methods.
- Do not confuse bounded contexts with aggregates; a bounded context is a team/model boundary, not a transaction boundary.

## Checklist

- [ ] Aggregate root identified; no external code mutates internal objects directly
- [ ] `version_number` (or equivalent) incremented on every write to the aggregate
- [ ] Domain events appended to `self.events` inside the aggregate, not in service layer
- [ ] No cross-aggregate references by Python object; use IDs
- [ ] Single aggregate per unit-of-work transaction

## See Also

- wiki/tier2-core/architecture-patterns/domain-model.md
- wiki/tier2-core/architecture-patterns/domain-events-message-bus.md
- wiki/tier2-core/architecture-patterns/unit-of-work.md
- wiki/tier2-core/distributed-systems/cap-pacelc.md
- wiki/tier1-sources/swebok-v4/ka03-design.md

## Source

Harry Percival and Bob Gregory, *Architecture Patterns with Python*, Chapter 7 (O'Reilly 2020). CC-BY-ND.
https://www.cosmicpython.com/book/chapter_07_aggregate.html
Eric Evans, *Domain-Driven Design* (Addison-Wesley 2003), Chapter 6: Aggregates.
