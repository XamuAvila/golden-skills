---
name: hooked-habit-design
description: Applies Nir Eyal's Hook Model ("Hooked", 2014) to design habit-forming features in any product. Use when planning/reviewing engagement, onboarding, notifications, retention, gamification, feeds, or whenever the question is "how do we get the user to come back on their own?". Skill-router: points to the sub-skill for each phase (Trigger, Action, Variable Reward, Investment) and to Habit Testing. Activates on: habit, retention, engagement, onboarding, notification, churn, "user doesn't come back", loop, hook.
allowed-tools: Read, Grep, Glob
---

# Hooked — Designing habit-forming products (Hook Model)

> Source: Nir Eyal with Ryan Hoover, *Hooked: How to Build Habit-Forming Products* (2014).
> This is the parent skill. Each phase has its own sub-skill — load the phase you are working on.

## What it is

The **Hook Model** is a four-phase cycle that, repeated, shifts the product's trigger from
**external** (notification, e-mail, icon) to **internal** (an emotion of the user's own). Once that
happens, the user returns **without paid prompting** — what Eyal calls "first-to-mind wins".

```
        ┌──────────────────────────────────────────────┐
        │                                              ▼
   [ TRIGGER ] ──▶ [ ACTION ] ──▶ [ VARIABLE REWARD ] ──▶ [ INVESTMENT ]
   external→internal simplest       reward with            "bit of work" that
                     possible act   variability            loads the next trigger
        ▲                                                        │
        └────────────────────────────────────────────────────────┘
                     the investment arms the next trigger
```

> Eyal: "The Hook Model describes an experience designed to connect the user's problem to a
> solution frequently enough to form a habit." (Introduction)

## The 4 phases → sub-skills

| Phase | What it answers | Sub-skill |
|------|-----------------|-----------|
| **Trigger** | What brings the user? Internal (emotion) vs external (Paid/Earned/Relationship/Owned) | `hooked-triggers` |
| **Action** | What is the simplest action? B=MAT (Fogg), motivation × ability | `hooked-action` |
| **Variable Reward** | Does the reward satisfy and leave them wanting more? Tribe / Hunt / Self | `hooked-variable-reward` |
| **Investment** | What "bit of work" does the user invest, and does it load the next trigger? | `hooked-investment` |
| **Validate** | Who is the habitual user? Where is the Habit Path? Is this ethical? | `hooked-habit-testing` |

## Before designing: 3 fitness filters

1. **Habit Zone** (frequency × perceived utility). A behavior only becomes a habit if it lands in
   the zone: **high frequency** or **high perceived utility**. Low on both axes → no habit, don't
   force it. Eyal: a frequent behavior (Google) becomes a habit even at low utility per use; high
   utility but rare (Amazon) becomes a habit via the other axis. (Ch. 1, Figure 1)
2. **Vitamin vs. painkiller → the "itch".** A habit product starts as a "nice-to-have" (vitamin)
   and, once the habit forms, becomes a "must-have" (painkiller), because *not using it now causes a
   slight pain/itch*. Ask: what itch does your product relieve? (Ch. 1)
3. **Not every business needs habit.** A rare/one-off purchase (e.g. insurance) does not require
   habitual use. Separate **daily-use** parts of your product from **occasional-use** ones, and apply
   the Hook Model where business value depends on **recurring, unprompted engagement**.

## The 5 fundamental questions (checklist for any feature)

> Eyal, Ch. 6 — "What Are You Going To Do With This?"

1. **What does the user really want? What pain does the product relieve?** *(Internal Trigger)*
2. **What brings the user to the service?** *(External Trigger)*
3. **What is the simplest action in anticipation of reward, and how do you simplify it?** *(Action)*
4. **Is the user rewarded, yet left wanting more?** *(Variable Reward)*
5. **What "bit of work" does the user invest? Does it load the next trigger and store value?** *(Investment)*

## Ethics gate — required before shipping (Manipulation Matrix)

Every habit product is a form of manipulation. Before building, answer 2 questions (Eyal, Ch. 6):

- **"Would I use this product myself?"**
- **"Does it materially improve the user's life?"**

| | Improves life | Doesn't improve |
|---|---|---|
| **Maker uses it** | **Facilitator** ✅ (highest chance of success) | **Entertainer** (ok, but fades fast) |
| **Maker doesn't use it** | **Peddler** ⚠️ (lacks empathy) | **Dealer** ❌ (exploitation) |

If you find yourself "squirming" to justify the answers, **stop**. Detail in `hooked-habit-testing`.
Never use **dark patterns** (tricking users into inviting/posting) — it burns trust and reverses the habit.

## How to use

1. Identify the phase/feature. Load the matching sub-skill.
2. Run the 5 fundamental questions. Wherever the answer is weak, that's the gap.
3. Pass the ethics gate (Manipulation Matrix).
4. After shipping, run **Habit Testing** (Identify → Codify → Modify) to find the Habit Path.
5. Implementation note: "loading the next trigger" usually becomes an event/job that schedules a
   future notification — model each hook as a use case that emits such an event when the investment
   step completes.

## Anti-patterns (the book is explicit)

- Gamification (points/badges/leaderboards) is **not fairy dust**: it only works if it scratches the
  user's real itch (Mahalo vs Quora case, Ch. 4).
- Boosting **motivation** before **ability**: almost always wrong — start with simplicity
  (Ch. 3, "start with ability").
- **Finite variability**: a reward that becomes predictable with use loses its pull (FarmVille). Aim
  for **infinite variability** (content/people).
- Threatening the user's **autonomy** triggers *reactance* (rebellion) — see `hooked-variable-reward`.
