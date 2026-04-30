# Data Model

*The database is the load-bearing artifact of OpenLabel. The prose docs describe methodology; this document describes what gets persisted, in what shape, and why. Modeled on PatientPunk's three-layer + run-traceability architecture (see `references/PATIENTPUNK_REFERENCE.md`).*

---

## Why persist at this depth

OpenLabel is two products:

1. **A consumer audit tool** that produces individual prescription labels for offerings.
2. **An industry research instrument** that, as audit volume accumulates, surfaces aggregate patterns no other tool currently sees.

The consumer tool needs every audit to be reproducible, resumable across crashes, and re-runnable under updated methodology. The research instrument needs every audit's findings to be queryable across the corpus — by category, by population, by tactic, by time period, by methodology version.

Both demands are satisfied by the same architecture: a structured database with run traceability, three layers separating raw / config / extracted, and explicit methodology versioning.

---

## Layer architecture

Same shape as PatientPunk. Three layers, with a fourth view layer for analytics.

```
┌─────────────────────────────────────────────────────┐
│  LAYER 1 — RAW                                      │
│  As-fetched material. The ground truth corpus.      │
└─────────────────────────────────────────────────────┘
              ▲                  ▲
              │                  │
┌─────────────┴──────────┐  ┌────┴──────────────────┐
│  LAYER 2 — CONFIG      │  │  LAYER 3 — EXTRACTED  │
│  Catalogs, methodology │  │  Findings, scores,    │
│  versions, prompts,    │  │  detections, labels,  │
│  models, anchors       │  │  verdicts             │
└────────────────────────┘  └───────────────────────┘
                                       │
                                       ▼
                             ┌────────────────────────┐
                             │  LAYER 4 — VIEWS       │
                             │  Aggregate analytics,  │
                             │  industry rollups,     │
                             │  cohort comparisons    │
                             └────────────────────────┘
```

---

## Layer 1 — Raw

The as-fetched material. Never mutated after capture.

```sql
-- One row per offering ever audited. Stable identity across runs.
CREATE TABLE audit_targets (
    target_id        INTEGER PRIMARY KEY,
    canonical_name   TEXT NOT NULL UNIQUE COLLATE NOCASE,
    company_name     TEXT,
    primary_url      TEXT,
    category_id      INTEGER REFERENCES categories(category_id),
    subcategory      TEXT,
    target_population TEXT,             -- JSON array of population_ids
    first_seen_at    INTEGER NOT NULL,
    notes            TEXT
);

-- One row per page fetched (across all audits). Pages may be reused across runs.
CREATE TABLE crawled_pages (
    page_id      INTEGER PRIMARY KEY,
    target_id    INTEGER NOT NULL REFERENCES audit_targets(target_id),
    audit_run_id INTEGER NOT NULL REFERENCES audit_runs(run_id),
    url          TEXT NOT NULL,
    page_kind    TEXT,                 -- 'homepage', 'product', 'science', 'faq', 'about',
                                       -- 'pricing', 'terms', 'privacy', 'wayback_snapshot'
    fetched_at   INTEGER NOT NULL,
    raw_html     TEXT,
    extracted_md TEXT,                 -- markdown form for downstream extraction
    wayback_date INTEGER,              -- if this is a historical snapshot
    http_status  INTEGER,
    notes        TEXT
);

CREATE INDEX idx_pages_target ON crawled_pages(target_id);
CREATE INDEX idx_pages_run    ON crawled_pages(audit_run_id);

-- Primary-source queries (PubMed, FDA, FTC, USPTO, ClinicalTrials.gov)
CREATE TABLE primary_source_queries (
    query_id     INTEGER PRIMARY KEY,
    audit_run_id INTEGER NOT NULL REFERENCES audit_runs(run_id),
    source       TEXT NOT NULL,        -- 'pubmed', 'fda_warning_letters', 'fda_devices',
                                       -- 'ftc_enforcement', 'clinicaltrials_gov',
                                       -- 'uspto', 'sec_filings', 'court_records'
    query_text   TEXT NOT NULL,
    queried_at   INTEGER NOT NULL,
    result_json  TEXT,                 -- raw API response
    n_results    INTEGER,
    notes        TEXT
);

CREATE INDEX idx_psq_run    ON primary_source_queries(audit_run_id);
CREATE INDEX idx_psq_source ON primary_source_queries(source);

-- Patient signal queries (Reddit, Patient Punk, patient organizations)
CREATE TABLE patient_signal_queries (
    query_id     INTEGER PRIMARY KEY,
    audit_run_id INTEGER NOT NULL REFERENCES audit_runs(run_id),
    source       TEXT NOT NULL,        -- 'reddit', 'patient_punk', 'mepedia', etc.
    query_text   TEXT NOT NULL,
    queried_at   INTEGER NOT NULL,
    result_json  TEXT,
    n_results    INTEGER
);
```

