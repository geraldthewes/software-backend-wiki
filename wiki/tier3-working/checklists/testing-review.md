# Testing Review Checklist

> **Tier 3** | Enforces: wiki/tier1-sources/swebok-v4/ka05-testing.md, wiki/tier1-sources/swebok-v4/ka12-quality.md

## Test Pyramid Balance

- [ ] Unit tests are the majority (~70% of test count) — fast, isolated, no I/O
- [ ] Integration tests exist for every external dependency boundary (~20%)
- [ ] End-to-end tests cover only critical user workflows (~10%) — not used as a substitute for unit tests
- [ ] No "testing ice cream cone" — E2E tests are not the primary test mechanism

## Coverage

- [ ] All public functions have at least one unit test
- [ ] All error paths and exception handlers are explicitly tested — not just the happy path
- [ ] Boundary conditions tested: empty input, single item, maximum values, zero, negative numbers, `None`
- [ ] Code coverage measured and tracked in CI; coverage must not decrease on merge

## Property-Based Tests

- [ ] Encoding/decoding logic tested with property-based tests (Hypothesis in Python, rapid in Go)
- [ ] Data transformation functions tested with property-based tests: round-trip properties, invariants
- [ ] Serialization/deserialization code has property test: `decode(encode(x)) == x`
- [ ] Properties defined capture the actual contract of the function, not just the implementation

## Mutation Testing

- [ ] Mutation testing run (`mutmut` for Python) on critical business logic paths
- [ ] Surviving mutations investigated — each survivor indicates a gap in test assertions
- [ ] Mutation score tracked for core domain modules; score should not regress

## Test Quality

- [ ] Test names are descriptive: `test_raises_ValueError_when_input_is_None` not `test_1`
- [ ] Each test verifies one behavior — not a long script testing 10 things in sequence
- [ ] No test makes real network calls without `@pytest.mark.integration` or equivalent marker
- [ ] Test doubles (fakes, stubs, mocks) used at the correct level — mock I/O boundaries, not business logic
- [ ] Test data (fixtures, factories) is self-contained and does not depend on external state
- [ ] Flaky tests (randomly failing) treated as bugs; tracked and fixed immediately — not ignored

## CI Integration

- [ ] All tests run automatically on every push (not just on PRs)
- [ ] Race detector enabled in CI (`go test -race ./...` for Go, thread sanitizer where applicable)
- [ ] Test results reported and retained as CI artifacts
- [ ] Slow tests (> 1 second) tagged and can be excluded from the fast local test run
- [ ] Test suite completes in < 5 minutes for the unit test tier; integration tests may be slower with explicit tagging

## See Also

- wiki/tier3-working/checklists/code-review.md
- wiki/tier1-sources/swebok-v4/ka05-testing.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md

## Source

SWEBOK V4, KA5 (Testing), KA12 (Quality). "Working Effectively with Legacy Code" (Feathers, 2004). Hypothesis documentation (hypothesis.readthedocs.io). mutmut documentation (github.com/boxed/mutmut).
