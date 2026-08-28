# AGENTS.md — Wiki Protocol for Coding Agents

> **Mandatory**: Read this file completely before reading any other file in this wiki.
> Every agent session begins here. This file defines how to navigate, interpret, and maintain the wiki.

---

## 1. Purpose

This wiki is the authoritative knowledge base for software engineering tasks: design, coding, testing, security, and review. It grounds every agent decision in professional standards — primarily IEEE SWEBOK v4, OWASP, and Python PEPs — and escalates through a tiered authority system that prevents lower-priority content from overriding higher-priority standards.

The wiki follows the **Karpathy LLM Wiki pattern**: raw sources stay immutable in `references/`, all synthesized knowledge lives in `wiki/` organized by authority tier, and three root-level files (`AGENTS.md`, `index.md`, `KNOWLEDGE_GRAPH.md`) form the navigation schema.

---

## 2. Mandatory First Actions

Execute in this exact order at the start of any session:

1. **Read `AGENTS.md`** (this file) — complete, not skimmed
2. **Read `index.md`** — locate the sections relevant to your current task
3. **Identify the task type** and follow the Task Routing Table (Section 4)
4. **Read Tier 1 pages** for the relevant domain
5. **Read Tier 2 pages** for applicable practices
6. **Read Tier 3 pages** for language guides, checklists, and worked examples
7. **Apply the relevant checklists** before producing output

---

## 3. Authority Tier System

Content is organized into four tiers by directory. When content in a lower tier conflicts with a higher tier, the **higher tier always wins**. Contradicting Tier 1 is an agent error.

| Tier | Directory | Sources | Rule |
|------|-----------|---------|------|
| **1** | `wiki/tier1-sources/` | IEEE SWEBOK v4, OWASP Top 10, Python PEPs, ACM/IEEE-CS Code of Ethics | **Immutable authority. Never contradicted. If your implementation conflicts with Tier 1, you are wrong — revise.** |
| **2** | `wiki/tier2-core/` | SOLID principles, 12-Factor App, CAP/PACELC, GoF design patterns, Pyscg | Applies unless Tier 1 overrides. These are established practices derived from Tier 1 standards. |
| **3** | `wiki/tier3-working/` | Language guides, checklists, worked examples, observability, API design | Context-dependent. Check that Tier 3 guidance aligns with Tier 2 and Tier 1 before applying. |
| **4** | `wiki/tier4-archive/` | Superseded or deprecated content | **Do not use** without explicit human instruction. |

**Resolution rule**: Tier 1 → Tier 2 → Tier 3 → Tier 4 (descending authority).

---

## 4. Task Routing Table

For each task type, read the listed files **in order** before producing output:

