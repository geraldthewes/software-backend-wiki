# Documentation Practices (Tier 2)

> **Tier 2** | Source: Microsoft ISE Engineering Fundamentals Playbook | Authority: established | Derives From: SWEBOK KA12, KA9

## Summary

Agile methodology prioritizes working software over comprehensive documentation, but this does not mean documentation is optional. Key information must be documented systematically — the question is *which* information, *where*, and *when*. Poor documentation degrades onboarding velocity, increases knowledge silos, and makes project transitions expensive.

The playbook identifies eight recurring documentation failure modes and seven essential documentation types. For a coding agent, documentation is a first-class deliverable: a PR that adds a feature without updating the relevant documentation is not done by Definition of Done standards.

## Key Concepts

### The 8 Documentation Anti-Patterns

Recognizing failure modes is the first step to avoiding them.

| Anti-Pattern | Description | Agent Signal |
|-------------|-------------|-------------|
| **Non-existent** | Missing setup guides, contribution guidelines, or repository explanation | If none exists, create minimal stubs before adding features |
| **Hidden** | Useful information scattered across Confluence, Slack, personal notes, and wikis — no canonical location | Single source of truth; link from README |
| **Incomplete** | Missing critical details (env vars, auth steps, dependencies) or inconsistent standards | Incomplete > non-existent — but only barely. Always finish what you start |
| **Inaccurate** | Content that no longer matches the actual code or process | Worse than non-existent — readers trust and are misled. Update docs in the same PR as the code |
| **Obsolete** | Outdated materials kept alongside current documentation with no indication of age | Archive or delete superseded docs; date-stamp time-sensitive content |
| **Disorganized** | Poor navigation, no table of contents, documents ordered by date rather than subject | Structure for the reader's task, not the writer's memory |
| **Duplicate** | Multiple conflicting versions of single-source-of-truth information | One canonical location; all other mentions link to it |
| **Afterthought** | Documentation created after the feature ships, from memory | Write docs as part of implementation, not after — `docs:` commit in same PR |

### The 7 Essential Documentation Types

| Type | What It Contains | Where It Lives |
|------|-----------------|---------------|
| **Project / Repository** | Purpose, architecture overview, how to run locally, how to contribute | `README.md` (root) |
| **Commit Messages** | Why a change was made, what it fixes or adds | Git history (Conventional Commits format) |
| **Pull Request Descriptions** | What the PR changes, why, how to test it, linked work items | PR body in source control platform |
| **Code Documentation** | Public API contracts, non-obvious algorithms, module-level context | Docstrings, inline comments |
| **Work Item Descriptions** | Acceptance criteria, context, dependencies, expected behavior | Backlog tool (Azure DevOps, GitHub Issues, Jira) |
| **REST API Documentation** | Endpoints, request/response schemas, authentication, error codes | OpenAPI 3.1 spec (`openapi.yaml`) |
| **Engineering Feedback** | Insights, gaps, and friction points for process improvement | Designated feedback channel or tool |

### Documentation Quality Standards

Documentation has the same quality bar as code:

| Standard | Detail |
|----------|--------|
| **Version-controlled** | Documentation lives in the repository alongside the code it describes, not in a separate tool that can drift |
| **Reviewed** | Documentation changes go through pull request review — inaccuracies caught before merge |
| **Updated in the same PR** | If a PR changes behavior, its PR description and relevant docs update atomically |
| **Tested for accuracy** | Where possible, code examples in docs are tested (doctests, CI-verified snippets) |
| **Written for the reader's task** | Organize around "how do I…" not "here is what exists" |

### README Minimum Standard

Every repository `README.md` must include:

- [ ] One-sentence description of what the project does
- [ ] Prerequisites (runtime versions, tools, accounts)
- [ ] Local setup instructions (clone → running in ≤ 10 steps)
- [ ] How to run tests
- [ ] How to contribute (or link to `CONTRIBUTING.md`)
- [ ] License

### API Documentation Standard

All HTTP APIs must have an OpenAPI 3.1 specification. Minimum fields per endpoint:

- [ ] Summary and description
- [ ] Request parameters (path, query, headers) with types and constraints
- [ ] Request body schema with examples
- [ ] All possible response codes with schemas and examples
- [ ] Authentication requirements
- [ ] Error response format

### Tools

| Tool | Use Case |
|------|---------|
| `MkDocs` | Markdown-based static site for project documentation (Python ecosystem) |
| `DocFx` | .NET/cross-platform documentation generation from XML comments and Markdown |
| `Sphinx` | Python-standard API documentation from docstrings |
| OpenAPI / Swagger UI | Interactive REST API documentation from OpenAPI spec |

## Agent Guidance

### Do

- Update documentation in the same PR as the code change — never as a follow-up.
- Create a `README.md` for any new repository or service before writing the first feature.
- Write PR descriptions that explain *why* the change was made and *how to test it*, not just what files changed.
- Use Conventional Commits `docs:` type for documentation-only commits.
- Delete or archive documentation that is no longer accurate; inaccurate docs are worse than none.
- Prefer a single canonical source of truth; link to it rather than duplicating.

### Do Not

- Do not leave environment variables, configuration keys, or setup steps undocumented.
- Do not document the *what* (the code already says that) — document the *why* and *how*.
- Do not create documentation in personal notes, Slack, or email — it must be discoverable.
- Do not ship a new public API endpoint without an OpenAPI spec entry.

## Checklist

- [ ] `README.md` exists and covers prerequisites, setup, test, and contribution steps
- [ ] Changed behavior documented in the same PR (README, docstring, OpenAPI, or ADR as appropriate)
- [ ] PR description explains what changed and why, with test instructions
- [ ] No inaccurate or obsolete documentation left in place
- [ ] New environment variables documented in `.env.template`

## See Also

- wiki/tier2-core/engineering-playbook/overview.md
- wiki/tier2-core/engineering-playbook/developer-experience.md
- wiki/tier3-working/api-design/openapi.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md
- wiki/tier1-sources/swebok-v4/ka09-engineering-management.md
- wiki/tier3-working/checklists/pre-commit.md
- wiki/tier2-core/conventional-commits/overview.md

## Source

Microsoft ISE Engineering Fundamentals Playbook — Documentation.
https://microsoft.github.io/code-with-engineering-playbook/documentation/
