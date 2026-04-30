# Literature Review

*Synthesis of academic and applied research informing OpenLabel's methodology. Drawn from research conducted April 30, 2026 across four research vectors: trust signals & disclaimers, manipulation tactics, industry composition / harm / enforcement, and existing aggregation efforts.*

*This document compiles citations and key findings. Per-finding implications for the catalog are summarized; for full integration, see `06-signal-catalog.md` (signals informed by these findings) and `07-evaluation-rules.md` (literature-backed predictiveness data).*

*Citation note: where pulled from prior recall, page numbers should be re-verified before any external publication use.*

---

## §1 — Trust signals and the disclaimer paradox

### The half-remembered disclaimer-trust finding

The original intuition that "companies with disclaimers tend to be trustworthy" appears to **conflate three distinct literatures** rather than reference one finding:

1. **Hastak & Mazis (2011)** — *Journal of Public Policy & Marketing* 30(2): 157–167. Content analysis showing FDA/FTC-style mandated disclaimers reduce *misleading inferences* but consumers often fail to integrate them. Methodology: review of 28 FTC/FDA cases plus consumer-perception experiments.

2. **Hoy & Andrews (2004)** — *Journal of Advertising* 33(2): 25–37. Experimental study (n≈400) finding disclaimers in DTC drug ads reduce risk-claim deception **only when proximity/prominence is high**; weak disclaimers backfire by producing false reassurance.

3. **Loewenstein, Cain & Sah (2011)** — *American Economic Review* 101(3): 423–428. The **paradoxical disclosure** effect: RCT-grade evidence that disclosure can *increase* persuasion via moral licensing (advisors disclosed-as-biased become more biased) and strategic exaggeration (advisees discount insufficiently). **Cain, Loewenstein & Moore (2005)** — *Journal of Legal Studies* 34(1): 1–25 — established this experimentally first.

### Implication for OpenLabel

There is **no robust empirical finding** that "companies with disclaimers have more trustworthy products." The literature actually supports the opposite under specific conditions: disclosure without paired reduction in pressure may indicate moral-licensed manipulation.

This converts to **inverse trust signal `inverse:disclosure-paradox`** (`06-signal-catalog.md` §4) — disclosure presence + continued tactic density flags moral-licensing pattern.

Disclosure *quality* (proximity, prominence, comprehensibility per FTC `.com Disclosures` 2013 guidance) remains a positive signal — but quality, not presence.

---

### Empirically-supported positive trust signals

Ranked by strength of evidence:

| Rank | Signal | Citation | Evidence type |
|---|---|---|---|
| 1 | Specific, retrievable citations to peer-reviewed studies | Eastin 2001 (J. Computer-Mediated Communication 6:4); Metzger 2007 (JASIST 58:13: 2078–2091) | Observational + experimental |
| 2 | Independent third-party certification (USP, NSF, ConsumerLab, Informed-Sport) | ConsumerLab Annual Reports; Cohen et al. 2014, 2018 (JAMA / Drug Test Anal) | Observational, large-N product testing |
| 3 | FTC-compliant disclosure proximity / prominence | Hoy & Andrews 2004; Hoy & Lwin 2007 | Experimental |
| 4 | Named, verifiable advisory board with institutional affiliations | Pornpitakpan 2004 (J. Applied Social Psychology 34:2: 243–281) meta-analysis | Meta-analysis |
| 5 | Founder credentials when domain-matched and verifiable | Hovland & Weiss 1951; Pornpitakpan 2004 meta | Foundational + meta |
| 6 | Regulatory classification accuracy ("FDA-cleared" vs "FDA-approved" vs "FDA-registered") | Schwartz, Woloshin et al. 2009 (Annals of Internal Medicine 150:8: 516–527) | Experimental, consumer-survey |
| 7 | Price as moderate quality signal in opaque categories | Rao & Monroe 1989 (JMR 26:3: 351–357) | Meta-analysis (r ≈ .26; weak) |
| 8 | Disclosed test methodology and lot-level COAs | Cohen 2014 (JAMA) emerging | Observational |

