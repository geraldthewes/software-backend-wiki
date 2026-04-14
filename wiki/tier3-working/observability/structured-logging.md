# Structured Logging

> **Tier 3** | Source: structlog docs, OWASP A09 | Enforces/Derives From: wiki/tier1-sources/owasp/a09-logging-monitoring.md, wiki/tier1-sources/swebok-v4/ka06-operations.md

## Summary

Structured logging emits machine-parseable JSON log lines instead of free-form text strings. This enables querying, filtering, and alerting on specific fields in log management systems (Splunk, ELK, Loki, Datadog). Free-form text logs cannot be reliably queried at scale.

## Why Structured Logging

```
# Bad — free-form text: unsearchable, unparseable
2025-01-15 10:30:00 ERROR Failed to process order 12345 for user alice

# Good — structured JSON: queryable on every field
{"timestamp":"2025-01-15T10:30:00Z","level":"error","service":"order-service","correlation_id":"abc-123","user_id":42,"order_id":12345,"error":"payment timeout","message":"Failed to process order"}
```

In Loki or Splunk: `{service="order-service"} | json | level="error" | order_id=12345` returns exactly the relevant lines.

## Log Levels

Use the correct level. Miscategorized logs make alerting unreliable and flood production with noise.

| Level | When to Use | Production Default |
|-------|-------------|-------------------|
| DEBUG | Step-by-step diagnostic detail; variable values; all I/O | OFF — never emit in production by default |
| INFO | Significant events: service started, request completed, job finished, user logged in | ON |
| WARNING | Unexpected but handled condition; operation continued with degraded behavior | ON |
| ERROR | Operation failed; investigation required; service continues running | ON — alert on elevated rate |
| CRITICAL | System integrity at risk; data loss possible; immediate action required | ON — always triggers alert |

## Required Fields in Every Log Entry

Every log line must contain:

| Field | Type | Example |
|-------|------|---------|
| `timestamp` | ISO 8601 UTC | `"2025-01-15T10:30:00.123Z"` |
| `level` | string | `"error"` |
| `service` | string | `"user-service"` |
| `correlation_id` | string | `"abc-123-def-456"` |
| `message` | string | `"User registration failed"` |

Additional context fields depend on what is being logged.

## Python: structlog

structlog is the preferred logging library for new Python services. It produces structured output and supports context binding.

### Configuration (production)

```python
import logging
import structlog

def configure_logging(debug: bool = False) -> None:
    shared_processors = [
        structlog.stdlib.add_log_level,
        structlog.stdlib.add_logger_name,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.StackInfoRenderer(),
    ]

    if debug:
        # Pretty output for local development
        processors = shared_processors + [
            structlog.dev.ConsoleRenderer()
        ]
    else:
        # JSON output for production
        processors = shared_processors + [
            structlog.processors.format_exc_info,
            structlog.processors.JSONRenderer(),
        ]

    structlog.configure(
        processors=processors,
        wrapper_class=structlog.stdlib.BoundLogger,
        context_class=dict,
        logger_factory=structlog.stdlib.LoggerFactory(),
        cache_logger_on_first_use=True,
    )

    # Configure stdlib logging to not conflict
    logging.basicConfig(
        format="%(message)s",
        level=logging.DEBUG if debug else logging.INFO,
    )
```

### Usage

```python
import structlog

logger = structlog.get_logger(__name__)

# Basic log
logger.info("service_started", port=8000, version="1.2.3")

# Context binding — all subsequent logs from this logger include these fields
request_log = logger.bind(
    correlation_id="abc-123",
    user_id=42,
    action="create_order",
)

request_log.info("order_processing_started", order_id=100)

try:
    result = process_order(order_id=100)
    request_log.info("order_created", order_id=100, total=result.total)
except PaymentError as e:
    request_log.error("order_payment_failed", order_id=100, error=str(e))
    raise
```

### Exception Logging

```python
# logger.exception() automatically includes the traceback
try:
    result = risky_operation()
except Exception:
    logger.exception("unexpected_error", operation="risky_operation", context=ctx)
    raise

# Or explicitly:
try:
    result = risky_operation()
except SpecificError as e:
    logger.error("operation_failed", error=str(e), exc_info=True)
```

## Python: Standard Library Alternative

For projects using the standard library `logging` module:

```python
import logging
import json

class JSONFormatter(logging.Formatter):
    def format(self, record: logging.LogRecord) -> str:
        log_object = {
            "timestamp": self.formatTime(record, "%Y-%m-%dT%H:%M:%S.%f") + "Z",
            "level": record.levelname.lower(),
            "logger": record.name,
            "message": record.getMessage(),
        }
        if record.exc_info:
            log_object["exception"] = self.formatException(record.exc_info)
        # Include extra fields
        for key, value in record.__dict__.items():
            if key not in logging.LogRecord.__dict__ and not key.startswith("_"):
                log_object[key] = value
        return json.dumps(log_object)

handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logging.getLogger().addHandler(handler)
```

## What to Log

Log these events at INFO or above:

- Authentication events: login success, login failure, logout, token refresh.
- Access control decisions: permission granted, permission denied (include resource and action).
- All ERROR and CRITICAL conditions with full context.
- Significant business events: order placed, payment processed, account created.
- External dependency failures: database connection failed, upstream HTTP 5xx, timeout.
- Slow queries or operations: queries exceeding 500ms, requests exceeding SLO threshold.
- System lifecycle: service started, configuration loaded, graceful shutdown initiated.

## What NEVER to Log (pyscg-0019)

Logging sensitive data creates compliance violations and security incidents. Never log:

- Passwords or password hashes.
- Session tokens, JWTs, API keys, OAuth tokens.
- Full credit card numbers, CVVs, bank account numbers.
- Social Security Numbers, national ID numbers, passport numbers.
- Raw PII in high-volume logs — log `user_id` not `email` for request-level logs. Aggregate PII access in audit logs.
- Private keys or cryptographic secrets.
- Full HTTP request/response bodies (may contain any of the above).

```python
# BAD — token in log
logger.info("auth_success", user=user.email, token=access_token)

# GOOD — log user_id only; no token
logger.info("auth_success", user_id=user.id)
```

## Log Sampling

High-volume debug and info logs can be sampled in production to reduce cost without losing visibility into errors.

- ERROR and CRITICAL: always log — never sample.
- INFO for health-check endpoints or high-frequency status: sample at 10% or 1%.
- DEBUG: off in production by default; enable per-service for incident investigation.

## Correlation ID Propagation

```python
# FastAPI middleware: extract or generate, set on context, forward in response
import contextvars
import uuid

correlation_id_var: contextvars.ContextVar[str] = contextvars.ContextVar(
    "correlation_id", default=""
)

# In log processor: include correlation_id automatically
def add_correlation_id(logger, method, event_dict):
    event_dict["correlation_id"] = correlation_id_var.get("unknown")
    return event_dict

# Forward to downstream HTTP calls
headers = {"X-Correlation-ID": correlation_id_var.get()}
response = await http_client.get(url, headers=headers)
```

## See Also

- wiki/tier3-working/observability/metrics.md
- wiki/tier3-working/observability/slo-sli-sla.md
- wiki/tier1-sources/owasp/a09-logging-monitoring.md

## Source

structlog documentation (structlog.readthedocs.io). OWASP A09:2021 — Security Logging and Monitoring Failures. OpenSSF Pyscg-0019: Exclude sensitive data from logs. "The Art of Monitoring" (Turnbull, 2016).
