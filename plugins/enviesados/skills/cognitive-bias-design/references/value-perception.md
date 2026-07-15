# Value perception — Anchoring, Cashless Effect, Framing

> Chapters 1, 3 and 6 of Rian Dutra, *ENVIESADOS* (2024).
> These three biases govern **what a price feels like** and **what a fact feels like** — not what
> either one is.

---

## 1. Anchoring — "we over-trust the first information we receive"

### Mechanism

We can't produce a correct value on the spot, so the brain grabs **any** information that seems
remotely relevant and builds the answer from it. The anchor bites hardest when the thing has **no
exact or known value** — which is most things a product sells.

The unsettling part: **the anchor works even when it is transparently irrelevant to the decision.**

### Evidence

| Study | Setup | Result |
|---|---|---|
| Tversky & Kahneman — wheel of fortune | Wheel rigged to stop on 10 or 65, then: "what % of UN nations are African?" | Saw 10 → answered **25%**. Saw 65 → answered **45%**. |
| Ariely, Loewenstein & Prelec — SSN auction | Bid on books/wine; last 2 digits of the participant's Social Security Number became the opening price | High-SSN participants paid up to **346% more**. Digits 0–19 → avg bid **US$16.09**; digits 80–99 → **US$55.64** for the same item. |

Ariely's conclusion (*Predictably Irrational*): the SSN was arbitrary, and **any** arbitrary
information would have worked — current temperature, manufacturer's suggested retail price.

### Design moves

| Move | Case in the book |
|---|---|
| Show the original price next to the discounted one | Udemy course pages — original struck through beside the sale price |
| Suggest a donation/bid amount | Change.org asks "R$20 or more?" first, then shows suggested amounts with a free-entry field. The suggestion **lowers cognitive load** and doubles as reference. |
| Highlight a recommended plan | Highlighting a recommended item increased lead conversion |
| Lead with the most expensive plan | MailChimp shows the priciest first — the options beside it, especially the recommended one, look more attractive |
| Open a negotiation first | The author's own Jeep purchase: asking R$22k, he anchored at R$17k, closed at R$18k — below the R$20k he'd have been happy with |

### Failure mode

Anchors that are too aggressive stop being believed. The author on Udemy: the discounts are so
exaggerated that he questions whether the courses are any good. **"Usar esse princípio com
parcimônia, sem exageros — do contrário, o visitante pode não acreditar na oferta."**

Related: a high price is itself read as a signal of quality, reliability and durability. That cuts
both ways — a too-low anchor devalues the product.

---

## 2. Cashless Effect — "we spend and buy more when we don't see the money"

### Mechanism

Paying with physical cash produces a real discomfort — Zellermayer's **"pain of payment"** — and
that discomfort makes us attribute **more value** to the purchase. Card and digital payments feel
less real. The rule the book states plainly:

> **The lower the payment transparency, or the less tangible it is, the more we consume.**

### Evidence

| Study | Result |
|---|---|
| Avni Shah — mug buyback | Bought at US$2. Card payers wanted **US$3.83** back on average; cash payers wanted **US$6.71**. Card/digital "parecem menos reais do que dinheiro". |
| Laundry machines — coins vs prepaid card | Residents spent **less** with coins than with the prepaid card system |
| 250,000 vending machines on a cashless platform | **+37% spend** when consumers paid by card instead of cash |

Consequence outside the product: nearly **8 in 10 Brazilians** carry credit card debt (2018), the
main household debt regardless of income; **47%** of US adults had credit card debt in early 2020.

### Design moves

| Move | Case in the book |
|---|---|
| Remove payment steps entirely | Amazon **1-Click** (1999) — skips the cart, and after the first order it auto-activated with the default payment method and address |
| Abstract money into a token for the whole journey | Disney **MagicBand** — pay for nearly everything from hotel to park with a wristband, minimizing pain of payment across the entire experience |

### The honest reading

