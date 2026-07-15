# Retention & inertia — Loss Aversion, Sunk Cost, Status Quo

> Chapters 2, 4 and 5 of Rian Dutra, *ENVIESADOS* (2024).
> These three biases explain why users **stay** — and they are the ones most easily turned into
> deceptive design. Read `ethics-and-dark-patterns.md` alongside this file, not after it.

---

## 1. Loss Aversion — "the pain of losing is twice the pleasure of gaining"

### Mechanism

Humans decide primarily on **losses rather than gains** (Prospect Theory, Kahneman & Tversky). The
aversive response reflects the critical role of **negative emotions — anxiety and fear**. The book
is direct: **Loss Aversion is an expression of fear.**

Two consequences that matter for product work:

1. **The magnitude barely matters; the loss itself weighs double.** The author's broken R$10 wine
   bottle in Gramado — his wife would not have been as *happy* receiving a R$10 bottle as she was
   *angry* losing one. Same with his R$250 trade that ended at +R$50: still up money, still
   devastated.
2. **Loss Aversion only fires when people believe there is something to lose.** No stake, no fear.

Side effect worth knowing: you are more likely to hear bad things about your product than good
ones, simply because people are more affected by negative emotions. Silence may mean they liked it.

### Evidence

| Study | Result |
|---|---|
| Kahneman — mug endowment | Gave a mug to one group, nothing to the other; asked both to price it. Owners priced it **far higher**. |
| Kahneman & Tversky — Asian disease problem | Frame A ("200 will be saved") → **72%** chose it. Frame D ("1/3 chance nobody dies") → **78%** chose it. Identical expected value; opposite risk appetite. |
| Anti-smoking framing | "You will **lose** 5 years of life if you keep smoking" is far more convincing than gaining 5 years — inversely proportional message, much stronger pull. |

### Design moves

| Move | Case in the book |
|---|---|
| Remove the *perceived* loss to unblock a purchase | Mercado Livre **"Compra Garantida"** — guarantees delivery or refund, shown right under the buy button. Addresses the top e-commerce fear. |
| Give it, then let them feel the absence | **LinkedIn Premium** free month. "conquiste, depois tire, e eles pagarão para ter." |
| Show the opportunity that's slipping | **XP Investimentos** startup popup: "Simular agora!" / "Quero perder essa oportunidade" — related to **FoMO** (Fear of Missing Out). |
| Enumerate what's lost at cancellation | **Adobe**: "Você não mais terá acesso à maioria dos seus aplicativos favoritos". The author assumes it works, since they keep the flow. |
| Enumerate what's lost at trial expiry | The author's own NCLEX nursing-prep product: told users they'd lose quiz scores, mock-exam results, and all consumed material unless they upgraded — and that this would hurt their exam prep. |

### Failure modes

- **The XP button is confirmshaming.** It appears in this chapter as a case *and* in Brignull's
  deceptive-pattern taxonomy. The book presents it without endorsing it — treat the overlap as the
  warning it is.
- **Adobe's flow "pode gerar estresse e ansiedade durante o cancelamento".** It reduces churn by
  spending goodwill. Know that's the trade you're making.
- **The credit-card retention call** the author describes (PERDERIA limit, PERDERIA benefits,
  PERDERIA points) worked for exactly **three months** — then he cancelled anyway, after "analisando
  friamente". Loss-framed saves buy time, not loyalty.
- Loss Aversion also harms *your* users: fear of loss stops people from taking well-calculated
  risks that would have benefited them.

---

## 2. Sunk Cost & Commitment — "we won't quit what we've invested time or money in"

### Mechanism

Having invested money, time or effort, we resist quitting **regardless of whether current costs
exceed benefits**. We decide on *past* costs instead of present and future costs and benefits —
the only ones that should rationally matter.

Two named siblings:
- **Sunk Cost Fallacy** (Falácia dos Custos Irrecuperáveis)
- **Commitment Bias / Irrational Escalation of Commitment** — continuing to back prior decisions
  even against evidence

