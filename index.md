# Software Engineering Wiki — Master Catalog

> **Start here after reading `AGENTS.md`.** This file maps every wiki page to its topic and lists 200+ searchable keywords. Find what you need, then follow the See Also links within each page.

---

## Quick Reference by Task

| I need to... | Read first | Then apply |
|--------------|-----------|-----------|
| Design a service | `wiki/tier1-sources/swebok-v4/ka02-architecture.md` | `checklists/design-review.md` |
| Design classes/modules | `wiki/tier1-sources/swebok-v4/ka03-design.md` | `checklists/design-review.md` |
| Write Python | `wiki/tier1-sources/swebok-v4/ka04-construction.md` + `python-peps/pep-008-style.md` | `checklists/pre-commit.md` |
| Write Go | `wiki/tier1-sources/swebok-v4/ka04-construction.md` + `golang/idioms.md` | `checklists/pre-commit.md` |
| Write tests | `wiki/tier1-sources/swebok-v4/ka05-testing.md` | `checklists/testing-review.md` |
| Security review | `wiki/tier1-sources/swebok-v4/ka13-security.md` + `owasp/top10-2021-overview.md` | `checklists/security-review.md` |
| Code review | `wiki/tier1-sources/swebok-v4/ka12-quality.md` | `checklists/code-review.md` |
| Design a distributed system | `wiki/tier2-core/distributed-systems/cap-pacelc.md` | `checklists/design-review.md` |
| Design a database layer | `wiki/tier3-working/database-patterns/database-design.md` | `checklists/security-review.md` |
| Design an API | `wiki/tier3-working/api-design/overview.md` | `checklists/design-review.md` |
| Add observability | `wiki/tier3-working/observability/overview.md` | `checklists/code-review.md` |
| Design a domain-driven service | `wiki/tier2-core/architecture-patterns/overview.md` | `checklists/design-review.md` |
| Build an event-driven service | `wiki/tier2-core/architecture-patterns/domain-events-message-bus.md` | `checklists/design-review.md` |
| Write a commit message | `wiki/tier2-core/conventional-commits/overview.md` | `checklists/pre-commit.md` |
| Decide where a CLI/service stores config, data, cache files | `wiki/tier2-core/xdg-base-directory/overview.md` | `checklists/design-review.md` |
| Set up source control / branching | `wiki/tier2-core/engineering-playbook/source-control.md` | `checklists/pre-commit.md` |
| Plan a sprint / manage backlog | `wiki/tier2-core/engineering-playbook/agile-development.md` | `wiki/tier1-sources/swebok-v4/ka09-engineering-management.md` |
| Improve developer experience | `wiki/tier2-core/engineering-playbook/developer-experience.md` | `checklists/pre-commit.md` |
| Write or review documentation | `wiki/tier2-core/engineering-playbook/documentation-practices.md` | `checklists/code-review.md` |

---

## Tier 1 — Sources of Truth

> These documents are authoritative and immutable. Never contradicted by Tier 2 or 3.

### SWEBOK V4 — 18 Knowledge Areas

| File | Knowledge Area | Status in V4 | Key Terms |
|------|---------------|-------------|-----------|
| `wiki/tier1-sources/swebok-v4/overview.md` | SWEBOK V4 Orientation | — | knowledge areas, SWEBOK, IEEE |
| `wiki/tier1-sources/swebok-v4/ka01-requirements.md` | Software Requirements | Core | elicitation, specification, NFRs, validation |
| `wiki/tier1-sources/swebok-v4/ka02-architecture.md` | Software Architecture | **New in V4** | architectural styles, ADR, fitness functions, quality attributes |
| `wiki/tier1-sources/swebok-v4/ka03-design.md` | Software Design | Core | coupling, cohesion, abstraction, design patterns, SOLID |
| `wiki/tier1-sources/swebok-v4/ka04-construction.md` | Software Construction | Core | coding standards, PEP 8, type hints, defensive programming |
| `wiki/tier1-sources/swebok-v4/ka05-testing.md` | Software Testing | Core | test pyramid, property-based, mutation testing, TDD |
| `wiki/tier1-sources/swebok-v4/ka06-operations.md` | Software Engineering Operations | **New in V4** | CI/CD, monitoring, SLO, DevOps, incident management |
| `wiki/tier1-sources/swebok-v4/ka07-maintenance.md` | Software Maintenance | Core | refactoring, technical debt, change management |
| `wiki/tier1-sources/swebok-v4/ka08-config-management.md` | Software Configuration Management | Core | version control, branching, release, SCM |
| `wiki/tier1-sources/swebok-v4/ka09-engineering-management.md` | Software Engineering Management | Core | estimation, risk, planning, project management |
| `wiki/tier1-sources/swebok-v4/ka10-process.md` | Software Engineering Process | Core | SDLC, Agile, Scrum, DevOps culture, process improvement |
| `wiki/tier1-sources/swebok-v4/ka11-models-methods.md` | Software Engineering Models and Methods | Core | UML, formal methods, modeling |
| `wiki/tier1-sources/swebok-v4/ka12-quality.md` | Software Quality | Core | quality attributes, SQA, reviews, metrics |
| `wiki/tier1-sources/swebok-v4/ka13-security.md` | Software Security and Privacy | **New in V4** | CIA triad, STRIDE, OWASP, threat modeling, Zero Trust |
| `wiki/tier1-sources/swebok-v4/ka14-professional-practice.md` | Professional Practice | Core | ethics, ACM code, communication, responsibility |
| `wiki/tier1-sources/swebok-v4/ka15-economics.md` | Software Engineering Economics | Core | ROI, cost estimation, make-vs-buy, technical debt |
| `wiki/tier1-sources/swebok-v4/ka16-computing-foundations.md` | Computing Foundations | Core | algorithms, data structures, OS, databases, networking |
| `wiki/tier1-sources/swebok-v4/ka17-mathematical-foundations.md` | Mathematical Foundations | Core | logic, set theory, graph theory, probability |
| `wiki/tier1-sources/swebok-v4/ka18-engineering-foundations.md` | Engineering Foundations | Core | measurement, empirical methods, root cause analysis |

### OWASP Top 10:2021

| File | Risk | Key Terms |
|------|------|-----------|
| `wiki/tier1-sources/owasp/top10-2021-overview.md` | All 10 risks overview | OWASP, web security, vulnerabilities |
| `wiki/tier1-sources/owasp/a01-broken-access-control.md` | A01: Broken Access Control | authorization, least privilege, IDOR |
| `wiki/tier1-sources/owasp/a02-cryptographic-failures.md` | A02: Cryptographic Failures | encryption, TLS, hashing, PII |
| `wiki/tier1-sources/owasp/a03-injection.md` | A03: Injection | SQL injection, command injection, parameterized queries |
| `wiki/tier1-sources/owasp/a04-insecure-design.md` | A04: Insecure Design | threat modeling, secure design patterns |
| `wiki/tier1-sources/owasp/a05-security-misconfiguration.md` | A05: Security Misconfiguration | default credentials, hardening, config |
| `wiki/tier1-sources/owasp/a06-vulnerable-components.md` | A06: Vulnerable Components | dependency scanning, CVE, pip-audit, SBOM |
| `wiki/tier1-sources/owasp/a07-auth-failures.md` | A07: Auth Failures | authentication, session management, MFA |
| `wiki/tier1-sources/owasp/a08-software-integrity-failures.md` | A08: Integrity Failures | CI/CD security, deserialization, supply chain |
| `wiki/tier1-sources/owasp/a09-logging-monitoring.md` | A09: Logging/Monitoring Failures | structured logging, alerting, SIEM |
| `wiki/tier1-sources/owasp/a10-ssrf.md` | A10: SSRF | server-side request forgery, URL validation |

