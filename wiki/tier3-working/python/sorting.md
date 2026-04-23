# Python Sorting (Tier 3)

> **Tier 3** | Source: Python Sorting HOWTO, docs.python.org/3/howto/sorting.html | Enforces/Derives From: wiki/tier3-working/python/overview.md, wiki/tier1-sources/python-peps/pep-020-zen.md

## Summary

Python provides two sorting interfaces: the built-in `sorted()` function (returns a new list from any iterable) and `list.sort()` (sorts in-place, returns `None`). Both use Timsort — a stable, adaptive sort with O(n log n) worst-case performance. The `key` parameter is the idiomatic way to control sort order: it is called exactly once per element, making complex sorts efficient. Nested sorts exploit stability to achieve multi-key ordering without multi-field key functions.

## Key Concepts

### sorted() vs list.sort()

| | `sorted(iterable)` | `list.sort()` |
|--|-------------------|--------------| 
| Returns | New sorted list | `None` |
| Works on | Any iterable | Lists only |
| Original unchanged | Yes | No |
| Use when | Input must be preserved | In-place OK, no copy needed |

```python
# sorted() — new list, original unchanged
result = sorted([3, 1, 2])         # [1, 2, 3]
result = sorted("hello")           # ['e', 'h', 'l', 'l', 'o']

# list.sort() — in-place, returns None
data = [3, 1, 2]
data.sort()                         # data is now [1, 2, 3]
x = data.sort()                     # x is None — common mistake
```

### The key Parameter

`key` is a one-argument function called on each element. Its return value is used for comparison. This is more efficient than a comparison function because it is called once per element:

```python
# Sort case-insensitively
words = ["Banana", "apple", "Cherry"]
sorted(words, key=str.casefold)     # ['apple', 'Banana', 'Cherry']

# Sort objects by attribute
from dataclasses import dataclass

@dataclass
class Employee:
    name: str
    department: str
    salary: float

employees = [...]
sorted(employees, key=lambda e: e.salary)
```

### operator Module — Preferred over Lambdas

`operator.attrgetter` and `operator.itemgetter` are faster than lambdas and support multiple keys:

```python
from operator import attrgetter, itemgetter

# By single attribute
sorted(employees, key=attrgetter('salary'))

# By multiple attributes — sort by department, then by name within department
sorted(employees, key=attrgetter('department', 'name'))

# Sort list of tuples by index 1, then index 2
records = [('Alice', 'Eng', 90), ('Bob', 'Mkt', 85), ('Carol', 'Eng', 95)]
sorted(records, key=itemgetter(1, 2))
# [('Alice', 'Eng', 90), ('Carol', 'Eng', 95), ('Bob', 'Mkt', 85)]
```

### Reverse Sorting

```python
# Reverse entire sort
sorted(employees, key=attrgetter('salary'), reverse=True)

# Multi-key with mixed directions — use sort stability
# Sort ascending by name first, then descending by salary
step1 = sorted(employees, key=attrgetter('name'))
step2 = sorted(step1, key=attrgetter('salary'), reverse=True)
```

### Sort Stability and Multi-Pass Sorting

Python's sort is **stable**: elements with equal keys retain their original relative order. This enables multi-pass sorting:

```python
from operator import attrgetter

def multisort(xs: list, specs: list[tuple[str, bool]]) -> list:
    """Sort by multiple keys with per-key direction.
    specs: [(field_name, reverse), ...] — later specs are primary.
    """
    for key, reverse in reversed(specs):
        xs.sort(key=attrgetter(key), reverse=reverse)
    return xs

# Sort by salary descending, then name ascending as tiebreaker
multisort(employees, [('name', False), ('salary', True)])
```

### Comparison Functions and cmp_to_key

If you have a legacy comparison function (returns negative/zero/positive), wrap it:

```python
from functools import cmp_to_key
from locale import strcoll

# Locale-aware string sort
sorted(words, key=cmp_to_key(strcoll))
```

### Partial Sorts — heapq

When you only need the N smallest or largest items, `heapq` is more efficient than full sort:

```python
from heapq import nsmallest, nlargest

top5 = nlargest(5, employees, key=attrgetter('salary'))
bottom3 = nsmallest(3, transactions, key=itemgetter('amount'))
```

### Handling Unorderable Elements

```python
from itertools import filterfalse
from math import isnan

# Remove NaN before sorting floats
clean = sorted(x for x in data if not isnan(x))

# Remove None
clean = sorted(x for x in data if x is not None)
```

## Agent Guidance

### Do

- Use `sorted()` when the original iterable must be preserved; use `list.sort()` when in-place mutation is acceptable.
- Use `operator.attrgetter` and `operator.itemgetter` instead of lambda for attribute/index key functions — they are faster and support multiple fields natively.
- Use multi-pass sorting to achieve mixed-direction multi-key sorts — Python's stability guarantees the correct result.
- Use `heapq.nsmallest` / `heapq.nlargest` when only the top/bottom N results are needed.
- Always sort on a consistent, well-defined key — sort behavior on equal keys is deterministic only when using stable sort (which Python always uses).

### Do Not

- Do not assign the return value of `list.sort()` — it returns `None`.
- Do not use `lambda x: (x.a, -x.b)` tricks for mixed-direction sorts on objects — use the multi-pass pattern instead for clarity.
- Do not use `cmp_to_key` for new code — define a `key` function instead.
- Do not sort heterogeneous collections with incompatible types (e.g., `int` and `None`) — filter out `None` and `NaN` values first.

## Checklist

- [ ] `sorted()` used when original must be preserved; `list.sort()` for in-place
- [ ] `operator.attrgetter`/`itemgetter` used instead of lambdas for attribute/index keys
- [ ] Multi-key mixed-direction sorts use the multi-pass stability pattern
- [ ] `heapq.nsmallest`/`nlargest` used for partial sorts
- [ ] Unorderable values (NaN, None) filtered before sorting

## See Also

- wiki/tier3-working/python/overview.md
- wiki/tier3-working/python/idioms.md
- wiki/tier3-working/python/functional-core.md
- wiki/tier1-sources/python-peps/pep-020-zen.md

## Source

Python Sorting HOWTO, docs.python.org/3/howto/sorting.html. Python `operator`, `heapq`, `functools` module documentation.
