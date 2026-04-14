# Python Secure Coding Guidelines (Pyscg)

> **Tier 2** | Source: OpenSSF Pyscg | Derives From: ka13-security, owasp/a03-injection | Authority: established practice

## Summary

The OpenSSF Python Secure Coding Guidelines (Pyscg) provide specific, actionable rules for avoiding common Python security vulnerabilities. Each rule has a rule ID (pyscg-XXXX), addresses a specific CWE, and maps to a concrete coding practice. These rules apply during code construction — apply them to every function that handles user input, external data, secrets, or system resources.

---

## pyscg-0010 — Parameterized Queries

**CWE**: CWE-89 (SQL Injection)

SQL injection is the most exploited web vulnerability. It occurs when user-controlled data is interpolated directly into SQL strings. The database executes the injected SQL as code.

### Bad — SQL Injection Vulnerability

```python
def get_user(user_id: str) -> dict:
    query = f"SELECT * FROM users WHERE id = {user_id}"
    cursor.execute(query)  # user_id = "1 OR 1=1" dumps the entire table
    return cursor.fetchone()
```

### Good — Parameterized Query

```python
def get_user(user_id: int) -> dict:
    cursor.execute(
        "SELECT * FROM users WHERE id = %s",
        (user_id,),  # always a tuple, even for one parameter
    )
    return cursor.fetchone()
```

The database driver separates the query structure from the data. No amount of malicious input in `user_id` can alter the SQL structure.

**Also applies to**: ORM `raw()` methods, NoSQL queries with string interpolation, shell commands with user input (`subprocess.run` with `shell=True`).

---

## pyscg-0019 — Sensitive Data in Logs

**CWE**: CWE-532 (Insertion of Sensitive Information into Log File)

Log files are often stored in plain text, forwarded to log aggregators, and retained for months. Sensitive data in logs becomes an information disclosure vulnerability.

### What to NEVER Log

- Passwords (including "wrong password" events)
- API keys, tokens, secrets
- Credit card numbers, CVV codes
- Social Security Numbers, national IDs
- Full session tokens
- Private keys or certificates

### Bad — Sensitive Data Exposed in Logs

```python
import logging
logger = logging.getLogger(__name__)

def authenticate(username: str, password: str) -> bool:
    logger.debug(f"Authenticating {username} with password {password}")  # NEVER
    ...
```

### Good — Filtered Logging

```python
import logging
import structlog

logger = structlog.get_logger()

def authenticate(username: str, password: str) -> bool:
    logger.info("authentication_attempt", username=username)  # log username, never password
    result = check_credentials(username, password)
    logger.info("authentication_result", username=username, success=result)
    return result
```

### Using `logging.Filter` for Automatic Scrubbing

```python
class SensitiveDataFilter(logging.Filter):
    SENSITIVE_KEYS = {"password", "token", "secret", "api_key", "authorization"}

    def filter(self, record: logging.LogRecord) -> bool:
        if isinstance(record.args, dict):
            record.args = {
                k: "***REDACTED***" if k.lower() in self.SENSITIVE_KEYS else v
                for k, v in record.args.items()
            }
        return True
```

---

## pyscg-0023 — Avoid Pickle on Untrusted Data

**CWE**: CWE-502 (Deserialization of Untrusted Data)

Python's `pickle` module can execute arbitrary Python code during deserialization. Any attacker who controls the pickled data can achieve remote code execution.

### Bad — Pickle Deserialization of Untrusted Data

```python
import pickle

def load_user_preferences(data: bytes) -> dict:
    return pickle.loads(data)  # RCE if data comes from an untrusted source
```

### Good — JSON for External Data

```python
import json

def load_user_preferences(data: bytes) -> dict:
    return json.loads(data)  # JSON cannot execute code
```

**Rule**: Only ever use `pickle` on data you generated yourself — never on data received from users, external APIs, databases populated by external parties, or message queues that external parties can write to. Use JSON, Protocol Buffers, or MessagePack for external data.

---

## pyscg-0034 — None Checks

**CWE**: CWE-476 (NULL Pointer Dereference)

Accessing attributes on `None` raises `AttributeError`. In Python, `None` is a valid return value from many functions — never assume a return value is non-None without checking.

### Bad — Unguarded Attribute Access

```python
def get_user_email(user_id: int) -> str:
    user = find_user(user_id)  # returns None if not found
    return user.email  # AttributeError if user is None
```

### Good — Explicit None Check

```python
from typing import Optional

def get_user_email(user_id: int) -> Optional[str]:
    user = find_user(user_id)
    if user is None:
        return None
    return user.email
```

Use `Optional[T]` type annotations to propagate None awareness through the type system. Run `mypy` to catch unguarded None access statically.

---

## pyscg-0035, pyscg-0052 — Resource Cleanup

**CWE**: CWE-772 (Missing Release of Resource after Effective Lifetime)

Files, database connections, network sockets, and locks are finite resources. Failing to release them causes resource exhaustion — the system runs out of file descriptors or connections, and all subsequent operations fail.