Implementation: see `06-signal-catalog.md` signals S-301 through S-305.

---

### Empirically-supported inverse trust signals

Features that look trustworthy but predict the opposite:

| Rank | Signal | Citation | Evidence type |
|---|---|---|---|
| 1 | "Doctor recommended" / "#1 recommended by" without source | Andrews, Netemeyer & Burton 1998 (J. Marketing 62:4: 62–75); FTC enforcement record (Listerine, Airborne) | Observational + regulatory |
| 2 | "FDA-registered" / "GMP facility" framed as approval (~39% of consumers misinterpret) | Schwartz & Woloshin 2009, 2011 | Experimental |
| 3 | Scientific-jargon density without retrievable citations ("seductive allure of neuroscience") | Weisberg et al. 2008 (J. Cognitive Neuroscience 20:3: 470–477); Fernandez-Duque et al. 2015 replication | RCT |
| 4 | Over-precise statistics ("87.3% saw results") | Schindler & Yalch 2006 (Adv. Consumer Research 33) | Experimental |
| 5 | Excessive credentialing stacks (multiple letters, niche board certifications, non-domain-matched) | Cialdini authority heuristic 1984/2007; Pornpitakpan 2004 — domain-match matters more than prestige | Theoretical / meta |
| 6 | Celebrity-physician / "as seen on Dr. Oz" halo | **Korownyk et al. 2014 (BMJ 349:g7346) — content analysis: only 33–46% of Dr. Oz recommendations had believable supporting evidence; ~15% contradicted evidence** | Content analysis |
| 7 | Institutional logo / hospital-partnership halo (when relationship is sponsorship not endorsement) | Simonin & Ruth 1998 (JMR 35:1: 30–42); Roe, Levy & Derby 1999 (JPP&M 18:1: 89–105) | Experimental |

Implementation: signals S-201 through S-207.

The **Korownyk 2014 BMJ finding** is particularly load-bearing for OpenLabel — if half of celebrity-physician recommendations lack believable evidence, then `inverse:celebrity-physician-halo` is a high-severity signal in its own right.

---

### Halo effect

**Thorndike (1920)** original; in health: **Andrews, Burton & Netemeyer 2000** (J. Advertising 29:3) — health-claim halo onto unrelated nutrient attributes. **Roe, Levy & Derby 1999** (JPP&M 18:1: 89–105) — FDA-approved health claims caused consumers to over-infer benefits on unmentioned dimensions. Institutional logos (university crests, hospital partnerships) transfer credibility even when the relationship is sponsorship rather than endorsement.

---

### Open questions / thin literature

- Whether **density of trust signals** is itself diagnostic (theoretical signal-jamming models; minimal direct empirical work in DTC health).
- Interaction between **supplement-specific COA publication** and adulteration — Cohen's lab is the main source; needs replication.
- **Founder-credential verification** at scale — almost no empirical work distinguishing "verifiable PhD/MD" from "credentialed-in-adjacent-field."
- Whether disclaimers signal *seller* trustworthiness — essentially uninvestigated; strongest related work (Loewenstein) suggests the opposite.

---

## §2 — Manipulation / persuasion tactics in health marketing

The original 10-tactic checklist was validated against the academic literature. Ten findings:

### Validation of original 10 tactics

