# Refactoring.Guru — Design Patterns in Python (catalog)

> Captured: 2026-11-09 from https://refactoring.guru/design-patterns/python
> Alexander Shvets, Refactoring.Guru, "Design Patterns in Python — The Catalog of Python Examples".
> The user listed this URL twice. It is one source.
> © Refactoring.Guru. Structured summary of the catalog page. Code-example pages were not fetched.
> Do not copy Refactoring.Guru text or examples into the wiki.

Read-only after ingest. Re-synthesize `wiki/tier3-working/software-design-practices/refactoring-guru-python.md` from this summary.

## What the page is

A language catalog. Each card has the pattern name, a one-sentence intent, and three links: the main article, "Usage in Python", and a code example. Those linked pages were not ingested. The intents below are paraphrased so the catalog can be mapped onto the existing GoF pages. The rules for when to use a pattern, and the Python-idiomatic substitutes, already live in `wiki/tier2-core/design-patterns/`.

## Creational

| Pattern | Intent on the catalog, paraphrased | Wiki page |
|---------|-------------------------------------|-----------|
| Abstract Factory | Produce a family of related objects without naming their concrete classes | `design-patterns/creational.md` |
| Builder | Build a complex object step by step, so one construction sequence can make different representations | `design-patterns/creational.md` |
| Factory Method | A superclass owns the creation interface; subclasses choose the concrete type | `design-patterns/creational.md` |
| Prototype | Copy an existing object without depending on its class | `design-patterns/creational.md` |
| Singleton | One instance, with a global way to reach it. The catalog links both a naive example and a thread-safe example | `design-patterns/creational.md` |

## Structural

| Pattern | Intent on the catalog, paraphrased | Wiki page |
|---------|-------------------------------------|-----------|
| Adapter | Let two incompatible interfaces collaborate | `design-patterns/structural.md` |
| Bridge | Split a large type into an abstraction hierarchy and an implementation hierarchy that change separately | `design-patterns/structural.md` |
| Composite | Treat a tree of objects like a single object | `design-patterns/structural.md` |
| Decorator | Add behavior by wrapping an object | `design-patterns/structural.md` |
| Facade | One simpler interface in front of a library or a set of classes | `design-patterns/structural.md` |
| Flyweight | Share the common part of object state so more objects fit in memory | `design-patterns/structural.md` |
| Proxy | A stand-in that runs something before or after the real object is reached | `design-patterns/structural.md` |

## Behavioral

| Pattern | Intent on the catalog, paraphrased | Wiki page |
|---------|-------------------------------------|-----------|
| Chain of Responsibility | Pass a request along handlers; each one handles it or forwards it | `design-patterns/behavioral.md` |
| Command | Turn a request into an object so it can be passed, queued, delayed, or undone | `design-patterns/behavioral.md` |
| Iterator | Walk a collection without exposing how it is stored | `design-patterns/behavioral.md` |
| Mediator | Objects talk through one mediator instead of to each other directly | `design-patterns/behavioral.md` |
| Memento | Save and restore an object's state without exposing its internals | `design-patterns/behavioral.md` |
| Observer | Subscribers are notified when the subject changes | `design-patterns/behavioral.md` |
| State | An object changes behavior when its internal state changes, as if its class changed | `design-patterns/behavioral.md` |
| Strategy | A family of interchangeable algorithms, each in its own type | `design-patterns/behavioral.md` |
| Template Method | The superclass fixes the skeleton; subclasses replace steps | `design-patterns/behavioral.md` |
| Visitor | Keep an algorithm separate from the objects it operates on | `design-patterns/behavioral.md` |

## Gaps and cautions

- Interpreter is on `wiki/tier2-core/design-patterns/behavioral.md` and was not on this catalog page. Absence from the catalog is not a reason to drop it from the wiki, and presence on the catalog is not a reason to add a pattern the Tier 2 page does not already justify.
- The catalog's Singleton card offers class-based naive and thread-safe forms. The Tier 2 page already warns that Python often does not need the Java-shaped Singleton. A module-level instance or an injected dependency comes first. Do not import a Singleton implementation from this catalog.
- "Usage in Python" and the code samples were not read. Do not describe them.
- Python substitutes already recorded on `design-patterns/overview.md` still apply: a callable for Strategy, a generator for Iterator, `@decorator` for the Decorator pattern, `singledispatch` for a simple Visitor, `Protocol` instead of a declared interface. Evaluate those before building the class hierarchy the catalog's Java-shaped articles describe.
