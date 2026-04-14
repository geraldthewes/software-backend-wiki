# SWEBOK V4 Overview

> **Tier 1** | Source: IEEE Computer Society, SWEBOK V4 (2024) | Authority: immutable

## Summary

The Guide to the Software Engineering Body of Knowledge (SWEBOK) is the IEEE Computer Society's authoritative consensus standard defining the boundaries and content of the software engineering discipline. It establishes what every practitioner must know to engage in "systematic, disciplined, and quantifiable" development — the three adjectives that distinguish engineering from ad-hoc programming. For a coding agent, SWEBOK is the foundational ontology: before generating a line of code, the agent must locate the task within one or more Knowledge Areas (KAs) and apply the relevant principles from that KA.

SWEBOK V4, released in 2024, is a significant revision of the previous V3 (2014). V4 expands from 15 to 18 Knowledge Areas, integrates Agile and DevOps as first-class methods throughout every KA rather than treating them as alternative practices, and elevates Software Architecture and Software Security into standalone KAs. This restructuring reflects a decade of industry evolution: cloud-native distributed systems, continuous delivery pipelines, and security-by-design practices are now baseline expectations, not advanced topics.

## Key Concepts

### V3 vs. V4 Comparison

| Dimension | SWEBOK V3 (2014) | SWEBOK V4 (2024) |
|-----------|-----------------|-----------------|
| Knowledge Areas | 15 | 18 |
| Agile/DevOps | Appendix / footnotes | Integrated throughout all KAs |
| Architecture | Sub-topic of Design KA | Standalone KA (KA 02) |
| Security | Sub-topic of Quality KA | Standalone KA (KA 13) |
| Operations | Absent | Standalone KA (Engineering Operations) |
| Emphasis | Process compliance | Systematic engineering + delivery speed |

### Three New KAs in V4

1. **Software Architecture (KA 02):** Elevated from inside the Design KA. Reflects the centrality of architectural decisions in distributed and cloud systems. Covers ADRs, fitness functions, CAP/PACELC, and the 4+1 view model.
2. **Engineering Operations (KA 17):** Covers CI/CD pipeline automation, incident response, SLIs/SLOs, and runbook practices. Formalizes the DevOps operational loop.
3. **Software Security (KA 13):** Elevated from Quality. Covers CIA Triad, STRIDE threat modeling, OWASP Top 10, and Python-specific Pyscg rules.

### The 18 Knowledge Areas

| # | Knowledge Area | One-Line Description |
|---|---------------|----------------------|
| 01 | Software Requirements | Elicitation, analysis, specification, and validation of stakeholder needs |
| 02 | Software Architecture | Fundamental structures: components, connectors, constraints, and quality attributes |
| 03 | Software Design | Detailed design of modules, classes, interfaces, and design patterns |
| 04 | Software Construction | Coding, unit verification, integration, debugging, and code standards |
| 05 | Software Testing | Verification, validation, test levels, techniques, and test automation |
| 06 | Software Engineering Operations | CI/CD, deployments, monitoring, incident response, SLOs |
| 07 | Software Maintenance | Evolution, bug fixing, refactoring, and end-of-life migration |
| 08 | Software Configuration Management | Version control, branching, build management, change tracking |
| 09 | Software Engineering Management | Project planning, estimation, risk management, measurement |
| 10 | Software Engineering Process | Process definition, assessment, and improvement models (CMMI, etc.) |
| 11 | Software Engineering Models and Methods | Agile, waterfall, spiral, DevOps, and formal methods |
| 12 | Software Quality | Quality attributes, SQA, reviews, inspections, and metrics |
| 13 | Software Security | CIA Triad, STRIDE, OWASP, secure coding, and privacy by design |
| 14 | Software Engineering Professional Practice | Ethics, communication, and team dynamics |
| 15 | Software Engineering Economics | Cost-benefit analysis, ROI, make-or-buy, and technical debt |
| 16 | Computing Foundations | Algorithms, data structures, OS, networking fundamentals |
| 17 | Mathematical Foundations | Logic, set theory, probability, and formal verification |
| 18 | Engineering Foundations | Systems thinking, measurement theory, and engineering trade-offs |

### How to Navigate to Individual KA Pages

Each KA has its own deep-dive page within this wiki. The naming convention is `kaNN-shortname.md` where `NN` is the zero-padded KA number.

| Priority KA | Wiki Path |
|-------------|-----------|
| Architecture | `wiki/tier1-sources/swebok-v4/ka02-architecture.md` |
| Design | `wiki/tier1-sources/swebok-v4/ka03-design.md` |
| Construction | `wiki/tier1-sources/swebok-v4/ka04-construction.md` |
| Testing | `wiki/tier1-sources/swebok-v4/ka05-testing.md` |
| Quality | `wiki/tier1-sources/swebok-v4/ka12-quality.md` |
| Security | `wiki/tier1-sources/swebok-v4/ka13-security.md` |

## Agent Guidance

### Do
- Read the relevant KA page before beginning any task in that domain (e.g., read `ka05-testing.md` before writing tests)
- Treat all Tier 1 SWEBOK pages as read-only authority; if agent output contradicts a KA page, the agent is wrong
- Apply Agile and DevOps practices (continuous integration, short cycles, automated testing) as mandated throughout V4
- Cross-reference multiple KAs when a task spans domains (e.g., construction + testing + security for a new endpoint)
- Cite the relevant KA number when explaining an architectural or design decision

### Do Not
- Treat SWEBOK V3 content as current — V4 supersedes it in all areas
- Treat Architecture, Security, or Operations as secondary concerns embedded in other KAs — each is now standalone
- Skip KA guidance because a task "seems small" — even a one-line change can have architecture, security, or quality implications
- Confuse programming (a sub-activity of KA 04 Construction) with software engineering (the full 18-KA lifecycle)

## Checklist
- [ ] Identified which KA(s) apply to the current task
- [ ] Read the relevant KA wiki page(s) before beginning
- [ ] Confirmed that Agile/DevOps practices are applied where relevant
- [ ] Verified that security (KA 13) is considered even for non-security-focused tasks

## See Also
- wiki/tier1-sources/swebok-v4/ka02-architecture.md
- wiki/tier1-sources/swebok-v4/ka03-design.md
- wiki/tier1-sources/swebok-v4/ka04-construction.md
- wiki/tier1-sources/swebok-v4/ka05-testing.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md
- wiki/tier1-sources/swebok-v4/ka13-security.md

## Source

IEEE Computer Society. *Guide to the Software Engineering Body of Knowledge (SWEBOK), Version 4.0*. IEEE Press, 2024. https://www.computer.org/education/bodies-of-knowledge/software-engineering