The book states both directions of this in the same sentence — it is the cleanest example of its
"alignment" ethic:

> "se você quer tentar economizar dinheiro e se disciplinar financeiramente, tente evitar usar
> dinheiro vivo, mas se quer fazer com que seus usuários gastem mais, tente eliminar (ou ao menos
> reduzir) a dor de pagamento dos clientes"

Removing pain of payment is legitimate when the purchase genuinely serves the user ("não apenas
mais prazerosa (para o usuário), mas também lucrativa (para os negócios)"). It is *not* legitimate
when the friction you removed was the user's only defense against a purchase they'd regret. If your
product's economics depend on users not noticing what they spent, re-read
`ethics-and-dark-patterns.md`.

---

## 3. Framing Effect — "we choose based on how information is presented"

### Mechanism

Attention is directed to the positive side (glass half full) or the negative (half empty) — and the
choice follows the frame, not the fact. Kahneman & Tversky explain it via **Prospect Theory**, tied
directly to Loss Aversion (see `retention-and-inertia.md`):

- A loss is perceived as more significant, and more worth avoiding, than an equivalent gain.
- A **certain gain** is preferred over a probable gain.
- A **probable loss** is preferred over a certain loss.

Net: we avoid uncertain outcomes and seek options framed as certain gain.

The author's definition: **framing is the manipulation of your attention, and the presentation of
selective information, in order to influence your choice.**

### Evidence

| Case | Positive frame | Negative frame (same fact) |
|---|---|---|
| Contrast imaging exam | "99.9% of people have no serious reaction" | "0.1% have serious problems" / "1 in 1,000 can have grave complications" |
| McDonald's, 1991 ad | "91% fat free" | "9% fat" |
| Iowa ground beef study | "75% lean" — rated **higher** | "25% fat" — rated lower (before *and* after tasting) |
| Nescau packaging | "33% menos açúcares" | "10 gramas de açúcar por porção" |
| Sensodyne | "9 out of 10 dentists recommend" | "1 in 10 dentists don't recommend" |

Note the exam example carefully: the doctor **told the pure truth** and framed it positively. Same
data, and a journalist could truthfully report "three people died from injectable contrast in
Campinas". Framing does not require lying — which is exactly why it needs an ethics gate.

### Design moves

- **Discount copy**: "20% off" vs "R$1,600 off" on an R$8k MacBook; "save 50%" vs "half price".
  Absolute numbers frame better at high prices, percentages at low ones — pick deliberately, and
  know you *are* picking.
- **Attribute labels**: state the attribute the target audience is looking for (less sugar, less
  fat), positively framed, from the real data.
- **Sales conversation**: the leather-jacket seller in Gramado detailed exactly the attributes that
  matched him (rock aesthetic, "since 1985"), never showed the plain alternative, and whispered the
  price. Selective attention, no lies told.

### Framing in user research — the failure mode that corrupts data

> "o quanto você **gosta** do aplicativo AiFome?" vs "**como é sua experiência** com o aplicativo
> AiFome?"

The word "gosta" presupposes liking and pulls the respondent to positives first. They may not like
it at all — they may use it because there's no alternative in their city. Result: over- or
under-estimation of the true opinion. **Improper framing produces imprecise and unreliable data**,
which the book calls potentially fatal for research.

Same risk in your assessor showing only an investment's upside, or a realtor detailing the carved
baseboards and the illuminated cove ceiling while never mentioning the flat's total 39 m².

---

## Cross-references

- Framing ↔ Loss Aversion: the Asian-disease experiment lives in `retention-and-inertia.md`; it is
  a framing experiment *and* a loss-aversion experiment.
- Anchoring ↔ Choice Overload: an anchor **reduces cognitive load** (Change.org). Where you can't
  remove options, a good anchor is a partial fix — see `choice-and-emotion.md`.
- Framing ↔ deception: framing is legal, deception is not. The line is in `ethics-and-dark-patterns.md`.
