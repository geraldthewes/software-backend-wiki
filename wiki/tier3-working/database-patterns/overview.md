# Database Patterns — Overview

> **Tier 3** | Source: "Architecture Patterns with Python", OWASP A03 | Enforces/Derives From: wiki/tier1-sources/owasp/a03-injection.md, wiki/tier2-core/solid-principles/dip.md

## Summary

How an application accesses its database determines its testability, security, and long-term maintainability. This page introduces the three key concerns for data access code and links to detailed sub-pages.

## Why Data Access Patterns Matter

Direct database access in domain logic creates three categories of problems:

1. **Maintainability**: Business logic tangled with SQL is hard to change. Altering a schema requires hunting through every caller.
2. **Security**: String-formatted SQL queries are the primary vector for SQL injection (OWASP A03). An abstraction layer makes parameterized queries the only path.
3. **Testability**: A function that calls `cursor.execute()` directly cannot be unit-tested without a real database. A repository pattern allows injection of an in-memory fake.

## Three Key Concerns

| Concern | Pattern | Risk Without It |
|---------|---------|----------------|
| Abstraction | Repository pattern | Untestable code, SRP/DIP violations |
| Schema evolution | Migrations (Alembic, golang-migrate) | Schema drift, broken environments |
| Query performance | Indexing, connection pooling, N+1 avoidance | Slow queries, connection exhaustion |

## OWASP A03 — The Primary Security Concern

SQL injection (OWASP A03:2021 — Injection) remains one of the most prevalent vulnerabilities in web applications. The single most effective mitigation is **never constructing SQL by string formatting**. Always use parameterized queries or an ORM that generates parameterized queries automatically.

```python
# BAD — SQL injection possible
query = f"SELECT * FROM users WHERE email = '{user_email}'"

# GOOD — parameterized query
cursor.execute("SELECT * FROM users WHERE email = ?", (user_email,))
```

The repository pattern enforces this by making the data access code the only place SQL is written, and making parameterized queries the standard pattern within that layer.

## Sub-Pages

| Page | What It Covers |
|------|----------------|
| wiki/tier3-working/database-patterns/repository-pattern.md | Abstraction layer between domain and DB; Protocol-based design; fake implementations for testing |
| wiki/tier3-working/database-patterns/migrations.md | Alembic (Python), golang-migrate; expand-contract pattern; dangerous operations |
| wiki/tier3-working/database-patterns/query-optimization.md | N+1 problem, indexing, connection pooling, EXPLAIN ANALYZE, cursor pagination |

## See Also

- wiki/tier3-working/database-patterns/repository-pattern.md
- wiki/tier3-working/database-patterns/migrations.md
- wiki/tier3-working/database-patterns/query-optimization.md
- wiki/tier1-sources/owasp/a03-injection.md
- wiki/tier2-core/solid-principles/dip.md

## Source

"Architecture Patterns with Python" (Percival & Gregory, O'Reilly 2020). OWASP Top 10: A03:2021 Injection. SWEBOK V4, KA13 Security.
