# Foord's 30 Practices (Tier 3)

> **Tier 3** | Source: `references/software-design-practices/foord-30-best-practices.md` | Authority: working | Derives From: SWEBOK KA3, SWEBOK KA4, SWEBOK KA5

## Summary

Michael Foord published thirty personal practices on Opensource.com in 2017, drawn from joining the Ansible team. He is a Python core developer and the author of the mock library that became `unittest.mock`. The article says the list is not dogma. It is licensed CC BY-SA 4.0.

The design points that this wiki keeps are YAGNI, the rule of three, fail fast, injection, no work in constructors, generators over iterator objects, and discussing design before the code is finished. The coverage number loses to SWEBOK KA5. The class-versus-function point is context-dependent and is decided on the overview page.

## Key Concepts

### Design rules to apply

| # | Rule |
|---|------|
| 1 | YAGNI. No code for an imaginary caller. A published API may still be shaped so a later use can be added. Commented-out code is deleted, and a ticket points at the commit if it might return |
| 3 | Rule of three. The third copy becomes a tested helper. A helper private to one test file is not a product |
| 4, 16 | Simple things are simple. Complex things are possible. Design that into external APIs up front, because breaking callers is expensive. Add options when a real case appears |
| 5, 10 | Fail fast on invalid input and invalid state, with an error that says what is wrong. Do not type-check a value that only needs to be usable |
| 8 | Code is a liability. Delete what has no caller |
| 11 | Pure logic lives in its own function. Test a side effect once |
| 14 | Pass dependencies in. A long parameter list of collaborators means the function is too big. Fowler's injection article is the citation Foord gives |
| 15 | A behavior that needs a crowd of mocks, or half the system standing, is a design problem |
| 17 | About 30 lines for a function and about 500 for a module are prompts to look, not gates. Test files may be longer |
| 18 | `__init__` stores dependencies. It does not open sockets or query. `__init__.py` publishes names and does not run behavior |
| 20 | Refactor when the abstraction no longer matches the domain, and put that cost in the feature estimate |
| 21 | Correct, then fast. Profile first. A timed test may guard a real regression |
| 23 | Owning a small specific helper can be better than a general dependency. Owning a second cryptography or HTTP stack is not. A06 still applies |
| 24 | Write the design decision down before the code lands. Review is a bad first conversation and a necessary last one |
| 25 | A generator beats a stateful object whose only job is to iterate |
| 26 | Match the design effort to the system. Ports and adapters that a domain service has earned are not the "onion" Foord is warning about. The same layers around a script are |
| 9 | Comments say why. Names say what. Foord cites Kernighan and Pike and notes the point is disputed. PEP 8 agrees with him on comment style |

### Testing points that already have a home

Points 2, 6, 7, 19, 22, 27, 28, and 29 are testing practice. Apply them from `wiki/tier3-working/python-testing/overview.md` and SWEBOK KA5.

- Test your code, not the framework, unless you have a concrete reason. Test the test infrastructure.
- Test behavior through the public surface so an implementation change does not break the test.
- Foord's "start at 100% and reject any drop" does not replace KA5 (statement coverage above 90%, branch coverage above 80% at unit level). A project may still reject a drop relative to its own baseline.
- DRY is weaker in tests than in production code. A readable test may repeat setup.
- A unit test is fast. Foord's thumb is 0.1 seconds. The pyramid's rule is the one to enforce: a unit test does not touch the network, a real service, or an arbitrary sleep.
- Fix or delete a flaky test. Wait for a condition. See a new test fail once.

Point 12 (globals are bad; functions beat types; objects beat complex structures) is decided on the overview: no globals; a function by default; a class for an invariant or a life cycle.

Point 13 (builtins are faster than hand-rolled types) is a performance hint after a profile, not a design law. It agrees with NSLS-II when the data has no invariant.

Point 30 is management: uninterrupted feature work with no debt payment slows the product. The agile and economics pages cover that. It is not a coding rule.

## Agent Guidance

### Do

- Delete speculative code and commented-out blocks. Record the commit on a ticket if the block might return.
- Shape a published function so the common call is short and a new keyword argument can be added later.
- Inject collaborators. Keep constructors free of I/O.
- Prefer a generator to a class that only walks a collection.
- Write the design note before a large change, not in the review of the finished change.

### Do Not

- Do not install a 100% coverage gate from point 7.
- Do not read point 12 as "always use classes" or "never use classes".
- Do not skip a dependency you need, and do not write your own crypto because point 23 says owning code can be fine.
- Do not treat 30 lines or 500 lines as a linter failure by themselves.
- Do not introduce ports and adapters around a one-module tool because a design book recommends layers. Do not refuse them for a domain service because Foord dislikes onion architecture.

## Checklist

- [ ] Every new branch has a current caller, or it is a compatibility seam in a published signature with no extra behavior
- [ ] A new helper is the third copy, and it has tests
- [ ] Invalid input fails before any side effect, and the error identifies the problem
- [ ] Collaborators are arguments; `__init__` does not perform I/O
- [ ] A class that only iterates has been compared with a generator
- [ ] The design of a large change is written down before the review

## See Also

- wiki/tier3-working/software-design-practices/overview.md
- wiki/tier3-working/software-design-practices/nsls-ii-principles.md
- wiki/tier1-sources/swebok-v4/ka05-testing.md
- wiki/tier3-working/python-testing/overview.md
- wiki/tier2-core/architecture-patterns/dependency-injection-bootstrap.md
- wiki/tier2-core/architecture-patterns/ports-and-adapters.md
- wiki/tier1-sources/owasp/a06-vulnerable-components.md
- wiki/tier2-core/design-patterns/behavioral.md

## Source

Michael Foord, "30 best practices for software development and testing", Opensource.com, CC BY-SA 4.0, https://opensource.com/article/17/5/30-best-practices-software-development-and-testing. Foord cites Kent Beck, *Extreme Programming Explained* and *Test Driven Development by Example*; Martin Fowler, "Inversion of Control Containers and the Dependency Injection Pattern"; Kernighan and Pike, *The Practice of Programming*; Michael Feathers, *Working Effectively with Legacy Code*; Gary Bernhardt, "Fast Test, Slow Test" (PyCon US 2012); David Beazley, "Generator Tricks for Systems Programmers"; Robert C. Martin, *Clean Architecture*. Summary in `references/software-design-practices/foord-30-best-practices.md`.
