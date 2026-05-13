# ADR 0001 — Buyer is the engineering manager; wedge is AI-spend ROI

Date: 2026-05-12
Status: Accepted

## Context

The product could be sold to either individual contributors (personal coaching SaaS) or engineering managers / VPs Eng (team-level ROI and adoption analytics). These are different products with different sales motions, pricing models, and metrics. Trying to be both at once is the #1 failure mode for idea-stage developer tools.

Concurrently, "ROI" can mean different things on the dashboard:

- **A. Activity & utilization** — sessions, prompts, acceptance rates, dormant seats. Easy to collect, already shown in-tool. Tells you attendance, not value.
- **B. Self-reported value** — in-tool ratings, manager surveys. Cheap but biased.
- **C. Outcome correlation** — cycle time, defect rate, throughput pre/post AI adoption. CFO-credible but slow to demonstrate and hard to attribute.
- **D. Code-level attribution** — per-commit / per-PR % AI-generated, mapped to downstream signals (bugs, reverts, review iterations, churn). Technically hard, very defensible.

## Decision

**Buyer: engineering manager / VP Eng.** The painful, budget-attached problem in 2026 is "I'm spending $40k/month on AI coding seats and I can't justify it to my CFO." That is the urgent willingness-to-pay. IC-curiosity products are nice-to-have and brutal to monetize.

**Layered product roadmap:**

- **Wedge (ship first): A — utilization + spend rationalization.** Day-1 demo answers "you're paying for 200 seats, 80 are dormant, here's $X you can reclaim." Honest, easy to collect, fast to demo, low-friction to procure.
- **Promise (sell on the slide): C — outcome correlation.** What carries the contract.
- **Moat (long-term defensibility): D — per-commit attribution.** Makes us uncopyable and justifies (or kills) the AI budget for the customer.

IC product is **phase 2** — emerges naturally from the OSS distribution layer (engineers self-hosting Revelio to coach themselves).

## Consequences

- All early decisions optimize for "eng-manager-credible," not "engineer-self-improvement-fun."
- Marketing copy and sales narrative orient around CFO-defensible numbers.
- The roadmap explicitly sequences A → C → D — do not let customers, sales, or roadmap pressure flip the order.
- Honest-numbers stance is mandatory (see [parking-lot.md §3](../parking-lot.md) — the productivity paradox question).
