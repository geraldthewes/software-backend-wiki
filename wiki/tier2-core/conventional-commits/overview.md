# Conventional Commits (Tier 2)

> **Tier 2** | Source: conventionalcommits.org 1.0.0 | Authority: established | Enforces: SWEBOK KA8

## Summary

Conventional Commits is a specification for adding human- and machine-readable meaning to commit messages. The convention defines a lightweight structure — `<type>[scope]: <description>` — on top of plain git commit messages, enabling automated tooling to parse change history and determine version bumps, generate changelogs, and trigger CI pipelines.

For a coding agent, Conventional Commits is the commit message standard to apply on any project that adopts it. Every commit produced should communicate its intent precisely: whether it patches a bug (`fix`), introduces a feature (`feat`), or breaks backward compatibility (`BREAKING CHANGE`). This precision is not stylistic — it directly drives automated release decisions through SemVer.

## Key Concepts

### Commit Message Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Type Taxonomy

| Type | SemVer Impact | Meaning |
|------|--------------|---------|
| `feat` | MINOR bump | Introduces a new feature |
| `fix` | PATCH bump | Patches a bug |
| `BREAKING CHANGE` (footer or `!`) | MAJOR bump | Backward-incompatible change — any type |
| `build` | None | Build system or dependency changes |
| `chore` | None | Routine maintenance, no production code change |
| `ci` | None | CI configuration changes |
| `docs` | None | Documentation only |
| `perf` | None | Performance improvement (no new feature) |
| `refactor` | None | Code restructuring without behavior change |
| `style` | None | Code style or formatting (no logic change) |
| `test` | None | Adding or fixing tests |
| `revert` | Varies | Reverts a previous commit |

> Additional types are permitted; only `feat`, `fix`, and `BREAKING CHANGE` carry SemVer weight by specification.

### SemVer Mapping

| Commit content | SemVer release |
|---------------|---------------|
| `BREAKING CHANGE:` footer **or** `!` after type/scope | MAJOR (x.0.0) |
| `feat:` (no breaking change) | MINOR (0.x.0) |
| `fix:` (no breaking change) | PATCH (0.0.x) |
| All other types | No automatic release |

### Scope

A scope is a noun in parentheses after the type, describing the affected section of the codebase: `feat(auth):`, `fix(parser):`. It is optional but consistent scope usage improves changelog grouping across monorepos and multi-module projects.

### Breaking Change Declaration

Two equivalent ways to mark a breaking change:

1. **Footer**: `BREAKING CHANGE: <description>` — uppercase, followed by colon and description
2. **Inline `!`**: `feat!: drop Python 3.8 support` — appended directly before the colon

Both may be combined: `feat(api)!:` with a `BREAKING CHANGE:` footer provides the detail. If `!` is used alone, the description line IS the breaking change description.

## Agent Guidance

### Do

- Use `feat:` for every new capability visible to callers or users.
- Use `fix:` for every bug correction, including test fixes that expose real bugs.
- Use `BREAKING CHANGE:` footer or `!` for any API, interface, or behavior change that requires callers to update.
- Include a scope when the repository has multiple distinct modules (monorepo pattern): `feat(billing):`.
- Write the description in imperative mood, lowercase, no trailing period: `fix: prevent race condition in cache loader`.
- Include a body when the "why" is non-obvious — one blank line after the description.
- Use `Refs: #123` or `Reviewed-by:` footers for traceability.
- Keep each commit to one logical change; if multiple types apply, split into multiple commits.

### Do Not

- Do not use `feat` for refactors that add no user-visible capability.
- Do not omit `BREAKING CHANGE` when removing a public function, changing a function signature, or altering default behavior.
- Do not write descriptions in past tense ("fixed") — use imperative ("fix").
- Do not leave the description blank or use generic messages like `fix: bug` or `feat: stuff`.
- Do not capitalize the description after the colon (unless it starts with a proper noun).
- Do not mix unrelated changes in one commit — this defeats changelog generation.
- Do not use `chore:` or `refactor:` as a catch-all to avoid thinking about the correct type.

## Checklist

- [ ] Type is one of the recognized types from the taxonomy above
- [ ] Description is in imperative mood, lowercase, no trailing period
- [ ] If backward-incompatible: `BREAKING CHANGE:` footer present OR `!` appended to type/scope
- [ ] Scope (if used) is a consistent noun identifying the affected module
- [ ] Body (if present) begins one blank line after description and explains WHY, not WHAT
- [ ] Footers (if present) use `Token: value` or `Token #value` format with hyphenated multi-word tokens

## See Also

- wiki/tier2-core/conventional-commits/specification.md
- wiki/tier1-sources/swebok-v4/ka08-config-management.md
- wiki/tier1-sources/swebok-v4/ka06-operations.md
- wiki/tier2-core/twelve-factor-app/factors.md
- wiki/tier2-core/code-review-method/overview.md
- wiki/tier3-working/checklists/pre-commit.md

## Source

Conventional Commits 1.0.0. conventionalcommits.org. CC BY 3.0.
https://www.conventionalcommits.org/en/v1.0.0/