Direct relative of **Loss Aversion** (previous section). And: people **value things more when they
invest their own effort and resources in them**.

### Evidence

| Study | Result |
|---|---|
| Concorde fallacy | Launched 1976 after **US$2.8bn** from the British and French governments. Once it was clear it wasn't profitable, investors kept funding it for **27 more years**. |
| Age study (US$10.95 video) | Groups aged 18–27 and 58–91 paid to watch, got bored. Given the choice to stop or continue: the **58–91 group was less prone** to sunk cost. Younger people are more influenced by it. |
| Duolingo 7-day challenge | Hypothesis: a continuous-study challenge makes users more engaged and more likely to finish a long task. A/B tested → **+14% retention**. |
| Retention research | **Frequency is the biggest predictor of retention.** For a news portal, bringing users back for <5 min daily beats one rare multi-hour session. |
| The Times / The Sunday Times — "James, Your Digital Butler" | AI-personalized news delivery (time, format, frequency, content) → **−49% churn**. |

### The author's own case — the sharpest one in the book

An education SaaS (monthly subscription) needed to convert site visitors into paying members.

1. Switched all CTAs from the pricing page to **"crie sua conta gratuitamente"** (Reciprocity bias
   — like the winery owner in Bento Gonçalves offering a sip, which made him buy a bottle).
2. Chose the free tier's contents **strategically, so users could perceive they were building
   something in there** — watch lessons, take tests and *keep performance results*, discover which
   content would improve their skills (that content being subscriber-only).
3. The user invested no money — they invested **time**.

Result: **30% of free-account users became subscribers, generating US$100k in a single month.**

The author names two reasons: (a) the product was genuinely good and they could try it — also
citing Loss Aversion, Zero Risk Bias, Zero Price Effect; (b) they were too committed to abandon
what they'd built or renounce the hours invested, for fear of losing their progress.

**Note the ordering.** Reason (a) is "the product was truly good". The mechanism amplified a real
value; it did not substitute for one.

### Design moves

