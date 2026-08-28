# Comprehensive Code Review Guidelines

> **Tier 3** | Source: Google Engineering Practices, Microsoft Engineering Fundamentals Playbook, SmartBear Best Practices, Palantir Code Review Best Practices, Thoughtbot Guides | Authority: working

## Summary
This page consolidates high-quality code review guidelines from multiple reputable sources including Google, Microsoft, SmartBear, Palantir, and Thoughtbot. It provides coding agents with a comprehensive framework for conducting effective code reviews that improve code health, foster positive team culture, and catch defects early in the development process. The guidelines emphasize principles over perfection, focusing on continuous improvement rather than seeking flawless code.

## Key Principles from Multiple Sources

### Google Engineering Practices
- **Primary Purpose**: Improve overall code health over time, not achieve perfection
- **Standard of Review**: Favor approving a change once it definitely improves code health, even if not perfect
- **Balance**: Need to make forward progress vs. importance of suggested changes
- **Mentoring**: Educational comments are valuable but should be prefixed with "Nit:" if not mandatory
- **Principles**:
  - Technical facts and data overrule opinions and personal preferences
  - Style guide is absolute authority for style points
  - Software design aspects should be weighed on engineering principles, not personal opinion
  - Maintain consistency with existing codebase when no other rule applies

### Microsoft Engineering Fundamentals Playbook
- **Focus Areas**: Correctness, readability, maintainability
- **Positive Review Culture**: Encourage appreciation for good practices
- **Be Considerate**: Use "we" language, ask questions, explain why changes are needed
- **First Design Pass**: Check PR description, user-facing changes, design interactions
- **Code Quality Pass**: Review complexity, naming/readability, error handling, functionality, style, tests
- **Understanding**: Read every line changed, ask for clarification when needed
- **Pacing**: Take breaks to maintain focus, don't rush or over-extend review sessions

### SmartBear Best Practices (Research-backed)
- **Review Size**: Limit to 200-400 lines of code at a time (70-90% defect discovery)
- **Pace**: Inspection rates under 500 LOC per hour, max 60 minutes per session
- **Goals & Metrics**: Set SMART goals, track inspection rate, defect rate, defect density
- **Author Preparation**: Annotate source code before review to guide reviewers
- **Checklists**: Use to eliminate frequent errors and combat omission defects
- **Fixing Process**: Establish systematic method for fixing defects found
- **Positive Culture**: Defects are opportunities for improvement, not basis for performance evaluation
- **Lightweight Reviews**: Tool-assisted reviews take <20% time of formal reviews with equal effectiveness

### Palantir Code Review Best Practices
- **Purpose Clarity**: Understand why we review (defect prevention, knowledge sharing, consistency)
- **Preparation**: Both author and reviewer should prepare before the review
- **What to Look For**: Design, functionality, complexity, tests, naming, comments, style, documentation
- **Good vs Bad Examples**: Specific examples of effective and ineffective review comments
- **Examples of Good Comments**: Specific, actionable, balanced with appreciation
- **Examples of Bad Comments**: Vague, harsh, dismissive, or overly prescriptive

### Thoughtbot Guides – Code Review
- **Constructive Feedback**: Emphasize kindness and effectiveness
- **GitHub PR Focus**: Practical guidance for pull request-based workflows
- **Tone Matters**: Frame feedback as suggestions, not demands
- **Context Awareness**: Consider the broader system and team practices
- **Iterative Improvement**: Focus on making code better, not perfect

## Unified Code Review Checklist

### Before the Review
- [ ] Author has annotated source code to guide reviewers (SmartBear)
- [ ] PR description clearly explains the purpose and context (Microsoft/Palantir)
- [ ] Change set is reasonable size (<400 LOC preferred) (SmartBear)
- [ ] Author has run tests locally and believes code is ready (Microsoft)

