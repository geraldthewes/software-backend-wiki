# Michael Foord — 30 best practices for software development and testing

> Captured: 2026-11-09 from https://opensource.com/article/17/5/30-best-practices-software-development-and-testing
> Michael Foord, Opensource.com. The URL path is 2017-05.
> Foord is a Python core developer. He wrote parts of unittest and the mock library that became unittest.mock.
> Written as personal practice when he joined the Ansible team, with thanks to that team and to Wayne Witzel.
> The article says the list is not definitive, not exhaustive, and should be applied with judgment.
> Some points are Python-specific. PEP 8 is named as the first stop for Python style.
> Licensed CC BY-SA 4.0. https://creativecommons.org/licenses/by-sa/4.0/
> Structured summary. Not a reproduction.

Read-only after ingest. Re-synthesize `wiki/tier3-working/software-design-practices/foord-30-practices.md` from this summary.

## The thirty points, paraphrased

1. YAGNI. Do not write code for a future use that does not exist yet. That code dies or gets rewritten because the real use differs. Commented-out code does not ship; if it might return, delete it and point a ticket at the commit. APIs may be shaped so a future use is possible. That is design, not speculative implementation. Foord cites Kent Beck, *Extreme Programming Explained*.
2. Do not test the browser, the framework, or someone else's library unless you have a concrete reason. Test the code you write. Test infrastructure and test libraries do need tests.
3. Extract a helper the third time you write the same code, and test that helper. A helper that lives inside one test file does not need its own tests until it is reused. By the third copy the general shape is visible.
4. API design, external or object API: simple things should be simple, complex things should be possible. Design the simple case first, with as little configuration as possible. Add options or extra methods when a real complex case appears.
5. Fail fast. Check inputs and invalid state early. The error should tell the caller exactly what is wrong. Still allow surprising but valid uses: do not type-check unless the type check is actually required.
6. A unit test targets a behavior, not a line of the implementation. Changing the implementation without changing the behavior should not force the test to change. Test through the public API. Private methods and poking at state are for the rare case where that is the only way to pin a bug. Writing the test first pushes the design toward smaller units. Foord cites Kent Beck, *Test Driven Development by Example*.
7. For unit tests, including tests of test infrastructure, cover the code paths. Foord calls 100% coverage a good place to start, and says combinatorial explosion still cannot be fully covered. A path may stay untested only when it is untestable in a meaningful way, impossible in practice, or already covered. Lack of time is not a reason. Untested code is a liability. He suggests failing a change that lowers the coverage percentage.
8. Code is a liability. Write less of it. Delete what you do not need.
9. Comments rot into lies because they are not updated. Make the code readable by names and ordinary style. Comment the rare thing that cannot be made obvious: a workaround, an unlikely condition, a necessary optimization. Comment why, not what. Foord cites Kernighan and Pike, *The Practice of Programming*, and notes the point is controversial.
10. Write defensively. Ask what invalid input does and what can fail, before the bug exists.
11. Stateless, side-effect-free logic is easy to unit test. Pull that logic out of the stateful code. Test the side effect once, and replace it elsewhere. Smaller side-effect functions are easier to substitute, and the tests stay fast.
12. Globals are bad. Functions are better than new types. Objects are likely to be better than complex data structures.
13. Builtin types and their methods are faster than hand-rolled types, unless the replacement is in C. If performance matters, try to stay on the builtins.
14. Dependency injection makes dependencies visible: they arrive as parameters instead of being constructed inside the function. A signature that needs ten dependencies is a sign the function does too much. Foord cites Martin Fowler, "Inversion of Control Containers and the Dependency Injection Pattern".
15. The more you must mock to test a behavior, the worse the design. The more of the system you must stand up to reach that behavior, the worse the design. Aim for small testable units, plus integration and functional tests that show the units work together.
16. External APIs are where up-front design matters. Changing them hurts callers. Backward-incompatible changes are sometimes unavoidable and always expensive. Still keep the simple case simple.
17. A function past about 30 lines should be considered for a split. A module past about 500 lines is a candidate to split. Test files run longer than that and that is expected.
18. Do not do work in constructors. Constructors are hard to test and the work surprises callers. Do not put logic in `__init__.py` beyond imports that form the package's public names. Callers do not look there for behavior.
19. DRY matters less in tests than in production code. A test is read on its own. Some repetition is cheaper than an abstraction that hides the case. Excessive repetition can still become a helper. This agrees with the Flowers page: readability of the test beats a fixture that only manufactures data.
20. Refactor when you see the need and have the chance. Abstractions should sit close to the problem domain. A growing system outgrows its structure; leaving that in place is technical debt. Include debt paydown in the estimate for the feature. Foord cites Michael Feathers, *Working Effectively with Legacy Code*.
21. Make it correct, then make it fast. Profile before optimizing. The bottleneck is usually not where you guessed. Obscure fast code is justified only by a profile. A timed test around the hot code tells you when you are done and guards the regression, with the caveat that timing itself perturbs the measurement.
22. A small unit test names the failure. A test that boots half the system does not. Foord's rule of thumb: a test over 0.1 seconds is not a unit test. There is no such thing as a slow unit test. Tight behavioral tests are the specification. He cites Gary Bernhardt, "Fast Test, Slow Test" (PyCon US 2012).
23. Not Invented Here is a trade-off, not a sin. Code you own can be understood, maintained, and extended, and it can stay specific (YAGNI) instead of pulling in a general library's unused complexity. Owning code you do not need is still a liability (point 8). Decide deliberately when you add a dependency.
24. Shared ownership is the goal. Knowledge that lives in one person is a risk. Design decisions should be discussed or written down before the code is finished. Review is a late time to discover a design disagreement, because the code is already written. It is still better to change a bad design in review than to leave it.
25. Generators are usually shorter and clearer than a stateful object that exists to iterate or to repeat. Foord cites David Beazley, "Generator Tricks for Systems Programmers".
26. Design on purpose. Organic growth without a design becomes a monster. Over-engineering, which Foord names as onion architecture used where it is not earned, hurts as much as under-design. Match the design effort to the system. He points at Robert C. Martin in general, at *Clean Architecture*, and at the GoF book.
27. A flaky test trains people to ignore the suite. Fix it or delete it.
28. Wait for a specific condition. Do not `sleep` for a guessed interval. Those sleeps are unclear and slow the suite down.
29. See the new test fail once, by a deliberate bug or by running it before the behavior exists. A test that cannot fail does not test.
30. For managers: a constant feature grind, with no room for pride in the work and no payment of technical debt, produces a slower and buggier product.

## Conflicts to keep visible

| Foord says | Wiki rule |
|------------|-----------|
| 100% coverage is the starting point; reject any drop | KA5 unit level: statement coverage above 90%, branch coverage above 80%. Do not add a test whose only purpose is the percentage. Do not lower KA5 to chase a different number, and do not raise a gate to 100% because of point 7. Rejecting a drop can still be a project policy on top of KA5 |
| Objects beat complex data structures (point 12) | Context split. Scientific and plain data: NSLS-II, functions on standard structures. Domain invariants: architecture-patterns, a real type. Globals stay banned. New types still need a reason |
| Functions are better than types (point 12) | Same split. A function is the default. A class earns its place |
| Onion architecture is the name he gives over-engineering | Ports and adapters in this wiki are justified for a domain service. They are not the default for a script or a numeric library. Foord's warning is against using them where they are not earned |
| Not Invented Here can be right | OWASP A06 still applies. Do not reimplement cryptography, HTTP, or the standard library. A new dependency needs a maintainer, a license, and a reason |
| 30-line functions and 500-line modules | A prompt to look, not a failing gate. SWEBOK sets no line count |
| Unit test means under 0.1 seconds and public behavior | Agrees with the test pyramid and the Python testing overview. Those pages stay the testing authority |
