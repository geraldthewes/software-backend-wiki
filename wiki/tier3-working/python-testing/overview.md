# Python Testing Best Practices (Tier 3)

> **Tier 3** | Source: six practitioner guides in `references/python-testing/` | Authority: working | Derives From: SWEBOK KA5, `wiki/tier2-core/testing-strategies/`

## Summary

This page is the working guide for writing and reviewing Python tests. It reconciles six practitioner sources captured on 2026-11-09. The sources agree on a small set of habits: one behavior per test, a name that states that behavior, an assertion of the outcome, deterministic inputs, and a suite that runs in CI. They disagree on how hard to isolate a unit and on the coverage number that should fail a build.

SWEBOK KA5 and the Tier 2 testing pages win those disagreements. Use this page before writing a Python test. Open a source page only when you need that author's specific rule. Open `wiki/tier3-working/python/pytest-patterns.md` for pytest mechanics.

## Key Concepts

### Source map

| Page | Source | Read it for |
|------|--------|-------------|
| `wiki/tier3-working/python-testing/realpython-code-testing.md` | Leodanis Pozo Ramos, Real Python reference, updated 2026-08-14 | The short card: focus, parametrize, deterministic unit tests, coverage judgment |
| `wiki/tier3-working/python-testing/madhu-unit-tests.md` | Madhu Deepak, Medium, 2025-01-20 | Arrange–Act–Assert, edges, names, run often, CI |
| `wiki/tier3-working/python-testing/realpython-getting-started.md` | Anthony Shaw, Real Python, updated 2018-10-22 | Suite shape: runners, assertions, side effects, integration layout, matrix, lint, bandit. Several named tools are dated |
| `wiki/tier3-working/python-testing/hitchhikers-guide.md` | Kenneth Reitz and Real Python, Hitchhiker's Guide, CC BY-NC-SA 3.0 | Independence, speed, tests as documentation, bug-first tests, doctest versus unit tests |
| `wiki/tier3-working/python-testing/thea-style-guide.md` | Stargirl Flowers, 2017-07-15 | Outcome versus call sequence, real collaborators, autospec, stubs, fakes, spies, factories versus fixtures |
| `wiki/tier3-working/python-testing/async-squad.md` | Async Squad Labs, 2025-10-29 | Plugin set, async tests, CI shape, anti-patterns. The coverage gate and one async example do not override higher tiers |

### Authority resolution

| Conflict | Winner | Rule an agent applies |
|----------|--------|------------------------|
| Flowers does not separate unit tests from integration tests | `test-pyramid.md` and KA5 | Keep unit, integration, and end-to-end. Her "system test leaves the machine" is the slow tier. Her collaborator rules apply inside a test that stays in-process |
| Real Python and Deepak: mock external dependencies. Flowers: prefer real objects; a string of `assert_called*` is a smell | Both, and the pyramid picks the boundary | Fake or mock the I/O boundary (network, clock, filesystem, external API). Use the real object or a fake for an in-process collaborator. Assert the outcome. Assert a call only when that call is the behavior under test |
| Real Python and Deepak: put setup in fixtures. Flowers: factories first | Flowers' narrower rule | A fixture that only builds data is a function. A fixture earns its place when it must tear down, or when it injects one of several implementations |
| Async Squad: fail the build under 80% and aim for 80–90%. Real Python: do not chase 100%. KA5 unit level: statement coverage above 90%, branch coverage above 80% | KA5 | Meet the KA5 targets on code you own. Do not add a test whose only purpose is the percentage. Mutation score on critical logic stays in `mutation-testing.md` |
| Shaw 2018: nose2, Travis CI, flake8, Python 2.7 | `wiki/tier3-working/python/overview.md` | New suites use pytest. Lint and format with ruff. CI is the workflow the repo already runs on every push. A matrix of interpreters is tox, nox, or CI. nose and Python 2 are not starting points |
| Deepak's mock sample calls the mock and asserts the canned return | This page | That sample tests the mock, not production code. Do not copy it |
| Async Squad sets a sync `Mock.return_value` and then awaits the caller | `pytest-patterns.md` | An async collaborator is an `AsyncMock` or an async fake |
| Hitchhiker's Guide Hypothesis sample calls `mean` as a function after binding a float | `property-based-testing.md` | Write properties from the Tier 2 page. Do not copy the guide's sample |

### Layout

```
tests/
  conftest.py
  unit/
  integration/
  e2e/
```

Mark integration, end-to-end, and slow tests. The unit tier does no network and no real database. A repository is an integration test against a real engine, including sqlite in memory, as `test-pyramid.md` already says.

