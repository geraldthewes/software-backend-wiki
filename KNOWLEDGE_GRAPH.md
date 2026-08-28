# Knowledge Graph — Entity Relationships and Navigation Paths

> Use this file when you need to understand how wiki topics relate to each other, or when following a multi-step workflow that crosses multiple domains.

---

## Entity Types

| Type | Description | Examples |
|------|-------------|---------|
| **KA** | SWEBOK V4 Knowledge Area | ka02-architecture, ka05-testing |
| **SEC** | OWASP Security Control | a01-broken-access-control, a03-injection |
| **PEP** | Python Enhancement Proposal | pep-008-style, pep-484-type-hints |
| **PRINCIPLE** | Design/Architecture Principle | SOLID-SRP, 12factor-III, CAP |
| **PATTERN** | Design or Resilience Pattern | repository-pattern, circuit-breaker |
| **CHECKLIST** | Review checklist | code-review, security-review |
| **EXAMPLE** | Worked implementation example | dependency-injection, error-handling |
| **GUIDE** | Language or tool guide | python/idioms, golang/concurrency, dotnet/microservices |
| **REF** | External Reference | redgate-database-design, microsoft-database-basics, aws-dynamodb-nosql-design |

---

## Relationship Types

| Relationship | Meaning |
|-------------|---------|
| **IMPLEMENTS** | A practical guide/example implements a principle or standard |
| **ENFORCES** | A checklist enforces a standard or principle |
| **EXTENDS** | Tier 2 content extends/elaborates a Tier 1 KA |
| **REFERENCES** | A page normatively cites another as its authority |
| **REQUIRES** | Understanding B requires first understanding A |
| **MITIGATES** | A control or practice mitigates a security risk |
| **CONFLICTS_WITH** | Known tension — both cannot be fully satisfied simultaneously |
| **COMPLEMENTS** | Two things work together; applying both is better than one |

---

## Adjacency Table

