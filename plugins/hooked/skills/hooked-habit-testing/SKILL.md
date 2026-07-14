---
name: hooked-habit-testing
description: Validation and ethics of the Hook Model (Nir Eyal, "Hooked", Ch. 6-8). Use AFTER shipping a habit feature to measure whether it really forms a habit, find the Habit Path, and decide if it's ethical. Covers Habit Testing (Identify → Codify → Modify), the 5% rule, cohort analysis, the Manipulation Matrix (Facilitator/Peddler/Entertainer/Dealer), and where to look for opportunities (introspection, nascent behaviors, enabling tech, interface change). Activates on: habit testing, retention, cohort, engagement metrics, ethics, manipulation, dark pattern, "worth building", habit path, habitual user.
allowed-tools: Read, Grep, Glob, Bash
---

# Habit Testing & Ethics — validating the Hook Model

> Source: Eyal, *Hooked*, Ch. 6 (Manipulation Matrix) and Ch. 8 (Habit Testing). Sub-skill of
> `hooked-habit-design`. Inspired by build-measure-learn (lean startup).

Running the idea through the 4 phases finds *design weaknesses*. But **only testing with real
users** tells you what works. Habit Testing answers: **who** your devotees are, **which parts** form
a habit, and **why**.

## Habit Testing — 3 steps

### 1. Identify — who are the habitual users?
- First define **how often the user "should" use** the product. This changes everything. A social
  network → several times/day; a movie site → 1-2×/week. Don't make an aggressive prediction that
  only counts the *uber-users*.
- Calibrate the target frequency **per user segment/role** (some use it several times a day, others
  weekly or only during a specific window).
- Dig into the numbers and see **how many and which types** hit the threshold. Use **cohort
  analysis** to measure change across iterations.

### 2. Codify — find the Habit Path
- **The 5% rule**: if at least ~5% of users use it as you predicted, there's signal. Below that →
  wrong users or product back to the drawing board.
- Analyze the steps your devotees took and look for the **Habit Path** — the series of actions
  common to your most loyal users. (Twitter: following **30** people = retention tipping point.)
- Find out **which step is critical** to creating devotees.

### 3. Modify — push new users down the same path
- Adjust onboarding/funnel/features to lead new users along the devotees' Habit Path (Twitter changed
  onboarding to encourage following people early).
- It's **continuous**: run it on every feature/iteration, comparing new cohorts to the habitual ones.

## Where to find habit opportunities (Ch. 8)
- **In the mirror / introspection**: build for your own itch (Buffer was born from the founder
  scheduling tweets by hand). Ask "what problem do I wish someone else would solve for me?".
- **Nascent behaviors**: niche behaviors that serve a mass need (Facebook started at Harvard). Much
  innovation was dismissed as a "toy".
- **Enabling technologies**: where new tech makes the Hook cycle faster/more frequent/more rewarding.
- **Interface change**: a change in how people interact reveals new behaviors (Instagram = camera in
  the smartphone; Pinterest = image canvas). → New surfaces (mobile, voice, an AI copilot) can reveal
  new habitual behaviors.

## ETHICS GATE — Manipulation Matrix (required)

Every habit product manipulates. Before building, the maker answers **2 questions**:
**"Would I use it?"** and **"Does it materially improve the user's life?"**

| | Improves life | Doesn't improve |
|---|---|---|
| **Maker uses it** | **Facilitator** ✅ highest chance of success; understands the user | **Entertainer** — can work, but no staying power |
| **Maker doesn't use it** | **Peddler** ⚠️ hubris/inauthenticity; lacks empathy | **Dealer** ❌ exploitation; moral risk and lower success |

- If you **squirm** to justify the answers → **stop, you failed**. Be a Facilitator.
- To build for a user who isn't you, you **must have experienced the pain first-hand** — otherwise,
  do real research (not declared-preference surveys).
- ~1% of users form a pathological dependency: there is a **moral obligation** to identify and
  protect those forming an unhealthy attachment. Offer controls/limits.

### Forbidden
- **Dark patterns**: tricking users into inviting/posting/paying. Short growth, destroyed trust.
- Threatening **autonomy** (forced opt-in, obligation) → reactance → churn.

## Validation checklist (per habit feature)
- [ ] Defined the target frequency per user segment/role?
- [ ] Measured via cohort how many hit the threshold? ≥ ~5%?
- [ ] Identified the **Habit Path** (devotees' critical step)?
- [ ] Modified onboarding to lead new users down that path?
- [ ] Passed the **Manipulation Matrix** gate as a **Facilitator** (without squirming)?
- [ ] Zero dark patterns; autonomy preserved; protection for the ~1% at risk?

→ Back to `hooked-habit-design` to iterate the full cycle.
