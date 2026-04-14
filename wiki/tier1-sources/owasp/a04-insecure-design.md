# A04: Insecure Design

> **Tier 1** | Source: OWASP Top 10:2021 | Authority: immutable

## Summary

Insecure Design represents missing or ineffective security controls at the architectural and design level. It is distinct from all other OWASP risks: implementation bugs can be patched, but design flaws require architectural rework. A perfectly implemented insecure design remains insecure. For a coding agent, this means security thinking must happen before writing a single line of implementation code.

## Key Concepts

**Insecure Design vs. Security Misconfiguration (A05):**

- A04 is a design-time failure — the control was never conceived or is structurally insufficient
- A05 is a configuration-time failure — the control exists but is incorrectly configured
- A04 cannot be fixed by tuning configuration or patching code; it requires architectural changes

**Common Design Failure Scenarios:**

- **No rate limiting on critical flows**: Password reset, credential stuffing, and enumeration attacks succeed because volume is not constrained
- **Missing anti-automation**: Business logic flows (ticket purchase, coupon redemption) have no protection against bots
- **Absent data segregation**: Multi-tenant system shares data store without logical or physical isolation; a bug in one tenant's query can expose another's data
- **Unbounded resource consumption**: No limits on file upload size, API response depth, or recursive data structures; leads to DoS
- **Trust boundary violations**: Internal services assume all traffic is trusted; a single compromised internal component escalates to full system access

## Mitigations

**Threat Modeling before implementation — use STRIDE:**

Apply STRIDE to every significant new feature, especially those involving auth, payment, PII, or inter-service communication:

| Threat Category     | Question to Ask                                        |
|---------------------|--------------------------------------------------------|
| **S**poofing        | Can an attacker impersonate a user or service?         |
| **T**ampering       | Can data be modified in transit or at rest?            |
| **R**epudiation     | Can actions be denied? Are audit trails present?       |
| **I**nformation Disclosure | What data can be exposed? To whom?            |
| **D**enial of Service | Can a resource be exhausted or made unavailable?    |
| **E**levation of Privilege | Can a user gain unauthorized higher access?   |

**Secure Design Patterns:**

- **Defense in depth**: Multiple independent security controls so that failure of one does not compromise the system
- **Fail-safe defaults**: Deny access by default; only grant what is explicitly permitted
- **Complete mediation**: Every access to every resource must be checked — no caching of authorization decisions that may become stale
- **Least privilege**: Each component operates with the minimum permissions required for its function

**Security requirements belong in user stories, not as an afterthought:**

Add acceptance criteria to every story involving sensitive data or privileged actions:
- "Given an unauthenticated user, when they request /api/v1/admin, they receive 401"
- "Given 10 failed login attempts in 5 minutes from one IP, the account is temporarily locked"

**Rate limit all public APIs:**

Every public-facing endpoint must have rate limiting. Design this into the API gateway or middleware before implementation, not as a retrofit:

```python
# Design decision: rate limiting is middleware, not per-endpoint logic
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@app.post("/auth/login")
@limiter.limit("5/minute")
def login():
    ...
```

**Test for business logic flaws, not just technical flaws:**

- Can a user apply the same discount coupon twice?
- Can a user modify the price of an item in their cart?
- Can a free-tier user access paid-tier features by manipulating request parameters?

## Agent Guidance

### Do
- Perform STRIDE threat modeling before writing auth, payment, or PII-handling code
- Design rate limiting and anti-automation into the API at the architecture phase
- Assume breach: design each component as if adjacent components may be compromised
- Include security acceptance criteria in task definitions
- Involve security review when designing multi-tenant data isolation

### Do Not
- Begin implementation of sensitive features without a documented threat model
- Assume internal network traffic is trusted without verification
- Treat rate limiting as optional or a "phase 2" concern for public APIs
- Design authorization logic that can be bypassed by manipulating request data

## Checklist
- [ ] STRIDE threat model documented before implementation of auth, payment, or PII features
- [ ] Rate limiting designed into all public API endpoints
- [ ] Multi-tenant data isolation strategy defined and tested
- [ ] Fail-safe defaults in place — deny by default, grant explicitly
- [ ] Business logic tests cover abuse scenarios, not just happy paths
- [ ] Resource limits defined: file upload sizes, query result depths, pagination enforced

## See Also
- wiki/tier1-sources/owasp/top10-2021-overview.md
- wiki/tier1-sources/owasp/a01-broken-access-control.md
- wiki/tier1-sources/owasp/a05-security-misconfiguration.md

## Source
OWASP Top 10:2021 — A04 Insecure Design. https://owasp.org/Top10/A04_2021-Insecure_Design/
