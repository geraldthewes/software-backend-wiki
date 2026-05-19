# .NET Microservices Architecture (Tier 3)

> **Tier 3** | Source: Microsoft Learn | Authority: working

## Summary
This guide introduces developing microservices-based applications and managing them using containers with .NET and Docker. It provides architectural design and implementation approaches, focusing on a reference containerized microservice-based application (eShopOnContainers). The guide is infrastructure-agnostic and development-environment-centric, intended for architects and developers new to Docker-based application development and microservices architecture.

## Key Concepts
| Concept | Description |
|---------|-------------|
| Microservices Architecture | Application built as collection of independently deployable services that can be developed, tested, deployed, and versioned independently |
| Containerization | Using Docker to bundle service and dependencies into isolated units for simplified deployment and testing |
| Reference Application | eShopOnContainers - open-source reference app showcasing architectural patterns (not production-ready) |
| Infrastructure Agnostic | Focus on development environment; infrastructure decisions made later for production-ready applications |
| .NET and Docker Integration | Primary technologies covered: .NET 7 (at time of writing) and Docker containers |

## Agent Guidance

### Do
- Consider microservices when team size, deployment cadence, domain boundary clarity, and operational maturity support service boundaries
- Use containers to encapsulate services and dependencies for consistent deployment across environments
- Focus on architectural patterns and implementation approaches rather than infrastructure specifics during initial design
- Explore the eShopOnContainers reference implementation to understand practical applications of concepts
- Apply Domain-Driven Design principles when defining service boundaries around business capabilities
- Design services to be independently deployable and scalable

### Do Not
- Treat this guide as a production-ready template for real-world applications
- Focus on Azure infrastructure details or specific orchestrators (covered in complementary guides)
- Neglect application lifecycle, DevOps, CI/CD pipelines, and team collaboration aspects (covered in Containerized Docker Application Lifecycle guide)
- Overlook that the reference application is intentionally in "permanent beta" for testing new technologies
- Assume this guide covers production-ready microservices on Microsoft Azure (separate learning path recommended)

## Checklist
- [ ] Understand the core concepts of microservices vs. monolithic architectures
- [ ] Review the eShopOnContainers reference application structure
- [ ] Identify bounded contexts in your domain for service decomposition
- [ ] Consider containerization benefits for your deployment scenarios
- [ ] Distinguish between development-focused guidance and production infrastructure concerns
- [ ] Plan for independent deployability, versioning, and scaling of services
- [ ] Review complementary guides for application lifecycle and Azure-specific guidance

## See Also
- wiki/tier1-sources/swebok-v4/ka02-architecture.md
- wiki/tier2-core/distributed-systems/overview.md
- wiki/tier2-core/architecture-patterns/overview.md
- wiki/tier3-working/api-design/overview.md
- wiki/tier3-working/database-patterns/overview.md
- wiki/tier3-working/observability/overview.md
- wiki/tier2-core/engineering-playbook/documentation-practices.md

## Source
Microsoft Developer Division, .NET and Visual Studio product teams. ".NET Microservices. Architecture for Containerized .NET Applications." Microsoft Learn. Edition v7.0. Updated 2024-02-15. https://learn.microsoft.com/en-us/dotnet/architecture/microservices/