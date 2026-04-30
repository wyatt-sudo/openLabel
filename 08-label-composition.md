# Label Composition

*Stage 7. How detected signals and extracted findings roll up into the prescription label, the warning panel, the verdict, and the three mode outputs. Composition is the last step — it queries the database, applies tone, and renders.*

---

## The prescription label is a query, not a write

Every section of the label is a structured query against detections + extractions + scores for the audit run. Composition rules are deterministic; tone is calibrated per `09-tone-and-stance.md`.

This means: any audit can be re-rendered (with current tone, current methodology) without re-running the pipeline. Composition is cheap and replayable.

---

## Label section composition rules

### Active ingredients

**What this section says:** the things this offering actually does, with evidence rating per ingredient.

**Inputs:**
- `mechanism_statements` (proposed mechanisms with plausibility/delivery ratings)
- `extracted_claims` filtered to type = efficacy and substantiation_tier 1–3
- `claim_evidence` for cited studies with population/dose/endpoint match

**Composition logic:**
- For each high-confidence well-substantiated claim → emit as Active ingredient
- For each mechanism with high plausibility AND high delivery → confirm as Active ingredient
- Evidence rating: strong / moderate / limited / none (computed from supporting `substantiation_tier`)
- If no claim qualifies → "Active ingredients: none clearly established"

**Tone:** name the ingredient and the evidence; do not equivocate when evidence is real.

### Also contains

**What this section says:** the non-active layers being sold — hope, identity, community, cosmology, habit structure, ritual, practitioner relationship.

