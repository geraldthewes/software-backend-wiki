# Software Engineering Management (KA09)

> **Tier 1** | Source: SWEBOK V4, Chapter 9 | Authority: immutable

## Summary

Software Engineering Management covers the planning, directing, coordinating, measuring, and controlling activities that ensure software projects deliver value within constraints of scope, schedule, cost, and quality. Management is not administration — it is the engineering discipline of making good decisions under uncertainty using structured techniques. Poor management is one of the top causes of project failure, independent of the technical quality of the code produced.

For agents, this KA establishes boundaries: agents must flag when work exceeds estimates, report risks honestly, and escalate rather than silently absorb scope changes. An agent that quietly accepts unlimited scope without surfacing the implications is providing misleading information to its principal — which is a professional ethics violation as much as an engineering failure.

## Key Concepts

### Management Activities

The five core management activities in software engineering:
1. **Planning**: Defining scope, schedule, resources, and risk mitigation before work begins
2. **Directing**: Guiding the team's day-to-day work in accordance with the plan
3. **Coordinating**: Ensuring dependent teams, systems, and stakeholders stay aligned
4. **Measuring**: Collecting data on progress, quality, and performance against the plan
5. **Controlling**: Comparing actuals to the plan and taking corrective action when they diverge

### Estimation Techniques

Estimation is inherently uncertain. The goal is calibrated uncertainty, not false precision.

**Function Points**: Count inputs, outputs, user interactions, internal files, and external interfaces; weight by complexity. Language-independent, suitable for early estimation before design.

**Story Points**: Relative sizing of user stories based on complexity, effort, and uncertainty. Calibrated through team velocity over multiple sprints. Not transferable between teams.

**COCOMO II**: Algorithmic model based on lines of code (or function points) with calibration factors for team capability, product complexity, and process maturity. Useful for large projects with historical data.

**Planning Poker**: Team members independently estimate using a card deck (Fibonacci: 1, 2, 3, 5, 8, 13, 21). Outliers discuss their reasoning. Reduces anchoring bias and surfaces hidden assumptions.

**Analogical Estimation**: "This task is similar to X, which took Y days." Requires a maintained record of past task actuals. More accurate than parametric models when analogues are good.

**Cone of Uncertainty**: Estimation accuracy improves as the project progresses. At project start, actual cost may be 4x the estimate (in either direction). At detailed design, the range narrows to ±25%. Stakeholders must understand this uncertainty — presenting early estimates as commitments is dishonest.

### Risk Management

**Risk identification**: What could go wrong? Sources: technical complexity, unclear requirements, key person dependencies, third-party integrations, regulatory changes, infrastructure instability.

**Risk assessment**: Evaluate each risk on two axes:
- **Probability**: How likely is this to occur? (Low / Medium / High)
- **Impact**: What is the consequence if it occurs? (Low / Medium / High)
- **Risk exposure** = Probability × Impact

**Mitigation strategies**:
- **Avoid**: Change the plan to eliminate the risk
- **Transfer**: Shift the impact to another party (insurance, vendor SLA)
- **Mitigate**: Reduce probability or impact through specific actions
- **Accept**: Acknowledge the risk and prepare a contingency plan

**Contingency planning**: For accepted risks, define in advance: "If X happens, we will do Y."

### Project Tracking

**Velocity**: The average number of story points completed per sprint. Used for forecasting future delivery.

**Burn-down chart**: Shows remaining work (story points or tasks) over time. A flat or rising burn-down is a signal that the team is blocked or scope is growing.

**Cumulative flow diagram**: Shows the number of items in each state (backlog, in progress, done) over time. Widening "in progress" bands signal work-in-progress (WIP) problems.

**Milestone tracking**: Agreed checkpoints (alpha, beta, release candidate) with defined criteria. Missing a milestone without an updated plan is a project control failure.

### Stakeholder Management

**Communication plan**: Define who needs what information, at what frequency, in what format. Stakeholders who receive surprises stop trusting the team.

**Status reporting**: Regular reports that cover: what was completed, what is in progress, risks/impediments, and next steps. Honest reporting of problems early enables corrective action. Late disclosure of problems eliminates options.

### Team Structure and Conway's Law

**Conway's Law**: "Organizations that design systems are constrained to produce designs that are copies of the communication structures of those organizations." (Melvin Conway, 1968)

Implications:
- A team organized around front-end, back-end, and database will produce a three-tier monolith
- A team organized around product domains will produce services aligned to those domains (microservices)
- Architectural decisions and team topology decisions are inseparable — changing one without changing the other produces friction
- **Inverse Conway Maneuver**: Design the target architecture first, then reorganize teams to match it

### Measurement: GQM Framework

**Goal-Question-Metric (GQM)** prevents metric selection by gut feeling:
1. **Goal**: What are you trying to achieve or understand? ("Improve release reliability")
2. **Question**: What questions must you answer to know if you're achieving the goal? ("How often do releases fail? How long does recovery take?")
3. **Metric**: What data answers each question? ("Deployment success rate, MTTR for failed deployments")

**Vanity metrics vs. actionable metrics**:
- Vanity: "We have 1M downloads" — feels good, doesn't drive decisions
- Actionable: "Error rate increased 0.3% after the last deployment" — drives a specific action

**Goodhart's Law**: When a measure becomes a target, it ceases to be a good measure. Teams optimize for the metric rather than the underlying goal (e.g., increasing story point velocity by inflating estimates).

## Agent Guidance

### Do
- Flag clearly when a new requirement or scope addition will exceed the original estimate by more than 20%
- Use GQM to select metrics — justify every metric with a goal and a question
- Apply Conway's Law thinking when proposing system decomposition — consider how teams are structured
- Provide estimation ranges, not point estimates, and explain the cone of uncertainty
- Surface risks proactively; do not wait to be asked

### Do Not
- Present early estimates as commitments
- Accept unlimited scope silently — surface the trade-off explicitly
- Choose metrics because they are easy to measure rather than because they answer a meaningful question
- Report only good news; honest reporting of problems is a professional obligation
- Design system architecture without considering the team structure that will maintain it

## Checklist
- [ ] Scope defined and agreed before estimation
- [ ] Estimates provided as ranges, not point values
- [ ] Risk register created and maintained for the project
- [ ] Communication plan defined — who gets what, at what cadence
- [ ] Metrics selected using GQM framework
- [ ] Velocity baseline established before making delivery commitments
- [ ] Conway's Law implications considered in architecture decisions
- [ ] Status reports issued on schedule, including problems not just progress

## See Also
- `wiki/tier1-sources/swebok-v4/ka10-process.md`
- `wiki/tier1-sources/swebok-v4/ka15-economics.md`
- `wiki/tier1-sources/swebok-v4/ka14-professional-practice.md`
- `wiki/tier1-sources/swebok-v4/ka01-requirements.md`

## Source
IEEE Computer Society. *Guide to the Software Engineering Body of Knowledge (SWEBOK), Version 4*. Chapter 9: Software Engineering Management. IEEE Press, 2024.