---

## Layer 2 — Config

Catalogs, methodology versions, prompts, models. The reproducible substrate.

```sql
-- Methodology version. Bump when dimension weights, signal catalog, scoring rules
-- change in any way that affects produced verdicts.
CREATE TABLE methodology_versions (
    version_id     INTEGER PRIMARY KEY,
    semver         TEXT NOT NULL UNIQUE,    -- '1.0.0', '1.1.0', '2.0.0'
    released_at    INTEGER NOT NULL,
    changelog      TEXT,
    weights_json   TEXT NOT NULL,           -- dimension weights for composite
    threshold_json TEXT NOT NULL            -- threshold disqualifiers
);

-- Run traceability — every audit run, agent version, model versions, prompt versions.
-- Same pattern as PatientPunk's extraction_runs.
CREATE TABLE audit_runs (
    run_id            INTEGER PRIMARY KEY,
    target_id         INTEGER NOT NULL REFERENCES audit_targets(target_id),
    started_at        INTEGER NOT NULL,
    completed_at      INTEGER,
    mode              TEXT NOT NULL,        -- 'rapid_triage' (1) | 'full_audit' (2) | 'company_portal' (3)
    methodology_version_id INTEGER NOT NULL REFERENCES methodology_versions(version_id),
    signal_catalog_version TEXT NOT NULL,
    agent_version     TEXT NOT NULL,
    git_commit        TEXT,
    config_json       TEXT NOT NULL,        -- model versions, prompt versions, source list
    crawl_plan_json   TEXT,
    source_coverage   TEXT,                 -- 'complete' | 'substantive' | 'provisional'
    status            TEXT NOT NULL         -- 'running' | 'completed' | 'failed'
);

CREATE INDEX idx_runs_target ON audit_runs(target_id);
CREATE INDEX idx_runs_method ON audit_runs(methodology_version_id);
CREATE INDEX idx_runs_started ON audit_runs(started_at);

-- The signal catalog. Each signal is a structured record. Loaded by the agent at
-- Stage 5. Versioned via signal_catalog_version on audit_runs.
CREATE TABLE signals (
    signal_id            INTEGER PRIMARY KEY,
    catalog_version      TEXT NOT NULL,
    name                 TEXT NOT NULL UNIQUE,
    description          TEXT NOT NULL,
    detection_kind       TEXT NOT NULL,    -- 'regex' | 'llm' | 'composite'
    detection_spec_json  TEXT NOT NULL,    -- patterns, prompts, inputs
    confidence_default   TEXT NOT NULL,    -- 'high' | 'medium' | 'low'
    severity_levels_json TEXT NOT NULL,    -- when this signal contributes to disqualifier
    dimension_tags       TEXT NOT NULL,    -- JSON array of dimension ids ('D1','D3',...)
    label_section_feeds  TEXT NOT NULL,    -- JSON array of which prescription label
                                           -- sections this signal contributes to
    notes                TEXT
);

CREATE INDEX idx_signals_catalog ON signals(catalog_version);
CREATE INDEX idx_signals_kind    ON signals(detection_kind);

-- Versioned LLM prompts referenced by signals
CREATE TABLE prompts (
    prompt_id   INTEGER PRIMARY KEY,
    name        TEXT NOT NULL,
    version     TEXT NOT NULL,
    body        TEXT NOT NULL,
    UNIQUE (name, version)
);

CREATE TABLE models (
    model_id          INTEGER PRIMARY KEY,
    name              TEXT NOT NULL UNIQUE,
    provider          TEXT NOT NULL,
    cost_input_per_m  REAL,
    cost_output_per_m REAL,
    role              TEXT NOT NULL       -- 'fast' | 'strong'
);

-- Offering taxonomy — categories and subcategories
CREATE TABLE categories (
    category_id INTEGER PRIMARY KEY,
    name        TEXT NOT NULL UNIQUE,
    parent_id   INTEGER REFERENCES categories(category_id),
    description TEXT,
    extension_schema_json TEXT             -- per-category extraction extensions,
                                           -- modeled on PatientPunk's per-subreddit schemas
);

-- Population taxonomy — chronic illness vs general wellness vs performance, etc.
CREATE TABLE populations (
    population_id INTEGER PRIMARY KEY,
    name          TEXT NOT NULL UNIQUE,
    icd10_codes   TEXT,                    -- JSON array where applicable
    notes         TEXT
);

-- Anchor library — calibration cases with their canonical audits
CREATE TABLE anchor_cases (
    anchor_id    INTEGER PRIMARY KEY,
    target_id    INTEGER NOT NULL REFERENCES audit_targets(target_id),
    audit_run_id INTEGER NOT NULL REFERENCES audit_runs(run_id),
    role         TEXT NOT NULL,            -- 'positive' | 'negative' | 'edge_case'
    pattern_tag  TEXT NOT NULL,            -- 'late-cycle-hype', 'phenomenological-colonization', etc.
    notes        TEXT
);
```

