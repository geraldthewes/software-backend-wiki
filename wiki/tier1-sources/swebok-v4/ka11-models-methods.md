# Software Engineering Models and Methods (KA11)

> **Tier 1** | Source: SWEBOK V4, Chapter 11 | Authority: immutable

## Summary

Software Engineering Models and Methods covers the systematic, organized approaches used to analyze, specify, design, and verify software systems. A model is a simplified representation of a system or problem used to reason about it before committing to implementation. A method is a structured process for producing models or making design decisions. Together, models and methods bridge the gap between abstract requirements and concrete code.

For agents, models are communication and analysis tools. A well-placed sequence diagram prevents hours of misunderstanding about integration behavior. A state machine specification prevents entire categories of protocol bugs. Agents should default to modeling complex interactions before coding them — the cost of a diagram is trivial; the cost of redesigning code is not.

## Key Concepts

### Purpose of Modeling

Software models serve four distinct purposes:
1. **Communication**: Shared visual representations align stakeholders, developers, and reviewers on system behavior without requiring code-level literacy
2. **Analysis**: Models expose ambiguities, conflicts, and missing cases that are invisible in prose requirements
3. **Simulation**: Executable or formally verifiable models can detect design defects before implementation
4. **Code generation**: In Model-Driven Engineering, models are the primary artifact from which code is (partially) generated

All models are abstractions — they omit detail by design. The skill is choosing which details to omit and which to preserve. A model that includes too much becomes as incomprehensible as the code it was meant to clarify.

### UML Diagram Types and When to Use Each

**Structural diagrams** (what the system is):
- **Class diagram**: Shows classes, attributes, methods, and relationships (inheritance, composition, association). Use to document domain models, database schemas, and API contracts. Most widely used structural diagram.
- **Component diagram**: Shows the high-level components of a system and their dependencies. Use for architectural documentation and service boundary definition.
- **Deployment diagram**: Maps software components to physical or virtual infrastructure (servers, containers, cloud regions). Use for infrastructure planning and operations documentation.

**Behavioral diagrams** (what the system does):
- **Sequence diagram**: Shows the time-ordered exchange of messages between objects or services. Use for documenting API interactions, complex multi-service flows, and protocol specifications. Most useful for catching missing messages and race conditions at design time.
- **Activity diagram**: Shows control flow through a business process or algorithm. Similar to a flowchart but with concurrency support. Use for complex workflow logic.
- **State machine diagram**: Shows the states a component can be in and the events that trigger transitions. Use for protocol-heavy components (connection management, authentication flows, payment state machines). Prevents entire classes of invalid-state bugs.

**Choosing the right diagram**:
- New API or service interaction → Sequence diagram
- Component boundary clarification → Component diagram  
- Complex object lifecycle → State machine
- Domain model → Class diagram
- Business workflow → Activity diagram
- Infrastructure layout → Deployment diagram

### Formal Methods

Formal methods use mathematical notation to specify system behavior with precision that eliminates ambiguity.

**Specification languages**:
- **Z notation**: Set theory and predicate logic-based specification language. Used in safety-critical UK systems (London Underground signaling, medical devices).
- **Alloy**: Lightweight formal specification language with automatic verification via constraint solving. Suitable for specifying and model-checking security protocols and access control policies.
- **TLA+**: Temporal Logic of Actions. Used at Amazon Web Services to verify distributed system protocols (DynamoDB, S3 replication). Finds subtle concurrency bugs before any code is written.

**When formal methods are worth the cost**:
- Safety-critical systems where a defect risks human life (medical devices, aviation, nuclear control)
- Security-critical protocols where a flaw enables system compromise
- Complex distributed consensus protocols where informal reasoning fails
- Regulatory environments requiring provable correctness

For most commercial software, formal methods are not cost-effective. Use them selectively for the highest-risk components.

### Model-Driven Engineering (MDE)

MDE elevates models from documentation artifacts to primary engineering artifacts:
- Models are the specification; code is derived from or generated from models
- **Platform-Independent Models (PIM)**: Capture business logic without technology specifics
- **Platform-Specific Models (PSM)**: Transform PIMs into technology-specific representations
- **Model-to-code generation**: Tools generate boilerplate code from models, reducing manual effort

MDE is most valuable for systems with high regularity (many similar components), where generated code quality exceeds hand-crafted code. It is overkill for systems with unique, complex business logic.

### Domain-Specific Languages (DSLs)

A DSL is a programming or specification language tailored to a specific problem domain (SQL for database queries, regex for pattern matching, Gherkin for acceptance tests).

**Create a DSL when**:
- Domain experts need to specify behavior directly without developer intermediation
- A class of problems has a regular, well-defined structure that benefits from dedicated syntax
- Existing general-purpose languages produce verbose, error-prone specifications for the domain

**Reuse existing DSLs when**:
- A mature, well-tooled DSL already exists (SQL, HCL, Dockerfile)
- The team lacks the resources to build and maintain a DSL toolchain
- The domain is not sufficiently regular to justify DSL investment

**Anti-pattern**: Creating a custom DSL because the team finds existing languages inconvenient. DSL maintenance is a long-term burden that must be justified by clear, sustained value.

### Architecture Frameworks

**TOGAF (The Open Group Architecture Framework)**: Enterprise architecture framework covering business, data, application, and technology architectures. Used in large organizations for aligning IT with business strategy. Relevant for agents working in enterprise contexts.

**Zachman Framework**: Matrix-based framework for classifying enterprise architecture artifacts by audience (who, what, where, when, why, how) and perspective (planner, owner, designer, builder, implementer). Useful as a completeness checklist for architectural documentation.

Agents need awareness of these frameworks to recognize artifacts and communicate effectively in enterprise contexts, not necessarily to apply them in full.

## Agent Guidance

### Do
- Create sequence diagrams to clarify complex multi-service interactions before writing integration code
- Use state machine diagrams to specify protocol-heavy components before implementation
- Choose UML diagram types based on the communication need, not habit
- Apply formal methods (Alloy, TLA+) for security-critical protocol specifications where the cost of a defect is high
- Use existing mature DSLs rather than inventing custom languages

### Do Not
- Write complex integration code without first sketching the message sequence
- Skip state machine modeling for components with complex lifecycle rules (connection management, payment flows, authentication)
- Apply formal methods to routine business logic where the overhead is not justified
- Create a DSL without a clear, sustained, documented justification for the investment
- Treat UML as mandatory documentation overhead — use it where it adds clarity, skip it where it does not

## Checklist
- [ ] Sequence diagrams created for all non-trivial multi-service interactions
- [ ] State machines defined for all components with complex lifecycle or protocol behavior
- [ ] Architecture diagrams (component, deployment) created before implementation begins
- [ ] Formal specification applied to security-critical protocol components if risk warrants it
- [ ] DSL selection justified against existing alternatives
- [ ] All models version-controlled alongside the code they describe

## See Also
- `wiki/tier1-sources/swebok-v4/ka02-architecture.md`
- `wiki/tier1-sources/swebok-v4/ka03-design.md`
- `wiki/tier1-sources/swebok-v4/ka05-testing.md`

## Source
IEEE Computer Society. *Guide to the Software Engineering Body of Knowledge (SWEBOK), Version 4*. Chapter 11: Software Engineering Models and Methods. IEEE Press, 2024.
