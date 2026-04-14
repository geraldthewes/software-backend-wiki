# Behavioral Design Patterns

> **Tier 2** | Source: Gang of Four (1994) | Derives From: ka03-design | Authority: established practice

## Summary

Behavioral patterns manage communication, responsibility, and algorithms between objects. They are the most directly relevant to everyday service design — Strategy eliminates `if-elif` dispatch chains, Observer decouples event producers from consumers, and Iterator/generators are built into Python's core language.

---

## 1. Strategy

### Intent
Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from the clients that use it.

### Python Implementation

Define a `Protocol`; inject implementations. This is the primary implementation of OCP (see `solid-principles/ocp.md`).

```python
from typing import Protocol

class DiscountStrategy(Protocol):
    def calculate(self, base_price: float) -> float: ...

class NoDiscount:
    def calculate(self, base_price: float) -> float:
        return base_price

class PremiumDiscount:
    def calculate(self, base_price: float) -> float:
        return base_price * 0.80  # 20% off

class SeasonalDiscount:
    def calculate(self, base_price: float) -> float:
        return base_price * 0.70  # 30% off

class PricingService:
    def __init__(self, discount: DiscountStrategy) -> None:
        self._discount = discount

    def get_final_price(self, base_price: float) -> float:
        return self._discount.calculate(base_price)

# Usage: swap strategy without changing PricingService
service = PricingService(discount=PremiumDiscount())
```

**Python shortcut — callable as strategy**:

```python
from typing import Callable

DiscountFn = Callable[[float], float]

def apply_pricing(base_price: float, discount: DiscountFn) -> float:
    return discount(base_price)

apply_pricing(100.0, lambda p: p * 0.80)
```

### When to Use
- Eliminate `if-elif` chains that dispatch based on a "type" or "mode" parameter
- When different algorithms for the same operation need to be swappable at runtime
- Payment processors, sorting strategies, export formats, discount rules

---

## 2. Observer

### Intent
Define a one-to-many dependency between objects so that when one object changes state, all dependents are notified automatically.

### Python Implementation

A simple list of callbacks is the idiomatic Python implementation.

```python
from typing import Callable
import weakref

class EventEmitter:
    def __init__(self) -> None:
        self._listeners: dict[str, list[Callable]] = {}

    def on(self, event: str, callback: Callable) -> None:
        self._listeners.setdefault(event, []).append(callback)

    def emit(self, event: str, *args, **kwargs) -> None:
        for callback in self._listeners.get(event, []):
            callback(*args, **kwargs)

class OrderService:
    def __init__(self) -> None:
        self.events = EventEmitter()

    def place_order(self, order_id: int, amount: float) -> None:
        # ... save order ...
        self.events.emit("order_placed", order_id=order_id, amount=amount)

service = OrderService()
service.events.on("order_placed", lambda **kw: print(f"Email: order {kw['order_id']} placed"))
service.events.on("order_placed", lambda **kw: print(f"Audit: ${kw['amount']}"))
```

**Memory leak note**: if observers are short-lived objects, use `weakref` to avoid preventing garbage collection:

```python
import weakref

class WeakEventEmitter:
    def __init__(self) -> None:
        self._listeners: list[weakref.ref] = []

    def subscribe(self, callback: Callable) -> None:
        self._listeners.append(weakref.ref(callback))

    def notify(self, *args, **kwargs) -> None:
        alive = []
        for ref in self._listeners:
            cb = ref()
            if cb is not None:
                cb(*args, **kwargs)
                alive.append(ref)
        self._listeners = alive
```

### When to Use
- Decoupling event producers from consumers
- Domain events: `order_placed`, `payment_failed`, `user_registered`
- Async event systems (asyncio event loops)

---

## 3. Command

### Intent
Encapsulate a request as an object, enabling parameterization of clients, queuing, logging of requests, and undoable operations.

### Python Implementation

Use `@dataclass` command objects with a common `execute()` interface.

