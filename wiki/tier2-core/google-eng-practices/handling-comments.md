# How to Handle Reviewer Comments (Tier 2)

> **Tier 2** | Source: Google Engineering Practices, `review/developer/handling-comments.md` | Authority: established | Derives From: SWEBOK KA12, KA14; ACM/IEEE-CS Code of Ethics Principle 7

## Summary

Review comments are an attempt to protect the codebase, not a judgment of the author. The author's first moves are: do not take it personally, do not reply in anger, and fix the code so the next reader understands it. If you disagree, disagree with the tradeoff in front of you, not with the reviewer's right to ask.

## Key Concepts

### Do not take it personally

Read a critique as help for you, the codebase, and the product. Reviewers sometimes write while frustrated. That is a bad reviewer habit (`review-comments.md` forbids it), and the author still has to extract the constructive request and answer that.

Never answer a review comment in anger. The reply stays in the review tool. If you cannot reply kindly yet, step away until you can.

If a reviewer is not constructive or polite, say so in person or on a video call. If that is impossible, send a private email. Describe what you want them to do differently, kindly. If the private conversation also goes badly or changes nothing, escalate to a manager. Do not litigate their tone in the review thread.

### Fix the code

When a reviewer does not understand something, the first response is to clarify the code. If the code cannot be made clear, add a comment that explains why it is there. Only when a comment would be pointless do you answer solely in the review tool.

A thread reply does not help the next reader. A clearer implementation or a code comment does. The same rule binds the reviewer: see "Accepting explanations" in `review-comments.md`.

### Think collaboratively

It is normal to feel finished when the change goes out, and frustrating to get more work back. Before arguing, check that you understand what was asked. If you do not, ask.

If you understand and disagree, discuss the tradeoff:

- State the option you chose, its pros, cons, and tradeoffs.
- State why you believe the reviewer's option is worse.
- Ask whether they want the other option because it serves the original tradeoff better, because the tradeoff should be weighed differently, or for some other reason.

Courtesy is the first priority. Bring facts the reviewer may not have: users, codebase history, constraints of this change. Consensus usually follows from those facts.

### Conflicts

Try for consensus first. If you cannot get there, follow `review-standard.md`: the principles in that page, then a conversation recorded on the change, then escalation. Do not let the change sit.

## Agent Guidance

### Do

- Translate a harsh comment into the technical request and answer that request.
- Change the code, or add a why-comment, before writing a long thread explanation.
- When disagreeing, write the tradeoff and ask which part the reviewer is challenging.
- Take tone problems to a private channel, then to a manager if needed.

### Do Not

- Do not reply while angry. The record is permanent.
- Do not answer "No, I'm not going to do that" with no tradeoff.
- Do not treat a thread explanation as the fix for code the reviewer could not read.
- Do not argue about the reviewer's manners inside the review thread.

## Checklist

- [ ] The reply addresses the technical request, including when the comment was blunt
- [ ] The reply was sent only once it could be polite
- [ ] Unclear code was clarified in the code or with a why-comment
- [ ] Disagreement states the chosen tradeoff and asks what the reviewer wants weighed differently
- [ ] Tone problems were handled privately, not in the review thread
- [ ] Unresolved conflict is on the path in `review-standard.md`

## See Also

- wiki/tier2-core/google-eng-practices/overview.md
- wiki/tier2-core/google-eng-practices/review-comments.md
- wiki/tier2-core/google-eng-practices/review-standard.md
- wiki/tier2-core/google-eng-practices/handling-pushback.md
- wiki/tier1-sources/acm-ieee-ethics/code-of-ethics.md
- wiki/tier1-sources/swebok-v4/ka14-professional-practice.md
- wiki/tier1-sources/swebok-v4/ka12-quality.md
- wiki/tier2-core/code-review-method/overview.md

## Source

Google. *How to handle reviewer comments*. Engineering Practices Documentation. CC BY 3.0. https://google.github.io/eng-practices/review/developer/handling-comments.html

Local immutable capture: `references/google-eng-practices/review/developer/handling-comments.md`. Snapshot `3bb3ec25b3b0199f4940b1aa75f0ac5c5753301c`.
