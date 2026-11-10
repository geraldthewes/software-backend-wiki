# Real Python — code testing (best-practices reference)

> Captured: 2026-11-09 from https://realpython.com/ref/best-practices/code-testing/
> Author: Leodanis Pozo Ramos. Reviewed by Brenda Weleschuk and Bartosz Zaczyński.
> Updated: 2026-08-14.
> © Real Python. Structured summary for wiki ingest. Not a reproduction.

Read-only after ingest. Re-synthesize `wiki/tier3-working/python-testing/realpython-code-testing.md` from this summary.

## Claim

Tests keep a codebase safe to change. `unittest` and pytest organize tests, share fixtures, and run them automatically. Without tests, small edits have surprising side effects.

## Practices the page lists

1. Write small, focused tests. One behavior or path per short test function.
2. Put shared setup (temporary files, sample objects, database connections) in fixtures so it is not copied into every test.
3. Use pytest parametrization to run one test across inputs and expected outputs, including edges, instead of near-duplicate tests.
4. Run tests often, locally and in CI.
5. Use coverage reports to find untested critical paths. Do not chase 100% with low-value tests.
6. Keep unit tests fast and deterministic. No randomness, no real network, no external services.
7. Replace databases, APIs, and other slow or unstable dependencies with mocks in unit tests, and still check how the code talks to those resources.
8. Add a smaller set of integration and end-to-end tests for real workflows and for components working together in a production-like environment.

## Contrast the page draws

A weak test uses `random.randint`, checks that a sum merely falls inside a wide interval, calls the test by hand, and prints "OK".

A sound test is a pytest function with `@pytest.mark.parametrize` rows of explicit inputs and exact expected results, and a plain `assert` of equality.

## Related tutorials named, not ingested

pytest tutorial, unittest tutorial, doctest, assert, mock object library, and several Real Python courses and quizzes. Those URLs were not fetched for this ingest.

## Tension with other ingested sources

Fixtures-for-all-setup is broader than Thea Flowers' "factories first, fixtures when there is teardown or injected implementations". Mocking external dependencies agrees with the test pyramid and is narrower than "mock any collaborator". "Do not chase 100%" agrees with quality-over-percentage, and does not lower the SWEBOK unit-level coverage targets.
