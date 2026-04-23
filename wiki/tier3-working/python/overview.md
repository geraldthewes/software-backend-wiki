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
| wiki/tier3-working/python/functional-core.md | Functional Core / Imperative Shell pattern; pure functions; itertools, functools, operator |
| wiki/tier3-working/python/async-patterns.md | asyncio: gather, TaskGroup, aiohttp, timeouts, sync/async boundaries, synchronization |
| wiki/tier3-working/python/logging.md | logging module: levels, handlers, formatters, configuration, NullHandler for libraries |
| wiki/tier3-working/python/logging-cookbook.md | Advanced logging recipes: rotation, multi-dest, context, QueueHandler, multiprocessing |
| wiki/tier3-working/python/regex.md | re module: pattern syntax, groups, flags, compilation, ReDoS security |
| wiki/tier3-working/python/enum.md | Enum, IntEnum, StrEnum, Flag, auto(), @unique, comparison rules |
| wiki/tier3-working/python/sorting.md | sorted(), list.sort(), key functions, operator module, stability, multi-pass sorts |
| wiki/tier3-working/python/annotations.md | __annotations__, inspect.get_annotations(), PEP 563, version-safe access |
| wiki/tier3-working/python/descriptors.md | __get__, __set__, __delete__, property, classmethod, staticmethod, __slots__ |
| wiki/tier3-working/python/unicode.md | str vs bytes, encode/decode, normalization (NFC/NFD), file I/O, security |
| wiki/tier3-working/python/argparse.md | ArgumentParser, positional/optional args, subcommands, testing CLI tools |
| wiki/tier3-working/python/urllib.md | urllib.request, POST/GET, error handling; when to use requests/httpx instead |
| wiki/tier3-working/python/sockets.md | TCP sockets, reliable send/recv, message framing, select, IPC |
| wiki/tier3-working/python/ipaddress.md | IPv4/IPv6 addresses and networks, CIDR containment, SSRF prevention |
| wiki/tier3-working/python/mro.md | C3 linearization, MRO, super(), cooperative multiple inheritance |
| wiki/tier3-working/python/curses.md | Terminal UI: windows, color, keyboard input, wrapper(), noutrefresh/doupdate |
| wiki/tier3-working/python/free-threading.md | Free-threaded CPython 3.13+, GIL removal, thread safety, migration guide |
| wiki/tier3-working/python/isolating-extensions.md | C extension modules: per-module state, heap types, multi-phase init, subinterpreters |

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
- wiki/tier3-working/python/logging.md
- wiki/tier3-working/python/async-patterns.md

## Source

Python Enhancement Proposals: PEP 8, PEP 20, PEP 484. Gary Bernhardt, "Boundaries" (Strange Loop, 2012). SWEBOK V4, KA4 Software Construction.
