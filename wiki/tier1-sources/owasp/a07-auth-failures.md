# A07: Identification and Authentication Failures

> **Tier 1** | Source: OWASP Top 10:2021 | Authority: immutable

## Summary

Authentication failures occur when weaknesses in authentication and session management allow attackers to compromise passwords, keys, or session tokens. Successful exploitation grants attackers the identity and privileges of the compromised user, potentially including administrator access. For a coding agent, the cardinal rule is: **never roll your own authentication — delegate to proven, battle-tested libraries**.

## Key Concepts

**Common Failure Scenarios:**

- **Credential stuffing**: Application allows automated login attempts using lists of breached credentials — no rate limiting or lockout
- **Weak passwords permitted**: No minimum length, complexity, or breach-list check
- **Session tokens in URLs**: Tokens logged in server logs, browser history, and referer headers
- **Sessions not invalidated on logout**: Old tokens remain valid after the user logs out — useful for attackers who obtained a previous token
- **No MFA on sensitive operations**: High-value actions (password change, payment) protected only by password
- **Weak session token generation**: Using predictable or insufficiently random tokens
- **Plaintext or weakly hashed passwords**: See A02 for password hashing requirements

## Python Mitigations

**Never roll your own auth — use proven libraries:**

```python
# Django — use built-in authentication
from django.contrib.auth import authenticate, login, logout

# Flask — use Flask-Login + passlib/argon2
from flask_login import LoginManager, login_user, logout_user

# General password hashing — passlib
from passlib.context import CryptContext
pwd_context = CryptContext(schemes=["argon2", "bcrypt"], deprecated="auto")

# FastAPI — use python-jose for JWT + passlib for hashing
from jose import JWTError, jwt
```

**Cryptographically secure session tokens:**

```python
import secrets

# Generate a session token with sufficient entropy (256-bit minimum)
session_token = secrets.token_urlsafe(32)    # 32 bytes = 256 bits = 43-char URL-safe string
api_key = secrets.token_hex(32)              # hex representation

# Set appropriate expiry: short-lived access tokens, longer refresh tokens
ACCESS_TOKEN_EXPIRY = 15 * 60          # 15 minutes in seconds
REFRESH_TOKEN_EXPIRY = 7 * 24 * 3600  # 7 days
```

**Rate limit authentication endpoints:**

```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@app.post("/auth/login")
@limiter.limit("5/minute;20/hour")
def login():
    ...

@app.post("/auth/password-reset")
@limiter.limit("3/hour")
def request_password_reset():
    ...
```

**Account lockout after repeated failures:**

```python
import redis
from datetime import timedelta

MAX_FAILURES = 5
LOCKOUT_DURATION = timedelta(minutes=15)

def check_account_lockout(username: str, redis_client: redis.Redis) -> bool:
    key = f"auth:failures:{username}"
    failures = int(redis_client.get(key) or 0)
    return failures >= MAX_FAILURES

def record_auth_failure(username: str, redis_client: redis.Redis) -> None:
    key = f"auth:failures:{username}"
    pipe = redis_client.pipeline()
    pipe.incr(key)
    pipe.expire(key, int(LOCKOUT_DURATION.total_seconds()))
    pipe.execute()
```

**Server-side session invalidation on logout — not just cookie deletion:**

```python
# WRONG — only removes cookie from client; token remains valid on server
@app.post("/auth/logout")
def logout():
    response = make_response({"status": "logged out"})
    response.delete_cookie("session_token")
    return response

# RIGHT — invalidate token server-side; cookie deletion is secondary
@app.post("/auth/logout")
def logout():
    token = request.cookies.get("session_token")
    if token:
        # Add to blocklist or delete from session store
        session_store.delete(token)
    response = make_response({"status": "logged out"})
    response.delete_cookie("session_token")
    return response
```

**Never store passwords in plaintext — always use bcrypt or argon2:**

See wiki/tier1-sources/owasp/a02-cryptographic-failures.md for password hashing requirements.

**MFA for sensitive roles and operations:**

Design MFA into the architecture for:
- Administrative user accounts
- Password and email change operations
- High-value financial transactions
- API access from new devices or locations

## Checklist
- [ ] Rate limiting implemented on all authentication endpoints (login, password reset, registration)
- [ ] Account lockout after configurable number of failed attempts
- [ ] Passwords hashed with bcrypt (min rounds=12) or argon2 — never plaintext or MD5/SHA1
- [ ] Session tokens generated with `secrets.token_urlsafe(32)` minimum
- [ ] Session tokens never transmitted in URLs — cookie or Authorization header only
- [ ] Server-side session invalidation on logout (not just cookie deletion)
- [ ] Session expiry configured: short-lived access tokens; absolute timeout on long-lived tokens
- [ ] MFA available and enforced for admin roles and sensitive operations
- [ ] Auth library used (passlib, authlib, django.contrib.auth) — no custom auth logic

## See Also
- wiki/tier1-sources/owasp/top10-2021-overview.md
- wiki/tier1-sources/owasp/a02-cryptographic-failures.md
- wiki/tier1-sources/owasp/a09-logging-monitoring.md

## Source
OWASP Top 10:2021 — A07 Identification and Authentication Failures. https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/
