# OpenLabel — Current State

*Living document for session-handoff context. Updated as work proceeds. For project foundation, read `README.md` and the numbered foundation docs.*

---

## Overview

OpenLabel is a clarity instrument for navigating consumer health and wellness — a prescription-bottle-style audit tool plus an aggregate industry research instrument. Foundation v2.x complete; pre-build for the Mode 1 MVP.

- **Repo:** private, planned remote `github.com/wyatt-sudo/openLabel` (push pending — see Immediate next work).
- **Local path:** `~/Code/openLabel`. Moved off Google Drive on 2026-04-30 because Drive's sync engine corrupts git `.git/` internals.
- **Foundation version:** v2.2 (as of 2026-04-30).
- **Methodology version:** v1.x (versioning lives in `methodology_versions` table per `03-data-model.md`; first formal semver bump happens at first audit run).
- **Signal-catalog sections:** 19 (manipulation tactics, inverse trust signals, conviviality, register, scope, cultural trend, regulatory arbitrage, business due-diligence, MLM, SDT, ethics, JTBD, HBM, Cialdini cross-reference, alternative developmental lenses, evidence-stage epistemic integrity, consumer-specific).

---

## Co-development model

This project is co-developed by two agents:

- **Claude Code (Anthropic CLI)** — methodology, architecture, content, foundation docs, literature integration, signal catalog. Default for substantive thinking and writing.
- **Codex (OpenAI desktop app)** — git operations, GitHub push, local runtime/build work, file-system tasks that require macOS sandbox traversal.

Division is loose — either can do anything; each leans toward the labels above. When one completes substantive structural work, append to "Recent state changes" below so the other (and the human) sees it.

---

## Recent state changes

### 2026-04-30 — project moved from Google Drive to local disk
- Full content (279 files, excluding broken partial `.git/`) copied to `~/Code/openLabel`.
- Codex's `~/.codex/config.toml` updated: removed Drive-path trust entry, added local-path trust entry.
- Claude Code session-state at `~/.claude/projects/` copied under the new path encoding (`-Users-wyattrodgers-Code-openLabel`) so future sessions inherit conversation history.
- Drive folder has `REDIRECT_TO_LOCAL.md` at root; full original copy still present pending user cleanup.
- Reason: Drive's sync engine interferes with git `.git/` writes; agent sandboxes (Codex specifically) couldn't write there.

### 2026-04-30 — active docs trimmed; .gitignore authored
- All `legacy/` path references removed from active docs.
- All `references/PatientPunk/...` paths replaced with conceptual references to `references/PATIENTPUNK_REFERENCE.md`.
- All personal references (employer name, salary parameters, consulting clients) removed.
- All OpenClaw / Grift-by-Clawd / v1-foundation breadcrumbs cleaned.
- `.gitignore` authored: ignores `legacy/`, `references/PatientPunk/`, `.claude/`, `.codex-*`, OS / IDE / build artifacts, future SQLite databases.

