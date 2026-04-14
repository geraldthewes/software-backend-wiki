# Software Maintenance (KA07)

> **Tier 1** | Source: SWEBOK V4, Chapter 7 | Authority: immutable

## Summary

Software Maintenance covers all activities performed on a software system after its initial delivery: fixing bugs, adapting to environmental changes, improving features, and proactively reducing future risk. Maintenance is not an afterthought — over the lifetime of most systems, maintenance effort exceeds initial development effort by a factor of 3 to 5. A system that is easy to build but hard to maintain is an expensive liability.

For agents working on existing codebases, this KA defines the discipline of responsible change. Every modification to a running system carries risk. That risk is managed through rigorous testing, careful scoping, explicit tracking of technical debt, and behavior-preserving refactoring before structural changes.

## Key Concepts

### Maintenance Types

- **Corrective**: Fixing defects — bugs, crashes, incorrect behavior discovered after release. The most visible maintenance type. Driven by failure reports and production incidents.
- **Adaptive**: Modifying the system to work in a changed environment — OS upgrades, dependency version updates, cloud platform migrations, regulatory compliance changes. The environment changes even when the code does not.
- **Perfective**: Enhancing or extending existing features based on new stakeholder needs. The largest category of maintenance by volume in most systems.
- **Preventive**: Refactoring, restructuring, documentation updates, and other work that does not change visible behavior but reduces the cost of future changes. Often deferred under schedule pressure, creating compounding technical debt.

### Technical Debt

Technical debt is a metaphor for design or implementation shortcuts that provide short-term speed at the cost of long-term maintainability.

**Intentional (deliberate) debt**: A conscious trade-off — "we know this design is not ideal, but we need to ship by Friday." Acceptable only when explicitly acknowledged, documented, and scheduled for repayment.

**Unintentional (reckless) debt**: Introduced through ignorance of better practices or lack of domain knowledge. Has the same cost as intentional debt but without the benefit of a deliberate decision.

**The economics of debt**:
- The *principal* is the original shortcut (the thing that must eventually be fixed)
- The *interest* is the extra effort every future change costs because of the debt
- Debt that is never repaid eventually makes the system unmaintainable ("technical bankruptcy")

**Tracking debt**: Technical debt items must be captured in the issue tracker (not just in code comments) with a description of the shortcut, its cost, and the planned remediation.

### Refactoring

Refactoring is the process of changing the internal structure of code without changing its observable behavior, to improve readability, reduce complexity, or enable future extension.

**Rules for safe refactoring**:
1. Tests must pass before refactoring begins
2. Refactor in small, verifiable steps
3. Tests must still pass after each step
4. Never change behavior and structure simultaneously

**When to refactor**:
- **Boy scout rule**: Leave the code cleaner than you found it — always make one small improvement when touching a module
- **Before adding a feature**: Refactor the existing code to make the new feature easy to add, then add the feature
- **After passing tests**: Once green, improve the implementation

**Common refactoring patterns**:
- *Extract Method/Function*: Move a block of code into a named function to clarify intent
- *Rename*: Rename variables, functions, or classes to accurately reflect their purpose
- *Introduce Parameter Object*: Replace a long parameter list with a structured object
- *Replace Magic Number with Named Constant*: Make intent explicit
- *Extract Class*: Split a class with multiple responsibilities into focused single-purpose classes

### Legacy Code

Legacy code is code without adequate test coverage, regardless of age. Working with legacy code safely requires a specific discipline.

**Characterization tests**: Write tests that capture the current behavior (even if that behavior is wrong) before making any change. This creates a safety net for detecting unintended regressions.

**Strangler fig pattern**: Incrementally replace a legacy system by routing new requests to the new implementation while the old system handles existing functionality. Over time, the old system "strangles" and can be removed. Avoids the risk of a big-bang rewrite.

**Seams**: Points in legacy code where behavior can be changed without modifying the surrounding code (e.g., dependency injection points, configuration parameters). Use seams to insert test doubles without restructuring.

### Change Management

- **Impact analysis**: Before modifying any module, identify all dependent modules, data stores, and external interfaces that may be affected
- **Regression testing**: Run the full test suite after every change, not just tests related to the modified module
- **Documentation updates**: Code changes that affect interfaces, behavior, or configuration must be accompanied by documentation updates in the same commit

### Software Aging

Systems degrade over time through:
- **Bit rot**: Code that once worked correctly fails as dependencies, operating systems, and platforms evolve
- **Technology drift**: The surrounding ecosystem moves forward while the system stays still, increasing the cost of integration
- **Accumulated complexity**: Each workaround and special case makes the system harder to reason about, slowing all future changes

Preventive maintenance combats software aging through dependency updates, periodic refactoring, and architectural review.

## Agent Guidance

### Do
- Write characterization tests before touching any legacy code without adequate coverage
- Refactor existing code to accommodate a new feature *before* adding the feature
- Track all identified technical debt in the issue tracker with a description and estimated remediation effort
- Apply the boy scout rule: improve something small every time you touch a module
- Use the strangler fig pattern for large-scale legacy replacements

### Do Not
- Change observable behavior and internal structure in the same commit
- Begin refactoring without a passing test suite
- Add features to a tangled, untested module without first establishing a safety net
- Leave technical debt undocumented — silent debt is unmanaged debt
- Attempt a big-bang rewrite of a complex legacy system

## Checklist
- [ ] Existing tests pass before any maintenance change begins
- [ ] Type of maintenance (corrective/adaptive/perfective/preventive) is identified and logged
- [ ] Impact analysis completed for the changed modules
- [ ] Technical debt items created in the issue tracker for known shortcuts
- [ ] Refactoring commits are separate from behavior-change commits
- [ ] Regression suite passes after every change
- [ ] Documentation updated to reflect behavioral or interface changes
- [ ] Characterization tests written before touching untested legacy code

## See Also
- `wiki/tier1-sources/swebok-v4/ka04-construction.md`
- `wiki/tier1-sources/swebok-v4/ka05-testing.md`
- `wiki/tier1-sources/swebok-v4/ka08-config-management.md`
- `wiki/tier1-sources/swebok-v4/ka12-quality.md`
- `wiki/tier1-sources/swebok-v4/ka15-economics.md`

## Source
IEEE Computer Society. *Guide to the Software Engineering Body of Knowledge (SWEBOK), Version 4*. Chapter 7: Software Maintenance. IEEE Press, 2024.