| Task Type | Read First | Then Read | Apply Checklist |
|-----------|-----------|-----------|-----------------|
| **Design a service/component** | `wiki/tier1-sources/swebok-v4/ka02-architecture.md` | `wiki/tier2-core/solid-principles/overview.md`, `wiki/tier2-core/twelve-factor-app/overview.md` | `wiki/tier3-working/checklists/design-review.md` |
| **Detailed design (classes/modules)** | `wiki/tier1-sources/swebok-v4/ka03-design.md` | `wiki/tier2-core/solid-principles/overview.md`, `wiki/tier2-core/design-patterns/overview.md` | `wiki/tier3-working/checklists/design-review.md` |
| **Write Python code** | `wiki/tier1-sources/swebok-v4/ka04-construction.md` | `wiki/tier1-sources/python-peps/pep-008-style.md`, `wiki/tier1-sources/python-peps/pep-484-type-hints.md`, `wiki/tier3-working/python/idioms.md` | `wiki/tier3-working/checklists/pre-commit.md` |
| **Write Go code** | `wiki/tier1-sources/swebok-v4/ka04-construction.md` | `wiki/tier3-working/golang/idioms.md`, `wiki/tier3-working/golang/toolchain.md` | `wiki/tier3-working/checklists/pre-commit.md` |
| **Write tests** | `wiki/tier1-sources/swebok-v4/ka05-testing.md` | `wiki/tier2-core/testing-strategies/test-pyramid.md`, `wiki/tier2-core/testing-strategies/property-based-testing.md` | `wiki/tier3-working/checklists/testing-review.md` |
| **Security review** | `wiki/tier1-sources/swebok-v4/ka13-security.md` | `wiki/tier1-sources/owasp/top10-2021-overview.md`, `wiki/tier2-core/security-practices/python-pyscg.md` | `wiki/tier3-working/checklists/security-review.md` |
| **Code review** | `wiki/tier1-sources/swebok-v4/ka12-quality.md` | `wiki/tier2-core/code-review-method/overview.md`, `wiki/tier2-core/code-review-method/triggers.md`, `wiki/tier3-working/checklists/code-review.md` | `wiki/tier3-working/checklists/security-review.md` |
| **Distributed system design** | `wiki/tier2-core/distributed-systems/cap-pacelc.md` | `wiki/tier2-core/distributed-systems/fallacies.md`, `wiki/tier2-core/twelve-factor-app/factors.md` | `wiki/tier3-working/checklists/design-review.md` |
| **Database work** | `wiki/tier3-working/database-patterns/overview.md` | `wiki/tier3-working/database-patterns/repository-pattern.md`, `wiki/tier1-sources/owasp/a03-injection.md` | `wiki/tier3-working/checklists/security-review.md` |
| **API design** | `wiki/tier3-working/api-design/overview.md` | `wiki/tier3-working/api-design/rest-conventions.md`, `wiki/tier1-sources/owasp/a01-broken-access-control.md` | `wiki/tier3-working/checklists/design-review.md` |
| **Observability / logging** | `wiki/tier3-working/observability/overview.md` | `wiki/tier3-working/observability/structured-logging.md`, `wiki/tier1-sources/owasp/a09-logging-monitoring.md` | `wiki/tier3-working/checklists/code-review.md` |
| **Ethics / professional judgment** | `wiki/tier1-sources/acm-ieee-ethics/code-of-ethics.md` | `wiki/tier1-sources/swebok-v4/ka14-professional-practice.md` | — |
| **Release / deployment** | `wiki/tier1-sources/swebok-v4/ka06-operations.md` | `wiki/tier2-core/twelve-factor-app/factors.md`, `wiki/tier3-working/observability/slo-sli-sla.md` | `wiki/tier3-working/checklists/pre-commit.md` |
| **Write commit messages** | `wiki/tier2-core/conventional-commits/overview.md` | `wiki/tier2-core/conventional-commits/specification.md` | `wiki/tier3-working/checklists/pre-commit.md` |
| **Set up / manage source control** | `wiki/tier2-core/engineering-playbook/source-control.md` | `wiki/tier1-sources/swebok-v4/ka08-config-management.md` | `wiki/tier3-working/checklists/pre-commit.md` |
| **Sprint planning / backlog management** | `wiki/tier2-core/engineering-playbook/agile-development.md` | `wiki/tier1-sources/swebok-v4/ka09-engineering-management.md` | — |
| **Developer experience / onboarding** | `wiki/tier2-core/engineering-playbook/developer-experience.md` | `wiki/tier2-core/engineering-playbook/documentation-practices.md` | `wiki/tier3-working/checklists/pre-commit.md` |
| **Write or review documentation** | `wiki/tier2-core/engineering-playbook/documentation-practices.md` | `wiki/tier3-working/api-design/openapi.md` | `wiki/tier3-working/checklists/code-review.md` |
| **Design a domain-driven service** | `wiki/tier2-core/architecture-patterns/overview.md` | `wiki/tier2-core/architecture-patterns/ports-and-adapters.md`, `wiki/tier2-core/architecture-patterns/domain-model.md`, `wiki/tier2-core/architecture-patterns/service-layer.md`, `wiki/tier2-core/architecture-patterns/unit-of-work.md`, `wiki/tier2-core/architecture-patterns/aggregates.md` | `wiki/tier3-working/checklists/design-review.md` |
| **Build an event-driven service** | `wiki/tier2-core/architecture-patterns/domain-events-message-bus.md` | `wiki/tier2-core/architecture-patterns/commands-vs-events.md`, `wiki/tier2-core/architecture-patterns/event-driven-integration.md`, `wiki/tier2-core/architecture-patterns/cqrs.md` | `wiki/tier3-working/checklists/design-review.md` |
| **Migrate a legacy monolith** | `wiki/tier2-core/architecture-patterns/legacy-migration.md` | `wiki/tier1-sources/swebok-v4/ka02-architecture.md`, `wiki/tier2-core/architecture-patterns/ports-and-adapters.md` | `wiki/tier3-working/checklists/design-review.md` |

---

## 5. Navigation Conventions

### File Naming Patterns

| Pattern | Meaning |
|---------|---------|
| `ka##-name.md` | SWEBOK V4 Knowledge Area (e.g., `ka05-testing.md`) |
| `a##-name.md` | OWASP Top 10 vulnerability (e.g., `a03-injection.md`) |
| `pep-###-name.md` | Python Enhancement Proposal |
| `*-review.md` | A review checklist (use at end of task) |
| `pre-commit.md` | Pre-commit/push checklist |
| `overview.md` | Entry point for a directory/topic |

### See Also Links

