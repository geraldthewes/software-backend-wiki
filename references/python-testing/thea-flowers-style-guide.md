# Stargirl Flowers — My Python testing style guide

> Captured: 2026-11-09 from https://blog.thea.codes/my-python-testing-style-guide/
> Author: Stargirl (Thea) Flowers. Posted 2017-07-15.
> The author says the post is a catalog of personal practices, not dogma, and that it would change over time.
> © Stargirl Flowers. Structured summary for wiki ingest. Not a reproduction.

Read-only after ingest. Re-synthesize `wiki/tier3-working/python-testing/thea-style-guide.md` from this summary.

The post illustrates with a JWT / signer and with `google.auth` credentials. Those listings were not copied. The rules below are the practices.

## Terms

- **Target**: the function, method, or behavior under test.
- **Collaborator**: an object the target uses. Usually an argument or an instance attribute. Sometimes something deeper, such as a socket.
- The author does **not** split "unit test" from "integration test". She uses "unit test" for tests that stay on the machine and that isolate a behavior, not a single function with every collaborator replaced.
- **System test**: a test that talks to a real external system. Unit tests, in her usage, must not leave the machine.

This vocabulary conflicts with SWEBOK KA5 and the test pyramid, which do separate unit, integration, and system/E2E. The wiki keeps the pyramid. Her collaborator rules still apply inside an in-process test.

## Names and layout

Follow pytest discovery. A module `transport.py` is tested by `test_transport.py`.

Match the test name to the target: `refresh` is tested by `test_refresh`. Add a short condition for other behaviors: `test_refresh_failure`, `test_refresh_with_timeout`.

Functions are tested with functions. Classes may be tested with functions or with a `TestThing` class. A class of tests is useful when the module has more than one type. Instance-creation checks are named `test_constructor`. `test_default_state` is a useful companion: it builds the object and asserts the flags and empty values it should start with.

## Assert the outcome, not the path

One test, one behavior, in the same sense that a function should do one thing. Several calls are fine when the outcome needs them.

A test that encodes a token and then asserts the exact string passed to `signer.sign` knows the implementation. It breaks when the encoding steps change but the token is still correct.

A test that decodes the token and asserts the header and payload checks the outcome. Prefer that.

Lots of `assert_called*` is a smell: too much mocking, or the test knows internals it should not. Use a call assertion when the behavior under test *is* the call to the collaborator. Otherwise ask whether the outcome is enough. Write the test as if you cannot see the target's body (black box).

If the test reimplements the target, the approach is wrong.

## Prefer real collaborators

Using the real collaborator catches bugs and pressures the design to be usable without hidden side effects. If a collaborator cannot be used without side effects, that is a reason to simplify it, not an automatic reason to mock it.

The post contrasts a `create_autospec` signer plus `assert_called_once_with` against a real signer loaded from key bytes plus a decode of the resulting token. The second is the one she wants when it is possible.

## A mock always has a spec

Do not construct a bare `Mock()`. It accepts any attribute, so the target can call methods that do not exist and the test still passes. A hand-written `spec=[...]` list is only slightly better: it is disconnected from the real type, so an interface change does not fail the test.

Use `mock.create_autospec(RealType, instance=True)` or `mock.patch(..., autospec=True)`. An interface change then fails the tests that would break in production.

Relax this only when the real type cannot be imported in the test environment, such as an optional module that is not installed.

## Stub, fake, spy

- **Stub**: a few canned responses. Prefer subclassing the real collaborator so linters and interface changes still apply. The post's example is a credentials subclass with a fixed token.
- **Fake**: a working shortcut, such as a dict-backed memcache with `get` and `set`. Subclass the real type when there is one.
- **Spy**: forwards to a real object and records the calls. `Mock(wraps=real)` is the mechanism the post names. Reach for a spy when the only reason you picked Mock was to see whether a method ran.

If the mock setup is harder than the production type, write a stub or a fake instead.

## Names of doubles

Name the double as you would name the real object: `signer`, not `mock_signer` or `fake_signer`. The test reads as the behavior, and you treat the stand-in as the collaborator.

The same applies to patch parameters: the argument is `utcnow`, not `mock_utcnow`. The patch object itself, when you hold it as a context manager, may be `utcnow_patch`, and the name inside the `with` is `utcnow`. An unused patch parameter may be `unused_utcnow`.

## Factories, then fixtures

Complex collaborators come from helper functions (`make_http(...)`) so each test can pass the status or body it cares about. The post's factory sketch assigns `http` and then returns a different name (`request`). That is a slip in the article. A factory returns the object it built.

pytest fixtures are for setup and teardown that is the same across tests (start a server, yield it, stop it; open a database client, yield it, delete the rows) and for injecting several implementations (`params=['urllib3', 'requests']`). Prefer a factory when the difference between tests is an argument. The author says factories are easier to reason about.

## What this page does not decide

Pyramid proportions, coverage numbers, property tests, mutation tests, and CI shape are outside this post. Those stay on the Tier 1 and Tier 2 testing pages and on the python-testing overview.
