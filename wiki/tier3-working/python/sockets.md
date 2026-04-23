# Python Sockets (Tier 3)

> **Tier 3** | Source: Python Sockets HOWTO, docs.python.org/3/howto/sockets.html | Enforces/Derives From: wiki/tier3-working/python/overview.md, wiki/tier2-core/distributed-systems/resilience-patterns.md, wiki/tier1-sources/swebok-v4/ka16-computing-foundations.md

## Summary

Python's `socket` module exposes BSD socket primitives for network and IPC communication. TCP (`SOCK_STREAM`) sockets are bidirectional, connection-oriented, and provide ordered byte delivery. The key insight for correct socket programming is that `send()` and `recv()` operate on buffers — a single call may not transfer all requested bytes, so all production code must loop until the full message is sent or received. For higher-level protocols, use `urllib`, `httpx`, or `asyncio`/`aiohttp` rather than raw sockets.

## Key Concepts

### Socket Roles

| Role | Creates | Data flow |
|------|---------|-----------|
| Server socket | Connection-accepting socket | Calls `bind()`, `listen()`, `accept()` — produces client sockets |
| Client socket | Connection to server | Calls `connect()` — sends and receives data |
| Accepted socket | Produced by `accept()` | Sends and receives data with one peer |

A server socket does **not** send or receive data — it produces client sockets via `accept()`.

### Creating Sockets

```python
import socket

# Server — bind and listen
server_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server_sock.bind(('', 8080))      # '' = all interfaces; use 'localhost' for local-only
server_sock.listen(5)             # backlog — max queued connections

# Accept loop
while True:
    client_sock, addr = server_sock.accept()
    # handle client_sock in a thread or process
    client_sock.close()

# Client — connect
client_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client_sock.connect(('server.example.com', 8080))
```

### Reliable Send and Receive

`send()` may not send all bytes; `recv()` may return fewer bytes than requested. Always loop:

```python
def send_all(sock: socket.socket, data: bytes) -> None:
    """Send all bytes, retrying until complete."""
    total_sent = 0
    while total_sent < len(data):
        sent = sock.send(data[total_sent:])
        if sent == 0:
            raise RuntimeError("Socket connection broken during send")
        total_sent += sent

def recv_exactly(sock: socket.socket, length: int) -> bytes:
    """Receive exactly `length` bytes."""
    chunks: list[bytes] = []
    received = 0
    while received < length:
        chunk = sock.recv(min(length - received, 4096))
        if chunk == b'':
            raise RuntimeError("Socket connection closed before receiving all data")
        chunks.append(chunk)
        received += len(chunk)
    return b''.join(chunks)
```

Note: `recv()` returning `b''` (empty bytes) means the connection was closed by the peer.

### Message Framing

Sockets are byte streams — there is no concept of "message boundary". Choose a framing strategy:

| Strategy | How | When to Use |
|----------|-----|-------------|
| Fixed length | All messages are N bytes | Simple binary protocols |
| Length prefix | 4-byte length header + payload | Variable-length messages |
| Delimiter | Newline or null byte separator | Line-oriented text protocols |
| Connection-per-message | Close after sending | Simple one-shot exchanges |

```python
import struct

# Length-prefix framing
def send_message(sock: socket.socket, data: bytes) -> None:
    header = struct.pack('>I', len(data))   # 4-byte big-endian length
    send_all(sock, header + data)

def recv_message(sock: socket.socket) -> bytes:
    header = recv_exactly(sock, 4)
    length = struct.unpack('>I', header)[0]
    return recv_exactly(sock, length)
```

### Non-Blocking Sockets and select

For multiplexing multiple connections without threading:

```python
import select

readable, writable, errored = select.select(
    read_list,    # sockets to monitor for readability
    write_list,   # sockets to monitor for writability
    error_list,   # sockets to monitor for errors
    timeout       # seconds (None = block forever)
)

for sock in readable:
    if sock is server_sock:
        client, addr = server_sock.accept()
        read_list.append(client)
    else:
        data = sock.recv(4096)
        if data == b'':
            sock.close()
            read_list.remove(sock)
        else:
            process(data)
```

### Proper Shutdown

```python
# Graceful shutdown — signal done sending, still able to receive
sock.shutdown(socket.SHUT_WR)   # or socket.SHUT_RD, SHUT_RDWR
sock.close()
```

Always call `close()` explicitly — do not rely on garbage collection. Unclosed sockets leave the remote end hanging in `CLOSE_WAIT`.

### IPC: Local Sockets vs Network Sockets

For inter-process communication on the same machine:

```python
# Unix domain sockets — no TCP overhead, stays in kernel
server = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
server.bind('/tmp/myapp.sock')

# If you must use AF_INET for IPC, bind to localhost only
server.bind(('127.0.0.1', 8080))   # not '0.0.0.0' — avoids network exposure
```

### Binary Data Byte Order

Network byte order is big-endian. Use `struct` or `socket` conversion functions:

```python
import struct

# Pack/unpack with explicit byte order
data = struct.pack('>HI', port, address)   # big-endian unsigned short + int
port, addr = struct.unpack('>HI', data)

# Socket convenience functions
socket.htonl(x)   # host to network long (32-bit)
socket.htons(x)   # host to network short (16-bit)
socket.ntohl(x)   # network to host long
socket.ntohs(x)   # network to host short
```

## Agent Guidance

### Do

- Always loop in both `send()` and `recv()` — single calls are not guaranteed to transfer all bytes.
- Treat `recv()` returning `b''` as a connection close signal, not an error.
- Use `SO_REUSEADDR` on server sockets to allow quick restart without "address already in use" errors.
- Use `select.select()` or `asyncio` for multiplexing multiple connections instead of blocking threads per connection.
- Call `sock.close()` explicitly in a `finally` block or use `contextlib.closing()`.
- Bind server sockets to `'localhost'`/`'127.0.0.1'` for IPC — do not expose on `'0.0.0.0'` unless the service is intended to be network-accessible.
- Use `asyncio` or a higher-level framework (Twisted, Trio) for production network servers rather than raw blocking sockets.

### Do Not

- Do not assume `send(data)` sends all of `data` — check the return value or use `sendall()`.
- Do not assume `recv(N)` returns exactly N bytes.
- Do not kill threads stuck on blocking `recv()` — use non-blocking sockets with `select` or redesign around async I/O.
- Do not use `recv()` on a connection-oriented socket to implement a request-response protocol without explicit message framing.
- Do not rely on garbage collection to close sockets — use `try/finally` or context managers.

## Checklist

- [ ] All `recv()` calls loop until the expected number of bytes is received
- [ ] `recv()` returning `b''` handled as connection close
- [ ] `send()` calls use `sendall()` or a looping wrapper
- [ ] `SO_REUSEADDR` set on server sockets before `bind()`
- [ ] Sockets closed explicitly in `finally` blocks
- [ ] IPC server sockets bound to `127.0.0.1` or Unix domain socket, not `0.0.0.0`
- [ ] Message framing implemented (length prefix or delimiter)

## See Also

- wiki/tier3-working/python/urllib.md
- wiki/tier3-working/python/async-patterns.md
- wiki/tier2-core/distributed-systems/resilience-patterns.md
- wiki/tier1-sources/swebok-v4/ka16-computing-foundations.md
- wiki/tier3-working/python/ipaddress.md

## Source

Python Sockets HOWTO, docs.python.org/3/howto/sockets.html. Python `socket`, `select`, `struct` module documentation.