| Tactic | Strength of evidence | Anchor citations |
|---|---|---|
| Urgency / scarcity manufacturing | **Strong** | Cialdini 1984/2007; Aggarwal et al. 2011 (J. Advertising); FTC v. Health Formulas 2014 |
| Social proof without methodology | **Strong** | Cialdini 2001; Amos et al. 2008 (Int. J. Advertising) meta; FTC Endorsement Guides |
| Authority without verification | **Strong** | Eastin 2001 (Health Communication); Kata 2010 (Vaccine); Pornpitakpan 2004 meta |
| Future faking / transformation | **Strong** | Tiggemann & Slater 2014 (Body Image); FTC "establishment claims" doctrine; FTC Red Flag report 2014 |
| Fear escalation | **Very strong** | Witte & Allen 2000 (Health Education & Behavior) meta; Tannenbaum et al. 2015 (Psychological Bulletin) — fear works but ethically fraught when paired with low-efficacy products |
| Narrative transportation | **Moderate-strong** | Green & Brock 2000 (JPSP) foundational; Murphy et al. 2013; Moyer-Gusé 2008 (Communication Theory) — direct evidence in deceptive health marketing is moderate |
| Grief / frustration mining (anti-mainstream framing) | **Moderate** | Kata 2012 (Vaccine); Bratich 2008 — less quantified than 1–5 |
| Exclusivity / access illusion | **Strong** | Cialdini scarcity; Lynn 1991 (Psychology & Marketing) |
| Testimonial as proof of mechanism | **Very strong — top FTC enforcement target** | Freeman et al. 2015 (Lancet); Kaptchuk 2003 |
| Register exploitation in distress states | **Weakly supported as named — Cook-Greuter is non-standard in marketing** | Reframed via mainstream literature (next section) |

### Tactic #10 reframe

The "register exploitation in distress states" item from the original Cook-Greuter ego-development application has stronger mainstream parallel literature:

- **Mani et al. 2013** (*Science*) on scarcity / poverty cognitive load
- **Brehm 1966** reactance theory
- **Baumeister 1998** ego depletion
- **Shiv & Fedorikhin 1999** (J. Consumer Research) on stress and impulsive choice
- **Dhar & Gorlin 2013** emotional regulation and consumption

**Recommendation implemented:** rename to `tactic:cognitive-load-distress-state-targeting` for the public catalog. Cook-Greuter framing preserved internally for the dimension D8 view.

### New tactics added from research

Tactics from the persuasion / influence / dark-patterns / health-marketing literature **not in the original list**, now added to the catalog:

#### A. Reciprocity / negative-option (free-trial-to-subscription)

Cialdini 1984 (reciprocity); FTC ROSCA enforcement (e.g., AgeForce 2018); Mathur et al. 2019 (CSCW) dark patterns. **Top FTC consumer-complaint category.**

Catalog signal: `tactic:reciprocity-negative-option`.

#### B. Pseudo-scientific jargon ("seductive allure")

Weisberg et al. 2008 (J. Cognitive Neuroscience) "seductive allure of neuroscience explanations"; Tal & Wansink 2014 (Public Understanding of Science) — graphs and molecular structures increase belief in unrelated claims; Fernandez-Duque et al. 2015 replication.

Catalog signal: `tactic:pseudo-scientific-jargon`.

#### C. Identity-based / parasocial influencer endorsement

Hoffner 1996; Chung & Cho 2017 (Psychology & Marketing) on parasocial trust; Pilgrim & Bohnet-Joschko 2019 (BMC Public Health) on Instagram health influencers.

Catalog signal: `tactic:parasocial-influencer-endorsement`.

#### D. Confusion / sludge / dark patterns

Thaler 2018 (*Science*); Mathur et al. 2019 (CSCW dark-pattern taxonomy); OFT/CMA 2010 drip-pricing report.

Catalog signal: `tactic:sludge-dark-patterns`.

#### E. Decoy pricing / anchoring

Tversky & Kahneman 1974; Huber et al. 1982 (J. Consumer Research) decoy effect; Wansink et al. 1998 (JMR) on supplement / portion anchoring; Rao & Monroe 1989 on price-as-quality heuristic.

Catalog signal: `tactic:decoy-pricing-anchoring`.

#### F. False dichotomy framing

Petty & Cacioppo 1986 (Elaboration Likelihood Model) peripheral cues; Lewandowsky et al. 2012 (Psychological Science in the Public Interest) on misinformation structure.

