# Threat Modeling — STRIDE

> **Tier 2** | Source: Microsoft STRIDE / OWASP | Derives From: ka13-security, owasp/a04-insecure-design | Authority: established practice

## Summary

Threat modeling is a structured technique for identifying security threats *before* implementation. STRIDE is the most widely used framework: it provides six categories of threats, each corresponding to a security property violation. Performing STRIDE at design time is far cheaper than finding the same vulnerabilities in production or in a penetration test.

## Key Concepts

### What Threat Modeling Is

Threat modeling answers four questions:
1. What are we building?
2. What can go wrong?
3. What are we going to do about it?
4. Did we do a good enough job?

STRIDE provides a systematic way to answer question 2 for every component in the system.

### When to Perform Threat Modeling

- At **design time**, before writing security-relevant code
- When the architecture changes significantly
- When adding new trust boundaries (a new external API consumer, a new admin interface, a new data store)
- Before writing: authentication, authorization, payment processing, PII handling, admin operations, inter-service communication

### The STRIDE Model

| Letter | Threat | Security Property Violated | Mitigation |
|--------|--------|--------------------------|------------|
| **S** | Spoofing | Authentication | Authenticate every caller; use strong identity |
| **T** | Tampering | Integrity | Integrity checks; signed tokens; input validation |
| **R** | Repudiation | Non-repudiation | Audit logging; signed audit trails |
| **I** | Information Disclosure | Confidentiality | Encryption at rest and in transit; access control; minimal data exposure |
| **D** | Denial of Service | Availability | Rate limiting; timeouts; circuit breakers; resource quotas |
| **E** | Elevation of Privilege | Authorization | Least privilege; explicit permission validation; no implicit admin access |

---

### STRIDE in Detail

#### S — Spoofing

**Threat**: An attacker impersonates a legitimate user or service.

**Examples**:
- User submits requests with a forged `user_id` in the request body
- A malicious service pretends to be the payments service
- Session token is stolen and replayed

**Mitigations**:
- Authenticate every request (JWT validation, API key verification)
- Use mutual TLS (mTLS) for service-to-service authentication
- Bind session tokens to client attributes; rotate tokens on privilege escalation
- Use short-lived tokens (15 minutes to 1 hour for access tokens)

#### T — Tampering

**Threat**: An attacker modifies data in transit or at rest.

**Examples**:
- Man-in-the-middle modifies an order's `amount` field in transit
- User modifies a JWT payload (if not signed or signature not verified)
- Database records are modified directly by a compromised internal process

**Mitigations**:
- TLS for all data in transit
- Verify JWT signatures — never trust the payload without verifying `alg` and `sig`
- Use HMAC or digital signatures for data integrity where headers are insufficient
- Validate all input: type, range, format, business rules

#### R — Repudiation

**Threat**: An attacker (or user) denies having performed an action. "I never deleted that record."

**Examples**:
- Admin deletes records; no audit trail shows who did it
- User disputes a charge; no log ties the charge to the user's session

**Mitigations**:
- Write an **audit log** for every sensitive mutation: who, what, when, from where
- Audit log entries must be immutable — append-only, not editable by the actor
- For high-value operations: digitally sign audit entries
- Include request ID, user ID, timestamp, action, and affected resource in every audit entry

#### I — Information Disclosure

**Threat**: Sensitive data is exposed to unauthorized parties.

**Examples**:
- Error messages include stack traces with database schema details
- API returns more fields than the caller is authorized to see (over-fetching)
- Logs contain passwords or PII
- Backup files are stored in a publicly accessible S3 bucket

**Mitigations**:
- Return generic error messages to clients; log detailed errors server-side
- Implement field-level access control — return only the fields the caller is authorized to see
- Apply pyscg-0019 (no sensitive data in logs)
- Encrypt sensitive data at rest; use column-level encryption for PII
- Apply principle of minimal data exposure: collect and store only what is necessary

#### D — Denial of Service

**Threat**: An attacker consumes resources to prevent legitimate use.

