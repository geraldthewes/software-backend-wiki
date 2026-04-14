# SWEBOK V4 — KA 13: Software Security

> **Tier 1** | Source: IEEE SWEBOK V4, KA 13; OWASP Top 10; OpenSSF Pyscg | Authority: immutable

## Summary

Software Security is the discipline of designing, implementing, and operating software systems so that they maintain confidentiality, integrity, and availability in the presence of adversaries. In SWEBOK V4, Security is a new standalone Knowledge Area (KA 13), elevated from a sub-topic of the Software Quality KA in V3. This elevation signals unambiguously that security is a first-class engineering concern — not an audit checkbox, not a pentest done before release, not someone else's job.

For a coding agent, the primary directive is: security analysis begins at requirements, not at deployment. Every design decision, every API endpoint, every data field, every dependency, and every configuration setting has a security surface. SWEBOK V4 integrates threat modeling, secure coding standards (OpenSSF Pyscg), and security testing into the normal engineering lifecycle. When in doubt, the agent must apply the Principle of Least Privilege, use established cryptographic libraries, parameterize all queries, externalize all secrets, and never use `assert` for security-critical checks.

## Key Concepts

### CIA Triad

The CIA Triad is the foundational framework for describing security properties. Every security requirement maps to one or more of these three:

| Property | Definition | Violated By | Example Control |
|----------|-----------|------------|-----------------|
| **Confidentiality** | Information is accessible only to authorized parties | Data breach, unauthorized read access, logging PII | Encryption at rest and in transit, RBAC, data minimization |
| **Integrity** | Information and systems are accurate and have not been tampered with | SQL injection modifying records, man-in-the-middle tampering, insecure deserialization | Parameterized queries, digital signatures, checksums, HMAC |
| **Availability** | Systems and data are accessible when needed by authorized parties | DDoS attack, resource exhaustion, ransomware | Rate limiting, auto-scaling, backups, redundancy |

All three must be considered for every system component. A system that is confidential and available but not integral (data can be silently corrupted) is still critically insecure.

---

### STRIDE Threat Model

STRIDE is a systematic framework for identifying threats, developed at Microsoft. Each letter names a category of attack. Apply STRIDE analysis to every trust boundary, entry point, and data flow in the system architecture.

| Threat | Violated Property | Definition | Example Attack |
|--------|-----------------|------------|----------------|
| **S**poofing | Authentication | Attacker pretends to be another user or system | Forged JWT tokens, ARP spoofing, email sender forgery |
| **T**ampering | Integrity | Attacker modifies data in transit or at rest | SQL injection, man-in-the-middle modification, log tampering |
| **R**epudiation | Non-repudiation | Actor denies having performed an action | User denies making a purchase; no audit log to prove otherwise |
| **I**nformation Disclosure | Confidentiality | Data exposed to unauthorized parties | Stack traces in error responses, logging credentials, directory listing |
| **D**enial of Service | Availability | System made unavailable to legitimate users | Memory exhaustion via large payloads, CPU exhaustion via regex |
| **E**levation of Privilege | Authorization | User gains permissions beyond their grant | Path traversal, IDOR, JWT alg=none attack |

### STRIDE Threat Modeling Workflow

1. **Draw a Data Flow Diagram (DFD):** Map all processes, data stores, external entities, and data flows
2. **Identify trust boundaries:** Lines where the trust level changes (user → API gateway, API → database, external service → internal service)
3. **Apply STRIDE to each element:** For each process, data store, and data flow, ask which STRIDE threats apply
4. **Rate each threat:** Use DREAD (Damage, Reproducibility, Exploitability, Affected users, Discoverability) or CVSS for severity
5. **Define mitigations:** For each threat, record the control that prevents or detects it
6. **Document as ADR:** Record the threat model and mitigations in an Architecture Decision Record

---

### Attack Surface Analysis

The attack surface is the sum of all points where an attacker can try to enter or extract data from a system.

**Entry points to identify:**
- All HTTP/HTTPS endpoints (REST, GraphQL, WebSocket)
- All message queue consumers
- All file upload handlers
- All CLI arguments and environment variables read at startup
- All database query parameters
- All deserialization of external data (JSON, XML, protobuf, pickle)
- All third-party webhooks and callbacks

**Trust boundaries to mark:**
- Internet → load balancer
- Load balancer → application server
- Application server → database
- Application server → external API
- Application server → message queue
- Internal service → another internal service (even within the same cluster)

