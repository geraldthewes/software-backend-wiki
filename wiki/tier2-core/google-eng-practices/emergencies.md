# Emergencies (Tier 2)

> **Tier 2** | Source: Google Engineering Practices, `review/emergencies.md` | Authority: established | Derives From: SWEBOK KA12, KA6, KA7

## Summary

An emergency is the only case in which a reviewer cares more about speed of the whole review, and about whether the change actually resolves the emergency, than about the rest of the code-health standard. The change still has to be small, and it still has to be correct for the emergency it claims to fix. After the emergency is over, review that change again against `what-to-look-for.md`.

Calling something an emergency to skip review quality is how projects accumulate technical debt. Most deadlines are not emergencies.

## Key Concepts

### What qualifies

An emergency change is **small**, and it does something like:

- lets a major launch continue instead of rolling back
- fixes a bug that significantly affects users in production
- handles a pressing legal issue
- closes a major security hole

These reviews take priority over every other review. The question that outranks polish, naming, and structural cleanup is: does this change actually resolve the emergency, and is that fix correct? Speed of the entire process matters here, not only speed of the first response (`review-speed.md`).

When the incident is over, give the emergency change a thorough review and clean up whatever the shortcut left behind.

### What does not qualify

- Wanting to launch this week rather than next week, unless a hard deadline is real.
- The author has worked on the feature for a long time and wants it merged.
- Reviewers are in another timezone, asleep, or offsite.
- It is Friday afternoon and it would be nice to merge before the weekend.
- A manager says it must merge today because of a soft deadline.
- Rolling back a change that is causing test failures or build breakages.

### Hard deadlines and release cycles

A hard deadline is one where something disastrous happens if you miss it. Examples from the source: a contractual obligation, a product that will fail in the market if it is not released by a date, or a hardware partner who ships once a year and will not take the code again until the next cycle. Slipping a release by a week is not disastrous. Missing a conference might be, and often is not.

Soft deadlines are desires. They matter. They are not a reason to sacrifice code health.

On a cycle of several weeks, the temptation is to waive review quality so a feature catches the cycle. Repeated, that is a standard way to build overwhelming technical debt. If "must get in" changes with only a superficial review show up at the end of every cycle, change the process so large feature work starts early enough to be reviewed properly.

## Agent Guidance

### Do

- Treat production-impacting bugs, major security holes, pressing legal issues, and launch-blocking fixes as emergencies when the change is small.
- Review the emergency for "does this fix the emergency, correctly?" first, and review it ahead of ordinary changes.
- Schedule a follow-up review against the normal standard once the emergency has passed.
- Move large feature work earlier in the cycle when end-of-cycle waivers are becoming normal.

### Do Not

- Do not label timezone inconvenience, Friday, author fatigue, a soft manager deadline, or a test-breaking rollback as an emergency.
- Do not use an emergency to land a large change.
- Do not skip the follow-up review.
- Do not relax the standard for a launch that can slip a week without disaster.

## Checklist

- [ ] The situation matches the emergency list, and the change is small
- [ ] The review answered whether the change actually resolves the emergency and is correct
- [ ] The review was prioritized over ordinary reviews
- [ ] A follow-up review on the normal standard is tracked
- [ ] Soft deadlines and end-of-cycle pressure were refused as emergency justifications
- [ ] Repeated end-of-cycle waivers were treated as a process bug, not as a series of emergencies

## See Also

- wiki/tier2-core/google-eng-practices/overview.md
- wiki/tier2-core/google-eng-practices/review-standard.md
- wiki/tier2-core/google-eng-practices/review-speed.md
- wiki/tier2-core/google-eng-practices/what-to-look-for.md
- wiki/tier2-core/google-eng-practices/handling-pushback.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md
- wiki/tier1-sources/swebok-v4/ka06-operations.md
- wiki/tier1-sources/swebok-v4/ka07-maintenance.md

## Source

Google. *Emergencies*. Engineering Practices Documentation. CC BY 3.0. https://google.github.io/eng-practices/review/emergencies.html

Local immutable capture: `references/google-eng-practices/review/emergencies.md`. Snapshot `3bb3ec25b3b0199f4940b1aa75f0ac5c5753301c`.
