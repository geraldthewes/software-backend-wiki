# A08: Software and Data Integrity Failures

> **Tier 1** | Source: OWASP Top 10:2021 | Authority: immutable

## Summary

Software and Data Integrity Failures occur when code and infrastructure lack protection against integrity violations. This includes insecure deserialization, unverified software updates, and compromised CI/CD pipelines. A supply chain attack through a single compromised dependency or pipeline step can affect thousands of downstream applications. For a coding agent, the critical rules are: **never deserialize untrusted data with pickle**, and protect the pipeline that produces and delivers software.

## Key Concepts

**Failure Scenarios:**

- **Unsafe deserialization with pickle**: Python's `pickle` module can execute arbitrary code during deserialization — using it on untrusted data is remote code execution
- **Unverified software updates**: Auto-update mechanisms that download and execute code without verifying the source or integrity
- **Insecure CI/CD pipeline**: Overly permissive pipeline permissions allow a pull request from an external contributor to access production secrets
- **Unsigned dependencies**: Package downloaded from a registry without hash verification — susceptible to tampering between registry and installation
- **Dependency confusion**: Internal package name matches a public package; attacker publishes a malicious version to PyPI with higher version number

## Python Mitigations

**NEVER use `pickle` for untrusted data (pyscg-0023):**

```python
import pickle

# CATASTROPHIC — pickle.loads() on untrusted input is remote code execution
# Attacker can craft a payload that executes arbitrary Python code
user_data = request.body
obj = pickle.loads(user_data)   # DO NOT DO THIS

# WRONG — also unsafe: any unpickling of externally-sourced data
cached = redis_client.get(key)
obj = pickle.loads(cached)       # if cache can be poisoned, this is RCE
```

Safe serialization alternatives:

```python
import json
import msgpack
from google.protobuf import ...  # protobuf

# RIGHT — JSON: human-readable, safe, widely supported
data = json.loads(untrusted_json_string)   # safe, no code execution

# RIGHT — msgpack: binary, fast, safe
data = msgpack.unpackb(untrusted_bytes)

# RIGHT — protobuf: schema-enforced, safe
msg = MyProto()
msg.ParseFromString(untrusted_bytes)       # schema-validated
```

If pickle must be used (e.g., ML model serialization), restrict its use to internally-generated data only and document that explicitly. Consider `safetensors` or ONNX format for ML models instead.

**Verify dependency integrity with hashes:**

```bash
# Generate requirements with hashes using pip-compile
pip-compile --generate-hashes requirements.in

# Install and verify hashes
pip install --require-hashes -r requirements.txt
```

The resulting `requirements.txt` contains SHA256 hashes:
```
requests==2.31.0 \
    --hash=sha256:58cd2187423d1... \
    --hash=sha256:942c5a758f98d...
```

**Protect CI/CD pipeline permissions:**

```yaml
# GitHub Actions — use OIDC, minimize permissions
permissions:
  contents: read        # only what's needed
  id-token: write       # for OIDC cloud auth (replaces long-lived secrets)

# Never use: permissions: write-all
# Never store: long-lived cloud credentials as secrets if OIDC is available
```

Key CI/CD security principles:
- Pull requests from forks should not have access to production secrets
- Use branch protection rules — require reviews before merging to main
- Pin CI action versions to commit SHA, not floating tags (`uses: actions/checkout@abc123f` not `@main`)
- Separate pipelines for test (low trust) and deploy (high trust, triggered only from protected branches)

**Verify integrity of downloaded artifacts:**

```bash
# Verify a downloaded file against its published SHA256
echo "expected_hash  filename.tar.gz" | sha256sum --check

# In Python
import hashlib

def verify_file_integrity(filepath: str, expected_sha256: str) -> bool:
    sha256 = hashlib.sha256()
    with open(filepath, "rb") as f:
        for chunk in iter(lambda: f.read(65536), b""):
            sha256.update(chunk)
    return sha256.hexdigest() == expected_sha256
```

**Avoid `eval()` and `exec()` on deserialized data:**

Any serialization format that allows embedded code (pickle, marshal, shelve) creates integrity failure risk. Even JSON data should be validated against a schema before use, not executed.

## Checklist
- [ ] No `pickle.loads()` on any externally-sourced data (network, user upload, external cache)
- [ ] Serialization uses JSON, msgpack, or protobuf for untrusted data
- [ ] `requirements.txt` generated with `--generate-hashes` for integrity verification
- [ ] CI/CD pipeline permissions minimized to least privilege
- [ ] CI actions pinned to commit SHA, not floating tags
- [ ] Pull requests from forks cannot access production secrets
- [ ] No `eval()` or `exec()` on deserialized or user-supplied data
- [ ] Downloaded artifacts verified against published checksums before use

## See Also
- wiki/tier1-sources/owasp/top10-2021-overview.md
- wiki/tier1-sources/owasp/a06-vulnerable-components.md
- wiki/tier1-sources/owasp/a03-injection.md

## Source
OWASP Top 10:2021 — A08 Software and Data Integrity Failures. https://owasp.org/Top10/A08_2021-Software_and_Data_Integrity_Failures/
