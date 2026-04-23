# Python Unicode Handling (Tier 3)

> **Tier 3** | Source: Python Unicode HOWTO, docs.python.org/3/howto/unicode.html | Enforces/Derives From: wiki/tier3-working/python/overview.md, wiki/tier1-sources/owasp/a02-cryptographic-failures.md, wiki/tier2-core/security-practices/python-pyscg.md

## Summary

Python 3 uses Unicode (`str`) for all text by default. The `str` type holds Unicode code points; `bytes` holds raw byte sequences. The fundamental skill is knowing when and how to decode bytes into strings (at the boundary where data enters the program) and encode strings back to bytes (at the boundary where data leaves). Failure to apply the correct encoding causes `UnicodeDecodeError`, `UnicodeEncodeError`, data corruption, or — in security contexts — encoding-based injection attacks.

## Key Concepts

### The Golden Rule

> Software should work with Unicode strings internally, decode input as soon as possible, and encode output only at the end.

```
bytes (from network/file/DB)
    ↓  decode(encoding)
str  (internal processing)
    ↓  encode(encoding)
bytes (to network/file/DB)
```

### str vs bytes

```python
# str: sequence of Unicode code points
greeting = "Héllo"
len(greeting)                   # 5 characters

# bytes: sequence of octets
encoded = greeting.encode('utf-8')   # b'H\xc3\xa9llo'
len(encoded)                         # 6 bytes — 'é' is 2 bytes in UTF-8

# Round-trip
encoded.decode('utf-8') == greeting  # True
```

### Encoding and Error Handling

```python
# encode with different error modes
text = "café"
text.encode('ascii', errors='strict')           # UnicodeEncodeError
text.encode('ascii', errors='ignore')           # b'caf'
text.encode('ascii', errors='replace')          # b'caf?'
text.encode('ascii', errors='xmlcharrefreplace') # b'caf&#233;'
text.encode('ascii', errors='backslashreplace') # b'caf\\xe9'

# decode with error modes
b'\x80abc'.decode('utf-8', errors='strict')      # UnicodeDecodeError
b'\x80abc'.decode('utf-8', errors='replace')     # '�abc'  (replacement char)
b'\x80abc'.decode('utf-8', errors='ignore')      # 'abc'
b'\x80abc'.decode('utf-8', errors='surrogateescape')  # '\udcc0abc'  (lossless)
```

### File I/O

Always specify encoding explicitly when opening text files:

```python
# Reading
with open('data.txt', encoding='utf-8') as f:
    text = f.read()

# Writing
with open('output.txt', 'w', encoding='utf-8') as f:
    f.write("Unicode content: café\n")

# Binary mode — handle encoding manually
with open('data.bin', 'rb') as f:
    raw = f.read()
    text = raw.decode('utf-8', errors='replace')
```

### Unicode Normalization

The same visual character can have multiple code-point representations. Without normalization, string comparisons fail:

```python
import unicodedata

# 'é' as a single code point (U+00E9)
single = 'é'
# 'é' as 'e' + combining accent (U+0065 + U+0302)
composed = 'é'

single == composed          # False — same appearance, different bytes
len(single)                 # 1
len(composed)               # 2

# Normalize before comparing
def normalize(s: str) -> str:
    return unicodedata.normalize('NFC', s)

normalize(single) == normalize(composed)   # True
```

**Normalization forms:**

| Form | Description | Use When |
|------|-------------|---------|
| `NFC` | Canonical Decomposition + Composition | Default for storage and comparison |
| `NFD` | Canonical Decomposition only | Sorting, diacritic stripping |
| `NFKC` | Compatibility Decomposition + Composition | Identifiers, search |
| `NFKD` | Compatibility Decomposition only | Aggressive normalization |

### Case-Insensitive Comparison

```python
import unicodedata

def caseless_equal(s1: str, s2: str) -> str:
    """Unicode-correct case-insensitive comparison."""
    def nfd_casefold(s: str) -> str:
        return unicodedata.normalize('NFD', s.casefold())
    return nfd_casefold(s1) == nfd_casefold(s2)

# German sharp-s
caseless_equal("Straße", "STRASSE")   # True
```

