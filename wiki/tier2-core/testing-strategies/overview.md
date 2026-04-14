# Testing Strategies — Overview

> **Tier 2** | Source: Mike Cohn, Hypothesis, mutmut | Derives From: ka05-testing | Authority: established practice

## Summary

Writing tests is necessary but not sufficient. A testing *strategy* defines what to test, at which level, with which techniques, and in which order of priority. Without a strategy, teams accumulate slow, brittle E2E tests while neglecting fast unit tests — the "ice cream cone" anti-pattern. With a strategy, every defect has a natural home in the test suite and is caught at the fastest and cheapest level possible.

## Key Concepts

The three documents in this section cover complementary techniques:

| Document | Technique | Primary Question It Answers |
|----------|-----------|---------------------------|
| `test-pyramid.md` | Test pyramid (unit / integration / E2E) | How many tests at each level, and what does each level verify? |
| `property-based-testing.md` | Hypothesis library | How do I find edge cases I haven't thought of? |
| `mutation-testing.md` | mutmut | Do my tests actually verify behavior, or do they just achieve line coverage? |

### How They Complement Each Other

The test pyramid provides the structure: most tests should be fast unit tests; few should be slow E2E tests. Property-based testing multiplies the value of unit tests by generating hundreds of inputs automatically — filling the space that example-based tests miss. Mutation testing audits the entire test suite by asking whether the tests would catch intentional bugs. Together, the three techniques build a test suite that is fast, thorough, and genuinely meaningful.

A common failure mode: 90% line coverage achieved with shallow assertions, where mutants survive because tests only check that a function runs, not what it returns. The pyramid tells you where to put tests; property-based testing makes unit tests more thorough; mutation testing validates that the assertions are meaningful.

## Agent Guidance

### Do
- Read `test-pyramid.md` before writing the first test for a new module
- Apply property-based testing (`property-based-testing.md`) for serialization, validation, and data transformation functions
- Run mutation testing (`mutation-testing.md`) on business-critical modules before marking them production-ready

### Do Not
- Do not equate line coverage with test quality — a 95% coverage score with no property tests and surviving mutants is not a strong test suite
- Do not write E2E tests for behavior that can be verified at the unit level
- Do not skip the test pyramid review step when tests are "slow" — slowness is usually a symptom of testing at the wrong level

## Checklist
- [ ] Test pyramid proportions are documented for the module (~70% unit, ~20% integration, ~10% E2E)
- [ ] Property-based tests exist for all serialization, encoding, and validation functions
- [ ] Mutation score is above 80% for business-critical modules
- [ ] Slow tests are marked with pytest marks and excluded from the fast feedback loop

## See Also
- `wiki/tier2-core/testing-strategies/test-pyramid.md`
- `wiki/tier2-core/testing-strategies/property-based-testing.md`
- `wiki/tier2-core/testing-strategies/mutation-testing.md`
- `wiki/tier1-sources/swebok-v4/ka05-testing.md`

## Source

Mike Cohn, *Succeeding with Agile* (2009); Hypothesis documentation; mutmut documentation. Synthesized from *Software Development Best Practices for Agent* reference document.