**Principle:** Minimize the attack surface. Every endpoint that does not need to exist should not exist. Every field that is not required should not be accepted. Every permission that is not needed should not be granted.

---

### Secure by Design vs. Bolted-On Security

| Approach | Characteristics | Risk |
|----------|----------------|------|
| **Bolted-on security** | Security added after construction via pentest, WAF, or post-hoc code review | Architectural flaws cannot be patched; WAF is easily bypassed; high remediation cost |
| **Secure by design** | Threat modeling at requirements phase; security controls in architecture; secure coding standards enforced at commit | Defects caught early (10–200× cheaper to fix); security properties are verifiable |

SWEBOK V4 mandates the secure by design approach. Security requirements must be elicited during KA 01 (Requirements) and realized in KA 02 (Architecture) before implementation in KA 04 (Construction).

---

### Privacy by Design

Privacy by Design (PbD) is the framework for embedding data privacy into systems from the ground up. It aligns directly with GDPR Article 25 ("data protection by design and by default").

**Seven foundational principles:**
1. **Proactive, not reactive:** Anticipate privacy risks before they occur
2. **Privacy as the default:** The most private setting is the default; users opt in to sharing
3. **Privacy embedded into design:** Not added as an add-on
4. **Full functionality:** Privacy does not require sacrificing functionality
5. **End-to-end security:** Protect data throughout its lifecycle
6. **Visibility and transparency:** Processes are open to verification
7. **Respect for user privacy:** Keep systems user-centric

**Data minimization in practice:**
- Collect only data fields explicitly required for the stated purpose
- Define and enforce retention periods; delete data when no longer needed
- Pseudonymize or anonymize data for analytics workloads
- Do not log PII (names, emails, SSNs, IP addresses where regulated) — see pyscg-0019
- Separate PII from behavioral data at the schema level

---

### Zero Trust Architecture

Zero Trust is an architectural model that assumes the network perimeter has already been breached. There is no "safe" internal network.

**Core principles:**
1. **Never trust, always verify:** Every request, regardless of origin (internal or external), must be authenticated and authorized
2. **Least privilege:** Every identity (user, service, machine) receives only the minimum permissions needed for the specific task
3. **Assume breach:** Design systems to limit blast radius; segment networks so a compromised component cannot reach everything
4. **Microsegmentation:** Divide the network into small zones; enforce access controls at every segment boundary
5. **Verify explicitly:** Use multiple signals for authorization (identity, device health, location, time of day)
6. **Continuous validation:** Re-verify trust at each request, not just at session establishment

**Implementation for services:**
- Use mTLS (mutual TLS) for service-to-service communication within the cluster
- Issue short-lived tokens (JWTs with 15-minute expiry, not 24 hours)
- Implement network policies to restrict which services can communicate
- Use a service mesh (Istio, Linkerd) to enforce mTLS and policy automatically

---

### OWASP Top 10 Cross-Reference Table

The OWASP Top 10 is the consensus list of the most critical web application security risks. Every item maps to a SWEBOK control and Python-specific Pyscg rules.

| # | OWASP Risk | SWEBOK Control | Python Pyscg Rule | Mitigation Code Pattern |
|---|-----------|---------------|-------------------|------------------------|
| A01 | Broken Access Control | Least privilege; server-side enforcement; deny by default | pyscg-0041 | Verify permissions in middleware; never rely on client-supplied role claims |
| A02 | Cryptographic Failures | Use vetted crypto libraries; TLS everywhere; key rotation | Avoid MD5/SHA1 for passwords | Use `cryptography` library; bcrypt for passwords; TLS 1.2+ enforced |
| A03 | Injection (SQL, OS, LDAP) | Parameterized queries; input validation; output encoding | pyscg-0010 | `cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))` |
| A04 | Insecure Design | Threat modeling; secure reference architectures; ADRs | STRIDE analysis | Apply STRIDE at design phase; document mitigations in ADR |
| A05 | Security Misconfiguration | Hardened defaults; remove unused features; externalize config | pyscg-0041 | No debug mode in prod; no default credentials; env vars for secrets |
| A06 | Vulnerable Components | `pip-audit`; SBOM; pin dependency versions | Dependency scanning in CI | `pip-audit` in pre-commit; `pip install --require-hashes` |
| A07 | Auth Failures | Strong password policy; session timeouts; MFA; rate limiting | Rate limiting on auth endpoints | Lockout after N failures; rotate session tokens after login |
| A08 | Software and Data Integrity Failures | CI signature checks; avoid unsafe deserialization | pyscg-0023 | Never use `pickle` on untrusted input; verify signatures on downloaded artifacts |
| A09 | Security Logging and Monitoring Failures | Sanitize logs; structured logging; alerting on anomalies | pyscg-0019 | Log events (not data); redact PII before logging; alert on auth failures |
| A10 | Server-Side Request Forgery (SSRF) | Allowlist external URLs; block metadata endpoints | URL validation | Allowlist external hosts; block `169.254.169.254` (AWS metadata) |

