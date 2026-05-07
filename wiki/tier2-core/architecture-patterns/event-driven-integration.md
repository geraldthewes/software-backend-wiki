# Event-Driven Architecture: External Integration (Tier 2)

> **Tier 2** | Source: "Architecture Patterns with Python" Ch 11 (Percival & Gregory) | Authority: established | Derives From: SWEBOK KA02, EDA patterns, microservice integration

## Summary

In-process domain events (Ch 8–9) extend naturally to external integration: events are published to a message broker (Redis Streams, Kafka, RabbitMQ) so other services can react. Chapter 11 covers this extension and warns against the two most common distributed architecture traps: the **Distributed Ball of Mud** (tightly coupled microservices calling each other's HTTP endpoints) and **"thinking in nouns"** (designing CRUD APIs that force callers to know your data model).

## Key Concepts

### External Event Publisher

```python
# adapters/redis_eventpublisher.py

import json
import redis

r = redis.Redis(**config.get_redis_host_and_port())


def publish(channel: str, event: events.Event) -> None:
    r.publish(channel, json.dumps(asdict(event)))
```

Register as an event handler in the message bus:

```python
HANDLERS = {
    events.Allocated: [
        handlers.publish_allocated_event,
    ],
}

def publish_allocated_event(event: events.Allocated, uow: AbstractUnitOfWork) -> None:
    redis_eventpublisher.publish("line_allocated", event)
```

### External Event Consumer

```python
# entrypoints/redis_eventconsumer.py

import json
import redis
from service_layer import messagebus, unit_of_work
from domain import commands

r = redis.Redis(**config.get_redis_host_and_port())


def main():
    pubsub = r.pubsub(ignore_subscribe_messages=True)
    pubsub.subscribe("change_batch_quantity")
    for m in pubsub.listen():
        handle_change_batch_quantity(m)


def handle_change_batch_quantity(m):
    data = json.loads(m["data"])
    cmd = commands.ChangeBatchQuantity(ref=data["batchref"], qty=data["qty"])
    messagebus.handle(cmd, uow=unit_of_work.SqlAlchemyUnitOfWork())
```

The consumer translates an external JSON event into an internal command and passes it through the same message bus as HTTP-triggered commands — uniform handling.

### Event Choreography vs Orchestration

| Approach | Description | Trade-offs |
|----------|-------------|-----------|
| **Choreography** | Services publish events; other services subscribe and react independently | Loose coupling; harder to trace end-to-end workflow |
| **Orchestration** | A central orchestrator calls each service in sequence and tracks workflow state | Easier to trace; orchestrator becomes a bottleneck and single point of coupling |

The book strongly prefers **choreography**: publish a fact, let subscribers decide what to do. This avoids the tight temporal and behavioral coupling of orchestration.

### "Thinking in Nouns" Antipattern

Designing microservices around CRUD resources (`POST /orders`, `PUT /batches`) forces the caller to know your internal data model and the sequence of operations. It recreates a distributed monolith:

```
# CRUD approach — caller knows too much:
POST /orders           → creates order
GET  /batches?sku=X    → finds available batch
PUT  /batches/{id}/allocate → allocates

# Event-driven approach — caller publishes a fact:
PUBLISH order_placed { orderid, sku, qty }
# → allocation service reacts and publishes line_allocated
```

"Nouns" == resources. Prefer events (verbs/facts) for integration.

### The Distributed Ball of Mud

A set of microservices that all call each other's HTTP APIs in real-time — with synchronous chains like A → B → C → D → respond to A — is a distributed monolith. Any service going down cascades. Latency accumulates multiplicatively. Deployments must coordinate across teams.

Indicators:
- Service A can't deploy without coordinating with B because B's response shape changed.
- A request to A triggers synchronous HTTP calls to 3+ other services.
- Integration tests require starting all services simultaneously.

Remedy: publish events, consume asynchronously, tolerate eventual consistency where business rules permit.

### Idempotency and At-Least-Once Delivery

External message brokers typically provide at-least-once delivery: a message may be delivered multiple times. Event handlers must be **idempotent**: processing the same event twice should produce the same outcome as processing it once.

```python
def allocate(event: AllocationRequired, uow: AbstractUnitOfWork) -> str:
    with uow:
        product = uow.products.get(event.sku)
        # If already allocated, allocate() is a no-op (OrderLine is in a set)
        batchref = product.allocate(OrderLine(event.orderid, event.sku, event.qty))
        uow.commit()
    return batchref
```

The domain model's use of `set[OrderLine]` makes `allocate()` naturally idempotent for duplicate lines.

## Agent Guidance

### Do

- Prefer event choreography over orchestration for asynchronous integration between services.
- Make event handlers idempotent — design for at-least-once delivery.
- Publish events via a thin adapter (`redis_eventpublisher.py`); the message bus handler calls the adapter.
- Translate incoming external events into internal commands at the boundary (consumer entrypoint); use the same internal message bus.

### Do Not

- Do not design inter-service communication as synchronous HTTP chains — that is orchestration in disguise.
- Do not expose internal ORM schemas or domain model details in the events published externally; use dedicated integration events with explicit contracts.
- Do not assume exactly-once delivery from any message broker; design for at-least-once.
- Do not mix business logic into the consumer entrypoint — translate to command/event and delegate to the bus.

## Checklist

- [ ] Integration event published via adapter, not directly from domain or service layer
- [ ] Consumer translates external events to internal commands at the boundary
- [ ] Event handlers are idempotent
- [ ] Integration events have an explicit contract (not just ORM serialization)
- [ ] No synchronous HTTP call chains between services — design reviewed against distributed-systems fallacies

## See Also

- wiki/tier2-core/architecture-patterns/commands-vs-events.md
- wiki/tier2-core/architecture-patterns/domain-events-message-bus.md
- wiki/tier2-core/architecture-patterns/cqrs.md
- wiki/tier2-core/distributed-systems/fallacies.md
- wiki/tier2-core/distributed-systems/resilience-patterns.md

## Source

Harry Percival and Bob Gregory, *Architecture Patterns with Python*, Chapter 11 (O'Reilly 2020). CC-BY-ND.
https://www.cosmicpython.com/book/chapter_11_external_events.html
