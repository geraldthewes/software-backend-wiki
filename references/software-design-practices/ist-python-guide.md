# IST Python best-practices guide (Rui Maranhão gist)

> Captured: 2026-11-09 from https://gist.github.com/ruimaranhao/4e18cbe3dad6f68040c32ed6709090a3
> File name on the gist: python-guide-ist.md
> Author: Rui Maranhão. Teaching guide for first-year students at Instituto Superior Técnico, University of Lisbon.
> Inspired by Steve Loria, https://gist.github.com/sloria/7001839
> Revision: f410e2be18b199f94895a3d1f4d455b2a50d8912. Gist "last active" 2026-07-16.
> No license is stated. Structured summary. Not a reproduction. Do not paste the gist back into the wiki.

Read-only after ingest. Re-synthesize `wiki/tier3-working/software-design-practices/ist-python-guide.md` from this summary.

## General stance

The guide opens from PEP 20: beautiful, explicit, readable. It adds three process rules: anybody can fix anything (attributed to Khan Academy development docs); fix a bad design, a wrong decision, or poor code as soon as it is found; test ruthlessly and write docs for new features. It ranks "human-driven development" above TDD as a slogan, without defining the phrase. It says the guidelines themselves will change.

## Style

Follow PEP 8 when sensible.

### Naming

- Functions, variables, methods, packages, modules: `lower_case_with_underscores`
- Classes and exceptions: `CapWords`
- Protected and internal: `_single_leading_underscore`
- Private methods: `__double_leading_underscore`
- Constants: `ALL_CAPS_WITH_UNDERSCORES`

Avoid one-letter names, especially `l`, `O`, and `I`, except in a very short block whose meaning is obvious. The guide's own example is `for e in elements`.

Avoid redundant prefixes that repeat the module name (`audio.AudioCore` after `import audio`). Prefer `audio.Core`.

"Reverse notation": group related names by the noun first (`elements`, `elements_active`, `elements_defunct`) rather than by the adjective (`active_elements`). This is the author's preference. PEP 8 does not require it. Do not impose it on a codebase that names the other way.

### Indentation

The guide says indentation width is the author's choice as long as it is consistent, and that a space is a fixed column while a tab is not. That loses to PEP 8, which requires 4 spaces. Do not cite this guide for indent width.

### Equality

Do not compare to `True`, `False`, or `None` with `==`. The guide's preferred forms are truthiness (`if attr` / `if not attr`) and identity (`if attr is True`, `if attr is None`). PEP 8 requires `is` / `is not` for `None`. `is True` is only for a real tri-state; ordinary checks use truthiness.

### Comprehensions

Prefer a comprehension (or `filter`) over a loop that only appends. The guide presents `filter(lambda ...)` as an equal alternative. This wiki prefers the comprehension; a lambda passed to `filter` is harder to read. See `wiki/tier3-working/python/idioms.md`.

### Files

Open files with `with`. The `with` block is what closes the file.

### Imports

Prefer importing a module (`import canteen`, `import canteen.sessions`, `from canteen import sessions`) over importing a function out of it. The stated reason is circular imports. Exception: third-party docs that tell you to import a symbol. Wildcard imports are in the guide's "no" example.

PEP 8 allows `from module import name` and forbids wildcards. The circular-import caution is real. The ban on symbol imports is stricter than PEP 8 and is not wiki policy. Follow PEP 8 and the project.

Order imports in three blocks separated by a blank line: standard library, third party, local. That matches PEP 8.

### Documentation

Follow PEP 257. The guide mentions reStructuredText and Sphinx as a way to enforce docstrings. One-line docstrings for obvious functions, written as an action ("Return the pathname") rather than a description ("Returns"). Multi-line docstrings: summary, usage example when it helps, arguments, and return type unless the return is `None`. Document `__init__` on the class docstring, not on the method. The sample uses Sphinx `:param:` and `:rtype:` fields. Google-style or NumPy-style docstrings are also valid; match the project. PEP 257 wins on the requirement to have a docstring. The field syntax does not.

### Comments

Use them sparingly. A small named function beats a comment that restates a condition. When a comment is needed, PEP 8's "Strunk and White apply" is the guide's citation: brief, complete sentences where the comment is prose.

### Line length

The guide says not to stress, and that 80–100 characters is fine, with implicit string concatenation inside parentheses for long strings. PEP 8's limit in this wiki is 79 characters (`pep-008-style.md`). A repository formatter config may set another limit. The gist's "don't stress" does not override either one.

## Testing section (advanced material in the gist)

Strive for high coverage and do not obsess over the score. That mood matches the testing overview; the number still comes from SWEBOK KA5, not from this gist.

- Long descriptive test names, so the test often needs no docstring.
- Tests stay isolated from real databases and the network. Use a throwaway database or a mock.
- Prefer factories to fixtures. The guide links factory_boy. The Flowers page already makes the narrower rule: a fixture that only builds data is a function.
- An unfinished test must fail (`assert False` with a TODO). Do not leave it green.

Unit tests: one small behavior, fast, and a slow test is better than none. One `TestCase` per class is often enough. The sample uses `unittest` and factory_boy.

Functional tests: scenario names that read as a user story, with the story written as comments before the code. That is a web/GUI practice. It does not replace the test pyramid.

## Conflicts to keep visible

| Guide says | Wiki rule |
|------------|-----------|
| Indent however you like | PEP 8: 4 spaces |
| `__` means private | PEP 8: `_` is non-public; `__` is name mangling |
| Never import a symbol | PEP 8 allows `from module import name`; wildcards stay forbidden |
| 80–100 characters is fine | PEP 8: 79, unless the repo formatter says otherwise |
| `filter(lambda)` equals a comprehension | Prefer the comprehension |
| Reverse notation | Local preference only |
| Coverage: strive for 100%, don't obsess | KA5 sets the unit-level targets |
