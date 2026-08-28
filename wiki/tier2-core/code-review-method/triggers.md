# Linus Torvalds Review Triggers (Tier 2)

> **Tier 2** | Source: Mte90/linus-torvalds-skill SKILL.md | Authority: established | Companion: wiki/tier2-core/code-review-method/overview.md

## Summary

This page is the operational trigger catalog for the Linus Torvalds review method. Triggers are language-agnostic patterns organized the way a reviewer actually scans a change: fatal invariants first, then structural concerns, then nits. Each trigger has a type (`invariant-false` or `general-guideline`), a detection cue, a default severity, and a language-neutral translation for Python and Go.

Use this catalog as the first pass of a review. Then apply `wiki/tier3-working/checklists/code-review.md` and `wiki/tier3-working/checklists/security-review.md`. A missing trigger is not a clean bill of health.

## Key Concepts

### Severity Scale

| Severity | Meaning | Merge? |
|----------|---------|--------|
| **reject** | The change is fundamentally wrong or unsafe as submitted | No. Rewrite or drop it. |
| **request-changes** | The approach may be salvageable; named defects must be fixed | No, until the defects are addressed. |
| **nitpick** | Cosmetic or local readability; does not hide a defect | Yes, optionally with a follow-up. |
| **discussion** | Subjective design debate; no clear rule violation | Yes, unless discussion reveals an invariant. |
| **approve** | Adds value, passes checks, no objections | Yes. |

### Corpus Calibration (38,303 moves)

Use these rates to calibrate *how hard* to lean on a category, not as a quota.

| Category | n | Dominant severity | Agent reading |
|----------|---|-------------------|---------------|
| API / ABI stability | 2,115 | reject 37.9% | Public-contract breaks are non-negotiable |
| Correctness | 10,580 | request-changes 47.7% (reject 28.7%) | Fix bugs; reject unrecoverable or security-critical ones |
| Memory safety | 453 | reject 28.3% | Unsafe memory / ownership bugs are blockers |
| Concurrency | 2,044 | reject 22.3% | Deadlock and data races: reject; subtler races: request-changes |
| Complexity | 1,935 | reject 26.4% / request-changes | Unjustified complexity is taken seriously |
| Performance | 4,307 | request-changes 38.1% | Slower-but-correct is usually fixable, not a reject |
| Error handling | 845 | request-changes | Missing checks: request-changes; crash-on-user-input: reject |
| Style | 2,565 | nitpick (reject only 12.6%) | Style almost never blocks unless it hides a defect |

Overall mix: reject 23.8% · request-changes 42.2% · discussion 20.2% · approve 7.0% · nitpick 6.8%.

### Severity Decision Tree

Walk top-down. Stop at the first yes.

1. Does the change break a public contract (API, ABI, CLI, schema, security check) without a migration path? → **reject**
2. Does it introduce a crash, data corruption, race, out-of-bounds, or use-after-free? → **reject**
3. Is a recoverable error currently aborting the process or being swallowed? → **request-changes** (reject if user-reachable abort)
4. Is it a performance regression with no correctness impact? → **request-changes**
5. Is it an unnecessary abstraction, duplicated algorithm, or undocumented magic number? → **request-changes**
6. Is it style or naming only? → **nitpick**
7. Is it a subjective design debate with no rule violation? → **discussion**
8. Otherwise → **approve**

## Level 1 — Global Invariants (non-negotiable)

Invariant-false rules are absolute blockers. Scan these before anything else.

