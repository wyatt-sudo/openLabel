# Evaluation Rules

*Stage 6. Convert detections into scores, apply threshold disqualifiers, compute composite verdict, aggregate confidence. Where the literature offers empirical predictiveness data, weight signals accordingly.*

---

## The substantiation hierarchy

Strongest to weakest:

1. Multiple well-conducted product-specific human RCTs
2. One strong product-specific human RCT with appropriate design and endpoints
3. Product-specific pilot / observational human evidence with clear limitations
4. Mechanism-adjacent human literature on similar ingredients or modalities
5. Animal / in vitro / theoretical mechanism only
6. Testimonials / practitioner anecdotes / elite use / consumer reviews

A claim's `risk_category` (A/B/C/D) is acceptable when supported at the corresponding tier:

| Risk category | Minimum supporting tier | Examples |
|---|---|---|
| **A — Low** | 4–6 (mechanism / anecdotal acceptable) | Comfort claims, subjective experience, process descriptions |
| **B — Moderate** | 3–4 | General wellness support, non-disease performance |
| **C — High** | 1–3 | Quantified outcomes, "clinically proven" language |
| **D — Severe** | 1–2 | Disease treatment / mitigation / prevention claims |

`substantiation-gap` fires when a claim's category exceeds the available tier. Severity is the size of the gap.

---

## The net-impression rule

Score the *net impression* claim, not the narrowest literal wording. The full ad — headline, body, visuals, percentages, charts, testimonials, expert language, product naming, page layout, qualifiers, disclosures — is what a reasonable consumer takes away.

Implementation: claim records carry `claim_context` (surrounding section, headline above, qualifier proximity). Net-impression evaluation considers context, not just the claim sentence.

Disclosure presence does **not** automatically cure a misleading impression. The `inverse:disclosure-paradox` signal explicitly catches the moral-licensing effect (Loewenstein, Cain & Sah 2011): disclosed conflicts can produce *more* manipulation, not less. Disclosures are scored on their structural effectiveness (proximity, prominence, comprehension) — `disclosures.likely_curing_misimpression` from Stage 3 extraction.

---

## Severity-to-score conversion

Each detection carries a severity (low / medium / high / disqualify). Detections roll up to dimensions per `signals.dimension_tags`. The dimension score is computed:

- **Base 1.0** — no detections in this dimension above low severity
- **+0.5 to +1.5 per detection** depending on severity (low: +0.5, medium: +1.0, high: +1.5)
- Confidence scaling: low-confidence detections contribute at 0.5x; medium at 0.8x; high at 1.0x
- Capped at 5.0

This produces a 1–5 dimension score that's continuous with the legacy framework's scale, computable from the signal catalog without separate manual scoring.

```
dimension_score(D) = clamp(
    1 + sum_over_detections_for_D(
        severity_weight * confidence_weight
    ),
    1.0, 5.0
)
```

`dimension_scores` table records both the score and the contributing `detection_id` list for explainability.

---

## Composite Mission Integrity Score

Weighted average across scored dimensions per `methodology_versions.weights_json`:

```
methodology_v2.0 (current default):
  D1 Evidence Posture                  0.18
  D2 Technology Validity               0.13
  D3 Marketing Integrity               0.18
  D4 Conviviality                      0.13
  D5 Regulatory/Ethical Posture        0.09
  D7 Phenomenological Honesty (7a+7b)  0.09
  D9c Scope Honesty                    0.07
  D10c Mechanism Honesty               0.05
  consumer-specific aggregate           0.08
  --
  Total                                1.00
```

Tier B context dimensions (D6, commercial incentives, team credibility, loophole dependence, cultural trend) inform but do not enter the composite directly — they appear in the audit body and the company-portal remediation pathway.

Tier C internal dimensions (D8, lock-in) also do not enter the composite. D8 produces its own ethics verdict that can trigger threshold disqualification.

---

## Threshold disqualifiers

Any of the following automatically produces verdict = `Disqualify`, regardless of composite score:

- `D1 Evidence Posture` ≥ 4.5 (claims systematically misleading)
- `D3 Marketing Integrity` ≥ 4.5 (predatory marketing)
- `D5 Regulatory/Ethical Posture` ≥ 4.5 (active or imminent enforcement risk)
- `D10c Mechanism Honesty` = 5 *and* false mechanism claims central to product thesis
- `tactic-density-aggregate` count ≥ 8 (Predatory Apparatus per Mathur et al. 2019 — density alone is structural)
- `register:exploitation-with-targeting` with high-confidence evidence of intent
- `arbitrage:wellness-classification-disease-claims` central to product thesis
- `scope:complementary-care-discouraged` in a serious-condition context (cancer, severe mental health, anaphylaxis-risk)
- Any active FDA warning letter or FTC enforcement action against the offering

---

## Verdict ladder

| Composite | Verdict |
|---|---|
| 1.0–2.0 | **Surface** — epistemically sound; worth engaging |
| 2.1–3.0 | **Flag** — genuine utility with real concerns; requires human review |
| 3.1–4.0 | **Caution** — significant problems; engage only with explicit caveat |
| 4.1–5.0 | **Disqualify** — do not surface; flag as negative anchor |

