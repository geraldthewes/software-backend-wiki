# Conventional Commits — Full Specification (Tier 2)

> **Tier 2** | Source: conventionalcommits.org 1.0.0 | Authority: established | Enforces: SWEBOK KA8

## Summary

This page contains the complete normative Conventional Commits 1.0.0 specification — all 16 rules — with agent-facing interpretation for each rule. Read `overview.md` first for the format summary and type taxonomy. Use this page when a commit situation is ambiguous and the exact rule is needed.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" are interpreted as described in RFC 2119.

## The 16 Specification Rules

### Rule 1 — Type Prefix Is Mandatory

> Commits MUST be prefixed with a type, which consists of a noun (`feat`, `fix`, etc.), followed by the OPTIONAL scope, OPTIONAL `!`, and REQUIRED terminal colon and space.

Every commit message must start with `<type>:` or `<type>(scope):`. A commit without this prefix does not conform.

```
# VALID
fix: correct null check in user validator
feat(payments): add Stripe webhook handler

# INVALID — no type prefix
correct null check in user validator
```

### Rule 2 — `feat` Type

> The type `feat` MUST be used when a commit adds a new feature to your application or library.

Use `feat` exclusively for new externally-visible capabilities. A new API endpoint, CLI flag, config option, or exportable function all qualify. This correlates with a MINOR release in SemVer.

### Rule 3 — `fix` Type

> The type `fix` MUST be used when a commit represents a bug fix for your application.

Use `fix` for any correction of incorrect behavior. This correlates with a PATCH release in SemVer.

### Rule 4 — Scope Format

> A scope MAY be provided after a type. A scope MUST consist of a noun describing a section of the codebase surrounded by parenthesis, e.g., `fix(parser):`.

Scope is free-form but must be a noun, lowercase, no spaces. Choose scopes that map to modules, packages, or bounded contexts. Keep scopes consistent across commits.

```
feat(auth): add OAuth2 PKCE flow
fix(db): handle connection timeout on startup
refactor(parser): extract token normalizer
```

### Rule 5 — Description Is Mandatory

> A description MUST immediately follow the colon and space after the type/scope prefix. The description is a short summary of the code changes.

The one-space gap after the colon is required. The description must exist — an empty description is invalid.

```
# VALID
fix: array parsing issue when multiple spaces in string

# INVALID — no description
fix:
```

### Rule 6 — Body Placement

> A longer commit body MAY be provided after the short description, providing additional contextual information about the code changes. The body MUST begin one blank line after the description.

If including a body, exactly one blank line must separate the description from the body.

```
fix: prevent racing of requests

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.
```

### Rule 7 — Body Is Free-Form

> A commit body is free-form and MAY consist of any number of newline-separated paragraphs.

No structural constraints on the body — write paragraphs as needed to explain context, motivation, and trade-offs.

### Rule 8 — Footer Placement and Format

> One or more footers MAY be provided one blank line after the body. Each footer MUST consist of a word token, followed by either a `:<space>` or `<space>#` separator, followed by a string value.

Footers go after the body (or after the description if no body). Common footers:

```
BREAKING CHANGE: old API is removed
Reviewed-by: Alice Smith
Refs: #456
Co-authored-by: Bob Jones <bob@example.com>
```

### Rule 9 — Footer Token Hyphenation

> A footer's token MUST use `-` in place of whitespace characters (e.g., `Acked-by`). Exception: `BREAKING CHANGE` MAY use a space.

Multi-word footer tokens are hyphenated. `BREAKING CHANGE` is the only permitted exception.

```
# VALID
Co-authored-by: Alice <alice@example.com>
BREAKING CHANGE: config schema renamed

# INVALID — space in multi-word token (not BREAKING CHANGE)
Co authored by: Alice <alice@example.com>
```

### Rule 10 — Footer Value Parsing

> A footer's value MAY contain spaces and newlines, and parsing MUST terminate when the next valid footer token/separator pair is observed.

Footer values can be multi-line. Tooling stops reading the current footer when it encounters a new `Token:` or `Token #` pattern.

### Rule 11 — Breaking Change Declaration Location

> Breaking changes MUST be indicated in the type/scope prefix of a commit, or as an entry in the footer.

Either `feat!:` or a `BREAKING CHANGE:` footer is required — not optional — when the change breaks backward compatibility.

### Rule 12 — `BREAKING CHANGE` Footer Syntax

> If included as a footer, a breaking change MUST consist of the uppercase text `BREAKING CHANGE`, followed by a colon, space, and description.

```
feat: switch config format to TOML

BREAKING CHANGE: JSON config files are no longer supported; migrate to config.toml
```

### Rule 13 — `!` Indicator

> If included in the type/scope prefix, breaking changes MUST be indicated by a `!` immediately before the `:`. If `!` is used, `BREAKING CHANGE:` MAY be omitted from the footer section, and the commit description SHALL be used to describe the breaking change.

`!` is shorthand. When using `!` alone, the description line must explain what breaks. `!` and `BREAKING CHANGE:` footer can be combined when more detail is needed.

