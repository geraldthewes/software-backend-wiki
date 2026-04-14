# Worked Example: Dependency Injection

> **Tier 3** | Source: SOLID Principles, "Architecture Patterns with Python" | Enforces/Derives From: wiki/tier2-core/solid-principles/dip.md, wiki/tier2-core/solid-principles/isp.md, wiki/tier2-core/solid-principles/ocp.md

## Summary

A complete, executable Python example showing constructor injection. Demonstrates how to replace a tightly coupled concrete dependency with a Protocol-based abstraction, enabling testing without side effects and extension without modification.

---

## 1. Problem

`OrderService` creates an `EmailNotifier` internally. This makes it impossible to:
- Unit-test `OrderService` without sending real emails.
- Swap the notification mechanism (SMS, Slack, logging) without editing `OrderService`.
- Test failure scenarios for the notification path.

---

## 2. Bad Code

```python
import smtplib
from email.mime.text import MIMEText

class EmailNotifier:
    """Sends real emails via SMTP."""

    def __init__(self):
        # Reads config from module-level globals — not injectable
        self._smtp_host = "smtp.example.com"
        self._smtp_port = 587

    def notify(self, recipient: str, message: str) -> None:
        msg = MIMEText(message)
        msg["To"] = recipient
        msg["From"] = "orders@example.com"
        msg["Subject"] = "Order Update"
        with smtplib.SMTP(self._smtp_host, self._smtp_port) as server:
            server.sendmail("orders@example.com", recipient, msg.as_string())


class OrderService:
    def __init__(self):
        # Tightly coupled — creates its own EmailNotifier
        # Cannot test without sending emails
        # Cannot switch to SMS without modifying this class
        self._notifier = EmailNotifier()

    def place_order(self, user_email: str, product_id: int, quantity: int) -> dict:
        # Business logic
        order = {
            "order_id": 42,
            "user_email": user_email,
            "product_id": product_id,
            "quantity": quantity,
            "status": "confirmed",
        }
        # Side effect: always sends a real email
        self._notifier.notify(user_email, f"Your order {order['order_id']} is confirmed.")
        return order
```

---

## 3. Solution with Constructor Injection

### Notifier Protocol (the abstraction)

```python
# domain/notifications.py
from typing import Protocol

class Notifier(Protocol):
    """Abstraction for sending notifications. Small, focused interface (ISP)."""

    def notify(self, recipient: str, message: str) -> None:
        """Send a notification message to the recipient."""
        ...
```

### Concrete Implementations

```python
# infrastructure/email_notifier.py
import smtplib
import logging
from email.mime.text import MIMEText

logger = logging.getLogger(__name__)

class EmailNotifier:
    """Production implementation: sends via SMTP."""

    def __init__(self, smtp_host: str, smtp_port: int, sender: str) -> None:
        self._smtp_host = smtp_host
        self._smtp_port = smtp_port
        self._sender = sender

    def notify(self, recipient: str, message: str) -> None:
        msg = MIMEText(message)
        msg["To"] = recipient
        msg["From"] = self._sender
        msg["Subject"] = "Order Update"
        try:
            with smtplib.SMTP(self._smtp_host, self._smtp_port) as server:
                server.sendmail(self._sender, recipient, msg.as_string())
            logger.info("email_sent", extra={"recipient": recipient})
        except smtplib.SMTPException as e:
            logger.error("email_send_failed", extra={"recipient": recipient, "error": str(e)})
            raise
```

```python
# infrastructure/log_notifier.py
import logging

logger = logging.getLogger(__name__)

class LogNotifier:
    """Testing/development implementation: logs instead of sending."""

    def notify(self, recipient: str, message: str) -> None:
        logger.info(
            "notification_sent",
            extra={"recipient": recipient, "message": message},
        )
```

```python
# infrastructure/sms_notifier.py
import logging
from typing import Any

logger = logging.getLogger(__name__)

class SMSNotifier:
    """SMS implementation via a third-party provider."""

    def __init__(self, api_client: Any, sender_number: str) -> None:
        self._client = api_client
        self._sender = sender_number

    def notify(self, recipient: str, message: str) -> None:
        # recipient is a phone number in this context
        self._client.messages.create(
            body=message,
            from_=self._sender,
            to=recipient,
        )
        logger.info("sms_sent", extra={"recipient": recipient})
```

### Domain Service (receives Notifier via injection)