### Regular Expressions and Unicode

```python
import re

# By default, \w, \d, \s match Unicode categories
re.findall(r'\w+', 'café résumé')    # ['café', 'résumé']

# Restrict to ASCII
re.findall(r'\w+', 'café résumé', re.ASCII)   # ['caf', 'r', 'sum', 'e']
```

### Byte Order Mark (BOM)

```python
# UTF-16 files include a BOM; use 'utf-16' codec to auto-detect
with open('file.txt', encoding='utf-16') as f:
    text = f.read()

# UTF-8 BOM (rare but exists in Windows files)
with open('file.txt', encoding='utf-8-sig') as f:    # strips BOM if present
    text = f.read()
```

## Security Considerations

Unicode encoding can be exploited to bypass security controls. These issues are related to the broader category of encoding-based injection attacks documented in `wiki/tier1-sources/owasp/a02-cryptographic-failures.md`:

1. **Normalization before validation**: Always normalize Unicode text before performing security checks. An attacker can use alternate code-point sequences to evade filters if validation runs on un-normalized text.
2. **Validate decoded strings, not bytes**: After decoding, validate the resulting `str` — not the raw `bytes`. Byte-level validation can be bypassed with encoding tricks.
3. **Confusable characters (homograph attacks)**: Unicode contains many visually similar characters (e.g., Cyrillic `а` vs Latin `a`). Use NFKC normalization and ASCII-only identifiers in security-sensitive contexts such as usernames and domain names.
4. **Null bytes in strings**: `'\x00'` is valid in Python `str` but may truncate C-library string operations. Strip or reject null bytes in strings passed to file paths or system calls.

```python
# Normalize before security-sensitive comparison or storage
import unicodedata

def safe_username(raw: str) -> str:
    normalized = unicodedata.normalize('NFKC', raw)
    if '\x00' in normalized:
        raise ValueError("Username contains null byte")
    return normalized
```

## Agent Guidance

### Do

- Open text files with an explicit `encoding=` parameter — never rely on the platform default.
- Decode bytes to `str` at the earliest possible point (I/O boundary); encode `str` to bytes at the latest possible point.
- Normalize Unicode strings with `unicodedata.normalize('NFC', s)` before comparing, storing, or searching.
- Use `str.casefold()` (not `str.lower()`) for Unicode-correct case-insensitive comparison.
- Specify `errors='replace'` or `errors='surrogateescape'` instead of `errors='ignore'` — ignoring decoding errors silently drops data.
- Normalize before validating any security-sensitive string (usernames, paths, identifiers).

### Do Not

- Do not concatenate `str` and `bytes` — it raises `TypeError`.
- Do not rely on `len(s)` as a byte count — it counts code points, not bytes.
- Do not compare Unicode strings without normalization when content comes from different sources.
- Do not use `errors='ignore'` in security-sensitive decode paths — dropped bytes can mask injected content.
- Do not assume ASCII-safe strings are safe without checking for confusable Unicode characters in security contexts.
- Do not open files without `encoding=` in code that must run on multiple platforms — the default varies by OS and locale.

## Checklist

- [ ] All `open()` calls for text files include `encoding='utf-8'` (or explicit encoding)
- [ ] Bytes decoded to `str` at I/O boundaries with explicit encoding
- [ ] `unicodedata.normalize('NFC', s)` applied before string comparison or storage
- [ ] `str.casefold()` used for case-insensitive comparison, not `str.lower()`
- [ ] Security-sensitive strings normalized before validation
- [ ] No null byte (`\x00`) passed through to file paths or system calls
- [ ] No `str + bytes` concatenation

## See Also

- wiki/tier1-sources/owasp/a02-cryptographic-failures.md
- wiki/tier2-core/security-practices/python-pyscg.md
- wiki/tier3-working/python/regex.md
- wiki/tier3-working/python/overview.md
- wiki/tier1-sources/owasp/a03-injection.md

## Source

Python Unicode HOWTO, docs.python.org/3/howto/unicode.html. Python `unicodedata`, `codecs` module documentation. OWASP A02:2021 Cryptographic Failures.
