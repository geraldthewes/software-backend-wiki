# Software Configuration Management (KA08)

> **Tier 1** | Source: SWEBOK V4, Chapter 8 | Authority: immutable

## Summary

Software Configuration Management (SCM) is the discipline of identifying, controlling, tracking, and auditing changes to all software artifacts throughout the development lifecycle. SCM ensures that the state of a software system at any point in time is known, reproducible, and traceable. Without SCM, teams cannot reliably answer: "What changed? When? By whom? Why?" — questions that are essential for debugging, compliance, rollback, and release management.

For agents, SCM defines the rules of engagement with source control: what to commit, when, how to describe changes, and how to structure branches. These are not preferences — they are engineering practices with direct impact on team coordination and system reliability. Every configuration change is a code change and must be treated with the same rigor.

## Key Concepts

### SCM Definition

SCM encompasses four core activities:
- **Identification**: Determining what artifacts are subject to configuration management (code, tests, configuration files, IaC, database migrations, documentation)
- **Control**: Managing changes through defined processes — no unreviewed changes reach the mainline
- **Status accounting**: Recording and reporting the state of artifacts over time (commit history, release notes, changelogs)
- **Auditing**: Verifying that the system built matches the expected configuration — ensuring reproducibility

### Version Control: Git Fundamentals

Git is the universal standard for source control. Key concepts every agent must understand:
- **Commit**: Atomic snapshot of changes with a message describing *why* the change was made
- **Branch**: Pointer to a commit that advances as new commits are added; enables parallel development
- **Merge/Rebase**: Integrating changes from one branch into another
- **Tag**: Immutable pointer to a specific commit, used for release versioning
- **Remote**: Shared repository (GitHub, GitLab, Gitea) that is the single source of truth

### Configuration Identification: What to Version Control

**Always version control**:
- All application source code
- All test code (unit, integration, end-to-end)
- Configuration files (non-secret values, feature flag defaults)
- Infrastructure as Code definitions (Terraform, Ansible playbooks, Nomad jobspecs)
- Database migration scripts
- Build scripts and CI/CD pipeline definitions
- Documentation

**Never version control**:
- Secrets (passwords, API keys, certificates) — use a secrets manager (Vault, AWS Secrets Manager)
- Generated artifacts (compiled binaries, build outputs) — these belong in an artifact registry
- Local developer environment overrides (`.env` files with real credentials)
- Large binary assets (use Git LFS or an artifact store)

### Branching Strategy

**Trunk-based development** (preferred for CI/CD):
- All developers commit directly to `main` or through very short-lived feature branches (< 1 day)
- Requires feature flags to hide incomplete work from users
- Enables true continuous integration — no long-lived branches to merge
- Minimizes merge conflicts and integration risk

**GitHub Flow** (lightweight, good for small teams):
- Short-lived feature branches off `main`
- Pull request → review → merge to main → deploy
- Every merge to main is potentially releasable

**GitFlow** (structured, suits scheduled release cycles):
- `main` (production), `develop` (integration), `feature/*`, `release/*`, `hotfix/*`
- More ceremony but provides clear release management for teams with scheduled releases
- Avoid if practicing continuous deployment — the extra branches add coordination cost

### Release Management

**Semantic Versioning (SemVer)**: `MAJOR.MINOR.PATCH`
- **PATCH**: Backward-compatible bug fix (1.2.3 → 1.2.4)
- **MINOR**: New backward-compatible feature (1.2.3 → 1.3.0)
- **MAJOR**: Breaking change (1.2.3 → 2.0.0)

**Release notes**: Human-readable summary of changes for each release. Generated from structured commit messages or manually authored for significant releases.

**Changelogs**: Cumulative record of changes across all versions. Format: [Keep a Changelog](https://keepachangelog.com/) is the standard.

### Build Reproducibility

A build is reproducible if the same source code always produces the same artifact:
- **Pin all dependency versions**: Use lock files (`requirements.txt` with hashes, `go.sum`, `package-lock.json`, `poetry.lock`). Never use floating version ranges (`>=1.0`) in production.
- **Pin base image versions**: Docker images referenced by digest (`image:tag@sha256:...`) not just tag
- **Deterministic builds**: Avoid timestamps, random seeds, or environment-specific values in build outputs
- **Artifact registry**: Store versioned build artifacts so past releases can be re-deployed without rebuilding

### Audit Trail: Commit Messages as Documentation

Every commit message is permanent engineering documentation. It must answer: "Why was this change made?"

**Conventional Commits format** (recommended):
```
<type>(<scope>): <short description>

<body: explain the why, not the what>

<footer: issue references, breaking change notices>
```

Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `ci`, `perf`

Example:
```
fix(auth): prevent session fixation on login

Session ID was not regenerated after authentication, allowing an
attacker with a pre-authentication session to elevate privileges.
Regenerate session on every successful login event.

Fixes: #1234
```

## Agent Guidance

### Do
- Commit early and often — small, focused commits are easier to review and revert
- Use Conventional Commits format for all commit messages
- Write commit messages that explain *why*, not just *what*
- Pin all dependency versions in lock files before merging
- Put every configuration change through the same code review process as application code
- Use short-lived branches (< 1 day) and merge frequently to avoid integration debt

### Do Not
- Commit secrets, credentials, or API keys — use a secrets manager
- Use floating dependency version ranges in production lock files
- Write commit messages like "fix bug" or "WIP" — these destroy the audit trail
- Commit generated build artifacts to source control
- Bypass code review for "small" or "urgent" changes — this is when most regressions are introduced
- Merge a branch that has been open for more than a few days without rebasing on main

## Checklist
- [ ] Lock files committed and all dependency versions pinned
- [ ] No secrets in version control (pre-commit hook scanning recommended)
- [ ] Conventional Commits format used for all commit messages
- [ ] Branch strategy documented and followed by the team
- [ ] Semantic versioning applied to all releases
- [ ] Release notes or changelog updated for every release
- [ ] CI/CD pipeline definition is version-controlled
- [ ] IaC definitions are version-controlled alongside application code

## See Also
- `wiki/tier1-sources/swebok-v4/ka06-operations.md`
- `wiki/tier1-sources/swebok-v4/ka04-construction.md`
- `wiki/tier2-core/twelve-factor-app/factors.md`

## Source
IEEE Computer Society. *Guide to the Software Engineering Body of Knowledge (SWEBOK), Version 4*. Chapter 8: Software Configuration Management. IEEE Press, 2024.
