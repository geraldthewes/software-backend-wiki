# Worked Example: Error Handling

> **Tier 3** | Source: PEP 20, Python docs | Enforces/Derives From: wiki/tier1-sources/python-peps/pep-020-zen.md, wiki/tier3-working/python/idioms.md, wiki/tier3-working/observability/structured-logging.md

## Summary

A complete, executable Python example demonstrating correct error handling patterns. Covers exception hierarchy design, specific exception handling, context managers for resource cleanup, logging errors correctly, and the Result type pattern for validation-heavy code.

---

## 1. PEP 20 Principle

> "Errors should never pass silently. Unless explicitly silenced."

In backend systems, silenced errors become data corruption, inconsistent state, and incidents that are discovered hours or days later through user complaints — not monitoring. Every exception must either be:
1. **Handled**: recovered from with explicit logic.
2. **Logged and re-raised**: recorded with context, then propagated.
3. **Converted**: wrapped into a domain-appropriate exception type.
4. **Truly suppressed**: with an explicit comment explaining why it is safe to ignore.

---

## 2. Exception Hierarchy Design

```python
# domain/exceptions.py


class AppError(Exception):
    """Base exception for all application errors.

    Catching AppError catches all known application errors without catching
    unexpected system errors (MemoryError, KeyboardInterrupt, etc.).
    """
    pass


class DatabaseError(AppError):
    """Raised when a database operation fails."""
    pass


class ConnectionError(DatabaseError):
    """Raised when a database connection cannot be established."""
    pass


class RecordNotFoundError(DatabaseError):
    """Raised when a required record does not exist."""

    def __init__(self, model: str, record_id: int) -> None:
        self.model = model
        self.record_id = record_id
        super().__init__(f"{model} with id={record_id} not found")


class ValidationError(AppError):
    """Raised when input data fails validation."""

    def __init__(self, field: str, message: str) -> None:
        self.field = field
        self.message = message
        super().__init__(f"Validation failed on field '{field}': {message}")


class AuthenticationError(AppError):
    """Raised when authentication fails."""
    pass


class AuthorizationError(AppError):
    """Raised when an authenticated user lacks permission."""

    def __init__(self, action: str, resource: str) -> None:
        self.action = action
        self.resource = resource
        super().__init__(f"Not authorized to {action} {resource}")
```

---

## 3. Explicit Error Handling

```python
# services/user_service.py
import logging
from domain.exceptions import RecordNotFoundError, ValidationError, DatabaseError

logger = logging.getLogger(__name__)


def get_user_profile(user_id: int, repo) -> dict:
    """Fetch user profile. Demonstrates correct exception handling."""
    try:
        user = repo.get(user_id)
        return {"id": user.id, "name": user.name, "email": user.email}

    except RecordNotFoundError:
        # Re-raise domain exceptions — caller decides how to handle
        raise

    except DatabaseError as e:
        # Convert low-level DB error to domain error with context
        # Exception chaining preserves the original traceback
        raise DatabaseError(
            f"Failed to fetch user {user_id} from database"
        ) from e


def register_user(name: str, email: str, repo) -> dict:
    """Register a new user with explicit validation."""
    # Validate each field separately — gives specific error messages
    name = name.strip()
    if not name:
        raise ValidationError(field="name", message="must not be empty")

    email = email.strip().lower()
    if "@" not in email:
        raise ValidationError(field="email", message="must be a valid email address")

    try:
        user = repo.save_new(name=name, email=email)
        logger.info("user_registered", extra={"user_id": user.id})
        return {"id": user.id, "name": user.name, "email": user.email}

    except DatabaseError as e:
        logger.error(
            "user_registration_failed",
            extra={"email": email, "error": str(e)},
            exc_info=True,
        )
        raise DatabaseError("Failed to save new user") from e
```

---

## 4. Anti-Patterns — What NOT to Do

```python
# BAD EXAMPLES — do not copy

# Anti-pattern 1: bare except swallows all exceptions
def bad_fetch_1(user_id: int, repo):
    try:
        return repo.get(user_id)
    except:          # catches KeyboardInterrupt, MemoryError, SystemExit
        pass         # silent failure — caller receives None with no explanation

# Anti-pattern 2: catch Exception without logging
def bad_fetch_2(user_id: int, repo):
    try:
        return repo.get(user_id)
    except Exception:
        return None  # error discarded — impossible to investigate later

# Anti-pattern 3: log AND re-raise — causes duplicate log entries
def bad_fetch_3(user_id: int, repo):
    try:
        return repo.get(user_id)
    except DatabaseError as e:
        logger.error("db error: %s", e)   # logged here
        raise                             # and logged again by caller — pick ONE

# Anti-pattern 4: raise new exception without chaining
def bad_convert(user_id: int, repo):
    try:
        return repo.get(user_id)
    except DatabaseError:
        raise AppError("fetch failed")   # original traceback is LOST
        # CORRECT: raise AppError("fetch failed") from e
```

