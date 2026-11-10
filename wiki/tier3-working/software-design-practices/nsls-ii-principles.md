# NSLS-II Guiding Design Principles (Tier 3)

> **Tier 3** | Source: `references/software-design-practices/nsls-ii-guiding-design-principles.md` | Authority: working | Derives From: SWEBOK KA3, `wiki/tier3-working/python/functional-core.md`

## Summary

The Scientific Python Cookiecutter states ten principles for scientific Python. Six of them are general library design: collaborate, refactor under tests, keep a design wide enough to reuse, split a guessing layer from a strict core, raise useful errors, and do not hide complexity. Two are specific to scientific data: duck-type the array-like interface, and prefer functions on standard arrays over a custom class. The second of those does not apply to a domain aggregate.

The repository is BSD 3-Clause, Copyright (c) 2019, Brookhaven National Laboratory. This page is a summary. The cookiecutter's packaging and Travis CI chapters were not ingested. Travis CI is not current guidance.

## Key Concepts

| Principle | Rule |
|-----------|------|
| Collaborate | More than one person should understand each part. Shared conventions beat a private style. Talk through assumptions before they harden |
| Refactor | The first structure is provisional. Change it once the trade-off is clear. Version control, tests, and lint are what make that safe |
| Wide over deep | A reusable piece should support a use the author did not plan, without a large jump in complexity. Understand the lowest level, then build up |
| I/O separate | A function that takes a path only does I/O and returns plain data. Scientific or business logic takes and returns plain data |
| Duck typing | Accept anything that provides the needed behavior. Skip `isinstance` unless the contract really is one concrete type. A `Protocol` is the typed form |
| Stop writing classes | For scientific data, prefer an array or a dict plus functions. Metadata travels beside the array. scikit-image's dropped `Image` class is the page's example |
| Strict core | A friendly guessing function sits on a strict function. Test the strict one. Required arguments stay required |
| Errors | Library code raises. The message names the bad value, what is wrong, and where it looked. Application code is what catches |
| Readability | Optimize for the next reader. Temporary code becomes permanent |
| Complexity is conserved | Do not fold real parameters into an `options` dict. Prefer keyword-only parameters. One function does one thing. The return kind does not depend on a flag |

Perlis, as quoted by the page: it is better to have many functions operate on one data structure than a few functions on many data structures. That quote supports the scientific-data rule. It is not a reason to remove an invariant from an aggregate.

Keyword-only form the page recommends:

```python
def get_image(filename: str, *, normalize: bool = True, beginning: int = 0, end: int | None = None) -> object:
    ...
```

`get_image(path, False)` is then a call-site error. `get_image(path, normalize=False)` is readable, and a new keyword can be added later. The page's `options={}` alternative is a mutable default and is forbidden.

## Agent Guidance

### Do

- Split path and format handling from the function that decides.
- Put guesses and defaults in a thin outer function. Keep the inner function strict and tested.
- Use keyword-only arguments for options that a positional call would obscure.
- Write error text that a caller can act on without reading the implementation.
- Refactor a narrow design once a second real use appears, with tests already in place.

### Do Not

- Do not wrap a NumPy array in a custom class so that metadata can live on the instance. Pass the metadata beside the array.
- Do not apply "stop writing classes" to a domain aggregate. That rule is on `architecture-patterns/aggregates.md`.
- Do not catch and print inside a library.
- Do not let the return type change when an optional flag changes.
- Do not treat the rest of the cookiecutter tutorial as ingested. Its CI chapter names Travis CI.

## Checklist

- [ ] I/O and decision logic are separate functions
- [ ] Optional flags are keyword-only, not keys in a dict argument
- [ ] The return type is the same for every successful call
- [ ] A custom class is absent unless it protects an invariant or matches a non-scientific collaborator
- [ ] `isinstance` is absent unless a concrete type is the contract
- [ ] Library errors propagate, and the message names the bad value

## See Also

- wiki/tier3-working/software-design-practices/overview.md
- wiki/tier3-working/python/functional-core.md
- wiki/tier3-working/python/type-system.md
- wiki/tier2-core/architecture-patterns/domain-model.md
- wiki/tier2-core/architecture-patterns/aggregates.md
- wiki/tier1-sources/python-peps/pep-020-zen.md
- wiki/tier3-working/software-design-practices/foord-30-practices.md

## Source

NSLS-II, "Guiding Design Principles", Scientific Python Cookiecutter 0.1 documentation, https://nsls-ii.github.io/scientific-python-cookiecutter/guiding-design-principles.html. Repository license BSD 3-Clause, Copyright (c) 2019, Brookhaven National Laboratory, https://github.com/NSLS-II/scientific-python-cookiecutter. Alan J. Perlis, "Epigrams in Programming", ACM SIGPLAN Notices, September 1982, as cited by that page. Jack Diederich, "Stop Writing Classes", as cited by that page. Summary in `references/software-design-practices/nsls-ii-guiding-design-principles.md`.
