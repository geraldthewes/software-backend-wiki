# How to Write Code Review Comments (Tier 2)

> **Tier 2** | Source: Google Engineering Practices, `review/reviewer/comments.md` | Authority: established | Derives From: SWEBOK KA12, KA14; ACM/IEEE-CS Code of Ethics Principle 7

## Summary

Review comments are kind, they explain the reasoning, they balance explicit direction with room for the author to decide, and they push the author to simplify the code or comment it rather than to explain the complexity only to the reviewer. Courtesy is also a Tier 1 requirement: ACM/IEEE Principle 7 requires fair, supportive, constructive reviews. This page is how to do that without becoming vague.

The primary goal of a comment is the best change possible. The secondary goal is teaching the author so later changes need less review.

## Key Concepts

### Courtesy

Comment on the code, not the author. This matters most when the point is contentious.

| Weaker | Stronger |
|--------|----------|
| "Why did you use threads here when there's obviously no benefit?" | "This concurrency model adds complexity without a performance benefit I can see. With no performance benefit, this should be single-threaded." |

The stronger form also explains why. You do not need a paragraph on every nit. Add the intent, the practice you are applying, or how the suggestion improves code health when the author would otherwise have to guess.

### Who fixes the change

Fixing the change is the author's job. The reviewer is not required to produce a detailed design or to write the code. Pointing at the problem and letting the author decide often teaches more and can produce a better fix, because the author is closer to the code.

Sometimes a direct instruction, a suggestion, or even a snippet is the more helpful move. Choose based on which one gets to the best change. People also learn from being told what they did well: a cleaned-up algorithm, strong tests, or something the reviewer learned. Say why you liked it.

### Severity labels

Label comments so authors do not treat every note as mandatory.

| Label | Meaning |
|-------|---------|
| `Nit:` | Minor. Technically worth doing. It will not hugely affect the change. Not a merge blocker. |
| `Optional:` or `Consider:` | May be a good idea. Not strictly required. |
| `FYI:` | Not expected in this change. Worth thinking about later. |

Unlabeled comments read as mandatory. If you did not mean that, label them.

### Accepting explanations

If you ask why a piece of code is the way it is, the usual result is that the author **rewrites it until it is clear**. A code comment is appropriate when the code cannot hold the reason. A comment that only restates overly complex code is not the fix; simplify the code.

An explanation that exists only in the review tool does not help the next reader. It is acceptable in narrow cases, such as when you are outside your familiar area and the author explains something a normal reader of that code would already know.

## Agent Guidance

### Do

- Write comments about the code, with the reason attached when the change is not obvious.
- Label `Nit`, `Optional`/`Consider`, or `FYI` whenever a comment is not a required change.
- Ask for a clearer implementation, or a why-comment, instead of accepting a thread-only explanation.
- Note specific things the change does well, with the reason.

### Do Not

- Do not comment on the author's competence, motives, or taste.
- Do not leave severity implicit.
- Do not require yourself to implement the fix before the author may proceed.
- Do not accept "I explained it in the review" as a substitute for readable code.

## Checklist

- [ ] Contentious comments name the code and the consequence, not the person
- [ ] Non-obvious requests include why they improve code health
- [ ] Required changes are distinguishable from `Nit`, `Optional`/`Consider`, and `FYI`
- [ ] Unclear code was sent back to be clarified or commented, not merely discussed
- [ ] At least the teaching goal was served: either a problem the author can decide how to fix, or a direct suggestion when that is more helpful
- [ ] Good work in the change was acknowledged with a reason

## See Also

- wiki/tier2-core/google-eng-practices/overview.md
- wiki/tier2-core/google-eng-practices/review-standard.md
- wiki/tier2-core/google-eng-practices/handling-comments.md
- wiki/tier2-core/google-eng-practices/handling-pushback.md
- wiki/tier2-core/code-review-method/overview.md
- wiki/tier1-sources/acm-ieee-ethics/code-of-ethics.md
- wiki/tier1-sources/swebok-v4/ka14-professional-practice.md
- wiki/tier3-working/checklists/code-review.md

## Source

Google. *How to write code review comments*. Engineering Practices Documentation. CC BY 3.0. https://google.github.io/eng-practices/review/reviewer/comments.html

Courtesy link in the upstream text: Chromium respect doc, https://chromium.googlesource.com/chromium/src/+/master/docs/cr_respect.md

Local immutable capture: `references/google-eng-practices/review/reviewer/comments.md`. Snapshot `3bb3ec25b3b0199f4940b1aa75f0ac5c5753301c`.
