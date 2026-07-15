# Ethics, deceptive design, and Human Experience Design

> "PADRÕES OBSCUROS EXISTEM?" and the Epilogue of Rian Dutra, *ENVIESADOS* (2024).
> This is not an appendix. The book's ethics chapter argues against the industry's standard framing,
> and the argument changes how you're supposed to use every other reference in this skill.

---

## The problem the chapter opens with

Once you know how unconscious biases are produced — **manipulating information, arranging visual
elements, and preparing a controlled environment** — you could, in theory, decide *for* users
without them knowing.

> "Nossos designs passariam a ser o titereiro, e os usuários, as marionetes."

So the question: if influencing someone's opinion and decision is manipulation regardless of intent
— is there any difference at all?

---

## The standard definition, and its gaps

**Harry Brignull coined "dark patterns" in 2010**, defining them as:

> "uma interface de usuário que foi cuidadosamente elaborada para induzir os usuários a fazer
> coisas… elas não são erros, foram cuidadosamente elaboradas com uma sólida compreensão da
> psicologia humana e não têm os interesses do usuário em mente"

The author accepts this is the widely replicated definition, and then attacks it:

- **Which** interests of the user should we have in mind?
- What's the difference between **inducing** and **motivating** a user toward a path we believe is
  ideal for both sides (business and user)?
- Is **deceiving** the user (hiding or forging information at the point of sale) the same thing as
  practicing dark patterns?

---

## The taxonomy — 12 deceptive patterns (deceptive.design, Brignull)

Use this as a review checklist against a real flow.

| Pattern | What it does |
|---|---|
| **Trick questions** (Perguntas-pegadinha) | A form question that reads as asking one thing when skimmed, but asks something else entirely when read carefully |
| **Sneak into basket** (Esgueirar-se na cesta) | The site inserts an extra item into your cart during the purchase journey |
| **Roach motel** (Motel barato) | Easy to get into, hard to get out of — e.g. a premium subscription |
| **Privacy Zuckering** | You're led to publicly share more about yourself than you intended |
| **Price comparison prevention** | The store makes it hard to compare an item's price with another, so you can't decide with full information |
| **Misdirection** (Desvio de atenção) | The design deliberately focuses your attention on one thing to distract you from another |
| **Hidden costs** (Custos escondidos) | Unexpected charges appear at the final checkout step (e.g. delivery fee) |
| **Bait and switch** (Isca e troca) | You set out to do one thing, a different, undesirable thing happens — the tempting offer isn't available at checkout, and the store offers business-friendlier alternatives |
| **Confirmshaming** (Vergonha de confirmação) | The decline option is worded to shame the user — "eu não quero ganhar desconto" vs "eu quero desconto!" |
| **Disguised ads** (Anúncios disfarçados) | Ads dressed as content or navigation so you click them |
| **Forced continuity** (Continuidade forçada) | The free trial ends and your card is silently charged with no warning |
| **Friend spam** (Spam de amigos) | The product requests email/social permissions under a desirable pretext (find friends), then spams your contacts with a message claiming to be from you |

