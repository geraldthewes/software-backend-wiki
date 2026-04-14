# Observability — Overview

> **Tier 3** | Source: Google SRE Book, OpenTelemetry | Enforces/Derives From: wiki/tier1-sources/swebok-v4/ka06-operations.md, wiki/tier1-sources/owasp/a09-logging-monitoring.md

## Summary

Observability is the ability to understand the internal state of a running system from its external outputs. A system that cannot be observed cannot be debugged, and failures cannot be detected until users report them. The three pillars — logs, metrics, and traces — together provide full visibility.

## Observability Definition

A system is observable when you can answer the question "what is happening inside?" using only the outputs the system produces. Contrast with monitoring (checking known failure modes) — observability handles *unknown* failures too.

The difference matters for autonomous agents and microservices: when a new failure mode emerges, you need enough data to investigate it without deploying new instrumentation first.

## Three Pillars

| Pillar | What It Answers | Format | Tool Examples |
|--------|----------------|--------|---------------|
| **Logs** | "What happened?" — discrete events | Structured JSON | Loki, ELK, Splunk, CloudWatch |
| **Metrics** | "How much / how often?" — numeric measurements | Time series | Prometheus, Datadog, CloudWatch |
| **Traces** | "Where did the time go?" — request flow across services | Spans with parent/child | Jaeger, Tempo, Zipkin |

All three are needed. Each answers a question the others cannot:
- Logs tell you a specific error occurred but not how frequent or how widespread.
- Metrics tell you the error rate spiked but not what the errors contain.
- Traces show which service in a chain caused the latency but not the log detail.

## OpenTelemetry

OpenTelemetry (OTel) is the vendor-neutral standard for instrumentation. A single OTel SDK emits logs, metrics, and traces, and the backend (Jaeger, Prometheus, etc.) is a configuration choice.

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter

provider = TracerProvider()
provider.add_span_processor(BatchSpanProcessor(OTLPSpanExporter(endpoint="http://otel-collector:4317")))
trace.set_tracer_provider(provider)

tracer = trace.get_tracer(__name__)

with tracer.start_as_current_span("fetch_user") as span:
    span.set_attribute("user.id", user_id)
    user = repository.get(user_id)
```

## Correlation IDs

A correlation ID is a unique identifier generated at the system entry point (HTTP request, message received) and propagated through every downstream call, log line, and trace span.

Without correlation IDs, debugging a failure across 10 microservices requires matching timestamps manually across log streams. With correlation IDs, a single query in Loki or Splunk returns every log line for the entire request.

```python
import uuid

# At the entry point (FastAPI middleware example)
@app.middleware("http")
async def correlation_id_middleware(request: Request, call_next):
    correlation_id = request.headers.get("X-Correlation-ID", str(uuid.uuid4()))
    # Store in context variable so all downstream code can access it
    token = correlation_id_var.set(correlation_id)
    response = await call_next(request)
    response.headers["X-Correlation-ID"] = correlation_id
    correlation_id_var.reset(token)
    return response
```

Include `correlation_id` in every log line, every outbound HTTP header, and every trace span.

## When to Instrument

Instrument at every:
- **Service entry point**: HTTP request received, message consumed, job started.
- **External dependency call**: database query, HTTP call to another service, cache access.
- **Significant business event**: user registered, payment processed, order shipped.
- **Error or recovery**: exception caught, retry attempted, circuit breaker opened.

## Sub-Pages

| Page | What It Covers |
|------|----------------|
| wiki/tier3-working/observability/structured-logging.md | Log levels, structured JSON logs, structlog, correlation IDs, what to log |
| wiki/tier3-working/observability/metrics.md | Counter/Gauge/Histogram, label design, Four Golden Signals, prometheus_client |
| wiki/tier3-working/observability/slo-sli-sla.md | SLA/SLO/SLI definitions, error budgets, alert design |

## See Also

- wiki/tier3-working/observability/structured-logging.md
- wiki/tier3-working/observability/metrics.md
- wiki/tier3-working/observability/slo-sli-sla.md
- wiki/tier1-sources/swebok-v4/ka06-operations.md
- wiki/tier1-sources/owasp/a09-logging-monitoring.md

## Source

"Site Reliability Engineering" (Google, O'Reilly 2016). OpenTelemetry documentation (opentelemetry.io). "Observability Engineering" (Majors, Fong-Jones & Miranda, O'Reilly 2022). OWASP A09:2021 — Security Logging and Monitoring Failures.