```
# ! alone — description explains the break
feat!: remove deprecated /v1 endpoints

# ! + BREAKING CHANGE footer — footer provides detail
feat(api)!: require authentication on all endpoints

BREAKING CHANGE: unauthenticated requests now return 401; update all callers to include Bearer token
```

### Rule 14 — Other Types Are Permitted

> Types other than `feat` and `fix` MAY be used in your commit messages.

`docs:`, `refactor:`, `test:`, `chore:`, `ci:`, `build:`, `perf:`, `style:`, `revert:` are all valid. They carry no SemVer weight unless combined with `BREAKING CHANGE`.

### Rule 15 — Case Insensitivity (Except `BREAKING CHANGE`)

> The units of information that make up Conventional Commits MUST NOT be treated as case-sensitive by implementors, with the exception of `BREAKING CHANGE` which MUST be uppercase.

`feat:`, `Feat:`, `FEAT:` are all valid to conforming tooling. Convention strongly prefers lowercase types. `BREAKING CHANGE` must be uppercase in footers.

### Rule 16 — `BREAKING-CHANGE` Synonym

> `BREAKING-CHANGE` MUST be synonymous with `BREAKING CHANGE`, when used as a token in a footer.

Both spellings are valid in footers. Prefer `BREAKING CHANGE` (with space) for readability.

## Examples Reference

**Minimal — type and description only:**
```
docs: correct spelling of CHANGELOG
```

**With scope:**
```
feat(lang): add Polish language
```

**Breaking change via footer:**
```
feat: allow provided config object to extend other configs

BREAKING CHANGE: `extends` key in config file is now used for extending other config files
```

**Breaking change via `!`:**
```
feat!: send an email to the customer when a product is shipped
```

**Both `!` and `BREAKING CHANGE` footer:**
```
feat!: drop support for Node 6

BREAKING CHANGE: use JavaScript features not available in Node 6.
```

**Full commit with body and multiple footers:**
```
fix: prevent racing of requests

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.

Remove timeouts which were used to mitigate the racing issue but are
obsolete now.

Reviewed-by: Z
Refs: #123
```

**Revert:**
```
revert: let us never again speak of the noodle incident

Refs: 676104e, a215868
```

## FAQ

**Should types be uppercase or lowercase?**
Any casing is accepted by Rule 15, but lowercase is the strong convention. Be consistent within a repository.

**What if the commit conforms to more than one type?**
Split into multiple commits. Atomic, single-purpose commits are the intent of the spec — they make changelogs accurate and revert operations safe.

**How do I handle squash-merge workflows?**
The lead maintainer provides the final conforming commit message on merge. Individual commits during development do not need to follow the spec as long as the squash merge commit does.

**How do I handle reverts?**
Use the `revert` type with a `Refs:` footer listing the SHA(s) being reverted:
```
revert: let us never again speak of the noodle incident

Refs: 676104e, a215868
```

**Can I use this in the initial development phase?**
Yes — treat it as if you have already released. Your fellow developers need to know what is fixed and what breaks.

**Does this discourage rapid iteration?**
It discourages disorganized speed. Conventional Commits enables fast long-term movement across multiple projects by making change history machine-readable and reviewer-friendly.

## Agent Guidance

### Do

- Default to `fix:` or `feat:` for any change that affects runtime behavior.
- Always include `BREAKING CHANGE` or `!` when removing or renaming public APIs, altering function signatures, or changing default behavior.
- When writing a body, focus on motivation and context — the diff already shows what changed.
- Use consistent scopes throughout a project; define a scope vocabulary in the project's contributing guide.

### Do Not

- Do not use `chore:` or `refactor:` as a catch-all to avoid thinking about the type.
- Do not omit `BREAKING CHANGE` to avoid a major version bump — that violates the spec and breaks consumers.
- Do not combine scope and `!` in non-standard order: `feat(api)!:` is valid; `feat!(api):` is not.

## Checklist

- [ ] Commit starts with `<type>[optional scope][optional !]: <description>`
- [ ] Type is one of the recognized types or a project-defined extension
- [ ] If backward-incompatible: `!` after type/scope OR `BREAKING CHANGE:` footer (or both)
- [ ] `BREAKING CHANGE` is uppercase in any footer
- [ ] Description is in imperative mood, no trailing period
- [ ] Body (if any) separated from description by exactly one blank line
- [ ] Footers (if any) use `Token: value` or `Token #value` format; multi-word tokens hyphenated

## See Also

- wiki/tier2-core/conventional-commits/overview.md
- wiki/tier1-sources/swebok-v4/ka08-config-management.md
- wiki/tier3-working/checklists/pre-commit.md
- wiki/tier1-sources/swebok-v4/ka06-operations.md
- wiki/tier2-core/twelve-factor-app/factors.md

## Source

Conventional Commits 1.0.0. conventionalcommits.org. CC BY 3.0.
https://www.conventionalcommits.org/en/v1.0.0/