Note which of these show up as **cases** in the other references: confirmshaming (XP), roach motel
(Adobe cancellation flow), forced continuity (auto-renewal without prior notice), sneak into basket
(GoDaddy's pre-selected add-ons). The book documents them as real, working, revenue-generating
designs *and* lists them here. That tension is deliberate — it's the whole point of the chapter.

---

## Where the author breaks with Nielsen Norman Group

NN/g distinguishes **Dark Patterns** from **Persuasive Patterns**, arguing the essential difference
is the **intent and the outcome** of interacting with the design. Their example: the **Scarcity
Principle** (people assign more value to what they perceive as scarce). NN/g says there's no harm in
using it to make users decide faster before a product or opportunity runs out — not inherently a
dark pattern, **unless the scarcity is forged**.

**The author disagrees fundamentally:**

> "Se um site é projetado de tal forma a influenciar uma compra, então talvez o usuário acabe
> comprando por impulso, ou optando por algo que não tivesse pensando em adquirir, ou simplesmente
> sendo persuadido a investir em algo que não o faria naquele momento; **mesmo sendo real a
> escassez**."
>
> "Logo, discordo da distinção entre Padrões Obscuros e Persuasivos. Ambos têm o poder de influenciar
> as pessoas a realizarem alguma ação, favorecendo o negócio em última instância."

---

## His resolution: influence is unavoidable, deception is not

The argument runs: **we are influenced by [almost] everything around us, all the time.**

His examples — the dog resting its paws under its head and staring at your steak; your partner using
affectionate words in a soft tone to ask for a gift; choosing a dish because of beautiful photos and
adjectives on the menu; grabbing a KitKat at the pharmacy checkout; buying the pricier product
because half a dozen people said it was worth it; your idol endorsing a politician and changing your
vote.

> "Intencionalmente ou não, um design (de um site, aplicativo, ou qualquer produto, ou serviço) pode
> alterar nossa percepção sobre algo e fazer com que tomemos decisões baseadas simplesmente pela
> forma como ele foi concebido."

Because of a color tone, an image, words read, element arrangement, the beauty (or lack of it) of a
screen. **"Tudo que vemos, ouvimos e sentimos pode moldar nossos julgamentos, e interferir em nossas
decisões."**

And so he raises the questions that make "intent" unusable as the dividing line:

- If the difference between dark patterns and persuasive design is intent, **what makes an intent
  good or bad?**
- How can we be sure a design that influences people **will really be good for the user** and fully
  meet their needs and interests?
- How could we know, that deeply, the countless humans who use our products?
- When money and data are involved, the scenario becomes **sensitive, dubious, and murky** as to what
  is right and wrong.

**His conclusion:** the term "dark patterns" isn't ideal. What actually exists are designs that
**deceive users, with false information and forged options** — and *that* doesn't necessarily have
anything to do with applying psychology or producing cognitive biases. He calls them **Designs
Impostores**. Brignull himself now calls them **Deceptive Design**.

---

## The gate to actually run

> "Como profissionais que criam e desenvolvem produtos e serviços que afetam a vida das pessoas,
> precisamos ter a responsabilidade de influenciar nossos usuários para o bem. Pense no negócio, mas
> principalmente nas pessoas que irão usufruí-lo. Até porque, se sua intenção é trapacear seu
> cliente, uma vez ciente que foi enganado, não irá querer tornar a vê-lo."

Three questions, in order:

1. **Is anything here false or forged?** Fake scarcity, fake original prices, hidden costs revealed
   at the last step, consent pre-checked where the user won't look, a decline button that lies about
   what declining means. If yes → it's a Designs Impostor. Stop. There is no version of this that's
   fine.
2. **Would the user still choose this if they saw the mechanism?** Show them the anchor, the default,
   the loss-framed copy. If the honest answer is "they'd feel cheated" → you're on the wrong side of
   the line the author drew.
3. **Does the business only win because the user loses?** The book's alignment test. Note the shape
   of every case that worked in it: the education platform converted at 30% because **"o produto era
   verdadeiramente bom e puderam experimentá-lo"** — the bias amplified a real value. It did not
   replace one.

The last argument is also the practical one: deception has a **one-shot payoff**. The user finds out,
and doesn't come back.

---

## Human Experience Design (Epilogue) — the frame behind the whole book

The author's proposed concept, also "HX Design". Not a new term he wants adopted — "a área já se
encontra saturada de nomenclaturas criadas amiudadamente" — but a different way of seeing the field,
the professionals, and especially the user.

| | UX Design (as practiced) | HX Design (proposed) |
|---|---|---|
| Focus | Strong concern with **methods and techniques**; design built from user needs and research insights | Directs effort at **transforming the world around the human** who uses the product |
| Unit of interaction | The individual **strictly in their relationship with the digital product**; the screen as the interaction point | The screen is one touchpoint of an integral human being |
| View of the person | The "user" | People are complex, full of particular emotions and perceptions — an aggregate of experiences and lived history. **"O 'usuário' representa apenas uma pequena parte do ser humano"** |
| Foundation | Interaction design, methods | Interaction design + **applied psychology** + cognitive science + research methods |

Supporting points from the Epilogue:

- **The field exploded**: per NN/g, from ~1,000 to ~1 million UX professionals between 1983 and 2017,
  with a forecast of ~100 million by 2050. A relatively small number of designers and engineers make
  decisions that impact billions.
- **Edward Tufte**, cited in *The Social Dilemma* (2020): "existem apenas duas indústrias que se
  referem a seus clientes como usuários, a de drogas e a de computadores". The author notes Netflix
  may have used the quote sensationally (the original refers to drugs generally, licit ones too), but
  keeps the emphasis: seeing people as mere pieces of a profitable game.
- **HXconf 2021** live poll: asked to choose between "Human Experience Design", "User Experience
  Design", or no preference, **44%** preferred "Human Experience Design" as the better description of
  the current design landscape.
- **Design longitudinally, not pointwise**: before drawing screens, don't just map the usage journey
  linearly (before, during, after) — map it **reverberated and longitudinal**, over a long period with
  the user. Identify the product's **impact on the lives** of those who'll use it. A to-do app
  designed wrong can disrupt someone's daily life. A well-designed CRM can help salespeople hit
  targets — "e gerar certa dependência de uso, também."

The closing definition:

> "O Design de Experiência Humana não é somente sobre desenhar telas com boa usabilidade e que
> proporcionem uma experiência agradável. É sobre entender o comportamento humano, os fenômenos
> psicológicos e vieses cognitivos que acontecem na mente das pessoas, realizar pesquisas para
> descobrir as melhores soluções e então criar designs que impactam a vida das pessoas."

---

## Cross-references

- The mechanisms this chapter governs: `value-perception.md`, `retention-and-inertia.md`,
  `choice-and-emotion.md`.
- `hooked:hooked-habit-testing` carries Nir Eyal's **Manipulation Matrix** ("Would I use this
  myself?" × "Does it materially improve the user's life?"). Eyal separates ethical from unethical by
  the maker's own use and the user's benefit; Dutra rejects intent-based lines and separates by
  **deception**. Running both gates is stricter than running either.
