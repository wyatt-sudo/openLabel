# Evidence and Sources

*Crawl plan and source priority. The most critical architectural decision in OpenLabel is which sources we trust and how we treat the rest. Research pipeline > dimension framework — a sophisticated framework run over poisoned inputs produces poisoned outputs.*

---

## The agentic poisoning problem

Fake articles at scale are flooding the web to manipulate search crawls and LLM training data. A tool that audits health-product marketing is a high-value target for content manipulation. A company wanting its product to score well has a strong incentive to flood the web with positive content, simulated review articles, and SEO-optimized "explainer" sites. The cost of doing this is collapsing as content generation becomes cheap.

**OpenLabel cannot take web search results at face value.** The architecture answer: a tiered source priority that weights primary, institutional, and patient-community sources heavily, and treats general web content as evidence about the *cultural environment*, not the offering itself.

---

## Source priority tiers

Rank every piece of evidence by tier. Higher tiers carry more weight; lower tiers are supplementary signal at most.

### Tier 1 — Authoritative primary

Difficult to fake at scale; carry institutional accountability:

- **PubMed and peer-reviewed literature** — not science journalism *about* studies, the studies themselves
- **ClinicalTrials.gov** registration records, including amendment history
- **FDA databases** — Devices@FDA, 510(k) clearances, PMA approvals, De Novo classifications, warning letters, enforcement actions
- **FTC enforcement database** — closing letters, complaints, consent orders, advertising substantiation rulings
- **SEC filings** for public companies (S-1, 10-K, 10-Q, 8-K)
- **Patent filings** (USPTO, EPO) — reveal mechanism claims under oath
- **Court records** — Lanham Act complaints, class actions, summary judgments
- **Wayback Machine** historical snapshots — reveals claim drift over time, especially valuable for cultural-trend signal `claim-expansion`
- **Institutional press releases** from named research universities and hospitals
- **Government statistical sources** — CDC, NIH, NCHS

### Tier 2 — Company-controlled primary (the object of audit)

The company's own marketing surface — the actual object of analysis. Not trusted as true; the point is what the company is choosing to communicate:

- Homepage and product pages (today + Wayback historical)
- Science / research / methodology pages
- About / team / leadership pages
- FAQ
- Press / blog / case studies
- Pricing pages
- Terms of service, privacy policy, disclaimers, safety language
- Careers page (when assessing maturity)

### Tier 3 — Community-derived patient signal

Lived experience from established patient communities. Significantly harder to poison at scale than SEO content:

- **Reddit patient communities** with established histories — r/cfs, r/POTS, r/MCAS, r/longcovid, r/eds, r/hashimoto, etc.
- **Patient Punk** aggregated signal once integration is live — provides queryable structured records of treatment outcomes (see `references/PATIENTPUNK_REFERENCE.md`)
- **Patient organization** publications and forums (Solve M.E., MEpedia, Dysautonomia International, The Mast Cell Disease Society, Phoenix Rising)

These have post histories, established users, community moderation. Communities call out fake-shilling. Signal is noisy at the individual level but converges with volume.

### Tier 4 — Conventional market reality

Customer-experience and support-pattern signals — not efficacy proof:

- BBB business profiles
- Trustpilot, app store reviews, Google reviews
- Crunchbase / PitchBook for funding data
- LinkedIn for team composition and tenure

Increasingly poisoned. Treat with skepticism. Cross-reference against Tier 3.

### Tier 5 — Web content (treat with explicit skepticism)

Everything else: news articles, blog posts, "explainer" sites, podcast transcripts, science journalism, influencer content.

**Default posture:** evidence about the cultural and marketing environment around the offering, not evidence about the offering itself. Do not weight as a source of truth on claims, mechanisms, or outcomes.

---

## Sources added in v2 expansion (Phase 2+)

The following sources extend the original tier list. Each carries the tier weighting noted.

### Regulatory / consumer-protection (Tier 1)

- **FTC Consumer Sentinel Network** — consumer-complaint database. Restricted programmatic access (law enforcement, consumer protection orgs); public summary statistics are accessible. Direct evidence of customer harm patterns by company name. Underused.
- **State Attorney General enforcement databases** — per-state, especially active in NY, CA, MA, WA. State-level enforcement frequently precedes federal. Programmatic access varies; manual scrape with caching.
- **FDA CAERS** (CFSAN Adverse Event Reporting) — adverse-event reports for foods, supplements, cosmetics. Underused.
- **FDA FAERS** (FDA Adverse Event Reporting System) — for drug-related products.
- **FDA MAUDE** — Medical Device Reporting database.
- **CDC VAERS** — vaccine adverse events (relevant when offerings touch immune-related claims).
- **EU regulatory** (EMA, MHRA, Health Canada) — international enforcement is sometimes ahead of US; same product / brand may be banned or restricted internationally.