---

### Python-Specific Security Rules (OpenSSF Pyscg)

The OpenSSF Secure Coding Guide for Python (Pyscg) provides authoritative, Python-specific rules. These are mandatory, not advisory.

#### pyscg-0010: Parameterized Queries

**Rule:** Never construct SQL queries by string concatenation. Always use parameterized queries.

```python
# VIOLATION — SQL Injection vulnerability
def get_user(username: str) -> User:
    query = f"SELECT * FROM users WHERE name = '{username}'"
    cursor.execute(query)  # attacker controls username

# CORRECT — parameterized query
def get_user(username: str) -> User:
    cursor.execute("SELECT * FROM users WHERE name = %s", (username,))
```

This applies to ALL query languages that accept external input: SQL, NoSQL (MongoDB), LDAP, XPath, OS commands.

#### pyscg-0019: Exclude Sensitive Data from Logs

**Rule:** Never log PII, credentials, tokens, or other sensitive data. Log event types and identifiers only.

```python
import logging
logger = logging.getLogger(__name__)

# VIOLATION — logs the password
logger.info(f"Login attempt: user={user.email}, password={password}")

# CORRECT — logs the event and non-sensitive identifier only
logger.info("Login attempt", extra={"user_id": user.id})
```

Sensitive data in logs: passwords, API keys, credit card numbers, SSNs, email addresses (context-dependent), IP addresses (GDPR-regulated in EU), session tokens, JWT contents.

#### pyscg-0023: Never Use pickle for Untrusted Data

**Rule:** `pickle` can execute arbitrary code during deserialization. Never deserialize pickled data from an untrusted source.

```python
# VIOLATION — arbitrary code execution if data is attacker-controlled
import pickle
data = pickle.loads(request.body)

# CORRECT — use JSON for untrusted external data
import json
data = json.loads(request.body)

# CORRECT — use protobuf for high-performance structured data
from myapp.proto import UserRequest
data = UserRequest.FromString(request.body)
```

Safe alternatives to pickle: `json`, `msgpack`, `protobuf`, `orjson`. If you must persist Python objects, use `json` with a custom encoder, not pickle.

#### pyscg-0034: Check for None Values Before Use

**Rule:** Explicitly check for `None` before accessing attributes or calling methods on values that may be `None`. Do not rely on `AttributeError` as a control flow mechanism.

```python
# VIOLATION — AttributeError if result is None
def process_user(user_id: int) -> str:
    user = db.find_user(user_id)
    return user.name.upper()  # crashes if user is None

# CORRECT — explicit None check
def process_user(user_id: int) -> str:
    user = db.find_user(user_id)
    if user is None:
        raise ValueError(f"User {user_id} not found")
    return user.name.upper()
```

Using type hints with `Optional[T]` and a strict mypy configuration (`--strict`) enforces this at the type-checker level.

#### pyscg-0035 / pyscg-0052: Complete Resource Cleanup

**Rule:** All resources (file handles, database connections, network sockets, locks) must be released even when exceptions occur. Use `with` statements (context managers) as the primary mechanism.

```python
# VIOLATION — connection leaked if exception occurs
def read_data(path: str) -> str:
    f = open(path)
    data = f.read()  # exception here leaves f open
    f.close()
    return data

# CORRECT — with statement guarantees cleanup
def read_data(path: str) -> str:
    with open(path) as f:
        return f.read()

# CORRECT — database connections
def get_record(conn: Connection, record_id: int) -> dict:
    with conn.cursor() as cursor:
        cursor.execute("SELECT * FROM records WHERE id = %s", (record_id,))
        return cursor.fetchone()
```

For async contexts: use `async with` for async context managers. For locks: always use `with lock:`, never `lock.acquire()` / `lock.release()` without a `try/finally`.

