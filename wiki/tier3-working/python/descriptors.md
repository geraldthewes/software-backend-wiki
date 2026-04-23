# Python Descriptors (Tier 3)

> **Tier 3** | Source: Python Descriptor HOWTO, docs.python.org/3/howto/descriptor.html | Enforces/Derives From: wiki/tier3-working/python/overview.md, wiki/tier3-working/python/type-system.md, wiki/tier2-core/solid-principles/srp.md

## Summary

Descriptors are the mechanism behind Python's `property`, `classmethod`, `staticmethod`, and `__slots__`. Any class defining `__get__`, `__set__`, or `__delete__` is a descriptor — when an instance of that class is stored as a class attribute, Python invokes these methods instead of returning the object directly. Descriptors enable reusable attribute validation, computed properties, and logging without cluttering domain classes.

## Key Concepts

### The Descriptor Protocol

| Method | Signature | Triggered By |
|--------|-----------|-------------|
| `__get__` | `(self, obj, objtype=None)` | Reading the attribute |
| `__set__` | `(self, obj, value)` | Writing the attribute |
| `__delete__` | `(self, obj)` | `del obj.attr` |
| `__set_name__` | `(self, owner, name)` | Class body assignment (Python 3.6+) |

### Data vs Non-Data Descriptors

| Type | Defines | Lookup Priority |
|------|---------|----------------|
| Data descriptor | `__set__` and/or `__delete__` | Beats instance `__dict__` |
| Non-data descriptor | Only `__get__` | Instance `__dict__` beats it |

This distinction is critical: storing a value in `instance.__dict__` will *override* a non-data descriptor but will *not* override a data descriptor.

### Attribute Lookup Order (instance.x)

1. Data descriptors from `type(instance)` and its MRO bases
2. Instance variables (`instance.__dict__`)
3. Non-data descriptors and other class variables
4. `__getattr__` if defined

### Built-in Descriptors

**property** — data descriptor for computed attributes:

```python
class Circle:
    def __init__(self, radius: float) -> None:
        self._radius = radius

    @property
    def radius(self) -> float:
        return self._radius

    @radius.setter
    def radius(self, value: float) -> None:
        if value < 0:
            raise ValueError("Radius must be non-negative")
        self._radius = value

    @property
    def area(self) -> float:
        import math
        return math.pi * self._radius ** 2
```

**staticmethod** — non-data descriptor that returns the bare function (no binding):

```python
class Util:
    @staticmethod
    def parse_date(s: str) -> tuple[int, int, int]:
        y, m, d = s.split('-')
        return int(y), int(m), int(d)
```

**classmethod** — non-data descriptor that binds the class (not the instance):

```python
class Config:
    _instance = None

    @classmethod
    def from_env(cls) -> "Config":
        obj = cls()
        obj.debug = os.getenv("DEBUG", "false") == "true"
        return obj
```

### Custom Descriptor with __set_name__

`__set_name__` is called when the descriptor is assigned to a class attribute, giving it the attribute name automatically:

```python
import logging
from typing import Any

logger = logging.getLogger(__name__)

class Validated:
    """Descriptor that enforces type and range constraints."""

    def __set_name__(self, owner: type, name: str) -> None:
        self.public_name = name
        self.private_name = "_" + name

    def __get__(self, obj: Any, objtype: type | None = None) -> Any:
        if obj is None:
            return self                     # accessed on class, not instance
        return getattr(obj, self.private_name, None)

    def __set__(self, obj: Any, value: Any) -> None:
        self.validate(value)
        logger.debug("Setting %s = %r on %r", self.public_name, value, obj)
        setattr(obj, self.private_name, value)

    def validate(self, value: Any) -> None:
        raise NotImplementedError


class PositiveFloat(Validated):
    def validate(self, value: Any) -> None:
        if not isinstance(value, (int, float)) or value <= 0:
            raise ValueError(f"{self.public_name} must be a positive number, got {value!r}")


class Product:
    price    = PositiveFloat()
    quantity = PositiveFloat()

    def __init__(self, price: float, quantity: float) -> None:
        self.price    = price
        self.quantity = quantity

    @property
    def total(self) -> float:
        return self.price * self.quantity
```

### __slots__ and Member Descriptors

`__slots__` replaces `__dict__` with a fixed-size array, managed internally by member descriptors. Benefits: ~68% less memory per instance, ~35% faster attribute access, and typo detection at assignment:

```python
class Point:
    __slots__ = ('x', 'y')

    def __init__(self, x: float, y: float) -> None:
        self.x = x
        self.y = y

p = Point(1.0, 2.0)
# p.z = 3.0  ← AttributeError: 'Point' object has no attribute 'z'
```

Limitations of `__slots__`:
- Incompatible with `functools.cached_property` (which needs `__dict__`)
- Subclasses without their own `__slots__` regain `__dict__`
- Multiple inheritance requires care to avoid slot conflicts

## Agent Guidance

### Do

- Use `@property` for computed attributes and attribute validation — it is the idiomatic data descriptor.
- Use `__set_name__` in custom descriptors to avoid manually passing the attribute name.
- Return `self` from `__get__` when `obj is None` (class-level access) — this is the expected pattern.
- Use `__slots__` in value objects and domain entities with many instances to reduce memory.
- Use custom descriptors to centralize reusable validation logic rather than duplicating it in every setter.

### Do Not

- Do not implement `__set__` or `__delete__` in a descriptor unless validation or side effects are needed — a non-data descriptor is simpler and lets instance `__dict__` override it.
- Do not access `__dict__` directly on instances using `__slots__` — it does not exist.
- Do not use descriptors as a first resort for simple attribute storage — `@dataclass` fields are simpler.
- Do not forget to handle `obj is None` in `__get__` — accessing a descriptor on the class itself passes `None` as `obj`.

## Checklist

- [ ] `@property` used for computed or validated attributes
- [ ] Custom descriptors implement `__set_name__` to receive their attribute name automatically
- [ ] `__get__` returns `self` when `obj is None`
- [ ] `__slots__` used for value objects or data classes with many instances
- [ ] Custom descriptors centralise validation — not duplicated per-class

## See Also

- wiki/tier3-working/python/type-system.md
- wiki/tier3-working/python/annotations.md
- wiki/tier3-working/python/idioms.md
- wiki/tier2-core/solid-principles/srp.md
- wiki/tier3-working/python/mro.md

## Source

Python Descriptor HOWTO, docs.python.org/3/howto/descriptor.html. Python Data Model documentation.
