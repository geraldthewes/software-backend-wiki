# Functional Core / Imperative Shell

> **Tier 3** | Source: Gary Bernhardt "Boundaries" (2012) | Enforces/Derives From: wiki/tier1-sources/swebok-v4/ka04-construction.md, wiki/tier2-core/solid-principles/srp.md, wiki/tier2-core/solid-principles/dip.md

## Summary

The Functional Core / Imperative Shell pattern separates pure business logic from I/O. The result is business logic that is trivially testable without mocks, and I/O that is confined to a thin shell that is hard to unit-test but easy to integration-test.

## Origin

Gary Bernhardt introduced this pattern in his talk "Boundaries" (Strange Loop, 2012). The core insight: I/O (network, disk, database) makes code hard to test. Pure functions (no I/O, no side effects) make code easy to test. Therefore, push all I/O to the outermost boundary of the system.

## The Pattern

### Functional Core

- Pure functions that transform data
- Same inputs always produce same outputs
- No I/O, no side effects, no global state mutation
- No calls to databases, clocks, random number generators, or external services
- Testable with simple `assert` statements — no mocks needed

### Imperative Shell

- Thin orchestration layer
- Reads from I/O sources (DB, file, HTTP)
- Passes data to the functional core
- Writes results back to I/O sinks
- Hard to unit-test (requires integration tests), but the logic is trivial

```
┌─────────────────────────────────────────────┐
│                  Shell                       │
│  read from DB  →  call core  →  write to DB │
└────────────────────┬────────────────────────┘
                     │
          ┌──────────▼──────────┐
          │    Functional Core   │
          │  pure transformations│
          │  no I/O              │
          └─────────────────────┘
```

## Pure Functions

A function is pure if:
1. Given the same arguments, it always returns the same result.
2. It does not mutate its arguments.
3. It does not read from or write to any external system.
4. It does not depend on global state.

```python
# PURE — deterministic, no side effects
def calculate_discount(price: float, customer_tier: str) -> float:
    rates = {"standard": 0.0, "premium": 0.10, "vip": 0.20}
    rate = rates.get(customer_tier, 0.0)
    return round(price * (1 - rate), 2)

# IMPURE — reads from DB inside business logic (anti-pattern)
def calculate_discount(price: float, user_id: int) -> float:
    tier = db.query("SELECT tier FROM users WHERE id = ?", user_id)  # I/O!
    rates = {"standard": 0.0, "premium": 0.10, "vip": 0.20}
    return round(price * (1 - rates.get(tier, 0.0)), 2)
```

## How to Implement

1. Identify business logic that mixes computation with I/O.
2. Extract all decision-making into pure functions that accept data as arguments.
3. Create immutable domain objects with `@dataclass(frozen=True)`.
4. Move I/O (DB reads, API calls, file operations) into the shell.
5. Shell: read data → call pure function → write result.

## Python Example

```python
from dataclasses import dataclass
from typing import Protocol
import logging

logger = logging.getLogger(__name__)


# --- Domain objects (immutable) ---

@dataclass(frozen=True)
class Order:
    order_id: int
    customer_tier: str
    subtotal: float

@dataclass(frozen=True)
class Invoice:
    order_id: int
    subtotal: float
    discount: float
    total: float


# --- Functional Core (pure functions, no I/O) ---

DISCOUNT_RATES: dict[str, float] = {
    "standard": 0.00,
    "premium": 0.10,
    "vip": 0.20,
}

def calculate_discount(order: Order) -> float:
    """Pure: same order always yields same discount."""
    rate = DISCOUNT_RATES.get(order.customer_tier, 0.0)
    return round(order.subtotal * rate, 2)

def build_invoice(order: Order) -> Invoice:
    """Pure: transforms Order into Invoice."""
    discount = calculate_discount(order)
    return Invoice(
        order_id=order.order_id,
        subtotal=order.subtotal,
        discount=discount,
        total=round(order.subtotal - discount, 2),
    )


# --- Repository Protocol (abstraction for I/O) ---

class OrderRepository(Protocol):
    def get(self, order_id: int) -> Order | None: ...

class InvoiceRepository(Protocol):
    def save(self, invoice: Invoice) -> None: ...


# --- Imperative Shell (I/O only, no business logic) ---

def process_order(
    order_id: int,
    orders: OrderRepository,
    invoices: InvoiceRepository,
) -> Invoice | None:
    """Shell: orchestrates I/O around the pure core."""
    order = orders.get(order_id)
    if order is None:
        logger.warning("Order %s not found", order_id)
        return None

    invoice = build_invoice(order)   # pure — no I/O
    invoices.save(invoice)           # I/O only in shell
    logger.info("Invoice created for order %s, total=%.2f", order_id, invoice.total)
    return invoice
```

Testing the core requires no mocks:

