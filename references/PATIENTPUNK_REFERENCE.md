# PatientPunk — Architectural Reference

PatientPunk is an open-source pipeline for aggregating, normalizing, and querying patient self-reports from Reddit and similar communities. We studied it as the architectural model for OpenLabel's three-layer + run-traceability + two-tier-LLM design.

## Source

- **Repository:** [github.com/Ely-S/PatientPunk](https://github.com/Ely-S/PatientPunk)
- **Author:** Eli Sakov (ME/CFS patient and biotech-hackathon builder)
- **Studied:** April 29, 2026

The actual codebase is **not committed to this repo** (gitignored at `references/PatientPunk/`). Clone it locally if you want to study or test against the actual code:

```bash
git clone https://github.com/Ely-S/PatientPunk.git ~/code/PatientPunk
```

## What it does

PatientPunk's pipeline reads posts from a SQLite database and produces a sentiment database: for each post/comment × drug pair, did this author have a positive, negative, or mixed experience? It also extracts demographic data per user (age bucket, sex, location, conditions, severity).

A key design principle is reply-chain context preservation: a short reply like "same, helped me" is correctly attributed to the drug being discussed in the parent post via a parent-chain walk.

## What we extracted — ten patterns

These are the load-bearing patterns we adopted from PatientPunk into OpenLabel's architecture:

### 1. Three-layer SQLite schema

```
LAYER 1 — Raw          (as-fetched material; never mutated)
LAYER 2 — Config       (catalogs, methodology versions, prompts, models, anchors)
LAYER 3 — Extracted    (findings, scores, detections, labels, verdicts)
```

PatientPunk's schema (`schema.sql` in their repo) explicitly separates these. **Where it landed in OpenLabel:** `03-data-model.md`.

### 2. Run traceability as first-class

Every row in extracted tables carries a `run_id` linking back to an `extraction_runs` table that records timestamp, git commit, extraction type, model versions, prompt versions, and config JSON. **Where it landed:** `audit_runs` table in `03-data-model.md`. Every audit is reproducible; methodology evolution preserves history.

### 3. Pipeline as discrete stages with persisted intermediates

Each pipeline stage writes a JSON checkpoint; the next stage reads it. Crash-resume is built in. **Where it landed:** Stage 3–7 intermediate-file contracts in `03-data-model.md` and `02-agent-workflow.md`.

### 4. Two-tier LLM use, explicit cost discipline

`MODEL_FAST` (Haiku-class) for high-volume extraction and prefiltering; `MODEL_STRONG` (Sonnet-class) for accuracy-critical classification. PatientPunk documents the cost differential explicitly. **Where it landed:** `02-agent-workflow.md` cost-discipline section, `05-extraction-spec.md`.

### 5. Regex-first, LLM-fill

Demographic extraction does regex extraction first (free, seconds), then LLM gap-fills. Schema files carry pattern lists per field. **Where it landed:** `06-signal-catalog.md` per-signal `detection_kind: regex | llm | composite`, with regex preferred where the pattern is detectable.

### 6. Schema-driven extraction with extensions

A `base_schema.json` defines universal fields. Per-subreddit extension schemas (e.g., `covidlonghaulers_schema.json`) add domain-specific fields and override base patterns. **Where it landed:** `04-evidence-and-sources.md` per-category crawl extensions and `05-extraction-spec.md` extension fields.

### 7. Reply-chain / context propagation

`drugs_direct` (mentioned in this comment) vs `drugs_context` (inherited from upstream parent chain) preserves attribution. **Where it landed:** OpenLabel's analog is `claim_direct` vs `claim_context` — a claim's net impression depends on surrounding page section, not just the literal sentence. See `05-extraction-spec.md` and `07-evaluation-rules.md` net-impression rule.

### 8. Confidence as first-class data

Every base-field entry has `confidence: high | medium | low`. **Where it landed:** every `extracted_*` and `detections` row in OpenLabel carries a confidence rating; the audit's overall `source_coverage` aggregates these.

### 9. SKILL.md as agentic orchestration spec

PatientPunk's `SKILL.md` is the agent's workflow contract — explore schema → propose plan → wait for approval → generate notebook. It's loaded as agent context, not just human documentation. **Where it landed:** `02-agent-workflow.md` is written in this style — a prescriptive workflow spec, with a "Common Mistakes" section.

### 10. Single-target mode (`--drug LDN`) for iteration

PatientPunk's `--drug` flag caches per-target aliases and skips LLM extract on subsequent runs. Lets you iterate on the classifier without re-running extract. **Where it landed:** OpenLabel's analog is Mode 3 single-offering iteration — cache the offering's extraction, iterate on signal definitions or scoring rules without re-extracting.

## Where each pattern lands in OpenLabel

| PatientPunk pattern | OpenLabel docs |
|---|---|
| Three-layer schema | `03-data-model.md` Layer 1 / Layer 2 / Layer 3 |
| Run traceability | `03-data-model.md` `audit_runs`, methodology versioning |
| Stage checkpoints | `02-agent-workflow.md` Stages 1–7 with persisted intermediates |
| Two-tier LLM | `02-agent-workflow.md` cost discipline; `05-extraction-spec.md` |
| Regex-first | `06-signal-catalog.md` `detection_kind` |
| Schema extensions | `04-evidence-and-sources.md` per-category; `05-extraction-spec.md` extensions |
| Context propagation | `05-extraction-spec.md` `claim_context`; `07-evaluation-rules.md` net-impression |
| Confidence first-class | All `extracted_*` and `detections` records |
| Orchestration as spec | `02-agent-workflow.md` |
| Single-target iteration | `02-agent-workflow.md` Mode 3 routing |

## Integration pathway

Direct integration (querying Patient Punk SQL from OpenLabel for patient-signal layer Tier 3) is **pending**. Eli Sakov outreach was initiated April 29, 2026; integration architecture is deferred until the pathway is confirmed. See `01-scope.md` open questions.

When integration lands, the patient-signal worker (`12-pipeline-architecture.md`) will query Patient Punk's database directly for offerings that map to conditions Patient Punk's corpus covers. This is the operational tie between OpenLabel's audit-time signal lookup and Patient Punk's continually-refreshed patient corpus.

## Why the actual code isn't in this repo

We deliberately gitignored `references/PatientPunk/` because:

1. The architectural learning is already extracted into our docs.
2. PatientPunk is publicly available at the URL above; future contributors can clone it directly.
3. Including a third-party codebase under our repo creates ambiguity about authorship and licensing.
4. Eli is a potential collaborator; the integration pathway is a relationship, not a fork.
