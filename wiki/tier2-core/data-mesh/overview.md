# Data Mesh Principles and Logical Architecture (Tier 2)

> **Tier 2** | Source: "Data Mesh Principles and Logical Architecture" by Martin Fowler (martinfowler.com, 2020) | Authority: established | Derives From: DDD, domain-driven design, data architecture patterns

## Summary

Martin Fowler's article on Data Mesh Principles and Logical Architecture presents a paradigm shift in managing data at scale. Data mesh addresses the limitations of traditional data architectures by applying Domain-Driven Design principles to data platforms through four underpinning principles: domain-oriented decentralized data ownership and architecture, data as a product, self-serve data infrastructure as a platform, and federated computational governance.

This approach recognizes that data management challenges at scale are not just technical but also organizational, requiring a decentralized yet interoperable model where domain teams own their data products while adhering to global standards that enable seamless integration across the enterprise.

## Key Concepts

### The Four Underpinning Principles

| Principle | Description | Benefit |
|-----------|-------------|---------|
| **Domain-oriented decentralized data ownership and architecture** | Data ownership follows domain boundaries, with teams closest to the data owning and serving their analytical data as products | Enables scaling as data sources, use cases, and access models increase |
| **Data as a product** | Analytical data is treated as a product with discoverability, security, explorability, understandability, and trustworthiness | Ensures data users can easily discover, understand, and securely use high-quality data |
| **Self-serve data infrastructure as a platform** | Provides high-level abstractions that remove infrastructure complexity, enabling domain autonomy | Allows domain teams to build, deploy, and manage data products without specialized expertise |
| **Federated computational governance** | Balances global standardization for interoperability with domain autonomy for local decision-making | Creates a healthy ecosystem where independent data products can be combined for higher-value insights |

### Logical Architecture Components

#### Data Product (Architectural Quantum)
The fundamental building block of a data mesh, consisting of:
- **Code**: Data pipelines, APIs with semantic/syntax schema, and policy enforcement (access control, compliance, provenance)
- **Data and Metadata**: Analytical data in polyglot form (events, batch, relational, graph) with intrinsic and operational metadata
- **Infrastructure**: Storage, compute, and deployment capabilities for running the data product

#### Multi-plane Data Platform
- **Data infrastructure provisioning plane**: Low-level infrastructure lifecycle management
- **Data product developer experience plane**: Main interface for developers with self-service capabilities
- **Data mesh supervision plane**: Global capabilities like product discovery and semantic querying

#### Federated Governance Model
- **Global standardization**: Rules applied to all data products for interoperability (data sensitivity levels, regulatory compliance)
- **Domain autonomy**: Local decision-making on domain-specific concerns (data models, semantic definitions)
- **Automated implementation**: Platform-enforced policies reducing manual intervention

## Agent Guidance

### Do
- Apply DDD principles to data platform design by treating analytical data as domain-owned products
- Implement self-service capabilities that reduce infrastructure complexity for domain teams
- Establish federated governance that balances global standards with local autonomy
- Design data products as composable units that can be combined for higher-value insights
- Use domain boundaries (aligned with business units) as the foundation for data ownership

### Do Not
- Create centralized bottlenecks that limit domain autonomy and innovation
- Ignore the organizational aspects of data management (focus only on technical solutions)
- Treat data as a byproduct rather than a first-class product with quality requirements
- Implement rigid centralization that prevents adaptation to changing data landscapes
- Overlook the need for interoperability standards that enable cross-domain data utilization

## Checklist
- [ ] Data ownership aligned with domain/bounded context boundaries
- [ ] Data treated as a product with quality, discoverability, and usability measures
- [ ] Self-service platform capabilities reducing infrastructure complexity
- [ ] Federated governance model with automated policy enforcement
- [ ] Interoperability standards enabling cross-domain data combination
- [ ] Domain autonomy respected for local data model and semantic decisions

## See Also
- wiki/tier2-core/architecture-patterns/overview.md
- wiki/tier2-core/architecture-patterns/domain-model.md
- wiki/tier2-core/architecture-patterns/aggregates.md
- wiki/tier1-sources/martin-fowler/bounded-context.md
- wiki/tier1-sources/eric-evans/ddd-reference.md
- wiki/tier2-core/distributed-systems/cap-pacelc.md
- wiki/tier3-working/api-design/overview.md
- wiki/tier3-working/observability/overview.md

## Source

Martin Fowler. *Data Mesh Principles and Logical Architecture*. martinfowler.com, 3 December 2020.
https://martinfowler.com/articles/data-mesh-principles.html