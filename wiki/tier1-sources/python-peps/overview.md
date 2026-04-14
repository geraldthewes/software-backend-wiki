# Python PEPs: Overview

> **Tier 1** | Source: Python Enhancement Proposals (python.org) | Authority: immutable

## Summary

A Python Enhancement Proposal (PEP) is the primary mechanism for proposing new features, documenting design decisions, and describing community standards for the Python language. PEPs are written by Python core developers, community members, and external contributors, and approved through a process involving the Python Steering Council. For a coding agent, PEPs are authoritative — they define the canonical "Pythonic" way to write code and are the baseline for all Python code quality decisions.

## Key Concepts

**Types of PEPs:**

- **Standards Track**: Propose new language features or changes (e.g., PEP 484 type hints)
- **Informational**: Describe design issues or provide guidelines (e.g., PEP 8 style guide)
- **Process**: Describe processes and policies (e.g., PEP 1 — PEP Purpose and Guidelines)

**PEP Status:** A PEP can be Draft, Accepted, Final, Superseded, Withdrawn, or Rejected. Only Final or Accepted PEPs represent authoritative guidance.

## Quick Reference: PEPs Most Relevant to Coding Agents

| PEP | Title                           | Status | Agent Impact                                                          |
|-----|---------------------------------|--------|-----------------------------------------------------------------------|
| 8   | Style Guide for Python Code     | Final  | All Python code must conform; enforced by ruff/flake8/black           |
| 20  | The Zen of Python               | Final  | Design philosophy; governs every architectural and naming decision    |
| 484 | Type Hints                      | Final  | Type annotations required on all public APIs; enables mypy/pyright   |
| 443 | Single-Dispatch Generic Functions | Final | Polymorphic dispatch without class hierarchies; alternative to isinstance chains |

## Why These Four?

- **PEP 8**: The mechanical standard — every Python file produced by an agent must pass a PEP 8 linter before being considered complete
- **PEP 20**: The philosophical standard — when multiple approaches are valid, PEP 20 aphorisms decide
- **PEP 484**: The contract standard — type hints make interfaces explicit and enable static analysis that catches entire classes of bugs before runtime
- **PEP 443**: The dispatch standard — provides a clean, extensible alternative to `isinstance` chains for type-based behavior

## Agent Guidance

### Do
- Apply PEP 8 to every Python file — run `ruff check` or `flake8` before considering code complete
- Use PEP 20 aphorisms as tiebreakers when evaluating design alternatives
- Add type hints (PEP 484) to all public function signatures and class attributes
- Consider `@singledispatch` (PEP 443) when writing type-based dispatch logic

### Do Not
- Override PEP 8 conventions without an explicit project-level decision documented in config
- Treat type hints as optional — they are required for agent-produced code
- Use `isinstance()` chains when `@singledispatch` would be cleaner and more extensible

## See Also
- wiki/tier1-sources/python-peps/pep-008-style.md
- wiki/tier1-sources/python-peps/pep-020-zen.md
- wiki/tier1-sources/python-peps/pep-484-type-hints.md
- wiki/tier1-sources/python-peps/pep-443-singledispatch.md

## Source
Python PEP Index. https://peps.python.org/
