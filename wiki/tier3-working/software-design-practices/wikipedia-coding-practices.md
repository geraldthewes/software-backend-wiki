# Wikipedia Coding Best Practices (Tier 3)

> **Tier 3** | Source: `references/software-design-practices/wikipedia-coding-best-practices.md` | Authority: working | Derives From: SWEBOK KA1, KA2, KA3, KA4, KA12

## Summary

The Wikipedia article "Coding best practices" is a tertiary survey under CC BY-SA 4.0. At capture it carried two maintenance banners: thin inline citations since July 2012, and a tone flagged since April 2008. Use it as a map onto SWEBOK, not as an authority beside SWEBOK.

The part worth keeping is the order of work and the warning that a stated goal crowds out the goals nobody stated. Naming, comments, config, and deployment on the article lose to PEP 8, 12-factor, and the operations pages wherever they differ.

## Key Concepts

### Order of work

The article, citing Meek and Heath, says the work before coding decides the outcome. Its sequence is the SWEBOK sequence:

| Step | Question | Authority |
|------|----------|-----------|
| Life cycle | How is the work structured? | KA10. Requirements move during a project. The article's "about 25%" figure is McConnell's, as Wikipedia reports it, not a quota |
| Requirements | What problem is solved, functionally and non-functionally? | KA1 |
| Architecture | Which component does what? | KA2. Hoare's test, as the article reports it: a design so simple that a deficiency would be obvious |
| Design | Which algorithm, attribute, and method? | KA3 |
| Language | Which language fits, if it is not already chosen? | Usually already Python or Go in this wiki. Know that language's weak spots either way |
| Conventions | Which style, before the first file? | PEP 8 for Python. Retrofitting style is expensive |

A small program may collapse architecture and design into one step.

### Quality goals

Attributes compete. Fast and heavily checked is the article's example. Sommerville's four, as the article lists them: maintainability, dependability, efficiency, usability. Weinberg's four questions: meets the specification, on time and budget, adaptable, efficient enough for the environment. Hoare's longer list on the article adds ruggedness, extensibility from experience, brevity, conformance to language standards, and accurate user documents.

KA12 remains the quality model. The actionable claim from Weinberg, as the article reports it: programmers hit the explicit goal and drop the others. Name the attribute this change is optimizing.

### Points that defer to an existing page

| Article says | Where it is decided |
|--------------|---------------------|
| Descriptive names; the article prefers Pascal case in its truck-weight example | PEP 8 `snake_case`. No Hungarian notation |
| Comment what a variable is for; old file headers with an author log | The name carries the what. Comment the why. Version control replaces the author log (KA8) |
| No hardcoded paths, hosts, ports, or user names | 12-factor III and `xdg-base-directory/overview.md` |
| Design for later features and reuse tested components | YAGNI and the rule of three still gate new abstraction. KA7 and KA15 cover debt and reuse cost |
| Daily build, CI, or continuous delivery | KA6 and KA8 |
| Plan tests before and during coding | KA5 and `python-testing/overview.md`. The article gives no coverage number |
| Debug a module when the module is done | Test pyramid. Do not save all debugging for the end of a large change |
| Eleven deployment concerns (lean install, rollback, automate, production-like rehearsal, do not edit the procedure mid-deploy) | KA6. Concerns, not a new deploy design |

The article's see-also list (MISRA, *The Cathedral and the Bazaar*, Davis's *201 Principles*, *Don't Make Me Think*, and others) was not read. A title on that list is not a rule.

The ninety-ninety rule is an observation about estimation, not a scheduling method. KA9 and KA15 own estimation.

## Agent Guidance

### Do

- Name the quality attribute the change is for, before choosing a structure that trades another attribute away.
- Confirm requirements and the component boundary before detailed design. Use the KA pages for the content of those steps.
- Set style conventions before adding files.
- Treat portability literals (hosts, ports, absolute paths, user names) as configuration.

### Do Not

- Do not cite this article against a SWEBOK page, a PEP, or a Tier 2 page.
- Do not adopt Pascal case or per-variable comments for Python from the article's examples.
- Do not treat the deployment list as a second operations standard.
- Do not treat the missing citations as a reason to invent numbers the article does not contain. The 25% requirements-change figure stays attributed to McConnell via Wikipedia, and it is not a planning constant.

## Checklist

- [ ] The quality attribute this change optimizes is named
- [ ] Requirements and component boundaries were checked before new types were added
- [ ] No Python name was styled from the article's Pascal-case example
- [ ] Hosts, ports, paths, and user names are configuration
- [ ] Deployment and test strategy come from KA6 and KA5, not from this survey

## See Also

- wiki/tier3-working/software-design-practices/overview.md
- wiki/tier1-sources/swebok-v4/ka01-requirements.md
- wiki/tier1-sources/swebok-v4/ka02-architecture.md
- wiki/tier1-sources/swebok-v4/ka03-design.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md
- wiki/tier1-sources/python-peps/pep-008-style.md
- wiki/tier2-core/twelve-factor-app/factors.md
- wiki/tier3-working/software-design-practices/foord-30-practices.md

## Source

Wikipedia contributors, "Coding best practices", https://en.wikipedia.org/wiki/Coding_best_practices, CC BY-SA 4.0. The article cites Steve McConnell, *Code Complete*; Ian Sommerville, *Software Engineering* (7th ed.); Gerald Weinberg, *The Psychology of Computer Programming*; C. A. R. Hoare; Jon Bentley, "Programming pearls: Bumper-Sticker Computer Science", Communications of the ACM 28(9), 1985. Summary in `references/software-design-practices/wikipedia-coding-best-practices.md`.
