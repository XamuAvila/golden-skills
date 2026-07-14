---
name: hooked-investment
description: Phase 4 of the Hook Model (Nir Eyal, "Hooked") — Investment. Use when designing features that make the user invest time/data/effort to improve the product and come back later. Covers the IKEA effect (we value our own effort), stored value (content, data, followers, reputation, skill), "loading the next trigger", progressive investments, and why switching is costly. Activates on: investment, retention, user data, personalization, lock-in, switching cost, reputation, "load the next trigger", loop, come back later.
allowed-tools: Read, Grep, Glob
---

# Investment — Phase 4 of the Hook Model

> Source: Eyal, *Hooked*, Ch. 5. Sub-skill of `hooked-habit-design`.

The investment phase is where the user does a **bit of work** that raises the odds of going through
the cycle again. **Unlike Action, it delivers no immediate gratification — it's about future
reward.** It's what closes the loop.

## Why investing creates preference (psychology)

1. **IKEA effect / "labor leads to love"**: we irrationally value what we helped create (the origami
   experiment — Dan Ariely: 5× more value on one's own origami).
2. **Consistency with the past**: small commitments lead to big ones ("Drive Carefully" sign: 17% →
   76% after a small prior request).
3. **Avoiding cognitive dissonance**: we change perception to justify effort already spent (Mafia
   Wars — "I spent time on it, so it must be worth spending $20").

> Golden rule: **ask for the investment ONLY after the variable reward**, when the user is "primed
> to reciprocate". And **stage the investment in small progressive steps** — start easy and increase
> each cycle (don't ask a heavy task upfront; if you do, you're asking too much → back to ability,
> B=MAT).

## Stored value — 5 forms (the product improves with use)

Software, unlike a physical good, **gets better as the user invests**. Each form raises the
switching cost without locking by force:

| Form | What it is | Product example |
|------|-----------|-----------------|
| **Content** | Content the user aggregates (playlists, timeline) | History, notes, saved templates |
| **Data** | Data/preferences added actively or passively (LinkedIn, Mint) | Profiles, configuration, budgets, categorization |
| **Followers** | A network that builds up (following/followers on Twitter) | Connected teammates, collaborators, contacts |
| **Reputation** | A score you "take to the bank" (eBay, Airbnb, Yelp) | Reviews, ratings, internal ranking |
| **Skill** | Learned effort that doesn't transfer (Photoshop) | Mastery of the product's flows — familiarity = less *non-routine* |

## Loading the next trigger — the heart of the phase

The investment should **arm the next external (Owned) trigger**. That's what pulls the user back and
closes the cycle:

- **Any.do**: connects to the calendar → app sends a notification after the meeting ("log the
  follow-up").
- **Snapchat/Tinder**: each message/swipe implicitly loads the other side's response.
- **Pinterest**: each pin/like grants tacit permission to notify when someone interacts.

→ **Implementation pattern:** the "bit of work" usually becomes an event/job that schedules a future
notification. Examples:
- A user assigns a task → event schedules a push to the assignee when it becomes actionable.
- A user sets a goal → event fires a digest when the goal is hit/missed.
- A user leaves a preference → loads the trigger for the next relevant moment/upsell.

Measure and **reduce the delay** between the "loaded trigger" and re-engagement — it shortens the
Hook's cycle-time.

## Phase checklist
- [ ] Does the investment come **after** the variable reward?
- [ ] Is it a **small step**, scaling up in future cycles (not asking too much)?
- [ ] Does it accrue **stored value** (content / data / followers / reputation / skill)?
- [ ] Does it **load the next trigger** (event → Owned notification)?
- [ ] Is the delay to re-engagement as small as possible?

→ Previous: `hooked-variable-reward`. Cycle closed → validate with `hooked-habit-testing`.