| Trigger | What to look for | Why | Default severity | Python / Go translation |
|---------|------------------|-----|------------------|-------------------------|
| Fatal abort for a recoverable error | `panic()`, `BUG_ON()`, `fatal_error()`, `sys.exit`, `os.Exit`, `raise SystemExit`, `assert` used as control flow on user-reachable paths | Recoverable conditions must return an error value; crashing the process is a correctness and availability bug | request-changes (reject if reachable from untrusted input) | Replace with `return err`, `raise ValueError`/`HTTPException`, log at warning, do not abort the process |
| Public API/ABI change without a migration path | Changed signatures, struct/protobuf layouts, exported constants, REST paths, CLI flags used outside the module, no deprecation | Existing users silently break; the contract is a correctness guarantee | reject | Add deprecation window, version bump (`BREAKING CHANGE`), or adapter; never rename a shipped function "because it's cleaner" |
| New interface that bypasses existing security checks | New endpoint, syscall-equivalent, public function, or admin path that skips authz/validation the old path performed | Security checks are part of the functional contract | reject | Route the new path through the same authn/authz and validation as the existing one |
| Control-flow jump that skips cleanup while holding a resource | `goto err` with a lock held; `return` inside `with`/`defer` that skips unlock; swallowed exception that skips `close()` | Deadlock, leak, undefined behavior | reject | Unlock/close on every path; use `defer`, context managers, or `try/finally` |
| Unbounded format-string or buffer-size mismatch | `snprintf` with a wrong size; `memcpy` without NUL; `%s` with untrusted format; f-string into SQL | Memory corruption or injection | reject | Bound every copy; never pass untrusted data as a format string; parameterized queries |

**Precedence rule encoded here:** Correctness > Performance > Complexity > Style. A fast but buggy change is useless.

## Level 2 — Structural Patterns

Serious design concerns. Severity is request-changes or reject depending on impact.

### A. Data-structure and special-case elimination

| Trigger | Detection | Why | Severity |
|---------|-----------|-----|----------|
| Special-case handling for a single value (`if is_head`, `if i == 0`) | Branch exists only because one element is modelled differently | Symptom of a poor data model; a pointer-to-pointer / sentinel / uniform node often deletes the branch | request-changes |
| Duplicated logic that could be a helper | Identical statement sequences, especially error handling, in two functions | One copy gets the bugfix; the other does not | nitpick if trivial; request-changes if the duplicated block is non-trivial |
| Hard-coded magic constants | Bare `12 * 1024**3`, `256`, `42` as sizes/limits with no name | Future readers cannot tell protocol limit from accident | request-changes |

### B. Duplication and reuse

| Trigger | Detection | Why | Severity |
|---------|-----------|-----|----------|
| Copy-paste of a complex algorithm across files | Same algorithmic steps in unrelated modules | Divergent bug fixes | request-changes |
| Re-implementing an existing helper | New timestamp / path / retry / parse logic that a well-tested utility already provides | Reinvented code is more likely to be wrong | request-changes |
| Exposing internal types as public interface | Public headers / `__all__` / exported Go types that leak internal structs | Ties callers to internal layout; future refactors become ABI breaks | request-changes (invariant-false) |

### C. Error-handling and return conventions

| Trigger | Detection | Why | Severity |
|---------|-----------|-----|----------|
| Mixed error-code conventions in one module | Some functions return `-1`, others `None`, others set a global, others raise | Callers cannot remember the contract; missed checks follow | reject |
| New error code / exception type without documentation | New sentinel or exception subclass with no docstring, changelog, or caller update | Callers cannot handle the new case | request-changes |
| Fatal assertion on expected failure | `assert user_id is not None` on a request path; `panic(err)` after a parse of external data | Turns a recoverable error into a crash | request-changes (reject if untrusted input) |

### D. Concurrency and synchronization

| Trigger | Detection | Why | Severity |
|---------|-----------|-----|----------|
| Inconsistent lock order (AB-BA) | Two paths acquire locks in opposite order | Deadlock | reject |
| Recursive lock acquisition | Function holds lock L, then calls another function that takes L, and L is not re-entrant | Self-deadlock | reject |
| Shared flag read/written without atomic or happens-before | Plain bool/int shared across goroutines/threads | Torn reads, lost updates, compiler/CPU reordering | reject |
| Lock upgrade (read → write) | Take shared lock, then try to upgrade to exclusive | Guarantees deadlock under contention | reject |
| Lock held across a blocking / user / network call | Mutex held while awaiting I/O | Latency and deadlock | request-changes |

### E. Memory safety and ownership

