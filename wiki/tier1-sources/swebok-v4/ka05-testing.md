# SWEBOK V4 — KA 05: Software Testing

> **Tier 1** | Source: IEEE SWEBOK V4, KA 05; Hypothesis docs; mutmut docs | Authority: immutable

## Summary

Software Testing is the disciplined process of evaluating a software system or its components to determine whether they satisfy specified requirements and to identify defects. In SWEBOK V4, Testing (KA 05) encompasses everything from unit-level verification through system-level acceptance validation, including the design of test cases, execution, reporting, and continuous integration into the delivery pipeline.

For a coding agent, Testing is non-negotiable: untested code is unfinished code. The agent must understand not just how to write tests but how to structure an entire testing strategy — choosing the right test level for each concern, applying advanced techniques like property-based and mutation testing, and integrating tests into CI/CD pipelines. SWEBOK V4 fully integrates Agile testing practices (TDD, continuous testing) and DevOps patterns (shift-left testing, automated pipelines) as mandatory, not optional, components of the testing KA.

## Key Concepts

### Testing Objectives: Verification vs. Validation

These two terms are frequently confused. The distinction is fundamental:

| Term | Question Answered | Focus | Example |
|------|------------------|-------|---------|
| **Verification** | "Did we build the system right?" | Conformance to specifications | Unit test confirms a function returns the expected value per its spec |
| **Validation** | "Did we build the right system?" | Fitness for user purpose | Acceptance test confirms the user can complete a purchase workflow |

Both are required. Verification without validation produces technically correct software that solves the wrong problem. Validation without verification finds system failures too late in the cycle when correction cost is high.

### Test Levels

Test levels are organized by the scope of what is being tested. Each level has a distinct purpose and is not a substitute for the others.

#### Unit Testing (Level 1)
- **Scope:** A single function, method, or class in isolation
- **Dependencies:** All external dependencies are mocked or stubbed
- **Speed:** Milliseconds per test; thousands run in seconds
- **Defect detection:** Logic errors, edge cases, boundary violations
- **Who writes:** The developer writing the code (often before, via TDD)
- **Python tooling:** pytest, unittest.mock, hypothesis
- **Target coverage:** Statement coverage >90%, branch coverage >80%

#### Integration Testing (Level 2)
- **Scope:** Interaction between two or more modules, or between a module and an external system (database, API, queue)
- **Dependencies:** Real or containerized external systems (Docker Compose, testcontainers)
- **Speed:** Seconds per test
- **Defect detection:** Interface mismatches, data contract violations, schema drift
- **Python tooling:** pytest with fixtures, testcontainers-python, httpx for async HTTP

#### System Testing (Level 3)
- **Scope:** The entire application behaving as a deployed system
- **Dependencies:** Staging environment with production-like data and configuration
- **Speed:** Minutes per test suite
- **Defect detection:** End-to-end workflow failures, configuration errors, performance regressions

#### Acceptance Testing (Level 4)
- **Scope:** Validation that the system satisfies user or business requirements
- **Variants:** User Acceptance Testing (UAT), operational acceptance (readiness), regulatory acceptance
- **Defect detection:** Requirements mismatches, usability failures, missing business rules
- **Python tooling:** Behave (BDD/Gherkin), Robot Framework

### Test Types

| Type | What It Tests | Techniques |
|------|--------------|------------|
| **Functional** | Correct behavior given inputs | Equivalence partitioning, boundary value analysis |
| **Structural (coverage-based)** | Code paths exercised | Statement, branch, path, MC/DC coverage |
| **Change-related (regression)** | No new defects from a change | Re-running existing test suite; mutation testing |
| **Non-functional** | Performance, security, usability | Load testing (Locust), SAST/DAST, accessibility audits |

### Test Techniques

#### Equivalence Partitioning
Divide the input domain into partitions where all members of a partition are expected to be treated the same. Test one representative from each partition rather than every possible value. Example: for an age field accepting 0–120, partitions are: negative (invalid), 0–120 (valid), >120 (invalid). Three tests cover hundreds of thousands of values.

