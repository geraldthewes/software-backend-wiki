# Writing Good Change Descriptions (Tier 2)

> **Tier 2** | Source: Google Engineering Practices, `review/developer/cl-descriptions.md` | Authority: established | Derives From: SWEBOK KA8, KA12

## Summary

A change description is a public record. It has to say **what** changed, in enough detail that a reader need not open the diff to know the shape of it, and **why** it changed: the context the author had, and the decisions that are not visible in the source. That text stays in version-control history and is how later engineers find the change and decide whether they may move it.

Code can show what the software does. It often cannot show why the software exists. A description that carries the why is what lets a later reader handle that fence. The first line is also the history summary people skim.

## Key Concepts

### Shape

| Part | Rule |
|------|------|
| **First line** | A short summary of specifically what this change does. A complete sentence in the imperative, as an order: "Delete the FizzBuzz RPC and replace it with the new system." Not "Deleting…". Followed by a blank line. It must stand alone in a history listing. |
| **Body** | The details: the problem, why this approach, shortcomings, bug numbers, benchmark results, design-doc links, and enough context that the change makes sense even if those links rot. |
| **Rest of the description** | Does not have to stay in the imperative. |

Clarity for the future reader outranks cleverness or brevity. Even a small change gets a description that places it in context.

### Bad and good

Inadequate first lines name neither the defect nor the action: "Fix bug.", "Fix build.", "Add patch.", "Moving code from A to B.", "Phase 1.", "Add convenience functions.", "kill weird URLs."

A useful description's first line says the action. The body says the problem, why this solution, anything specific about the implementation, and — for a refactor — why the intermediate step exists and what remains unfinished. Shortcomings belong in the body. If the approach is a compromise on the way to a larger goal, say so.

### Tags

Tags (`[tag]`, `#tag`, `tag:`) are optional labels, by tool or team convention. Keep them out of the first line unless they are short. Long or numerous tags belong in the body so they do not hide the summary.

### Generated descriptions

Tool-generated changes follow the same rules: a short standalone first line, and a body a reviewer and a later searcher can use.

### Before submit

Review changes the diff. Re-read the description before submitting and fix it if it no longer matches the change.

### Conventional Commits in this wiki

The first line follows Conventional Commits: `type(scope): imperative summary`, lowercase description, no trailing period. See `wiki/tier2-core/conventional-commits/overview.md`. Google's imperative-what rule is the summary after the type prefix. Google's body rule and the Conventional Commits body rule agree: the body explains why. Put breaking-change footers where Conventional Commits requires them. Do not drop the type prefix in order to imitate an unprefixed Google first line.

## Agent Guidance

### Do

- Write the first line so a history search shows what this change did without opening it.
- Put decisions, shortcomings, bug links, and "why this approach" in the body.
- Copy enough context into the description that a dead link does not make the change opaque.
- Re-read the description after the last review round, before merge.
- Keep the Conventional Commits subject format on the first line.

### Do Not

- Do not use a first line that could describe any change ("Fix bug", "Phase 1").
- Do not leave the why only in the diff or only in a review comment.
- Do not let a long tag crowd out the summary.
- Do not submit a description that still describes an earlier revision of the diff.

## Checklist

- [ ] First line is a standalone imperative summary of what this change does
- [ ] First line matches Conventional Commits (`type(scope): summary`)
- [ ] A blank line separates the subject from the body
- [ ] Body states why, including non-obvious decisions and known shortcomings
- [ ] External links are backed by enough local context to survive link rot
- [ ] Tags on the first line are short or absent
- [ ] Description was re-checked against the final diff

## See Also

- wiki/tier2-core/google-eng-practices/overview.md
- wiki/tier2-core/google-eng-practices/small-cls.md
- wiki/tier2-core/google-eng-practices/navigating-a-cl.md
- wiki/tier2-core/conventional-commits/overview.md
- wiki/tier2-core/conventional-commits/specification.md
- wiki/tier2-core/engineering-playbook/source-control.md
- wiki/tier1-sources/swebok-v4/ka08-config-management.md
- wiki/tier3-working/checklists/pre-commit.md

## Source

Google. *Writing good CL descriptions*. Engineering Practices Documentation. CC BY 3.0. https://google.github.io/eng-practices/review/developer/cl-descriptions.html

The upstream page cites Chesterton's fence via *Software Engineering at Google*, chapter 3.

Local immutable capture: `references/google-eng-practices/review/developer/cl-descriptions.md`. Snapshot `3bb3ec25b3b0199f4940b1aa75f0ac5c5753301c`.
