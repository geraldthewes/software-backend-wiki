# Refactoring.Guru Python Catalog (Tier 3)

> **Tier 3** | Source: `references/software-design-practices/refactoring-guru-python-catalog.md` | Authority: working | Derives From: `wiki/tier2-core/design-patterns/overview.md`

## Summary

https://refactoring.guru/design-patterns/python is Alexander Shvets's catalog of GoF patterns with Python example links. The user listed that URL twice. It is one source. The catalog page was summarized. The "Usage in Python" pages and the code examples were not fetched. © Refactoring.Guru. Do not copy those examples.

Pattern rules, including when not to use one, stay on the Tier 2 pages. This page is the map from the catalog's names onto those pages, plus the Python substitutes the Tier 2 overview already prefers.

## Key Concepts

### Catalog map

Creational, all on `wiki/tier2-core/design-patterns/creational.md`: Abstract Factory, Builder, Factory Method, Prototype, Singleton.

Structural, all on `wiki/tier2-core/design-patterns/structural.md`: Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy.

Behavioral, all on `wiki/tier2-core/design-patterns/behavioral.md`: Chain of Responsibility, Command, Iterator, Mediator, Memento, Observer, State, Strategy, Template Method, Visitor.

Interpreter is on the behavioral page and was not on the catalog. Keep it. Do not add a pattern because a future visit to the catalog might list it, and do not remove Interpreter because this capture did not.

### Python forms to try first

These are the Tier 2 rules, repeated so a reader of the catalog does not implement the Java shape by default:

| Catalog pattern | Try first |
|-----------------|-----------|
| Strategy | A callable or a `Protocol` with one method |
| Iterator | A generator, or `__iter__` / `__next__` |
| Decorator | A function decorator. The GoF object wrapper is for when the function form cannot carry the state |
| Visitor | `functools.singledispatch` when one new operation over a fixed set of types is enough |
| Factory Method | A function that returns the object |
| Singleton | A module-level instance, or an object injected from the composition root. The catalog's naive and thread-safe class examples were not ingested and are not the default |
| Adapter | A small function that translates one call into another, until a type is actually required |

A class hierarchy is justified when there is more than one variant, the variant must carry state, and a callable or a generator cannot express it. Name the pattern in a short comment when you do use it.

## Agent Guidance

### Do

- Identify the problem, then open the Tier 2 page for that pattern.
- Compare the Python form in the table before writing a family of classes.
- Keep Interpreter available even though this catalog page did not list it.

### Do Not

- Do not copy a Refactoring.Guru example into the repository or into this wiki.
- Do not describe the "Usage in Python" sections. They were not read.
- Do not add a Singleton class because the catalog offers a thread-safe one.
- Do not treat the catalog as higher authority than `design-patterns/overview.md`.

## Checklist

- [ ] The pattern's Tier 2 page was read before the catalog was used as a menu
- [ ] The simpler Python form in the table was considered and rejected for a stated reason
- [ ] No code from Refactoring.Guru appears in the change
- [ ] Singleton, if present, is a module or an injected object, unless a written reason requires a class

## See Also

- wiki/tier3-working/software-design-practices/overview.md
- wiki/tier2-core/design-patterns/overview.md
- wiki/tier2-core/design-patterns/creational.md
- wiki/tier2-core/design-patterns/structural.md
- wiki/tier2-core/design-patterns/behavioral.md
- wiki/tier2-core/solid-principles/ocp.md
- wiki/tier1-sources/python-peps/pep-443-singledispatch.md
- wiki/tier3-working/software-design-practices/manning-mak.md

## Source

Alexander Shvets, Refactoring.Guru, "Design Patterns in Python", catalog page captured 2026-11-09, https://refactoring.guru/design-patterns/python. © Refactoring.Guru. Code examples were not ingested. Summary in `references/software-design-practices/refactoring-guru-python-catalog.md`. The pattern intents and the Python substitutes derive from `wiki/tier2-core/design-patterns/`, which cites Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides, *Design Patterns* (1994).
