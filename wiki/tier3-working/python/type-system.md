# Python Type System Guide

> **Tier 3** | Source: PEP 484, Python typing docs | Enforces/Derives From: wiki/tier1-sources/python-peps/pep-484-type-hints.md, wiki/tier1-sources/swebok-v4/ka04-construction.md

## Summary

Python's `typing` module provides a formal contract layer over dynamic dispatch. Used consistently with `mypy --strict`, it catches an entire class of bugs before runtime. This page covers all major typing constructs with usage notes and the mypy configuration required in CI.

## Builtin Generics (Python 3.9+)

Use lowercase builtin generics. Do not import `List`, `Dict`, `Tuple`, `Set` from `typing` in new code.

```python
# Python 3.9+ — preferred
def process(items: list[int]) -> dict[str, int]:
    return {str(i): i for i in items}

def coordinates() -> tuple[float, float, float]:
    return (1.0, 2.0, 3.0)

def unique(values: list[str]) -> set[str]:
    return set(values)

# Python 3.8 and below — legacy, avoid in new code
from typing import List, Dict, Tuple, Set
def process(items: List[int]) -> Dict[str, int]: ...
```

## Optional and Union

```python
# Python 3.10+ — prefer X | Y syntax
def find_user(user_id: int) -> User | None:
    ...

def parse_value(raw: str) -> int | float | None:
    ...

# Python 3.9 and below — use Optional / Union from typing
from typing import Optional, Union

def find_user(user_id: int) -> Optional[User]:  # equivalent to User | None
    ...

def parse_value(raw: str) -> Union[int, float, None]:
    ...
```

## TypeVar — Generic Functions

```python
from typing import TypeVar

T = TypeVar("T")

def first(items: list[T]) -> T:
    """Return the first item from a non-empty list."""
    return items[0]

# Bound TypeVar — constrain to a type or its subclasses
from typing import TypeVar
from collections.abc import Comparable

C = TypeVar("C", bound="Comparable")

def maximum(a: C, b: C) -> C:
    return a if a > b else b
```

## Protocol — Structural Subtyping

Prefer `Protocol` over ABCs for loose coupling. A class satisfies a Protocol implicitly — no `implements` keyword needed.

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Drawable(Protocol):
    def draw(self) -> None: ...

class Circle:
    def draw(self) -> None:
        print("Drawing circle")

class Square:
    def draw(self) -> None:
        print("Drawing square")

def render(shape: Drawable) -> None:
    shape.draw()

render(Circle())   # works — Circle satisfies Drawable implicitly
render(Square())   # works

# runtime_checkable allows isinstance checks
assert isinstance(Circle(), Drawable)  # True
```

Keep Protocols small (ISP): a Protocol with more than 5 methods is a design smell.

## TypedDict — Typed Dictionary Structures

```python
from typing import TypedDict

class UserRecord(TypedDict):
    id: int
    name: str
    email: str

class PartialUpdate(TypedDict, total=False):
    """All keys are optional — for PATCH-style updates."""
    name: str
    email: str

def update_user(user_id: int, changes: PartialUpdate) -> None:
    ...
```

## Literal — String Constants

```python
from typing import Literal

HTTPMethod = Literal["GET", "POST", "PUT", "PATCH", "DELETE"]
LogLevel = Literal["DEBUG", "INFO", "WARNING", "ERROR", "CRITICAL"]

def make_request(url: str, method: HTTPMethod) -> None:
    ...

make_request("/api/users", "GET")     # OK
make_request("/api/users", "FETCH")   # mypy error — "FETCH" not in Literal
```

## Final — Constants

```python
from typing import Final

MAX_RETRIES: Final = 3
API_BASE_URL: Final[str] = "https://api.example.com"

# MAX_RETRIES = 5  # mypy error: cannot assign to final name
```

## Dataclass Fields

```python
from dataclasses import dataclass, field
from typing import ClassVar

@dataclass
class Event:
    name: str
    tags: list[str] = field(default_factory=list)
    _id: int = field(default=0, repr=False, compare=False)
    description: str = field(default="", metadata={"max_length": 500})
    registry: ClassVar[dict[str, "Event"]] = {}  # shared across instances
```

## Callable — Function Types

```python
from collections.abc import Callable

# Callable[[arg_type, ...], return_type]
def apply(func: Callable[[int, int], int], a: int, b: int) -> int:
    return func(a, b)

# Variadic — unknown arguments
def retry(func: Callable[..., bool], attempts: int) -> bool:
    for _ in range(attempts):
        if func():
            return True
    return False
```

## Overload — Multiple Signatures

```python
from typing import overload

@overload
def parse(value: str) -> int: ...
@overload
def parse(value: bytes) -> str: ...

def parse(value: str | bytes) -> int | str:
    if isinstance(value, str):
        return int(value)
    return value.decode("utf-8")
```

## TYPE_CHECKING — Avoid Circular Imports

```python
from __future__ import annotations  # defers evaluation of all annotations
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    # This import only runs during type checking, not at runtime
    from mymodule.models import HeavyModel

def process(model: "HeavyModel") -> None:  # string annotation if no __future__
    ...
```

## mypy Configuration

Place in `pyproject.toml`:

```toml
[tool.mypy]
python_version = "3.11"
strict = true
warn_return_any = true
warn_unused_configs = true
show_error_codes = true

# Per-module overrides for third-party libraries lacking stubs
[[tool.mypy.overrides]]
module = ["boto3.*", "botocore.*"]
ignore_missing_imports = true
```

`strict = true` enables: `disallow_untyped_defs`, `disallow_any_generics`, `no_implicit_optional`, `warn_redundant_casts`, `warn_unused_ignores`, and more.

## Common Typed Patterns

### Typed Repository Protocol

```python
from typing import Protocol

class UserRepository(Protocol):
    def get(self, user_id: int) -> User | None: ...
    def save(self, user: User) -> User: ...
    def list_all(self) -> list[User]: ...
```

### Typed Factory

```python
from typing import TypeVar, Type

T = TypeVar("T")

def create(cls: Type[T], **kwargs) -> T:
    return cls(**kwargs)
```

### Typed Event Handler

```python
from collections.abc import Callable
from typing import TypeVar

EventT = TypeVar("EventT")
Handler = Callable[[EventT], None]

class EventBus:
    def subscribe(self, event_type: type[EventT], handler: Handler[EventT]) -> None: ...
    def publish(self, event: EventT) -> None: ...
```

## See Also

- wiki/tier1-sources/python-peps/pep-484-type-hints.md
- wiki/tier1-sources/python-peps/pep-443-singledispatch.md
- wiki/tier2-core/solid-principles/isp.md
- wiki/tier3-working/python/idioms.md

## Source

PEP 484 — Type Hints. Python docs: typing module, dataclasses. mypy documentation (mypy-lang.org). "Architecture Patterns with Python" (Percival & Gregory, 2020).