### What one test contains

Arrange the inputs. Act once, on one behavior. Assert the outcome: the return value, the raised exception, or the state the behavior is supposed to leave. A test may call the target more than once when the outcome needs that. A test does not assert the private steps that produced a correct outcome.

Name the file `test_<module>.py`. Name the function for the behavior and the condition, for example `test_withdraw_with_insufficient_funds_raises_error`. Group methods in `Test<Type>` only when one module has several types. `test_constructor` and `test_default_state` cover instance creation.

## Agent Guidance

### Do

- Use pytest for a new suite. Keep `unittest` when the project already standardizes on it or cannot take a third-party dependency. Django and Flask tests use the framework's `TestCase` or test client.
- Write the failing test that pins a bug before changing production code, when you can reproduce the bug.
- Parametrize rows that share one relationship between input and output. Include empty input, boundaries, and the expected exception (`pytest.raises`).
- Give every mock a spec (`create_autospec` or `patch(..., autospec=True)`). Name the double the way you would name the real object.
- Prefer a stub or a fake that subclasses the real type when the mock setup starts reimplementing the collaborator. Use `Mock(wraps=real)` when you need the real behavior and a record of the call.
- Run the fast suite on every push. Keep a heavy or scheduled suite out of that loop. Run doctests with the suite; they document the main path and do not replace edge tests.
- Lint and format tests with the same tools as production code. Treat a red test as a defect, including a flaky one.
- Scan the application with bandit as part of the security check. The security-review checklist still applies.

### Do Not

- Do not collapse unit and integration tests into one bucket because the Flowers guide uses "unit" for every in-process test.
- Do not mock internal helpers so the test only checks that mocks were called. That is the "mockery" anti-pattern.
- Do not use `random` in a unit test, `time.sleep` to wait for readiness, or a shared global written by one test and read by another.
- Do not assert a call sequence when the observable outcome would catch the same bug.
- Do not copy a sample from these sources that the authority table marks as broken.
- Do not lower the KA5 unit-level coverage targets to Async Squad's 80% gate.
- Do not start a new suite on nose, nose2, or Python 2.
- Do not add a snapshot assertion when asserting the fields that matter is still readable. A snapshot that is updated whenever it fails does not protect behavior.

## Checklist

- [ ] The test file is `test_<module>.py` and the test name states the behavior and the condition
- [ ] The test arranges, acts, and asserts one behavior, and the assertion is the outcome
- [ ] Inputs are explicit. Empty, boundary, and error cases are present or parametrized
- [ ] The test passes alone and does not depend on another test's state
- [ ] Unit tests do not touch the network, a real external service, or the clock without a fake
- [ ] Any mock is autospecced, named as the collaborator, and used at an I/O boundary
- [ ] In-process collaborators are real objects or fakes unless a side effect forces a boundary double
- [ ] Integration and end-to-end tests are marked and are not the fast suite
- [ ] The suite runs in CI on every push
- [ ] Unit-level statement and branch coverage meet KA5, and critical logic has a mutation-score plan from the Tier 2 page

## See Also

- wiki/tier1-sources/swebok-v4/ka05-testing.md
- wiki/tier2-core/testing-strategies/overview.md
- wiki/tier2-core/testing-strategies/test-pyramid.md
- wiki/tier2-core/testing-strategies/property-based-testing.md
- wiki/tier2-core/testing-strategies/mutation-testing.md
- wiki/tier3-working/python/pytest-patterns.md
- wiki/tier3-working/python-testing/thea-style-guide.md
- wiki/tier3-working/checklists/testing-review.md

## Source

Structured summaries in `references/python-testing/` of: Leodanis Pozo Ramos, Real Python, "code testing" (updated 2026-08-14), https://realpython.com/ref/best-practices/code-testing/ ; Madhu Deepak, "Best Practices for Writing Unit Tests in Python" (Medium, 2025-01-20), https://madhudeepak.medium.com/best-practices-for-writing-unit-tests-in-python-cd1da23d3b79 ; Anthony Shaw, "Getting Started With Testing in Python" (Real Python, updated 2018-10-22), https://realpython.com/python-testing/ ; Kenneth Reitz and Real Python, "Testing Your Code", The Hitchhiker's Guide to Python, CC BY-NC-SA 3.0, https://docs.python-guide.org/writing/tests/ ; Stargirl Flowers, "My Python testing style guide" (2017-07-15), https://blog.thea.codes/my-python-testing-style-guide/ ; Async Squad Labs, "Python Testing Best Practices" (2025-10-29), https://asyncsquadlabs.com/blog/python-testing-best-practices/.