---

## Layer 3 — Extracted

The findings of every audit. Every row carries `audit_run_id`.

```sql
-- One row per claim extracted from the marketing surface.
CREATE TABLE extracted_claims (
    claim_id      INTEGER PRIMARY KEY,
    audit_run_id  INTEGER NOT NULL REFERENCES audit_runs(run_id),
    page_id       INTEGER NOT NULL REFERENCES crawled_pages(page_id),
    claim_text    TEXT NOT NULL,
    claim_context TEXT,                    -- surrounding section / headline / qualifier proximity
    page_position TEXT,                    -- selector or location hint
    claim_type    TEXT,                    -- 'efficacy', 'mechanism', 'comparative', 'testimonial'
    risk_category TEXT,                    -- 'A' | 'B' | 'C' | 'D' (FTC framework)
    confidence    TEXT NOT NULL,           -- 'high' | 'medium' | 'low'
    extracted_by  TEXT                     -- 'regex' | 'fast_model' | 'strong_model'
);

CREATE INDEX idx_claims_run ON extracted_claims(audit_run_id);

-- Evidence linking a claim to a primary source result (or its absence)
CREATE TABLE claim_evidence (
    evidence_id          INTEGER PRIMARY KEY,
    claim_id             INTEGER NOT NULL REFERENCES extracted_claims(claim_id),
    source_query_id      INTEGER REFERENCES primary_source_queries(query_id),
    substantiation_tier  INTEGER NOT NULL, -- 1 (multiple RCTs) — 6 (testimonial only)
    population_match     TEXT,             -- 'matches' | 'partial' | 'mismatch' | 'unknown'
    dose_match           TEXT,
    endpoint_match       TEXT,
    note                 TEXT,
    confidence           TEXT NOT NULL
);

-- Mechanism statements (what the company says produces the benefit)
CREATE TABLE mechanism_statements (
    mech_id            INTEGER PRIMARY KEY,
    audit_run_id       INTEGER NOT NULL REFERENCES audit_runs(run_id),
    page_id            INTEGER REFERENCES crawled_pages(page_id),
    statement_text     TEXT NOT NULL,
    proposed_mechanism TEXT NOT NULL,
    plausibility_rating TEXT,              -- 'plausible' | 'speculative' | 'contradicted'
    delivery_rating    TEXT,               -- 'product delivers it' | 'uncertain' | 'cannot deliver'
    confidence         TEXT NOT NULL
);

-- Testimonials surfaced
CREATE TABLE testimonials (
    testimonial_id INTEGER PRIMARY KEY,
    audit_run_id   INTEGER NOT NULL REFERENCES audit_runs(run_id),
    page_id        INTEGER REFERENCES crawled_pages(page_id),
    text           TEXT NOT NULL,
    attribution    TEXT,
    page_position  TEXT,
    flagged_as_proof_of_mechanism INTEGER  -- 0/1
);

-- Persuasion-architecture features (countdowns, urgency badges, social-proof counters)
CREATE TABLE persuasion_features (
    feature_id   INTEGER PRIMARY KEY,
    audit_run_id INTEGER NOT NULL REFERENCES audit_runs(run_id),
    page_id      INTEGER REFERENCES crawled_pages(page_id),
    feature_kind TEXT NOT NULL,            -- 'countdown_timer' | 'scarcity_badge' |
                                           -- 'social_proof_counter' | 'urgency_text' | etc.
    evidence     TEXT,
    confidence   TEXT NOT NULL
);

-- Team and advisors
CREATE TABLE team_members (
    member_id    INTEGER PRIMARY KEY,
    audit_run_id INTEGER NOT NULL REFERENCES audit_runs(run_id),
    target_id    INTEGER NOT NULL REFERENCES audit_targets(target_id),
    name         TEXT NOT NULL,
    role         TEXT,
    claimed_credentials TEXT,
    verified_credentials TEXT,
    institution_verified INTEGER,          -- 0/1/null
    decorative_flag      INTEGER,          -- 0/1 — credential is impressive but irrelevant
    confidence           TEXT NOT NULL
);

-- Pricing and subscription structure
CREATE TABLE pricing_records (
    pricing_id   INTEGER PRIMARY KEY,
    audit_run_id INTEGER NOT NULL REFERENCES audit_runs(run_id),
    target_id    INTEGER NOT NULL REFERENCES audit_targets(target_id),
    headline_price NUMERIC,
    annual_cost  NUMERIC,
    subscription_required INTEGER,
    upsell_path  TEXT,
    notes        TEXT
);

-- Disclosures (with proximity to triggering claim)
CREATE TABLE disclosures (
    disclosure_id INTEGER PRIMARY KEY,
    audit_run_id  INTEGER NOT NULL REFERENCES audit_runs(run_id),
    page_id       INTEGER REFERENCES crawled_pages(page_id),
    text          TEXT NOT NULL,
    triggering_claim_id INTEGER REFERENCES extracted_claims(claim_id),
    proximity_rating    TEXT,              -- 'proximate' | 'distant' | 'buried'
    prominence_rating   TEXT,              -- 'prominent' | 'modest' | 'whisper'
    likely_curing_misimpression INTEGER    -- 0/1
);

-- THE central artifact — every signal detection across all audits.
CREATE TABLE detections (
    detection_id INTEGER PRIMARY KEY,
    audit_run_id INTEGER NOT NULL REFERENCES audit_runs(run_id),
    signal_id    INTEGER NOT NULL REFERENCES signals(signal_id),
    severity     TEXT NOT NULL,            -- 'low' | 'medium' | 'high' | 'disqualify'
    confidence   TEXT NOT NULL,
    evidence_quote TEXT,
    page_id      INTEGER REFERENCES crawled_pages(page_id),
    page_section TEXT,
    contributing_inputs_json TEXT,         -- which extractions / queries fed this detection
    detected_at  INTEGER NOT NULL
);

CREATE INDEX idx_detections_run    ON detections(audit_run_id);
CREATE INDEX idx_detections_signal ON detections(signal_id);
CREATE INDEX idx_detections_sev    ON detections(severity);

-- Dimension rollups (the legacy view, computed from detections)
CREATE TABLE dimension_scores (
    score_id     INTEGER PRIMARY KEY,
    audit_run_id INTEGER NOT NULL REFERENCES audit_runs(run_id),
    dimension    TEXT NOT NULL,            -- 'D1', 'D2', etc., plus 'composite'
    score        REAL NOT NULL,
    contributing_detection_ids TEXT,       -- JSON array
    UNIQUE (audit_run_id, dimension)
);

-- Verdict per audit run
CREATE TABLE verdicts (
    verdict_id            INTEGER PRIMARY KEY,
    audit_run_id          INTEGER NOT NULL REFERENCES audit_runs(run_id) UNIQUE,
    verdict               TEXT NOT NULL,    -- 'Surface' | 'Flag' | 'Caution' | 'Disqualify'
    composite_score       REAL,
    threshold_disqualifier_triggered TEXT, -- which one, if any
    one_sentence_rationale TEXT NOT NULL,
    source_coverage       TEXT NOT NULL    -- inherited from audit_runs but also computed
);

-- Consumer-specific findings (Safety, Cost-to-Value, etc.)
CREATE TABLE consumer_findings (
    finding_id   INTEGER PRIMARY KEY,
    audit_run_id INTEGER NOT NULL REFERENCES audit_runs(run_id),
    layer        TEXT NOT NULL,            -- 'safety' | 'cost_to_value' | 'practical_burden' |
                                           -- 'data_rights' | 'alternatives' | 'consumer_fit'
    finding_json TEXT NOT NULL,
    confidence   TEXT NOT NULL
);

-- Composed prescription label (final output)
CREATE TABLE prescription_labels (
    label_id     INTEGER PRIMARY KEY,
    audit_run_id INTEGER NOT NULL REFERENCES audit_runs(run_id) UNIQUE,
    composed_json TEXT NOT NULL,           -- structured label content
    rendered_html TEXT,
    rendered_image_path TEXT,
    composed_at  INTEGER NOT NULL
);

-- Patient-signal aggregations per audit
CREATE TABLE patient_signal_results (
    result_id    INTEGER PRIMARY KEY,
    audit_run_id INTEGER NOT NULL REFERENCES audit_runs(run_id),
    source       TEXT NOT NULL,
    population   TEXT,
    metric       TEXT NOT NULL,            -- 'sentiment_positive_pct', 'gap_score', etc.
    value        REAL NOT NULL,
    n            INTEGER,
    confidence   TEXT NOT NULL
);

-- Cultural trend signals (4 sub-signals from 06-cultural-trend-layer)
CREATE TABLE trend_signals (
    trend_id              INTEGER PRIMARY KEY,
    audit_run_id          INTEGER NOT NULL REFERENCES audit_runs(run_id),
    target_id             INTEGER NOT NULL REFERENCES audit_targets(target_id),
    commercial_velocity   INTEGER,         -- 0–3
    market_saturation     INTEGER,         -- 0–2
    claim_expansion       INTEGER,         -- 0–2
    patient_precedence    INTEGER,         -- 0–3
    total_score           INTEGER,
    interpretation        TEXT,            -- 'established' | 'active_trend' | 'peak_capitalization'
    notes                 TEXT
);
```

