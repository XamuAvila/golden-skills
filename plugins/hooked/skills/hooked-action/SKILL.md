---
name: hooked-action
description: Phase 2 of the Hook Model (Nir Eyal, "Hooked") — Action. Use when simplifying flows, reducing friction, designing onboarding, forms, buttons, or any action the user must perform. Covers the Fogg Behavior Model (B=MAT), the 3 Core Motivators, the 6 elements of simplicity, "start with ability", and heuristics (scarcity, framing, anchoring, endowed progress). Activates on: action, friction, simplify, usability, onboarding, form, button, clicks, UX, conversion, ability, motivation.
allowed-tools: Read, Grep, Glob
---

# Action — Phase 2 of the Hook Model

> Source: Eyal, *Hooked*, Ch. 3. Sub-skill of `hooked-habit-design`.

Action = "the behavior done in anticipation of a reward". Core rule: **doing must be easier than
thinking**. The more effort (physical or mental), the less likely the action.

## Fogg Behavior Model: B = MAT

> Dr. BJ Fogg (Stanford). A behavior (**B**) occurs when **M**otivation, **A**bility and a
> **T**rigger are present **at the same time and in sufficient degree**. Miss any one → doesn't
> cross the "Action Line" → doesn't happen.

### 3 Core Motivators (motivation)
Every human is driven by one of three axes (two sides each):
1. Seek **pleasure** / avoid **pain**
2. Seek **hope** / avoid **fear**
3. Seek **social acceptance** / avoid **rejection**

### 6 elements of simplicity (ability)
Simplicity = a function of the user's **scarcest resource at that moment**. Identify what's missing:

| Element | Question |
|---------|----------|
| **Time** | How long does the action take? |
| **Money** | What's the financial cost? |
| **Physical Effort** | How much physical labor? |
| **Brain Cycles** | How much mental effort/focus? (is it confusing?) |
| **Social Deviance** | How far from what others accept? |
| **Non-Routine** | How much it departs from an existing routine? |

> Hauptly's rule of thumb: understand why they use it → lay out the steps → **remove steps until the
> simplest possible process**.

## Start with ABILITY, not motivation

> Eyal: "The answer is always to start with ability." (Ch. 3)

Increasing motivation is expensive and slow (users ignore instructional text). Reducing effort is
more effective. **Make it so simple the user already knows how to use it.** (e.g. one-click login,
iPhone camera on the lock screen, Pinterest's infinite scroll.)

## Heuristics (mental shortcuts that raise M or A)

- **Scarcity**: "only 14 left in stock" changes perceived value (the cookie experiment). → In a
  product: "only 2 slots left on this date".
- **Framing**: context shapes perception (Joshua Bell in the subway; expensive wine "tastes better").
- **Anchoring**: users anchor on one piece of info (the "30% off" isn't always cheaper).
- **Endowed Progress**: giving pre-started progress raises motivation (a punch card with 2 of 10
  stamps pre-filled → +82% completion; LinkedIn's "Profile Strength" bar). → In a product: an
  onboarding progress bar that already starts partly filled.

## Application

- **Signup / first-run**: each field removed raises the completion rate. Pre-fill what you can;
  offer social login/magic link instead of a password (reduces *brain cycles*).
- **Core repeated action**: make it 1 tap, not a form (reduces *physical effort* and *time*).
- **Key flow (from trigger to reward)**: cut the step count. Compare to competitors.
- When reviewing UX PRs, count the **steps of the target action** and attack the *scarcest resource*.

## Phase checklist
- [ ] Identified which of the 3 Core Motivators is in play?
- [ ] Identified which of the 6 elements of simplicity is the **scarcest resource** right now?
- [ ] Removed steps down to the simplest possible action (started with ability)?
- [ ] Do the trigger, motivation and ability coexist at the same moment (B=MAT)?
- [ ] Considered at least one heuristic (endowed progress in onboarding, scarcity in availability)?

→ Previous: `hooked-triggers`. Next: `hooked-variable-reward`.
