---
name: hooked-triggers
description: Phase 1 of the Hook Model (Nir Eyal, "Hooked") — Triggers. Use when designing notifications, e-mails, push, onboarding, invites, or when defining WHEN/WHY the user comes back. Covers external triggers (Paid, Earned, Relationship, Owned), internal triggers (emotions), the 5 Whys method, and the external→internal transition. Activates on: notification, push, e-mail, trigger, onboarding, invite, "bring the user back", CTA, call-to-action.
allowed-tools: Read, Grep, Glob
---

# Trigger — Phase 1 of the Hook Model

> Source: Eyal, *Hooked*, Ch. 2. Sub-skill of `hooked-habit-design`.

The trigger is "the actuator of behavior — the spark plug in the engine". It comes in **two types**:
**external** (in the environment) and **internal** (in the user's memory/emotion). End goal: shift
from external to internal.

## External triggers — 4 types

They carry the information of what to do next *embedded* in the trigger itself (a link means click,
an icon means tap). Golden rule: **reduce the options**. Too many choices → hesitation/abandonment
(the Mint e-mail has a single "Log in to Mint" button).

| Type | What it is | Good for | Product example |
|------|-----------|----------|-----------------|
| **Paid** | Ads, SEM, paid media | Acquiring new users. Unsustainable for return. | Acquiring new accounts — never for daily retention |
| **Earned** | Press, viral, store feature | Attention, but short-lived | PR, referrals — don't rely on it for ongoing engagement |
| **Relationship** | One person tells another (invite, "Like", word of mouth) | Viral growth | A user invites a teammate; a customer shares a link |
| **Owned** | Occupies space in the user's environment (icon, push, newsletter) — **only after opt-in** | **Repetition until a habit forms** | App push, task badge, digest e-mail |

> **Owned triggers are the engine of repetition.** Without opt-in and a share of the user's
> attention, it is hard to cue frequently enough to change behavior. Paid/Earned/Relationship
> acquire; **Owned** brings back.

⚠️ **Dark patterns forbidden**: tricking the user into firing mass invites/messages yields short
growth and burns *social currency* — once discovered, they stop using the product.

## Internal triggers

When the product couples to **a thought, an emotion, or a pre-existing routine**. You can't see or
touch it — it fires on its own in the mind. **Negative emotions are powerful internal triggers**:
boredom, loneliness, frustration, confusion, indecision, fear of missing out (FOMO).

> "In the case of internal triggers, the information about what to do next is encoded as a learned
> association in the user's memory." (Ch. 2)

The association takes **weeks/months of frequent use** to form. An external trigger *starts* the
habit; the association with the internal trigger is what *keeps* the user.

### Map the emotion, not the feature
Trace each core action back to the emotion that fires it, e.g.:
- A user feels **anxiety** about missing something time-sensitive → opens the relevant panel.
- A user feels **uncertainty** about what to do next → opens the task/work queue.
- A user feels **fear of being out of the loop** on performance → opens the dashboard.

## The 5 Whys method (find the real internal trigger)

Adapted from the Toyota Production System (Taiichi Ohno). Ask "why?" until you reach an **emotion**
(usually by the 5th). Book example (Julie/e-mail):

```
Why use it? → to send/receive messages
Why? → to share/receive information quickly
Why? → to know what's going on with coworkers/friends/family
Why? → to know if someone needs her
Why? → she fears being out of the loop   ← EMOTION = fear
```

Careful: **"declared preferences" ≠ "revealed preferences"**. The user doesn't know which emotion
drives them — don't rely on surveys alone; observe what they *do* (user narratives, Jack Dorsey).

## Phase checklist
- [ ] Ran the 5 Whys and reached an **emotion** (internal trigger)?
- [ ] Wrote the narrative: "Every time the user *(internal trigger)*, they *(first action of the
      habit)*"?
- [ ] Have an **Owned trigger** with opt-in to bring them back (push/badge/e-mail)?
- [ ] Is the external trigger **coupled as closely as possible** to when the internal trigger fires?
- [ ] Does the CTA have **minimal options** (ideally one obvious action)?
- [ ] Zero dark patterns?

→ Next phase: `hooked-action` (turn the trigger into a real action with B=MAT).
