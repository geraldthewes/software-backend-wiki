# Structural Design Patterns

> **Tier 2** | Source: Gang of Four (1994) | Derives From: ka03-design | Authority: established practice

## Summary

Structural patterns describe how to compose classes and objects into larger structures. They focus on simplifying the structure by identifying relationships. Python's dynamic typing and special methods make many structural patterns more natural than their Java equivalents.

---

## 1. Adapter

### Intent
Convert the interface of a class into another interface that clients expect. Allows incompatible interfaces to work together.

### Python Implementation

Wrap the adaptee in a class that implements the target `Protocol`.

```python
from typing import Protocol

class PaymentGateway(Protocol):
    """Target interface the application expects."""
    def charge(self, amount_cents: int, token: str) -> str: ...

class LegacyPaymentSystem:
    """Existing class with an incompatible interface."""
    def process_payment(self, dollars: float, card_token: str) -> dict:
        return {"transaction_id": "TXN123", "status": "ok"}

class LegacyPaymentAdapter:
    """Adapter: wraps LegacyPaymentSystem to implement PaymentGateway."""
    def __init__(self, legacy: LegacyPaymentSystem) -> None:
        self._legacy = legacy

    def charge(self, amount_cents: int, token: str) -> str:
        dollars = amount_cents / 100.0
        result = self._legacy.process_payment(dollars, token)
        return result["transaction_id"]
```

### When to Use
- Integrating third-party libraries with your domain protocol
- Making a legacy class conform to a new interface without modifying it

---

## 2. Facade

### Intent
Provide a simplified interface to a complex subsystem.

### Python Implementation

A module-level function or a thin class that delegates to several complex classes.

```python
# Complex subsystem classes
class VideoDecoder: ...
class AudioDecoder: ...
class SubtitleParser: ...
class MediaRenderer: ...

# Facade: simple interface to the complex subsystem
def play_media_file(file_path: str) -> None:
    """Simplified interface — caller doesn't need to know about the subsystem."""
    video = VideoDecoder().decode(file_path)
    audio = AudioDecoder().decode(file_path)
    subtitles = SubtitleParser().parse(file_path)
    renderer = MediaRenderer()
    renderer.render(video=video, audio=audio, subtitles=subtitles)
```

### When to Use
- Providing a simple entry point to a complex library or subsystem
- Reducing coupling between clients and a complex internal API
- Layered architecture: the facade is the boundary between layers

---

## 3. Decorator

### Intent
Attach additional behavior to an object dynamically without subclassing.

### Python Implementation

Python's `@decorator` syntax directly implements this pattern. Use `functools.wraps` to preserve the wrapped function's metadata.

```python
import functools
import time
import logging

logger = logging.getLogger(__name__)

def timed(func):
    """Decorator: adds timing behavior without modifying the function."""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        logger.info("function_timed", function=func.__name__, elapsed_ms=elapsed * 1000)
        return result
    return wrapper

def retry_on_failure(max_attempts: int = 3):
    """Parametrized decorator: adds retry behavior."""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as exc:
                    if attempt == max_attempts:
                        raise
                    logger.warning("retry_attempt", attempt=attempt, error=str(exc))
        return wrapper
    return decorator

@timed
@retry_on_failure(max_attempts=3)
def fetch_user(user_id: int) -> dict:
    ...
```

**Class Decorators** add behavior to an entire class:

```python
def register_handler(registry: dict):
    """Class decorator: registers a handler class in a registry."""
    def decorator(cls):
        registry[cls.event_type] = cls
        return cls
    return decorator
```

### When to Use
- Cross-cutting concerns: logging, timing, caching, retry, authentication
- Adding behavior to existing functions without modifying them
- Prefer function decorators over class decorators for simple behavior wrapping

---

## 4. Proxy

### Intent
Provide a surrogate or placeholder for another object to control access to it.

### Python Implementation

Use `__getattr__` delegation to transparently wrap an object.

```python
class CachingProxy:
    """Proxy: caches results of an expensive operation."""
    def __init__(self, target) -> None:
        self._target = target
        self._cache: dict = {}

    def __getattr__(self, name: str):
        attr = getattr(self._target, name)
        if callable(attr):
            @functools.wraps(attr)
            def cached_call(*args, **kwargs):
                key = (name, args, tuple(sorted(kwargs.items())))
                if key not in self._cache:
                    self._cache[key] = attr(*args, **kwargs)
                return self._cache[key]
            return cached_call
        return attr
```

### When to Use
- **Lazy initialization**: defer expensive object creation until first use
- **Caching**: cache results of expensive method calls
- **Access control**: add authorization checks before delegating to the real object
- **Remote proxy**: represent a remote object locally (gRPC client stub is an example)

---

## 5. Composite

### Intent
Compose objects into tree structures to represent part-whole hierarchies. Treat individual objects and compositions uniformly.

