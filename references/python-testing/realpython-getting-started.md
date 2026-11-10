# Anthony Shaw — Getting Started With Testing in Python

> Captured: 2026-11-09 from https://realpython.com/python-testing/
> Author: Anthony Shaw. Real Python. Updated 2018-10-22. Long intermediate tutorial.
> © Real Python. Structured summary for wiki ingest. Not a reproduction.
> The page's runnable tutorials, IDE screenshots, and full listings were not copied.

Read-only after ingest. Re-synthesize `wiki/tier3-working/python-testing/realpython-getting-started.md` from this summary.

## What the tutorial is

A start-from-zero tour: why automate, unit vs integration, runners, a first `unittest` case, assertions, side effects, discovery, web-framework runners, fixtures and parameterization, expected exceptions, integration layout, data fixtures, tox, CI, linters, benchmark, and bandit.

## Manual vs automated

Exploratory clicking is manual testing. A manual plan lists features, inputs, and expected results and is repeated by a person after every change. Automated testing executes that plan with a script. The tutorial's point is to move the plan into code.

## Unit vs integration

An integration check can only say that a collaboration failed. A unit check isolates which part failed. Both are written in Python. A unit test compares a component's output to a known value. The tutorial's illustration is `assert` on `sum`.

Hand-run scripts that print "Everything passed" hide every failure after the first. A test runner collects every case, reports each failure, and is the thing to use once there is more than one check.

## Runners (as of the 2018 text)

Three runners are named: `unittest` (standard library since 2.1), `nose` / `nose2`, and pytest.

- `unittest`: tests are methods on a `unittest.TestCase` subclass and use `assertEqual` and the other `assert*` methods, not a bare `assert`. Entry point is `unittest.main()`. Output marks pass as `.` and failure as `F`.
- The text says `nose` fell behind and a new project should use `nose2`, which discovers `unittest` cases. This is historical. Do not start a suite on nose or nose2.
- pytest runs `unittest` cases and, more usefully, plain functions named `test_*` that use `assert`. It can filter, rerun the last failure, and has a large plugin set.

The Python 2 note (`unittest2`) is obsolete for this wiki. Target current Python 3 only.

## Where tests live

Start with a module that can be imported (`package/__init__.py` plus the function). Put the first test in `test.py` beside the package so the import works. Split into a `tests/` directory of `test_*.py` files before one file becomes a junk drawer. Large projects may add subdirectories by purpose.

A single script that is not a package can be loaded by name. The tutorial shows `__import__("my_sum.py")`. Prefer a real package when you control the layout. A filename that clashes with the standard library (`math.py`) is a reason to be careful with imports.

## Shape of one test

Decide what behavior you are testing and whether it is a unit or an integration test. Then: build inputs, run the code, compare the output to an expected result. The tutorial's `sum` questions are the right kind of list: integers, tuples, sets, floats, a non-iterable, a negative number.

`unittest` discovery details the tutorial gives:

- `python -m unittest test` runs one module.
- `python -m unittest -v test` prints each test name.
- `python -m unittest discover` finds `test*.py`.
- `discover -s tests` sets the start directory.
- `discover -s tests -t src` sets the top-level directory that must be importable.

Failure output names the method, the case class, the traceback line, and both sides of the assertion.

The tutorial also points at PyCharm and VS Code as ways to launch `unittest` or pytest. The IDE is irrelevant to the practices.

## Assertions

Make the test repeatable. Assert a result that is tied to the input (the actual sum), not a vague range. Common `unittest` methods and their plain equivalents:

| Method | Plain check |
| --- | --- |
| `assertEqual(a, b)` | `a == b` |
| `assertTrue(x)` / `assertFalse(x)` | truth value |
| `assertIs(a, b)` | `a is b` |
| `assertIsNone(x)` | `x is None` |
| `assertIn(a, b)` | `a in b` |
| `assertIsInstance(a, b)` | `isinstance(a, b)` |

Each has a negative form (`assertIsNot`, and so on). pytest code uses the plain check instead of these methods.

Expected exceptions: `with self.assertRaises(TypeError):` around the call. The test passes only when that type is raised.

## Side effects

A side effect is a change outside the return value: an attribute, a file, a database row. Assert a side effect only when that effect is the behavior under test. Many side effects on one unit are a single-responsibility problem. Ways to deal with them: split the unit, mock the calls that leave the process, or test that collaboration as an integration test instead of pretending it is a unit test.

## Web frameworks

Django and Flask ship `unittest`-style runners because a request needs the framework's setup. Django tests subclass `django.test.TestCase`, which matches the `unittest` API and adds framework fixtures. Flask tests use the framework's test client against the app. Do not invent a second runner beside the one the framework already runs in CI.

## Advanced scenarios the tutorial names

- Fixture: reusable input data. The same word later means pytest fixtures; here it first means "the inputs you build".
- Parameterization: one test, many input rows, when the expected relationship stays the same.
- Integration tests follow the same input / execute / assert shape, touch more components, and need heavier fixtures (database, socket, config). Keep them in `tests/integration/` and do not make them part of the fastest commit check. `python -m unittest discover -s tests/integration` is the selection example. pytest marks are the current equivalent.
- Data-driven integration tests should load files from `tests/integration/fixtures/` inside `setUp`, so the case does not depend on a live remote service. For HTTP, the tutorial names the `responses` library as a way to store response fixtures. A remote API that is down must not fail the suite.

## Several environments

tox builds virtual environments and runs the suite on a matrix of Python versions and dependency sets, configured in an INI file. The idea (a matrix) is current. tox is one tool that implements it. A CI matrix or nox is an acceptable substitute. The tutorial's sample Python versions include 2.7; drop those.

## CI

Run the suite on every push, not only by hand. The tutorial's concrete file is a Travis CI config for Python 2.7 and 3.7 that installs `requirements.txt` and runs `python -m unittest discover`. Travis and those versions are historical. Keep the rule: the project CI runs discovery (or pytest) on every push.

## Linters, cleanliness, speed, security

- A linter can run in the same CI job as the tests. The tutorial shows flake8, a config that ignores a rule and raises line length, and a note that test lines are often longer (it mentions 120 as a common test-line budget). This wiki's Python overview already standardizes on ruff, which replaces flake8. Apply ruff to tests. Do not add flake8 beside ruff.
- `black` is described as an opinionated formatter with no options. Current Black does have a small configuration surface. The practice is: format in CI. The repo's formatter wins.
- Test code rots by copy-paste. Apply DRY. Fixtures and helpers exist so a behavior change does not require a sweep of duplicated setup. Lint the tests.
- Performance: `timeit` or `pytest-benchmark` can record timing of a callable. That is a separate check from correctness. Do not `sleep` inside a correctness test to "measure" it.
- Security: `bandit -r <package>` scans for common mistakes. Rules can be selected in config. Bandit complements the security-review checklist; it does not replace it. The tutorial's sample exclude of `/test` and a short test-id list is an illustration, not a required config.

## Conclusion of the tutorial

Start with small `unittest` methods if that is all you know. Move to pytest when you want plain `assert` and plugins. The practices above do not depend on staying with `unittest`.