Catalog signal: `tactic:false-dichotomy`.

#### G. Loss aversion / sunk-cost activation (especially MLM-wellness)

Kahneman & Tversky 1979 (loss aversion); Bosley & McKeage 2015 (J. Consumer Affairs) on MLM persuasion; Taylor 2012 (FTC report on MLM).

Catalog signal: `tactic:loss-aversion-sunk-cost`.

### Tactic density evidence

Density is **empirically validated as predictive**:

- **Mathur et al. 2019** (CSCW) — shopping sites combining 3+ dark patterns had significantly higher consumer-complaint rates.
- **Bratich & Banet-Weiser 2019** (Int. J. Communication) on MLM wellness — density is the *structural feature*, not individual tactics.
- **Ernst & Pittler 2006** (J. Royal Soc. Medicine) on supplement marketing — tactic-stacking correlates with absent clinical evidence.
- Vladeck (former FTC) — enforcement actions cluster around "stacked" deception.

This validates the OpenLabel methodology choice that tactic density of 8+ triggers automatic Disqualify.

### Tactics most predictive of FTC enforcement / consumer harm

Ranked by FTC case frequency + harm evidence:

1. **Testimonial as mechanism** — most-cited tactic in FTC supplement enforcement
2. **Future faking / establishment claims** — weight-loss, anti-aging cases
3. **Authority without verification** — fake doctors, fake studies
4. **Fear escalation × low-efficacy product** — Tannenbaum 2015
5. **Reciprocity / negative-option** — top consumer-complaint category, FTC ROSCA enforcement (was missing from original list)
6. **Social proof without methodology**

Implementation: `07-evaluation-rules.md` "Empirical predictiveness" — these tactics get higher severity ceilings.

### Narrative transportation — caveat

Green & Brock 2000 is foundational. Health-specific evidence: Murphy et al. 2013; Shen et al. 2015 (J. Health Communication) meta — narratives reduce counter-arguing in health contexts. **Direct evidence in deceptive wellness marketing is thinner than for fear/testimonial.** Catalog implementation: keep the signal but acknowledge it as mechanism rather than tactic-marker.

---

## §3 — Industry composition, harm patterns, regulatory enforcement

### Supplement label accuracy and contamination

| Finding | Citation |
|---|---|
| **59% of herbal supplements contained DNA from plants not on labels; 33% had outright substitution; only 2 of 12 companies had products without substitution, contamination, or fillers** | Newmaster et al. 2013 (*BMC Medicine*) DNA-barcoding |
| Only 21% of herbal supplements at GNC, Target, Walmart, Walgreens contained DNA from plants on labels | NY AG investigation 2015 (Schneiderman press release) |
| **746 dietary supplements adulterated with unapproved pharmaceuticals from 2007–2016; only 48% led to voluntary recalls.** Sexual enhancement (45.5%), weight loss (40.9%), muscle building (11.9%) accounted for ~98% | Tucker et al. 2018 (*JAMA Network Open*) |
| ~1 in 5 supplements tested annually fails quality testing | ConsumerLab Annual Reports 2019–2023 |
| 33–50% of probiotic products contain different or fewer strains than labeled | Patro-Gołąb & Szajewska 2019 (*Nutrients*); Marcobal et al. 2008 (*J Clin Gastroenterol*); Morovic et al. 2016 (*J AOAC Int*) |
| SARMs: only 52% contained labeled SARM; 39% contained unapproved drugs; 9% had no active ingredient | Van Wagoner et al. 2017 (*JAMA*) |
| USP-verified represents <1% of supplements on market; NSF Certified for Sport covers ~700 of >90,000 SKUs | USP 2022; CRN 2023 |

### FTC / FDA enforcement statistics