| From | Relationship | To | Notes |
|------|-----------|----|-------|
| ka02-architecture | REQUIRES | cap-pacelc | Architecture decisions need CAP/PACELC analysis for data stores |
| ka02-architecture | REQUIRES | fallacies | Service design requires knowing the 8 distributed fallacies |
| ka02-architecture | EXTENDS | twelve-factor-app/factors | 12-factor is a practical application of architecture principles |
| ka03-design | REFERENCES | solid-principles/overview | SOLID is the primary design principle framework |
| ka03-design | REQUIRES | ka02-architecture | Detailed design follows architectural decisions |
| ka04-construction | IMPLEMENTS | pep-008-style | Python construction must follow PEP 8 |
| ka04-construction | IMPLEMENTS | pep-484-type-hints | Python construction must use type hints |
| ka05-testing | IMPLEMENTS | test-pyramid | Test pyramid is the primary test strategy |
| ka05-testing | IMPLEMENTS | property-based-testing | Property testing is mandated for data-transformation code |
| ka05-testing | IMPLEMENTS | mutation-testing | Mutation testing verifies suite strength |
| ka05-testing | REQUIRES | ka04-construction | Tests are part of construction; design for testability first |
| ka12-quality | ENFORCES | code-review | Quality assurance is operationalized by code review |
| ka13-security | IMPLEMENTS | owasp/top10 | SWEBOK security KA maps directly to OWASP controls |
| ka13-security | IMPLEMENTS | python-pyscg | Pyscg rules are the Python implementation of ka13 controls |
| ka13-security | IMPLEMENTS | threat-modeling | STRIDE threat modeling is mandated in ka13 |
| ka13-security | IMPLEMENTS | zero-trust | Zero Trust is the architecture model for ka13 |
| ka14-professional-practice | REFERENCES | code-of-ethics | Professional practice grounds in ACM/IEEE ethics |
| owasp/a01 | MITIGATES | least-privilege-principle | Server-side access control enforces least privilege |
| owasp/a02 | MITIGATES | cryptography-library | Using vetted crypto library prevents cryptographic failures |
| owasp/a03 | MITIGATES | pyscg-0010 | Parameterized queries prevent SQL injection (pyscg-0010) |
| owasp/a06 | MITIGATES | pip-audit | pip-audit identifies vulnerable dependencies (A06) |
| owasp/a09 | MITIGATES | structured-logging | Proper structured logging prevents A09 |
| solid-principles/srp | IMPLEMENTS | repository-pattern | Repository pattern demonstrates SRP in data access |
| solid-principles/dip | IMPLEMENTS | dependency-injection | DI example demonstrates dependency inversion |
| solid-principles/dip | IMPLEMENTS | repository-pattern | Repository abstracts persistence (DIP) |
| solid-principles/ocp | COMPLEMENTS | design-patterns/behavioral | Strategy/Observer patterns enable OCP |
| property-based-testing | COMPLEMENTS | mutation-testing | Both together give the strongest test suite |
| test-pyramid | REQUIRES | unit-testing-concept | Unit testing knowledge precedes pyramid strategy |
| resilience-patterns | IMPLEMENTS | fallacies | Resilience patterns are the remediation for each fallacy |
| resilience-patterns | REQUIRES | cap-pacelc | Resilience strategy depends on CP vs AP choice |
| twelve-factor-app/factors | IMPLEMENTS | ka06-operations | 12-factor is the operational implementation guide |
| threat-modeling | REQUIRES | ka02-architecture | Threat modeling requires architectural diagram as input |
| zero-trust | EXTENDS | ka13-security | Zero Trust is the network/identity model within KA13 |
| python/functional-core | IMPLEMENTS | solid-principles/srp | Functional core isolates pure functions (SRP) |
| python/functional-core | IMPLEMENTS | solid-principles/dip | Functional core separates I/O (DIP) |
| golang/concurrency | REQUIRES | ka04-construction | Concurrency is a construction-level concern |
| database-patterns/repository | IMPLEMENTS | solid-principles/dip | Repository abstracts data access (DIP) |
| database-patterns/repository | IMPLEMENTS | solid-principles/srp | Each layer has single responsibility |
| database-patterns/database-design | REFERENCES | redgate-database-design | Redgate's Top 11 Best Practices for Database Design |
| database-patterns/database-design | REFERENCES | microsoft-database-basics | Microsoft Database Design Basics |
| database-patterns/database-design | REFERENCES | aws-dynamodb-nosql-design | AWS DynamoDB NoSQL Design Best Practices |
| database-patterns/database-design | REFERENCES | mongodb-data-modeling | MongoDB Official Data Modeling |
| database-patterns/database-design | REFERENCES | neo4j-graph-modeling | Neo4j Graph Data Modeling Tips & Best Practices |
| database-patterns/database-design | REFERENCES | influxdb-timeseries-model | InfluxData Time Series Database Explained + Data Model |
| database-patterns/database-design | REFERENCES | timescaledb-hypertables | TimescaleDB Best Practices for Time-Series Data Modeling & Hypertables |
| database-patterns/database-design | REFERENCES | aws-s3-performance | AWS S3 Optimizing Performance (Object Key Design & Partitioning) |
| database-patterns/database-design | IMPLEMENTS | solid-principles/srp | Database design layer has single responsibility |
| database-patterns/database-design | IMPLEMENTS | solid-principles/dip | Database design abstracts persistence concerns (DIP) |
| dotnet/microservices | REFERENCES | ka02-architecture | .NET microservices guide builds on software architecture principles |
| dotnet/microservices | REFERENCES | ka03-design | .NET microservices guide applies software design principles |
| dotnet/microservices | EXTENDS | distributed-systems/overview | .NET microservices guide extends distributed systems concepts |
| dotnet/microservices | REFERENCES | api-design/overview | .NET microservices guide includes API design considerations |
| dotnet/microservices | REFERENCES | observability/overview | .NET microservices guide includes observability practices |
| dotnet/microservices | COMPLEMENTS | twelve-factor-app/factors | .NET microservices guide aligns with 12-factor app principles |
| api-design/rest-conventions | REFERENCES | owasp/a01 | REST APIs must enforce server-side access control |
| api-design/grpc | COMPLEMENTS | api-design/openapi | Choose REST+OpenAPI or gRPC based on use case |
| observability/structured-logging | MITIGATES | owasp/a09 | Structured logging addresses A09 |
| pep-020-zen | IMPLEMENTS | ka04-construction | Zen principles are the Python construction philosophy |
| code-review checklist | ENFORCES | ka12-quality | Code review is the quality gate |
| code-review checklist | ENFORCES | pep-008-style | Style checked at review |
| code-review checklist | ENFORCES | pep-484-type-hints | Type hints checked at review |
| design-review checklist | ENFORCES | ka02-architecture | Architecture reviewed before implementation |
| design-review checklist | ENFORCES | solid-principles/overview | SOLID compliance is a design review item |
| security-review checklist | ENFORCES | ka13-security | Security review operationalizes the security KA |
| security-review checklist | ENFORCES | owasp/top10 | OWASP Top 10 is the security review framework |
| testing-review checklist | ENFORCES | ka05-testing | Testing review verifies test quality |
| testing-review checklist | ENFORCES | mutation-testing | Mutation score is a testing review item |
| cap-pacelc | CONFLICTS_WITH | availability-vs-consistency | CP systems sacrifice availability; AP sacrifice consistency |
| twelve-factor/III-config | COMPLEMENTS | pyscg-0041 | Both mandate externalizing configuration and secrets |
| conventional-commits/overview | EXTENDS | ka08-config-management | Conventional Commits formalizes commit message discipline within SCM |
| conventional-commits/overview | IMPLEMENTS | ka08-config-management | Provides the machine-readable commit format for automated release tooling |
| conventional-commits/overview | COMPLEMENTS | twelve-factor-app/factors | Both enforce process discipline: commits communicate scope; 12-factor governs deployment |
| conventional-commits/specification | REFERENCES | conventional-commits/overview | Detailed 16-rule normative spec; overview is the entry point |
| code-review checklist | ENFORCES | conventional-commits/overview | Commit message format checked at review time |
| engineering-playbook/source-control | EXTENDS | ka08-config-management | Playbook operationalizes SCM policy as Git-specific branch and PR practices |
| engineering-playbook/source-control | COMPLEMENTS | conventional-commits/overview | Both govern commit discipline; CC defines format, playbook defines workflow |
| engineering-playbook/source-control | IMPLEMENTS | ka08-config-management | Branch naming, merge strategy, and PR workflow implement KA8 change management |
| engineering-playbook/agile-development | EXTENDS | ka09-engineering-management | Playbook operationalizes estimation, planning, risk as sprint ceremonies and DoR/DoD |
| engineering-playbook/agile-development | EXTENDS | ka10-process | Sprint model, retrospectives, and working agreements are the KA10 process model in practice |
| engineering-playbook/agile-development | IMPLEMENTS | ka12-quality | DoD is the team-level quality gate; retrospectives drive process improvement |
| engineering-playbook/developer-experience | EXTENDS | ka06-operations | F5 Contract and essential tasks operationalize KA6's operational readiness requirements |
| engineering-playbook/developer-experience | IMPLEMENTS | solid-principles/dip | Dependency injection for local mocks is the DevEx application of DIP |
| engineering-playbook/developer-experience | REQUIRES | engineering-playbook/documentation-practices | Onboarding docs are a DevEx dependency |
| engineering-playbook/documentation-practices | IMPLEMENTS | ka12-quality | Documentation anti-patterns and DoD standards implement KA12 quality assurance |
| engineering-playbook/documentation-practices | COMPLEMENTS | api-design/openapi | Documentation practices mandate OpenAPI for all HTTP APIs |
| engineering-playbook/overview | REFERENCES | engineering-playbook/source-control | Primary sub-page for source control practices |
| engineering-playbook/overview | REFERENCES | engineering-playbook/agile-development | Primary sub-page for agile practices |
| engineering-playbook/overview | REFERENCES | engineering-playbook/developer-experience | Primary sub-page for DevEx |
| engineering-playbook/overview | REFERENCES | engineering-playbook/documentation-practices | Primary sub-page for documentation |
| architecture-patterns/overview | EXTENDS | ka02-architecture | Architecture Patterns derives from SWEBOK KA2 (architectural styles) |
| architecture-patterns/overview | EXTENDS | ka03-design | Architecture Patterns derives from SWEBOK KA3 (design principles) |
| architecture-patterns/ports-and-adapters | IMPLEMENTS | solid-principles/dip | Ports-and-adapters is the architectural application of DIP |
| architecture-patterns/repository | IMPLEMENTS | architecture-patterns/ports-and-adapters | Repository pattern is the outbound port for persistence |
| architecture-patterns/repository | IMPLEMENTS | solid-principles/dip | Repository abstracts persistence behind a Protocol (DIP) |
| architecture-patterns/service-layer | COMPLEMENTS | architecture-patterns/domain-model | Service layer orchestrates domain without containing business logic |
| architecture-patterns/unit-of-work | COMPLEMENTS | architecture-patterns/repository | UoW provides transactional scope for repository operations |
| architecture-patterns/aggregates | ENFORCES | architecture-patterns/domain-model | Aggregate root enforces domain invariants as a consistency boundary |
| architecture-patterns/domain-events-message-bus | ENABLES | architecture-patterns/event-driven-integration | In-process bus extends naturally to external brokers |
| architecture-patterns/domain-events-message-bus | REQUIRES | architecture-patterns/aggregates | Aggregates emit events; bus cannot function without event sources |
| architecture-patterns/commands-vs-events | COMPLEMENTS | architecture-patterns/domain-events-message-bus | Commands and events share the same bus with different routing rules |
| architecture-patterns/cqrs | COMPLEMENTS | architecture-patterns/domain-events-message-bus | Event-fed read models are the CQRS approach for high-throughput reads |
| architecture-patterns/dependency-injection-bootstrap | IMPLEMENTS | solid-principles/dip | Bootstrap is the composition root — the single place that wires DIP |
| architecture-patterns/legacy-migration | MITIGATES | ka07-maintenance | Strangler Fig and incremental adoption address large-scale maintenance |
| architecture-patterns/validation | REFERENCES | ka01-requirements | Validation placement is a requirements boundary concern |
| architecture-patterns/overview | REFERENCES | architecture-patterns/ports-and-adapters | Ports-and-adapters is the cross-cutting architectural frame |
| architecture-patterns/overview | REFERENCES | architecture-patterns/domain-model | Domain model is the starting point for all other patterns |
| design-review checklist | ENFORCES | architecture-patterns/ports-and-adapters | Design review checks that dependency arrows point inward |
| xdg-base-directory | COMPLEMENTS | twelve-factor-app/factors | XDG governs where user-machine files live; 12-factor governs deploy config — both externalize environment specifics |
| xdg-base-directory | EXTENDS | ka06-operations | File placement (config/data/cache/state/runtime) is an operational concern for deployed tools |
| xdg-base-directory | MITIGATES | owasp/a05 | 0700 directory creation and predictable file locations prevent misconfigured, world-readable user files |
| code-review-method/overview | EXTENDS | ka12-quality | Technical review method operationalizes KA12 reviews: invariants, precedence, severity |
| code-review-method/overview | REFERENCES | ka14-professional-practice | "Say no early" is professional judgment; comment tone is constrained by KA14 |
| code-review-method/overview | CONFLICTS_WITH | code-of-ethics | Source persona's abusive LKML voice is rejected; ACM/IEEE Principle 7 (colleagues) wins |
| code-review-method/overview | COMPLEMENTS | code-review-guidelines | Method = what to look for and how to grade; guidelines = how to conduct the review |
| code-review-method/overview | COMPLEMENTS | owasp-code-review | Invariant security triggers complement OWASP Top 10 checkpoints; neither replaces the other |
| code-review-method/overview | COMPLEMENTS | conventional-commits/overview | Conventional Commits is message format; the method requires the *why* |
| code-review-method/triggers | REFERENCES | code-review-method/overview | Trigger catalog is the operational companion to the method |
| code-review-method/triggers | MITIGATES | owasp/a03 | Unbounded format/copy and trust-boundary validation catch injection-class defects |
| code-review checklist | ENFORCES | code-review-method/overview | Checklist's invariant section is the method's first scan |
| code-review-method/overview | REQUIRES | ka12-quality | Method is a quality-gate practice, not a substitute for SWEBOK quality attributes |

