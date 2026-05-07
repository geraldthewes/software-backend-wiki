# Validation Placement (Tier 2)

> **Tier 2** | Source: "Architecture Patterns with Python" Appendix E (Percival & Gregory) | Authority: established | Derives From: SWEBOK KA01, SWEBOK KA03, Postel's Law

## Summary

"Where should validation happen?" is a persistent question in layered architectures. The answer depends on *what kind* of validation you are performing. The book distinguishes three kinds, each with a different home:

1. **Syntactic validation** — is the input well-formed? → validate at the entrypoint (HTTP layer, CLI parser)
2. **Semantic validation** — does the input make sense in the domain? → validate in the domain model
3. **Pragmatic validation** — is this operation permissible given current system state? → validate in the service layer or domain

Placing validation in the wrong layer either leaks infrastructure concerns into the domain or duplicates business rules across layers.

## Key Concepts

### Three Kinds of Validation

| Kind | Question | Where | Example |
|------|----------|-------|---------|
| **Syntactic** | Is the input structurally valid? | Entrypoint / schema validation | Is `qty` an integer? Is `sku` a non-empty string? |
| **Semantic** | Does the input have meaning in the domain? | Domain model | Is this SKU a known product? Is qty positive? |
| **Pragmatic** | Is this operation allowed right now? | Service layer + domain | Is there available stock? Is the batch reference unique? |

### Syntactic Validation at the Entrypoint

```python
# entrypoints/flask_app.py

from marshmallow import Schema, fields, ValidationError


class AllocateSchema(Schema):
    orderid = fields.String(required=True)
    sku = fields.String(required=True)
    qty = fields.Integer(required=True, strict=True)


@app.route("/allocate", methods=["POST"])
def allocate_endpoint():
    schema = AllocateSchema()
    try:
        data = schema.load(request.json)
    except ValidationError as e:
        return jsonify(e.messages), 400

    cmd = commands.Allocate(**data)
    results = bus.handle(cmd)
    return jsonify({"batchref": results.pop(0)}), 201
```

The schema ensures the command object receives correctly-typed primitives. No business logic.

### Semantic Validation in the Domain

```python
# domain/model.py

@dataclass(frozen=True)
class OrderLine:
    orderid: str
    sku: str
    qty: int

    def __post_init__(self) -> None:
        if self.qty < 1:
            raise ValueError(f"OrderLine qty must be positive, got {self.qty}")
        if not self.orderid:
            raise ValueError("OrderLine requires a non-empty orderid")
```

`ValueError` raised from the domain is a business rule violation, not an infrastructure concern. It propagates up through the service layer and is caught at the entrypoint.

### Service Layer: Pragmatic Validation

```python
def allocate(orderid: str, sku: str, qty: int, uow: AbstractUnitOfWork) -> str:
    with uow:
        product = uow.products.get(sku)
        if product is None:
            raise InvalidSku(f"Invalid sku {sku}")    # <-- pragmatic: sku exists in the system?
        batchref = product.allocate(OrderLine(orderid, sku, qty))
        uow.commit()
    return batchref
```

`product is None` is not a syntactic error (the string `sku` is valid) nor purely a domain-semantic error (the model can't know what SKUs exist without querying). The service layer is the right place to check current system state.

### Postel's Law and Its Limits

Postel's Law: *"Be conservative in what you send, liberal in what you accept."*

Applied naively, it means accepting loosely-shaped input and coercing it. This makes the system easier to integrate with but harder to debug and reason about. The book recommends being conservative at the entrypoint: validate inputs strictly at the edge so that downstream code can trust what it receives. Lax input acceptance multiplies the internal branches needed to handle edge cases.

### Avoid Validation Duplication

Placing the same check in multiple layers is a maintenance hazard. The rule:

- **Entrypoint** validates structure/type (schema).
- **Domain** validates semantic constraints that are always true regardless of system state.
- **Service layer** validates state-dependent constraints.
- **Nowhere else.**

If a validation rule appears in both the controller and the service layer, consolidate it.

## Agent Guidance

### Do

- Use a schema library (marshmallow, pydantic, or `@dataclass` with `__post_init__`) at the entrypoint to validate structure before creating commands.
- Express domain invariants (qty must be positive) in domain object constructors so they cannot be created in invalid states.
- Raise specific domain exceptions (`InvalidSku`, `OutOfStock`) from service/domain; catch and translate to HTTP codes in the entrypoint.

### Do Not

- Do not re-check syntactic constraints deep in the service or domain layer if the entrypoint already enforces them.
- Do not query the database from domain model `__post_init__` — that is pragmatic validation and belongs in the service layer.
- Do not apply Postel's Law to accept malformed inputs "just in case" — strict edge validation simplifies all downstream code.
- Do not raise `HTTP 422` or `HTTP 400` from the domain layer — domain exceptions should be transport-agnostic.

## Checklist

- [ ] Entrypoint schema validates all structural constraints before creating command objects
- [ ] Domain objects raise `ValueError` or a domain exception for semantic invariants in `__post_init__`
- [ ] Service layer checks state-dependent constraints (existence, availability, uniqueness)
- [ ] HTTP status codes are only set in the entrypoint, not in the domain or service layer
- [ ] No duplicate validation across layers

## See Also

- wiki/tier2-core/architecture-patterns/service-layer.md
- wiki/tier2-core/architecture-patterns/domain-model.md
- wiki/tier2-core/architecture-patterns/ports-and-adapters.md
- wiki/tier1-sources/swebok-v4/ka01-requirements.md
- wiki/tier1-sources/owasp/a03-injection.md

## Source

Harry Percival and Bob Gregory, *Architecture Patterns with Python*, Appendix E (O'Reilly 2020). CC-BY-ND.
https://www.cosmicpython.com/book/appendix_validation.html