#### Boundary Value Analysis
Defects cluster at the edges of equivalence partitions. Test the exact boundary values and one step on either side. For the age field: test -1, 0, 1, 119, 120, 121.

#### Decision Table Testing
Enumerate all combinations of conditions and the expected action for each combination. Used when multiple boolean conditions interact. Each column in the table is a test case.

#### State Transition Testing
Model the system as a finite state machine. Test transitions between states, including invalid transitions. Particularly important for workflow systems, session management, and protocol implementations.

#### Use Case Testing
Derive tests from use case scenarios. Each use case generates at least: the main success scenario, all alternate flows, and all exception flows.

### Test Activities (IEEE 29119)

1. **Test Planning:** Define scope, strategy, resources, schedule, and entry/exit criteria
2. **Test Monitoring and Control:** Track progress against the plan; escalate when exit criteria will not be met
3. **Test Design:** Derive test conditions and test cases from the test basis (requirements, code, architecture)
4. **Test Implementation:** Create test procedures, scripts, and data; set up the test environment
5. **Test Execution:** Run tests; log actual vs. expected results; report defects
6. **Test Completion:** Evaluate exit criteria; produce a summary report; archive test artifacts

---

## The Test Pyramid

The Test Pyramid is the most important structural concept in practical testing strategy. It dictates the *proportion* of tests at each level for a healthy, sustainable test suite.

```
        /\
       /  \
      / E2E\      ~10%  (slow, brittle, expensive)
     /------\
    /        \
   /Integration\  ~20%  (moderate speed, real I/O)
  /------------\
 /              \
/   Unit Tests   \  ~70%  (fast, isolated, cheap)
/----------------\
```

### Why This Shape Matters

- **Unit tests are cheap:** They run in milliseconds, require no environment setup, and pinpoint failures to a single function. Maximizing their proportion maximizes feedback speed.
- **Integration tests are moderately costly:** They require environment setup (database, external service) and are slower. They should cover interface contracts and I/O behavior, not business logic that units already cover.
- **E2E tests are expensive:** They are slow (minutes), brittle (any infrastructure blip fails them), and when they fail they give broad, imprecise failure signals. Reserve them for the most critical user workflows only.

### The Anti-Pattern: Ice Cream Cone (Inverted Pyramid)

```
/----------------\
/   Manual Tests  \  large
/------------------\
/    E2E / UI Tests  \  large
/--------------------\
/  Integration Tests  \  moderate
/---------------------\
/       Unit Tests      \  tiny
```

Teams with an Ice Cream Cone suffer:
- Slow feedback loops (hours to know if a commit broke something)
- High maintenance burden (UI tests break on every front-end change)
- Low defect detection rate (manual tests miss regressions)
- Developer reluctance to refactor (fear of breaking fragile E2E tests)

The cure is always the same: write more unit tests, reduce manual tests, and surgically select E2E tests for critical paths only.

### Test Pyramid in Numbers

| Level | Target % | Max Run Time (CI) | Failure Signal Precision |
|-------|----------|-------------------|--------------------------|
| Unit | ~70% | <60 seconds total | Single function |
| Integration | ~20% | <5 minutes total | Module interface or I/O layer |
| E2E | ~10% | <15 minutes total | User workflow (broad) |

---

## Test-Driven Development (TDD)

TDD is a construction technique, not just a testing technique. It produces design benefits that cannot be achieved by writing tests after the fact.

### The Red-Green-Refactor Cycle

```
RED:     Write a failing test for the behavior you want to add.
          (The test must fail for the right reason — not compilation error.)

GREEN:   Write the minimum code necessary to make the test pass.
          (Resist the urge to add anything not needed by the test.)

REFACTOR: Clean up the code (remove duplication, improve names, simplify logic)
           without changing its behavior.
           (The tests protect you — they must still pass after refactoring.)

Repeat.
```

### TDD Benefits for Design

