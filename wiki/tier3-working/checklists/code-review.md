# Code Review Checklist

> **Tier 3** | Enforces: wiki/tier1-sources/swebok-v4/ka04-construction.md, wiki/tier1-sources/swebok-v4/ka12-quality.md, wiki/tier1-sources/python-peps/pep-008-style.md, wiki/tier1-sources/python-peps/pep-484-type-hints.md, wiki/tier2-core/solid-principles/overview.md, wiki/tier2-core/code-review-method/overview.md

Scan invariants first (Correctness > Performance > Complexity > Style). Style findings never block a merge that fails an invariant. Method: wiki/tier2-core/code-review-method/overview.md. Triggers: wiki/tier2-core/code-review-method/triggers.md.

## Invariants and precedence (Linus review method)

- [ ] No fatal abort (`panic`, `sys.exit`, `os.Exit`, `assert` as control flow) on a recoverable, user-reachable condition
- [ ] No public API / ABI / CLI / schema change without a deprecation or versioned migration path
- [ ] New endpoints or exported functions run the same authz and validation as existing paths
- [ ] Every resource (lock, file, connection) is released on all error paths; no cleanup-while-held
- [ ] Copies, format strings, and indexes are bounded; no untrusted data used as a format string
- [ ] Error-return convention is consistent inside the module; no mixed `None` / `-1` / exception / errno contracts
- [ ] Shared mutable state uses atomic/happens-before semantics; lock order is consistent; no read-to-write lock upgrade
- [ ] Special-case branches have a data-structure justification — not a one-off `if is_head`
- [ ] No new abstraction used only once; no copy-paste of a non-trivial algorithm
- [ ] Commit message explains *why*; comments match the code they annotate
- [ ] Every non-approval finding is [REASON] → [ACT] with a named principle and a suggested fix
- [ ] Findings are technically direct; no personal attacks (ACM/IEEE Principle 7)

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

- wiki/tier2-core/code-review-method/overview.md
- wiki/tier2-core/code-review-method/triggers.md
- wiki/tier3-working/checklists/security-review.md
- wiki/tier3-working/checklists/testing-review.md
- wiki/tier3-working/code-review-guidelines/overview.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md
- wiki/tier2-core/solid-principles/overview.md

## Source

SWEBOK V4, KA4 (Construction), KA12 (Quality). PEP 8, PEP 484. OWASP Top 10. Effective Go. "Clean Code" (Martin, 2008). Linus Torvalds review method: wiki/tier2-core/code-review-method/overview.md (Mte90/linus-torvalds-skill, CC0).
