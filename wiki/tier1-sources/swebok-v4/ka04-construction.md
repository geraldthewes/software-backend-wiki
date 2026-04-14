# SWEBOK V4 — KA 04: Software Construction

> **Tier 1** | Source: IEEE SWEBOK V4, KA 04; PEP 8; OpenSSF Pyscg | Authority: immutable

## Summary

Software Construction is the Knowledge Area covering the practical creation of working software: coding, unit verification, integration of components, debugging, and the management of technical trade-offs during implementation. In SWEBOK V4, Construction is KA 04, and it is deliberately defined more broadly than "programming." Programming is a sub-activity within Construction; Construction also encompasses anticipating change, managing complexity, adhering to coding standards, and ensuring that built software can be maintained by others.

For a coding agent, Construction is where principles from Requirements (KA 01), Architecture (KA 02), and Design (KA 03) are realized in code. The agent must produce code that is not only functionally correct but readable, type-safe, statically analyzable, and compliant with language standards (PEP 8 for Python). SWEBOK V4 integrates Agile construction practices — continuous integration, short feedback cycles — and DevOps automation as mandatory, not optional.

## Key Concepts

### Construction vs. Programming

The distinction is essential for an agent's self-understanding:

| Concept | Programming | Software Construction (KA 04) |
|---------|------------|------------------------------|
| Scope | Writing code to implement a function | Writing code + verifying it + integrating it + managing complexity throughout |
| Primary concern | Does the function work? | Does the function work, is it maintainable, and does it fit the architecture? |
| Mindset | Solve the immediate problem | Anticipate change; minimize complexity for the next engineer |
| Standards | Informal | PEP 8, type hints, static analysis, code review |

Programming is necessary but not sufficient for professional software construction.

### Why Minimizing Complexity Is a First-Order Goal

Software maintenance accounts for 40–80% of total lifecycle cost. Every complexity introduced during construction is a maintenance cost multiplied over years. Key sources of complexity to minimize:

- **Accidental complexity:** Complexity introduced by implementation choices, not inherent in the problem (over-engineering, unnecessary abstraction layers, unclear variable names)
- **Essential complexity:** Complexity inherent in the problem domain (cannot be eliminated, only managed)

The agent must minimize accidental complexity while keeping essential complexity explicit and documented.

---

### Coding Standards: Why They Matter

Coding standards are not aesthetic preferences — they are engineering requirements. For a coding agent, the following consequences are direct:

1. **Machine readability:** Consistent formatting allows tools (linters, formatters, static analyzers) to work reliably
2. **Context efficiency:** Standard layouts reduce the tokens needed for another agent to understand a file
3. **Diff readability:** Standard formatting prevents spurious diffs from reformatting, making code review focused on logic
4. **Onboarding cost:** New engineers and agents orient faster in a consistent codebase

### PEP 8: Python Style Standard

PEP 8 is the authoritative style guide for Python. Compliance is mandatory, not optional. Key rules:

| Rule | PEP 8 Requirement | Rationale |
|------|------------------|-----------|
| **Indentation** | 4 spaces per level; no tabs | Prevents syntax errors; standard across tools |
| **Line length** | Maximum 79 characters (99 is acceptable for modern projects) | Fits in diff views and agent context windows without horizontal scrolling |
| **Blank lines** | 2 blank lines between top-level definitions; 1 between methods | Visual separation of logical components |
| **Import order** | Standard library → blank line → third-party → blank line → local | Clarifies dependency origin; enforced by `isort` |
| **Function names** | `snake_case` | Python convention; distinguishes functions from classes |
| **Class names** | `PascalCase` | Python convention; distinguishes classes from functions/variables |
| **Constants** | `UPPER_CASE` | Signals immutability intent |
| **Private members** | `_single_underscore` prefix | Signals "internal, do not use from outside" |
| **Dunder methods** | `__double_underscore__` | Reserved for Python protocol methods |

**Import ordering example:**
```python
# Standard library
import os
import sys
from typing import Optional, List

# Third-party (blank line above and below)
import httpx
from pydantic import BaseModel

# Local (blank line above)
from myapp.models import User
from myapp.db import get_connection
```

---

### Code Complexity Metrics

