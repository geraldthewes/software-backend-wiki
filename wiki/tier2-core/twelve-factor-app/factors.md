# 12-Factor App — All Factors in Detail

> **Tier 2** | Source: Adam Wiggins / Heroku (2011) | Derives From: ka02-architecture, ka06-operations, ka04-construction | Authority: established practice

## Summary

Detailed treatment of all twelve factors with agent implications and Python/Go implementation examples.

---

## Factor I — Codebase

**Principle**: One codebase tracked in version control; many deploys from the same codebase.

- One Git repository per service
- Branches (not forks) for feature development
- Multiple environments (dev, staging, prod) are different deploys of the **same** codebase; configuration differs, code does not
- Multiple services sharing a codebase → extract a shared library; each service still has its own repo

**Agent implication**: Never create environment-specific code branches in source. An `if env == "production":` block in application logic is a violation. Use environment variables for environment-specific values (Factor III).

```python
# Violation
if os.environ.get("ENVIRONMENT") == "production":
    database_url = "postgres://prod-server/mydb"
else:
    database_url = "sqlite:///local.db"

# Correct: one codebase, config via environment
database_url = os.environ["DATABASE_URL"]  # differs per deploy, not per branch
```

---

## Factor II — Dependencies

**Principle**: Explicitly declare and isolate all dependencies.

- Declare all dependencies in a manifest: `pyproject.toml`, `Pipfile`, or `requirements.txt` for Python; `go.mod` for Go
- Use isolation: `virtualenv` or `venv` for Python; Go modules are isolated by default
- Never assume system-installed libraries (`libxml2`, `imagemagick`, etc.)
- Pin direct dependency versions; use lock files (`Pipfile.lock`, `poetry.lock`) for reproducible builds

**Agent implication**: Always use a virtual environment. Never call `pip install` globally. Never assume `curl`, `ffmpeg`, or any system binary is present — declare it or bundle it.

```toml
# pyproject.toml — explicit dependency declaration
[project]
dependencies = [
    "fastapi>=0.111.0",
    "psycopg[binary]>=3.1.18",
    "structlog>=24.1.0",
    "tenacity>=8.3.0",
]
```

---

## Factor III — Config

**Principle**: Store ALL configuration in environment variables.

- Strict separation of config from code
- Config is anything that varies between deploys: database URLs, API keys, feature flags, hostnames, port numbers
- No config files committed to the repository (no `settings.py` with environment-specific values)
- `.env` files are for local development only — **never commit them**
- `.env.template` (showing required variables with placeholder values) **must always be committed**

**Agent implication**: Every configurable value goes through `os.environ`. Use `python-dotenv` locally. Use a secrets manager (Vault, AWS Secrets Manager) in production. Document every environment variable in the Dockerfile.

```python
import os
from dotenv import load_dotenv

load_dotenv()  # loads .env for local dev; no-op in prod where vars are injected

DATABASE_URL: str = os.environ["DATABASE_URL"]       # required — fail fast if missing
LOG_LEVEL: str = os.environ.get("LOG_LEVEL", "INFO") # optional with default
SECRET_KEY: str = os.environ["SECRET_KEY"]
```

```bash
# .env.template — COMMIT THIS
DATABASE_URL=postgres://user:password@localhost:5432/mydb
SECRET_KEY=change-me-in-production
LOG_LEVEL=DEBUG
```

---

## Factor IV — Backing Services

**Principle**: Treat backing services (databases, queues, caches, email services) as attached resources.

- A backing service is any service the app consumes over the network
- Local services (PostgreSQL running on localhost) and third-party services (AWS RDS, SendGrid) are treated identically — accessed via URL from config
- Swap a local database for a managed cloud database by changing an environment variable, not code

**Agent implication**: The database URL goes in an environment variable. Code never references `localhost` for a backing service. Swapping databases must require zero code changes.

```python
# Go example
import "os"

dbURL := os.Getenv("DATABASE_URL")        // postgres://...
cacheURL := os.Getenv("REDIS_URL")        // redis://...
queueURL := os.Getenv("AMQP_URL")         // amqp://...
```

---

## Factor V — Build, Release, Run

**Principle**: Strictly separate the build, release, and run stages.

- **Build stage**: convert source code into an executable bundle (compile, fetch dependencies, build assets)
- **Release stage**: combine the build with environment-specific config to produce a deployable release
- **Run stage**: execute a release in the target environment

Each release is immutable and uniquely identified. You cannot change code at runtime. You cannot deploy uncommitted code.

**Agent implication**: Builds must be reproducible. Never modify running containers. Rollback means deploying a previous release artifact, not reverting running state. CI/CD pipelines enforce this separation.

---

## Factor VI — Processes

**Principle**: Execute the app as one or more stateless, share-nothing processes.

- Processes are stateless: no in-memory user sessions, no in-process caches that must be "warm"
- No sticky sessions: any process instance must be able to handle any request
- Session state, file uploads, and accumulated state live in a backing service (Redis, S3, PostgreSQL) — not in process memory
- Horizontal scaling (running more process instances) must work without coordination

