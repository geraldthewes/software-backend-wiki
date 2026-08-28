# OWASP Code Review Guide

> **Tier 3** | Source: OWASP Code Review Guide v2, OWASP Top 10:2021 | Authority: working

## Summary
The OWASP Code Review Guide provides a comprehensive framework for identifying security vulnerabilities during the code review process. This guide maps the OWASP Top 10 security risks to specific code review checkpoints, enabling teams to systematically detect and remediate security issues early in the development lifecycle. For coding agents, this guide serves as a critical resource for performing security-focused code reviews that align with industry standards and the wiki's authority tier system.

## Key Concepts

### OWASP Top 10:2021 Mapping to Code Review

| OWASP Risk | Code Review Focus | Key Checkpoints |
|------------|-------------------|-----------------|
| **A01: Broken Access Control** | Authorization enforcement | Verify server-side checks, least privilege, no bypasses |
| **A02: Cryptographic Failures** | Encryption & key management | Strong algorithms, no hardcoded secrets, TLS usage |
| **A03: Injection** | Input handling | Parameterized queries, input validation, no shell injection |
| **A04: Insecure Design** | Security architecture | Threat modeling, secure design patterns, reference architectures |
| **A05: Security Misconfiguration** | Configuration safety | Secure defaults, debugging disabled, proper headers |
| **A06: Vulnerable Components** | Dependency management | Scanning, version pinning, maintenance status |
| **A07: Identification & Authentication Failures** | Auth mechanisms | Strong auth, session management, rate limiting |
| **A08: Software & Data Integrity Failures** | Code integrity | Secure updates, CI/CD security, deserialization safety |
| **A09: Security Logging & Monitoring** | Logging practices | Event logging, no sensitive data in logs, alerting |
| **A10: Server-Side Request Forgery** | External request safety | URL validation, private IP blocking, SSRF protection |

### Security-Focused Code Review Checklist Items

#### Authentication & Session Management
- [ ] Authentication uses proven libraries/frameworks (no custom crypto)
- [ ] Passwords are hashed with strong, adaptive hashing algorithms
- [ ] Multi-factor authentication is implemented where required
- [ ] Sessions are invalidated on logout, password change, and timeout
- [ ] Session tokens are secure (HttpOnly, Secure, SameSite attributes)
- [ ] Rate limiting is applied to authentication endpoints

#### Input Validation & Injection Prevention
- [ ] All input is validated at system boundaries (whitelist where possible)
- [ ] Output encoding is applied based on context (HTML, JS, SQL, etc.)
- [ ] SQL queries use parameterized statements or ORM safely
- [ ] No shell command execution with user input (avoid `shell=True`)
- [ ] LDAP queries are parameterized to prevent injection
- [ ] XML parsers are configured to prevent XXE attacks
- [ ] No unsafe deserialization (avoid `pickle`, `eval()`, `exec()` on untrusted data)

#### Access Control
- [ ] Access controls are enforced server-side on every protected resource
- [ ] Principle of least privilege is applied (users have minimum necessary permissions)
- [ ] Authorization checks are consistent and cannot be bypassed via direct object references
- [ ] Administrative functions require additional authorization checks
- [ ] Access to business-critical functions is denied by default

#### Cryptography & Data Protection
- [ ] Strong, industry-standard algorithms are used (AES-256, RSA-2048+, ECDSA)
- [ ] Cryptographic keys are managed securely (no hardcoded secrets, use key vaults)
- [ ] TLS is used for all external communications and internal sensitive data transfer
- [ ] Passwords are salted and hashed using bcrypt, scrypt, or PBKDF2
- [ ] Sensitive data is encrypted at rest when required by regulations

#### Configuration Management
- [ ] Security settings are defined as secure defaults
- [ ] Debugging and verbose error messages are disabled in production
- [ ] Security headers are properly configured (CSP, HSTS, X-Frame-Options, etc.)
- [ ] Error handling does not leak stack traces or system information
- [ ] File uploads are restricted (type, size, scan for malware)
- [ ] Administrative interfaces are not accessible from untrusted networks

#### Logging & Monitoring
- [ ] Security-relevant events are logged (login attempts, access denials, config changes)
- [ ] Logs do not contain sensitive information (passwords, session tokens, PII)
- [ ] Logs are protected from tampering and unauthorized access
- [ ] Alerting is configured for security thresholds (failed logins, policy violations)
- [ ] Logs are retained for sufficient time to support investigations
- [ ] Log format supports parsing and analysis (structured logging preferred)

