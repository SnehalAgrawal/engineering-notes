# Engineering execution

## Overview
In lead and staff roles you drive projects, not just tickets. Execution means shipping safely and predictably, writing down decisions, and making choices that affect several teams for years.

## Key concepts
*   **ADR (Architecture Decision Record)**: A short file with one decision: the context, the options, the choice, and its consequences. It answers "why did we do this?" years later.
*   **RFC / design doc**: A proposal written before coding and shared for review, so problems are found on paper.
*   **Tech debt**: Shortcuts that slow future work. Debt can be deliberate ("ship now, fix next quarter") or accidental (we learned a better design later). Agree with Product on a standing share of capacity to pay it down.
*   **Buy vs. build**: Use a vendor product (Auth0, Datadog) or build in-house. Compare total cost, including the people to run it.

## Real-world usage
*   **Database migration**: A design doc for moving from MongoDB to PostgreSQL without downtime: backfill, dual writes, shadow reads that compare results, switch reads, then stop old writes.
*   **Framework choice**: An ADR picking React over Vue for a dashboard: "Vue is simpler, but the whole team already knows React."
*   **Project slicing**: Split a six-month plan into monthly deliveries so the business gets value earlier and feedback comes sooner.

## Tradeoffs
*   **Analysis vs. speed**: Weeks of RFC work for a small feature wastes time. No design at all for a large feature invites failure. Match the effort to the risk.
*   **Build vs. buy**: Building gives control and can lower per-unit cost, but takes engineers away from the core product. Buying costs money and adds vendor dependence.
*   **Perfect vs. good enough**: The right design fits the company's current stage and can change later.

## When not to use
*   **Heavy process at a tiny startup**: Three people searching for product-market fit need short notes, not full ADRs for every choice.
*   **RFC as a mandate**: An RFC asks for comments. Do not use it to push a favorite technology past real objections.

## Interview focus
*   **Decision making**: "Tell me about a technical decision with no obvious right answer."
    *   Describe the options, the criteria, the tradeoffs, and how you recorded the decision (ADR).
*   **Influencing**: "How do you convince leadership to pause features to rewrite billing?"
    *   Tie the debt to business numbers: slower delivery, incidents, churn, or lost deals.
    *   Offer an incremental plan instead of a big-bang rewrite.
*   **Project rescue**: "You join a project three months late. What do you do in week one?"
    *   Find the real scope, the critical path, and the biggest risks.
    *   Cut or defer scope, and reset the plan with stakeholders.

## Common mistakes
*   **ADRs without consequences**: Every decision has a downside. Write it down.
*   **Rewrite for fashion**: Proposing a rewrite because a tool is new, with no business return.
*   **Sunk cost**: Continuing a failing project because of the effort already spent.
*   **Adding people late**: Adding engineers to a late project usually makes it later (Brooks's law, from *The Mythical Man-Month*).
