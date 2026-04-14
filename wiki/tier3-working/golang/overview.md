# Go Best Practices — Overview

> **Tier 3** | Source: Effective Go, Go documentation | Enforces/Derives From: wiki/tier1-sources/swebok-v4/ka04-construction.md

## Summary

Go is designed for simplicity, explicit error handling, and first-class concurrency. Its design philosophy — readable code over clever code, explicit over implicit — makes it predictable at scale. This page introduces Go's guiding principles and links to sub-pages.

## Go's Design Philosophy

**Simplicity**: Go has a small language spec by design. There is no inheritance, no exceptions, no generics-through-interfaces overload. When in doubt, the Go team chose the simpler option.

**Explicit error handling**: Functions return `(result, error)`. There is no exception mechanism to hide failures. Every error must be checked — the compiler will warn about unused returns but the programmer must handle them.

**Concurrency as a first-class feature**: Goroutines (lightweight threads) and channels (typed communication) are built into the language. The Go scheduler multiplexes goroutines onto OS threads, making concurrent programs efficient without thread management overhead.

## How Go Differs from Python

| Concept | Python | Go |
|---------|--------|----|
| Error handling | Exceptions (`try/except`) | Return values `(result, error)` |
| Interfaces | Explicit (`class Foo(Protocol)`) | Implicit (structural duck typing) |
| Concurrency | `asyncio` / `threading` | Goroutines + channels |
| Generics | `TypeVar` / `Generic` | Type parameters (Go 1.18+) |
| Null safety | `None` — no static check | `nil` — some static checking via linters |
| Packaging | pip / pyproject.toml | `go mod` / `go.sum` |

## Sub-Pages

| Page | What It Covers |
|------|----------------|
| wiki/tier3-working/golang/idioms.md | Error handling, interfaces, defer, naming, anti-patterns |
| wiki/tier3-working/golang/concurrency.md | Goroutines, channels, sync primitives, context, patterns |
| wiki/tier3-working/golang/toolchain.md | go mod, go test, golangci-lint, gofmt, CI targets |

## Key Tools

- **go vet** — basic static analysis; always run before commit
- **golangci-lint** — comprehensive linting; replaces many individual linters
- **gofmt / goimports** — canonical formatting; non-negotiable; auto-run on save
- **go test -race** — race detector; always enable in CI
- **govulncheck** — vulnerability scanner against known CVEs in dependencies

## See Also

- wiki/tier1-sources/swebok-v4/ka04-construction.md
- wiki/tier3-working/golang/idioms.md
- wiki/tier3-working/golang/concurrency.md
- wiki/tier3-working/golang/toolchain.md

## Source

Effective Go (golang.org). "The Go Programming Language" (Donovan & Kernighan, 2015). Go blog (go.dev/blog). SWEBOK V4, KA4 Software Construction.
