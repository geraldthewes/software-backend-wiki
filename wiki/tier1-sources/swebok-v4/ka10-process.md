# Software Engineering Process (KA10)

> **Tier 1** | Source: SWEBOK V4, Chapter 10 | Authority: immutable

## Summary

Software Engineering Process covers the structured set of activities, constraints, and resources that produce a software product. A process is not a bureaucratic formality — it is the mechanism by which teams make engineering work repeatable, predictable, and improvable. Without a defined process, quality depends entirely on individual heroics. With a well-tailored process, quality emerges from the system even as team members change.

SWEBOK V4 integrates Agile and DevOps practices as first-class process elements rather than alternatives to "real" software engineering. For agents, this means following the team's defined process is not optional — it is how work gets done. Agents should respect the Definition of Done, surface impediments, and prefer continuous delivery over batch delivery.

## Key Concepts

### Process Definition

A software process is a set of:
- **Activities**: Defined work tasks (requirements elicitation, design, code review, testing)
- **Artifacts**: Inputs and outputs of activities (specifications, code, test reports, deployment manifests)
- **Roles**: Who is responsible for each activity
- **Constraints**: Policies, standards, and regulations that govern how activities are performed
- **Resources**: People, tools, time, and infrastructure required

A process is *defined* when it is documented. It is *followed* when behavior matches the documentation. It is *measured* when outcomes are tracked against goals. It is *improved* when measurement data drives changes.

### Process Models

**Waterfall**: Sequential phases (requirements → design → implementation → testing → deployment). Each phase must complete before the next begins. Low flexibility to accommodate change. Appropriate only when requirements are fully stable at project start (rare).

**Iterative**: Cycles through phases repeatedly, building up the system incrementally. Each iteration produces a partial but functional system. Requirements can evolve between iterations.

**Incremental**: Delivers the system in functional slices (increments), each of which adds new capability. Users can begin using early increments while later ones are still in development.

**Spiral**: Risk-driven model that evaluates risks at each cycle before committing to the next phase. Appropriate for high-risk, large-scale systems where risk management drives the schedule.

**Agile**: Umbrella term for iterative, incremental approaches that prioritize working software, customer collaboration, and responding to change over documentation and plan following. Encompasses Scrum, Kanban, XP, and others.

### Agile Fundamentals

**The Agile Manifesto** (2001) — four values:
1. **Individuals and interactions** over processes and tools
2. **Working software** over comprehensive documentation
3. **Customer collaboration** over contract negotiation
4. **Responding to change** over following a plan

The right-hand items have value; the left-hand items are valued *more*.

**12 Principles** (selected, most relevant to agents):
- Deliver working software frequently (weeks, not months)
- Welcome changing requirements, even late in development
- Working software is the primary measure of progress
- Simplicity — the art of maximizing the amount of work not done — is essential
- The best architectures emerge from self-organizing teams
- Regularly reflect on how to become more effective and adjust behavior accordingly

**Scrum** (most widely adopted Agile framework):
- **Sprints**: Time-boxed iterations of 1-4 weeks. Each sprint produces a potentially shippable increment.
- **Roles**: Product Owner (what to build), Scrum Master (process health), Development Team (how to build it)
- **Ceremonies**: Sprint Planning, Daily Standup, Sprint Review, Sprint Retrospective
- **Artifacts**: Product Backlog, Sprint Backlog, Increment

**Kanban**: Flow-based approach. Work items move through defined stages (columns) on a board. WIP (work in progress) limits are enforced at each stage to surface bottlenecks. No fixed iterations — continuous flow.

**Extreme Programming (XP)**: Emphasizes engineering practices over process ceremonies. Core practices: test-driven development, pair programming, continuous integration, refactoring, small releases, simple design.

### DevOps Integration (V4)

SWEBOK V4 recognizes DevOps as a process capability, not just a toolchain:
- **Continuous delivery culture**: The team owns the software from commit to production. No handoff to a separate "ops" team.
- **Feedback loops**: Fast feedback from automated tests, staging deployments, and production monitoring informs the next iteration. Shorter feedback loops reduce the cost of defects.
- **Value stream mapping**: Visualize the entire path from idea to production. Identify and eliminate waste (wait time, rework, manual steps). The value stream is the unit of optimization, not the team.

### Process Improvement

**CMMI (Capability Maturity Model Integration)** levels:
- Level 1 — Initial: Ad hoc, success depends on individuals
- Level 2 — Managed: Project-level processes defined and followed
- Level 3 — Defined: Organizational processes standardized and documented
- Level 4 — Quantitatively Managed: Processes measured and controlled using data
- Level 5 — Optimizing: Continuous process improvement driven by quantitative feedback

Most teams operate effectively at Level 2-3. Higher CMMI levels add significant overhead and are appropriate for regulated domains (aerospace, medical devices).

**Retrospectives**: The Agile mechanism for process improvement. Regular reflection on: what went well, what did not, what to do differently. Action items must be specific, assigned, and tracked — otherwise the retrospective is theater.

### Process Tailoring

No single process fits all contexts. Effective process tailoring considers:
- **Team size**: A 3-person team needs far less ceremony than a 50-person team
- **Domain risk**: Safety-critical systems (medical, aviation) require more rigor and documentation than internal tools
- **Requirements stability**: Stable requirements favor plan-driven processes; volatile requirements favor Agile
- **Regulatory environment**: Compliance requirements may mandate specific documentation and audit trails

### Definition of Done

The Definition of Done (DoD) is the team's explicit agreement on what conditions must be satisfied before a work item is considered complete. A strong DoD prevents the "90% done forever" trap.

Example DoD:
- All acceptance criteria pass
- Unit and integration tests written and passing
- Code reviewed and approved
- Security scan clean
- Documentation updated
- Deployed to staging and smoke-tested
- No known high-severity defects outstanding

## Agent Guidance

### Do
- Follow the team's defined process — adapt your behavior to the agreed workflow, not your preferences
- Respect the Definition of Done — do not declare work complete until all DoD criteria are satisfied
- Surface impediments to the process as soon as they are discovered; do not work around them silently
- Prefer continuous delivery: small, frequent changes reduce risk and accelerate feedback
- Contribute to retrospectives with specific, actionable observations
- Estimate conservatively — Agile velocity is calibrated by actuals, not aspirations

### Do Not
- Skip process steps under schedule pressure — this is when process matters most
- Declare a story "done" when acceptance criteria are partially met
- Ignore failing tests in the CI pipeline on the premise of "fixing it later"
- Work in large batches — batch size is inversely proportional to feedback speed and directly proportional to risk

## Checklist
- [ ] Team's process is documented and understood
- [ ] Definition of Done is defined, visible, and enforced
- [ ] Sprint/iteration cadence is established
- [ ] CI/CD pipeline is part of the Definition of Done
- [ ] Retrospectives are scheduled and action items tracked
- [ ] Process is tailored to team size, domain risk, and regulatory context
- [ ] Feedback loops from production to development are defined

## See Also
- `wiki/tier1-sources/swebok-v4/ka09-engineering-management.md`
- `wiki/tier1-sources/swebok-v4/ka06-operations.md`
- `wiki/tier1-sources/swebok-v4/ka12-quality.md`

## Source
IEEE Computer Society. *Guide to the Software Engineering Body of Knowledge (SWEBOK), Version 4*. Chapter 10: Software Engineering Process. IEEE Press, 2024.
