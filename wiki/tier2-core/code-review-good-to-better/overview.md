# Good and Better Code Reviews (Tier 2)

> **Tier 2** | Source: Gergely Orosz, "How to Make Good Code Reviews Better" (The Pragmatic Engineer; Stack Overflow Blog, 2019-09-30) | Authority: established | Derives From: SWEBOK KA12, KA14, KA9

## Summary

This page is the conduct layer of code review: how thorough the pass is, how comments are phrased, when to approve, when to leave the review tool and talk, what to do with nits, how to review a new joiner, how to handle cross-time-zone reviews, and what the organization owes the practice. The technical scan stays in `wiki/tier2-core/code-review-method/overview.md`. The approval bar stays in `wiki/tier2-core/google-eng-practices/review-standard.md`.

Orosz splits the practice into two bars. **Good** is the minimum any team can hold: review the change itself, stay unopinionated, mark nits, use one bar for every author, and make cross-zone reviews less painful. **Better** keeps that bar and adds context: the system, the author, and the situation, plus fixes for pain that keeps showing up outside the review tool. Apply the good bar on every review. Apply the better bar when the context is actually known.

## Key Concepts

### The Two Bars

| Topic | Good (minimum on every review) | Better (when context is known) |
|-------|--------------------------------|--------------------------------|
| Scope | The change, its fit in the codebase, title and description, the reason for the change, correctness, tests, behavior, and the written guides. Call out unclear code, unclear names, commented-out code, missing tests, unhandled edges, and a change that should have been split. | The change in the larger system. Whether it is necessary. How it affects other parts. Whether a new abstraction fits the architecture. Maintainability: simpler logic, clearer tests, less duplication. A light pass, then a contextual pass. |
| Tone | Open questions. Alternatives offered as options. Ask when the reviewer may be missing context. | The same, plus recognition of effort and of solutions that work. |
| Status | Do not approve while a verdict-changing question is open. Mark non-blocking comments. State the decision in words. State a request for changes in words, using the tool. | Same principles. A non-blocking note may move to a tracked follow-up change. An urgent change gets a faster review, not a skipped one. |
| Conversation | Comment on what matters. If a revision misses a comment, say so. A long thread moves to a direct conversation. | After a first pass with many comments, talk before another written round. Many comments usually mean a misunderstanding. |
| Nits | A nit is a comment the change could merge without. Prefix `nit:`. A flood of them hides the real findings. | The same nit, repeated, is a missing linter or a missing written standard. Fix it outside the review. |
| New joiners | One bar for every author: title, level, and start date do not change approve or reject. | The first reviews add explanation, pointers to the guides, and notes on what worked. The bar stays. |
| Time zones | Review during overlapping hours when possible. Many comments get an offer to talk. | The same dependency, reviewed across zones again and again, is an ownership and design question. |
| Organization | Everyone reviews, including people on solo work. Review time is part of the job, and a larger share of it for senior engineers. Large codebases get a written review guide. | Nothing reaches production without a review. Urgent work has an expedited path. Tooling and process keep getting investment. A hostile review is a process defect. |

### What a Pass Covers

The good pass reads the title, the description, and the reason for the change before the diff. It then checks correctness, tests, behavior, and the project's written guides. It asks for a split when one review carries more than one purpose. Size numbers live in `wiki/tier2-core/google-eng-practices/small-cls.md`.

The better pass asks three questions the diff alone cannot answer: is this change needed, what else does it affect, and does a new abstraction match the architecture that is already there. Joel Kemp's light-then-contextual order, which Orosz cites, matches Google's "broad view, then the main file" in `wiki/tier2-core/google-eng-practices/navigating-a-cl.md`. Send a design objection as soon as it is clear.

### Authority Resolution

