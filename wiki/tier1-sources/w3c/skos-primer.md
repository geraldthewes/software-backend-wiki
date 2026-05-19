# SKOS Simple Knowledge Organization System Primer (Tier 1)

> **Tier 1** | Source: W3C SKOS Primer | Authority: immutable

## Summary

The W3C SKOS Primer provides a user guide for representing concept schemes using the Simple Knowledge Organization System (SKOS). SKOS is an application of the Resource Description Framework (RDF) for expressing the basic structure and content of concept schemes such as thesauri, classification schemes, subject heading lists, taxonomies, and folksonomies. It enables concepts to be composed, published on the World Wide Web, linked with data on the Web, and integrated into other concept schemes.

## Key Concepts

### Core SKOS Vocabulary Elements

| Element | Purpose | Description |
|---------|---------|-------------|
| `skos:Concept` | Concept identification | Units of thought identified with URIs |
| `skos:prefLabel` | Preferred label | Strong, univocal denotation (one per language tag) |
| `skos:altLabel` | Alternative label | Synonyms, abbreviations, acronyms |
| `skos:hiddenLabel` | Hidden label | For text indexing/search but not display |
| `skos:broader` / `skos:narrower` | Hierarchical links | Broader/narrower concept relationships |
| `skos:related` | Associative links | Non-hierarchical concept relationships |
| `skos:note` | Documentation | Scope notes, definitions, examples, history |
| `skos:inScheme` | Concept scheme membership | Links concepts to concept schemes |
| `skos:hasTopConcept` | Top concepts | Entry points to broader/narrower hierarchies |

### Semantic Relationship Properties

| Property | Type | Use Case |
|----------|------|----------|
| `skos:broadMatch` | Mapping | Asserts broader meaning across schemes |
| `skos:narrowMatch` | Mapping | Asserts narrower meaning across schemes |
| `skos:closeMatch` | Mapping | Asserts similar meaning across schemes |
| `skos:exactMatch` | Mapping | Asserts equivalent meaning across schemes |
| `skos:relatedMatch` | Mapping | Asserts associative relationship across schemes |

### Conceptual Mappings

SKOS provides mapping properties to establish semantic links between concepts from different schemes:
- **Exact Match (`skos:exactMatch`)**: Equivalent meaning, transitive
- **Close Match (`skos:closeMatch`)**: Similar but not equivalent meaning, not transitive
- **Broad Match (`skos:broadMatch`)**: Concept A is broader than concept B
- **Narrow Match (`skos:narrowMatch`)**: Concept A is narrower than concept B
- **Related Match (`skos:relatedMatch`)**: Associative relationship between concepts

## Agent Guidance

### Do
- Use HTTP URIs when minting concept URIs for resolvability
- Follow KOS design guidelines (ISO2788/BS8723-2) when compiling SKOS concept schemes
- Ensure no two concepts have the same preferred lexical label in a given language within a scheme
- Use mapping properties (`skos:exactMatch`, `skos:closeMatch`, etc.) for cross-scheme concept alignment
- Leverage SKOS as a bridging technology between formal ontologies (OWL) and informal tagging systems

### Do Not
- Assume `skos:broader` or `skos:related` are transitive properties (they are not by default)
- Use `owl:sameAs` for cross-scheme equivalence (causes label conflicts)
- Assert mapping relationships within the same concept scheme (intended for cross-scheme use)
- Treat upward posting as best practice (introduce explicit concepts instead)

## Checklist
- [ ] Concepts identified with HTTP URIs and typed as `skos:Concept`
- [ ] Preferred labels assigned with `skos:prefLabel` (language-tagged when needed)
- [ ] Hierarchical relationships expressed with `skos:broader`/`skos:narrower`
- [ ] Associative relationships expressed with `skos:related`
- [ ] Concepts aggregated into schemes using `skos:inScheme`
- [ ] Top concepts identified with `skos:hasTopConcept`
- [ ] Cross-scheme mappings established with appropriate match properties
- [ ] Documentation provided via `skos:note` specializations when needed

## See Also
- wiki/tier1-sources/swebok-v4/ka02-architecture.md
- wiki/tier1-sources/swebok-v4/ka03-design.md
- wiki/tier2-core/architecture-patterns/overview.md
- wiki/tier3-working/api-design/overview.md
- wiki/tier3-working/observability/structured-logging.md
- wiki/tier2-core/design-patterns/overview.md
- wiki/tier2-core/testing-strategies/overview.md
- wiki/tier3-working/database-patterns/overview.md

## Source

W3C Semantic Web Deployment Working Group. *SKOS Simple Knowledge Organization System Primer*. W3C Working Group Note, 18 August 2009.
http://www.w3.org/TR/skos-primer