```python
def test_vip_discount():
    order = Order(order_id=1, customer_tier="vip", subtotal=100.0)
    invoice = build_invoice(order)
    assert invoice.discount == 20.0
    assert invoice.total == 80.0

def test_unknown_tier_gets_no_discount():
    order = Order(order_id=2, customer_tier="mystery", subtotal=50.0)
    invoice = build_invoice(order)
    assert invoice.discount == 0.0
```

## Benefits

- Business logic is testable without databases, HTTP clients, or mocks.
- I/O is confined to one layer — bugs in I/O don't contaminate logic.
- Pure functions are composable: `build_invoice(calculate_discount(order))`.
- Clear audit trail: shell logs what went in and what came out.

## Relationship to SOLID

- **SRP**: pure core has one responsibility (transform data); shell has one responsibility (orchestrate I/O).
- **DIP**: shell depends on Repository Protocols, not concrete implementations.

## Relationship to 12-Factor

- **Factor VI (Stateless Processes)**: functional core is inherently stateless — no session state, no local disk state.

## Anti-Pattern to Avoid

```python
# BAD: I/O inside computation
def process_order(order_id: int) -> Invoice:
    order = db.query(...)           # I/O
    tax_rate = external_api.get_tax_rate(order.region)  # I/O
    discount = order.subtotal * 0.1  # mixed with I/O above
    total = order.subtotal - discount + tax_rate * order.subtotal
    db.save(Invoice(...))           # I/O
    return Invoice(...)
```

This function is impossible to unit-test without mocking the database and external API. Extract the calculation into a pure function first.

## itertools, functools, and operator in the Functional Core

The Python standard library provides tools specifically designed for functional-style data transformation. These belong in the functional core because they are pure (no I/O, no side effects) and composable.

### itertools — Lazy Iteration

```python
import itertools

# chain — flatten iterables without building intermediate lists
all_events = list(itertools.chain(monday_events, tuesday_events))

# islice — take the first N items from any iterator
first_10 = list(itertools.islice(infinite_stream(), 10))

# groupby — group sorted data by key (input must be sorted by the same key)
from operator import attrgetter
records.sort(key=attrgetter('category'))
for category, items in itertools.groupby(records, key=attrgetter('category')):
    process_group(category, list(items))

# combinations and permutations — combinatoric generators
pairs = list(itertools.combinations(['a', 'b', 'c', 'd'], 2))
# [('a','b'), ('a','c'), ('a','d'), ('b','c'), ('b','d'), ('c','d')]

# takewhile / dropwhile — conditional slicing
valid = list(itertools.takewhile(lambda x: x > 0, [3, 2, 1, -1, 2]))
# [3, 2, 1]

# filterfalse — complement of filter
odd = list(itertools.filterfalse(lambda x: x % 2 == 0, range(10)))
```

### functools — Higher-Order Functions

```python
import functools

# partial — create specialized functions from general ones
from operator import add
add5 = functools.partial(add, 5)
add5(3)   # 8

# reduce — fold a sequence into a single value
total = functools.reduce(lambda acc, x: acc + x, [1, 2, 3, 4], 0)  # 10

# lru_cache — memoize pure functions (safe because they are pure)
@functools.lru_cache(maxsize=None)
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

# cached_property — compute once, cache on instance
from functools import cached_property

class Report:
    def __init__(self, data: list[dict]) -> None:
        self.data = data

    @cached_property
    def summary(self) -> dict:
        """Expensive computation — runs once, cached as instance attribute."""
        return {
            'count': len(self.data),
            'total': sum(r['amount'] for r in self.data),
        }
```

### operator — Functional Operators

```python
from operator import attrgetter, itemgetter, add, mul, eq

# Attribute and item access as callables (faster than lambdas)
get_name = attrgetter('name')
get_first = itemgetter(0)

names = list(map(attrgetter('name'), employees))
sorted_by_salary = sorted(employees, key=attrgetter('salary'))

# Operator functions for functools.reduce
from functools import reduce
product = reduce(mul, [1, 2, 3, 4, 5])   # 120
```

### Composing the Functional Core with These Tools

```python
import itertools, functools
from operator import attrgetter, itemgetter

# PURE: aggregate invoice totals by department (no I/O)
def totals_by_department(invoices: list[Invoice]) -> dict[str, float]:
    sorted_invoices = sorted(invoices, key=attrgetter('department'))
    return {
        dept: functools.reduce(
            lambda acc, inv: acc + inv.total,
            group,
            0.0
        )
        for dept, group in itertools.groupby(sorted_invoices, key=attrgetter('department'))
    }
```

## See Also

- wiki/tier3-working/python/idioms.md
- wiki/tier3-working/python/type-system.md
- wiki/tier2-core/solid-principles/srp.md
- wiki/tier2-core/solid-principles/dip.md
- wiki/tier3-working/worked-examples/dependency-injection.md
- wiki/tier3-working/python/sorting.md

## Source

Gary Bernhardt, "Boundaries," Strange Loop (2012). "Architecture Patterns with Python" (Percival & Gregory, 2020). SWEBOK V4, KA4 Software Construction. Python Functional Programming HOWTO, docs.python.org/3/howto/functional.html.
