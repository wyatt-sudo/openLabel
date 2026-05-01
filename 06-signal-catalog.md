# Signal Catalog

*The structured catalog of patterns OpenLabel detects. Loaded at Stage 5 (Pattern-match). Each signal is independently runnable, individually testable, and feeds explicit prescription label sections.*

*This document presents the catalog as readable markdown. The agent loads a YAML/JSON form for execution. Each signal entry follows a stable schema; new signals are additive — never renumber.*

---

## Catalog schema

Every signal entry has:

```yaml
id: <stable-snake-case-name>
title: <short human title>
description: <one paragraph>
detection_kind: regex | llm | composite
detection_spec:
  regex_patterns: [...]              # for regex / composite
  llm_prompt_id: <prompt name>       # for llm / composite
  inputs: [<extraction tables / queries needed>]
severity_levels:
  low: <when this is mildly present>
  medium: <when this is clearly present>
  high: <when this is structural>
  disqualify: <when this triggers automatic Disqualify, if applicable>
confidence_default: high | medium | low
dimension_tags: [D1, D3, ...]        # rollup view
label_section_feeds: [active-ingredients, also-contains, contraindications,
                      alternatives, warning-panel, verdict, ...]
literature_support:
  - <author year, journal>
notes: <implementation notes, edge cases>
```

---

## Signal categories

The catalog groups by purpose. Categories are not exclusive — a signal can have multiple `dimension_tags`.

1. **Claim-substantiation signals** — the gap between claim language and evidence tier
2. **Mechanism signals** — what the company says vs. what plausibly explains the benefit
3. **Manipulation tactics** — persuasion architecture (the original 10 + 7 additions from research)
4. **Inverse trust signals** — features that look trustworthy but predict the opposite
5. **Conviviality signals** — capacity erosion, dependency, iatrogenic potential
6. **Register signals** — Cook-Greuter four-layer pattern + mismatch detection
7. **Cultural-trend signals** — the four trend-layer sub-signals
8. **Regulatory-arbitrage signals** — wellness-vs-disease classification gaming
9. **Scope-honesty signals** — universal claims for population-specific offerings
10. **Consumer-specific signals** — safety, cost-asymmetry, burden, data rights, alternatives, fit

---

# 1. Claim-substantiation signals

## `substantiation-gap`

**Description:** A claim's language tier exceeds the substantiation tier of its supporting evidence. (FTC core failure mode.)

- **Detection:** Composite. Per-claim record + claim_evidence record from Stage 4. If claim is risk-category C/D and substantiation_tier is 4–6, fire.
- **Severity:** medium = one C/D claim with weak support; high = multiple; disqualify = central product thesis rests on uncited or contradicted evidence.
- **Dimension tags:** D1, D3
- **Label section feeds:** active-ingredients, realistic-dose, verdict
- **Literature:** FTC Health Products Compliance Guidance (2022); Hastak & Mazis 2011

## `quantified-outcome-without-methods`

**Description:** Precise percentages or numeric outcome claims without visible methodology.

- **Detection:** Regex (`\d+\s*%|reduces?|increases?|improves?|reverses?` near outcome words) + claim_evidence absence.
- **Severity:** medium per occurrence; high if multiple.
- **Dimension tags:** D1, D3
- **Literature:** FTC Advertising Substantiation Policy Statement (1984); Schindler & Yalch 2006 (over-precise stats)

## `clinical-language-without-rct`

**Description:** "Clinically proven" / "validated" / "studies show" without retrievable RCT.

- **Detection:** Regex on trigger phrases + Stage-4 PubMed search returning no matching RCT.
- **Severity:** medium = one occurrence; high = multiple; disqualify = central thesis.
- **Dimension tags:** D1, D3
- **Literature:** FTC enforcement record; Hoy & Andrews 2004

## `mechanism-to-outcome-leap`

**Description:** Plausible mechanism cited as if it proves the marketed outcome.

- **Detection:** LLM-strong. Pattern: company invokes pathway language (HRV, nervous system, inflammation) in proximity to outcome claim without product-specific causal evidence.
- **Severity:** medium / high
- **Dimension tags:** D1, D2, D10

## `subgroup-rescue`

**Description:** Subgroup finding highlighted after null topline; or treatment-only data without control comparison.

- **Detection:** LLM-strong against cited studies after Stage 4 fetch.
- **Severity:** medium / high
- **Dimension tags:** D1

---

# 2. Mechanism signals

## `phenomenological-colonization`

**Description:** Real patient experience converted into proof of a specific (often proprietary) mechanism, when the benefit is more plausibly produced by non-specific factors (alliance, expectancy, ritual, community).

- **Detection:** LLM-strong. Composite: testimonial flagged-as-proof-of-mechanism + mechanism statement with low plausibility/delivery + offering category typically active in non-specific factors.
- **Severity:** medium = single occurrence; high = systematic pattern; disqualify = central product thesis is a false-mechanism story for a real-but-non-specific benefit.
- **Dimension tags:** D7, D10
- **Label section feeds:** also-contains, realistic-dose
- **Literature:** Kaptchuk 2003; Freeman 2015; placebo / non-specific factors meta-research

## `mechanism-overclaim`

**Description:** Stated mechanism is speculative, contradicted by independent science, or undeliverable at the product's specifications.

- **Detection:** LLM-strong on mechanism_statements after first-principles check.
- **Severity:** medium / high / disqualify
- **Dimension tags:** D2, D10

## `practice-maturity-mismatch`

**Description:** Approach is presented as established / proven when it is recently introduced and has not accumulated distributed clinical refinement.

- **Detection:** LLM-strong + cultural-trend signals.
- **Severity:** medium / high
- **Dimension tags:** D10

---

# 3. Manipulation tactics

The original 10 plus 7 additions from research. Tactic density (count of present tactics) is itself a high-signal aggregate; see `tactic-density-aggregate` at the end of this section.

## `tactic:urgency-scarcity`

**Description:** Artificial time pressure or limited availability to bypass deliberate evaluation.

