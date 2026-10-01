# Navigating a Change in Review (Tier 2)

> **Tier 2** | Source: Google Engineering Practices, `review/reviewer/navigate.md` | Authority: established | Derives From: SWEBOK KA12

## Summary

A multi-file review has an order. Reading files in the order the tool lists them, before deciding whether the change should exist, spends the review on code that may be thrown away. Take a broad view, examine the main part, then walk the rest.

## Key Concepts

### Step 1 — Does this change make sense?

Read the description (`cl-descriptions.md`) and what the change does in general. If the change should not happen, reply immediately with why, and with what the author should do instead. Courtesy matters here: disagreement is not a reason to dismiss the work.

If several changes arrive that the team does not want, fix the development process or the contributor process so people hear "no" before they write the code. Rejecting finished work is more expensive than an earlier conversation.

### Step 2 — Examine the main part first

Find the file or files with the largest logical change. Reviewing that part first supplies context for the smaller pieces and speeds the rest of the review. If the change is too large to identify a main part, ask the author what to read first, or ask them to split it (`small-cls.md`).

If the main part has a major design problem, send that comment immediately, even when you have not read the rest. The rest may disappear once the design changes. Two reasons to send it now:

- Authors often start the next change on top of this one while they wait. A late design rejection forces them to redo that follow-on work.
- Major redesigns take longer than small edits. The author needs to start the rework as soon as possible if a deadline and code health both still have to be met.

### Step 3 — Walk the rest in a deliberate order

Once the overall design is acceptable, pick a sequence that does not skip a file. After the main files, the tool's file order is usually enough. Reading the tests before the main code is sometimes better, because the tests state what the change is supposed to do.

## Agent Guidance

### Do

- Reject a change that should not exist before commenting on its internals, and offer an alternative.
- Send design-level comments on the main files as soon as you see them.
- Ask for a split when you cannot tell what the main part is.

### Do Not

- Do not finish a line-by-line pass after you already know the design will be replaced.
- Do not reject a misdirected change without saying what to do instead.
- Do not wait until the end of the review to reveal that the whole direction is wrong.

## Checklist

- [ ] The description was read and the change was judged as something that should exist
- [ ] A wrong direction was rejected immediately, with an alternative, in a courteous comment
- [ ] The main file or files were reviewed before the satellite files
- [ ] Major design comments were sent without waiting for a full pass
- [ ] Every file was then covered in an explicit order
- [ ] A change too large to navigate was sent back to be split

## See Also

- wiki/tier2-core/google-eng-practices/overview.md
- wiki/tier2-core/google-eng-practices/what-to-look-for.md
- wiki/tier2-core/google-eng-practices/cl-descriptions.md
- wiki/tier2-core/google-eng-practices/small-cls.md
- wiki/tier2-core/google-eng-practices/review-speed.md
- wiki/tier2-core/code-review-method/overview.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md

## Source

Google. *Navigating a CL in review*. Engineering Practices Documentation. CC BY 3.0. https://google.github.io/eng-practices/review/reviewer/navigate.html

Local immutable capture: `references/google-eng-practices/review/reviewer/navigate.md`. Snapshot `3bb3ec25b3b0199f4940b1aa75f0ac5c5753301c`.