#### Cyclomatic Complexity
Measures the number of linearly independent paths through a function. Each conditional (`if`, `elif`, `while`, `for`, `and`, `or`, `except`) adds 1 to the score.

- **Target:** ≤10 per function
- **Warning:** 10–20 (consider refactoring)
- **Refactor required:** >20 (too complex to test reliably)

A function with cyclomatic complexity of 15 requires 15 independent test cases to achieve full branch coverage. This is a testing and maintenance burden.

```bash
# Measure with radon
radon cc src/ -a -s
```

#### Cognitive Complexity
Measures how difficult code is to understand for a human reader (not just path count). Nesting depth, non-linear flow, and recursion increase cognitive complexity more than cyclomatic complexity does.

**High cognitive complexity signals:**
- Deeply nested conditionals (>3 levels)
- Multiple early returns mixed with regular returns
- Complex boolean expressions without intermediate variables
- Long functions (>30 lines is a smell; >50 is almost always wrong)

---

### Defensive Programming

Defensive programming means writing code that behaves correctly even when its callers violate preconditions or its environment behaves unexpectedly.

**Core defensive practices:**

1. **Validate at boundaries:** Every function that accepts external input (HTTP, database, filesystem, user input) must validate before processing. Do not validate only at the "front door" — validate at every layer that receives untrusted data.

2. **Never trust external input:** Treat all input from outside the process as potentially malicious or malformed. This includes: HTTP request bodies, query parameters, headers, environment variables, database records, files, and messages from queues.

3. **Explicit error handling:** Every exception that can occur in a critical path must be handled explicitly. Use specific exception types, not bare `except:`. Log the exception with context before re-raising or converting.

4. **Fail fast and loud:** When a precondition is violated, raise an exception immediately with a clear message. Do not silently continue with a corrupted state — this leads to data corruption that is harder to diagnose than an immediate crash.

5. **Explicit None checks (pyscg-0034):** Before accessing any attribute or calling any method on a value that may be `None`, check explicitly:

```python
# Defensive: explicit check
def format_user_name(user: Optional[User]) -> str:
    if user is None:
        raise ValueError("Cannot format name: user is None")
    return f"{user.first_name} {user.last_name}"
```

6. **Never silence exceptions:**
```python
# VIOLATION — silent exception swallowing
try:
    result = risky_operation()
except Exception:
    pass  # This hides bugs; never do this

# CORRECT — log and re-raise, or handle specifically
try:
    result = risky_operation()
except SpecificError as e:
    logger.error("risky_operation failed: %s", e, exc_info=True)
    raise  # or raise a domain-specific exception
```

---

### Construction Techniques

#### Top-Down Integration
Build high-level modules first with stubs for lower-level dependencies. Integrates from the top of the call graph downward. Advantage: validates high-level design early. Disadvantage: stubs can obscure lower-level complexity.

#### Bottom-Up Integration
Build and test low-level utility modules first, then integrate them into higher-level modules. Advantage: real components tested together. Disadvantage: system design validation is delayed.

#### Continuous Integration (CI)
The SWEBOK V4 preferred approach. Every commit triggers an automated build and test run. Defects are found in minutes rather than during a dedicated integration phase weeks later. Requirements:
- Commits pushed at least daily
- Automated unit tests pass before merge
- Build must be reproducible from the repository alone

---

### Refactoring

**Definition (Fowler):** The process of changing a software system such that its external behavior is not changed but its internal structure is improved.

**When to refactor:**
- Before adding a new feature to a tangled area ("preparatory refactoring")
- When understanding code requires significant effort ("comprehension refactoring")
- When a code review identifies a structural issue
- When a test is difficult to write because the code is not testable
- Opportunistically: Boy Scout Rule — "Leave the campground cleaner than you found it"

**The Boy Scout Rule:** Every time you modify a function or class, leave it slightly better than you found it: rename an unclear variable, extract a complex conditional, add a missing type hint, add a missing test.

**What refactoring is not:**
- Adding new behavior (that is a feature, not a refactoring)
- Fixing a bug (that changes behavior — do it separately from refactoring)
- Rewriting from scratch (that is not refactoring; it discards accumulated knowledge)

---

### Python Construction Rules (Detailed)

#### Type Hints: Mandatory on All Public Interfaces