---

## Layer 4 — Aggregate views

Materialized views and persistent aggregations supporting the industry research instrument. These are the queries OpenLabel runs to publish state-of-the-industry insights.

Views to define (each is a SQL view or periodically refreshed materialized view):

| View | What it answers |
|---|---|
| `tactic_prevalence_by_category` | What % of audits in category X have manipulation tactic Y, sliced by 12-month period |
| `verdict_distribution_by_category` | Verdict breakdown (Surface / Flag / Caution / Disqualify) per category over time |
| `tactic_density_distribution` | Histogram of manipulation tactic densities across the corpus, by category and population |
| `cultural_trend_heatmap` | Trend-layer scores by category, time series — surfaces gold rushes |
| `claim_substantiation_gap_by_category` | Distribution of claim-language tier vs evidence tier gaps |
| `patient_marketing_gap` | Where patient-reported outcomes most diverge from marketing claims, by offering and category |
| `enforcement_correlation` | Do dimension scores predict subsequent FTC/FDA enforcement (longitudinal lag analysis) |
| `methodology_version_drift` | Same offerings audited across methodology versions — score divergence |
| `population_exposure` | What aggregate score profile does each target population face? (chronic illness vs general wellness, etc.) |
| `inverse_signal_correlation` | Where do "trustworthy-looking" signals correlate with low-quality outcomes |
| `anchor_drift` | Anchor cases re-audited under current methodology — how scores have shifted |

