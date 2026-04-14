# Python Idioms and Patterns

> **Tier 3** | Source: PEP 8, PEP 20, Python docs | Enforces/Derives From: wiki/tier1-sources/python-peps/pep-008-style.md, wiki/tier1-sources/python-peps/pep-020-zen.md

## Summary

Pythonic code is code that uses the language's features as intended. This page documents idiomatic patterns with examples and a catalogue of anti-patterns that must be avoided in all agent-generated code.

## Context Managers

Use `with` for ALL resource management. Never rely on garbage collection to release file handles, connections, or locks.

```python
# Reading a file
with open("data.txt", "r") as f:
    contents = f.read()
# File is guaranteed closed here, even if an exception was raised

# Custom context manager using __enter__ / __exit__
class Timer:
    def __enter__(self):
        import time
        self._start = time.perf_counter()
        return self

    def __exit__(self, *args):
        self.elapsed = time.perf_counter() - self._start

with Timer() as t:
    result = expensive_computation()
print(f"Took {t.elapsed:.3f}s")

# Preferred: contextlib.contextmanager for simple cases
import contextlib

@contextlib.contextmanager
def managed_cursor(conn):
    cursor = conn.cursor()
    try:
        yield cursor
        conn.commit()
    except Exception:
        conn.rollback()
        raise
    finally:
        cursor.close()
```

## Dataclasses

Use `@dataclass` for typed data containers. Prefer `frozen=True` for value objects that should not be mutated.

```python
from dataclasses import dataclass, field

@dataclass(frozen=True)
class Point:
    x: float
    y: float

@dataclass
class UserProfile:
    user_id: int
    name: str
    tags: list[str] = field(default_factory=list)   # mutable default — always use field()
    metadata: dict[str, str] = field(default_factory=dict)
    _cache: str = field(default="", repr=False, compare=False)  # excluded from repr and equality
```

## List Comprehensions Over Loops

Prefer comprehensions for simple transformations. Use generator expressions when the result is consumed once or the sequence is large.

```python
# List comprehension
squares = [x**2 for x in range(10)]

# With condition
even_squares = [x**2 for x in range(10) if x % 2 == 0]

# Generator — no intermediate list in memory
total = sum(x**2 for x in range(1_000_000))

# Dict comprehension
word_lengths = {word: len(word) for word in ["alpha", "beta", "gamma"]}

# When NOT to use comprehensions: complex logic belongs in a named function
# Bad: unreadable nested comprehension
matrix = [[row[i] for row in matrix] for i in range(4)]
# Better: extract to a named function
```

## f-strings

Use f-strings for all string formatting. Avoid `%` formatting and `.format()` in new code.

```python
name = "Alice"
value = 3.14159

# Basic
greeting = f"Hello, {name}!"

# Format spec
formatted = f"{value:.2f}"          # "3.14"
padded = f"{name:>10}"              # right-aligned in 10 chars

# repr (useful for debugging)
debug = f"Got value: {name!r}"      # "Got value: 'Alice'"

# Multi-line f-string (use backslash continuation or parentheses)
message = (
    f"User {name!r} submitted value {value:.2f}. "
    f"Processing complete."
)
```

## Unpacking

Use unpacking to avoid index-based access. Indices are brittle; names are self-documenting.

```python
# Tuple unpacking
first, second, third = (1, 2, 3)

# Star unpacking — capture rest
head, *tail = [1, 2, 3, 4, 5]      # head=1, tail=[2,3,4,5]
*init, last = [1, 2, 3, 4, 5]      # init=[1,2,3,4], last=5

# Swap without temp variable
a, b = b, a

# Dict merge (Python 3.9+)
defaults = {"timeout": 30, "retries": 3}
overrides = {"retries": 5, "verify": True}
config = defaults | overrides       # {"timeout": 30, "retries": 5, "verify": True}

# Unpack in function call
def connect(host: str, port: int, timeout: int) -> None: ...
params = {"host": "localhost", "port": 5432, "timeout": 10}
connect(**params)
```

## enumerate and zip

Never use manual index variables. Always use `enumerate` or `zip`.

```python
items = ["a", "b", "c"]

# enumerate — index and value
for i, item in enumerate(items):
    print(f"{i}: {item}")

# enumerate with start
for i, item in enumerate(items, start=1):
    print(f"{i}: {item}")

# zip — pair two sequences
names = ["Alice", "Bob"]
scores = [95, 87]
for name, score in zip(names, scores):
    print(f"{name}: {score}")

# zip with strict=True (Python 3.10+) — raises ValueError if lengths differ
for name, score in zip(names, scores, strict=True):
    print(f"{name}: {score}")
```

