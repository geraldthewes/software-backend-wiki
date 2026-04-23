# Python ipaddress Module (Tier 3)

> **Tier 3** | Source: Python ipaddress HOWTO, docs.python.org/3/howto/ipaddress.html | Enforces/Derives From: wiki/tier3-working/python/overview.md, wiki/tier1-sources/owasp/a10-ssrf.md

## Summary

Python's `ipaddress` module provides classes for working with IPv4 and IPv6 addresses, networks, and host interfaces. It replaces manual string parsing and integer arithmetic for IP address manipulation, with type-safe comparison, containment testing, iteration over host addresses, and correct handling of CIDR notation. This module is essential for network configuration tools, firewall rule generation, and SSRF-prevention URL validation.

## Key Concepts

### Object Type Hierarchy

| Class | Example Input | Represents |
|-------|--------------|------------|
| `IPv4Address` | `'192.0.2.1'` | Single IPv4 host address |
| `IPv6Address` | `'2001:db8::1'` | Single IPv6 host address |
| `IPv4Network` | `'192.0.2.0/24'` | IPv4 network range (no host bits) |
| `IPv6Network` | `'2001:db8::/32'` | IPv6 network range |
| `IPv4Interface` | `'192.0.2.1/24'` | IPv4 address + network prefix |
| `IPv6Interface` | `'2001:db8::1/32'` | IPv6 address + network prefix |

### Creating Objects

```python
import ipaddress

# Version-agnostic factory functions (prefer these when accepting input)
addr = ipaddress.ip_address('192.0.2.1')        # → IPv4Address
addr = ipaddress.ip_address('2001:db8::1')       # → IPv6Address
net  = ipaddress.ip_network('192.0.2.0/24')     # → IPv4Network
iface = ipaddress.ip_interface('10.0.0.1/24')    # → IPv4Interface

# From integers
ipaddress.ip_address(3221225985)                 # → IPv4Address('192.0.2.1')

# Networks with host bits — strict=False silently zeroes host bits
# strict=True (default) raises ValueError
net = ipaddress.ip_network('192.0.2.1/24', strict=False)  # → '192.0.2.0/24'

# Class constructors for explicit version
ipaddress.IPv4Address('192.168.1.1')
ipaddress.IPv6Network('fe80::/10')
```

### Key Attributes

```python
addr = ipaddress.ip_address('192.0.2.42')
addr.version       # 4
addr.is_private    # False
addr.is_loopback   # False
addr.is_multicast  # False

net = ipaddress.ip_network('192.0.2.0/24')
net.num_addresses          # 256
net.network_address        # IPv4Address('192.0.2.0')
net.broadcast_address      # IPv4Address('192.0.2.255')
net.netmask                # IPv4Address('255.255.255.0')
net.prefixlen              # 24

iface = ipaddress.ip_interface('10.0.1.5/28')
iface.ip                   # IPv4Address('10.0.1.5')  — the host address
iface.network              # IPv4Network('10.0.1.0/28')
```

### Containment Testing (SSRF Prevention)

```python
import ipaddress

BLOCKED_NETWORKS = [
    ipaddress.ip_network('10.0.0.0/8'),       # private
    ipaddress.ip_network('172.16.0.0/12'),     # private
    ipaddress.ip_network('192.168.0.0/16'),    # private
    ipaddress.ip_network('169.254.0.0/16'),    # link-local / cloud metadata
    ipaddress.ip_network('127.0.0.0/8'),       # loopback
    ipaddress.ip_network('::1/128'),           # IPv6 loopback
    ipaddress.ip_network('fc00::/7'),          # IPv6 ULA
]

def is_blocked(host: str) -> bool:
    """Returns True if the resolved IP is in a blocked (private/metadata) range."""
    try:
        addr = ipaddress.ip_address(host)
    except ValueError:
        return False   # not an IP literal — check after DNS resolution
    return any(addr in net for net in BLOCKED_NETWORKS)
```

### Iteration

```python
net = ipaddress.ip_network('192.0.2.0/28')

# All addresses (includes network and broadcast)
list(net)           # [IPv4Address('192.0.2.0'), ..., IPv4Address('192.0.2.15')]

# Usable host addresses (excludes network and broadcast)
list(net.hosts())   # [IPv4Address('192.0.2.1'), ..., IPv4Address('192.0.2.14')]

# Subnetting
list(net.subnets(prefixlen_diff=1))   # Split into two /29 networks
net.supernet(prefixlen_diff=1)        # Enclosing /27 network
```

### Comparisons

```python
a1 = ipaddress.ip_address('192.0.2.1')
a2 = ipaddress.ip_address('192.0.2.2')
a1 < a2    # True

# Membership
a1 in ipaddress.ip_network('192.0.2.0/24')   # True

# Cross-version comparison raises TypeError
# ipaddress.ip_address('::1') < ipaddress.ip_address('127.0.0.1')  → TypeError
```

### Type Conversion

```python
addr = ipaddress.ip_address('192.0.2.1')
str(addr)     # '192.0.2.1'
int(addr)     # 3221225985  — useful for storage
bytes(addr)   # b'\xc0\x00\x02\x01'  — packed form for socket calls

# IPv6 string forms
v6 = ipaddress.ip_address('2001:db8::1')
v6.exploded    # '2001:0db8:0000:0000:0000:0000:0000:0001'
v6.compressed  # '2001:db8::1'
```

### Error Handling

```python
try:
    addr = ipaddress.ip_address(user_input)
except ValueError as e:
    # Invalid address literal — both ValueError and its subclasses are caught
    raise ValidationError(f"Invalid IP address: {user_input}") from e
```

`AddressValueError` and `NetmaskValueError` both inherit from `ValueError`, so catching `ValueError` is sufficient.

## Agent Guidance

### Do

- Use version-agnostic factory functions (`ip_address`, `ip_network`, `ip_interface`) when accepting user input — they raise `ValueError` on invalid input and handle both IPv4 and IPv6.
- Use `in` operator for CIDR containment tests — it is readable and correct.
- Pre-compute `BLOCKED_NETWORKS` as module-level constants — parsing CIDR strings is not free.
- Use `strict=False` when parsing CIDR from external sources — many tools emit `192.168.1.5/24` with host bits set.
- Use `ipaddress` for SSRF prevention: resolve hostnames, then check the resolved IP against blocked ranges (see `wiki/tier1-sources/owasp/a10-ssrf.md`).

### Do Not

- Do not compare addresses from different IP versions (`IPv4Address` vs `IPv6Address`) — it raises `TypeError`.
- Do not manually parse IP strings with `str.split()` — `ipaddress` handles edge cases correctly.
- Do not rely on string-based blocklists for private IP detection — use `ipaddress` network containment.
- Do not assume `net.hosts()` and `list(net)` are equivalent — `hosts()` excludes network and broadcast addresses.

## Checklist

- [ ] IP addresses parsed with `ipaddress.ip_address()` not manual string parsing
- [ ] SSRF prevention blocklist uses `ipaddress.ip_network` containment tests
- [ ] `strict=False` used when parsing CIDRs from external sources
- [ ] Cross-version comparisons avoided
- [ ] `ValueError` caught from all `ipaddress` factory functions

## See Also

- wiki/tier1-sources/owasp/a10-ssrf.md
- wiki/tier3-working/python/sockets.md
- wiki/tier3-working/python/urllib.md
- wiki/tier3-working/python/overview.md
- wiki/tier2-core/security-practices/python-pyscg.md

## Source

Python ipaddress HOWTO, docs.python.org/3/howto/ipaddress.html. Python `ipaddress` module documentation. OWASP A10:2021 SSRF.
