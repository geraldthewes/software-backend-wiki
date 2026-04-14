# Python Best Practices — Overview

> **Tier 3** | Source: Python Enhancement Proposals, Gary Bernhardt | Enforces/Derives From: wiki/tier1-sources/python-peps/pep-008-style.md, wiki/tier1-sources/python-peps/pep-020-zen.md, wiki/tier1-sources/swebok-v4/ka04-construction.md

## Summary

Python's flexibility is both its strength and its risk. Without discipline, codebases accumulate anti-patterns that compound over time. This overview grounds Python development in PEP 20's principle that "there should be one obvious way to do it," and links to sub-pages covering idioms, the type system, functional architecture, and async patterns.

## Why Python Idioms Matter

PEP 20 (The Zen of Python) states: "There should be one — and preferably only one — obvious way to do it." Idiomatic Python is not stylistic preference; it is the standard that makes code readable by other developers, linters, and AI agents.

Unidiomatic Python has measurable costs:
- Mutable default arguments cause silent bugs that are hard to trace.
- Bare `except:` swallows errors and makes failures invisible.
- Missing type hints remove the compiler-equivalent safety net from the codebase.
- Direct I/O inside business logic makes unit testing impossible without mocks.

## Quick-Start Rules

These four rules apply to every Python file an agent writes or reviews:

1. **Type hints on all public functions.** Use builtin generics (`list[int]`, `dict[str, int]`) for Python 3.9+ and `str | None` unions for Python 3.10+. Run `mypy --strict` in CI.
2. **Context managers for all resources.** Files, database connections, locks, and network sockets must always be opened inside `with` blocks. Never rely on garbage collection for cleanup.
3. **Functional core, imperative shell.** Business logic lives in pure functions with no I/O. Only the outermost shell reads from databases or calls external services.
4. **Specific exceptions, never bare `except`.** Catch `ValueError`, `KeyError`, `sqlite3.OperationalError` — not `Exception` and never bare `except`. Always log or re-raise.

## Sub-Pages

| Page | What It Covers |
|------|----------------|
| wiki/tier3-working/python/idioms.md | Pythonic patterns: comprehensions, dataclasses, unpacking, generators, anti-patterns |
| wiki/tier3-working/python/type-system.md | `typing` module guide: Protocols, TypeVar, Literal, TypedDict, mypy config |
| wiki/tier3-working/python/functional-core.md | Functional Core / Imperative Shell pattern; pure functions; testable business logic |
| wiki/tier3-working/python/async-patterns.md | asyncio: gather, TaskGroup, aiohttp, timeouts, sync/async boundaries |

## Key Tools

- **ruff** — fast linter and formatter; replaces flake8 + isort + pyupgrade
- **mypy** — static type checker; run with `--strict` in CI
- **bandit** — security vulnerability scanner
- **pytest** — test runner with rich plugin ecosystem
- **hypothesis** — property-based testing

## See Also

- wiki/tier1-sources/python-peps/pep-008-style.md
- wiki/tier1-sources/python-peps/pep-020-zen.md
- wiki/tier1-sources/python-peps/pep-484-type-hints.md
- wiki/tier1-sources/swebok-v4/ka04-construction.md
- wiki/tier3-working/python/idioms.md
- wiki/tier3-working/python/type-system.md
- wiki/tier3-working/python/functional-core.md

## Source

Python Enhancement Proposals: PEP 8, PEP 20, PEP 484. Gary Bernhardt, "Boundaries" (Strange Loop, 2012). SWEBOK V4, KA4 Software Construction.