### Python PEPs

| File | PEP | Key Terms |
|------|-----|-----------|
| `wiki/tier1-sources/python-peps/overview.md` | PEP Index | Python standards, style, typing |
| `wiki/tier1-sources/python-peps/pep-008-style.md` | PEP 8 — Style Guide | indentation, naming, line length, imports |
| `wiki/tier1-sources/python-peps/pep-020-zen.md` | PEP 20 — Zen of Python | explicit, simple, readability, errors |
| `wiki/tier1-sources/python-peps/pep-484-type-hints.md` | PEP 484 — Type Hints | typing, annotations, mypy, Generic |
| `wiki/tier1-sources/python-peps/pep-443-singledispatch.md` | PEP 443 — singledispatch | polymorphism, functools, dispatch |

### Ethics

| File | Standard | Key Terms |
|------|---------|-----------|
| `wiki/tier1-sources/acm-ieee-ethics/code-of-ethics.md` | ACM/IEEE-CS Code of Ethics | public interest, judgment, profession, reporting risks |

### Martin Fowler

| File | Standard | Key Terms |
|------|---------|-----------|
| `wiki/tier1-sources/martin-fowler/bounded-context.md` | Bounded Context | domain-driven design, ubiquitous language, strategic design |

### Eric Evans

| File | Standard | Key Terms |
|------|---------|-----------|
| `wiki/tier1-sources/eric-evans/ddd-reference.md` | Domain-Driven Design Reference | bounded context, ubiquitous language, aggregates, entities, value objects |

### W3C Standards

| File | Standard | Key Terms |
|------|---------|-----------|
| `wiki/tier1-sources/w3c/skos-primer.md` | SKOS Simple Knowledge Organization System Primer | semantic web, concept schemes, mappings, controlled vocabularies |

---

## Tier 2 — Core Knowledge

> Established practices derived from Tier 1 standards. Apply unless Tier 1 overrides.

| File | Topic | Derives From | Key Terms |
|------|-------|-------------|-----------|
| `wiki/tier2-core/solid-principles/overview.md` | SOLID Overview | SWEBOK KA3 | SRP, OCP, LSP, ISP, DIP |
| `wiki/tier2-core/solid-principles/srp.md` | Single Responsibility | SWEBOK KA3 | cohesion, one reason to change |
| `wiki/tier2-core/solid-principles/ocp.md` | Open/Closed Principle | SWEBOK KA3 | extension, ABC, Protocol |
| `wiki/tier2-core/solid-principles/lsp.md` | Liskov Substitution | SWEBOK KA3 | behavioral subtyping, inheritance |
| `wiki/tier2-core/solid-principles/isp.md` | Interface Segregation | SWEBOK KA3 | small interfaces, Protocol |
| `wiki/tier2-core/solid-principles/dip.md` | Dependency Inversion | SWEBOK KA3 | abstractions, dependency injection |
| `wiki/tier2-core/twelve-factor-app/overview.md` | 12-Factor Overview | SWEBOK KA2/KA6 | cloud-native, 12factor |
| `wiki/tier2-core/twelve-factor-app/factors.md` | All 12 Factors | 12factor.net | codebase, config, backing services, stateless |
| `wiki/tier2-core/distributed-systems/overview.md` | Distributed Systems | SWEBOK KA2 | distributed, microservices |
| `wiki/tier2-core/distributed-systems/cap-pacelc.md` | CAP and PACELC | SWEBOK KA2 | consistency, availability, partition, latency |
| `wiki/tier2-core/distributed-systems/fallacies.md` | 8 Fallacies | Peter Deutsch | network reliability, latency, bandwidth |
| `wiki/tier2-core/distributed-systems/resilience-patterns.md` | Resilience Patterns | SWEBOK KA2 | retry, circuit breaker, bulkhead, timeout, backoff |
| `wiki/tier2-core/testing-strategies/overview.md` | Testing Strategies | SWEBOK KA5 | TDD, BDD, test pyramid |
| `wiki/tier2-core/testing-strategies/test-pyramid.md` | Test Pyramid | SWEBOK KA5 | unit, integration, E2E |
| `wiki/tier2-core/testing-strategies/property-based-testing.md` | Property-Based Testing | Hypothesis | invariants, strategies, shrinking |
| `wiki/tier2-core/testing-strategies/mutation-testing.md` | Mutation Testing | mutmut | mutation score, killed mutants |
| `wiki/tier2-core/design-patterns/overview.md` | GoF Design Patterns | SWEBOK KA3 | creational, structural, behavioral |
| `wiki/tier2-core/design-patterns/creational.md` | Creational Patterns | GoF | factory, builder, singleton, prototype |
| `wiki/tier2-core/design-patterns/structural.md` | Structural Patterns | GoF | adapter, facade, decorator, proxy |
| `wiki/tier2-core/design-patterns/behavioral.md` | Behavioral Patterns | GoF | strategy, observer, command, template method |
| `wiki/tier2-core/security-practices/overview.md` | Security Practices | SWEBOK KA13 | OWASP, Pyscg, threat modeling |
| `wiki/tier2-core/security-practices/python-pyscg.md` | Python Secure Coding | Pyscg | pyscg-0010, parameterized queries, pickle, secrets |
| `wiki/tier2-core/security-practices/threat-modeling.md` | Threat Modeling | STRIDE | STRIDE, attack surface, trust boundaries |
| `wiki/tier2-core/security-practices/zero-trust.md` | Zero Trust Architecture | NIST SP 800-207 | never trust, always verify, microsegmentation |
| `wiki/tier2-core/conventional-commits/overview.md` | Conventional Commits Overview | conventionalcommits.org 1.0.0 / SWEBOK KA8 | commit message, feat, fix, BREAKING CHANGE, SemVer, changelog |
| `wiki/tier2-core/conventional-commits/specification.md` | Conventional Commits Specification | conventionalcommits.org 1.0.0 | 16 rules, RFC 2119, footer format, type prefix, scope |
| `wiki/tier2-core/xdg-base-directory/overview.md` | XDG Base Directory Specification | freedesktop.org basedir-spec 0.8 | XDG_DATA_HOME, XDG_CONFIG_HOME, XDG_STATE_HOME, XDG_CACHE_HOME, XDG_RUNTIME_DIR, dotfiles, config directory |
| `wiki/tier2-core/engineering-playbook/overview.md` | Engineering Playbook Overview | Microsoft ISE / SWEBOK KA8, KA9, KA10, KA12 | playbook, ISE, engineering fundamentals, coverage map |
| `wiki/tier2-core/engineering-playbook/source-control.md` | Source Control Practices | Microsoft ISE / SWEBOK KA8 | git, branch naming, PR workflow, commit hygiene, merge strategy, secrets |
| `wiki/tier2-core/engineering-playbook/agile-development.md` | Agile Development Practices | Microsoft ISE / SWEBOK KA9, KA10 | agile, sprint, backlog, DoR, DoD, retrospective, working agreements, technical debt |
| `wiki/tier2-core/engineering-playbook/developer-experience.md` | Developer Experience (DevEx) | Microsoft ISE / SWEBOK KA6, KA4 | DevEx, F5 contract, onboarding, build, test, start, debug, local mock |
| `wiki/tier2-core/engineering-playbook/documentation-practices.md` | Documentation Practices | Microsoft ISE / SWEBOK KA12 | documentation, README, anti-patterns, OpenAPI, onboarding docs, PR description |
| `wiki/tier2-core/architecture-patterns/overview.md` | Architecture Patterns Overview | "Architecture Patterns with Python" / SWEBOK KA02, KA03, DDD | Cosmic Python, hexagonal, DDD, ports and adapters, event-driven, domain model |
| `wiki/tier2-core/data-mesh/overview.md` | Data Mesh Principles and Logical Architecture | Martin Fowler / DDD | data mesh, domain ownership, data as product, self-serve platform, federated governance |
| `wiki/tier2-core/architecture-patterns/domain-model.md` | Domain Model | "Architecture Patterns with Python" Ch 1 | domain model, entity, value object, domain service, rich model, plain Python |
| `wiki/tier2-core/architecture-patterns/repository.md` | Repository Pattern (Tier 2) | "Architecture Patterns with Python" Ch 2 | repository, classical ORM mapping, SQLAlchemy, collection abstraction |
| `wiki/tier2-core/architecture-patterns/service-layer.md` | Service Layer | "Architecture Patterns with Python" Ch 4 | service layer, orchestration, use case, Flask adapter, thin entrypoint |
| `wiki/tier2-core/architecture-patterns/unit-of-work.md` | Unit of Work | "Architecture Patterns with Python" Ch 6 | unit of work, transaction, context manager, session, event collection |
| `wiki/tier2-core/architecture-patterns/aggregates.md` | Aggregates and Consistency Boundaries | "Architecture Patterns with Python" Ch 7 | aggregate, aggregate root, consistency boundary, invariant, optimistic concurrency, bounded context |
| `wiki/tier2-core/architecture-patterns/domain-events-message-bus.md` | Domain Events and Message Bus | "Architecture Patterns with Python" Ch 8–9 | domain event, message bus, event handler, in-process bus, cascading dispatch |
| `wiki/tier2-core/architecture-patterns/commands-vs-events.md` | Commands vs Events | "Architecture Patterns with Python" Ch 10 | command, event, command handler, fire-and-forget, routing rules |
| `wiki/tier2-core/architecture-patterns/event-driven-integration.md` | Event-Driven Integration | "Architecture Patterns with Python" Ch 11 | external events, Redis pub/sub, choreography, orchestration, distributed ball of mud |
| `wiki/tier2-core/architecture-patterns/cqrs.md` | CQRS | "Architecture Patterns with Python" Ch 12 | CQRS, read model, write model, N+1, raw SQL, denormalized view |
| `wiki/tier2-core/architecture-patterns/dependency-injection-bootstrap.md` | Dependency Injection and Bootstrap | "Architecture Patterns with Python" Ch 13 | composition root, bootstrap, DI, partial application, inject |
| `wiki/tier2-core/architecture-patterns/ports-and-adapters.md` | Ports and Adapters | "Architecture Patterns with Python" / Cockburn 2005 | hexagonal architecture, onion architecture, clean architecture, ports, adapters, functional core |
| `wiki/tier2-core/architecture-patterns/validation.md` | Validation Placement | "Architecture Patterns with Python" App E | validation, syntactic, semantic, pragmatic, Postel's Law |
| `wiki/tier2-core/architecture-patterns/legacy-migration.md` | Legacy Migration | "Architecture Patterns with Python" Epilogue | strangler fig, big ball of mud, architecture tax, incremental adoption, Django |

