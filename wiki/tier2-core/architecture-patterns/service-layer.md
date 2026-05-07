# Service Layer (Tier 2)

> **Tier 2** | Source: "Architecture Patterns with Python" Ch 4 (Percival & Gregory) | Authority: established | Derives From: SWEBOK KA03, Martin Fowler PoEAA

## Summary

The service layer (also called the orchestration layer or use-case layer) sits between the entrypoints (Flask, CLI, Redis consumer) and the domain model. Its job is pure orchestration: fetch the required objects from the repository, invoke domain operations on them, and commit the unit of work. Flask becomes a thin adapter that parses the HTTP request and calls a service function; it contains no business logic.

The key benefit: service functions can be tested directly without an HTTP client, and the same function can be called from a web endpoint, a CLI command, or a message bus handler.

## Key Concepts

### Service Function Structure

Each service function represents a single use case:

```python
# service_layer/services.py

from domain import model, events
from service_layer.unit_of_work import AbstractUnitOfWork


def allocate(
    orderid: str,
    sku: str,
    qty: int,
    uow: AbstractUnitOfWork,
) -> str:
    """Use case: allocate an order line to a batch."""
    with uow:
        product = uow.products.get(sku)
        if product is None:
            raise InvalidSku(f"Invalid sku {sku}")
        batchref = product.allocate(model.OrderLine(orderid, sku, qty))
        uow.commit()
    return batchref


def add_batch(
    ref: str,
    sku: str,
    qty: int,
    eta: Optional[date],
    uow: AbstractUnitOfWork,
) -> None:
    """Use case: record a new batch of stock."""
    with uow:
        product = uow.products.get(sku)
        if product is None:
            product = model.Product(sku=sku, batches=[])
            uow.products.add(product)
        product.batches.append(model.Batch(ref, sku, qty, eta))
        uow.commit()


class InvalidSku(Exception):
    pass
```

The pattern for every service function:

1. Open the unit of work (`with uow:`)
2. Retrieve aggregate(s) via repository (`uow.products.get(...)`)
3. Invoke domain operation (`product.allocate(line)`)
4. Commit (`uow.commit()`)
5. Return a primitive value or raise a domain exception

### Flask as a Thin Adapter

```python
# entrypoints/flask_app.py

from flask import Flask, jsonify, request
from service_layer import services
from service_layer.unit_of_work import SqlAlchemyUnitOfWork

app = Flask(__name__)


@app.route("/allocate", methods=["POST"])
def allocate_endpoint():
    try:
        batchref = services.allocate(
            orderid=request.json["orderid"],
            sku=request.json["sku"],
            qty=request.json["qty"],
            uow=SqlAlchemyUnitOfWork(),
        )
    except (model.OutOfStock, services.InvalidSku) as e:
        return jsonify({"message": str(e)}), 400
    return jsonify({"batchref": batchref}), 201
```

Flask only: parses the request, calls the service, formats the response. No business logic.

### Testing the Service Layer Directly

```python
# tests/unit/test_services.py

def test_allocates_to_a_batch():
    uow = FakeUnitOfWork()
    services.add_batch("b1", "COMPLICATED-LAMP", 100, None, uow)

    result = services.allocate("o1", "COMPLICATED-LAMP", 10, uow)

    assert result == "b1"


def test_raises_for_invalid_sku():
    uow = FakeUnitOfWork()
    services.add_batch("b1", "AREALSKU", 100, None, uow)

    with pytest.raises(services.InvalidSku, match="NONEXISTENTSKU"):
        services.allocate("o1", "NONEXISTENTSKU", 10, uow)
```

`FakeUnitOfWork` holds a `FakeBatchRepository`. No database, no Flask test client required.

### What Belongs in the Service Layer vs the Domain

| Decision | Where it lives |
|----------|---------------|
| Can this batch accommodate this order line? | Domain model (`Batch.can_allocate`) |
| Which batch to pick when multiple are available? | Domain service (`allocate` function in domain) |
| What if the SKU doesn't exist? | Service layer (fetch returns None → raise `InvalidSku`) |
| Parse JSON body, check auth token | Entrypoint (Flask view) |
| Which unit of work to use (real vs fake) | Caller / composition root |

### Service Layer Granularity

One service function = one use case. Use cases correspond roughly to user stories or commands. If two things always happen together, they are one use case. If they can happen independently, they are two service functions.

## Agent Guidance

### Do

- Accept primitive types (`str`, `int`, `date`) as service function arguments — never accept HTTP request objects or ORM models.
- Accept the unit-of-work as a dependency so tests can inject a fake.
- Keep service functions to the fetch-operate-commit pattern; any branching on business rules belongs in the domain.
- Raise domain exceptions from the service layer; let entrypoints translate them to HTTP status codes.

### Do Not

- Do not access `request.json` or `request.headers` from service functions — that is the entrypoint's job.
- Do not commit inside the domain model or repository — the service layer owns the transaction boundary (via the unit of work).
- Do not put service functions in the same module as domain classes — `service_layer/services.py` vs `domain/model.py`.
- Do not grow service functions into orchestrating multiple aggregates in one transaction; each service function should work within a single aggregate's consistency boundary.

## Checklist

- [ ] Service function parameters are primitives, not HTTP or ORM objects
- [ ] Unit of work is injected as a parameter
- [ ] Domain exceptions are raised from service functions; entrypoints catch and translate
- [ ] Service layer functions are tested directly without HTTP stack
- [ ] Flask / CLI entrypoints contain no business logic

## See Also

- wiki/tier2-core/architecture-patterns/unit-of-work.md
- wiki/tier2-core/architecture-patterns/domain-model.md
- wiki/tier2-core/architecture-patterns/ports-and-adapters.md
- wiki/tier2-core/architecture-patterns/commands-vs-events.md
- wiki/tier2-core/testing-strategies/test-pyramid.md

## Source

Harry Percival and Bob Gregory, *Architecture Patterns with Python*, Chapter 4 (O'Reilly 2020). CC-BY-ND.
https://www.cosmicpython.com/book/chapter_04_service_layer.html
Martin Fowler, *Patterns of Enterprise Application Architecture*, Service Layer pattern (2002).
