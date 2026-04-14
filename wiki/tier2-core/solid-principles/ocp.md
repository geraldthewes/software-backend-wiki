# Open/Closed Principle (OCP)

> **Tier 2** | Source: Bertrand Meyer / Robert C. Martin | Derives From: ka03-design | Authority: established practice

## Summary

Software entities (classes, modules, functions) should be **open for extension** but **closed for modification**. Once a module is written and tested, new behavior should be added by writing new code — not by editing existing code. This protects the tested codebase from unintended regressions.

## Key Concepts

### Definition

- **Closed for modification**: existing, tested code is not edited to support new behavior
- **Open for extension**: new behavior is introduced by adding new implementations — new classes, new functions, new modules — without touching the core logic

OCP is a direct consequence of applying SRP and a precondition for LSP: if every new feature forces edits to existing classes, the codebase becomes fragile. If the abstractions are correct, new behavior slots in without touching tested code.

### Implementation in Python

#### Strategy Pattern via `Protocol`

Define a `Protocol` that captures the extension point. Each new behavior is a new class implementing that protocol. The caller never needs to change.

```python
from typing import Protocol

class PaymentProcessor(Protocol):
    def charge(self, amount_cents: int, token: str) -> str:
        ...
```

New processors implement `PaymentProcessor`; the checkout module never changes.

#### ABC Abstract Methods

```python
from abc import ABC, abstractmethod

class ReportRenderer(ABC):
    @abstractmethod
    def render(self, data: dict) -> str:
        ...
```

New report formats are new subclasses; the report generation pipeline is closed to modification.

#### `functools.singledispatch` for Type-Based Extension

```python
from functools import singledispatch

@singledispatch
def serialize(obj):
    raise TypeError(f"No serializer for {type(obj)}")

@serialize.register(int)
def _(obj: int) -> str:
    return str(obj)

@serialize.register(list)
def _(obj: list) -> str:
    return "[" + ", ".join(serialize(x) for x in obj) + "]"
```

Adding support for a new type registers a new function; `serialize` itself is never edited.

### Bad Example — OCP Violation

```python
def calculate_area(shape) -> float:
    if isinstance(shape, Circle):
        return 3.14159 * shape.radius ** 2
    elif isinstance(shape, Rectangle):
        return shape.width * shape.height
    elif isinstance(shape, Triangle):
        return 0.5 * shape.base * shape.height
    # Adding Square requires editing THIS function — a violation
    elif isinstance(shape, Square):
        return shape.side ** 2
```

Every new shape type forces an edit to `calculate_area`, risking regressions in existing shape handling.

### Good Example — OCP Applied

```python
from typing import Protocol

class Shape(Protocol):
    def area(self) -> float:
        ...

class Circle:
    def __init__(self, radius: float) -> None:
        self.radius = radius

    def area(self) -> float:
        return 3.14159 * self.radius ** 2

class Rectangle:
    def __init__(self, width: float, height: float) -> None:
        self.width = width
        self.height = height

    def area(self) -> float:
        return self.width * self.height

def calculate_area(shape: Shape) -> float:
    return shape.area()
```

Adding `Square` means writing a new `Square` class with `.area()`. The `calculate_area` function is never touched — it is closed to modification.

### Classic Examples

- **Payment processor**: add `StripeProcessor` without touching `PayPalProcessor` or the checkout module
- **Shape area calculator**: add `Hexagon` without modifying existing shape logic
- **Log formatter**: add `JsonFormatter` without touching `PlainTextFormatter` or the logging pipeline

### Relationship to Strategy Pattern

OCP is the *principle*; the Strategy pattern is a common *implementation* of it. When you model extension points as Protocol-typed parameters and inject implementations, you are applying both OCP and the Strategy pattern simultaneously. See `wiki/tier2-core/design-patterns/behavioral.md` for the full Strategy pattern treatment.

## Agent Guidance

### Do
- Identify extension points early in design; define a `Protocol` or `ABC` for each
- Write new behavior as new classes; resist the urge to add `elif` to existing dispatch logic
- Use `functools.singledispatch` for type-based polymorphism instead of `isinstance` chains
- Treat an `elif isinstance(...)` block as a code smell requiring OCP refactoring

### Do Not
- Do not edit a tested class to add new behavior if a new class can accomplish the same goal
- Do not use `isinstance` chains for dispatch — use `Protocol` and polymorphism
- Do not place extension logic (new payment types, new serializers) inside a central orchestration class

## Checklist
- [ ] New behavior is implemented in a new class, not by editing existing classes
- [ ] Extension points are expressed as `Protocol` or `ABC`
- [ ] No `isinstance` chains exist that would require editing when a new type is added
- [ ] `singledispatch` is used for type-based dispatch where appropriate
- [ ] Existing unit tests still pass without modification after adding new behavior

## See Also
- `wiki/tier2-core/solid-principles/srp.md`
- `wiki/tier2-core/solid-principles/lsp.md`
- `wiki/tier2-core/solid-principles/overview.md`
- `wiki/tier2-core/design-patterns/behavioral.md`
- `wiki/tier1-sources/swebok-v4/ka01-requirements.md`

## Source

Bertrand Meyer, *Object-Oriented Software Construction* (1988); restated by Robert C. Martin (2002). `functools.singledispatch` documented in PEP 443. Synthesized from *Software Development Best Practices for Agent* reference document.
