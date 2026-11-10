# Madhu Deepak — Best Practices for Writing Unit Tests in Python

> Captured: 2026-11-09 from https://madhudeepak.medium.com/best-practices-for-writing-unit-tests-in-python-cd1da23d3b79
> Author: Madhu Deepak. Medium, 2025-01-20. About a 4 minute read.
> © Madhu Deepak. Structured summary for wiki ingest. Not a reproduction.

Read-only after ingest. Re-synthesize `wiki/tier3-working/python-testing/madhu-unit-tests.md` from this summary.

Unrelated posts shown by Medium beside this article (including security-bypass writeups) are not part of the article and were not summarized.

## Claim

A unit test checks one function or method in isolation, finds bugs early, guards later edits, and documents expected behavior. It should not depend on a database or an API.

## Framework

`unittest` is the standard-library baseline. pytest is preferred for larger suites because tests are shorter, fixtures and parametrization are built in, and plugins exist. The article says unittest is enough for beginners and pytest fits more complex projects.

## Twelve practices

1. Know the role of a unit test: one unit, no external systems.
2. Pick pytest for complexity; unittest when you are starting or must stay in the standard library.
3. Arrange–Act–Assert. Arrange data, call the code, assert the result.
4. Test edges: empty inputs, min/max boundaries, wrong types, out-of-range values. Expected exceptions use `pytest.raises`.
5. Isolate with mocks for databases and APIs.
6. Name the behavior under test, not `test_1`.
7. One behavior per test. Do not stack unrelated assertions.
8. Share setup with pytest fixtures.
9. Run tests often. The article names `pytest-watch` (`ptw`) to rerun on file changes.
10. Measure coverage with coverage.py. Prefer critical and complex paths over a raw percentage.
11. Comment or docstring a test when the scenario is not obvious from the name.
12. Run the suite in CI on every push and pull request. The sketch is GitHub Actions: checkout, setup Python, install pytest, run pytest.

## Weak example in the source

The mocking sample constructs a `Mock`, sets `get_data.return_value`, calls `mock_api.get_data()`, and asserts that canned value. It never calls production code. Do not treat it as a pattern. The isolation *advice* (replace external systems) still stands; the sample does not demonstrate it.

## Tension with other ingested sources

Same fixture-vs-factory and mock-vs-real tensions as the Real Python reference card. `pytest-watch` is optional local feedback, not a substitute for CI. Coverage "quality over quantity" does not set a number; SWEBOK KA5 still owns the unit-level targets.
