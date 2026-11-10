# Hitchhiker's Guide to Python — Testing Your Code

> Captured: 2026-11-09 from https://docs.python-guide.org/writing/tests/
> Authors: Kenneth Reitz and the Real Python / Hitchhiker's Guide contributors.
> © 2011–2026 Kenneth Reitz & Real Python. Licensed CC BY-NC-SA 3.0.
> https://creativecommons.org/licenses/by-nc-sa/3.0/
> Structured summary for wiki ingest. Not a reproduction. Do not paste the guide back into the wiki.

Read-only after ingest. Re-synthesize `wiki/tier3-working/python-testing/hitchhikers-guide.md` from this summary.

## Why the guide says to test

Writing tests alongside the code clarifies intent and pushes the design toward smaller, decoupled pieces.

## Rules of testing

1. One test checks one small behavior and shows it is correct.
2. Tests are independent. Each one passes alone and in any order. Load a fresh dataset and clean up afterward (`setUp` / `tearDown`, or a fixture).
3. Tests should finish in milliseconds. A test that needs a heavy structure belongs in a separate suite run on a schedule, so the fast suite still runs constantly.
4. Know how to run one test or one case. While you work on a function, run its tests often, ideally on save.
5. Run the full suite before a session and again after it.
6. A hook should run the suite before code is pushed to a shared repository.
7. If you must stop mid-task, leave a failing test that names the next behavior. It is a bookmark.
8. When debugging, the first step is a new test that pins the bug. Those tests are among the most valuable in the project. It is not always possible; try anyway.
9. Test function names are long and descriptive because they are the failure message. `test_square_of_number_2` and `test_square_negative_number` beat `test_square`. Production code may keep short names; tests should not.
10. Tests are read as much as production code, often more, when behavior has to change. An unclear test does not help the next edit.
11. Tests introduce a newcomer to the hot spots and the corner cases. Adding a behavior starts by adding a test, including to discover that the path already exists and is not yet wired to an interface.

## Tools the guide maps

### unittest

Standard-library, xUnit-style. Subclass `unittest.TestCase` and use its assert methods. Discovery has been included since Python 2.7 (and is standard in Python 3). The guide's sample is a one-method case: a function that adds one, asserted equal to 4 for input 3.

### doctest

`doctest` finds interactive sessions in docstrings and runs them. Use them as executable examples of the main path. They are a poor net for edges and rare regressions. Run them with the rest of the suite, not as a manual extra. The guide shows a `square` docstring with two examples and a `__main__` call to `doctest.testmod()`.

### pytest

Presented as py.test, the no-boilerplate alternative. Install pytest. A test module is functions using plain `assert`. The guide's sample asserts `func(3) == 5` while `func` returns `x + 1`, so the run fails and pytest prints the assertion introspection (`4 == 5`). That sample is a picture of failure output, not a behavior to copy. The guide's claim is that this is less ceremony than the `unittest` equivalent.

### Hypothesis

Property tests are parametrized by a strategy. Hypothesis generates examples and shrinks a failure to a small one. The guide says it finds bugs that example tests miss and that it fits pytest. The captured snippet tests a mean-in-range property over lists of finite floats and then calls `mean` as if it were a function. That call does not match the local variable. Do not copy the snippet. The practice "use Hypothesis for properties" is already specified in `wiki/tier2-core/testing-strategies/property-based-testing.md`.

### tox

tox builds environments and runs the suite across interpreters from an INI file. The guide does not include a sample config.

### mock

`unittest.mock` has been in the standard library since Python 3.3. The older `mock` PyPI package is only relevant for very old interpreters. The guide shows two ideas: assign a `MagicMock` onto an instance and assert the call, and `@mock.patch` a method for the duration of one test so a search returns a fixed sequence. Patch where the name is used, not where it is defined. "Mock has many other controls" is the extent of the guide's mock advice. Autospec, fakes, and spies are not in this page; they are in the Thea Flowers summary.

## Defects and age to preserve as warnings

- The Hypothesis sample as printed is inconsistent (`mean` used as a float and as a call).
- The pytest sample is intentionally failing so the reader can see output.
- `py.test` is the old command name. The current command is `pytest`.
- The mock examples use unspecialized `MagicMock`. Prefer autospec when this wiki's style page applies.
