# Dependency Injection and Bootstrap (Tier 2)

> **Tier 2** | Source: "Architecture Patterns with Python" Ch 13 (Percival & Gregory) | Authority: established | Derives From: SWEBOK KA03, SOLID DIP, Composition Root pattern

## Summary

By Chapter 13, the service layer depends on abstract interfaces for the unit of work, message bus, and notification adapters. The final piece is a **composition root** — a single location where all concrete dependencies are wired together. In Python, this is `bootstrap.py`.

This chapter also surveys three DI styles in Python (explicit constructor injection, closures/partial application, and `@inject`-style decorators) and argues that explicit injection — passing dependencies as parameters — is the most debuggable and testable approach for most services.

## Key Concepts

### The Composition Root

Everything that was previously scattered — creating the session factory, instantiating repositories, building the handler map — moves into one place:

```python
# bootstrap.py

from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

from adapters import orm, redis_eventpublisher
from service_layer import handlers, messagebus, unit_of_work


def bootstrap(
    start_orm: bool = True,
    uow: unit_of_work.AbstractUnitOfWork = None,
    send_mail_func=None,
    publish_func=None,
) -> messagebus.MessageBus:
    if start_orm:
        orm.start_mappers()

    if send_mail_func is None:
        send_mail_func = email.send
    if publish_func is None:
        publish_func = redis_eventpublisher.publish
    if uow is None:
        uow = unit_of_work.SqlAlchemyUnitOfWork()

    dependencies = {
        "uow": uow,
        "send_mail": send_mail_func,
        "publish": publish_func,
    }
    injected_handlers = inject_dependencies(handlers.HANDLERS, dependencies)
    return messagebus.MessageBus(uow=uow, event_handlers=injected_handlers)
```

`bootstrap()` is called once at application startup and returns a fully-wired `MessageBus`. Tests call it with fake dependencies:

```python
bus = bootstrap(
    start_orm=False,
    uow=FakeUnitOfWork(),
    send_mail_func=lambda *args: None,
    publish_func=lambda *args: None,
)
```

### Explicit Dependency Injection (Preferred)

Inject dependencies as function parameters. The function signature is the explicit contract:

```python
# Handlers receive dependencies as parameters
def send_out_of_stock_notification(
    event: events.OutOfStock,
    send_mail: Callable,
) -> None:
    send_mail(
        "stock@made.com",
        f"Out of stock for {event.sku}",
    )
```

To wire this into the bus, use `functools.partial` or a factory that closes over the dependency:

```python
def inject_dependencies(handlers, dependencies):
    params = inspect.signature(handler).parameters
    deps = {name: dep for name, dep in dependencies.items() if name in params}
    return functools.partial(handler, **deps)
```

### Three Python DI Styles Compared

| Style | Example | Verdict |
|-------|---------|---------|
| **Explicit constructor injection** | `def __init__(self, repo: AbstractRepo)` | Best: visible, testable, no magic |
| **Closures / partial application** | `partial(send_notification, send_mail=real_send)` | Good for functions; clean and composable |
| **`@inject` decorator** | `@inject; def handler(event, uow: UoW)` | Convenient but uses reflection magic; harder to debug |
| **Global / module-level** | `import send_mail` inside handler | Avoid: hard to test, hidden coupling |
| **`unittest.mock.patch`** | Patches a module global at test time | Last resort: hides the coupling instead of fixing it |

### The Message Bus Class

After Chapter 9's functional bus, Chapter 13 encapsulates it as a class that holds its own dependencies:

```python
# service_layer/messagebus.py

class MessageBus:
    def __init__(
        self,
        uow: AbstractUnitOfWork,
        event_handlers: dict[type[events.Event], list[Callable]],
        command_handlers: dict[type[commands.Command], Callable],
    ) -> None:
        self.uow = uow
        self.event_handlers = event_handlers
        self.command_handlers = command_handlers

    def handle(self, message: Message) -> list:
        results = []
        queue = [message]
        while queue:
            message = queue.pop(0)
            if isinstance(message, events.Event):
                self._handle_event(message, queue, results)
            elif isinstance(message, commands.Command):
                self._handle_command(message, queue, results)
        return results
```

### Flask and the Bootstrap

```python
# entrypoints/flask_app.py

from bootstrap import bootstrap

bus = bootstrap()
app = Flask(__name__)


@app.route("/allocate", methods=["POST"])
def allocate_endpoint():
    try:
        cmd = commands.Allocate(**request.json)
        results = bus.handle(cmd)
        batchref = results.pop(0)
    except InvalidSku as e:
        return jsonify({"message": str(e)}), 400
    return jsonify({"batchref": batchref}), 201
```

`bus` is created once at module load; Flask endpoints simply call `bus.handle(cmd)`.

## Agent Guidance

### Do

- Locate all concrete dependency creation in `bootstrap.py`; this is the only file that imports both domain and infrastructure.
- Use explicit parameter injection (or `functools.partial`) rather than `unittest.mock.patch` for test setup.
- Call `bootstrap()` with fake dependencies in integration and unit tests — no patching needed.
- Call `orm.start_mappers()` exactly once in `bootstrap()`, guarding with `start_orm` flag.

### Do Not

- Do not create `Session`, `Engine`, or infrastructure objects inside domain or service layer functions.
- Do not use global variables to hold injected dependencies — they are invisible in signatures and prevent parallel test runs.
- Do not scatter dependency wiring across multiple modules — one composition root, one call to `bootstrap()`.
- Do not use an IoC container framework unless the dependency graph is large enough to justify the indirection.

## Checklist

- [ ] `bootstrap.py` is the single file that imports both domain and infrastructure
- [ ] `bootstrap()` accepts fake dependencies for testing (`uow=FakeUnitOfWork()` etc.)
- [ ] Handler functions take dependencies as explicit parameters, not module globals
- [ ] `orm.start_mappers()` called exactly once, guarded against re-invocation
- [ ] No `unittest.mock.patch` in tests — injected fakes instead

## See Also

- wiki/tier2-core/solid-principles/dip.md
- wiki/tier3-working/worked-examples/dependency-injection.md
- wiki/tier2-core/architecture-patterns/domain-events-message-bus.md
- wiki/tier2-core/architecture-patterns/ports-and-adapters.md
- wiki/tier2-core/architecture-patterns/service-layer.md

## Source

Harry Percival and Bob Gregory, *Architecture Patterns with Python*, Chapter 13 (O'Reilly 2020). CC-BY-ND.
https://www.cosmicpython.com/book/chapter_13_dependency_injection.html
Mark Seemann and Steven van Deursen, *Dependency Injection: Principles, Practices, and Patterns* (2019) — Composition Root pattern.
