# Python urllib — HTTP Requests (Tier 3)

> **Tier 3** | Source: Python urllib2 HOWTO, docs.python.org/3/howto/urllib2.html | Enforces/Derives From: wiki/tier3-working/python/overview.md, wiki/tier1-sources/owasp/a10-ssrf.md, wiki/tier2-core/distributed-systems/resilience-patterns.md

## Summary

Python's `urllib.request` module provides low-level HTTP request handling from the standard library. For production services, the third-party `requests` or `httpx` library is strongly preferred for its simpler API, better timeout control, and session management. This page covers `urllib.request` patterns and explains when to use each option.

## Key Concepts

### urllib.request Core API

```python
import urllib.request
import urllib.parse
import urllib.error

# Simple GET
with urllib.request.urlopen('https://api.example.com/data', timeout=10) as resp:
    body = resp.read().decode('utf-8')
    print(resp.status)          # HTTP status code
    print(resp.getheader('Content-Type'))

# GET with query parameters
params = {'page': '1', 'limit': '100'}
url = 'https://api.example.com/items?' + urllib.parse.urlencode(params)
with urllib.request.urlopen(url, timeout=10) as resp:
    data = resp.read()
```

### POST Request

```python
values = {'name': 'Alice', 'email': 'alice@example.com'}
data = urllib.parse.urlencode(values).encode('ascii')
req = urllib.request.Request(
    'https://api.example.com/users',
    data=data,
    method='POST'
)
req.add_header('Content-Type', 'application/x-www-form-urlencoded')
req.add_header('User-Agent', 'MyApp/1.0')

with urllib.request.urlopen(req, timeout=10) as resp:
    result = resp.read().decode('utf-8')
```

### Error Handling

`HTTPError` is a subclass of `URLError` — always catch it first:

```python
from urllib.error import HTTPError, URLError

try:
    with urllib.request.urlopen(req, timeout=10) as resp:
        return resp.read()
except HTTPError as e:
    # Server returned an error status (4xx, 5xx)
    raise ServiceError(f"HTTP {e.code}: {e.reason}") from e
except URLError as e:
    # Network-level error (DNS failure, connection refused, timeout)
    raise NetworkError(f"Request failed: {e.reason}") from e
```

### Response Object

```python
with urllib.request.urlopen(url) as resp:
    resp.status          # int status code (e.g., 200)
    resp.reason          # status reason string
    resp.read()          # body as bytes
    resp.read(size)      # read up to N bytes
    resp.getheader(name) # single header value
    resp.geturl()        # final URL (after redirects)
    resp.info()          # header dict-like object
```

### Timeout — Always Set

```python
# Per-request timeout (recommended)
urllib.request.urlopen(url, timeout=10)

# Global socket timeout (fallback — affects all sockets)
import socket
socket.setdefaulttimeout(10)
```

## When to Use requests or httpx Instead

`urllib.request` is the standard library choice for minimal-dependency scripts. For production services, prefer:

| Library | When to Use |
|---------|-------------|
| `urllib.request` | Scripts with no external deps; standard library only |
| `requests` | Synchronous service code; simplest API |
| `httpx` | Async-capable; supports both sync and async; recommended for new code |

```python
# requests — simpler, recommended for sync code
import requests

resp = requests.get(
    'https://api.example.com/items',
    params={'page': 1},
    headers={'Authorization': f'Bearer {token}'},
    timeout=10
)
resp.raise_for_status()   # raises HTTPError for 4xx/5xx
data = resp.json()

# httpx — async support, same API shape
import httpx

async def fetch(url: str) -> dict:
    async with httpx.AsyncClient(timeout=10.0) as client:
        resp = await client.get(url)
        resp.raise_for_status()
        return resp.json()
```

## Security Considerations (SSRF)

User-supplied URLs must be validated before being passed to any HTTP client. See `wiki/tier1-sources/owasp/a10-ssrf.md`:

```python
from urllib.parse import urlparse

ALLOWED_SCHEMES = {'https'}
BLOCKED_HOSTS = {'169.254.169.254', 'metadata.google.internal'}  # cloud metadata

def validate_url(url: str) -> str:
    """Validate URL before fetching — prevent SSRF."""
    parsed = urlparse(url)
    if parsed.scheme not in ALLOWED_SCHEMES:
        raise ValueError(f"Scheme '{parsed.scheme}' not allowed")
    if parsed.hostname in BLOCKED_HOSTS:
        raise ValueError(f"Host '{parsed.hostname}' is blocked")
    return url
```

## Agent Guidance

### Do

- Always set a `timeout` on every `urlopen()` call — never leave it unbounded.
- Catch `HTTPError` before `URLError` (since `HTTPError` is a subclass).
- Decode the response body explicitly with a known encoding rather than assuming ASCII.
- Validate and allowlist user-supplied URLs before fetching them — prevent SSRF.
- Prefer `requests` or `httpx` over `urllib.request` in service code.

### Do Not

- Do not leave `timeout=None` (the default) on network requests in service code — hung connections block the thread or event loop.
- Do not pass user-controlled values directly into `urlopen()` without URL validation.
- Do not use `urllib.request.install_opener()` globally in library code — it affects all requests in the process.
- Do not fetch `http://` URLs from services that should use TLS — enforce HTTPS-only in production.
- Do not ignore HTTP error responses — always check `resp.status` or call `raise_for_status()`.

## Checklist

- [ ] Timeout set on every request (`timeout=N` seconds)
- [ ] `HTTPError` caught before `URLError`
- [ ] User-supplied URLs validated against allowlist before fetching
- [ ] Response decoded with explicit encoding
- [ ] Production code uses `requests` or `httpx` rather than raw `urllib.request`
- [ ] No `http://` URLs in production without TLS justification

## See Also

- wiki/tier1-sources/owasp/a10-ssrf.md
- wiki/tier3-working/python/async-patterns.md
- wiki/tier2-core/distributed-systems/resilience-patterns.md
- wiki/tier3-working/python/sockets.md
- wiki/tier3-working/python/overview.md

## Source

Python urllib2 HOWTO, docs.python.org/3/howto/urllib2.html. Python `urllib.request`, `urllib.error`, `urllib.parse` module documentation. OWASP A10:2021 SSRF.
