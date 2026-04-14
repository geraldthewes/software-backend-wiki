# PEP 8: Style Guide for Python Code

> **Tier 1** | Source: PEP 8 (python.org) | Authority: immutable

## Summary

PEP 8 is the definitive style guide for Python code, authored by Guido van Rossum, Barry Warsaw, and Nick Coghlan. It defines layout, naming, and formatting conventions that make Python code consistent, readable, and maintainable across authors and tools. For a coding agent, PEP 8 compliance is a non-negotiable baseline — all produced Python code must pass a PEP 8 linter (`ruff check` or `flake8`) before being considered complete.

## Key Concepts

### Indentation

- Use **4 spaces per indentation level** — never tabs
- Mixing tabs and spaces causes `IndentationError` in Python 3 and silent bugs in Python 2
- **Continuation lines** — two acceptable styles:

```python
# Aligned with opening delimiter (implicit continuation)
result = some_function(argument_one, argument_two,
                       argument_three, argument_four)

# Hanging indent — 4 additional spaces, nothing on first line
result = some_function(
    argument_one, argument_two,
    argument_three, argument_four,
)
```

- Closing bracket on its own line at the same level as the opening line, OR at the first character of the last argument.

### Line Length

- Maximum **79 characters** for code
- Maximum **72 characters** for docstrings and comments (to allow for comment delimiters in diffs)
- Use **implicit line continuation** inside parentheses, brackets, or braces — preferred over backslash continuation:

```python
# RIGHT — implicit continuation inside parentheses
result = (
    first_term
    + second_term
    + third_term
)

# WRONG — backslash continuation (fragile, invisible trailing space breaks it)
result = first_term \
    + second_term
```

### Blank Lines

- **2 blank lines** before and after top-level function and class definitions
- **1 blank line** between method definitions inside a class
- Use blank lines within functions sparingly to separate logical sections
- Extra blank lines may be omitted between related one-liner methods

```python
class Foo:
    def method_one(self):
        ...

    def method_two(self):
        ...


class Bar:
    ...
```

### Imports

- **One import per line** — never `import os, sys`
- **Import order** (separated by blank lines):
  1. Standard library imports
  2. Third-party library imports
  3. Local application/library-specific imports
- **Absolute imports preferred** over relative imports for clarity
- **Avoid `from module import *`** — pollutes namespace, makes dependencies opaque
- Place all imports at the top of the file, after module docstring and before module globals

```python
# RIGHT
import os
import sys

import requests
import structlog

from myapp.models import User
from myapp.utils import format_date

# WRONG
import sys, os
from os import *
import requests; import structlog
```

### Whitespace

- **Spaces around operators**: `x = x + 1`, `y = y*2 + 1` (omit around `*` when mixing with `+` for precedence clarity)
- **No space** before a colon, semicolon, or comma: `a[1:4]`, `f(1, 2)`, `x = 1; y = 2`
- **No space** between function name and opening parenthesis: `func(arg)` not `func (arg)`
- **No trailing whitespace** on any line
- **Space after comma**: `a, b = 1, 2`
- **No space inside brackets**: `[1, 2, 3]` not `[ 1, 2, 3 ]`

```python
# RIGHT
spam(ham[1], {eggs: 2})
x = 1
y = 2
long_variable = 3

# WRONG
spam( ham[ 1], { eggs: 2} )
x             = 1
y             = 2
long_variable = 3
```

### Comments

- **Block comments**: same indentation level as surrounding code; each line starts with `# ` (hash + single space); complete sentences with period
- **Inline comments**: at least **2 spaces** before `#`; use sparingly; do not state the obvious
- **Docstrings**: use triple double-quotes `"""..."""`; one-liners on a single line; multi-line: summary on first line, blank line, elaboration

```python
# This is a block comment explaining the following section.
# It can span multiple lines.

x = x + 1  # Compensate for border

def calculate_area(radius: float) -> float:
    """Calculate the area of a circle.

    Args:
        radius: The radius of the circle in meters.

    Returns:
        The area in square meters.
    """
    return 3.14159 * radius ** 2
```

### Naming Conventions

| Entity                  | Convention                        | Example                        |
|-------------------------|-----------------------------------|--------------------------------|
| Functions               | `snake_case`                      | `get_user_by_id`               |
| Variables               | `snake_case`                      | `user_count`                   |
| Methods                 | `snake_case`                      | `calculate_total`              |
| Classes                 | `PascalCase` (CapWords)           | `UserRepository`               |
| Constants               | `UPPER_CASE_WITH_UNDERSCORES`     | `MAX_RETRY_COUNT`              |
| Non-public attributes   | `_leading_underscore`             | `_internal_cache`              |
| Name-mangled attributes | `__double_leading`                | `__private_field`              |
| Modules                 | `short_lowercase`                 | `utils`, `models`              |
| Packages                | `shortlowercase` (no underscores) | `mypackage`                    |

- Avoid single-character names except for loop counters and well-known conventions (`i`, `j`, `k` for integers; `e` for exceptions in `except` clauses)
- Avoid names that shadow built-ins: `list`, `type`, `id`, `input`, `filter`, `map`

### Other Conventions

**Comparisons to None — always use `is`/`is not`, never `==`:**

```python
# RIGHT
if value is None:
    ...
if result is not None:
    ...

# WRONG
if value == None:
    ...
```

**Comparisons to True/False — use truthiness, not equality:**

```python
# RIGHT
if items:      # True if list is non-empty
    ...
if not errors:
    ...

# WRONG
if items == True:
    ...
if len(items) > 0:    # acceptable but unnecessarily verbose
    ...
```

**Avoid mutable default arguments:**

```python
# WRONG — default list is shared across all calls; mutating it persists
def append_item(item, lst=[]):
    lst.append(item)
    return lst

# RIGHT
def append_item(item, lst=None):
    if lst is None:
        lst = []
    lst.append(item)
    return lst
```

**Use `isinstance()` for type checks, not `type()`:**

```python
# RIGHT
if isinstance(x, int):
    ...

# WRONG — does not work with subclasses
if type(x) == int:
    ...
```

## Agent Guidance

### Do
- Run `ruff check .` or `flake8 .` before finalizing any Python file
- Use `black` or `ruff format` for automatic formatting — eliminates whitespace decisions
- Configure `ruff` or `flake8` in `pyproject.toml` at the project root
- Apply naming conventions mechanically — no exceptions without documented justification

### Do Not
- Use tabs for indentation in any Python file
- Exceed 79 characters on any line of code
- Write `import os, sys` or any multi-import on one line
- Use `from module import *` in non-interactive code
- Write `if x == None:` or `if x == True:`
- Use mutable objects (lists, dicts) as default argument values

## Checklist
- [ ] All files pass `ruff check` with no errors or explicitly justified ignores
- [ ] Indentation uses 4 spaces throughout — no tabs
- [ ] No line exceeds 79 characters
- [ ] Imports ordered: stdlib, third-party, local; one per line; at top of file
- [ ] Naming follows conventions: snake_case functions/variables, PascalCase classes, UPPER_CASE constants
- [ ] No trailing whitespace on any line
- [ ] Docstrings present on all public modules, classes, and functions
- [ ] No mutable default arguments
- [ ] None comparisons use `is`/`is not`
- [ ] No shadowing of built-in names

## See Also
- wiki/tier1-sources/python-peps/overview.md
- wiki/tier1-sources/python-peps/pep-020-zen.md
- wiki/tier1-sources/python-peps/pep-484-type-hints.md

## Source
PEP 8 — Style Guide for Python Code. https://peps.python.org/pep-0008/