#### pyscg-0037: Never Use assert for Security Checks

**Rule:** Python's `assert` statement is a no-op when the interpreter runs with the `-O` (optimize) flag. Many production deployments use `-O`. Security-critical checks performed with `assert` silently vanish in production.

```python
# VIOLATION — assert is stripped in optimized mode
def delete_post(user_id: int, post_id: int) -> None:
    assert user_id == get_post_owner(post_id), "Not authorized"
    db.delete_post(post_id)

# CORRECT — use explicit conditional raise
def delete_post(user_id: int, post_id: int) -> None:
    if user_id != get_post_owner(post_id):
        raise PermissionError(f"User {user_id} cannot delete post {post_id}")
    db.delete_post(post_id)
```

`assert` is appropriate only for internal invariant checking in development (debugging aid), never for authorization, authentication, or input validation in production paths.

#### pyscg-0041: Externalize All Configuration and Secrets

**Rule:** No credentials, API keys, database URLs, or configuration values that differ between environments may appear in source code. Use environment variables or a secret manager.

```python
# VIOLATION — hardcoded credentials in source (exposed in git history)
DB_URL = "postgresql://admin:s3cr3t@prod-db:5432/mydb"
API_KEY = "sk-abc123..."

# CORRECT — environment variables
import os
DB_URL = os.environ["DATABASE_URL"]  # raises KeyError if not set — intentional
API_KEY = os.environ["API_KEY"]

# CORRECT — with validation at startup
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    api_key: str
    debug: bool = False

    class Config:
        env_file = ".env"  # for local development only

settings = Settings()  # fails fast if required vars are missing
```

**Local development:** Create a `.env.template` (never `.env`) committed to the repo showing required variables without values. Add `.env` to `.gitignore`.

---

### Security Testing Integration

Security testing is part of the CI/CD pipeline, not a separate security review:

| Stage | Test Type | Tool | Trigger |
|-------|----------|------|---------|
| Pre-commit | SAST (static analysis) | `bandit` | Every commit |
| Pre-commit | Dependency audit | `pip-audit` | Every commit |
| PR | SAST full scan | `bandit -r` with full report | Every PR |
| PR | Type checking (catches None deref) | `mypy --strict` | Every PR |
| Nightly | DAST (dynamic analysis) | OWASP ZAP | Nightly against staging |
| Nightly | Container image scan | Trivy or Snyk | Nightly |
| Release | Full pentest / DAST | Burp Suite | Pre-release |

**bandit configuration example:**

```ini
# .bandit
[bandit]
exclude_dirs = tests,venv
skips =          # only skip if you have a documented reason
```

Run: `bandit -r src/ -f json -o bandit-report.json`

---

### Incident Response Basics

When a security incident occurs, follow the five-phase NIST incident response lifecycle:

1. **Detect:** Monitoring alerts, anomaly detection, or user report identifies the incident. Log the time of detection.
2. **Contain:** Immediately limit the blast radius. Isolate affected systems. Revoke compromised credentials. Do not destroy evidence.
3. **Eradicate:** Remove the root cause — patch the vulnerability, remove malicious code, clean compromised accounts.
4. **Recover:** Restore systems from known-good backups. Validate integrity before bringing systems back online.
5. **Lessons Learned:** Conduct a blameless post-mortem within 48 hours. Document: what happened, timeline, impact, root cause, and preventive controls added.

**Agent responsibility during incident response:** If a coding agent discovers a security vulnerability (e.g., an exposed credential in git history, an unparameterized query), it must report it immediately before proceeding. ACM/IEEE Code of Ethics Principle 1 (Public Interest) requires this; an agent that conceals a vulnerability to maintain workflow velocity is acting unethically.

---

### New in V4: Security as Standalone KA

In SWEBOK V3, security was a sub-topic within Software Quality (KA 10). V4 promotes it to standalone KA 13, with the following implications for agents:

- Security must be elicited as a requirement (KA 01), not discovered as a defect
- Security architecture decisions must be documented in ADRs (KA 02)
- Security controls must be implemented via secure coding standards during construction (KA 04)
- Security tests (SAST, DAST, dependency audit) must be part of the test plan (KA 05)
- Security is not "someone else's job" — it is part of every KA

---

## Agent Guidance