| Trigger | Detection | Why | Severity |
|---------|-----------|-----|----------|
| Missing ownership / refcount on a shared object | Freed (or closed) in two places; no single owner | Double-free, use-after-close | reject |
| Allocation without size validation | `malloc(n)`, `make([]T, n)`, `bytes(n)` where `n` is user-derived or can overflow | Zero-size or wraparound allocations | reject |
| Returning a pointer/reference to a stack-allocated / gone object | Return `&local`; return slice into a buffer that is then reused; Python returning a closed file | Dangling reference | reject |
| Allocation without a matching free/close on every error path | Resource acquired, error returned, resource not released | Leak | request-changes |

### F. Security validation

| Trigger | Detection | Why | Severity |
|---------|-----------|-----|----------|
| Skipping input validation on a trust-boundary crossing | Copy from request/socket/file/CLI without length or type checks | Overflow, injection, privilege escalation | reject |
| Insecure copy/format in code that claims to be hardened | `strlcpy`, unbounded `strcpy`, f-string SQL, `shell=True` | Truncation or injection despite "hardening" | reject |
| Exposing internal data structures across a trust boundary | Kernel struct in a public header; ORM object dumped as JSON; stack traces to clients | Information leak and coupling | request-changes |
| New public surface that skips existing permission checks | See Level 1 | Same | reject |

Also apply `wiki/tier3-working/owasp-code-review/overview.md` and `wiki/tier3-working/checklists/security-review.md`. Security findings that have no row here are still blockers.

### G. Complexity and unnecessary abstractions

| Trigger | Detection | Why | Severity |
|---------|-----------|-----|----------|
| New abstraction used only once | Helper, type, or interface with a single call site and no second planned caller | Cognitive load with no reuse | request-changes |
| Configuration knob almost nobody needs | New compile/runtime flag for a niche path | Hidden incompatibilities, `#ifdef` bug surface | request-changes |
| Special-casing a rarely used path | `if booting`, `if debug`, `if customer_x` sprinkled through core logic | Makes the normal path unreadable | request-changes |

### H. Documentation and commit messages

| Trigger | Detection | Why | Severity |
|---------|-----------|-----|----------|
| Commit message missing rationale (large or behavioral change) | One-line subject, empty body, no *why* | Reviewers cannot assess impact; bisect later has no context | request-changes |
| Comment that does not match the code | Comment describes a state the code never reaches | Misleading comments hide bugs | request-changes |
| Link/URL standing in for a description | Body is only `See JIRA-1234` or a GitHub URL | Commit must be self-contained for offline/`git log` review | request-changes |

Format of the message is Conventional Commits (`wiki/tier2-core/conventional-commits/overview.md`). Substance — the *why* — is this trigger.

### I. Performance-sensitive hot paths

Only after correctness. Default severity is lower.

| Trigger | Detection | Why | Severity |
|---------|-----------|-----|----------|
| Virtual / interface call inside a tight inner loop | Per-row, per-packet, per-byte dispatch through an interface | Indirect calls prevent inlining; cost must be justified | nitpick (request-changes if measured) |
| Wrapper that only forwards arguments | Extra function with no added logic on a hot path | Call overhead and noise | request-changes |
| Heavyweight instruction / dependency for a trivial operation | SIMD, regex, JSON round-trip, ORM load for a scalar check | Size, portability, and often slower | nitpick |

### J. Process and governance

| Trigger | Detection | Why | Severity |
|---------|-----------|-----|----------|
| Out-of-tree / one-off consumer dictating core changes | "We need this in core because our private plugin requires it" | Core stability is not driven by unsupported peripherals | reject |
| Merged or submitted without testing evidence | "Committed an hour before the PR"; no test, no CI, no repro | Unverified changes regress | request-changes |
| New public flag without a migration plan | New bit in an existing flag set; old callers ignore it | Inconsistent behavior across versions | request-changes |

## Cross-File Review

Triggers apply to the **whole change set**, not one file.

- Header / `__all__` / exported Go API vs implementation: types introduced publicly must be used consistently and not leak internals.
- Caller vs callee contract: every caller respects the callee's error-return convention (Go: no `_` on errors; Python: no swallowed exceptions).
- Module boundaries: if a module exports a struct/class, no other module reaches into private fields.
- Public vs internal: functions marked as user-facing must not be the only path missing validation.
- Renames: every dependent module updated, or the rename is an ABI break.