### During the Review - Design
- [ ] Does the overall design make sense for the system? (Google/Microsoft/Palantir)
- [ ] Does this change belong in the codebase or should it be in a library? (Google)
- [ ] Does it integrate well with the rest of the system? (Google)
- [ ] Is now a good time to add this functionality? (Google)
- [ ] Are architecture and coding patterns properly incorporated? (Microsoft)
- [ ] Do interactions between code pieces make sense? (Palantir)

### During the Review - Functionality
- [ ] Does the code do what the developer intended? (Google/Microsoft)
- [ ] Is the intended behavior good for users (end-users and future developers)? (Google)
- [ ] Are edge cases handled properly? (Google/Microsoft)
- [ ] Are there any concurrency problems (race conditions, deadlocks)? (Google/Microsoft)
- [ ] For user-facing changes, has functionality been validated? (Google/Microsoft)

### During the Review - Complexity
- [ ] Is the code more complex than it needs to be? (Google/Microsoft/Palantir)
- [ ] Check complexity at all levels: lines, functions, classes (Google)
- [ ] Is there over-engineering (solving future problems unnecessarily)? (Google)
- [ ] Can the code be understood quickly by readers? (Google)
- [ ] Are developers likely to introduce bugs when modifying this code? (Google)
- [ ] Are functions/methods overly complex (too many arguments, nesting)? (Microsoft)
- [ ] Is functionality added that isn't currently needed? (Microsoft)

### During the Review - Tests
- [ ] Are appropriate tests included (unit, integration, end-to-end)? (Google/Microsoft)
- [ ] Are tests in the same CL as production code (unless emergency)? (Google)
- [ ] Are tests correct, sensible, and useful? (Google)
- [ ] Will tests actually fail when code is broken? (Google)
- [ ] Do tests make simple, useful assertions? (Google)
- [ ] Are tests separated appropriately between test methods? (Google)
- [ ] Is test complexity acceptable (tests are maintainable code)? (Google)
- [ ] Are edge cases explicitly tested? (Microsoft/Thoughtbot)
- [ ] Are test doubles at the appropriate level (mock I/O boundary, not business logic)? (Existing wiki)

### During the Review - Naming
- [ ] Are names clear and descriptive? (Google/Microsoft/Palantir/Thoughtbot)
- [ ] Are names long enough to communicate purpose without being hard to read? (Google)
- [ ] Do names follow language-specific conventions (snake_case, PascalCase, etc.)? (Existing wiki)

### During the Review - Comments
- [ ] Are comments clear and in understandable language? (Google/Microsoft)
- [ ] Are comments actually necessary? (Google)
- [ ] Do comments explain WHY code exists rather than WHAT it does? (Google)
- [ ] Are there exceptions for complex algorithms/regex that benefit from explanatory comments? (Google)
- [ ] Have pre-existing TODO comments been addressed or removed? (Google)
- [ ] Are comments different from documentation (which should express purpose/usage)? (Google)

### During the Review - Style
- [ ] Does code follow the appropriate style guide? (Google/Microsoft/Palantir)
- [ ] Are purely style points (not in style guide) treated as personal preferences? (Google)
- [ ] Are minor style points prefixed with "Nit:" to indicate they're optional? (Google)
- [ ] Are major style changes separated from functional changes in different CLs? (Google)
- [ ] If existing code is inconsistent with style guide, should new code follow guide or surroundings? (Google)
- [ ] Bias toward following style guide unless local inconsistency would be too confusing (Google)
- [ ] Should author file a bug/TODO for cleaning up existing inconsistent code? (Google)

### During the Review - Documentation
- [ ] If CLI changes how users build/test/interact/release code, is documentation updated? (Google)
- [ ] If code is deleted/deprecated, should documentation also be deleted? (Google)
- [ ] If documentation is missing, has it been requested? (Google)
- [ ] Are public APIs and non-obvious functions documented with docstrings? (Existing wiki)
- [ ] Is README updated for setup/configuration/behavior changes? (Existing wiki)
- [ ] Are breaking API changes documented in CHANGELOG or migration guide? (Existing wiki)

