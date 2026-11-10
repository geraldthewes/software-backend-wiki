# Wikipedia — Coding best practices

> Captured: 2026-11-09 from https://en.wikipedia.org/wiki/Coding_best_practices
> Wikipedia article "Coding best practices". Text is available under CC BY-SA 4.0.
> https://creativecommons.org/licenses/by-sa/4.0/
> Structured summary. Not a reproduction. Do not paste the article back into the wiki.
>
> Maintenance banners on the article at capture time:
> - Insufficient inline citations, flagged since July 2012.
> - Writing style may not match an encyclopedic tone, flagged since April 2008.
> The article is a tertiary survey. Where it restates a SWEBOK knowledge area, the KA page wins.

Read-only after ingest. Re-synthesize `wiki/tier3-working/software-design-practices/wikipedia-coding-practices.md` from this summary.

## Definition the article uses

Coding best practices, also called programming best practices, are informal and sometimes personal rules that developers follow to improve software quality. Programs that must stay reliable for a long time need rules that help the first author and every later maintainer.

The article cites Tom Cargill's ninety-ninety rule, via Jon Bentley (1985): the first 90% of the code takes the first 90% of the time, and the last 10% takes another 90%. It also says project size changes error rates, productivity, and how much management is required, citing McConnell, *Code Complete*, second edition, pages 649–659.

## Software quality

The article points at the software-quality article and makes three claims an agent can use:

- Quality attributes can contradict each other. Very fast and heavily checked are the example. Customers do not rank them the same way.
- Gerald Weinberg (*The Psychology of Computer Programming*, 1998 silver anniversary edition) is cited for the claim that programmers will aim at whatever goal is stated explicitly, and will drop other attributes to hit it. State the quality goal.
- Ian Sommerville (*Software Engineering*, 7th ed., 2004) is summarized as four attributes of how well a program does its job: maintainability, dependability, efficiency, and usability.
- Weinberg's four questions, as the article states them: does it meet its specification for every input; was it produced on schedule and budget; can it adapt when requirements change; is it efficient enough for where it runs.
- Tony Hoare is credited with a longer list. The article includes: a clear purpose, simplicity of use, ruggedness (hard to misuse, kind to errors), early availability, reliability, extensibility in the light of experience, brevity, efficiency enough for the purpose, minimum development cost, conformity to relevant standards including language standards, and clear accurate user documents.

SWEBOK KA12 is the quality authority. This list is a reminder to name which attributes matter for the change. It does not add a new quality model.

## Prerequisites

The article says, citing Meek and Heath, that what happens before coding often decides whether the project succeeds. If the prerequisites are thin, finishing the code does not rescue it. For a small program, architecture and design may be one step, and the life cycle may be simple.

The five prerequisites:

1. Life cycle. How the work is structured. The article lists waterfall, prototyping, iterative and incremental, spiral, agile, RAD, and extreme programming. It says waterfall assumes requirements can be fixed up front, and cites McConnell that requirements change by about 25% on average during a project. The other named methods exist to absorb that change. Agile's growth after 2001 is noted. SWEBOK KA10 and `engineering-playbook/agile-development.md` are the process pages. Do not pick a life cycle from this article.
2. Requirements. McConnell is quoted in substance: before construction, state the problem the system solves. Meek and Heath want that statement clear, complete, precise, and unambiguous, and the article notes the target moves. Sommerville's split: user requirements versus system requirements, and functional versus non-functional (the article's example of a non-functional requirement is a response time under one second). Authority: SWEBOK KA1.
3. Architecture. Hoare is quoted in substance: there are two ways to design, so simple that there are obviously no deficiencies, or so complicated that there are no obvious deficiencies. The first is harder. Architecture decides what is done and which component does it. How it is done is design. Interfaces between programs, including a sketch of the user interface, belong here, as do non-functional requirements. Stakeholders use the architecture to see whether their requirements can be met. Authority: SWEBOK KA2.
4. Design. Fill in what architecture left open, far enough to guide coding, including which algorithm is used. The article's examples: architecture says "sort", design picks the algorithm; an object-oriented design names attributes and methods. Authority: SWEBOK KA3.
5. Language. No language is best in general. Start from the problem and the relative importance of its requirements, then pick the least bad fit. Different parts of a problem may justify different languages, including mixed-language routines when the toolchain allows it. If the language is already chosen, know its strengths and weaknesses. This wiki's task is usually Python or Go already chosen. Do not reopen the language choice from this article.

## Coding standards

