# Unit-Test Practices from Madhu Deepak (Tier 3)

> **Tier 3** | Source: Madhu Deepak, "Best Practices for Writing Unit Tests in Python", Medium, 2025-01-20 | Authority: working | Derives From: SWEBOK KA5

## Summary

Deepak's article is a twelve-point unit-test checklist for Python: why the test exists, which runner to pick, Arrange–Act–Assert, edges, isolation, names, size, fixtures, cadence, coverage, comments, and CI. Most of the twelve points match the overview. Two items need care. The article's mock sample never calls production code, so it is not a pattern. The article does not set a coverage percentage; KA5 still does.

The Medium page that hosts the article also recommends unrelated posts. Those posts are not this source. Do not follow them from this page.

## Key Concepts

### Role of the test

A unit test checks one function or method, finds the defect before a user does, stops a later edit from changing that behavior silently, and documents the expected result. External systems are not part of the unit.

### Runner

`unittest` is the standard-library baseline and is appropriate for a small suite or a constraint against third-party dependencies. pytest is the article's choice once the suite needs fixtures, parameters, or plugins. That matches this wiki's default for new Python suites.

### Arrange–Act–Assert

| Phase | Content |
|-------|---------|
| Arrange | Inputs and collaborators |
| Act | The single call or state change under test |
| Assert | The expected result or the expected exception |

Separate the phases with blank lines or short comments when the arrange block is long. Do not interleave a second behavior's act and assert in the same function.

### Edges

Empty collections and strings, minimum and maximum values, wrong types, and out-of-range values. An expected exception is `pytest.raises` around the act, not an assertion that happens to fail for some other reason.

### The other points

- Isolation: replace a database or an API. Apply autospec and the boundary rule from the overview. Do not copy the article's sample, which builds a `Mock`, calls the mock, and asserts the return value it just configured.
- Names describe the behavior (`test_calculate_discount_for_loyal_customer`), not `test_1`.
- One behavior per test. Two functions, two tests.
- Fixtures hold setup that more than one test shares. Factories still win for data that varies; see the Flowers page.
- Run the suite often. `pytest-watch` (`ptw`) is an optional local loop on save. It does not replace CI, and it is not required if the editor or a file watcher already reruns pytest.
- coverage.py shows which paths ran. Spend the next test on a critical or complex path. The article's "quality over quantity" does not authorize skipping the KA5 unit-level targets.
- A comment or docstring belongs on a test whose scenario is not already in the name. Do not restate `assert result == 10`.
- CI runs pytest on push and on pull request. The article's GitHub Actions sketch (checkout, setup Python, install pytest, run pytest) is the shape, not a pin set to copy. Use the repository's existing workflow.

## Agent Guidance

### Do

- Structure every new unit test as arrange, act, assert.
- Add the empty, boundary, and invalid-input cases for the behavior you touch.
- Put the suite on the existing CI workflow if this change introduces the first tests.
- Comment a test only when the name cannot carry the scenario.

### Do Not

- Do not use the article's mock sample as a template. A test that does not execute production code is not a test of the unit.
- Do not install `pytest-watch` as a project requirement unless the team asked for a watch loop.
- Do not treat the article's silence on a coverage number as a lower bar than KA5.
- Do not open the unrelated posts linked from the Medium sidebar as testing guidance.

## Checklist

- [ ] The test has an arrange phase, one act, and an assert of the outcome
- [ ] Edge and invalid inputs for the changed behavior are covered, including `pytest.raises` where the unit reports an error
- [ ] The name states the behavior
- [ ] External systems are isolated at the boundary, and the test calls the production unit
- [ ] The suite is runnable locally and is on CI
- [ ] A comment is present only where the name is not enough

## See Also

- wiki/tier3-working/python-testing/overview.md
- wiki/tier3-working/python/pytest-patterns.md
- wiki/tier3-working/python-testing/realpython-code-testing.md
- wiki/tier3-working/python-testing/thea-style-guide.md
- wiki/tier1-sources/swebok-v4/ka05-testing.md
- wiki/tier3-working/checklists/testing-review.md

## Source

Madhu Deepak, "Best Practices for Writing Unit Tests in Python", Medium, 2025-01-20. https://madhudeepak.medium.com/best-practices-for-writing-unit-tests-in-python-cd1da23d3b79 . Structured summary: `references/python-testing/madhu-deepak-unit-tests.md`. © Madhu Deepak.
