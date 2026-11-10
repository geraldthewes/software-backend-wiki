# Async Squad Labs — Python Testing Practices (Tier 3)

> **Tier 3** | Source: Async Squad Labs, "Python Testing Best Practices", 2025-10-29 | Authority: working | Derives From: SWEBOK KA5, `wiki/tier2-core/testing-strategies/test-pyramid.md`

## Summary

The Async Squad Labs guide restates the test pyramid, recommends a pytest plugin set, and lists anti-patterns that show up in web APIs and similar services: too many patches, `sleep`, and tests coupled by globals. Those anti-patterns are worth enforcing. The pyramid proportions are not new; they already live in `test-pyramid.md`. Two concrete recommendations lose to higher tiers or are incorrect as written: a coverage fail-under of 80, and an async mock that uses a sync `Mock.return_value`.

The closing section of the article is a services pitch. It is not engineering guidance.

## Key Concepts

### What the guide repeats from higher tiers

The 70% unit / 20% integration / 10% end-to-end split, Hypothesis for properties, and mutmut for mutation score are already specified. Follow the Tier 2 pages. The guide's directory sketch matches this wiki's layout: `tests/unit/`, `tests/integration/`, `tests/e2e/`, and `tests/conftest.py`.

Its unit examples (one behavior, a name that states the condition, arrange-act-assert, parametrized invalid emails) match the overview. Its integration sketches (sqlite memory session per test, FastAPI `TestClient` for a create-then-read inside one test) are integration tests, not unit tests. A repository is tested at that level, against a real engine, which `test-pyramid.md` already requires.

### Plugin set

For a new pytest suite that needs them, the guide's install list is the right short list: `pytest`, `pytest-cov`, `pytest-asyncio`, `pytest-mock`. Add Hypothesis and mutmut from the Tier 2 pages when the code under test is a transformation or is business-critical. Do not add plugins the suite does not use.

`unittest` remains the fallback for a standard-library-only environment or an existing `unittest` codebase.

### Mocks

Mock at the boundary to remove network, database, and filesystem latency, and to force an error. `side_effect` as a list is a legitimate way to express "fail, then succeed" on a sync client. A test that patches five internal helpers is the anti-pattern the guide calls the mockery. Test through the public behavior or refactor. Autospec is not discussed in the guide; it is still required by `thea-style-guide.md`.

### Async

Mark coroutine tests with `pytest.mark.asyncio`. Use `asyncio.gather` inside a test whose behavior is concurrency. Replace an async collaborator with `AsyncMock` or an async fake. The guide's listing sets `return_value` on the object `mocker.patch` returns and then awaits the service. That stand-in is not an awaitable. Do not copy it. When the call is the behavior, assert with `assert_awaited_once` (or `assert_awaited_with`), not only `assert_called_once`, so a missing await fails the test.

### Coverage gate

The guide says to aim at 80–90% and shows `--cov-fail-under=80`, focused on business logic, algorithms, error paths, and edges, and ignoring accessors, boilerplate, and third-party code. The "ignore generated and third-party code" half is right. The number 80 is not the unit-level bar. KA5 requires statement coverage above 90% and branch coverage above 80% for the unit level, on code you own. Set the CI gate from those targets. Do not fail the build on vendored trees. "Not 100% for its own sake" still holds: a test that only touches a line is not required.

### CI shape

Run pytest with coverage on push and on pull request. Provide the datastore the integration tests need as a service or as an in-memory engine. The guide shows GitHub Actions and GitLab CI sketches with Python 3.11 and Postgres, and with old action pins (`checkout@v3`, `setup-python@v4`). Copy the repository's workflow conventions and current major versions of those actions. Do not paste the sketch in.

### Anti-patterns

| Name | Shape | Fix |
|------|--------|-----|
| The mockery | Several patches of internal functions | Assert the outcome, or refactor the unit |
| The sleeper | `time.sleep`, then assert | Await the completion signal, or poll a condition with a timeout that fails the test |
| The hidden dependency | Test A stores an id in a global, test B reads it | Each test builds its own row through a fixture or a factory |

Other do-nots in the guide: skipping tests for code because it looks simple, asserting private steps, ignoring a red test, testing a third-party library's behavior, and coupling the assert to today's implementation. The guide also says unit tests should stay under 100 ms. That is a budget consistent with the pyramid's "milliseconds" rule. A test that cannot meet it is a candidate for the integration mark.

Snapshot tests (`syrupy` is the library the guide names) are a last resort for a large structured document. Prefer asserting the fields that matter. A snapshot rewritten on every failure protects nothing.

## Agent Guidance

### Do

- Place each test in `unit`, `integration`, or `e2e` according to the pyramid, not according to convenience.
- Add `pytest-asyncio` only when the suite has coroutine tests, and mock async boundaries with `AsyncMock` or an async fake.
- Fail CI on coverage of owned code using the KA5 unit-level targets.
- Treat a sleeper, a mockery, or a cross-test global as a defect in the suite.

### Do Not

- Do not adopt `--cov-fail-under=80` as the project standard.
- Do not copy the guide's async mock listing or its GitHub Actions pins.
- Do not restate the pyramid percentages as if this guide defined them.
- Do not add a snapshot test where a field assertion is still clear.
- Do not follow the article's services pitch as technical guidance.

## Checklist

- [ ] The test lives in the pyramid tier that matches its dependencies
- [ ] Async tests use `pytest-asyncio` and an async double or fake
- [ ] No test waits with `time.sleep`
- [ ] No test reads state created only by another test
- [ ] Patches sit on the I/O boundary and are autospecced
- [ ] The coverage gate matches KA5 for owned code and excludes vendored and generated files
- [ ] CI runs the suite on push and on pull request

## See Also

- wiki/tier3-working/python-testing/overview.md
- wiki/tier3-working/python/pytest-patterns.md
- wiki/tier3-working/python-testing/thea-style-guide.md
- wiki/tier2-core/testing-strategies/test-pyramid.md
- wiki/tier2-core/testing-strategies/property-based-testing.md
- wiki/tier2-core/testing-strategies/mutation-testing.md
- wiki/tier1-sources/swebok-v4/ka05-testing.md
- wiki/tier3-working/python/async-patterns.md

## Source

Async Squad Labs Team, "Python Testing Best Practices: A Complete Guide to Test Automation", 2025-10-29. https://asyncsquadlabs.com/blog/python-testing-best-practices/ . Structured summary: `references/python-testing/async-squad-best-practices.md`. © Async Squad Labs.
