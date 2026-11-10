# Hitchhiker's Guide — Code Style (Tier 3)

> **Tier 3** | Source: `references/software-design-practices/hitchhikers-guide-style.md` | Authority: working | Derives From: PEP 8, PEP 20, `wiki/tier3-working/python/idioms.md`

## Summary

The Hitchhiker's Guide style chapter is a readability guide: be explicit, keep one idea per line, design the signature so the simple call is obvious, and refuse magic that static tools cannot follow. It is © 2011–2026 Kenneth Reitz and Real Python, CC BY-NC-SA 3.0. This page is a summary. The guide's reprint of the Zen of Python is not a second authority. PEP 20 is.

Idioms that duplicate `python/idioms.md` are listed only where the guide adds a choice the idioms page does not settle. Formatter advice on the guide loses to ruff.

## Key Concepts

### Signatures

Positional parameters are the ones whose order is obvious and which the function cannot do without: `send(message, recipient)`. Optional parameters have defaults and are passed by name. Past two or three positional parameters, further parameters should be keyword parameters.

`*args` is for an open list of one kind of value. If the caller already has a sequence, take the sequence as one argument so iterators work. `**kwargs` is for an open set of names. Both are the wrong place to hide parameters the author could have named.

YAGNI, as the guide uses it: an optional argument added for a caller who does not exist is harder to remove than it is to add later.

### Control flow

Unrelated statements do not share a line. A compound condition that is hard to read becomes named booleans.

Error cases return or raise as soon as they are known, so the rest of the function can assume the good case. The successful result has one exit. The guide's own sketch says raising is better than returning `None` for bad input. Follow that note. It matches the NSLS-II page.

### Privacy and magic

There is no `private` keyword. A single leading underscore marks the name as not part of the contract. Promoting a private name is easy. Hiding a public name is a break. Use the underscore on anything callers must not depend on.

PEP 8 still reserves `__double_leading` for name mangling. Do not use it as a stronger "private".

Metaclasses, import hooks, and packing `locals()` into a dict are available and usually the wrong tool. Use them only when a plain function cannot express the requirement. Linters often cannot see through them.

### Idioms the guide adds or qualifies

- Extended unpacking (`a, *rest = seq`, PEP 3132) is ordinary Python 3.
- The guide prefers `__` as a throwaway name because `_` is gettext and the REPL's last result. It admits many guides use `_`. Match the file.
- `[None] * n` is safe. `[[]] * n` is not. Build a list of lists with a comprehension.
- A large collection that is searched often should be a `set` or a `dict`. A short list that is scanned rarely should stay a list.
- Build strings with `''.join`.
- Do not delete from a list while iterating it. Build a new list. Slice assignment is for the case where other names must observe the change.
- Continue lines with parentheses, not a trailing backslash.
- `dict.has_key` is gone. Use `key in d` or `d.get`.
- Do not compare to `True` or `None` with `==`. Use truthiness, and `is None` when `None` is distinct from other false values.

### Tools

The guide demonstrates `pycodestyle` and names autopep8, yapf, and black. New Python in this wiki uses ruff to lint and format, and mypy for types. Do not add those formatters beside ruff.

## Agent Guidance

### Do

- Make the signature readable without a comment: required positional parameters first, then keyword parameters with defaults.
- Return or raise at the point the input is known to be unusable. Return the success value from one place.
- Prefix non-public attributes with `_`.
- Use a set or dict for repeated membership tests on a large collection.

### Do Not

- Do not add `*args` or `**kwargs` to avoid naming a parameter.
- Do not reach for a metaclass or an import hook as a first design.
- Do not spell ordinary internals with `__`.
- Do not change a file's throwaway name from `_` to `__` because this guide prefers `__`.
- Do not install black, yapf, or autopep8 for a repository that uses ruff.
- Do not copy the guide's Zen text into a new page. Link PEP 20.

## Checklist

- [ ] Required arguments are positional only when their order is obvious
- [ ] Optional arguments are keyword parameters, added because a caller needs them
- [ ] `*args` and `**kwargs` represent an open set, not unnamed known parameters
- [ ] Invalid input leaves the function at the check; the success value has one return
- [ ] Non-public names use a single leading underscore
- [ ] No metaclass, import hook, or `locals()` splat was added without a requirement that a plain function cannot meet

## See Also

- wiki/tier3-working/software-design-practices/overview.md
- wiki/tier1-sources/python-peps/pep-008-style.md
- wiki/tier1-sources/python-peps/pep-020-zen.md
- wiki/tier3-working/python/idioms.md
- wiki/tier3-working/python/overview.md
- wiki/tier3-working/software-design-practices/ist-python-guide.md
- wiki/tier3-working/software-design-practices/nsls-ii-principles.md
- wiki/tier3-working/python-testing/hitchhikers-guide.md

## Source

Kenneth Reitz and Real Python, "Code Style", The Hitchhiker's Guide to Python, © 2011–2026, CC BY-NC-SA 3.0, https://docs.python-guide.org/writing/style/. Summary in `references/software-design-practices/hitchhikers-guide-style.md`.
