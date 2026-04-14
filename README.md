# Software Engineering Wiki for Coding Agents

A comprehensive, agent-navigable knowledge base grounded in professional software engineering standards. Built using the [Karpathy LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f): raw sources stay immutable, synthesized knowledge lives in a tiered directory structure, and a schema layer enables fast, structured navigation.

## Getting Started (for agents)

Read `AGENTS.md` first — it defines the authority tier system, task routing table, and navigation protocol. Then use `index.md` to find the pages relevant to your task, or `KNOWLEDGE_GRAPH.md` for named multi-step navigation paths.

## Structure

```
AGENTS.md              # Mandatory first read — protocol and task routing
index.md               # 200+ keyword searchable catalog
KNOWLEDGE_GRAPH.md     # Entity relationships and navigation paths
log.md                 # Append-only ingest history
references/            # Raw source documents (read-only)
wiki/
  tier1-sources/       # Immutable authority: SWEBOK v4, OWASP, PEPs, ethics
  tier2-core/          # Established practices: SOLID, 12-factor, patterns
  tier3-working/       # Language guides, checklists, worked examples
  tier4-archive/       # Superseded content
```

## Coverage

| Area | Pages |
|------|-------|
| SWEBOK V4 (all 18 knowledge areas) | 19 |
| OWASP Top 10:2021 | 11 |
| Python PEPs (8, 20, 484, 443) | 5 |
| ACM/IEEE-CS Code of Ethics | 1 |
| SOLID principles | 6 |
| 12-Factor App | 2 |
| Distributed systems (CAP/PACELC, fallacies, resilience patterns) | 4 |
| Testing strategies (pyramid, property-based, mutation) | 4 |
| GoF design patterns | 4 |
| Security practices (Pyscg, STRIDE, Zero Trust) | 4 |
| Python best practices | 5 |
| Go best practices | 4 |
| Database patterns | 4 |
| API design (REST, OpenAPI, gRPC) | 4 |
| Observability (logging, metrics, SLO/SLI) | 4 |
| Review checklists | 5 |
| Worked examples (executable Python code) | 3 |
| **Total** | **94** |

## Extending the Wiki

1. Place the new source document in `references/`
2. Create wiki page(s) in the appropriate `wiki/tierN-*/` directory following the standard template in `AGENTS.md`
3. Add entries to `index.md` (catalog table + keyword index rows)
4. Add relationships to `KNOWLEDGE_GRAPH.md`
5. Append an entry to `log.md`

## License

The wiki content in this repository is released under the [MIT License](LICENSE).

### Third-Party Copyright Notice

This wiki synthesizes and summarizes content from authoritative third-party sources. The copyright of the original material remains with the respective owners and authors:

- **SWEBOK V4** — © IEEE Computer Society. All rights reserved. [computer.org](https://www.computer.org/education/bodies-of-knowledge/software-engineering)
- **OWASP Top 10** — © OWASP Foundation. Licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). [owasp.org](https://owasp.org/Top10/)
- **Python PEPs** (PEP 8, 20, 484, 443) — © Python Software Foundation. Licensed under the [PSF License](https://docs.python.org/3/license.html). [python.org](https://www.python.org/dev/peps/)
- **ACM/IEEE-CS Software Engineering Code of Ethics** — © ACM / IEEE Computer Society. [acm.org](https://www.acm.org/code-of-ethics)
- **12-Factor App** — © Heroku. Licensed under [MIT](https://github.com/heroku/12factor/blob/master/LICENSE). [12factor.net](https://12factor.net/)
- **GoF Design Patterns** — © Erich Gamma, Richard Helm, Ralph Johnson, John Vlissides. Published by Addison-Wesley.
- **SOLID Principles** — © Robert C. Martin.
- **The 8 Fallacies of Distributed Computing** — © Peter Deutsch / Sun Microsystems.
- **CAP Theorem** — © Eric Brewer.
- **PACELC Theorem** — © Daniel Abadi.
- **Zero Trust Architecture (NIST SP 800-207)** — © National Institute of Standards and Technology (public domain).
- **Hypothesis (property-based testing)** — © David R. MacIver and contributors. Licensed under [MPL 2.0](https://github.com/HypothesisWorks/hypothesis/blob/master/LICENSE.txt).

The summaries and wiki pages in this repository are transformative works for educational purposes. No claim of ownership is made over the original standards, specifications, or publications referenced herein.