1. **Forces interface-first thinking:** Writing the test before the code means designing the API from the caller's perspective, which produces cleaner, more usable interfaces.
2. **Prevents over-engineering:** The "minimum code to pass" rule stops agents from building speculative features.
3. **Creates a living specification:** The test suite documents exactly what the code is supposed to do, in executable form.
4. **Enables fearless refactoring:** A comprehensive test suite from TDD provides a safety net that allows structural improvement without regression risk.
5. **Catches design smells early:** If a unit test requires excessive setup to reach the code under test, the code is probably doing too much — a design signal, not just a testing inconvenience.

### TDD in Agile and DevOps

SWEBOK V4 mandates that TDD is treated as a standard construction practice in Agile iterations. In a DevOps context, TDD feeds the CI pipeline directly: every commit is accompanied by tests that were written before (or alongside) the code.

---

## Property-Based Testing (Hypothesis)

### What It Is

Property-based testing inverts the traditional test structure. Instead of specifying concrete input/output examples, you define a *property* — an invariant that must hold for all valid inputs — and the framework generates hundreds of random inputs to try to falsify it.

The Python library **Hypothesis** is the standard tool. It is empirically ~50x more effective at finding mutations than equivalent example-based tests.

### The `@given` Decorator

```python
from hypothesis import given, settings
from hypothesis import strategies as st

# Traditional example-based test:
def test_reverse_example():
    assert list(reversed([1, 2, 3])) == [3, 2, 1]

# Property-based test:
@given(st.lists(st.integers()))
def test_reverse_involution(lst):
    """Reversing a list twice returns the original list."""
    assert list(reversed(list(reversed(lst)))) == lst
```

The `@given` decorator instructs Hypothesis to generate many values matching the strategy (`st.lists(st.integers())`) and run the test body for each. If any generated value falsifies the property, Hypothesis reports the failure.

### Core Strategies

| Strategy | Generates | Notes |
|----------|-----------|-------|
| `st.integers()` | Arbitrary integers | Includes 0, negatives, MAX_INT |
| `st.integers(min_value=0, max_value=100)` | Bounded integers | For domain-constrained inputs |
| `st.text()` | Unicode strings | Includes empty string, control chars, emoji |
| `st.text(alphabet=st.characters(whitelist_categories=('Ll',)))` | Lowercase letters only | Constrained alphabet |
| `st.lists(strategy)` | Lists of elements from strategy | Includes empty list |
| `st.lists(st.integers(), min_size=1)` | Non-empty lists | Minimum length constraint |
| `st.dictionaries(keys, values)` | Dicts with typed keys/values | For JSON-like payloads |
| `st.one_of(st.integers(), st.text())` | Union of types | For polymorphic inputs |
| `st.fixed_dictionaries({"key": st.integers()})` | Structured dicts | For typed schemas |
| `st.builds(MyClass, field=st.integers())` | Instances of a class | For domain objects |

### Defining Invariants, Not Examples

The key shift in thinking: you are not describing what specific output to expect; you are describing *relationships that must always hold*. The framework handles finding the inputs.

**Property Identification Guide:**

| Property Pattern | Description | Example |
|-----------------|-------------|---------|
| **Roundtrip** | `decode(encode(x)) == x` | Serialization/deserialization |
| **Idempotency** | `f(f(x)) == f(x)` | Normalization, formatting, deduplication |
| **Commutativity** | `f(a, b) == f(b, a)` | Addition, set union |
| **Associativity** | `f(f(a,b),c) == f(a,f(b,c))` | String concatenation, list append |
| **Invariant under transformation** | Property holds before and after | Sorting preserves length and elements |
| **Boundary invariant** | Edge inputs don't crash | Empty list, zero, empty string are handled |
| **Monotonicity** | If `a <= b` then `f(a) <= f(b)` | Scoring functions, ranking |
| **Conservation** | Aggregate unchanged by operation | Sum preserved by shuffle |

### Shrinking: Minimal Failing Examples

When Hypothesis finds a failing input, it automatically *shrinks* it — reducing it to the smallest, simplest version that still triggers the failure. This is one of Hypothesis's most powerful features: instead of reporting a 1,000-character string as the counterexample, it will report a 3-character string. This makes debugging dramatically faster.

