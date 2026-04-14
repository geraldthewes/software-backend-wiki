# A09: Security Logging and Monitoring Failures

> **Tier 1** | Source: OWASP Top 10:2021 | Authority: immutable

## Summary

Insufficient logging and monitoring allows attackers to persist in a system, pivot to other systems, and exfiltrate data without detection. Industry data shows the average time to detect a breach exceeds 200 days — largely because logging is inadequate to detect the attack pattern. For a coding agent, structured security logging is not optional: it is a core system feature that must be designed in from the start.

## Key Concepts

**What Makes Logging "Insufficient":**

- Authentication events not logged — impossible to detect credential stuffing or brute force
- Access control failures not logged — impossible to detect privilege escalation or IDOR attempts
- Input validation failures not logged — attack reconnaissance goes undetected
- High-value transactions not logged — financial fraud cannot be audited
- Logs stored only locally — destroyed when the host is compromised
- No alerting on anomalous patterns — logs exist but no one is notified

**Average breach detection time: 200+ days** — a direct consequence of inadequate logging.

## What to Log

**Always log these security events:**

| Event Category           | What to Capture                                              |
|--------------------------|--------------------------------------------------------------|
| Authentication           | Login attempts (success/failure), logout, MFA events        |
| Access control failures  | 403 responses, permission denied errors, IDOR attempts       |
| Input validation failures| Rejected inputs, schema validation errors                    |
| High-value transactions  | Payments, privilege changes, data exports, deletions         |
| System events            | Service start/stop, config changes, dependency errors        |

## What NOT to Log (pyscg-0019)

The following must NEVER appear in log output:

- Passwords (in any form, even hashed)
- Session tokens and API keys
- PII: names, email addresses, SSNs, phone numbers, addresses
- Credit card numbers or financial account numbers
- Private keys or cryptographic material
- Full request/response bodies when they may contain the above

```python
# WRONG — logging sensitive fields
logger.info(f"User {user.email} logged in with password {password}")
logger.debug(f"Request body: {request.body}")   # may contain passwords

# RIGHT — log identifiers, not sensitive values
logger.info("User login successful", extra={"user_id": user.id, "ip": request.remote_addr})
logger.warning("Login failed", extra={"user_id": user.id, "reason": "invalid_credentials"})
```

## Python Structured Logging

**Use `structlog` or `logging` with JSON formatter:**

```python
import structlog
import logging

# structlog configuration
structlog.configure(
    processors=[
        structlog.contextvars.merge_contextvars,
        structlog.processors.add_log_level,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.JSONRenderer(),
    ],
    logger_factory=structlog.PrintLoggerFactory(),
)

logger = structlog.get_logger()

# Structured log output — machine-readable, query-able
logger.info(
    "auth.login.success",
    user_id=user.id,
    ip=request.remote_addr,
    duration_ms=elapsed,
)

logger.warning(
    "auth.login.failed",
    user_id=attempted_user_id,
    ip=request.remote_addr,
    reason="invalid_credentials",
    attempt_count=failure_count,
)
```

**Required fields in every log entry:**

- `timestamp`: ISO 8601 format
- `level`: DEBUG, INFO, WARNING, ERROR, CRITICAL
- `correlation_id`: trace ID to connect related log events across services
- `user_id`: internal identifier (not email, name, or other PII)
- `action`: what was attempted
- `result`: success, failure, error
- `duration_ms`: for performance tracking (optional but valuable)

**Propagate correlation IDs across service boundaries:**

```python
import uuid
from contextvars import ContextVar

correlation_id: ContextVar[str] = ContextVar("correlation_id", default="")

def get_or_create_correlation_id(request) -> str:
    # Accept from upstream service or create new
    cid = request.headers.get("X-Correlation-ID") or str(uuid.uuid4())
    correlation_id.set(cid)
    return cid
```

**Log levels — use appropriately:**

| Level    | When to Use                                                        |
|----------|--------------------------------------------------------------------|
| DEBUG    | Development only; detailed internal state; never in production     |
| INFO     | Significant business events; successful operations worth recording |
| WARNING  | Unexpected but handled; something that should be investigated      |
| ERROR    | Failed operations; exceptions caught but operation could not complete |
| CRITICAL | System integrity at risk; service cannot continue safely           |

## Alerting Requirements

Define and configure alerts for:

- Authentication failures: >N failures from same IP in M minutes
- Privilege escalation attempts: any 403 on an admin endpoint
- Repeated input validation errors from same source: possible attack reconnaissance
- ERROR or CRITICAL log events: immediate notification
- Log volume anomalies: sudden drop may indicate logging failure

## Checklist
- [ ] All authentication events logged: attempts, successes, failures, logouts
- [ ] Access control failures (403) logged with sufficient context
- [ ] No PII, passwords, tokens, or secrets in any log output
- [ ] Structured JSON log format with timestamp, level, correlation_id, user_id, action, result
- [ ] Log levels used appropriately — no DEBUG in production output
- [ ] Alerts defined for authentication failure thresholds
- [ ] Alerts defined for CRITICAL and sustained ERROR conditions
- [ ] Logs shipped to centralized store — not stored only on the application host
- [ ] Correlation IDs created at system entry point and propagated to all downstream calls
- [ ] Log retention policy defined and configured

## See Also
- wiki/tier1-sources/owasp/top10-2021-overview.md
- wiki/tier1-sources/owasp/a01-broken-access-control.md
- wiki/tier1-sources/owasp/a07-auth-failures.md

## Source
OWASP Top 10:2021 — A09 Security Logging and Monitoring Failures. https://owasp.org/Top10/A09_2021-Security_Logging_and_Monitoring_Failures/
