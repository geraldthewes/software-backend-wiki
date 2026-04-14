# Design Patterns — Overview

> **Tier 2** | Source: Gang of Four (GoF), 1994 | Derives From: ka03-design | Authority: established practice

## Summary

Design patterns are reusable solutions to commonly occurring problems in software design. The canonical reference is *Design Patterns: Elements of Reusable Object-Oriented Software* (1994) by Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides — the "Gang of Four" (GoF). Patterns provide a shared vocabulary for communicating design decisions and encode hard-won engineering experience.

## Key Concepts

### Three Categories

| Category | Patterns | Primary Use |
|----------|----------|-------------|
| **Creational** | Factory Method, Abstract Factory, Builder, Singleton, Prototype | Controlling how objects are created |
| **Structural** | Adapter, Facade, Decorator, Proxy, Composite, Bridge, Flyweight | Composing classes and objects into larger structures |
| **Behavioral** | Strategy, Observer, Command, Template Method, Chain of Responsibility, Iterator, State, Visitor, Mediator, Memento, Interpreter | Managing communication and responsibility between objects |

### When to Use Patterns vs. Simple Code

Design patterns are tools, not requirements. The governing principle is: **use the simplest code that correctly solves the problem.**

| Situation | Apply a Pattern When... | Stay Simple When... |
|-----------|------------------------|---------------------|
| Extensibility needed | New behavior types will be added regularly | Only one or two behaviors exist and change is unlikely |
| Complexity justified | The abstraction is used in multiple places | The abstraction is used in exactly one place |
| Testing required | The pattern makes the code more testable | A simple function is already easy to test |
| Team vocabulary | The pattern name communicates the intent | Pattern name would confuse rather than clarify |

**"Premature patterns" are as harmful as premature optimization.** Adding a Factory hierarchy for a function that creates one type of object is over-engineering.

### Python-Specific Notes

Many GoF patterns are simpler in Python than in Java or C++ due to:

- **First-class functions**: Strategy pattern is often just a callable (function) rather than a class
- **Duck typing and Protocol**: Structural subtyping eliminates the need for explicit interface declarations
- **Decorators**: Python's `@decorator` syntax directly implements the Decorator pattern
- **Dynamic dispatch**: `singledispatch` implements the Visitor pattern trivially
- **`__dunder__` methods**: Iterator pattern via `__iter__`/`__next__` is built into the language

When a GoF pattern requires boilerplate in Java, check whether Python already provides an idiomatic equivalent before implementing the full class hierarchy.

### Sub-Pages

- **`creational.md`**: Factory Method, Abstract Factory, Builder, Singleton, Prototype — how to construct objects
- **`structural.md`**: Adapter, Facade, Decorator, Proxy, Composite, Bridge, Flyweight — how to compose objects
- **`behavioral.md`**: Strategy, Observer, Command, Template Method, Chain of Responsibility, Iterator, State — how objects communicate

## Agent Guidance

### Do
- Name patterns explicitly when using them: `# Strategy pattern: PaymentProcessor protocol` communicates intent
- Choose patterns based on the problem, not the pattern catalog — identify the problem first
- Use Python-idiomatic alternatives where they are simpler: a callable instead of a Strategy class, a generator instead of an Iterator class

### Do Not
- Do not introduce pattern abstraction when a simple function suffices
- Do not mix patterns unnecessarily — prefer the simplest composition
- Do not implement GoF patterns in Python exactly as described for Java — use Pythonic equivalents

## Checklist
- [ ] Pattern selection is justified by the problem being solved (extensibility, testability, communication)
- [ ] Pattern name is commented in code for team clarity
- [ ] Simpler alternatives were considered before introducing the pattern
- [ ] Python-idiomatic alternatives (callable, generator, decorator) were evaluated

## See Also
- `wiki/tier2-core/design-patterns/creational.md`
- `wiki/tier2-core/design-patterns/structural.md`
- `wiki/tier2-core/design-patterns/behavioral.md`
- `wiki/tier2-core/solid-principles/overview.md`
- `wiki/tier1-sources/swebok-v4/ka01-requirements.md`

## Source

Erich Gamma, Richard Helm, Ralph Johnson, John Vlissides, *Design Patterns: Elements of Reusable Object-Oriented Software* (1994). Synthesized from *Software Development Best Practices for Agent* reference document.