Every wiki page ends with a **See Also** section using wiki-root-relative paths:
```
## See Also
- wiki/tier1-sources/swebok-v4/ka05-testing.md
- wiki/tier2-core/testing-strategies/test-pyramid.md
```

### KNOWLEDGE_GRAPH.md Navigation Paths

`KNOWLEDGE_GRAPH.md` contains named navigation paths for common multi-step workflows. Use these when you need to traverse multiple related pages rather than discovering the path yourself.

---

## 6. Standard Page Template

Every wiki page (except stubs and checklists) uses this structure in order:

```markdown
# [Title] ([Tier label])

> **Tier N** | Source: [canonical source] | Authority: [immutable / established / working]

## Summary
1–2 paragraphs: what this topic is and why it matters specifically to a coding agent.

## Key Concepts
Definitions, rules, classifications. Use tables where items have multiple attributes.

## Agent Guidance

### Do
- Specific, actionable, imperative statements

### Do Not
- Specific prohibitions with reasons

## Checklist
- [ ] Item one
- [ ] Item two

## See Also
- Relative paths to 3–8 related wiki pages

## Source
Full citation: author, document name, version, URL or reference doc section.
```

**Checklist pages** (in `tier3-working/checklists/`) omit Summary and Key Concepts; they consist entirely of task-grouped checkbox items followed by See Also.

**Worked example pages** (in `tier3-working/worked-examples/`) include a Problem statement, full executable code, and an annotation of which principles the code demonstrates.

---

## 7. Ingest Workflow

When adding a new reference source to the wiki:

1. Place the raw document in `references/` (read-only; never edited after ingest)
2. Determine its authority tier (Tier 1 = IEEE/OWASP/PEP standard; Tier 2 = established practice; Tier 3 = tooling/examples)
3. Create one or more wiki pages following the Standard Page Template
4. Add entries to `index.md`: catalog table row + keyword index entries
5. Add relationships to `KNOWLEDGE_GRAPH.md`: new entities + edges
6. Append an entry to `log.md`:

```
| YYYY-MM-DD | CREATE | wiki/path/to/file.md | references/source.md | agent | Brief note |
```

7. Do **not** delete or modify existing Tier 1 pages without explicit human approval

---

## 8. Maintenance Rules

- **LLMs maintain this wiki**; humans approve changes to Tier 1 content
- **Never truncate** existing content — append or create new pages
- **No orphan pages** — every new page must be linked from `index.md` and referenced by at least one other page
- **No silent updates** — every material change to a page appends a `log.md` entry
- **Idempotent ingest** — re-running ingest on an existing source updates existing pages rather than creating duplicates (file path = page identity)
- **Cross-reference density** — every page links to at least 3 other pages in See Also; Tier 1 priority pages link to 8+

---

## 9. Quick Reference: Named Navigation Paths

These paths are defined in full in `KNOWLEDGE_GRAPH.md`. Abbreviated here for quick lookup:

| Goal | Navigation Path |
|------|----------------|
| Design a new service | ka02 → ka03 → solid-principles → twelve-factor-app → design-review |
| Write production Python | ka04 → pep-008 → pep-484 → python/idioms → pre-commit |
| Write production Go | ka04 → golang/idioms → golang/toolchain → pre-commit |
| Test a module | ka05 → test-pyramid → property-based-testing → mutation-testing → testing-review |
| Security audit | ka13 → owasp/top10 → python-pyscg → security-review |
| Review a PR | ka12 → code-review-method → triggers → code-review checklist → security-review |
| Build a distributed system | ka02 → cap-pacelc → fallacies → resilience-patterns → twelve-factor-app |
| Design a REST API | api-design/overview → rest-conventions → openapi → a01 + a03 → design-review |
| Add observability | observability/overview → structured-logging → metrics → slo-sli-sla |
| Write commit messages | conventional-commits/overview → conventional-commits/specification → pre-commit |
| Set up source control | engineering-playbook/source-control → ka08-config-management → pre-commit |
| Plan a sprint | engineering-playbook/agile-development → ka09-engineering-management |
| Build a domain-driven service | architecture-patterns/overview → ports-and-adapters → domain-model → repository → service-layer → unit-of-work → aggregates → dependency-injection-bootstrap |
| Build an event-driven service | architecture-patterns/overview → domain-events-message-bus → commands-vs-events → event-driven-integration → cqrs |
| Migrate a legacy monolith | architecture-patterns/legacy-migration → ka02-architecture → ports-and-adapters |
| Onboard a developer | engineering-playbook/developer-experience → engineering-playbook/documentation-practices |
| Write documentation | engineering-playbook/documentation-practices → api-design/openapi (if API) |

---

*This file is part of the software-backend-wiki. Maintained by LLM agents; Tier 1 changes require human approval.*
*Last updated: 2026-08-26*
