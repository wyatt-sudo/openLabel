# Methodology Validation Roadmap

*How OpenLabel earns trust in its own audit method. Patient Punk's first paper is the model: validate specific extraction and aggregation choices against an external truth source before claiming predictive value.*

---

## Lesson from Patient Punk

Patient Punk did not validate a broad agent in one step. It validated a narrow, inspectable pipeline:

1. Preserve reply-chain context so short replies inherit the correct treatment reference.
2. Extract treatment mentions at scale.
3. Filter to personal-use reports rather than questions, research citations, or encouragement.
4. Classify sentiment with explicit label definitions.
5. Canonicalize synonyms without collapsing specific drugs into broad categories.
6. Deduplicate to one user / one drug.
7. Compare aggregated response rates to RCT outcomes that function as ground truth.

That pattern matters more than the exact Reddit / RCT setup. The transferable method is: **turn each ambiguous judgment into a measurable subtask, validate that subtask against a stronger reference, then only aggregate once the pieces are stable.**

---

## What this means for OpenLabel

OpenLabel has more facets than Patient Punk. Not all of them can be validated against RCTs, and not all should be refined at the same time.

The audit decomposes into validation lanes:

| Lane | What it validates | Ground truth / benchmark | Early metric |
|---|---|---|---|
| Claim extraction | Did we capture the claims a consumer would reasonably perceive? | Human-labeled claim spans from anchor pages | Precision / recall on claims |
| Claim risk category | Did we classify low / moderate / high / severe claims correctly? | Human adjudication using `07-evaluation-rules.md` | Agreement rate + disagreement notes |
| Evidence retrieval | Did we find the relevant PubMed / FDA / FTC / ClinicalTrials sources? | Hand-built expected source set per anchor | Source recall |
| Evidence matching | Does the cited / found evidence actually support the claim, population, dose, endpoint, and product? | Human adjudication, RCTs where available | Support-label agreement |
| Signal detection | Did tactics / inverse trust / arbitrage signals fire only when justified? | Anchor-specific expected detections | False positive / false negative review |
| Score conversion | Do detections produce the intended dimension score and verdict? | Anchor-library expected verdicts | Anchor drift |
| Tone / label composition | Is the output clear, non-cruel, and defensible from sources? | Human review against `09-tone-and-stance.md` | Pass / revise notes |
| Patient-signal integration | Do community reports predict or contextualize outcomes? | Patient Punk SQL + RCT / observational outcomes where available | Concordance / subgroup signal |
| Cultural-trend and business-health layers | Do context signals improve risk interpretation without becoming generic vibes? | Retrospective cases, enforcement / litigation / claim drift | Qualitative drift review first |

RCTs are strongest for claim-substantiation and patient-signal validation. They are not the right ground truth for dark patterns, regulatory arbitrage, pricing asymmetry, or tone. Those need enforcement history, human adjudication, anchor drift, and later outcome correlation.

---

## Clearest next step

Build the smallest validation harness around **Mode 1 Rapid Triage claim substantiation**.

This is the right first target because:

- It is central to consumer trust: if claim-vs-evidence is wrong, the whole label becomes suspect.
- It has the clearest external references: PubMed, ClinicalTrials.gov, FDA warning letters, FTC enforcement records, and RCT / systematic-review evidence where available.
- It is already in the MVP scope.
- It creates reusable infrastructure for every later facet: fixtures, expected outputs, source coverage, confidence labels, and anchor drift.

The first harness should run on a deliberately mixed 12-anchor set: the strongest original anchors plus expansion candidates that cover regulated-device positives, honest early-stage devices, chronic-illness grey zones, charismatic healing, MLM / frequency-device negatives, supplement marketing, biomarker interpretation, and women's-health regulation.

For each anchor, create a fixture with:

- Canonical offering name and URL / page text snapshot.
- 5-15 expected consumer-facing claims.
- Expected claim risk category.
- Expected source queries.
- Expected evidence-support label per claim: supported, partially supported, unsupported, misleading, or not-checkable.
- Expected core detections: `substantiation-gap`, `clinical-language-without-rct`, `quantified-outcome-without-methods`, `mechanism-overclaim`, and applicable regulatory-arbitrage signals.
- Expected verdict and confidence label.

The first measurable goal:

> On the 12-anchor validation set, Mode 1 should preserve expected verdict tier for at least 10 / 12 anchors and correctly classify the support status of the highest-risk central claim for at least 10 / 12 anchors.

This is intentionally small. It avoids pretending OpenLabel is validated because one polished audit reads well.

---

## Sequence after the first harness

1. **Claim-substantiation harness.** Seven anchors, one central claim each, then expand to 5-15 claims per anchor.
2. **Extraction reliability pass.** Compare regex / fast-model / strong-model outputs against human-labeled fields; decide which fields require the strong model.
3. **Signal false-positive pass.** Especially MLM, SDT, business-health, cultural-trend. Use anchors plus hand-picked edge cases.
4. **Source-query refinement.** Tune PubMed / ClinicalTrials / FDA / FTC query templates by category.
5. **Patient-signal validation.** Integrate Patient Punk-style reports only after the core claim-substantiation pipeline is stable.
6. **Runtime anchor drift.** Once the database pipeline exists, rerun anchors as persisted `audit_runs` and compare old / new results.
7. **Prospective validation.** For categories with future studies or enforcement outcomes, freeze predictions before outcomes are known.

---

## Validation rule

Every new audit facet should answer four questions before it contributes strongly to a verdict:

1. **What exact object is it extracting or judging?**
2. **What stronger reference can check it?**
3. **What failure mode would harm a consumer if we got it wrong?**
4. **How will the audit expose uncertainty when the facet is weak?**

If a facet cannot answer those questions yet, it can still appear as context, but it should not silently drive the verdict.

---

## Near-term implementation artifact

Create a local validation set before building broad product features:

```
validation/
  anchors/
    maps-pbc.yaml
    virta-health.yaml
    oura-ring.yaml
    apollo-neuro.yaml
    primal-trust.yaml
    joe-dispenza.yaml
    healy.yaml
    doterra-or-young-living.yaml
    ag1.yaml
    function-health-or-insidetracker.yaml
    natural-cycles.yaml
    neuronic.yaml
  expected/
    mode1-claim-substantiation.yaml
  reports/
```

This is not busywork. It is how OpenLabel avoids becoming a convincing prose generator with unmeasured reliability.

### Why these twelve

The first validation set should not be only the original score ladder. It should include enough adjacent cases to catch false positives and unfair penalties:

- **MAPS PBC / Virta Health** — established-evidence positives.
- **Oura Ring / Apollo Neuro** — validated wearable and honest early-stage wearable.
- **Primal Trust / Neuronic** — chronic-illness grey-zone vs chronic-illness predation stress test.
- **Joe Dispenza** — charismatic retreat / testimonial-healing stress test.
- **Healy / doTERRA or Young Living** — frequency-device and MLM negative controls.
- **AG1** — influencer supplement grey zone.
- **Function Health or InsideTracker** — DTC biomarker interpretation and clinical-utility risk.
- **Natural Cycles** — regulated women's-health app with high-stakes efficacy disclosure.

---

*Update this document when a validation lane graduates from manual review to automated tests, when an external ground truth source is added, or when prospective validation begins.*
