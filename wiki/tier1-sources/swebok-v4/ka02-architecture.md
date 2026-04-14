# SWEBOK V4 — KA 02: Software Architecture

> **Tier 1** | Source: IEEE SWEBOK V4, KA 02; 12-Factor App; CAP/PACELC | Authority: immutable

## Summary

Software Architecture is the discipline of defining the fundamental structures of a software system — its components, the connectors between them, and the constraints governing their relationships. Architecture addresses the decisions that are difficult or expensive to change once made: the choice of communication style (synchronous vs. asynchronous), the deployment model (monolith vs. microservices), the data consistency model (CP vs. AP), and the quality attribute trade-offs (performance vs. consistency, availability vs. accuracy).

In SWEBOK V4, Architecture is a new standalone Knowledge Area (KA 02), elevated from a sub-topic within the Design KA in V3. This elevation reflects the increasing centrality of architectural decisions in distributed, cloud-native systems: a wrong architectural choice cannot be fixed by good code. For a coding agent, the architecture defines the context within which all construction and design decisions are made. The agent must understand the active architectural style, consult existing Architecture Decision Records (ADRs), and escalate new architectural questions rather than resolving them silently in implementation.

## Key Concepts

### Architecture Definition

Architecture describes a software system along three dimensions:
1. **Components:** The computational units — services, modules, databases, queues, CDNs
2. **Connectors:** The interaction mechanisms — REST calls, gRPC, message queues, shared databases, event streams
3. **Constraints:** The rules governing the interactions — layer dependencies, team ownership boundaries, SLA requirements, data residency rules

Architecture is not the same as design: design deals with the internal structure of individual components (classes, methods, algorithms). Architecture deals with the structure of the system of components. A component can be beautifully designed while the architectural choice to make it synchronously coupled to 12 other services is catastrophically wrong.

---

### New in V4: Why Architecture Became Standalone

In V3, architectural decisions were embedded in the Design KA. V4's separation reflects three industry realities:

1. **Distributed systems are the default:** Microservices, cloud functions, managed databases, and event streaming systems require explicit architectural reasoning that does not fit within the scope of "detailed design"
2. **ADRs need first-class status:** Architectural decisions must be documented, version-controlled, and treated as authoritative — a practice that requires its own knowledge area framework
3. **Quality attributes drive architecture:** Performance, security, scalability, and availability are architectural properties, not design properties. They require systematic analysis at the architectural level (fitness functions, threat models, load tests)

---

### Architectural Styles

An architectural style is a recurring pattern for organizing software systems. Choosing the right style for a given context is one of the most consequential architectural decisions.

| Style | Description | Strengths | Weaknesses | Best For |
|-------|-------------|-----------|------------|----------|
| **Layered (N-tier)** | Horizontal layers (presentation, business, data); each depends only on the layer below | Simple mental model; natural separation of concerns | Can become "lasagna code"; cross-cutting concerns span all layers | Enterprise apps, CRUD services, monoliths |
| **Microservices** | Independently deployable services organized around business capabilities | Independent scaling and deployment; team autonomy; fault isolation | Distributed system complexity; network latency; operational overhead | Large orgs with multiple teams; high-scale services |
| **Event-driven** | Components communicate via events on a message bus; decoupled producers and consumers | High decoupling; natural async processing; audit trail via event log | Eventual consistency complexity; debugging across services is hard | Real-time processing, IoT, audit-heavy domains |
| **Pipe-and-filter** | Processing in sequential stages (filters) connected by channels (pipes) | Composable; each filter is independently testable; easy to add/remove stages | Linear processing only; backpressure management required | ETL pipelines, data transformation, compilers |
| **SOA (Service-Oriented)** | Coarse-grained services with an enterprise service bus (ESB) | Reuse across business units; formal contracts | ESB is a central point of failure; complex governance | Legacy enterprise integration |
| **Serverless (FaaS)** | Functions triggered by events; infrastructure managed by cloud provider | No server management; scales to zero; pay per use | Cold start latency; vendor lock-in; state management complexity | Event-triggered workloads, infrequent or spiky load |

**Selecting an architectural style — key questions:**
- How many teams will own different parts? (Microservices needs team boundaries to justify complexity)
- What is the required deployment frequency? (Independent services allow independent deployment)
- What are the consistency requirements? (Event-driven favors eventual consistency)
- What is the operational maturity? (Microservices requires mature DevOps to not become a maintenance nightmare)