## Anti-Patterns

| Anti-pattern | Why it is wrong | Governing principle |
|--------------|-----------------|---------------------|
| Special-case branching | Hides edge cases; makes the generic path fragile | Eliminate the special case so the edge case has nowhere to hide |
| Duplicated logic | Bugfixes diverge | Factor complicated logic into one helper |
| Fatal assertions for recoverable errors | User mistakes become crashes | Never abort on recoverable conditions |
| Exposing internal structs publicly | ABI freeze + security leak | Do not export implementation details |
| New public interface without justification | Surface area and maintenance | Reuse existing abstractions |
| Magic numbers | Obscure intent | Named constants or configuration |
| Unnecessary configuration knobs | Build-system bloat, hidden `#ifdef` bugs | Do not add flags with marginal benefit |
| Lock upgrades (read → write) | Deadlock | Never design an upgrade path |
| Blind allocation without size checks | Overflow / zero-size | Validate sizes; handle failure |
| Skipping validation on a trust boundary | Security holes | Never skip validation at a boundary |
| Cleanup while still holding a resource | Leak or deadlock | Release in reverse order of acquisition |
| Over-engineering for a single use | Complexity without payoff | No one-shot abstractions |
| Silent error swallowing | Bugs disappear | Every failure is reported |
| Premature optimization | Obscure tricks before the code is correct | Correctness > Performance |
| Undocumented workarounds | Future maintainers cannot tell hack from design | Document the deviation or fix the root cause |

## Agent Guidance

### Do

- Walk Level 1 before Level 2 before nits.
- Use the decision tree; do not invent a fifth severity.
- Translate C/kernel cues into the language under review (table above).
- Apply every trigger across the full change set, not just the file you opened first.
- Keep OWASP and SWEBOK checklists in play for anything this catalog does not name.

### Do Not

- Do not reject on style when the decision tree says nitpick.
- Do not approve a "small" ABI break because the new shape is cleaner.
- Do not treat "out of tree / our plugin needs this" as justification for a core change.
- Do not skip a security finding because it is not in this catalog.

## Checklist

### API / ABI stability

- [ ] No exported signature, layout, constant, CLI flag, or schema change without version bump or deprecation
- [ ] New public flags/endpoints are justified and documented
- [ ] Internal types are not exported

### Correctness and safety

- [ ] No fatal abort on recoverable, user-reachable conditions
- [ ] All untrusted inputs validated at the boundary
- [ ] Indexing and copies are bounded
- [ ] No use-after-free / double-close / dangling reference
- [ ] Allocations check failure and validate size

### Concurrency

- [ ] Lock order is consistent
- [ ] No recursive lock on a non-re-entrant mutex
- [ ] Shared flags use atomic or language happens-before
- [ ] No lock held across blocking I/O

### Complexity

- [ ] No one-shot abstraction
- [ ] No duplicated non-trivial algorithm
- [ ] No undocumented magic numbers
- [ ] Special-case branches have a data-structure justification or an external-standard justification

### Documentation and process

- [ ] Commit message explains why
- [ ] Comments match the code
- [ ] Evidence of testing exists
- [ ] Out-of-tree needs do not dictate core changes

If any reject-level item fails, the verdict is **reject** or **request-changes** per the decision tree. Do not approve around it.

## See Also

- wiki/tier2-core/code-review-method/overview.md
- wiki/tier3-working/checklists/code-review.md
- wiki/tier3-working/checklists/security-review.md
- wiki/tier3-working/owasp-code-review/overview.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md
- wiki/tier1-sources/owasp/a03-injection.md
- wiki/tier2-core/conventional-commits/overview.md
- wiki/tier2-core/distributed-systems/resilience-patterns.md

## Source

Mte90. *linus-torvalds-skill*, `linus-torvalds-skill/SKILL.md` (sections: Review Triggers, Severity Calibration, Severity Decision Tree, Anti-Patterns, Cross-File Review, Quick Reference Checklist). CC0 1.0. https://github.com/Mte90/linus-torvalds-skill

Local immutable capture: `references/linus-torvalds-skill/SKILL.md`.
