# The 8 Fallacies of Distributed Computing

> **Tier 2** | Source: Peter Deutsch, Sun Microsystems (1994) | Derives From: ka02-architecture | Authority: established practice

## Summary

The 8 Fallacies of Distributed Computing are false assumptions that programmers new to distributed systems commonly make. First documented by Peter Deutsch at Sun Microsystems in 1994 and later extended by James Gosling, each fallacy leads to a specific class of production failure. Understanding and actively coding against these fallacies is a prerequisite for building reliable distributed services.

---

## Fallacy 1: "The Network Is Reliable"

### The False Assumption
Network calls always succeed. If I send a request, I will receive a response.

### Why It Is Wrong
Networks drop packets. Routers reboot. Cables are cut. NIC firmware crashes. Cloud providers have partial outages affecting specific availability zones. A TCP connection established successfully provides no guarantee that future packets on that connection will arrive. In microservice architectures, every inter-service call is an opportunity for a transient network failure.

### Agent Remediation
- Implement **retry with exponential backoff and jitter** on all network calls (see `resilience-patterns.md`)
- Make mutations **idempotent**: a retried operation should produce the same result as the original
- Implement **circuit breakers** to stop retrying when a downstream service is consistently failing
- Log all transient failures — even when retries succeed — for operational visibility

---

## Fallacy 2: "Latency Is Zero"

### The False Assumption
Network calls are essentially instantaneous. I can treat them like in-process function calls.

### Why It Is Wrong
LAN latency is 1–10 ms; WAN latency is 50–200 ms; cross-region is 100–400 ms. A page that makes 20 sequential API calls with 50 ms each takes at least 1 second — before any computation. N+1 query patterns (fetching a list, then making one call per item) routinely cause 10–100x latency penalties in distributed systems.

### Agent Remediation
- **Batch requests**: fetch many items in one call rather than one item per call
- **Parallel I/O**: use `asyncio.gather()` (Python) or goroutines (Go) to make independent calls concurrently
- **Async communication**: use message queues for work that does not need an immediate response
- Avoid N+1 distributed call patterns — detect them in code review
- Instrument all service calls with latency metrics (P50, P95, P99)

```python
import asyncio, httpx

async def fetch_users(user_ids: list[int]) -> list[dict]:
    async with httpx.AsyncClient() as client:
        tasks = [client.get(f"/users/{uid}") for uid in user_ids]
        responses = await asyncio.gather(*tasks)  # parallel, not sequential
        return [r.json() for r in responses]
```

---

## Fallacy 3: "Bandwidth Is Infinite"

### The False Assumption
I can send as much data as I want over the network without cost or consequence.

### Why It Is Wrong
Bandwidth is finite and shared. Large payloads cause higher latency, increased memory pressure, and — in cloud environments — real financial cost per GB transferred. A service that returns 10,000 records when a client needs 10 is wasting bandwidth and creating unnecessary load on both sides.

### Agent Remediation
- Use **efficient serialization**: prefer Protocol Buffers / gRPC over JSON/REST for high-throughput internal services
- Implement **pagination** on all list endpoints — never return unbounded collections
- **Compress large payloads**: use gzip/brotli for HTTP responses over a certain size threshold
- Select only needed fields — avoid `SELECT *` or returning full objects when a subset suffices

---

## Fallacy 4: "The Network Is Secure"

### The False Assumption
If a service is inside the corporate network or cluster, I can trust requests from it without authentication.

### Why It Is Wrong
Internal networks are regularly compromised. Insider threats, misconfigured firewall rules, compromised containers, and lateral movement after perimeter breach all mean that an attacker can be inside the network making requests that appear internal. Trusting the network perimeter is the assumption exploited in the majority of large-scale data breaches.

### Agent Remediation
- Apply **Zero Trust**: authenticate and authorize every request regardless of origin (see `wiki/tier2-core/security-practices/zero-trust.md`)
- Use **mutual TLS (mTLS)** for service-to-service communication: both parties present certificates
- Use **short-lived JWT tokens** with explicit expiry for API authentication
- Apply **network policies**: deny by default; allow only required service-to-service paths
- Never put credentials in URLs or log them

---

## Fallacy 5: "Topology Doesn't Change"

### The False Assumption
The IP address of `database-server` is always `10.0.1.42`. I can hardcode it.

### Why It Is Wrong
Cloud-native infrastructure is ephemeral. Containers restart with new IPs. Auto-scaling adds and removes instances. Blue/green deployments swap entire fleets. A hardcoded IP is a single point of failure that breaks silently when the topology changes.

