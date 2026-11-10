# Manning — Software Design for Python Programmers (product page)

> Captured: 2026-11-09 from https://www.manning.com/books/software-design-for-python-programmers
> Ronald Mak, Software Design for Python Programmers, Manning Publications, January 2026.
> ISBN 9781633439498. 456 pages. Catalog path: Software Development / Software Engineering / Software Architecture and Design / Software Design.
> © Manning Publications Co. All rights reserved.
> The book text was not ingested. The "look inside" reader, chapter briefs, and the GitHub tree
> https://github.com/RonMakBooks/SoftwareDesignPython were not fetched and must not be copied.
> This file records only what the public product page states.

Read-only after ingest. Re-synthesize `wiki/tier3-working/software-design-practices/manning-mak.md` from this summary.

## What the page claims the book teaches

The subtitle on the page is "Principles and patterns". The audience is a programmer who is comfortable with Python syntax. The page says the examples are Python and the techniques apply to any object-oriented language.

Public learning outcomes, paraphrased from the sales copy:

- Turn a vague requirement into an application architecture, and change that design iteratively.
- Shape classes so each one is cohesive and coupling stays loose.
- Use decorators to add an abstraction, to enforce a constraint, or to add behavior.
- Apply established design principles so code stays modular.
- Pick a design pattern for a real problem, and implement it.
- Avoid the bugs that come from unmanaged state, poorly formed classes, and inflexible functions.
- Decide how data flows, and structure functions, classes, and modules, before writing the implementation.
- The page advertises before-and-after examples of improved code.

The author bio on the page: Ronald Mak is a former NASA senior scientist, teaches data science and computer science at San Jose State University, and wrote *Object-Oriented Software Design in C++*.

## What an agent may use

The page confirms a design agenda this wiki already has:

- Cohesion and coupling are the class-design goals (SWEBOK KA3, SOLID SRP).
- Requirements precede structure (SWEBOK KA1, then KA2, then KA3).
- Iterative change of the design is expected (NSLS-II refactor principle, Foord on technical debt).
- Decorators are a Python tool for wrapping behavior (already noted on `design-patterns/overview.md`).
- Patterns are chosen for a problem (already the rule on `design-patterns/overview.md`).
- Unmanaged state is a defect. That agrees with functional core and with "do no work in `__init__`" from Foord.

## What an agent must not do

- Do not attribute a specific pattern implementation, chapter rule, or code sample to this book. Those were not read.
- Do not treat the book as higher authority than SWEBOK, PEP 8, SOLID, or the architecture-patterns pages.
- The book is an object-oriented design text. Its class-centric framing does not cancel NSLS-II's rule for scientific data: prefer functions over standard structures when there is no invariant to protect. It also does not cancel the domain-model pages when there is an invariant.

## Bibliographic facts worth keeping

| Fact | Value |
|------|--------|
| Title | Software Design for Python Programmers |
| Author | Ronald Mak |
| Publisher | Manning |
| Date on the page | January 2026 |
| ISBN | 9781633439498 |
| Length | 456 pages |
| Code link on the page | https://github.com/RonMakBooks/SoftwareDesignPython |
| Forum link on the page | Manning liveBook forum, product `mak2` |
