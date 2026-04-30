# Extraction Spec

*What the agent pulls from each source at Stage 3. Schema-style. Maps directly to the Layer 3 tables in `03-data-model.md`. Modeled on PatientPunk's `base_schema.json` + per-context extension pattern.*

---

## Why a structured spec

A free-form "extract whatever seems relevant" prompt produces inconsistent extractions across audits. Aggregate analytics (Product 2) require comparable records. Schema-driven extraction with explicit fields, confidence ratings, and per-category extensions delivers that comparability without sacrificing depth.

The spec is loaded by the agent at Stage 3. Each field has:
- A type (text, number, enum, boolean, JSON)
- A description of what to extract
- Default detection method (regex, LLM-fast, LLM-strong)
- Confidence rating
- Required vs. optional

---

## Extraction targets — base fields

These are extracted for every offering, regardless of category.

### Claims

```yaml
claim:
  table: extracted_claims
  description: A specific outcome assertion or efficacy statement on the marketing surface
  fields:
    claim_text:
      type: text
      method: llm-fast
      description: Exact quote of the claim sentence
      confidence: high
    claim_context:
      type: text
      method: llm-fast
      description: Section / headline / qualifier proximity surrounding the claim
      confidence: high
    page_position:
      type: text
      method: regex
      description: Selector or location hint
      confidence: high
    claim_type:
      type: enum [efficacy, mechanism, comparative, testimonial-implied, structure-function, regulatory]
      method: llm-fast
      confidence: medium
    risk_category:
      type: enum [A, B, C, D]
      method: llm-fast
      description: Per FTC framework — A (low) to D (severe)
      confidence: medium
    contains_quantified_outcome:
      type: boolean
      method: regex
      description: Numeric outcome present (e.g., "81%", "in 12 weeks")
      confidence: high
    contains_clinical_lang:
      type: boolean
      method: regex
      description: "clinically proven", "validated", "studies show", etc.
      confidence: high
    explicit_population:
      type: text
      method: llm-fast
      description: Stated population (e.g., "for women over 40", "adults with chronic fatigue")
      confidence: medium
    cited_evidence:
      type: text
      method: regex+llm-fast
      description: Footnote / citation / link present? What does it point to?
      confidence: high
```

### Mechanism statements

```yaml
mechanism_statement:
  table: mechanism_statements
  fields:
    statement_text: { type: text, method: llm-fast }
    proposed_mechanism: { type: text, method: llm-fast }
    plausibility_rating: { type: enum [plausible, speculative, contradicted], method: llm-strong }
    delivery_rating: { type: enum [delivers, uncertain, cannot-deliver], method: llm-strong }
```

Plausibility and delivery require the strong model — they need first-principles physics/biology reasoning.

### Testimonials

```yaml
testimonial:
  table: testimonials
  fields:
    text: { type: text, method: llm-fast }
    attribution: { type: text, method: llm-fast }
    page_position: { type: text, method: regex }
    asserts_mechanism: { type: boolean, method: llm-fast,
      description: Does this testimonial assert a specific causal mechanism (e.g., "the LED therapy fixed my mitochondria") }
    suggests_typical_outcome: { type: boolean, method: llm-fast }
    flagged_population: { type: text, method: llm-fast,
      description: Is the testimonial subject from a vulnerable population (chronic illness, mental health crisis)? }
```

### Persuasion-architecture features

These are mostly **regex-detectable** — cost-discipline matters.

```yaml
persuasion_feature:
  table: persuasion_features
  detection: regex-first
  features:
    countdown_timer: 'time(?:r|left)|expires|countdown|hurry|ends (?:today|in)'
    scarcity_badge: 'only \d+ left|limited stock|while supplies last|low stock'
    urgency_text: 'limited time|act now|hurry|today only|last chance'
    social_proof_counter: '\d{3,}\+? (?:customers|users|members|happy patients)'
    credentials_display: 'Dr\.|MD|PhD|RN|LMHC|board-?certified'
    "as_seen_on": 'as seen on|featured in|trusted by'
    countdown_to_subscribe: 'sign up before|cohort closes|enrollment ends'
    institutional_logo: 'university|hospital|institute|harvard|stanford|mayo|johns hopkins'
    money_back_guarantee: 'money[- ]back|risk[- ]free|guarantee'
```

Each detected feature writes a row with the matched text, page position, and confidence (high for regex-clear, medium for ambiguous).

### Team and advisors