```python
# domain/order_service.py
import logging
from dataclasses import dataclass
from domain.notifications import Notifier

logger = logging.getLogger(__name__)


@dataclass(frozen=True)
class Order:
    order_id: int
    user_email: str
    product_id: int
    quantity: int
    status: str


class OrderService:
    """
    Business logic for placing orders.
    Depends on Notifier abstraction — not on any concrete notification mechanism.
    """

    def __init__(self, notifier: Notifier) -> None:
        # DIP: depends on Protocol, not EmailNotifier
        self._notifier = notifier

    def place_order(self, user_email: str, product_id: int, quantity: int) -> Order:
        """Place an order and notify the user."""
        if quantity <= 0:
            raise ValueError(f"Quantity must be positive, got {quantity}")
        if not user_email or "@" not in user_email:
            raise ValueError(f"Invalid email: {user_email!r}")

        order = Order(
            order_id=self._generate_order_id(),
            user_email=user_email,
            product_id=product_id,
            quantity=quantity,
            status="confirmed",
        )

        logger.info("order_placed", extra={"order_id": order.order_id, "product_id": product_id})

        self._notifier.notify(
            recipient=user_email,
            message=f"Your order #{order.order_id} for {quantity}x product {product_id} is confirmed.",
        )

        return order

    def _generate_order_id(self) -> int:
        """In production, this would use a DB sequence or UUID."""
        import random
        return random.randint(10000, 99999)
```

---

## 4. Tests (no side effects)

```python
# tests/test_order_service.py
import pytest
from unittest.mock import MagicMock, call
from domain.order_service import OrderService
from infrastructure.log_notifier import LogNotifier


class RecordingNotifier:
    """Test double that records calls for assertion."""

    def __init__(self) -> None:
        self.calls: list[dict] = []

    def notify(self, recipient: str, message: str) -> None:
        self.calls.append({"recipient": recipient, "message": message})


def test_place_order_returns_confirmed_order():
    notifier = RecordingNotifier()
    service = OrderService(notifier=notifier)

    order = service.place_order("alice@example.com", product_id=99, quantity=2)

    assert order.status == "confirmed"
    assert order.user_email == "alice@example.com"
    assert order.product_id == 99
    assert order.quantity == 2


def test_place_order_sends_notification():
    notifier = RecordingNotifier()
    service = OrderService(notifier=notifier)

    order = service.place_order("alice@example.com", product_id=99, quantity=2)

    assert len(notifier.calls) == 1
    call = notifier.calls[0]
    assert call["recipient"] == "alice@example.com"
    assert str(order.order_id) in call["message"]


def test_place_order_raises_on_zero_quantity():
    notifier = RecordingNotifier()
    service = OrderService(notifier=notifier)

    with pytest.raises(ValueError, match="Quantity must be positive"):
        service.place_order("alice@example.com", product_id=99, quantity=0)


def test_place_order_raises_on_invalid_email():
    notifier = RecordingNotifier()
    service = OrderService(notifier=notifier)

    with pytest.raises(ValueError, match="Invalid email"):
        service.place_order("not-an-email", product_id=99, quantity=1)


def test_place_order_does_not_notify_on_validation_failure():
    notifier = RecordingNotifier()
    service = OrderService(notifier=notifier)

    with pytest.raises(ValueError):
        service.place_order("alice@example.com", product_id=99, quantity=-1)

    assert len(notifier.calls) == 0  # no notification sent on error
```

---

## 5. Wiring in Application Factory

Show how to compose real implementations at the application entry point.

```python
# app.py
import os
from domain.order_service import OrderService
from infrastructure.email_notifier import EmailNotifier
from infrastructure.log_notifier import LogNotifier

def create_app():
    """Compose the application. Real dependencies injected here."""
    if os.environ.get("ENVIRONMENT") == "production":
        notifier = EmailNotifier(
            smtp_host=os.environ["SMTP_HOST"],
            smtp_port=int(os.environ.get("SMTP_PORT", "587")),
            sender=os.environ["EMAIL_SENDER"],
        )
    else:
        # Development / testing: log notifications instead of sending
        notifier = LogNotifier()

    order_service = OrderService(notifier=notifier)
    return order_service


if __name__ == "__main__":
    service = create_app()
    order = service.place_order("test@example.com", product_id=1, quantity=3)
    print(f"Order placed: {order}")
```

---

## 6. Principles Demonstrated

| Principle | How This Example Applies |
|-----------|--------------------------|
| **DIP** | `OrderService` depends on `Notifier` Protocol, not `EmailNotifier`. High-level module doesn't depend on low-level module. |
| **OCP** | Add `SMSNotifier` or `SlackNotifier` without modifying `OrderService`. |
| **ISP** | `Notifier` protocol has exactly one method — no client forced to implement unused methods. |
| **Testability** | `RecordingNotifier` enables testing notification behavior without SMTP, credentials, or network. |
| **SRP** | `OrderService` handles order logic; `EmailNotifier` handles SMTP; neither knows about the other's internals. |

---

## See Also

- wiki/tier2-core/solid-principles/dip.md
- wiki/tier2-core/solid-principles/isp.md
- wiki/tier2-core/solid-principles/ocp.md
- wiki/tier3-working/python/functional-core.md
- wiki/tier3-working/worked-examples/repository-pattern.md

## Source

"Architecture Patterns with Python" (Percival & Gregory, O'Reilly 2020). SOLID Principles (Robert C. Martin). "Growing Object-Oriented Software, Guided by Tests" (Freeman & Pryce, 2009).