| Conflict | Winner | Rule an agent applies |
|----------|--------|------------------------|
| This page vs SWEBOK, OWASP, a PEP, or the ACM/IEEE Code of Ethics | Tier 1 | Revise the review. Comment tone is already required by Principle 7. |
| Follow-up change vs "clean it up now" | `google-eng-practices/handling-pushback.md` and `review-standard.md` | A health decrease, an invariant break, a security defect, or missing tests for the new behavior stay on this change. A `nit:` or other non-blocking note may move to a tracked follow-up. A promise of a later change does not justify landing a regression. |
| "One bar for everyone" vs "say no early" | Both | Reject invariant-false findings for every author, including a new joiner. Extra explanation does not waive the finding. |
| Explicit approval vs LGTM | Both | LGTM is an approval under Google's definition. Also say the decision in words when the reader may not know the acronym. |
| Talk it through vs Google's in-person review | Both | Use a conversation to clear a misunderstanding. Record the outcome on the change. The reviewer asks; the author does not drive the review. |
| Single-purpose change vs "~100 lines" | `small-cls.md` for the size; both for one purpose | Split a multi-purpose change. Use Google's size guidance for how small. |
| "No production without review" vs emergencies | `google-eng-practices/emergencies.md` | The only skip of the normal bar is a true emergency, and it requires a follow-up review. Expedited means faster, not absent. |
| This page vs the Tier 3 multi-source guidelines | This page | `wiki/tier3-working/code-review-guidelines/overview.md` is a mix. It does not override this page or the Google directory. |
| Repeated nits vs "nits never block" | Both | Label and do not block. Then remove the recurrence with a linter or a written standard. |
| Repeated cross-zone review load | This page | Ask whether the change is in the right codebase, whether the rate will fall, and whether an interface, a service boundary, or tooling would remove the dependency. |

This page does not ingest SmartBear's process guide or Philipp Hauer's guidelines. Orosz names both as further reading. SmartBear's line-count research on the Tier 3 guidelines page still loses to `small-cls.md`.

## Agent Guidance

### Do

- Run the good column on every review you write or judge.
- Run the better column when you know the system, the author, or the delivery pressure. Ask when a better-column question would change the verdict and the context is missing.
- Prefix optional comments with `nit:` and keep them few.
- State approval or a change request in words. Mark which comments block.
- Move a long thread, or a first pass that produced many comments, to a conversation, then write the outcome on the change.
- For a new joiner, point at the written guide and explain the alternative. Keep the same approve/reject bar.
- Review during overlap hours when the author is in another zone.
- Treat a repeated nit as a tooling or standards task. Treat a repeated cross-zone dependency as a design task.
- For urgent work, review sooner. Reserve a skipped bar for `emergencies.md`.

### Do Not

- Do not lower the bar because the author is new, senior, or in a hurry.
- Do not approve while a question that could change the verdict is still open.
- Do not let a follow-up promise carry a health regression, an invariant break, or a missing test of the new behavior.
- Do not copy a hostile or opinionated tone. Principle 7 and this page both forbid it.
- Do not flood a review with nits, and do not keep writing the same nit after a linter or a team standard could own it.
- Do not treat this page as permission to skip review, including for a solo project or a small change.

## Checklist

- [ ] Title, description, and the reason for the change were read before the diff
- [ ] Correctness, tests, behavior, guides, naming, dead code, and single purpose were checked
- [ ] A system-level objection (necessity, blast radius, a new abstraction) was raised when the context was known
- [ ] Verdict-changing questions are resolved or still blocking; everything else is marked non-blocking
- [ ] The decision is stated in words
- [ ] Nits use `nit:` and do not block; a repeated nit has a tooling or standards follow-up
- [ ] A long thread or a comment-heavy first pass moved to a conversation, with the outcome recorded on the change
- [ ] A new joiner received pointers and explanation, and the same bar as everyone else
- [ ] Cross-zone timing was considered; a repeated dependency was raised as a design question
- [ ] Urgent work was expedited; it was not merged unreviewed unless `emergencies.md` applies

## See Also

- wiki/tier2-core/google-eng-practices/overview.md
- wiki/tier2-core/google-eng-practices/review-standard.md
- wiki/tier2-core/google-eng-practices/review-comments.md
- wiki/tier2-core/google-eng-practices/handling-pushback.md
- wiki/tier2-core/google-eng-practices/emergencies.md
- wiki/tier2-core/code-review-method/overview.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md
- wiki/tier1-sources/acm-ieee-ethics/code-of-ethics.md
- wiki/tier2-core/engineering-playbook/developer-experience.md
- wiki/tier3-working/checklists/code-review.md

## Source

Gergely Orosz. "How to Make Good Code Reviews Better." *The Pragmatic Engineer*, republished on the Stack Overflow Blog, 30 September 2019. https://stackoverflow.blog/2019/09/30/how-to-make-good-code-reviews-better/

Local structured summary (not a reproduction of the article): `references/orosz-how-to-make-good-code-reviews-better.md`.