The `11-aggregate-research-and-analytics.md` document specifies the exact queries and the published-report templates. This document just defines the schema that supports them.

---

## Intermediate-file contracts

Each pipeline stage writes a checkpoint file to disk in addition to the database. PatientPunk's pattern: persist intermediates so stages are independently inspectable, restartable, and replayable.

| Stage | Checkpoint file | Purpose |
|---|---|---|
| Stage 2 — Plan crawl | `runs/<run_id>/crawl_plan.json` | The plan; resume Stage 3 from here |
| Stage 3 — Extract | `runs/<run_id>/extracted.json` | Cumulative extraction; saved every 1,000 items |
| Stage 4 — Verify | `runs/<run_id>/verifications.json` | Per-claim verification results; resume on crash |
| Stage 5 — Detections | `runs/<run_id>/detections.json` | Per-signal detection results |
| Stage 6 — Scores | `runs/<run_id>/scores.json` | Computed scores + threshold check results |
| Stage 7 — Label | `runs/<run_id>/label.json` + `label.png` | Composed label, structured + rendered |

Each file mirrors a database table; the database is the source of truth. The files exist for: human inspection, debugging, replay across stage boundaries, and methodology-research access without needing to query the database.

---

## Run traceability — the most important pattern

Every row in Layer 3 carries `audit_run_id`. Every audit run carries `methodology_version_id`, `signal_catalog_version`, `agent_version`, `git_commit`, and a `config_json` capturing model versions, prompt versions, source list.

