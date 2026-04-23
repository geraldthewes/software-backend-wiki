# Python Regular Expressions (Tier 3)

> **Tier 3** | Source: Python Regex HOWTO, docs.python.org/3/howto/regex.html | Enforces/Derives From: wiki/tier3-working/python/overview.md, wiki/tier2-core/security-practices/python-pyscg.md

## Summary

Python's `re` module implements Perl-compatible regular expressions. Patterns compiled to `re.Pattern` objects are reusable and faster in loops. The module provides methods for matching, searching, splitting, and substituting text. Regex is powerful but carries two principal risks: incorrect patterns silently match the wrong text, and pathologically crafted patterns can cause catastrophic backtracking (ReDoS), a real denial-of-service vector when patterns are applied to user-supplied input.

## Key Concepts

### Pattern Syntax Quick Reference

| Syntax | Meaning |
|--------|---------|
| `.` | Any character except newline (use `re.DOTALL` to include newline) |
| `^` / `$` | Start / end of string (or line with `re.MULTILINE`) |
| `\A` / `\Z` | Absolute start / end of string (unaffected by MULTILINE) |
| `\d`, `\w`, `\s` | Digit, word character, whitespace (Unicode-aware by default) |
| `\D`, `\W`, `\S` | Negations of the above |
| `\b` | Word boundary (zero-width assertion) |
| `*`, `+`, `?` | Greedy: 0+, 1+, 0 or 1 occurrences |
| `*?`, `+?`, `??` | Non-greedy (minimal) versions |
| `{m,n}` | Between m and n repetitions |
| `[abc]` | Character class — any of a, b, c |
| `[^abc]` | Negated class |
| `(...)` | Capturing group |
| `(?:...)` | Non-capturing group |
| `(?P<name>...)` | Named capturing group |
| `(?=...)` / `(?!...)` | Positive / negative lookahead |
| `(?<=...)` / `(?<!...)` | Positive / negative lookbehind |
| `\|` | Alternation (OR) |

### Core Methods

```python
import re

# Compile once, reuse many times
pattern = re.compile(r'\d{4}-\d{2}-\d{2}')

# match() — anchored at start only
m = pattern.match('2024-01-15 extra')   # matches
m = pattern.match('prefix 2024-01-15') # None — not at start

# search() — first match anywhere
m = pattern.search('date: 2024-01-15')  # matches at pos 6

# findall() — all non-overlapping matches as list of strings
dates = pattern.findall('2024-01-01 and 2024-06-30')  # ['2024-01-01', '2024-06-30']

# finditer() — iterator of match objects (preferred for large text)
for m in pattern.finditer(text):
    print(m.group(), m.span())

# sub() — replace matches
clean = re.sub(r'\s+', ' ', text)       # collapse whitespace
# sub() with function
result = re.sub(r'\d+', lambda m: str(int(m.group()) * 2), '1 + 2 = 3')

# split() — split by pattern
parts = re.split(r'[;,\s]+', 'one two,three;four')
```

### Groups

```python
p = re.compile(r'(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})')
m = p.search('date: 2024-03-15')
m.group('year')    # '2024'
m.group('month')   # '03'
m.groupdict()      # {'year': '2024', 'month': '03', 'day': '15'}
m.groups()         # ('2024', '03', '15')
```

### Compilation Flags

| Flag | Short | Effect |
|------|-------|--------|
| `re.IGNORECASE` | `re.I` | Case-insensitive |
| `re.MULTILINE` | `re.M` | `^`/`$` match line boundaries |
| `re.DOTALL` | `re.S` | `.` matches `\n` |
| `re.VERBOSE` | `re.X` | Allows inline comments and whitespace |
| `re.ASCII` | `re.A` | Restrict `\w`, `\d`, `\s` to ASCII only |

### Verbose Mode for Readability

```python
date_pattern = re.compile(r"""
    (?P<year>  \d{4}) -   # 4-digit year
    (?P<month> \d{2}) -   # 2-digit month
    (?P<day>   \d{2})     # 2-digit day
""", re.VERBOSE)
```

## Security: ReDoS

**Catastrophic backtracking** occurs when a regex engine explores an exponentially growing number of paths against a non-matching string. An attacker who can supply input to a regex can stall or crash the process.

Dangerous patterns exhibit nested quantifiers on overlapping character classes:

```python
# DANGEROUS — ReDoS risk on malicious input
re.match(r'(a+)+b', 'a' * 30 + 'c')          # exponential backtracking
re.match(r'(\w+\s*)+end', 'x ' * 30 + 'y')   # same class repeated

# SAFER alternatives
re.match(r'a+b', ...)           # no nested quantifier
re.match(r'(\w+)(\s+\w+)*end', ...)  # non-overlapping groups
```

Rules to avoid ReDoS:
1. Never nest quantifiers over the same character class: `(a+)+`, `(a*)*`, `(\w+\s*)+`.
2. Prefer possessive quantifiers or atomic groups if your regex engine supports them.
3. Limit input length before applying complex patterns to user-supplied text.
4. Test patterns with pathological non-matching inputs (e.g., long strings of repeated chars with a mismatching suffix).

## Agent Guidance

### Do

- Always use raw strings for patterns: `r'\d+'` not `'\\d+'`.
- Use `re.compile()` for any pattern used more than once, especially in loops.
- Prefer `re.search()` over adding `.*` prefix to a pattern used with `re.match()`.
- Use named groups `(?P<name>...)` in complex patterns — `m.group('name')` is more readable than `m.group(3)`.
- Use `re.VERBOSE` for patterns longer than one line to add inline documentation.
- Use `re.ASCII` flag when matching ASCII-only identifiers to avoid surprising Unicode matches.
- Prefer string methods (`str.replace`, `str.split`, `str.startswith`) when regex is not needed — they are faster and more readable.
- Gate regex operations on user input with an input-length check to limit backtracking exposure.

### Do Not

- Do not apply patterns with nested quantifiers (`(a+)+`, `(\w+\s*)+`) to user-supplied input without sanitizing or bounding its length.
- Do not use `re.match()` when you want to find a match anywhere in the string — use `re.search()`.
- Do not use `re.DOTALL` carelessly on multi-line input if `^` and `$` anchoring matters.
- Do not ignore the return value of `re.match()` / `re.search()` — they return `None` on no match; calling `.group()` on `None` raises `AttributeError`.
- Do not build regex patterns by string-formatting untrusted user input without `re.escape()`.

## Checklist

- [ ] All patterns use raw strings (`r'...'`)
- [ ] Patterns reused in loops are compiled with `re.compile()`
- [ ] No nested quantifiers on overlapping classes applied to user input
- [ ] Input length bounded before applying complex patterns to user-supplied data
- [ ] Untrusted input interpolated into patterns only via `re.escape()`
- [ ] Named groups used for multi-group patterns
- [ ] `None` return from `match()`/`search()` handled before calling `.group()`

## See Also

- wiki/tier3-working/python/overview.md
- wiki/tier2-core/security-practices/python-pyscg.md
- wiki/tier3-working/python/unicode.md
- wiki/tier1-sources/owasp/a03-injection.md
- wiki/tier1-sources/swebok-v4/ka13-security.md

## Source

Python Regular Expression HOWTO, docs.python.org/3/howto/regex.html. Python `re` module documentation.