### During the Review - Every Line & Context
- [ ] Have I reviewed every line of human-written code (not scanning over)? (Google)
- [ ] For data files/generated code/large structures, is scanning appropriate? (Google)
- [ ] Have I looked at the full file context when seeing isolated changes? (Google)
- [ ] Have I considered the CL in context of the whole system? (Google)
- [ ] Am I improving code health or making the system more complex/less tested? (Google)
- [ ] If I don't understand code, have I asked the developer to clarify? (Google/Microsoft)
- [ ] If I'm not qualified for certain aspects (security, concurrency, etc.), is there a qualified reviewer? (Google)

### After the Review
- [ ] Have I provided specific, actionable feedback? (Palantir/Thoughtbot)
- [ ] Have I balanced criticism with appreciation for good practices? (Palantir/Microsoft)
- [ ] Have I prefixed optional/style points with "Nit:"? (Google)
- [ ] Have I asked questions rather than made accusatory statements? (Microsoft/Palantir)
- [ ] Have I explained WHY changes are needed with examples when helpful? (Microsoft)
- [ ] Have I avoided language that points fingers ("you" statements)? (Microsoft)
- [ ] If disagreements exist, have I sought consensus or escalated appropriately? (Google)
- [ ] Have I noted which parts I reviewed if only reviewing certain files/aspects? (Google)

## Agent Guidance for Code Review

### Do
- Focus on improving code health over time, not achieving perfection (Google)
- Favor approving changes that definitely improve code health, even if not perfect (Google)
- Balance forward progress with importance of suggested changes (Google)
- Treat style guide as absolute authority for style points (Google)
- Weigh software design aspects on engineering principles, not personal opinion (Google)
- Maintain consistency with existing codebase when no other rule applies (Google)
- Read every line changed and ask for clarification when needed (Microsoft/Google)
- Take breaks to maintain focus during review sessions (Microsoft/SmartBear)
- Limit reviews to 200-400 lines of code when possible (SmartBear)
- Keep inspection rates under 500 LOC per hour and sessions under 60 minutes (SmartBear)
- Set SMART goals for code review effectiveness and track metrics (SmartBear)
- Expect authors to annotate source code before review (SmartBear)
- Use checklists to eliminate frequent errors and combat omission defects (SmartBear)
- Establish systematic process for fixing defects found in review (SmartBear)
- View defects as opportunities for improvement, not basis for performance evaluation (SmartBear)
- Embrace lightweight, tool-assisted review processes (SmartBear)
- Be positive and encouraging, appreciating good practices (Microsoft/Palantir/Thoughtbot)
- Use "we" language and ask questions rather than making accusations (Microsoft)
- Explain WHY code needs to be changed, preferably with examples (Microsoft)
- Prefix optional/style points with "Nit:" to indicate they're not mandatory (Google/Microsoft)
- Provide specific, actionable feedback rather than vague comments (Palantir)
- Frame feedback as suggestions rather than demands (Thoughtbot)
- Consider the broader system and team practices when reviewing (Thoughtbot)
- Focus on iterative improvement rather than perfection (Thoughtbot)
- Note which specific files/aspects you reviewed if not reviewing the entire change (Google)
- Seek consensus or escalate when coming to consensus is difficult (Google)
- Compliment developers on good practices they demonstrate (Google)

### Do Not
- Require authors to polish every tiny piece before granting approval (Google)
- Seek perfection; instead seek continuous improvement (Google)
- Block CLs based solely on personal style preferences not in style guide (Google)
- Accept CLs that definitely worsen overall code health of the system (Google)
- Review code hastily or for excessively long periods without breaks (Microsoft/SmartBear)
- Review more than 400 lines of code at a time without strong justification (SmartBear)
- Exceed 500 LOC per hour inspection rate or 60 minutes per session (SmartBear)
- Assume a risk does not apply without explicit verification (Microsoft)
- Rely on client-side controls alone for security mitigations (Microsoft)
- Treat passing a security scanner as equivalent to addressing security issues (Microsoft)
- Skip review for "small" or "urgent" changes (Microsoft)
- Accept "we'll fix it later" as a plan without explicit tracking (Microsoft)
- Use language that points fingers or assigns blame (Microsoft)
- Make vague, unactionable comments (Palantir)
- Give harsh, dismissive, or overly prescriptive feedback (Palantir)
- Focus only on mistakes without recognizing good practices (Palantir/Microsoft)
- Treat code review as a battleground of egos rather than quality improvement (Microsoft)
- Forget that the goal is a better product, not who made/found/fixed the bug (Microsoft)