**Consequence 1 — Reproducibility.** Any audit can be exactly replayed. If a verdict is contested, the entire chain — pages crawled, sources queried, signals fired, scores computed — is on disk.

**Consequence 2 — Methodology evolution without history loss.** When methodology updates, old audits stay valid under their version. Re-running under the new methodology produces a new `audit_run_id` for the same `target_id`. Both verdicts coexist; comparison is queryable.

**Consequence 3 — Industry research validity.** Aggregate analytics can filter by methodology version, ensuring cross-audit comparisons are calibrated. "Did manipulation tactic density rise in supplements between 2026 Q1 and 2027 Q1?" is meaningful only if both quarters used the same methodology version (or with explicit normalization across versions).

**Consequence 4 — Drift monitoring.** When the same anchor case is re-audited under newer methodology, score deltas reveal whether methodology updates produce intended effects. Anchor drift is a methodology QA signal.

---

## Idempotency rules

- `INSERT OR IGNORE` for catalog tables (signals, prompts, models, categories, populations, anchor_cases). Re-loading the catalog never duplicates.
- New `audit_run_id` for every re-audit. Never overwrite a prior run.
- `extracted_claims`, `detections`, etc. are run-scoped. A re-audit produces a new set; old set persists.
- `verdicts` table has `UNIQUE(audit_run_id)` — one verdict per run, computed once.

PatientPunk's `--reclassify` flag (force re-classification even on already-processed pairs) maps to OpenLabel's "re-run audit under new methodology" — produces new rows, preserves old.

---

## Two timescales

OpenLabel runs at two cadences. The schema supports both.

**Per-audit (latency-sensitive).** Mode 1 / 2 / 3 — single offering, on-demand. Stages execute sequentially with intermediate checkpoints. Most rows in this audit's `audit_run_id` are populated during this single run.

**Periodic (corpus-wide).** Background tasks that:
- Re-audit anchor cases under new methodology versions for drift QA
- Refresh patient-signal queries for live offerings (Patient Punk corpus pulls)
- Run aggregate-view materialization
- Refresh trend-layer metrics (PubMed publication-date monitoring, market-saturation scans)
- Score new offerings discovered through the consumer-submission database-building loop

The schema does not distinguish: same tables, same `audit_runs` row pattern. The `mode` column carries the cadence flag.

---

## Cache and normalizer schemas (added v2.1)

Per-source TTL caching prevents redundant queries across audits. Per-source normalizers convert raw API/HTML responses to structured records. Full architectural detail in `12-pipeline-architecture.md`.

