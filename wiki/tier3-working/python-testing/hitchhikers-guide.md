# Hitchhiker's Guide — Testing Your Code (Tier 3)

> **Tier 3** | Source: Kenneth Reitz and Real Python, The Hitchhiker's Guide to Python, "Testing Your Code" | Authority: working | Derives From: SWEBOK KA5 | License of the guide: CC BY-NC-SA 3.0

## Summary

The Hitchhiker's Guide chapter is the etiquette of a Python suite: tests are small, independent, fast, named so a failure is readable, and treated as documentation. It then maps the tools a reader in that tradition would install: `unittest`, doctest, pytest, Hypothesis, tox, and `unittest.mock`. The tool map is a pointer. pytest mechanics are in `pytest-patterns.md`. Hypothesis and mutation testing stay on their Tier 2 pages. Mock discipline beyond "you can patch" is on the Flowers page.

The guide is licensed CC BY-NC-SA 3.0. This page is a summary. Do not copy the guide's prose or samples into new documents.

## Key Concepts

### Rules that change how an agent works

| Rule | Agent action |
|------|----------------|
| One tiny behavior | Split a test that proves two things |
| Independent of order | Fresh inputs in every test, cleanup in teardown. No module-level mutable that another test reads |
| Milliseconds | A slow case moves to a marked or scheduled suite so the fast suite still runs constantly |
| Run one test | Use a node id while editing. Do not run the world to see one failure |
| Full suite before and after a session | Run the fast suite at the start of a change and again before handing it over |
| Gate before a shared push | CI, or a pre-push hook, runs the suite. A red result blocks the push |
| Leave a failing test when you stop | The next session starts from a named, failing behavior instead of a blank diff |
| First debugging step is a test | Add a test that fails on the bug, then change production code until it passes |
| Long test names | The name is the failure message. `test_square_negative_number`, not `test_sq` |
| Tests are read | Write them so a maintainer can change behavior from the test, without reading the implementation first |
| Tests onboard | A new behavior starts with a test, which also checks that the path does not already exist unexposed |

These rules agree with KA5 and with the testing-review checklist. They are more specific about session cadence and about the bug-first test than the Real Python card is.

### Tool map

| Tool | Role in this wiki |
|------|-------------------|
| `unittest` | Standard-library runner. Valid when pytest is not allowed, and as the base Django's `TestCase` extends. Discovery is built in |
| doctest | Executable examples of the main path inside docstrings. Run them with the suite. They are not the edge-case net |
| pytest | Default runner. The guide's command name `py.test` is the old one. Use `pytest`. Its sample test is written to fail so the reader can see assertion output; do not treat the expected value in that sample as correct |
| Hypothesis | Use it. Write the properties from `property-based-testing.md`. The guide's mean example, as printed, uses `mean` both as a float and as a call. Do not copy it |
| tox | One way to run a matrix of interpreters. nox or CI matrix is equivalent. See the Shaw page for the same rule |
| `unittest.mock` | In the standard library since Python 3.3. Patch the name where it is used. The guide's samples use a bare `MagicMock`. Autospec, from the Flowers page, is required in this wiki |

## Agent Guidance

### Do

- Add a failing test that names the bug before editing the production function, when the bug can be reproduced in-process.
- If a session has to stop mid-feature, leave one failing test that states the next behavior.
- Run the fast suite at the start and at the end of the change.
- Keep doctest examples on the main path and a `test_` function on every edge.
- Name tests so the failure line is the explanation.

### Do Not

- Do not let one test's data satisfy the next test.
- Do not park a multi-second case in the fast suite.
- Do not copy the guide's Hypothesis sample or its intentionally failing pytest sample.
- Do not stop at a bare `MagicMock` because the guide's mock section stops there.
- Do not reproduce the guide's text. The license is CC BY-NC-SA 3.0 and this wiki's summary is the artifact agents read.

## Checklist

- [ ] The new test checks one behavior and passes by itself
- [ ] The test name is specific enough to read as the failure message
- [ ] A reproduced bug has a failing test added before the fix
- [ ] Slow tests are outside the fast suite
- [ ] Doctests, if any, run with the suite and do not replace edge tests
- [ ] The fast suite was run after the change
- [ ] No sample from the guide was copied into the repository

## See Also

- wiki/tier3-working/python-testing/overview.md
- wiki/tier3-working/python/pytest-patterns.md
- wiki/tier3-working/python-testing/thea-style-guide.md
- wiki/tier3-working/python-testing/realpython-getting-started.md
- wiki/tier2-core/testing-strategies/property-based-testing.md
- wiki/tier2-core/testing-strategies/test-pyramid.md
- wiki/tier1-sources/swebok-v4/ka05-testing.md
- wiki/tier3-working/checklists/testing-review.md

## Source

Kenneth Reitz and Real Python, "Testing Your Code", The Hitchhiker's Guide to Python. https://docs.python-guide.org/writing/tests/ . © 2011–2026 Kenneth Reitz & Real Python, CC BY-NC-SA 3.0, https://creativecommons.org/licenses/by-nc-sa/3.0/ . Structured summary: `references/python-testing/hitchhikers-guide-testing.md`.