### Bad — No Cleanup Guarantee

```python
def read_config(path: str) -> str:
    f = open(path)
    content = f.read()  # if an exception occurs here, f.close() is never called
    f.close()
    return content
```

### Good — Context Manager Guarantees Cleanup

```python
def read_config(path: str) -> str:
    with open(path) as f:
        return f.read()  # f.close() called even if an exception is raised

# Database connections
with psycopg.connect(DATABASE_URL) as conn:
    with conn.cursor() as cur:
        cur.execute("SELECT 1")

# Locks
import threading
lock = threading.Lock()
with lock:
    # critical section
    pass
```

**Rule**: always use `with` for files, database connections, network sockets, and locks. Never rely on `__del__` for cleanup — it is called at unpredictable times and may not be called at all in some Python implementations.

---

## pyscg-0037 — Do Not Use `assert` for Security Checks

**CWE**: CWE-617 (Reachable Assertion)

Python's `assert` statement is **removed entirely when Python is run with the `-O` (optimize) flag**. This is the default in many production deployments and Docker optimized images. Any security check implemented with `assert` is silently removed in production.

### Bad — Assert for Security

```python
def delete_user(requesting_user_id: int, target_user_id: int) -> None:
    assert requesting_user_id == target_user_id, "Cannot delete another user"
    # In production with python -O: assert is gone; any user can delete any user
    db.delete_user(target_user_id)
```

### Good — Explicit Security Check

```python
def delete_user(requesting_user_id: int, target_user_id: int) -> None:
    if requesting_user_id != target_user_id:
        raise PermissionError(
            f"User {requesting_user_id} cannot delete user {target_user_id}"
        )
    db.delete_user(target_user_id)
```

**Rule**: `assert` is for development-time invariants and programmer errors only. Use explicit `if not condition: raise` for all security-relevant checks.

---

## pyscg-0041 — Externalize Configuration and Secrets

**CWE**: CWE-798 (Use of Hard-coded Credentials)

Hardcoded credentials in source code are exposed whenever the repository is accessed — by any developer, CI system, or code review tool. They cannot be rotated without a code change and deployment.

### Bad — Hardcoded Credentials

```python
DATABASE_URL = "postgresql://admin:SuperSecret123@prod-db.example.com/mydb"
API_KEY = "sk-live-abc123xyz789"
SECRET_KEY = "my-very-secret-key-do-not-share"
```

### Good — Environment Variables

```python
import os

DATABASE_URL: str = os.environ["DATABASE_URL"]   # required — crash if missing
API_KEY: str = os.environ["EXTERNAL_API_KEY"]     # required
SECRET_KEY: str = os.environ["SECRET_KEY"]        # required
LOG_LEVEL: str = os.environ.get("LOG_LEVEL", "INFO")  # optional with default
```

```bash
# .env.template — ALWAYS commit this
DATABASE_URL=postgres://user:password@localhost:5432/mydb
EXTERNAL_API_KEY=replace-with-real-key
SECRET_KEY=replace-with-real-secret
LOG_LEVEL=DEBUG
```

For production: use a secrets manager (HashiCorp Vault, AWS Secrets Manager, GCP Secret Manager). Never store production secrets in `.env` files on servers.

---

## Running bandit

`bandit` is a Python static analysis tool for security vulnerabilities. Run it on all Python code before marking it production-ready.

```bash
pip install bandit
bandit -r src/ -ll  # -ll = report medium and high severity only
bandit -r src/ -f json -o bandit-report.json  # for CI integration
```

Zero high-severity findings is the minimum bar. Address medium-severity findings unless there is explicit documented justification.

## Agent Guidance

### Do
- Apply all pyscg rules during initial implementation — retrofitting is harder
- Run `bandit` as part of CI
- Use `mypy` for None propagation tracking (catches pyscg-0034 violations statically)
- Use `.env.template` for documenting required environment variables

### Do Not
- Do not use f-strings or string formatting to build SQL queries
- Do not log any value that could be a credential, token, or PII
- Do not use `pickle` for any data from external sources
- Do not use `assert` for security-relevant checks
- Do not hardcode any secret or credential in source code

## Checklist
- [ ] All SQL uses parameterized queries (`%s` placeholders, not f-strings)
- [ ] Log statements contain no passwords, tokens, API keys, or PII
- [ ] `pickle` is used only for internally-generated data
- [ ] All resource handles (files, connections, sockets) use `with` statements
- [ ] No security checks use `assert`
- [ ] All secrets and config come from `os.environ`
- [ ] `bandit -r src/ -ll` passes with zero high-severity findings

## See Also
- `wiki/tier2-core/security-practices/overview.md`
- `wiki/tier2-core/security-practices/threat-modeling.md`
- `wiki/tier1-sources/swebok-v4/ka13-security.md`
- `wiki/tier1-sources/owasp/a03-injection.md`

## Source

OpenSSF Python Secure Coding Guidelines (Pyscg), https://github.com/ossf/wg-best-practices-os-developers. Synthesized from *Software Development Best Practices for Agent* reference document.