### Property-Based Testing Example: Serialization

```python
from hypothesis import given
from hypothesis import strategies as st
import json

@given(st.dictionaries(
    keys=st.text(min_size=1),
    values=st.one_of(st.integers(), st.text(), st.booleans(), st.none())
))
def test_json_roundtrip(data):
    """JSON serialization and deserialization must be lossless."""
    serialized = json.dumps(data)
    deserialized = json.loads(serialized)
    assert deserialized == data
```

---

## Mutation Testing (mutmut)

### How It Works

Mutation testing evaluates the quality of a test suite itself. It asks: "If the code were subtly wrong, would any test catch it?"

The tool (mutmut for Python) performs these steps:
1. **Parse** the source code into an AST
2. **Apply mutations** — small syntactic changes that introduce bugs:
   - `>` becomes `>=` (off-by-one)
   - `+` becomes `-` (arithmetic inversion)
   - `True` becomes `False` (boolean flip)
   - `and` becomes `or` (logical inversion)
   - `return x` becomes `return None`
   - String constants changed
3. **Run the full test suite** against each mutated version
4. **Classify the mutation:**
   - **Killed:** At least one test failed → the tests caught the bug → good
   - **Survived:** All tests passed → the tests did not catch the bug → gap identified

### Mutation Score

```
Mutation Score = (Killed Mutations / Total Mutations) × 100%
```

**Target: >80% mutation score**

A mutation score below 80% means more than 20% of single-point bugs would go undetected by the test suite. High code coverage (e.g., 90% line coverage) does not guarantee a high mutation score — a test can execute a line without asserting anything about its output.

### mutmut Workflow

```bash
# Step 1: Run mutation testing
mutmut run

# Step 2: Review results
mutmut results

# Step 3: See the specific surviving mutations
mutmut show <id>

# Step 4: Fix gaps — write tests that kill the surviving mutations
# Step 5: Re-run to confirm
mutmut run --rerun-all
```

### Relationship Between Coverage and Mutation Score

| Coverage | Mutation Score | Interpretation |
|----------|---------------|----------------|
| Low | Low | Tests are inadequate — cover more paths and assert outcomes |
| High | Low | Tests execute code but don't assert results — "dead assertions" |
| Low | High | Tests are strong but incomplete — add more paths |
| High | High | Ideal: comprehensive tests that verify behavior |

Coverage is a necessary but not sufficient condition for quality. Mutation score measures whether assertions are meaningful.

### CI/CD Scheduling for Mutation Testing

Mutation testing is computationally expensive (it runs the full suite N times, where N = number of mutations). Schedule it appropriately:
- **Every commit:** Unit tests only
- **Every PR:** Unit + integration tests
- **Nightly:** Mutation testing (mutmut) — results inform next day's test improvements
- **Pre-release:** E2E tests + security scans

---

## Testing in CI/CD

SWEBOK V4 mandates that testing is fully integrated into the continuous delivery pipeline. "Shift-left" means moving testing as early as possible in the development cycle — ideally before the code is written (TDD).

### Pipeline Structure

```
Commit pushed
    │
    ▼
Pre-commit hooks
  - ruff (lint)
  - mypy (type check)
  - bandit (security scan)
    │
    ▼
Unit tests (pytest)
  - Must pass before PR can be opened
  - Target: <60 seconds
    │
    ▼
PR opened → Integration tests
  - Containerized dependencies (Docker Compose / testcontainers)
  - Contract tests (Pact or equivalent)
  - Target: <5 minutes
    │
    ▼
Merge to main → System tests
  - Staging environment
  - Target: <15 minutes
    │
    ▼
Nightly → Mutation tests (mutmut)
  - Target mutation score >80%
    │
    ▼
Pre-release → E2E tests + DAST
  - Critical user workflows only
  - OWASP ZAP / Burp Suite
```

### Independence of Testing

