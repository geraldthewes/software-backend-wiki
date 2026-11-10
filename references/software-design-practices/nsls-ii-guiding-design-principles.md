# NSLS-II Scientific Python Cookiecutter — Guiding Design Principles

> Captured: 2026-11-09 from https://nsls-ii.github.io/scientific-python-cookiecutter/guiding-design-principles.html
> Scientific Python Cookiecutter 0.1 documentation. NSLS-II, Brookhaven National Laboratory.
> Repository: https://github.com/NSLS-II/scientific-python-cookiecutter
> License of the repository: BSD 3-Clause, Copyright (c) 2019, Brookhaven National Laboratory.
> Structured summary of the design-principles page. Not a reproduction.
> Other chapters of the tutorial (packaging, Travis CI, doctr, conda) were not ingested.
> Travis CI, named elsewhere in that tutorial, is historical and is not wiki guidance.

Read-only after ingest. Re-synthesize `wiki/tier3-working/software-design-practices/nsls-ii-principles.md` from this summary.

The page is about designing and organizing scientific Python code. Several principles apply to any Python library. The "prefer arrays and functions over custom classes" principle is scoped to scientific data. It does not cancel a domain aggregate.

## Principles

### Collaborate

Software written by several people is preferable to software written by one. Shared conventions make it easier to contribute and easier to reuse other scientific projects. Talking through a design and its assumptions clarifies the design. Being wrong in that conversation is part of improving it. More than one person should understand each part of the code, so the project is not stuck on a single author. Contributors from different scientific fields make it easier to see which behavior should be generalized.

### Don't be afraid to refactor

The first version is not the right structure. Refactor once the problem and the trade-offs are better understood. Version control, tests, and linting are the safety net.

### Prefer wide over deep

A piece should be reusable in a way the author did not plan. Branching from the first use case should add unplanned behavior without a large jump in complexity. When building something new, go down to the lowest level, understand it, then build back up. Ask what another group, a related experiment, or next year would need from the same capability. A slower robust shape beats a fast brittle one.

### Keep I/O separate

Mixing file paths, names, formats, or layouts into the scientific logic blocks reuse. An I/O function only does I/O: it accepts a path and returns an array, or a dictionary of arrays and metadata. The scientific function accepts and returns ordinary data types. That function can be tested, can survive a format change, and can be reused.

This is the same split as functional core / imperative shell.

### Duck typing

Treat an object by what it can do, not by its class. The page's example is `numpy.sum` accepting a pandas DataFrame because the DataFrame provides the methods NumPy needs, even though pandas did not exist when `numpy.sum` was written. Avoid `isinstance` checks. Accept the widest set of inputs that provide the required behavior.

Where a type checker is required, a `Protocol` is the typed form of this rule. See `wiki/tier3-working/python/type-system.md`. `isinstance` remains appropriate when the contract really is one concrete type (for example a security boundary that must reject a lookalike).

### Stop writing classes

Not every design needs objects. The page quotes Alan J. Perlis (ACM SIGPLAN, September 1982, "Epigrams in Programming"): it is better to have 100 functions operate on one data structure than 10 functions on 10 data structures.

Custom types such as `Image` or `DiffractionAnalysis` have repeatedly been hard to extend. Prefer standard structures (dicts, NumPy arrays) and plain functions. The page cites Jack Diederich's "Stop Writing Classes" talk, and scikit-image's decision to drop an `Image` class and use arrays. Other libraries understand an array. They do not understand a private class. Pass application metadata beside the array.

This principle governs scientific and numeric code. A domain model that protects an invariant is a different case and stays on `wiki/tier2-core/architecture-patterns/domain-model.md` and `aggregates.md`.

### Permissiveness is not always convenient

Overly permissive code hides bugs. If a user-facing function guesses "the right thing", split it. A thin friendly layer may accept convenient input. Under it, a strict layer accepts only what it needs and does the work. The strict layer is the one that is easy to test. Many friendly layers can sit on one strict layer.

When in doubt, make arguments required. Optional arguments are harder to discover and can hide a choice the caller should see.

Raise exceptions. Do not catch them and print. An exception says what the code needs and lets the caller decide. Application code such as a GUI catches errors so the process does not die. Library code raises unless it knows how the caller wants the error handled.

### Useful error messages

Name the wrong value, what is wrong with it, and how it might be fixed. A missing file message says what was sought and where it was sought.

### Readability

Code is read more often than it is written. A temporary script often becomes permanent. Prefer clarity to brevity. Names are descriptive and consistent.

### Complexity is conserved

Complexity of the code is at least the complexity of the system it models. Hiding it from the caller usually backfires. The page's example replaces explicit parameters (`normalize`, `beginning`, `end`) with a single `options` dict. The signature looks shorter and the caller must now discover the keys. Scientific code should expose its real complexity because new work reuses old ideas in ways the author did not plan.

The `options={}` snippet is also a mutable default argument. That is already forbidden. The page's better shape is keyword-only parameters:

```python
def get_image(filename, *, normalize=True, beginning=0, end=None):
    ...
```

Arguments after `*` must be passed by name. `get_image('thing.png', False)` is a type error at the call. `get_image('thing.png', normalize=False)` is readable, and a new parameter can be added without breaking callers.

A second form of hidden complexity is one function that does many steps and takes many options. Prefer many small functions:

- A small function is easy to explain because its behavior is scoped.
- A small function can be tested, and untested paths are visible.
- A small function composes in ways the author did not plan. The page cites the Unix philosophy: do one thing well.
- The number of interactions grows with the number of arguments. An argument whose meaning depends on another argument should be avoided.

Return the same kind of value no matter which arguments were passed. A return type that changes with an optional flag forces the caller to know the function's internals. Python will not enforce that stability. If a function must return different kinds of things, it should be more than one function.

## Scope note

The page's closing line is about scientific tools that last. Apply the I/O split, the strict core, keyword-only options, stable returns, and useful errors in any Python library. Apply "stop writing classes" and "prefer arrays" when the data is scientific or otherwise a standard structure with no invariant of its own. Do not apply it to an aggregate.