---

## Named Navigation Paths

These are the recommended reading sequences for common agent tasks. Follow in order.

### "Design a New Service"
```
1. wiki/tier1-sources/swebok-v4/ka02-architecture.md       # Understand architectural styles and ADRs
2. wiki/tier2-core/distributed-systems/cap-pacelc.md       # Data store selection framework
3. wiki/tier2-core/distributed-systems/fallacies.md        # Distributed system assumptions to avoid
4. wiki/tier1-sources/swebok-v4/ka03-design.md             # Detailed design principles
5. wiki/tier2-core/solid-principles/overview.md            # SOLID application
6. wiki/tier2-core/twelve-factor-app/factors.md            # Cloud-native operational requirements
7. wiki/tier3-working/checklists/design-review.md          # Apply before finalizing design
```

### "Write Production Python"
```
1. wiki/tier1-sources/swebok-v4/ka04-construction.md       # Construction principles
2. wiki/tier1-sources/python-peps/pep-008-style.md         # Style requirements
3. wiki/tier1-sources/python-peps/pep-484-type-hints.md    # Type annotation requirements
4. wiki/tier1-sources/python-peps/pep-020-zen.md           # Python philosophy
5. wiki/tier3-working/python/idioms.md                     # Pythonic patterns
6. wiki/tier3-working/python/functional-core.md            # Separate pure logic from I/O
7. wiki/tier3-working/checklists/pre-commit.md             # Apply before every commit
```