- **Detection:** Regex (countdown timers, "limited time", "ends Friday", "only N left") + LLM-fast for nuanced cases.
- **Severity:** present / absent (binary; counted in density)
- **Literature:** Cialdini 1984; Aggarwal et al. 2011; FTC v. Health Formulas 2014

## `tactic:social-proof-without-methodology`

**Description:** Quantity claims that imply consensus without verifiability.

- **Detection:** Regex (`\d{3,}\+? (?:customers|users|members|patients)`) + LLM-fast on outcome statistics without methodology.
- **Literature:** Cialdini 2001; Amos et al. 2008

## `tactic:authority-without-verification`

**Description:** Credentials or prestige cues that cannot be checked or do not match the actual claim.

- **Detection:** Composite — team_member.domain_relevance = decorative + persuasion_features.credentials_display + Stage 4 institution-verification result.
- **Literature:** Eastin 2001; Kata 2010; Pornpitakpan 2004

## `tactic:future-faking`

**Description:** Promises transformation as expected outcome, not best case.

- **Detection:** LLM-fast on claim_text + testimonial.suggests_typical_outcome.
- **Literature:** Tiggemann & Slater 2014; FTC weight-loss enforcement record

## `tactic:fear-escalation`

**Description:** Health anxiety amplified beyond what evidence warrants.

- **Detection:** LLM-fast on page text — catastrophizing, "if you don't address this now," worst-case framing.
- **Literature:** Witte & Allen 2000; Tannenbaum et al. 2015

## `tactic:narrative-transportation`

**Description:** Story arc itself used as persuasion technology — extended emotional arcs, identification cues, resolution beats.

- **Detection:** LLM-strong (mechanism, not tactic-marker).
- **Literature:** Green & Brock 2000; Murphy et al. 2013; Shen et al. 2015. (Validated as mechanism; tactic-specific deceptive-marketing evidence is moderate.)

## `tactic:grief-frustration-mining`

**Description:** Anti-mainstream-medicine framing as proof; mainstream dismissal harvested into allegiance.

- **Detection:** LLM-fast.
- **Literature:** Kata 2012; Bratich 2008

## `tactic:exclusivity-access-illusion`

**Description:** This specific framework / platform / community is the only real path to the benefit.

- **Detection:** LLM-fast + Alternative-Comparison consumer-specific layer cross-check.
- **Literature:** Cialdini scarcity; Lynn 1991

## `tactic:testimonial-as-mechanism-proof`

**Description:** Individual outcome implies a specific causal mechanism, not just that some people report benefit.

- **Detection:** testimonial.asserts_mechanism boolean (LLM-fast at extraction).
- **Literature:** Freeman 2015; Kaptchuk 2003. **Most-cited tactic in FTC supplement enforcement.**

## `tactic:cognitive-load-distress-state-targeting`

**Description:** Marketing calibrated to crisis regression states rather than the customer's fuller operating capacity.

(Renamed from "register exploitation" — same construct, mainstream framing.)

- **Detection:** LLM-strong. Composite of target population (chronic illness, mental health crisis), tactic density, fear escalation, register-mismatch detection.
- **Literature:** Mani et al. 2013 (scarcity cognitive load); Brehm 1966 (reactance); Shiv & Fedorikhin 1999 (stress + impulsive choice); Baumeister 1998 (ego depletion)

## `tactic:reciprocity-negative-option`  **NEW**

**Description:** Free-trial-to-subscription traps; "free" entry that auto-enrolls into recurring billing; cancellation friction.

- **Detection:** Composite — pricing_record.free_trial_with_subscription + cancellation_friction != "easy" + persuasion_features.money_back_guarantee + LLM-fast on pricing-page language.
- **Severity:** medium / high; **disqualify** if combined with vulnerable-population targeting and high cancellation friction.
- **Dimension tags:** D3, D5, commercial incentives
- **Literature:** Cialdini 1984 (reciprocity); FTC ROSCA enforcement record; Mathur et al. 2019 (dark patterns). **Top consumer-complaint category per FTC.**

## `tactic:pseudo-scientific-jargon`  **NEW**

**Description:** Density of scientific-sounding language without retrievable citations; "sciency" graphics (molecular structures, brain images) used to increase persuasion absent supporting evidence.

- **Detection:** Composite — high jargon-token density (regex over scientific-vocabulary list) ÷ low cited_evidence count + LLM-fast on graphic/diagram analysis.
- **Severity:** medium / high
- **Dimension tags:** D1, D3
- **Literature:** Weisberg et al. 2008 ("seductive allure of neuroscience explanations"); Tal & Wansink 2014; Fernandez-Duque 2015

## `tactic:parasocial-influencer-endorsement`  **NEW**

**Description:** Influencer relationship leveraged for credibility transfer in absence of expertise; identity-based endorsement.

- **Detection:** LLM-strong on cited endorsements + Stage 4 verification of expertise + disclosure check.
- **Severity:** medium / high
- **Dimension tags:** D3, D5
- **Literature:** Hoffner 1996; Chung & Cho 2017; Pilgrim & Bohnet-Joschko 2019 (Instagram health influencers)

## `tactic:sludge-dark-patterns`  **NEW**

**Description:** Information overload, confusion, drip pricing, forced action, friction asymmetry between sign-up and cancellation.

- **Detection:** Composite — pricing_record.cancellation_friction + persuasion_features + extracted disclosures pattern.
- **Severity:** medium / high
- **Dimension tags:** D3, D5
- **Literature:** Thaler 2018; Mathur et al. 2019 (CSCW dark-pattern taxonomy)

## `tactic:decoy-pricing-anchoring`  **NEW**

**Description:** Three-tier pricing where the middle tier is structured to make a higher tier look reasonable; price anchoring; quality-via-price signaling absent quality data.

- **Detection:** pricing_record.decoy_pricing_present + LLM-fast on tier-structure analysis.
- **Severity:** medium
- **Dimension tags:** D3, commercial incentives, cost-to-value
- **Literature:** Tversky & Kahneman 1974; Huber et al. 1982; Wansink et al. 1998; Rao & Monroe 1989

## `tactic:false-dichotomy`  **NEW**

**Description:** "Conventional medicine OR our approach" framing; either-or peripheral-cue persuasion.

