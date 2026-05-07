# Commands vs Events (Tier 2)

> **Tier 2** | Source: "Architecture Patterns with Python" Ch 10 (Percival & Gregory) | Authority: established | Derives From: SWEBOK KA03, CQRS/ES pattern language (Greg Young)

## Summary

Commands and Events are both messages, but they express different semantics and carry different routing rules. Mixing them — treating every message the same — leads to subtle bugs around error handling, retry semantics, and intent. The distinction becomes essential when the message bus (Ch 9) dispatches both kinds.

## Key Concepts

### The Fundamental Difference

| Dimension | Command | Event |
|-----------|---------|-------|
| **Grammar** | Imperative: "Do X" | Past tense: "X happened" |
| **Intent** | Request an action | Announce a fact |
| **Handler count** | Exactly one | Zero or more |
| **Failure semantics** | Raises an exception; caller must handle | Handler failure is logged; bus continues |
| **Sender knows recipient?** | Yes (implicitly) | No — fires and forgets |
| **Examples** | `Allocate`, `CreateBatch` | `OutOfStock`, `AllocationRequired`, `BatchCreated` |

### Command Definitions

```python
# domain/commands.py

from dataclasses import dataclass
from datetime import date
from typing import Optional


class Command:
    """Base class for all commands."""


@dataclass
class Allocate(Command):
    orderid: str
    sku: str
    qty: int


@dataclass
class CreateBatch(Command):
    ref: str
    sku: str
    qty: int
    eta: Optional[date] = None


@dataclass
class ChangeBatchQuantity(Command):
    ref: str
    qty: int
```

Commands are named in the imperative; they carry the inputs the handler needs to perform the operation.

### Routing Rules in the Message Bus

```python
# service_layer/messagebus.py

Message = Union[commands.Command, events.Event]


def handle(message: Message, uow: AbstractUnitOfWork) -> list:
    results = []
    queue = [message]
    while queue:
        message = queue.pop(0)
        if isinstance(message, events.Event):
            _handle_event(message, queue, uow, results)
        elif isinstance(message, commands.Command):
            _handle_command(message, queue, uow, results)
        else:
            raise Exception(f"Unknown message type {type(message)}")
    return results


def _handle_event(event, queue, uow, results):
    for handler in HANDLERS[type(event)]:
        try:
            handler(event, uow)
            queue.extend(uow.collect_new_events())
        except Exception:
            logger.exception("Exception handling event %s", event)
            continue     # log and continue; don't propagate


def _handle_command(command, queue, uow, results):
    try:
        handler = HANDLERS[type(command)]     # exactly one handler
        results.append(handler(command, uow))
        queue.extend(uow.collect_new_events())
    except Exception:
        logger.exception("Exception handling command %s", command)
        raise            # propagate; caller must decide what to do
```

A command failure propagates — the caller (e.g., Flask) catches it and returns an HTTP 4xx/5xx. An event handler failure is logged and skipped — the bus continues processing remaining events.

### Why the Distinction Matters

**Commands are directed**: `Allocate` is sent to exactly one handler that performs the allocation. If allocation fails, the caller must know — raising the exception is correct.

**Events are broadcast**: `OutOfStock` might trigger an email, update a dashboard, and publish to an external message broker. If the email handler fails, the dashboard update and the broker publish should still happen.

**Retry semantics differ**: A failed command should not be automatically retried without the caller's knowledge. A failed event handler can be retried independently (and should be idempotent).

### Flask Endpoint Using Commands

```python
@app.route("/allocate", methods=["POST"])
def allocate_endpoint():
    try:
        cmd = commands.Allocate(
            orderid=request.json["orderid"],
            sku=request.json["sku"],
            qty=request.json["qty"],
        )
        results = messagebus.handle(cmd, SqlAlchemyUnitOfWork())
        batchref = results.pop(0)
    except InvalidSku as e:
        return jsonify({"message": str(e)}), 400
    return jsonify({"batchref": batchref}), 201
```

Flask creates a Command, passes it to the bus, and reads the result. The service layer is no longer called directly.

## Agent Guidance

### Do

- Use a `Command` when the sender expects a result and the operation must either fully succeed or raise.
- Use an `Event` when announcing a fact that multiple interested parties might react to independently.
- Propagate command handler exceptions; catch and log event handler exceptions.
- Keep handlers idempotent — especially event handlers, which may be retried.

### Do Not

- Do not raise exceptions from event handlers when they should be resilient; log and continue.
- Do not register multiple handlers for a single command — that violates the single-handler rule.
- Do not use events as a substitute for return values from commands; the bus returns the command handler's result.
- Do not put multiple business operations in one command handler; each command = one unit of work.

## Checklist

- [ ] Commands named in imperative; events named in past tense
- [ ] Command handlers: exactly one per command type, propagate exceptions
- [ ] Event handlers: zero or more per event type, catch and log exceptions
- [ ] All commands and events are dataclasses in `domain/commands.py` and `domain/events.py`
- [ ] Event handlers are idempotent

## See Also

- wiki/tier2-core/architecture-patterns/domain-events-message-bus.md
- wiki/tier2-core/architecture-patterns/event-driven-integration.md
- wiki/tier2-core/architecture-patterns/cqrs.md
- wiki/tier2-core/architecture-patterns/service-layer.md
- wiki/tier1-sources/swebok-v4/ka03-design.md

## Source

Harry Percival and Bob Gregory, *Architecture Patterns with Python*, Chapter 10 (O'Reilly 2020). CC-BY-ND.
https://www.cosmicpython.com/book/chapter_10_commands.html
Greg Young — CQRS and Event Sourcing (dddcqrs.com).
