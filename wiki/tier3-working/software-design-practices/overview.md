# Software Design Practices (Tier 3)

> **Tier 3** | Source: eight practitioner and survey sources in `references/software-design-practices/` | Authority: working | Derives From: SWEBOK KA3, SWEBOK KA4, PEP 8, PEP 20, SOLID, `wiki/tier2-core/design-patterns/`

## Summary

This page is the working guide for designing Python modules and small services. It reconciles eight sources captured on 2026-11-09. Refactoring.Guru was listed twice and is one source. The sources agree on a short list: state the problem before coding, keep each unit small and explicit, separate I/O from logic, fail with a useful error, and add abstraction only after the shape of the problem has been seen more than once.

They disagree on classes versus functions, on coverage numbers, on line length, and on how private a name is. SWEBOK, the PEPs, SOLID, the architecture-patterns pages, and the testing pages win those disagreements. Use this page while designing. Open a source page only for that author's specific rule.

The Manning book and the Refactoring.Guru example pages were not read. Do not attribute chapter rules or pattern code to them.

## Key Concepts

### Source map

| Page | Source | Read it for |
|------|--------|-------------|
| `wiki/tier3-working/software-design-practices/nsls-ii-principles.md` | NSLS-II Scientific Python Cookiecutter, BSD 3-Clause | I/O split, wide versus deep, strict core under a friendly layer, keyword-only options, stable returns |
| `wiki/tier3-working/software-design-practices/foord-30-practices.md` | Michael Foord, Opensource.com, 2017, CC BY-SA 4.0 | YAGNI, rule of three, fail fast, injection, generators, when design is discussed |
| `wiki/tier3-working/software-design-practices/hitchhikers-style.md` | Hitchhiker's Guide, Code Style, CC BY-NC-SA 3.0 | Explicit signatures, `*args` and `**kwargs` used sparingly, one normal return, no magic |
| `wiki/tier3-working/software-design-practices/ist-python-guide.md` | Rui Maranhão gist, no stated license | Teaching-guide style. Several rules lose to PEP 8 |
| `wiki/tier3-working/software-design-practices/wikipedia-coding-practices.md` | Wikipedia, CC BY-SA 4.0, weakly cited | Order of work: life cycle, requirements, architecture, design, then code. Quality goals must be named |
| `wiki/tier3-working/software-design-practices/realpython-catalog.md` | Real Python best-practices index, 2026-11-09 | What "Pythonic" and a project layout mean on that index. Tutorial bodies were not ingested |
| `wiki/tier3-working/software-design-practices/manning-mak.md` | Manning product page, January 2026 | Bibliographic pointer. Book text was not ingested |
| `wiki/tier3-working/software-design-practices/refactoring-guru-python.md` | Refactoring.Guru Python catalog | Map of 22 GoF names onto the Tier 2 pattern pages. Examples were not ingested |

### Authority resolution

| Conflict | Winner | Rule an agent applies |
|----------|--------|------------------------|
| Foord: objects beat complex data structures, and functions beat new types. NSLS-II: stop writing classes; many functions on one standard structure. Manning's page is class-centric | Split by whether an invariant exists | Default to a function over a standard structure (dict, tuple, dataclass used as a record, array). Write a class when it protects an invariant, or when it is a collaborator with a life cycle (repository, client, unit of work). Domain aggregates stay on `architecture-patterns/aggregates.md`. Scientific arrays stay as arrays |
| IST guide: indent as you like; 80–100 columns; never import a symbol; `__` means private; reverse noun notation | PEP 8 | 4 spaces. 79 columns unless the repository formatter sets another limit. `from module import name` is allowed; wildcards are not. `_` is non-public. `__` is name mangling. Name things to match the file |
| Hitchhiker's Guide: throwaway target is `__`. Many codebases use `_` | Neither is Tier 1 | Match the file. Do not introduce `__` throwaways into a file that uses `_` |
| Hitchhiker's Guide names black, yapf, autopep8, and pycodestyle. Real Python's index also names Pylint and black. IST mentions Sphinx as the docstring tool | `python/overview.md` | Lint and format with ruff. Check types with mypy. Docstring field style matches the project |
| Foord: start at 100% coverage and reject any drop | SWEBOK KA5 and `python-testing/overview.md` | Unit-level statement coverage above 90% and branch coverage above 80% on code you own. Do not add a test only to move the percentage. Do not lower KA5 because a source names a different number |
| Foord calls onion architecture the shape of over-engineering | `architecture-patterns/ports-and-adapters.md` | Ports and adapters are for a domain service that has earned them. They are not the default for a script or a numeric library |
| Foord: owning code can be better than a dependency | OWASP A06 | Do not reimplement cryptography, HTTP, or the standard library. A new dependency needs a maintainer, a license, and a reason. A one-off helper you will not maintain can stay local |
| Wikipedia: Pascal case is often the better variable name. Comment what each variable is for | PEP 8 | Python variables are `snake_case`. The name carries the meaning. Comment why, and only when the code cannot show it |
| Real Python index and the Hitchhiker's Guide restate the Zen and PEP 8 | The PEP pages | Do not maintain a second copy of PEP 8 or PEP 20 from these guides |
| Refactoring.Guru catalog shows class-shaped Singleton, Strategy, Iterator, Visitor | `design-patterns/overview.md` | Try a module-level instance or an injected object, a callable, a generator, a decorator, or `singledispatch` before the class hierarchy |

