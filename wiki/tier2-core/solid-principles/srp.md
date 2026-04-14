# Single Responsibility Principle (SRP)

> **Tier 2** | Source: Robert C. Martin | Derives From: ka03-design | Authority: established practice

## Summary

A class should have only one reason to change. SRP is the most foundational of the SOLID principles and the first to apply. Small, focused classes are easier to read, test, and modify without unintended side effects.

## Key Concepts

### Definition

A class should have **only one reason to change**. The word "reason" here means "responsibility to one actor or stakeholder." If a class serves two different stakeholders — for example, a finance team and a reporting team — then changes requested by either can break behavior relied upon by the other.

### How to Identify Violations

- The class has more than one cohesive section of methods that do not relate to each other
- The class name contains "And" (`UserSaverAndEmailer`) or "Manager" (catch-all names are a warning sign)
- Methods within the class have no meaningful relationship — for instance, a class has both `query_database()` and `send_email()`
- The class requires changes for multiple, unrelated reasons across its lifetime

### Bad Example — SRP Violation

```python
class UserService:
    """Does too many things: DB queries, email, and formatting."""

    def get_user(self, user_id: int) -> dict:
        conn = psycopg2.connect("host=localhost dbname=mydb")
        cur = conn.cursor()
        cur.execute("SELECT * FROM users WHERE id = %s", (user_id,))
        return cur.fetchone()

    def send_welcome_email(self, user_email: str) -> None:
        smtp = smtplib.SMTP("mail.example.com")
        smtp.sendmail("noreply@example.com", user_email, "Welcome!")

    def format_user_display(self, user: dict) -> str:
        return f"{user['first_name']} {user['last_name']} <{user['email']}>"
```

This class changes if the database schema changes, if the email provider changes, and if the display format changes — three distinct reasons.

### Good Example — SRP Applied

```python
from dataclasses import dataclass
from typing import Protocol

@dataclass
class User:
    id: int
    first_name: str
    last_name: str
    email: str


class UserRepository:
    """Responsible for: persistence only."""

    def get(self, user_id: int) -> User:
        # database interaction isolated here
        ...


class EmailService:
    """Responsible for: email delivery only."""

    def send_welcome(self, user: User) -> None:
        # SMTP interaction isolated here
        ...


class UserFormatter:
    """Responsible for: display formatting only."""

    def format(self, user: User) -> str:
        return f"{user.first_name} {user.last_name} <{user.email}>"
```

Each class now has a single reason to change. Swapping the email provider does not touch `UserRepository` or `UserFormatter`.

### Testing Benefit

Small, single-responsibility classes are straightforward to unit test because:
- Each class has a narrow, well-defined contract
- There are fewer interactions to stub or mock
- Test failures point directly to the responsible component

A test for `UserFormatter` needs no database or SMTP fixture — it only constructs a `User` object and asserts on the formatted string.

### Relationship to Functional Core

Pure functions are the extreme case of SRP: by definition, a pure function takes inputs, returns output, and does nothing else. Designing with SRP pushes object-oriented code toward this ideal — small classes that transform or store data, rather than orchestrating multiple concerns simultaneously.

## Agent Guidance

### Do
- Identify all the "actors" (stakeholders) that could request changes to a class; if there is more than one, split the class
- Name classes after their single responsibility (`UserRepository`, `InvoiceFormatter`, `PaymentGateway`)
- Place data (`@dataclass`) and behavior (service/repository) in separate classes
- Treat the presence of "And" or "Manager" in a class name as a trigger to review and refactor

### Do Not
- Do not create God classes that handle multiple concerns
- Do not put database queries and business logic in the same class
- Do not put I/O operations (network, filesystem, email) in the same class as domain logic
- Do not conflate data formatting/presentation with data retrieval

## Checklist
- [ ] Class name reflects a single, clear responsibility
- [ ] Class has no methods belonging to a different concern
- [ ] Class name does not contain "And," "Manager," or "Handler" as a catch-all
- [ ] Unit test for this class requires mocking only one type of external dependency (or none)
- [ ] Changing the database schema does not require changing the email or formatting class

## See Also
- `wiki/tier2-core/solid-principles/overview.md`
- `wiki/tier2-core/solid-principles/ocp.md`
- `wiki/tier2-core/solid-principles/dip.md`
- `wiki/tier2-core/design-patterns/overview.md`
- `wiki/tier1-sources/swebok-v4/ka01-requirements.md`

## Source

Robert C. Martin, *Agile Software Development: Principles, Patterns, and Practices* (2002). Synthesized from *Software Development Best Practices for Agent* reference document.
