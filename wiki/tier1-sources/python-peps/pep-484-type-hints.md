# PEP 484: Type Hints

> **Tier 1** | Source: PEP 484 (python.org) | Authority: immutable

## Summary

PEP 484 introduced a formal syntax for type annotations in Python, enabling static analysis tools like `mypy` and `pyright` to verify type correctness before code is executed. Type hints act as machine-verifiable formal contracts between callers and implementations. For a coding agent, type hints on all public APIs are required — they make interfaces explicit, enable automated type checking in CI, and catch entire categories of bugs that would otherwise surface only at runtime.

## Key Concepts

**Type Hints Are Formal Contracts, Not Documentation:**

Unlike docstrings (which describe intent), type hints are verifiable specifications. A function annotated as `def get_user(user_id: int) -> User | None:` tells both humans and tools exactly what is accepted and returned. Type checkers enforce these contracts statically.

**Runtime Behavior:**

Type hints are not enforced at runtime by default — they are metadata used by static analyzers. To enforce at runtime, use `beartype` or Pydantic models. The primary value of type hints is enabling `mypy`/`pyright` to catch errors before execution.

## Basic Syntax

**Function annotations:**

```python
def add(x: int, y: int) -> int:
    return x + y

def process_user(user_id: int, name: str, active: bool = True) -> None:
    ...

def find_user(user_id: int) -> "User | None":  # forward reference as string
    ...
```

**Variable annotations:**

```python
user_count: int = 0
names: list[str] = []
config: dict[str, str] = {}
```

**Class attributes:**

```python
from dataclasses import dataclass

@dataclass
class User:
    user_id: int
    name: str
    email: str
    active: bool = True
```

## Common Types from `typing` Module

### Optional and Union

```python
# Python 3.9 and earlier
from typing import Optional, Union

def find(key: str) -> Optional[str]:   # same as Union[str, None]
    ...

def parse(value: Union[str, int]) -> str:
    ...

# Python 3.10+ — preferred syntax
def find(key: str) -> str | None:      # X | None replaces Optional[X]
    ...

def parse(value: str | int) -> str:   # X | Y replaces Union[X, Y]
    ...
```

### Collection Types

```python
# Python 3.9+ — use lowercase built-ins directly (preferred)
names: list[str] = []
lookup: dict[str, int] = {}
coords: tuple[float, float] = (0.0, 0.0)
unique_ids: set[int] = set()

# Python 3.8 and earlier — import from typing
from typing import List, Dict, Tuple, Set
names: List[str] = []
```

### Callable

```python
from typing import Callable

def apply(func: Callable[[int, str], bool], value: int, name: str) -> bool:
    return func(value, name)

# Callable with arbitrary arguments
def register(callback: Callable[..., None]) -> None:
    ...
```

### Any — Use Sparingly

```python
from typing import Any

# ANY defeats the purpose of type hints — use only at genuine system boundaries
def deserialize_external(raw: bytes) -> Any:   # acceptable at boundary
    ...

# WRONG — using Any to avoid type work
def process(data: Any) -> Any:   # no type information provided
    ...
```

Rule: `Any` is acceptable only when interfacing with untyped external systems (legacy APIs, raw deserialization output). Within your own codebase, use concrete types.

### TypeVar — Generic Functions

```python
from typing import TypeVar

T = TypeVar("T")

def first(items: list[T]) -> T | None:
    """Return first element or None if empty."""
    return items[0] if items else None

# Usage: type is inferred correctly
x: int | None = first([1, 2, 3])
y: str | None = first(["a", "b"])
```

### Protocol — Structural Subtyping (Duck Typing with Type Safety)

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Closeable(Protocol):
    def close(self) -> None:
        ...

class Saveable(Protocol):
    def save(self) -> bool:
        ...

# Any class that implements close() satisfies Closeable — no inheritance needed
def cleanup(resource: Closeable) -> None:
    resource.close()
```

**Prefer Protocol over ABC for loose coupling.** Protocol enables structural subtyping: any class that implements the required methods satisfies the Protocol, without importing or inheriting from it. This is the ISP (Interface Segregation Principle) in practice.

### TypedDict — Typed Dictionary Structures

```python
from typing import TypedDict

class UserRecord(TypedDict):
    user_id: int
    name: str
    email: str

# Optional keys
class Config(TypedDict, total=False):
    timeout: int
    retries: int
```

### Literal — Restrict to Specific Values

```python
from typing import Literal

def set_log_level(level: Literal["DEBUG", "INFO", "WARNING", "ERROR"]) -> None:
    ...

HttpMethod = Literal["GET", "POST", "PUT", "DELETE", "PATCH"]
```

### Final — Constants That Must Not Be Reassigned

```python
from typing import Final

MAX_CONNECTIONS: Final = 100
API_VERSION: Final[str] = "v2"
```

### dataclass — Preferred for Typed Data Containers

```python
from dataclasses import dataclass, field
from typing import Final

@dataclass(frozen=True)       # immutable: aligns with functional programming
class Point:
    x: float
    y: float

@dataclass
class Config:
    host: str
    port: int = 8080
    tags: list[str] = field(default_factory=list)  # mutable default — use field()
```

## mypy Configuration

**`pyproject.toml` — strict mode is the target:**

```toml
[tool.mypy]
python_version = "3.12"
strict = true
warn_unused_configs = true
warn_return_any = true
disallow_any_generics = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
check_untyped_defs = true
no_implicit_optional = true
warn_redundant_casts = true
warn_unused_ignores = true
```

**For libraries — add `py.typed` marker:**

```bash
touch src/mypackage/py.typed   # signals to mypy that the package has type info
```

**Running mypy in CI:**

```bash
mypy src/ --strict
```

Treat mypy errors as blocking in CI — they represent real type contract violations.

## Agent Guidance

### Do
- Add type hints to all public function signatures and class attributes
- Use `Protocol` for abstractions — prefer over ABC for loose coupling (ISP)
- Use lowercase collection types (`list[str]`, `dict[str, int]`) in Python 3.9+
- Use `X | None` syntax instead of `Optional[X]` in Python 3.10+
- Run `mypy --strict` in CI and treat errors as blocking
- Use `Final` for module-level constants
- Use `@dataclass(frozen=True)` for immutable data containers

### Do Not
- Use `Any` except at genuine system boundaries with external untyped systems
- Use `Optional[X]` when None is not a meaningful semantic value — use `X` with a required argument
- Skip type hints on internal helper functions if they are called by typed public functions
- Suppress mypy errors with `# type: ignore` without a comment explaining why

## Checklist
- [ ] All public function signatures have parameter and return type annotations
- [ ] All class attributes annotated in `__init__` or as class-level annotations
- [ ] No `Any` used without explicit justification comment
- [ ] `Optional[X]` replaced with `X | None` in Python 3.10+ code
- [ ] `mypy --strict src/` passes with no errors in CI
- [ ] `Protocol` used for abstractions instead of ABC where loose coupling is desired
- [ ] `@dataclass(frozen=True)` used for immutable value objects

## See Also
- wiki/tier1-sources/python-peps/overview.md
- wiki/tier1-sources/python-peps/pep-008-style.md
- wiki/tier1-sources/python-peps/pep-443-singledispatch.md

## Source
PEP 484 — Type Hints. https://peps.python.org/pep-0484/