---

### The 4+1 Architectural View Model

A single architectural diagram cannot capture all concerns. The 4+1 model provides five complementary views, each addressing a different stakeholder concern:

| View | What It Shows | Primary Audience | Common Notation |
|------|--------------|-----------------|----------------|
| **Logical** | Classes, packages, and their relationships | Developers; analysts | Class diagrams, package diagrams |
| **Process** | Runtime behavior: processes, threads, concurrency, synchronization | Performance/concurrency engineers | Sequence diagrams, activity diagrams |
| **Development** | Source code organization: packages, modules, build artifacts, layer structure | Developers; build engineers | Package diagrams, component diagrams |
| **Physical** | Deployment: nodes, network topology, containers, cloud regions | Infrastructure; DevOps | Deployment diagrams |
| **+Use Case** | Scenarios that connect the other four views; illustrates key behaviors | All stakeholders | Use case diagrams, scenario descriptions |

Every architecture documentation must include all five views. Omitting any view creates blind spots for specific stakeholder groups.

---

### Architecture Drivers: Quality Attributes

Architecture is primarily determined by quality attribute requirements, not functional requirements. Functional requirements can usually be met by many architectural styles; quality attributes constrain the style.

| Quality Attribute | Definition | Architectural Impact |
|------------------|-----------|---------------------|
| **Performance** | Response time, throughput, resource utilization | Caching strategy, async vs. sync, data store choice |
| **Scalability** | Ability to handle increased load | Stateless services, horizontal scaling, sharding |
| **Availability** | Fraction of time system is operational | Redundancy, failover, circuit breakers, SLO definition |
| **Reliability** | Probability of correct operation over time | Retry logic, idempotency, saga patterns |
| **Security** | Resistance to unauthorized access and data compromise | Zero Trust, mTLS, STRIDE analysis, least privilege |
| **Maintainability** | Ease of understanding and modifying the system | Low coupling, high cohesion, documented ADRs |
| **Portability** | Ability to run on different environments | 12-Factor compliance, containerization, no hardcoded infra |

Quality attributes often trade off against each other (strong consistency vs. availability; security overhead vs. performance). The architectural decision must make these trade-offs explicit and justified.

---

### Architecture Decision Records (ADRs)

An ADR is a short document capturing a significant architectural decision: the context, the decision made, the alternatives considered, and the consequences.

**Why ADRs are Tier 1:**
- They are the living record of why the system is the way it is
- Without them, the next engineer (human or agent) will make the same mistakes or undo good decisions
- They are version-controlled alongside the code, so the architectural history is traceable

**ADR Template:**

```markdown
# ADR-NNN: [Short title — present tense]

## Status
[Proposed | Accepted | Deprecated | Superseded by ADR-NNN]

## Date
YYYY-MM-DD

## Context
[What is the situation? What forces are at play? What problem are we solving?
Be factual and neutral here — this is not the argument for the decision.]

## Decision
[What is the architectural decision we are making?
State it clearly in one or two sentences.]

## Consequences
### Positive
- [What becomes easier or better as a result?]

### Negative
- [What becomes harder or worse? What new problems does this create?]

### Neutral
- [What changes but is neither clearly positive nor negative?]

## Alternatives Rejected
### [Alternative 1 name]
[Why was it rejected? What made the chosen option better for this context?]

### [Alternative 2 name]
[Why was it rejected?]
```

**ADR rules for agents:**
- An agent must read all existing ADRs before making an architectural decision
- An agent must not contradict an accepted ADR without escalating to a human
- When asked to make a new architectural decision, the agent must produce a draft ADR and await approval before implementing

---

### Fitness Functions

A fitness function is an automated test that verifies an architectural property. Unlike unit tests (which test behavior), fitness functions test structure.

**Examples:**
- **Dependency check:** "No class in the `domain` package imports from the `infrastructure` package" — verified by ArchUnit or a custom pytest fixture that scans import graphs
- **Layer violation check:** "The `api` layer does not import directly from the `persistence` layer"
- **Cyclic dependency check:** "No circular imports exist between packages"
- **Performance fitness function:** "The p99 response time for the `/search` endpoint must be <200ms under 1,000 concurrent users" — verified by a load test in CI

Fitness functions turn architectural rules from documents into automated enforcement. They run in CI and fail builds when architectural properties are violated.

---

### Technical Debt as Architecture Liability

