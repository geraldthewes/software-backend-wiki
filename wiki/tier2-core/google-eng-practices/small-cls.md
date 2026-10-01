# Small Changelists (Tier 2)

> **Tier 2** | Source: Google Engineering Practices, `review/developer/small-cls.md` | Authority: established | Derives From: SWEBOK KA8, KA12

## Summary

The right size for a change is **one self-contained change that does one thing**. Small changes are reviewed faster and more thoroughly, carry fewer bugs, waste less work if rejected, merge and roll back more easily, are easier to design well, and let the author keep coding while one slice is in review.

Reviewers may reject a change for size alone. Splitting after the fact is harder than writing the small change first. Authors should err toward too small. Reviewers rarely complain that a change was too small.

## Key Concepts

### What counts as small

| Test | Pass condition |
|------|----------------|
| Scope | One thing. Usually one part of a feature, not the whole feature. |
| Tests | Related test code is in this change. See below. |
| Self-contained | Everything the reviewer needs, except future work, is in the change, its description, the existing codebase, or a change they already reviewed. |
| Safe to land | Users and developers still have a working system after this change is submitted. |
| Not cryptic | The change is not so small that its implications disappear. A new API includes a usage in the same change, so the API is not dead on arrival and reviewers can see how it is called. |

There is no hard line count. About **100 lines** is usually reasonable. About **1000 lines** is usually too large. The reviewer's judgment decides. Spread matters: 200 lines in one file may be fine; the same 200 lines across 50 files is usually too large. The author has context the reviewer lacks. When unsure, write a smaller change than you think you need.

The Tier 3 SmartBear 200–400 LOC figure is a defect-discovery research bound. It does not replace this page. See `overview.md`.

### When a large change is acceptable

- Deleting an entire file can count as a one-line review. It does not take long to review.
- A change produced by an automatic refactoring tool the reviewer trusts can be larger. The reviewer's job is to verify that the change is wanted. Merge and test caveats still apply.
- If every split has failed — the source treats this as extremely rare — get the reviewers' consent **before** sending the large change. Expect a long review, watch for bugs, and be extra careful with tests.

### How to keep moving while a change is in review

Do not sit idle waiting for approval before starting the next slice. Options: another project in parallel, a reviewer who can respond immediately, an in-person review, pair programming, or a split that lets the next slice start at once.

### Splitting strategies

Think about the split before coding.

| Strategy | What you do |
|----------|-------------|
| **Stack** | Send one small change, then immediately write the next change on top of it. |
| **By files** | Separate self-contained groups that need different reviewers. Example: a schema change and the code that uses it, reviewed in parallel, schema submitted first. Tell each reviewer about the other change. A code change and the config or experiment that turns it on can split the same way, and the config is easier to roll back. |
| **Horizontal** | Isolate layers with shared code or stubs (a shared API contract, a client stub) so layers move independently. |
| **Vertical** | Slice a full-stack feature into smaller full-stack features that proceed in parallel (multiplication and division as separate tracks, even if they share some styling or validation). |
| **Grid** | Combine horizontal and vertical: each layer × feature cell is its own change, built from the model upward. |
| **Refactor apart** | Renames and moves go in a different change from the bugfix or feature. A tiny local rename may stay inside the feature change. If the refactor makes the review harder, it is its own change. |

### Tests and the build

Tests are part of "small," and small means one focused concept, not a line-count trick. A change that adds or changes logic includes new or updated tests. A pure refactor should already be covered; if it is not, add the tests.

These test-only changes may land first, like refactors:

- New tests for code that is already submitted, so a later refactor can prove behavior did not change.
- Refactoring the tests themselves (helpers).
- Larger test-framework work, such as an integration harness.

Stacked changes must leave the system working after each one lands. Otherwise the team is broken between submissions.

## Agent Guidance

### Do

- Split a feature into one-thing changes before writing a large diff.
- Include the tests that belong to the behavior, and a usage when adding an API.
- Stack or slice so review wait does not stop the next slice.
- Put behavioral refactors in their own change.
- Ask for a split when reviewing a change you cannot navigate.

### Do Not

- Do not mix a wide reformat or a rename-and-move with a functional fix.
- Do not land a slice that breaks the build for the slices that have not merged yet.
- Do not treat "100 lines" as a hard quota that justifies cutting tests out of the change.
- Do not send a knowingly huge change without prior reviewer consent.

## Checklist

- [ ] The change does one self-contained thing
- [ ] Related tests are in the change, or a listed exception applies (emergency, prior test-only change, trusted generated refactor)
- [ ] A new API is exercised in the same change
- [ ] The system still works if this change lands alone
- [ ] Refactors that would obscure the behavior change were split out
- [ ] Stacked changes were ordered so intermediate states are green
- [ ] Size was judged by reviewer comprehension, not only by line count

## See Also

- wiki/tier2-core/google-eng-practices/overview.md
- wiki/tier2-core/google-eng-practices/navigating-a-cl.md
- wiki/tier2-core/google-eng-practices/review-speed.md
- wiki/tier2-core/google-eng-practices/what-to-look-for.md
- wiki/tier2-core/google-eng-practices/cl-descriptions.md
- wiki/tier2-core/engineering-playbook/source-control.md
- wiki/tier1-sources/swebok-v4/ka08-config-management.md
- wiki/tier2-core/testing-strategies/test-pyramid.md

## Source

Google. *Small CLs*. Engineering Practices Documentation. CC BY 3.0. https://google.github.io/eng-practices/review/developer/small-cls.html

Local immutable capture: `references/google-eng-practices/review/developer/small-cls.md`. Snapshot `3bb3ec25b3b0199f4940b1aa75f0ac5c5753301c`.
