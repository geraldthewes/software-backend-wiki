# SWEBOK V4 — KA 12: Software Quality

> **Tier 1** | Source: IEEE SWEBOK V4, KA 12; ISO/IEC 25010 | Authority: immutable

## Summary

Software Quality is the degree to which a software system satisfies stated requirements, implied needs, and the expectations of stakeholders. Quality is not a single property but a structured set of attributes — functional correctness, performance, security, maintainability, reliability, and others — that collectively determine whether software fulfills its purpose. In SWEBOK V4, Quality remains KA 12 but is significantly updated: security (formerly a sub-topic here) has been elevated to standalone KA 13, and the integration of quality assurance activities into Agile and DevOps pipelines is now a first-class requirement, not a note.

For a coding agent, the primary quality directive is: quality is built in, not tested in. No amount of testing at the end of construction can repair a poorly designed or carelessly coded system. Quality assurance activities — reviews, static analysis, testing — must be integrated throughout every phase of the lifecycle. The agent must understand the difference between assurance (process-focused) and control (product-focused) and apply appropriate activities at each stage.

## Key Concepts

### Software Quality Attributes (ISO/IEC 25010)

ISO 25010 defines the standard quality model for software product quality. Every quality requirement must map to one of these attributes:

| Attribute | Definition | Sub-characteristics | Example Metric |
|-----------|-----------|-------------------|----------------|
| **Functional Suitability** | Functions meet stated and implied needs | Completeness, correctness, appropriateness | Test pass rate against acceptance criteria |
| **Performance Efficiency** | Resource usage relative to performance under stated conditions | Time behavior, resource utilization, capacity | p99 response time <200ms; CPU <70% at peak |
| **Compatibility** | Coexistence and interoperability with other systems | Co-existence, interoperability | API contract test pass rate |
| **Usability** | Users can use the system effectively and efficiently | Learnability, operability, user error protection | Task completion rate in user testing |
| **Reliability** | System performs its functions under stated conditions for a stated period | Maturity, availability, fault tolerance, recoverability | MTBF, availability %, error rate |
| **Security** | Protection of information and data against unauthorized access or modification | Confidentiality, integrity, non-repudiation, accountability, authenticity | Zero critical findings in SAST; 0 OWASP Top 10 violations |
| **Maintainability** | Effectiveness and efficiency of modification | Modularity, reusability, analyzability, modifiability, testability | Cyclomatic complexity ≤10; test coverage >80% |
| **Portability** | Ability to transfer to another environment | Adaptability, installability, replaceability | 12-Factor compliance score |

**Agent instruction:** Every quality requirement stated in a ticket or specification must be mapped to an ISO 25010 attribute before implementation begins. This ensures the quality requirement is measurable and testable.

---

### Quality Assurance vs. Quality Control

These terms are frequently confused. The distinction matters for an agent because it determines where and how quality activities are applied:

| Concept | Focus | When Applied | Examples |
|---------|-------|-------------|---------|
| **Quality Assurance (QA)** | Process — ensuring the process will produce quality | Throughout the lifecycle | Code review process, CI pipeline enforcement, coding standards, Definition of Done |
| **Quality Control (QC)** | Product — inspecting the product for defects | After or during construction | Test execution, static analysis results, code inspection findings |

Good QA reduces the need for extensive QC: if the process reliably produces quality code, less inspection is needed to find defects. An agent that only runs tests (QC) without enforcing standards and reviews (QA) is doing half the job.

---

### Software Quality Assurance Activities

| Activity | Type | Purpose | Timing |
|----------|------|---------|--------|
| **Reviews** | QA | Identify defects via systematic examination | During design and construction |
| **Inspections** | QA/QC | Formal review with defined roles (reader, moderator, author) | Before major milestones |
| **Walkthroughs** | QA | Author leads reviewers through the design or code | Early design phase |
| **Audits** | QA | Independent examination for compliance | Process milestone; release gates |
| **Static Analysis** | QC | Automated tool examination of source code without execution | Every commit (pre-commit hooks) |
| **Dynamic Testing** | QC | Execution-based verification | Unit tests: every commit; integration: every PR |
| **Performance Testing** | QC | Validation of performance requirements | Pre-release; nightly for regression |
| **Security Testing** | QC | SAST, DAST, dependency audit | Every commit (SAST); nightly (DAST) |

---

### Software Quality Metrics

Quality must be measured, not asserted. The following metrics are tracked continuously:

| Metric | Definition | Target | Measurement Tool |
|--------|-----------|--------|-----------------|
| **Defect Density** | Number of defects per KLOC (thousand lines of code) | Trending downward | Bug tracker + LOC counter |
| **Test Coverage** | Percentage of code executed by the test suite | Statement: >90%; Branch: >80% | `pytest --cov` |
| **Mutation Score** | Percentage of intentional code mutations caught by tests | >80% | `mutmut` |
| **Cyclomatic Complexity** | Average cyclomatic complexity per function | Mean <5; Max <10 | `radon cc` |
| **Technical Debt Ratio** | Estimated remediation time / development time | <5% (SQALE model) | SonarQube or manual |
| **Mean Time to Failure (MTTF)** | Average time between system failures | Increasing trend | Observability platform |
| **Mean Time to Recovery (MTTR)** | Average time to restore after a failure | Decreasing trend | Incident tracking |
| **Code Review Coverage** | Percentage of commits reviewed before merge | 100% | Git branch protection rules |
| **Flakiness Rate** | Percentage of test runs with unexpected failures | <1% | CI test result tracking |

