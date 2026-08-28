# Microsoft Engineering Fundamentals Playbook — Overview (Tier 2)

> **Tier 2** | Source: Microsoft ISE Engineering Fundamentals Playbook | Authority: established | Derives From: SWEBOK KA8, KA9, KA10, KA12

## Summary

The Microsoft Engineering Fundamentals Playbook is an open-source collection of software engineering practices maintained by Microsoft's ISE (Intelligent Systems Engineering) division. It distills field-tested guidance from hundreds of engineering engagements into actionable practices covering the full software delivery lifecycle: source control, agile development, developer experience, documentation, code reviews, observability, and security.

For a coding agent, this playbook is the practical execution layer on top of SWEBOK's theoretical foundations. Where SWEBOK defines *what* configuration management and quality assurance are, the playbook defines *how* to run them on a real team using modern tooling. Its authority derives from the breadth of Microsoft ISE's engagement history, not formal standardization — it is Tier 2: apply it unless a Tier 1 authority (SWEBOK, OWASP, PEP) conflicts.

## Coverage Map

| Playbook Section | Wiki Page | Primary SWEBOK Alignment |
|-----------------|-----------|--------------------------|
| Source Control | `wiki/tier2-core/engineering-playbook/source-control.md` | KA8 Configuration Management |
| Agile Development | `wiki/tier2-core/engineering-playbook/agile-development.md` | KA9 Engineering Management, KA10 Process |
| Developer Experience | `wiki/tier2-core/engineering-playbook/developer-experience.md` | KA6 Operations, KA4 Construction |
| Documentation | `wiki/tier2-core/engineering-playbook/documentation-practices.md` | KA12 Quality, KA9 Management |
| Code Reviews | `wiki/tier2-core/code-review-method/overview.md` + `wiki/tier2-core/engineering-playbook/source-control.md` + `checklists/code-review.md` | KA12 Quality |
| Observability | `wiki/tier3-working/observability/` (existing) | KA6 Operations |
| Security | `wiki/tier2-core/security-practices/` + OWASP (existing) | KA13 Security |

## Core Philosophy

Four principles run through every section of the playbook:

| Principle | Meaning |
|-----------|---------|
| **Mentorship over gatekeeping** | Every practice (code review, PR, retrospective) is a teaching opportunity, not a quality gate alone |
| **Quality through process** | Automated gates — CI, linters, required reviewers — over individual discipline |
| **Iterative delivery** | Small, frequent, customer-validated increments over big-bang releases |
| **Full lifecycle ownership** | Teams own design, code, test, deploy, and operate — no hand-off mentality |

## Agent Guidance

### Do

- Use specific sub-pages (source-control.md, agile-development.md, etc.) rather than reasoning from this overview.
- Treat the playbook as the *operational* reference for source control, agile, and developer experience decisions.
- Defer to SWEBOK Tier 1 pages when playbook guidance conflicts with formal standards.

### Do Not

- Do not treat playbook guidance as immutable — it is Tier 2 and context-dependent.
- Do not use the playbook as a substitute for OWASP controls or SWEBOK security guidance (those are Tier 1).

## Checklist

- [ ] Relevant sub-page consulted (not just this overview)
- [ ] Playbook guidance checked against applicable SWEBOK KA before applying
- [ ] Conflicts with Tier 1 resolved in favor of Tier 1

## See Also

- wiki/tier2-core/engineering-playbook/source-control.md
- wiki/tier2-core/engineering-playbook/agile-development.md
- wiki/tier2-core/engineering-playbook/developer-experience.md
- wiki/tier2-core/engineering-playbook/documentation-practices.md
- wiki/tier2-core/code-review-method/overview.md
- wiki/tier1-sources/swebok-v4/ka08-config-management.md
- wiki/tier1-sources/swebok-v4/ka09-engineering-management.md
- wiki/tier1-sources/swebok-v4/ka10-process.md

## Source

Microsoft ISE Engineering Fundamentals Playbook. Open-source, MIT License.
https://microsoft.github.io/code-with-engineering-playbook/