### "Write Production Go"
```
1. wiki/tier1-sources/swebok-v4/ka04-construction.md       # Construction principles
2. wiki/tier3-working/golang/overview.md                   # Go-specific context
3. wiki/tier3-working/golang/idioms.md                     # Effective Go patterns, error handling
4. wiki/tier3-working/golang/concurrency.md                # goroutines, channels, sync
5. wiki/tier3-working/golang/toolchain.md                  # Build, test, lint workflow
6. wiki/tier3-working/checklists/pre-commit.md             # Apply before every commit
```

### "Test a Module"
```
1. wiki/tier1-sources/swebok-v4/ka05-testing.md                       # Testing theory and strategy
2. wiki/tier2-core/testing-strategies/test-pyramid.md                 # Proportion and type
3. wiki/tier2-core/testing-strategies/property-based-testing.md       # For data-transformation logic
4. wiki/tier2-core/testing-strategies/mutation-testing.md             # Verify suite quality
5. wiki/tier3-working/checklists/testing-review.md                    # Apply before PR
```

### "Security Audit"
```
1. wiki/tier1-sources/swebok-v4/ka13-security.md                      # Security framework
2. wiki/tier1-sources/owasp/top10-2021-overview.md                    # 10 risk categories
3. wiki/tier2-core/security-practices/python-pyscg.md                 # Python-specific rules
4. wiki/tier2-core/security-practices/threat-modeling.md              # STRIDE analysis
5. wiki/tier3-working/checklists/security-review.md                   # Apply systematically
```

