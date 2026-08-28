# Linus Torvalds Review Method (Tier 2)

> **Tier 2** | Source: Mte90/linus-torvalds-skill (LKML review distillation, CC0 1.0) | Authority: established | Derives From: SWEBOK KA12, KA04, KA14; ACM/IEEE-CS Code of Ethics

## Summary

This page captures a language-agnostic code-review method distilled from Linus Torvalds' Linux kernel mailing-list reviews: 38,293 review moves extracted from 31,397 emails (2002–2026) plus interview transcripts. The distilled skill (recommended variant: `SKILL.md`) encodes a reviewer mindset, a precedence hierarchy, invariant-level rejection rules, and a severity calibration drawn from that corpus.

For a coding agent, this is the *technical* review method: what to look for, in what order, and when a change is a hard blocker. It complements — it does not replace — the process and culture guidance in `wiki/tier3-working/code-review-guidelines/overview.md` (Google, Microsoft, Palantir, Thoughtbot) and the OWASP security review guide. Apply the method unless a Tier 1 authority (SWEBOK, OWASP, ACM/IEEE ethics) overrides it. The source skill's blunt personal-attack voice is **not** adopted here; see Authority Override below.

## Key Concepts

### Authority Override — Technical Method, Not Abusive Tone

The source skill includes a "blunt, evidence-driven" reviewing voice with examples of harsh rejection language. That voice **conflicts with** ACM/IEEE-CS Principle 7 (Colleagues: be fair and supportive; give honest, *constructive* reviews) and SWEBOK KA14 professional practice. Tier 1 wins.

| Adopt from the source | Do not adopt |
|-----------------------|--------------|
| Precedence: Correctness > Performance > Complexity > Style | Personal attacks, mockery, or insults |
| Say no early to fundamentally wrong changes | Blocking a merge over style when the style guide is silent |
| Data-structure-first diagnosis | Treating "clever" as a personality judgment |
| Concrete [REASON] → [ACT] findings | Vague "this is crap" with no principle or fix |
| Reject public-contract breakage | Using severity as a way to shame the author |

State the rule, cite the principle, name the consequence, propose the fix. Be direct. Be specific. Do not be cruel.

### Reviewer Mindset

| Attitude | Principle | Agent application |
|----------|-----------|-------------------|
| Say "no" early | If a change is fundamentally wrong, reject it immediately; discussion of details is wasted | Invariant-false findings are blockers. Do not nitpick around a broken contract. |
| Talk is cheap. Show the code. | Opinions must be backed by a concrete patch or a failing test | Do not speculate about "maybe this races." Point at the line, the invariant, and the consequence. |
| Data-structure first | Good programmers worry about data structures and their relationships, not the code that manipulates them | When you see special-case branches, ask whether a better structure would make the branch disappear. |
| Preserve existing users | Breaking a public contract is a bug unless the benefit is overwhelming and a migration path exists | Treat API, ABI, CLI, protobuf, and REST contract changes as correctness bugs until proven otherwise. |
| Trust is structured, not assumed | A small, trusted maintainer path replaces blind trust in every contributor | Required review, CI, and branch protection are the structure. Do not skip them for "trusted" authors. |
| Simplicity beats cleverness | If a solution can be expressed with fewer branches and no special cases, it is better | Reject one-off abstractions used once. Prefer deleting a special case over decorating it. |
| Performance only after correctness | A fast program that crashes is useless | Do not accept a hot-path optimization that drops a bounds check, a lock, or an error return. |

### Precedence Hierarchy

When two review rules clash, the higher tier wins:

1. **Correctness** — functional correctness, memory/resource safety, security, public contracts. Hard blocker.
2. **Performance** — measurable regressions on hot paths. Request-changes, unless the "optimization" introduced a correctness bug (then it is correctness).
3. **Complexity** — unnecessary abstractions, duplicated logic, special-case hacks. Request-changes unless they enable a correctness or performance gain that cannot be achieved otherwise.
4. **Style / nitpicks** — naming, formatting, cosmetic comments. Never block a merge unless they hide a deeper problem or violate a written style guide (PEP 8, `gofmt`, project style).

Additional precedence rules from the corpus:

| Rule | Wins over | When it does not apply |
|------|-----------|------------------------|
| Correctness | Performance | Pure micro-optimization with no observable behavior change, already proven correct |
| Protecting existing users | Adding new features | Major version bump with a documented migration path and notified callers |
| Security | Convenience | Code that cannot be reached from an untrusted boundary (prove it) |
| Bisectability | Quick fixes | Trivial one-liners with no observable side effects |
| Generic data flow | Special-case branches | Special case mandated by an external standard or hardware constraint |

### Key Definitions

| Term | Meaning |
|------|---------|
| **Bug** | A condition that causes incorrect behavior, crashes, data corruption, or a security vulnerability |
| **Hack / workaround** | A temporary fix that masks the root cause without addressing it |
| **Patch** | Neutral term for any code change, regardless of size or intent |
| **Invariant-false** | A rule with no exceptions; violating it is a reject or request-changes blocker |
| **Recoverable error** | A condition that must be reported via an error value, not by aborting the process |
| **API contract** | Documented or implied behavior that external code depends on; changing it without a migration path is a bug |
| **Special case** | A branch that exists solely because the data model treats one value differently |
| **Data structure** | The concrete representation of data (lists, maps, trees, tables) that determines how algorithms are expressed |

