# Python Async Patterns

> **Tier 3** | Source: Python asyncio docs, PEP 492 | Enforces/Derives From: wiki/tier1-sources/swebok-v4/ka04-construction.md

## Summary

Python's `asyncio` enables concurrent I/O-bound operations on a single thread using cooperative multitasking. This page covers when to use async, core patterns, common pitfalls, and the boundary between sync and async code.

## When to Use Async

| Scenario | Recommended Approach |
|----------|---------------------|
| Concurrent HTTP calls, DB queries, file I/O | `asyncio` — I/O-bound concurrency |
| CPU-intensive computation (parsing, ML inference) | `multiprocessing` or `concurrent.futures.ProcessPoolExecutor` |
| Simple sequential scripts | Synchronous Python — no async needed |
| Calling a sync library from async code | `asyncio.to_thread()` |

**Rule**: `asyncio` does not speed up CPU-bound work. It only allows I/O to overlap. Using `asyncio.sleep()` while waiting for I/O is cooperative; using `time.sleep()` inside async code blocks the entire event loop.

## Core Syntax

```python
import asyncio

async def fetch_user(user_id: int) -> dict:
    """A coroutine — must be awaited."""
    await asyncio.sleep(0.1)   # simulates I/O wait
    return {"id": user_id, "name": "Alice"}

async def main() -> None:
    user = await fetch_user(1)
    print(user)

# Entry point — creates and runs the event loop
asyncio.run(main())
```

## Concurrency with gather

`asyncio.gather()` runs multiple coroutines concurrently and waits for all to finish.

```python
import asyncio
import aiohttp

async def fetch(session: aiohttp.ClientSession, url: str) -> str:
    async with session.get(url) as response:
        return await response.text()

async def fetch_all(urls: list[str]) -> list[str]:
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(session, url) for url in urls]
        results = await asyncio.gather(*tasks)
    return list(results)
```

`gather()` returns results in the same order as the input coroutines, regardless of completion order.

## Structured Concurrency with TaskGroup (Python 3.11+)

Prefer `asyncio.TaskGroup` over `gather()` for new code. If any task raises, all remaining tasks are cancelled automatically.

```python
import asyncio

async def process_all(items: list[str]) -> list[str]:
    results: list[str] = []
    async with asyncio.TaskGroup() as tg:
        tasks = [tg.create_task(process_item(item)) for item in items]
    # All tasks have completed (or all were cancelled on first error)
    return [task.result() for task in tasks]
```

## Timeouts — Always Set on Network Calls

Never make a network call without a timeout. Stuck connections block the coroutine forever.

```python
import asyncio

async def fetch_with_timeout(url: str) -> bytes:
    try:
        return await asyncio.wait_for(download(url), timeout=5.0)
    except asyncio.TimeoutError:
        raise ServiceTimeoutError(f"Request to {url} timed out after 5s")
```

## aiohttp — Async HTTP Client

```python
import aiohttp
import asyncio

async def get_json(url: str) -> dict:
    # Reuse the session across requests — do not create per-request
    async with aiohttp.ClientSession() as session:
        async with session.get(url, timeout=aiohttp.ClientTimeout(total=10)) as resp:
            resp.raise_for_status()
            return await resp.json()

# For long-lived applications, keep the session at module / app level
class ApiClient:
    def __init__(self) -> None:
        connector = aiohttp.TCPConnector(limit=100)   # connection pool
        self._session = aiohttp.ClientSession(connector=connector)

    async def close(self) -> None:
        await self._session.close()

    async def get(self, url: str) -> dict:
        async with self._session.get(url) as resp:
            resp.raise_for_status()
            return await resp.json()
```

## Async Context Managers

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def managed_db_connection(dsn: str):
    conn = await asyncpg.connect(dsn)
    try:
        yield conn
    finally:
        await conn.close()

async def main() -> None:
    async with managed_db_connection("postgresql://...") as conn:
        rows = await conn.fetch("SELECT * FROM users LIMIT 10")
```

## Async Generators

```python
async def paginate(url: str, page_size: int = 100):
    """Yield pages lazily — don't load all records into memory."""
    offset = 0
    while True:
        page = await fetch_page(url, offset, page_size)
        if not page:
            break
        for item in page:
            yield item
        offset += page_size

async def main() -> None:
    async for user in paginate("/api/users"):
        await process(user)
```

## Error Handling

Exceptions propagate through `await` normally. With `TaskGroup`, the first exception cancels sibling tasks.

```python
import asyncio

async def safe_fetch(url: str) -> dict | None:
    try:
        return await fetch_json(url)
    except aiohttp.ClientError as e:
        logger.error("Fetch failed for %s: %s", url, e)
        return None

# gather with return_exceptions=True — collects errors as values instead of raising
results = await asyncio.gather(*tasks, return_exceptions=True)
for result in results:
    if isinstance(result, Exception):
        logger.error("Task failed: %s", result)
```

## Sync / Async Boundaries

```python
import asyncio
from concurrent.futures import ProcessPoolExecutor

# Call sync (blocking) code from async context
async def run_blocking_io() -> str:
    loop = asyncio.get_event_loop()
    result = await loop.run_in_executor(None, sync_file_operation, path)
    return result

# Call blocking CPU-bound code in a process pool
executor = ProcessPoolExecutor()

async def run_cpu_bound(data: bytes) -> bytes:
    loop = asyncio.get_event_loop()
    return await loop.run_in_executor(executor, compress, data)

# Simpler syntax with asyncio.to_thread (Python 3.9+)
async def query_sync_db() -> list:
    return await asyncio.to_thread(sync_db_query, "SELECT ...")
```

## Common Pitfalls

| Pitfall | Wrong | Correct |
|---------|-------|---------|
| Blocking sleep in async | `time.sleep(1)` | `await asyncio.sleep(1)` |
| Nested event loop | `asyncio.run()` inside running loop | `await` inside async context |
| Creating a new session per request | `aiohttp.ClientSession()` per call | One session, reused |
| No timeout on network calls | `await fetch(url)` | `await asyncio.wait_for(fetch(url), timeout=5.0)` |
| CPU work in event loop | `result = heavy_compute(data)` | `await asyncio.to_thread(heavy_compute, data)` |

## See Also

- wiki/tier3-working/python/idioms.md
- wiki/tier3-working/golang/concurrency.md
- wiki/tier3-working/observability/structured-logging.md

## Source

Python docs: asyncio, aiohttp. PEP 492 — Coroutines with async and await. PEP 654 — Exception Groups (Python 3.11+). "Using Asyncio in Python" (Caleb Hattingh, O'Reilly 2020).
