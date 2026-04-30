# Aggregate Research and Industry Analytics

*OpenLabel's second product surface. As audit volume accumulates, the database becomes an industry research instrument that produces insight no other tool currently produces. This document specifies what gets aggregated, what insights to publish, what we can learn that informs methodology back, and how the research instrument creates incentive structure on the industry.*

---

## The thesis

The consumer audit tool (Product 1) is the visible surface. The aggregate research instrument (Product 2) is the compounding asset.

Per the competitive research (`references/literature-review.md` §4), no incumbent currently aggregates **marketing-integrity / claim-substantiation / manipulation-tactics signal across the consumer health industry at scale.** ConsumerLab tests physical product. Examine evaluates ingredient evidence. Industry analysts (McKinsey, Statista) report market size. NCCIH publishes evidence summaries at academic pace. Nobody publishes "manipulation-tactic prevalence by category, time-series" or "patient-signal-vs-marketing-claim gap by population" or "which categories are at peak capitalization risk right now."

This is OpenLabel's defensible center.

---

## What gets aggregated

Every audit writes to the same database under run traceability (see `03-data-model.md`). The aggregate research instrument is built from queries against that database. Every publishable insight has a SQL view behind it.

The aggregations fall into four families:

| Family | What it shows | Time horizon |
|---|---|---|
| **Industry composition snapshots** | What does the industry look like right now? | Annual / quarterly |
| **Trend tracking** | How is it changing? | Monthly / quarterly |
| **Cohort comparisons** | How do sub-segments differ? | On-demand |
| **Predictive validation** | Does our methodology actually predict outcomes? | Longitudinal |

---

## Aggregation families — detailed

### Family 1 — Industry composition snapshots

Run quarterly. Publish annually as the "State of the Wellness Industry — Marketing Integrity" report.

**Verdict distribution by category.**
For each category (supplement, device, app, biomarker platform, coaching, etc.), what % of audited offerings score Surface / Flag / Caution / Disqualify? Time-series over 8+ quarters.

**Manipulation tactic prevalence by category.**
For each category, what % of audited offerings have each tactic? Heatmap: tactics × categories.

**Tactic density distribution.**
Histogram of manipulation tactic densities (0–10+) across the corpus, sliced by category and target population.

**Cultural-trend score distribution.**
Histogram of cultural-trend layer scores (0–10) by category. Surfaces which categories are at "peak capitalization risk."

**Source-coverage realism.**
What % of audits achieve "Complete" source coverage in a given period? Tracks the ground-truth limit on what OpenLabel can know about an offering.

**Population-exposure profile.**
What aggregate score profile does each target population face? The chronic illness population vs. general wellness vs. performance vs. longevity. *This is one of the most important publications because it answers "how bad is the marketplace I'm forced to navigate."*

### Family 2 — Trend tracking

Run monthly internally; publish significant shifts.

**Tactic-prevalence trajectories.**
Has manipulation tactic X become more or less prevalent in category Y over the last 12 months? E.g., did "Heavy persuasion pressure" rise in peptide products from 2026 Q1 to 2026 Q4?

**Claim inflation curves.**
For specific ingredients/modalities: are claims expanding as commercial activity grows? (Wayback-derived; cross-audited.)

**Gold-rush detection.**
Categories where market-saturation signal is rising AND claim-expansion signal is rising. Early warning of a wave.

**Enforcement correlation.**
For offerings that received FTC/FDA enforcement, what were their OpenLabel scores 6 / 12 / 24 months prior? Lag analysis. Validates whether OpenLabel detects enforcement-bound offerings before regulators do.

**Patient-signal divergence over time.**
For offerings with patient-signal coverage: how does the gap between marketing claims and patient-reported outcomes change over time? Some products grow into their claims; others diverge.

### Family 3 — Cohort comparisons

On-demand for journalist / regulator / academic queries.

**Chronic-illness-targeting vs general-wellness.**
Score distributions, tactic prevalence, regulatory posture. Does the chronic illness corpus differ from the general wellness corpus in measurable ways? *(Hypothesis from `references/literature-review.md` §3: chronic illness population sees significantly higher tactic density and lower source-coverage by anchor offerings.)*

**VC-backed vs bootstrapped.**
Different tactic profiles? Different LTV / extraction patterns? Different scope-honesty?

**Influencer-promoted vs not.**
Does the path-to-discovery (influencer ad vs editorial vs clinical referral) correlate with the marketing-integrity score?

**First-mover incumbents vs late-cycle entrants.**
For offerings in trended categories, do early entrants score better than late entrants? Tests the "early adopters benefit, late entrants buy peak hype" theory.

