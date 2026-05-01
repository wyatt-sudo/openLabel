# Anchor Library

*Calibration cases. The anchor library is a structured corpus of audited offerings used to calibrate the methodology, validate scoring conversions, and check drift across methodology versions. Supplemented with citation-grade industry context where relevant.*

---

## How to use this library

Load alongside `06-signal-catalog.md` and `07-evaluation-rules.md` when running an audit. Cross-check your dimension scores against these cases before finalizing.

When you score a new offering, ask: *does my score for this dimension make sense relative to how the anchor offering scored?*

The anchor library is also the QA substrate for methodology evolution. Before any `methodology_versions` semver bump is committed, the new methodology is run against every anchor case and scored deltas are reviewed. See `07-evaluation-rules.md` "Anchor calibration" section.

The anchors are queryable in the database (`anchor_cases` table with each case's frozen canonical `audit_run_id`).

---

## Quick reference table

| Offering | Verdict | MI score | Pattern type | Tactic density | Trend layer |
|---|---|---|---|---|---|
| MAPS PBC | **Surface** | 1.1 | Score-1 anchor | 0–1 | Established |
| Virta Health | **Surface** | 1.4 | Score-1 anchor | 1–2 | Established |
| HeartMath | **Flag** | 2.4 | Score-2/3 anchor | 3–4 | Active trend |
| Levels Health | **Flag** | 2.9 | Score-3 anchor | 3–4 | Active trend |
| Eight Sleep | **Caution** | 3.3 | Score-3/4 anchor | 4–5 | Active trend |
| Superpower | **Disqualify** | 4.05 | Disqualify anchor | 6–7 | Peak capitalization |
| Neuronic | **Disqualify** | 4.0 | Disqualify anchor | 6–8 | Active trend (PBM/POTS) |

*Tactic density and trend-layer figures are illustrative for the original audits. The v2.2 manual rerun below preserves these canonical verdicts and documents drift.*

---

## v2 signal-cluster calibration matrix

This first-pass calibration maps the v2.2 clusters added after the original anchor audits. It is a drift-control aid, not a substitute for full reruns.

| Offering | Cultural-trend calibration | Business-health calibration | MLM calibration | SDT calibration | Evidence-stage calibration |
|---|---|---|---|---|---|
| MAPS PBC | Established / patient-led precedent; low commercial-velocity concern. | Regulatory and financing risk exists, but not consumer-extraction risk by itself. | Negative control. | Autonomy-supportive protocol; competence and agency preserved. | Established Evidence; strong uncertainty-holding history. |
| Virta Health | Established clinical category; low trend-hype pressure relative to evidence. | Subscription-continuity risk is honest and clinically tied to maintenance. | Negative control. | Coaching can build competence when framed as skill acquisition. | Established Evidence; ongoing publication culture. |
| HeartMath | Active-trend / re-emergent practice edge case; avoid punishing practice maturity. | Certification and proprietary-wrapper economics are the key business questions. | Negative control unless downline compensation appears. | Mild competence / relatedness displacement through proprietary cosmology and certification identity. | Practice-maturity middle; not fake, but commercial claims outrun evidence. |
| Levels Health | Active commercial trend in non-diabetic CGM. | Subscription, data, and investor-pressure questions matter; not disqualifying alone. | Negative control. | Dashboard dependency risk; competence-building depends on whether users learn transferable skills. | Pioneer / emerging evidence with overclaim risk. |
| Eight Sleep | Active sleep-tech / recovery optimization trend. | Premium hardware, subscription continuity, AI interpretation, and exit/privacy risk matter. | Negative control. | Proprietary score and black-box interpretation can erode competence. | Plausible mechanism with clinical-language overreach. |
| Superpower | Peak capitalization in longevity / personalized diagnostics. | Unit-economics, investor-pressure, litigation, privacy, and exit-pathway risks are central. | Negative control unless distributor recruitment appears. | Autonomy erosion via rescue narrative and biomarker anxiety loops. | Real diagnostic substrate plus unearned certainty. |
| Neuronic | Active PBM/POTS/chronic-illness trend; patient precedence does not rescue the case. | High price + thin evidence + small-company continuity risk become consumer harm. | Negative control; chronic-illness exploitation is not MLM by default. | High-severity autonomy / competence risk despite possible felt agency restoration. | Thin evidence + performed certainty + implausible delivery + vulnerable-population targeting. |

**Calibration decisions:**

- MLM signals require compensation-structure evidence. Community, certification, affiliate, influencer, ambassador, or subscription language is insufficient on its own.
- Business due-diligence signals should not punish ordinary startup risk. They matter when capital pressure, continuity risk, pricing, data rights, or conflicts of interest change the consumer's expected harm profile.
- Cultural-trend scores are context multipliers, not verdicts. Patient precedence is positive only when paired with plausible delivery, safety, and honest claim scope.
- SDT signals should distinguish expert-supported agency from agency replacement. Coaching and protocols can be autonomy-supportive when they build durable user competence.
- Evidence-stage tags should prevent two errors: over-penalizing honest early offerings and under-penalizing thin-evidence offerings that perform certainty.

---

## v2.2 manual anchor rerun

Manual pre-runtime rerun against `06-signal-catalog.md` v2.2 and `07-evaluation-rules.md`. This is the methodology QA pass available before the audit pipeline exists; the database-backed rerun should preserve old / new `audit_run_id` pairs once runtime infrastructure is built.

| Offering | v2.2 verdict tag | Prior MI | v2.2 MI | Drift | Confidence | Rerun note |
|---|---|---:|---:|---:|---|---|
| MAPS PBC | Surface — Established Evidence | 1.1 | 1.1 | 0.0 | High | New clusters are negative / positive controls: patient precedence, uncertainty-holding, and low tactic density preserve the score. |
| Virta Health | Surface — Established Evidence | 1.4 | 1.4 | 0.0 | High | Subscription dependency remains a low-medium conviviality signal because it is clinically tied to maintenance and honestly framed. |
| HeartMath | Flag — Distributed Refinement with Commercial Overreach | 2.4 | 2.45 | +0.05 | Medium | SDT and business-health notes add mild certification / proprietary-wrapper concern, while practice maturity prevents over-penalizing the category. |
| Levels Health | Flag — Pioneer with Over-claiming | 2.9 | 2.95 | +0.05 | Medium | Evidence-stage tagging clarifies the case as emerging CGM biofeedback with quantified-outcome / scope overreach, not fraud. |
| Eight Sleep | Caution — Pioneer with Over-claiming | 3.3 | 3.35 | +0.05 | Medium | SDT reinforces dashboard / AI-score dependency; evidence-stage remains plausible mechanism plus broad clinical-language overreach. |
| Superpower | Disqualify — Real Substrate, Unearned Certainty | 4.05 | 4.2 | +0.15 | Medium | Evidence-stage and business-health layers strengthen the existing disqualify pattern without introducing an MLM false positive. |
| Neuronic | Disqualify — Predatory Pre-RCT / Chronic-Illness Targeting | 4.0 | 4.25 | +0.25 | High | New SDT and evidence-stage distinctions sharpen the disqualifier: felt agency may be real, while the product thesis still performs unearned certainty around implausible delivery. |

**Acceptance check:** 7 / 7 anchors preserve canonical verdict tier. Both Disqualify anchors remain at or above 4.0, and no case moves more than one verdict tier. The only upward movement is intentional: v2.2 makes SDT, evidence-stage, and business-health risks more legible where they were previously implicit.

---

## MAPS PBC (Lykos Therapeutics) — primary positive anchor

**What it is:** MDMA-assisted psychotherapy for treatment-resistant PTSD. Phase 3 clinical-trial program. FDA rejection (2024) with transparent response.

**Why it anchors at Surface (~1.1):** 40-year arc of building evidence under sustained commercial and regulatory pressure. Marketing reports rather than promises. FDA rejection was disclosed publicly and integrated into ongoing work rather than spun.

**Active ingredients (OpenLabel framing):** MDMA-assisted psychotherapy protocol with trained therapists. Real, specific, replicable.

**Also contains:** therapeutic alliance, integration support, the legitimacy of a long institutional arc, the patient agency required by the protocol itself.

**Signal-catalog highlights:**
- `cultural-trend:patient-precedence` = 0 (patient communities led adoption — strong positive)
- `tactic-density-aggregate` ≤ 1 (essentially absent)
- No `inverse:*` signals fire
- `register:*` analysis: Autonomous-Construct-Aware founder register; Autonomous-Pluralist marketing; Expert-Autonomous business model — coherent across layers

**Tone calibration use:** when tempted to score across the board at 1, ask whether they have a 40-year arc of holding genuine uncertainty. If not, calibrate down. Score 1 is rare.

**Verdict:** Surface

---

## Virta Health — Score 1–2 anchor (clinical-evidence-based DTC)

**What it is:** Continuous remote care for type 2 diabetes via nutritional ketosis. Multiple peer-reviewed RCTs and cohort publications.

**Why it anchors high:** evidence-led from founding. Publishes outcomes regardless of result. Honest about subscription dependency for outcome maintenance — does not claim resolution-and-walk-away.

**Active ingredients:** nutritional ketosis intervention plus continuous coaching. Real mechanism, real delivery, real evidence.

**Also contains:** coaching relationship; identity shift around food and metabolic health; access to a clinical channel many people otherwise can't reach.

**Signal-catalog highlights:**
- D1 high (multiple PMID-resolvable RCTs)
- `conviviality:dependency-generation` low-medium (subscription required for outcome maintenance — but honestly stated)
- `tactic:future-faking` flagged at low severity (best-case outcome framing in some marketing) — does not elevate verdict

**Watch in OpenLabel framing:** D7b journey positioning is "Tool" not "Stepping stone" — appropriately scoped, not predatory.

**Verdict:** Surface

---

## HeartMath — Score 2–3 anchor (real mechanism + accumulated practice + inflated commercial framing)

**What it is:** HRV biofeedback training, breath-pacing tools, heart-coherence framework, related certifications.

**Why it anchors here:** the underlying HRV / RSA / coherence science is real; the practice has decades of refinement. The commercial framing extends from "useful biofeedback tool" toward proprietary-cosmology territory ("heart intelligence," coherence as generalized health driver) faster than evidence supports.

**Active ingredients:** paced breathing protocols. HRV biofeedback. Real, mostly accessible without the branded wrapper.

**Also contains:** identity (heart-as-source language), certification community, proprietary cosmological framing layered on accessible practices.

**Signal-catalog highlights:**
- `tactic:exclusivity-access-illusion` fires (proprietary cosmology around accessible practice)
- `consumer:cost-active-ingredient-asymmetric` — paced breathing and HRV awareness are accessible through public-domain practices, smartphone apps, basic biofeedback courses

**OpenLabel-native note:** strong Alternative Comparison signal. The active ingredient is freely accessible.

**Verdict:** Flag

---

## Levels Health — Score 3 anchor (real science, overstretched claims)

**What it is:** Continuous glucose monitoring for non-diabetic populations as a metabolic-health biofeedback tool. Subscription with hardware.

**Why it anchors at 3:** real metabolic science underneath. CGM as biofeedback for non-diabetics has emerging evidence. But "81% improved HbA1c, with many reversing prediabetes" is presented without accessible methodology — clinically weighty language without clinical-grade methods transparency. Optimization framing applied where clinical language would be more accurate.

**Active ingredients:** CGM hardware and glucose-variability awareness.

**Also contains:** optimization-culture identity, dashboard dependency, biohacker community membership.

**Signal-catalog highlights:**
- `quantified-outcome-without-methods` fires on the 81% / prediabetes-reversal claims
- `inverse:over-precise-statistics` fires
- `cultural-trend:*` — CGM-for-non-diabetics is an active commercial trend (4–6 range)
- `consumer:burden-incompatible-chronic-illness` does NOT fire (target population is biohacker self-optimizer, not chronic-illness — appropriate scope)

**Watch in OpenLabel framing:** D9c scope honesty is a 3 — offering is calibrated for biohacker self-optimizers but is sold as more universally applicable.

**Verdict:** Flag

---

## Eight Sleep — Score 3–4 anchor (real mechanism, broad claim-surfacing)

**What it is:** Active-cooling/heating mattress system with HRV / sleep / recovery tracking and AI interpretation. Premium hardware plus subscription.

**Why it anchors here:** bed-surface thermoregulation is a real and plausible mechanism with at least one peer-reviewed product-adjacent paper. But "clinically proven," "every night," and broad performance / recovery / cardiovascular language overstates what a small, company-funded, short-duration study can justify. Heavy testimonial / celebrity / influencer architecture. Subscription dependence with proprietary AI scoring layer.

**Active ingredients:** active mattress thermoregulation. Real and plausibly effective for comfort and sleep onset.

**Also contains:** optimization-culture identity, performance-tracking dependency, AI-interpretation black box, status / luxury signaling.

**Signal-catalog highlights:**
- `clinical-language-without-rct` fires
- `tactic:social-proof-without-methodology` fires
- `tactic:future-faking` fires
- `tactic:authority-without-verification` fires (celebrity / influencer architecture)
- `tactic:narrative-transportation` fires
- D4 conviviality at 4 (low autonomous health agency built; AI-interpretation black box concentrates power)
- `consumer:cost-active-ingredient-asymmetric` — comfort-grade benefit at premium price

**Verdict:** Caution. *Worth noting — the science is not fake; the product likely helps many people. The label should reflect that. The criticism is the gap between modest evidence and broad claim-surfacing.*

---

## Superpower — Disqualify anchor (real diagnostic substrate, inflated commercial story)

**What it is:** Concierge biomarker testing platform with broad disease-detection / longevity / biological-age framing. "100+ biomarkers," "early signs of 1,000+ conditions," member-outcome statistics, physician/influencer endorsement architecture.

**Why it anchors at Disqualify:** underlying components (laboratory testing, biomarker measurement, clinician interpretation) are real and have legitimate uses. But the product-level thesis — early detection, longevity optimization, biological-age management, root-cause clarity from broad biomarker panels — extends well beyond what the science supports. Active Lanham Act false-advertising suit on biomarker count and support model. Heavy rescue-narrative testimonial use, grievance-with-mainstream-medicine framing, optimization futurism.

**Active ingredients:** laboratory blood draws and biomarker panel results. Clinician interpretation.

**Also contains:** identity capture around longevity / optimization, rescue narrative replacing relationship with mainstream medicine, dashboard / score / "your biological age" psychological hooks, biomarker-anxiety-driven upsell pathways.

**Signal-catalog highlights:**
- `mechanism-overclaim` fires (broad detection claims from biomarker panels)
- `tactic:grief-frustration-mining` fires (anti-mainstream-medicine rescue narrative)
- `tactic:parasocial-influencer-endorsement` fires
- `tactic:reciprocity-negative-option` fires (subscription concierge with retention friction)
- `arbitrage:enforcement-history-present` fires (Lanham Act litigation)
- `cultural-trend:*` aggregate at 7+ (peak capitalization in the longevity / personalized-diagnostics category)

**OpenLabel-native note:** the case to point at when explaining why "real diagnostic substrate" is not enough. The label-format framing is especially clarifying here: the Active Ingredients line is "lab tests and clinician interpretation"; the Also Contains line carries the actual commercial story.

**Verdict:** Disqualify

---

## Neuronic (Neuradiant 1070 Helmet) — Disqualify anchor + chronic-illness-population stress test

**What it is:** $3,000 photobiomodulation helmet (LED-based) marketed for autonomic dysfunction in POTS / MCAS / hEDS. Single-testimonial-driven marketing. Wellness-classification regulatory positioning.

**Why it anchors at Disqualify:** PBM at therapeutic doses in controlled clinical settings has an emerging evidence base. The Henderson (2024) analysis demonstrates that low-power LED devices likely cannot deliver sufficient fluence through scalp and skull to brain tissue — the mechanism is plausible in principle, but **the product as sold cannot deliver it**. Single testimonial as primary evidence. Implicit disease-treatment claims through narrative while maintaining wellness classification. Targets a population with limited conventional options and high desperation.

**Active ingredients:** sub-therapeutic dose of red / near-IR LED exposure to the scalp. Mechanism does not plausibly reach the target tissue at this device's specifications.

**Also contains:** hope; permission to act on chronic illness; identity reinforcement around the cosmology of "autonomic dysfunction is the root cause and PBM is the answer"; community membership through the testimonial subject's narrative arc; ritual structure.

**Signal-catalog highlights:**
- `mechanism-overclaim` fires (delivery rating "cannot-deliver" per Henderson 2024)
- `tactic:testimonial-as-mechanism-proof` fires
- `tactic:cognitive-load-distress-state-targeting` fires (chronic-illness population targeting)
- `arbitrage:wellness-classification-disease-claims` fires (disqualifier)
- `consumer:cost-monetizes-desperation` fires
- `phenomenological-colonization` fires
- `register:exploitation-with-targeting` — high-confidence detection (Conformist regression targeting via Expert-language wrap)

**Tone calibration use — most important case in the library.** This is where OpenLabel's tone has to be perfect. The patient who paid $3,000 is real, exhausted, and may have actually felt better. **They did not become wrong about feeling better.** The audit names the gap between what the device can do (probably very little, mechanistically) and what the patient experienced (potentially real benefit through hope, ritual, agency restoration, expectancy). The label format makes this honest without being cruel.

This is also the case where the **D8 register exploitation framing** is most important internally and **most carefully translated externally**. Founder is sincere — Expert-Pluralist register, genuine belief. Marketing-business apparatus reaches the population at Conformist regression state. The pattern is what's predatory, not the founder's intent. Internal language captures this; external language describes the dynamic without character-assassinating the founder.

**Verdict:** Disqualify

---

## Industry context — what these anchors imply at population scale

The seven anchor cases represent points on a continuum. The **chronic illness corpus** (Neuronic-shaped offerings) is qualitatively different from the **biohacker corpus** (Levels-shaped offerings) and the **clinical-evidence corpus** (Virta- and MAPS-shaped offerings).

Aggregate research findings (`references/literature-review.md` §3) confirm the corpus-level patterns:

- **Long COVID predatory marketing** is now a documented public-health concern. Brennan et al. 2023 (BMJ) identified >200 distinct unproven Long COVID treatments marketed online with no RCT evidence, often $500–$5,000 OOP. Neuronic-shaped offerings are a sub-segment of this larger pattern.
- **Treatment-delay risk** for cancer-adjacent and serious-condition offerings is well-documented. Johnson et al. 2018 (JNCI) — patients using complementary medicine alone had 2.5x mortality vs. conventional treatment. Anchors in this category trigger `consumer:safety-treatment-delay-risk`.
- **Supplement label inaccuracy** is the industry baseline: 59% of herbal supplements contained DNA from plants not on labels (Newmaster 2013, BMC Medicine); only 21% of herbal supplements at major retailers contained DNA from plants on labels (NY AG 2015). Anchors in the supplement category should be calibrated against this base rate.
- **Third-party certification penetration is <1% of supplements** (USP-verified). Presence of S-301 (`third-party-certification-present`) is a strong positive signal precisely because it is rare.
- **Adulteration-prone categories** (sexual enhancement, weight loss, muscle building) account for 98% of pharma-adulteration cases (Tucker et al. 2018, JAMA Network Open). Anchors in these categories should auto-elevate `consumer:safety-undisclosed-contraindications` scrutiny.

These industry data points are not anchor cases themselves; they are the empirical backdrop against which anchor cases should be interpreted.

---

## Anchor-coverage gaps (calibration cases needed)

The seven cases above cover the first score ladder, but they are thin on regulated-device positives, women's health, chronic-illness programs, charismatic retreat ecosystems, direct-selling wellness, mainstream supplements, and DTC biomarker interpretation. The next expansion should add structurally informative anchors rather than merely famous ones.

### Anchor Expansion v1 candidates

| Candidate | Category | Calibration role | Expected posture |
|---|---|---|---|
| **Dexcom G7** | Regulated CGM / medical device | Highest-integrity device ceiling; distinguishes FDA-cleared indication from wellness repurposing. | Positive |
| **Oura Ring** | Consumer wearable | Mainstream wearable with real validation plus readiness / fertility / longevity claim-creep risk. | Grey-positive |
| **Apollo Neuro** | Stress / nervous-system wearable | Honest early-stage device; tests whether low-N evidence is penalized fairly when claims are hedged. | Grey-positive |
| **Natural Cycles** | Women's health / fertility app | Regulated high-stakes app; calibrates FDA posture, efficacy disclosure, and user-error boundary. | Positive |
| **Midi Health** | Menopause telehealth | Positive remote-care anchor for evidence-based women's health and hormone-care delivery. | Positive |
| **Equip** | Eating-disorder telehealth | Positive high-complexity clinical-care anchor; tests remote care in a serious condition. | Positive |
| **Primal Trust** | Chronic-illness nervous-system program | Grey-zone chronic-illness program with hope, community, somatic practice, and broad applicability claims. | Grey / Flag |
| **Gupta Program or DNRS** | Brain-retraining / limbic retraining | Older chronic-illness brain-retraining comparator; calibrates psychogenic-framing and medical-gaslighting risk. | Grey / Caution |
| **Joe Dispenza** | Meditation / retreat / spiritual healing | Charismatic retreat ecosystem; testimonial healing and neuroscience-flavored spirituality. | Caution / Negative |
| **Healy** | Frequency device / direct selling | Frequency-device negative stress test with weak mechanism, distributor channels, and regulatory scrutiny. | Negative |
| **doTERRA or Young Living** | Essential oils / MLM | Canonical wellness MLM and distributor disease-claim structure. | Negative |
| **AG1** | Influencer supplement | Podcast / influencer supplement grey zone: real ingredients, proprietary blends, broad implied benefits. | Grey / Caution |
| **Seed or Thorne** | Science-forward supplement | Positive supplement contrast against AG1 / MLM / Goop-style claim drift. | Grey-positive |
| **Function Health or InsideTracker** | DTC biomarkers / longevity | Biomarker interpretation, panel breadth, clinical utility, and data-rights risk. | Grey |
| **Tally Health** | Epigenetic age / longevity | Academic-authority paradox; real clocks, weak clinical actionability, longevity claim pressure. | Caution |
| **BetterHelp or Cerebral** | Digital mental health / telehealth | Platform-level failure mode where evidence-based ingredients can be degraded by acquisition, privacy, or prescribing incentives. | Caution / Negative |
| **Waking Up or Headspace** | Meditation / mental wellness | Positive non-clinical or evidence-generating mental wellness app; contrast with Dispenza. | Positive |
| **Enagic / Kangen Water or BEMER** | Wellness device / MLM | Expensive wellness-device MLM; calibrates device claims plus compensation structure. | Negative |

### First validation expansion set

For the Mode 1 claim-substantiation harness, expand from seven to twelve anchors:

1. MAPS PBC
2. Virta Health
3. Oura Ring
4. Apollo Neuro
5. Primal Trust
6. Joe Dispenza
7. Healy
8. doTERRA or Young Living
9. AG1
10. Function Health or InsideTracker
11. Natural Cycles
12. Neuronic

This set gives the first harness a balanced spread: clinical evidence ceiling, digital / device positives, honest early-stage, chronic-illness grey zone, charismatic healing, frequency-device negative, MLM negative, supplement influencer grey zone, biomarker interpretation, women's-health regulatory positive, and chronic-illness predation stress test.

When these audits run, promote selected candidates into full anchor entries below using the template.

---

## Anchor-entry template

For new anchor cases, use this structure:

```markdown
## [Offering Name] — [Verdict] anchor / [pattern type]

**What it is:** [1–2 sentences on category, mechanism, business model]

**Why it anchors here:** [1 paragraph]

**Active ingredients:** [OpenLabel-frame: what it actually does]

**Also contains:** [non-active layers being sold]

**Signal-catalog highlights:**
- [Notable signals firing with severity and reasoning]

**Industry context citation:** [if applicable, link to literature-review.md statistic]

**Tone calibration use:** [what this case teaches about how to phrase similar audits]

**Verdict:** [Surface / Flag / Caution / Disqualify]
**MI score:** [N.NN]
**Methodology version at canonical audit:** v[N.N.N]
```

---

*This is a living calibration document. Add entries after each completed audit. When a new entry stretches the framework or reveals a pattern the existing dimensions don't catch, flag it — that is the trigger for a methodology update.*
