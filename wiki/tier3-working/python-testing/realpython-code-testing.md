# Real Python Code-Testing Reference (Tier 3)

> **Tier 3** | Source: Leodanis Pozo Ramos, Real Python, "code testing", updated 2026-08-14 | Authority: working | Derives From: SWEBOK KA5

## Summary

The Real Python best-practices reference is a one-page card, not a tutorial. It is the shortest statement of the habits the other five sources expand. An agent that only needs the card can stop at the list below, then apply the authority notes in `wiki/tier3-working/python-testing/overview.md` before choosing mocks or a coverage gate.

The card's job is to keep a suite small, exact, and runnable. It is not a second test pyramid and it does not define pytest mechanics.

## Key Concepts

### The eight practices

| Practice | What it requires | Where this wiki is stricter or narrower |
|----------|------------------|-----------------------------------------|
| Small, focused tests | One behavior or path in a short function | Same rule on the overview and on the testing-review checklist |
| Fixtures for shared setup | Temporary files, sample objects, and connections live in one place | Flowers' rule wins when the only difference is an argument: use a factory. Fixtures stay for teardown and injected implementations |
| Parametrize | One test, many explicit input and output rows, including edges | Mechanics are in `pytest-patterns.md` |
| Run often | Local runs and CI | Hitchhiker's Guide adds "before a session and before a push" |
| Reasonable coverage | Use coverage to find untested critical paths. Do not add low-value tests to reach 100% | KA5 still requires unit-level statement coverage above 90% and branch coverage above 80% on code you own |
| Fast, deterministic unit tests | No randomness, no real network, no external service | Same rule as the pyramid's unit tier |
| Mocks for external dependencies | Databases, APIs, and other unstable systems are replaced in unit tests, and the interaction is still checked | Mock the boundary. Prefer a real in-process collaborator. Assert a call only when the call is the behavior. Autospec is required |
| Fewer integration and end-to-end tests | Higher tests cover a real workflow and components together | Proportions stay 70% / 20% / 10% from `test-pyramid.md` |

### The card's contrast

A weak test draws random inputs, accepts any sum inside a wide interval, invokes the test function itself, and prints a success string. Failures are intermittent and the assertion does not say what broke.

A sound test names explicit rows, expects an exact result, and lets pytest run it. That contrast is the card's only example, and it is the pattern to copy: explicit inputs, exact expected value, runner in charge.

### Tutorials the card points at

The reference links to Real Python tutorials and courses on pytest, `unittest`, doctest, `assert`, and `unittest.mock`. Those were not ingested. Do not treat an uningested tutorial as wiki guidance. Shaw's getting-started tutorial is ingested separately because it was one of the six requested URLs.

## Agent Guidance

### Do

- Turn a vague or random assertion into explicit parametrized rows with exact results.
- Read a coverage report for critical paths that have no test, then add a behavior test, not a line-touching test.
- Keep the unit tier free of network and of unseeded randomness. A property test's generator is Hypothesis, configured from the Tier 2 page, and it is not `random.randint` inside an example test.

### Do Not

- Do not satisfy "use fixtures" by hiding every input in a fixture the test does not mention.
- Do not satisfy "use mocks" by mocking in-process logic.
- Do not treat "do not chase 100%" as permission to ship a unit with no test, or to set a fail-under of 80%.

## Checklist

- [ ] Each new test function covers one behavior
- [ ] Example tests use explicit inputs and exact expected values
- [ ] Equivalent rows are parametrized
- [ ] Unit tests are deterministic and do not call the network
- [ ] External dependencies are replaced at the boundary
- [ ] A smaller integration or end-to-end test exists for the workflow this change affects
- [ ] Coverage was used to find a gap, not as a target to game

## See Also

- wiki/tier3-working/python-testing/overview.md
- wiki/tier3-working/python/pytest-patterns.md
- wiki/tier3-working/python-testing/thea-style-guide.md
- wiki/tier2-core/testing-strategies/test-pyramid.md
- wiki/tier1-sources/swebok-v4/ka05-testing.md

## Source

Leodanis Pozo Ramos, "code testing", Real Python best-practices reference, updated 2026-08-14, reviewed by Brenda Weleschuk and Bartosz Zaczyński. https://realpython.com/ref/best-practices/code-testing/ . Structured summary: `references/python-testing/realpython-code-testing.md`. © Real Python.
