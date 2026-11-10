# pytest Patterns (Tier 3)

> **Tier 3** | Source: pytest documentation practices as used by the guides in `references/python-testing/` | Authority: working | Derives From: SWEBOK KA5, `wiki/tier3-working/python-testing/overview.md`

## Summary

`wiki/tier1-sources/swebok-v4/ka05-testing.md` sends Python test work here. This page is the pytest mechanics: discovery, plain `assert`, parametrization, fixtures, marks, expected exceptions, and async tests. The habits around what to assert and what to mock live in `wiki/tier3-working/python-testing/overview.md`. Proportions live in the test pyramid. Properties and mutation scores live in the Tier 2 pages.

Prefer pytest for a new suite. Keep `unittest.TestCase` when the repository already uses it, when a framework's runner subclasses it (Django), or when a third-party test dependency is forbidden. pytest collects `unittest` cases, so an old module can stay while new tests are functions.

## Key Concepts

### Discovery

pytest collects `test_*.py` and `*_test.py`, functions named `test_*`, and methods named `test_*` on classes named `Test*`. Match the test module to the production module: `transport.py` is exercised by `test_transport.py`. Put shared fixtures in `tests/conftest.py`. Do not give a test module a name that shadows a standard-library module.

Run one test by node id while you are editing it. Run the fast suite before a push. The full suite, including marked integration tests, runs in CI.

`python -m unittest discover -s tests` remains the right command only for a suite that has stayed on `unittest`.

### Assert

Use the `assert` statement. pytest rewrites it so a failure shows both sides. Assert equality to a value computed from the inputs, not a wide range and not a print statement. `unittest` methods (`assertEqual`, `assertIsNone`, `assertIn`, `assertIsInstance`, and their negatives) are the equivalent when the case is a `TestCase`.

Expected failure is an exception assertion:

```python
import pytest

def test_parse_rejects_empty_text():
    with pytest.raises(ValueError):
        parse("")
```

`unittest` uses `with self.assertRaises(ValueError):`. Match the exception type the caller is supposed to handle. A bare `Exception` hides the wrong failure.

### Parametrize

When several rows share one relationship, use one test and a table. Each row needs an obvious expected value.

```python
import pytest

@pytest.mark.parametrize(
    ("raw", "expected"),
    [("0", 0), ("1", 1), ("-2", -2)],
)
def test_parse_int(raw: str, expected: int) -> None:
    assert parse_int(raw) == expected
```

Do not parametrize rows that assert different behaviors. Split them so a failure name still says which behavior broke.

### Fixtures

A fixture is for work that must be undone, or for substituting one of several implementations. Yield, then clean up.

```python
import pytest

@pytest.fixture
def db_session():
    session = open_memory_session()
    yield session
    session.close()
```

Building a slightly different input belongs in a factory function that takes the difference as an argument. Tests call the factory directly so the reader sees the inputs. Scope a fixture as narrowly as the resource allows. A database session is function-scoped unless the suite has measured a reason to share it. An integration fixture that opens a server or a client belongs on the integration tests, not on the unit tier.

`@pytest.fixture(params=[...])` runs the same test against each implementation. That is the fixture use Flowers describes for dependency injection of adapters.

### Marks

| Mark | Use |
|------|-----|
| `pytest.mark.integration` | Crosses a process or a real engine. Out of the fast local loop |
| `pytest.mark.e2e` | A user-level journey. Few of these |
| `pytest.mark.slow` | Anything that cannot stay in the millisecond unit tier |
| `pytest.mark.asyncio` | An `async def` test. Requires `pytest-asyncio` |

Register custom marks in `pyproject.toml` so an unknown mark is an error. Select the fast loop with `-m "not integration and not e2e and not slow"`.

### Mocks from pytest

`unittest.mock.create_autospec` and `patch(..., autospec=True)` are the default. `pytest-mock`'s `mocker` fixture is the same library with easier cleanup. A bare `Mock()` accepts every attribute and will not notice a renamed method. Details and the fake/stub/spy split are in `wiki/tier3-working/python-testing/thea-style-guide.md`.

### Async

Install `pytest-asyncio` for `async def` tests. Patch an async collaborator with `AsyncMock`, or pass an async fake. Assert `assert_awaited_once` only when the await itself is the behavior. Do not assign `return_value` on a sync `Mock` and then await the caller. Do not `time.sleep` for readiness. `asyncio.gather` is appropriate inside a test that is itself about concurrency.

### Doctest

`doctest` examples in docstrings run as documentation of the main path. Include them in the suite (`--doctest-modules` on the package, or a small pytest module). They do not carry edge cases. A regression still gets a real test function.

### Coverage and plugins

`pytest-cov` reports coverage. The fail-under gate for code you own follows KA5 at the unit level (statement coverage above 90%, branch coverage above 80%), not a flat 80% for the whole repository. Exclude generated and vendored trees. Hypothesis and mutmut are configured from their Tier 2 pages, not reinvented here.

`pytest-benchmark` may record a callable's timing. It is not a correctness test.

## Agent Guidance

### Do

- Add a `test_` function next to the behavior you change, in the matching `test_` module.
- Use `pytest.raises` for every error path the unit claims to report.
- Parametrize tables. Use factories for varied inputs and fixtures for teardown.
- Mark anything slower than the unit tier, and keep the default local command on the fast set.
- Use `AsyncMock` or an async fake for async boundaries.

### Do Not

- Do not call the test function by hand and print "OK". Use the runner so later failures are still reported.
- Do not put integration setup in a fixture that unit tests inherit through `conftest.py`.
- Do not copy the 2018 advice to adopt nose2, Travis-specific config, or flake8 beside ruff.
- Do not set `--cov-fail-under=80` as a substitute for the KA5 unit-level targets.

## Checklist

- [ ] New tests are `test_` functions (or `unittest` methods where the repo requires them) in a `test_` module
- [ ] Assertions use `assert` or the matching `unittest` method, and expected errors use `pytest.raises` or `assertRaises`
- [ ] Repeated input rows are parametrized
- [ ] Fixtures tear down or inject an implementation; data variation goes through a factory
- [ ] Integration, end-to-end, and slow tests are marked and excluded from the fast command
- [ ] Async tests are marked `asyncio` and mock async collaborators with `AsyncMock` or an async fake
- [ ] Coverage is measured with `pytest-cov` against the KA5 unit-level targets for owned code

## See Also

- wiki/tier3-working/python-testing/overview.md
- wiki/tier3-working/python-testing/thea-style-guide.md
- wiki/tier3-working/python-testing/hitchhikers-guide.md
- wiki/tier1-sources/swebok-v4/ka05-testing.md
- wiki/tier2-core/testing-strategies/test-pyramid.md
- wiki/tier2-core/testing-strategies/property-based-testing.md
- wiki/tier3-working/python/overview.md
- wiki/tier3-working/checklists/testing-review.md

## Source

pytest discovery, fixtures, and marks as described by the structured summaries in `references/python-testing/`, checked against the Tier 1 and Tier 2 testing pages. pytest documentation: https://docs.pytest.org/
