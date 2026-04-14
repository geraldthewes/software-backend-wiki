# Liskov Substitution Principle (LSP)

> **Tier 2** | Source: Barbara Liskov (1987) | Derives From: ka03-design | Authority: established practice

## Summary

If `S` is a subtype of `T`, then objects of type `T` may be replaced with objects of type `S` without altering the correctness of the program. LSP defines what it means for inheritance to be semantically correct — not merely type-compatible.

## Key Concepts

### Definition

Barbara Liskov introduced the principle in her 1987 conference keynote. The formal statement:

> Let `q(x)` be a property provable about objects `x` of type `T`. Then `q(y)` should be provable for objects `y` of type `S` where `S` is a subtype of `T`.

In plain terms: **callers should not need to know which subtype they are using**. If they do, the subtype violates LSP.

### Behavioral Subtyping

LSP is stronger than mere type compatibility (what a type checker enforces). It requires **behavioral compatibility**:

- Preconditions of a method cannot be strengthened in a subclass
- Postconditions of a method cannot be weakened in a subclass
- Invariants of the base class must be preserved by the subclass
- Exceptions raised must be the same type or subtypes of those declared by the base

### Common Violation Patterns

| Pattern | Example | Why It Violates LSP |
|---------|---------|---------------------|
| `NotImplementedError` in subclass | `def save(self): raise NotImplementedError` | Caller that expected `save()` to work now receives an exception |
| Narrowed preconditions | Base accepts any `int`; subclass only accepts positive `int` | Callers passing negative values are broken by substitution |
| Strengthened postconditions | Base guarantees non-empty return; subclass may return `None` | Callers relying on non-empty return get `AttributeError` |
| New exception types | Subclass raises `PermissionError`; base only raised `ValueError` | Callers whose exception handling was written for the base are broken |

### Bad Example — Rectangle / Square LSP Violation

```python
class Rectangle:
    def __init__(self, width: float, height: float) -> None:
        self.width = width
        self.height = height

    def set_width(self, w: float) -> None:
        self.width = w

    def set_height(self, h: float) -> None:
        self.height = h

    def area(self) -> float:
        return self.width * self.height


class Square(Rectangle):
    """Violates LSP: changes Rectangle's behavior invariant."""

    def set_width(self, w: float) -> None:
        self.width = w
        self.height = w  # Forces height = width — breaks caller assumptions

    def set_height(self, h: float) -> None:
        self.width = h
        self.height = h


def resize_and_check(shape: Rectangle) -> None:
    shape.set_width(5)
    shape.set_height(10)
    assert shape.area() == 50  # Passes for Rectangle; FAILS for Square
```

A `Square` cannot be substituted for a `Rectangle` because the caller's assumptions about independent width/height are broken.

### Good Example — Refactored Without Inheritance

```python
from typing import Protocol

class Shape(Protocol):
    def area(self) -> float:
        ...

class Rectangle:
    def __init__(self, width: float, height: float) -> None:
        self._width = width
        self._height = height

    def area(self) -> float:
        return self._width * self._height

class Square:
    def __init__(self, side: float) -> None:
        self._side = side

    def area(self) -> float:
        return self._side ** 2

def print_area(shape: Shape) -> None:
    print(shape.area())  # Works correctly for both types
```

Neither inherits from the other. Both satisfy the `Shape` protocol. The caller is never surprised.

### Python Implications

- Be cautious with inheritance; it is a strong coupling mechanism
- Prefer composition over inheritance when behavior differs meaningfully between types
- If you override a method in a subclass, verify you are not changing the contract: same accepted inputs, same guaranteed outputs, same or fewer exception types
- Abstract base classes with `@abstractmethod` help catch missing implementations at class definition time, but they do not enforce behavioral contracts — that requires careful design

## Agent Guidance

### Do
- Before subclassing, ask: "Can a caller use the subtype wherever the parent is expected, with no surprises?"
- Prefer composition (hold an instance of another class) over inheritance when behavior differs
- When using `ABC`, document the behavioral contract in the docstring, not just the signature
- Write tests that run the same assertions against every subtype — a suite that passes for the parent and fails for a subtype reveals an LSP violation

### Do Not
- Do not override a method to raise `NotImplementedError` — if the method makes no sense for the subtype, do not inherit
- Do not narrow the accepted input range in a subclass
- Do not return `None` from a method that the parent guarantees returns a value
- Do not add new exception types in a subclass method that callers are not prepared to handle

## Checklist
- [ ] No subclass method raises `NotImplementedError`
- [ ] No subclass narrows the accepted input range of a parent method
- [ ] No subclass weakens the return value guarantee of a parent method
- [ ] The same test suite passes for all implementations of the same base type / protocol
- [ ] Composition is used instead of inheritance when full behavioral substitution is not possible

## See Also
- `wiki/tier2-core/solid-principles/ocp.md`
- `wiki/tier2-core/solid-principles/isp.md`
- `wiki/tier2-core/solid-principles/overview.md`
- `wiki/tier2-core/design-patterns/overview.md`
- `wiki/tier1-sources/swebok-v4/ka01-requirements.md`

## Source

Barbara Liskov, "Data Abstraction and Hierarchy," OOPSLA 1987. Robert C. Martin's formulation in *Agile Software Development* (2002). Synthesized from *Software Development Best Practices for Agent* reference document.
