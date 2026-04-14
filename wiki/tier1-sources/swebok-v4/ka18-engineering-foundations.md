# Engineering Foundations (KA18)

> **Tier 1** | Source: SWEBOK V4, Chapter 18 | Authority: immutable

## Summary

Engineering Foundations covers the general engineering principles applicable to software: measurement theory, empirical methods, root cause analysis, and systems thinking. These are the meta-principles that determine how engineers approach problems — how they measure, how they experiment, how they diagnose failures, and how they reason about complex systems with emergent behavior. Without these foundations, engineering degrades into ad-hoc problem-solving driven by intuition rather than evidence.

For agents, this KA provides the reasoning discipline that underlies all decision-making. Measure before optimizing. Use root cause analysis in postmortems. Apply systems thinking before adding a microservice. Beware Goodhart's Law when selecting metrics. These principles prevent common failure modes that are invisible to pure technical thinking.

## Key Concepts

### Measurement Theory

**Scales of measurement**: The type of scale determines what mathematical operations are valid.
- **Nominal**: Categories without order (bug type: UI, logic, data). Valid: counting, mode. Invalid: mean, ordering.
- **Ordinal**: Ordered categories without fixed intervals (severity: low/medium/high). Valid: median, ordering. Invalid: mean (the interval between "low" and "medium" is not defined).
- **Interval**: Fixed intervals, no true zero (calendar dates, temperature in Celsius). Valid: mean, standard deviation. Invalid: ratios ("twice as warm" is meaningless in Celsius).
- **Ratio**: Fixed intervals with a true zero (lines of code, response time, cost). Valid: all operations including ratios.

**Validity of metrics**:
- **Internal validity**: Does the metric actually measure what we claim it measures? (Story points measure relative complexity, not time)
- **External validity**: Does the metric's meaning generalize beyond the current context? (Coverage % means different things for unit tests vs. integration tests)
- **Construct validity**: Does the metric map to the underlying concept? (Velocity is not productivity — it measures planned work completed, not value delivered)

**Goodhart's Law**: "When a measure becomes a target, it ceases to be a good measure." (Charles Goodhart, 1975; popularized by Marilyn Strathern)

Practical examples:
- Measuring developer productivity by lines of code → developers write verbose, padded code
- Measuring QA by bugs found → testers avoid difficult modules where bugs are hard to find
- Measuring team velocity → teams inflate story point estimates to show "improvement"
- Measuring code coverage % → developers write tests that execute code without asserting behavior

Mitigation: Rotate metrics, use multiple metrics that are hard to simultaneously game, measure outcomes not outputs.

### Empirical Methods

Empirical engineering makes decisions from data produced by controlled observation or experiment rather than intuition or authority.

**Hypothesis formation**: A clear, falsifiable statement about expected system behavior.
- Good: "Adding a database index on user_id will reduce the 90th percentile query latency below 50ms under 100 RPS load"
- Bad: "The database query is probably slow because of missing indexes"

**Controlled experiments**: Hold all variables constant except the one being tested. In software:
- A/B tests for user-facing feature impact
- Load tests for performance hypothesis validation
- Fault injection for resilience hypothesis validation

**Data collection**: Instrument before experimenting. You cannot measure what you did not instrument.

**Analysis**: Apply appropriate statistical methods (see KA17). Report results with confidence intervals, not just point estimates. Acknowledge sample size limitations.

**A/B testing as engineering experiment**: A properly run A/B test is a controlled experiment. Pre-register the hypothesis, determine sample size via power analysis, run to completion, analyze the pre-specified primary metric. Do not peek at results mid-run and stop early when you see a favorable result — this is p-hacking and produces false positives.

### Root Cause Analysis

Root cause analysis (RCA) identifies the underlying cause of a failure, not just its symptoms. Treating symptoms without addressing root causes leads to recurrence.

**5 Whys**: Recursively ask "why" until you reach a systemic root cause (typically 4-6 iterations before the root cause is structural rather than individual).

Example:
1. Why did the service go down? → Database connection pool was exhausted
2. Why was the connection pool exhausted? → A slow query was holding connections open
3. Why was the query slow? → A new index was missing after the schema migration
4. Why was the index missing? → The migration script was not reviewed by a DBA
5. Why was the migration script not reviewed? → The review process did not include DBA sign-off for schema changes

Root cause: The code review process does not require DBA review for schema migrations.
Fix: Update the review checklist, not just "add the index."

**Fishbone diagram (Ishikawa)**: Visualizes categories of potential causes branching from the problem statement. Categories for software: People, Process, Tools, Technology, Environment, Measurement. Useful for structured brainstorming when the 5 Whys leads to premature conclusions.

**Fault tree analysis (FTA)**: Top-down deductive analysis starting from an undesired event (the top event) and working backward through combinations of failures (AND/OR gates) that could cause it. Used in safety-critical systems to calculate the probability of catastrophic failures.

