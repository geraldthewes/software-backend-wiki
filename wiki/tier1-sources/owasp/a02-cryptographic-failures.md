# A02: Cryptographic Failures

> **Tier 1** | Source: OWASP Top 10:2021 | Authority: immutable

## Summary

Cryptographic Failures (formerly "Sensitive Data Exposure") covers vulnerabilities arising from weak, missing, or improperly implemented cryptography. Sensitive data — passwords, PII, financial data, health records — must be protected both in transit and at rest. For a coding agent, the rule is absolute: never implement custom cryptography, always delegate to well-audited libraries, and treat any cleartext transmission of sensitive data as a critical vulnerability.

## Key Concepts

**Sensitive Data** includes: passwords and credentials, PII (names, addresses, SSNs), financial data (card numbers, bank accounts), health records, session tokens, API keys, and private keys.

**Attack Scenarios:**

- **Cleartext transmission**: Sensitive data sent over HTTP instead of HTTPS; intercepted by network adversary
- **Weak password hashing**: MD5 or SHA1 without salt — crackable via rainbow tables in seconds; unsalted SHA256 is also insufficient
- **Insecure storage of PII**: Database fields storing sensitive data in plaintext — readable by any DB admin or via SQL injection
- **Missing TLS**: Internal service-to-service communication over plaintext HTTP inside a "trusted" network
- **Hardcoded keys**: Encryption keys or API secrets committed to source control or embedded in code
- **Weak algorithms**: Using DES, RC4, or ECB mode for AES encryption

**Root Causes:**

- Outdated algorithms still in use from legacy code
- Implementing custom cryptographic primitives instead of audited libraries
- Missing encryption at rest — assuming the database server is "safe"
- Missing encryption in transit — assuming internal networks are "safe"
- Key management neglect — keys stored alongside the data they protect

## Python Mitigations

**Never implement crypto primitives — always use `cryptography` library:**

```python
from cryptography.fernet import Fernet

# Symmetric encryption for data at rest
key = Fernet.generate_key()
f = Fernet(key)
encrypted = f.encrypt(b"sensitive data")
decrypted = f.decrypt(encrypted)
```

**Password hashing — use bcrypt or argon2, NEVER MD5/SHA1/SHA256:**

```python
# WRONG — never use these for passwords
import hashlib
hashed = hashlib.md5(password.encode()).hexdigest()     # insecure
hashed = hashlib.sha1(password.encode()).hexdigest()    # insecure
hashed = hashlib.sha256(password.encode()).hexdigest()  # insufficient for passwords

# RIGHT — bcrypt
import bcrypt
hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt(rounds=12))
verified = bcrypt.checkpw(password.encode(), hashed)

# RIGHT — argon2 (preferred for new implementations)
from argon2 import PasswordHasher
ph = PasswordHasher()
hashed = ph.hash(password)
ph.verify(hashed, password)  # raises VerifyMismatchError on failure
```

**TLS — enforce TLS 1.2+, never disable verification:**

```python
import requests
import certifi

# WRONG — never disable certificate verification
response = requests.get(url, verify=False)

# RIGHT — explicit CA bundle; TLS verified
response = requests.get(url, verify=certifi.where())
```

For services, enforce minimum TLS version in server configuration and reject TLS 1.0/1.1 connections.

**Cryptographically secure random tokens — use `secrets`, not `random`:**

```python
import secrets
import os

# For session tokens, API keys, CSRF tokens
token = secrets.token_urlsafe(32)         # URL-safe base64, 32 bytes entropy
hex_token = secrets.token_hex(32)          # hex string, 32 bytes entropy

# For raw crypto-random bytes
random_bytes = os.urandom(32)
```

**Key management — use secret managers, never hardcode:**

```python
# WRONG — key in source code
ENCRYPTION_KEY = b"hardcoded-secret-key-12345678901"

# RIGHT — load from environment/secret manager
import os
ENCRYPTION_KEY = os.environ["ENCRYPTION_KEY"].encode()

# For production, integrate with Vault or AWS Secrets Manager
```

## Checklist
- [ ] No MD5 or SHA1 used for password hashing anywhere in the codebase
- [ ] Password hashing uses bcrypt (min rounds=12) or argon2
- [ ] TLS enforced on all external HTTP connections; `verify=False` is absent
- [ ] No hardcoded keys, secrets, or API credentials in source code
- [ ] PII and sensitive fields encrypted at rest in the database
- [ ] Session tokens and API keys generated with `secrets` module, not `random`
- [ ] Key material stored in a secret manager, not in config files or environment file in VCS
- [ ] `certifi` CA bundle used for certificate validation

## See Also
- wiki/tier1-sources/owasp/top10-2021-overview.md
- wiki/tier1-sources/owasp/a07-auth-failures.md
- wiki/tier1-sources/owasp/a09-logging-monitoring.md

## Source
OWASP Top 10:2021 — A02 Cryptographic Failures. https://owasp.org/Top10/A02_2021-Cryptographic_Failures/
