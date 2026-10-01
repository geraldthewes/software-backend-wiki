# Speed of Code Reviews (Tier 2)

> **Tier 2** | Source: Google Engineering Practices, `review/reviewer/speed.md` | Authority: established | Derives From: SWEBOK KA12, KA9

## Summary

Optimize for how fast a team produces a product together, not for how fast one person types. Slow reviews delay everyone else's features and fixes, make authors describe the process as "too strict," and create pressure to merge changes that are worse than they should be. Slow reviews also discourage cleanups. Most complaints about strictness go away when the same strict comments come back quickly.

Speed here means **response time**, not how long the whole review takes. Quick individual responses matter more than a fast end-to-end, because they remove the frustration of silence. The whole process should still be fast: a typical change should be able to go through multiple rounds in one day.

## Key Concepts

### Response targets

| Situation | Expected response |
|-----------|-------------------|
| You are not in the middle of a focused task | Review shortly after the request arrives. |
| You are writing code or otherwise in flow | Do not interrupt. Respond at a breakpoint: task done, after lunch, after a meeting. Interrupting flow costs the team more than a short wait. |
| Maximum | One business day. First thing the next morning is the limit, not the goal. |
| You cannot do a full review now | At a breakpoint, say when you will, suggest another reviewer, or send the broad design comments from `navigating-a-cl.md`. |
| Cross-timezone | Reply while the author still has working hours left to respond. If their day is over, finish before they start the next day. |

An LGTM still means the change meets `review-standard.md`. Speed does not lower that bar. Do not trade the standard for an imagined gain in velocity; over time that trade does not make the team faster.

### LGTM with comments

Approve while leaving unresolved comments when at least one of these is true:

1. You are confident the author will address the remaining comments appropriately.
2. The comments do not have to be addressed.
3. The suggestions are minor: sort imports, fix a nearby typo, apply an obvious fix, drop an unused dependency.

Say which of the three you mean. This is especially useful across timezones, where waiting another day only to receive approval wastes a day.

### Large changes

If you cannot tell when you will have time to review a change because of its size, ask the author to split it into smaller changes that build on each other (`small-cls.md`). If it truly cannot be split and you cannot review all of it quickly, comment on the overall design and send it back. The goal is to unblock the author, or to give them a next action, without sacrificing code health.

### What speed does over time

Strict, fast reviews get faster. Authors learn what healthy code looks like and send better changes. Reviewers stop adding latency. Lowering the standard to "go faster" does the opposite over the long run.

True emergencies, where the entire process must be fast and the quality bar relaxes, are defined only in `emergencies.md`.

## Agent Guidance

### Do

- Respond within one business day, and sooner when you are between tasks.
- Send a scheduling note or a design-level pass when a full review will not fit today.
- Use LGTM-with-comments for minor or clearly-owned follow-ups, and say which case applies.
- Ask for a split before a large change sits in your queue.

### Do Not

- Do not interrupt focused implementation to review. Wait for a breakpoint.
- Do not approve quickly by skipping the standard.
- Do not leave an author blocked with no next action.
- Do not treat a slow, strict review as a virtue. The strictness stays; the latency goes.

## Checklist

- [ ] First response is within one business day, sent at a breakpoint rather than mid-flow
- [ ] The response either reviews, schedules the review, suggests another reviewer, or returns design comments
- [ ] Any LGTM still means the review standard is met
- [ ] LGTM-with-comments states whether the leftovers are required, optional, or minor
- [ ] A change too large to review promptly was sent back to be split
- [ ] Timezone delay was reduced by replying inside the author's remaining hours when possible

## See Also

- wiki/tier2-core/google-eng-practices/overview.md
- wiki/tier2-core/google-eng-practices/review-standard.md
- wiki/tier2-core/google-eng-practices/navigating-a-cl.md
- wiki/tier2-core/google-eng-practices/small-cls.md
- wiki/tier2-core/google-eng-practices/emergencies.md
- wiki/tier2-core/google-eng-practices/handling-pushback.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md
- wiki/tier1-sources/swebok-v4/ka09-engineering-management.md

## Source

Google. *Speed of Code Reviews*. Engineering Practices Documentation. CC BY 3.0. https://google.github.io/eng-practices/review/reviewer/speed.html

Local immutable capture: `references/google-eng-practices/review/reviewer/speed.md`. Snapshot `3bb3ec25b3b0199f4940b1aa75f0ac5c5753301c`.
