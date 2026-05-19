# Database Design Best Practices

> **Tier 3** | Source: Synthesized from Redgate, Microsoft, AWS, MongoDB, Neo4j, InfluxData, TimescaleDB | Authority: Working

## Summary

This page synthesizes database design best practices across relational, NoSQL, graph, time-series, and object storage systems. It emphasizes modeling for specific access patterns, proper normalization/denormalization strategies, indexing considerations, and avoiding common pitfalls. The core principle is to design databases around how the application will actually query and update data, rather than forcing a one-size-fits-all approach.

## Key Concepts

### Cross-Cutting Principles
| Principle | Description |
|-----------|-------------|
| **Access Pattern First** | Always start by listing your top 5-10 queries/access patterns before choosing a model or technology |
| **Model for Queries** | Design the database to make your most important operations fast and cheap at scale |
| **Iterative Refinement** | Database models are highly refactorable — embrace iteration based on real-world usage |
| **Document Everything** | Maintain up-to-date ERDs, data dictionaries, and model versions |

### Relational/SQL Databases
| Aspect | Best Practice |
|--------|---------------|
| **Normalization** | Aim for 3NF in transactional systems (OLTP); denormalize for dimensional/BI models |
| **Keys** | Every table needs a Primary Key; use surrogate keys thoughtfully (not blindly) |
| **Referential Integrity** | Define Foreign Keys to enforce relationships |
| **Data Types** | Choose precise data types and sizes (avoid generic varchar(255)) |
| **Indexing** | Plan indexes during design (on PK/FK + high-cardinality filter columns) |
| **Partitioning** | Partition large schemas by subject, update frequency, or application |
| **Documentation** | Create Conceptual → Logical → Physical diagrams; keep data dictionary updated |

### NoSQL/Document Databases
| Aspect | Best Practice |
|--------|---------------|
| **Access Patterns** | Start with queries the application will run, not data entities |
| **Data Locality** | Keep related data together; use embedding for "one-to-few" relationships |
| **Embed vs Reference** | Embed when data is read together and updated through parent; reference when data is large, updated independently, or accessed separately |
| **Schema Validation** | Leverage flexible schema but apply validation where needed |
| **Indexing** | Index frequently queried fields; avoid ignoring indexing until performance issues appear |
| **Document Size** | Plan for document size limits (e.g., 16 MB in MongoDB) |
| **Avoid** | Modeling like relational databases; unbounded array growth; treating as completely schemaless |

### Graph Databases
| Aspect | Best Practice |
|--------|---------------|
| **Query-Driven** | Write queries first — the questions you need to answer drive the model |
| **Relationships as Verbs** | Model verbs as relationships (not nodes) unless the event has independent identity/properties |
| **Specific Relationship Types** | Use specific relationship types; avoid generic types with properties |
| **Avoid Super-Nodes** | Prevent nodes with millions of same-type relationships |
| **Indexed Traversals** | Anchor traversals on indexed properties |
| **Iterative Approach** | Test early with real data volumes; embrace model refactoring |

### Time-Series Databases
| Aspect | Best Practice |
|--------|---------------|
| **Time as Primary Dimension** | Design partitioning/chunking around time ranges |
| **Tags vs Fields** | Use tags (indexed, low-cardinality) vs fields (not indexed) wisely |
| **Chunking Strategy** | In TimescaleDB: Use hypertables with appropriate chunk intervals (balance insert/query performance) |
| **Retention Policies** | Implement retention policies and continuous aggregates/downsampling aggressively |
| **Indexing** | Index on time + other high-cardinality dimensions used in filters |
| **Single vs Multiple Tables** | Single hypertable is usually simpler unless you have very different access patterns or retention needs |

### Object Storage (S3, GCS, Azure Blob, MinIO)
| Aspect | Best Practice |
|--------|---------------|
| **Key Design** | Object key (prefix) design is critical for performance and parallelism |
| **Prefix Distribution** | Spread load across many prefixes to achieve high request rates (use date-based, hash-based, or tenant-based prefixes) |
| **Avoid Hot Prefixes** | Prevent concentration of requests on single prefixes (e.g., everything starting with a date) |
| **Data Lake Pattern** | Use Hive-style partitioning (year=2026/month=05/...) for query engines |
| **Lifecycle Management** | Combine with lifecycle policies, versioning, and intelligent tiering |
| **Bucket Organization** | Separate buckets or prefixes by environment/layer (raw, processed, archive) |

## Common Pitfalls by Database Type

| Database Type | Common Mistakes |
|---------------|-----------------|
| **Relational/SQL** | Over-normalization, ignoring indexes until performance problems appear, using natural keys that can change, skipping early modeling phases |
| **NoSQL/Document** | Modeling like SQL (unnecessary joins/heavy normalization), letting arrays grow unbounded, ignoring indexing, treating as completely schemaless |
| **Graph** | Ignoring query patterns, creating super-nodes, using generic relationship types, not testing with real data volumes |
| **Time-Series** | No time-based partitioning, treating time-series data like regular relational rows, ignoring compression opportunities |
| **Object Storage** | Hot prefixes/poor key design, lack of partitioning strategy, ignoring eventual consistency in some operations |

## Dos and Don'ts Summary

### Do
- Normalize for OLTP (reduce anomalies); denormalize for BI/read-heavy workloads
- Enforce constraints (PK/FK/CHECK/NOT NULL) in relational systems
- Model for specific access patterns first in all database types
- Choose appropriate data types and sizes
- Index strategically based on query patterns
- Document everything (ERDs, data dictionary, model versions)
- Plan for evolution (versioning, migrations, iterative refinement)

### Don't
- Over-denormalize in transactional systems
- Use natural keys that can change as primary keys
- Ignore indexing until performance problems appear
- Skip conceptual/logical modeling phases
- Model NoSQL/graph/time-series like relational databases
- Let arrays/documents grow without bounds in NoSQL systems
- Create super-nodes in graph databases
- Use hot prefixes in object storage
- Treat time-series data without time-based partitioning

## See Also
- wiki/tier3-working/database-patterns/repository-pattern.md
- wiki/tier3-working/database-patterns/migrations.md
- wiki/tier3-working/database-patterns/query-optimization.md
- wiki/tier1-sources/owasp/a03-injection.md
- wiki/tier2-core/solid-principles/dip.md
- wiki/tier3-working/api-design/overview.md
- wiki/tier3-working/observability/overview.md

## Source
Synthesized from:
- Redgate: Top 11 Best Practices for Database Design (2023)
- Microsoft: Database Design Basics
- AWS DynamoDB: NoSQL Design Best Practices
- MongoDB: Data Modeling and Schema Design Patterns
- Neo4j: Graph Data Modeling Tips & Best Practices
- InfluxData: Time Series Database Explained + Data Model
- TimescaleDB: Best Practices for Time-Series Data Modeling & Hypertables
- AWS S3: Optimizing Performance (Object Key Design & Partitioning)