Technical debt is the accumulated cost of architectural and design shortcuts taken to deliver faster. Unlike financial debt, technical debt accrues interest automatically: every feature added to a compromised architecture costs more than if the architecture were clean.

**Types of technical debt:**
| Type | Example | Resolution |
|------|---------|-----------|
| **Intentional/strategic** | Shipped MVP without proper error handling to validate market | Schedule remediation sprint immediately after validation |
| **Unintentional/inadvertent** | Team didn't know about the pattern they should have used | Refactoring + team learning |
| **Bit rot** | Outdated dependencies, deprecated APIs still in use | Dependency update sprints, ADR for migration |
| **Architectural** | Layering violation; God service doing everything | May require significant refactoring or service split |

**Tracking:** Every item of technical debt must be tracked as a ticket in the backlog with a cost estimate. Untracked debt is invisible debt — and invisible debt always accumulates faster.

---

### CAP/PACELC Decision Framework

When selecting a data store or designing a distributed system, the CAP and PACELC theorems provide the decision framework.

#### CAP Theorem

A distributed system can guarantee at most two of three properties:
- **C**onsistency: Every read returns the most recent write or an error
- **A**vailability: Every request receives a non-error response (not necessarily the most recent write)
- **P**artition Tolerance: The system continues to operate despite network partitions

Because network partitions are inevitable in any distributed system, the real choice is **C vs. A when a partition occurs:**

| Choice | Behavior During Partition | Data stores | Use case |
|--------|--------------------------|------------|----------|
| **CP** | Reject requests rather than return stale data | Zookeeper, HBase, etcd, most RDBMS with synchronous replication | Financial transactions, inventory counts, leader election |
| **AP** | Return possibly stale data rather than reject | Cassandra, DynamoDB (eventual), CouchDB | Social media, DNS, shopping carts, discovery services |

#### PACELC Theorem

PACELC extends CAP by addressing normal operation (no partition): Even in normal operation, there is a trade-off between **Latency** and **Consistency**.

| Database | Partition behavior | Normal operation trade-off |
|----------|--------------------|---------------------------|
| PostgreSQL (sync replication) | CP | High C, higher L |
| DynamoDB (eventually consistent) | AP | Low L, lower C |
| DynamoDB (strongly consistent) | CP | Higher L, high C |
| Cassandra (ONE consistency level) | AP | Very low L, low C |
| Cassandra (QUORUM) | AP → CP-like | Moderate L, moderate C |

**Agent protocol:** Before selecting a data store, determine the consistency requirement:
1. Can the application tolerate reading stale data? → AP is acceptable
2. Must every read reflect the most recent write? → CP required
3. Is low latency more important than consistency? → AP with eventual consistency
4. Is accuracy non-negotiable (financial, medical, inventory)? → CP with synchronous replication

---

### 12-Factor Alignment

The 12-Factor App methodology addresses which architectural concerns:

| Factor | What It Addresses | Architectural Implication |
|--------|-----------------|--------------------------|
| I: Codebase | One repo, one app | Service boundary definition |
| II: Dependencies | Explicit declaration | No implicit system dependencies |
| III: Config | Environment variables | Separation of config from code; enables portability |
| IV: Backing Services | Treat all I/O as attached resources | Loose coupling between app and infrastructure |
| V: Build/Release/Run | Strict separation of stages | Reproducible builds; rollback capability |
| VI: Processes | Stateless | Horizontal scalability; no sticky sessions |
| VII: Port Binding | Self-contained | Services are independently deployable |
| VIII: Concurrency | Scale via process model | Horizontal scaling over vertical |
| IX: Disposability | Fast startup, graceful shutdown | Cloud-native ephemeral environments |
| X: Dev/Prod Parity | Same backing services in all envs | Eliminate "works on my machine" |
| XI: Logs | Treat as event streams | Observability integration |
| XII: Admin Processes | Run as one-off processes | Database migrations, scripts as first-class |

---

### Agent Decision Protocol

When an agent must make an architectural decision, it must follow this protocol:

