# Async Squad Labs — Python Testing Best Practices

> Captured: 2026-11-09 from https://asyncsquadlabs.com/blog/python-testing-best-practices/
> Author: Async Squad Labs Team. Page date: 2025-10-29.
> Title on the page: "Python Testing Best Practices: A Complete Guide to Test Automation".
> © Async Squad Labs. Structured summary for wiki ingest. Not a reproduction.
> Code listings on the page were not copied. Several of them are illustrative and one is wrong for async code; see the warnings.

Read-only after ingest. Re-synthesize `wiki/tier3-working/python-testing/async-squad.md` from this summary.

## Claim

Tests give confidence to deploy, document behavior, catch defects while they are cheap, make refactoring safe, and hold a team to a quality bar. The page frames this for web APIs, data pipelines, and machine-learning code, then stays at general Python.

## Pyramid

The page repeats the standard pyramid and the 70% unit / 20% integration / 10% end-to-end split already in `wiki/tier2-core/testing-strategies/test-pyramid.md`. It does not add a new proportion. Unit tests are many, fast, and cheap. Integration tests are fewer. End-to-end tests are few, slow, and expensive.

Suggested tree:

```
tests/unit/
tests/integration/
tests/e2e/
tests/conftest.py
```

Unit scope the page names: business logic, validation, utilities, model methods. Integration: database, API endpoints, service interactions, external clients with the remote side replaced. E2E: critical journeys such as authentication, payment, and the core business process.

## Framework

pytest is the recommendation: plain asserts, fixtures, plugins, failure reports, parametrization, mocking. The install line includes `pytest`, `pytest-cov`, `pytest-asyncio`, and `pytest-mock`.

`unittest` is for a standard-library-only environment, an existing `unittest` suite, or a small need. New work in this wiki follows the pytest preference unless the repo has already chosen `unittest`.

## Unit-test habits

- One behavior per test. The page shows a "bad" test that asserts name, active flag, timestamp, and email together.
- Names state the behavior: `test_withdraw_with_insufficient_funds_raises_error`, not `test_1` or `test_account`.
- Arrange, act, assert, with a comment or a blank line between the phases when the test is long enough to need it.
- Fixtures for a standard object and for a session that rolls back and closes.
- `@pytest.mark.parametrize` for tables of inputs, including a table of invalid values that expect `pytest.raises`.

## Mocks

Mock to isolate, to avoid network, database, and filesystem, to force an error, and to reach an edge. The page shows `unittest.mock.patch` and `Mock`, and `pytest-mock`'s `mocker.patch`, including a `side_effect` list for "fail, then succeed".

Mock at the boundary. A test with five patches of internal helpers is the anti-pattern the page calls too fragile. It tells the reader to test at a higher level or to refactor.

The page does not mention autospec. Thea Flowers' rule (always spec the mock) still applies on top of this page.

## Integration sketches

- Database: a function-scoped fixture builds a sqlite memory engine, creates tables, yields a session, closes it. The test uses a repository and then reads back the row.
- HTTP API: FastAPI `TestClient` posts JSON and checks status and body. A second test creates then reads, so it is ordered inside one test rather than across tests.

These match the pyramid's integration tier. They are not unit tests.

## Async

`pytest.mark.asyncio` on `async def` tests. Concurrent work uses `asyncio.gather`.

The page's "mock an async function" sets `mock_fetch.return_value` from `mocker.patch` and then `await`s the service. A sync `Mock` return value is not an awaitable. Do not copy that listing. Use `AsyncMock`, or an async fake, and assert with the same care as a sync mock (`assert_awaited_once` when the call itself is the behavior).

## Coverage

The page says to aim for 80–90%, not 100%, and shows `--cov-fail-under=80` in `pytest.ini` or `pyproject.toml`. Focus on business logic, algorithms, error paths, and edges. Skip trivial accessors, framework boilerplate, and third-party code.

SWEBOK KA5 is stricter at the unit level (statement coverage above 90%, branch coverage above 80%). The "not 100% for its own sake" half agrees with the Real Python reference card. The fail-under number 80 does not replace the KA5 unit-level targets. A gate in CI is still a good idea; set it from the KA5 targets for the packages under test, and do not fail the build on vendored or generated code.

## CI

The page shows GitHub Actions and GitLab CI sketches: Python 3.11, a Postgres service, pytest with coverage, and an upload or a coverage artifact. The checkout action pins in the sketch are old (`actions/checkout@v3`, `setup-python@v4`). The practice is "the suite runs on push and on pull request, with the datastore the integration tests need". Copy the repo's existing workflow style rather than these pins.

## Lists the page ends with

Do: test first or immediately after the change; keep unit tests fast (the page says under 100 ms); no shared state; descriptive names; edges and errors; mock external dependencies; CI; maintain tests as production code.

Do not: skip "simple" code; test implementation details; share state; ignore a red test; `sleep` to wait; test third-party libraries; couple tests to the current implementation.

## Further techniques, already owned elsewhere

- Hypothesis examples for round-trip and commutativity. Details live in `wiki/tier2-core/testing-strategies/property-based-testing.md`.
- `mutmut run` as a way to see whether tests notice a changed program. Details live in `wiki/tier2-core/testing-strategies/mutation-testing.md`.
- Snapshot testing with `syrupy` for a large structured response. The page mentions it once. Use it only when an assertion of the fields that matter would be noise. A snapshot that is rewritten on every failure tests nothing.

## Anti-patterns

1. **The mockery**: many `@patch` decorators on internal functions. The test is brittle and can pass while the behavior is wrong.
2. **The sleeper**: `time.sleep` and then an assertion. Timing tests flake. Wait on the completion signal, or use the async test style.
3. **The hidden dependency**: test A writes a global that test B reads. Order becomes part of the suite. Each test builds what it needs, usually through a fixture.

## Marketing close

The page ends with a services pitch for Async Squad Labs. That pitch is not guidance.