## Process Integration

### With Development Workflow
- Code review should be a standard gate before merging to main branch
- Reviews should be tied to specific tasks/issues in tracking systems
- Review feedback should be addressed in the same PR unless doing so would violate principles
- Authors should be responsive to review comments in a timely manner
- Reviewers should be available to clarify their feedback when needed

### With Testing Strategy
- Review should verify that appropriate tests exist and are well-designed
- Test code should be reviewed with the same rigor as production code
- Review should check that tests are maintainable and not overly complex
- Property-based testing should be considered for data transformation logic
- Tests should validate both happy paths and error paths

### With CI/CD Pipeline
- Automated checks (linting, formatting, basic tests) should run before human review
- Code review should focus on aspects that automation cannot easily assess
- Review outcomes should feed into pipeline gating (approved changes can proceed)
- Review metrics should be tracked and visible in dashboard views

### With Team Culture
- Regular retrospectives should include discussion of code review effectiveness
- Training should be provided on both giving and receiving effective feedback
- Recognition should be given for consistently high-quality reviews
- Metrics should be used for process improvement, not individual performance evaluation

## Relationship to Other Wiki Pages

This page connects to several important wiki sections:
- Linus Torvalds review method — what to look for, in what order, and how to grade severity (wiki/tier2-core/code-review-method/overview.md)
- Code review checklist (wiki/tier3-working/checklists/code-review.md)
- OWASP Code Review Guide (wiki/tier3-working/owasp-code-review/overview.md)
- Security review checklist (wiki/tier3-working/checklists/security-review.md)
- Testing review checklist (wiki/tier3-working/checklists/testing-review.md)
- SWEBOK v4 KA12 on quality assurance (wiki/tier1-sources/swebok-v4/ka12-quality.md)
- SWEBOK v4 KA4 on construction (wiki/tier1-sources/swebok-v4/ka04-construction.md)
- Engineering playbook source control practices (wiki/tier2-core/engineering-playbook/source-control.md)
- Conventional commits specification (wiki/tier2-core/conventional-commits/specification.md)

Google's "approve when the change improves code health" and the Linus method's "say no early" combine as: reject invariant-false findings immediately; approve incremental improvements that do not violate invariants; never block on style nits. The Linus source persona's abusive tone is not adopted — ACM/IEEE Principle 7 and this page's culture rules win on how comments are written.

## See Also
- wiki/tier2-core/code-review-method/overview.md
- wiki/tier2-core/code-review-method/triggers.md
- wiki/tier3-working/checklists/code-review.md
- wiki/tier3-working/owasp-code-review/overview.md
- wiki/tier3-working/checklists/security-review.md
- wiki/tier3-working/checklists/testing-review.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md
- wiki/tier1-sources/swebok-v4/ka04-construction.md
- wiki/tier2-core/engineering-playbook/source-control.md
- wiki/tier2-core/conventional-commits/overview.md
- wiki/tier2-core/conventional-commits/specification.md

## Source
- Google. *Engineering Practices Documentation*. https://google.github.io/eng-practices/
- Microsoft. *Engineering Fundamentals Playbook - Code Reviews*. https://microsoft.github.io/code-with-engineering-playbook/
- SmartBear. *Best Practices for Peer Code Review*. https://smartbear.com/learn/code-review/best-practices-for-peer-code-review/
- Palantir. *Code Review Best Practices*. https://blog.palantir.com/code-review-best-practices-19e02780015f
- Thoughtbot. *Guides - Code Review*. https://github.com/thoughtbot/guides/blob/master/code-review/README.md