SWEBOK V4 emphasizes that testers (or test-writing agents) must maintain independence from the authors of the code being tested. When the same entity writes both the implementation and the sole tests, blind spots propagate — the same misunderstanding of requirements shows up in both. In practice:
- TDD provides structural independence (the test is written first, before the implementation bias forms)
- Code review of tests by someone other than the author provides social independence
- Mutation testing provides mechanical independence (it does not rely on human judgment)

---

## Mocking Principles

Mocks isolate units under test from their dependencies. Used incorrectly, they produce tests that pass against code that is fundamentally broken.

### Correct Mocking Boundaries

- **Mock at I/O boundaries:** File system, database, network calls, external APIs, time (`datetime.now()`)
- **Do not mock:** Pure functions, domain logic, value objects
- **Do not mock what you do not own:** Mock your own gateway/adapter, not the third-party library internals

### Mock Best Practices in Python

```python
from unittest.mock import patch, MagicMock
import pytest

# Good: mock at the boundary (the I/O call), not deep inside
@patch("myapp.db.connection.execute")
def test_user_lookup(mock_execute):
    mock_execute.return_value = [{"id": 1, "name": "Alice"}]
    result = get_user(1)
    assert result.name == "Alice"
    mock_execute.assert_called_once_with("SELECT ...", (1,))  # assert the call too

# Bad: mocking so many things the test proves nothing
# (If every dependency is mocked, you're only testing the mock wiring)
```

### Determinism Requirements

