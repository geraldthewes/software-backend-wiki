# Mutation Testing

> **Tier 2** | Source: mutmut / mutation testing theory | Derives From: ka05-testing | Authority: established practice

## Summary

Mutation testing tools systematically modify your source code — creating "mutants" — and check whether your test suite detects each change. A mutant that survives (tests still pass) reveals a gap in the test suite: there is production code behavior that no test verifies. Mutation score is a stronger signal of test quality than line coverage.

## Key Concepts

### What Mutation Testing Is

A mutation testing tool:
1. Reads your source code
2. Applies a library of small, syntactically valid modifications ("mutation operators") to create mutant versions
3. Runs your full test suite against each mutant
4. Reports: **killed mutants** (at least one test failed — good) and **surviving mutants** (all tests passed — gap)

The **mutation score** is: `(killed mutants / total mutants) × 100%`

### Why Mutation Testing Is Better Than Line Coverage

80% line coverage can be achieved with tests that make no assertions:

```python
def test_something():
    result = my_function(42)  # called — covered
    # no assertion — what does result contain? nobody knows
```

This test achieves 100% line coverage but kills zero mutants. Mutation testing exposes this.

### Common Mutation Operators

| Operator | Example Mutation | What It Tests |
|----------|-----------------|---------------|
| Arithmetic replacement | `+` → `-` | Arithmetic results are verified |
| Comparison replacement | `>` → `>=` | Boundary conditions are tested |
| Boolean replacement | `True` → `False` | Boolean logic is verified |
| Statement deletion | `result = compute()` removed | Return value is used |
| Conditional negation | `if condition:` → `if not condition:` | Both branches are tested |
| Logical operator replacement | `and` → `or` | Logical combinations are verified |
| Return value replacement | `return value` → `return None` | Return value is asserted |

---

### Python Tool: mutmut

#### Installation

```bash
pip install mutmut
```

#### Running Mutation Tests

```bash
# Run mutations on the entire project
mutmut run

# Run mutations on a specific module only (recommended to start)
mutmut run --paths-to-mutate src/my_service/domain/order.py

# Run with a specific test command
mutmut run --test-time-multiplier 3  # allow 3× normal test time per mutant
```

#### Reviewing Results

```bash
# Show summary: killed, survived, timeout, suspicious
mutmut results

# Show surviving mutants for a file
mutmut results --show-suspicious-count

# Show the diff for a specific mutant by ID
mutmut show 42

# Show all surviving mutants
mutmut show
```

#### Target Mutation Score

| Module Criticality | Target Score |
|-------------------|-------------|
| Business-critical (payments, auth, domain logic) | > 80% |
| Standard application code | > 70% |
| Glue code, serialization adapters | > 60% |
| Configuration, logging, CLI | Accept surviving mutants |

---

### Workflow Integration

Mutation testing is **too slow to run on every commit** (it runs the full test suite N times, where N = number of mutants). Integrate it appropriately:

| Trigger | Stage | Scope |
|---------|-------|-------|
| Every commit | Unit test CI | Not used — too slow |
| Nightly CI | Scheduled pipeline | All critical modules |
| Pre-release | Release gate | All modules with score threshold |
| Code review | On-demand | Changed files only (`--paths-to-mutate`) |

Focus mutation testing on **business-critical modules first**: payment processing, authentication, authorization, data validation, domain rules. Apply it to logging, CLI, and glue code last (if at all).

---

### Interpreting Results

**Surviving mutant on critical logic**: this is a real gap. Write a test that would detect the behavior change.

```bash
$ mutmut show 23
--- src/my_service/domain/pricing.py
+++ src/my_service/domain/pricing.py
@@ -12,7 +12,7 @@
 def apply_discount(price: float, tier: str) -> float:
     if tier == "premium":
-        return price * 0.80
+        return price * 0.81
     return price
```

Surviving mutant: the 20% discount is changed to 19%. No test caught it. Fix: add an exact assertion.

**Surviving mutant on logging/error messages**: usually acceptable. Mutating a log message string is not a meaningful behavioral change.

```bash
$ mutmut show 47
--- src/my_service/infrastructure/repo.py
+++ src/my_service/infrastructure/repo.py
@@ -8,7 +8,7 @@
 except Exception as exc:
-    logger.error("Failed to save order", exc_info=exc)
+    logger.error("XX Failed to save order", exc_info=exc)
```

Surviving mutant: log message changed. Accept this — do not write brittle tests that assert on log message text.

---

### Example: 100% Line Coverage, Surviving Mutants, Then Fixed

**Function under test**:

```python
def calculate_late_fee(days_overdue: int, base_fee: float) -> float:
    if days_overdue > 30:
        return base_fee * 2.0
    return base_fee
```

**Weak test (100% line coverage, poor mutation score)**:

```python
def test_late_fee():
    result = calculate_late_fee(days_overdue=31, base_fee=10.0)
    assert result is not None  # covers the line, kills zero arithmetic mutants
```

Mutant `base_fee * 2.0` → `base_fee * 3.0` survives. Mutant `days_overdue > 30` → `days_overdue >= 30` survives.

**Strong tests (kills the surviving mutants)**:

```python
def test_late_fee_doubled_after_30_days():
    assert calculate_late_fee(days_overdue=31, base_fee=10.0) == 20.0  # kills arithmetic mutant

def test_no_late_fee_at_exactly_30_days():
    assert calculate_late_fee(days_overdue=30, base_fee=10.0) == 10.0  # kills boundary mutant

def test_no_late_fee_before_30_days():
    assert calculate_late_fee(days_overdue=15, base_fee=10.0) == 10.0

def test_late_fee_boundary_day_31():
    assert calculate_late_fee(days_overdue=31, base_fee=50.0) == 100.0  # different base fee
```

---

### Relationship to Line Coverage

Line coverage and mutation score together provide a two-dimensional view of test quality:

| Line Coverage | Mutation Score | Interpretation |
|--------------|---------------|----------------|
| High | High | Strong test suite |
| High | Low | Tests run the code but don't verify behavior — most common gap |
| Low | High | Impossible (coverage is a prerequisite for killing mutants) |
| Low | Low | Severely undertested |

Aim for both metrics: line coverage ensures all paths are exercised; mutation score ensures assertions are meaningful.

## Agent Guidance

### Do
- Run mutmut on business-critical modules before declaring them production-ready
- Aim for >80% mutation score on payment, auth, and domain logic modules
- Use `mutmut run --paths-to-mutate <file>` during code review of changed files
- Schedule nightly mutation test runs in CI for critical modules

### Do Not
- Do not run mutation testing on every commit — it is too slow
- Do not write assertions against log message text to kill logging mutants — those are acceptable survivors
- Do not treat a 70% line coverage score as sufficient without checking the mutation score
- Do not fix surviving mutants by weakening the assertion — strengthen the test

## Checklist
- [ ] mutmut is installed and configured in the project
- [ ] Mutation score for business-critical modules is documented
- [ ] Nightly or pre-release mutation test CI job exists
- [ ] Surviving mutants on critical logic are tracked and fixed
- [ ] Line coverage and mutation score are both reported in CI

## See Also
- `wiki/tier2-core/testing-strategies/property-based-testing.md`
- `wiki/tier2-core/testing-strategies/test-pyramid.md`
- `wiki/tier2-core/testing-strategies/overview.md`
- `wiki/tier1-sources/swebok-v4/ka05-testing.md`

## Source

Richard Lipton, mutation testing theory (1971); mutmut documentation (github.com/boxed/mutmut). Synthesized from *Software Development Best Practices for Agent* reference document.
