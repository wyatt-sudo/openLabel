# Pipeline Architecture

*Orchestrated multi-agent pipeline with shared persistence. PatientPunk's three-layer + run-traceability foundation, extended with parallel workers, a cadence-driven background tier, and shared services. Not a full agent OS — a job-orchestration system with worker agents.*

---

## Why this architecture

OpenLabel runs at two timescales (per-audit, latency-sensitive vs. periodic, corpus-wide), serves three modes with different depth requirements (rapid triage / full audit / company portal), accumulates a queryable database for the industry research instrument, and must operate under explicit cost discipline (Mode 1 has a 60-second budget; Stage-4 source verification across PubMed + FDA + FTC + ClinicalTrials + USPTO + Wayback would take >30 seconds sequentially).

Three structural features force the architecture beyond PatientPunk's batch pipeline:

1. **Mode 1 latency** — sequential stages are impossible inside 60s. Stage 4 verification and Stage 5 signal detection must run in parallel.
2. **Aggregate research instrument** — Mode 1/2/3 audits cannot also be running anchor-drift QA, view materialization, and patient-signal corpus refresh. Background scheduler is non-negotiable.
3. **Methodology versioning** — workers must fetch the catalog at run start; each audit records the version used. Catalog must be served as an immutable resource per `signal_catalog_version`.

This is between PatientPunk's batch (sequential, single-author, intermediate-files) and a full agent OS (autonomous agents, agent-to-agent messaging, service discovery). It's a **job-orchestration pattern with single-purpose worker agents and a shared persistence layer.**

---

## Component overview

```
┌──────────────────────────────────────────────────────────────────┐
│                          Submission API                          │
│  (consumer URL, Mode 3 company submission, internal cron)        │
└──────────────────────────────────┬───────────────────────────────┘
                                   ▼
┌──────────────────────────────────────────────────────────────────┐
│                  Orchestrator Agent (per-audit)                  │
│  Identify → plan crawl → dispatch workers → compose output       │
└─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────────────────────────┘
      │     │     │     │     │     │     │      parallel
      ▼     ▼     ▼     ▼     ▼     ▼     ▼      Stage 3-4
   ┌────┐┌────┐┌────┐┌────┐┌────┐┌────┐┌──────┐
   │Crwl││PMd ││FDA ││FTC ││USP ││Wbk ││Soc.  │   workers
   │ Wkr││Wkr ││Wkr ││Wkr ││Wkr ││Wkr ││Sgnl  │   (one per
   │    ││    ││    ││    ││    ││    ││Wkr   │    source)
   └─┬──┘└─┬──┘└─┬──┘└─┬──┘└─┬──┘└─┬──┘└──┬───┘
     │     │     │     │     │     │      │
     └─────┴─────┴──┬──┴─────┴─────┴──────┘
                    ▼
       ┌──────────────────────────┐
       │  Shared persistence      │
       │  SQLite → Postgres       │
       │  (raw / config /         │
       │   extracted)             │
       └──────────┬───────────────┘
                  ▼
   ┌──────────────────────────────────────────────────────┐
   │  Signal-Detector Workers (parallel, by category)     │   Stage 5
   │  Tactic / Inverse-trust / Conviviality / Register /  │
   │  Cultural-trend / Business / MLM / SDT / Ethics /    │
   │  Evidence-stage — independently runnable             │
   └────────────────────┬─────────────────────────────────┘
                        ▼
        ┌───────────────────────────────────┐
        │  Scorer + Verdict Worker          │   Stage 6
        │  (loads methodology version;      │
        │   applies thresholds)             │
        └───────────────┬───────────────────┘
                        ▼
        ┌───────────────────────────────────┐
        │  Composer Agent                   │   Stage 7
        │  (assembles label sections;       │
        │   applies tone calibration)       │
        └───────────────────────────────────┘
```

Plus a **cadence-driven background tier** (separate from per-audit work):

