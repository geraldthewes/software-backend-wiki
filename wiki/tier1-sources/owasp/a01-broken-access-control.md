# A01: Broken Access Control

> **Tier 1** | Source: OWASP Top 10:2021 | Authority: immutable

## Summary

Broken Access Control is the most common web application security risk as of OWASP Top 10:2021, moving up from position 5 in 2017. It occurs when users can act outside their intended permissions. For a coding agent, this means every protected resource must have an explicit server-side authorization check — the absence of a check is itself the vulnerability.

## Key Concepts

**Access Control** enforces policy that prevents users from acting outside their intended permissions. Failures lead to unauthorized information disclosure, modification, or destruction of data, and execution of business functions outside the user's authority.

**Most Common Failure Modes:**

- **IDOR (Insecure Direct Object Reference)**: User supplies an object ID (e.g., `?account_id=1234`) and the server returns data without verifying the requester owns that record
- **URL manipulation / force browsing**: Directly accessing privileged URLs (e.g., `/admin/panel`) without authentication checks
- **Privilege escalation (vertical)**: A standard user accesses admin-only functionality
- **Privilege escalation (horizontal)**: A user accesses another user's data at the same privilege level
- **CORS misconfiguration**: API accepts requests from unauthorized or wildcard origins
- **Metadata manipulation**: Tampering with JWT claims, cookies, or hidden form fields to elevate privileges

**Root Causes:**

- Access control logic implemented only on the client side (e.g., hiding a button in the UI)
- Missing server-side checks on individual object access (IDOR)
- Over-broad permissions: service accounts or users granted more access than required
- Access checks performed after the database query (data already retrieved before authorization)
- Assume trust based on headers or session data that can be manipulated

## Python Mitigations

**Principle: Enforce permissions server-side, before any data is retrieved.**

Use decorator-based access control to make authorization explicit and auditable:

```python
from functools import wraps
from flask import abort, g

def require_permission(permission: str):
    def decorator(f):
        @wraps(f)
        def decorated(*args, **kwargs):
            if not g.current_user.has_permission(permission):
                abort(403)
            return f(*args, **kwargs)
        return decorated
    return decorator

@app.route("/admin/users")
@require_permission("admin:read")
def list_users():
    ...
```

**Never trust client-supplied user IDs for ownership checks:**

```python
# WRONG — trusts client-supplied user_id
def get_order(order_id: int, user_id: int) -> Order:
    return db.query(Order).filter_by(id=order_id, user_id=user_id).first()

# RIGHT — uses authenticated session identity, not client input
def get_order(order_id: int) -> Order:
    order = db.query(Order).filter_by(id=order_id).first()
    if order is None or order.user_id != current_user.id:
        raise PermissionError("Access denied")
    return order
```

**Check permissions before the database query, not after:**

```python
# WRONG — data retrieved before authorization verified
def delete_document(doc_id: int) -> None:
    doc = db.get(Document, doc_id)
    if doc.owner_id != current_user.id:
        raise PermissionError()  # too late, doc already fetched
    db.delete(doc)

# RIGHT — authorize first, then fetch
def delete_document(doc_id: int) -> None:
    if not current_user.can_delete_document(doc_id):
        raise PermissionError("Access denied")
    doc = db.get(Document, doc_id)
    db.delete(doc)
```

**Implement least privilege for service accounts:**
- Service accounts should only have SELECT on tables they read from
- No service account should have DROP, TRUNCATE, or broad schema permissions
- API keys and tokens should be scoped to minimum required operations

**CORS configuration — explicit allowlist, never wildcard for authenticated APIs:**

```python
# WRONG
CORS(app, origins="*")

# RIGHT
CORS(app, origins=["https://app.example.com", "https://admin.example.com"])
```

## Pyscg Alignment

- **pyscg-0041**: Externalize configuration, including role definitions and permission mappings — do not hardcode role checks in business logic

## Checklist
- [ ] Server-side authorization check on every protected route — not just authentication, but authorization
- [ ] IDOR protection: ownership verified using session identity, not client-supplied IDs
- [ ] No permission decisions based solely on client-supplied data (headers, form fields, JWT claims without server-side validation)
- [ ] Access check happens before any database query returns data
- [ ] Least privilege: service accounts and API keys scoped to minimum required permissions
- [ ] CORS allowlist is explicit — no wildcard origins for authenticated endpoints
- [ ] Admin and privileged routes are protected by role/permission decorators, not just by obscurity
- [ ] Access control failures are logged (see A09)

## See Also
- wiki/tier1-sources/owasp/top10-2021-overview.md
- wiki/tier1-sources/owasp/a09-logging-monitoring.md

## Source
OWASP Top 10:2021 — A01 Broken Access Control. https://owasp.org/Top10/A01_2021-Broken_Access_Control/
