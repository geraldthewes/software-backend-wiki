# Security Practices — Overview

> **Tier 2** | Source: OWASP, OpenSSF Pyscg, NIST SP 800-207 | Derives From: ka13-security, owasp/top10 | Authority: established practice

## Summary

Security must be an integrated part of the development cycle, not an afterthought. For backend services, security failures fall into three broad categories: insecure code (injection, unsafe deserialization, unprotected secrets), insecure design (missing threat modeling, improper access control), and insecure infrastructure (flat network trust, long-lived credentials). The three sub-pages address each category with actionable, agent-ready guidance.

## Key Concepts

### Why Security Practices Are Tier 2

Security practices derive from authoritative standards:
- **ka13-security** (SWEBOK): the foundational knowledge area for software security
- **OWASP Top 10**: the industry consensus on the most critical web application security risks, updated regularly
- **OpenSSF Pyscg**: Python-specific secure coding guidelines from the Open Source Security Foundation

These standards are the Tier 1 sources. The Tier 2 pages here translate them into agent-actionable guidance with concrete Python examples.

### The Three Sub-Topics

| Sub-Page | What It Covers | When to Apply |
|----------|---------------|--------------|
| `python-pyscg.md` | Python Secure Coding Guidelines (Pyscg) | During code construction — every function that handles input, secrets, or resources |
| `threat-modeling.md` | STRIDE threat modeling | At design time — before writing security-relevant code |
| `zero-trust.md` | Zero Trust Architecture | During service and infrastructure design |

### Quick Cross-Reference: Most Critical Rules

| Priority | Rule | Document |
|----------|------|----------|
| Critical | Use parameterized queries — never string-format SQL | `python-pyscg.md` (pyscg-0010) |
| Critical | Never log passwords, tokens, or PII | `python-pyscg.md` (pyscg-0019) |
| Critical | Never use pickle on untrusted data | `python-pyscg.md` (pyscg-0023) |
| Critical | Use `with` for all resource management | `python-pyscg.md` (pyscg-0035, 0052) |
| Critical | Never use `assert` for security checks | `python-pyscg.md` (pyscg-0037) |
| Critical | All config and secrets via environment variables | `python-pyscg.md` (pyscg-0041) |
| High | Do STRIDE before writing auth or payment code | `threat-modeling.md` |
| High | Authenticate every service-to-service request | `zero-trust.md` |
| High | Use mTLS for internal service communication | `zero-trust.md` |

## Agent Guidance

### Do
- Apply pyscg rules during construction — they are fast to apply and high-impact
- Run `bandit` on all Python code before declaring it production-ready
- Perform STRIDE threat modeling before writing authentication, payment, PII-handling, or admin code
- Treat internal network requests with the same skepticism as external requests (Zero Trust)

### Do Not
- Do not treat security as a final review step — integrate it at design and construction time
- Do not hardcode secrets, credentials, or API keys in source code
- Do not skip threat modeling for "simple" features — attackers find the simple cases too

## Checklist
- [ ] `bandit` passes with no high-severity findings
- [ ] No hardcoded secrets in source (checked with `git grep` or `truffleHog`)
- [ ] STRIDE threat model completed for security-relevant components
- [ ] All service-to-service calls use authentication (JWT, mTLS, or equivalent)
- [ ] All pyscg rules applied to code handling user input, secrets, or resources

## See Also
- `wiki/tier2-core/security-practices/python-pyscg.md`
- `wiki/tier2-core/security-practices/threat-modeling.md`
- `wiki/tier2-core/security-practices/zero-trust.md`
- `wiki/tier1-sources/swebok-v4/ka13-security.md`
- `wiki/tier1-sources/owasp/top10-2021-overview.md`

## Source

OWASP Top 10 (2021); OpenSSF Pyscg; NIST SP 800-207 Zero Trust Architecture. Synthesized from *Software Development Best Practices for Agent* reference document.
