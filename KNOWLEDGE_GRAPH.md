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
| **GUIDE** | Language or tool guide | python/idioms, golang/concurrency |

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
2. wiki/tier3-working/checklists/code-review.md                       # Correctness, style, design, security
3. wiki/tier3-working/checklists/security-review.md                   # Security concerns
4. wiki/tier3-working/checklists/testing-review.md                    # Test adequacy
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
1. wiki/tier3-working/database-patterns/overview.md                   # Pattern overview
2. wiki/tier3-working/database-patterns/repository-pattern.md         # Separation of concerns
3. wiki/tier3-working/database-patterns/migrations.md                 # Schema versioning strategy
4. wiki/tier3-working/database-patterns/query-optimization.md         # Performance concerns
5. wiki/tier1-sources/owasp/a03-injection.md                          # SQL injection prevention
6. wiki/tier3-working/worked-examples/repository-pattern.md           # Concrete Python example
```

### "Write a Commit Message"
```
1. wiki/tier2-core/conventional-commits/overview.md     # Format, type taxonomy, SemVer mapping
2. wiki/tier2-core/conventional-commits/specification.md # 16 formal rules for ambiguous cases
3. wiki/tier3-working/checklists/pre-commit.md          # Apply before every commit
```

---

*This file is part of the software-backend-wiki. Update when new entities or relationships are added.*
*Last updated: 2026-04-22*
