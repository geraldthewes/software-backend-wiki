# What to Look For in a Code Review (Tier 2)

> **Tier 2** | Source: Google Engineering Practices, `review/reviewer/looking-for.md` | Authority: established | Derives From: SWEBOK KA12, KA4, KA5

## Summary

Apply this list under the approval bar in `review-standard.md`. The question on each item is not "is this perfect?" It is "does this change improve code health, and is anything here bad enough that the change should not land as it stands?"

Design is the most important item. A line-by-line pass on a change whose design is wrong wastes the review.

## Key Concepts

### Review Dimensions

| Dimension | Question | Agent action |
|-----------|----------|--------------|
| **Design** | Do the pieces interact sensibly? Does this belong in this codebase or in a library? Does it integrate? Is this the right time to add it? | If the design is wrong, say so before reviewing the rest. |
| **Functionality** | Does it do what the author intended, and is that behavior good for end users and for the next developer? | Think through edge cases, concurrency, and user impact. Authors are expected to have tested; reviewers still read for bugs. |
| **Complexity** | Can a reader understand it quickly? Will the next editor introduce bugs? | Reject over-engineering: generality or features for a problem that is not here yet. Solve the problem that exists now. |
| **Tests** | Are the tests correct, sensible, and useful? Will they fail when the code breaks? Do they assert something simple? Are they split across cases? | Tests ship in the same change as the production code, except emergencies. Do not accept complex tests merely because they are not in the main binary. Tests do not test themselves. |
| **Naming** | Is the name long enough to say what the thing is or does, and still readable? | Rename when a name hides the concept. |
| **Comments** | Are they necessary, clear, and mostly *why* rather than *what*? | If the code cannot explain itself, simplify the code. Exceptions: regular expressions and complex algorithms often need a *what* comment. Also read pre-existing comments: a TODO may now be done, or a comment may advise against this change. |
| **Style** | Does it follow the written style guide? | Guide violations are required fixes. Anything else is `Nit:` and must not block. Major reformatting is its own change, never mixed with a functional change. |
| **Documentation** | If users build, test, interact with, or release the code differently, did the docs change? | Includes READMEs and generated reference docs. Deleting or deprecating code includes considering deletion of its docs. If docs are missing, ask. |
| **Consistency** | The guide wins when it states a requirement. When it only recommends, prefer the guide unless matching the surroundings would be too confusing. If no rule applies, match existing code. | Ask the author to file a bug and leave a TODO for cleaning up the old inconsistency. |

Documentation of a class, module, or function is not a comment. It states purpose, how to use the thing, and how it behaves. Comments hold reasoning the code cannot hold.

### User-facing and concurrent changes

For a UI or other user-facing change, reading the diff is often not enough. Ask for a demo, or apply the change and exercise it. For parallel code that can deadlock or race, both author and reviewer have to think it through; running the code rarely catches these. That difficulty is itself a reason to prefer concurrency models where races and deadlocks are hard to express.

### Every line, with stated exceptions

Read every line you were assigned. Data files, generated code, and large data structures may be scanned. Do not scan a human-written class, function, or block and assume the inside is fine. Some code deserves more scrutiny than other code; that is judgment. You still need to understand what all of it does.

If the code is too hard to read and that is slowing the review, stop and ask the author to clarify before continuing. If you cannot understand it, later readers will not either.

If you are not qualified for part of the change — privacy, security, concurrency, accessibility, internationalization — make sure a qualified reviewer covers that part.

When you are one of several reviewers and you were asked to cover only certain files or certain aspects (design, privacy, security):

- Comment which parts you reviewed.
- Prefer LGTM-with-comments (`review-speed.md`) for your part.
- If your LGTM waits on the other reviewers, say that explicitly, and respond quickly once the change reaches the state you were waiting for.

### Context and credit

The review tool shows a few lines around each edit. Open the whole file. Four added lines inside a 50-line function may mean the function should be split.

Judge the change against the system. A change that makes the system more complex or less tested is a health decrease. Most systems become complex through many small additions, so small complexity still counts. Do not accept a change that degrades code health.

When the author does something well, especially when they address a comment well, say so. Telling an author what they did right is sometimes the more useful mentoring move.

## Agent Guidance

### Do

- Start with design, then functionality and complexity, then tests, names, comments, style, and docs.
- Read every assigned human-written line in the context of the file and the system.
- Keep tests in the same change as the behavior they cover.
- Separate a large reformat from the behavior change.
- Name the files or aspects you actually reviewed when you are not the sole reviewer.

### Do Not

- Do not block on a style preference the style guide does not require.
- Do not accept extra generality for a future requirement that has not arrived.
- Do not accept an explanation that lives only in the review thread when the code itself is unclear. See `review-comments.md`.
- Do not approve a health decrease because each individual addition looks small.

## Checklist

- [ ] Design belongs in this codebase and integrates with the system
- [ ] Behavior matches the intent and is good for users and future editors
- [ ] Edge cases and any concurrency were thought through, not only executed
- [ ] Complexity is what the current problem requires
- [ ] Tests are in this change, fail when the code breaks, and are themselves maintainable
- [ ] Names communicate the concept; comments explain why; docs match user-visible changes
- [ ] Style-guide violations are fixed; other style notes are nits
- [ ] Every assigned human-written line was understood in file context
- [ ] Partial review scope is written on the change
- [ ] At least one specific good practice was acknowledged when one was present

## See Also

- wiki/tier2-core/google-eng-practices/review-standard.md
- wiki/tier2-core/google-eng-practices/navigating-a-cl.md
- wiki/tier2-core/google-eng-practices/small-cls.md
- wiki/tier2-core/code-review-method/overview.md
- wiki/tier2-core/code-review-method/triggers.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md
- wiki/tier1-sources/swebok-v4/ka05-testing.md
- wiki/tier3-working/checklists/code-review.md
- wiki/tier3-working/checklists/testing-review.md

## Source

Google. *What to look for in a code review*. Engineering Practices Documentation. CC BY 3.0. https://google.github.io/eng-practices/review/reviewer/looking-for.html

Local immutable capture: `references/google-eng-practices/review/reviewer/looking-for.md`. Snapshot `3bb3ec25b3b0199f4940b1aa75f0ac5c5753301c`.
