# Planning & Estimation — Complete Guide

Based on Mike Cohn's *User Stories Applied*, Chapters 8-11.

## Story Points

Story points are RELATIVE estimates of size/complexity/effort. Each team defines them as they see fit:
- Some teams use ideal days (a day without interruptions)
- Some teams use pure complexity units
- The specific unit matters less than consistency within the team

**Key facts:**
- Team A's story points ≠ Team B's story points
- When an epic is split, constituent stories don't need to sum to the epic's original estimate
- Similarly, a story's tasks don't need to sum to the story's estimate

## Planning Poker (Wideband Delphi)

1. Customer reads a story to the developers
2. Developers ask clarifying questions
3. Each developer secretly writes an estimate on a card
4. All cards revealed simultaneously
5. High and low estimators explain their reasoning
6. Repeat until convergence (usually 2-3 rounds)

**Constrained scale:** ½, 1, 2, 3, 5, 8, 13, 20, 40, 80

This scale reflects reality: as estimates grow, precision decreases. Don't argue whether something is 7 or 8 — at that scale, choose 8 or 5.

**Triangulation:** After the first few estimates, compare: "We said Story X is 4 points and Story Y is 2 points. Is X really twice as big as Y?" Pin cards to a wall organized by point value to facilitate quick comparison.

## Velocity

**Velocity** = story points completed per iteration.

- First iteration: guess. You'll be wrong. That's OK.
- After iteration 1: use measured velocity for future planning.
- Velocity self-corrects estimation bias: if the team consistently over-estimates, velocity will be proportionally higher, and the same number of stories will get planned.

**Conditions for stable velocity:**
1. No unusual factors (overtime, extra people, holidays)
2. Estimates were generated consistently (team estimation, not one person)
3. Stories are independent (not all UI work or all DB work in one iteration)

## Release Planning

A release = one or more iterations. Planning a release answers: "What can we deliver by when?"

### Step 1: Determine the Date (or Date Range)
- Fixed date (trade show, contract deadline) → easier to plan, harder to decide scope
- Date range ("May to July") → more flexibility

### Step 2: Prioritize Stories with MoSCoW

| Priority | Meaning |
|---|---|
| **Must have** | Fundamental. Release fails without these. |
| **Should have** | Important, but short-term workaround exists. |
| **Could have** | Desirable. Can drop if time runs short. |
| **Won't have (this time)** | Acknowledged desire, deferred to a future release. |

### Step 3: Allocate Stories to Iterations

Given velocity V and stories sorted by priority:
1. Fill Iteration 1 with the highest-priority stories (sum ≤ V)
2. Fill Iteration 2 with the next set (sum ≤ V)
3. Continue until all stories are allocated or you run out of iterations

If a story is too large to fit → split it. Move the must-have portion into the current iteration and the rest into a later one.

### Step 4: Adjust Every Iteration

Before each new iteration:
- Re-assess priority (business needs change)
- Use MEASURED velocity instead of estimated
- Add/remove stories from future iterations as needed

## Iteration (Sprint) Planning

More detailed than release planning. At the start of each iteration:

1. **Select stories** that fit within measured velocity
2. **Break stories into tasks** (developer-level work items, typically 4-16 hours each)
3. **Developers volunteer for tasks** — tasks are owned by individuals, unlike stories which are owned by the team
4. **Write/verify acceptance tests** — customer team specifies, ideally before coding starts

### Task Estimation

Tasks are estimated in IDEAL HOURS (not story points):
- Include everything: coding, testing, talking to customer, helping with acceptance tests
- Remember: "Everything in the world takes four hours minimum" — factor in the overhead

## Monitoring Progress

### Burndown Charts
Track remaining story points over time. The slope tells you if you're on track.

### Velocity Tracking
Track velocity per iteration. Look for trends:
- Stable velocity → planning is reliable
- Declining velocity → investigate (technical debt? morale? changing team?)
- Increasing velocity → team is ramping up (common in early iterations)

### Adjusting the Plan
At the end of each iteration:
1. Count completed story points → new velocity measurement
2. Recalculate how many iterations remain
3. If behind: negotiate scope (remove Could-have stories), or extend timeline
4. If ahead: pull in additional stories from the backlog

## Common Pitfalls

- **Confusing story points with hours**: They're relative measures, not time units
- **Comparing velocity across teams**: Meaningless. Each team's points are their own currency.
- **Adding padding**: Don't inflate estimates. Let velocity naturally account for "real world" overhead.
- **Ignoring velocity data**: Using wished-for velocity instead of measured velocity leads to chronic over-commitment.
- **Estimating alone**: Story estimates are team estimates. One person's estimate lacks the triangulation benefit.
