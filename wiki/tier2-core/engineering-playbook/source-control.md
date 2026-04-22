# Source Control Practices (Tier 2)

> **Tier 2** | Source: Microsoft ISE Engineering Fundamentals Playbook | Authority: established | Enforces: SWEBOK KA8

## Summary

Source control is the foundation of team-based software delivery. The playbook defines concrete practices for repository setup, branching, commit discipline, and pull request workflows that apply to any Git-based team. These practices enforce SWEBOK KA8's mandate for systematic change management by translating abstract policy into operational defaults a team can adopt on day one.

A coding agent touching any repository should verify that these practices are in place and follow them when producing commits, branches, and pull requests. Git's flexibility makes these decisions consequential: a poorly structured repository history is expensive to recover from and undermines traceability, rollback, and automated release tooling.

## Key Concepts

### Repository Setup Checklist (New Repositories)

| Action | Why |
|--------|-----|
| Agree on branch, release, and merge strategy before first commit | Retroactive strategy changes are disruptive |
| Lock the default branch (`main`) — no direct pushes | All changes flow through pull requests |
| Require pull request approval before merge | Enforces code review on every change |
| Define branch naming conventions | Enables automation and tooling |
| Configure branch protection and PR policies | Prevents accidental force-pushes and bypassed reviews |
| Add `LICENSE`, `README.md`, `CONTRIBUTING.md` to public repositories | Enables external contributors |

### Branch Naming Conventions

Consistent naming enables automation (CI triggers, changelogs, auto-labeling):

| Pattern | Example | Purpose |
|---------|---------|---------|
| `feature/<short-description>` | `feature/add-oauth-login` | New functionality |
| `fix/<short-description>` | `fix/null-check-validator` | Bug corrections |
| `chore/<short-description>` | `chore/upgrade-dependencies` | Maintenance, no behavior change |
| `release/<version>` | `release/2.4.0` | Release preparation branches |
| `hotfix/<short-description>` | `hotfix/csrf-token-missing` | Critical production fixes |

> Branch names: lowercase, hyphens not underscores, no special characters. Keep short but descriptive.

### Commit Best Practices

Git commits are the atomic unit of change history. Every commit should be independently meaningful.

| Rule | Detail |
|------|--------|
| **Small and logically grouped** | One concern per commit — avoid mixing whitespace fixes with logic changes |
| **Complete and tested** | Never commit broken code to a shared branch |
| **Descriptive subject line** | Max 50 characters; imperative mood: `Add OAuth2 PKCE flow`, not `Added oauth` |
| **Body when needed** | Separated by blank line, wrapped at 72 characters; explain *why*, not *what* |
| **Avoid binary files** | Use Git LFS for large assets; never commit generated files or build artifacts |

**Commit message structure:**
```
<subject line — 50 chars max, imperative mood>

<body — wrap at 72 chars, separated by blank line>
Explains the motivation and context for the change.
What problem does this solve? What was the alternative?

Refs: #123
```

> The playbook recommends Conventional Commits (`feat:`, `fix:`, `BREAKING CHANGE:`) as the structured format that links commits to SemVer. See `wiki/tier2-core/conventional-commits/overview.md`.

### Merge Strategies

| Strategy | When to Use | Trade-off |
|----------|-------------|-----------|
| **Merge commit** | Preserving full branch history matters | Noisier history; clearer where branches diverged |
| **Squash and merge** | PR history is messy; clean main branch matters | Loses granular commit history |
| **Rebase and merge** | Linear history required; small PRs | Rewrites history; requires discipline |

Pick one strategy per repository and enforce it — mixing creates an unreadable history.

### Pull Request Workflow

Every change to the default branch flows through a pull request. Four-stage workflow:

```
1. Implementation      → Develop in a feature branch
2. Pre-submission      → Verify: compiles, tests pass, linted, docs updated
3. PR creation         → Write description, link work item, assign reviewers
4. Code review         → Address feedback, approval, merge
```

**PR size guidance:**

Small PRs are faster to review, easier to deploy, and produce fewer merge conflicts. Target PRs reviewable in under 30 minutes.

| Technique | How |
|-----------|-----|
| Break large features into vertical slices | Each slice is independently deployable |
| Hide incomplete features behind feature flags | Merge code before the feature is "on" |
| Organize by architectural layer | UI, API, persistence as separate PRs |

**Every PR must:**
- [ ] Address a single objective (one feature or one fix — not loosely related changes)
- [ ] Not break the build (all CI checks green before review)
- [ ] Include tests for any changed behavior
- [ ] Have a meaningful description (what changed and why, not just a ticket number)

### Rolling Back Changes

| Operation | Command | Effect |
|-----------|---------|--------|
| Undo committed changes (safe) | `git revert <sha>` | Creates new commit that undoes — history preserved |
| Remove last commit (destructive) | `git reset --hard HEAD~1` | Deletes commit from history — only on private branches |
| Recover a lost commit | `git reflog` then `git checkout <sha>` | `reflog` tracks all HEAD movements for 90 days |
| Clean up commits before PR | `git rebase -i HEAD~N` | Interactive rebase to squash, reorder, or edit |

> Never force-push (`git push --force`) to a shared branch without team coordination. Prefer `--force-with-lease` which fails if the remote has updates you haven't seen.

### Secrets and Sensitive Data

- Never commit credentials, API keys, tokens, or passwords — even temporarily.
- Use `.gitignore` to exclude `.env` files; use `.env.template` to document required variables.
- Install `git-secrets` or `pre-commit` hooks to block accidental secret commits.
- If a secret is committed, treat the credential as compromised and rotate it immediately — git history is not erasable in practice.

## Agent Guidance

### Do

- Follow branch naming conventions for every branch created.
- Write commit messages with a 50-character subject in imperative mood; add a body when the motivation is not obvious from the diff.
- Keep PRs small and focused on one objective; split large changes across multiple PRs.
- Use `git revert` for safe rollback on shared branches; reserve `git reset --hard` for local/private branches only.
- Reference work items in PR descriptions and commit trailers (`Refs: #123`).
- Verify no secrets are staged before committing.

### Do Not

- Do not push directly to `main` or any protected branch.
- Do not commit generated files, build artifacts, or binary files that change frequently.
- Do not mix whitespace/formatting changes with logic changes in the same commit.
- Do not force-push to shared branches without explicit team coordination.
- Do not leave PR descriptions blank or populated only with ticket IDs.

## Checklist

- [ ] Branch name follows the team's naming convention
- [ ] Each commit is logically isolated (one concern per commit)
- [ ] Commit subject is 50 chars or fewer, imperative mood
- [ ] No secrets, credentials, or `.env` files staged
- [ ] PR addresses a single objective and all CI checks pass
- [ ] PR description explains what changed and why
- [ ] Tests included for any changed behavior

## See Also

- wiki/tier2-core/conventional-commits/overview.md
- wiki/tier2-core/engineering-playbook/overview.md
- wiki/tier1-sources/swebok-v4/ka08-config-management.md
- wiki/tier3-working/checklists/pre-commit.md
- wiki/tier3-working/checklists/code-review.md
- wiki/tier2-core/security-practices/python-pyscg.md

## Source

Microsoft ISE Engineering Fundamentals Playbook — Source Control.
https://microsoft.github.io/code-with-engineering-playbook/source-control/
https://microsoft.github.io/code-with-engineering-playbook/source-control/git-guidance/
https://microsoft.github.io/code-with-engineering-playbook/code-reviews/pull-requests/
