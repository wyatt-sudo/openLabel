# OpenLabel — Scope

*Version 2.0 — April 30, 2026.*

---

## OpenLabel is two products

**Product 1 — Consumer audit tool.** A consumer encounters a health/wellness product through social media or an influencer link. They submit the URL to OpenLabel. They get a prescription-bottle-style label that names what the offering is actually selling, with calibrated honesty about evidence, manipulation tactics, alternatives, and population fit. Three modes: rapid triage (30–60s), full audit, company portal.

**Product 2 — Industry research instrument.** As audit volume accumulates, OpenLabel publishes aggregate insight on the consumer health/wellness industry that no other tool currently produces. Manipulation-tactic prevalence by category over time. Cultural-trend heatmaps showing where commercial velocity outpaces scientific velocity. Patient-signal-vs-marketing-claim gaps. Enforcement-correlation tracking. Population-exposure analyses (what aggregate score profile does the chronic-illness population face).

These are the same underlying audit, persisted at depth in a queryable database. The same architecture serves both. See `03-data-model.md`.

---

## The core question

Not "is this a fraud?" — too narrow, too binary. The real question is:

**What is this offering actually selling you, across all its layers, and is that worth your trust, money, time, attention, and explanatory framework?**

An offering might be selling:
- A well-tested active ingredient with real evidence
- Early-stage research that is promising but not established
- A cosmological framework that reorganizes how you understand your body
- Hope and expectancy — which have real healing value when deployed honestly
- Community and belonging
- Identity and tribal membership
- A habit structure that works regardless of mechanism
- Access to a practitioner relationship that provides real support

None of these are automatically bad. The tool names what is actually being sold across all layers, not just flags the fraudulent ones. An offering that sells hope and delivers hope is not a grift. An offering that sells mechanism and delivers hope while charging for mechanism is a different thing.

It is epistemology, intuition, and analysis of the offering, the marketing, the culture, the business, and the consumer — simultaneously.

---

## The prescription label metaphor

The hero artifact is a prescription bottle label that lists:

- **Active ingredients** — what it actually does
- **Inactive ingredients** — what else it is selling (hope, identity, community, cosmology, habit structure)
- **Expected dosage** — what you can realistically expect
- **Contraindications** — who should not use this
- **Alternatives** — what else could serve this need

The label is descriptive rather than condemning. "Also contains: hope and expectancy activation" is more honest and less cruel than "this is a placebo." See `09-tone-and-stance.md` for the tone calibration that governs every word on the label.

---

## Who this is for

**Primary population: people with chronic illness and complex health challenges.**
MCAS, POTS, Long COVID, ME/CFS, dysautonomia, EDS, autoimmune conditions, environmental sensitivity, histamine intolerance, chronic inflammatory conditions. This population is the stress test for the whole industry. Every dynamic that affects all health consumers shows up in exaggerated form when the map is unclear, the stakes are immediate, and the desperation is real. They are the most epistemically vulnerable and the most underserved by existing review tools.

**Co-primary population: anyone navigating the consumer health and wellness industry.**
Anyone receiving health and wellness content through social media, influencer accounts, blogs, and landing pages — now nearly everyone. The scroll-and-encounter pattern (Instagram ad, influencer recommendation, TikTok trend, wellness newsletter) is the primary way most people are exposed to new health offerings. The tool should work as a rapid triage layer at the point of that encounter.

**Secondary populations:**
- Health practitioners evaluating offerings for patients
- Journalists and researchers mapping the wellness landscape
- Investors and founders wanting a quick epistemic read
- Companies wanting to understand how they will be evaluated by agentic systems

---

## Two surfaces

**Consumer side:** "What is this offering actually giving me?"
Entry: someone encounters a health product via social media ad, influencer recommendation, blog post, or friend referral.

**Company / agentic side:** "Will this offering survive agentic health management?"
Entry: a company wants to know how their offering will be evaluated by AI health agents that will increasingly filter what reaches consumers. Or a sophisticated consumer wants a full audit.

Same underlying assessment delivered at different depths and through different frames.

---

## Three modes

**Mode 1 — Rapid Triage** (social media scroll entry point)
30–60 second return. One-line verdict, manipulation-tactic density score, OpenLabel quick summary, link to full audit. Designed to be shareable. The OpenLabel visual — the prescription bottle label with honest contents listed — is the shareable artifact.

**Mode 2 — Full Audit** (deep entry point)
Full dimensional rollup. Patient signal layer (where Patient Punk data is available). Cultural trend analysis. Alternative comparison. Consumer fit summary. Safety flags.

