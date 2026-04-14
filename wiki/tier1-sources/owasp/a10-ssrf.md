# A10: Server-Side Request Forgery (SSRF)

> **Tier 1** | Source: OWASP Top 10:2021 | Authority: immutable

## Summary

Server-Side Request Forgery (SSRF) occurs when an application fetches a remote resource based on a user-supplied URL without validating or sanitizing it. An attacker can use SSRF to bypass firewalls, access internal services, read cloud provider metadata (including credentials), and enumerate internal network topology — all from outside the network perimeter. For a coding agent, any feature that fetches a URL based on user input requires strict allowlist validation.

## Key Concepts

**Attack Scenarios:**

- **Cloud metadata exfiltration**: On AWS, GCP, or Azure, the metadata endpoint at `169.254.169.254` returns instance credentials. An SSRF vulnerability allows an external attacker to retrieve cloud IAM credentials with a single request.
- **Internal service enumeration**: An attacker uses the server as a proxy to scan internal IP ranges and ports, discovering databases, admin panels, and internal APIs.
- **Firewall bypass**: Internal services that do not authenticate requests from within the network are accessible via SSRF on a perimeter-facing service.
- **File system access**: On some frameworks, `file://` URLs can read local files; `dict://` and `gopher://` can interact with internal services.

**Why SSRF Is Newly Critical (added in 2021):**

SSRF moved from an obscure finding to a critical threat with the widespread adoption of cloud infrastructure. Cloud providers expose a metadata endpoint on every virtual machine that returns temporary credentials. This single endpoint has been the root cause of several high-profile breaches.

## Python Mitigations

**Allowlist approach — only permit explicitly approved hosts:**

```python
from urllib.parse import urlparse
import ipaddress
import socket

ALLOWED_HOSTS = frozenset([
    "api.example.com",
    "cdn.example.com",
    "trusted-partner.com",
])

def is_safe_url(url: str) -> bool:
    """Validate URL against allowlist and private IP block."""
    try:
        parsed = urlparse(url)
        if parsed.scheme not in ("http", "https"):
            return False
        if parsed.hostname not in ALLOWED_HOSTS:
            return False
        return True
    except Exception:
        return False

# Usage
def fetch_user_supplied_url(url: str) -> bytes:
    if not is_safe_url(url):
        raise ValueError(f"URL not permitted: {url}")
    response = requests.get(url, timeout=10)
    return response.content
```

**Block private IP ranges — resolve DNS before checking:**

DNS rebinding allows an attacker to provide a domain that resolves to a public IP during validation but switches to a private IP for the actual request. Always resolve DNS and validate the resolved IP:

```python
import ipaddress
import socket

PRIVATE_NETWORKS = [
    ipaddress.ip_network("10.0.0.0/8"),
    ipaddress.ip_network("172.16.0.0/12"),
    ipaddress.ip_network("192.168.0.0/16"),
    ipaddress.ip_network("169.254.0.0/16"),   # link-local / cloud metadata
    ipaddress.ip_network("127.0.0.0/8"),       # loopback
    ipaddress.ip_network("::1/128"),            # IPv6 loopback
    ipaddress.ip_network("fc00::/7"),           # IPv6 private
]

def is_safe_ip(hostname: str) -> bool:
    """Resolve hostname and verify it does not point to a private/reserved range."""
    try:
        resolved_ip = socket.gethostbyname(hostname)
        ip = ipaddress.ip_address(resolved_ip)
        return not any(ip in network for network in PRIVATE_NETWORKS)
    except (socket.gaierror, ValueError):
        return False
```

**Block cloud metadata endpoints explicitly:**

```python
BLOCKED_HOSTS = frozenset([
    "169.254.169.254",    # AWS/GCP/Azure instance metadata
    "metadata.google.internal",
    "169.254.170.2",       # AWS ECS task metadata
])

def is_blocked_host(hostname: str) -> bool:
    return hostname in BLOCKED_HOSTS
```

**Use a dedicated HTTP client with SSRF protection:**

The `ssrf-guard` library provides a drop-in replacement for `requests` with SSRF mitigations built in:

```bash
pip install ssrf-guard
```

```python
from ssrf_guard import SsrfGuard

guard = SsrfGuard()
response = guard.get(user_supplied_url)   # private IPs and metadata blocked automatically
```

## Agent Guidance

### Do
- Apply allowlist validation on every URL-fetching feature before implementation
- Resolve DNS and validate resolved IPs, not just the hostname
- Block all private IP ranges and cloud metadata endpoints explicitly
- Use `ssrf-guard` or equivalent for any service that fetches user-supplied URLs
- Log all SSRF validation failures — they indicate active reconnaissance

### Do Not
- Accept user-supplied URLs and pass them directly to `requests.get()` or equivalent
- Rely on blocklist-only approaches — blocklists are easier to bypass than allowlists
- Assume internal network security compensates for SSRF — it does not
- Omit DNS rebinding protection (checking only the hostname, not the resolved IP)

## Checklist
- [ ] URL allowlist implemented for all user-supplied URL inputs
- [ ] DNS resolved and resulting IP validated against private range blocklist before request
- [ ] Cloud metadata endpoint (`169.254.169.254`) and vendor-specific variants explicitly blocked
- [ ] Only `http` and `https` schemes permitted — `file://`, `dict://`, `gopher://` rejected
- [ ] SSRF validation failures logged with source IP and attempted URL
- [ ] Timeout configured on all outbound HTTP requests to prevent slow-loris style attacks

## See Also
- wiki/tier1-sources/owasp/top10-2021-overview.md
- wiki/tier1-sources/owasp/a01-broken-access-control.md
- wiki/tier1-sources/owasp/a09-logging-monitoring.md

## Source
OWASP Top 10:2021 — A10 Server-Side Request Forgery. https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/