### "Review a Pull Request"
```
1. wiki/tier1-sources/swebok-v4/ka12-quality.md                       # Quality framework
2. wiki/tier2-core/code-review-method/overview.md                     # Precedence, invariants, [REASON]→[ACT]
3. wiki/tier2-core/code-review-method/triggers.md                     # Level 1/2 triggers and severity tree
4. wiki/tier3-working/checklists/code-review.md                       # Operational checklist (invariants first)
5. wiki/tier3-working/code-review-guidelines/overview.md              # Process, pacing, comment culture
6. wiki/tier3-working/checklists/security-review.md                   # Security concerns
7. wiki/tier3-working/checklists/testing-review.md                    # Test adequacy
```

### "Build a Distributed System"
```
1. wiki/tier1-sources/swebok-v4/ka02-architecture.md                  # Architectural foundation
2. wiki/tier2-core/distributed-systems/cap-pacelc.md                  # Consistency/availability trade-offs
3. wiki/tier2-core/distributed-systems/fallacies.md                   # Don't assume these
4. wiki/tier2-core/distributed-systems/resilience-patterns.md         # Retry, circuit breaker, etc.
5. wiki/tier2-core/twelve-factor-app/factors.md                       # Operational requirements
6. wiki/tier3-working/observability/overview.md                       # Make it observable
7. wiki/tier3-working/checklists/design-review.md                     # Apply before implementation
```

