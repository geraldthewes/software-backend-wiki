# Agile Development Practices (Tier 2)

> **Tier 2** | Source: Microsoft ISE Engineering Fundamentals Playbook | Authority: established | Derives From: SWEBOK KA9, KA10

## Summary

Agile development is the delivery model for cross-functional software teams ("Crews" in Microsoft ISE terminology). It prioritizes responding quickly to change, delivering working software iteratively, maintaining continuous customer involvement, and valuing individuals and interactions over process documentation.

For a coding agent, agile context matters because it defines the unit of work (user story/sprint), the quality gates (Definition of Done), and the team norms (working agreements) that govern every code contribution. A change that satisfies acceptance criteria and passes DoD is complete; one that does not is not, regardless of technical correctness.

## Key Concepts

### The 8 Essential Elements

| Element | What It Is | Agent Relevance |
|---------|-----------|----------------|
| **1. Shared Backlog** | Single prioritized list of work items in a shared tool (Azure DevOps, GitHub, Jira) | Work items provide acceptance criteria; always link PRs to backlog items |
| **2. Iterative Cycles** | Time-boxed sprints with defined goals | PRs should advance sprint goals, not introduce scope outside the sprint |
| **3. Definition of Ready (DoR)** | Criteria a work item must meet before a developer picks it up | Do not start items lacking acceptance criteria or design clarity |
| **4. Definition of Done (DoD)** | Criteria every item must satisfy before it can be closed | Code + tests + docs + CI green + review approved = done |
| **5. Progress Tracking** | Sprint boards, daily standups, burndown charts | Make work visible; update work items as code merges |
| **6. Retrospectives** | Regular structured reflection on process — what worked, what didn't | Surface blockers and technical debt; don't defer process problems |
| **7. Clear Roles** | Dev Lead, TPM, Process Lead, contributors — distinct responsibilities | Know who makes design decisions vs. prioritization decisions |
| **8. Working Agreements** | Team-defined norms for collaboration, communication, and review | Working agreements override individual preferences on process questions |

### Definition of Ready (DoR)

A work item is ready to enter a sprint when it satisfies all of these:

- [ ] Acceptance criteria are written and agreed upon
- [ ] Any required design (architecture, data model, API contract) is done and reviewed
- [ ] Dependencies on other teams or external systems are identified and unblocked
- [ ] The item is sized (estimated) and fits within a single sprint
- [ ] The item is understood by at least one developer who can implement it

> Design work is backlog work. If a story needs design, schedule the design as a separate item *before* the implementation item.

### Definition of Done (DoD)

A work item is done when all of the following are true:

- [ ] Code implements the acceptance criteria
- [ ] Unit and integration tests written and passing
- [ ] CI pipeline passes (lint, type check, tests, security scan)
- [ ] Code reviewed and approved per team policy
- [ ] Documentation updated (API docs, README, ADR if architecture changed)
- [ ] Work item closed and linked to the merged PR
- [ ] No new technical debt introduced without a corresponding backlog item

### Backlog Management

The backlog is a living artifact, not a fixed requirements document.

| Practice | Why |
|----------|-----|
| **Continuous refinement** | Refine backlog throughout the sprint, not only in scheduled meetings; items should always have a "ready" buffer 1–2 sprints ahead |
| **Acceptance criteria first** | Write acceptance criteria before estimation — criteria reveal ambiguity that changes size |
| **Split large stories** | Stories too large to complete in one sprint must be split before entering a sprint |
| **Capture technical debt explicitly** | When shortcuts are made under time pressure, immediately create a backlog item for the debt; do not normalize hidden debt |

**Technical debt policy:**
> "Technical debt is mostly due to shortcuts made in the implementation as well as the future maintenance cost as the natural result of continuous improvement." — Playbook

Avoid shortcuts when possible. When unavoidable, log the debt item immediately with enough context for future resolution. Never treat debt as free — it has a carrying cost in velocity and risk.

### Working Agreements

Working agreements are team-defined norms that eliminate recurring decision overhead. Common areas:

| Area | Example Agreement |
|------|------------------|
| Core hours | "Team available for synchronous collaboration 10am–3pm local time" |
| PR review SLA | "Reviews completed within one business day of request" |
| Branch strategy | "Feature branches off main; squash-merge on PR completion" |
| Commit convention | "All commits follow Conventional Commits spec" |
| Definition of Done | Team-specific DoD checklist (see above) |
| Meeting norms | "Cameras on for design sessions; standups async-first" |
| AI tooling | "AI-generated code reviewed with same rigor as human-written code" |

Working agreements must be documented (not just verbally agreed), regularly revisited in retrospectives, and updated as the team evolves.

### Sprint Ceremonies

| Ceremony | Cadence | Purpose | Output |
|----------|---------|---------|--------|
| Sprint Planning | Start of sprint | Select and commit to sprint backlog | Sprint goal, committed items |
| Daily Standup | Daily | Identify blockers, coordinate | Updated board, unblocked work |
| Sprint Review | End of sprint | Demo working software to stakeholders | Feedback, accepted items |
| Retrospective | End of sprint | Reflect on process, not product | Action items for next sprint |
| Backlog Refinement | Mid-sprint (ongoing) | Ensure future items meet DoR | Ready items 1–2 sprints ahead |

### AI Tooling Integration

The playbook acknowledges AI tools (code generation, planning assistance) within agile frameworks:

- Maintain human oversight and review for all AI-generated artifacts.
- Favor shorter validation cycles — AI output requires more frequent review checkpoints.
- AI-assisted planning tools can accelerate story drafting but DoR/DoD standards still apply.
- Track AI-generated code the same as human-written code: it must pass all CI checks and code review.

## Agent Guidance

### Do

- Always verify a work item has acceptance criteria before implementing it.
- Link every PR to its work item and close the item when the PR merges.
- Create a backlog item for any technical debt introduced, even under time pressure.
- Respect working agreements as team policy — do not unilaterally deviate.
- Update sprint boards as work progresses — visibility is a team responsibility.

### Do Not

- Do not start work that does not meet DoR (missing acceptance criteria, unresolved design questions).
- Do not close a work item before all DoD criteria are met.
- Do not scope-creep a sprint by adding unplanned work without team agreement.
- Do not treat a retrospective action item as optional — if a process problem is identified, it must be tracked.

## Checklist

- [ ] Work item has written acceptance criteria before implementation starts
- [ ] Implementation satisfies all acceptance criteria
- [ ] All DoD criteria met: tests, CI, review, docs, work item updated
- [ ] Any technical debt introduced has a corresponding backlog item
- [ ] PR linked to work item

## See Also

- wiki/tier2-core/engineering-playbook/overview.md
- wiki/tier2-core/engineering-playbook/source-control.md
- wiki/tier1-sources/swebok-v4/ka09-engineering-management.md
- wiki/tier1-sources/swebok-v4/ka10-process.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md
- wiki/tier3-working/checklists/code-review.md

## Source

Microsoft ISE Engineering Fundamentals Playbook — Agile Development.
https://microsoft.github.io/code-with-engineering-playbook/agile-development/
https://microsoft.github.io/code-with-engineering-playbook/agile-development/backlog-management/