A threshold disqualifier always overrides the ladder.

---

## Grey-area epistemology — the load-bearing principle

**Thin evidence is not the same as misleading.** Many offerings — especially in newly-emerging categories, integrative medicine, contemplative practice, somatic work, or chronic-illness-population care — will inherently lack rigorous RCT support. That makes them *more risky*, not ineffective or dishonest. The harm is in performing certainty unearned, not in being early.

OpenLabel's distinctive value is in this grey area. Generic LLMs handle established-evidence and obviously-fraudulent cases. OpenLabel's job is the harder middle.

The Evidence-Stage signal cluster (`06-signal-catalog.md` §19) does the work. Verdict logic incorporates these signals as follows:

### Verdict tagging by evidence stage

The four-tier verdict (Surface / Flag / Caution / Disqualify) gets an **evidence-stage tag** when relevant. The tag appears in the verdict rationale and the prescription label.

**Surface** can be reached three ways:

- **Surface — Established Evidence.** Substantiation tiers 1–3 dominant. RCT-grade evidence supports central claims. Honest framing throughout. The MAPS / Virta canonical pattern.
- **Surface — Pioneer.** Substantiation tiers 4–5 dominant, but `evidence-stage:reasonable-pilot-user-fit` fires (honest pilot framing + plausible mechanism + active evidence trajectory + reasonable safety + appropriate population fit). The offering is a reasonable choice for early adopters who explicitly accept the uncertainty.
- **Surface — Distributed Refinement.** `evidence-stage:traditional-lineage-faithful` fires for a category where `evidence-stage:RCT-design-mismatch` applies (contemplative practice, somatic work, narrative interventions). Distributed clinical refinement is the appropriate evidence form; the offering honors and faithfully transmits the tradition.

**Flag** can be reached:

- **Flag — Established with Concerns.** Real evidence base, but D3/D5 marketing-integrity or regulatory concerns warrant caution.
- **Flag — Pioneer with Over-claiming.** Pre-RCT evidence stage, but framing exceeds what evidence supports — not fully Surface-Pioneer because the company is leaning beyond its honest pilot framing.

**Caution** can be reached:

- **Caution — Established with Material Risk.** Real evidence base but safety, conviviality, or regulatory issues are significant.
- **Caution — Pre-RCT with Hidden Risk.** Thin evidence + safety concerns + population mismatch + opaque framing.
- **Caution — Pseudo-Lineage Extraction.** Borrowing legitimacy of established tradition while making claims beyond what the tradition supports (`evidence-stage:traditional-lineage-extracted`).

**Disqualify** is reached:

- **Disqualify — Predatory.** Across evidence stages — predation is what disqualifies, not stage. An offering can be predatory at established-evidence stage (claim inflation on real RCT findings) or predatory at pre-RCT stage (`evidence-stage:certainty-performed-without-earning`).

### Verdict-rationale composition

When composing the verdict rationale at Stage 7, surface the evidence stage explicitly:

- "Surface — Pioneer offering. Early evidence stage, honest framing, plausible mechanism, active trajectory. Reasonable for early adopters who accept the uncertainty."
- "Flag — Established offering with marketing-integrity concerns; the science supports a narrower claim than the marketing implies."
- "Caution — Pre-RCT offering performing certainty unearned. Mechanism plausibility is uncertain; framing exceeds it."
- "Disqualify — predatory across the evidence picture."

This gives consumers calibrated information instead of false binary judgments. **A Pioneer-stage Surface verdict is reachable** — and is exactly what OpenLabel should produce for an honest early-adopter offering.

### What this prevents

The grey-area framing prevents three failure modes:

1. **Punishing the underfunded.** Categories that haven't received research investment (chronic illness, contemplative practice, women's health, conditions affecting non-white populations, rare disease) shouldn't be auto-Disqualified for thin evidence when their thin evidence reflects research-funding patterns more than the offerings' integrity.

2. **Punishing the new.** A newly-launched offering with honest framing about being early shouldn't score worse than a long-running offering performing certainty.

3. **Punishing the appropriate epistemology.** A meditation practice with 2,000 years of distributed clinical refinement and no RCT shouldn't score the same as a recently-invented LED helmet with no RCT.

### What this still catches

The grey-area framing does not soften OpenLabel toward predatory offerings. It catches the difference. The signal `evidence-stage:certainty-performed-without-earning` is the gateway to Caution / Disqualify for offerings that perform certainty their evidence stage doesn't support — which is the actual harm pattern.

The Neuronic anchor case is unchanged: thin evidence + performed-certainty + chronic-illness-population targeting + mechanism-physically-incapable = Disqualify. The architecture preserves that.

---

## Empirical predictiveness — literature-informed weighting

The catalog assigns each tactic a literature-supported predictiveness for actual harm or enforcement. This determines severity ceilings, not weights directly.

Strongest predictors of FTC enforcement / consumer harm (ranked by evidence):

