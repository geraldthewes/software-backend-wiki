# Test Pyramid

> **Tier 2** | Source: Mike Cohn (2009) | Derives From: ka05-testing | Authority: established practice

## Summary

The Test Pyramid provides a model for balancing test types by cost, speed, and scope. The pyramid shape communicates the recommended proportions: many fast unit tests at the base, fewer integration tests in the middle, and very few slow E2E tests at the top. Inverting this pyramid is the most common testing anti-pattern in production codebases.

## Key Concepts

### Mike Cohn's Original Pyramid (2009)

```
          /────────\
         /  E2E ~10%\
        /────────────\
       / Integration  \
      /     ~20%       \
     /──────────────────\
    /    Unit Tests ~70%  \
   /──────────────────────\
```

### Unit Tests (~70%)

- Test a **single function or class** in complete isolation
- **No I/O**: no database, no network, no filesystem
- **Milliseconds to run**: a suite of 1,000 unit tests should complete in under 10 seconds
- Failures point precisely to the broken unit — no guessing which component failed
- Should make up the majority of tests — hundreds or thousands per service

**What to test**: pure functions, domain logic, data transformations, validation, business rules

### Integration Tests (~20%)

- Test **one module interacting with one real external dependency** (e.g., a repository against a real database)
- External services other than the one under test are still mocked
- Seconds to run; slower than unit tests, faster than E2E
- Catch interface mismatches that unit tests cannot detect (schema mismatch, serialization errors)
- Use pytest fixtures to spin up and tear down database containers

**What to test**: repository implementations against real PostgreSQL, message producers against a real Redis, file parsers against real files on disk

### End-to-End Tests (~10%)

- Test the **full application from the outside** — as a user or external caller would
- All real dependencies (database, cache, message queue) are running
- Minutes to run; brittle; accept a baseline level of flakiness
- Reserve for the **most critical user journeys**: payment flow, authentication, data export
- Failing E2E tests are expensive to diagnose — do not over-invest here

**What to test**: the checkout flow end-to-end, user login and session management, a full data processing pipeline

### The Anti-Pattern: Ice Cream Cone

```
   /──────────────────────\
  /   Manual Testing        \     ← slowest, most expensive
 /────────────────────────────\
/         E2E Tests             \  ← slow, brittle, expensive
/──────────────────────────────────\
/   Integration Tests                \
/──────────────────────────────────────\
              Unit Tests                  ← fewest; fast feedback is lost
```

The ice cream cone inversion occurs when teams rely on manual or E2E tests because they are "easier to write" than unit tests. The result is a test suite that takes hours to run, fails intermittently, and provides no fast feedback during development.

### Why Proportions Matter

| Test Level | Cost Per Test | Feedback Speed | Defect Detection Rate |
|------------|--------------|----------------|----------------------|
| Unit | Very low | < 1 s | High (logic errors) |
| Integration | Medium | 5–30 s | Medium (interface errors) |
| E2E | High | 1–10 min | Low (runtime errors) |

Running unit tests 100 times per day × very low cost = negligible CI cost and instant feedback. Running E2E tests 100 times per day × high cost = significant CI cost and slow feedback loops.

### What to Mock

| Level | What to Mock | What to Use Real |
|-------|-------------|-----------------|
| Unit | All I/O: database, network, filesystem, clock | Only the function/class under test |
| Integration | Other services; only use real for the single dependency under test | The database, filesystem, or queue being tested |
| E2E | Nothing (or use a test environment with all real services) | Everything |

**Do not mock what you don't own**: avoid mocking third-party libraries. Mock the boundary adapter you write around the third-party library instead.

### Test Doubles

| Double | Definition | When to Use |
|--------|------------|-------------|
| **Stub** | Returns predetermined responses | When you need controlled return values |
| **Mock** | Verifies that specific calls were made | When the test is about *interactions*, not return values |
| **Spy** | Records calls for later assertion | When you want to verify calls but keep real behavior |
| **Fake** | Working implementation (e.g., in-memory repository) | When a stub would be too fragile; simplest for DIP-compliant code |
| **Dummy** | Placeholder that is never called | When a parameter is required but not used in the test |

**Prefer fakes over mocks** where DIP is applied correctly: an in-memory `FakeRepository` is more readable and more robust than a `MagicMock` with `side_effect` chains.

### Organizing Tests in Python

```
my_service/
├── src/
│   └── my_service/
│       ├── domain/
│       │   ├── order.py
│       │   └── order_test.py       # co-located unit tests
│       └── infrastructure/
│           └── postgres_repo.py
└── tests/
    ├── integration/
    │   └── test_postgres_repo.py   # integration tests in separate directory
    └── e2e/
        └── test_checkout_flow.py   # E2E tests in separate directory
```

Mark slow tests to exclude from fast feedback:

```python
import pytest

@pytest.mark.integration
def test_repository_saves_order(db_session):
    ...

@pytest.mark.e2e
def test_full_checkout_flow(test_client):
    ...
```

```ini
# pytest.ini
[pytest]
markers =
    integration: marks tests as integration tests (deselect with '-m "not integration"')
    e2e: marks tests as end-to-end tests (deselect with '-m "not e2e"')
```

### Python Examples

```python
# Unit test — no I/O, fast
from my_service.domain.order import calculate_discount

def test_discount_applied_for_premium_customer():
    discount = calculate_discount(base_price=100.0, customer_tier="premium")
    assert discount == 20.0

def test_no_discount_for_standard_customer():
    discount = calculate_discount(base_price=100.0, customer_tier="standard")
    assert discount == 0.0


# Integration test — real database via pytest fixture
import pytest
import psycopg
from my_service.infrastructure.postgres_repo import PostgresOrderRepository

@pytest.fixture
def db_session(tmp_postgres):  # tmp_postgres from pytest-postgresql or similar
    conn = psycopg.connect(tmp_postgres.url)
    yield conn
    conn.close()

@pytest.mark.integration
def test_repository_persists_and_retrieves_order(db_session):
    repo = PostgresOrderRepository(db_session)
    saved = repo.save(customer_id=1, amount=99.99)
    retrieved = repo.get(saved.id)
    assert retrieved.amount == 99.99
```

## Agent Guidance

### Do
- Write unit tests first; add integration tests only for boundary behavior that unit tests cannot cover
- Co-locate unit tests with source files for fast discovery
- Mark integration and E2E tests with pytest marks; run only unit tests in the fast feedback loop
- Use fakes (in-memory implementations) over mocks where DIP is applied

### Do Not
- Do not write an E2E test for behavior verifiable at the unit level
- Do not mock the database in unit tests if the class under test is a repository — test repositories with integration tests against a real database
- Do not aim for 100% E2E coverage — it is impossible to maintain and very expensive

## Checklist
- [ ] Test suite has more unit tests than integration tests, and more integration tests than E2E tests
- [ ] Unit tests run in under 10 seconds for the whole suite
- [ ] Integration and E2E tests are marked and excluded from fast feedback CI stages
- [ ] No I/O occurs in unit tests (no database, no network calls)
- [ ] Test doubles are documented — fakes preferred over complex mock chains

## See Also
- `wiki/tier2-core/testing-strategies/overview.md`
- `wiki/tier2-core/testing-strategies/property-based-testing.md`
- `wiki/tier2-core/testing-strategies/mutation-testing.md`
- `wiki/tier1-sources/swebok-v4/ka05-testing.md`

## Source

Mike Cohn, *Succeeding with Agile* (2009). Martin Fowler, "TestPyramid" (martinfowler.com). Synthesized from *Software Development Best Practices for Agent* reference document.