### Python Implementation

Implement `__iter__` on container types to allow uniform traversal.

```python
from typing import Iterator

class FileSystemItem:
    def __init__(self, name: str) -> None:
        self.name = name

    def size(self) -> int:
        raise NotImplementedError

class File(FileSystemItem):
    def __init__(self, name: str, size_bytes: int) -> None:
        super().__init__(name)
        self._size = size_bytes

    def size(self) -> int:
        return self._size

    def __iter__(self) -> Iterator["FileSystemItem"]:
        yield self  # leaf node iterates only itself

class Directory(FileSystemItem):
    def __init__(self, name: str) -> None:
        super().__init__(name)
        self._children: list[FileSystemItem] = []

    def add(self, item: FileSystemItem) -> None:
        self._children.append(item)

    def size(self) -> int:
        return sum(child.size() for child in self._children)

    def __iter__(self) -> Iterator[FileSystemItem]:
        yield self
        for child in self._children:
            yield from child  # recursive traversal
```

### When to Use
- Tree structures: file systems, organization charts, UI component hierarchies, expression trees
- When clients should treat leaves and branches uniformly

---

## 6. Bridge

### Intent
Separate an abstraction from its implementation so both can vary independently.

### Python Implementation

Use composition with `Protocol`: the abstraction holds a reference to an implementation protocol.

```python
from typing import Protocol

class Renderer(Protocol):
    """Implementation: how to render."""
    def render_text(self, text: str) -> str: ...
    def render_image(self, path: str) -> str: ...

class HtmlRenderer:
    def render_text(self, text: str) -> str:
        return f"<p>{text}</p>"
    def render_image(self, path: str) -> str:
        return f'<img src="{path}">'

class MarkdownRenderer:
    def render_text(self, text: str) -> str:
        return text
    def render_image(self, path: str) -> str:
        return f"![image]({path})"

class ReportDocument:
    """Abstraction: uses a renderer without knowing its type."""
    def __init__(self, renderer: Renderer) -> None:
        self._renderer = renderer

    def generate(self, title: str, image_path: str) -> str:
        return (
            self._renderer.render_text(title)
            + "\n"
            + self._renderer.render_image(image_path)
        )
```

### When to Use
- Abstraction and implementation should be extensible independently
- Avoiding a combinatorial explosion of subclasses (2 abstractions × 3 implementations = 2 classes + 3 implementations, not 6 subclasses)

---

## 7. Flyweight

### Intent
Use sharing to support large numbers of fine-grained objects efficiently.

### Python Implementation

Use `__slots__` for memory-efficient objects. Use interning or a registry for shared instances.

```python
import sys

class Point:
    """Without __slots__: each instance carries a __dict__."""
    def __init__(self, x: float, y: float) -> None:
        self.x = x
        self.y = y

class EfficientPoint:
    """With __slots__: no __dict__ per instance — 3-5× less memory."""
    __slots__ = ("x", "y")

    def __init__(self, x: float, y: float) -> None:
        self.x = x
        self.y = y

# For shared instances: use a registry (flyweight factory)
_point_cache: dict[tuple, "EfficientPoint"] = {}

def get_point(x: float, y: float) -> EfficientPoint:
    key = (x, y)
    if key not in _point_cache:
        _point_cache[key] = EfficientPoint(x, y)
    return _point_cache[key]
```

### When to Use
- Creating very large numbers of similar objects (e.g., rendering thousands of map tiles, game sprites, or UI elements)
- Memory is a constraint and object state is largely shared
- `__slots__` is useful whenever a class will have many instances, even without full flyweight sharing

---

## Agent Guidance

### Do
- Use Adapter to integrate third-party libraries without polluting domain code
- Use Facade at layer boundaries — it is the simplest way to enforce separation of concerns
- Always use `functools.wraps` in function decorators to preserve metadata
- Use `__slots__` on data-heavy classes with many instances

### Do Not
- Do not use Proxy to add business logic — use it only for cross-cutting concerns (caching, logging, access control)
- Do not create deep Composite hierarchies unless the domain genuinely requires tree structure
- Do not use Flyweight prematurely — profile first to confirm memory is the bottleneck

## Checklist
- [ ] Third-party library integrations go through an Adapter implementing a domain Protocol
- [ ] Layer boundaries use Facade to expose a simple interface
- [ ] Function decorators use `@functools.wraps`
- [ ] Data-heavy classes with many instances use `__slots__`

## See Also
- `wiki/tier2-core/design-patterns/overview.md`
- `wiki/tier2-core/design-patterns/creational.md`
- `wiki/tier2-core/design-patterns/behavioral.md`
- `wiki/tier2-core/solid-principles/dip.md`

## Source

Gamma et al., *Design Patterns* (1994). Synthesized from *Software Development Best Practices for Agent* reference document.