```yaml
team_member:
  table: team_members
  fields:
    name: { type: text, method: llm-fast }
    role: { type: text, method: llm-fast }
    claimed_credentials: { type: text, method: llm-fast }
    institution_named: { type: text, method: llm-fast }
    domain_relevance: { type: enum [direct, adjacent, decorative], method: llm-strong,
      description: Are credentials relevant to the actual offering? Decorative = impressive but unrelated. }
    photo_present: { type: boolean, method: regex }
    linkedin_referenced: { type: boolean, method: regex }
```

Verification (whether the credential is real and the institution claims them) happens at Stage 4 (Verify), not extraction. Stage 3 just captures what's claimed.

### Studies / publications cited

```yaml
study_citation:
  table: studies_cited
  fields:
    citation_text: { type: text, method: llm-fast }
    pubmed_id: { type: text, method: regex,
      description: 'PMID extracted via regex — pmid:?\s*(\d+)' }
    doi: { type: text, method: regex }
    full_url: { type: text, method: regex }
    cited_for_claim: { type: text, method: llm-fast,
      description: Which extracted claim does this citation support? }
    citation_specificity: { type: enum [primary-source, vague-allusion, news-reference],
      method: llm-fast }
```

Stage 4 then PubMed-fetches each cited study and checks population/dose/endpoint match against the claim.

### Pricing

```yaml
pricing_record:
  table: pricing_records
  fields:
    headline_price: { type: number, method: regex }
    pricing_tiers: { type: json, method: llm-fast }
    annual_cost: { type: number, method: composite,
      description: Computed from headline + subscription terms + required add-ons }
    subscription_required: { type: boolean, method: regex+llm-fast }
    upsell_path: { type: text, method: llm-fast,
      description: What additional purchases are positioned as the natural next step? }
    decoy_pricing_present: { type: boolean, method: llm-fast,
      description: Three-tier pricing where the middle tier is structured to make a higher tier look reasonable }
    free_trial_with_subscription: { type: boolean, method: regex,
      description: Free trial that auto-enrolls into paid subscription }
    cancellation_friction: { type: enum [easy, multi-step, must-call, requires-letter, opaque],
      method: llm-fast }
```

`free_trial_with_subscription` and `cancellation_friction` enable detection of the reciprocity / negative-option dark pattern (`06-signal-catalog.md`) — the most-complained-about consumer pattern per FTC ROSCA enforcement.

### Disclosures and disclaimers

```yaml
disclosure:
  table: disclosures
  fields:
    text: { type: text, method: llm-fast }
    triggering_claim_id: { type: ref, method: llm-fast,
      description: Which extracted_claim does this disclosure attach to? }
    proximity: { type: enum [proximate, near, distant, separate-page], method: llm-fast }
    prominence: { type: enum [prominent, modest, whisper], method: llm-fast }
    likely_curing_misimpression: { type: boolean, method: llm-strong }
    disclosure_kind: { type: enum [results-not-typical, not-medical-advice, FTC-affiliate,
      conflict-of-interest, side-effects, regulatory], method: llm-fast }
```

The presence of a disclosure does **not** signal trust (per Loewenstein, Cain, Sah 2011 — the paradoxical disclosure effect). It signals legal awareness. The signal catalog evaluates whether the disclosure is structurally effective vs. cosmetic.

---

## Per-category extension fields

Per-category extensions add fields to the base extraction. Loaded from `categories.extension_schema_json` at Stage 2.

### Supplement extension

```yaml
extension: supplement
adds:
  ingredients_list:
    type: json
    method: llm-fast
    description: Each ingredient with stated dose, claimed activity
  proprietary_blend_present:
    type: boolean
    method: regex
    description: '"proprietary blend"' detected — obscures individual ingredient doses
  third_party_certification:
    type: enum [USP, NSF, Informed-Sport, ConsumerLab, none, claimed-but-unverified]
    method: llm-fast
  coa_published:
    type: boolean
    method: llm-fast
    description: Lot-level certificate of analysis available?
  manufacturing_country: { type: text, method: regex }
  cgmp_claim: { type: boolean, method: regex }
```

### Device extension

```yaml
extension: device
adds:
  fda_classification_claimed: { type: enum [510k, PMA, DeNovo, listed-only,
    registered-only, none, ambiguous], method: llm-fast }
  intended_use_language: { type: text, method: llm-fast }
  dose_or_intensity_specified: { type: boolean, method: llm-fast }
  delivery_specifications: { type: json, method: llm-fast }
```

### App / digital therapeutic extension