Type hints (PEP 484) are required on all public function signatures and complex variables. They serve as executable documentation, enable mypy type checking, and make code comprehensible to other agents without reading the implementation.

```python
from typing import Optional, List
from datetime import datetime

# Fully typed public function signature
def find_users_by_role(
    role: str,
    active_only: bool = True,
    limit: Optional[int] = None,
) -> List[User]:
    ...

# Complex variable type hint
cache: dict[str, datetime] = {}
```

Run `mypy --strict` to enforce. Do not disable mypy checks with `# type: ignore` without a documented reason.

#### Dataclasses for Value Objects

Use `@dataclass(frozen=True)` for immutable value objects. Frozen dataclasses are hashable, thread-safe, and enforce immutability at the interpreter level.

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Money:
    amount: int        # in cents, avoid float for currency
    currency: str      # ISO 4217

    def __post_init__(self) -> None:
        if self.amount < 0:
            raise ValueError(f"Amount cannot be negative: {self.amount}")
        if len(self.currency) != 3:
            raise ValueError(f"Currency must be 3-letter ISO code: {self.currency}")
```

#### Resource Management: Always Use `with`

Every resource that requires cleanup (files, database connections, network sockets, locks, subprocess handles) must use a context manager (`with` statement). This guarantees cleanup even when exceptions occur. See pyscg-0035/0052.

```python
# File I/O
with open("data.json") as f:
    data = json.load(f)

# Database
with db.connection() as conn:
    with conn.cursor() as cursor:
        cursor.execute("SELECT ...")

# Threading lock
with state_lock:
    state["count"] += 1
```

#### Dependency Declaration

All dependencies must be declared explicitly. Do not rely on system-installed packages. Use `pyproject.toml` (modern standard) or `Pipfile` (Pipenv).

```toml
# pyproject.toml
[project]
dependencies = [
    "httpx>=0.27",
    "pydantic>=2.0",
    "sqlalchemy>=2.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0",
    "mypy>=1.10",
    "ruff>=0.4",
    "bandit>=1.7",
    "hypothesis>=6.100",
    "mutmut>=2.4",
]
```

Pin versions in production deployments using `pip install --require-hashes` with a locked requirements file.

---

### Static Analysis: Required Before Commit

All four static analysis tools must pass before a commit is accepted:

| Tool | What It Checks | Command | Must Pass? |
|------|---------------|---------|------------|
| **ruff** | Style, formatting, common bugs (replaces flake8 + isort + pyupgrade) | `ruff check src/` | Yes |
| **mypy** | Type correctness | `mypy --strict src/` | Yes |
| **bandit** | Security vulnerabilities | `bandit -r src/` | Yes (no HIGH) |
| **pip-audit** | Known vulnerable dependencies | `pip-audit` | Yes (no CRITICAL/HIGH) |

Configure these in pre-commit hooks to catch issues before they reach CI:

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.4.0
    hooks:
      - id: ruff
      - id: ruff-format
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.10.0
    hooks:
      - id: mypy
        args: [--strict]
  - repo: https://github.com/PyCQA/bandit
    rev: 1.7.9
    hooks:
      - id: bandit
        args: [-r, src/]
```

---

### Code Review as Construction Quality Gate

Code review is a mandatory construction activity, not a bureaucratic step. Its purpose is defect detection and knowledge transfer.

**What to verify in review:**

| Category | Questions |
|----------|-----------|
| Correctness | Does it handle all edge cases? Are preconditions validated? Are exceptions handled? |
| Complexity | Is cyclomatic complexity ≤10? Can the logic be simplified? |
| Type safety | Are all public functions typed? Does mypy pass? |
| Security | No hardcoded secrets? Queries parameterized? No unsafe deserialization? |
| Testability | Is the code testable? Are dependencies injected? Are tests sufficient? |
| Standards | PEP 8 compliance? Import order? Naming conventions? |
| Documentation | Are docstrings present for public API? Are non-obvious decisions commented? |

---

### V4 Integration: Agile and DevOps

**Agile construction in KA 04:**
- Work in small, vertically sliced increments — a complete feature (requirements + design + code + tests) per sprint
- Continuous integration: integrate and test at least daily
- Technical debt is tracked explicitly (not hidden) and scheduled for remediation
- Refactoring is planned work, not "if time permits"

