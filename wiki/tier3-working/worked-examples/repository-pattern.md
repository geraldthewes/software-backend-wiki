# Worked Example: Repository Pattern

> **Tier 3** | Source: "Architecture Patterns with Python" | Enforces/Derives From: wiki/tier2-core/solid-principles/dip.md, wiki/tier2-core/solid-principles/srp.md, wiki/tier1-sources/owasp/a03-injection.md

## Summary

A complete, executable Python example demonstrating the repository pattern. Shows the bad starting point, the refactored solution, and tests — all runnable without a real database.

---

## 1. Problem Statement

`UserService` directly queries a SQLite database. This is the most common pattern in beginner code and the most common source of problems:

- **SRP violation**: the class handles business rules AND database I/O in the same place.
- **DIP violation**: `UserService` depends directly on `sqlite3` — a concrete implementation.
- **Untestable**: cannot test `UserService` without a real SQLite file on disk.
- **SQL injection**: f-string interpolation in SQL is exploitable.
- **OCP violation**: switching from SQLite to PostgreSQL requires modifying `UserService`.

---

## 2. Bad Code

```python
import sqlite3

class UserService:
    def __init__(self):
        # Hardcoded dependency on sqlite3 — cannot be swapped for testing
        self.conn = sqlite3.connect("app.db")

    def get_user(self, user_id: int):
        cursor = self.conn.cursor()
        # SQL INJECTION: attacker can pass user_id="1 OR 1=1"
        cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")
        return cursor.fetchone()

    def create_user(self, name: str, email: str):
        cursor = self.conn.cursor()
        # SQL INJECTION: attacker controls name and email
        cursor.execute(
            f"INSERT INTO users (name, email) VALUES ('{name}', '{email}')"
        )
        self.conn.commit()

    def validate_email(self, email: str) -> bool:
        # Business logic buried next to SQL
        return "@" in email and "." in email.split("@")[-1]
```

---

## 3. Refactored Solution

### Domain Model

```python
# domain/models.py
from dataclasses import dataclass

@dataclass(frozen=True)
class User:
    """Immutable domain object. No persistence logic."""
    id: int
    name: str
    email: str
```

### Repository Protocol (the abstraction)

```python
# domain/repositories.py
from typing import Protocol
from .models import User

class UserRepository(Protocol):
    """Abstraction over user persistence. Domain depends on this, not on sqlite3."""
    def get(self, user_id: int) -> User | None: ...
    def save(self, user: User) -> User: ...
    def list_all(self) -> list[User]: ...
    def find_by_email(self, email: str) -> User | None: ...
```

### SQLite Implementation

```python
# infrastructure/sqlite_user_repository.py
import sqlite3
from domain.models import User
from domain.repositories import UserRepository

class SQLiteUserRepository:
    """Concrete implementation using SQLite. Parameterized queries only."""

    def __init__(self, conn: sqlite3.Connection) -> None:
        self._conn = conn
        self._ensure_schema()

    def _ensure_schema(self) -> None:
        self._conn.execute(
            """CREATE TABLE IF NOT EXISTS users (
                id    INTEGER PRIMARY KEY AUTOINCREMENT,
                name  TEXT NOT NULL,
                email TEXT NOT NULL UNIQUE
            )"""
        )
        self._conn.commit()

    def get(self, user_id: int) -> User | None:
        cursor = self._conn.execute(
            "SELECT id, name, email FROM users WHERE id = ?",
            (user_id,),   # parameterized — injection impossible
        )
        row = cursor.fetchone()
        return User(id=row[0], name=row[1], email=row[2]) if row else None

    def save(self, user: User) -> User:
        if user.id == 0:
            cursor = self._conn.execute(
                "INSERT INTO users (name, email) VALUES (?, ?)",
                (user.name, user.email),
            )
            self._conn.commit()
            return User(id=cursor.lastrowid, name=user.name, email=user.email)
        else:
            self._conn.execute(
                "UPDATE users SET name = ?, email = ? WHERE id = ?",
                (user.name, user.email, user.id),
            )
            self._conn.commit()
            return user

    def list_all(self) -> list[User]:
        cursor = self._conn.execute("SELECT id, name, email FROM users")
        return [User(id=r[0], name=r[1], email=r[2]) for r in cursor.fetchall()]

    def find_by_email(self, email: str) -> User | None:
        cursor = self._conn.execute(
            "SELECT id, name, email FROM users WHERE email = ?",
            (email,),
        )
        row = cursor.fetchone()
        return User(id=row[0], name=row[1], email=row[2]) if row else None
```

### In-Memory Implementation (for tests)