```sql
-- Per-source query cache. Keyed by (source, normalized_query).
CREATE TABLE source_cache (
    cache_id        INTEGER PRIMARY KEY,
    source          TEXT NOT NULL,        -- 'pubmed' | 'fda_warning_letters' | etc.
    normalized_key  TEXT NOT NULL,        -- canonical query string for cache key
    raw_response    TEXT NOT NULL,        -- raw API/HTML response
    fetched_at      INTEGER NOT NULL,
    expires_at      INTEGER NOT NULL,     -- TTL boundary, computed at insert
    n_results       INTEGER,
    methodology_version_id INTEGER REFERENCES methodology_versions(version_id),
    UNIQUE (source, normalized_key)
);

CREATE INDEX idx_cache_source_key ON source_cache(source, normalized_key);
CREATE INDEX idx_cache_expires    ON source_cache(expires_at);

-- Per-source TTL configuration (managed via config, persisted for audit trail).
CREATE TABLE source_ttl (
    source              TEXT PRIMARY KEY,
    default_ttl_seconds INTEGER NOT NULL,
    rationale           TEXT
);

-- Source freshness monitoring — when tracked offerings have source content that changes,
-- trigger re-audit logic.
CREATE TABLE source_freshness_watches (
    watch_id        INTEGER PRIMARY KEY,
    target_id       INTEGER NOT NULL REFERENCES audit_targets(target_id),
    source          TEXT NOT NULL,
    query_key       TEXT NOT NULL,
    last_check_at   INTEGER NOT NULL,
    last_hash       TEXT,                 -- content hash of last fetch
    on_change       TEXT NOT NULL,        -- 'reaudit' | 'flag' | 'log_only'
    UNIQUE (target_id, source, query_key)
);

-- Normalizer registry. Each source has a versioned normalizer; normalizer outputs
-- to the per-source structured tables (study_record, enforcement_record, etc.).
CREATE TABLE normalizers (
    normalizer_id     INTEGER PRIMARY KEY,
    source            TEXT NOT NULL,
    version           TEXT NOT NULL,
    output_schema     TEXT NOT NULL,      -- JSON describing output structure
    introduced_at     INTEGER NOT NULL,
    deprecated_at     INTEGER,
    UNIQUE (source, version)
);
```

**Cache TTL defaults** (per `12-pipeline-architecture.md`):

| Source | Default TTL |
|---|---|
| PubMed, USPTO, ClinicalTrials, SEC EDGAR | 30 days |
| FDA Warning Letters, FTC enforcement | 7 days |
| Wayback Machine snapshots (once captured) | infinite |
| Reddit / Patient Punk active queries | 24 hours |
| Reddit / Patient Punk historical snapshots | infinite |
| Meta Ad Library | 24 hours |
| DomainTools / Whois | 30 days |
| Crunchbase / PitchBook | 7 days |
| State AG enforcement | 7 days |
| Glassdoor | 30 days |
| Cochrane reviews | 30 days |

**Cross-audit query coalescing.** Cache key is `(source, normalized_key)`. Multiple concurrent audits requesting the same query share cached results. Reduces cost and accelerates common-category audits.

**Source freshness monitoring.** Background worker walks `source_freshness_watches`, re-fetches sources for tracked offerings on cadence, compares content hash. When changed: triggers re-audit per `on_change` policy. PatientPunk's incremental-corpus pattern at the source level.

**Normalizer versioning.** When a source's API changes, normalizer version bumps. Old data remains queryable under its original normalizer version; re-running an audit may re-normalize under a newer version (preserving both).

---

## Schema evolution

Schema changes happen. They are logged separately:

```sql
CREATE TABLE schema_versions (
    version_id   INTEGER PRIMARY KEY,
    semver       TEXT NOT NULL UNIQUE,
    applied_at   INTEGER NOT NULL,
    migration_sql TEXT NOT NULL
);
```

Migrations are forward-only; old data is preserved. Where a schema change requires re-deriving old data (e.g., a new field in `extracted_claims`), the migration explicitly notes whether back-fill is feasible or not.

---

## Open design questions

1. **SQLite vs Postgres.** PatientPunk uses SQLite, which is correct for a single-author research pipeline. OpenLabel as a public consumer service will likely outgrow SQLite once concurrent user submissions exceed light load. Current decision: start in SQLite (fastest path to MVP); migrate to Postgres when concurrency requires it. Schema is portable.

2. **Storage of raw HTML.** Page captures can be large. Options: store in DB blob (simple), store on object storage with DB pointer (scales). Current decision: DB blob until it hurts, then move.

3. **PII / privacy.** Patient-signal queries return content that may contain identifiable details. Apply PatientPunk's pattern: SHA-256 hash any usernames before persistence, never store raw usernames. Comments / posts retained as text but linked to hash, not username.

4. **Public vs private aggregations.** Some Layer 4 views are publishable; some are internal QA only. Maintain a flag per view.

5. **Anchor-case canonical-audit lock.** Should anchor cases have a "canonical" audit at a frozen methodology version, plus a series of re-audits under newer versions? Yes — proposal: `anchor_cases.canonical_audit_run_id` is the frozen reference; `anchor_audit_history` tracks re-audits.

---

*Update this document when the schema evolves or when a new aggregation view is added. The DDL here is the source of truth for the database. Methodology versioning is non-negotiable — every change that affects a verdict bumps `methodology_versions.semver`.*
