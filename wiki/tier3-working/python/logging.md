# Python Logging (Tier 3)

> **Tier 3** | Source: Python Logging HOWTO, docs.python.org/3/howto/logging.html | Enforces/Derives From: wiki/tier3-working/python/overview.md, wiki/tier3-working/observability/structured-logging.md, wiki/tier1-sources/owasp/a09-logging-monitoring.md

## Summary

Python's `logging` module provides a production-grade framework for recording application events. Unlike `print()`, it supports log levels, hierarchical loggers, multiple output destinations, and structured formatting — all configurable without changing application code. A coding agent must use `logging` rather than `print()` for all event recording in library and service code, and must never swallow exceptions silently.

The logging hierarchy is built from logger names using dot notation (`scan.html` is a child of `scan`), enabling fine-grained control. Libraries add only a `NullHandler`; applications configure handlers at startup.

## Key Concepts

### Log Levels

| Level | Numeric | When to Use |
|-------|---------|-------------|
| `DEBUG` | 10 | Detailed diagnostic — enabled only in development |
| `INFO` | 20 | Confirmation of normal operation |
| `WARNING` | 30 | Unexpected situation that is recoverable (default threshold) |
| `ERROR` | 40 | Function could not complete; will affect the caller |
| `CRITICAL` | 50 | Program may be unable to continue |

### Logger Hierarchy and Naming

```python
# Always use module-level __name__ — creates the correct hierarchy automatically
logger = logging.getLogger(__name__)
# e.g., in package/module.py → logger name becomes 'package.module'
```

Child loggers propagate log records to parent handlers unless `propagate=False`. The root logger is the ultimate ancestor.

### Configuration Approaches

| Approach | When to Use |
|----------|-------------|
| `logging.basicConfig()` | Scripts, small applications — call once before any log call |
| Explicit handler/formatter code | Medium applications needing multiple outputs |
| `dictConfig()` with YAML/dict | Production services — separates config from code |
| `fileConfig()` | Legacy INI-based config |

### Handler Types

| Handler | Use Case |
|---------|----------|
| `StreamHandler` | Console output (stdout/stderr) |
| `FileHandler` | Persistent log files |
| `RotatingFileHandler` | Log files with size-based rotation |
| `TimedRotatingFileHandler` | Log files with time-based rotation |
| `QueueHandler` + `QueueListener` | Non-blocking logging (SMTP, network, slow I/O) |
| `NullHandler` | Library default — silences "no handlers" warning |
| `SocketHandler` | Send log records over TCP to a central log server |

## Agent Guidance

### Do

- Use `logger = logging.getLogger(__name__)` at module level — one logger per module.
- Configure logging once at application startup using `dictConfig()` or `basicConfig()` before any log calls are made.
- Use `logger.exception("message")` inside `except` blocks — it automatically includes the full traceback.
- Pass variables as format arguments, not with string concatenation: `logger.info("User %s logged in", user_id)` — this defers string formatting until the record is actually emitted.
- In library code, add only `logging.NullHandler()` to the package logger so the application controls output.
- Set `disable_existing_loggers: False` in `dictConfig`/`fileConfig` to avoid silencing loggers created before configuration runs.
- Gate expensive debug computations: `if logger.isEnabledFor(logging.DEBUG): logger.debug("Computed: %s", expensive())`.
- For multiprocessing, route all log records through a `QueueHandler` to a single listener process — file handlers are not process-safe.

### Do Not

- Do not use `print()` for diagnostic or operational logging in library or service code.
- Do not call `logging.basicConfig()` in library code — only the application should configure logging.
- Do not add non-`NullHandler` handlers in library packages.
- Do not call `logger.debug(expensive_func())` — the argument is evaluated even when DEBUG is disabled.
- Do not swallow exceptions and log nothing — always log at `ERROR` or higher before suppressing.
- Do not log sensitive data (passwords, tokens, PII) — see `wiki/tier3-working/observability/structured-logging.md`.
- Do not create per-request or per-user loggers in a loop — logger names are global and unbounded creation leaks memory.

## Common Configuration Pattern

```python
import logging
import logging.config

LOGGING_CONFIG = {
    "version": 1,
    "disable_existing_loggers": False,
    "formatters": {
        "standard": {
            "format": "%(asctime)s %(name)s %(levelname)s %(message)s"
        }
    },
    "handlers": {
        "console": {
            "class": "logging.StreamHandler",
            "level": "INFO",
            "formatter": "standard",
            "stream": "ext://sys.stdout"
        },
        "file": {
            "class": "logging.handlers.RotatingFileHandler",
            "level": "DEBUG",
            "formatter": "standard",
            "filename": "app.log",
            "maxBytes": 10485760,   # 10 MB
            "backupCount": 5
        }
    },
    "root": {
        "level": "DEBUG",
        "handlers": ["console", "file"]
    }
}

logging.config.dictConfig(LOGGING_CONFIG)
logger = logging.getLogger(__name__)
```

## Exception Logging

```python
# Use logger.exception() — includes full traceback automatically
try:
    result = process(data)
except ValueError as e:
    logger.exception("Processing failed for data=%r", data)
    raise  # re-raise after logging, do not swallow

# For non-fatal errors use logger.error() with exc_info=True
try:
    notify_user(user_id)
except NotificationError:
    logger.error("Failed to notify user %s", user_id, exc_info=True)
```

## CLI Log Level Pattern

```python
import argparse, logging

parser = argparse.ArgumentParser()
parser.add_argument("--log-level", default="INFO")
args = parser.parse_args()

level = getattr(logging, args.log_level.upper(), None)
if not isinstance(level, int):
    raise ValueError(f"Invalid log level: {args.log_level}")
logging.basicConfig(level=level)
```

## Checklist

- [ ] Every module defines `logger = logging.getLogger(__name__)` at module level
- [ ] No `print()` calls used for operational logging
- [ ] `dictConfig` or `basicConfig` is called once at application startup
- [ ] Library packages add only `NullHandler()` to their root logger
- [ ] `logger.exception()` is used inside all `except` blocks
- [ ] No sensitive data (tokens, passwords, PII) in log messages
- [ ] `disable_existing_loggers: False` set in `dictConfig`
- [ ] Rotating handlers used for file-based logging in services
- [ ] `QueueHandler`/`QueueListener` used in multiprocess applications
- [ ] Log levels are appropriate: DEBUG for dev detail, INFO for operations, WARNING+ for problems

## See Also

- wiki/tier3-working/observability/structured-logging.md
- wiki/tier3-working/python/logging-cookbook.md
- wiki/tier1-sources/owasp/a09-logging-monitoring.md
- wiki/tier3-working/python/overview.md
- wiki/tier3-working/checklists/pre-commit.md

## Source

Python Logging HOWTO, docs.python.org/3/howto/logging.html. Python Standard Library `logging` module documentation.