```
┌──────────────────────────────────────────────────────────────────┐
│                     Background Scheduler                         │
│                                                                  │
│  Daily:    aggregate-view materialization                        │
│            source-freshness check (FDA, FTC, court records)      │
│  Weekly:   anchor-drift re-audit                                 │
│            patient-signal corpus refresh (Reddit / Patient Punk) │
│  Monthly:  cultural-trend monitoring                             │
│            methodology-validation cohort study                   │
│  On-demand: Mode 3 client-specific queries                       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

Plus **shared services** every layer touches:

- **Catalog / methodology version manager** (immutable per version, served to workers at start)
- **Rate limiter / API budget tracker** (per-source caps, per-audit budget)
- **Cache** (per-source TTL — see `03-data-model.md` cache schema)
- **Normalizer registry** (per-source raw→structured transforms)
- **Tone calibration service** (Composer's last pass)
- **Database access layer** (shared connection pool; transaction boundaries per stage)

---

## Worker contract

Every worker exposes the same interface. This makes the orchestrator agnostic to worker internals.

```
worker:
  id: <stable-snake-case-name>
  stage: 1 | 2 | 3 | 4 | 5 | 6 | 7 | background
  inputs:
    - {table: <db-table>, filter: <constraints>}
    - {service: <shared-service-name>}
  outputs:
    - {table: <db-table>, semantics: insert | upsert}
  idempotency_key: <function of inputs>
  timeout_ms: <int>
  cost_budget:
    fast_model_calls: <int>
    strong_model_calls: <int>
    api_quota_units: <int>
  retry_policy:
    max_retries: <int>
    backoff: linear | exponential
    on_failure: skip | block_audit | flag_provisional
  parallel_safe: true | false   # can multiple of these run concurrently?
