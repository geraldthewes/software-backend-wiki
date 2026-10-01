# The Standard of Code Review (Tier 2)

> **Tier 2** | Source: Google Engineering Practices, `review/reviewer/standard.md` | Authority: established | Derives From: SWEBOK KA12, KA14

## Summary

The primary purpose of code review is to make the overall code health of the codebase improve over time. Every other review mechanic — speed, comment style, small changes, LGTM-with-comments — exists to serve that purpose.

The senior principle, which outranks the other rules in this directory: **approve a change once it definitely improves the code health of the system, even if it is not perfect.** There is no perfect code, only better code. Reviewers seek continuous improvement, and they balance the need to make progress against the importance of the remaining suggestions.

## Key Concepts

### The Approval Bar

| Situation | Decision |
|-----------|----------|
| The change as a whole improves maintainability, readability, or understandability | Approve. Do not hold it for days to polish tiny pieces. |
| Something could be better but is not important | Comment, prefixed `Nit:`. The author may ignore it. |
| The comment is purely educational | Allowed. Prefix `Nit:` or otherwise mark it as not required for this change. |
| The change definitely worsens code health | Do not approve. The only exception is a true emergency (`emergencies.md`). |
| The change is well-designed but adds a feature the reviewer does not want in the system | Denial is allowed. Design quality does not force acceptance of unwanted scope. |
| Several designs are equally valid on data or engineering principle | Accept the author's preference. |
| Design choices are not equally valid | The choice follows design principles, not taste. Design is almost never "just style." |

Blocking progress on every change trains authors to stop improving the codebase. Accepting a stream of small health decreases, especially under time pressure, is how a codebase degrades. The reviewer owns that balance and is responsible for the code they approve.

### Principles

1. Technical facts and data overrule opinions and personal preferences.
2. On style, the written style guide is the absolute authority. A pure style point that the guide does not cover (whitespace and similar) is personal preference. Match the surrounding code. If there is no previous style, accept the author's.
3. Software design is weighed on principles. When the author shows that several approaches are equally valid, the reviewer accepts the author's preference. Otherwise standard design principles decide.
4. If no other rule applies, the reviewer may ask for consistency with the current codebase, provided that consistency does not worsen code health.

In this wiki the style guide is PEP 8, Effective Go, or the project's written guide. See the authority table in `overview.md`.

### Mentoring

Teaching a language, a framework, or a design principle is part of improving code health over time. Educational comments are always fine. If the lesson is not required to meet the standard, say so.

### Resolving Conflicts

1. Author and reviewer try to reach consensus using this standard, the reviewer guide, and the author guide.
2. If comments are not converging, meet face to face or on video. Write the result back onto the change so future readers can see it.
3. If that fails, escalate: a broader team discussion, a technical lead, a maintainer of the code, or an engineering manager.
4. Do not leave the change sitting because the two sides cannot agree.

## Agent Guidance

### Do

- State whether a comment is required or a nit before asking for a change.
- Approve when the change is a definite health improvement and no invariant is false.
- Cite a technical fact, a measurement, or a named design principle when rejecting a design.
- Prefer the author's preference when two designs are honestly equivalent.

### Do Not

- Do not demand that every tiny piece be polished before approval.
- Do not approve a change that worsens health because it is "good enough for now," outside a true emergency.
- Do not treat a design objection as a style nit to avoid the argument.
- Do not let an unresolved disagreement stall the change with no escalation.

## Checklist

- [ ] The change definitely improves code health, or it was rejected for worsening it
- [ ] Remaining polish is labeled `Nit:` and is not a merge blocker
- [ ] Educational comments are marked as not required for this change
- [ ] Style objections cite the written style guide, or they are nits
- [ ] Design objections cite a principle or data, not taste
- [ ] Any off-thread resolution is recorded on the change
- [ ] Invariant-false findings from the review method were not waved through under this standard

## See Also

- wiki/tier2-core/google-eng-practices/overview.md
- wiki/tier2-core/google-eng-practices/what-to-look-for.md
- wiki/tier2-core/google-eng-practices/emergencies.md
- wiki/tier2-core/google-eng-practices/handling-pushback.md
- wiki/tier2-core/code-review-method/overview.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md
- wiki/tier1-sources/python-peps/pep-008-style.md
- wiki/tier3-working/checklists/code-review.md

## Source

Google. *The Standard of Code Review*. Engineering Practices Documentation. CC BY 3.0. https://google.github.io/eng-practices/review/reviewer/standard.html

Local immutable capture: `references/google-eng-practices/review/reviewer/standard.md`. Snapshot `3bb3ec25b3b0199f4940b1aa75f0ac5c5753301c`.