- **Detection:** LLM-fast on page text.
- **Severity:** medium
- **Dimension tags:** D3, D7
- **Literature:** Petty & Cacioppo 1986 (ELM); Lewandowsky et al. 2012

## `tactic:loss-aversion-sunk-cost`  **NEW**

**Description:** Activation of loss aversion or sunk-cost framing — especially common in MLM-wellness and protocol-based programs ("don't give up now after all the work you've done").

- **Detection:** LLM-fast on extracted page text + program-structure context.
- **Severity:** medium / high (MLM context elevates)
- **Dimension tags:** D3, D4 (conviviality), commercial incentives
- **Literature:** Kahneman & Tversky 1979; Bosley & McKeage 2015 (MLM); Taylor 2012 (FTC MLM report)

## `tactic-density-aggregate`

**Description:** Count of present tactics across the catalog. Density itself is a high-information-density signal — Mathur et al. 2019 shows sites combining 3+ dark patterns have significantly higher consumer-complaint rates.

- **Detection:** Composite. Run after all individual tactic detections.
- **Thresholds:**
  - 0–2 tactics → Normal
  - 3–4 tactics → Yellow Flag
  - 5–7 tactics → Red Flag
  - 8+ tactics → Predatory Apparatus → **automatic Disqualify**
- **Dimension tags:** D3
- **Label section feeds:** warning-panel, verdict
- **Literature:** Mathur et al. 2019; Bratich & Banet-Weiser 2019 (MLM-wellness — density is the structural feature)

---

# 4. Inverse trust signals

These look trustworthy but empirically predict the opposite. Detecting them is part of OpenLabel's distinctive value — most consumer tools take these signals at face value.

## `inverse:doctor-recommended-without-source`  **NEW**

**Description:** "Doctor recommended" / "#1 recommended by" without verifiable source; survey base rates unverifiable.

- **Detection:** Regex + Stage 4 source verification.
- **Severity:** medium / high
- **Dimension tags:** D3, D5
- **Literature:** Andrews, Netemeyer & Burton 1998; FTC Listerine, Airborne enforcement record

## `inverse:fda-registered-as-approval`  **NEW**

**Description:** "FDA-registered" or "GMP facility" framed as if it were FDA approval; ~39% of consumers misinterpret.

- **Detection:** Regex (`FDA[- ](?:registered|listed|approved)`) + LLM-fast for context evaluation + Stage 4 actual FDA classification check.
- **Severity:** high (because directly misleading)
- **Dimension tags:** D5
- **Literature:** Schwartz, Woloshin et al. 2009 (Annals of Internal Medicine 150:8 — consumer FDA literacy)

## `inverse:over-precise-statistics`  **NEW**

**Description:** Suspiciously precise efficacy stats ("87.3% saw results") — precision boosts credibility but, in marketing claims, correlates with internal/uncontrolled studies.

- **Detection:** Regex on claim_text matching `\d{1,3}\.\d` followed by outcome word + claim_evidence.substantiation_tier check.
- **Severity:** medium
- **Dimension tags:** D1, D3
- **Literature:** Schindler & Yalch 2006

## `inverse:credential-stack-decorative`  **NEW**

**Description:** Excessive credentialing — multiple letters, niche board certifications — used as authority halo where domain relevance is weak.

- **Detection:** team_member.domain_relevance = decorative across multiple members.
- **Severity:** medium
- **Dimension tags:** D3, D6
- **Literature:** Cialdini authority heuristic; Pornpitakpan 2004

## `inverse:celebrity-physician-halo`  **NEW**

**Description:** "As seen on Dr. Oz" / featured-by-physician-celebrity used as evidentiary substitute.

- **Detection:** Regex on persuasion_features.as_seen_on + LLM-fast on celebrity-physician identification.
- **Severity:** high (strong inverse signal)
- **Dimension tags:** D3, D5, D6
- **Literature:** Korownyk et al. 2014 (BMJ 349:g7346 — only 33–46% of Dr. Oz recommendations had believable supporting evidence; ~15% contradicted evidence)

## `inverse:institutional-logo-halo`  **NEW**

**Description:** University crests, hospital partnerships, "research collaboration with X" — when relationship is sponsorship rather than endorsement.

- **Detection:** persuasion_features.institutional_logo + Stage 4 institutional-verification.
- **Severity:** medium / high
- **Dimension tags:** D3, D6
- **Literature:** Simonin & Ruth 1998 (brand alliances); Roe, Levy & Derby 1999

## `inverse:disclosure-paradox`  **NEW**

**Description:** Disclosed conflicts (sponsorships, affiliate relationships, study funding) being followed by *more* persuasion / pressure rather than less — the moral-licensing effect. Disclosure does not signal trustworthy seller; the literature is unambiguous.

- **Detection:** Composite — disclosure present + tactic-density still high or rising in same content.
- **Severity:** medium
- **Dimension tags:** D3, D6
- **Literature:** Loewenstein, Cain & Sah 2011 (American Economic Review 101:3); Cain, Loewenstein & Moore 2005

---

# 5. Conviviality signals

The seven sub-patterns from D4. Each is its own catalog entry now.

## `conviviality:capacity-substitution`
Tool substitutes for the user's own capacity, leaving them more dependent. Detected by LLM-strong on use-pattern + dependency analysis. Dimension: D4.

## `conviviality:dependency-generation`
Business model depends on ongoing subscription / repeat purchase / escalating engagement. Detected via pricing_record + program structure. Dimension: D4, commercial incentives.

## `conviviality:power-concentration`
Interpretive / decision-making power concentrated in proprietary algorithms / expert systems users cannot inspect. Detected via app/biomarker extension fields. Dimension: D4.

## `conviviality:access-barrier`
Pricing creates two-tier access; exploits the most desperate. Detected via pricing_record + target-population. Dimension: D4, cost-to-value.

## `conviviality:life-reorganization`
Tool requires life to reorganize around the tool — fragments rather than supports holistic living. Dimension: D4.