```python
from dataclasses import dataclass
from typing import Protocol

class Command(Protocol):
    def execute(self) -> None: ...
    def undo(self) -> None: ...

@dataclass
class CreateOrderCommand:
    customer_id: int
    amount: float
    _saved_order_id: int = 0  # set after execution

    def execute(self) -> None:
        # ... save to database; set _saved_order_id
        ...

    def undo(self) -> None:
        # ... delete the order with _saved_order_id
        ...

class CommandHistory:
    def __init__(self) -> None:
        self._history: list[Command] = []

    def execute(self, command: Command) -> None:
        command.execute()
        self._history.append(command)

    def undo_last(self) -> None:
        if self._history:
            self._history.pop().undo()
```

### When to Use
- Undo/redo systems
- Job queues: serialize commands and execute them later
- Audit logging: log every command object before execution
- Transactional scripts: group commands into a transaction

---

## 4. Template Method

### Intent
Define the skeleton of an algorithm in a base class, deferring specific steps to subclasses. Subclasses can override specific steps without changing the overall algorithm structure.

### Python Implementation

Use `ABC` with `@abstractmethod` for mandatory steps; provide hook methods with default implementations for optional steps.

```python
from abc import ABC, abstractmethod

class DataExporter(ABC):
    """Template Method: defines the export algorithm skeleton."""

    def export(self, data: list[dict]) -> bytes:
        """Fixed algorithm; steps are customized by subclasses."""
        validated = self._validate(data)
        transformed = self._transform(validated)
        return self._serialize(transformed)

    def _validate(self, data: list[dict]) -> list[dict]:
        """Hook method: default implementation passes data through."""
        return data

    @abstractmethod
    def _transform(self, data: list[dict]) -> list:
        """Mandatory step: subclass must implement."""
        ...

    @abstractmethod
    def _serialize(self, data: list) -> bytes:
        """Mandatory step: subclass must implement."""
        ...

class CsvExporter(DataExporter):
    def _transform(self, data: list[dict]) -> list:
        return [list(row.values()) for row in data]

    def _serialize(self, data: list) -> bytes:
        import csv, io
        buf = io.StringIO()
        writer = csv.writer(buf)
        writer.writerows(data)
        return buf.getvalue().encode()
```

### When to Use
- When multiple classes share the same algorithm structure but differ in specific steps
- Report generation, data processing pipelines, test fixtures
- Prefer composition (Strategy) over Template Method when the algorithm steps need to vary independently

---

## 5. Chain of Responsibility

### Intent
Pass a request along a chain of handlers. Each handler decides whether to handle the request or pass it to the next handler.

### Python Implementation

A list of handlers with a common `Protocol`; each is tried in sequence.

```python
from typing import Protocol, Optional

class RequestHandler(Protocol):
    def handle(self, request: dict) -> Optional[dict]: ...

class AuthenticationHandler:
    def handle(self, request: dict) -> Optional[dict]:
        if not request.get("token"):
            return {"error": "Unauthorized", "status": 401}
        return None  # pass to next handler

class RateLimitHandler:
    def handle(self, request: dict) -> Optional[dict]:
        if request.get("calls_per_minute", 0) > 100:
            return {"error": "Rate limit exceeded", "status": 429}
        return None

class BusinessLogicHandler:
    def handle(self, request: dict) -> Optional[dict]:
        return {"result": "processed", "status": 200}

def process_request(request: dict, handlers: list[RequestHandler]) -> dict:
    for handler in handlers:
        result = handler.handle(request)
        if result is not None:
            return result
    return {"error": "No handler matched", "status": 500}

pipeline = [AuthenticationHandler(), RateLimitHandler(), BusinessLogicHandler()]
response = process_request({"token": "abc123", "calls_per_minute": 5}, pipeline)
```

This is the **middleware pattern** used by all major Python web frameworks (WSGI middleware, FastAPI dependencies, Django middleware).

### When to Use
- Web middleware (authentication, logging, rate limiting, compression)
- Event handling pipelines
- Command processing with validation stages

---

## 6. Iterator

### Intent
Provide a way to access elements of a collection sequentially without exposing its underlying representation.

### Python Implementation

Implement `__iter__` and `__next__` on the class, or — the preferred Python approach — use **generators** (`yield`).