| Finding | Citation |
|---|---|
| ~150 health/wellness deceptive-marketing FTC cases 2010–2023; **average settlement $3–15M; individual cases reach hundreds of millions (Teami $15.2M, NeuroMetrix Quell $4M)** | FTC Cases & Proceedings database |
| **AMG Capital v. FTC (2021) gutted FTC monetary restitution under §13(b)** — major reduction in recovery capability | SCOTUS |
| ~50–100 FDA warning letters/year to dietary supplement firms | FDA Inspections Database |
| 23–25% of supplement firms inspected have GMP violations | FDA 2022 21 CFR 111 inspection reports |
| **67% of FDA-recalled supplements still contained banned drugs 6+ months later when retested** | Cohen 2014 (*JAMA*) |
| **<1% of likely violative products receive any enforcement action** | Cohen 2018 (*Drug Test Anal*) |

The AMG Capital ruling and the <1% enforcement rate together describe a regulatory environment where consumer-facing self-defense tools (like OpenLabel) carry disproportionate weight.

### Consumer harm patterns

| Finding | Citation |
|---|---|
| **~23,000 ER visits per year attributable to dietary supplements; ~2,150 hospitalizations annually**. Weight-loss and energy products caused 72% of cardiovascular AEs | Geller et al. 2015 (*NEJM*) |
| **Herbal/dietary supplements caused 20% of US drug-induced liver injury cases by 2014, up from 7% in 2004** | Navarro et al. 2017 (*Hepatology*); DILIN |
| **Cancer patients using complementary medicine alone had 2.5x mortality vs. conventional treatment** — delay or refusal of conventional care drove mortality | Johnson et al. 2018 (*JNCI*) |
| Class I device recalls increased ~97% from 2003–2012 | Ardaugh et al. 2013 (*NEJM*) |

### Wellness market size (2023–2026)

| Sector | Size | Citation |
|---|---|---|
| Global wellness economy | $6.3T (2023), projected $9T by 2028 | Global Wellness Institute 2024 Monitor |
| US dietary supplements | $59.4B (2023), projected ~$75B by 2027 | Nutrition Business Journal 2024 |
| Personalized wellness / biomarker testing | $26B (2024), 18% CAGR | Grand View Research 2024 |
| Mental wellness apps | $7.5B (2024), 15% CAGR | Statista 2024 |
| Longevity / anti-aging | $27B (2024), projected $44B by 2030 | Precedence Research 2024 |
| Peptide therapeutics consumer | ~$4B and growing >12%/yr | Fortune Business Insights 2024 |

Post-pandemic supplement-sales jump: 14.5% in 2020 vs. ~5% prior baseline (CRN Consumer Survey 2021).

### Vulnerable-population targeting

| Finding | Citation |
|---|---|
| **>200 distinct unproven Long COVID treatments marketed online; often $500–$5,000 OOP** | Brennan et al. 2023 (*BMJ*) |
| Documented ME/CFS predation history (decades of supplements, "rife machines", Lyme-adjacent) | Tuller 2015–2023 (Virology Blog); Geraghty 2017 (*J Health Psychol*) "desperation marketing" |
| MCAS / POTS predation: Dysautonomia International + The Mast Cell Disease Society warnings | Patient organization documentation |
| Cancer "desperation": **66% of advanced cancer patients use CAM with marketing language explicitly targeting prognosis anxiety** | Davis et al. 2012 (*Support Care Cancer*) |

### Influencer / RCT funding asymmetry

**Influencer ad spend in wellness reached $5.2B in 2023** (Influencer Marketing Hub 2024) — vs. **<$50M** in published RCT funding for the same categories (estimate from NIH RePORTER 2023).

This 100x asymmetry is one of the structural features OpenLabel aggregate analytics should track over time.

---

## §4 — Existing aggregation efforts and the OpenLabel white space

### Profiled efforts