```yaml
extension: app
adds:
  data_collected: { type: json, method: llm-fast }
  data_export_available: { type: boolean, method: llm-fast }
  data_sharing_disclosed: { type: enum [none, vendors, advertising, training-AI, undisclosed],
    method: llm-fast }
  account_required_to_use: { type: boolean, method: regex }
  hipaa_covered_entity_claim: { type: boolean, method: regex }
  fda_samd_classification: { type: text, method: llm-fast }
```

### Biomarker / diagnostic extension

```yaml
extension: biomarker
adds:
  test_panel_size: { type: number, method: regex }
  marker_list_published: { type: boolean, method: llm-fast }
  clia_certification_named: { type: boolean, method: llm-fast }
  cap_accreditation_named: { type: boolean, method: llm-fast }
  clinical_utility_disclosed: { type: text, method: llm-fast }
  retest_cadence: { type: text, method: llm-fast }
  data_portability_disclosed: { type: enum [exportable, viewable-only, locked-in],
    method: llm-fast }
```

### Coaching / program extension

```yaml
extension: coaching
adds:
  practitioner_license_named: { type: text, method: llm-fast }
  practitioner_state_board_lookup_possible: { type: boolean, method: llm-strong }
  group_vs_individual: { type: enum [group, individual, mixed], method: llm-fast }
  protocol_described: { type: boolean, method: llm-fast }
  protocol_evidence_cited: { type: boolean, method: llm-fast }
  ongoing_subscription: { type: boolean, method: llm-fast }
```

---

## Confidence and method discipline

Every extracted record carries a `confidence` rating (high / medium / low). PatientPunk's pattern: confidence is first-class data, not commentary.

| Confidence | When to assign |
|---|---|
| **high** | Regex-detected feature with clear pattern match; structured field clearly stated on page; primary-source citation that resolves cleanly |
| **medium** | LLM-extracted from clear context; unambiguous interpretation; standard-pattern claim |
| **low** | Ambiguous or implicit; LLM had to infer; multiple plausible interpretations |

The audit's overall `source_coverage` is computed in part from the distribution of confidences across required fields.

---

## Cost discipline at extraction

PatientPunk's two-tier model:

- **MODEL_FAST (Haiku-class)** — high-volume extraction, batched 10–20 items per call, with retry on output-count mismatch (per PatientPunk's batched-extraction pattern; see `references/PATIENTPUNK_REFERENCE.md`)
- **MODEL_STRONG (Sonnet-class)** — accuracy-critical fields only: plausibility/delivery rating, mechanism overclaiming detection, decorative-credential classification, disclosure-effectiveness rating

**Regex-first wherever possible.** Persuasion-architecture features, structured claim phrases ("clinically proven"), pricing patterns, citation extraction (PMIDs, DOIs) — all regex. Reserve LLM calls for ambiguity.

---

## Crash recovery

Per PatientPunk's extraction-internals pattern (see `references/PATIENTPUNK_REFERENCE.md`):

- Save extraction checkpoints every 1,000 items.
- Use `INSERT OR IGNORE` so re-runs are safe.
- Each batch failure (LLM returns wrong number of results) splits into smaller batches and retries up to 2 levels of recursion before giving up on those items.
- Re-running an audit re-uses prior extractions where the page hash matches; only re-extracts pages that changed.

---

## Output validation

Every extraction stage emits a coverage report:

```
EXTRACTION COVERAGE
Pages crawled: 14 / 14 planned
Claims extracted: 23 (high: 15, medium: 6, low: 2)
Mechanism statements: 4
Testimonials: 18 (3 flagged as proof-of-mechanism)
Persuasion features: 7 detected (countdown timer, social proof counter, scarcity badge, ...)
Team members: 6 (4 with verifiable credentials at extraction; verification in Stage 4)
Studies cited: 3 (all PMIDs resolvable)
Pricing: complete
Disclosures: 4 (1 proximate, 3 distant or buried)
Category: supplement (extension fields populated)

LOW-CONFIDENCE FIELDS:
- 2 claims with ambiguous risk-category assignment
- 1 mechanism statement with mixed plausibility rating

ESTIMATED COST:
- MODEL_FAST: $0.04
- MODEL_STRONG: $0.12
- Total stage cost: $0.16
```

This stage report writes to `runs/<run_id>/extraction_report.json` for human inspection and methodology QA.

---

*Extension schemas evolve as we audit more categories. Add a new category extension when a category's extraction needs more than two non-base fields beyond the existing taxonomy. Per-category extension lifecycle: draft → run on 5 cases → review extracted-field hit rate → adjust → commit version bump.*
