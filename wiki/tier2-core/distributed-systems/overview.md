# Distributed Systems — Overview

> **Tier 2** | Source: Multiple (Brewer, Deutsch, Abadi) | Derives From: ka02-architecture | Authority: established practice

## Summary

A distributed system is a collection of independent computers that appears to its users as a single coherent system. The defining characteristic is that components communicate exclusively by passing messages over a network. This fundamentally changes the failure model: in a single-process application, a function call either succeeds or raises an exception; in a distributed system, the call may succeed, fail, or — critically — produce an unknown result (the message was sent but the response was lost).

Distributed systems introduce challenges that do not exist in monolithic applications. Agents must understand these challenges before designing or modifying any service that communicates over a network.

## Key Challenges

| Challenge | Description | Sub-Page |
|-----------|-------------|----------|
| **Coordination** | Multiple nodes must agree on shared state without a shared clock or memory | `cap-pacelc.md` |
| **Consistency** | Different nodes may see different versions of the same data | `cap-pacelc.md` |
| **Availability** | The system should respond even when some nodes are unavailable | `cap-pacelc.md` |
| **False assumptions** | Developers assume network properties that are not true | `fallacies.md` |
| **Partial failure** | A service can be partially available — some requests succeed, some fail | `resilience-patterns.md` |
| **Cascading failure** | One slow service causes all callers to queue and eventually crash | `resilience-patterns.md` |

## Sub-Pages

- **`cap-pacelc.md`**: CAP Theorem and PACELC extension — how to choose between consistency and availability when picking a data store
- **`fallacies.md`**: The 8 Fallacies of Distributed Computing — false assumptions that cause production failures
- **`resilience-patterns.md`**: Retry, circuit breaker, timeout, bulkhead, idempotency, and health checks — the standard toolbox for building resilient distributed services

## Agent Guidance

### Do
- Read `fallacies.md` before writing any service that makes a network call
- Apply `resilience-patterns.md` patterns on every external call: retry, timeout, circuit breaker
- Use `cap-pacelc.md` to justify data store selection

### Do Not
- Do not assume any network call will succeed on the first attempt
- Do not make network calls without explicit timeouts
- Do not treat a local development environment (localhost) as representative of distributed behavior

## Checklist
- [ ] All network calls have explicit timeouts
- [ ] All network calls to non-idempotent endpoints are protected from duplicate execution
- [ ] Data store selection is justified by CAP/PACELC analysis
- [ ] Retry logic uses exponential backoff with jitter
- [ ] Circuit breaker is in place for high-traffic downstream dependencies

## See Also
- `wiki/tier2-core/distributed-systems/cap-pacelc.md`
- `wiki/tier2-core/distributed-systems/fallacies.md`
- `wiki/tier2-core/distributed-systems/resilience-patterns.md`
- `wiki/tier2-core/twelve-factor-app/factors.md`
- `wiki/tier1-sources/swebok-v4/ka02-architecture.md`

## Source

Synthesized from CAP Theorem (Brewer 2000), PACELC (Abadi 2012), Fallacies of Distributed Computing (Deutsch 1994), and *Software Development Best Practices for Agent* reference document.
