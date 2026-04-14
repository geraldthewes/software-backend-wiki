# SOLID Principles — Overview

> **Tier 2** | Source: Robert C. Martin / Michael Feathers | Derives From: ka03-design | Authority: established practice

## Summary

SOLID is an acronym for five object-oriented design principles formulated by Robert C. Martin ("Uncle Bob") and named as an acronym by Michael Feathers. Together they guide practitioners toward code that is manageable, extensible, and testable. Applying SOLID reduces coupling between modules, makes individual classes easier to understand in isolation, and dramatically lowers the cost of change over time.

## Key Concepts

### The Five Principles at a Glance

| Acronym | Full Name | One-Line Rule | Python Implementation | Violation Smell |
|---------|-----------|---------------|----------------------|-----------------|
| **S** | Single Responsibility Principle | A class should have only one reason to change | Separate `@dataclass` (data) from repository class (persistence) | Class with more than one cohesive responsibility section |
| **O** | Open/Closed Principle | Open for extension, closed for modification | Use `Protocol` as extension points; add new implementations without touching existing code | Long `if-elif` chains that dispatch on type |
| **L** | Liskov Substitution Principle | Subtypes must be behaviorally substitutable for their base type | Never override a method to raise `NotImplementedError`; honor the parent contract | Subclass silently changes behavior or narrows accepted inputs |
| **I** | Interface Segregation Principle | No client should be forced to depend on methods it does not use | Define small `Protocol` classes with five methods or fewer | One giant `ABC` or `Protocol` covering every possible operation |
| **D** | Dependency Inversion Principle | Depend on abstractions, not concretions | Inject dependencies via constructor; domain layer defines `Protocol`; infra layer provides implementation | `DatabaseService()` instantiated directly inside a domain class |

### How the Principles Reinforce Each Other

The five principles are not independent rules — they form a mutually reinforcing system. Applying SRP naturally surfaces the need for DIP: once a class is split into focused units, those units must be wired together via abstractions rather than direct instantiation. OCP relies on LSP: extensions are only safe if all implementations honor the base contract. ISP keeps the abstractions introduced by DIP small enough to be practically mockable in tests, which in turn makes OCP extensions straightforward. Following all five principles consistently produces a codebase where any module can be replaced, extended, or tested without cascading changes to unrelated code.

## Agent Guidance

### Do
- Apply SRP first; it reveals where the other principles are needed
- Define `Protocol` classes in the domain layer to express abstractions (enables OCP and DIP)
- Keep each `Protocol` to five methods or fewer (ISP)
- Inject all dependencies through `__init__` parameters (DIP)
- Verify LSP compliance by asking: "Can I swap this implementation without changing any caller?"

### Do Not
- Do not combine data modeling and persistence logic in one class (SRP violation)
- Do not add a new `elif` branch for every new type — add a new implementation instead (OCP violation)
- Do not override methods to raise `NotImplementedError` in a subclass (LSP violation)
- Do not create one large `ABC` with every possible method (ISP violation)
- Do not call `SomeConcreteService()` inside a domain class constructor (DIP violation)

## Checklist
- [ ] Each class has a single, clearly stated responsibility
- [ ] Extension points use `Protocol` or `ABC`; new behavior is added by new classes, not new `elif` branches
- [ ] All subclasses can substitute their parent without callers needing to know the difference
- [ ] Each `Protocol` / `ABC` has five or fewer methods
- [ ] Dependencies are injected; domain layer imports no concrete infrastructure classes

## See Also
- `wiki/tier2-core/solid-principles/srp.md`
- `wiki/tier2-core/solid-principles/ocp.md`
- `wiki/tier2-core/solid-principles/lsp.md`
- `wiki/tier2-core/solid-principles/isp.md`
- `wiki/tier2-core/solid-principles/dip.md`
- `wiki/tier2-core/design-patterns/overview.md`
- `wiki/tier1-sources/swebok-v4/ka01-requirements.md`

## Source

Robert C. Martin, *Agile Software Development: Principles, Patterns, and Practices* (2002); Michael Feathers coined the SOLID acronym (2004). Synthesized from *Software Development Best Practices for Agent* reference document.