**Mode 3 — Company Portal** (B2B entry point)
A company submits their own offering. Sees their score. Understands what would need to change. Natural upsell to consulting engagement.

See `08-label-composition.md` for the full output specification of each mode.

---

## The database building loop

The social media scroll pattern creates a natural database-building mechanism. Every consumer submission of a product from a social media ad builds a database of:

- What products are being actively advertised
- What claims they are making
- How those claims score against the framework
- Which influencers are promoting which products

Over time this creates a map of the consumer health landscape that is more current and more comprehensive than any manually curated list. **The users build the database by using the tool.** The aggregate research instrument (Product 2) runs against this corpus.

---

## MVP build sequence

The MVP must be trustworthy before it is comprehensive.

### Phase 1 — Rapid Triage (current focus)

The minimum viable audit that produces a trustworthy verdict in under 60 seconds. The hardest product constraint in the brief. It must be fast enough to use at the point of social media encounter and honest enough to be worth trusting.

**Phase 1 outputs:**
- One-line verdict: what this is actually selling you
- Manipulation tactic density score (the fastest and most legible single signal)
- Verdict badge: Surface / Flag / Caution / Disqualify
- Two-to-three sentence summary in plain language

**Phase 1 sources** (primary only, no general web crawl):
- Company marketing surface (homepage, sales page)
- PubMed search on primary claims
- FDA warning letter database
- FTC enforcement database

This phase proves the pipeline and the output format before adding dimension depth. It also produces the first reliable corpus for aggregate analytics.

### Phase 2 — Full Label

Complete prescription label format with all consumer verdict layers, the consumer-specific layers, and the cultural trend analysis.

### Phase 3 — Patient Punk Integration

Real-world evidence layer added once pipeline and output format are stable. Pending Eli Sakov collaboration. (Outreach initiated April 29, 2026; status: pending.)

### Phase 4 — Company Portal and B2B

Company-facing entry point and remediation consulting upsell.

### Phase 5 — Aggregate Research Instrument

Public industry-trend reports, condition-specific deep dives, methodology validation studies. See `11-aggregate-research-and-analytics.md`.

---

## Open architectural decisions

1. **Stack.** Three-layer SQLite + run-traceability is the model (informed by PatientPunk's architecture — see `references/PATIENTPUNK_REFERENCE.md`). Migrate to Postgres when concurrent load requires it.
2. **Patient Punk integration pathway.** Outreach to Eli Sakov initiated April 29, 2026; integration architecture deferred until pathway is confirmed.
3. **Condition-specific prioritization.** Which community to serve first — MCAS/histamine, Long COVID (largest), or another starting point.
4. **Cultural trend data sources** accessible without paid APIs — see `06-signal-catalog.md` (the four trend sub-signals are individual catalog entries).
5. **Liability framing.** "Not medical advice" is table stakes. What else is needed?
6. **Aggregate-research publishing model.** Open-source / fully public, gated, or freemium with deeper cuts behind a B2B layer. Default open until pressure changes the calculation.

---

## Where OpenLabel excels — the grey area

Generic LLMs already surface offerings with established RCT evidence well, and obvious frauds are visible to anyone paying attention. **OpenLabel's distinctive value is in the middle — the grey area where the answer is genuinely hard.**

A core epistemic commitment: **thin evidence is not the same as misleading.** Many offerings — especially in newly-emerging categories, integrative medicine, contemplative practice, somatic work, or chronic-illness-population care — will inherently lack rigorous RCT support. That makes them *more risky*, not ineffective or dishonest. The harm is in performing certainty unearned, not in being early.

OpenLabel's verdict ladder reflects this. A "Surface — Pioneer" verdict is reachable for thin-evidence offerings that operate with epistemic integrity (honest framing, plausible mechanism, active evidence trajectory, reasonable safety, appropriate population fit). A "Surface — Distributed Refinement" verdict is reachable for traditional / contemplative practices where RCT design is poorly suited and distributed clinical refinement is the appropriate evidence form.

The Evidence-Stage signal cluster (`06-signal-catalog.md` §19) and the grey-area verdict guidance in `07-evaluation-rules.md` make this load-bearing. Without these, OpenLabel would default to punishing the underfunded, the new, and the appropriately epistemological — three failure modes the methodology explicitly avoids.

---

## Guiding principle

*We want people to feel better. To have better information. To not get taken advantage of. To have clarity in understanding and executing their own agency when it comes to health.*

The tool is not a debunking site. It is not primarily a fraud detector. It is a clarity instrument for people navigating a complex, commercially saturated, culturally noisy health landscape. The frameworks are tools in service of this. They are not the point.