**Cross-methodology-version comparison.**
Same offerings audited under v1 and v2 methodology. How do scores shift? Where does methodology produce different verdicts?

### Family 4 — Predictive validation

The most important and most slow.

**Does dimension D1 (Evidence Posture) predict subsequent FTC/FDA enforcement?**
Cohort: offerings audited at time t. Outcome: enforcement action by t+24mo. Statistical test for predictive validity.

**Does manipulation-tactic density predict subsequent enforcement?**
Same structure. *Hypothesis from research: density should be the strongest single predictor.*

**Does cultural-trend layer score predict market exit / category collapse?**
Cohort: offerings audited at peak-capitalization (trend score 7+). Outcome: market presence / pricing / claim retention at t+24mo.

**Does Mode 1 rapid triage agree with Mode 2 full audit?**
Inter-mode reliability. If Mode 1 verdict diverges meaningfully from Mode 2 on the same offering, the rapid-triage methodology needs work.

**Does Mode 1 / Mode 2 verdict agree with patient-signal verdict?**
When patient-signal is available: do offerings with low marketing-integrity scores actually under-deliver per patient reports?

**Inverse trust signals — empirical validation.**
For offerings with high `inverse-trust-signal-density` (S-T02), do they actually underperform on subsequent indicators? Validates the literature-derived inverse-signal catalog (S-201–S-207) using OpenLabel's own data.

---

## Publication formats

The aggregate research instrument outputs to multiple surfaces:

| Surface | Audience | Cadence | Format |
|---|---|---|---|
| **State of the Wellness Industry — Marketing Integrity** | Public / press | Annual | Long-form report with visualizations |
| **Category deep-dives** | Public / press | Quarterly | Single-category focus reports (e.g., "Peptides — 2026 Q3") |
| **Population reports** | Patient orgs, advocacy | Annual | "What chronic illness consumers actually face" |
| **Methodology notes** | Academic | Quarterly | Methodology validation studies, version drift, anchor recalibration |
| **Real-time dashboard** | Internal + B2B subscribers | Live | Trend-tracking, gold-rush detection, enforcement correlation |
| **Press / journalist queries** | Press | On-demand | Customized cohort analyses for stories |

The State of report is the flagship. Done well, it's the document that establishes OpenLabel as the definitive source on industry marketing integrity — the way IDC's market reports define the IT industry, or Edelman Trust Barometer defines trust measurement.

---

## What we can infer (speculation, ranked by likely value)

These are the kinds of insights the aggregate dataset could surface. None are guaranteed; all are structurally accessible given the data model.

### High-value (most likely to land)

1. **Manipulation tactic density predicts FTC enforcement** — if density-at-audit-time correlates with enforcement-by-t+24mo, OpenLabel becomes a leading indicator of regulatory action. This is the single most B2B-valuable insight (insurers, retailers, payment processors all want this signal).

2. **Population-targeted predation density.** "The chronic illness corpus has 2.3x the average tactic density of the general wellness corpus." This kind of finding, well-documented, is press-worthy and policy-relevant.

3. **Category gold-rush early warning.** "Mushroom adaptogen tactic density rose 40% YoY through 2026; new entrant rate doubled; patient-community precedence dropped from 0.6 to 2.1." Categories at risk become legibly identifiable before consumer harm peaks.

4. **Patient-claim gap by population.** "For Long COVID offerings, average gap between marketing efficacy claims and patient-reported outcomes is X." Quantifies what the chronic illness population already knows experientially.

5. **Inverse trust signal density correlates with low scores.** Validates the literature catalog using OpenLabel's own data and provides a self-reinforcing mechanism for catalog maintenance.

### Medium-value (more speculative but possible)

6. **Founder register patterns predict trajectory.** Expert-Pluralist sincere-belief founders produce different scoring profiles than Achiever-Expert commercial-press founders. If reproducible, this is a methodology validation of the developmental framework.

7. **Disclaimer-quality reverse correlation.** Offerings with poor-quality disclaimers (present but buried) score worse on D1 than offerings with no disclaimers. Validates Loewenstein paradoxical-disclosure findings in our own corpus.

8. **Third-party certification predicts D2 score.** USP/NSF/Informed-Sport-certified offerings systematically score better on D2 (technology validity / delivery). Validates the literature signal.

9. **Pricing extraction patterns by population.** "The chronic illness corpus has higher average price for equivalent active ingredient than the general wellness corpus." Quantifies pricing-of-desperation if it exists in the data.

10. **Patent-claim language vs marketing-claim language gap.** USPTO assignee filings reveal claimed mechanism scope under oath; marketing language often exceeds it. The gap is queryable.

### Speculative (interesting if true)

11. **Methodology drift validation.** Re-auditing anchor cases under newer methodology versions reveals whether updates produce intended effects.