McConnell, as paraphrased: set the conventions before programming starts, because retrofitting them is nearly impossible. Conventions differ by language, so one house style across languages can be harmful. There is no single convention per language; an organization picks one per project. Shared conventions matter once more than one person writes the code. The article points at Roedy Green's satirical "unmaintainable code" piece as a negative example. Do not follow that piece.

### Comments

The article says time pressure pushes comments aside, and that a team still benefits from them because work is cyclical and more than one person touches a module. An old file-header practice (module name, purpose, description, original author, modification log) is described, and then the article says the author and modification log are obsolete because version control records them. A comment block next to complicated logic is recommended so the next person can see what is happening. Unit tests are named as another way to show intended use.

Foord, the IST guide, and PEP 8 are stricter: comment why, not what, and prefer a name to a comment that restates the code. Version control replacing a modification log agrees with those sources and with KA8. The "comment every variable" mood in the construction list below loses to PEP 8.

### Names

Descriptive names. The article warns against leftovers such as `X1` and `Y1`. Its example of a truck-weight parameter offers `TrkWeight`, `TruckWeightKilograms`, and `Truck_Weight_Kilograms`, and calls Pascal case often preferable. For Python that loses to PEP 8: `truck_weight_kilograms`. Hungarian notation is mentioned only as a see-also. Do not introduce it.

### Simplicity

The logic should be as simple as a later programmer will need it to be. Complicated logic for a simple job is a defect because the next author will not reconstruct the first author's intent. The article cites a generic "keep it simple" source here. PEP 20 and KA3 already say this.

### Portability

No literal absolute paths, file names, user names, host names, IP addresses, URLs, or port numbers in program code. Parameterize them outside the program (the article suggests property files, an application server, or a database) so there is a single point of definition. The same rule for resources such as XML files. This is 12-factor III (config) plus the XDG page for local paths. OWASP A05 covers unsafe defaults. Do not invent a second config mechanism from this paragraph.

### Scalability and reuse

Design so a feature can be added. Reuse of an already tested component cuts cost and time. Projects often start from a previous baseline and reuse modules. These are aims, not a license to build a framework before the second caller exists. YAGNI (Foord) and the rule of three still gate extraction. SWEBOK KA7 and KA15 cover debt and reuse economics.

### Construction list at the end of that section

The article's own short list: know what the block must do; keep names uniform; indicate what a variable is for; fix errors as they occur; keep the code simple; design for scale and reuse. Item "indicate what a variable is for" is satisfied by the name. A comment on every variable is not required.

## Code development

- Building. Daily builds and tests, or continuous integration, or continuous delivery. Authority: SWEBOK KA6 and KA8. The article does not specify a CI product.
- Testing. Plan it. The article says test cases should be planned before coding and developed while the application is designed and coded. Authority: SWEBOK KA5 and the testing pages. Do not import a coverage number from this article. It does not give one.
- Debugging. Writing the entire program and then debugging is described as tolerable for a small project and expensive for a large one. Debug a module when that module is done. Unit tests for modules, and functional tests for web services and web applications, are the article's mechanism. The test pyramid is finer than that split and wins.
- Deployment. The article's list, paraphrased, is deferred to existing pages rather than adopted as a new policy:
  1. Install only files that are used. Keep the tree small.
  2. Archive unused resources elsewhere so the build stays lean (KA8).
  3. Deploy from current versions. If unsure, deploy from scratch rather than an uncertain delta.
  4. More than one stage when the project is large enough.
  5. A rollback to a known-good version.
  6. Automate the repeatable steps. Manual deployment is where human error lands.
  7. The rehearsal environment includes the real network path, firewalls, servers, browsers, and filesystems.
  8. Do not edit the deploy procedure in the middle of a deploy. Record the change and apply it on the next iteration.
  9. APIs and microservices need their own deploy considerations. The article does not specify them.
  10. Bad testing or bad configuration management makes the deploy fail.
  11. Stakeholders outside engineering (organizational, social, governmental) affect the deploy.

  Operational authority remains SWEBOK KA6, 12-factor, and `observability/slo-sli-sla.md`. This list is a checklist of concerns, not a deploy design.

## See-also topics on the article that this ingest does not adopt

Best practice (the general phrase), static-analysis tool lists, MISRA, software assurance, the Cathedral and the Bazaar, Davis's *201 Principles of Software Development*, a paper titled "Where's the Theory for Software Engineering?", and *Don't Make Me Think*. They are named on the article. They were not read. Do not treat a name on that list as a rule.