```python
# infrastructure/in_memory_user_repository.py
from domain.models import User

class InMemoryUserRepository:
    """Fake repository backed by a dict. No files, no connections, runs instantly."""

    def __init__(self) -> None:
        self._store: dict[int, User] = {}
        self._next_id: int = 1

    def get(self, user_id: int) -> User | None:
        return self._store.get(user_id)

    def save(self, user: User) -> User:
        if user.id == 0:
            new_user = User(id=self._next_id, name=user.name, email=user.email)
            self._store[self._next_id] = new_user
            self._next_id += 1
            return new_user
        self._store[user.id] = user
        return user

    def list_all(self) -> list[User]:
        return list(self._store.values())

    def find_by_email(self, email: str) -> User | None:
        return next(
            (u for u in self._store.values() if u.email == email),
            None,
        )
```

### Domain Service (pure business logic, no SQL)

```python
# domain/user_service.py
import logging
from domain.models import User
from domain.repositories import UserRepository

logger = logging.getLogger(__name__)

class UserService:
    """Business logic only. No SQL, no I/O, no concrete dependencies."""

    def __init__(self, users: UserRepository) -> None:
        # DIP: depends on Protocol, not sqlite3
        self._users = users

    def register(self, name: str, email: str) -> User:
        """Register a new user. Raises ValueError on invalid input."""
        name = name.strip()
        email = email.strip().lower()

        if not name:
            raise ValueError("Name must not be empty")
        if not email or "@" not in email or "." not in email.split("@")[-1]:
            raise ValueError(f"Invalid email address: {email!r}")

        existing = self._users.find_by_email(email)
        if existing is not None:
            raise ValueError(f"Email {email!r} is already registered")

        new_user = User(id=0, name=name, email=email)
        saved = self._users.save(new_user)
        logger.info("user_registered", extra={"user_id": saved.id, "email": email})
        return saved

    def find(self, user_id: int) -> User:
        """Find a user by ID. Raises KeyError if not found."""
        user = self._users.get(user_id)
        if user is None:
            raise KeyError(f"User with id={user_id} not found")
        return user

    def list_users(self) -> list[User]:
        """Return all registered users."""
        return self._users.list_all()
```

---

## 4. Tests

These tests run in milliseconds. No database, no files, no network.

```python
# tests/test_user_service.py
import pytest
from domain.models import User
from domain.user_service import UserService
from infrastructure.in_memory_user_repository import InMemoryUserRepository


def make_service() -> tuple[UserService, InMemoryUserRepository]:
    repo = InMemoryUserRepository()
    service = UserService(users=repo)
    return service, repo


def test_register_creates_user_with_id():
    service, _ = make_service()
    user = service.register("Alice", "alice@example.com")
    assert user.id > 0
    assert user.name == "Alice"
    assert user.email == "alice@example.com"


def test_register_normalizes_email_to_lowercase():
    service, _ = make_service()
    user = service.register("Bob", "BOB@EXAMPLE.COM")
    assert user.email == "bob@example.com"


def test_register_strips_whitespace_from_name():
    service, _ = make_service()
    user = service.register("  Carol  ", "carol@example.com")
    assert user.name == "Carol"


def test_register_raises_on_invalid_email():
    service, _ = make_service()
    with pytest.raises(ValueError, match="Invalid email"):
        service.register("Dave", "not-an-email")


def test_register_raises_on_empty_name():
    service, _ = make_service()
    with pytest.raises(ValueError, match="Name must not be empty"):
        service.register("   ", "valid@example.com")


def test_register_raises_on_duplicate_email():
    service, _ = make_service()
    service.register("Alice", "alice@example.com")
    with pytest.raises(ValueError, match="already registered"):
        service.register("Alice2", "alice@example.com")


def test_find_returns_user_by_id():
    service, _ = make_service()
    created = service.register("Eve", "eve@example.com")
    found = service.find(created.id)
    assert found == created


def test_find_raises_when_user_not_found():
    service, _ = make_service()
    with pytest.raises(KeyError):
        service.find(999)


def test_list_users_returns_all_registered():
    service, _ = make_service()
    service.register("Alice", "alice@example.com")
    service.register("Bob", "bob@example.com")
    users = service.list_users()
    assert len(users) == 2
```

---

## 5. Principles Demonstrated

| Principle | How This Example Applies |
|-----------|--------------------------|
| **SRP** | `UserService` handles business rules; `SQLiteUserRepository` handles SQL; each has one reason to change |
| **DIP** | `UserService.__init__` accepts `UserRepository` (Protocol) — not `sqlite3.Connection` |
| **OCP** | Add `PostgreSQLUserRepository` or `MongoUserRepository` without modifying `UserService` |
| **Security (A03)** | All SQL uses `?` placeholders — injection is structurally impossible in the repository layer |
| **Testability** | `InMemoryUserRepository` enables full unit testing without any real I/O |

---

## See Also

- wiki/tier2-core/solid-principles/dip.md
- wiki/tier2-core/solid-principles/srp.md
- wiki/tier3-working/database-patterns/repository-pattern.md
- wiki/tier1-sources/owasp/a03-injection.md
- wiki/tier3-working/worked-examples/dependency-injection.md

## Source

"Architecture Patterns with Python" (Percival & Gregory, O'Reilly 2020). OWASP SQL Injection Prevention Cheat Sheet. SOLID Principles (Robert C. Martin).