### Do
- Begin threat modeling (STRIDE) at the architecture phase, before writing any code
- Classify every data element as PII/sensitive or non-sensitive; apply appropriate controls to sensitive data
- Use parameterized queries for all database interactions without exception (pyscg-0010)
- Externalize all secrets, credentials, and environment-specific config via environment variables (pyscg-0041)
- Use established cryptographic libraries (`cryptography`, `bcrypt`, `PyNaCl`) — never implement custom crypto
- Enforce TLS 1.2+ for all network communication; use mTLS for service-to-service calls
- Run `bandit -r src/` and `pip-audit` in pre-commit hooks and CI pipelines
- Use `with` statements for all resource management: files, connections, locks, cursors (pyscg-0035/0052)
- Check for `None` explicitly before attribute access; use `mypy --strict` to enforce this (pyscg-0034)
- Apply the Principle of Least Privilege to every identity: users, services, CI pipelines, database users
- Use `json` or `protobuf` instead of `pickle` for all serialization of untrusted or external data (pyscg-0023)
- Sanitize log output: never log passwords, tokens, API keys, or PII (pyscg-0019)
- Report any discovered security vulnerability immediately; do not proceed silently

### Do Not
- Use `assert` for authorization, authentication, or input validation — it is stripped in production (pyscg-0037)
- Construct SQL queries with string formatting or concatenation — always use parameterized queries (pyscg-0010)
- Store secrets, API keys, or credentials in source code, even in comments or test files (pyscg-0041)
- Use `pickle` to deserialize data from any external, user-controlled, or untrusted source (pyscg-0023)
- Log request bodies, user inputs, or session data without first redacting sensitive fields (pyscg-0019)
- Use MD5 or SHA-1 for password hashing — use bcrypt, Argon2, or scrypt
- Use `http://` for any service that handles sensitive data — enforce HTTPS/TLS
- Disable SSL certificate verification (`verify=False`) in any HTTP client call
- Rely on client-supplied role or permission claims — always verify authorization server-side
- Grant database users more permissions than required for the application's specific operations
- Skip dependency auditing — a vulnerable transitive dependency is your vulnerability

## Checklist
- [ ] STRIDE threat model completed for all trust boundaries and entry points
- [ ] CIA Triad requirements identified for every system component
- [ ] All SQL queries use parameterized form (pyscg-0010); grep for string formatting in queries
- [ ] All secrets and credentials externalized to environment variables or secret manager (pyscg-0041)
- [ ] No `assert` used for security-critical checks in production code paths (pyscg-0037)
- [ ] No `pickle` used to deserialize external or user-supplied data (pyscg-0023)
- [ ] Logging reviewed: no PII, passwords, tokens, or API keys in log output (pyscg-0019)
- [ ] All None values explicitly checked before attribute access (pyscg-0034)
- [ ] All resources managed with `with` statements (pyscg-0035/0052)
- [ ] `bandit -r src/` passes with no high-severity findings
- [ ] `pip-audit` passes with no known vulnerabilities in dependencies
- [ ] `mypy --strict` passes (catches None dereference and type contract violations)
- [ ] TLS enforced for all external-facing endpoints
- [ ] Least privilege applied to all database users, service accounts, and API permissions
- [ ] OWASP Top 10 checklist reviewed for the specific feature being built
- [ ] Sensitive data fields identified and classified; retention period defined
- [ ] Incident response runbook exists and is referenced in service documentation

## See Also
- wiki/tier1-sources/swebok-v4/overview.md
- wiki/tier1-sources/swebok-v4/ka02-architecture.md
- wiki/tier1-sources/swebok-v4/ka04-construction.md
- wiki/tier1-sources/swebok-v4/ka05-testing.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md
- wiki/tier1-sources/owasp/top10.md
- wiki/tier2-core/security-practices/threat-modeling.md
- wiki/tier2-core/security-practices/zero-trust.md
- wiki/tier3-working/python/secure-coding.md
- wiki/tier3-working/checklists/security-review.md

## Source

IEEE Computer Society. *Guide to the Software Engineering Body of Knowledge (SWEBOK), Version 4.0*, KA 13: Software Security. IEEE Press, 2024.

OWASP Foundation. *OWASP Top 10:2025*. https://owasp.org/Top10/2025/

OpenSSF. *Secure Coding Guide for Python (Pyscg)*. https://best.openssf.org/Secure-Coding-Guide-for-Python/

NIST. *Computer Security Incident Handling Guide (SP 800-61 Rev 2)*. https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final