#### ConsumerLab.com
- **What:** Independent lab testing of ~1,000+ supplement products across ~200+ categories
- **Output:** Per-product pass/fail; long-form category reviews; "ConsumerLab Approved" seal
- **Business model:** Subscription (~$50/yr) + paid certification program for manufacturers
- **Gap for OpenLabel:** tests *physical product only*, not marketing claims; subscription paywall limits public-good function; no industry-wide reporting

#### Labdoor
- **What:** Independent lab testing on ~1,500 supplements; 5-pillar score (label accuracy, purity, nutritional value, ingredient safety, projected efficacy); 0–100 composite
- **Output:** Free public rankings; affiliate-linked product pages
- **Business model:** Affiliate revenue + B2B "Labdoor for Brands" certification (acquired by NSF 2021; activity has slowed)
- **Failure mode:** affiliate revenue conflicted with audit independence

#### Examine.com
- **What:** Human-curated systematic synthesis of peer-reviewed studies on ~400+ supplements/interventions and ~600+ health outcomes; per-supplement-per-outcome evidence grade
- **Output:** Supplement pages; Examine Database
- **Business model:** Paid subscription (~$29/mo or ~$228/yr)
- **Gap for OpenLabel:** evaluates *ingredients*, not *products* and not *brands*; no marketing-claim audit; no industry trend lens

#### NCCIH (NIH National Center for Complementary and Integrative Health)
- **What:** Government-funded research summaries; consumer fact sheets ("Herbs at a Glance")
- **Business model:** Federally funded
- **Gap:** slow publication cadence; no product-level or brand-level data; no marketing-integrity work

#### Cochrane Reviews
- **What:** Systematic reviews following PRISMA/GRADE methodology
- **Business model:** Nonprofit, member organizations, grants
- **Lesson for OpenLabel:** methodology transparency — pre-registered protocols, conflict-of-interest declarations, reproducibility — is the model OpenLabel should emulate

#### Information Is Beautiful "Snake Oil?"
- **What:** Visual aggregation of supplement evidence (one-shot static)
- **Lesson for OpenLabel:** visual format is consumer-grade, but one-shot doesn't compound

#### Industry analysts (McKinsey, Statista, Grand View, etc.)
- **What:** Market sizing, category growth, consumer-segment data
- **Gap:** treats wellness as a *market*, not a *trust ecosystem*

#### Other candidates with no incumbent doing the work
- AI-health entrants 2024–2026: Yuka (food/cosmetics scoring), Perplexity Health, Consensus.app — aggregate evidence but not products' marketing integrity
- Patient orgs (NORD, Solve M.E., Dysautonomia International, MEpedia, Phoenix Rising) — aggregate disease-specific lived experience but not product-level integrity audits

### OpenLabel white space — confirmed

**A. Marketing-integrity & claim-substantiation aggregation: nobody is doing this at scale.** ConsumerLab/Labdoor test physical product. Examine evaluates ingredient evidence. NCCIH/Cochrane review studies. SBM critiques anecdotally. Industry analysts size markets. No one aggregates: claim-vs-evidence delta, manipulation tactics, influencer-disclosure compliance, FTC-substantiation gaps, cross-category claim inflation.

**B. Cross-product industry trend reporting on integrity (not market size): nobody.** GlobalData reports tell you mushroom adaptogens grew 34% YoY. Nobody tells you "mushroom adaptogen claims inflated 60% YoY" or "menopause category has the highest rate of unsupported hormonal claims" or "perimenopausal women aged 38–52 are the most aggressively targeted underserved population."

**C. Failure modes to avoid:**
- Labdoor stalled because affiliate revenue conflicted with audit independence
- ConsumerLab never crossed into mainstream because of paywall + dry presentation
- Snake Oil infographic was beloved but static — one-shot visualizations don't compound
- NCCIH shows that government-pace publishing can't keep up with TikTok-pace marketing

### Structural lessons OpenLabel should adopt

