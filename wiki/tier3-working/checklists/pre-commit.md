# Pre-Commit / Pre-Push Checklist

> **Tier 3** | Enforces: wiki/tier1-sources/swebok-v4/ka04-construction.md, wiki/tier1-sources/python-peps/pep-008-style.md, wiki/tier1-sources/python-peps/pep-484-type-hints.md, wiki/tier1-sources/swebok-v4/ka13-security.md

## Before Every Commit

### Secrets and Sensitive Data

- [ ] No secrets, API keys, or credentials in staged files (use `git-secrets` or `trufflehog` scan)
- [ ] No `.env` files staged (only `.env.template`)
- [ ] No TODO comments hiding security issues (e.g., `# TODO: add auth`)
- [ ] No hardcoded connection strings, database URLs, or IP addresses

### Code Style (Python)

- [ ] `ruff check .` or `flake8` passes with no errors
- [ ] `ruff format --check .` or `black --check .` — no formatting changes needed
- [ ] `mypy --strict` (or project config) passes with no new errors
- [ ] Import order correct: stdlib → third-party → local (enforced by ruff/isort)
- [ ] No unused imports or variables

### Code Style (Go)

- [ ] `gofmt` / `goimports` applied — `diff <(gofmt -d .) /dev/null` shows no output
- [ ] `go vet ./...` passes
- [ ] `golangci-lint run ./...` passes

### Type Safety

- [ ] All new public functions/methods have type hints (Python) or explicit types (Go)
- [ ] No new `Any` types without a justification comment
- [ ] No new `interface{}` without justification (Go) — use generics or specific types
- [ ] No `cast()` calls that mask real type errors

### Python Logging

- [ ] No `print()` calls used for operational or diagnostic logging (use `logger.*` instead)
- [ ] Every module defines `logger = logging.getLogger(__name__)` at module level
- [ ] Library packages add only `NullHandler()` to their root logger — no `StreamHandler`
- [ ] No sensitive data (passwords, tokens, PII) in any log message or format string
- [ ] `logger.exception()` used inside `except` blocks (not `logger.error` + manual traceback)
- [ ] `dictConfig`/`basicConfig` called once at startup with `disable_existing_loggers: False`

### Security (pyscg / OWASP)

- [ ] No new hardcoded configuration values — use environment variables
- [ ] No new bare `except:` or `except Exception: pass`
- [ ] No `shell=True` with any user-controlled input
- [ ] No `pickle.loads()`, `eval()`, or `exec()` on untrusted data
- [ ] No new string-formatted SQL — all queries use parameterized form

## Before Every Push / PR

### Testing

- [ ] `pytest` / `go test ./...` passes locally
- [ ] No new public functions without unit tests
- [ ] Error paths explicitly tested (not just the happy path)
- [ ] `go test -race ./...` passes (Go projects)
- [ ] `pip-audit` (Python) or `govulncheck ./...` (Go) passes — no known vulnerabilities

### Self-Review

- [ ] Code review checklist self-applied (wiki/tier3-working/checklists/code-review.md)
- [ ] Security review checklist applied for any auth, data handling, or external-call changes
- [ ] Design review checklist applied for any new components or services
- [ ] Commit messages describe why, not just what

## See Also

- wiki/tier3-working/checklists/code-review.md
- wiki/tier3-working/checklists/security-review.md
- wiki/tier1-sources/python-peps/pep-008-style.md
- wiki/tier1-sources/python-peps/pep-484-type-hints.md
- wiki/tier3-working/golang/toolchain.md
- wiki/tier3-working/python/logging.md
- wiki/tier3-working/observability/structured-logging.md

## Source

OpenSSF Pyscg secure coding standard. PEP 8, PEP 484. OWASP Top 10. SWEBOK V4, KA13 Security.
