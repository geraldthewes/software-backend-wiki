# Code Review Checklist

> **Tier 3** | Enforces: wiki/tier1-sources/swebok-v4/ka04-construction.md, wiki/tier1-sources/swebok-v4/ka12-quality.md, wiki/tier1-sources/python-peps/pep-008-style.md, wiki/tier1-sources/python-peps/pep-484-type-hints.md, wiki/tier2-core/solid-principles/overview.md

## Correctness

- [ ] Logic handles all edge cases: `None`, empty collections, zero, max values, negative numbers
- [ ] Error paths tested and return or raise something meaningful
- [ ] No silent exception swallowing — bare `except: pass` or `except Exception: pass` without logging
- [ ] Recursive functions have a base case and respect Python's recursion limit (default 1000)
- [ ] Off-by-one errors checked in loops, slices, and pagination
- [ ] Concurrent code: no race conditions; proper synchronization via mutex, channel, or atomic
- [ ] All return values from fallible functions are checked (Go: no `_` discarding errors)

## Style (PEP 8 / PEP 484 / Effective Go)

- [ ] Type hints present on all public function signatures (Python)
- [ ] `snake_case` for Python functions/variables; `PascalCase` for classes; `camelCase` for Go unexported names; `PascalCase` for Go exported names
- [ ] No line exceeds 79 characters (Python) or `gofmt`-compliant (Go)
- [ ] Import order correct (stdlib → third-party → local for Python)
- [ ] No unused imports or variables
- [ ] No commented-out code without an explanation comment

## Design (SOLID)

- [ ] Each class/function has a single, clearly stated responsibility (SRP)
- [ ] Dependencies injected via constructor or parameter — not instantiated inside logic (DIP)
- [ ] No `isinstance()` chains that replace polymorphism — use Protocol/interface dispatch (OCP violation)
- [ ] Interfaces / Protocols are small and focused — no more than 5 methods (ISP)
- [ ] Subtypes do not break the parent contract — no `NotImplementedError` for inherited methods (LSP)
- [ ] Cyclomatic complexity ≤ 10 per function (count branches: `if`, `for`, `while`, `except`, `and`, `or`)
- [ ] New abstractions have clear names that explain their purpose without requiring the reader to inspect the implementation

## Security (OWASP / ka13)

- [ ] No hardcoded secrets, credentials, API keys, or private URLs
- [ ] All SQL uses parameterized queries — no string formatting or f-string interpolation in SQL
- [ ] No `shell=True` with any user-derived input (command injection)
- [ ] No `pickle.loads()`, `eval()`, or `exec()` on untrusted external input
- [ ] Log statements free of PII, passwords, session tokens, and API keys
- [ ] All inputs from external sources validated at the system boundary (not assumed safe inside)
- [ ] File paths from user input validated before use (path traversal)

## Testing

- [ ] All public functions have at least one unit test
- [ ] Error paths explicitly tested — not only the happy path
- [ ] Property-based tests used for data transformation, encoding, and parsing logic
- [ ] Tests have descriptive names: `test_raises_ValueError_when_email_is_empty`
- [ ] No test makes real network calls without `@pytest.mark.integration` or equivalent marker
- [ ] Test doubles (fakes, mocks) are at the right level — mock the I/O boundary, not the business logic

## Documentation

- [ ] Complex logic has inline comments explaining WHY a decision was made, not just WHAT the code does
- [ ] Public APIs and non-obvious functions have docstrings
- [ ] README updated if setup steps, configuration, or visible behavior changed
- [ ] Breaking API changes documented in CHANGELOG or migration guide

## See Also

- wiki/tier3-working/checklists/security-review.md
- wiki/tier3-working/checklists/testing-review.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md
- wiki/tier2-core/solid-principles/overview.md

## Source

SWEBOK V4, KA4 (Construction), KA12 (Quality). PEP 8, PEP 484. OWASP Top 10. Effective Go. "Clean Code" (Martin, 2008).
