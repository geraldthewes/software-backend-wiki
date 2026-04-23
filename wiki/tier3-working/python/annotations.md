# Python Annotations Best Practices (Tier 3)

> **Tier 3** | Source: Python Annotations HOWTO, docs.python.org/3/howto/annotations.html | Enforces/Derives From: wiki/tier3-working/python/type-system.md, wiki/tier1-sources/python-peps/pep-484-type-hints.md

## Summary

Python annotations (`__annotations__`) store type hints and other metadata on functions, classes, and modules. Their behavior has changed across Python versions (3.0–3.14), creating pitfalls when accessing them directly. The safe rule is: use `inspect.get_annotations()` (Python 3.10+) or `annotationlib.get_annotations()` (Python 3.14+) rather than accessing `__annotations__` directly, and never access `__annotations__` on instances.

## Key Concepts

### What Annotations Are

```python
def greet(name: str) -> str:
    return "Hello, " + name

# Stored as:
greet.__annotations__   # {'name': <class 'str'>, 'return': <class 'str'>}

class Point:
    x: float
    y: float

Point.__annotations__   # {'x': <class 'float'>, 'y': <class 'float'>}
```

Annotations are a dict stored on the function or class object. They are not enforced at runtime by default — only type checkers (mypy, pyright) use them.

### Version-Safe Access Patterns

| Python Version | Safe Access Method |
|---------------|-------------------|
| 3.14+ | `annotationlib.get_annotations(obj)` |
| 3.10–3.13 | `inspect.get_annotations(obj)` |
| 3.9 and older | Version-specific manual access (see below) |

```python
# Python 3.10+
import inspect
hints = inspect.get_annotations(my_function)
hints = inspect.get_annotations(MyClass)
hints = inspect.get_annotations(my_module)

# Python 3.14+
import annotationlib
hints = annotationlib.get_annotations(my_function)
```

### Why Direct Access Is Risky

**Problem 1 — class inheritance (Python 3.9 and older):**
```python
class Base:
    x: int

class Child(Base):
    pass

# In Python 3.9 and older, this returns Base's annotations, not an empty dict!
Child.__annotations__   # {'x': <class 'int'>} — inherited, potentially wrong

# Safe for Python 3.9 and older:
ann = Child.__dict__.get('__annotations__', {})  # {} — correct
```

**Problem 2 — stringized annotations with `from __future__ import annotations`:**
```python
from __future__ import annotations

def greet(name: str) -> str:
    ...

greet.__annotations__   # {'name': 'str', 'return': 'str'} — STRINGS, not types!
```
`inspect.get_annotations()` handles un-stringizing automatically.

**Problem 3 — instances have no `__annotations__`:**
```python
p = Point(1.0, 2.0)
p.__annotations__     # AttributeError or wrong result — use type(p).__annotations__
```

### Evaluating String Annotations

When annotations are stringized (deferred evaluation), use `typing.get_type_hints()` to evaluate them:

```python
import typing

def greet(name: str) -> str: ...

typing.get_type_hints(greet)   # Always returns evaluated types, not strings
```

`typing.get_type_hints()` requires the function's module globals to be available and may raise `NameError` for forward references that cannot be resolved. `inspect.get_annotations()` with `eval_str=True` is safer in Python 3.10+:

```python
import inspect
hints = inspect.get_annotations(greet, eval_str=True)
```

### from __future__ import annotations (PEP 563)

Adding this import to a module makes all annotations lazy strings:

```python
from __future__ import annotations   # All annotations become strings

def process(data: list[DataRecord]) -> dict[str, int]:
    ...

# __annotations__ = {'data': 'list[DataRecord]', 'return': 'dict[str, int]'}
```

This was intended to become the default in Python 3.10 but was deferred. Use `inspect.get_annotations(eval_str=True)` or `typing.get_type_hints()` when you need resolved types at runtime.

## Agent Guidance

### Do

- Use `inspect.get_annotations(obj)` (Python 3.10+) or `annotationlib.get_annotations(obj)` (Python 3.14+) to access annotations at runtime.
- Use `typing.get_type_hints(obj)` when you specifically need evaluated (not stringized) type objects.
- Validate that the return is a dict before iterating — `get_annotations()` guarantees a dict, but direct `__annotations__` access on unusual objects may not.
- Access class annotations via the class, not an instance: `type(obj).__annotations__` at minimum, or `inspect.get_annotations(type(obj))` for correctness.

### Do Not

- Do not access `obj.__annotations__` directly if the code must work across Python versions or with `from __future__ import annotations`.
- Do not access `instance.__annotations__` — instances rarely have the attribute; the result would be inherited from the class and may be wrong.
- Do not modify `__annotations__` directly — let Python manage it.
- Do not delete `__annotations__` — it can break tooling and type checkers.
- Do not call `eval()` on annotation strings manually — use `inspect.get_annotations(eval_str=True)` which handles scoping correctly.

## Checklist

- [ ] Runtime annotation access uses `inspect.get_annotations()` not `.__annotations__`
- [ ] `typing.get_type_hints()` used when resolved types are needed
- [ ] No access of `__annotations__` on instances
- [ ] `eval_str=True` passed to `get_annotations()` when string annotations must be evaluated
- [ ] Code tested against both stringized (`from __future__ import annotations`) and normal annotation modes

## See Also

- wiki/tier3-working/python/type-system.md
- wiki/tier1-sources/python-peps/pep-484-type-hints.md
- wiki/tier3-working/python/overview.md
- wiki/tier3-working/python/descriptors.md

## Source

Python Annotations HOWTO, docs.python.org/3/howto/annotations.html. PEP 484, PEP 563. Python `inspect`, `typing`, `annotationlib` module documentation.
