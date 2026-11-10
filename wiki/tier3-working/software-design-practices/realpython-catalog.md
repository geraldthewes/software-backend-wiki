# Real Python Best-Practices Catalog (Tier 3)

> **Tier 3** | Source: `references/software-design-practices/realpython-best-practices-index.md` | Authority: working | Derives From: PEP 8, PEP 20, `wiki/tier3-working/python/overview.md`

## Summary

https://realpython.com/tutorials/best-practices/ is a category index, captured 2026-11-09. The page defines what it means by Pythonic code and lists tutorials, quizzes, and podcasts. The tutorial bodies were not fetched. This page records the index's own claims and points each design-relevant title at the wiki page that already covers that topic. © Real Python. Summary only.

## Key Concepts

### Claims on the index itself

| Topic | What the index says | Wiki authority |
|-------|---------------------|----------------|
| Pythonic | Readable and simple. Built-ins, comprehensions, context managers, and iterators rather than patterns brought from another language | PEP 20 and `python/idioms.md` |
| Readable | Clear names, PEP 8, short functions, docstrings, shallow nesting, explicit rather than clever | PEP 8, this overview's shape rules |
| Conventions | 4 spaces, `snake_case` functions, 79-character lines, docstrings, standard import order, type hints where they clarify | `pep-008-style.md`, `pep-484-type-hints.md` |
| Layout | Modules by responsibility, `src/` or a package, tests in a parallel tree, README, `requirements.txt` or `pyproject.toml`, `.gitignore`, config apart from code | 12-factor III for config. Match the repository's existing layout |
| Tools | ruff or Pylint; black or `ruff format`; mypy; pytest; coverage.py; pre-commit | ruff, mypy, pytest, bandit. Do not add Pylint or black beside ruff |

The learning path "Write More Pythonic Code" lists these skills: Zen of Python, PEP 8, list comprehensions, application layouts, refactoring, duck typing, type checking, type hints, Protocols, code documentation, MkDocs, code quality, Pylint. Pylint on that list does not change the toolchain.

### Titles on the index, and where to read instead

These articles were not ingested. Use the wiki page in the third column. If that column says "not ingested", do not invent the article.

| Index title | Index date | Read instead |
|-------------|------------|--------------|
| Absolute vs Relative Imports in Python | 2026-11-07 | PEP 8 import rules. No separate import-layout page was added |
| Python Project Structure | 2026-11-07 | The layout row above |
| Documenting Python Code | 2026-11-01 | `engineering-playbook/documentation-practices.md` and PEP 257 via the IST page |
| Inheritance and Composition | 2026-10-30 | SOLID, especially LSP. Composition is the default; inheritance must preserve behavior |
| How to Write Beautiful Python Code With PEP 8 | 2026-10-30 | `pep-008-style.md` |
| Single and Double Underscores | 2026-10-26 | `pep-008-style.md` naming table |
| Logging in Python | 2026-10-25 | `python/logging.md` |
| Python Code Quality | 2026-10-25 | `python/overview.md` |
| Python's property() | 2026-10-23 | `python/descriptors.md` |
| Supply Chain Security for the Age of AI-Speed Exploits | 2026-10-22 | OWASP A06 and A08 |
| Threading, concurrency, performance titles | 2026-10 | `python/async-patterns.md`, `python/free-threading.md`. Profile before optimizing (Foord) |
| How to Review AI-Generated Python Code | 2026-09-16 | The code-review pages. The tutorial was not ingested |
| How to Write an AGENTS.md File | 2026-08-24 | This repository's `AGENTS.md`. The tutorial was not ingested |

Quizzes, podcast episodes, and off-topic tutorials on the same index (HTTP clients, Django setup, rounding, string formatting, installing Python) are outside this ingest.

## Agent Guidance

### Do

- Use the index's definitions of Pythonic and readable as a restatement of PEP 8 and PEP 20.
- Keep config out of code, and keep tests in a parallel tree, when starting a layout from nothing.
- Follow a title in the table to the wiki page that already covers it.

### Do Not

- Do not cite a linked Real Python tutorial for a rule. Its body is not in this wiki.
- Do not switch a repository to Pylint or black because the learning path names them.
- Do not treat the index as stable. It is a moving catalog. This page is the 2026-11-09 capture.

## Checklist

- [ ] A claim attributed to Real Python best practices comes from the table of index claims, or from an ingested page, not from an unread tutorial
- [ ] Layout and toolchain match the repository, with ruff as the default for new Python
- [ ] Type hints are present where they clarify the public signature, as the index and PEP 484 both ask

## See Also

- wiki/tier3-working/software-design-practices/overview.md
- wiki/tier3-working/python/overview.md
- wiki/tier3-working/python/idioms.md
- wiki/tier1-sources/python-peps/pep-008-style.md
- wiki/tier1-sources/python-peps/pep-020-zen.md
- wiki/tier1-sources/python-peps/pep-484-type-hints.md
- wiki/tier2-core/twelve-factor-app/factors.md
- wiki/tier3-working/python-testing/realpython-code-testing.md

## Source

Real Python, "Python Best Practices" category index, captured 2026-11-09, https://realpython.com/tutorials/best-practices/. © Real Python. Summary of the index only, in `references/software-design-practices/realpython-best-practices-index.md`.
