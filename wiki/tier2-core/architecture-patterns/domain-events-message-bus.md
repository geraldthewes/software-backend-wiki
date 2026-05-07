# Domain Events and Message Bus (Tier 2)

> **Tier 2** | Source: "Architecture Patterns with Python" Ch 8–9 (Percival & Gregory) | Authority: established | Derives From: SWEBOK KA03, DDD Event Storming, EDA patterns

## Summary

Domain events represent something significant that happened in the domain — "order allocated", "stock depleted", "batch quantity changed". Aggregates emit them as side-effects of business decisions. An in-process **message bus** collects events after each transaction and dispatches them to registered handlers.

Chapter 8 introduces events as a way to decouple handlers (e.g., "send an email when stock runs out") from the core allocation logic. Chapter 9 promotes the message bus to the **primary entrypoint** for the service layer: every incoming operation — command or event — flows through the bus, and every service function becomes a handler. This inversion yields a uniform, introspectable dispatch mechanism that extends naturally to external message brokers.

## Key Concepts

### Domain Events as Dataclasses

```python
# domain/events.py

from dataclasses import dataclass


class Event:
    """Base class for all domain events."""


@dataclass
class OutOfStock(Event):
    sku: str


@dataclass
class AllocationRequired(Event):
    orderid: str
    sku: str
    qty: int


@dataclass
class BatchCreated(Event):
    ref: str
    sku: str
    qty: int
    eta: Optional[date] = None


@dataclass
class BatchQuantityChanged(Event):
    ref: str
    qty: int
```

Events are value objects: immutable, named in the past tense, carrying only the data needed by downstream handlers.

### Aggregates Emit Events

```python
class Product:
    def __init__(self, sku: str, batches: list[Batch] | None = None) -> None:
        ...
        self.events: list[Event] = []

    def allocate(self, line: OrderLine) -> str | None:
        try:
            batch = next(b for b in sorted(self.batches) if b.can_allocate(line))
        except StopIteration:
            self.events.append(OutOfStock(line.sku))
            return None
        batch.allocate(line)
        return batch.reference
```

The aggregate appends to `self.events`; it does not call handlers directly. This keeps the domain model free of knowledge about what happens next.

### Simple Message Bus (Ch 8)

```python
# service_layer/messagebus.py

from typing import Callable
from domain import events

HANDLERS: dict[type[events.Event], list[Callable]] = {
    events.OutOfStock: [handlers.send_out_of_stock_notification],
    events.AllocationRequired: [handlers.allocate],
}


def handle(event: events.Event, uow: AbstractUnitOfWork) -> None:
    for handler in HANDLERS[type(event)]:
        handler(event, uow)
```

The service layer calls `messagebus.handle(event, uow)` for each event collected from the UoW after commit.

### Message Bus as Primary Entrypoint (Ch 9)

In Chapter 9 the message bus becomes the entry point for all operations — the Flask endpoint no longer calls `services.allocate()` directly; it creates a `Command` or `Event` and calls `messagebus.handle()`:

```python
# service_layer/messagebus.py (Ch 9 version)

Message = Union[commands.Command, events.Event]

HANDLERS: dict[type[Message], list[Callable]] = {
    commands.Allocate:            [handlers.allocate],
    commands.CreateBatch:         [handlers.add_batch],
    commands.ChangeBatchQuantity: [handlers.change_batch_quantity],
    events.OutOfStock:            [handlers.publish_allocated_event],
    events.AllocationRequired:    [handlers.allocate],
    events.BatchQuantityChanged:  [handlers.change_batch_quantity],
}


def handle(message: Message, uow: AbstractUnitOfWork) -> list:
    results = []
    queue = [message]
    while queue:
        message = queue.pop(0)
        for handler in HANDLERS[type(message)]:
            results.append(handler(message, uow))
            queue.extend(uow.collect_new_events())
    return results
```

The bus processes the initial message, then drains events emitted by handlers into the queue and processes them too — cascading dispatch without recursive calls.

### Handler Functions

```python
# service_layer/handlers.py

def allocate(
    event: events.AllocationRequired | commands.Allocate,
    uow: AbstractUnitOfWork,
) -> str:
    with uow:
        product = uow.products.get(event.sku)
        if product is None:
            raise InvalidSku(f"Invalid sku {event.sku}")
        return product.allocate(model.OrderLine(event.orderid, event.sku, event.qty))


def send_out_of_stock_notification(
    event: events.OutOfStock,
    uow: AbstractUnitOfWork,
) -> None:
    email.send(
        "stock@made.com",
        f"Out of stock for {event.sku}",
    )
```

Handlers are thin: they receive the event or command, interact with the UoW, and return. They do not contain business logic — that lives in the domain.

### UoW and Event Collection

```python
class AbstractUnitOfWork(abc.ABC):
    def collect_new_events(self) -> Iterator[events.Event]:
        for product in self.products.seen:
            while product.events:
                yield product.events.pop(0)
```

The bus drains events after each handler call, not only after the top-level commit.

## Agent Guidance

### Do

- Name events in the past tense: `OutOfStock`, `BatchCreated`, `AllocationRequired`.
- Emit events from aggregate methods where the business decision is made, not from service layer functions.
- Process events from the queue — never call handlers directly from within other handlers.
- Keep event dataclasses in `domain/events.py`; keep handler functions in `service_layer/handlers.py`.

### Do Not

- Do not put side effects (email, Redis publish) in domain model methods — only emit events.
- Do not share a `Session` or UoW across event handler calls; each handler call gets the same UoW within one message-handling cycle.
- Do not raise exceptions from event handlers — exceptions in event handlers should be logged and the event skipped, not propagated to the caller (contrast with commands — see `commands-vs-events.md`).
- Do not use the message bus for pure reads — only writes and side-effectful operations need event dispatch.

## Checklist

- [ ] Events defined as `@dataclass` in `domain/events.py`; named in past tense
- [ ] Aggregates append to `self.events` — no direct handler calls from domain methods
- [ ] `HANDLERS` mapping is the single source of truth for which handlers respond to which events
- [ ] Bus drains `uow.collect_new_events()` after each handler call (cascading dispatch)
- [ ] Event handlers are idempotent (safe to run more than once)

## See Also

- wiki/tier2-core/architecture-patterns/commands-vs-events.md
- wiki/tier2-core/architecture-patterns/aggregates.md
- wiki/tier2-core/architecture-patterns/event-driven-integration.md
- wiki/tier2-core/architecture-patterns/unit-of-work.md
- wiki/tier2-core/architecture-patterns/dependency-injection-bootstrap.md

## Source

Harry Percival and Bob Gregory, *Architecture Patterns with Python*, Chapters 8–9 (O'Reilly 2020). CC-BY-ND.
https://www.cosmicpython.com/book/chapter_08_events_and_message_bus.html
https://www.cosmicpython.com/book/chapter_09_all_messagebus.html