## Named Tuples and Dataclasses

Prefer over plain tuples when fields have meaning.

```python
from typing import NamedTuple

class Coordinate(NamedTuple):
    latitude: float
    longitude: float
    altitude: float = 0.0

coord = Coordinate(48.8566, 2.3522)
print(coord.latitude)   # clear; not coord[0]
```

## Dictionary Operations

```python
data = {"key": "value"}

# Safe access with default
val = data.get("missing_key", "default")

# setdefault — initialize if absent
data.setdefault("counts", {})["page_views"] = 1

# Iterate key-value pairs
for key, value in data.items():
    print(f"{key}={value}")

# Dict comprehension with filter
filtered = {k: v for k, v in data.items() if v is not None}
```

## Exception Handling Patterns

```python
import logging

logger = logging.getLogger(__name__)

# Catch specific exceptions
try:
    result = int(user_input)
except (TypeError, ValueError) as e:
    logger.warning("Invalid input: %s", e)
    raise ValidationError(f"Expected integer, got {user_input!r}") from e

# Exception chaining — always chain when converting exception types
try:
    record = db.fetch(user_id)
except DatabaseConnectionError as e:
    raise ServiceUnavailableError("User service is down") from e

# contextlib.suppress — only for truly ignorable exceptions
import contextlib
with contextlib.suppress(FileNotFoundError):
    os.remove(tmp_file)   # OK if file already gone
```

## Generators

Use generators for lazy evaluation of large or infinite sequences.

```python
def read_lines(path: str):
    """Yield lines without loading the entire file into memory."""
    with open(path) as f:
        for line in f:
            yield line.rstrip()

# Generator expression
total = sum(len(line) for line in read_lines("large_file.txt"))

# Pipeline of generators — each stage is lazy
def parse(lines):
    for line in lines:
        if line and not line.startswith("#"):
            yield line.split(",")

def to_dicts(rows):
    headers = next(rows)
    for row in rows:
        yield dict(zip(headers, row))
```

## Walrus Operator `:=`

Use sparingly. Appropriate when the same expression would otherwise be computed twice.

```python
import re

# Avoids calling re.search() twice
if m := re.search(r"\d+", text):
    print(f"Found number: {m.group()}")

# Avoids reading a chunk twice in a loop
while chunk := f.read(8192):
    process(chunk)
```

---

## Anti-Patterns — What NOT To Do

### `except: pass` — Never swallow exceptions

```python
# BAD — silent failure; bugs become invisible
try:
    result = risky_operation()
except:
    pass

# BAD — slightly less wrong, still swallows errors
try:
    result = risky_operation()
except Exception:
    pass

# GOOD — log and re-raise, or handle explicitly
try:
    result = risky_operation()
except ValueError as e:
    logger.error("Unexpected value: %s", e)
    raise
```

### Mutable default arguments

```python
# BAD — the list is shared across all calls
def append_item(item, collection=[]):
    collection.append(item)
    return collection

# GOOD — use None sentinel
def append_item(item, collection=None):
    if collection is None:
        collection = []
    collection.append(item)
    return collection
```

### Wildcard imports

```python
# BAD — pollutes namespace, hides dependencies
from os.path import *

# GOOD — explicit imports
from os.path import join, exists, dirname
```

### Type checking with `type()` instead of `isinstance()`

```python
# BAD — misses subclasses
if type(x) == int:
    ...

# GOOD — handles subclasses correctly
if isinstance(x, int):
    ...
```

### Checking length instead of truthiness

```python
# BAD
if len(items) == 0:
    ...
if len(items) > 0:
    ...

# GOOD — sequences are falsy when empty
if not items:
    ...
if items:
    ...

# Exception: when you need to distinguish None from empty
if items is None:
    ...  # not provided at all
```

### Global mutable state

```python
# BAD — functions that read/write global state are untestable
_cache = {}

def get_user(user_id):
    if user_id not in _cache:
        _cache[user_id] = db.fetch(user_id)
    return _cache[user_id]

# GOOD — inject the cache (dependency injection)
def get_user(user_id: int, cache: dict[int, User]) -> User:
    if user_id not in cache:
        cache[user_id] = db.fetch(user_id)
    return cache[user_id]
```

## See Also

- wiki/tier3-working/python/functional-core.md
- wiki/tier3-working/python/type-system.md
- wiki/tier1-sources/python-peps/pep-008-style.md
- wiki/tier1-sources/python-peps/pep-020-zen.md

## Source

PEP 8 — Style Guide for Python Code. PEP 20 — The Zen of Python. Python documentation: dataclasses, contextlib, itertools. "Effective Python" (Brett Slatkin, 3rd ed.).