1. **Methodology transparency (Cochrane):** publish the audit rubric, version it, declare conflicts, make scoring reproducible
2. **Consumer-grade UX (Yuka, Snake Oil):** prescription-label format aligns with this — instantly legible scoring beats PDF reports
3. **Free-with-aggregate-paid (Examine + analyst hybrid):** individual audits free for trust and SEO; industry research instrument paid for B2B (regulators, journalists, retailers, investors)
4. **Avoid affiliate revenue on audited products** — Labdoor's lesson; poisons independence
5. **Compounding data asset:** every consumer audit feeds the industry research instrument
6. **Time-series from day one:** claim inflation, manipulation-tactic prevalence, category gold-rushes only become visible longitudinally — log everything timestamped

---

## §5 — Key statistics for OpenLabel public communication

Vetted, citable statistics to use in methodology pages, press, and the State of the Wellness Industry report:

1. **59% of herbal supplements contained unlisted plant species** — Newmaster et al. 2013 (BMC Medicine)
2. **746 supplements found adulterated with pharmaceutical drugs 2007–2016; only 48% recalled** — Tucker et al. 2018 (JAMA Network Open)
3. **23,000 ER visits per year in the US attributable to supplements** — Geller et al. 2015 (NEJM)
4. **Herbal/dietary supplements cause 20% of US drug-induced liver injury, up from 7% in 2004** — Navarro et al. 2017 (Hepatology)
5. **Cancer patients using alternative-only treatment had 2.5x mortality vs. conventional care** — Johnson et al. 2018 (JNCI)
6. **Global wellness economy reached $6.3T in 2023, projected $9T by 2028** — Global Wellness Institute 2024
7. **67% of FDA-recalled supplements still contained banned drugs months later** — Cohen 2014 (JAMA)
8. **<1% of likely violative supplements receive any enforcement action** — Cohen 2018 (Drug Test Anal)
9. **>200 distinct unproven Long COVID treatments marketed online, often $500–$5,000 OOP** — Brennan et al. 2023 (BMJ)
10. **Influencer ad spend in wellness was $5.2B in 2023; published RCT funding for same categories was <$50M** — Influencer Marketing Hub 2024 / NIH RePORTER

---

## How this document feeds the architecture

| Finding | Where it lives | How it shapes OpenLabel |
|---|---|---|
| Disclaimer paradox (Loewenstein 2011) | Catalog signal `inverse:disclosure-paradox` | Disclosure presence is not a positive trust signal |
| 7 new tactics from research | Catalog signals `tactic:reciprocity-negative-option` through `tactic:loss-aversion-sunk-cost` | Tactic catalog expanded from 10 to 17 |
| Cook-Greuter rename | Catalog signal `tactic:cognitive-load-distress-state-targeting` | Mainstream-citation framing for the same construct |
| Korownyk 2014 Dr. Oz study | Catalog signal `inverse:celebrity-physician-halo` | High-severity inverse signal |
| Tactic density predicts harm (Mathur 2019) | Evaluation rules — 8+ tactics → automatic Disqualify | Methodology choice grounded in empirical evidence |
| Newmaster / Tucker / Cohen supplement data | Industry context in `10-anchor-library.md` and public methodology page | Calibrates anchor cases against industry base rates |
| Aggregation white space (Agent 4) | `11-aggregate-research-and-analytics.md` | Confirms OpenLabel's defensible center |

---

## Citation hygiene

This document should be the source of truth for any external publication. Before citing in press / public methodology / academic presentations:

1. Confirm the citation through the original source (page numbers, exact methodology). Some citations here are from prior recall and need verification.
2. Check for newer literature that updates the finding. Several of these studies are 5–15 years old; replication or updated meta-analyses may exist.
3. Note where evidence is RCT-grade vs. observational vs. theoretical — the methodology integrity OpenLabel demands of others applies to OpenLabel's own claims.

When OpenLabel cites these findings in published reports, citations include full bibliographic detail and link to the source where available.

---

*Update this document as new research is identified. Annual literature scan recommended — the catalog should not drift behind the field.*
