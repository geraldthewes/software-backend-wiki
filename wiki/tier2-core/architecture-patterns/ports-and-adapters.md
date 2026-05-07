# Ports and Adapters (Hexagonal Architecture) (Tier 2)

> **Tier 2** | Source: "Architecture Patterns with Python" Preface + Ch 2, 4 (Percival & Gregory) | Authority: established | Derives From: SWEBOK KA02, Alistair Cockburn (2005), Dependency Inversion Principle

## Summary

Ports and Adapters (also called Hexagonal Architecture, Onion Architecture, and Clean Architecture) is an architectural style where business logic lives at the center and all external concerns — databases, HTTP, message brokers, email, the filesystem — are **adapters** that plug in through well-defined **ports** (interfaces/Protocols).

The governing rule is the same as DIP: dependency arrows always point **inward** toward the domain. Infrastructure never determines the domain's design; the domain is oblivious to the infrastructure that surrounds it.

## Key Concepts

### The Three Names Mean the Same Thing

| Name | Author | Key Metaphor |
|------|--------|-------------|
| **Hexagonal Architecture** | Alistair Cockburn (2005) | A hexagon with ports on the sides; any adapter can plug into any port |
| **Onion Architecture** | Jeffrey Palermo (2008) | Concentric rings; each ring depends only on inner rings |
| **Clean Architecture** | Robert C. Martin (2012) | Concentric circles; "dependency rule" — source code dependencies point inward |

These are the same underlying idea. The book uses "Ports and Adapters" as the preferred term.

### Ports and Adapters Defined

**Port**: An abstraction (Python `Protocol`, ABC, or interface) defined by the domain or service layer. It expresses what the domain needs without knowing how it will be provided.

**Adapter**: A concrete implementation of a port that translates between the domain's abstraction and a specific technology (SQLAlchemy, Flask, Redis, SMTP).

```
Application core (domain + service layer)
├── Ports (defined here):
│   ├── AbstractBatchRepository (Protocol)  → inbound: what service layer needs from storage
│   ├── AbstractUnitOfWork (ABC)            → inbound: transaction management
│   └── AbstractNotifications (Protocol)    → outbound: what service layer pushes out
│
Adapters (defined in adapters/):
├── SqlAlchemyBatchRepository    → implements AbstractBatchRepository
├── SqlAlchemyUnitOfWork         → implements AbstractUnitOfWork
├── EmailNotifications           → implements AbstractNotifications
└── FakeNotifications            → implements AbstractNotifications (for tests)

Entrypoints (inbound adapters — drive the application):
├── Flask app                    → HTTP → Commands → message bus
└── Redis consumer               → pub/sub message → Commands → message bus
```

### Inbound vs Outbound Ports/Adapters

**Inbound (driving) adapters** call the application from outside: Flask views, CLI commands, Redis consumers. They translate an external representation (HTTP JSON, CLI args, Redis message) into a command or event and pass it to the message bus.

**Outbound (driven) adapters** are called by the application: database repositories, email senders, external event publishers. The application defines the port (Protocol); the adapter implements it.

```python
# Domain defines the port
class AbstractNotifications(Protocol):
    def send(self, destination: str, message: str) -> None: ...


# Infrastructure provides the adapter
class EmailNotifications:
    def __init__(self, smtp_host: str, port: int = 587) -> None:
        self._server = smtplib.SMTP(smtp_host, port=port)

    def send(self, destination: str, message: str) -> None:
        msg = MIMEText(message)
        msg["Subject"] = "Important notification"
        msg["From"] = "noreply@made.com"
        msg["To"] = destination
        self._server.sendmail("noreply@made.com", [destination], msg.as_string())


# For tests
class FakeNotifications:
    def __init__(self) -> None:
        self.sent: list[tuple[str, str]] = []

    def send(self, destination: str, message: str) -> None:
        self.sent.append((destination, message))
```

### Functional Core, Imperative Shell

Gary Bernhardt's **Functional Core, Imperative Shell** is the implementation-level expression of the same idea (see `wiki/tier3-working/python/functional-core.md`):

- **Functional core** = domain model; pure functions, no I/O, deterministic, fully testable.
- **Imperative shell** = adapters; performs I/O, calls core with values, handles side effects.

### Layering Summary

```
Entrypoints (Flask, Redis consumer, CLI)
     ↓ calls
Service Layer (handlers, message bus, unit of work)
     ↓ calls
Domain Model (aggregates, value objects, domain services, events, commands)
     ↑ implemented by
Adapters (SQLAlchemy repos, email, Redis publisher)
     ↑ provided by
Infrastructure (Postgres, Redis, SMTP server)
```

Every arrow (dependency) points inward. Infrastructure depends on domain; domain never depends on infrastructure.

## Agent Guidance

### Do

- Define all ports (`Protocol`, ABC) in the domain or service layer, never in `adapters/`.
- Place all infrastructure-touching code (SQLAlchemy, Redis, SMTP, HTTP client) in `adapters/`.
- Use `bootstrap.py` as the composition root that wires adapters to ports.
- Confirm that `import` statements in `domain/` contain no third-party library names.

### Do Not

- Do not import `sqlalchemy`, `redis`, `smtplib`, or `requests` from domain or service layer modules.
- Do not let entrypoints (Flask views) contain business logic — they are adapters.
- Do not put port definitions in `adapters/` — the adapter depends on the port, not the reverse.
- Do not use this style as a reason to create unnecessary interfaces; add a port only when you need to swap the adapter (e.g., for testing or for replacing the technology).

## Checklist

- [ ] `domain/` and `service_layer/` import only stdlib and other domain/service modules
- [ ] `adapters/` contains all concrete infrastructure implementations
- [ ] Every outbound dependency used by the service layer is expressed as a `Protocol` or ABC
- [ ] A `Fake*` adapter exists for every port used in unit tests
- [ ] Composition root (`bootstrap.py`) is the only module that imports both sides

## See Also

- wiki/tier2-core/architecture-patterns/overview.md
- wiki/tier2-core/architecture-patterns/dependency-injection-bootstrap.md
- wiki/tier2-core/solid-principles/dip.md
- wiki/tier3-working/python/functional-core.md
- wiki/tier1-sources/swebok-v4/ka02-architecture.md

## Source

Harry Percival and Bob Gregory, *Architecture Patterns with Python*, Preface, Introduction, Chapters 2 and 4 (O'Reilly 2020). CC-BY-ND.
https://www.cosmicpython.com/book/introduction.html
Alistair Cockburn, "Hexagonal Architecture" (2005). https://alistair.cockburn.us/hexagonal-architecture/
Robert C. Martin, *Clean Architecture* (2017), Part V.