---

## Tier 3 — Working Documents

> Language guides, checklists, and worked examples. Verify alignment with Tier 1/2 before applying.

| File | Topic | Key Terms |
|------|-------|-----------|
| `wiki/tier3-working/python/overview.md` | Python Best Practices | Python, idiomatic, PEP |
| `wiki/tier3-working/ddd/overview.md` | Domain-Driven Design Crew Resources | DDD Crew / DDD | welcome to DDD, starter modelling, bounded context canvas, event storming |
| `wiki/tier3-working/python/idioms.md` | Python Idioms | list comprehension, dataclass, context manager, Pythonic |
| `wiki/tier3-working/python/type-system.md` | Python Type System | typing, Optional, Union, TypeVar, Protocol |
| `wiki/tier3-working/python/functional-core.md` | Functional Core / Imperative Shell | pure functions, side effects, immutability |
| `wiki/tier3-working/python/async-patterns.md` | Python Async | asyncio, await, async def, TaskGroup |
| `wiki/tier3-working/python/logging.md` | Python Logging | logging, handlers, formatters, NullHandler, log levels |
| `wiki/tier3-working/python/logging-cookbook.md` | Python Logging Cookbook | QueueHandler, LoggerAdapter, rotation, multiprocessing logging |
| `wiki/tier3-working/python/regex.md` | Python Regular Expressions | re module, regex, pattern, ReDoS, groups, flags |
| `wiki/tier3-working/python/enum.md` | Python Enumerations | Enum, IntEnum, StrEnum, Flag, auto, unique |
| `wiki/tier3-working/python/sorting.md` | Python Sorting | sorted, key function, attrgetter, itemgetter, stability |
| `wiki/tier3-working/python/annotations.md` | Python Annotations | __annotations__, inspect.get_annotations, PEP 563, type hints |
| `wiki/tier3-working/python/descriptors.md` | Python Descriptors | __get__, __set__, property, classmethod, staticmethod, __slots__ |
| `wiki/tier3-working/python/unicode.md` | Python Unicode | str, bytes, encode, decode, NFC, NFD, normalization, encoding |
| `wiki/tier3-working/python/argparse.md` | Python argparse | ArgumentParser, CLI, subcommands, positional, optional arguments |
| `wiki/tier3-working/python/urllib.md` | Python urllib | HTTP client, urllib.request, requests, httpx, SSRF |
| `wiki/tier3-working/python/sockets.md` | Python Sockets | socket, TCP, recv, send, select, framing, IPC |
| `wiki/tier3-working/python/ipaddress.md` | Python ipaddress | IPv4, IPv6, CIDR, network, address, SSRF prevention |
| `wiki/tier3-working/python/mro.md` | Python MRO | method resolution order, C3, super, multiple inheritance |
| `wiki/tier3-working/python/curses.md` | Python curses | terminal UI, ncurses, window, color, keyboard input |
| `wiki/tier3-working/python/free-threading.md` | Python Free-Threading | GIL, free-threaded, CPython 3.13, thread safety, parallelism |
| `wiki/tier3-working/python/isolating-extensions.md` | Python Extension Isolation | C extension, per-module state, heap types, subinterpreters |
| `wiki/tier3-working/golang/overview.md` | Go Best Practices | Go, Effective Go, idiomatic |
| `wiki/tier3-working/awesome-ddd/overview.md` | Awesome Domain-Driven Design | heynickc / DDD, CQRS, Event Sourcing | books, free eBooks, training, videos, community, blogs, sample projects |
| `wiki/tier3-working/golang/idioms.md` | Go Idioms | error wrapping, interfaces, defer, named returns |
| `wiki/tier3-working/golang/concurrency.md` | Go Concurrency | goroutines, channels, WaitGroup, select, context |
| `wiki/tier3-working/golang/toolchain.md` | Go Toolchain | go build, go test, golangci-lint, go mod |
| `wiki/tier3-working/database-patterns/overview.md` | Database Patterns | ORM, repository, migrations, connection pool |
| `wiki/tier3-working/database-patterns/repository-pattern.md` | Repository Pattern | persistence abstraction, domain separation |
| `wiki/tier3-working/database-patterns/migrations.md` | Database Migrations | alembic, golang-migrate, schema versioning |
| `wiki/tier3-working/database-patterns/query-optimization.md` | Query Optimization | N+1, indexing, EXPLAIN, connection pooling |
| `wiki/tier3-working/database-patterns/database-design.md` | Database Design Best Practices | relational, NoSQL, graph, time-series, object storage |
| `wiki/tier3-working/api-design/overview.md` | API Design | REST, gRPC, OpenAPI |
| `wiki/tier3-working/api-design/rest-conventions.md` | REST Conventions | naming, status codes, versioning, pagination, HATEOAS |
| `wiki/tier3-working/api-design/openapi.md` | OpenAPI / Contract-First | OpenAPI 3.1, spec-first, code generation |
| `wiki/tier3-working/api-design/grpc.md` | gRPC | protobuf, service definition, streaming, error model |
| `wiki/tier3-working/observability/overview.md` | Observability | logging, metrics, tracing, three pillars |
| `wiki/tier3-working/observability/structured-logging.md` | Structured Logging | JSON logs, log levels, correlation ID, no PII |
| `wiki/tier3-working/observability/metrics.md` | Metrics | counter, gauge, histogram, Prometheus, labels |
| `wiki/tier3-working/observability/slo-sli-sla.md` | SLO / SLI / SLA | error budget, availability, latency percentiles |
| `wiki/tier3-working/dotnet/overview.md` | .NET Microservices Architecture | .NET, microservices, Docker, containers, eShopOnContainers |
| `wiki/tier3-working/checklists/pre-commit.md` | Pre-Commit Checklist | secrets, linting, type check, audit |
| `wiki/tier3-working/checklists/code-review.md` | Code Review Checklist | correctness, style, design, security, testing |
| `wiki/tier3-working/checklists/design-review.md` | Design Review Checklist | architecture, SOLID, distributed, security design |
| `wiki/tier3-working/checklists/testing-review.md` | Testing Review Checklist | coverage, property tests, mutation score |
| `wiki/tier3-working/checklists/security-review.md` | Security Review Checklist | OWASP, injection, auth, crypto, dependencies |
| `wiki/tier3-working/owasp-code-review/overview.md` | OWASP Code Review Guide | OWASP Top 10, security review, vulnerability identification |
| `wiki/tier3-working/code-review-guidelines/overview.md` | Comprehensive Code Review Guidelines | Google, Microsoft, SmartBear, Palantir, Thoughtbot |
| `wiki/tier3-working/worked-examples/repository-pattern.md` | Repository Pattern (Python) | SQLite, Protocol, dataclass, SRP, DIP |
| `wiki/tier3-working/worked-examples/dependency-injection.md` | Dependency Injection (Python) | Protocol, constructor injection, mock, testable |
| `wiki/tier3-working/worked-examples/error-handling.md` | Error Handling (Python) | PEP 20, context manager, Result type, logging |