#### Dependency & Component Security
- [ ] All dependencies are scanned for known vulnerabilities (SAST/SCA tools)
- [ ] Dependency versions are explicitly pinned (no floating versions)
- [ ] Only actively maintained components are used
- [ ] License compliance is verified for all third-party components
- [ ] Unused dependencies and features are removed

#### Communication Security
- [ ] All network communications use TLS with strong cipher suites
- [ ] Certificate validation is properly implemented (hostname, chain validation)
- [ ] HSTS is enforced for web applications
- [ ] Sensitive data in URLs is avoided (use POST bodies or headers)
- [ ] External requests are validated against allowlists to prevent SSRF
- [ ] Internal services use mutual TLS where appropriate

#### File & Resource Security
- [ ] File path validation prevents directory traversal attacks
- [ ] File uploads are restricted to safe extensions and scanned for malware
- [ ] Uploaded files are stored outside web root or served via secure endpoints
- [ ] Resource consumption limits are applied (rate limiting, quotas)
- [ ] Temporary files are securely deleted after use

## Agent Guidance

### Do
- Apply the OWASP Top 10 checklist during design phase before implementation
- Treat any OWASP Top 10 violation found in code review as a blocking issue
- Link security findings to specific OWASP IDs in bug reports and code comments
- Run through A01 + A03 + A09 for every new endpoint as a mandatory gate
- Check A06 every time a dependency is added or updated
- Use automated tools (SAST, SCA) to augment manual review, not replace it
- Verify that security controls are implemented consistently across the codebase

### Do Not
- Defer security review to a separate "security sprint" - integrate into every task
- Assume a risk does not apply without explicit verification
- Rely on client-side controls alone for any Top 10 mitigations
- Treat passing a security scanner as equivalent to addressing the Top 10 (scanners miss design flaws)
- Skip security review for "small" or "urgent" changes (these often introduce vulnerabilities)
- Accept "we'll fix security later" as a plan - track debt explicitly with remediation timeline

## Process Integration

### With Threat Modeling
- Code review should verify that threat model mitigations are correctly implemented
- Review architecture diagrams against actual implementation for security controls
- Check that trust boundaries are properly validated and enforced

### With Testing
- Security test cases should be derived from OWASP Top 10 findings in code review
- Dynamic application security testing (DAST) should validate fixes found in code review
- Penetration testing should focus on areas identified as risky during code review

### With CI/CD Pipeline
- Security scanning tools should run as part of CI pipeline
- Code review findings should trigger automatic tickets in tracking systems
- Deployment pipelines should block on critical security findings
- Infrastructure as Code should undergo the same security review process

## Relationship to Other Wiki Pages

This page connects to several important wiki sections:
- Linus Torvalds review method — invariant-level security triggers (trust-boundary validation, no new public path that skips existing checks) (wiki/tier2-core/code-review-method/overview.md)
- Code review checklist (wiki/tier3-working/checklists/code-review.md)
- Security review checklist (wiki/tier3-working/checklists/security-review.md)
- SWEBOK v4 KA12 on quality assurance (wiki/tier1-sources/swebok-v4/ka12-quality.md)
- SWEBOK v4 KA13 on security (wiki/tier1-sources/swebok-v4/ka13-security.md)
- OWASP Top 10 overview (wiki/tier1-sources/owasp/top10-2021-overview.md)
- Individual OWASP risk pages (wiki/tier1-sources/owasp/a01-broken-access-control.md, etc.)

## See Also
- wiki/tier2-core/code-review-method/overview.md
- wiki/tier2-core/code-review-method/triggers.md
- wiki/tier3-working/checklists/code-review.md
- wiki/tier3-working/checklists/security-review.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md
- wiki/tier1-sources/swebok-v4/ka13-security.md
- wiki/tier1-sources/owasp/top10-2021-overview.md
- wiki/tier1-sources/owasp/a01-broken-access-control.md
- wiki/tier1-sources/owasp/a03-injection.md
- wiki/tier1-sources/owasp/a09-logging-monitoring.md

## Source
- OWASP Foundation. *OWASP Code Review Guide Version 2*. 2008. https://owasp.org/www-project-code-review-guide/
- OWASP Foundation. *OWASP Top Ten Project - 2021*. https://owasp.org/www-project-top-ten/