---

## 5. Context Managers for Resource Cleanup

```python
# infrastructure/resources.py
import contextlib
import logging
from typing import Generator

logger = logging.getLogger(__name__)


@contextlib.contextmanager
def managed_transaction(conn) -> Generator:
    """Ensure a database transaction is committed or rolled back."""
    try:
        yield conn
        conn.commit()
        logger.debug("transaction_committed")
    except Exception:
        conn.rollback()
        logger.warning("transaction_rolled_back")
        raise  # always re-raise — do not swallow


@contextlib.contextmanager
def timed_operation(operation_name: str) -> Generator:
    """Log duration of a named operation."""
    import time
    start = time.perf_counter()
    try:
        yield
    finally:
        # finally runs even if an exception was raised
        duration = time.perf_counter() - start
        logger.info("operation_complete", extra={
            "operation": operation_name,
            "duration_ms": round(duration * 1000, 1),
        })


# Usage: both resources cleaned up even if an exception is raised
def transfer_funds(src_id: int, dst_id: int, amount: float, conn) -> None:
    with managed_transaction(conn):
        with timed_operation("fund_transfer"):
            debit_account(conn, src_id, amount)
            credit_account(conn, dst_id, amount)

# contextlib.suppress — use ONLY for truly ignorable exceptions
import os

def cleanup_temp_file(path: str) -> None:
    with contextlib.suppress(FileNotFoundError):
        os.remove(path)
    # If file already removed: fine — we wanted it gone
    # Any other exception (PermissionError, IsADirectoryError): propagates normally
```

---

## 6. Logging Errors Correctly

```python
# The correct tools for logging exceptions:

import logging
logger = logging.getLogger(__name__)

# logger.exception() — includes full traceback automatically
# Use when handling an exception you're NOT re-raising
try:
    result = risky_io_operation()
except OSError:
    logger.exception("io_operation_failed", extra={"path": "/tmp/data"})
    return None   # handled; not re-raising

# logger.error() with exc_info=True — same traceback, explicit
try:
    result = risky_io_operation()
except OSError as e:
    logger.error("io_operation_failed", extra={"path": "/tmp/data", "error": str(e)}, exc_info=True)
    raise

# What context to include in error logs:
# - Operation being performed
# - Identifiers (user_id, order_id, file path) — not PII like email
# - The error message (not the raw exception type)
# - correlation_id (via structlog binding or LoggerAdapter)

# NEVER log:
# - Passwords or tokens
# - Full credit card numbers
# - Raw user input that may contain secrets
```

---

## 7. Result Type Pattern

For validation-heavy code where exceptions are too expensive or awkward (e.g., validating a batch of 10,000 records):

```python
# domain/result.py
from __future__ import annotations
from dataclasses import dataclass
from typing import TypeVar, Generic

T = TypeVar("T")


@dataclass
class Result(Generic[T]):
    """
    Lightweight alternative to exceptions for expected error conditions.
    Use when: validation in a hot path; batch processing where partial success is OK.
    Do NOT replace all exception handling with Result — use exceptions for unexpected errors.
    """
    ok: bool
    value: T | None
    error: str | None

    @classmethod
    def success(cls, value: T) -> "Result[T]":
        return cls(ok=True, value=value, error=None)

    @classmethod
    def failure(cls, message: str) -> "Result[T]":
        return cls(ok=False, value=None, error=message)


# Usage: validate without raising exceptions
def validate_email(email: str) -> Result[str]:
    email = email.strip().lower()
    if not email:
        return Result.failure("email must not be empty")
    if "@" not in email:
        return Result.failure("email must contain @")
    parts = email.split("@")
    if len(parts) != 2 or "." not in parts[1]:
        return Result.failure("email domain must contain a dot")
    return Result.success(email)


def process_batch(emails: list[str]) -> tuple[list[str], list[dict]]:
    """Process a batch, collecting both successes and errors."""
    valid = []
    errors = []
    for i, email in enumerate(emails):
        result = validate_email(email)
        if result.ok:
            valid.append(result.value)
        else:
            errors.append({"index": i, "email": email, "error": result.error})
    return valid, errors


# Demonstration
if __name__ == "__main__":
    test_emails = ["alice@example.com", "not-an-email", "BOB@EXAMPLE.COM", ""]
    valid, errors = process_batch(test_emails)
    print(f"Valid: {valid}")
    print(f"Errors: {errors}")
    # Valid: ['alice@example.com', 'bob@example.com']
    # Errors: [{'index': 1, ...}, {'index': 3, ...}]
```

---

## See Also

- wiki/tier1-sources/python-peps/pep-020-zen.md
- wiki/tier3-working/python/idioms.md
- wiki/tier3-working/observability/structured-logging.md

## Source

PEP 20 — The Zen of Python. Python docs: exceptions, contextlib. "Robust Python" (Patrick Viafore, O'Reilly 2021). structlog documentation.