---

## Tier 4 — Archive

| File | Original Topic | Superseded By |
|------|---------------|--------------|
| `wiki/tier4-archive/README.md` | Archive policy | — |

---

## Keyword Index

Alphabetical. Each term → primary wiki page. Use Ctrl+F / grep to find a term quickly.

| Term | Primary Page | Secondary Page |
|------|-------------|---------------|
| Abstract Base Class (ABC) | `tier2-core/solid-principles/ocp.md` | `tier3-working/python/idioms.md` |
| Aggregate | `tier2-core/architecture-patterns/aggregates.md` | `tier2-core/architecture-patterns/overview.md` |
| Aggregate Root | `tier2-core/architecture-patterns/aggregates.md` | `tier2-core/architecture-patterns/domain-model.md` |
| Annotations (__annotations__) | `tier3-working/python/annotations.md` | `tier1-sources/python-peps/pep-484-type-hints.md` |
| argparse | `tier3-working/python/argparse.md` | `tier3-working/python/overview.md` |
| Access control | `tier1-sources/owasp/a01-broken-access-control.md` | `tier1-sources/swebok-v4/ka13-security.md` |
| Agile development | `tier2-core/engineering-playbook/agile-development.md` | `tier1-sources/swebok-v4/ka10-process.md` |
| ADR (Architecture Decision Record) | `tier1-sources/swebok-v4/ka02-architecture.md` | — |
| Agile | `tier1-sources/swebok-v4/ka10-process.md` | `tier2-core/twelve-factor-app/overview.md` |
| Algorithm complexity | `tier1-sources/swebok-v4/ka16-computing-foundations.md` | — |
| alembic | `tier3-working/database-patterns/migrations.md` | — |
| API versioning | `tier3-working/api-design/rest-conventions.md` | — |
| assert (security risk) | `tier2-core/security-practices/python-pyscg.md` | `tier1-sources/swebok-v4/ka13-security.md` |
| asyncio | `tier3-working/python/async-patterns.md` | `tier3-working/python/free-threading.md` |
| Availability (CAP) | `tier2-core/distributed-systems/cap-pacelc.md` | — |
| Backoff (exponential) | `tier2-core/distributed-systems/resilience-patterns.md` | `tier2-core/distributed-systems/fallacies.md` |
| Big Ball of Mud | `tier2-core/architecture-patterns/legacy-migration.md` | `tier2-core/architecture-patterns/overview.md` |
| Bootstrap (composition root) | `tier2-core/architecture-patterns/dependency-injection-bootstrap.md` | `tier3-working/worked-examples/dependency-injection.md` |
| Bounded Context | `tier2-core/architecture-patterns/aggregates.md` | `tier2-core/architecture-patterns/overview.md` |
| Bandwidth fallacy | `tier2-core/distributed-systems/fallacies.md` | — |
| bcrypt | `tier1-sources/owasp/a02-cryptographic-failures.md` | `tier2-core/security-practices/python-pyscg.md` |
| Behavioral patterns | `tier2-core/design-patterns/behavioral.md` | `tier2-core/design-patterns/overview.md` |
| bytes vs str | `tier3-working/python/unicode.md` | `tier3-working/python/overview.md` |
| Backlog management | `tier2-core/engineering-playbook/agile-development.md` | `tier1-sources/swebok-v4/ka09-engineering-management.md` |
| Branch naming | `tier2-core/engineering-playbook/source-control.md` | `tier1-sources/swebok-v4/ka08-config-management.md` |
| Branch strategy | `tier1-sources/swebok-v4/ka08-config-management.md` | `tier2-core/engineering-playbook/source-control.md` |
| BREAKING CHANGE | `tier2-core/conventional-commits/overview.md` | `tier2-core/conventional-commits/specification.md` |
| Builder pattern | `tier2-core/design-patterns/creational.md` | — |
| Bulkhead pattern | `tier2-core/distributed-systems/resilience-patterns.md` | — |
| Cache directory (XDG_CACHE_HOME) | `tier2-core/xdg-base-directory/overview.md` | — |
| CAP theorem | `tier2-core/distributed-systems/cap-pacelc.md` | `tier1-sources/swebok-v4/ka02-architecture.md` |
| CHANGELOG | `tier2-core/conventional-commits/overview.md` | `tier1-sources/swebok-v4/ka08-config-management.md` |
| CI/CD | `tier1-sources/swebok-v4/ka06-operations.md` | `tier2-core/twelve-factor-app/factors.md` |
| Commit message | `tier2-core/conventional-commits/overview.md` | `tier2-core/conventional-commits/specification.md` |
| Conventional Commits | `tier2-core/conventional-commits/overview.md` | `tier2-core/conventional-commits/specification.md` |
| CIA triad | `tier1-sources/swebok-v4/ka13-security.md` | — |
| Circuit breaker | `tier2-core/distributed-systems/resilience-patterns.md` | — |
| Clean Architecture | `tier2-core/architecture-patterns/ports-and-adapters.md` | `tier2-core/architecture-patterns/overview.md` |
| Code coverage | `tier1-sources/swebok-v4/ka05-testing.md` | `tier2-core/testing-strategies/mutation-testing.md` |
| Command (domain) | `tier2-core/architecture-patterns/commands-vs-events.md` | `tier2-core/architecture-patterns/domain-events-message-bus.md` |
| Command Handler | `tier2-core/architecture-patterns/commands-vs-events.md` | `tier2-core/architecture-patterns/service-layer.md` |
| Composition Root | `tier2-core/architecture-patterns/dependency-injection-bootstrap.md` | `tier2-core/solid-principles/dip.md` |
| Consistency Boundary | `tier2-core/architecture-patterns/aggregates.md` | `tier2-core/distributed-systems/cap-pacelc.md` |
| CQRS | `tier2-core/architecture-patterns/cqrs.md` | `tier2-core/architecture-patterns/commands-vs-events.md` |
| Code review | `tier1-sources/swebok-v4/ka12-quality.md` | `tier3-working/checklists/code-review.md` |
| Cohesion | `tier1-sources/swebok-v4/ka03-design.md` | `tier2-core/solid-principles/srp.md` |
| CONTRIBUTING.md | `tier2-core/engineering-playbook/source-control.md` | `tier2-core/engineering-playbook/documentation-practices.md` |
| Command pattern | `tier2-core/design-patterns/behavioral.md` | — |
| Config (12-Factor III) | `tier2-core/twelve-factor-app/factors.md` | `tier1-sources/owasp/a05-security-misconfiguration.md` |
| Config directory (XDG_CONFIG_HOME) | `tier2-core/xdg-base-directory/overview.md` | `tier2-core/twelve-factor-app/factors.md` |
| Connection pooling | `tier3-working/database-patterns/query-optimization.md` | — |
| Consistency (CAP) | `tier2-core/distributed-systems/cap-pacelc.md` | — |
| Context manager | `tier3-working/python/idioms.md` | `tier2-core/security-practices/python-pyscg.md` |
| Contract-first API | `tier3-working/api-design/openapi.md` | — |
| Correlation ID | `tier3-working/observability/structured-logging.md` | — |
| Cost estimation | `tier1-sources/swebok-v4/ka15-economics.md` | — |
| Coupling | `tier1-sources/swebok-v4/ka03-design.md` | `tier2-core/solid-principles/overview.md` |
| Creational patterns | `tier2-core/design-patterns/creational.md` | `tier2-core/design-patterns/overview.md` |
| Cryptography (library) | `tier1-sources/owasp/a02-cryptographic-failures.md` | `tier2-core/security-practices/python-pyscg.md` |
| CVE | `tier1-sources/owasp/a06-vulnerable-components.md` | — |
| Cyclomatic complexity | `tier1-sources/swebok-v4/ka03-design.md` | `tier1-sources/swebok-v4/ka04-construction.md` |
| dataclass | `tier3-working/python/idioms.md` | `tier3-working/python/type-system.md` |
| Descriptors | `tier3-working/python/descriptors.md` | `tier3-working/python/type-system.md` |
| Database migrations | `tier3-working/database-patterns/migrations.md` | — |
| Decorator pattern | `tier2-core/design-patterns/structural.md` | — |
| Defensive programming | `tier1-sources/swebok-v4/ka04-construction.md` | — |
| Definition of Done (DoD) | `tier2-core/engineering-playbook/agile-development.md` | `tier1-sources/swebok-v4/ka12-quality.md` |
| Definition of Ready (DoR) | `tier2-core/engineering-playbook/agile-development.md` | `tier1-sources/swebok-v4/ka09-engineering-management.md` |
| Dependency injection | `tier2-core/architecture-patterns/dependency-injection-bootstrap.md` | `tier3-working/worked-examples/dependency-injection.md` |
| Domain event | `tier2-core/architecture-patterns/domain-events-message-bus.md` | `tier2-core/architecture-patterns/aggregates.md` |
| Domain-Driven Design | `tier2-core/architecture-patterns/aggregates.md` | `tier2-core/architecture-patterns/overview.md` |
| Domain Model | `tier2-core/architecture-patterns/domain-model.md` | `tier2-core/architecture-patterns/overview.md` |
| Developer experience | `tier2-core/engineering-playbook/developer-experience.md` | `tier1-sources/swebok-v4/ka06-operations.md` |
| DevEx | `tier2-core/engineering-playbook/developer-experience.md` | — |
| Documentation | `tier2-core/engineering-playbook/documentation-practices.md` | `tier1-sources/swebok-v4/ka12-quality.md` |
| Dependency Inversion Principle | `tier2-core/solid-principles/dip.md` | `tier1-sources/swebok-v4/ka03-design.md` |
| Deserialization (insecure) | `tier1-sources/owasp/a08-software-integrity-failures.md` | `tier2-core/security-practices/python-pyscg.md` |
| DevOps | `tier1-sources/swebok-v4/ka06-operations.md` | `tier1-sources/swebok-v4/ka10-process.md` |
| DIP | `tier2-core/solid-principles/dip.md` | — |
| Distributed systems | `tier2-core/distributed-systems/overview.md` | `tier1-sources/swebok-v4/ka02-architecture.md` |
| Domain service | `tier2-core/architecture-patterns/domain-model.md` | `tier2-core/architecture-patterns/service-layer.md` |
| Dotfiles | `tier2-core/xdg-base-directory/overview.md` | — |
| E2E testing | `tier2-core/testing-strategies/test-pyramid.md` | `tier1-sources/swebok-v4/ka05-testing.md` |
| Effective Go | `tier3-working/golang/idioms.md` | `tier3-working/golang/overview.md` |
| Engineering Fundamentals Playbook | `tier2-core/engineering-playbook/overview.md` | — |
| Enum (Python) | `tier3-working/python/enum.md` | `tier3-working/python/overview.md` |
| Error budget | `tier3-working/observability/slo-sli-sla.md` | — |
| Error handling | `tier3-working/worked-examples/error-handling.md` | `tier1-sources/python-peps/pep-020-zen.md` |
| Ethics | `tier1-sources/acm-ieee-ethics/code-of-ethics.md` | `tier1-sources/swebok-v4/ka14-professional-practice.md` |
| Entity (domain) | `tier2-core/architecture-patterns/domain-model.md` | `tier2-core/architecture-patterns/aggregates.md` |
| Event-driven architecture | `tier2-core/architecture-patterns/event-driven-integration.md` | `tier2-core/architecture-patterns/domain-events-message-bus.md` |
| Exponential backoff | `tier2-core/distributed-systems/resilience-patterns.md` | — |
| Facade pattern | `tier2-core/design-patterns/structural.md` | — |
| F5 Contract | `tier2-core/engineering-playbook/developer-experience.md` | — |
| Factory pattern | `tier2-core/design-patterns/creational.md` | — |
| Feature flag | `tier2-core/engineering-playbook/source-control.md` | — |
| feat (commit type) | `tier2-core/conventional-commits/overview.md` | `tier2-core/conventional-commits/specification.md` |
| fix (commit type) | `tier2-core/conventional-commits/overview.md` | `tier2-core/conventional-commits/specification.md` |
| Fallacies of distributed computing | `tier2-core/distributed-systems/fallacies.md` | — |
| Formal methods | `tier1-sources/swebok-v4/ka11-models-methods.md` | — |
| Functional core / imperative shell | `tier3-working/python/functional-core.md` | `tier2-core/architecture-patterns/ports-and-adapters.md` |
| Functional Core Imperative Shell | `tier2-core/architecture-patterns/ports-and-adapters.md` | `tier3-working/python/functional-core.md` |
| Free-threading (Python) | `tier3-working/python/free-threading.md` | `tier3-working/python/async-patterns.md` |
| Functional programming | `tier3-working/python/functional-core.md` | `tier3-working/python/idioms.md` |
| Generic (typing) | `tier3-working/python/type-system.md` | `tier1-sources/python-peps/pep-484-type-hints.md` |
| GoF design patterns | `tier2-core/design-patterns/overview.md` | `tier1-sources/swebok-v4/ka03-design.md` |
| golang-migrate | `tier3-working/database-patterns/migrations.md` | — |
| goroutine | `tier3-working/golang/concurrency.md` | — |
| gRPC | `tier3-working/api-design/grpc.md` | — |
| Hardcoded secrets | `tier1-sources/swebok-v4/ka13-security.md` | `tier2-core/security-practices/python-pyscg.md` |
| Hexagonal Architecture | `tier2-core/architecture-patterns/ports-and-adapters.md` | `tier2-core/architecture-patterns/overview.md` |
| git revert | `tier2-core/engineering-playbook/source-control.md` | `tier1-sources/swebok-v4/ka08-config-management.md` |
| Histogram (metrics) | `tier3-working/observability/metrics.md` | — |
| Hypothesis (library) | `tier2-core/testing-strategies/property-based-testing.md` | `tier1-sources/swebok-v4/ka05-testing.md` |
| Idempotency | `tier2-core/distributed-systems/resilience-patterns.md` | — |
| IDOR | `tier1-sources/owasp/a01-broken-access-control.md` | — |
| Immutability | `tier3-working/python/functional-core.md` | `tier3-working/python/idioms.md` |
| Import ordering | `tier1-sources/python-peps/pep-008-style.md` | — |
| Incident management | `tier1-sources/swebok-v4/ka06-operations.md` | — |
| Indexing (database) | `tier3-working/database-patterns/query-optimization.md` | — |
| Injection (SQL/command) | `tier1-sources/owasp/a03-injection.md` | `tier2-core/security-practices/python-pyscg.md` |
| ipaddress (Python) | `tier3-working/python/ipaddress.md` | `tier1-sources/owasp/a10-ssrf.md` |
| Integration testing | `tier2-core/testing-strategies/test-pyramid.md` | `tier1-sources/swebok-v4/ka05-testing.md` |
| Interface Segregation Principle | `tier2-core/solid-principles/isp.md` | — |
| ISP | `tier2-core/solid-principles/isp.md` | — |
| Jitter (backoff) | `tier2-core/distributed-systems/resilience-patterns.md` | — |
| JSON logging | `tier3-working/observability/structured-logging.md` | — |
| Keyword argument | `tier3-working/python/idioms.md` | — |
| Knowledge silo | `tier2-core/engineering-playbook/documentation-practices.md` | — |
| Latency (distributed) | `tier2-core/distributed-systems/cap-pacelc.md` | `tier2-core/distributed-systems/fallacies.md` |
| Layered architecture | `tier1-sources/swebok-v4/ka02-architecture.md` | `tier2-core/architecture-patterns/ports-and-adapters.md` |
| Legacy migration | `tier2-core/architecture-patterns/legacy-migration.md` | `tier1-sources/swebok-v4/ka07-maintenance.md` |
| Least privilege | `tier1-sources/owasp/a01-broken-access-control.md` | `tier1-sources/swebok-v4/ka13-security.md` |
| Line length | `tier1-sources/python-peps/pep-008-style.md` | — |
| Liskov Substitution Principle | `tier2-core/solid-principles/lsp.md` | — |
| log.md (this wiki) | `log.md` | — |
| Logging | `tier3-working/observability/structured-logging.md` | `tier1-sources/owasp/a09-logging-monitoring.md` |
| Logging (Python stdlib) | `tier3-working/python/logging.md` | `tier3-working/python/logging-cookbook.md` |
| LSP | `tier2-core/solid-principles/lsp.md` | — |
| Make-vs-buy | `tier1-sources/swebok-v4/ka15-economics.md` | — |
| Merge strategy | `tier2-core/engineering-playbook/source-control.md` | `tier1-sources/swebok-v4/ka08-config-management.md` |
| Message Bus | `tier2-core/architecture-patterns/domain-events-message-bus.md` | `tier2-core/architecture-patterns/commands-vs-events.md` |
| Microservices | `tier1-sources/swebok-v4/ka02-architecture.md` | `tier2-core/distributed-systems/overview.md` |
| Migration (database) | `tier3-working/database-patterns/migrations.md` | — |
| Mock (testing) | `tier2-core/testing-strategies/test-pyramid.md` | `tier3-working/worked-examples/dependency-injection.md` |
| Monitoring | `tier1-sources/swebok-v4/ka06-operations.md` | `tier3-working/observability/metrics.md` |
| MRO (Method Resolution Order) | `tier3-working/python/mro.md` | `tier2-core/solid-principles/lsp.md` |
| mTLS | `tier2-core/security-practices/zero-trust.md` | — |
| Mutation testing | `tier2-core/testing-strategies/mutation-testing.md` | `tier1-sources/swebok-v4/ka05-testing.md` |
| mutmut | `tier2-core/testing-strategies/mutation-testing.md` | — |
| mypy | `tier1-sources/python-peps/pep-484-type-hints.md` | `tier3-working/python/type-system.md` |
| N+1 query | `tier3-working/database-patterns/query-optimization.md` | `tier2-core/architecture-patterns/cqrs.md` |
| Onboarding | `tier2-core/engineering-playbook/developer-experience.md` | `tier2-core/engineering-playbook/documentation-practices.md` |
| ncurses / curses | `tier3-working/python/curses.md` | `tier3-working/python/overview.md` |
| Named return values (Go) | `tier3-working/golang/idioms.md` | — |
| Naming conventions | `tier1-sources/python-peps/pep-008-style.md` | `tier3-working/golang/idioms.md` |
| NFR (non-functional requirements) | `tier1-sources/swebok-v4/ka01-requirements.md` | — |
| Observer pattern | `tier2-core/design-patterns/behavioral.md` | — |
| Onion Architecture | `tier2-core/architecture-patterns/ports-and-adapters.md` | `tier2-core/architecture-patterns/overview.md` |
| Optimistic concurrency | `tier2-core/architecture-patterns/aggregates.md` | `tier2-core/distributed-systems/cap-pacelc.md` |
| OCP | `tier2-core/solid-principles/ocp.md` | — |
| Open/Closed Principle | `tier2-core/solid-principles/ocp.md` | — |
| OpenAPI | `tier3-working/api-design/openapi.md` | — |
| ORM | `tier3-working/database-patterns/overview.md` | `tier1-sources/owasp/a03-injection.md` |
| OWASP | `tier1-sources/owasp/top10-2021-overview.md` | `tier1-sources/swebok-v4/ka13-security.md` |
| PACELC theorem | `tier2-core/distributed-systems/cap-pacelc.md` | — |
| Pagination | `tier3-working/api-design/rest-conventions.md` | — |
| Parameterized queries | `tier1-sources/owasp/a03-injection.md` | `tier2-core/security-practices/python-pyscg.md` |
| Partition tolerance | `tier2-core/distributed-systems/cap-pacelc.md` | — |
| Password hashing | `tier1-sources/owasp/a02-cryptographic-failures.md` | — |
| PEP 8 | `tier1-sources/python-peps/pep-008-style.md` | `tier1-sources/swebok-v4/ka04-construction.md` |
| PEP 20 | `tier1-sources/python-peps/pep-020-zen.md` | — |
| PEP 443 | `tier1-sources/python-peps/pep-443-singledispatch.md` | — |
| PEP 484 | `tier1-sources/python-peps/pep-484-type-hints.md` | `tier3-working/python/type-system.md` |
| pickle (insecure) | `tier2-core/security-practices/python-pyscg.md` | `tier1-sources/swebok-v4/ka13-security.md` |
| PII (in logs) | `tier2-core/security-practices/python-pyscg.md` | `tier3-working/observability/structured-logging.md` |
| pip-audit | `tier1-sources/owasp/a06-vulnerable-components.md` | `tier3-working/checklists/security-review.md` |
| Process model | `tier1-sources/swebok-v4/ka10-process.md` | — |
| Project management | `tier1-sources/swebok-v4/ka09-engineering-management.md` | — |
| Pull request | `tier2-core/engineering-playbook/source-control.md` | `tier3-working/checklists/code-review.md` |
| Pull request template | `tier2-core/engineering-playbook/source-control.md` | — |
| Property-based testing | `tier2-core/testing-strategies/property-based-testing.md` | `tier1-sources/swebok-v4/ka05-testing.md` |
| Protocol (Python) | `tier2-core/solid-principles/isp.md` | `tier3-working/python/type-system.md` |
| protobuf | `tier3-working/api-design/grpc.md` | — |
| Proxy pattern | `tier2-core/design-patterns/structural.md` | — |
| Pure functions | `tier3-working/python/functional-core.md` | — |
| pyscg | `tier2-core/security-practices/python-pyscg.md` | — |
| pyscg-0010 | `tier2-core/security-practices/python-pyscg.md` | `tier1-sources/owasp/a03-injection.md` |
| pyscg-0019 | `tier2-core/security-practices/python-pyscg.md` | `tier3-working/observability/structured-logging.md` |
| pyscg-0023 | `tier2-core/security-practices/python-pyscg.md` | — |
| pyscg-0037 | `tier2-core/security-practices/python-pyscg.md` | — |
| pyscg-0041 | `tier2-core/security-practices/python-pyscg.md` | — |
| Quality attributes | `tier1-sources/swebok-v4/ka12-quality.md` | `tier1-sources/swebok-v4/ka02-architecture.md` |
| Rate limiting | `tier1-sources/owasp/a07-auth-failures.md` | — |
| Regex (regular expressions) | `tier3-working/python/regex.md` | `tier2-core/security-practices/python-pyscg.md` |
| ReDoS | `tier3-working/python/regex.md` | `tier1-sources/swebok-v4/ka13-security.md` |
| README.md | `tier2-core/engineering-playbook/documentation-practices.md` | `tier2-core/engineering-playbook/source-control.md` |
| Refactoring | `tier1-sources/swebok-v4/ka07-maintenance.md` | `tier1-sources/swebok-v4/ka04-construction.md` |
| Retrospective | `tier2-core/engineering-playbook/agile-development.md` | `tier1-sources/swebok-v4/ka10-process.md` |
| Release management | `tier1-sources/swebok-v4/ka08-config-management.md` | `tier2-core/twelve-factor-app/factors.md` |
| Release automation | `tier2-core/conventional-commits/overview.md` | `tier1-sources/swebok-v4/ka08-config-management.md` |
| Read Model | `tier2-core/architecture-patterns/cqrs.md` | `tier2-core/architecture-patterns/domain-events-message-bus.md` |
| Repository pattern | `tier2-core/architecture-patterns/repository.md` | `tier3-working/database-patterns/repository-pattern.md` |
| Requirements | `tier1-sources/swebok-v4/ka01-requirements.md` | — |
| Resilience | `tier2-core/distributed-systems/resilience-patterns.md` | — |
| REST | `tier3-working/api-design/rest-conventions.md` | `tier3-working/api-design/overview.md` |
| Retry pattern | `tier2-core/distributed-systems/resilience-patterns.md` | `tier2-core/distributed-systems/fallacies.md` |
| Risk management | `tier1-sources/swebok-v4/ka09-engineering-management.md` | — |
| ROI | `tier1-sources/swebok-v4/ka15-economics.md` | — |
| Runbook | `tier1-sources/swebok-v4/ka06-operations.md` | — |
| SBOM | `tier1-sources/owasp/a06-vulnerable-components.md` | — |
| Scrum | `tier1-sources/swebok-v4/ka10-process.md` | — |
| Source control practices | `tier2-core/engineering-playbook/source-control.md` | `tier1-sources/swebok-v4/ka08-config-management.md` |
| Sprint | `tier2-core/engineering-playbook/agile-development.md` | `tier1-sources/swebok-v4/ka09-engineering-management.md` |
| Sprint planning | `tier2-core/engineering-playbook/agile-development.md` | — |
| Secret management | `tier2-core/security-practices/python-pyscg.md` | `tier1-sources/owasp/a05-security-misconfiguration.md` |
| Semantic versioning | `tier2-core/conventional-commits/overview.md` | `tier1-sources/swebok-v4/ka08-config-management.md` |
| SemVer | `tier2-core/conventional-commits/overview.md` | `tier2-core/conventional-commits/specification.md` |
| Security misconfiguration | `tier1-sources/owasp/a05-security-misconfiguration.md` | — |
| Service discovery | `tier2-core/distributed-systems/fallacies.md` | — |
| Service Layer | `tier2-core/architecture-patterns/service-layer.md` | `tier2-core/architecture-patterns/overview.md` |
| Session management | `tier1-sources/owasp/a07-auth-failures.md` | — |
| singledispatch | `tier1-sources/python-peps/pep-443-singledispatch.md` | — |
| Sockets (Python) | `tier3-working/python/sockets.md` | `tier3-working/python/urllib.md` |
| Sorting (Python) | `tier3-working/python/sorting.md` | `tier3-working/python/idioms.md` |
| Single Responsibility Principle | `tier2-core/solid-principles/srp.md` | `tier1-sources/swebok-v4/ka03-design.md` |
| SLI | `tier3-working/observability/slo-sli-sla.md` | — |
| SLO | `tier3-working/observability/slo-sli-sla.md` | `tier1-sources/swebok-v4/ka06-operations.md` |
| snake_case | `tier1-sources/python-peps/pep-008-style.md` | — |
| SOLID | `tier2-core/solid-principles/overview.md` | `tier1-sources/swebok-v4/ka03-design.md` |
| SQA (Software Quality Assurance) | `tier1-sources/swebok-v4/ka12-quality.md` | — |
| SRP | `tier2-core/solid-principles/srp.md` | — |
| SSRF | `tier1-sources/owasp/a10-ssrf.md` | — |
| Stateless processes | `tier2-core/twelve-factor-app/factors.md` | — |
| Strangler Fig | `tier2-core/architecture-patterns/legacy-migration.md` | `tier1-sources/swebok-v4/ka07-maintenance.md` |
| Status codes (HTTP) | `tier3-working/api-design/rest-conventions.md` | — |
| Strategy pattern | `tier2-core/design-patterns/behavioral.md` | `tier2-core/solid-principles/ocp.md` |
| STRIDE | `tier2-core/security-practices/threat-modeling.md` | `tier1-sources/swebok-v4/ka13-security.md` |
| Structural patterns | `tier2-core/design-patterns/structural.md` | `tier2-core/design-patterns/overview.md` |
| Structured logging | `tier3-working/observability/structured-logging.md` | — |
| Supply chain security | `tier1-sources/owasp/a08-software-integrity-failures.md` | — |
| SWEBOK | `wiki/tier1-sources/swebok-v4/overview.md` | — |
| TDD | `tier1-sources/swebok-v4/ka05-testing.md` | `tier2-core/testing-strategies/overview.md` |
| Technical debt | `tier1-sources/swebok-v4/ka07-maintenance.md` | `tier2-core/engineering-playbook/agile-development.md` |
| Time to First Commit | `tier2-core/engineering-playbook/developer-experience.md` | — |
| Template Method pattern | `tier2-core/design-patterns/behavioral.md` | — |
| Test doubles | `tier2-core/testing-strategies/test-pyramid.md` | — |
| Test pyramid | `tier2-core/testing-strategies/test-pyramid.md` | `tier1-sources/swebok-v4/ka05-testing.md` |
| Threat modeling | `tier2-core/security-practices/threat-modeling.md` | `tier1-sources/swebok-v4/ka13-security.md` |
| Timeout pattern | `tier2-core/distributed-systems/resilience-patterns.md` | — |
| TLS | `tier1-sources/owasp/a02-cryptographic-failures.md` | `tier2-core/security-practices/zero-trust.md` |
| Type hints | `tier1-sources/python-peps/pep-484-type-hints.md` | `tier3-working/python/type-system.md` |
| TypeVar | `tier3-working/python/type-system.md` | `tier1-sources/python-peps/pep-484-type-hints.md` |
| UML | `tier1-sources/swebok-v4/ka11-models-methods.md` | — |
| Unit of Work | `tier2-core/architecture-patterns/unit-of-work.md` | `tier2-core/architecture-patterns/service-layer.md` |
| Use Case | `tier2-core/architecture-patterns/service-layer.md` | `tier2-core/architecture-patterns/commands-vs-events.md` |
| Unicode | `tier3-working/python/unicode.md` | `tier1-sources/owasp/a02-cryptographic-failures.md` |
| Unit testing | `tier2-core/testing-strategies/test-pyramid.md` | `tier1-sources/swebok-v4/ka05-testing.md` |
| URL validation | `tier1-sources/owasp/a10-ssrf.md` | `tier3-working/python/ipaddress.md` |
| urllib | `tier3-working/python/urllib.md` | `tier1-sources/owasp/a10-ssrf.md` |
| Validation (where to place) | `tier2-core/architecture-patterns/validation.md` | `tier1-sources/owasp/a03-injection.md` |
| Value Object | `tier2-core/architecture-patterns/domain-model.md` | `tier2-core/architecture-patterns/aggregates.md` |
| Version control | `tier1-sources/swebok-v4/ka08-config-management.md` | `tier2-core/engineering-playbook/source-control.md` |
| Virtual environment | `tier3-working/python/overview.md` | — |
| WaitGroup (Go) | `tier3-working/golang/concurrency.md` | — |
| Working agreements | `tier2-core/engineering-playbook/agile-development.md` | — |
| with statement | `tier3-working/python/idioms.md` | `tier2-core/security-practices/python-pyscg.md` |
| XDG Base Directory | `tier2-core/xdg-base-directory/overview.md` | `tier2-core/twelve-factor-app/factors.md` |
| XDG_RUNTIME_DIR | `tier2-core/xdg-base-directory/overview.md` | — |
| XDG_STATE_HOME | `tier2-core/xdg-base-directory/overview.md` | — |
| XXE | `tier1-sources/owasp/a03-injection.md` | — |
| Zen of Python | `tier1-sources/python-peps/pep-020-zen.md` | `tier1-sources/swebok-v4/ka04-construction.md` |
| Zero Trust | `tier2-core/security-practices/zero-trust.md` | `tier1-sources/swebok-v4/ka13-security.md` |

