# Security Review Checklist

> **Tier 3** | Enforces: wiki/tier1-sources/owasp/top10-2021-overview.md, wiki/tier1-sources/swebok-v4/ka13-security.md, wiki/tier1-sources/owasp/a01-broken-access-control.md, wiki/tier1-sources/owasp/a03-injection.md

Apply this checklist to every change that touches authentication, authorization, data handling, external calls, or cryptography. Apply it fully to any new service.

## Input Handling (OWASP A03 — Injection, A04 — Insecure Design)

- [ ] All inputs from external sources (HTTP, message queue, file upload, CLI args) validated at the system boundary before use
- [ ] SQL queries use parameterized form only — no string formatting, no f-string interpolation in SQL
- [ ] No `shell=True` with any string derived from user input (command injection)
- [ ] No `subprocess` with user-controlled arguments without explicit allow-list validation
- [ ] No `eval()`, `exec()`, or `compile()` on any untrusted string
- [ ] No `pickle.loads()` or `yaml.load()` (without `Loader=SafeLoader`) on untrusted data (pyscg-0023)
- [ ] File upload paths validated and sanitized — no path traversal (`..\..` or absolute paths from user input)
- [ ] XML parsing protected against XXE attacks (use `defusedxml` in Python)
- [ ] HTML output from user content escaped before rendering (XSS prevention)

## Authentication and Authorization (OWASP A01 — Broken Access Control, A07 — Auth Failures)

- [ ] Authentication required on all endpoints that return or modify non-public data
- [ ] Authorization checked server-side on every request — not just on login
- [ ] Principle of Least Privilege applied — tokens/roles grant minimum permissions needed
- [ ] Permission check is performed at the resource level, not just the URL level
- [ ] No authorization decisions based solely on client-provided data (user ID in request body)
- [ ] Session tokens are random, sufficiently long (≥ 128 bits), and stored securely
- [ ] Session invalidation on logout is verified — token is actually expired or revoked
- [ ] Failed authentication attempts do not reveal whether the user exists (`User not found` vs `Invalid password`)
- [ ] Password reset tokens expire within 1 hour and are single-use
- [ ] Account lockout or rate limiting applied after repeated failed authentication attempts

## Cryptography (OWASP A02 — Cryptographic Failures)

- [ ] No "rolling your own" crypto — use `cryptography` library or OS primitives
- [ ] No MD5 or SHA-1 for security purposes (use SHA-256 or SHA-3)
- [ ] No DES, RC4, or 1024-bit RSA (use AES-256-GCM, RSA-2048+, or Ed25519)
- [ ] Passwords hashed with bcrypt, scrypt, argon2, or PBKDF2 — never plaintext or MD5/SHA-1
- [ ] TLS required for all network connections carrying sensitive data
- [ ] TLS version ≥ 1.2; TLS 1.3 preferred where available
- [ ] Secret keys and certificates not stored in source code — loaded from environment or secret manager

## Data Handling

- [ ] PII (email, name, phone, address) identified; retention policy defined and enforced
- [ ] Sensitive data fields encrypted at rest where required by policy or regulation
- [ ] No sensitive data in URLs or query strings (logged by proxies and servers)
- [ ] Sensitive data not logged — passwords, tokens, full card numbers, SSNs excluded from logs (pyscg-0019)
- [ ] Database records with sensitive data have access controls limiting which services can read them
- [ ] Data deleted when no longer needed — no indefinite retention of user data

## Dependencies (OWASP A06 — Vulnerable and Outdated Components)

- [ ] Dependency vulnerability scan passed: `pip-audit` (Python) or `govulncheck` (Go)
- [ ] No dependencies with known high/critical CVEs without documented mitigation
- [ ] `pip-audit` or `safety check` integrated into CI — fails the build on new critical CVEs
- [ ] No direct use of `eval`, `exec`, or dynamic code from external packages without review
- [ ] Third-party code changes reviewed before dependency version bump

## Logging and Monitoring (OWASP A09 — Logging and Monitoring Failures)

- [ ] Authentication events logged: successful login, failed login, logout, token refresh
- [ ] Authorization failures logged: resource, action, and identity that was denied
- [ ] All ERROR and CRITICAL conditions logged with sufficient context for investigation
- [ ] Logs do not contain secrets, tokens, passwords, or raw PII
- [ ] Security-relevant events generate alerts (repeated auth failures, permission escalation attempts)
- [ ] Log entries include correlation ID to enable end-to-end request tracing
- [ ] Audit log written for all changes to sensitive data (who, what, when)

## Distributed and External Calls

- [ ] SSRF protection: outbound HTTP calls validate target host against an allow-list if URL is user-influenced (OWASP A10)
- [ ] mTLS or API key authentication used for service-to-service calls
- [ ] Timeouts set on all outbound HTTP/gRPC calls
- [ ] Error responses from external services do not expose internal stack traces to callers
- [ ] Sensitive headers (Authorization, Cookie) stripped from logs and error messages

## See Also

- wiki/tier3-working/checklists/code-review.md
- wiki/tier1-sources/owasp/a01-broken-access-control.md
- wiki/tier1-sources/owasp/a02-cryptographic-failures.md
- wiki/tier1-sources/owasp/a03-injection.md
- wiki/tier1-sources/owasp/a09-logging-monitoring.md
- wiki/tier1-sources/swebok-v4/ka13-security.md

## Source

OWASP Top 10 2021. OpenSSF Pyscg Secure Coding Guidelines. SWEBOK V4, KA13 Security. NIST SP 800-53 (Security and Privacy Controls). CWE Top 25 Most Dangerous Software Weaknesses.
