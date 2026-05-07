# Command-Query Responsibility Segregation (CQRS) (Tier 2)

> **Tier 2** | Source: "Architecture Patterns with Python" Ch 12 (Percival & Gregory) | Authority: established | Derives From: SWEBOK KA03, Greg Young CQRS, Bertrand Meyer CQS

## Summary

Command-Query Responsibility Segregation (CQRS) separates **write operations** (commands that mutate state via the domain model) from **read operations** (queries that return views of current state). The write side goes through the full stack — domain model, service layer, repository, unit of work — to enforce invariants. The read side bypasses the domain model entirely and queries storage directly, returning purpose-built view objects.

The motivation is practical: the ORM representation optimized for writing (aggregate roots, consistency boundaries, lazy-loading relationships) is rarely the right shape for reading. Forcing reads through the aggregate model introduces N+1 queries, unnecessary object construction, and version-lock contention.

## Key Concepts

### The N+1 Read Problem

Fetching an "allocation view" through the ORM:

```python
# The naive ORM approach
def allocations_view(orderid: str, uow: AbstractUnitOfWork):
    with uow:
        products = uow.products.list()     # SELECT * FROM products
        return [
            {"sku": p.sku, "batchref": b.reference}
            for p in products
            for b in p.batches             # SELECT * FROM batches WHERE product_id = ?  (N times)
            for line in b._allocations     # SELECT * FROM allocations WHERE batch_id = ?  (N*M times)
            if line.orderid == orderid
        ]
```

This fires O(N×M) queries for N products with M batches each. The aggregate model was not designed for this query shape.

### Approach 1: Raw SQL Read Model

Bypass the ORM entirely for reads:

```python
# views.py (read side)

from sqlalchemy import text


def allocations(orderid: str, uow: AbstractUnitOfWork):
    with uow:
        results = uow.session.execute(
            text(
                "SELECT ol.sku, b.reference"
                " FROM allocations a"
                " JOIN order_lines ol ON a.orderline_id = ol.id"
                " JOIN batches b ON a.batch_id = b.id"
                " WHERE ol.orderid = :orderid"
            ),
            dict(orderid=orderid),
        )
    return [{"sku": sku, "batchref": batchref} for sku, batchref in results]
```

One SQL query. Returns exactly what the caller needs. The write model's complexity does not leak into the read path.

### Approach 2: Event-Fed Read Model (Fully Decoupled)

Maintain a separate denormalized table updated by event handlers:

```python
# Event handler updates a read table on every Allocated event
def update_read_model_table(event: events.Allocated, uow: AbstractUnitOfWork) -> None:
    with uow:
        uow.session.execute(
            text(
                "INSERT INTO allocations_view (orderid, sku, batchref)"
                " VALUES (:orderid, :sku, :batchref)"
                " ON CONFLICT (orderid, sku) DO UPDATE SET batchref = :batchref"
            ),
            dict(orderid=event.orderid, sku=event.sku, batchref=event.batchref),
        )
        uow.commit()
```

Reads against the view table are trivial:

```python
def allocations(orderid: str, uow: AbstractUnitOfWork):
    with uow:
        results = uow.session.execute(
            text("SELECT sku, batchref FROM allocations_view WHERE orderid = :orderid"),
            dict(orderid=orderid),
        )
    return [{"sku": sku, "batchref": batchref} for sku, batchref in results]
```

This enables: separate read replicas, separate data stores for reads, eventual consistency on the read side.

### When to Apply CQRS

| Situation | Recommendation |
|-----------|---------------|
| Read shape matches write model | Use ORM query directly; CQRS not needed |
| Multiple N+1 hops through aggregate | Raw SQL read model (Approach 1) |
| Read side needs scale, separate DB, or different data store | Event-fed read model (Approach 2) |
| Complex reporting across many aggregates | Event-fed model or dedicated reporting DB |

Start simple. Add CQRS to the read side only when ORM reads become a bottleneck or when the query shape diverges from the write model.

### CQS at the Method Level

Bertrand Meyer's original Command-Query Separation principle applies at the method level too: a method either mutates state (command — returns nothing) or returns a value (query — no side effects). Domain model methods should respect this.

```python
def allocate(self, line: OrderLine) -> str:    # returns batchref — this is OK
    batch = next(...)
    batch.allocate(line)                        # but also mutates state — a pragmatic exception
    return batch.reference
```

The book is pragmatic: returning the `batchref` from `allocate` is acceptable because it is closely coupled to the write. Pure CQRS would have the command return nothing and a separate query retrieve the result.

## Agent Guidance

### Do

- Keep the write side (service layer, domain model, repository) completely separate from the read side (views).
- Use raw SQL queries for read models when ORM traversal would require multiple hops through aggregates.
- Maintain event-fed read model tables when the read side must scale independently or use a different data store.
- Return simple dicts or named tuples from read functions — not domain model objects.

### Do Not

- Do not route read requests through the domain model's aggregate root just to satisfy the pattern; CQRS is a tool, not a law.
- Do not put business invariant enforcement in read-side query code.
- Do not use the same SQLAlchemy `Session` in a way that mixes dirty-write tracking with read queries — reads and writes should use sessions cleanly.
- Do not build a fully separate event store / event sourcing setup unless the problem specifically demands it (that is a much larger commitment than what this book covers).

## Checklist

- [ ] Read functions (`views.py`) do not call domain model methods or trigger aggregate loading
- [ ] Complex reads use raw SQL with parameterized queries (not string formatting)
- [ ] Event-fed read model tables updated via dedicated event handlers
- [ ] Read functions return dicts or named tuples, not ORM model instances
- [ ] Raw SQL queries use `text()` with `dict()` parameters — no string interpolation

## See Also

- wiki/tier2-core/architecture-patterns/commands-vs-events.md
- wiki/tier2-core/architecture-patterns/domain-events-message-bus.md
- wiki/tier3-working/database-patterns/query-optimization.md
- wiki/tier1-sources/owasp/a03-injection.md
- wiki/tier1-sources/swebok-v4/ka03-design.md

## Source

Harry Percival and Bob Gregory, *Architecture Patterns with Python*, Chapter 12 (O'Reilly 2020). CC-BY-ND.
https://www.cosmicpython.com/book/chapter_12_cqrs.html
Greg Young — CQRS Documents (cqrs.nu). Bertrand Meyer — Object-Oriented Software Construction (1988).
