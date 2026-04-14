# PEP 443: Single-Dispatch Generic Functions

> **Tier 1** | Source: PEP 443 (python.org) | Authority: immutable

## Summary

PEP 443 introduced `functools.singledispatch`, a decorator that enables function overloading based on the type of the first argument. It provides polymorphic dispatch without requiring class hierarchies or `isinstance` chains. For a coding agent, `@singledispatch` is the preferred solution when you need type-based behavior dispatch across unrelated types, particularly when extending behavior for third-party types you cannot modify.

## Key Concepts

**Purpose:** Implement different behavior for different argument types in a single, extensible function — the functional alternative to method overriding in OOP.

**Mechanism:** The `@functools.singledispatch` decorator registers a base function (handles the fallback case). Additional implementations are registered with `@func.register(Type)`. At call time, the correct implementation is selected based on the runtime type of the first argument.

**Open/Closed Principle Connection:** `@singledispatch` enables the Open/Closed Principle — the dispatch function is closed for modification, but open for extension. New type handling can be added by calling `.register()` without touching the original function.

## Syntax and Example

```python
from functools import singledispatch
from typing import Any

@singledispatch
def serialize(value: Any) -> str:
    """Base implementation — called when no specific type matches."""
    raise TypeError(f"Cannot serialize type: {type(value).__name__}")

@serialize.register(int)
def _serialize_int(value: int) -> str:
    return str(value)

@serialize.register(float)
def _serialize_float(value: float) -> str:
    return f"{value:.6f}"

@serialize.register(list)
def _serialize_list(value: list) -> str:
    return "[" + ", ".join(serialize(item) for item in value) + "]"

@serialize.register(dict)
def _serialize_dict(value: dict) -> str:
    pairs = [f"{serialize(k)}: {serialize(v)}" for k, v in value.items()]
    return "{" + ", ".join(pairs) + "}"

# Usage
serialize(42)          # "42"
serialize(3.14)        # "3.140000"
serialize([1, 2, 3])  # "[1, 2, 3]"
```

**Python 3.7+ type annotation syntax for registration:**

```python
@singledispatch
def process(value: object) -> None:
    raise NotImplementedError(f"No handler for {type(value)}")

@process.register
def _(value: int) -> None:   # type inferred from annotation
    print(f"Processing int: {value}")

@process.register
def _(value: str) -> None:
    print(f"Processing string: {value}")
```

**Checking dispatch and adding implementations from external modules:**

```python
# In a third-party type integration module
import numpy as np
from mypackage.serializers import serialize

@serialize.register(np.ndarray)
def _serialize_ndarray(value: np.ndarray) -> str:
    return value.tolist().__repr__()

# This extends serialize() without modifying mypackage
```

## When to Use vs. When Not to Use

### Use singledispatch when:

- You need different behavior for **unrelated types** that do not share a common base class
- You want to **extend behavior for third-party types** you cannot modify (no inheritance)
- The dispatch logic is in a utility or serialization function where type-based switching is natural
- You want to enable **plugin-style extensibility** — callers can add new type handlers without forking the code
- You are replacing an `isinstance` chain that has grown beyond 2-3 branches

### Do not use singledispatch when:

- A simple `isinstance` check covering 1-2 types is clearer and will not grow
- Class-based polymorphism (OCP via inheritance or Protocol) fits naturally — when types share a meaningful relationship and the behavior belongs to the type itself
- You need dispatch on multiple arguments — `singledispatch` only dispatches on the first argument (use `multipledispatch` library for multi-argument dispatch)
- The types are all related by inheritance and `super()` delegation is appropriate

```python
# Simple isinstance check — singledispatch is overkill here
def format_value(value):
    if isinstance(value, bool):
        return "yes" if value else "no"
    return str(value)

# Growing isinstance chain — singledispatch is appropriate
def serialize(value):
    if isinstance(value, int): ...
    elif isinstance(value, float): ...
    elif isinstance(value, list): ...
    elif isinstance(value, dict): ...
    elif isinstance(value, datetime): ...
    # ... and growing
```

## Relationship to OOP and Protocol

`@singledispatch` complements, rather than replaces, Protocol-based polymorphism:

- **Protocol**: behavior belongs to the type; the type "knows how" to do something — use for domain objects
- **singledispatch**: behavior belongs to the operation; the function "knows how" to handle each type — use for utility functions and serialization

```python
# Protocol approach — type knows how to serialize itself
class Serializable(Protocol):
    def to_dict(self) -> dict: ...

def serialize_model(obj: Serializable) -> dict:
    return obj.to_dict()

# singledispatch approach — function knows how to handle each type
# Appropriate when types are third-party or unrelated
@singledispatch
def to_json(value: object) -> str:
    raise TypeError(...)
```

## Agent Guidance

### Do
- Prefer Protocol-based polymorphism for new code where types share a domain relationship
- Use `@singledispatch` for serialization, formatting, and visitor patterns over unrelated types
- Register handlers for third-party types in the integration module, not the core module
- Provide a meaningful base function that raises `TypeError` or `NotImplementedError` with a helpful message

### Do Not
- Use `@singledispatch` when the types are related and inheritance/Protocol is more appropriate
- Expect dispatch on multiple arguments — only the first argument type is used for dispatch
- Replace every `isinstance` check with `@singledispatch` — use judgment based on complexity

## Checklist
- [ ] `@singledispatch` used instead of `isinstance` chains with 3+ type branches
- [ ] Base function raises `TypeError` or `NotImplementedError` with a descriptive message
- [ ] Third-party type handlers registered in integration modules, not core modules
- [ ] Type annotation syntax used for registration (Python 3.7+) rather than `register(Type)` argument syntax where possible

## See Also
- wiki/tier1-sources/python-peps/overview.md
- wiki/tier1-sources/python-peps/pep-484-type-hints.md
- wiki/tier1-sources/python-peps/pep-020-zen.md

## Source
PEP 443 — Single-dispatch generic functions. https://peps.python.org/pep-0443/