**Examples**:
- Sending millions of requests per second (volumetric DDoS)
- Sending a single large payload that consumes all CPU (algorithmic DoS — ReDoS)
- Exhausting database connection pool with slow queries

**Mitigations**:
- Rate limiting at the API gateway and application level
- Request size limits; payload validation before expensive processing
- Explicit timeouts on all operations (see `resilience-patterns.md`)
- Circuit breakers to prevent cascade failure
- Avoid regex patterns with catastrophic backtracking (ReDoS)

#### E — Elevation of Privilege

**Threat**: A user or service gains more permissions than it should have.

**Examples**:
- Regular user accesses admin endpoint because no authorization check is performed
- Service account has database admin privileges when it only needs read access
- Path traversal: `GET /files/../../etc/passwd`

**Mitigations**:
- Apply **principle of least privilege**: every process, user, and service gets the minimum permissions needed
- Perform authorization checks server-side on every request — never trust client-provided role claims without server verification
- Use role-based access control (RBAC) or attribute-based access control (ABAC)
- Validate file paths; use `os.path.realpath` and assert the result is within the allowed directory

---

### The Threat Modeling Process

1. **Draw a Data Flow Diagram (DFD)**
   - Identify: external entities (users, external services), processes (your service code), data stores (databases, caches), data flows (arrows between components), trust boundaries (lines between trusted and untrusted zones)

2. **Apply STRIDE to each element**
   - For each process, data store, data flow, and external entity: consider which STRIDE threats apply

3. **Document each threat**
   | Field | Description |
   |-------|-------------|
   | ID | T-001, T-002, ... |
   | Type | S / T / R / I / D / E |
   | Component | Which element of the DFD |
   | Description | Specific attack scenario |
   | Likelihood | Low / Medium / High |
   | Impact | Low / Medium / High |
   | Mitigation | Specific control |
   | Status | Open / Mitigated / Accepted |

4. **Select mitigations** for each identified threat

5. **Verify mitigations in code review** — threat model findings become code review checklist items

---

### Agent Guidance for Threat Modeling

Before writing authentication, payment, PII-handling, or admin code:

1. Draw a DFD for the feature (even a rough sketch)
2. Identify all trust boundaries (where does data cross from untrusted to trusted?)
3. Apply all 6 STRIDE categories to each element
4. For each identified threat, select a mitigation from the examples above
5. Add the mitigations to the implementation plan before writing code

## Agent Guidance

### Do
- Perform STRIDE before writing any security-relevant feature
- Draw the data flow diagram first — threats become obvious when data flows are visible
- Document identified threats in a threat register, even informally
- Treat "Elevation of Privilege" as the highest priority — it is the most exploited category in web applications

### Do Not
- Do not skip threat modeling for "simple" features — simple features often have the most overlooked threat vectors
- Do not rely on a single mitigation for a high-impact threat — apply defense in depth
- Do not treat threat modeling as a one-time activity — update it when architecture changes

## Checklist
- [ ] Data flow diagram drawn for the feature or component
- [ ] All 6 STRIDE categories applied to each DFD element
- [ ] Threats are documented with likelihood, impact, and mitigation
- [ ] Authentication applied to every external-facing endpoint (S)
- [ ] All inputs are validated (T)
- [ ] Audit logging exists for all sensitive mutations (R)
- [ ] Error responses do not leak internal details (I)
- [ ] Rate limiting is applied to public endpoints (D)
- [ ] Authorization is checked server-side on every protected operation (E)

## See Also
- `wiki/tier2-core/security-practices/overview.md`
- `wiki/tier2-core/security-practices/zero-trust.md`
- `wiki/tier2-core/security-practices/python-pyscg.md`
- `wiki/tier1-sources/swebok-v4/ka13-security.md`
- `wiki/tier1-sources/owasp/a04-insecure-design.md`

## Source

Microsoft STRIDE threat model; OWASP Threat Modeling Cheat Sheet; Adam Shostack, *Threat Modeling: Designing for Security* (2014). Synthesized from *Software Development Best Practices for Agent* reference document.