- Tests must produce identical results every run (no flakiness)
- Mock `datetime.utcnow()` and `random` in tests that depend on time or randomness
- Reset mocks between tests (pytest-mock's `mocker` fixture does this automatically)
- Never share mutable state between tests; use fixtures with proper scope

---

## Test Naming Conventions

Test names are the primary documentation of system behavior. They must describe what happens, not just which function is called.

| Pattern | Bad Name | Good Name |
|---------|----------|-----------|
| Describe behavior | `test_validate` | `test_raises_ValueError_when_email_missing_at_sign` |
| Include precondition | `test_process_payment` | `test_process_payment_returns_declined_when_card_expired` |
| Include expected outcome | `test_user_creation` | `test_create_user_persists_record_and_sends_welcome_email` |
| Parameterized | `test_ages` | `test_age_valid_for_[0, 18, 120]` |

### The AAA Pattern

Every test body should follow Arrange-Act-Assert:

```python
def test_calculate_discount_applies_20_percent_for_premium_users():
    # Arrange
    user = User(tier="premium")
    cart = Cart(items=[Item(price=100)])

    # Act
    discounted_total = calculate_discount(user, cart)

    # Assert
    assert discounted_total == 80.0
```

---

## V4 Integration: Agile and DevOps

**Agile integration in KA 05:**
- TDD is the standard construction companion to Agile iterations
- Tests are part of the Definition of Done — a story is not complete without passing tests
- Acceptance criteria are written as executable tests (BDD/Gherkin or pytest scenarios)
- Continuous testing replaces "test phase" — testing happens throughout the sprint

**DevOps integration in KA 05:**
- Shift-left: every commit triggers tests; defects are found in minutes, not weeks
- Test environments are provisioned as code (Docker Compose, Kubernetes manifests)
- Test results feed observability dashboards (flakiness rate, coverage trends)
- Security tests (SAST, dependency audit) are part of the same pipeline as functional tests

---

## Defect Detection Cost Curve

Research consistently shows that the cost to fix a defect increases by an order of magnitude for each phase it survives past its origin:

| Phase Defect Found | Relative Fix Cost |
|-------------------|------------------|
| Requirements | 1× |
| Design | 5× |
| Construction (code review) | 10× |
| Unit test | 15× |
| Integration test | 50× |
| System test | 100× |
| Production | 200–1000× |

This curve is the economic justification for TDD, shift-left testing, and the Test Pyramid. Every defect caught by a unit test costs ~15x less to fix than one caught in production.

---

## Agent Guidance

### Do
- Follow the Test Pyramid: aim for ~70% unit, ~20% integration, ~10% E2E by test count
- Write tests before or alongside code (TDD red-green-refactor cycle)
- Use Hypothesis `@given` with appropriate strategies for any function with a non-trivial input domain
- Run `mutmut run` on new modules; target mutation score >80% before marking a module complete
- Name tests to describe behavior: `test_raises_ValueError_when_input_is_None`, not `test_function`
- Follow AAA (Arrange-Act-Assert) structure in every test body
- Mock only at I/O boundaries (database, network, filesystem, time)
- Run unit tests on every commit, integration tests on every PR, mutation tests nightly, E2E pre-release
- Write at least one property for every serialization/deserialization, normalization, or sorting function
- Use `pytest.mark.parametrize` to test boundary values and equivalence partitions systematically
- Assert the mock was called correctly (not just that the return value is right) when testing I/O paths
- Treat test coverage as a floor, not a ceiling; mutation score is the real quality signal

### Do Not
- Write tests after the code is "done" without TDD — it produces weaker tests with lower mutation scores
- Create an Ice Cream Cone (more E2E tests than unit tests) — this destroys CI speed and reliability
- Use `@patch` to mock internal implementation details — mock at the boundary, not inside the unit
- Write a test that passes if the function runs without asserting any specific output — this is worse than no test
- Skip integration tests and test only units — unit tests cannot catch schema drift or API contract violations
- Allow flaky tests to persist in CI — investigate and fix or delete them; flakiness destroys trust in the suite
- Use `time.sleep` in tests — mock time instead
- Share mutable state between test functions — always use fixtures for setup/teardown
- Write tests that require a specific order to pass — each test must be fully independent
- Accept a mutation score below 80% as "good enough" — identify and kill surviving mutations

## Checklist
- [ ] Test pyramid proportions verified: ~70% unit, ~20% integration, ~10% E2E
- [ ] All new functions have unit tests with TDD (test written before or with implementation)
- [ ] Equivalence partitions and boundary values tested for all input-processing functions
- [ ] At least one Hypothesis property test written for serialization, normalization, or sorting functions
- [ ] `mutmut run` executed on new modules; mutation score >80%
- [ ] Surviving mutations reviewed and additional tests written to kill them
- [ ] All mocks placed at I/O boundaries (database, network, filesystem, time)
- [ ] Tests reset state between runs; no shared mutable fixtures
- [ ] Test names describe behavior (what happens when what condition exists)
- [ ] AAA (Arrange-Act-Assert) structure used in all test bodies
- [ ] Unit tests run in CI on every commit with <60 second total runtime
- [ ] Integration tests run on every PR against containerized dependencies
- [ ] No flaky tests in CI; any flaky test quarantined and fixed
- [ ] E2E tests limited to critical user workflows; not used for logic verification
- [ ] Test completion report reviewed: coverage trends, mutation score, flakiness rate

## See Also
- wiki/tier1-sources/swebok-v4/overview.md
- wiki/tier1-sources/swebok-v4/ka04-construction.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md
- wiki/tier1-sources/swebok-v4/ka13-security.md
- wiki/tier1-sources/swebok-v4/ka02-architecture.md
- wiki/tier1-sources/swebok-v4/ka03-design.md
- wiki/tier2-core/testing-strategies/test-pyramid.md
- wiki/tier2-core/testing-strategies/property-based-testing.md
- wiki/tier2-core/testing-strategies/mutation-testing.md
- wiki/tier3-working/python/pytest-patterns.md
- wiki/tier3-working/checklists/definition-of-done.md

## Source

IEEE Computer Society. *Guide to the Software Engineering Body of Knowledge (SWEBOK), Version 4.0*, KA 05: Software Testing. IEEE Press, 2024.

Hypothesis Documentation. https://hypothesis.readthedocs.io/

mutmut Documentation. https://mutmut.readthedocs.io/

Papadakis, M. et al. "An Empirical Evaluation of Property-Based Testing in Python." OOPSLA 2025. https://cseweb.ucsd.edu/~mcoblenz/assets/pdf/OOPSLA_2025_PBT.pdf