---

### Code Review as Primary Quality Gate

Code review is the single highest-ROI quality activity: it catches defects before they enter the codebase, transfers knowledge between team members, and enforces standards consistently.

**What code review must check:**

| Category | Key Questions |
|----------|--------------|
| **Correctness** | Does it handle edge cases? Are errors handled? Are preconditions validated? |
| **Design** | Is coupling low? Is cohesion high? Are SOLID principles applied? |
| **Security** | No hardcoded secrets? Queries parameterized? No unsafe deserialization? |
| **Testability** | Are tests present? Do they test behavior? Is mutation score tracked? |
| **Standards** | PEP 8 compliant? Typed? Static analysis passes? |
| **Maintainability** | Are names clear? Are complex decisions documented? Is complexity within limits? |

**Code review rules for agents:**
- Every pull request must be reviewed before merge (enforced by branch protection)
- A review that only says "LGTM" without examining the items above is not a review
- Test code must be reviewed with the same rigor as production code — bad tests are worse than no tests

---

### Quality in Agile: Definition of Done

In Agile, "done" means quality-complete. The Definition of Done (DoD) is the shared checklist that every story must satisfy before being accepted. A standard DoD for backend services:

- [ ] All acceptance criteria pass
- [ ] Unit tests written and passing (mutation score >80% for new code)
- [ ] Integration tests passing
- [ ] Static analysis passes: ruff, mypy --strict, bandit
- [ ] `pip-audit` shows no HIGH or CRITICAL vulnerabilities
- [ ] Code reviewed and approved
- [ ] Documentation updated (docstrings, API docs, README if relevant)
- [ ] No new technical debt without a corresponding tracking ticket
- [ ] Observability: structured logs, metrics, and traces emitted for new paths

---

### V4 Integration: Quality is Built In

SWEBOK V4's primary shift in KA 12 is the explicit rejection of the "test phase" model — the notion that quality is verified at the end of construction. V4 mandates:

1. **Shift-left quality:** Quality activities begin at requirements (Are acceptance criteria testable? Are quality attributes measurable?)
2. **Continuous quality measurement:** Coverage, complexity, and security metrics are tracked in CI dashboards, not collected at release
3. **Quality as architecture concern:** Maintainability, reliability, and performance are quality attributes that must be specified and architected for (KA 02), not discovered as defects during testing
4. **Automated enforcement:** Quality gates (static analysis, test pass rates, coverage thresholds) are automated in CI — not dependent on individual discipline

---

## Agent Guidance

### Do
- Map every quality requirement to an ISO 25010 attribute to make it measurable
- Apply quality assurance (process) activities — code review, standards enforcement, DoD — in addition to quality control (testing)
- Maintain all four static analysis checks (ruff, mypy, bandit, pip-audit) passing before marking code complete
- Track quality metrics continuously in CI: coverage trends, complexity averages, mutation scores
- Treat the Definition of Done as mandatory, not aspirational
- Review test code with the same rigor as production code — bad tests degrade quality

### Do Not
- Rely on testing alone to achieve quality — quality cannot be tested in; it must be built in
- Accept "we'll fix the quality later" as a plan — track the debt explicitly with a remediation timeline
- Report high test coverage as a quality guarantee without also checking mutation score
- Skip code review because a change is "small" — small changes introduce subtle defects
- Leave flaky tests in the test suite — they destroy trust in quality metrics

## Checklist
- [ ] Quality requirements mapped to ISO 25010 attributes with measurable targets
- [ ] Definition of Done checklist satisfied before marking any story complete
- [ ] All static analysis tools pass: ruff, mypy --strict, bandit, pip-audit
- [ ] Test coverage ≥90% statement, ≥80% branch for new code
- [ ] Mutation score ≥80% for new modules
- [ ] Code review completed with all categories examined
- [ ] Quality metrics (coverage, complexity, defect density) reviewed in CI dashboard
- [ ] No new technical debt without a tracking ticket and remediation plan

## See Also
- wiki/tier1-sources/swebok-v4/overview.md
- wiki/tier1-sources/swebok-v4/ka04-construction.md
- wiki/tier1-sources/swebok-v4/ka05-testing.md
- wiki/tier1-sources/swebok-v4/ka13-security.md
- wiki/tier1-sources/swebok-v4/ka02-architecture.md
- wiki/tier2-core/testing-strategies/test-pyramid.md
- wiki/tier3-working/checklists/definition-of-done.md
- wiki/tier3-working/checklists/code-review.md

## Source

IEEE Computer Society. *Guide to the Software Engineering Body of Knowledge (SWEBOK), Version 4.0*, KA 12: Software Quality. IEEE Press, 2024.

ISO/IEC 25010:2023. *Systems and software engineering — Systems and software Quality Requirements and Evaluation (SQuaRE) — Product quality model*.

Kan, S.H. *Metrics and Models in Software Quality Engineering, 2nd Edition*. Addison-Wesley, 2002.
