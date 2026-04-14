# A05: Security Misconfiguration

> **Tier 1** | Source: OWASP Top 10:2021 | Authority: immutable

## Summary

Security Misconfiguration occurs when security controls exist but are incorrectly or incompletely configured. It is distinct from A04 (Insecure Design), where controls are absent at the design level. Misconfiguration is the most pervasive OWASP category — it spans web servers, databases, application frameworks, cloud services, and containers. For a coding agent, the guiding principle is: **disable everything not explicitly needed, and never ship development configuration to production**.

## Key Concepts

**Common Failure Scenarios:**

- **Default credentials**: Admin panels, databases, and cloud services deployed with vendor default usernames and passwords
- **Unnecessary features enabled**: Debug endpoints, admin consoles, sample applications, or directory listing left active in production
- **Missing security headers**: HTTP responses lack Content-Security-Policy, X-Frame-Options, or X-Content-Type-Options
- **Overly permissive CORS**: `Access-Control-Allow-Origin: *` on authenticated APIs
- **Verbose error messages in production**: Stack traces, database schema, and internal paths exposed in error responses
- **Unpatched systems**: Framework or OS components with known CVEs not updated
- **Cloud storage misconfiguration**: S3 buckets or blob storage set to public when they should be private

**Root Cause:** The principle of secure-by-default is violated — systems are deployed in their permissive "developer-friendly" state rather than hardened for production.

## Python Mitigations

**Never use DEBUG=True in production:**

```python
# settings/production.py
DEBUG = False
TESTING = False

# Django — also ensure these are off
SHOW_TRACEBACK = False

# Flask
app.config["DEBUG"] = False
app.config["TESTING"] = False
app.config["PROPAGATE_EXCEPTIONS"] = False
```

**Set secure HTTP headers:**

```python
# Flask with flask-talisman
from flask_talisman import Talisman

Talisman(app, 
    content_security_policy={
        "default-src": "'self'",
        "script-src": "'self'",
    },
    x_frame_options="DENY",
    x_content_type_options=True,
    strict_transport_security=True,
    strict_transport_security_max_age=31536000,
)

# FastAPI with starlette middleware
from starlette.middleware.httpsredirect import HTTPSRedirectMiddleware
app.add_middleware(HTTPSRedirectMiddleware)
```

**Custom error handlers — never expose internals:**

```python
# Flask
@app.errorhandler(500)
def internal_error(error):
    app.logger.error("Internal error", exc_info=True)   # log internally
    return {"error": "An internal error occurred"}, 500  # generic message to client

# FastAPI
@app.exception_handler(Exception)
async def generic_exception_handler(request, exc):
    logger.error("Unhandled exception", exc_info=exc)
    return JSONResponse(status_code=500, content={"error": "Internal server error"})
```

**Environment-specific configuration — never commit .env files:**

Follow pyscg-0041 — externalize all configuration:

```
# .env.template — COMMIT THIS: documents required variables, no values
DATABASE_URL=
SECRET_KEY=
DEBUG=
ALLOWED_HOSTS=

# .env — NEVER COMMIT: contains actual values
DATABASE_URL=postgresql://user:pass@localhost/mydb
SECRET_KEY=actual-secret-key
DEBUG=False
```

Always add `.env` to `.gitignore`. Use `.env.template` to document required variables for operators.

**Minimal enabled features checklist for frameworks:**

- Disable auto-index / directory listing on web servers
- Remove or disable admin interfaces not needed in production
- Set `SERVER_TOKENS off` (nginx) or `ServerTokens Prod` (Apache) to avoid version disclosure
- Disable unused authentication backends (e.g., remove `BasicAuth` if only JWT is used)
- Disable swagger/OpenAPI UI in production if not needed, or protect it behind authentication

## Agent Guidance

### Do
- Apply the principle: disable everything not explicitly required for the service to function
- Use `.env.template` to document all required environment variables alongside the service
- Set security headers via middleware — do not rely on the web server alone
- Validate production configuration as part of CI/CD before deployment
- Run configuration scanners (e.g., `bandit`, `checkov` for IaC) in the CI pipeline

### Do Not
- Commit `.env` files, credentials, or any file containing secret values to version control
- Leave DEBUG=True, verbose error pages, or stack traces exposed in any environment accessible outside development
- Use default credentials for any service component
- Assume the framework's default settings are secure — review and harden explicitly

## Checklist
- [ ] `DEBUG=False` and verbose error pages disabled in production configuration
- [ ] Security headers set: Content-Security-Policy, X-Frame-Options, X-Content-Type-Options, Strict-Transport-Security
- [ ] No default credentials on any service component (databases, admin panels, message queues)
- [ ] Directory listing and unnecessary endpoints disabled
- [ ] Error responses return generic messages; stack traces logged internally only
- [ ] `.env` in `.gitignore`; `.env.template` committed and up to date
- [ ] CORS allowlist is explicit — no wildcard origins for authenticated APIs
- [ ] Config scanner integrated in CI pipeline

## See Also
- wiki/tier1-sources/owasp/top10-2021-overview.md
- wiki/tier1-sources/owasp/a04-insecure-design.md
- wiki/tier1-sources/owasp/a09-logging-monitoring.md

## Source
OWASP Top 10:2021 — A05 Security Misconfiguration. https://owasp.org/Top10/A05_2021-Security_Misconfiguration/
