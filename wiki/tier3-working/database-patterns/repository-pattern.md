# Repository Pattern

> **Tier 3** | Source: "Architecture Patterns with Python", DDD | Enforces/Derives From: wiki/tier2-core/solid-principles/dip.md, wiki/tier2-core/solid-principles/srp.md, wiki/tier1-sources/owasp/a03-injection.md

## Summary

The repository pattern creates an abstraction layer between the domain model and data persistence. Domain logic speaks in domain objects; the repository handles SQL. This enforces SRP, enables DIP, and eliminates SQL injection by design.

## Problem

`UserService` that directly queries the database mixes concerns and is untestable:

```python
import sqlite3

class UserService:
    def __init__(self):
        # DIP violation: hardcoded dependency on concrete DB
        self.conn = sqlite3.connect("app.db")

    def get_user(self, user_id: int):
        # SRP violation: business logic AND data access in one class
        # Security violation: string formatting SQL (injection risk)
        cursor = self.conn.cursor()
        cursor.execute(
            f"SELECT * FROM users WHERE id = {user_id}"  # SQL INJECTION
        )
        return cursor.fetchone()

    def create_user(self, name: str, email: str):
        cursor = self.conn.cursor()
        cursor.execute(
            f"INSERT INTO users (name, email) VALUES ('{name}', '{email}')"  # INJECTION
        )
        self.conn.commit()
```

Problems:
- Cannot test `UserService` without a real SQLite file.
- SQL injection via f-string formatting.
- SRP violation: one class handles business logic and persistence.
- DIP violation: `UserService` depends on the concrete `sqlite3` module.
- OCP violation: switching to PostgreSQL requires modifying `UserService`.

## Refactored Solution

### 1. Domain Model (pure data, no persistence)

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class User:
    id: int
    name: str
    email: str
```

### 2. Repository Protocol (the abstraction)

```python
from typing import Protocol

class UserRepository(Protocol):
    def get(self, user_id: int) -> User | None: ...
    def save(self, user: User) -> User: ...
    def list_all(self) -> list[User]: ...
```

### 3. SQLite Implementation

```python
import sqlite3

class SQLiteUserRepository:
    def __init__(self, conn: sqlite3.Connection) -> None:
        self._conn = conn

    def get(self, user_id: int) -> User | None:
        cursor = self._conn.cursor()
        cursor.execute(
            "SELECT id, name, email FROM users WHERE id = ?",
            (user_id,),  # parameterized — injection impossible
        )
        row = cursor.fetchone()
        if row is None:
            return None
        return User(id=row[0], name=row[1], email=row[2])

    def save(self, user: User) -> User:
        cursor = self._conn.cursor()
        cursor.execute(
            "INSERT OR REPLACE INTO users (id, name, email) VALUES (?, ?, ?)",
            (user.id, user.name, user.email),
        )
        self._conn.commit()
        return user

    def list_all(self) -> list[User]:
        cursor = self._conn.cursor()
        cursor.execute("SELECT id, name, email FROM users")
        return [User(id=r[0], name=r[1], email=r[2]) for r in cursor.fetchall()]
```

### 4. In-Memory Implementation (for unit tests)

```python
class InMemoryUserRepository:
    def __init__(self) -> None:
        self._store: dict[int, User] = {}
        self._next_id: int = 1

    def get(self, user_id: int) -> User | None:
        return self._store.get(user_id)

    def save(self, user: User) -> User:
        if user.id == 0:
            user = User(id=self._next_id, name=user.name, email=user.email)
            self._next_id += 1
        self._store[user.id] = user
        return user

    def list_all(self) -> list[User]:
        return list(self._store.values())
```

### 5. Domain Service (depends on Protocol, not concrete type)

```python
class UserService:
    def __init__(self, users: UserRepository) -> None:
        # DIP: depends on the Protocol abstraction
        self._users = users

    def register(self, name: str, email: str) -> User:
        """Business logic only — no SQL here."""
        if not email or "@" not in email:
            raise ValueError(f"Invalid email: {email!r}")
        new_user = User(id=0, name=name.strip(), email=email.lower())
        return self._users.save(new_user)

    def find(self, user_id: int) -> User:
        user = self._users.get(user_id)
        if user is None:
            raise KeyError(f"User {user_id} not found")
        return user
```

## Tests Using InMemoryUserRepository

```python
import pytest

def test_register_creates_user():
    repo = InMemoryUserRepository()
    service = UserService(users=repo)

    user = service.register("Alice", "alice@example.com")

    assert user.name == "Alice"
    assert user.email == "alice@example.com"
    assert user.id > 0

def test_register_normalizes_email():
    repo = InMemoryUserRepository()
    service = UserService(users=repo)

    user = service.register("Bob", "BOB@EXAMPLE.COM")
    assert user.email == "bob@example.com"

def test_register_rejects_invalid_email():
    repo = InMemoryUserRepository()
    service = UserService(users=repo)

    with pytest.raises(ValueError, match="Invalid email"):
        service.register("Charlie", "not-an-email")

def test_find_raises_when_user_missing():
    repo = InMemoryUserRepository()
    service = UserService(users=repo)

    with pytest.raises(KeyError):
        service.find(999)
```

No database required. No mocks. Tests run in microseconds.

## Principles Demonstrated

| Principle | How It Applies |
|-----------|---------------|
| **SRP** | `UserService` handles business logic; `SQLiteUserRepository` handles persistence |
| **DIP** | `UserService` depends on `UserRepository` Protocol, not `sqlite3` |
| **OCP** | Adding `PostgreSQLUserRepository` requires no changes to `UserService` |
| **Security** | Parameterized queries in the repository make SQL injection structurally impossible |

## Python ORMs and the Repository Pattern

When using SQLAlchemy, the ORM model should stay inside the repository. The domain model passed to and from the service should be a plain dataclass, not an ORM model.

```
Domain Layer:  User (dataclass)
               UserRepository (Protocol)
               UserService (business logic)

Data Layer:    SQLAlchemy UserModel (ORM)
               SQLAlchemyUserRepository (maps ORM → domain model)
```

Never let SQLAlchemy `Session` or ORM model objects leak into domain code.

## See Also

- wiki/tier3-working/worked-examples/repository-pattern.md
- wiki/tier2-core/solid-principles/dip.md
- wiki/tier2-core/solid-principles/srp.md
- wiki/tier3-working/database-patterns/query-optimization.md
- wiki/tier1-sources/owasp/a03-injection.md

## Source

"Architecture Patterns with Python" (Percival & Gregory, O'Reilly 2020). "Domain-Driven Design" (Evans, 2003). OWASP: SQL Injection Prevention Cheat Sheet.