### Agent Remediation
- Use **service discovery**: Consul, Kubernetes DNS, or AWS Service Discovery
- Never hardcode IP addresses — always use DNS names or service discovery endpoints
- Use **environment variables** for all service endpoints (Factor III of 12-Factor App)
- Implement **health checks** so that load balancers stop routing to unhealthy instances

```python
# Violation
DB_HOST = "10.0.1.42"

# Correct: resolved at runtime via DNS or service discovery
DB_HOST = os.environ["DATABASE_HOST"]  # consul DNS or k8s service name
```

---

## Fallacy 6: "There Is One Administrator"

### The False Assumption
One person or team controls all services and can coordinate changes atomically.

### Why It Is Wrong
In organizations with multiple teams, services evolve independently. Team A deploys a breaking API change without coordinating with Team B, which depends on it. Different teams have different deployment schedules, different SLAs, and different on-call rotations. Assuming coordination exists leads to undetected breaking changes.

### Agent Remediation
- **Version your APIs**: use URL versioning (`/v1/`, `/v2/`) or request/response versioning
- Maintain **backward compatibility**: new fields are optional; removed fields are deprecated before removal
- Define explicit **SLAs and contracts** for each service: uptime, latency P99, error rate
- Use **contract testing** (e.g., Pact) to detect breaking changes before deployment
- Document the deprecation policy and communicate changes to consumers

---

## Fallacy 7: "Transport Cost Is Zero"

### The False Assumption
Making a network call is free. I can call a remote service as often as I call a local function.

### Why It Is Wrong
Network calls have real CPU cost (TLS handshake, serialization, context switching), real financial cost (cloud egress fees, API pricing), and real time cost. A service that calls a downstream API 1,000 times per second at $0.001/call generates $86/day in API costs alone, and the downstream service experiences load it may not be designed for.

### Agent Remediation
- **Cache aggressively**: cache responses from expensive calls at appropriate TTL
- **Colocate compute and data**: run computation close to the data it needs
- **Measure call frequency**: instrument all external calls; alert on unexpected frequency increases
- Use **batching** where the downstream API supports it

---

## Fallacy 8: "The Network Is Homogeneous"

### The False Assumption
All systems I communicate with use the same operating system, the same encoding, the same network MTU, and the same character set.

### Why It Is Wrong
Real environments include Linux containers, Windows servers, ARM and x86 instances, services using Latin-1 vs. UTF-8, different endianness, and different MTU limits between network segments. A service that works perfectly between two Linux containers may fail when one is replaced with a Windows container or when the network path changes.

### Agent Remediation
- Use **standard protocols**: HTTP/2, gRPC, or JSON over HTTPS; avoid platform-specific serialization (pickle, Java serialization, .NET binary formatter)
- Specify **character encoding explicitly**: UTF-8 everywhere
- Test under **different network conditions**: inject latency, packet loss, and partial failures in integration tests
- Use **content negotiation** in HTTP APIs rather than assuming a specific format

---

## Agent Guidance

### Do
- Internalize all 8 fallacies before designing any service that makes network calls
- Treat every network call as potentially failing, slow, or returning stale data
- Test failure scenarios explicitly: kill a downstream service and verify the caller degrades gracefully

### Do Not
- Do not hardcode IP addresses or hostnames
- Do not make network calls without timeouts
- Do not trust requests because they come from inside the cluster
- Do not assume downstream services will be fast, reliable, or well-coordinated

## Checklist
- [ ] No hardcoded IP addresses or hostnames in source code
- [ ] All network calls have explicit timeouts (connect + read)
- [ ] Retry logic is implemented with exponential backoff and jitter
- [ ] All mutations are idempotent or protected by idempotency keys
- [ ] All list endpoints are paginated
- [ ] mTLS or equivalent authentication is used for service-to-service calls
- [ ] Service discovery is used for all downstream addresses

## See Also
- `wiki/tier2-core/distributed-systems/cap-pacelc.md`
- `wiki/tier2-core/distributed-systems/resilience-patterns.md`
- `wiki/tier2-core/distributed-systems/overview.md`
- `wiki/tier2-core/security-practices/zero-trust.md`
- `wiki/tier2-core/twelve-factor-app/factors.md`
- `wiki/tier1-sources/swebok-v4/ka02-architecture.md`

## Source

Peter Deutsch, "The Eight Fallacies of Distributed Computing" (Sun Microsystems, 1994); extended by James Gosling. Synthesized from *Software Development Best Practices for Agent* reference document.
