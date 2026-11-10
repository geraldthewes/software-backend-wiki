# IST Python Guide (Tier 3)

> **Tier 3** | Source: `references/software-design-practices/ist-python-guide.md` | Authority: working | Derives From: PEP 8, PEP 20, PEP 257

## Summary

Rui Maranhão's gist is a teaching guide for first-year students at Instituto Superior Técnico, inspired by Steve Loria's gist. No license is stated. The revision captured is `f410e2be18b199f94895a3d1f4d455b2a50d8912`.

Where the guide restates PEP 8, PEP 20, or PEP 257, follow the PEP. Several of its local rules are stricter or looser than PEP 8, and those lose. The testing section agrees in spirit with the Python testing pages and does not set the coverage number.

## Key Concepts

### Rules that match the PEPs

- Explicit, readable code. Fix a bad design when it is found. Test, and document new behavior.
- Names: `snake_case` for functions, variables, methods, modules, and packages; `CapWords` for classes and exceptions; `UPPER_CASE` for constants.
- Avoid `l`, `O`, and `I` as names. A short loop variable whose meaning is obvious is fine. The guide's example is `for e in elements`. PEP 8 also allows `i`, `j`, `k` and `e` in an `except`.
- Do not repeat the module name inside the class (`audio.AudioCore` after `import audio`).
- Compare to `None` with `is`. Prefer truthiness over `== True` and `== False`.
- Open files with `with`.
- Imports in three blocks: standard library, third party, local.
- Docstrings follow PEP 257. One line, imperative mood, for an obvious function. Document `__init__` on the class docstring.
- A small named function beats a comment that restates a condition.

### Rules that do not become wiki policy

| Guide | Decision |
|-------|----------|
| Indent width is a personal choice if it is consistent | PEP 8 requires 4 spaces |
| 80–100 columns is fine | PEP 8's 79 columns, or the repository formatter's limit |
| Never import a symbol, only a module, to avoid cycles | PEP 8 allows `from module import name`. Wildcards stay forbidden. Cycles are a design problem, not a reason to ban symbol imports |
| `__method` means private | PEP 8: `_name` is non-public, `__name` is name mangling |
| `filter(lambda ...)` is equal to a comprehension | Prefer the comprehension |
| Reverse notation (`elements_active` rather than `active_elements`) | Author preference. Match the codebase |
| Strive for 100% coverage and do not obsess | KA5 sets the targets. The "do not obsess" half is already wiki policy |
| Human-driven development outranks TDD | The phrase is undefined in the gist. Do not treat it as a process |

### Testing notes worth keeping

Long test names. Isolation from real networks and databases. Factories rather than fixtures that only build data. An unfinished test fails loudly instead of staying green. Functional tests for a GUI or a web flow may be written as a scenario. Those points already live on `python-testing/overview.md` and the Flowers page. The gist's sample uses `unittest` and factory_boy. New suites in this wiki use pytest unless the project already standardizes on `unittest`.

The guide says the guidelines will change. This page is the capture. Re-ingest if the gist changes. Do not treat the gist as a living standard.

## Agent Guidance

### Do

- Follow the PEP rows in the first table.
- Drop a redundant type prefix that repeats the module name.
- Prefer a named predicate over a comment that explains a boolean expression.
- Fail an unfinished test instead of leaving a passing placeholder.

### Do Not

- Do not change indentation, line length, import style, or privacy spelling to match this gist when PEP 8 or the repository already decides them.
- Do not rename `active_elements` to `elements_active` as a drive-by.
- Do not add factory_boy because the gist links it. The standard library and pytest are enough until a project already uses a factory library.
- Do not paste the gist into a repository. No license is stated.

## Checklist

- [ ] Names follow PEP 8, not the gist's reverse notation or its `__` privacy rule
- [ ] Imports follow PEP 8, including legal symbol imports and a ban on wildcards
- [ ] `None` is compared with `is`; truthiness is used for ordinary flags
- [ ] Files are opened with `with`
- [ ] A new docstring matches PEP 257 and the project's field style
- [ ] No rule from the "does not become wiki policy" table was applied over a PEP

## See Also

- wiki/tier3-working/software-design-practices/overview.md
- wiki/tier3-working/software-design-practices/hitchhikers-style.md
- wiki/tier1-sources/python-peps/pep-008-style.md
- wiki/tier1-sources/python-peps/pep-020-zen.md
- wiki/tier3-working/python/idioms.md
- wiki/tier3-working/python-testing/overview.md
- wiki/tier3-working/python-testing/thea-style-guide.md

## Source

Rui Maranhão, "A Guide of Best Practices for Python", GitHub gist `ruimaranhao/4e18cbe3dad6f68040c32ed6709090a3`, revision `f410e2be18b199f94895a3d1f4d455b2a50d8912`, https://gist.github.com/ruimaranhao/4e18cbe3dad6f68040c32ed6709090a3. Inspired by Steve Loria, https://gist.github.com/sloria/7001839. No license stated. Summary in `references/software-design-practices/ist-python-guide.md`.
