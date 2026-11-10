# Python Testing Style Guide — Stargirl Flowers (Tier 3)

> **Tier 3** | Source: Stargirl Flowers, "My Python testing style guide", 2017-07-15 | Authority: working | Derives From: SWEBOK KA5, `wiki/tier2-core/testing-strategies/test-pyramid.md`

## Summary

Flowers' post is the style guide for collaborators and assertions. It is the source that tells an agent to assert the outcome, to use a real object when the test can stay in-process, to spec every mock, and to prefer a factory over a fixture. The author calls the post a personal catalog, not dogma.

One term in the post does not carry over. Flowers uses "unit test" for any test that stays on the machine and "system test" for a test that talks to a real external system. She does not maintain a separate integration tier. SWEBOK KA5 and `test-pyramid.md` do. Keep the three levels. Apply her collaborator rules inside whichever in-process test you are writing. Her system test is the slow end of the pyramid.

## Key Concepts

### Terms

| Term | Meaning here |
|------|----------------|
| Target | The function, method, or behavior under test |
| Collaborator | An object the target uses, usually an argument or an attribute |
| In-process test | Does not leave the machine. Pyramid level is unit or integration depending on whether a real engine is involved |
| System test | Talks to a real external system. Pyramid: end-to-end, or an integration test that cannot be faked |

### Names

pytest discovery. `transport.py` is tested by `test_transport.py`. `refresh` is tested by `test_refresh`. Other cases add the condition: `test_refresh_failure`, `test_refresh_with_timeout`. Test functions with functions. Use a `TestThing` class when the module has several types and grouping helps. `test_constructor` covers creation. `test_default_state` asserts the flags and empty values a new instance should have.

### Outcome, not path

One test, one behavior. More than one call is allowed when the outcome needs it. A test that asserts the exact string passed into a signer is coupled to the encoding steps. A test that decodes the resulting token and checks the header and payload survives an internal rewrite. A run of `assert_called*` is a smell unless the call is the behavior. Write the test as if the target's body is hidden. If the test reimplements the target, stop and assert the observable result instead.

### Real objects, then doubles

Use the real collaborator when doing so stays in-process and has no unwanted side effect. That finds integration bugs inside the process and forces the collaborator to be usable without hidden I/O. If the real object cannot be constructed without a side effect, simplify the production design or move the test up a pyramid level. Do not reach for a mock as the first move.

When a double is required:

| Double | What it is | When |
|--------|------------|------|
| Stub | Canned answers. Subclass the real type | The test needs a few fixed responses |
| Fake | A working shortcut, such as a dict-backed client. Subclass the real type | The test needs behavior, not a call log |
| Spy | `Mock(wraps=real)` so calls are recorded and still happen | You need the real behavior and proof the call ran |
| Autospec mock | `create_autospec(Real, instance=True)` or `patch(..., autospec=True)` | The boundary cannot run for real, and you need a stand-in that rejects unknown attributes |

A bare `Mock()` accepts any attribute, so a renamed or misspelled call still passes. A hand-written `spec=["method"]` list is disconnected from the type and goes stale. Autospec fails the test when the interface changes. Skip autospec only when the real type cannot be imported in the test environment.

Name the double as the real object: `signer`, not `mock_signer` or `fake_signer`. A patch parameter is `utcnow`, not `mock_utcnow`. The patch object in a `with` statement may be `utcnow_patch`; the bound name is `utcnow`. An unused patch parameter may be `unused_utcnow`.

### Factories and fixtures

A factory (`make_http(body, status)`) builds a collaborator and takes the values that differ per test. Each test calls it and passes what matters. The article's sketch of a factory returns the wrong variable name; return the object you built.

A pytest fixture is for setup and teardown that is identical across tests (start a server, yield it, stop it; open a client, yield it, delete rows) and for running one test against several implementations via `params`. Do not use a fixture where an argument would do. The reader should see the inputs in the test function.

## Agent Guidance

### Do

- Assert decoded or returned state, not the intermediate string the current implementation happens to pass.
- Construct the real collaborator when it is in-process and free of I/O.
- Subclass the real type for a stub or a fake.
- Autospec every mock. Name it as the collaborator.
- Add `test_constructor` and `test_default_state` for a type with meaningful initial state.
- Pass variation through a factory.

### Do Not

- Do not drop the integration tier because this source folds it into "unit".
- Do not use a bare `Mock()` or a hand-maintained spec list.
- Do not prefix doubles with `mock_` or `fake_`.
- Do not put ordinary inputs in a fixture to avoid passing arguments.
- Do not copy the article's factory return-name slip or its JWT listings.

## Checklist

- [ ] The assertion checks the outcome a caller can observe
- [ ] Call assertions are present only where the call is the contract
- [ ] In-process collaborators are real or faked, not mocked by default
- [ ] Every mock is created with autospec or `create_autospec`
- [ ] Doubles are named as the real collaborator
- [ ] Stubs and fakes subclass the collaborator type when that type exists
- [ ] Varied inputs come from a factory; fixtures are limited to teardown or implementation injection
- [ ] The test's pyramid level is still explicit (unit, integration, or end-to-end)

## See Also

- wiki/tier3-working/python-testing/overview.md
- wiki/tier3-working/python/pytest-patterns.md
- wiki/tier2-core/testing-strategies/test-pyramid.md
- wiki/tier3-working/worked-examples/dependency-injection.md
- wiki/tier2-core/solid-principles/dip.md
- wiki/tier1-sources/swebok-v4/ka05-testing.md
- wiki/tier3-working/checklists/testing-review.md

## Source

Stargirl (Thea) Flowers, "My Python testing style guide", 2017-07-15. https://blog.thea.codes/my-python-testing-style-guide/ . Structured summary: `references/python-testing/thea-flowers-style-guide.md`. © Stargirl Flowers. The author describes the post as personal practice, not dogma.