### Business intelligence (Tier 1, promoted from Tier 4)

- **Crunchbase / PitchBook** — funding rounds, investor lists, founder portfolios. Promoted to Tier 1 because the new business-due-diligence cluster (`06-signal-catalog.md` §11) depends on this. Required for `business:capital-runway-risk`, `business:investor-pressure-misalignment`, `business:founder-execution-track-record`.
- **SEC EDGAR** — already Tier 1 for public companies. Now also: founder-share-sale records, insider-transaction filings (Form 4) for `business:smart-money-exit-signal`.
- **USPTO TESS (trademark)** — supplements patent search. Trademark assignee overlap reveals founder's other brands. Critical for `business:conflict-of-interest-network`.
- **DomainTools / Whois history** — domain ownership history, founder's other domains, recent ownership transfers. Catches recently-pivoted scam operations and founder-network mapping.
- **State business registrations** — Secretary of State databases per-state. Founder's other LLCs, registered agents, address overlaps.
- **Court records (PACER + state)** — already in Tier 1; emphasized here because the new business cluster needs this for execution-track-record analysis.
- **Glassdoor** — company culture / churn signal. Tier 4 default but useful for `business:smart-money-exit-signal` (executive churn) and D6 (epistemic culture).

### Social / influencer / ad-campaign signal (Tier 1 for ad data; Tier 3 for community)

