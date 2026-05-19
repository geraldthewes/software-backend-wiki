# Data Mesh Principles and Logical Architecture by Martin Fowler

**Title**: Data Mesh Principles and Logical Architecture  
**Author**: Martin Fowler  
**Publisher**: martinfowler.com  
**Date**: 03 December 2020  
**URL**: https://martinfowler.com/articles/data-mesh-principles.html  

## Abstract

Data mesh addresses the challenges of managing data at scale by founding on four principles: domain-oriented decentralized data ownership and architecture, data as a product, self-serve data infrastructure as a platform, and federated computational governance. Each principle drives a new logical view of the technical architecture and organizational structure.

The article explains how data mesh creates a foundation for getting value from analytical data and historical facts at scale, addressing constant change in the data landscape, proliferation of data sources and consumers, diversity of transformation and processing requirements, and speed of response to change.

## Key Concepts

### Four Underpinning Principles

1. **Domain-oriented decentralized data ownership and architecture**: Teams closest to the data own and serve their analytical data as products, following domain boundaries.

2. **Data as a product**: Analytical data provided by domains must be treated as a product with discoverability, security, explorability, understandability, and trustworthiness.

3. **Self-serve data infrastructure as a platform**: Provides high-level abstractions that remove complexity and friction, allowing domain teams to autonomously own their data products.

4. **Federated computational governance**: Embracing decentralization and domain self-sovereignty while ensuring interoperability through global standardization and automated policy execution.

### Logical Architecture Components

- **Data Product**: The architectural quantum consisting of code (data pipelines, APIs, policies), data/metadata (analytical data in polyglot form with documentation), and infrastructure (storage, compute, deployment).
- **Multi-plane Data Platform**: Infrastructure provisioning plane, data product developer experience plane, and data mesh supervision plane.
- **Federated Governance Model**: Balance between global standardization (for interoperability) and domain autonomy (for local decision making).

## Relevance to Software Engineering

Data mesh represents the modern evolution of Domain-Driven Design (DDD) applied to data platforms, where:
- Individual teams own their data "domains" 
- Global interoperability standards (often Markdown/YAML ontologies) ensure seamless linking between domains
- The approach combines Tom Gruber's AI definition of an ontology with DDD principles
- It addresses vocabulary collisions and language silos across software development teams through standardized, product-oriented data

This architectural paradigm extends DDD beyond code into corporate knowledge management and data architecture, creating a decentralized yet interoperable ecosystem.