12. **Influencer-promotion is a decisive verdict input.** Offerings discovered via influencer path systematically score worse — or systematically equal, after controlling for other features.

13. **Geographic clustering.** Certain claim categories cluster in certain regions (LA-based wellness, NYC-based optimization, Bay-area-biohacker, etc.). Tactical signatures may differ.

14. **Funding-round-stage predicts tactic density.** Series A vs Series C vs bootstrapped — does growth-pressure correlate with claim inflation?

15. **Same-active-ingredient-different-marketing.** When 3+ companies market the same active ingredient, what's the claim-language variance? Identifies "marketing arbitrage" — same ingredient, different claim density.

---

## How the research instrument informs assessment back

Aggregate findings feed methodology iteration. This is the validation flywheel.

**Validate signals empirically.** A signal in the catalog is a hypothesis. Aggregate analysis tests whether detection of S-201 actually correlates with downstream low scores or enforcement. If a signal doesn't correlate, retire it.

**Discover new signals.** Cross-audit pattern recognition — clusters of audits that score similarly for unobvious reasons — reveals patterns the current catalog misses. Add them.

**Calibrate weights.** Dimension weights in `07-evaluation-rules.md` are inherited from the legacy methodology. With aggregate data, weight them by predictive validity for actual outcomes (enforcement, market exit, patient-signal divergence).

**Identify population-specific calibration.** Generic methodology may produce poor verdicts for niche populations. Aggregate analysis reveals when the chronic illness corpus needs different threshold disqualifiers than the general wellness corpus.

**Anchor library curation.** Anchor cases that have "drifted" — re-audited at a methodology version that scores them dramatically differently — get re-evaluated. Anchors are the calibration substrate; aggregate drift is QA.

---

## Incentive structure speculation

If OpenLabel's aggregate reporting becomes credible and visible, what happens to industry behavior?

### First-order effects

**Companies start tracking their own OpenLabel score.** Public visibility of methodology means founders / boards / investors can predict their score before submission. Internal pressure to "score well" emerges.

**Tactic-level pressure.** Specific tactics (e.g., S-109 testimonial-as-mechanism-proof) become category-specific shame markers. "Don't be the only one in the category still doing this" pressure.

**Disclosure quality improvement.** Once disclaimer-quality (S-207) is published as an inverse signal, companies have incentive to comply with FTC .com Disclosures guidance, not just include any disclaimer.

**Third-party certification becomes commercially rational.** Once S-301 (USP/NSF certification presence) is published as a strong positive signal, certification ROI improves. Currently <1% of supplements are certified; that number could move materially.

### Second-order effects (the more interesting ones)

**A "race to honesty" sub-segment emerges.** Companies that *want* to score well find each other and become a coherent commercial cluster — distinguishable from the manipulation-saturated cluster. The chronic illness population gains a reliable filter for "this company is operating differently."

**Late-cycle category exit accelerates.** When OpenLabel publishes that a category is at peak capitalization, late-entrant capital becomes harder to raise. Categories collapse faster, with less harm at the back end.

**Regulatory action becomes more efficient.** FTC / FDA can use OpenLabel aggregate data as a triage layer — focus enforcement on the highest-tactic-density / highest-population-targeted offerings. This already happens informally with consumer complaints; OpenLabel structures it.

**Patient organizations gain a tool.** Solve M.E., Dysautonomia International, MAPS, NORD, similar — gain a methodology and dataset to back up advocacy. "Here is the structured evidence that this category is preying on our population."

**Investor diligence shifts.** Health-tech VCs incorporate marketing-integrity score into diligence. Companies that would previously have been valued purely on growth metrics now carry a quantifiable epistemic-risk discount.

### Third-order effects (most speculative, highest leverage)

**Shifts in cultural-trend behavior.** Public visibility of the cultural-trend layer (Family 2 above) makes "you are at peak capitalization risk" an early-warning consumer signal — the gold-rush dynamic itself dampens. Categories may have shorter, smaller bubbles.

**The "convivial cluster" becomes a venture category.** Companies designed from inception to score well become a recognizable investment thesis — analogous to "B Corps" but specifically for health/wellness epistemic integrity.

**A "wellness product literacy" public-education arc.** Sustained publication of OpenLabel data across press, schools, patient orgs raises baseline consumer epistemic capacity. The "doctor recommended" claim loses force at population scale.

**OpenLabel becomes infrastructure.** Like the FDA Adverse Event Reporting System or CDC VAERS — a public-good database that other tools and research depend on. Researchers cite OpenLabel; journalists cite OpenLabel; regulators cite OpenLabel.

