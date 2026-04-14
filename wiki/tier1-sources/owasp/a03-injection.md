# A03: Injection

> **Tier 1** | Source: OWASP Top 10:2021 | Authority: immutable

## Summary

Injection flaws occur when hostile data is sent to an interpreter as part of a command or query. An attacker can use injection to read, modify, or delete any database data; execute operating system commands; or exfiltrate sensitive files. This was the #1 OWASP risk for over a decade. For a coding agent, the rule is absolute: **never construct commands or queries by concatenating or formatting user-supplied input**.

## Key Concepts

**Injection Types:**

- **SQL Injection**: User input embedded in SQL query strings — can read/write/delete any database table, bypass authentication
- **Command Injection**: User input passed to OS shell — can execute arbitrary system commands with the application's privileges
- **LDAP Injection**: User input embedded in LDAP filter strings — bypasses directory authentication
- **XPath Injection**: User input embedded in XPath queries against XML data
- **XXE (XML External Entity)**: Malicious XML references external entities — can read server files, trigger SSRF
- **SSTI (Server-Side Template Injection)**: User input rendered by a template engine — can execute arbitrary code in many engines

**Why It's Dangerous:**

An SQL injection can dump an entire database, including password hashes, PII, and financial records, in a single request. A command injection can give an attacker a remote shell with application-level privileges. These are not theoretical — they are among the most commonly exploited vulnerabilities in production systems.

**Root Cause:** The application fails to distinguish between code and data, allowing user-controlled data to be interpreted as code by the target interpreter.

## Python Mitigations

### SQL: Always Use Parameterized Queries

```python
import sqlite3

# WRONG — string formatting: allows SQL injection
user_id = request.args.get("id")
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")           # catastrophic
cursor.execute("SELECT * FROM users WHERE id = " + user_id)           # catastrophic
cursor.execute("SELECT * FROM users WHERE id = '%s'" % user_id)       # catastrophic

# RIGHT — parameterized query: user_id is never interpreted as SQL
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))        # sqlite3
```

With an ORM, use the ORM's query API — never fall back to raw string construction:

```python
# RIGHT with SQLAlchemy
from sqlalchemy import select
stmt = select(User).where(User.id == user_id)        # parameterized automatically
session.execute(stmt)

# WRONG — bypasses ORM safety
session.execute(f"SELECT * FROM users WHERE id = {user_id}")
```

**Pyscg rule pyscg-0010**: Use parameterized queries for ALL SQL — no exceptions.

### Shell Commands: Never Use shell=True with User Input

```python
import subprocess

filename = request.args.get("filename")

# WRONG — shell=True with user input allows command injection
subprocess.run(f"cat {filename}", shell=True)                         # catastrophic
os.system(f"process_file {filename}")                                 # catastrophic

# RIGHT — list form, no shell interpolation
subprocess.run(["cat", filename], shell=False, capture_output=True)

# RIGHT — if shell features are needed, validate input strictly first
import shlex
safe_name = shlex.quote(filename)   # last resort; prefer list form
```

### XML: Use defusedxml to Prevent XXE

```python
# WRONG — lxml with untrusted input is vulnerable to XXE
from lxml import etree
tree = etree.parse(untrusted_xml_file)

# RIGHT — defusedxml disables dangerous XML features
import defusedxml.ElementTree as ET
tree = ET.parse(untrusted_xml_file)   # XXE, entity expansion, and DTD attacks disabled
```

### Templates: Enable Autoescaping; Never eval() User Input

```python
from jinja2 import Environment

# WRONG — autoescaping disabled or eval/exec on user input
env = Environment(autoescape=False)
eval(user_supplied_expression)       # remote code execution
exec(user_supplied_code)             # remote code execution

# RIGHT — autoescaping enabled (default for HTML templates in Jinja2)
env = Environment(autoescape=True)
template = env.from_string("Hello {{ name }}")
output = template.render(name=user_input)   # safely escaped
```

### LDAP: Use Parameterized LDAP Queries

```python
import ldap3

# WRONG — string formatting in LDAP filter
ldap_filter = f"(uid={username})"
conn.search(search_base, ldap_filter)

# RIGHT — escape user input in LDAP filter
from ldap3.utils.conv import escape_filter_chars
safe_username = escape_filter_chars(username)
ldap_filter = f"(uid={safe_username})"
conn.search(search_base, ldap_filter)
```

## Checklist
- [ ] No string-formatted SQL anywhere in the codebase (grep for `%s` string formatting in SQL, f-strings in SQL)
- [ ] All database queries use parameterized queries or ORM query API
- [ ] No `shell=True` with any user-controlled input
- [ ] `defusedxml` used for all XML parsing of untrusted data
- [ ] Jinja2 and other template engines have autoescaping enabled
- [ ] No `eval()` or `exec()` on user-supplied strings
- [ ] LDAP filters escape user input with `escape_filter_chars`
- [ ] All user inputs validated and sanitized before reaching any interpreter

## See Also
- wiki/tier1-sources/owasp/top10-2021-overview.md
- wiki/tier1-sources/owasp/a01-broken-access-control.md
- wiki/tier1-sources/owasp/a10-ssrf.md

## Source
OWASP Top 10:2021 — A03 Injection. https://owasp.org/Top10/A03_2021-Injection/
