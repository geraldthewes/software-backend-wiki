# Tier 4 — Archive

> **Tier 4** | Non-authoritative. Do not use archived content without explicit instruction from a human reviewer.

## What This Directory Is

The archive holds documentation that was once active in the wiki but has been retired. Archived content is preserved for historical reference but must not be cited as current guidance.

## Why Content Ends Up Here

Content is moved to the archive when it is:

- **Superseded**: a newer standard, library, or pattern has replaced it (e.g., an older API design guide superseded by the OpenAPI 3.1 page).
- **Deprecated technology**: the library, tool, or framework the document describes is no longer in use or recommended (e.g., documentation for a deprecated ORM, an obsolete deployment tool).
- **Contradicted by Tier 1**: the document's recommendations conflict with a subsequently adopted Tier 1 source (SWEBOK, OWASP, Python PEPs, ACM/IEEE ethics). The Tier 1 source always wins.
- **Organizationally superseded**: an ADR (Architecture Decision Record) or leadership decision explicitly replaces the content.

## How to Move Content Here

Moving content to the archive is a human decision. Agents must not archive content autonomously.

The process:
1. A human engineer identifies that a document meets one of the archival criteria above.
2. The engineer moves the file into `wiki/tier4-archive/` and prefixes the filename with the retirement date: e.g., `2025-06-01-old-deployment-guide.md`.
3. The engineer adds a deprecation header to the top of the moved file:

```markdown
> **ARCHIVED** on 2025-06-01. Superseded by wiki/tier3-working/....
> Do not follow this guidance without explicit human approval.
```

4. Any pages that linked to the archived document are updated to point to the replacement, or their links are removed.
5. The move is committed with a clear commit message explaining the reason for archival.

## How to Reference Archived Content

If a page in the active wiki needs to mention an archived document (e.g., to explain why a pattern was retired), use an explicit deprecation note:

```markdown
> **Deprecated**: The foo pattern described in wiki/tier4-archive/2025-06-01-foo.md
> was retired because it conflicts with OWASP A03. See
> wiki/tier3-working/database-patterns/repository-pattern.md for the current approach.
```

## Warning for Agents

**Do not use archived content to justify design decisions.**

If an agent retrieves an archived document during a search, the agent must:
1. Recognize the `ARCHIVED` header.
2. Ignore the document's recommendations.
3. Search for a replacement document in `wiki/tier3-working/` or `wiki/tier1-sources/`.
4. If no replacement exists, flag this to the human engineer rather than applying retired guidance.

Archived content is preserved only for audit and historical understanding — not for active use.