1. **`tactic:testimonial-as-mechanism-proof`** — most-cited tactic in FTC supplement enforcement actions
2. **`tactic:future-faking`** — central to weight-loss / anti-aging cases (FTC *Red Flag* report 2014)
3. **`tactic:authority-without-verification`** — fake doctors, fake studies cases
4. **`tactic:fear-escalation` × low-efficacy product** — Tannenbaum et al. 2015
5. **`tactic:reciprocity-negative-option`** — top consumer-complaint category per FTC ROSCA enforcement
6. **`tactic:social-proof-without-methodology`** — frequent FTC enforcement target
7. **`inverse:fda-registered-as-approval`** — directly misleading (Schwartz & Woloshin 2009)
8. **`inverse:celebrity-physician-halo`** — Korownyk 2014 found ~50% of celebrity-physician recommendations lack believable evidence

These get higher severity ceilings (a single high-confidence detection can elevate D3 to ≥4 on its own).

Weakly evidenced or context-dependent:

- `tactic:narrative-transportation` — validated as mechanism but tactic-marker evidence is moderate (Green & Brock established; deceptive-marketing-specific evidence thinner)
- `tactic:grief-frustration-mining` — real phenomenon but operationalization fuzzier (Kata 2012)

These get standard severity weighting and require corroboration from other signals to elevate dimension scores.

---

## Confidence aggregation

The audit's overall confidence label is computed from the distribution of per-detection confidences and source coverage.

```
audit_confidence = function(
    source_coverage,           # complete / substantive / provisional
    detection_confidence_distribution,  # % high / % medium / % low
    cross_source_agreement,    # do Tier 3 patient signal and Tier 1 evidence corroborate?
    methodology_version_age    # current methodology vs. drift since calibration
)
```

Output:

- **High** — Source coverage Complete AND ≥80% of contributing detections are high-confidence
- **Medium** — Source coverage Substantive AND ≥60% contributing detections high-or-medium-confidence
- **Low / Provisional** — Source coverage Provisional, OR significant cross-source disagreement, OR predominantly low-confidence detections

The audit's surface label uses this:
- High → "Deep audit"
- Medium → "Full audit"
- Low / Provisional → "Quick scan"

---

## When evidence is conflicting

Patient signal (Tier 3) and marketing claim sometimes diverge sharply. Examples:

- Patient community reports adverse effects the marketing does not mention.
- Patient community reports the offering helps via a different mechanism than the company claims.
- Patient community reports it does not work; marketing claims it does.

These are first-class data points, not edge cases. They feed:

- `phenomenological-colonization` (when patients report benefit through what is plausibly a non-specific factor while company claims a specific mechanism)
- `consumer:safety-undisclosed-contraindications` (when patients report harms not in marketing)
- `consumer:fit-misaligned` (when patient signal indicates a different best-fit population than marketed)

The `patient-marketing-gap` aggregate metric in `11-aggregate-research-and-analytics.md` tracks this systematically.

---

## Anchor calibration

Before promoting a methodology version, it is run against the anchor library (`10-anchor-library.md`). For each anchor case, the produced score and verdict are compared to the canonical / expected outcome.

Calibration acceptance criteria:

- Anchor cases scoring as designed (Surface anchors at 1.0–2.0, Disqualify anchors at 4.0+) for at least 6 of 7 cases.
- No anchor moves more than one verdict tier vs. its prior version (unless intentional).
- Disqualify-anchor disqualifications all triggered by the same threshold-disqualifier as in their canonical audit, OR a documented improvement in detection (e.g., a previously-missed signal now detected).

`audit_runs.config_json` for each anchor recalibration audit records the methodology version under test and the comparison result. The `anchor_drift` aggregate view shows movement over methodology iterations.

---

## Reproducibility check

Before any methodology update is committed:

1. Run the prior version on a test set (anchor library + last 100 audits).
2. Run the proposed version on the same set.
3. Diff the verdicts. Each verdict change must have a documented reason — a new signal added, a threshold tightened, an empirical recalibration.
4. Unexplained verdict changes block the methodology update.

This is the QA bar for methodology evolution. PatientPunk's `--reclassify` pattern is the operational model: re-run, preserve old, compare.

---

## Confidence in the methodology itself

OpenLabel's methodology is itself epistemically contestable — that's the point of `09-tone-and-stance.md` calibrated uncertainty. The evaluation rules carry their own uncertainty:

- Some signal weights are well-empirically calibrated (testimonial-as-mechanism, FDA-registered misuse).
- Some are theoretically well-grounded but less empirically tested in deceptive-marketing contexts (narrative transportation, register patterns).
- Some are intentionally novel and need calibration (cultural trend layer signals).

The aggregate research instrument (`11-aggregate-research-and-analytics.md`) lets us validate the methodology's predictiveness over time — comparing OpenLabel scores to subsequent FTC enforcement, patient-signal divergence, market-persistence outcomes. This is methodology QA at population scale.

---

*Update this document when scoring math changes, thresholds move, or empirical predictiveness data warrants reweighting. Methodology version bump is required for any change that affects produced verdicts.*
