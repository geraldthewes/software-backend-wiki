# Design Review Checklist

> **Tier 3** | Enforces: wiki/tier1-sources/swebok-v4/ka02-architecture.md, wiki/tier1-sources/swebok-v4/ka03-design.md, wiki/tier2-core/solid-principles/overview.md, wiki/tier2-core/distributed-systems/overview.md, wiki/tier1-sources/swebok-v4/ka13-security.md

## Architecture

- [ ] The design maps to a recognized architectural style (layered, hexagonal, event-driven, microservice) — not an ad-hoc structure
- [ ] Layers have defined boundaries: presentation → application → domain → infrastructure; dependencies point inward only
- [ ] New service/component justified — existing services considered and ruled out before adding a new one
- [ ] A clear decision record (ADR) written for significant architectural choices
- [ ] The design fits the system's scale requirements — no over-engineering for current load; no under-engineering for stated growth
- [ ] Data ownership is clear — each entity has exactly one authoritative data store; no duplicated sources of truth

## Design (SOLID and Principles)

- [ ] Each new class/module has a single clearly stated responsibility (SRP)
- [ ] Abstractions defined before implementations — interfaces/Protocols written first, implementations second (DIP)
- [ ] Extension points identified — where will this grow? Is extension possible without modifying existing code? (OCP)
- [ ] Interfaces are narrow and client-specific — no fat interfaces forcing implementors to provide unused methods (ISP)
- [ ] Subtypes preserve invariants of base types — no surprising behavior differences between concrete implementations (LSP)
- [ ] Dependency graph is a DAG (directed acyclic graph) — no circular dependencies between modules or services
- [ ] Pure functions and immutable data objects used for business logic where possible (Functional Core)

## Distributed Systems (if applicable)

- [ ] CAP/PACELC trade-off understood and documented — is this system CP or AP? What happens during partition?
- [ ] Network calls have defined timeouts — no call without a timeout
- [ ] Retry logic includes exponential backoff with jitter — no fixed-interval retries
- [ ] Idempotency defined for all mutating operations — safe to retry without duplicate side effects
- [ ] Service discovery used for all inter-service calls — no hardcoded IPs or hostnames
- [ ] Circuit breaker pattern applied to calls to unreliable dependencies
- [ ] Message delivery semantics defined — at-most-once, at-least-once, or exactly-once? Consumer handles duplicates?
- [ ] Data consistency model defined — strong, eventual, or causal consistency? Acceptable for the use case?

## Security Design

- [ ] Threat model considered — what assets are protected? Who are the adversaries? What are the attack surfaces?
- [ ] Authentication mechanism defined — how are callers identified?
- [ ] Authorization model defined — how are permissions checked? Principle of Least Privilege applied?
- [ ] Data at rest encryption: sensitive data fields identified; encryption-at-rest configured for those stores
- [ ] Data in transit: TLS required for all external and internal service communication
- [ ] PII identified and minimized — only data necessary for the function is collected and stored
- [ ] Audit trail defined — what actions generate audit log entries? Who can read them?

## Observability Design

- [ ] Structured logs defined — what events are logged at INFO or above? Correlation IDs propagated?
- [ ] Key metrics identified — Four Golden Signals (latency, traffic, errors, saturation) for each service endpoint
- [ ] Distributed tracing planned — which operations emit spans? Are context headers propagated?
- [ ] Alerting criteria defined — what failure condition triggers a page? What triggers a ticket?
- [ ] Runbook / on-call guide identified or planned for the new component

## See Also

- wiki/tier3-working/checklists/security-review.md
- wiki/tier2-core/solid-principles/overview.md
- wiki/tier2-core/distributed-systems/overview.md
- wiki/tier1-sources/swebok-v4/ka02-architecture.md

## Source

SWEBOK V4, KA2 (Architecture), KA3 (Design), KA13 (Security). "Designing Distributed Systems" (Burns, O'Reilly 2018). "Building Microservices" (Newman, O'Reilly 2021). OWASP Threat Modeling Cheat Sheet.