### Order of work

Wikipedia's prerequisite list matches the SWEBOK order. Follow it, and follow the KA page for the detail:

1. The life cycle and the quality attributes that matter for this change (KA10, KA12). Weinberg's observation, as the article reports it: people will hit the goal you state and drop the ones you leave implicit. Name the goal.
2. The problem the system solves, including the non-functional requirements (KA1).
3. Which component does what (KA2). Hoare's test, as the article reports it: prefer a design so simple that missing deficiencies would be obvious.
4. The algorithms, types, and interfaces inside that component (KA3, this page).
5. Conventions before the first file, not after (PEP 8, and the repository's formatter).

For a small change, steps 3 and 4 can be one decision. Do not skip step 2.

### Shape of a Python unit

- YAGNI for behavior. Do not implement a use that has no caller. An external API may be shaped so a later use can be added without a break. That is part of the signature, not dead code. Commented-out code does not ship.
- Simple case first, with as few options as possible. Complex cases become extra arguments or extra functions when they are real.
- A friendly function that guesses may sit on a strict function that accepts only what it needs. Test the strict function. Library code raises. Application code, such as a GUI or a request handler, catches and decides. The exception names the bad value, what is wrong, and what would fix it.
- Keep I/O in functions that only do I/O. Logic takes and returns ordinary values. This is `python/functional-core.md`.
- Arguments that would be misread by position are keyword-only (`*` in the signature). Do not hide them in an `options` dict. A mutable default is already forbidden.
- The return type does not depend on an optional flag. If it must, split the function.
- Early returns are for the error cases. The success value has one exit. Raising beats returning `None` to mean "the arguments were invalid".
- Extract a helper the third time the same code appears, and test the helper. Two copies can stay. A helper that exists only to support one test does not need its own tests until something else calls it.
- A function past about 30 lines, or a module past about 500, is a prompt to look for a split. Those numbers are Foord's heuristics. They are not a failing gate. Test modules may be longer.
- Do not do work in `__init__` beyond storing the already-built dependencies. Do not put behavior in `__init__.py` beyond the imports that define the package's public names.
- Dependencies arrive as parameters. A signature with a long list of collaborators is a sign the function does too much. See `architecture-patterns/dependency-injection-bootstrap.md`.
- No literal hosts, ports, user names, or absolute paths in code. Config is 12-factor III. Local file locations follow `xdg-base-directory/overview.md`.
- `*args` and `**kwargs` are for a genuinely open set of values. A known sequence is one parameter. A known option is a named parameter.
- No metaclasses, import hooks, or `**locals()` tricks unless a specific requirement cannot be met another way.
- Mark non-public names with a single leading underscore. Callers who use them own the breakage.
- Discuss the design before the code is finished. Review is a late and expensive time to discover that the structure is wrong. It is still the right time to reject a wrong structure.

### Patterns

Use `wiki/tier2-core/design-patterns/overview.md`. A pattern needs a problem: more than one variant, a real extension point, or a name the team already shares. A callable, a generator, a decorator, or a `Protocol` comes before a GoF class tree. The Refactoring.Guru catalog does not add patterns and does not supply code.

### What this page does not decide

Testing mechanics, coverage numbers, and doubles: `python-testing/overview.md`. Security: OWASP and the security-review checklist. Service architecture: KA2, ports and adapters, and 12-factor. The text of Mak's book and of the Real Python tutorials linked from the index.

## Agent Guidance

### Do

- State the problem, the quality attribute that matters, and the component boundary before writing the module.
- Prefer a function on a standard data structure until an invariant or a life cycle requires a type.
- Separate I/O from the decision the function is actually making.
- Make the simple call obvious. Use keyword-only arguments for options. Keep the return type stable.
- Fail in library code with an exception that identifies the bad value.
- Leave room in a published signature for a later parameter. Do not implement the later behavior.
- Choose a pattern from the Tier 2 pages after a simpler Python form has been rejected.
- Refactor when tests and version control can catch a mistake, and include that work in the change that needs it.

### Do Not

- Do not invent an `Image`-style wrapper around one array and some metadata when the array plus a side value would do. Do not dissolve an aggregate that protects a business invariant into loose functions.
- Do not adopt the IST guide's indent, line length, import ban, privacy spelling, or reverse notation over PEP 8.
- Do not set a 100% coverage gate from Foord, and do not lower the KA5 targets.
- Do not add black, yapf, autopep8, or Pylint because a guide names them. Use ruff.
- Do not copy Refactoring.Guru examples, and do not describe chapters of the Manning book.
- Do not treat the Real Python category index as if its linked tutorials had been read.
- Do not hide parameters in an `options` dict or a mutable default.
- Do not catch an exception in library code only to print it.

## Checklist

- [ ] The problem and the quality attribute that matters are stated before the structure
- [ ] I/O functions only do I/O; logic functions take and return ordinary values
- [ ] A new class protects an invariant or owns a life cycle; otherwise the unit is a function
- [ ] The simple call needs no configuration; options are keyword-only; the return type is stable
- [ ] Library failures raise and name the bad value; application code is what catches them
- [ ] No behavior was added for a caller that does not exist; published signatures can still grow by a keyword argument
- [ ] A helper was extracted because the code existed three times, or an existing abstraction was reused on purpose
- [ ] Constructors only store dependencies; `__init__.py` only publishes names
- [ ] Hosts, ports, users, and absolute paths are configuration, not literals
- [ ] A pattern, if used, is named, justified, and not a class copy of a callable, generator, or decorator
- [ ] Style conflicts in the table above were resolved in favor of PEP 8, ruff, and the KA pages

## See Also

- wiki/tier1-sources/swebok-v4/ka03-design.md
- wiki/tier1-sources/swebok-v4/ka04-construction.md
- wiki/tier1-sources/python-peps/pep-008-style.md
- wiki/tier1-sources/python-peps/pep-020-zen.md
- wiki/tier2-core/solid-principles/overview.md
- wiki/tier2-core/design-patterns/overview.md
- wiki/tier2-core/architecture-patterns/domain-model.md
- wiki/tier3-working/python/functional-core.md
- wiki/tier3-working/python/idioms.md
- wiki/tier3-working/checklists/design-review.md
- wiki/tier3-working/software-design-practices/nsls-ii-principles.md
- wiki/tier3-working/software-design-practices/foord-30-practices.md

## Source

Structured summaries in `references/software-design-practices/` of: Real Python best-practices category index, captured 2026-11-09, https://realpython.com/tutorials/best-practices/ ; Rui Maranhão, gist `4e18cbe3dad6f68040c32ed6709090a3`, revision `f410e2be`, https://gist.github.com/ruimaranhao/4e18cbe3dad6f68040c32ed6709090a3 ; NSLS-II, "Guiding Design Principles", Scientific Python Cookiecutter, BSD 3-Clause, Copyright (c) 2019 Brookhaven National Laboratory, https://nsls-ii.github.io/scientific-python-cookiecutter/guiding-design-principles.html ; Kenneth Reitz and Real Python, "Code Style", The Hitchhiker's Guide to Python, CC BY-NC-SA 3.0, https://docs.python-guide.org/writing/style/ ; Ronald Mak, *Software Design for Python Programmers*, Manning, January 2026, ISBN 9781633439498, product page only, https://www.manning.com/books/software-design-for-python-programmers ; Alexander Shvets, Refactoring.Guru, "Design Patterns in Python", catalog only, https://refactoring.guru/design-patterns/python ; Michael Foord, "30 best practices for software development and testing", Opensource.com, CC BY-SA 4.0, https://opensource.com/article/17/5/30-best-practices-software-development-and-testing ; Wikipedia, "Coding best practices", CC BY-SA 4.0, https://en.wikipedia.org/wiki/Coding_best_practices.
