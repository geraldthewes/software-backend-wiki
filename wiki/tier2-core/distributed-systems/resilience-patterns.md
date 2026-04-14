# Resilience Patterns

> **Tier 2** | Source: Michael Nygard / Netflix OSS / industry practice | Derives From: ka02-architecture | Authority: established practice

## Summary

Resilience patterns are the standard toolbox for building distributed services that handle failures gracefully. A service that does not implement these patterns will fail in production in predictable ways. Every service that makes network calls must apply at minimum: explicit timeouts, retry with backoff, and idempotent mutations.

---

## Pattern 1: Retry with Exponential Backoff and Jitter

### Problem
Transient network failures — a connection reset, a brief service restart, a momentary overload — are common. Failing immediately on the first error is too aggressive; retrying as fast as possible causes a "thundering herd" that overwhelms an already-struggling service.

### Solution

Apply exponential backoff: each retry waits longer than the previous one. Add random jitter to prevent all callers from retrying simultaneously.

**Formula**: `wait = min(cap, base * 2^attempt) + random(0, jitter)`

| Parameter | Typical Value |
|-----------|---------------|
| `base` | 100 ms |
| `cap` | 30 seconds |
| `jitter` | 0–1000 ms (random) |
| Max attempts | 3–5 |

### When to Retry

- **Retry**: transient network errors, HTTP 429 (rate limit), HTTP 503 (service unavailable)
- **Do not retry**: HTTP 400 (bad request), HTTP 401 (unauthorized), HTTP 404 (not found), business logic failures
- **Only retry idempotent operations**: retrying a non-idempotent mutation (e.g., charge a credit card) can cause double-execution

### Python Example — tenacity Library

```python
from tenacity import (
    retry,
    stop_after_attempt,
    wait_exponential_jitter,
    retry_if_exception_type,
)
import httpx

@retry(
    stop=stop_after_attempt(4),
    wait=wait_exponential_jitter(initial=0.1, max=30, jitter=1),
    retry=retry_if_exception_type((httpx.TransportError, httpx.TimeoutException)),
)
def fetch_order(order_id: int) -> dict:
    response = httpx.get(
        f"https://orders-service/orders/{order_id}",
        timeout=(2.0, 10.0),  # (connect, read)
    )
    response.raise_for_status()
    return response.json()
```

---

## Pattern 2: Circuit Breaker

### Problem
When a downstream service is failing, retrying every request burns resources on the caller and adds load to a service that is already struggling to recover. The caller should "give up temporarily" and allow the downstream service to recover.

### Solution

The circuit breaker monitors failure rate. When failures exceed a threshold, it "opens" the circuit: subsequent calls fail immediately without attempting the network call. After a timeout, it "half-opens" to test whether the downstream has recovered.

### States

```
Closed (normal)  ──[N failures in window]──►  Open (fast fail)
                                                    │
                                             [timeout expires]
                                                    ▼
                                              Half-Open (test)
                                             /              \
                                    [success]              [failure]
                                        │                      │
                                        ▼                      ▼
                                     Closed                  Open
```

| State | Behavior | Transitions |
|-------|----------|-------------|
| **Closed** | Normal operation; calls pass through | → Open after N failures in window |
| **Open** | Fast fail: raise error immediately | → Half-Open after timeout (e.g., 60 s) |
| **Half-Open** | Allow one test call through | → Closed on success; → Open on failure |

### Python Example — pybreaker Library

```python
import pybreaker
import httpx

breaker = pybreaker.CircuitBreaker(
    fail_max=5,          # open after 5 failures
    reset_timeout=60,    # half-open after 60 seconds
)

@breaker
def call_payment_service(amount: float, token: str) -> dict:
    response = httpx.post(
        "https://payments-service/charge",
        json={"amount": amount, "token": token},
        timeout=(2.0, 15.0),
    )
    response.raise_for_status()
    return response.json()
```

---

## Pattern 3: Timeout

### Problem
Without an explicit timeout, a network call that does not receive a response waits indefinitely. One slow downstream can exhaust all available threads or connections in the caller, bringing it down.

### Solution

Set **explicit timeouts on every network call**. Never rely on the library default (which is often "no timeout" or extremely large).

### Timeout Values

| Timeout Type | Typical Range | Notes |
|-------------|---------------|-------|
| Connection timeout | 1–5 s | Time to establish the TCP connection |
| Read timeout | 5–60 s | Time to receive the response after the request is sent |
| Total timeout | connection + read | Set both independently when the library supports it |

### Python and Go Examples

```python
# Python — httpx
import httpx
response = httpx.get(
    "https://service/resource",
    timeout=httpx.Timeout(connect=2.0, read=30.0, write=5.0, pool=1.0),
)

# Python — psycopg (PostgreSQL)
import psycopg
conn = psycopg.connect(DATABASE_URL, connect_timeout=5)
conn.execute("SET statement_timeout = '30s'")
```

