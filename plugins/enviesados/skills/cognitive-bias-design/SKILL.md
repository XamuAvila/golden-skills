---
name: cognitive-bias-design
description: >
  Use when a product/UX design decision depends on how users actually decide rather than on how
  they "should" decide — pricing and plan pages, discounts, checkout and payment flows, free
  trials and upgrade prompts, cancellation/churn flows, form defaults and pre-selected options,
  microcopy and framing of offers, menus and catalogs with too many options, onboarding, survey
  and research question wording, thumbnails/visuals, and gamification. Also use when reviewing an
  interface for deceptive design (dark patterns) or when asked "why don't users convert/come
  back/finish this flow?". Maps 8 cognitive biases to concrete design moves, each with the study
  or case that backs it. Source: Rian Dutra, "ENVIESADOS" (2024).
allowed-tools: Read, Grep, Glob
---

# ENVIESADOS — Cognitive biases in product & experience design

> Source: Rian Dutra, *ENVIESADOS: Psicologia e Vieses Cognitivos no Design para criar produtos e
> serviços que ajudam usuários a tomarem MELHORES DECISÕES* (2024).
> Companion to `hooked` (habit loops) — this skill is about the **single decision**, not the loop.

## The premise

A **cognitive bias is a systematic and predictable judgment error** that happens while we process
and interpret information. It is not a designer's tool and not a weapon — it is a *consequence* of
the brain trying to cope with roughly **35,000 conscious decisions a day** (Introduction).
Heuristics (mental shortcuts from fast, intuitive thinking — Kahneman's System 1) are usually
useful and often produce adequate answers; sometimes they produce biases instead.

The author is explicit about how to read this:

> "Peço para não interpretar os vieses cognitivos como ferramentas para o processo de design de um
> produto ou serviço. Muito menos, armadilhas para enganar seus clientes. Pois, não são
> ferramentas nem armas." (Introduction)

So the working question is never *"which bias do I exploit here?"*. It is **"how is my design
already biasing this decision, and does that bias point the user somewhere good?"** — because the
design biases the user whether you thought about it or not.

## Decision → bias map (start here)

Find the decision you are actually making. Load the reference in the last column.

| The design decision in front of you | Biases in play | Reference |
|---|---|---|
| Pricing page, plans, discounts, "from R$X" | Anchoring | `references/value-perception.md` |
| Checkout, payment method, 1-click, wallets, subscriptions | Cashless Effect (pain of payment) | `references/value-perception.md` |
| Offer copy, labels, %-vs-absolute, nutrition/risk claims | Framing Effect | `references/value-perception.md` |
| Free trial, upgrade prompt, "you'll lose X", guarantees | Loss Aversion (Prospect Theory), FoMO | `references/retention-and-inertia.md` |
| Cancellation flow, churn saves, progress/streaks, gamification | Sunk Cost, Commitment | `references/retention-and-inertia.md` |
| Form defaults, pre-selected checkboxes, opt-in/opt-out, ordering | Status Quo, position effects | `references/retention-and-inertia.md` |
| Long menus, big catalogs, filters, long forms, ballots | Decision Fatigue, Choice Overload, satisficing | `references/choice-and-emotion.md` |
| Thumbnails, hero images, imagery, brand feel, packaging | Affect Heuristic, proportion dominance | `references/choice-and-emotion.md` |
| Research/survey question wording | Framing (question framing) | `references/value-perception.md` |
| "Is this a dark pattern?", ethics review of a flow | Deceptive design, Manipulation | `references/ethics-and-dark-patterns.md` |

## The 8 biases at a glance

Each is one chapter of the book. Do **not** optimize for naming them — the author is blunt that the
labels don't matter for daily work; the *mechanism* does.

| # | Bias | One-line mechanism | Canonical evidence in the book |
|---|---|---|---|
| 1 | **Anchoring** | The first number seen frames every number after it, even an irrelevant one | Tversky & Kahneman's rigged wheel of fortune: anchor 10 → 25% average answer, anchor 65 → 45% |
| 2 | **Loss Aversion** | Losing hurts ~2× more than the equivalent gain pleases | Kahneman's mug endowment; Asian-disease framing (72% chose A, 78% chose D — same expected value) |
| 3 | **Cashless Effect** | The less tangible the payment, the more we spend | 250k vending machines: **+37% spend** on card vs cash |
| 4 | **Sunk Cost / Commitment** | Time or money already spent makes us stay, against present evidence | Concorde: US$2.8bn spent, then 27 more years of funding a known loser |
| 5 | **Status Quo** | We keep things as they are; change reads as potential loss | Pension auto-enrollment: **+48%** adoption among new hires after 15 months |
| 6 | **Framing** | Same fact, opposite pull, depending on presentation | McDonald's 1991: "91% fat free" instead of "9% fat" |
| 7 | **Decision Fatigue / Overload** | Each decision costs mental effort; drained users default, abstain, or leave | Parole hearings: **70%** granted early in the day vs **10%** late |
| 8 | **Affect Heuristic** | Feeling substitutes for analysis, especially under time pressure | Netflix: thumbnails are **82%** of user focus, ~**1.8s** per title |

## How to use

1. **Name the decision, not the bias.** "User is choosing a plan" / "user is deciding whether to
   cancel" / "user is filling a signup form". Look it up in the decision → bias map.
2. **Load only the matching reference.** Each one carries the mechanism, the evidence, the design
   moves, and the failure modes.
3. **Find the bias that is already there.** Every default, every ordering, every first number is
   already an anchor/default/frame. There is no neutral design. Ask what the current design biases
   *toward*, and whether that is where the user should go.
4. **Run the ethics gate** (below) before shipping anything that moves a metric via a bias.
5. **Design for the whole context, not the screen.** The author's central claim (Epilogue) is that
   the interface is one touchpoint of a human life — decisions are shaped by the environment around
   the screen (hunger, queue length, time of day, mood). See `references/ethics-and-dark-patterns.md`.

## Ethics gate — required, not optional

The book's dedicated chapter ("PADRÕES OBSCUROS EXISTEM?") lands on a harder position than the
industry consensus. Nielsen Norman Group separates "dark patterns" from "persuasive patterns" by
**intent**; the author **explicitly disagrees** — both influence the user toward the business's
benefit, and intent is not observable from the outside.

His resolution: the real, nameable sin is not *influence* (which is unavoidable) but **deception** —
false information and forged options. He calls those **Designs Impostores**; Harry Brignull, who
coined "dark patterns" in 2010, now calls them **Deceptive Design**.

Before shipping, answer:

- **Am I influencing, or am I lying?** Forged scarcity, hidden costs, and pre-checked consent the
  user never sees are lies. Real scarcity, an honest anchor, and a visible default are not.
- **Would the user still choose this if they saw the mechanism?** If seeing it makes them feel
  cheated, you built a Designs Impostor.
- **Does the business win only because the user loses?** The book's whole framing is *alignment* —
  design that helps users decide better, which happens to be what keeps them.

> "se sua intenção é trapacear seu cliente, uma vez ciente que foi enganado, não irá querer tornar
> a vê-lo." (Padrões Obscuros Existem?)

Full taxonomy of the 12 deceptive patterns → `references/ethics-and-dark-patterns.md`.

## Anti-patterns the book calls out by name

- **Overusing anchors until they stop being believed.** The author names Udemy's permanent
  strikethrough discounts: the exaggeration made *him* doubt the courses' quality. Anchor "com
  parcimônia, sem exageros — do contrário, o visitante pode não acreditar na oferta."
- **Adding options because "more choice serves the user".** Barry Schwartz: with too many options
  consumers are *less* likely to buy anything, and less satisfied when they do.
- **Aggressive loss-framing in cancellation.** Adobe's flow works, but the author flags it
  generates "estresse e ansiedade" — it buys churn reduction with goodwill.
- **Confirmshaming as a decline button.** XP's "Quero perder essa oportunidade" is listed as a case
  *and* appears in the deceptive-pattern taxonomy. Know which side you're on.
- **Gamification as fairy dust.** Streaks work at Duolingo because the streak *is* the progress. See
  also `hooked:hooked-variable-reward`, which makes the same point from the habit-loop side.
- **Framing your research questions.** "O quanto você gosta do app X?" presupposes liking and
  contaminates the data. Ask "como é sua experiência com X?".

## Relationship to `hooked`

`hooked` models the recurring **loop** (Trigger → Action → Variable Reward → Investment). This skill
models the **individual decision** inside it. They overlap at exactly two points, and the books
agree:

| Overlap | `hooked` frame | `enviesados` frame |
|---|---|---|
| Stored progress makes users stay | Investment / IKEA effect | Sunk Cost + Loss Aversion |
| Simplify the action | Ability first (B=MAT) | Decision Fatigue / Choice Overload |

Both are explicit that dark patterns burn the very trust the mechanism depends on.