## `conviviality:iatrogenic-risk`
Health anxiety amplification, somatic hypervigilance, illness identity reinforcement, nocebo from tracking, displacement of effective lower-tech interventions. Dimension: D4. **Especially load-bearing in chronic-illness population audits.**

## `conviviality:community-erosion`
Privatizes health intelligence that could be held collectively; redirects energy from structural / environmental / relational root causes. Dimension: D4.

---

# 6. Register signals (Cook-Greuter four-layer)

## `register:founder-marketing-mismatch`
Higher-register founder writing lower-register marketing — likely deliberate stage exploitation. Dimension: D8.

## `register:pluralist-marketing-conformist-model`
"Holistic, whole-person, community" marketing with ongoing-dependency subscription model. Most common form of register dishonesty in integrative wellness. Dimension: D8.

## `register:company-customer-state-mismatch`
Expert/Achiever-register company marketing to customers in Conformist regression. Cross-reference `tactic:cognitive-load-distress-state-targeting`. Dimension: D8.

## `register:exploitation-with-targeting`  **DISQUALIFY-CAPABLE**
Deliberate targeting of regression state with evidence of intent. Composite signal — requires multiple supporting detections. Dimension: D8. **Threshold disqualifier when supporting evidence is high-confidence.**

---

# 7. Cultural-trend signals

The four trend-layer sub-signals as individual catalog entries.

**Anchor calibration v2.2:** MAPS and Virta are the negative controls: long evidence arcs, low saturation pressure, and patient / clinical precedence before major commercialization. HeartMath, Levels, and Eight Sleep are active-trend controls where the signal should elevate scrutiny without collapsing into Disqualify. Superpower is the peak-capitalization control: trend score should be high enough to color the whole audit because longevity / personalized-diagnostics hype expands the product thesis. Neuronic is the chronic-illness trend control: patient-community precedence may be present, but it does not offset disease-claim arbitrage, weak product delivery, or desperation targeting.

## `cultural-trend:commercial-velocity`
Compare PubMed earliest-pub-date for core mechanism vs. company founding date. 0–3 score. Dimension: cultural-trend.

## `cultural-trend:market-saturation`
New brands launched in same category in last 24 months. 0–2 score. Dimension: cultural-trend.

## `cultural-trend:claim-expansion`
Wayback Machine comparison of historical vs. current claim posture. 0–2 score. Dimension: cultural-trend.

## `cultural-trend:patient-precedence`
Patient-community discussion pre-dating commercial activity. 0–3 score (positive when patient communities led). Dimension: cultural-trend.

---

# 8. Regulatory-arbitrage signals

## `arbitrage:wellness-classification-disease-claims`
Product classified as general wellness but making implicit disease-treatment claims through testimonial or cosmology. **Disqualify-capable** when central to product thesis. Dimension: D5, D3.

## `arbitrage:supplement-as-pharmaceutical`
Supplement classification with claim language that approaches pharmaceutical territory (treats / cures / mitigates / prevents disease). Dimension: D5, D1.

## `arbitrage:loophole-dependence`
Business model survives only under current regulatory ambiguity; would not survive tighter enforcement. Dimension: D5, loophole-dependence.

## `arbitrage:enforcement-history-present`
FDA warning letters / FTC enforcement / Lanham Act complaints in company history. **High-severity** signal. Dimension: D5.

---

# 9. Scope-honesty signals

## `scope:universal-claims-population-specific-offering`
Universal efficacy claims for an offering whose evidence is population-specific. Dimension: D9c, D3.

## `scope:contraindications-absent`
Vulnerable populations who should not use this aren't named. Dimension: D9c, safety.

## `scope:exclusivity-as-default`
Alternatives framed as incomplete; exit costs because practices bundled with belonging. Dimension: D9c, exclusivity-tactic.

## `scope:complementary-care-discouraged`
Conventional or complementary care actively discouraged. **Disqualify-capable** in serious-condition contexts. Dimension: D9c, safety.

---

# 10. Consumer-specific signals

## `consumer:safety-undisclosed-contraindications`
Direct harms or population-specific risks not disclosed. Especially MCAS triggers, PEM risk, dysautonomia escalation, drug interactions. Dimension: safety.

## `consumer:safety-treatment-delay-risk`
Offering positioned to substitute for conventional care where conventional care has clear benefit. Dimension: safety.

## `consumer:cost-active-ingredient-asymmetric`
Active ingredient available elsewhere at substantially lower cost; company does not acknowledge. Cross-references `tactic:exclusivity-access-illusion`. Dimension: cost-to-value.

## `consumer:cost-monetizes-desperation`
Pricing scales with severity of customer condition; predatory pricing of desperation. Dimension: cost-to-value, conviviality.

## `consumer:burden-incompatible-chronic-illness`
Time / cognitive / lifestyle burden incompatible with chronic-illness energy budgets. Dimension: practical-burden.

## `consumer:data-portability-absent`
Data collected by offering is not exportable in standard formats; user has no portable record. Dimension: data-rights.

## `consumer:data-third-party-undisclosed`
Data shared with third parties (advertising, training-AI, research partners) without prominent disclosure. Dimension: data-rights.

## `consumer:alternatives-suppressed`
Cheaper / better-evidenced / public-domain alternatives not acknowledged. Dimension: alternatives, exclusivity.

## `consumer:fit-misaligned`
Offering's actual best-fit population is narrow but marketing addresses everyone. Cross-references `scope:universal-claims-population-specific-offering`. Dimension: consumer-fit, D9c.

---

# 11. Business due-diligence signals

Consolidated cluster of business-side signals that materially affect consumer outcomes. These were thinly distributed across other signals in v1; the cluster makes them legible. **Surface in Mode 2 full audit and Mode 3 company portal as a "Business Health" panel.** Do not enter Tier-A composite directly but can trigger threshold disqualifiers.

**Anchor calibration v2.2:** Treat this cluster as a consumer-continuity and incentive-risk layer, not as a generic startup-quality score. MAPS and Virta may carry ordinary financing / regulatory risk without consumer exploitation. Levels, Eight Sleep, and Superpower should surface investor-pressure, subscription-continuity, pricing, privacy, and exit-pathway questions. Neuronic demonstrates that small-company business risk can become consumer harm when high price, thin evidence, and chronic-illness targeting converge. HeartMath primarily calibrates unit-economics asymmetry and certification / community economics rather than venture-capital pressure.

