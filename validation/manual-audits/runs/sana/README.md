# Manual Audit Workbook - Sana

Run ID: `manual-sana-2026-05-29-v0`

This folder contains the first-pass OpenLabel manual audit workbook for Sana Health / Sana Device at https://www.sana.io/.

The trace is:

`target identity -> salience plan -> source -> claim/extraction -> evidence check -> signal -> subdimension judgment -> dimension concern score -> interpretation -> panel/verdict`

## Key Finding

Sana has a real FDA De Novo-cleared prescription adjunctive indication for temporary relief of neuropathic pain in adults. The first-pass concern is not absence of all evidence; it is public-scope clarity. Several public pages appear stale or inconsistent after the January 16, 2026 De Novo grant, and broader PTSD/rest/recovery/app-store benefit claims require careful separation from the cleared pain indication.

## Artifact Index

- `00-run-metadata.md`
- `00a-salience-plan.csv`
- `00b-source-expansion-queue.csv`
- `01-source-ledger.csv`
- `01a-source-query-log.csv`
- `02-claim-inventory.csv`
- `03-evidence-verification.csv`
- `04-extractions.csv`
- `05-signal-detections.csv`
- `06-dimension-scores.csv`
- `06a-subdimension-scores.csv`
- `06b-interpretation-layer.csv`
- `06c-d8-developmental-profile.csv`
- `07-panels.md`
- `08-prompt-log.md`
- `09-final-audit.md`
- `10-fixture-seed.md`
- `11-coverage-gaps.md`
