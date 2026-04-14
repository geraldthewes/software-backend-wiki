# Query Optimization

> **Tier 3** | Source: PostgreSQL docs, SQLAlchemy docs | Enforces/Derives From: wiki/tier1-sources/owasp/a03-injection.md, wiki/tier3-working/database-patterns/repository-pattern.md

## Summary

Slow queries and missing indexes are among the most common and fixable performance problems in backend services. This page covers the N+1 problem, indexing strategy, query analysis, connection pooling, and pagination patterns.

## The N+1 Query Problem

The N+1 problem occurs when you fetch a list of N records, then issue one additional query per record — N+1 total queries instead of 1 or 2.

```python
# BAD — N+1 queries
users = session.query(User).all()          # 1 query
for user in users:
    orders = user.orders                   # 1 query per user = N queries
    print(f"{user.name}: {len(orders)} orders")

# Result: 1 + N queries to the database
```

### Solutions: JOIN or Eager Loading

```python
# SQLAlchemy — selectinload (1 extra query, no Cartesian product)
from sqlalchemy.orm import selectinload

users = session.query(User).options(selectinload(User.orders)).all()
# 2 queries total: one for users, one for all orders

# SQLAlchemy — joinedload (1 query with JOIN)
from sqlalchemy.orm import joinedload

users = session.query(User).options(joinedload(User.orders)).all()
# 1 query with LEFT OUTER JOIN (can duplicate rows; use for to-one)

# Django equivalents
User.objects.select_related("profile")       # JOIN — to-one relationships
User.objects.prefetch_related("orders")      # 2 queries — to-many relationships
```

### Detection

```python
# Enable SQL logging in SQLAlchemy to count queries during development
import logging
logging.getLogger("sqlalchemy.engine").setLevel(logging.INFO)

# Assert query count in integration tests
from sqlalchemy import event

def count_queries(conn, cursor, statement, parameters, context, executemany):
    count_queries.count += 1

count_queries.count = 0
event.listen(engine, "before_cursor_execute", count_queries)

users_with_orders = get_users_with_orders(session)

assert count_queries.count <= 2, f"Expected ≤2 queries, got {count_queries.count}"
```

## Indexing

### Always Index Foreign Keys

```sql
-- Every foreign key column should have an index
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
```

### Index Columns Used in WHERE, ORDER BY, GROUP BY

```sql
-- Frequently filtered columns
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_status ON users(status);

-- Sorting
CREATE INDEX idx_orders_created_at ON orders(created_at DESC);
```

### Composite Indexes — Order Matters

Put the most selective (highest cardinality) column first. The index can be used for prefix queries.

```sql
-- Composite index: usable for queries on (status), (status, user_id), but NOT (user_id) alone
CREATE INDEX idx_orders_status_user ON orders(status, user_id);

-- For range queries, put the range column last
CREATE INDEX idx_events_type_ts ON events(event_type, created_at);
```

### Avoid Over-Indexing

Every index slows INSERT, UPDATE, and DELETE operations because the index must be updated too. As a guideline:
- Tables with heavy reads: index aggressively.
- Tables with heavy writes: index conservatively; only add what query analysis proves is needed.

## Query Analysis

Use `EXPLAIN ANALYZE` (PostgreSQL) to understand what the query planner does.

```sql
EXPLAIN ANALYZE
SELECT u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE u.status = 'active'
GROUP BY u.id, u.name;
```

Key output patterns:

| Output | Meaning | Action |
|--------|---------|--------|
| `Seq Scan` on large table | No usable index | Add index on the filtered column |
| `Index Scan` | Index used | Good |
| `Bitmap Index Scan` | Index used for range | Usually good |
| `Hash Join` on large tables | Full scan join | May need composite index |
| `Nested Loop` on large tables | O(n*m) — potential problem | Consider JOIN or index |

```bash
# PostgreSQL: visualize with explain.tensor.ru or pgMustard
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON) SELECT ...;
```

## Connection Pooling

Never create a new database connection per request. Connections are expensive (TCP handshake, authentication, memory allocation).

### SQLAlchemy Connection Pool

```python
from sqlalchemy import create_engine

engine = create_engine(
    DATABASE_URL,
    pool_size=5,         # connections to keep open
    max_overflow=10,     # additional connections when pool is exhausted
    pool_timeout=30,     # seconds to wait for a connection
    pool_recycle=1800,   # recycle connections after 30 minutes (avoid stale connections)
)
```

Pool size starting point: `(CPU_count * 2) + 1` per application instance. Adjust based on observed connection wait times.

### asyncpg / aiopg Pool

```python
import asyncpg

pool = await asyncpg.create_pool(
    DATABASE_URL,
    min_size=2,
    max_size=10,
)

async with pool.acquire() as conn:
    rows = await conn.fetch("SELECT id, name FROM users")
```

## Query Patterns

### Avoid `SELECT *`

```sql
-- BAD — transfers all columns, prevents index-only scans
SELECT * FROM users WHERE status = 'active';

-- GOOD — only fetch what you need
SELECT id, name, email FROM users WHERE status = 'active';
```

### Always Use Parameterized Queries

```python
# GOOD — prevents SQL injection and enables query plan caching
cursor.execute(
    "SELECT id, name FROM users WHERE email = %s AND status = %s",
    (email, status)
)
```

Query plan caching: the database plans the query once for the parameterized form and reuses the plan on subsequent calls with different values.

### Pagination

**Offset pagination** (simple, but slow for large offsets):

```sql
SELECT id, name FROM users ORDER BY id LIMIT 20 OFFSET 1000;
-- Problem: DB must scan and skip 1000 rows to find the 20 you want
```

**Cursor-based pagination** (scalable — recommended for large datasets):

```sql
-- First page
SELECT id, name FROM users WHERE id > 0 ORDER BY id LIMIT 20;

-- Next page — use the last id from previous page as cursor
SELECT id, name FROM users WHERE id > :last_id ORDER BY id LIMIT 20;
```

Cursor pagination is O(log N) per page (index seek) vs O(N) for OFFSET.

### Read Replicas

For read-heavy workloads, direct read queries to a read replica and writes to the primary:

```python
read_engine = create_engine(READ_REPLICA_URL, ...)
write_engine = create_engine(PRIMARY_URL, ...)
```

## See Also

- wiki/tier3-working/database-patterns/repository-pattern.md
- wiki/tier1-sources/owasp/a03-injection.md
- wiki/tier3-working/observability/metrics.md

## Source

PostgreSQL documentation: EXPLAIN, indexes. SQLAlchemy documentation: relationship loading techniques. "High Performance MySQL" (Schwartz et al., O'Reilly). pgBouncer documentation: connection pooling.
