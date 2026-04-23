# Python Enumerations (Tier 3)

> **Tier 3** | Source: Python Enum HOWTO, docs.python.org/3/howto/enum.html | Enforces/Derives From: wiki/tier3-working/python/overview.md, wiki/tier1-sources/python-peps/pep-008-style.md

## Summary

Python's `enum` module provides `Enum`, `IntEnum`, `StrEnum`, `Flag`, and `IntFlag` classes for defining symbolic constants. Enumerations improve over bare constants by providing type safety, preventing unintended value comparisons, enabling iteration, and producing readable `repr()` output. A coding agent should use `Enum` by default for domain constants and reach for subtypes only when interoperability with `int` or `str` is required.

## Key Concepts

### Enum Basics

```python
from enum import Enum, auto

class OrderStatus(Enum):
    PENDING   = auto()
    CONFIRMED = auto()
    SHIPPED   = auto()
    DELIVERED = auto()
    CANCELLED = auto()

# Member access
OrderStatus.PENDING           # <OrderStatus.PENDING: 1>
OrderStatus.PENDING.name      # 'PENDING'
OrderStatus.PENDING.value     # 1
OrderStatus['CONFIRMED']      # by name
OrderStatus(2)                # by value → <OrderStatus.CONFIRMED: 2>
```

### Enum Type Taxonomy

| Type | Inherits | Use Case | Comparison to primitives |
|------|----------|----------|--------------------------|
| `Enum` | object | Default; domain constants | Not equal to int/str |
| `IntEnum` | int, Enum | Legacy integer constants, array indexing | Equal to matching int |
| `StrEnum` (3.11+) | str, Enum | String constants, HTTP headers, config keys | Equal to matching str |
| `Flag` | Enum | Bitwise-combinable flags | Not equal to int |
| `IntFlag` | int, Flag | Legacy bitmask constants | Equal to matching int |

### Auto-Numbering

```python
from enum import Enum, auto

class Color(Enum):
    RED   = auto()   # 1
    GREEN = auto()   # 2
    BLUE  = auto()   # 3

# Override for string-valued auto
class AutoName(Enum):
    @staticmethod
    def _generate_next_value_(name, start, count, last_values):
        return name.lower()

class Direction(AutoName):
    NORTH = auto()   # 'north'
    SOUTH = auto()   # 'south'
```

### Preventing Duplicate Values

```python
from enum import Enum, unique

@unique
class Priority(Enum):
    LOW    = 1
    MEDIUM = 2
    HIGH   = 3
    # ALSO_HIGH = 3  # ← Would raise ValueError at class definition
```

Without `@unique`, a second member with the same value becomes an alias (returned as the first member). Aliases do not appear in iteration.

### Flag Enums for Bitwise Combinations

```python
from enum import Flag, auto

class Permission(Flag):
    READ    = auto()   # 1
    WRITE   = auto()   # 2
    EXECUTE = auto()   # 4

# Compose
admin = Permission.READ | Permission.WRITE | Permission.EXECUTE
user  = Permission.READ

# Test membership
if Permission.WRITE in admin:
    print("Can write")

# Iterate constituent flags
for perm in admin:
    print(perm)
```

### Data-Carrying Members

```python
from enum import Enum

class Planet(Enum):
    EARTH = (5.976e+24, 6.37814e6)
    MARS  = (6.421e+23, 3.3972e6)

    def __init__(self, mass: float, radius: float) -> None:
        self.mass   = mass
        self.radius = radius

    @property
    def surface_gravity(self) -> float:
        G = 6.67300e-11
        return G * self.mass / (self.radius ** 2)

Planet.EARTH.surface_gravity  # 9.802...
```

### Iteration and Membership

```python
# list() returns canonical members only (no aliases)
list(OrderStatus)

# __members__ includes aliases
for name, member in OrderStatus.__members__.items():
    print(name, member)

# Containment test
OrderStatus.PENDING in OrderStatus  # True
```

## Agent Guidance

### Do

- Use `Enum` (not bare module-level constants) whenever a variable can take one of a fixed set of values.
- Use `auto()` when the specific integer value is irrelevant.
- Apply `@unique` to prevent accidental aliases unless aliasing is intentional.
- Use `Flag` for combinable permission/feature-flag enumerations — bitwise `|` composition is explicit and type-safe.
- Use `StrEnum` (Python 3.11+) for string constants that need to interoperate with string APIs (HTTP headers, config file keys).
- Use `IntEnum` only when backward compatibility with integer-indexed code is required.
- Add methods and properties to enums when behavior belongs with the value (e.g., `Planet.surface_gravity`).

### Do Not

- Do not compare plain `Enum` members to integer literals — `Color.RED == 1` is always `False`; use `IntEnum` explicitly if integer comparison is needed.
- Do not use ordered comparisons (`<`, `>`) on plain `Enum` — they raise `TypeError`; use `IntEnum` or add `__lt__` explicitly.
- Do not use `Flag` members where only a single value is valid at a time — use `Enum` instead.
- Do not define `Enum` members whose names start and end with a single underscore — those are reserved for enum internals.
- Do not rely on alias iteration — aliases are silently excluded from `list(MyEnum)`.

## Checklist

- [ ] Domain constants defined as `Enum` rather than bare module-level variables
- [ ] `@unique` applied where duplicate values would be a bug
- [ ] `auto()` used when specific numeric values are not externally significant
- [ ] `Flag` used instead of ad-hoc integer bitmasks
- [ ] `StrEnum` used for string constants requiring string API compatibility (Python 3.11+)
- [ ] No comparisons of plain `Enum` to primitive values

## See Also

- wiki/tier3-working/python/overview.md
- wiki/tier3-working/python/type-system.md
- wiki/tier1-sources/python-peps/pep-008-style.md
- wiki/tier3-working/python/idioms.md
- wiki/tier1-sources/python-peps/pep-020-zen.md

## Source

Python Enum HOWTO, docs.python.org/3/howto/enum.html. Python `enum` module documentation.
