# Google Engineering Practices — Overview (Tier 2)

> **Tier 2** | Source: Google Engineering Practices (google/eng-practices, CC BY 3.0) | Authority: established | Derives From: SWEBOK KA12, KA8, KA14

## Summary

Google's public engineering-practices site is the canonical description of how Google reviews code. As of the archived upstream snapshot, the site contains one practice: code review, split into a reviewer's guide and a change author's guide. The purpose of that practice is to improve the code health of a codebase over time, while still letting authors make progress.

For a coding agent this is the process layer of review: when to approve, how fast to respond, how to write comments, how large a change should be, and what the author owes the reviewer. It does not replace the technical scan in `wiki/tier2-core/code-review-method/overview.md` (what is invariant-false, and in what order to grade it) or the security checkpoints in the OWASP review guide. Apply these pages unless a Tier 1 authority overrides them.

The upstream repository was archived read-only by Google on 2025-11-21. This wiki synthesizes that frozen tree, not a living Google-internal policy.

## Coverage Map

| Upstream document | Wiki page | Role |
|-------------------|-----------|------|
| `README.md`, `review/index.md`, both guide indexes | `overview.md` (this page) | Terminology, reviewer selection, what review covers, authority resolution |
| `review/reviewer/standard.md` | `review-standard.md` | The senior approval rule and conflict resolution |
| `review/reviewer/looking-for.md` | `what-to-look-for.md` | Design through documentation, every line, context |
| `review/reviewer/navigate.md` | `navigating-a-cl.md` | Order of reading a multi-file change |
| `review/reviewer/speed.md` | `review-speed.md` | Response time, LGTM-with-comments, large changes |
| `review/reviewer/comments.md` | `review-comments.md` | Courtesy, severity labels, accepting explanations |
| `review/reviewer/pushback.md` | `handling-pushback.md` | Disagreement, "clean it up later", strictness |
| `review/developer/small-cls.md` | `small-cls.md` | Size, splitting, tests in the same change |
| `review/developer/cl-descriptions.md` | `cl-descriptions.md` | What and why; first line; body |
| `review/developer/handling-comments.md` | `handling-comments.md` | Author response: fix the code, stay collaborative |
| `review/emergencies.md` | `emergencies.md` | The only case where health rules relax |

## Key Concepts

### Terminology

| Term | Meaning in this wiki |
|------|----------------------|
| **CL** (changelist) | One self-contained change submitted for review or already merged. Read this as pull request, patch, or change. |
| **LGTM** | "Looks Good to Me." Approval. An LGTM means the change meets the review standard, not merely that the reviewer skimmed it. |
| **Code health** | Maintainability, readability, understandability, consistency, test coverage, and fitness for the system. The quantity review is trying to increase over time. |
| **Nit** | A comment that is polish. The author may ignore it. It must not block approval. |

### What Review Examines

Every review looks at design, functionality, complexity, tests, naming, comments, style, and documentation. The operational list is `what-to-look-for.md`. The approval bar is `review-standard.md`.

### Choosing Reviewers

The best reviewer is the person who can give the most thorough and correct review and still respond within a reasonable time. That is usually the owner of the code, who may or may not be listed in an owners file. Different parts of one change can have different reviewers. If the ideal reviewer is unavailable, still CC them.

Pair programming with someone qualified to review the code counts as review. An in-person review is valid when the reviewer asks questions and the author speaks only when spoken to.

### Authority Resolution

| Conflict | Winner | Rule an agent applies |
|----------|--------|------------------------|
| These pages vs SWEBOK, OWASP, a PEP, or the ACM/IEEE Code of Ethics | Tier 1 | Revise the review. Courtesy in comments is already required by Principle 7; Google's comment rules reinforce it. |
| "Approve once it improves code health" vs the Linus method's "say no early" | Both, on different findings | Reject invariant-false findings immediately. Approve a change that improves health and does not violate an invariant. Never block on a style nit. A change that worsens health is not "imperfect but improving." |
| Google style guide vs personal preference | The written style guide | In this wiki the style guide is PEP 8, Effective Go, or the project's written guide. Points absent from that guide are nits. |
| Google CL description vs Conventional Commits | Both, on different parts | Subject line uses Conventional Commits (`type(scope): imperative summary`). The body follows Google's what/why rules. |
| Google "~100 lines / one concern" vs the Tier 3 SmartBear 200–400 LOC research bound | Tier 2 Google rule | One self-contained change. About 100 lines is usually reasonable; about 1000 is usually too large. A reviewer may reject for size alone. |
| Emergency speed vs the review standard | `emergencies.md`, narrowly | Only a true emergency relaxes anything other than "does this resolve the emergency, correctly?" Follow with a thorough review. |

The multi-source working summary at `wiki/tier3-working/code-review-guidelines/overview.md` remains useful for Microsoft, SmartBear, Palantir, and Thoughtbot material. Where it restates Google and a page in this directory disagrees, this directory wins.

## Agent Guidance

### Do

- Read the sub-page for the decision you are making. This overview is the map, not the rule set.
- Treat a CL as a pull request when applying these pages outside Google.
- Record the outcome of any off-thread conflict discussion as a comment on the change, so later readers can see it.
- Send major design objections immediately, before finishing a line-by-line pass.

### Do Not

- Do not treat this directory as Tier 1. It is established practice, not an IEEE or OWASP standard.
- Do not import Google-internal tools (g3doc, owners-file workflow, Borg) as requirements. Translate them: docs, code owners, the project's production environment.
- Do not use the archived date as a reason to ignore the rules. The snapshot is the source of record for this wiki.

## Checklist

- [ ] The relevant sub-page was read, not only this overview
- [ ] Tier 1 was checked before a Google rule was applied against SWEBOK, OWASP, a PEP, or ethics
- [ ] Invariant-false findings were rejected even when the rest of the change improves health
- [ ] Style nits were labeled and did not block approval
- [ ] Commit subject follows Conventional Commits; description body states what and why

## See Also

- wiki/tier2-core/google-eng-practices/review-standard.md
- wiki/tier2-core/google-eng-practices/what-to-look-for.md
- wiki/tier2-core/google-eng-practices/small-cls.md
- wiki/tier2-core/google-eng-practices/cl-descriptions.md
- wiki/tier2-core/google-eng-practices/emergencies.md
- wiki/tier2-core/code-review-method/overview.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md
- wiki/tier1-sources/acm-ieee-ethics/code-of-ethics.md
- wiki/tier3-working/checklists/code-review.md

## Source

Google. *Engineering Practices Documentation*. CC BY 3.0. https://google.github.io/eng-practices/

Upstream repository `google/eng-practices`, archived read-only on 2025-11-21. Snapshot tree `3bb3ec25b3b0199f4940b1aa75f0ac5c5753301c`.

Local immutable capture: `references/google-eng-practices/` (`README.md`, `review/index.md`, `review/reviewer/index.md`, `review/developer/index.md`).