---

## Source-to-Wiki Mapping

| Source in `references/` | Wiki Pages Generated |
|-------------------------|---------------------|
| `Software Development Best Practices for Agent.md` | All wiki pages (primary synthesis source) |
| `architecture-patterns-with-python.md` | `wiki/tier2-core/architecture-patterns/` (13 pages) |
| `w3c-skos-primer.md` | `wiki/tier1-sources/w3c/skos-primer.md` |
| `martin-fowler-bounded-context.md` | `wiki/tier1-sources/martin-fowler/bounded-context.md` |
| `domainlanguage-ddd-reference.md` | `wiki/tier1-sources/eric-evans/ddd-reference.md` |
| `ddd-crew-welcome-to-ddd.md` | `wiki/tier3-working/ddd/overview.md` |
| `awesome-ddd.md` | `wiki/tier3-working/awesome-ddd/overview.md` |
| `dotnet-microservices-architecture.md` | `wiki/tier3-working/dotnet/overview.md` |
| `OWASP_Code_Review_Guide_v2.pdf` | `wiki/tier3-working/owasp-code-review/overview.md` |
| `Google-eng-practices-review/*.html` | `wiki/tier3-working/code-review-guidelines/overview.md` |
| `xdg-basedir-spec-0.8.md` | `wiki/tier2-core/xdg-base-directory/overview.md` |

---

*This file is part of the software-backend-wiki. Updated whenever new pages are created.*
*Last updated: 2026-07-09*
