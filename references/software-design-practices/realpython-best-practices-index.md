# Real Python — Best Practices category index

> Captured: 2026-11-09 from https://realpython.com/tutorials/best-practices/
> © Real Python. Structured summary of the category page. Not a reproduction.
> Linked tutorial, quiz, and podcast bodies were not fetched. Do not invent their contents.

Read-only after ingest. Re-synthesize `wiki/tier3-working/software-design-practices/realpython-catalog.md` from this summary.

## What the page is

A category index titled "Python Best Practices for More Pythonic Code". It is a moving list. This capture is the page as fetched on 2026-11-09, not a frozen curriculum. The page also sells a cheat sheet and points at a learning path.

The intro says idiomatic Python is the set of patterns that make code readable, maintainable, and efficient: how functions are structured, how names are chosen, how errors are handled, and how modules are organized.

## Learning path named on the page

"Write More Pythonic Code" — 16 resources. Skills the page lists: Zen of Python, PEP 8, list comprehensions, application layouts, refactoring, duck typing, type checking, type hints, Protocols, code documentation, MkDocs, code quality, Pylint.

Pylint is one tool the path names. The wiki's current lint and format tool is ruff (`wiki/tier3-working/python/overview.md`). Do not switch a repo to Pylint because this path lists it.

## FAQ claims on the page (this is the ingested guidance)

**Pythonic.** Follows Python's philosophy of readability and simplicity. Uses built-in features such as list comprehensions, context managers, and iterators instead of patterns brought over from other languages. Feels natural to an experienced Python developer.

**Readable.** Clear variable names. PEP 8. Short, focused functions. Docstrings. Avoid deep nesting. Whitespace between logical blocks. Explicit code rather than clever tricks.

**Conventions the page calls most important.** PEP 8 for formatting. 4 spaces. Functions named `snake_case`. Lines limited to 79 characters. Docstrings on modules and functions. Imports in the standard order. Type hints where they add clarity.

**Project structure.** Modules split by responsibility. A `src/` layout or a package directory for the code. Tests in a parallel tree. A `README.md`. `requirements.txt` or `pyproject.toml`. A `.gitignore`. Configuration kept separate from code.

**Quality tools the page names.** Linters such as ruff or Pylint. Formatters such as black or `ruff format`. mypy for types. pytest. coverage.py. Pre-commit hooks.

The 79-character limit and 4-space indent match `wiki/tier1-sources/python-peps/pep-008-style.md`. If a repository's formatter config sets another limit, match that repository. Do not adopt a new limit from this index.

## Design-relevant tutorial titles on the page

These titles were visible on the index. Their articles were not ingested. Dates are the index's dates.

| Title | Index date | Already covered in this wiki by |
|-------|------------|----------------------------------|
| Absolute vs Relative Imports in Python | 2026-11-07 | PEP 8 import order; no dedicated import-layout page |
| Python Project Structure: How to Choose the Right Layout | 2026-11-07 | The FAQ above, plus 12-factor and the playbook |
| Documenting Python Code: A Complete Guide | 2026-11-01 | `engineering-playbook/documentation-practices.md`, PEP 8 docstrings |
| Inheritance and Composition: A Python OOP Guide | 2026-10-30 | SOLID, especially LSP and composition in design |
| How to Write Beautiful Python Code With PEP 8 | 2026-10-30 | `pep-008-style.md` |
| Single and Double Underscores in Python Names | 2026-10-26 | `pep-008-style.md` naming table |
| Logging in Python | 2026-10-25 | `python/logging.md` and `python/logging-cookbook.md` |
| Python Code Quality: Best Practices and Tools | 2026-10-25 | `python/overview.md` tool list |
| An Intro to Threading in Python | 2026-10-24 | `python/free-threading.md` is the newer GIL story; this title was not read |
| Python's property(): Add Managed Attributes to Your Classes | 2026-10-23 | `python/descriptors.md` |
| Speed Up Your Python Program With Concurrency | 2026-10-22 | `python/async-patterns.md` for asyncio; this title was not read |
| Supply Chain Security for the Age of AI-Speed Exploits | 2026-10-22 | OWASP A06 and A08 |
| Python Performance Optimization: A Practical Guide | 2026-10-21 | Not ingested. Foord's "correct first, profile second" is the rule this wiki applies |
| How to Review AI-Generated Python Code Efficiently | 2026-09-16 | Not ingested. Review still uses the code-review pages |
| How to Write an AGENTS.md File for a Python Project | 2026-08-24 | This repository's own `AGENTS.md` is the wiki protocol, not that tutorial |

The index also lists quizzes, podcast episodes, and tutorials outside this design ingest (HTTPX, Django project setup, rounding, string formatting, for-loops, installing Python, CSV practice, Pointblank, MCP vs API, Python 3.15 sentinels). Those were not ingested.

## Defects to preserve

- The page is an index. Quoting it as if a linked tutorial had been read is an error.
- Tool names on the page include both ruff and older alternatives (Pylint, black). The wiki default remains ruff plus mypy, pytest, and bandit.
