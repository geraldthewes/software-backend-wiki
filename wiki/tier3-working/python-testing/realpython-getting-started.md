# Getting Started With Testing in Python (Tier 3)

> **Tier 3** | Source: Anthony Shaw, "Getting Started With Testing in Python", Real Python, updated 2018-10-22 | Authority: working | Derives From: SWEBOK KA5

## Summary

Shaw's tutorial is the long start-from-zero companion to the Real Python reference card. It explains why a runner replaces a hand-run script, how a single test is shaped, where files live, how integration tests and data fixtures differ from unit tests, and which extra checks (matrix, lint, bandit, benchmark) sit beside the suite. The tutorial was updated in 2018. Its practices about independence, assertions, and layout are still usable. Its recommendations of nose2, Travis CI, flake8, and Python 2.7 are not current for this wiki.

Read `wiki/tier3-working/python/pytest-patterns.md` for the commands to type. Read this page when you are deciding what kind of test a change needs, or when you are laying out a suite that does not have one yet.

## Key Concepts

### Automate the plan

Exploratory use of the program is manual testing. A manual plan is the list of features, inputs, and expected results, repeated by a person after every edit. An automated test is that plan executed by a script. Once more than one check exists, a runner must execute them. A script that calls tests in order and prints "Everything passed" stops at the first failure and hides the rest.

### Unit and integration

An integration failure says the collaboration failed. A unit failure says which part failed. Write both. A unit compares one component's result to a known value. An integration test uses the same three steps (inputs, execute, compare) across more of the program and therefore has more side effects and heavier fixtures. Keep integration tests in `tests/integration/` or under a mark, and keep them out of the fastest commit check.

### A single test

Decide the behavior and the level first. Then build inputs, run the code, and compare. For a function like a sum, the interesting cases are each accepted container, floats, a rejected type, and negatives. Assert a result that is a function of those inputs. Repeating the test yields the same answer.

Side effects (an attribute, a file, a row) are asserted only when they are the behavior. A unit with many side effects is a single-responsibility problem: split it, replace the call that leaves the process, or promote the test to integration. That split agrees with `wiki/tier2-core/solid-principles/srp.md`.

Expected exceptions are part of the contract. `assertRaises` or `pytest.raises` wraps the act. The test fails if the exception is missing or is a different type.

### Layout and discovery

Make the code importable as a package. Name test modules `test_*.py` so discovery finds them. Start in one module if that is honest; split by area before the file is a junk drawer. `unittest` discovery flags the tutorial lists (`-s` start directory, `-t` top directory, `-v`) still work for a `unittest` suite. pytest's equivalent is a path or a node id, documented in `pytest-patterns.md`.

A one-file script can be loaded by name. Prefer a package when you control the project. Do not name a module after the standard library.

Framework tests subclass the framework's case (`django.test.TestCase`) or use its client (Flask). The assert style stays the same. The framework's runner is the one CI calls.

### Data fixtures for integration

Store the rows or payloads the integration test needs under the integration tree and load them in setup. A remote API that is down must not fail the suite. Recorded HTTP responses (the tutorial names the `responses` library) are one way to keep a client test repeatable. They are integration fixtures, not a reason to call the live service from a unit test.

### Matrix, CI, lint, speed, security

| Tutorial recommendation | Current rule |
|-------------------------|--------------|
| tox for a matrix of interpreters and dependencies | Keep the matrix. tox, nox, or the CI matrix all qualify. Drop Python 2 |
| Travis CI running discovery on every push | The suite runs on every push in the repo's CI. Do not add Travis because the tutorial shows it |
| flake8, with a longer line length for tests | ruff lints and formats tests and production code. Do not add flake8 beside ruff |
| black as a formatter with no options | The repository's formatter wins. Current Black has a small configuration surface |
| `timeit` or `pytest-benchmark` | Optional timing check. Never `sleep` inside a correctness test |
| `bandit -r` on the package | Run bandit. The security-review checklist and OWASP pages still define what a finding means |

DRY applies to tests. Copied setup becomes a factory or a fixture so a behavior change is one edit. Tests are read when the behavior changes, so a messy test is production debt.

## Agent Guidance

### Do

- Replace a hand-run assert script with a discovered test module as soon as there is a second check.
- Separate unit and integration by directory or by mark before the first integration test lands.
- Load integration data from files in the test tree, or from an in-memory engine, so the test does not depend on a shared environment.
- Add the interpreter matrix and bandit to CI when the repository does not already have them.
- Refactor a unit whose test has to assert many unrelated side effects.

### Do Not

- Do not introduce nose, nose2, Travis, flake8, or Python 2.7 because this tutorial names them.
- Do not point a unit test at a live API so that an outage fails the suite.
- Do not copy the tutorial's `__import__("file.py")` pattern when the code can be a package.
- Do not treat a benchmark as a substitute for an outcome assertion.

## Checklist

- [ ] The suite is discovered by pytest or `unittest`, not executed as a script that prints success
- [ ] Unit tests and integration tests are separated by directory or mark
- [ ] Each test builds inputs, executes, and compares to a result determined by those inputs
- [ ] Expected exceptions are asserted with `pytest.raises` or `assertRaises`
- [ ] Integration data lives in the test tree or an isolated engine
- [ ] CI runs the suite on every push
- [ ] Tests are linted with ruff, and bandit covers the application package
- [ ] No nose, Travis-only, or Python 2 configuration was added from this tutorial

## See Also

- wiki/tier3-working/python-testing/overview.md
- wiki/tier3-working/python/pytest-patterns.md
- wiki/tier3-working/python-testing/realpython-code-testing.md
- wiki/tier3-working/python-testing/hitchhikers-guide.md
- wiki/tier2-core/solid-principles/srp.md
- wiki/tier2-core/testing-strategies/test-pyramid.md
- wiki/tier1-sources/swebok-v4/ka05-testing.md
- wiki/tier3-working/checklists/security-review.md

## Source

Anthony Shaw, "Getting Started With Testing in Python", Real Python, updated 2018-10-22. https://realpython.com/python-testing/ . Structured summary: `references/python-testing/realpython-getting-started.md`. © Real Python.