| Decision Type | Decision Framework | Required Artifact |
|--------------|-------------------|------------------|
| **Data store selection** | Apply CAP/PACELC; identify consistency requirement | ADR documenting the CP/AP choice and rationale |
| **Monolith vs. microservices** | Evaluate: team size, deployment cadence, domain boundary clarity, operational maturity | ADR with team/domain/operational analysis |
| **Sync vs. async communication** | Evaluate: latency requirements, coupling tolerance, failure handling | ADR documenting the coupling and failure model |
| **State management** | Prefer stateless (12-Factor VI); state in backing service (Redis, DB) | ADR if deviation from stateless |
| **New external dependency** | Security review; license check; maintenance status; 12-Factor II | Dependency review in PR; `pip-audit` check |
| **Caching strategy** | Consistency requirement; cache invalidation approach; failure behavior | ADR if cache introduces consistency risk |

---

### V4 Integration: Agile and DevOps

**Agile architecture in KA 02:**
- "Evolutionary architecture": make architectural decisions at the last responsible moment, with enough upfront structure to avoid costly redesign
- Architecture spikes: time-boxed experiments to validate architectural hypotheses (e.g., "Can we achieve the required latency with a message queue?")
- Architectural debt is tracked in the backlog and prioritized alongside feature work

**DevOps architecture in KA 02:**
- Infrastructure as Code: all deployment topology is defined in committed code (Nomad job specs, Kubernetes manifests, Terraform)
- Architecture is testable via fitness functions in CI
- Observability is an architectural requirement: every service must emit structured logs, metrics, and traces from day one

---

## Agent Guidance

### Do
- Read all existing ADRs in the repository before making or proposing any architectural decision
- Apply STRIDE threat modeling to every new trust boundary and entry point identified in the architecture
- Apply CAP/PACELC analysis when selecting a data store; document the CP/AP choice in an ADR
- Use the 4+1 view model when documenting system architecture; provide all five views
- Create a draft ADR for any new significant architectural decision; await human approval before implementing
- Prefer stateless services (12-Factor VI) for all new service designs; document any stateful exception
- Enforce architectural rules with fitness functions in CI (dependency checks, layer violation detection)
- Track all technical debt as tickets with cost estimates; never let debt accumulate invisibly
- Apply 12-Factor alignment to all new service designs
- Escalate immediately when implementation constraints suggest the chosen architecture cannot meet quality attribute requirements

### Do Not
- Contradict an accepted ADR without escalating — the ADR represents an approved decision
- Make an architectural-level decision (new data store, new communication pattern, new service boundary) silently in code without an ADR
- Choose a data store without applying CAP/PACELC analysis
- Design stateful services without explicit justification and an approved ADR
- Violate layer dependencies — the architecture's layer structure is enforced by fitness functions, not just convention
- Mix business domains in a single service for convenience — domain boundaries are architectural boundaries
- Add an external dependency without a security review and license check
- Assume the network is reliable, low-latency, or homogeneous — all eight fallacies of distributed computing apply

## Checklist
- [ ] All existing ADRs read before making architectural decisions
- [ ] CAP/PACELC analysis completed for any new data store choice
- [ ] STRIDE threat model applied to all new trust boundaries and entry points
- [ ] 12-Factor compliance verified for new services
- [ ] ADR drafted for any new significant architectural decision
- [ ] 4+1 views documented for new systems or major components
- [ ] Quality attributes explicitly specified and their trade-offs documented
- [ ] Architectural style selected with explicit justification
- [ ] Fitness functions defined for critical architectural constraints
- [ ] Technical debt tracked in backlog with cost estimates
- [ ] Infrastructure as Code committed alongside application code

## See Also
- wiki/tier1-sources/swebok-v4/overview.md
- wiki/tier1-sources/swebok-v4/ka03-design.md
- wiki/tier1-sources/swebok-v4/ka04-construction.md
- wiki/tier1-sources/swebok-v4/ka13-security.md
- wiki/tier2-core/distributed-systems/cap-theorem.md
- wiki/tier2-core/distributed-systems/twelve-factor.md
- wiki/tier2-core/design-patterns/adr-template.md
- wiki/tier3-working/checklists/architecture-review.md

## Source

IEEE Computer Society. *Guide to the Software Engineering Body of Knowledge (SWEBOK), Version 4.0*, KA 02: Software Architecture. IEEE Press, 2024.

Kruchten, P. "The 4+1 View Model of Architecture." *IEEE Software*, 12(6), 1995.

Wiggins, A. *The Twelve-Factor App*. https://12factor.net/

Brewer, E. "Towards Robust Distributed Systems." PODC Keynote, 2000.

Abadi, D. "Consistency Tradeoffs in Modern Distributed Database System Design." *IEEE Computer*, 45(2), 2012.