The market-shaping ambition materializes through this data layer, not through individual audits. **The audits are the breadcrumbs; the database is the road.**

---

## What we should also assess (informed by research findings)

The aggregate-research lens reveals assessment dimensions worth adding to the methodology beyond what `06-signal-catalog.md` already enumerates.

### From the literature review

1. **Reciprocity / negative-option subscription patterns.** Already added as S-111. Top FTC enforcement category. Make sure the aggregate dashboard tracks this prominently.

2. **Pseudo-scientific jargon density without citations.** Already added as S-112. The "seductive allure of neuroscience" is documented; aggregate-level tracking shows category drift.

3. **Influencer-disclosure compliance.** Already proposed as a future signal. Aggregate-level tracking reveals which categories are most §255-noncompliant.

4. **Adulteration-prone-category flag (S-803).** Tucker 2018 documents weight loss / sexual enhancement / muscle building as the highest-pharmaceutical-adulteration categories. Aggregate-level tracking quantifies this in our corpus.

5. **Treatment-delay risk (S-802).** Johnson 2018 — alternative-only cancer treatment correlates with 2.5x mortality. The aggregate dataset can track which offerings position themselves as alternatives to evidence-based care for serious conditions, by population.

### Newly-warranted additions

6. **Compensation structure analysis** for offerings sold via MLM / multi-level structures. Bosley & McKeage 2015 documents specific MLM persuasion patterns; signal worth adding to catalog.

7. **Synthetic-review density** detection on third-party review platforms. Post-2024 LLM-review-generation has created a measurable distortion; OpenLabel should monitor it.

8. **Affiliate-link density** in "comparison sites" / "review sites" promoting an offering. Ecology distortion signal.

9. **Founder-as-only-testimonial-subject** flag. Frequently observed pattern, especially in chronic illness offerings. Worth a dedicated signal.

10. **Wayback historical-claim-retraction tracking.** Did the company quietly remove claims that were challenged? Positive (transparency) or negative (silent walkback) depending on disclosure.

These should be incorporated into `06-signal-catalog.md` as the catalog matures.

---

## Privacy and ethics

Aggregate analysis must protect:

**Individual offerings' audit confidentiality** for Mode 3 company-portal submissions. Companies submit knowing their audit is private to them; aggregate analytics never expose a single offering's score within a small bucket where they're identifiable.

**Patient-signal source privacy.** Patient Punk pattern: SHA-256 hash usernames, never store raw. When publishing aggregate patient-signal findings, use category-level rollups; never quote individual patient posts identifiably.

**Researcher access.** External researchers who want to query the OpenLabel database for academic work get access under data-use agreements that prevent re-identification and require methodology-version disclosure in any publication.

**Public publication threshold.** Aggregate findings are published only when N > some threshold (e.g., minimum 30 audits per cell) to prevent reverse-identification. Smaller cells reported as "insufficient sample."

---

## Cadence and operational pattern

The aggregate research instrument runs on a cadence that doesn't burden the per-audit pipeline:

- **Daily:** materialized-view refresh on small-N aggregates
- **Weekly:** anchor-case re-audit for drift QA
- **Monthly:** trend-tracking reports for internal review
- **Quarterly:** category deep-dives published
- **Annually:** State of the Wellness Industry report
- **Continuously:** Mode 3 client-specific queries on-demand

Background workers, separate from the latency-sensitive Mode 1 pipeline. PatientPunk's two-timescale pattern (per-audit vs corpus-wide) maps directly.

---

## Open questions

1. **Publishing model.** Open-source / fully public, gated, or freemium with deeper cuts behind a B2B layer? Default position: aggregate trend reports public; raw queryable database access B2B (regulators, journalists, retailers, insurers, investors).

2. **Audit submission incentives.** What gets a consumer to submit an audit? The verdict for them. What gets a company to submit Mode 3? The remediation pathway and the hope of improving their score before public exposure.

3. **Patient Punk integration impact.** Once integrated, the patient-signal aggregations become qualitatively different. Most of the speculation above assumes Patient Punk is online; some Family 2 / Family 4 insights aren't possible without it.

4. **Cross-vendor data sharing.** Does OpenLabel's aggregate data get shared with regulators, retailers, payment processors? Each relationship changes the leverage profile — and the political target the project becomes.

5. **Methodology contestation.** When OpenLabel publishes that a specific category is at peak capitalization, companies in that category will push back. The methodology must be public, versioned, reproducible. Cochrane-level transparency is the defense.

---

*This document specifies the second product surface. Update when new aggregation views are defined, when publication formats change, or when the incentive analysis is refined by actual industry response. The aggregate dataset is a long-horizon asset; build the data architecture to capture what we will want to query 5 years from now, not just what we want today.*