```

All workers are stateless beyond what they read from / write to the database. Workers do not communicate with each other directly — they communicate *through* the database.

This gives us:
- **Resumability** — restart the audit at any stage; previous workers' output is in the database
- **Observability** — every worker's I/O is inspectable as DB rows
- **Testability** — workers can be tested in isolation with seeded DB fixtures
- **Cost auditability** — budget tracking is per-worker per-run

---

## Worker types

### Orchestrator (one per audit)

**Stage 1–7 lifecycle.**
- Resolves `audit_target` identity from input
- Computes the crawl + source query plan based on category, mode, and methodology version
- Dispatches Stage 3–4 workers in parallel
- Awaits all Stage 3–4 completion (or timeout)
- Dispatches Stage 5 signal-detector workers in parallel
- Awaits Stage 5 completion
- Dispatches Scorer
- Dispatches Composer
- Returns final output to caller

For Mode 1 with strict latency: orchestrator may degrade to "best-effort" — return verdict with `source_coverage = "provisional"` if Stage 3–4 doesn't fully complete within budget.

### Crawler worker

Stage 3. Fetches pages per crawl plan. Persists to `crawled_pages`. Runs structured extraction (`05-extraction-spec.md`). Per-page cost + latency tracking.

### Source-querier workers (one per source class)

Stage 4. Each external source (PubMed, FDA, FTC, ClinicalTrials, USPTO, Wayback, SEC, Crunchbase, court records, state AGs, Meta Ad Library, etc.) gets a dedicated worker. They run in parallel.

Each source-querier:
- Reads its required inputs from the audit run
- Constructs queries
- Hits the cache first; on miss, queries the source
- Normalizes raw response to structured records via the normalizer registry
- Persists to `primary_source_queries` + the source-specific normalized record table

Per-source workers handle:
- Rate limiting (each source has its own budget)
- Authentication (where applicable)
- Retry on transient failure
- Output normalization

### Patient-signal worker

Stage 4. Specialized worker for Tier 3 signal — Reddit corpus queries, Patient Punk SQL once integrated, patient-organization forum scraping. Slower than other Stage 4 workers; may run in extended Mode 2/3 budget but skipped in Mode 1.

### Social-signal worker (NEW)

Stage 4. Cross-platform: Meta Ad Library, TikTok Creative Center, YouTube Data API, Reddit (overlap with patient-signal worker). Persists ad-campaign timelines, influencer endorsements with disclosure-compliance state, social-discussion volume metrics.

### Signal-detector workers (one per category)

Stage 5. The 11+ signal categories from `06-signal-catalog.md` each get a worker. They run in parallel where their inputs don't overlap.

Each signal-detector:
- Loads its assigned signals from the catalog (filtered by `dimension_tags` or category)
- Reads the relevant `extracted_*` records and `claim_evidence` for this audit run
- Runs detection rules (regex first, LLM-fast for ambiguous, LLM-strong for accuracy-critical)
- Persists `detections` rows with severity, confidence, evidence quote

### Scorer + Verdict worker

Stage 6. Sequential after Stage 5. Aggregates detections by dimension, computes composite, applies threshold disqualifiers, produces verdict.

### Composer agent

Stage 7. Reads all extractions + detections + scores + verdict. Assembles label sections per `08-label-composition.md`. Applies tone-calibration pass via Tone-calibration service. Renders to mode-appropriate format.

### Background workers (cadence-driven)

- **Anchor-drift worker** — weekly; re-audits anchor library under current methodology version; flags significant drift
- **Aggregate-view materializer** — daily; refreshes the Layer 4 views in `03-data-model.md`
- **Patient-signal refresher** — weekly; pulls Reddit / Patient Punk updates for tracked offerings
- **Cultural-trend monitor** — monthly; recomputes trend-layer signals across the corpus
- **Source-freshness watcher** — variable per source; detects when source content changes for tracked offerings; triggers re-audit if material

---

## Shared services

### Catalog / methodology version manager

The signal catalog (`06-signal-catalog.md`) and evaluation rules (`07-evaluation-rules.md`) are versioned. The version manager:

- Serves the current versions to running workers (immutable per version; never mutate in-place)
- Supports concurrent multiple versions (a Mode 1 audit might use v1.2 while a background anchor-drift run uses v1.3 to test)
- Records `methodology_version_id` and `signal_catalog_version` on every `audit_runs` row
- Provides version-comparison utilities for the methodology-validation cohort study

### Rate limiter / API budget tracker

Per-source rate limits (PubMed: 3 req/s without API key; FDA: per-IP throttling; Crunchbase: tier-dependent; etc.). Per-audit cost budget. Tracks cumulative cost per audit and per worker.

When a worker would exceed its budget: the worker either (a) defers to next budget window, (b) flags `source_coverage = "provisional"` and exits, or (c) escalates to orchestrator depending on retry policy.

### Cache

Per-source TTL caching. See `03-data-model.md` cache table schema.

- PubMed, USPTO, ClinicalTrials, SEC EDGAR: 30+ days
- FDA Warning Letters, FTC enforcement: 7 days
- Wayback Machine snapshots once captured: infinite (the snapshot is the historical artifact)
- Reddit subreddit queries: 24h for active queries; infinite for historical snapshots
- Meta Ad Library: 24h for active queries
- DomainTools / Whois: 30 days
- Crunchbase: 7 days

Cache key: `(source, normalized_query, methodology_version_if_relevant)`. Cross-audit query coalescing: if 10 audits in the same category need the same PubMed query, cache returns the same result.

### Normalizer registry

Each source has a normalizer that converts raw API/HTML response → structured record. The normalizer is versioned alongside the source-querier worker.

Normalizers handle:
- Schema unification (PubMed result → `study_record`; FDA letter → `enforcement_record`; etc.)
- Quality filtering (drop malformed responses; flag low-confidence parses)
- De-duplication (the same study cited via multiple sources resolves to one record)

Adding a new source = adding a new source-querier worker + a normalizer.

### Tone calibration service

Composer's final pass. Loads `09-tone-and-stance.md` rules. Runs an LLM-strong post-composition check on every consumer-facing string. If the check fails (internal jargon present, condemning phrasing, dismissive framing), it re-renders the section. This is implementable as a stateless service called once per composed audit.

### Database access layer

Connection pooling, transaction boundaries per stage, prepared-statement cache. Migrates SQLite → Postgres when concurrent-write pressure requires it (see migration triggers below).

---

## Communication: workers talk to the DB, not each other

Every worker reads inputs from the DB, writes outputs to the DB. No direct worker-to-worker messaging.

This is intentional:

- **Decoupling** — adding a new worker doesn't require updating other workers
- **Audit-trail** — every input and output is persisted; reproducibility is automatic
- **Failure isolation** — a worker failure doesn't break others; the orchestrator handles cascade decisions
- **Testability** — workers can be tested with fixture DB seeds

The cost: a database becomes a hot path. SQLite is fine for early MVP; Postgres migration is required when concurrent-write pressure exceeds SQLite's WAL serialization (typically several concurrent audits with overlapping write sets).

---

## Failure modes and retry policy

Per-worker retry policy is in the worker contract. Common patterns:

| Failure | Policy |
|---|---|
| Transient API failure (rate limit, timeout) | Exponential backoff, max 3 retries |
| Cache miss + source unavailable | Flag this source as "not checked"; downgrade `source_coverage`; continue audit |
| Normalizer parse failure | Log; flag low-confidence record; continue |
| Stage-3 extraction batch mismatch | PatientPunk pattern: split batch in half, retry each, max 2 levels of recursion |
| Stage-5 signal-detector failure | Log; mark this signal as "not run"; continue to other signals |
| Stage-6 scorer failure | Block audit; orchestrator returns failure; persist what's there for replay |
| Stage-7 composer failure | Block audit; persist all upstream work; replay later |
| Tone-calibration repeated failure | Block; require manual review; do not surface unsafely |

---

## When this architecture is wrong (migration triggers)

This pattern works for Phases 1–4 of the MVP build. Migration triggers:

| Trigger | Move to |
|---|---|
| Concurrent audits exceed SQLite's WAL throughput (typically 50+ concurrent writers) | Postgres |
| Background workers contend with foreground audit workers for connection pool | Postgres + read-replicas |
| Per-source rate limits become the bottleneck rather than DB | Per-source dedicated worker pools with persistent state |
| Signal catalog grows large enough that selective-load matters | Catalog as service with on-demand signal fetching |
| Mode 3 client-specific methodology variants emerge | Methodology-version-aware orchestrator routing |
| Aggregate analytics queries become resource-heavy enough to interfere with real-time audits | Separate analytics warehouse (read replica or column store) |
| Multi-tenant deployment (per-customer databases) | Tenant-isolation layer + sharded persistence |
| Need for human-in-the-loop verification at scale | Workflow engine (Temporal / Airflow) replacing simple orchestrator |

Most of these are Phase 5+ concerns. For now: SQLite + simple orchestrator + worker pool is correct.

---

## Implementation milestones

The build sequence aligns with `01-scope.md`:

### Phase 1 — Rapid Triage MVP

Minimum viable architecture:
- Orchestrator + Crawler + 4 source-querier workers (PubMed, FDA, FTC, ClinicalTrials)
- Stage 5 signal-detector workers for: tactics, inverse-trust, claim-substantiation, regulatory-arbitrage
- Scorer + Composer
- SQLite database
- Cache with simple per-source TTLs
- Tone-calibration service

Latency target: 30–60s for typical Mode 1.

### Phase 2 — Full Audit

Add:
- Wayback worker
- USPTO worker
- SEC + Crunchbase workers
- All remaining signal-detector workers (conviviality, register, scope, consumer-specific, business, MLM, SDT, ethics, evidence-stage, cultural-trend)
- Background aggregate-view materializer (start populating Layer 4)

### Phase 3 — Patient Punk Integration

Add:
- Patient-signal worker with Patient Punk SQL access
- Patient-language-to-clinical-vocabulary translation layer
- Patient-signal aggregations in cache

### Phase 4 — Company Portal + Social

Add:
- Mode 3 routing in orchestrator
- Remediation-pathway composition rules
- Social-signal worker (Meta Ad Library, TikTok, YouTube, influencer-disclosure)
- Background social-trend monitoring

### Phase 5 — Aggregate Research Instrument

Add:
- Anchor-drift worker (weekly)
- Methodology-validation cohort study automation
- Aggregate-publication composer (annual State of Wellness, quarterly category reports)
- B2B query API (regulators, journalists, retailers, insurers)
- Postgres migration as concurrency demands

---

## Operational notes

**Cost discipline.** Same pattern as PatientPunk (see `references/PATIENTPUNK_REFERENCE.md`):
- Test new methodology versions on `--limit 50` before running on the full corpus or anchor library
- MODEL_FAST for high-volume extraction and prefilter; MODEL_STRONG for accuracy-critical
- Per-source caching is the biggest cost lever — most queries are repeats
- Regex-first detection wherever applicable

**Reproducibility.** Every audit reproducibly replayable from `audit_runs.config_json`. PatientPunk's `extraction_runs` pattern; OpenLabel's `audit_runs` extends it.

**Observability.** Every worker logs cost, latency, and outcome. Per-stage timing surfaces in audit metadata. Methodology-version drift is visible in anchor-drift weekly reports.

**Privacy.** PatientPunk pattern: SHA-256 hash usernames before persistence. Never store raw usernames. Patient signal records are authored-by-hash.

**Versioning of this document.** Bump when the orchestrator semantics change, when shared services are added/removed, when worker contract changes. Schema-level changes also bump `03-data-model.md`'s `schema_versions`.

---

*This is the architectural spine. Per-worker implementation detail goes in worker-specific docs as the build progresses; this document stays at the orchestration / contract / shared-service level.*