### "Design a REST API"
```
1. wiki/tier3-working/api-design/overview.md                          # API design context
2. wiki/tier3-working/api-design/rest-conventions.md                  # Naming, status codes, versioning
3. wiki/tier3-working/api-design/openapi.md                           # Contract-first specification
4. wiki/tier1-sources/owasp/a01-broken-access-control.md              # Auth/authz requirements
5. wiki/tier1-sources/owasp/a03-injection.md                          # Input validation requirements
6. wiki/tier3-working/checklists/design-review.md                     # Apply before implementation
```

### "Add Observability"
```
1. wiki/tier3-working/observability/overview.md                       # Three pillars: logs, metrics, traces
2. wiki/tier3-working/observability/structured-logging.md             # Structured log format, PII rules
3. wiki/tier3-working/observability/metrics.md                        # Prometheus model, label design
4. wiki/tier3-working/observability/slo-sli-sla.md                    # Define SLOs before alerting
5. wiki/tier1-sources/owasp/a09-logging-monitoring.md                 # Security logging requirements
```

### "Design a Database Layer"
```
1. wiki/tier3-working/database-patterns/database-design.md            # Design best practices
2. wiki/tier3-working/database-patterns/overview.md                   # Pattern overview
3. wiki/tier3-working/database-patterns/repository-pattern.md         # Separation of concerns
4. wiki/tier3-working/database-patterns/migrations.md                 # Schema versioning strategy
5. wiki/tier3-working/database-patterns/query-optimization.md         # Performance concerns
6. wiki/tier1-sources/owasp/a03-injection.md                          # SQL injection prevention
7. wiki/tier3-working/worked-examples/repository-pattern.md           # Concrete Python example
```

### "Write a Commit Message"
```
1. wiki/tier2-core/conventional-commits/overview.md        # Format, type taxonomy, SemVer mapping
2. wiki/tier2-core/conventional-commits/specification.md   # 16 formal rules for ambiguous cases
3. wiki/tier3-working/checklists/pre-commit.md             # Apply before every commit
```

### "Set Up a New Repository"
```
1. wiki/tier2-core/engineering-playbook/source-control.md         # Branch strategy, protection rules, essential files
2. wiki/tier1-sources/swebok-v4/ka08-config-management.md         # SCM policy and release strategy
3. wiki/tier2-core/conventional-commits/overview.md               # Configure commit convention
4. wiki/tier2-core/engineering-playbook/documentation-practices.md # README and CONTRIBUTING.md minimum standard
5. wiki/tier3-working/checklists/pre-commit.md                    # Pre-commit hook setup
```