- **Explicit commitment**: challenge the user to a defined streak (Duolingo's 7 days).
- **Visible progress the user built**: scores, history, results, rankings, badges. "Perder uma
  posição no ranking, por exemplo, parece um desperdício, então as pessoas continuam voltando."
- **Small wins along the way** make the interaction memorable and build a positive association
  between using the product and feeling good.
- **Personalization to build frequency**: newsletters work as habit-builders specifically when the
  content is genuinely personalized per consumer.

### Failure modes

- **Gamification only works if it's real progress.** Streaks work at Duolingo because the streak
  *is* the learning. See `hooked:hooked-variable-reward` — points/badges/leaderboards are "not
  fairy dust".
- **Sunk cost hits the team, not just the user.** "Quando imerso em projetos, torna-se difícil
  separar-se o bastante para se dar conta que investiu tempo e dinheiro demais numa solução errada,
  ou que está tentando resolver um problema que, na verdade, nunca existiu." Ask this in your own
  roadmap reviews.
- The pull is strongest **when other people are involved**, regardless of the result.

---

## 3. Status Quo — "we leave things as they are, even when better alternatives exist"

### Mechanism

A natural preference for how things currently are. We treat our **current state as a reference
point**, and any change from it is perceived as a **potential loss**. Comparing alternatives, we
perceive greater disadvantages in leaving the current position than in staying — *even when that
doesn't reflect reality* (Kahneman et al.: risk aversion and change aversion).

Relevant to **brand loyalty** and **acceptance of product innovations**.

**Distinction the book insists on:** Status Quo Bias ≠ **Psychological Inertia**. Inertia is
unwillingness to change — it inhibits *any* action. Status Quo Bias is specifically about avoiding
change that would be perceived as a loss or as risky.

Also related: the **mere exposure effect** — we develop preference for things simply because we're
familiar with them.

### Evidence

| Study | Result |
|---|---|
| Samuelson & Zeckhauser (1988) | Harvard rolled out a new faculty health plan. **New** faculty mostly chose the new plan; **existing** faculty hardly switched. People decide differently when a status-quo alternative exists at all — even a hypothetical one. |
| Pension auto-enrollment (US company) | Default "don't contribute" → many stayed out, **even knowing the benefits**. Flipped default to "contribute" (with free opt-out): after 15 months, **+48%** adoption among new hires, **+11%** overall participation. |
| Search results click study | **42%** clicked the first link, **8%** the second. Researchers then silently swapped positions 1 and 2 → **34%** clicked the (new) first, **12%** the second. Most stayed with position, not quality. |
| Google Manhattan cafeteria | Salads at the entrance, M&M's moved far away. After 7 weeks: **3.1 million fewer calories** consumed. |
| Healthy-menu study | Menu 1 showed clear calorie info → little effect on choice. Menu 2 made healthier sandwiches slightly **more convenient to order** → positive effect, enough to lower total meal calories. |
| De Brigard — experience machine | Told your whole life was a simulated experience by mistake — stay or leave? **59% chose to stay connected**, 41% to leave. Familiar-but-simulated beat unknown-but-real. |

Note the pattern across the last three: **information changed little; convenience and position
changed a lot.**

### Design moves

| Move | Case in the book |
|---|---|
| Set the default to the outcome that serves the user | Pension auto-enrollment (+48%), with freedom to leave the program whenever they wish |
| Put the good option first / make it convenient | Google cafeteria; search position effect; menu study |
| Auto-renewing subscriptions | Adobe — renews yearly without prior notice until cancelled; cancelling requires visiting a page or contacting support |
| Pre-select add-ons in the cart | GoDaddy pre-selects "Proteção de Domínio completa" and "Comece seu site GRÁTIS", labelled "Recommended" |
| Offer prepaid renewal at checkout | GoDaddy's 2nd-year prepay — **~3× the average ticket** |

### Failure modes — this is the bias most often weaponized

The book flags these as real cases of harm, not techniques:

- Newsletter signup you only notice when your inbox fills up.
- Software installers and signup forms with **pre-selected default options** the user just accepts —
  the book shows a real large e-commerce signup page with a pre-checked marketing consent field.
- The chapter ends by naming the tension itself: *"Há uma questão sensível que surge no limiar entre
  utilizar o conhecimento da psicologia para proporcionar facilidades às pessoas, e praticar designs
  com 'dark patterns' (...) como poderíamos ser capazes de saber qual é a melhor escolha para o
  usuário?"*

**The test that separates the pension case from the GoDaddy case:** both use a default. The pension
default routes the user to the outcome *they already agreed was good* and leaves the exit free. The
pre-checked marketing consent routes the user somewhere they never considered, and hides the exit.
Same mechanism, opposite ethics. Defaults are not neutral — pick the one you could defend out loud.

### Why defaults win at all

People are **too busy (or too lazy)** to think about details that impact their lives. They avoid
cognitive effort, especially in routine tasks, so they look for mental shortcuts. The more complex
the decision, the harder it is to approximate rational thinking — so we accept the default: the
first one we see, or the one chosen for us. This is the direct bridge to Decision Fatigue →
`choice-and-emotion.md`.

---

## Cross-references

- All three biases compound in a **cancellation flow**: Status Quo (don't act), Loss Aversion
  (you'll lose X), Sunk Cost (you built all this). Compounding them is exactly how a save flow turns
  into a **roach motel** — `ethics-and-dark-patterns.md`.
- Sunk Cost ↔ `hooked:hooked-investment` (IKEA effect, stored value) — same phenomenon, loop view.
- Status Quo ↔ Decision Fatigue: the fatigued user takes the default. Load `choice-and-emotion.md`.
