# Handling Pushback in Code Reviews (Tier 2)

> **Tier 2** | Source: Google Engineering Practices, `review/reviewer/pushback.md` | Authority: established | Derives From: SWEBOK KA12, KA14

## Summary

Authors sometimes disagree with a suggestion, or object that the review is too strict. The reviewer's job is to check whether the author is actually right, and, when the suggestion still improves code health enough to justify the work, to keep asking for it politely. Code health improves in small steps. Giving in to "I'll clean it up later" is a common way codebases degenerate.

## Key Concepts

### Who is right

The author is often closer to the code and may have the better picture. If their argument holds, including from a code-health perspective, say so and drop the point.

When the author is not right, explain again. A good explanation shows that you understood their reply and adds information about why the change is still requested. It can take more than one round before the point lands. Stay polite (`review-comments.md`). Show that you heard them and still disagree.

### Upset and strictness

Reviewers often fear that insisting will upset the author. Upset is usually brief, and it is more often about how the comment was written than about the insistence. Polite comments frequently produce no upset at all. Authors who were helped to improve the code are often grateful later.

If a team moves from lax reviews to strict ones, some authors will complain loudly. Faster reviews (`review-speed.md`) are what make those complaints fade, sometimes over months. The loudest objectors sometimes become the strongest supporters once they see the code the stricter bar produces.

### Cleaning it up later

Authors want the change landed and will offer a follow-up. A few people do write that follow-up immediately. Experience in the source: the longer the delay after the original change, the less likely the cleanup is. Unless it happens immediately, it usually never happens. That is workload, not bad faith. Insist that this change be cleaned up before it is "done" in the codebase.

| Case | Rule |
|------|------|
| The change introduces new complexity | Clean it up before submission, unless this is an emergency (`emergencies.md`). |
| The change exposes a surrounding problem that cannot be fixed in this change | The author files a bug, assigns it to themselves, and may add a TODO that references the bug. |
| The author promises an untracked follow-up | Do not accept that as the plan. |

### Conflicts

If the disagreement still will not resolve, follow the conflict path in `review-standard.md`: consensus from the guides, then a conversation whose result is written on the change, then escalation. Do not park the change.

## Agent Guidance

### Do

- Re-read the author's argument for a code-health reason before repeating the request.
- Drop the point, explicitly, when they are right.
- Keep advocating a health improvement that justifies the extra work, in polite, specific comments.
- Require cleanup of new complexity in this change.
- Require a filed, self-assigned bug when the leftover is pre-existing and out of scope.

### Do Not

- Do not accept "later" for complexity this change introduced.
- Do not soften a real health problem into a nit to avoid conflict.
- Do not match hostility. Courtesy is the mechanism that keeps strict reviews tolerable.
- Do not lower the bar because a team is newly unhappy with stricter reviews. Speed up the responses instead.

## Checklist

- [ ] The author's counterargument was considered on the merits before the request was repeated
- [ ] Dropped points were acknowledged as dropped
- [ ] Repeated requests explain why, and stay about the code
- [ ] New complexity is resolved before merge, or the change is an emergency
- [ ] Out-of-scope surrounding issues have a bug assigned to the author
- [ ] Unresolved conflict is on the escalation path, not sitting idle

## See Also

- wiki/tier2-core/google-eng-practices/review-standard.md
- wiki/tier2-core/google-eng-practices/review-comments.md
- wiki/tier2-core/google-eng-practices/review-speed.md
- wiki/tier2-core/google-eng-practices/handling-comments.md
- wiki/tier2-core/google-eng-practices/emergencies.md
- wiki/tier2-core/code-review-method/overview.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md
- wiki/tier1-sources/acm-ieee-ethics/code-of-ethics.md

## Source

Google. *Handling pushback in code reviews*. Engineering Practices Documentation. CC BY 3.0. https://google.github.io/eng-practices/review/reviewer/pushback.html

Local immutable capture: `references/google-eng-practices/review/reviewer/pushback.md`. Snapshot `3bb3ec25b3b0199f4940b1aa75f0ac5c5753301c`.