```go
// Go — context with timeout
ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
defer cancel()

req, _ := http.NewRequestWithContext(ctx, "GET", "https://service/resource", nil)
resp, err := http.DefaultClient.Do(req)
```

---

## Pattern 4: Bulkhead

### Problem
A single pool of threads or connections services all downstream dependencies. When one slow dependency saturates the pool, no other dependency can be reached — a single point of failure cascades to affect the entire service.

### Solution

Isolate resources for different consumers. Use separate thread pools, connection pools, or async task queues for each distinct downstream dependency.

### Implementation

```python
# Separate connection pool per downstream service
import httpx

order_client = httpx.AsyncClient(
    base_url="https://orders-service",
    limits=httpx.Limits(max_connections=20, max_keepalive_connections=10),
)
payment_client = httpx.AsyncClient(
    base_url="https://payments-service",
    limits=httpx.Limits(max_connections=5, max_keepalive_connections=2),
)
# payment_service slowness cannot exhaust order_service connections
```

---

## Pattern 5: Idempotency

### Problem
Retries cause mutations to execute more than once. Charging a credit card twice, creating two accounts, or sending two welcome emails are all retry-induced double-execution failures.

### Solution

Design mutations to be safe to execute multiple times with the same result. Strategies:

| Strategy | Implementation | Example |
|----------|---------------|---------|
| **Natural idempotency** | Operations that are inherently idempotent | `PUT /users/{id}` with full replacement |
| **Idempotency keys** | Client generates unique key; server deduplicates | `POST /charges` with `Idempotency-Key: uuid` |
| **DB upserts** | `INSERT ... ON CONFLICT DO NOTHING / DO UPDATE` | Create-or-update semantics |
| **Conditional updates** | Check precondition before mutation | `UPDATE SET status='shipped' WHERE status='pending'` |

```python
import uuid, psycopg

idempotency_key = str(uuid.uuid4())  # generated by caller, sent in request

def create_charge(amount: int, token: str, key: str) -> dict:
    with psycopg.connect(DATABASE_URL) as conn:
        result = conn.execute(
            """
            INSERT INTO charges (idempotency_key, amount, token, status)
            VALUES (%s, %s, %s, 'pending')
            ON CONFLICT (idempotency_key) DO NOTHING
            RETURNING id, status
            """,
            (key, amount, token),
        ).fetchone()
        if result is None:
            # Already processed — return existing result
            return conn.execute(
                "SELECT id, status FROM charges WHERE idempotency_key = %s", (key,)
            ).fetchone()
        return dict(result)
```

---

## Pattern 6: Health Checks

### Problem
A process is running but cannot serve traffic (database connection lost, background worker deadlocked, configuration missing). Without health checks, load balancers route traffic to a broken instance.

### Solution

Implement two distinct health check endpoints:

| Endpoint | Purpose | Checks |
|----------|---------|--------|
| `/health/live` | **Liveness**: is the process alive? | Process is running; not deadlocked |
| `/health/ready` | **Readiness**: is it ready to serve traffic? | Database reachable; required config present; dependencies available |

```python
from fastapi import FastAPI, HTTPException
import psycopg

app = FastAPI()

@app.get("/health/live")
def liveness():
    return {"status": "ok"}

@app.get("/health/ready")
def readiness():
    try:
        with psycopg.connect(DATABASE_URL, connect_timeout=2) as conn:
            conn.execute("SELECT 1")
        return {"status": "ok", "database": "connected"}
    except Exception as exc:
        raise HTTPException(status_code=503, detail=f"Database unavailable: {exc}")
```

---

## Agent Guidance

### Do
- Apply timeouts on every network call — no exceptions
- Apply retry with exponential backoff and jitter on transient failures
- Apply circuit breakers on high-traffic paths to any downstream service
- Design all mutations to be idempotent before implementing retry
- Implement both `/health/live` and `/health/ready` on every service

### Do Not
- Do not retry non-idempotent mutations without idempotency keys
- Do not retry on business logic failures (4xx except 429)
- Do not use a single connection pool for multiple downstream services (bulkhead violation)
- Do not rely on library default timeouts — always set explicitly

## Checklist
- [ ] All network calls have explicit connect and read timeouts
- [ ] Retry is implemented with exponential backoff and jitter
- [ ] Circuit breaker is in place for high-traffic downstream dependencies
- [ ] All mutations are idempotent or use idempotency keys
- [ ] Separate connection/resource pools are used for each downstream dependency
- [ ] `/health/live` and `/health/ready` endpoints are implemented and tested

## See Also
- `wiki/tier2-core/distributed-systems/fallacies.md`
- `wiki/tier2-core/distributed-systems/cap-pacelc.md`
- `wiki/tier2-core/distributed-systems/overview.md`
- `wiki/tier1-sources/swebok-v4/ka02-architecture.md`

## Source

Michael Nygard, *Release It!* (2007); Netflix Hystrix patterns; tenacity and pybreaker library documentation. Synthesized from *Software Development Best Practices for Agent* reference document.