**DevOps construction in KA 04:**
- Infrastructure as Code: deployment configuration is committed alongside application code
- Automated pipelines enforce static analysis, testing, and security scanning on every commit
- Feature flags allow deployment decoupled from release
- Build reproducibility: the same source commit always produces the same artifact

---

## Agent Guidance

### Do
- Apply the Boy Scout Rule on every file touched: leave it cleaner than you found it
- Add type hints to all public function signatures before considering a function complete
- Use `@dataclass(frozen=True)` for value objects to enforce immutability
- Use `with` statements for all resource management without exception
- Check `None` explicitly before attribute access; use `mypy --strict` to enforce (pyscg-0034)
- Run ruff, mypy, bandit, and pip-audit before every commit
- Keep functions focused: one responsibility, cyclomatic complexity ≤10, ideally ≤30 lines
- Follow PEP 8 import ordering: stdlib → third-party → local
- Declare all dependencies in `pyproject.toml` with minimum version constraints
- Use `snake_case` for functions/variables, `PascalCase` for classes, `UPPER_CASE` for constants
- Raise specific, named exceptions with context messages; never use bare `except: pass`
- Write docstrings for all public functions and classes; document parameters and return types

### Do Not
- Use global mutable state — it causes hidden coupling and breaks test isolation
- Silence exceptions with bare `except: pass` — this hides bugs and makes debugging impossible
- Write functions with cyclomatic complexity >20 — split into smaller, testable units
- Use string formatting to construct SQL queries — always use parameterized queries (pyscg-0010)
- Mix business logic with I/O in the same function — separate pure logic from side effects
- Use `assert` for production validation — it is stripped with `-O` flag (pyscg-0037)
- Hardcode configuration, credentials, or URLs — externalize via environment variables (pyscg-0041)
- Import from `*` — always import specific names for clarity and static analysis
- Write a function longer than 50 lines without a strong justification — length is a complexity signal
- Skip static analysis claiming "it's a quick fix" — static analysis catches class of bugs, not individual bugs

## Checklist
- [ ] All public function signatures have complete type hints
- [ ] `ruff check src/` passes with no errors
- [ ] `mypy --strict src/` passes with no errors
- [ ] `bandit -r src/` passes with no HIGH or CRITICAL findings
- [ ] `pip-audit` passes with no HIGH or CRITICAL vulnerabilities
- [ ] Cyclomatic complexity ≤10 for all new functions (verified with radon)
- [ ] All resources managed with `with` statements
- [ ] All `None` values explicitly checked before use
- [ ] No bare `except: pass` in any new code
- [ ] Import order: stdlib → third-party → local (enforced by ruff)
- [ ] Naming conventions: `snake_case` functions/variables, `PascalCase` classes, `UPPER_CASE` constants
- [ ] No hardcoded secrets, URLs, or environment-specific configuration
- [ ] Dependencies declared in `pyproject.toml` with version constraints
- [ ] Docstrings present for all public functions and classes
- [ ] Boy Scout Rule applied: files touched are cleaner than before the change

## See Also
- wiki/tier1-sources/swebok-v4/overview.md
- wiki/tier1-sources/swebok-v4/ka02-architecture.md
- wiki/tier1-sources/swebok-v4/ka03-design.md
- wiki/tier1-sources/swebok-v4/ka05-testing.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md
- wiki/tier1-sources/swebok-v4/ka13-security.md
- wiki/tier1-sources/python-peps/pep8-style.md
- wiki/tier3-working/python/type-hints.md
- wiki/tier3-working/checklists/code-review.md

## Source

IEEE Computer Society. *Guide to the Software Engineering Body of Knowledge (SWEBOK), Version 4.0*, KA 04: Software Construction. IEEE Press, 2024.

van Rossum, G., Warsaw, B., Coghlan, N. *PEP 8 — Style Guide for Python Code*. https://peps.python.org/pep-0008/

OpenSSF. *Secure Coding Guide for Python (Pyscg)*. https://best.openssf.org/Secure-Coding-Guide-for-Python/

Fowler, M. *Refactoring: Improving the Design of Existing Code, 2nd Edition*. Addison-Wesley, 2018.