## `business:capital-runway-risk`

Estimated runway is short relative to the customer relationship the offering implies. Subscription / device / coaching offerings depend on the company's continued existence; if runway is tight, the consumer carries that risk.
- **Detection:** Composite — total raised (Crunchbase), stage, last-round date, bridge-round signals, hiring/layoff news, SEC filings if public.
- **Severity:** medium / high; **disqualify** when survival is in genuine question for an offering requiring multi-year continuity.
- **Tags:** business, D5
- **Citations:** SEC EDGAR; Crunchbase / PitchBook patterns

## `business:investor-pressure-misalignment`

Investor base is generalist / growth-stage / late-stage in a category requiring evidence-pace patience. Creates pressure for extraction faster than evidence supports.
- **Detection:** LLM-strong on investor list cross-referenced with portfolio composition + stage at last round + implied valuation pressure.
- **Severity:** medium / high
- **Tags:** business, D6
- **Citations:** standard health-business diligence frameworks (capital-position assessment)

## `business:exit-pathway-volatility`

Likely acquirer landscape implies post-acquisition product changes (privacy posture, pricing, scope, data-policy reversal). Especially relevant for biomarker / app / data-rich offerings.
- **Detection:** LLM-strong on plausible acquirers + acquirer-class historical product-evolution patterns.
- **Severity:** medium / high
- **Tags:** business, L4 (data rights)

## `business:founder-execution-track-record`

Distinct from credentials. Has the founding team built and shipped operational businesses before? First-time founder vs. repeat operator, with prior outcomes assessed.
- **Detection:** LLM-strong on founder LinkedIn + Crunchbase company history + court-records check.
- **Severity:** low (informational) → medium when track record is mismatched to offering complexity.
- **Tags:** business, team-credibility
- **Citations:** standard team-and-execution diligence frameworks

## `business:conflict-of-interest-network`

