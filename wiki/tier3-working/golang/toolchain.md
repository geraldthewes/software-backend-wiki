# Go Toolchain Guide

> **Tier 3** | Source: Go documentation, golangci-lint docs | Enforces/Derives From: wiki/tier1-sources/swebok-v4/ka04-construction.md

## Summary

Go ships a comprehensive standard toolchain. This page covers the tools every Go project must use, their flags, and how to integrate them into CI.

## go mod — Module Management

```bash
# Initialize a new module
go mod init github.com/myorg/myservice

# Add missing and remove unused module requirements
go mod tidy

# Download all dependencies to local cache
go mod download

# Verify downloaded modules against go.sum
go mod verify
```

`go.sum` records cryptographic hashes of module versions. Commit both `go.mod` and `go.sum`. Never edit them manually.

## go build

```bash
# Build all packages in the module
go build ./...

# Build with race detector (use during development and CI)
go build -race ./...

# Reproducible builds — strip local paths from binary
go build -trimpath -o bin/myservice ./cmd/myservice

# Cross-compile
GOOS=linux GOARCH=amd64 go build -o bin/myservice-linux ./cmd/myservice
```

## go test

```bash
# Run all tests
go test ./...

# Verbose output — see each test name
go test -v ./...

# Run a specific test by name (regex)
go test -run TestCreateUser ./...

# Run all tests with race detector — ALWAYS enable in CI
go test -race ./...

# Benchmarks
go test -bench=. -benchmem ./...

# Coverage
go test -cover ./...
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out -o coverage.html
```

Minimum CI command: `go test -race -cover ./...`

## go vet — Basic Static Analysis

```bash
go vet ./...
```

`go vet` catches common mistakes: unreachable code, incorrect `sync.Mutex` usage, `Printf` format mismatches, and more. Run before every commit and in CI. It ships with the Go toolchain — no installation needed.

## golangci-lint — Comprehensive Linting

Install once per machine or CI environment:

```bash
# Install latest version
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
```

Run:

```bash
golangci-lint run ./...
golangci-lint run --fix ./...   # auto-fix where possible
```

Configuration in `.golangci.yaml` at the project root:

```yaml
linters:
  enable:
    - errcheck       # unhandled errors
    - gosec          # security issues (hardcoded credentials, shell injection, etc.)
    - staticcheck    # advanced static analysis
    - govet          # same as go vet
    - exhaustive     # missing switch cases for enums
    - nolintlint     # must use specific linter names in //nolint comments
    - revive         # general Go style

linters-settings:
  errcheck:
    check-type-assertions: true
  gosec:
    severity: medium
  exhaustive:
    default-signifies-exhaustive: true

issues:
  max-issues-per-linter: 0
  max-same-issues: 0
```

Key linters and what they catch:

| Linter | Catches |
|--------|---------|
| `errcheck` | Unhandled `error` return values |
| `gosec` | Hardcoded credentials, `shell=true` equivalent, weak crypto |
| `staticcheck` | Deprecated API usage, impossible conditions, unused parameters |
| `exhaustive` | Missing cases in `switch` over custom enum types |
| `nolintlint` | Vague `//nolint` comments without specifying the linter |

## gofmt / goimports — Formatting

Formatting is not optional in Go. All code must be `gofmt`-compliant.

```bash
# Format all files in place
gofmt -w .

# goimports — also manages imports (add missing, remove unused)
go install golang.org/x/tools/cmd/goimports@latest
goimports -w .
```

Configure your editor to run `goimports` on save. In CI, verify no diff remains after formatting:

```bash
# Fail CI if formatting is needed
diff <(gofmt -d .) /dev/null
```

## go generate — Code Generation

```go
// In your .go file, add a directive:
//go:generate protoc --go_out=. --go-grpc_out=. api/service.proto
//go:generate mockgen -source=repository.go -destination=mock_repository.go
```

```bash
go generate ./...
```

Always commit generated files. Generated files should have a `// Code generated ... DO NOT EDIT.` header.

## Makefile Patterns

A standard `Makefile` for Go projects:

```makefile
.PHONY: build test lint vet fmt clean

build:
	go build -trimpath -o bin/myservice ./cmd/myservice

test:
	go test -race -cover ./...

lint:
	golangci-lint run ./...

vet:
	go vet ./...

fmt:
	goimports -w .

clean:
	rm -rf bin/

# Run everything that CI runs, locally
ci: fmt vet lint test
```

## CI/CD — Required Checks

All of the following must pass in CI before merge:

```yaml
# Example GitHub Actions step
steps:
  - name: Format check
    run: diff <(gofmt -d .) /dev/null

  - name: Vet
    run: go vet ./...

  - name: Lint
    run: golangci-lint run ./...

  - name: Test with race detector
    run: go test -race -cover ./...

  - name: Vulnerability scan
    run: govulncheck ./...
```

Install `govulncheck`:

```bash
go install golang.org/x/vuln/cmd/govulncheck@latest
govulncheck ./...
```

## See Also

- wiki/tier3-working/golang/idioms.md
- wiki/tier1-sources/swebok-v4/ka04-construction.md
- wiki/tier3-working/checklists/pre-commit.md

## Source

Go documentation: go command (go.dev/ref/mod). golangci-lint documentation (golangci-lint.run). govulncheck (go.dev/blog/vuln). Effective Go (golang.org).
