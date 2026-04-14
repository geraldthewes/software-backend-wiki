# Dependency Inversion Principle (DIP)

> **Tier 2** | Source: Robert C. Martin | Derives From: ka03-design | Authority: established practice

## Summary

High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details; details should depend on abstractions. DIP is the architectural backbone of testable, modular Python services.

## Key Concepts

### Definition — Two Rules

1. **High-level modules should not depend on low-level modules.** Both should depend on abstractions (interfaces / protocols).
2. **Abstractions should not depend on details.** Details (concrete implementations) should depend on abstractions.

Without DIP, the domain layer (business logic) imports infrastructure directly — database connections, HTTP clients, file systems. This makes the domain untestable without a real database and untransferable to a different infrastructure.

### Dependency Injection — The Implementation Technique

DIP is the principle. Dependency injection (DI) is the primary implementation technique. Three forms:

| Form | Description | When to Use |
|------|-------------|-------------|
| **Constructor injection** | Dependency passed via `__init__` | Default; makes dependencies explicit |
| **Factory injection** | A factory callable is injected; the object is created lazily | When the dependency is expensive to create or varies per call |
| **Property injection** | Dependency set after construction | Rare; use only when constructor injection is impossible |

Constructor injection is the preferred form in Python because dependencies are visible in the signature, immutable after construction, and trivially replaced in tests.

### Bad Example — DIP Violation

```python
import psycopg2

class OrderService:
    """High-level module directly instantiates a low-level detail."""

    def __init__(self) -> None:
        self._conn = psycopg2.connect("host=localhost dbname=orders")

    def get_order(self, order_id: int) -> dict:
        cur = self._conn.cursor()
        cur.execute("SELECT * FROM orders WHERE id = %s", (order_id,))
        return cur.fetchone()

    def place_order(self, customer_id: int, amount: float) -> int:
        cur = self._conn.cursor()
        cur.execute(
            "INSERT INTO orders (customer_id, amount) VALUES (%s, %s) RETURNING id",
            (customer_id, amount),
        )
        self._conn.commit()
        return cur.fetchone()[0]
```

`OrderService` is hard-coupled to PostgreSQL. Testing it requires a real database. Swapping to a different database requires editing `OrderService` itself.

### Good Example — DIP Applied

```python
from typing import Protocol
from dataclasses import dataclass

@dataclass
class Order:
    id: int
    customer_id: int
    amount: float


class OrderRepository(Protocol):
    """Abstraction defined in the domain layer — no import of psycopg2."""

    def get(self, order_id: int) -> Order: ...
    def save(self, customer_id: int, amount: float) -> Order: ...


class OrderService:
    """High-level module depends only on the abstraction."""

    def __init__(self, repository: OrderRepository) -> None:
        self._repo = repository

    def get_order(self, order_id: int) -> Order:
        return self._repo.get(order_id)

    def place_order(self, customer_id: int, amount: float) -> Order:
        return self._repo.save(customer_id, amount)


# --- In the infrastructure layer (low-level module): ---
class PostgresOrderRepository:
    """Concrete detail that depends on the abstraction, not the reverse."""

    def __init__(self, connection_string: str) -> None:
        import psycopg2
        self._conn = psycopg2.connect(connection_string)

    def get(self, order_id: int) -> Order:
        ...

    def save(self, customer_id: int, amount: float) -> Order:
        ...


# --- In the composition root (main.py or app factory): ---
def create_app() -> OrderService:
    repo = PostgresOrderRepository(connection_string="host=localhost dbname=orders")
    return OrderService(repository=repo)
```

### Python Layering Pattern

| Layer | Contains | Imports |
|-------|----------|---------|
| **Domain** | Business logic, `Protocol` definitions, `@dataclass` entities | Only standard library and other domain modules |
| **Infrastructure** | Concrete implementations (Postgres, Redis, SMTP) | Domain protocols + third-party libraries |
| **Composition root** | `main.py` or app factory | Both domain and infrastructure; wires them together |

The domain layer never imports infrastructure. Dependency arrows always point inward toward the domain.

### Why This Enables Testing

```python
class FakeOrderRepository:
    """In-memory fake — no database required."""

    def __init__(self) -> None:
        self._store: dict[int, Order] = {}
        self._next_id = 1

    def get(self, order_id: int) -> Order:
        return self._store[order_id]

    def save(self, customer_id: int, amount: float) -> Order:
        order = Order(id=self._next_id, customer_id=customer_id, amount=amount)
        self._store[self._next_id] = order
        self._next_id += 1
        return order


def test_place_order_returns_persisted_order() -> None:
    service = OrderService(repository=FakeOrderRepository())
    order = service.place_order(customer_id=42, amount=99.99)
    assert order.customer_id == 42
    assert order.amount == 99.99
```

No `unittest.mock.patch`, no database fixture, no SMTP server. The test runs in milliseconds.

### Relationship to SRP

DIP often forces SRP: when you extract the abstraction, you realize the class was doing both business logic and infrastructure management. Separating them into `OrderService` + `OrderRepository` applies SRP as a natural consequence of DIP.

## Agent Guidance

### Do
- Define `Protocol` interfaces in the domain layer; place concrete implementations in the infrastructure layer
- Wire everything together in a single composition root (`main.py`, `app.py`, or an app factory)
- Use constructor injection as the default; inject all dependencies at object creation time
- Write a `FakeRepository` or `FakeService` for unit tests — avoid `unittest.mock.patch` where a fake is practical

### Do Not
- Do not call `SomeConcreteService()` or `psycopg2.connect()` inside a domain class `__init__` or method
- Do not import infrastructure modules (database drivers, HTTP clients) from the domain layer
- Do not use `unittest.mock.patch` to patch a concrete dependency when injecting a fake is simpler
- Do not use property or method injection as the default — it hides dependencies

## Checklist
- [ ] Domain layer imports no concrete infrastructure classes
- [ ] All external dependencies are passed via `__init__` parameters typed as `Protocol`
- [ ] A `Fake*` or in-memory implementation exists for each `Protocol` used in domain tests
- [ ] Composition root is the only place that imports both domain and infrastructure
- [ ] Swapping the database requires only changing the composition root, not the domain class

## See Also
- `wiki/tier2-core/solid-principles/isp.md`
- `wiki/tier2-core/solid-principles/srp.md`
- `wiki/tier2-core/solid-principles/overview.md`
- `wiki/tier1-sources/swebok-v4/ka01-requirements.md`

## Source

Robert C. Martin, *Agile Software Development: Principles, Patterns, and Practices* (2002). Synthesized from *Software Development Best Practices for Agent* reference document.
