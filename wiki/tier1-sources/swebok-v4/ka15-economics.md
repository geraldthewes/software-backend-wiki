# Software Engineering Economics (KA15)

> **Tier 1** | Source: SWEBOK V4, Chapter 15 | Authority: immutable

## Summary

Software Engineering Economics establishes that every technical decision is also an economic decision. Architecture choices, build-vs-buy decisions, quality investments, and technical debt acceptance all have financial consequences that can be analyzed, quantified, and optimized. Economic literacy separates engineers who can reason about trade-offs from those who can only implement specifications.

For agents, this KA provides the analytical vocabulary for framing technical recommendations in terms stakeholders can act on. "This design has lower latency" is less actionable than "This design requires 40% more infrastructure cost but reduces p99 latency by 120ms, which avoids contract penalties under our SLA." Economic reasoning transforms technical options into business decisions.

## Key Concepts

### Cost Estimation Models

**COCOMO II (Constructive Cost Model)**: Algorithmic model for estimating development effort from software size (lines of code or function points) with calibration factors:
- **Scale factors**: Process maturity, architectural precedent, team cohesion, risk resolution
- **Cost drivers**: Personnel capability, tool use, reliability requirements, complexity
- Best used with historical calibration data; generic parameters yield wide uncertainty ranges

**Function Point Analysis**: Technology-independent size measure based on counting: external inputs, external outputs, external inquiries, internal logical files, external interface files — each weighted by complexity.
- Can be estimated before any code is written
- Enables comparison across languages and platforms
- Basis for many commercial estimation tools

**Use-Case Points**: Extends function points for object-oriented systems; weights use cases and actors by complexity. More natural for teams already writing use cases.

**Estimation uncertainty**: All models yield ranges, not point estimates. The Cone of Uncertainty means early estimates can be off by 4x in either direction. Communicate this uncertainty explicitly.

### ROI Calculation

**Basic ROI**:
```
ROI = (Benefits - Costs) / Costs × 100%
```

**Net Present Value (NPV)**: Accounts for time value of money — a dollar today is worth more than a dollar in 3 years:
```
NPV = Σ (Cash Flow_t / (1 + discount_rate)^t)
```

For long-lived systems (5+ year horizon), NPV analysis is more accurate than simple ROI because it properly values the cost of delayed returns and the future maintenance burden.

**Payback period**: Time until cumulative benefits exceed cumulative costs. Simple to communicate; ignores time value and post-payback benefits.

### Make-vs-Buy Analysis

Every capability needed by a system should be explicitly evaluated against four options:
1. **Build in-house**: Full control, full cost, full maintenance burden
2. **Buy commercial**: Immediate capability, licensing cost, vendor dependency, less flexibility
3. **Open source**: Low acquisition cost, must assess license, support, and maintenance risk
4. **Outsource**: Transfer development and operation to a third party

**Evaluation criteria**:
- **Strategic value**: Does this capability differentiate the product, or is it commodity infrastructure? Build what differentiates; buy what doesn't.
- **Expertise**: Does the team have the knowledge to build it well? Expertise gaps add hidden cost and quality risk.
- **Total cost**: Acquisition cost + integration cost + ongoing maintenance cost + vendor lock-in risk
- **Time to value**: How long does each option take to produce usable capability?
- **Risk**: Vendor viability, license change risk, security responsibility, operational complexity

**Common mistake**: Underestimating the long-term maintenance cost of in-house solutions. Building is cheap; maintaining is expensive.

### Technical Debt Economics

Ward Cunningham's original metaphor (1992): Taking a shortcut to ship faster is like borrowing money — you get the immediate benefit, but you pay interest on the debt until you repay the principal.

**Principal**: The cost of the remediation work needed to fix the shortcut (the thing you must eventually do)

**Interest**: The extra effort every future change costs because of the shortcut (slower development, higher defect rate, harder onboarding)

**Debt accumulation**: Unrepaid debt compounds. A system carrying heavy technical debt can reach a state where the interest payments (slow development, constant firefighting) consume all available capacity — "technical bankruptcy."

**Debt quantification**: Tools like SonarQube estimate remediation effort in developer-hours. While imprecise, quantification makes debt visible and enables prioritization.

**Cunningham's original intent**: Cunningham intended the metaphor to describe *deliberate, acknowledged* debt taken on with a plan to repay — not as justification for indefinite poor practice. Reckless debt (shortcuts taken without acknowledgment or repayment plan) was never the intent.

### Value Analysis

**Minimum Viable Product (MVP)**: The smallest version of a product that delivers enough value to validate the core hypothesis and begin learning from real users. An MVP is a learning instrument, not a half-finished product.

**Feature prioritization methods**:
- **WSJF (Weighted Shortest Job First)**: `WSJF = Cost of Delay / Job Duration`. Prioritizes features where the cost of waiting is highest relative to the effort to deliver. Used in SAFe.
- **Kano model**: Classifies features as basic (must-have), performance (more is better), or excitement (unexpected delight). Guides where to invest for maximum customer satisfaction impact.
- **MoSCoW**: Must-have, Should-have, Could-have, Won't-have. Simple prioritization for sprint planning.

### Cost of Quality

**Prevention costs**: Investment in preventing defects — training, good design process, code reviews, static analysis tools. Highest ROI quality investment.

**Appraisal costs**: Inspection and testing to detect defects before release — test execution, security audits, performance testing.

**Failure costs**:
- **Internal**: Defects found before release — rework, redesign, retest
- **External**: Defects found after release — customer support, refunds, reputational damage, regulatory penalties, security breach costs

The Cost of Quality model shows that investing in prevention and appraisal is almost always cheaper than paying external failure costs. A defect caught in code review costs ~10x less to fix than one caught in testing, and ~100x less than one found in production.

## Agent Guidance

### Do
- Surface the economic consequences of technical decisions — not just the technical trade-offs
- Apply make-vs-buy criteria explicitly when recommending build vs. buy vs. open source
- Quantify technical debt in remediation effort when recommending whether to repay it now or later
- Use WSJF or Kano to frame feature prioritization discussions in business terms
- Communicate estimates as ranges with explicit uncertainty, not false-precision point values
- Frame quality investment in terms of cost of quality — prevention is cheaper than failure

### Do Not
- Gold-plate: implement features beyond what is specified or required
- Accept unlimited technical debt without surfacing the compounding interest cost
- Recommend building in-house without explicitly analyzing the long-term maintenance burden
- Present cost estimates as commitments before requirements are adequately defined
- Optimize for development cost while ignoring operational and maintenance costs

## Checklist
- [ ] Make-vs-buy analysis documented for all significant capability decisions
- [ ] Technical debt items quantified in estimated remediation effort
- [ ] Cost estimates provided as ranges, not point values
- [ ] NPV analysis applied to decisions with multi-year financial impact
- [ ] Feature priority justified using WSJF, Kano, or equivalent method
- [ ] Cost of quality framing used when justifying quality investments
- [ ] Build-in-house recommendations include long-term maintenance cost estimate

## See Also
- `wiki/tier1-sources/swebok-v4/ka09-engineering-management.md`
- `wiki/tier1-sources/swebok-v4/ka07-maintenance.md`
- `wiki/tier1-sources/swebok-v4/ka12-quality.md`

## Source
IEEE Computer Society. *Guide to the Software Engineering Body of Knowledge (SWEBOK), Version 4*. Chapter 15: Software Engineering Economics. IEEE Press, 2024.