**Agent implication**: Never store user session data in a Python dict or global variable. If a user's next request can be routed to a different process instance by a load balancer, that state must be in Redis or the database.

```python
# Violation: in-memory session
_sessions: dict[str, dict] = {}  # lost on restart or second instance

# Correct: session in Redis
import redis
r = redis.from_url(os.environ["REDIS_URL"])
r.setex(f"session:{session_id}", 3600, json.dumps(session_data))
```

---

## Factor VII — Port Binding

**Principle**: Export services via port binding.

- The app is self-contained and binds to a port to listen for requests
- It does not rely on runtime injection of an external web server (Apache, Nginx) to function
- In development: `http://localhost:8000`; in production: the platform routes traffic to the bound port

**Agent implication**: Use a production-grade ASGI/WSGI server embedded in the app (uvicorn, gunicorn). The app exports an HTTP interface on the port specified by the `PORT` environment variable.

```python
import uvicorn, os
uvicorn.run("app:app", host="0.0.0.0", port=int(os.environ.get("PORT", 8000)))
```

---

## Factor VIII — Concurrency

**Principle**: Scale out via the process model.

- Assign work to process types (web, worker, scheduler)
- Scale each process type independently: 10 web processes + 2 worker processes
- Rely on the operating system process manager (systemd, Nomad, Kubernetes) — not internal thread multiplexing as the primary scaling mechanism
- Horizontal scaling of stateless processes (Factor VI) must work

**Agent implication**: Design services so that running multiple instances is safe by default. Avoid shared mutable global state. Use a process supervisor for long-running services.

---

## Factor IX — Disposability

**Principle**: Maximize robustness with fast startup and graceful shutdown.

- **Fast startup**: processes should be ready to serve traffic within 30 seconds of launch
- **Graceful shutdown**: on `SIGTERM`, the process stops accepting new requests, finishes in-flight requests, and exits cleanly
- **Crash-only design**: it is acceptable for a process to exit on unrecoverable error; the process manager will restart it

**Agent implication**: Implement SIGTERM handlers. In Go, use `context.WithTimeout` for cancellation. In Python, use the `signal` module. Never use `atexit` as the sole cleanup mechanism — it does not fire on SIGTERM.

```python
import signal, sys

def handle_sigterm(signum, frame):
    # finish in-flight work, close DB connections
    sys.exit(0)

signal.signal(signal.SIGTERM, handle_sigterm)
```

```go
ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
defer stop()
// pass ctx to all goroutines; they shut down when ctx is cancelled
```

---

## Factor X — Dev/Prod Parity

**Principle**: Keep development, staging, and production as similar as possible.

- Minimize the gap in time (deploy often), personnel (same team deploys), and tools (same backing services)
- Do not substitute SQLite for PostgreSQL in development if production uses PostgreSQL — behavior differences cause bugs that only appear in production
- Use Docker locally to match the production runtime environment
- Use the same version of every backing service in all environments

**Agent implication**: If production uses PostgreSQL 16, development must use PostgreSQL 16. If production uses Redis 7, development must use Redis 7. Use `docker-compose.yml` to run backing services locally.

---

## Factor XI — Logs

**Principle**: Treat logs as event streams; write to stdout and stderr only.

- The app never concerns itself with routing or storing its log output
- Logs are written to stdout as a stream of time-ordered events
- The execution environment captures stdout and routes it to a log aggregator (Loki, Splunk, CloudWatch)
- No log file management: no log rotation, no log directories, no log file handles

**Agent implication**: Use `structlog` in Python. Configure it to write JSON to stdout. Never open a file handler in the logging configuration. Never call `logging.FileHandler`. Let the platform route logs.

```python
import structlog, logging, sys

logging.basicConfig(
    format="%(message)s",
    stream=sys.stdout,
    level=logging.INFO,
)
structlog.configure(
    processors=[
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.JSONRenderer(),
    ]
)
logger = structlog.get_logger()
logger.info("order_placed", order_id=42, customer_id=7)
```

---

## Factor XII — Admin Processes

**Principle**: Run admin/management tasks as one-off processes in the identical environment.

- Database migrations, data backups, one-off scripts, REPL sessions are admin processes
- They run against a release with the same config, the same codebase, the same environment variables
- They are one-off: not part of the long-running process model
- They are never run inside the application's startup sequence

**Agent implication**: Database migrations are never called inside `app.run()` or the FastAPI lifespan event. They are run as a separate command (`alembic upgrade head`) before or alongside deployment — but as a distinct process.

```bash
# Correct: migration as admin process (run before or alongside deploy)
alembic upgrade head

# Violation: migration inside app startup
@app.on_event("startup")
async def startup():
    run_migrations()  # Never do this in production
```

---

## See Also
- `wiki/tier2-core/twelve-factor-app/overview.md`
- `wiki/tier1-sources/swebok-v4/ka02-architecture.md`
- `wiki/tier1-sources/swebok-v4/ka06-operations.md`
- `wiki/tier2-core/distributed-systems/overview.md`

## Source

Adam Wiggins, *The Twelve-Factor App* (2011), https://12factor.net. Synthesized from *Software Development Best Practices for Agent* reference document.