### 2026-04-30 — Pipeline architecture authored
- `12-pipeline-architecture.md` documents orchestrator + workers + cadence + shared services pattern (modeled on PatientPunk's three-layer + run-traceability foundation, extended with parallel workers and a cadence-driven background tier).
- `03-data-model.md` extended with cache, normalizer registry, source-freshness watch tables.
- `04-evidence-and-sources.md` extended with new source classes — Meta Ad Library, FTC Consumer Sentinel, state AGs, USPTO TESS, DomainTools, Glassdoor, FAERS/MAUDE/CAERS, EU regulatory, Cochrane, NICE, Semantic Scholar citation graph traversal — plus a dedicated social-media architecture note.

### 2026-04-30 — Signal catalog extended; grey-area epistemology added
- §11 Business due-diligence (8 signals: capital-runway-risk, investor-pressure-misalignment, exit-pathway-volatility, founder-execution-track-record, conflict-of-interest-network, unit-economics-asymmetry, smart-money-exit-signal, capital-structure-distress)
- §12 MLM compensation-structure cluster (5 signals)
- §13 Self-Determination Theory layer (3 signals as Conviviality complement)
- §14 Beauchamp & Childress Four Principles ethics overlay
- §15 Jobs-to-be-Done — D9 enhancement
- §16 Health Belief Model — Mode 1 lens
- §17 Cialdini cross-reference table
- §18 Alternative developmental lenses (Spiral Dynamics, Kegan, Capabilities Approach)
- §19 Evidence-Stage Epistemic Integrity — load-bearing for OpenLabel's "grey-area excellence" thesis. Pioneer-stage Surface verdict now reachable for thin-evidence offerings with honest framing + plausible mechanism + active evidence trajectory.
- `07-evaluation-rules.md` extended with grey-area verdict guidance — verdict tagging by evidence stage (Established / Pioneer / Distributed Refinement).

### 2026-04-29 to 2026-04-30 — foundation v2 build
- Restructure from v1 dimensional organization to agent-runtime-shaped pipeline.
- 12 numbered foundation docs + README + literature-review.md + PATIENTPUNK_REFERENCE.md.
- 4 background research agents synthesized into the catalog (trust signals & disclaimers, manipulation tactics, industry composition / harm / enforcement, existing aggregation efforts).

---

## Immediate next work

### 1. Finish the GitHub push (highest priority)

Pending. Codex's `git init` from inside the Drive folder hit a hidden-directory write block. Now that the project is on local disk, this should work cleanly.

Steps:
1. Open Codex pointed at `~/Code/openLabel`.
2. Verify Codex has write access (Full Disk Access for Codex.app should already be granted from earlier macOS Privacy & Security step; if not, grant it).
3. Have Codex run: `git init` → `git add .` (respects .gitignore) → `git commit -m "Initial OpenLabel foundation docs"` → `git branch -M main` → `git remote add origin git@github.com:wyatt-sudo/openLabel.git` → `git push -u origin main`.
4. Confirm push succeeded; verify on github.com.
5. Delete or rename Drive folder once verified.

### 2. Calibrate the new signal clusters against anchor library

The cultural-trend signals (`§7`), business-due-diligence cluster (`§11`), MLM cluster (`§12`), SDT layer (`§13`), and Evidence-Stage cluster (`§19`) all need first calibration runs against the existing anchors (MAPS, Virta, HeartMath, Levels, Eight Sleep, Superpower, Neuronic) before they're reliable for production audits. See `06-signal-catalog.md` notes within each cluster and `10-anchor-library.md` "Anchor-coverage gaps".

### 3. YAML form of signal catalog

Currently the catalog lives in `06-signal-catalog.md` as structured markdown. Target is a YAML/JSON catalog file that the agent loads directly. Conversion is mechanical once the schema is final.

### 4. Phase 1 MVP — Rapid Triage build

Per `01-scope.md` MVP sequence. Smallest viable agent pipeline: Orchestrator + Crawler + 4 source-querier workers (PubMed, FDA, FTC, ClinicalTrials) + 4 signal-detector workers (tactics, inverse-trust, claim-substantiation, regulatory-arbitrage) + Scorer + Composer. SQLite. Tone-calibration service. Latency target 30–60s.

---

## Pending / longer-horizon

- **Patient Punk integration** — pending Eli Sakov pathway confirmation.
- **Anchor library coverage gaps** — peptides / GLP-1 microdosing case, honest-chronic-illness offering at Surface, traditional-practice commercialization, MLM-wellness offering, condition-specific app or wearable (`10-anchor-library.md` end-of-doc list).
- **Aggregate research instrument first publications** — which to start with (Annual State of the Wellness Industry, category deep-dives, population-exposure reports).
- **Mode 3 company portal output format** — refine after first real Mode 3 engagement.
- **Worker-contract implementation** — `12-pipeline-architecture.md` defines the contract; building actual workers comes during MVP build.

---

## Open architectural questions (still tracked)

From `01-scope.md`:

- **Stack:** SQLite for MVP; Postgres at concurrency limits.
- **Patient Punk integration pathway** — pending.
- **Condition-specific prioritization** — MCAS/histamine, Long COVID, or another starting point.
- **Cultural-trend data sources** without paid APIs (Meta Ad Library is the leading candidate — see `04-evidence-and-sources.md`).
- **Liability framing** — "Not medical advice" plus what else?
- **Aggregate-publishing model** — open-source / freemium / B2B-gated cuts.

---

## How to update this file

After a substantive work session, append to "Recent state changes" with a date-stamped entry. Adjust "Immediate next work" if priorities shifted. Keep "Pending / longer-horizon" current.

This file should answer in under five minutes of reading: *what's the current state, what was recently decided, what's next.* If it doesn't, prune or restructure.

---

*Last updated: 2026-04-30.*
