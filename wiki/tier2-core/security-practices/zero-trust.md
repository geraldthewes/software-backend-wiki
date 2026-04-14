# Zero Trust Architecture

> **Tier 2** | Source: NIST SP 800-207 | Derives From: ka13-security, owasp/a07-auth-failures | Authority: established practice

## Summary

Zero Trust is a security model based on the principle "never trust, always verify." No entity — user, device, or service — is trusted by default, regardless of whether it is inside or outside the network perimeter. Zero Trust is the architectural response to Fallacy #4 of Distributed Computing ("The Network Is Secure").

## Key Concepts

### Definition

NIST Special Publication 800-207 defines Zero Trust Architecture as:

> A collection of concepts and ideas designed to minimize uncertainty in enforcing accurate, least privilege per-request access decisions in information systems and services in the face of a network viewed as compromised.

In practical terms: treat every request as if it comes from an untrusted network, even if it originates from inside the data center or cluster.

### Core Principles

| Principle | Description | Implementation |
|-----------|-------------|---------------|
| **Never Trust, Always Verify** | Authenticate and authorize every request, regardless of origin | JWT validation on every API call; mTLS for service-to-service |
| **Least Privilege** | Grant the minimum access necessary; time-limit access where possible | Scoped tokens; RBAC; per-service database credentials |
| **Assume Breach** | Design as if the perimeter is already compromised; minimize blast radius | Microsegmentation; encrypt internal traffic; limit lateral movement |
| **Microsegmentation** | Divide the network into small zones; limit what can reach what | Network policies deny by default; only required paths allowed |
| **Continuous Verification** | Validate identity and permissions throughout the session, not just at login | Short-lived tokens; re-authenticate for sensitive operations |

---

### Implementation for Services

#### Service-to-Service Authentication

Internal services must authenticate to each other. Assuming that "if it's inside the cluster, it's trusted" is the assumption that Zero Trust rejects.

**Mutual TLS (mTLS)**:
- Both the calling service and the called service present certificates
- Each verifies the other's certificate against a trusted CA
- Provides authentication (who is calling?), integrity (was the request tampered?), and confidentiality (is the channel encrypted?)
- Implemented at the infrastructure layer via a service mesh (Istio, Linkerd) or sidecar proxies

**Short-lived certificates**:
- Certificate lifetime: hours, not years
- Automated rotation via a PKI (HashiCorp Vault PKI, SPIFFE/SPIRE)
- Revocation is trivial: certificates expire naturally; no CRL management needed

#### API Authentication

```python
import jwt
from fastapi import HTTPException, Security
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

security = HTTPBearer()

def verify_token(credentials: HTTPAuthorizationCredentials = Security(security)) -> dict:
    try:
        payload = jwt.decode(
            credentials.credentials,
            key=PUBLIC_KEY,
            algorithms=["RS256"],
            options={"require": ["exp", "iat", "sub", "iss"]},
        )
        return payload
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="Token expired")
    except jwt.InvalidTokenError as exc:
        raise HTTPException(status_code=401, detail=f"Invalid token: {exc}")
```

Key points:
- Use asymmetric algorithms (RS256, ES256) not HS256 — the signing key is not shared with verifiers
- Short expiry: 15 minutes for access tokens; use refresh tokens for longer sessions
- Include required claims: `exp` (expiry), `iat` (issued at), `sub` (subject), `iss` (issuer)
- Validate the `iss` claim — accept tokens only from your own authorization server

#### API Keys

- Transmit API keys in the `Authorization` header (`Authorization: Bearer sk-...`), never in the URL
- URLs are logged by web servers, proxies, and browser history — API keys in URLs are exposed in logs
- Hash stored API keys (bcrypt or SHA-256); store the hash, not the plaintext

```python
# Bad: API key in URL
GET /api/orders?api_key=sk-live-abc123

# Good: API key in header
GET /api/orders
Authorization: Bearer sk-live-abc123
```

#### Network Policies

In Kubernetes/Nomad environments:
- Default policy: **deny all ingress and egress**
- Explicitly allow only required service-to-service communication paths
- Log all denied connections for anomaly detection

```yaml
# Kubernetes NetworkPolicy — deny by default, allow only from specific service
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: orders-service
spec:
  podSelector:
    matchLabels:
      app: orders-service
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: api-gateway   # only api-gateway can call orders-service
```

#### Secrets Management

- **Never** use long-lived credentials (static passwords, permanent API keys)
- Use dynamic secrets: HashiCorp Vault generates database credentials on demand with a TTL
- Database credentials are unique per service and expire automatically
- If a credential is compromised, it expires within minutes; blast radius is limited to one service

```python
import hvac

def get_database_credentials() -> tuple[str, str]:
    client = hvac.Client(url=os.environ["VAULT_ADDR"])
    client.token = os.environ["VAULT_TOKEN"]
    secret = client.secrets.database.generate_credentials(
        name="orders-service"
    )
    return secret["data"]["username"], secret["data"]["password"]
```

---

### Relationship to the 8 Fallacies

Zero Trust is the direct response to **Fallacy #4: "The Network Is Secure."** The fallacy says the internal network can be trusted; Zero Trust says it cannot. Every implementation choice in Zero Trust — mTLS, short-lived tokens, network policies, dynamic secrets — exists to handle the reality that the network is not secure and attackers may already be inside.

See `wiki/tier2-core/distributed-systems/fallacies.md` for the complete fallacy list.

---

### Zero Trust Maturity Model (Summary)

| Level | Identity | Network | Data |
|-------|----------|---------|------|
| Traditional | Single sign-on | Firewall perimeter | At-rest encryption optional |
| Initial ZT | MFA + JWT | Some microsegmentation | Encryption required |
| Advanced ZT | mTLS everywhere | Deny-by-default policies | Dynamic secrets; field-level encryption |
| Optimal ZT | Continuous verification; short-lived certs | Automated policy enforcement | Zero standing privileges |

## Agent Guidance

### Do
- Authenticate every service-to-service request — even within the same cluster
- Use mTLS for internal service communication (or a service mesh that enforces it)
- Issue short-lived JWT access tokens (15 minutes maximum)
- Transmit API keys in headers, never in URLs
- Use dynamic secrets from a secrets manager instead of static passwords

### Do Not
- Do not assume internal network requests are trusted
- Do not issue long-lived tokens or permanent API keys for service accounts
- Do not store API keys or passwords in environment variables in plaintext when a secrets manager is available
- Do not skip authentication for "internal" endpoints

## Checklist
- [ ] All API endpoints validate authentication tokens before processing requests
- [ ] Service-to-service communication uses mTLS or equivalent authentication
- [ ] JWT tokens have expiry of 15 minutes or less for access tokens
- [ ] API keys are transmitted in `Authorization` headers, not URL parameters
- [ ] Network policies deny traffic by default; only required paths are explicitly allowed
- [ ] Database and service credentials are short-lived and rotated automatically

## See Also
- `wiki/tier2-core/security-practices/overview.md`
- `wiki/tier2-core/security-practices/threat-modeling.md`
- `wiki/tier2-core/distributed-systems/fallacies.md`
- `wiki/tier1-sources/swebok-v4/ka13-security.md`
- `wiki/tier1-sources/owasp/a02-cryptographic-failures.md`

## Source

NIST SP 800-207, *Zero Trust Architecture* (2020). CISA Zero Trust Maturity Model. Synthesized from *Software Development Best Practices for Agent* reference document.
