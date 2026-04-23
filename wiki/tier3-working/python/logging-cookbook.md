# Python Logging Cookbook (Tier 3)

> **Tier 3** | Source: Python Logging Cookbook, docs.python.org/3/howto/logging-cookbook.html | Enforces/Derives From: wiki/tier3-working/python/logging.md, wiki/tier3-working/observability/structured-logging.md, wiki/tier1-sources/owasp/a09-logging-monitoring.md

## Summary

The Python Logging Cookbook provides production-ready recipes for common logging scenarios beyond basic setup: routing logs to multiple destinations, adding contextual information (correlation IDs, user context), handling multiprocessing and threading, non-blocking logging for slow handlers, log rotation with compression, and structured log output. These patterns are essential for building observable, production-grade services.

## Key Concepts

### Multi-Module Logging with Hierarchy

Logger names track the package hierarchy. Child loggers propagate to parents automatically:

```python
# main.py — application root logger
logger = logging.getLogger("myapp")

# services/auth.py — child logger, inherits handlers from 'myapp'
logger = logging.getLogger("myapp.services.auth")

# All 'myapp.*' loggers write to the same handlers without extra config
```

### Multiple Handlers with Different Levels

Route DEBUG to file, ERROR to console:

```python
logger = logging.getLogger("myapp")
logger.setLevel(logging.DEBUG)

fh = logging.FileHandler("debug.log")
fh.setLevel(logging.DEBUG)

ch = logging.StreamHandler()
ch.setLevel(logging.ERROR)

formatter = logging.Formatter("%(asctime)s %(name)s %(levelname)s %(message)s")
fh.setFormatter(formatter)
ch.setFormatter(formatter)

logger.addHandler(fh)
logger.addHandler(ch)
```

### Non-Blocking Logging with QueueHandler

SMTP, network, and slow file handlers block the request thread. Use a queue to decouple logging I/O:

```python
import queue
import logging.handlers

log_queue: queue.Queue = queue.Queue(-1)
queue_handler = logging.handlers.QueueHandler(log_queue)

slow_handler = logging.handlers.SMTPHandler(...)
listener = logging.handlers.QueueListener(log_queue, slow_handler, respect_handler_level=True)
listener.start()

# In Python 3.14+, use as a context manager
# with logging.handlers.QueueListener(log_queue, slow_handler) as listener:
#     ...

root = logging.getLogger()
root.addHandler(queue_handler)
```

### Adding Context with LoggerAdapter

Attach per-request context (correlation IDs, user IDs) without touching every log call:

```python
import logging
from typing import Any

class RequestAdapter(logging.LoggerAdapter):
    def process(self, msg: str, kwargs: dict) -> tuple[str, dict]:
        return "[req=%s] %s" % (self.extra.get("request_id", "-"), msg), kwargs

base_logger = logging.getLogger(__name__)

def handle_request(request_id: str) -> None:
    log = RequestAdapter(base_logger, {"request_id": request_id})
    log.info("Processing request")   # → "[req=abc123] Processing request"
```

### Context Variables for Async and Thread Safety (Python 3.7+)

Use `contextvars` rather than thread-locals for correct propagation in async code:

```python
from contextvars import ContextVar
import logging

ctx_request_id: ContextVar[str] = ContextVar("request_id", default="-")

class ContextFilter(logging.Filter):
    def filter(self, record: logging.LogRecord) -> bool:
        record.request_id = ctx_request_id.get()
        return True

handler = logging.StreamHandler()
handler.addFilter(ContextFilter())
logging.basicConfig(
    format="%(asctime)s %(levelname)s [%(request_id)s] %(message)s",
    handlers=[handler]
)

# In request handler:
ctx_request_id.set("abc-123")
logging.info("Request started")
```

### Multiprocessing Logging

File handlers are not safe to share across processes. Use a dedicated listener process:

