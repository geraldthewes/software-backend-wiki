# Python Free-Threading (No-GIL) (Tier 3)

> **Tier 3** | Source: Python Free-Threading HOWTO, docs.python.org/3/howto/free-threading-python.html | Enforces/Derives From: wiki/tier3-working/python/overview.md, wiki/tier3-working/python/async-patterns.md

## Summary

Python 3.13 introduced a free-threaded build where the Global Interpreter Lock (GIL) is disabled, enabling true parallel execution of CPU-bound Python threads across multiple cores. This is an opt-in experimental feature (CPython still ships GIL-enabled builds by default). Free-threading is appropriate for CPU-bound multi-threaded workloads; for I/O-bound concurrency, `asyncio` remains the right choice. All code relying on the GIL for implicit thread safety must be audited before migrating.

## Key Concepts

### What the GIL Is and Why Its Removal Matters

The GIL (Global Interpreter Lock) is a mutex that prevents multiple Python threads from executing Python bytecode in parallel. In GIL-enabled builds, Python threads provide concurrency (interleaving) but not parallelism (simultaneous execution). Free-threading removes this constraint:

| Model | Concurrency | Parallelism | GIL |
|-------|-------------|-------------|-----|
| GIL-enabled + threads | Yes (interleaved) | No | Present |
| `asyncio` | Yes (cooperative) | No | Present |
| `multiprocessing` | Yes | Yes | Separate per process |
| Free-threaded + threads | Yes | Yes (true multi-core) | Removed |

### Checking for Free-Threading

```python
import sys, sysconfig

# Is this a free-threaded build?
is_free_threaded = sysconfig.get_config_var("Py_GIL_DISABLED") == 1

# Is the GIL actually disabled at runtime?
gil_enabled = sys._is_gil_enabled()  # False in free-threaded mode
```

From the command line: `python -VV` — output includes `free-threading build` if applicable.

### Enabling / Building Free-Threaded Python

```bash
# From source
./configure --disable-gil
make && make install

# Re-enable GIL at runtime (for compatibility testing)
PYTHON_GIL=1 python script.py
python -X gil script.py
```

Official installers for Python 3.13+ include free-threaded binaries as a separate download.

### Thread Safety: What Changed

Free-threaded builds retain **similar safety guarantees** to GIL-enabled builds for built-in types (`dict`, `list`, `set`). However, the GIL previously provided implicit mutual exclusion that masked race conditions in application code:

```python
# UNSAFE — relied on GIL for atomicity
shared_counter = 0

def increment() -> None:
    global shared_counter
    shared_counter += 1   # read-modify-write is NOT atomic without GIL

# SAFE — explicit lock
import threading
lock = threading.Lock()
shared_counter = 0

def increment() -> None:
    global shared_counter
    with lock:
        shared_counter += 1
```

### Known Limitations

**1. Immortalized objects:** To reduce reference-count contention, free-threaded builds immortalize certain objects (string/number literals, interned strings). They are never deallocated, causing slightly higher baseline memory.

**2. Frame locals are not thread-safe:**
```python
# UNSAFE in free-threaded builds
# Do not read frame.f_locals while the frame executes in another thread
```

**3. Iterator thread safety:** Iterating a container (`list`, `dict`) from multiple threads may yield duplicates or skipped items — use explicit locks.

**4. Single-threaded overhead:** 1–8% performance degradation vs GIL-enabled builds (platform-dependent: ~1% on macOS aarch64, ~8% on x86-64 Linux).

**5. Context variable inheritance:** Free-threaded builds default `thread_inherit_context=True` — threads inherit the caller's context variables. GIL-enabled builds default to `False`. This changes behavior when using `contextvars` across threads.

### When to Use What

| Workload | Recommended Approach |
|----------|---------------------|
| I/O-bound (network, disk) | `asyncio` |
| CPU-bound, isolated tasks | `multiprocessing` |
| CPU-bound, shared memory, many threads | Free-threaded CPython 3.13+ |
| Mixed I/O + CPU | `asyncio` + `asyncio.to_thread()` |

### Migration Checklist

```python
# 1. Check C extension compatibility
import sysconfig
if sysconfig.get_config_var("Py_GIL_DISABLED"):
    # C extensions that are NOT free-threading compatible will auto-enable the GIL
    # with a warning — watch for "GIL re-enabled" warnings at import time
    pass

# 2. Add explicit locks where GIL was the implicit guard
import threading

class Counter:
    def __init__(self) -> None:
        self._value = 0
        self._lock  = threading.Lock()

    def increment(self) -> None:
        with self._lock:
            self._value += 1

    @property
    def value(self) -> int:
        with self._lock:
            return self._value

# 3. Replace mutable global state with thread-local or contextvar equivalents
from contextvars import ContextVar
_current_user: ContextVar[str | None] = ContextVar("current_user", default=None)
```

## Agent Guidance

### Do

- Use explicit `threading.Lock()` or `threading.RLock()` for any shared mutable state — do not rely on the GIL for implicit mutual exclusion.
- Check `sysconfig.get_config_var("Py_GIL_DISABLED")` at runtime rather than assuming GIL state.
- Test C extension dependencies for free-threading compatibility before migrating a service (check https://py-free-threading.github.io/tracking/).
- Accept 1–8% single-threaded overhead as a trade-off when multi-core parallelism is the goal.
- Use `contextvars.ContextVar` instead of thread-local storage for context propagation in both threaded and async code.

### Do Not

- Do not use free-threading for I/O-bound workloads — `asyncio` is more efficient and better supported.
- Do not assume `dict` and `list` operations are race-condition-free with free-threading — they are protected against corruption but not against logical races (read-modify-write).
- Do not access `frame.f_locals` from a different thread than the frame is executing on.
- Do not share iterators across threads without locking.
- Do not deploy free-threaded Python in production without auditing all C extensions for compatibility.

## Checklist

- [ ] All shared mutable state protected by explicit `threading.Lock()`
- [ ] No read-modify-write operations on shared state without a lock
- [ ] C extension dependencies checked for free-threading compatibility
- [ ] `sysconfig.get_config_var("Py_GIL_DISABLED")` used for runtime detection, not `sys._is_gil_enabled()`
- [ ] `contextvars.ContextVar` used instead of thread-local storage
- [ ] Performance baseline measured — 1–8% overhead accepted or optimized

## See Also

- wiki/tier3-working/python/async-patterns.md
- wiki/tier3-working/python/overview.md
- wiki/tier3-working/python/isolating-extensions.md
- wiki/tier1-sources/swebok-v4/ka16-computing-foundations.md

## Source

Python Free-Threading HOWTO, docs.python.org/3/howto/free-threading-python.html. CPython 3.13 release notes. https://py-free-threading.github.io/