Founder also runs / owns / advises related companies — especially horizontal integration (founder's other company makes the supplement they recommend; founder's clinic prescribes the device they sell; founder's nonprofit funds the trial they cite).
- **Detection:** Composite — founder's other companies (LinkedIn, Crunchbase, state business registrations), trademark assignee overlap (USPTO TESS), cited research authorship overlap with company leadership.
- **Severity:** medium → high; **disqualify** when undisclosed and material to the product thesis.
- **Tags:** business, D6, D5
- **Label section feeds:** business-health-panel, also-contains (when conflict shapes what's being sold)

## `business:unit-economics-asymmetry`

Pricing relative to delivered value patterns. Inferred from price + claimed-active-ingredient cost elsewhere + LTV signals (subscription term, churn signals from app reviews) + CAC signals (paid-acquisition heaviness, influencer-spend patterns).
- **Detection:** Composite of pricing_record + alternative-cost lookup + Tier 4 review patterns.
- **Severity:** medium / high
- **Tags:** business, L2 (cost-to-value)
- **Citations:** standard unit-economics diligence frameworks

## `business:smart-money-exit-signal`

Insider selling, VC partial exits, founder share-sale patterns, executive churn — those closest to the company are reducing exposure.
- **Detection:** SEC filings (insider transactions), Crunchbase exit / secondary records, LinkedIn tenure-trend analysis.
- **Severity:** medium / high
- **Tags:** business

## `business:capital-structure-distress`

Bridge rounds, down rounds, structured debt in startup context, unusual financing terms — survival-mode signals.
- **Detection:** Crunchbase financing-round details, SEC filings, news reports.
- **Severity:** medium → high → **disqualify** (survival in question).
- **Tags:** business, D5

---

# 12. MLM compensation-structure cluster

MLM wellness has a recognizable structural signature distinct from generic dependency marketing. Worth its own cluster because the consumer is being recruited as much as sold to.

**Anchor calibration v2.2:** None of the current seven anchors are MLM-positive. They therefore serve as negative controls: subscription, certification, influencer, ambassador, or community language must not trigger this cluster unless compensation depends on recruiting / downline economics or distributor inventory incentives. HeartMath can resemble MLM at the edge because of certification community dynamics, but should not fire `mlm:*` absent hierarchical compensation. Superpower / Eight Sleep influencer programs should stay in tactic and business-health signals unless distributor recruitment is present. Neuronic remains a chronic-illness exploitation anchor, not an MLM anchor.

## `mlm:hierarchical-compensation`

Compensation depends on recruiting others into the same selling structure (downline), not just selling product.
- **Detection:** Regex on "compensation plan", "downline", "upline", "team commission", "sponsoring", "consultant"/"ambassador"/"advocate" + LLM-fast on plan-document analysis.
- **Severity:** medium informational; high when combined with other MLM signals.
- **Tags:** mlm, business, D3
- **Citations:** Taylor 2012 (FTC report on MLM); Bosley & McKeage 2015

## `mlm:income-disclosure-shows-low-earning`

Required FTC income-disclosure statements typically show >90% of distributors earn negligible or negative income.
- **Detection:** Regex for "income disclosure"/"earnings statement" + LLM-fast on percentage / median-income content.
- **Severity:** medium / high (more aggressively recruitment-driven → higher).
- **Tags:** mlm, business
- **Citations:** FTC MLM income-disclosure rules

## `mlm:inventory-loading-pattern`

Distributors required or strongly incentivized to purchase inventory beyond personal use to qualify for compensation.
- **Detection:** LLM-strong on compensation-plan language, "minimum order" / "PV" / "personal volume" terminology.
- **Severity:** high
- **Tags:** mlm, D3, D5
- **Citations:** Taylor 2012 FTC MLM report

## `mlm:retention-via-events-and-identity`

Retention through retreat / convention / weekly-zoom culture-building rather than product value. Identity capture through "X consultant" / "founding member" / branded community membership.
- **Detection:** LLM-fast on community / event language, public conference visibility.
- **Severity:** medium
- **Tags:** mlm, D4, D8 (identity dynamics)
- **Citations:** Bratich & Banet-Weiser 2019 (Int. J. Communication)

## `mlm:wellness-MLM-composite`

Composite — combines hierarchical compensation + wellness-category products + retention-via-events + heavy testimonial use + identity-as-distributor framing.
- **Detection:** Composite over the four signals above.
- **Severity:** high; **disqualify** when chronic-illness-population-targeted MLM (predatory pattern documented in Long COVID, MCAS, Lyme communities).
- **Tags:** mlm, D3, D4, D8
- **Label section feeds:** warning-panel, business-health-panel, also-contains
- **Citations:** Bratich & Banet-Weiser 2019; Taylor 2012; Bosley & McKeage 2015

---

# 13. Self-Determination Theory layer (Conviviality complement)

Empirically-grounded complement to D4 Conviviality, especially load-bearing for chronic-illness contexts where SDT has decades of RCT evidence on outcomes. Three needs (autonomy, competence, relatedness) parallel D4 sub-patterns with stronger empirical foundation.

**Anchor calibration v2.2:** MAPS and Virta are positive controls for autonomy-supportive, competence-building intervention design, even where expert guidance is substantial. HeartMath should flag mild competence / relatedness displacement when proprietary cosmology or certification identity becomes the product. Levels and Eight Sleep calibrate dashboard dependency: SDT signals should rise when interpretation migrates from user learning to proprietary scoring. Superpower calibrates autonomy erosion through rescue-narrative biomarker interpretation. Neuronic calibrates the high-severity chronic-illness case: hope, community, and ritual may restore agency phenomenologically, while the commercial thesis can still displace autonomous care-seeking.

## `sdt:autonomy-violation`

Offering subverts or replaces user agency in their health decisions. Distinct from `conviviality:capacity-substitution` — focused specifically on the agency-respect dimension.
- **Detection:** LLM-strong on protocol / decision-architecture / interpretive-control language.
- **Severity:** medium / high
- **Tags:** D4, sdt
- **Citations:** Deci & Ryan 1985, 2000 (Psychological Inquiry 11:4); Ryan & Deci 2017

## `sdt:competence-erosion`

Offering displaces or undermines user's growing competence rather than building it.
- **Detection:** LLM-strong on offering's relationship to user competence development over time.
- **Severity:** medium / high
- **Tags:** D4, sdt
- **Citations:** Ryan & Deci 2017; Sheldon, Ryan & Reis 1996

## `sdt:relatedness-displacement`

Offering replaces real relational connection with parasocial / company-mediated / transactional relationships. Especially common in chronic-illness offerings that promise community but deliver gatekept identity.
- **Detection:** LLM-strong + cross-reference with `tactic:parasocial-influencer-endorsement`.
- **Severity:** medium / high
- **Tags:** D4, sdt, D7
- **Citations:** Baumeister & Leary 1995; Ryan & Deci 2017

---

# 14. Beauchamp & Childress Four Principles — ethics overlay

Triggered for **serious-condition offerings**: cancer-adjacent, severe mental health, infectious disease, severe chronic illness, anaphylaxis-risk, suicidality-relevant. Adds an explicit medical-ethics frame.

## `ethics:autonomy-violation`

Informed consent absent, subverted, or vitiated by manipulation tactics.
- **Detection:** Composite — informed-consent language + claim-substantiation gaps + manipulation-tactic density.
- **Severity:** high in serious-condition contexts
- **Tags:** ethics, D5, safety

## `ethics:beneficence-failure`

Offering does not meaningfully benefit (no real active ingredient and no honest non-specific factors).
- **Detection:** Composite of D1 + D2 + patient-signal divergence + alternatives-comparison.
- **Severity:** medium / high
- **Tags:** ethics, D1, D2

## `ethics:maleficence-risk`

Net harm risk — direct, opportunity, or treatment-delay harms exceed benefits.
- **Detection:** Composite — `consumer:safety-*` signals + treatment-delay risk + cost / opportunity-cost analysis.
- **Severity:** high → **disqualify** in serious-condition contexts.
- **Tags:** ethics, safety
- **Citations:** Johnson et al. 2018 (JNCI) — alternative-only cancer treatment correlates with 2.5x mortality

## `ethics:justice-violation`

Inequitable access; predatory pricing of vulnerable populations; offering structurally unavailable to those it claims to help.
- **Detection:** Composite — `consumer:cost-monetizes-desperation` + access-barrier signals + population-fit analysis.
- **Severity:** medium / high
- **Tags:** ethics, D4, L2

**Overlay citation:** Beauchamp & Childress 1979/2019 (*Principles of Biomedical Ethics*, 8th ed.); standard medical-ethics framework adopted across clinical practice and bioethics.

---

# 15. Jobs-to-be-Done — D9 enhancement

Christensen Jobs-to-be-Done as a structured supplement to D9 Actual Function. Same question with more business-actionable scaffolding. **Analytical layer, not new signals.**

For each audit, populate (LLM-strong):

- **Functional job:** what is the user hiring this offering to do?
- **Emotional job:** what feeling-state is the user hiring this to produce or relieve?
- **Social job:** what identity / belonging is the user hiring this to perform?
- **Fired:** what is being displaced by this hire? (Existing routine, alternative product, professional support, peer community.)
- **Progress measure:** how does the user know it's working? (Honest answer often differs from claimed answer.)
- **Competing solutions:** what else hires for this same job? (Free, public-domain, lower-cost, clinical alternatives.)

Populates the prescription label's Active Ingredients (functional), Also Contains (emotional + social), Realistic Dose (progress), Alternatives (competing solutions).

**Citation:** Christensen, Hall, Dillon, Duncan 2016 (*Competing Against Luck*); Christensen 1997, 2003 prior development.

---

# 16. Health Belief Model — Mode 1 lens

Rosenstock's Health Belief Model is the canonical framework for consumer health decision-making. Use as analytical lens on Mode 1 — what consumer perception is the marketing manufacturing? **Analytical layer, not new signals.**

For Mode 1 rapid triage, evaluate:

- **Susceptibility manipulation:** is perceived risk manufactured or amplified beyond actual?
- **Severity manipulation:** is perceived severity inflated?
- **Benefits inflation:** are perceived benefits exceeding evidence?
- **Barriers minimization:** are real barriers (cost, time, side effects, treatment-delay risk) downplayed?
- **Cues to action:** artificial (countdown timers, "limited spots") vs. natural (symptom onset, doctor referral)?
- **Self-efficacy:** is the offering replacing user self-efficacy or building it?

Composition input for the Mode 1 surface — translates to consumer-readable framing such as "this marketing manufactures a sense that you are at higher risk than you actually are."

**Citations:** Rosenstock 1966 (*Milbank Memorial Fund Quarterly*); Janz & Becker 1984 (*Health Education Quarterly*).

---

# 17. Cialdini cross-reference

Map our 17 manipulation tactics to Cialdini's 7 principles. Industry-standard vocabulary for B2B / academic / press audiences. **Don't replace the catalog — provide vocabulary overlay.**

| OpenLabel tactic | Cialdini principle(s) |
|---|---|
| `tactic:urgency-scarcity` | **Scarcity** |
| `tactic:social-proof-without-methodology` | **Social Proof** |
| `tactic:authority-without-verification` | **Authority** |
| `tactic:future-faking` | **Commitment / Consistency** (committing to future-state-self) |
| `tactic:fear-escalation` | (extension of Scarcity / Loss Aversion — not a separate Cialdini principle) |
| `tactic:narrative-transportation` | **Liking** + **Unity** |
| `tactic:grief-frustration-mining` | **Unity** (us-vs-them solidarity) |
| `tactic:exclusivity-access-illusion` | **Scarcity** + **Liking** |
| `tactic:testimonial-as-mechanism-proof` | **Social Proof** + **Liking** |
| `tactic:cognitive-load-distress-state-targeting` | (no Cialdini equivalent — cognitive-load research) |
| `tactic:reciprocity-negative-option` | **Reciprocity** |
| `tactic:pseudo-scientific-jargon` | **Authority** (sciency-as-authority) |
| `tactic:parasocial-influencer-endorsement` | **Liking** + **Authority** + **Unity** |
| `tactic:sludge-dark-patterns` | (extension; relates to Commitment / Consistency through forced action) |
| `tactic:decoy-pricing-anchoring` | (extension; behavioral economics, not Cialdini per se) |
| `tactic:false-dichotomy` | (extension; ELM peripheral cues, not Cialdini per se) |
| `tactic:loss-aversion-sunk-cost` | **Commitment / Consistency** |

**Citation:** Cialdini 1984/2007 (*Influence*); Cialdini 2016 (*Pre-Suasion*) for the seventh "Unity" principle.

---

# 18. Alternative developmental lenses (Cook-Greuter complements)

Cook-Greuter remains primary for the founder-marketing-model-customer four-layer analysis (D8). Other developmental frameworks are deployed selectively:

- **Spiral Dynamics (Beck/Cowan from Graves):** more legible to business audiences. Same general territory as Cook-Greuter; different vocabulary (vMemes Beige → Purple → Red → Blue → Orange → Green → Yellow → Turquoise). **Use for** Mode 3 company portal where the audience would find Cook-Greuter alien.

- **Kegan's Orders of Consciousness:** stronger empirical grounding for adult development; subject-object framing more applicable to professional / institutional contexts. **Use for** assessing how an organization (vs. an individual founder) holds complexity — especially for late-stage companies with many decision-makers.

- **Capabilities Approach (Sen, Nussbaum):** for the question "what does this offering enable people to do or be?" — particularly relevant to D4 Conviviality and consumer-fit. **Use for** chronic-illness or disability-context audits where restored capability is load-bearing.

Default: Cook-Greuter for the consumer-side analysis. Layer alternatives in Mode 3 or in academic / advocacy publications when alternate vocabulary serves the audience.

---

# 19. Evidence-stage epistemic integrity

A core OpenLabel commitment: **thin evidence is not the same as misleading.** Many new offerings will inherently lack rigorous research — that makes them *more risky*, not ineffective or dishonest. The harm is in performing certainty unearned, not in being early. This signal cluster catches the difference.

This cluster is the load-bearing apparatus for OpenLabel's "grey-area excellence" — generic LLMs handle established-evidence and obviously-fraudulent cases well; OpenLabel's distinctive value is in the middle, where these signals do the work.

**Anchor calibration v2.2:** MAPS and Virta anchor "Established Evidence." HeartMath anchors the traditional / practice-maturity middle: some real practice and distributed refinement, but commercial cosmology can outpace what the evidence supports. Levels and Eight Sleep anchor "Pioneer with overclaim risk": plausible mechanisms and early evidence, but quantified / clinical language must be audited tightly. Superpower anchors "real substrate, unearned certainty": legitimate diagnostics do not justify broad longevity / disease-detection claims. Neuronic anchors the disqualifying inverse: thin evidence plus performed certainty plus implausible product delivery plus chronic-illness targeting.

## `evidence-stage:honest-pilot-framing`

**Description:** Offering acknowledges its pre-RCT evidence stage with appropriate language. Phrases like "early evidence," "preliminary research," "we're learning," "this is what we know so far," "for early adopters who accept the uncertainty." Crucially: the company does not claim a maturity it has not earned.

- **Detection:** LLM-strong on company's overall epistemic-stance language across science page, FAQ, marketing surface.
- **Effect:** **Positive signal.** When present alongside thin evidence, downgrades severity of `claim-substantiation-gap` and shifts the verdict toward "Pioneer" rather than "Unsubstantiated."
- **Tags:** D1, D6, D7c, evidence-stage
- **Citations:** No specific lit; reflects calibrated-uncertainty principle (`09-tone-and-stance.md`)

## `evidence-stage:trajectory-active`

**Description:** Company is actively building its evidence: registered clinical trials (ClinicalTrials.gov), published pilots, observational data sharing, registry studies, pre-registered protocols, IRB-approved research. Visible signal that the company treats its own evidence stage as something to be improved, not preserved.

- **Detection:** Stage 4 ClinicalTrials.gov + PubMed search on company-authored / company-funded research; LLM-strong on science-page commitment language.
- **Effect:** **Positive signal.** Strong indicator of D6 epistemic culture. Materially distinguishes "Pioneer with integrity" from "Pre-RCT performing certainty."
- **Tags:** D6, evidence-stage

## `evidence-stage:trajectory-stalled`

**Description:** Long-running offering (>3 years on market) with no visible evidence-building activity — same testimonial-driven marketing surface for years, no trial registrations, no published research, no observational reporting. Time has passed without epistemic progress.

- **Detection:** Composite — first-launch date (Wayback / founding date) + Stage 4 PubMed/ClinicalTrials check + Wayback comparison of historical science-page content.
- **Severity:** medium → high. **Worse than missing evidence in a new offering** — this is missing evidence with time to have built it.
- **Tags:** D6, evidence-stage

## `evidence-stage:traditional-lineage-faithful`

**Description:** Practice draws on documented historical / contemplative / somatic tradition with distributed clinical refinement (yoga, certain herbal traditions, meditation lineages, breathwork, traditional medical systems). Transmission is faithful to source teaching; teachers are properly credentialed within the tradition; the offering does not extract a brand-differentiating shell from the practice.

- **Detection:** LLM-strong on tradition-attribution language + lineage-verification (where possible) + comparison of taught content to documented tradition.
- **Effect:** **Positive signal.** Distributed clinical refinement is its own form of evidence — different from RCT but not weaker. Especially load-bearing for mind-body, contemplative, somatic, and integrative-medicine offerings.
- **Tags:** D2, D7c, D10b (practice maturity), evidence-stage

## `evidence-stage:traditional-lineage-extracted`

**Description:** Borrowing the legitimacy of an established tradition (yoga, traditional medicine, meditation, indigenous healing) while making claims beyond what the tradition itself supports — typically by isolating a single technique, applying it to a population not in the tradition's scope, or making efficacy claims the tradition itself doesn't make.

- **Detection:** Composite — lineage-attribution language + comparison to documented tradition's scope and claims.
- **Severity:** medium / high
- **Tags:** D7c, D10b, evidence-stage

## `evidence-stage:RCT-design-mismatch`

**Description:** Category where RCT design itself is poorly suited as the dominant evidence form: contemplative practice, narrative interventions, complex psychotherapy modalities, body-based / somatic interventions, group / community / ritual-mediated practices. For these, distributed clinical refinement, single-case methodology, and qualitative research are more epistemically appropriate.

- **Detection:** LLM-strong on category classification.
- **Effect:** **Calibration signal.** When present, downgrades the weight of "missing RCT" in claim-substantiation evaluation; upgrades the weight of practice-maturity (D10b) and traditional-lineage signals.
- **Tags:** D1 (calibration), D10b, evidence-stage
- **Citations:** Reference to single-case methodology (Krasny-Pacini & Evans 2018); qualitative-evidence frameworks (Lincoln & Guba 1985); Cochrane Qualitative and Implementation Methods Group

## `evidence-stage:reasonable-pilot-user-fit` (composite)

**Description:** Composite positive signal — the offering is reasonable for an early-adopter / pilot user who explicitly accepts the uncertainty:
- Honest pilot framing present (`evidence-stage:honest-pilot-framing`)
- Mechanism plausible (D2 ≤ 3) and product can deliver it
- Active evidence trajectory (`evidence-stage:trajectory-active`) OR traditional-lineage-faithful
- Safety profile is reasonable (no `consumer:safety-treatment-delay-risk` for serious conditions; no contraindications hidden)
- Population fit is appropriate and acknowledged (D9c ≤ 3)

- **Detection:** Composite over the listed inputs.
- **Effect:** **Strong positive signal.** Enables a "Surface — Pioneer" or "Flag — Pioneer with caveats" verdict for thin-evidence offerings that are operating with epistemic integrity. Materially shifts how the audit reads.
- **Tags:** D1, D6, D7, D9c, evidence-stage, verdict-tag
- **Notes:** This is the signal that prevents OpenLabel from becoming a tool that unfairly punishes new / underrepresented / underfunded categories. Especially load-bearing for chronic-illness-population offerings where conventional research has failed to engage.

## `evidence-stage:certainty-performed-without-earning`

**Description:** Offering performs certainty its evidence stage doesn't support. Sells "the answer," "the breakthrough," "what conventional medicine missed" while sitting at evidence tier 4–6.

- **Detection:** Composite — certainty language (`tactic:future-faking`, `tactic:exclusivity-access-illusion`) + low substantiation tier + absence of `evidence-stage:honest-pilot-framing`.
- **Severity:** medium → high → **disqualify** when central to product thesis.
- **Tags:** D1, D3, D6, D7, evidence-stage
- **Notes:** This is the inverse of `reasonable-pilot-user-fit`. The harm here is not thin evidence — it is the dishonesty about being at thin evidence.

---

The catalog evolves. Versioning rules:

- **Patch bump** (1.0.x): adding a literature citation, refining detection regex, fixing a label_section_feeds entry.
- **Minor bump** (1.x.0): adding a new signal, deprecating one (without removal), adjusting severity thresholds.
- **Major bump** (x.0.0): renaming a signal id, removing a signal, changing the rollup-dimension mapping, changing what triggers disqualifiers.

`audit_runs.signal_catalog_version` records which version a given audit was run under. Cross-version aggregation requires explicit normalization.

Anchor cases re-run under each new version flag drift — see `11-aggregate-research-and-analytics.md`.

---

## Adding a new signal

1. Observe the pattern in 3+ audits.
2. Draft the catalog entry with detection rule, severity levels, dimension tags, label-section feeds, and literature support.
3. Run the new signal in shadow mode (computed but not contributing to verdict) on the anchor library + last 50 audits.
4. Review false-positive / false-negative rates.
5. Calibrate thresholds.
6. Promote to active. Bump catalog version.
7. Document in `references/literature-review.md` if literature-backed.

---

*This catalog is the load-bearing artifact for Stage 5. Treat it as data, not as prose. New entries are append-only; existing entries are stable. Detection details may evolve within an entry; the entry's identity (id) does not.*