### "Onboard a New Developer"
```
1. wiki/tier2-core/engineering-playbook/developer-experience.md   # F5 Contract expectations, essential tasks
2. wiki/tier2-core/engineering-playbook/documentation-practices.md # README/CONTRIBUTING standards
3. wiki/tier2-core/engineering-playbook/source-control.md          # Branch naming and PR workflow
4. wiki/tier2-core/engineering-playbook/agile-development.md       # Working agreements and DoD
```

### "Start a Sprint"
```
1. wiki/tier2-core/engineering-playbook/agile-development.md      # DoR, DoD, sprint ceremonies
2. wiki/tier1-sources/swebok-v4/ka09-engineering-management.md    # Estimation and risk management
3. wiki/tier1-sources/swebok-v4/ka10-process.md                   # Process model authority
```

### "Build a Domain-Driven Service"
```
1. wiki/tier1-sources/swebok-v4/ka02-architecture.md                     # Architectural styles and ADRs
2. wiki/tier1-sources/swebok-v4/ka03-design.md                           # Design principles
3. wiki/tier2-core/architecture-patterns/overview.md                     # Pattern catalog and layering philosophy
4. wiki/tier2-core/architecture-patterns/ports-and-adapters.md           # Architectural frame: hexagonal/clean
5. wiki/tier2-core/architecture-patterns/domain-model.md                 # Domain objects: entities, value objects
6. wiki/tier2-core/architecture-patterns/repository.md                   # Persistence abstraction
7. wiki/tier2-core/architecture-patterns/service-layer.md                # Orchestration layer; Flask as adapter
8. wiki/tier2-core/architecture-patterns/unit-of-work.md                 # Transaction boundary
9. wiki/tier2-core/architecture-patterns/aggregates.md                   # Consistency boundaries
10. wiki/tier2-core/architecture-patterns/dependency-injection-bootstrap.md  # Wire everything together
11. wiki/tier3-working/checklists/design-review.md                       # Apply before implementation
```

### "Build an Event-Driven Service"
```
1. wiki/tier2-core/architecture-patterns/overview.md                     # Context for Part 2 patterns
2. wiki/tier2-core/architecture-patterns/domain-events-message-bus.md    # Domain events and in-process bus
3. wiki/tier2-core/architecture-patterns/commands-vs-events.md           # Distinguish commands from events
4. wiki/tier2-core/architecture-patterns/event-driven-integration.md     # External broker integration
5. wiki/tier2-core/architecture-patterns/cqrs.md                         # Read model separation
6. wiki/tier2-core/architecture-patterns/dependency-injection-bootstrap.md  # Composition root wiring
7. wiki/tier2-core/distributed-systems/fallacies.md                      # Distributed system assumptions to avoid
8. wiki/tier3-working/checklists/design-review.md                        # Apply before implementation
```

### "Design a .NET Microservice"
```
1. wiki/tier1-sources/swebok-v4/ka02-architecture.md                     # Architectural styles and ADRs
2. wiki/tier1-sources/swebok-v4/ka03-design.md                           # Design principles
3. wiki/tier3-working/dotnet/overview.md                                 # .NET microservices architecture guide
4. wiki/tier2-core/distributed-systems/cap-pacelc.md                     # Data store selection framework
5. wiki/tier2-core/distributed-systems/fallacies.md                      # Distributed system assumptions to avoid
6. wiki/tier2-core/twelve-factor-app/factors.md                          # Cloud-native operational requirements
7. wiki/tier3-working/api-design/overview.md                             # API design context
8. wiki/tier3-working/observability/overview.md                          # Observability practices
9. wiki/tier3-working/checklists/design-review.md                        # Apply before finalizing design
```

---

*This file is part of the software-backend-wiki. Update when new entities or relationships are added.*
*Last updated: 2026-07-09 (xdg-base-directory added)*