- **Meta Ad Library** — public archive of ads on Facebook + Instagram. Free. **Highest-leverage social-media source.** Provides:
  - Ad campaign timeline (when did they start running ads, how many concurrently, when did they stop?)
  - Ad creative comparison over time (claim drift via `cultural-trend:claim-expansion`)
  - Targeting categories (who they're trying to reach)
  - Active vs. paused state
- **Google Ads Transparency Center** — equivalent for Google's ad inventory. Less detailed than Meta but covers a different platform.
- **TikTok Creative Center** — limited public data on category-level / trend-level activity. Useful for cultural-trend signal even when individual videos aren't accessible.
- **YouTube Data API** — channel + video metadata, transcript fetching. Useful for review / affiliate / influencer content discovery.
- **Reddit (Arctic Shift)** — already Tier 3 via PatientPunk pattern; doubles as cultural-trend Tier 1 signal for patient-community precedence.
- **Influencer-marketplace data** — when accessible (some leak / are scraped). Reveals brand-influencer relationships.
- **FTC §255 disclosure compliance** — cross-platform check. For each identified influencer endorsement, is the material connection clearly disclosed per FTC Endorsement Guides? Feeds `tactic:parasocial-influencer-endorsement`.

### Third-party testing and review (Tier 2)

- **ConsumerLab.com / Labdoor** — paywalled but public summaries accessible. Strong corroboration for supplement-category audits.
- **NSF Certified for Sport / USP Verified / Informed-Sport / Informed-Choice** — third-party certification databases. Direct positive trust signal (`S-301`).
- **Consumer Reports / Wirecutter / Choice (Australia)** — occasional product testing in wellness category.

### Research aggregation (Tier 1)

- **Cochrane reviews** — systematic-review database. Authoritative for category-level evidence.
- **NICE / CADTH / AHRQ** — health-technology assessments. Relevant when offerings touch covered conditions.
- **NCCIH (NIH)** — research summaries on complementary/integrative interventions.
- **NIH RePORTER** — federal grant-funded research database. Enables comparison of research investment to commercial-marketing investment by category (the asymmetry signal).
- **Citation graph traversal via Semantic Scholar API** — when a study is cited, what does it cite, what cites it? Reveals whether evidence is substantive or circular.

### Community / patient (Tier 3)

- **MEpedia** — community-curated reference for ME/CFS treatments and patient experience.
- **Phoenix Rising forums** — long-tenured ME/CFS discussion archives.
- **Solve M.E. / #MEAction / Dysautonomia International / The Mast Cell Disease Society** — patient-organization advisory documents.
- **Patient-organization newsletters and forums** by condition (MAPS Society for cluster headache; CHADD for ADHD; etc.).
- **Patient Punk** (when integrated) — direct SQL on Reddit-derived structured records.

---

## Social-media-specific notes

Social media is its own architecture problem. Specific challenges and the OpenLabel approach:

**Challenges:**
- Most platforms restrict programmatic access
- TikTok is especially closed
- APIs change frequently
- ToS complications for scraping
- Captcha / rate limiting
- Content disappears (deleted, account banned)

**Approach:**
1. **Meta Ad Library is the highest-leverage** free source — campaign timeline, claim drift, targeting. Run a dedicated worker.
2. **Reddit via Arctic Shift** is already in plan; PatientPunk model.
3. **TikTok Creative Center + YouTube Data API** for what's accessible; aggregate-level data even when individual content isn't.
4. **Cross-platform influencer-disclosure compliance** is the most legally-grounded signal layer — FTC §255 violations are enforceable. Per-platform but the legal frame is uniform.
5. **Cache aggressively.** Social-media content is expensive to fetch; aggressive caching is mandatory.
6. **Capture ad-creative snapshots at scrape time.** Marketing changes rapidly; the historical record is what matters.
7. **Engagement signals are deceptive** (fake engagement is rampant). Treat raw engagement counts as Tier 5; cross-reference with other signals.

---

## Data-handling architecture

Each new source needs a normalizer that converts raw API/HTML response → structured record. The normalizer registry (`12-pipeline-architecture.md` shared services) handles this.

**Per-source TTL caching** prevents redundant queries across audits:
- PubMed, USPTO, ClinicalTrials, SEC EDGAR: 30+ days
- FDA Warning Letters, FTC enforcement: 7 days
- Wayback Machine snapshots: infinite (the snapshot is the historical artifact)
- Reddit / Patient Punk / patient signal: 24h active queries; infinite for captured historical snapshots
- Meta Ad Library: 24h
- DomainTools / Whois: 30 days
- Crunchbase: 7 days

**Cross-audit query coalescing** — if 10 audits in the same category need the same PubMed query, cache returns the shared result.

**Source freshness monitoring** — when Tier 1 source content changes for tracked offerings, trigger re-audit. Schema in `03-data-model.md`.

---

## Skepticism filters for content that looks like journalism but is not

When Tier 5 content appears in a search, run these checks before treating it as informational:

- **Domain age and authority.** New domains pretending to be established publications. Sites with fewer than 100 articles. Sites whose content is overwhelmingly product-affiliate.
- **Citation behavior.** Does the article cite primary sources, or only other articles? Click through one citation chain — does it terminate in a primary source, or in a circular reference?
- **Coordinated content.** Same claims appearing across multiple recent articles, often with similar phrasing — coordinated SEO push.
- **Timing.** Article appeared shortly after the product launched. Reviews dated within days of each other across "independent" sites.
- **Author identity.** Author has no other writing presence. Author bio is generic. Author is an obvious pseudonym.
- **Disclosure pattern.** Affiliate links throughout, disclosure buried or absent.
- **AI generation tells.** Generic structure ("In this article we will explore..."), uniform paragraph lengths, hallmark phrasings, lack of specific anchored detail.

When in doubt, downgrade to Tier 5 and deprioritize.

---

## Per-category crawl extensions

Different offering categories need different page sets and primary-source queries. The pattern follows PatientPunk's per-subreddit extension-schema model (see `references/PATIENTPUNK_REFERENCE.md`).

Each category has a JSON extension schema in the database (`categories.extension_schema_json`). At Stage 2 (plan crawl), the agent loads the relevant schema and produces a category-specific plan.

### Supplement / nutraceutical / herbal product

| Source | Why |
|---|---|
| Product label image | Ingredient list, dosage, structure-function claim language |
| Manufacturer COA / lot testing | If published, gives quality signal directly |
| ConsumerLab / Labdoor lookup | Third-party testing if available |
| USP / NSF / Informed-Sport certified database | Verification of certification badges |
| FDA Tainted Products database | Adulteration history |
| FDA dietary supplement warning letters | Per-firm enforcement |
| Pharmacopeia ingredient pages | Dose/safety reference |

### Medical or wellness device

| Source | Why |
|---|---|
| Devices@FDA / 510(k) / PMA / De Novo | Regulatory classification |
| FDA recall database | Class I/II/III recall history |
| Specifications page / spec sheet | Technical delivery claims |
| Independent engineering reviews | If available |
| Patent filings on the device | Mechanism claims under oath |

### App / digital therapeutic / wearable

| Source | Why |
|---|---|
| Privacy policy + iOS/Android terms | Data rights layer (`05-extraction-spec.md`) |
| App store reviews and rating distribution | Tier 4 signal |
| HIPAA covered-entity status | Data protection claims |
| FDA SaMD classification (if applicable) | Regulatory positioning |
| Crunchbase / funding pages | Investor profile |

### Biomarker / diagnostic platform

| Source | Why |
|---|---|
| CLIA certification of testing laboratory | Lab quality |
| Specific test methodology disclosures | Sensitivity/specificity |
| HIPAA + GINA compliance | Data protection |
| FDA LDT correspondence (if any) | Regulatory positioning |
| Active litigation records | Lanham Act / consumer protection |
| Lot-level COA practice | Transparency signal |

### Coaching / program / membership / clinic

| Source | Why |
|---|---|
| Practitioner credential verification | License lookup with state board |
| Group / individual program structure | Conviviality and dependency analysis |
| Pricing and tiering | Cost-to-value layer |
| Patient testimonials | D7 phenomenological honesty + tactic 9 |
| State-board complaints (where applicable) | Regulatory exposure |

### Diagnostic biomarker / longevity-test platform

| Source | Why |
|---|---|
| CLIA / CAP accreditation | Lab certification |
| Specific markers offered + scientific basis | Direct evaluation |
| Insurance reimbursement claims (if any) | Validity signal |
| Active false-advertising litigation | E.g., recent biomarker-count suits |
| Clinical-utility evidence | Beyond analytical validity |

---

## Required source classes per audit mode

### Mode 1 — Rapid Triage (Phase 1 MVP)

Minimum required:
- Tier 2: Company homepage and one product page
- Tier 1: PubMed search on the single most important claim
- Tier 1: FDA warning letter database check on company name
- Tier 1: FTC enforcement database check on company name

If any cannot be completed in the time window, return verdict with explicit "limited source coverage" flag.

### Mode 2 — Full Audit

All Mode 1, plus:
- Tier 1: ClinicalTrials.gov for any company-claimed trials
- Tier 1: USPTO assignee patent search
- Tier 1: Wayback Machine review of historical claim posture (3-month, 12-month, launch)
- Tier 2: Team / about / careers / pricing / privacy pages
- Tier 3: Reddit search in relevant condition subreddits (or Patient Punk query when available)
- Tier 4: Funding (Crunchbase or equivalent), LinkedIn team composition
- Cultural trend layer signals (`06-signal-catalog.md` — `cultural-trend:*`)
- Per-category extension queries from above

### Mode 3 — Company Portal

Same as Mode 2, plus the company's own submitted materials and documentation. The company tells us what they have; we verify against the public record.

---

## Source-coverage labels

Every audit publishes a coverage label. This is data, not commentary.

- **Complete** — All major required source classes checked or consciously ruled out as not applicable. High confidence.
- **Substantive but incomplete** — Tier 2 + Tier 1 partial coverage, but one or more important source classes missing. Medium confidence.
- **Provisional** — Only company pages and light external search reviewed. Do not present as fully diligenced. Low confidence — use only as a quick read.

Consumer-facing translation:
- Complete → "Deep audit"
- Substantive but incomplete → "Full audit"
- Provisional → "Quick scan"

The label appears prominently on every output.

---

## Honest disclosure to users

OpenLabel publishes that:

1. Web-sourced evidence is treated with explicit skepticism by default.
2. Primary sources are weighted heavily.
3. Patient community signal is integrated as a poison-resistant evidence layer.
4. Confidence varies by source coverage, and confidence is labeled.

**This is itself a signal of epistemic integrity.** Most consumer-facing health review tools do not say this. Saying it is part of why OpenLabel deserves to be trusted.

A short methodology page on the public surface describes the source pipeline at consumer-readable depth. The full detail in this document is the reference for the audit team and the agent.

---

## Per-source output format

Every audit makes sources explicit. Same shape as PatientPunk's run-traceability — every detection traces back to which source class verified it.

```
SOURCES CHECKED:
- Tier 1 — Primary authoritative:
  - PubMed: [search terms, n_results, key references found]
  - FDA warning letters: [present/absent, references]
  - FTC enforcement: [present/absent, references]
  - ClinicalTrials.gov: [trials registered, status]
  - USPTO patents: [assignee, count, key claims]
  - Wayback Machine: [snapshots reviewed at intervals]
  - Other Tier 1: [list]
- Tier 2 — Company-controlled:
  - Pages reviewed: [list]
- Tier 3 — Patient signal:
  - Communities: [searched, query results, or "not available"]
- Tier 4 — Market reality:
  - Funding, team, reviews data
- Tier 5 — Web content:
  - [used / not used / used with skepticism for: ___]

SOURCE COVERAGE: Complete / Substantive but incomplete / Provisional
```

This non-negotiable structure is the epistemic-integrity signal that distinguishes OpenLabel from review-aggregator competitors. Source transparency is part of the product, not an afterthought.

---

*Update this document when a new authoritative source class becomes available, when a new poisoning pattern is observed in audit work, or when Patient Punk integration brings Tier 3 signal online and changes the operational pipeline.*
