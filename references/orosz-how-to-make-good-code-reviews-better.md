# How to Make Good Code Reviews Better — Structured Reference

> Captured: 2026-10-01 from https://stackoverflow.blog/2019/09/30/how-to-make-good-code-reviews-better/
> Author: Gergely Orosz
> Republication: Stack Overflow Blog, 30 September 2019, "Code for a Living"
> Original venue: The Pragmatic Engineer (https://blog.pragmaticengineer.com/), as stated on the republication
> Context given by the author: engineering lead; reviews at Uber, and earlier at Skype/Microsoft and Skyscanner
> © Gergely Orosz / Stack Overflow. This file is a structured summary for wiki ingest. It is not a reproduction of the article.

Read-only after ingest. Re-synthesize `wiki/tier2-core/code-review-good-to-better/overview.md` from this summary. Do not paste the article back into the wiki.

## What the article is

A personal conduct guide that splits code review into two bars:

- **Good** — practices any team can start with, aimed at a correct, helpful review of the change.
- **Better** — the same review, adjusted to the codebase, the author, and the situation, and aimed at the author's productivity as well as the quality of the change. Repeated pain is fixed outside the review tool.

The author points at three other guides and does not replace them: Google's reviewer guide, SmartBear's code-review process guide, and Philipp Hauer's code-review guidelines. Those three are not captured in this file. Google's guide is already synthesized under `wiki/tier2-core/google-eng-practices/`.

## Benefits the article names

A second person checks the work, the reviewer learns from the solution, and the discussion improves the organization's tooling and automation. The article's opening advice is to start doing reviews if the organization does not.

## Good versus better, by topic

### What the review covers

- **Good:** the change and how it fits the codebase. Clarity of the title and description, and the reason for the change. Correctness, test coverage, behavior changes, and the written coding guides. Obvious problems: hard-to-understand code, unclear names, commented-out code, untested code, unhandled edge cases. A change that bundles too much should be split or kept to one purpose.
- **Better:** the change in the larger system, and whether it will stay maintainable. Questions about whether the change is necessary and how it affects other parts. New abstractions judged against the existing architecture. Maintainability notes: complex logic that could be simpler, test structure, duplication. The article cites Joel Kemp for a light first pass followed by a contextual pass.

### Tone

Harsh or opinionated comments make people defensive. A professional, positive tone makes constructive discussion possible.

- **Good:** open questions rather than strong statements. Alternatives offered without insisting they are the only way. The reviewer assumes they may be missing context and asks for clarification.
- **Better:** the reviewer remembers the effort already spent. Comments are kind and unassuming, and they name solutions that worked.

### Approve versus request changes

Statuses the article uses: approved, changes requested, or left without a status.

- **Good:** do not approve while open questions are unresolved. Mark non-blocking or unimportant comments so they are distinct. Say the approval in words (a short positive comment). Acronyms such as LGTM work, and newcomers can misread them. A request for follow-up is equally explicit, using the tool or the team convention.
- **Better:** firm on the principle, flexible on the practice. Some comments are handled in a separate follow-up change. For more urgent changes, the reviewer makes themselves available sooner.

### When to stop typing and talk

Reviews are usually asynchronous so people can work remotely and so several people can review one change.

- **Good:** leave the comments the review needs. If a revision misses some, say so. A long back-and-forth moves to a direct conversation.
- **Better:** after a first pass that produced many comments, contact the author before another written round. Many comments usually mean a misunderstanding, which a conversation resolves with less time and less friction.

### Nitpicks

A nitpick is a comment the change could merge without addressing: declaration order, a preferred test layout, bracket placement.

- **Good:** mark them, conventionally with a `nit:` prefix. Too many of them crowd out the important comments.
- **Better:** a stream of the same nits means missing tooling or a missing standard. Fix that outside the review. Lint the ones a tool can catch. Agree, and later automate, the ones a tool cannot.

### New joiners

- **Good:** one quality bar for everyone, independent of title, level, or start date. Kind tone, request changes where needed, and talk when there are many comments.
- **Better:** the first reviews for a new joiner get extra care. They may not know the guides or the code. Explain alternatives, point at the guides, and mark early changes positively. The bar does not drop.

### Cross-office and cross-time-zone reviews

- **Good:** review during overlapping hours when possible. When there are many comments, offer a direct conversation or a video call.
- **Better:** repeated timezone pain is a system problem. Ask why one site keeps changing a service owned somewhere else, whether the change belongs in that codebase, and whether the rate will fall. If the dependency is real and durable, look for a way to break it: a refactor, a new service or interface, or tooling. The article treats that investment as high leverage.

### Organizational support

- **Good:** every engineer reviews, including people on solo work. The organization encourages a higher bar and discussion of how reviews are done. Larger codebases get written review guides that engineers wrote. Review time is expected, and senior engineers are expected to spend more of their time reviewing.
- **Better:** no code reaches production without a review, in the same way business-logic changes do not reach production without automated tests. Urgent work has an expedited review path. The organization keeps investing in review tooling and process. People can raise a hostile review and get support. Senior engineers and managers treat a review that misses the bar as seriously as sloppy code or poor behavior. Engineers and managers are able to change how reviews work.

## Closing claim

Good reviews are already a high bar: thorough on the change, unopinionated in tone, explicit about nits, one bar for every author, extra care across time zones, and everyone both receiving and doing reviews. Better reviews add the high-level pass, an empathetic tone, removal of repeated nits outside the review, a welcoming first experience for new people, systemic fixes for timezone pain, and organizational investment in tooling and process.

## Companion links named by the article (not ingested here)

- Google's reviewer guide: https://google.github.io/eng-practices/review/reviewer/ (wiki: `wiki/tier2-core/google-eng-practices/`)
- SmartBear, guide to the code review process: https://smartbear.com/learn/code-review/guide-to-code-review-process/
- Philipp Hauer, code review guidelines: https://phauer.com/2018/code-review-guidelines
- Joel Kemp, giving better code reviews (light pass, then contextual pass): https://medium.com/@mrjoelkemp/giving-better-code-reviews-16109e0fdd36
- The Pragmatic Engineer, pull-request and diff practices, including the "why" of a change: https://blog.pragmaticengineer.com/pull-request-or-diff-best-practices/
