# Software Requirements (KA01)

> **Tier 1** | Source: SWEBOK V4, Chapter 1 | Authority: immutable

## Summary

The Software Requirements Knowledge Area covers the process of eliciting, analyzing, specifying, validating, and managing requirements before and during software construction. Requirements are the foundation on which all subsequent engineering decisions rest — architecture, design, testing, and maintenance all trace back to what the system is supposed to do and how well it must do it. Getting requirements wrong is the single most expensive category of error in software development, because defects introduced at this stage compound through every downstream phase.

For autonomous agents, requirements are the authoritative contract for what to build. An agent that proceeds to construction without a clear, validated understanding of requirements risks building the wrong system correctly — producing code that passes tests but fails to deliver the intended value.

## Key Concepts

### Requirements Elicitation Techniques

- **Interviews**: Structured or semi-structured conversations with stakeholders to surface needs, constraints, and expectations. Most effective for capturing tacit knowledge held by individuals.
- **Workshops (JAD/RAD sessions)**: Facilitated group sessions that bring multiple stakeholders together to resolve conflicts and align on shared requirements. Reduces the round-trip time of iterative interviews.
- **Prototyping**: Low-fidelity (paper) or high-fidelity (clickable) mockups that make abstract requirements concrete and reveal misunderstandings early.
- **Observation (ethnographic study)**: Watching users perform actual tasks in their real environment. Surfaces requirements that stakeholders cannot articulate because they are implicit in their workflow.
- **Surveys and questionnaires**: Scalable techniques for gathering quantitative data from large user populations on feature priorities or usage patterns.

### Types of Requirements

**Functional Requirements** — what the system does:
- Specific behaviors, outputs, or transformations the system must perform
- Expressed as: "The system shall allow users to reset their password via email verification"
- Directly testable with acceptance criteria

**Non-Functional Requirements (Quality Attributes)** — how well the system does it:
- **Performance**: response time, throughput, latency targets (e.g., "p99 response < 200ms under 1000 RPS")
- **Security**: authentication, authorization, encryption, audit logging requirements
- **Reliability**: availability targets (e.g., 99.9% uptime SLA), MTBF, MTTR
- **Scalability**: capacity at peak load, horizontal vs. vertical scaling constraints
- **Maintainability**: modularity, testability, documentation standards
- **Usability**: accessibility standards (WCAG), learnability, error recovery

NFRs are architecture-drivers. They constrain or eliminate entire design options and must be understood before architectural decisions are made.

### Requirements Specification

- **Use Cases**: Actor-goal-system interaction documented as main success scenario plus extensions. Best for capturing complex business workflows.
- **User Stories**: "As a [persona], I want [goal] so that [benefit]." Lightweight format suited to Agile contexts; must be accompanied by acceptance criteria.
- **Acceptance Criteria**: Specific, testable conditions that define when a requirement is satisfied. Written in Given-When-Then (Gherkin) or structured English.
- **Formal Specification**: Mathematical notation (Z, Alloy) for safety-critical or security-critical components where ambiguity is unacceptable.

### Requirements Validation

- **Reviews and inspections**: Structured walkthroughs of requirements documents with stakeholders and technical reviewers to catch ambiguity, inconsistency, and missing cases.
- **Prototyping**: Using executable prototypes to validate that the specified behavior matches stakeholder intent before committing to full construction.
- **Model validation**: Verifying that formal or semi-formal models (UML, state machines) correctly represent the specified behavior.
- **Test-driven validation**: Writing acceptance tests before implementation forces requirements to be concrete and testable.

### Requirements Management

- **Traceability**: Each requirement must be traceable forward to design/code/tests and backward to its stakeholder source. Enables impact analysis when requirements change.
- **Change control**: All requirement changes go through a defined process (impact analysis → approval → versioning) to prevent scope creep and surprise rework.
- **Version control of requirements**: Requirements documents are artifacts that must be version-controlled alongside code. A requirement that cannot be found is a requirement that will be missed.
- **Prioritization**: MoSCoW (Must/Should/Could/Won't), Kano model, or WSJF to sequence delivery of requirements by value and risk.

## Agent Guidance

### Do
- Read and confirm all functional requirements before writing a single line of implementation code
- Identify all NFRs that affect architecture (performance, security, availability) before proposing a design
- Write acceptance criteria for each requirement before beginning implementation
- Flag requirements that are ambiguous, conflicting, or underspecified — ask for clarification rather than guessing
- Trace every code module back to at least one requirement
- Confirm the priority of requirements when time or scope is constrained

### Do Not
- Begin construction without validated requirements
- Assume unstated NFRs — explicitly surface and document them with stakeholders
- Gold-plate: implement features beyond what requirements specify
- Change scope without going through change control
- Treat "the code works" as equivalent to "the requirements are met" — these are different claims

## Checklist
- [ ] All functional requirements are documented with acceptance criteria
- [ ] NFRs (performance, security, reliability) are quantified and acknowledged
- [ ] Requirements have been reviewed and approved by stakeholders
- [ ] Conflicts between requirements are identified and resolved
- [ ] Each requirement is uniquely identified and traceable
- [ ] Change control process is defined for requirements updates
- [ ] Ambiguous requirements have been clarified before construction starts

## See Also
- `wiki/tier1-sources/swebok-v4/ka02-architecture.md`
- `wiki/tier1-sources/swebok-v4/ka03-design.md`
- `wiki/tier1-sources/swebok-v4/ka09-engineering-management.md`
- `wiki/tier1-sources/swebok-v4/ka05-testing.md`

## Source
IEEE Computer Society. *Guide to the Software Engineering Body of Knowledge (SWEBOK), Version 4*. Chapter 1: Software Requirements. IEEE Press, 2024.
