# SLO / SLI / SLA

> **Tier 3** | Source: Google SRE Book | Enforces/Derives From: wiki/tier1-sources/swebok-v4/ka06-operations.md, wiki/tier2-core/twelve-factor-app/factors.md

## Summary

SLAs, SLOs, and SLIs form a layered reliability framework. SLIs measure reality; SLOs set targets; SLAs make commitments. Error budgets translate reliability targets into engineering decisions about risk.

## Definitions

### SLA — Service Level Agreement

A contractual commitment between provider and customer. Breach has defined consequences — refunds, credits, contract termination.

- Set by: business and legal teams.
- Stricter than your internal targets? No — SLA must be achievable.
- Example: "99.9% availability per calendar month; breach triggers 25% credit."

### SLO — Service Level Objective

An internal reliability target, always stricter than the SLA. The SLO is what the engineering team aims for. When the SLO is met, the SLA is safe.

- Set by: engineering and product teams.
- More aggressive than SLA — gives a buffer before the SLA is breached.
- Example: internal target of 99.95% availability, SLA promises 99.9%.

### SLI — Service Level Indicator

The measurement — a ratio of good events to total events observed over a time window.

```
SLI = (good events) / (total events)
```

- Set by: engineering — defined from available metrics.
- Example: `SLI = (requests with status < 500) / (total requests)`

## Error Budget

```
Error Budget = (1 - SLO) × time period

Example for 99.9% monthly SLO:
Error Budget = (1 - 0.999) × 30 days = 0.001 × 43,200 minutes = 43.2 minutes/month
```

The error budget is the amount of unreliability you are allowed before breaching the SLO. It governs engineering decisions:

- **Budget remaining > 50%**: deploy freely; experiment; iterate fast.
- **Budget remaining < 20%**: slow deployments; prioritize reliability work.
- **Budget exhausted**: freeze all non-reliability changes until budget is replenished.

The error budget makes reliability a shared concern between product and engineering: shipping a risky feature spends budget that could otherwise be spent on failures.

## Common SLIs and SLOs

### Availability

```
SLI = successful_requests / total_requests
      where successful = status code < 500

SLO examples:
  99.0%  = 87.6 hours downtime/year  (4.38 hours/month)
  99.9%  = 8.76 hours downtime/year  (43.8 min/month)
  99.95% = 4.38 hours downtime/year  (21.9 min/month)
  99.99% = 52.6 minutes downtime/year (4.38 min/month)
```

### Latency

```
SLI = (requests_completed_within_threshold) / (total_requests)

SLO example:
  99% of requests complete in < 200ms
  95% of requests complete in < 100ms
```

### Throughput

```
SLI = (actual_rps) / (target_rps)

SLO example:
  System handles target load 99% of the time
```

## Designing SLOs

1. **Start with the user journey**: what does "working" mean to the user? A user does not care that the database query returned in 5ms if the full page load took 8 seconds.

2. **Measure at the user-facing boundary**: measure the SLI where the user experiences the service — at the load balancer or API gateway, not inside internal microservices.

3. **Don't set SLOs you cannot measure**: if you have no latency histogram, you cannot have a latency SLO. Instrument first.

4. **Start achievable, tighten incrementally**: begin at your current measured reliability minus a buffer. Tighten as reliability improves.

5. **Fewer, more meaningful SLOs**: 3–5 well-chosen SLOs are more useful than 20 that nobody monitors.

## Alert Design — Burn Rate Alerting

Alerting when the SLO threshold is breached is too late — by then the error budget is already gone. Alert on burn rate: how fast the error budget is being consumed.

```
Burn rate = (error rate observed) / (error rate that exactly meets SLO)

Example for 99.9% SLO (0.1% allowed error rate):
  If observed error rate = 1%
  Burn rate = 1% / 0.1% = 10x
  Interpretation: consuming monthly budget 10x faster than allowed
```

### Recommended Alert Thresholds (Google SRE)

| Severity | Window | Burn Rate | Budget Consumed |
|----------|--------|-----------|----------------|
| Page (critical) | 1 hour | 14.4x | ~2% in 1 hour (exhausts budget in 2 days) |
| Page (high) | 6 hours | 6x | ~5% in 6 hours |
| Ticket (medium) | 3 days | 1x | Budget exhausted exactly on schedule |

```promql
# Burn rate over 1-hour window for availability SLO
(
  rate(http_requests_total{status_code=~"5.."}[1h])
  /
  rate(http_requests_total[1h])
) / 0.001 > 14.4
```

## Python: SLI Calculation from Prometheus Metrics

```python
# Record request outcomes
from prometheus_client import Counter, Histogram

request_count = Counter(
    "http_requests_total",
    "Total requests",
    ["status_code"],
)

request_duration = Histogram(
    "http_request_duration_seconds",
    "Request latency",
    buckets=[0.1, 0.2, 0.5, 1.0, 2.0, 5.0],
)

# SLI availability query in PromQL:
# sum(rate(http_requests_total{status_code!~"5.."}[5m]))
# /
# sum(rate(http_requests_total[5m]))

# SLI latency query in PromQL (% under 200ms):
# sum(rate(http_request_duration_seconds_bucket{le="0.2"}[5m]))
# /
# sum(rate(http_request_duration_seconds_count[5m]))
```

## Alertmanager Rule Example

```yaml
# prometheus/rules/slo_alerts.yaml
groups:
  - name: slo_burn_rate
    rules:
      - alert: HighErrorBurnRate
        expr: |
          (
            rate(http_requests_total{status_code=~"5.."}[1h])
            /
            rate(http_requests_total[1h])
          ) / 0.001 > 14.4
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Error budget burning at {{ $value }}x rate"
          description: "At current burn rate, monthly error budget will be exhausted in {{ printf \"%.1f\" (720 / $value) }} hours"
```

## See Also

- wiki/tier3-working/observability/metrics.md
- wiki/tier1-sources/swebok-v4/ka06-operations.md
- wiki/tier2-core/twelve-factor-app/factors.md

## Source

"Site Reliability Engineering" (Google, O'Reilly 2016) — Chapter 4: Service Level Objectives. "The SRE Workbook" (Google, O'Reilly 2018) — Chapter 5: Alerting on SLOs. Prometheus alerting documentation.
