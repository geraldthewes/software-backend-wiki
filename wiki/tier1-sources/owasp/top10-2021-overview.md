# OWASP Top 10:2021 Overview

> **Tier 1** | Source: OWASP Top 10:2021 | Authority: immutable

## Summary

The Open Worldwide Application Security Project (OWASP) Top 10 is the most widely adopted security awareness document for web application security. Published by a global non-profit foundation of security experts, it represents broad consensus about the most critical security risks to web applications. For a coding agent, the Top 10 is the baseline security checklist that must be applied to every feature, endpoint, and dependency decision.

## Key Concepts

OWASP is a non-profit that produces freely available security guidance. The Top 10 list is updated periodically based on real-world vulnerability data from hundreds of organizations. The 2021 edition reflects a data-driven approach covering 500,000+ applications. It is considered **authoritative and immutable** — not a preference, but a minimum professional standard.

## Top 10 Risks Reference Table

| Rank | ID  | Name                              | Root Cause                                    | Primary Mitigation                                      |
|------|-----|-----------------------------------|-----------------------------------------------|---------------------------------------------------------|
| 1    | A01 | Broken Access Control             | Missing server-side enforcement               | Least privilege; server-side checks on every request    |
| 2    | A02 | Cryptographic Failures            | Weak or missing encryption                    | Use `cryptography` lib; TLS everywhere; no MD5/SHA1     |
| 3    | A03 | Injection                         | Unsanitized input in commands or queries      | Parameterized queries; input validation; no shell=True  |
| 4    | A04 | Insecure Design                   | Architecture-level security flaws             | Threat modeling (STRIDE); reference architectures       |
| 5    | A05 | Security Misconfiguration         | Default or incomplete configuration           | Hardened defaults; config scanning; no DEBUG in prod    |
| 6    | A06 | Vulnerable Components             | Outdated or insecure dependencies             | pip-audit in CI; SBOM; pin all versions                 |
| 7    | A07 | Identification & Auth Failures    | Weak auth or session management               | Strong auth libraries; session timeouts; rate limiting  |
| 8    | A08 | Software & Data Integrity Failures| Unverified updates; unsafe deserialization    | CI signing; avoid pickle; verify artifact checksums     |
| 9    | A09 | Security Logging & Monitoring     | Inadequate logging; no alerting               | Structured logging; alerts on failure thresholds        |
| 10   | A10 | Server-Side Request Forgery       | Unvalidated URL fetching                      | Allowlist external URLs; block metadata endpoints       |

## Quick Reference: When to Apply Each Risk

**For every new endpoint, verify:**
- A01 — Is access control enforced server-side for this route?
- A03 — Is all user input parameterized or escaped before use in queries/commands?
- A09 — Are authentication events and errors logged with sufficient detail?

**For every new dependency added:**
- A06 — Does this package have known CVEs? Is it pinned? Is it actively maintained?

**For every auth or session feature:**
- A07 — Is auth delegated to a proven library? Are sessions invalidated on logout?

**For any URL-fetching feature:**
- A10 — Are user-supplied URLs validated against an allowlist?

**For any deserialization or update mechanism:**
- A08 — Is pickle avoided? Are update artifacts verified?

## Agent Guidance

### Do
- Apply this checklist at the design phase before writing any implementation code
- Run through A01 + A03 + A09 for every new endpoint as a mandatory gate
- Check A06 every time a dependency is added or updated
- Treat any Top 10 violation found in code review as a blocking issue, not a suggestion
- Link findings to the specific OWASP ID in bug reports and code comments

### Do Not
- Defer security review to a separate "security sprint" — integrate it into every task
- Assume a risk does not apply without explicitly verifying it
- Rely on client-side controls alone for any of the Top 10 mitigations
- Treat passing a security scanner as equivalent to addressing the Top 10 — scanners miss design flaws (A04)

## Checklist
- [ ] A01: Server-side access control check on every protected endpoint
- [ ] A02: No MD5/SHA1 for passwords; TLS on all external connections; no hardcoded keys
- [ ] A03: Parameterized queries used for all SQL; no shell=True with user input
- [ ] A04: Threat model reviewed before implementing auth, payment, or PII features
- [ ] A05: DEBUG=False in production; security headers configured; no default credentials
- [ ] A06: pip-audit passes in CI; all dependency versions pinned
- [ ] A07: Auth delegated to proven library; sessions invalidated on logout; rate limiting on login
- [ ] A08: No pickle on untrusted data; CI pipeline permissions minimized
- [ ] A09: Auth events logged; no PII in logs; alerts defined for failure thresholds
- [ ] A10: URL allowlist enforced; private IP ranges and metadata endpoints blocked

## See Also
- wiki/tier1-sources/owasp/a01-broken-access-control.md
- wiki/tier1-sources/owasp/a02-cryptographic-failures.md
- wiki/tier1-sources/owasp/a03-injection.md
- wiki/tier1-sources/owasp/a04-insecure-design.md
- wiki/tier1-sources/owasp/a05-security-misconfiguration.md
- wiki/tier1-sources/owasp/a06-vulnerable-components.md
- wiki/tier1-sources/owasp/a07-auth-failures.md
- wiki/tier1-sources/owasp/a08-software-integrity-failures.md
- wiki/tier1-sources/owasp/a09-logging-monitoring.md
- wiki/tier1-sources/owasp/a10-ssrf.md

## Source
OWASP Top 10:2021. https://owasp.org/Top10/
