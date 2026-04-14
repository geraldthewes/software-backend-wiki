# A06: Vulnerable and Outdated Components

> **Tier 1** | Source: OWASP Top 10:2021 | Authority: immutable

## Summary

Using libraries, frameworks, and other components with known vulnerabilities is one of the most common causes of real-world breaches. A single vulnerable transitive dependency — one you may not have explicitly chosen — can compromise an entire application. For a coding agent, every dependency decision carries security implications: choose carefully, pin precisely, and audit continuously.

## Key Concepts

**Why Transitive Dependencies Are the Hidden Risk:**

When you add `requests` to your project, you also accept its dependencies and their dependencies. The full dependency tree of a modern Python service typically contains dozens to hundreds of packages. Any one of those packages with a known CVE (Common Vulnerabilities and Exposures) is a potential attack vector — even if your direct dependencies are all trusted.

**Common Failure Scenarios:**

- Running an application with a dependency that has a published exploit (e.g., a remote code execution CVE in a parsing library)
- Failing to notice a security advisory because no one monitors dependency channels
- Using a package that has been abandoned (no commits in >2 years) — security issues will not be patched
- Dependency confusion attack: attacker publishes a malicious package to PyPI with the same name as an internal private package
- Pinning a version range (`>=1.2,<2.0`) instead of an exact version — a malicious release within the range can be installed

## Python Mitigations

**Run `pip-audit` in CI/CD pipeline:**

```bash
# Install pip-audit
pip install pip-audit

# Audit installed packages against known CVE databases
pip-audit

# Audit from a requirements file
pip-audit -r requirements.txt

# Fail CI if any vulnerability found
pip-audit --fail-on-severity high
```

Pin ALL dependency versions — exact pins prevent unexpected updates:

```
# requirements.txt — WRONG (floating versions)
requests>=2.28.0
flask~=2.3

# requirements.txt — RIGHT (exact pinning)
requests==2.31.0
flask==3.0.3
certifi==2024.2.2
```

Use `pip-compile` from `pip-tools` to generate a fully resolved, pinned `requirements.txt` from a `requirements.in` file:

```bash
pip-tools compile requirements.in --generate-hashes
```

The `--generate-hashes` flag adds SHA256 verification of each downloaded package.

**Generate SBOM for releases:**

```bash
# List all installed packages with versions for SBOM
pip list --format=json > sbom.json

# More comprehensive: cyclonedx-bom
pip install cyclonedx-bom
cyclonedx-py environment > sbom-cyclonedx.xml
```

**Evaluate packages before adding:**

Before adding any new dependency, check:
1. PyPI page: when was the last release?
2. GitHub repository: is it actively maintained? (issues being addressed, recent commits)
3. Download count and stars — low-traffic packages receive less security scrutiny
4. License compatibility

**Subscribe to security advisories:**

- GitHub Dependabot alerts for repositories
- PyPI Advisory Database: https://pypi.org/security/
- Python Security mailing list
- Safety DB: https://github.com/pyupio/safety-db

**Remove unused dependencies:**

```bash
# Find potentially unused imports
pip install pip-check-reqs
pip-extra-reqs .      # packages imported but not in requirements.txt
pip-missing-reqs .    # packages in requirements.txt but not imported
```

## Agent Guidance

### Do
- Add `pip-audit` as a CI step that blocks merges on high-severity CVEs
- Pin all dependencies to exact versions using `pip-compile`
- Review security posture of any new dependency before adding it
- Generate SBOM as part of release artifacts
- Check for abandoned packages (last release >2 years ago is a warning sign)

### Do Not
- Add new dependencies without checking for known CVEs first
- Use version ranges in production `requirements.txt`
- Ignore pip-audit warnings as "low priority"
- Assume transitive dependencies are safe because direct dependencies are

## Checklist
- [ ] `pip-audit` integrated in CI pipeline and blocks on high/critical CVEs
- [ ] All dependency versions exactly pinned in `requirements.txt`
- [ ] Dependencies compiled with `--generate-hashes` for integrity verification
- [ ] No known-critical CVEs in the dependency tree
- [ ] SBOM generated as part of the release process
- [ ] Unused dependencies identified and removed
- [ ] No packages with last release >2 years without explicit justification

## See Also
- wiki/tier1-sources/owasp/top10-2021-overview.md
- wiki/tier1-sources/owasp/a08-software-integrity-failures.md

## Source
OWASP Top 10:2021 — A06 Vulnerable and Outdated Components. https://owasp.org/Top10/A06_2021-Vulnerable_and_Outdated_Components/
