# Platform Target State and Principles — Architect Guide

## What you will produce and why

Architect produces one document for their area: a two-page target state and three to six platform principles. Together these become our platform direction.

**Why it matters.** The business owns outcomes, product owns priorities, and engineering owns delivery. Architecture owns what the platforms should become and in what order. Without a written target state, that direction gets set one project at a time, in design reviews, by whoever is in the room.

**What it is used for:**

- Shaping product prioritization with sequencing options and business cases
- Judging new designs and exceptions against an agreed direction
- Explaining to business and engineering leaders where the platforms are going and why
- Moving from application thinking to platform thinking

**What it is not:** a project plan, a list of systems, or an enterprise standard. It does not replace product's roadmap. It should make that roadmap easier to sequence.

## How to build it

Work outside-in: start with the business, then the capabilities, and only then the systems. Plan for about two to three days of effort over four weeks.

1. **Listen to the business (week 1).** Spend 30 minutes each with the business owner and the product owner for your area. Ask: What outcomes matter most over two years? Where does technology cause the most pain? What is coming: new products, volume, regulation?
2. **Describe the capabilities, not the systems.** List the 6–10 business capabilities your area provides (for example customer onboarding, order fulfilment, payments, reporting). For each, note which platforms support it today.
3. **Map the current state honestly.** For each capability, rate it Green, Amber or Red for stability, scalability, cost to change and manual effort. Note duplicates: the same data or logic held in more than one place.
4. **Find the platform opportunities.** Look for capabilities that other areas also need, data that should have one source, and integration that is point-to-point. This is where platform thinking pays off.
5. **Write your principles (week 2).** Three to six, specific to your platform. See the next section.
6. **Describe the target state at 24 months.** For each capability, say what it runs on, who else reuses it, and what has been retired. Write it so a business leader understands it.
7. **Define the three steps to get there.** The order matters more than the dates. Each step should deliver something the business or product will notice, and name what it retires.
8. **Test it (week 3).** Walk your engineering lead and product owner through it. Change what they show is wrong or unrealistic. Note what they disagree with.
9. **Peer review and publish (week 4).** Swap with another architect for review, then bring it to the team review for sign-off by the head of architecture.

## Writing platform principles

A good principle helps someone make a design decision without asking you. If it would not change a decision, cut it.

Platform principles sit under your organization's architecture principles and make them concrete for your platform.

**Each principle has three parts:**

- **Statement:** one sentence, a clear choice.
- **Why:** the business or platform reason.
- **What it means in practice:** what designs must do or must not do, and what it rules out.

**Weak versus strong (illustrative):**

| Weak | Strong |
| --- | --- |
| "Data should be high quality." | "Customer data has one source. Every consumer reads from the customer service; no new copies are created. Why: copies drift apart and cause reconciliation errors. In practice: new consumers subscribe to change events instead of extracting tables." |
| "Use modern technology." | "New products are configured, not coded. Why: launches should not need a release. In practice: product and pricing rules are reference data, changed through a controlled workflow." |
| "Systems must be resilient." | "Peak-period processing finishes with 2 hours to spare at twice today's volume. Why: volume is growing faster than the processing window. In practice: new jobs show capacity test results before go-live." |

**Tests for a good principle:**

- Could someone reasonably disagree with it? If not, it is too obvious to help.
- Does it rule something out?
- Could an engineer apply it without you in the room?
- Does it trace to a business outcome or a platform pain?

## Template

Copy this into a new document for your area. Keep it to two pages; put supporting detail in an appendix.

```markdown
# [Area] Platform Target State
Owner: [architect] · Backup: [architect] · Version / date

## 1. Summary (3 sentences)
Where the platform is today, where it will be in 24 months,
and what that means for the business.

## 2. Business drivers
The 2–3 outcomes the business and product need (from your conversations).
What is coming: products, volume, regulatory change.

## 3. Capabilities today
| Capability | Platforms today | Stability | Scalability | Cost to change | Manual effort |
Note duplicates and point-to-point integrations.

## 4. Platform principles (3–6)
For each: Statement · Why · What it means in practice

## 5. Target state at 24 months
| Capability | Runs on | Reused by | Retired |
One simple picture of the target platform.

## 6. Three steps to get there
For each step: what changes · what the business or product will notice
· what it retires · key dependency

## 7. Decisions and open questions
Decisions needed (and who decides) · disagreements from review

## 8. How we will know it is working
2–3 measures with today's baseline
```

## Quality checklist

Check these before peer review.

- [ ] A business leader could read the summary and target state without a glossary
- [ ] Business drivers come from real conversations with business and product owners, not assumptions
- [ ] Capabilities are described before systems
- [ ] Each principle passes the tests above and rules something out
- [ ] The target state names at least one shared capability other areas could reuse
- [ ] Every step names what it retires, not only what it adds
- [ ] The steps are realistic for engineering capacity, as tested with your engineering lead
- [ ] Disagreements from review are recorded, not hidden
- [ ] Measures have a baseline today

## Timeline and review

Four weeks from kickoff to sign-off, then refreshed quarterly.

| Week | What | Output |
| --- | --- | --- |
| 1 | Business conversations; capabilities and current state | Notes, capability table |
| 2 | Principles and target state draft | Draft sections 1–5 |
| 3 | Steps; walk-through with engineering lead and product owner | Full draft with their feedback |
| 4 | Peer review with a partner architect; team review | Sign-off; published |

**Peer pairs:** pair architects across teams or locations where possible, so each area has a second person who knows its direction.

**After sign-off:**

- Share a short version with the business and product leads for your area.
- Use it in every design review and exception decision: does this move us toward the target state?
- Refresh it each quarter at planning. Note what changed and why.
