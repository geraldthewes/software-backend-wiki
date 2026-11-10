# Hitchhiker's Guide to Python — Code Style

> Captured: 2026-11-09 from https://docs.python-guide.org/writing/style/
> Authors: Kenneth Reitz and the Real Python / Hitchhiker's Guide contributors.
> © 2011–2026 Kenneth Reitz & Real Python. Licensed CC BY-NC-SA 3.0.
> https://creativecommons.org/licenses/by-nc-sa/3.0/
> Structured summary for wiki ingest. Not a reproduction. Do not paste the guide back into the wiki.

Read-only after ingest. Re-synthesize `wiki/tier3-working/software-design-practices/hitchhikers-style.md` from this summary.

The guide's claim: Python is valued for readability, code is read more often than it is written, and "not Pythonic" means the code missed the common guidelines or failed to state its intent in the most readable way. Borderline cases with no agreed form are rare.

## General concepts

### Explicit code

Prefer the straightforward form. The guide's bad example packs `*args` into `locals()` and builds a dict by splatting that. The good example takes `x` and `y` and returns `{'x': x, 'y': y}`. The caller can see the contract from the signature and the return.

### One statement per line

A comprehension that is one idea is fine. Two unrelated statements on one line are not, including `print('one'); print('two')` and a one-line `if`. A compound condition that is hard to read should be split into named booleans, then combined.

### Function arguments

Four forms, and the simpler one wins:

1. Positional arguments are required and have no default. Use them when they are the meaning of the function and the order is obvious: `send(message, recipient)`, `point(x, y)`. Calling those by keyword is legal and rarely clearer.
2. Keyword arguments have defaults and mark optional parameters. Past two or three positional parameters, the signature is hard to remember, so further parameters should be keyword parameters with defaults: `send(message, to, cc=None, bcc=None)`. Call them in definition order and by name once they are optional: `send('Hello', 'World', cc='Cthulhu', bcc='God')`. The guide ties this to YAGNI: an optional argument added "just in case" is harder to remove later than it is to add when a real caller needs it.
3. `*args` is for a genuinely open list of positional values of one kind. If the values are a list the caller already has, take the sequence as one argument (`send(message, recipients)`). That accepts any sequence, including an iterator that cannot be unpacked.
4. `**kwargs` is for a genuinely open set of named arguments. Same caution: use it when the open set is the requirement, not as a bin for parameters the author did not want to name.

A function written this way is readable from its signature and can grow by a new keyword argument without breaking callers.

### Avoid the magical wand

Python can change how objects are created, how imports work, and can embed C. Those tools are real, and a developer should know they exist. They damage readability, and linters such as pylint and pyflakes often cannot follow them. Use the straightforward form. The guide's image is a kung fu master who knows the one-finger strike and does not use it.

Metaclasses, import hooks, and `**locals()` tricks need a specific reason. "It is possible" is not that reason.

### We are all responsible users

There is no `private` keyword. Client code can override attributes. The community convention, not a language wall, marks what callers must not use: a leading underscore on any method or attribute that is not part of the public contract. Breaking that mark makes the caller responsible for the breakage. Promoting a private name to public is easy. Hiding a name that was public is a breaking change. Use the underscore generously.

PEP 8 still distinguishes `_non_public` from `__name_mangled`. The guide's advice is the single leading underscore for "not for callers". Do not spell ordinary internals with `__` just because this section says "private".

### Returning values

Two kinds of exit: the normal result, and the error case (bad input, or the function cannot finish).

If the error case does not raise, return `None` or `False` as soon as the bad context is known. Code after that return may assume the good context. Several such early returns are normal.

The normal result should have one exit point. Several successful returns make the result hard to debug and usually mean the function should be split. The guide's sketch returns `None` early for missing `a` or `b`, computes `x`, may run a fallback if `x` is missing, and returns `x` once. It notes that raising would be better than returning `None` for those early cases. That note agrees with NSLS-II: library code raises.

## Idioms the guide teaches

These overlap `wiki/tier3-working/python/idioms.md`. Recorded here so the style page can be re-synthesized without the guide.

- Unpack when the length is known: `for index, item in enumerate(some_list)`, swap with `a, b = b, a`, and nested unpacking. PEP 3132 extended unpacking: `a, *rest = [1, 2, 3]` and `a, *middle, c = [1, 2, 3, 4]`.
- Throwaway targets. The guide uses `__` (`basename, __, ext = filename.rpartition('.')`) because `_` is the gettext alias and the REPL's last value. The guide itself says many style guides use `_`. PEP 8 requires neither. Match the file. Do not start a new `__` convention in a file that uses `_`.
- `[None] * 4` is a list of four Nones and is safe because `None` is immutable.
- A list of lists must be a comprehension: `[[] for __ in range(4)]`. `[ [] ] * 4` is four names for one list.
- Build a string with `''.join(parts)`, not repeated `+` in a loop. The guide shows the join. It does not show the loop anti-pattern; the reason to prefer join is the usual one.
- Membership tests that are large and repeated belong on a `set` or `dict`, which are hash tables. A `list` scan is linear. For a small collection, or one that is rarely searched, the hash table can cost more than it saves. Duplicates cannot live in a set.
- Truthiness instead of `== True`, `== None`, or `== 0`. Use `is None` when `None` must be distinguished from other false values. This matches PEP 8 for `None`.
- Dict membership is `key in d` or `d.get(key, default)`. `dict.has_key` is gone.
- A generator expression beats a list comprehension when the result is consumed once. A list comprehension is for a real list that will be reused. A generator function is for logic too big for one expression. A comprehension used only for a side effect (`[print(x) for x in sequence]`) is a loop.
- Do not remove items from a list while iterating it, and do not scan the list twice to delete. Build a new list or generator. Slice assignment (`sequence[:] = [...]`) can replace the contents in place when other names must see the change. That is the sharper tool, not the default.
- Do not mutate a list through one name when another name still refers to it, unless that alias is the point. Rebind the name to a new list.
- `enumerate` beats a manual index.
- Continue a long line with parentheses, brackets, or braces. A trailing backslash breaks if trailing whitespace is added. A line that needs splitting is often a sign the statement does too much.

## PEP 8 and formatters

The guide says PEP 8 is the community style document, points at pep8.org for a readable copy, and shows `pycodestyle` (formerly the `pep8` command) as the checker. It then names three formatters:

- autopep8 fixes PEP 8 violations. `--aggressive` does more and can be repeated.
- yapf formats beyond the PEP 8 minimum.
- black is opinionated and aims at one style with little configuration, so diffs stay about the change.

The wiki's current tool is ruff, which lints and formats (`wiki/tier3-working/python/overview.md`). Do not add black, yapf, or autopep8 beside ruff. `pycodestyle` as a separate program is redundant once ruff's pycodestyle rules are on.

The guide reprints the Zen of Python and points at PEP 20. The Zen page in this wiki is the authority for that text. Do not treat the guide's reprint as a second copy to maintain.

## Defects and age to preserve

- The ignored-variable recommendation (`__`) disagrees with common `_` practice. The guide admits that. Neither is a Tier 1 rule.
- Formatter advice predates ruff as the default in this wiki. ruff wins for new Python in this wiki.
- pylint and pyflakes are named as tools that fail on magic. They are examples of static analysis, not a required toolchain.
- The Zen text and the PEP 8 rules are not redefined here.
