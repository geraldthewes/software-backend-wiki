# Software Engineering Operations (KA06)

> **Tier 1** | Source: SWEBOK V4, Chapter 6 | Authority: immutable

## Summary

Software Engineering Operations is a new Knowledge Area introduced in SWEBOK V4, reflecting the industry's recognition that deploying, running, and evolving software in production is as much an engineering discipline as building it. This KA covers the full operational lifecycle: CI/CD pipelines, infrastructure as code, monitoring and observability, incident management, and the DevOps culture that ties them together. Its inclusion in SWEBOK V4 signals that operations is no longer a handoff to a separate "ops" team — it is an integral part of the engineering process.

For agents, this KA establishes that a working build is not a finished product. Every service an agent creates must include operational hooks: health checks, structured logging, metrics endpoints, and deployment manifests. Software that cannot be observed in production cannot be operated safely.

## Key Concepts

### CI/CD Pipeline

**Continuous Integration (CI)**: Every commit to the mainline triggers an automated pipeline that builds the software and runs the full test suite. CI enforces that the codebase is always in a buildable, tested state.
- Build must be fast (< 10 minutes) to maintain developer flow
- All tests (unit, integration, static analysis, security scan) run on every commit
- A failing build is a team emergency — it blocks everyone

**Continuous Delivery (CD)**: The software is always in a releasable state. Every passing CI build produces an artifact that could be deployed to production, even if the actual deployment is a manual decision.

**Continuous Deployment**: Extends delivery by automating the production push. Every passing pipeline results in automatic deployment. Requires high test coverage, feature flags for risky changes, and robust rollback capability.

### Infrastructure as Code (IaC)

All infrastructure — servers, networks, databases, load balancers — is defined in version-controlled code rather than configured manually:
- **Terraform**: Declarative, provider-agnostic IaC for provisioning cloud resources
- **Ansible**: Procedural configuration management for software installation and system setup
- **Pulumi**: IaC using general-purpose programming languages (Python, TypeScript)

IaC principles:
- Infrastructure definitions live in the same repository as application code (or a linked IaC repo)
- Changes to infrastructure go through the same code review and CI process as application changes
- Environments are reproducible: staging is built from the same code as production
- Manual changes to production infrastructure are prohibited — "snowflake" servers are a liability

### Monitoring and Observability

The **three pillars of observability**:

1. **Logs**: Structured, timestamped records of discrete events. Use structured JSON logs, not free-form text. Include correlation IDs to trace a request across services.
2. **Metrics**: Numeric measurements aggregated over time (counters, gauges, histograms). Expose via Prometheus-compatible endpoints. Track the four golden signals: latency, traffic, errors, saturation.
3. **Traces**: Distributed tracing captures the path of a single request through multiple services. Tools: OpenTelemetry, Jaeger, Zipkin. Essential for diagnosing latency in microservice architectures.

**SLO/SLI/SLA**:
- **SLI (Service Level Indicator)**: A specific, measurable metric (e.g., request success rate)
- **SLO (Service Level Objective)**: A target value for an SLI (e.g., 99.9% success rate over 30 days)
- **SLA (Service Level Agreement)**: A contractual commitment to an SLO, with consequences for breach
- **Error budget**: The allowable failure rate implied by an SLO. If the error budget is exhausted, reliability work takes priority over feature development.

### Incident Management

Structured response process for production failures:

1. **Detect**: Alerting fires from SLO violations, anomaly detection, or user reports
2. **Triage**: Assess severity, assign an incident commander, form a response team
3. **Contain**: Limit blast radius — roll back, disable the feature, shed load, failover
4. **Resolve**: Implement a durable fix; confirm SLIs return to normal
5. **Postmortem (blameless)**: Document the timeline, contributing factors, and action items without assigning individual blame. The goal is system improvement, not punishment.

### Runbooks

Operational documentation for known failure modes. A runbook answers: "Given this alert, what do I do?" Structure:
- Alert name and description
- Likely causes
- Step-by-step diagnosis procedure
- Remediation steps
- Escalation path if steps do not resolve the issue

Every new failure mode encountered in production should result in a runbook update.

### DevOps Culture: CALMS

- **Culture**: Shared responsibility for delivery and operations; no "throw it over the wall"
- **Automation**: Automate repetitive toil — deployments, testing, scaling, incident detection
- **Lean**: Minimize work in progress; optimize value stream flow; eliminate waste
- **Measurement**: Instrument everything; make decisions from data
- **Sharing**: Shared tools, shared postmortems, shared on-call responsibility

### 12-Factor Alignment for Operations

Key factors relevant to operations:
- **VI (Stateless processes)**: Application instances share no state between requests; sessions live in an external store. Enables horizontal scaling and safe restarts.
- **IX (Disposability)**: Processes start fast and shut down gracefully on SIGTERM. Critical for rolling deployments and autoscaling.
- **XI (Logs as event streams)**: Apps write logs to stdout; the platform captures and routes them. Decouples logging from application logic.
- **XII (Admin processes)**: One-off tasks (migrations, scripts) run as one-off processes in the same environment as the app, not as ad-hoc server logins.

## Agent Guidance

### Do
- Include health check endpoints (`/healthz`, `/readyz`) in every service created
- Emit structured (JSON) logs with correlation IDs on every service boundary
- Expose Prometheus-compatible metrics for latency, error rate, and saturation
- Define deployment manifests (Nomad jobspec, Kubernetes manifest, Docker Compose) alongside application code
- Write a runbook entry for every new failure mode discovered
- Use feature flags for high-risk deployments rather than full cutover
- Pin all base image versions in Dockerfiles to ensure reproducible builds

### Do Not
- Deploy software that has no logging, metrics, or health checks
- Make manual changes to production infrastructure outside the IaC process
- Merge a failing CI pipeline
- Write logs as unstructured free-form text
- Skip postmortems after production incidents — every incident is a learning opportunity

## Checklist
- [ ] CI pipeline defined and runs on every commit (build + test + lint + security scan)
- [ ] Health check endpoints implemented and registered with the load balancer
- [ ] Structured logging with correlation IDs in place
- [ ] Metrics exposed (at minimum: request rate, error rate, latency histogram)
- [ ] Deployment manifest version-controlled in the repository
- [ ] SLO defined for the service with corresponding alerts
- [ ] Runbook created for all expected failure modes
- [ ] Rollback procedure tested before first production deployment
- [ ] Infrastructure defined as code, not configured manually

## See Also
- `wiki/tier1-sources/swebok-v4/ka08-config-management.md`
- `wiki/tier1-sources/swebok-v4/ka12-quality.md`
- `wiki/tier3-working/observability/overview.md`
- `wiki/tier2-core/twelve-factor-app/factors.md`

## Source
IEEE Computer Society. *Guide to the Software Engineering Body of Knowledge (SWEBOK), Version 4*. Chapter 6: Software Engineering Operations. IEEE Press, 2024.
