# Metrics Guide

> **Tier 3** | Source: Prometheus docs, Google SRE Book | Enforces/Derives From: wiki/tier1-sources/swebok-v4/ka06-operations.md

## Summary

Metrics are numeric measurements sampled over time. They answer "how much?" and "how often?" — questions that logs cannot answer efficiently at scale. This page covers metric types, label design, the Four Golden Signals, and Python/Go implementation.

## Metric Types (Prometheus Model)

### Counter

A counter only increases (or resets to zero on restart). Use for counting discrete events.

```python
from prometheus_client import Counter

http_requests_total = Counter(
    "http_requests_total",
    "Total number of HTTP requests",
    ["method", "endpoint", "status_code"],
)

# Record a request
http_requests_total.labels(method="GET", endpoint="/users", status_code="200").inc()

# In PromQL: rate gives per-second rate over a window
# rate(http_requests_total[5m])
```

Use `_total` suffix for counters. In PromQL, `rate()` gives the per-second rate over a time window — this is the primary way to query counters.

### Gauge

A gauge can go up or down. Use for current state measurements.

```python
from prometheus_client import Gauge

active_connections = Gauge(
    "db_active_connections",
    "Number of active database connections",
)

queue_depth = Gauge(
    "job_queue_depth",
    "Number of jobs waiting in the queue",
    ["queue_name"],
)

# Usage
active_connections.set(42)
active_connections.inc()
active_connections.dec()

queue_depth.labels(queue_name="email").set(150)
```

### Histogram

A histogram samples observations and counts them in configurable buckets. Use for request latency and response sizes.

```python
from prometheus_client import Histogram

request_duration_seconds = Histogram(
    "http_request_duration_seconds",
    "HTTP request duration in seconds",
    ["method", "endpoint"],
    buckets=[0.01, 0.05, 0.1, 0.25, 0.5, 1.0, 2.5, 5.0, 10.0],
)

# Record a duration
with request_duration_seconds.labels(method="GET", endpoint="/users").time():
    result = process_request()

# Or manually
import time
start = time.perf_counter()
result = process_request()
request_duration_seconds.labels(method="GET", endpoint="/users").observe(
    time.perf_counter() - start
)
```

In PromQL, histograms give you percentiles:
```promql
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

### Summary

Summaries compute client-side percentiles. Avoid in distributed systems — summaries from different instances cannot be aggregated. Prefer histograms.

## Label Design

Labels are dimensions that let you slice and aggregate metrics. Good label design is critical — bad labels create millions of time series and bring down Prometheus.

### Good Labels (low cardinality)

```python
# Good — finite set of values (< 100 unique values per label)
Counter("requests_total", "...", ["http_method", "status_code", "service"])
# http_method: GET, POST, PUT, PATCH, DELETE  — 5 values
# status_code: 200, 201, 400, 401, 403, 404, 500  — ~10 values
# service: user-service, order-service  — ~10 values
# Total time series: 5 * 10 * 10 = 500 — manageable
```

### Bad Labels (high cardinality)

```python
# BAD — millions of unique values
Counter("requests_total", "...", ["user_id", "order_id", "session_id"])
# user_id alone: 10 million unique values = 10 million time series
```

Rule: keep label cardinality under 100 unique values per label. If you need to query by user ID, use logs or traces — not metrics.

## The Four Golden Signals (Google SRE)

Every service must measure all four. These are the minimum viable metrics for production services.

| Signal | Metric | Example PromQL |
|--------|--------|----------------|
| **Latency** | Request duration — distinguish success vs error | `histogram_quantile(0.99, rate(http_request_duration_seconds_bucket{status!~"5.."}[5m]))` |
| **Traffic** | Request rate | `rate(http_requests_total[5m])` |
| **Errors** | Error rate | `rate(http_requests_total{status_code=~"5.."}[5m])` |
| **Saturation** | How close to capacity | `db_active_connections / db_max_connections` |

## RED Method (for Services)

A focused subset of the Four Golden Signals, useful for microservice dashboards:

- **Rate**: requests per second — `rate(http_requests_total[5m])`
- **Errors**: errors per second — `rate(http_requests_total{status_code=~"5.."}[5m])`
- **Duration**: distribution of request latencies — histogram percentiles

## USE Method (for Resources)

For infrastructure components (databases, queues, servers):

- **Utilization**: % time the resource is busy — `cpu_usage_percent`
- **Saturation**: amount of extra work queued — `run_queue_length`, `io_wait_percent`
- **Errors**: count of error events — `disk_io_errors_total`

## Python: prometheus_client

```python
from prometheus_client import (
    Counter, Gauge, Histogram,
    start_http_server, push_to_gateway,
)

# Start a metrics endpoint for Prometheus to scrape
start_http_server(8000)  # metrics available at http://service:8000/metrics

# Long-running service — Prometheus scrapes the endpoint
# Batch job — push metrics to Pushgateway when done
push_to_gateway(
    "pushgateway:9091",
    job="batch_processor",
    registry=REGISTRY,
)
```

### Example: Instrument an HTTP Handler (FastAPI)

```python
from prometheus_client import Counter, Histogram
import time
from fastapi import FastAPI, Request

app = FastAPI()

REQUEST_COUNT = Counter(
    "http_requests_total",
    "Total HTTP requests",
    ["method", "endpoint", "status_code"],
)

REQUEST_LATENCY = Histogram(
    "http_request_duration_seconds",
    "HTTP request latency",
    ["method", "endpoint"],
)

@app.middleware("http")
async def metrics_middleware(request: Request, call_next):
    start = time.perf_counter()
    response = await call_next(request)
    duration = time.perf_counter() - start

    REQUEST_COUNT.labels(
        method=request.method,
        endpoint=request.url.path,
        status_code=str(response.status_code),
    ).inc()

    REQUEST_LATENCY.labels(
        method=request.method,
        endpoint=request.url.path,
    ).observe(duration)

    return response
```

## Go: prometheus/client_golang

```go
import (
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promauto"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
    requestsTotal = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total HTTP requests",
        },
        []string{"method", "endpoint", "status_code"},
    )

    requestDuration = promauto.NewHistogramVec(
        prometheus.HistogramOpts{
            Name:    "http_request_duration_seconds",
            Help:    "HTTP request latency",
            Buckets: prometheus.DefBuckets,
        },
        []string{"method", "endpoint"},
    )
)

// Expose metrics endpoint
http.Handle("/metrics", promhttp.Handler())
```

## See Also

- wiki/tier3-working/observability/slo-sli-sla.md
- wiki/tier3-working/observability/structured-logging.md
- wiki/tier1-sources/swebok-v4/ka06-operations.md

## Source

Prometheus documentation (prometheus.io). "Site Reliability Engineering" (Google, O'Reilly 2016) — Four Golden Signals, USE Method. "The USE Method" (Brendan Gregg, brendangregg.com). prometheus_client Python library (github.com/prometheus/client_python).
