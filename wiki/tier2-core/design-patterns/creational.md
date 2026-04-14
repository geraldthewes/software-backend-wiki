# Creational Design Patterns

> **Tier 2** | Source: Gang of Four (1994) | Derives From: ka03-design | Authority: established practice

## Summary

Creational patterns control how objects are created. They decouple the creation logic from the consuming code, enabling flexibility in what is created and how. Python's dynamic typing and first-class functions simplify many GoF creational patterns.

---

## 1. Factory Method

### Intent
Define an interface for creating an object, but let subclasses or callables decide which class to instantiate.

### Python Implementation

In Python, factory methods are typically `@classmethod` factories on the target class, or standalone module-level factory functions.

```python
from dataclasses import dataclass
from typing import Literal

@dataclass
class DatabaseConnection:
    host: str
    port: int
    database: str

    @classmethod
    def from_url(cls, url: str) -> "DatabaseConnection":
        """Factory method: parse URL into a DatabaseConnection."""
        from urllib.parse import urlparse
        parsed = urlparse(url)
        return cls(
            host=parsed.hostname or "localhost",
            port=parsed.port or 5432,
            database=parsed.path.lstrip("/"),
        )

    @classmethod
    def for_testing(cls) -> "DatabaseConnection":
        """Named factory for test environments."""
        return cls(host="localhost", port=5432, database="test_db")
```

### When to Use
- Creating an object from multiple different input formats (`from_url`, `from_dict`, `from_env`)
- Providing named constructors that communicate intent better than `__init__` parameter combinations

---

## 2. Abstract Factory

### Intent
Provide an interface for creating families of related objects without specifying their concrete classes.

### Python Implementation

Use a `Protocol` with factory methods. Implementations provide a consistent family of related objects.

```python
from typing import Protocol

class Button(Protocol):
    def render(self) -> str: ...

class Checkbox(Protocol):
    def render(self) -> str: ...

class UIFactory(Protocol):
    """Abstract factory: creates a family of UI components."""
    def create_button(self) -> Button: ...
    def create_checkbox(self) -> Checkbox: ...

class WebButton:
    def render(self) -> str:
        return "<button>Click me</button>"

class WebCheckbox:
    def render(self) -> str:
        return '<input type="checkbox">'

class WebUIFactory:
    def create_button(self) -> WebButton:
        return WebButton()

    def create_checkbox(self) -> WebCheckbox:
        return WebCheckbox()

def render_form(factory: UIFactory) -> str:
    button = factory.create_button()
    checkbox = factory.create_checkbox()
    return f"{checkbox.render()}\n{button.render()}"
```

### When to Use
- Multiple related implementations must be used together (e.g., a consistent UI theme, or a matching set of test doubles)
- Dependency injection of matching families of collaborators (see `dip.md`)

---

## 3. Builder

### Intent
Construct a complex object step by step. Separate the construction process from the representation.

### Python Implementation

Python's `@dataclass` with default values handles simple cases. Use a dedicated `Builder` class for objects with many optional parameters that have complex validation relationships.

```python
from dataclasses import dataclass, field
from typing import Optional

@dataclass
class HttpRequest:
    url: str
    method: str = "GET"
    headers: dict = field(default_factory=dict)
    body: Optional[bytes] = None
    timeout: float = 30.0
    follow_redirects: bool = True

class HttpRequestBuilder:
    """Builder for HttpRequest — avoids telescoping constructor calls."""

    def __init__(self, url: str) -> None:
        self._url = url
        self._method = "GET"
        self._headers: dict = {}
        self._body: Optional[bytes] = None
        self._timeout = 30.0
        self._follow_redirects = True

    def method(self, method: str) -> "HttpRequestBuilder":
        self._method = method
        return self

    def header(self, name: str, value: str) -> "HttpRequestBuilder":
        self._headers[name] = value
        return self

    def body(self, data: bytes) -> "HttpRequestBuilder":
        self._body = data
        return self

    def timeout(self, seconds: float) -> "HttpRequestBuilder":
        self._timeout = seconds
        return self

    def build(self) -> HttpRequest:
        return HttpRequest(
            url=self._url,
            method=self._method,
            headers=self._headers,
            body=self._body,
            timeout=self._timeout,
            follow_redirects=self._follow_redirects,
        )

# Usage
request = (
    HttpRequestBuilder("https://api.example.com/orders")
    .method("POST")
    .header("Content-Type", "application/json")
    .body(b'{"item": "book"}')
    .timeout(10.0)
    .build()
)
```

