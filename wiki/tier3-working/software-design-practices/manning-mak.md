# Manning — Software Design for Python Programmers (Tier 3)

> **Tier 3** | Source: `references/software-design-practices/manning-mak-software-design.md` | Authority: working | Derives From: SWEBOK KA3, SOLID, `wiki/tier2-core/design-patterns/overview.md`

## Summary

Ronald Mak, *Software Design for Python Programmers* (Manning, January 2026, ISBN 9781633439498, 456 pages) is an object-oriented design book. Only the public product page was ingested. The book text, the liveBook preview, and the GitHub tree linked from the page were not read and must not be quoted or reconstructed.

The page is useful as a checklist of themes this wiki already teaches. It is not a source of pattern implementations.

## Key Concepts

### What the product page claims

The audience is a programmer who already knows Python syntax. The page says the examples are Python and the ideas apply to any object-oriented language. Public outcomes, paraphrased:

- Turn requirements into an architecture and revise that design iteratively.
- Give each class high cohesion and loose coupling.
- Use decorators to introduce an abstraction, enforce a constraint, or add behavior.
- Apply established principles so modules stay maintainable.
- Choose a design pattern for a real problem.
- Avoid bugs from unmanaged state, poorly formed classes, and inflexible functions.
- Decide data flow and the structure of functions, classes, and modules before writing the implementation.
- The page advertises before-and-after rewrites. Those rewrites were not ingested.

The author bio on the page: former NASA senior scientist, teaches at San Jose State University, author of *Object-Oriented Software Design in C++*.

### Where those claims already live

| Claim on the page | Wiki page that decides it |
|-------------------|---------------------------|
| Requirements, then architecture, revised iteratively | KA1, KA2, KA3, and the NSLS-II refactor principle |
| Cohesion and coupling | KA3 and SOLID, especially SRP |
| Decorators as a design tool | `design-patterns/structural.md` and `design-patterns/overview.md` |
| Pick a pattern for a problem | `design-patterns/overview.md` |
| Unmanaged state is a defect | `python/functional-core.md`, Foord on constructors |
| Structure before implementation | The overview's order of work |

The book is class-centric sales copy. That framing does not override the class-versus-function split on `software-design-practices/overview.md`.

## Agent Guidance

### Do

- Treat cohesion, coupling, iterative design, and deliberate pattern choice as already-required practices, with the Tier 1 and Tier 2 pages as the text.
- Cite this page only as a bibliographic pointer when someone asks whether the Manning book is in the wiki.

### Do Not

- Do not invent a Mak example, a chapter rule, or a decorator recipe and attribute it to the book.
- Do not fetch or paste the Manning source tree into the wiki. It is copyrighted book code.
- Do not rank the book above SWEBOK, the PEPs, or SOLID.
- Do not let "design classes" on the sales page create a class that has no invariant.

## Checklist

- [ ] No design decision is justified by a chapter or sample from this book
- [ ] Cohesion, coupling, and pattern choice cite KA3, SOLID, or the GoF pages
- [ ] A decorator used as design is the Decorator pattern or a plain function wrapper, documented as such

## See Also

- wiki/tier3-working/software-design-practices/overview.md
- wiki/tier1-sources/swebok-v4/ka03-design.md
- wiki/tier2-core/solid-principles/overview.md
- wiki/tier2-core/design-patterns/overview.md
- wiki/tier2-core/design-patterns/structural.md
- wiki/tier3-working/python/functional-core.md
- wiki/tier3-working/software-design-practices/refactoring-guru-python.md
- wiki/tier3-working/software-design-practices/nsls-ii-principles.md

## Source

Ronald Mak, *Software Design for Python Programmers*, Manning Publications, January 2026, ISBN 9781633439498. Product page only: https://www.manning.com/books/software-design-for-python-programmers. © Manning Publications Co. The page links source at https://github.com/RonMakBooks/SoftwareDesignPython. That tree was not ingested. Summary in `references/software-design-practices/manning-mak-software-design.md`.
