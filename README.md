# OpenLabel — Project Foundation

OpenLabel is a clarity instrument for navigating the consumer health and wellness landscape. The hero artifact is a prescription bottle label that names what an offering is actually selling across all its layers — active ingredients, hope, cosmology, identity, community — not just whether it is fraudulent.

It is two products built on one architecture:

1. **Consumer audit tool** — prescription-label-format audit of any health/wellness offering, in three modes (rapid triage, full audit, company portal)
2. **Industry research instrument** — as audit volume accumulates, the database becomes a queryable lens on industry-wide marketing-integrity, claim-substantiation, manipulation-tactic prevalence, and cultural-trend signals at population scale. Per literature review (`references/literature-review.md` §4), no incumbent currently aggregates this signal.

OpenLabel's distinctive value is in the **grey area** — the cases where the answer is genuinely hard. Generic LLMs handle established-evidence and obvious frauds. OpenLabel handles the middle: offerings with thin formal evidence but plausible mechanism, traditional practices being commercialized, integrative-medicine offerings, chronic-illness-population care. The methodology explicitly distinguishes "thin evidence + honest framing" (reasonable Pioneer-stage choice) from "thin evidence + performed certainty" (predatory). See `01-scope.md` and `07-evaluation-rules.md` for the grey-area verdict logic.

---

## Reading order

For a new contributor or for the agent loading context, read in this order:

| File | Purpose | Load when |
|---|---|---|
| [`01-scope.md`](01-scope.md) | What OpenLabel is, who for, three modes, MVP sequence, open questions | Always — orientation |
| [`02-agent-workflow.md`](02-agent-workflow.md) | Agent orchestration spec — stages 1–7 of the audit pipeline | Agent: load at audit start |
| [`03-data-model.md`](03-data-model.md) | Three-layer SQLite schema, run traceability, intermediate-file contracts, cache/normalizer | Anyone implementing storage / queries |
| [`04-evidence-and-sources.md`](04-evidence-and-sources.md) | Source priority tiers, anti-poisoning, per-category crawl extensions, social media specifics | Stages 1–2 |
| [`05-extraction-spec.md`](05-extraction-spec.md) | What to pull from each source — schema for claims, mechanism, team, pricing, etc. | Stage 3 |
| [`06-signal-catalog.md`](06-signal-catalog.md) | Flat catalog of detection signals — manipulation tactics, inverse trust signals, conviviality, register, cultural trend, scope, business, MLM, SDT, ethics, evidence-stage, consumer-specific | Stage 5 |
| [`07-evaluation-rules.md`](07-evaluation-rules.md) | Substantiation hierarchy, scoring conversions, threshold disqualifiers, composite, grey-area verdict logic | Stage 6 |
| [`08-label-composition.md`](08-label-composition.md) | How signals roll into the prescription label; three modes; format specs | Stage 7 |
| [`09-tone-and-stance.md`](09-tone-and-stance.md) | Describe-don't-condemn tone calibration | Cross-cutting; loaded for any composition |
| [`10-anchor-library.md`](10-anchor-library.md) | Calibration cases (MAPS, Virta, HeartMath, Levels, Eight Sleep, Superpower, Neuronic) | Calibration; methodology QA |
| [`11-aggregate-research-and-analytics.md`](11-aggregate-research-and-analytics.md) | The industry research instrument — aggregate views, publication formats, incentive-structure speculation | Product 2 |
| [`12-pipeline-architecture.md`](12-pipeline-architecture.md) | Orchestrator + workers + cadence + shared services. Worker contract, failure modes, migration triggers | Anyone implementing the runtime |

Plus:

- [`STATE.md`](STATE.md) — **living document** capturing current state, recent decisions, and immediate next work. Read right after this README to get the live status. Updated as work proceeds.
- [`references/literature-review.md`](references/literature-review.md) — synthesized findings from research: trust signals & disclaimers, manipulation tactics, industry composition / harm / enforcement, existing aggregation efforts. Citation-rich; informs catalog and evaluation rules.
- [`references/PATIENTPUNK_REFERENCE.md`](references/PATIENTPUNK_REFERENCE.md) — what we learned from PatientPunk (github.com/Ely-S/PatientPunk) and how those patterns landed in our architecture.

---

## Project structure

```
OpenLabel/
├── README.md                                  ← you are here
├── 01-scope.md
├── 02-agent-workflow.md
├── 03-data-model.md
├── 04-evidence-and-sources.md
├── 05-extraction-spec.md
├── 06-signal-catalog.md
├── 07-evaluation-rules.md
├── 08-label-composition.md
├── 09-tone-and-stance.md
├── 10-anchor-library.md
├── 11-aggregate-research-and-analytics.md
├── 12-pipeline-architecture.md
└── references/
    ├── literature-review.md                   ← compiled academic findings
    └── PATIENTPUNK_REFERENCE.md               ← architecture lessons + repo URL
```

---

## Status

| Item | Status |
|---|---|
| Foundation architecture | Complete |
| Literature integration in catalog | Complete (`06-signal-catalog.md`) |
| Aggregate research instrument specified | Complete (`11-aggregate-research-and-analytics.md`) |
| Pipeline architecture documented | Complete (`12-pipeline-architecture.md`) |
| Mode 1 MVP build | Pre-build (Phase 1 of `01-scope.md`) |
| Cultural-trend layer calibration against anchors | First-pass + v2.2 manual rerun complete (`10-anchor-library.md`) |
| Patient Punk integration | Pending (Eli Sakov outreach initiated 2026-04-29) |
| YAML form of signal catalog | Not yet authored; markdown is current source of truth |

---

## Open questions

Tracked in `01-scope.md` "Open architectural decisions":

- **Stack:** SQLite for MVP; migrate to Postgres at concurrency limits
- **Patient Punk integration pathway:** outreach pending; integration architecture deferred
- **Condition prioritization:** which population to serve first (MCAS/histamine, Long COVID, or other)
- **Cultural-trend data sources** without paid APIs
- **Liability framing:** "Not medical advice" plus what else?
- **Aggregate-publishing model:** open-source / freemium / B2B-gated cuts

---

## How to update this foundation

- **Catalog growth** (new signals): edit `06-signal-catalog.md`. Bump catalog version. Run against anchor library before promoting.
- **Methodology change** (weights, thresholds, disqualifiers): edit `07-evaluation-rules.md`. Bump `methodology_versions.semver`. Re-run against anchor library; document deltas.
- **New literature finding:** edit `references/literature-review.md`. Cross-reference into catalog or evaluation rules where applicable.
- **New anchor case:** append to `10-anchor-library.md` using the entry template. Link to canonical `audit_run_id`.
- **Tone clarification:** edit `09-tone-and-stance.md`. Re-render existing labels through the updated tone if material.
- **Schema change:** edit `03-data-model.md`. Add a `schema_versions` migration row. Run forward migration; confirm no historical data is lost.
- **Pipeline change:** edit `02-agent-workflow.md` and / or `12-pipeline-architecture.md`. If stages are added/removed, update `03-data-model.md` intermediate-file contracts.
- **Aggregate view added:** edit `11-aggregate-research-and-analytics.md`. Specify the SQL view; note the publication target.