### Reasoning Protocol

Every finding must be a two-step **[REASON] → [ACT]** record. Findings that skip the reason are not reviews.

```
[REASON]
- Identify the exact pattern in the code (file, function, line).
- Cite the principle or trigger that is violated.
- Name the concrete consequence (crash, silent data loss, ABI break, deadlock, injection).

[ACT]
- What must change.
- Severity: reject | request-changes | nitpick | discussion | approve.
- A suggested fix, or a pointer to an existing helper that already does the right thing.
```

Example (tone is direct, not abusive):

```
[REASON] createClient copies nicklen bytes into a freshly allocated buffer and never writes the
terminating NUL. Subsequent C-string reads walk off the buffer. Principle: unbounded
buffer-size mismatch is invariant-false.

[ACT] request-changes. Copy nicklen+1 bytes, or set c->nick[nicklen] = '\0' after memcpy.
```

### How This Method Fits Other Review Pages

| Page | Role |
|------|------|
| This page + `triggers.md` | What to look for, in what order, and how to grade severity |
| `wiki/tier3-working/code-review-guidelines/overview.md` | How to conduct the review: pacing, comment culture, when to approve |
| `wiki/tier3-working/owasp-code-review/overview.md` | Security-specific checkpoints mapped to OWASP Top 10 |
| `wiki/tier3-working/checklists/code-review.md` | Operational checklist; apply after this method's invariant scan |
| `wiki/tier3-working/checklists/security-review.md` | Security checklist; still required even if a trigger is not in the catalog |

Google's "approve when the change improves code health" and this method's "say no early" are not opposites. Combined rule: **reject invariant-false findings immediately; approve incremental improvements that do not violate invariants; never block on style nits.**

### Validation Caveat

The source skill was validated on antirez/smallchat (706 LOC, C). One model gained net critical findings with the skill; two others lost criticals the baseline had caught because the skill narrowed focus too aggressively. For this wiki:

- Use the trigger catalog as the *first* scan (fatal invariants, then structure).
- Then still apply the full code-review, security-review, and testing-review checklists.
- Do not drop a finding because it is absent from the trigger catalog. Absence of a trigger is not evidence of safety.

## Agent Guidance

### Do

- Scan Level 1 invariants before style, naming, or "is this Pythonic?" comments.
- Apply Correctness > Performance > Complexity > Style when rules conflict.
- Prefer diagnosing a bad data structure over adding another special-case branch.
- Treat public API, ABI, CLI flag, protobuf, and REST contract changes as bugs until a migration path exists.
- Require a [REASON] → [ACT] pair for every non-approval finding.
- Reject fatal aborts (`panic`, `BUG_ON`, `sys.exit`, bare `assert` as control flow) on recoverable, user-reachable conditions.
- Demand a commit message that explains *why*, not just *what*. Conventional Commits supplies the format; this method supplies the substance.
- Review the whole change set (headers vs implementation, callers vs callees, public vs internal) — not one file in isolation.
- Keep findings technically blunt and specific: name the invariant, the line, and the fix.

### Do Not

- Do not imitate abusive or mocking review language from LKML excerpts. ACM/IEEE Principle 7 overrides the source persona.
- Do not block a merge on style when the project style guide is silent and no correctness issue is hiding under the style.
- Do not accept "we'll fix the invariant later" without a tracked remediation and a reason it is safe *now*.
- Do not treat a performance win as justification for dropping validation, locking, or error returns.
- Do not let this method suppress OWASP or SWEBOK findings that have no matching trigger. Run the security-review checklist anyway.
- Do not approve a change that is "almost right" if it breaks a public contract or introduces a recoverable-error abort.
- Do not review only the diff hunks. Read surrounding functions and call sites for contract mismatches.

## Checklist

- [ ] Level 1 invariants scanned before any style or naming comments
- [ ] Precedence hierarchy applied when two rules conflict
- [ ] Every non-approval finding has [REASON] → [ACT] with a named principle
- [ ] Public contracts (API/ABI/CLI/schema) unchanged, or a migration path is documented
- [ ] No fatal abort on a recoverable, user-reachable condition
- [ ] Special-case branches questioned: can a data-structure change delete them?
- [ ] Commit message states why, not only what
- [ ] Full change set reviewed, not a single file
- [ ] Code-review and security-review checklists still applied after this scan
- [ ] Findings are direct and specific; no personal attacks

## See Also

- wiki/tier2-core/code-review-method/triggers.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md
- wiki/tier1-sources/swebok-v4/ka04-construction.md
- wiki/tier1-sources/swebok-v4/ka14-professional-practice.md
- wiki/tier1-sources/acm-ieee-ethics/code-of-ethics.md
- wiki/tier3-working/checklists/code-review.md
- wiki/tier3-working/code-review-guidelines/overview.md
- wiki/tier3-working/owasp-code-review/overview.md
- wiki/tier2-core/conventional-commits/overview.md
- wiki/tier2-core/solid-principles/overview.md

## Source

Mte90. *linus-torvalds-skill* (recommended variant `linus-torvalds-skill/SKILL.md`). Distilled from 38,293 Linux kernel mailing-list review moves (31,397 emails, 2002–2026) plus interview transcripts. CC0 1.0. https://github.com/Mte90/linus-torvalds-skill

Local immutable capture: `references/linus-torvalds-skill/SKILL.md`.
