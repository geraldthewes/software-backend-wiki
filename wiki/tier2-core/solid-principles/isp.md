# Interface Segregation Principle (ISP)

> **Tier 2** | Source: Robert C. Martin | Derives From: ka03-design | Authority: established practice

## Summary

No client should be forced to depend on methods it does not use. Large, monolithic interfaces create unnecessary coupling: a change to one method forces recompilation (or re-testing) of all clients, even those that never call it. ISP directs you to design many small, focused interfaces rather than one large one.

## Key Concepts

### Definition

Clients should not be forced to depend on methods they do not use. When a class is required to implement an interface, it must implement all of the interface's methods — even those that are irrelevant to it. This creates coupling between the client and functionality it does not need.

### The Fat Interface Anti-Pattern

A "fat interface" is an `ABC` or `Protocol` with many methods that no single implementation genuinely needs in full. Signs of a fat interface:

- Implementations frequently raise `NotImplementedError` or return `None` for methods they do not support
- Mock objects in tests are cluttered with stub methods that are not relevant to the scenario under test
- A change to one method signature requires touching all implementations, even unrelated ones
- The interface name is generic: `DatabaseService`, `StorageBackend`, `FullService`

### Python-Specific Guidance

Python's `typing.Protocol` (PEP 544) makes ISP natural:

- Define `Protocol` classes with **five methods or fewer**
- Prefer multiple small `Protocol` classes over one large `ABC`
- Python's structural subtyping (duck typing) means a class satisfies a `Protocol` without explicitly declaring so — this makes composing small protocols effortless
- Use `runtime_checkable` sparingly; prefer static type checking (mypy / pyright)

### Bad Example — Fat Interface

```python
from abc import ABC, abstractmethod

class DatabaseService(ABC):
    """One interface with 7 methods — most implementations only need 2-3."""

    @abstractmethod
    def connect(self) -> None: ...

    @abstractmethod
    def disconnect(self) -> None: ...

    @abstractmethod
    def query(self, sql: str) -> list: ...

    @abstractmethod
    def execute(self, sql: str) -> None: ...

    @abstractmethod
    def migrate(self, version: int) -> None: ...

    @abstractmethod
    def backup(self, path: str) -> None: ...

    @abstractmethod
    def restore(self, path: str) -> None: ...
```

A read-only reporting service that only needs `query()` is forced to implement `migrate()`, `backup()`, and `restore()` — or leave them as stubs.

### Good Example — Segregated Interfaces

```python
from typing import Protocol

class Queryable(Protocol):
    """Read-only access: only what a query client needs."""
    def query(self, sql: str) -> list: ...

class Writable(Protocol):
    """Write access: only what a write client needs."""
    def execute(self, sql: str) -> None: ...

class Migratable(Protocol):
    """Schema management: only what a migration tool needs."""
    def migrate(self, version: int) -> None: ...

class Backupable(Protocol):
    """Backup operations: only what a backup agent needs."""
    def backup(self, path: str) -> None: ...
    def restore(self, path: str) -> None: ...
```

The reporting service depends only on `Queryable`. Its mock needs only one method. The migration tool depends only on `Migratable`. Changes to backup logic do not touch the reporting service.

### Testing Benefit

Small interfaces translate directly to small mocks:

```python
class FakeQueryable:
    def query(self, sql: str) -> list:
        return [{"id": 1, "name": "Alice"}]

def test_report_generation() -> None:
    service = ReportingService(db=FakeQueryable())
    result = service.generate_summary()
    assert result == "Summary: 1 user"
```

No `backup()`, `migrate()`, or `connect()` stub needed. The test is focused and readable.

### Relationship to DIP

ISP and DIP work together: DIP says "depend on abstractions"; ISP says "keep those abstractions small." When you inject a `Queryable` protocol rather than a concrete `PostgresDatabase`, you get both principles working in concert — the domain class is decoupled from infrastructure, and the interface is minimal.

## Agent Guidance

### Do
- Decompose large interfaces into role-based protocols (`Queryable`, `Writable`, `Migratable`)
- Keep each `Protocol` to five or fewer methods
- Let callers declare which small interface they need; provide a concrete class that satisfies all of them
- Favor `typing.Protocol` over `ABC` for interface segregation in Python (no explicit registration needed)

### Do Not
- Do not create a single `Protocol` or `ABC` that covers every operation a subsystem might support
- Do not force implementations to provide `raise NotImplementedError` stubs for irrelevant methods
- Do not design interfaces that a class only partially satisfies in practice
- Do not name interfaces generically (`FullService`, `AllOperations`) — name them by role

## Checklist
- [ ] No `Protocol` or `ABC` has more than five methods
- [ ] No implementation has `raise NotImplementedError` stubs for methods it does not support
- [ ] Each protocol is named after the role it represents, not the system it covers
- [ ] Test mocks are small — only the methods actually called in the test under review
- [ ] Concrete classes may satisfy multiple small protocols without being forced into one large one

## See Also
- `wiki/tier2-core/solid-principles/srp.md`
- `wiki/tier2-core/solid-principles/dip.md`
- `wiki/tier2-core/solid-principles/overview.md`
- `wiki/tier1-sources/swebok-v4/ka01-requirements.md`

## Source

Robert C. Martin, *Agile Software Development: Principles, Patterns, and Practices* (2002). PEP 544 — Protocols: Structural subtyping (static duck typing). Synthesized from *Software Development Best Practices for Agent* reference document.