```python
import multiprocessing
import logging.handlers

def worker(queue: multiprocessing.Queue) -> None:
    h = logging.handlers.QueueHandler(queue)
    root = logging.getLogger()
    root.addHandler(h)
    root.setLevel(logging.DEBUG)
    logging.getLogger(__name__).info("Worker running in pid=%d", os.getpid())

def listener(queue: multiprocessing.Queue) -> None:
    handler = logging.FileHandler("app.log")
    handler.setFormatter(
        logging.Formatter("%(asctime)s %(processName)s %(levelname)s %(message)s")
    )
    while True:
        record = queue.get()
        if record is None:
            break
        logging.getLogger(record.name).handle(record)

if __name__ == "__main__":
    q: multiprocessing.Queue = multiprocessing.Queue(-1)
    lp = multiprocessing.Process(target=listener, args=(q,))
    lp.start()
    workers = [multiprocessing.Process(target=worker, args=(q,)) for _ in range(4)]
    for w in workers:
        w.start()
    for w in workers:
        w.join()
    q.put(None)
    lp.join()
```

### Log Rotation with Compression

```python
import gzip, os, shutil
import logging.handlers

def namer(name: str) -> str:
    return name + ".gz"

def rotator(source: str, dest: str) -> None:
    with open(source, "rb") as f_in, gzip.open(dest, "wb") as f_out:
        shutil.copyfileobj(f_in, f_out)
    os.remove(source)

handler = logging.handlers.RotatingFileHandler("app.log", maxBytes=10_485_760, backupCount=5)
handler.rotator = rotator
handler.namer = namer
```

### Custom LogRecord Factory

Add custom fields to every log record without changing each call site:

```python
_old_factory = logging.getLogRecordFactory()

def record_factory(*args: object, **kwargs: object) -> logging.LogRecord:
    record = _old_factory(*args, **kwargs)
    record.app_version = "1.2.3"
    record.environment = os.environ.get("APP_ENV", "unknown")
    return record

logging.setLogRecordFactory(record_factory)
```

## Agent Guidance

### Do

- Use `QueueHandler` + `QueueListener` for any handler that performs I/O (SMTP, network, compressed rotation) to avoid blocking the application thread.
- Use `LoggerAdapter` or `ContextFilter` to inject per-request context (correlation IDs, user IDs) without modifying individual log calls.
- Use `contextvars.ContextVar` for context propagation in async and multi-threaded applications — thread-local storage does not propagate correctly through `asyncio` tasks.
- Configure `dictConfig` with `"disable_existing_loggers": False` to avoid breaking loggers created at import time.
- Use `RotatingFileHandler` with `backupCount` in all services that write to disk — unbounded log files fill disks.
- For multiprocessing, route all workers through a `QueueHandler` to a single listener that owns the file handler.

### Do Not

- Do not open the same log file from multiple processes without using a queue — file writes will interleave and corrupt records.
- Do not create per-user or per-request loggers dynamically — logger names are global and unbounded creation leaks memory.
- Do not add `StreamHandler` to library package loggers — only `NullHandler`.
- Do not use `%(message)s` string formatting if the message construction is expensive — use lazy `%s` arguments so the string is built only if the level is enabled.
- Do not log PII (emails, passwords, tokens) even at DEBUG — see `wiki/tier3-working/observability/structured-logging.md`.

## Checklist

- [ ] Slow handlers (SMTP, network) wrapped in `QueueHandler`/`QueueListener`
- [ ] Per-request context (correlation ID) injected via `LoggerAdapter` or `ContextFilter`
- [ ] `contextvars` used for async/thread context propagation, not thread-locals
- [ ] Multiprocessing workers use `QueueHandler` pointing to single listener
- [ ] `RotatingFileHandler` or `TimedRotatingFileHandler` used for file output in services
- [ ] `"disable_existing_loggers": False` set in `dictConfig`
- [ ] No PII in log messages at any level
- [ ] `backupCount` set on rotating handlers to limit disk usage

## See Also

- wiki/tier3-working/python/logging.md
- wiki/tier3-working/observability/structured-logging.md
- wiki/tier1-sources/owasp/a09-logging-monitoring.md
- wiki/tier3-working/python/async-patterns.md
- wiki/tier3-working/python/overview.md

## Source

Python Logging Cookbook, docs.python.org/3/howto/logging-cookbook.html. Python Standard Library `logging.handlers` module documentation.