**Blameless postmortem**: RCA in operations. The goal is to understand what systemic factors allowed the failure to occur and to improve the system, not to assign blame to individuals. Blameless culture is a prerequisite — if individuals fear punishment, they will conceal contributing factors, making systemic improvement impossible.

### Systems Thinking

Systems thinking recognizes that software systems are complex adaptive systems with emergent behaviors that cannot be predicted by analyzing components in isolation.

**Feedback loops**:
- **Reinforcing (positive) feedback**: An effect that amplifies the original cause. Example: cache hit rate increases as the cache warms up, which reduces latency, which increases throughput, which... Reinforcing loops can produce runaway growth or collapse.
- **Balancing (negative) feedback**: An effect that dampens the original cause. Example: autoscaling responds to load by adding capacity, which reduces load per instance. Balancing loops produce stability — but with delays, they can oscillate (adding capacity after the load peak has passed).

**Emergent behavior**: System-level properties that are not present in any individual component. Distributed systems exhibit emergent failures (cascading failures, thundering herds, split-brain) that are invisible when components are tested in isolation. Test the system under realistic conditions, not just individual components.

**Unintended consequences**: Every change to a complex system perturbs the equilibrium in ways that may not be predictable. Adding a cache reduces database load but introduces cache invalidation complexity and the possibility of serving stale data. Microservices decomposition reduces deployment coupling but introduces network latency, distributed transactions, and operational complexity. Always ask: what does this change make worse?

**Relevant to microservices design**: Each service boundary is a feedback loop interruption. Services that communicate frequently over the network may be better merged into a single service — the communication overhead may exceed the operational independence benefit. Apply systems thinking before adding service boundaries.

### Modeling and Abstraction

**"All models are wrong, but some are useful"** (George Box): A model that captured every detail of the system it represents would be the system itself. Useful models simplify by omitting irrelevant detail. The challenge is choosing what to omit.

**Appropriate abstraction levels**: A model at the wrong level of abstraction creates confusion.
- Too abstract: "The system processes user requests" — tells nothing actionable
- Too detailed: Full class diagram with all fields and methods — misses the forest for the trees
- Appropriate: Component diagram showing the three major services and their integration points — actionable for architecture review

**Leaky abstractions**: (Joel Spolsky's Law of Leaky Abstractions) All non-trivial abstractions leak — the underlying complexity eventually surfaces. TCP looks like a reliable stream, but retransmits leak into latency spikes. ORMs look like objects, but SQL query plans leak into N+1 problems. Design systems knowing that abstractions will eventually leak.

### Engineering Trade-offs

**No Free Lunch**: Every optimization improves some dimension at the cost of another. There is no universally optimal solution.

Common trade-offs in system design:
- Consistency vs. availability (CAP theorem)
- Latency vs. throughput
- Storage cost vs. query performance (indexing)
- Coupling vs. operational independence (microservices)
- Simplicity vs. flexibility
- Safety vs. velocity

**The cost of every decision is real**: When choosing a solution, explicitly name what it makes worse. "We chose eventual consistency, which means users may see stale data for up to 30 seconds after an update. The mitigation is..."

## Agent Guidance

### Do
- Measure current behavior before proposing optimization — baseline first
- Use 5 Whys in incident postmortems to reach systemic root causes, not just immediate fixes
- Apply Goodhart's Law scrutiny to every metric selection — ask "how could this be gamed?"
- Use systems thinking before adding service boundaries — identify the feedback loops being interrupted
- Name the trade-offs explicitly when recommending a design: "This improves X at the cost of Y"
- Write hypotheses before running performance experiments; report results with confidence intervals

### Do Not
- Optimize without a measured baseline — "it feels slow" is not an engineering basis for optimization
- Stop the 5 Whys at the first technical answer — continue until you reach a systemic/process cause
- Select metrics without asking what they would look like if being gamed
- Add microservice boundaries without analyzing the emergent complexity introduced
- Present trade-offs as one-sided wins — all decisions have costs

## Checklist
- [ ] Performance baseline measured before optimization work begins
- [ ] Postmortems use 5 Whys to reach systemic root causes
- [ ] Metric selection reviewed for Goodhart's Law vulnerability
- [ ] Architecture decisions include explicit trade-off analysis
- [ ] A/B tests have pre-registered hypotheses and predetermined sample sizes
- [ ] Systems thinking applied to proposed architectural changes (emergent effects identified)
- [ ] Abstraction levels in models chosen to match the audience and decision being supported

## See Also
- `wiki/tier1-sources/swebok-v4/ka09-engineering-management.md`
- `wiki/tier1-sources/swebok-v4/ka06-operations.md`
- `wiki/tier1-sources/swebok-v4/ka12-quality.md`
- `wiki/tier1-sources/swebok-v4/ka17-mathematical-foundations.md`

## Source
IEEE Computer Society. *Guide to the Software Engineering Body of Knowledge (SWEBOK), Version 4*. Chapter 18: Engineering Foundations. IEEE Press, 2024.