### When to Use
- Objects with many optional parameters where most defaults are acceptable most of the time
- When the construction order matters or validation across parameters is complex
- Avoids "telescoping constructors" (10-parameter `__init__` calls)

---

## 4. Singleton

### Intent
Ensure that a class has only one instance and provide a global access point to it.

### Python Implementation

**Preferred: module-level instance** — Python modules are singletons by design. Import the instance, not the class.

```python
# config.py
import os

class AppConfig:
    def __init__(self) -> None:
        self.database_url: str = os.environ["DATABASE_URL"]
        self.log_level: str = os.environ.get("LOG_LEVEL", "INFO")
        self.debug: bool = os.environ.get("DEBUG", "false").lower() == "true"

# Module-level singleton — Python module import is cached
config = AppConfig()
```

```python
# In any other module:
from myapp.config import config
print(config.database_url)
```

### WARNING: Singletons and Testing

Singletons are **global state**. Global state makes tests order-dependent and difficult to isolate. For any dependency that varies between test cases, prefer **dependency injection** (DIP) over singletons.

| Use Case | Prefer Singleton? | Prefer DI? |
|----------|------------------|-----------|
| Application config (read-only) | Yes | Optional |
| Database connection pool | No — inject the pool | Yes |
| Logger | Yes (structlog instance) | No |
| External service client | No — inject the client | Yes |

```python
# Anti-pattern: singleton used as a dependency
class OrderService:
    def save(self, order):
        DatabaseSingleton.get_instance().execute(...)  # untestable

# Preferred: inject the dependency
class OrderService:
    def __init__(self, db: DatabaseProtocol) -> None:
        self._db = db  # testable; swap with FakeDatabase in tests
```

### When to Use
- Read-only configuration objects that are expensive to construct
- Logging instances
- Never for stateful dependencies that need to be replaced in tests

---

## 5. Prototype

### Intent
Create new objects by copying an existing object (the prototype).

### Python Implementation

Use `copy.deepcopy()` for general objects or `dataclasses.replace()` for frozen dataclasses.

```python
import copy
from dataclasses import dataclass, replace

@dataclass(frozen=True)
class OrderTemplate:
    currency: str = "USD"
    tax_rate: float = 0.10
    shipping_method: str = "standard"
    priority: bool = False

# Base template
standard_order = OrderTemplate()

# Create variants by replacing specific fields — prototype pattern
express_order = replace(standard_order, shipping_method="express", priority=True)
eu_order = replace(standard_order, currency="EUR", tax_rate=0.20)

# For mutable objects: deepcopy
class MutableConfig:
    def __init__(self):
        self.settings = {"retries": 3, "timeout": 30}

base_config = MutableConfig()
test_config = copy.deepcopy(base_config)
test_config.settings["timeout"] = 1  # does not affect base_config
```

### When to Use
- Creating many similar objects that differ in only a few fields
- Avoiding expensive re-initialization by cloning a configured prototype
- `dataclasses.replace()` is the idiomatic Python prototype for frozen dataclasses

---

## Agent Guidance

### Do
- Prefer `@classmethod` factory methods over complex `__init__` overloads
- Use Builder when an object has more than five optional parameters
- Use module-level instances for true singletons (configuration, loggers)
- Use `dataclasses.replace()` for prototype cloning of frozen dataclasses

### Do Not
- Do not use Singleton for any dependency that needs to be replaced in tests
- Do not use Abstract Factory when one implementation is all that will ever exist
- Do not implement Builder for objects with only a few parameters — `@dataclass` with defaults is simpler

## Checklist
- [ ] Factory methods are named descriptively (`from_url`, `for_testing`, `from_env`)
- [ ] Builder is used for objects with > 5 optional parameters
- [ ] Singletons are read-only or logging-only; stateful dependencies are injected
- [ ] `dataclasses.replace()` is used for prototype cloning (not manual copying)

## See Also
- `wiki/tier2-core/design-patterns/overview.md`
- `wiki/tier2-core/design-patterns/structural.md`
- `wiki/tier2-core/design-patterns/behavioral.md`
- `wiki/tier2-core/solid-principles/dip.md`
- `wiki/tier1-sources/swebok-v4/ka01-requirements.md`

## Source

Gamma et al., *Design Patterns* (1994). Synthesized from *Software Development Best Practices for Agent* reference document.