```python
# Classic Iterator Protocol
class CountDown:
    def __init__(self, start: int) -> None:
        self._current = start

    def __iter__(self):
        return self

    def __next__(self) -> int:
        if self._current < 0:
            raise StopIteration
        value = self._current
        self._current -= 1
        return value

# Preferred: Generator (simpler, idiomatic Python)
def count_down(start: int):
    for i in range(start, -1, -1):
        yield i

# Infinite generator with early termination
def paginate(fetch_page, page_size: int = 100):
    """Generator that lazily fetches pages from a data source."""
    page = 0
    while True:
        results = fetch_page(offset=page * page_size, limit=page_size)
        if not results:
            return
        yield from results
        if len(results) < page_size:
            return
        page += 1
```

### When to Use
- Traversing collections without loading all elements into memory
- Database cursor iteration, file processing, pagination
- **Always prefer generators over explicit `__iter__`/`__next__` in Python** — generators are more readable and less error-prone

---

## 7. State

### Intent
Allow an object to alter its behavior when its internal state changes. The object will appear to change its class.

### Python Implementation

Define a `State` protocol; a `Context` class holds the current state and delegates to it.

```python
from typing import Protocol

class OrderState(Protocol):
    def confirm(self, order: "Order") -> None: ...
    def ship(self, order: "Order") -> None: ...
    def cancel(self, order: "Order") -> None: ...

class PendingState:
    def confirm(self, order: "Order") -> None:
        print("Order confirmed")
        order.state = ConfirmedState()

    def ship(self, order: "Order") -> None:
        raise ValueError("Cannot ship an unconfirmed order")

    def cancel(self, order: "Order") -> None:
        print("Order cancelled")
        order.state = CancelledState()

class ConfirmedState:
    def confirm(self, order: "Order") -> None:
        raise ValueError("Already confirmed")

    def ship(self, order: "Order") -> None:
        print("Order shipped")
        order.state = ShippedState()

    def cancel(self, order: "Order") -> None:
        print("Confirmed order cancelled")
        order.state = CancelledState()

class ShippedState:
    def confirm(self, order): raise ValueError("Already shipped")
    def ship(self, order): raise ValueError("Already shipped")
    def cancel(self, order): raise ValueError("Cannot cancel shipped order")

class CancelledState:
    def confirm(self, order): raise ValueError("Cannot confirm cancelled order")
    def ship(self, order): raise ValueError("Cannot ship cancelled order")
    def cancel(self, order): raise ValueError("Already cancelled")

class Order:
    def __init__(self) -> None:
        self.state: OrderState = PendingState()

    def confirm(self) -> None: self.state.confirm(self)
    def ship(self) -> None: self.state.ship(self)
    def cancel(self) -> None: self.state.cancel(self)
```

### When to Use
- Objects with well-defined states and explicit, constrained transitions between them
- Order workflows, document lifecycle (draft → review → published), connection states
- Prefer State pattern over large `if-elif` blocks that check `self.status`

---

## Agent Guidance

### Do
- Use Strategy to eliminate `if-elif` type dispatch — replace with `Protocol` + injection
- Use generators (`yield`) instead of implementing `__iter__`/`__next__` explicitly
- Use Chain of Responsibility for middleware pipelines and multi-stage validation
- Use State when an object has 3+ distinct states with constrained transitions

### Do Not
- Do not implement Observer without considering memory leaks from long-lived listeners — use `weakref` for ephemeral observers
- Do not use Template Method when Strategy would provide more flexibility — prefer composition over inheritance
- Do not use State for simple flag-based behavior — use it only when transitions are complex or constrained

## Checklist
- [ ] `if-elif` type dispatch is replaced by Strategy protocol
- [ ] Event notification uses callback lists or dedicated event emitters (not inheritance)
- [ ] Generators are used instead of manual `__iter__`/`__next__` implementations
- [ ] State machine classes handle invalid transitions explicitly (raise `ValueError`)
- [ ] Command objects are used when operations need to be queued, logged, or undone

## See Also
- `wiki/tier2-core/design-patterns/overview.md`
- `wiki/tier2-core/design-patterns/creational.md`
- `wiki/tier2-core/design-patterns/structural.md`
- `wiki/tier2-core/solid-principles/ocp.md`
- `wiki/tier1-sources/swebok-v4/ka01-requirements.md`

## Source

Gamma et al., *Design Patterns* (1994). Synthesized from *Software Development Best Practices for Agent* reference document.