**Inputs:**
- `phenomenological-colonization` detections
- `mechanism-overclaim` detections (where claimed mechanism doesn't deliver but offering still helps people)
- `tactic:narrative-transportation`, `tactic:exclusivity-access-illusion`, `tactic:grief-frustration-mining` — surface as identity / community / cosmology layers
- `extracted_claims` filtered to subjective / experiential
- Patient-signal results from Tier 3 (when available) — what patients report receiving vs. what's claimed

**Composition logic:**
- Where a benefit is real but the claimed mechanism is unlikely to be the cause → name what's plausibly happening (hope and expectancy activation; community membership; ritual structure; therapeutic alliance; nervous system co-regulation through trusted relational presence)
- Where identity / cosmology / belonging are clearly part of what the offering sells → name them descriptively
- Where the offering provides a coherent explanatory framework for suffering → name that explicitly

**Tone — load-bearing here.** This is where `09-tone-and-stance.md` matters most. "Also contains: hope and expectancy activation" is descriptive. "This is a placebo" is condemning. The first is correct; the second is wrong.

### Realistic dose

**What this section says:** what a typical user can actually expect — not the marketing-implied outcome, not the worst case, the typical case.

**Inputs:**
- `extracted_claims` (the marketing-implied outcome)
- `claim_evidence` (the substantiation tier and the typical effect size from cited studies)
- Patient-signal results (Tier 3) — what patients actually report
- `tactic:future-faking` detections — pattern of best-case-as-typical framing

**Composition logic:**
- State what evidence supports as a typical outcome.
- Name the gap between marketing-implied and evidence-supported expectation if material.
- For Mode 1 rapid triage: 1–2 sentences. For Mode 2: 2–3 sentences with specifics.

### Contraindications

**What this section says:** who should not use this; population-specific risks.

**Inputs:**
- `consumer:safety-undisclosed-contraindications` detections
- `consumer:safety-treatment-delay-risk` detections
- `scope:contraindications-absent` detections
- Per-category extension extractions (drug interactions for supplements; PEM risk flags; MCAS triggers; environmental sensitivity ingredients)
- Patient-signal results from Tier 3 — adverse events reported

**Composition logic:**
- Specific contraindication populations named (pregnancy, anticoagulation, MCAS-trigger sensitivity, PEM risk for ME/CFS, severe dysautonomia, cancer where treatment delay is a concern, immune compromise, etc.).
- Drug interactions if applicable.
- Treatment-delay risk flagged when serious.

**Tone:** specific and concrete. Vague "may not be suitable for some users" is useless. See `09-tone-and-stance.md` chronic-illness-specific commitments.

### Cheaper or better-evidenced alternatives

**What this section says:** lower-cost or better-supported options that serve the same need.

**Inputs:**
- `consumer:cost-active-ingredient-asymmetric` detections
- `consumer:alternatives-suppressed` detections
- `tactic:exclusivity-access-illusion` detections
- Category-specific known alternatives database (curated; expands over time)
- For chronic-illness population: peer-community resources, patient-organization resources

**Composition logic:**
- Lower-cost equivalents listed when the active ingredient is accessible elsewhere.
- Better-evidenced equivalents listed where applicable.
- Public-domain practices where applicable (breathwork, polyvagal exercises, dietary patterns, meditation traditions).
- Clinical alternatives when more appropriate (primary care, specialist referral, supervised programs).
- For chronic-illness population audits: lead with peer-community and patient-organization resources before clinical alternatives. (Per `09-tone-and-stance.md` — this population's relationship with mainstream medicine is often broken.)

### Warning panel (manipulation tactics)

**What this section says:** named persuasion tactics detected in the marketing.

**Inputs:**
- All `detections` rows where `signals.dimension_tags` contains `warning_panel`
- `tactic-density-aggregate` count

**Composition logic:**
- Density signal in plain language:
  - 0–2 → "Normal marketing — no significant red flags"
  - 3–4 → "Some persuasion pressure — read carefully"
  - 5–7 → "Heavy persuasion pressure — likely value asymmetry"
  - 8+ → "Predatory marketing pattern — recommend avoiding"
- Named tactics in consumer-readable language (translation table below)

**Translation table** (internal name → consumer surface):

| Catalog name | Surface phrasing |
|---|---|
| `tactic:urgency-scarcity` | "Urgency manufactured to bypass deliberation" |
| `tactic:social-proof-without-methodology` | "Stats without methodology" |
| `tactic:authority-without-verification` | "Authority signals that don't check out" |
| `tactic:future-faking` | "Promises bigger than the evidence" |
| `tactic:fear-escalation` | "Fear amplified beyond what evidence warrants" |
| `tactic:narrative-transportation` | "Story structure used as persuasion" |
| `tactic:grief-frustration-mining` | "Anti-mainstream-medicine framing used as proof" |
| `tactic:exclusivity-access-illusion` | "Positioned as the only path" |
| `tactic:testimonial-as-mechanism-proof` | "Testimonials used to prove mechanism" |
| `tactic:cognitive-load-distress-state-targeting` | "Targets the crisis state, not the person" |
| `tactic:reciprocity-negative-option` | "Free trial that auto-charges; cancellation is hard" |
| `tactic:pseudo-scientific-jargon` | "Scientific-sounding language without citations" |
| `tactic:parasocial-influencer-endorsement` | "Influencer credibility used in place of expertise" |
| `tactic:sludge-dark-patterns` | "Dark patterns in checkout / cancellation" |
| `tactic:decoy-pricing-anchoring` | "Pricing structured to anchor your perception" |
| `tactic:false-dichotomy` | "Either-or framing that isn't real" |
| `tactic:loss-aversion-sunk-cost` | "Pressure not to give up after partial commitment" |

**Inverse trust signals** also appear here, with surface phrasing:

| Catalog name | Surface phrasing |
|---|---|
| `inverse:doctor-recommended-without-source` | '"Doctor recommended" without verifiable source' |
| `inverse:fda-registered-as-approval` | '"FDA-registered" framed as if it means FDA-approved (it does not)' |
| `inverse:over-precise-statistics` | "Suspiciously precise outcome statistics" |
| `inverse:credential-stack-decorative` | "Credentials stacked but not domain-relevant" |
| `inverse:celebrity-physician-halo` | "Celebrity-physician endorsement (these have a poor evidence track record)" |
| `inverse:institutional-logo-halo` | "Institutional logos used without verified relationship" |
| `inverse:disclosure-paradox` | "Disclosed conflicts not paired with reduced pressure" |

### Business health panel (Mode 2 + Mode 3)

**What this section says:** business-side concerns that materially affect the consumer's decision — capital runway, investor pressure, exit-pathway volatility, founder track record, conflict-of-interest network, MLM structure, unit-economics asymmetry.

**Inputs:**
- All `detections` rows for signals tagged `business` or `mlm` (per `06-signal-catalog.md` §11–12)
- `team_members` with verification results
- `pricing_records`

**Composition logic:**
- Surface only when at least one business-cluster detection fires above low severity, OR when category requires multi-year continuity (subscriptions, devices, biomarker platforms).
- For each fired signal, emit a one-line plain-language statement.
- For MLM detections: surface prominently with the structural recognition ("This is a multi-level marketing company; you are being recruited as well as sold to") — this is consumer-decision-relevant.

**Tone:** matter-of-fact; describe the structure, do not condemn the people in it. "This company has a 9-month runway" is neutral data; "the founder is also the CEO of the supplement company they recommend" is neutral data; the consumer makes the judgment.

**Threshold disqualifier hooks:**
- Active enforcement / litigation in business cluster
- Capital insolvency for offerings requiring multi-year continuity
- Undisclosed material conflict of interest

### Ethics panel (serious-condition trigger)

**What this section says:** Beauchamp & Childress Four Principles assessment when the offering targets a serious condition (cancer-adjacent, severe mental health, infectious disease, severe chronic illness, anaphylaxis-risk, suicidality-relevant).

**Inputs:**
- `ethics:*` detections (per `06-signal-catalog.md` §14)
- Cross-references to `consumer:safety-treatment-delay-risk`

**Composition logic:**
- Surface only when category triggers and at least one ethics-cluster detection fires.
- One line per principle (Autonomy, Beneficence, Non-maleficence, Justice) with violation-or-honor language.

**Tone:** This is the most serious panel that can surface. Specific and concrete. Treatment-delay risk in serious-condition contexts is a `disqualify` capable signal — when it fires, do not soften it.

### Verdict badge

**What this section says:** Surface / Flag / Caution / Disqualify, plus one-sentence rationale.

**Inputs:**
- `verdicts.verdict`
- `verdicts.composite_score`
- `verdicts.threshold_disqualifier_triggered` (if any)

**Composition logic:**
- Verdict label.
- One-sentence rationale that names the dominant reason for the verdict.
- For Disqualify: name the threshold disqualifier.
- For Surface: name the strongest positive signal.

---

## Mode 1 — Rapid Triage output

**Constraint:** 30–60 second return; shareable.

**Structure:**

```
OPENLABEL — RAPID TRIAGE
[Product Name] · [Company]
[Category]

ACTIVE INGREDIENTS:
- [name] — [strong / moderate / limited evidence]
- ...

ALSO CONTAINS:
- [non-active layer]
- [non-active layer]

REALISTIC DOSE:
[1–2 sentences]

CONTRAINDICATIONS:
[1–2 sentences — population-specific where relevant]

⚠ MANIPULATION TACTICS: [N]/17
[Density signal in plain language]

VERDICT: [Surface / Flag / Caution / Disqualify]
[One-sentence rationale]

→ Tap for full audit
```

**Source coverage label:** prominent. "Quick scan — based on company marketing surface, PubMed, and FDA / FTC databases."

**Sharing affordances:** renders as PNG share card with watermark (OpenLabel URL + audit date). Suitable for Instagram / TikTok / X.

---

## Mode 2 — Full Audit output

**Constraint:** trustworthy, comprehensive, consumer-readable. Length budget: ~2 screens scrolled.

**Structure:**

```
OPENLABEL — FULL AUDIT
[Product Name] · [Company]
URL: [URL]
Audit date: [YYYY-MM-DD]
Source coverage: [Complete / Substantive but incomplete / Provisional]
Methodology: v[N.N.N]

────────────────────────────
[The label — full Active / Also Contains / Realistic Dose / Contraindications / Alternatives blocks]

────────────────────────────
⚠ WARNING PANEL
Manipulation tactics: [count]/17 — [signal]
Inverse trust signals: [count]/7 — [signal]
[Named tactics with one-line evidence each]

────────────────────────────
VERDICT: [verdict]
[1-sentence rationale]

────────────────────────────
DIMENSION-BY-DIMENSION
- D1 Evidence Posture: [score] — [one sentence]
- D2 Technology Validity: [score] — [one sentence]
- D3 Marketing Integrity: [score] — [one sentence]
- D4 Conviviality: [score] — [one sentence]
- D5 Regulatory Posture: [score] — [one sentence]
- D7 Phenomenological Honesty: [score] — [one sentence]
- D9c Scope Honesty: [score] — [one sentence]
- D10c Mechanism Honesty: [score] — [one sentence]
Composite Mission Integrity Score: [N.NN]

────────────────────────────
CULTURAL TREND LAYER
[Established / Active trend / Peak capitalization risk]
Where on the curve: [one-sentence positioning]
Score breakdown:
- Commercial vs scientific velocity: [0–3]
- Market saturation: [0–2]
- Claim expansion: [0–2]
- Patient-community precedence: [0–3]

────────────────────────────
PATIENT SIGNAL
Source: [Patient Punk / Reddit aggregate / "not yet integrated"]
What patients actually report:
- [pattern with evidence count]
- [pattern]
Gap from marketing claims: [one sentence]

────────────────────────────
CONSUMER FIT
Best for: [population description]
Poor fit for: [population description]
Red flag — do not use if: [specific contraindications]
Reasonable as a tool when: [conditions]
Risky as a destination when: [conditions]

────────────────────────────
PRACTICAL CONCERNS
Cost: $[annual]
Time commitment: [hours per week]
Data collected: [list]
Data portability: [exportable / company-only / not exportable]

────────────────────────────
SOURCES CHECKED
[Per-source list per `04-evidence-and-sources.md`]
```

---

## Mode 3 — Company Portal output

**Constraint:** the company sees what a sophisticated consumer would see, plus a remediation pathway. The point is to make the score *legible as a business signal*, not to surprise or shame.

**Structure:** same as Mode 2 in full, then appended:

```
────────────────────────────
WHAT WOULD NEED TO CHANGE FOR YOUR SCORE TO IMPROVE

D1 Evidence Posture (current: [score], target: [score]):
- [specific change tied to a specific detection]
- [specific change]

D3 Marketing Integrity (current: [score], target: [score]):
- [specific tactic to remove or modify]
- [specific change]

[and so on for any dimension scoring 3+]

CONSUMER-SPECIFIC LAYERS — DELIVERED VS CLAIMED:
- L2 Cost-to-Value: [what re-pricing review would change]
- L3 Practical Burden: [if burden is misaligned with claimed simplicity]
- L4 Data Rights: [specific portability or sharing improvements]

CULTURAL TREND POSITIONING:
[If at peak capitalization, what to do to differentiate from the trend rather than ride it]

ESTIMATED EFFORT TO REACH TARGET SCORE:
[Light editing of marketing surface / Moderate restructuring of claims and evidence /
Substantive product or business-model changes]

────────────────────────────
For a structured remediation engagement, contact [link to consulting intake]
```

---

## Information ordering — locked

The order of the primary label is intentional and **should not change**:

1. Offering name and category — orient the reader
2. Active ingredients — what does this actually do
3. Also contains — what else is being sold
4. Realistic dose — typical outcome
5. Contraindications — who should not use it
6. Cheaper or better-evidenced alternatives — what else could serve

Reading top-to-bottom should produce a coherent answer to *what is this offering, what does it do, what else is happening here, what should I expect, who shouldn't use it, what else could I do.* That is the experience the consumer needs.

---

## Tone application checkpoint

Every composed output passes through a tone check before rendering:

- Does any phrase fall under "what never goes in consumer output" (`09-tone-and-stance.md`)?
- Is internal scoring jargon present without consumer translation?
- Is the chronic-illness population's experience framed as a personal failure anywhere?
- Does the audit imply someone who got better should have known better?

If any check fails, the composition rule re-renders that section with the corrected phrasing. This is implementable as a post-composition LLM-strong pass with the tone document loaded.

---

## What never appears in any mode

- Personal attacks on founders or team members
- Mockery of the patient population, even by implication
- Speculation about founder motivation framed as fact
- Brand-voice catchphrases that turn the audit into entertainment
- Internal scoring jargon without translation
- Cosmological dismissal
- Verdicts that cannot be defended from the source list
- Decorative emoji except where intentional and consistent

---

*Update this document when output formats are user-tested and revised. Likely revision points: the Active vs. Also Contains distinction in plain language; how to surface manipulation-tactic detail without overwhelming a 60-second triage; the Mode 3 remediation pathway after first real Mode 3 engagements.*
