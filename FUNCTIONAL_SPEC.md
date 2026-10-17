# Market White Space Analyst — Functional Specification

| | |
|---|---|
| **Status** | Draft v0.1 |
| **Date** | 2026-10-17 |
| **Purpose** | Identify under-served B2B software sub-categories ("white space") by mapping the G2 category taxonomy, cross-referencing analyst taxonomies, and scoring opportunities on weighted, configurable criteria. |

---

## 1. Overview

### 1.1 Problem statement
Founders, product strategists, and investors lack a systematic, repeatable way to see which B2B software sub-categories are under-served. G2 provides a rich taxonomy and product-level ratings, but it must be combined with analyst taxonomies, market-size data, regulatory drivers, and AI-disruption signals to judge where a new product could win.

### 1.2 Goals
1. Build a complete, refreshable map of G2 categories → sub-categories → top products.
2. Cross-reference the G2 taxonomy with Gartner and IDC (and optionally Forrester, Capterra) where possible.
3. Restrict the universe to B2B software products.
4. Detect sub-categories that are *white space* (weak incumbents, unmet demand, tailwinds).
5. Produce transparent, weighted, configurable scores and a ranked recommendation list per white-space sub-category.

### 1.3 Non-goals
- Not a financial model or investment advice engine.
- Not a real-time competitive intelligence monitor (batch/scheduled refresh only).
- Not a replacement for primary customer research; outputs are hypotheses to validate.

### 1.4 Users
| Persona | Needs |
|---|---|
| Product strategist / founder | Ranked white-space list, drill-down into why a sub-category scored well |
| VC / corp-dev analyst | Market-size and growth evidence, export to spreadsheet |
| Analyst-administrator | Configure thresholds, weights, sources; monitor scan health |

### 1.5 Key terms
| Term | Definition |
|---|---|
| Category | Top-level grouping in the G2 taxonomy (e.g., "Marketing") |
| Sub-category | Child node of a category (e.g., "Account-Based Marketing"). Deeper levels are flattened into a path but scored at leaf level |
| Top product | One of the 5 highest-ranked products in a sub-category that meets the minimum score threshold |
| White space | A sub-category with high opportunity and weak/absent incumbent strength, as defined in §5 |
| Canonical sub-category | A merged entity linking equivalent nodes across G2, Gartner, IDC, etc. |

---

## 2. Scope and Assumptions

- **Primary source:** G2 (categories, products, ratings, review counts, vendor attributes).
- **Secondary sources (best effort):** Gartner market guides / Magic Quadrant categories, IDC market taxonomy, optional Capterra / Forrester / Crunchbase / government and regulatory feeds, news and job-posting signals.
- **Data access:** Must use licensed API access or permitted data feeds. Where a source offers no API and its terms prohibit scraping, the system uses licensed exports or manual uploads (see §9, Compliance). Gartner and IDC data are paywalled; the integration model is *import-based* (CSV/XLSX upload or licensed API) rather than crawling.
- **Taxonomy depth:** G2 may expose 2–4 levels. The system stores the full tree and treats leaf nodes as "sub-categories".
- **Currency/regions:** Default global/USD; region filter is a configurable extension (Phase 2).

---

## 3. System Context

```
 ┌─────────────┐   ┌──────────────┐   ┌──────────────────┐
 │ G2 source   │   │ Gartner/IDC  │   │ Signal sources   │
 │ connector   │   │ import/API   │   │ (reg., AI, news) │
 └──────┬──────┘   └──────┬───────┘   └────────┬─────────┘
        └──────────┬──────┴───────────────────┘
                   ▼
        ┌─────────────────────┐
        │ Ingestion & Raw     │  (F1, F2)
        │ Snapshot Store      │
        └─────────┬───────────┘
                  ▼
        ┌─────────────────────┐
        │ Normalization,      │  (F1.4, F3)
        │ Taxonomy Mapping,   │
        │ Product Filtering   │
        └─────────┬───────────┘
                  ▼
        ┌─────────────────────┐
        │ Analysis & Scoring  │  (F4, F5)
        │ Engine              │
        └─────────┬───────────┘
                  ▼
        ┌─────────────────────┐
        │ UI / API / Exports  │
        └─────────────────────┘
```

---

## 4. Functional Requirements

Priority: **M** = Must, **S** = Should, **C** = Could.

### F1. Market Category Scan

| ID | Requirement | Pri |
|---|---|---|
| F1.1 | The system shall scan the full G2 category tree and persist every category and sub-category with: G2 ID, name, slug, parent, depth, URL, description, product count, and scan timestamp. | M |
| F1.2 | The system shall support scheduled (default: monthly) and on-demand scans. | M |
| F1.3 | Each scan shall create an immutable, timestamped **snapshot**; prior snapshots are retained to enable trend and diff views (new / removed / renamed categories). | M |
| F1.4 | The system shall detect taxonomy changes between snapshots (added, removed, merged, renamed nodes) and flag them in a change log. | S |
| F1.5 | **Cross-reference:** the system shall map each G2 sub-category to equivalent Gartner and IDC market definitions where available, producing a *canonical sub-category* with a link to each source node. | S |
| F1.6 | Cross-reference matching shall use (a) exact/normalized name match, (b) synonym dictionary, (c) embedding/semantic similarity, and (d) manual override. Each match stores a **confidence score (0–1)**. | S |
| F1.7 | Matches below a configurable confidence (default 0.75) shall go to a **review queue** for manual confirmation; unconfirmed matches are excluded from cross-source metrics. | S |
| F1.8 | The system shall record, per canonical sub-category, which sources cover it (coverage matrix) and expose *source-coverage* as a quality indicator. | S |
| F1.9 | Scan jobs shall be resumable, rate-limited, retried with back-off, and report progress, failures, and coverage (% categories successfully captured). | M |

### F2. Full Mapping: Categories → Sub-categories → Top 5 Products

| ID | Requirement | Pri |
|---|---|---|
| F2.1 | For every sub-category, the system shall retrieve products with: name, vendor, G2 product ID, overall star rating (0–5), review count, G2 satisfaction/market-presence indicators (where available), pricing model, company size segments served, URL, and last-updated date. | M |
| F2.2 | **Top-5 selection:** rank products in a sub-category by a configurable ranking basis (default: G2 rating, tie-break by review count, then G2 satisfaction score) and select the top **N** (default N = 5, configurable). | M |
| F2.3 | **Quality threshold:** a product qualifies as a *top product* only if its rating is **≥ `min_top_product_score`** (default **4.0**, configurable globally and per category). The prompt's "above 4" is implemented as `> 4.0` or `≥ 4.0` per a configurable comparison operator (default `>`). | M |
| F2.4 | Configurable **minimum review count** (default 10) to prevent low-sample ratings from qualifying. | S |
| F2.5 | If fewer than N products qualify, the sub-category shall list those that do (0–N) and record the shortfall; shortfall itself is a white-space signal (see F4). Qualification is evaluated *after* the B2B/software filter (F3) so that the Top-5 are the top 5 *valid* products. | M |
| F2.6 | The system shall produce a **hierarchical map** (tree and flat table) of Category → Sub-category → Top products, filterable and sortable, with snapshot-date selector. | M |
| F2.7 | The map shall be exportable to CSV, XLSX, and JSON, and available through a REST API. | M |
| F2.8 | The map shall show, per sub-category: product count (total / qualified), average rating of top products, rating spread, count of vendors, and top-vendor concentration. | S |
| F2.9 | Where a product is listed in multiple sub-categories, it shall appear in each, with a cross-listing flag. | S |

### F3. B2B and Software Filter

| ID | Requirement | Pri |
|---|---|---|
| F3.1 | The system shall classify each product on two independent attributes: **Is Software** (SaaS, on-prem, mobile, API/platform) vs. non-software (services, hardware, consulting, staffing, marketplaces of services, agencies), and **Is B2B** (sold to businesses/organizations) vs. B2C/consumer. | M |
| F3.2 | Classification shall use layered evidence: (a) G2 metadata (category type, "services" vs "software" sections, target segment tags), (b) vendor attributes (customer segment, pricing page, "contact sales"), (c) rule-based keyword heuristics, (d) LLM-assisted classification with a structured, auditable output (label, confidence, rationale). | M |
| F3.3 | Products labeled non-B2B **or** non-software shall be excluded from top-5 ranking and all downstream scoring, but retained in the data store with an `excluded_reason` for audit. | M |
| F3.4 | Products with classification confidence below a threshold (default 0.8) shall be flagged for manual review; configurable behavior: *exclude pending review* (default) or *include pending review*. | S |
| F3.5 | **Hybrid products** (e.g., B2B2C platforms, prosumer tools sold to SMBs) shall be handled by a configurable policy (default: include if the buyer is a business). | S |
| F3.6 | Entire sub-categories where no products survive filtering shall be marked `no_valid_products` and excluded from white-space scoring (but listed for transparency). | M |
| F3.7 | Users shall be able to override a classification; overrides are versioned, attributed, and take precedence over automated results on future scans. | S |
| F3.8 | A **filter report** shall summarize counts excluded per reason and per category. | S |

### F4. White Space Analysis and Recommendation

**Definition (default, configurable):** A sub-category is *white space* when demand and market attractiveness are high while incumbent supply is weak, fragmented, or aging. The engine evaluates two groups of indicators.

**Supply-side gap indicators (derived from F2/F3 data):**

| Indicator | Signal of white space |
|---|---|
| Qualified-product shortfall | Fewer than N products meet the score threshold |
| Low rating level / high rating dispersion | Incumbents are mediocre or inconsistent |
| Low review volume | Small adoption base or immature market |
| Few vendors / low vendor count growth | Limited competition |
| Vendor concentration (HHI) | Either a dominant, complacent leader (disruptable) or highly fragmented long tail (consolidation opportunity); both modes are scored and labeled |
| Product age / stagnation | Incumbents with old products, low release cadence (where available) |
| Adjacent-category demand | Strong demand in neighbouring categories with no direct product |

**Demand-side and tailwind indicators:** market size, growth, regulatory pressure, AI openness, funding activity, search/hiring interest (scored in F5).

| ID | Requirement | Pri |
|---|---|---|
| F4.1 | The engine shall compute all supply-side indicators for every eligible sub-category and normalize them (percentile rank or min-max, configurable) to a 0–100 scale. | M |
| F4.2 | The engine shall compute a **Supply Gap Score** and a **Market Attractiveness Score** (from F5 criteria) and combine them into an **Opportunity Score** (see F5.4). | M |
| F4.3 | Sub-categories shall be classified into tiers: **White Space** (Opportunity ≥ threshold, default 70), **Emerging** (50–69), **Contested** (< 50), **Saturated** (high supply strength, low gap). Thresholds are configurable. | M |
| F4.4 | Sub-categories shall be assigned an **opportunity archetype** to explain the nature of the gap, for example: *Unserved*, *Underserved (weak incumbents)*, *Fragmented (roll-up)*, *Legacy-disruptable*, *Regulation-forced*, *AI-native reinvention*. | S |
| F4.5 | The system shall generate a **recommendation** for each white-space sub-category: a short narrative (what, why now, who buys, competitive set, risks, suggested wedge) built from structured evidence, with citations to underlying data points. | M |
| F4.6 | All narrative claims must be traceable to stored evidence; the generator must not assert facts that are absent from the evidence set. Missing data is stated explicitly as "not available". | M |
| F4.7 | The system shall rank white-space sub-categories and support filtering by category, archetype, region (Phase 2), company-size segment, and minimum confidence. | M |
| F4.8 | The system shall provide **sensitivity analysis**: show how the ranking changes when weights are varied (e.g., ±20%) and flag unstable rankings. | S |
| F4.9 | The system shall support **comparison across snapshots** (e.g., "became white space since last quarter"). | C |
| F4.10 | The system shall allow users to save, name, and share scenarios (a set of weights/thresholds). | S |

### F5. Weighted Scoring per White Space Sub-category

#### F5.1 Criteria catalogue

Each criterion is scored on a normalized **0–100** scale (higher = more attractive), has a configurable **weight**, a defined data source, and a **confidence** value. Default weights sum to 100%.

| # | Criterion | What it measures | Example inputs | Default weight |
|---|---|---|---|---|
| 1 | **Compliance / Regulation Impact** | Strength of regulatory or compliance mandates that force or accelerate purchase | New/impending laws (e.g., EU AI Act, DORA, NIS2, CSRD, SEC cyber rules, HIPAA/PCI updates), fines, deadlines, number of affected firms | 15% |
| 2 | **Revenue Pool** | Current addressable spend in the sub-category | Analyst market-size estimates (Gartner/IDC), public-vendor revenue, bottom-up (buyers × ACV) | 20% |
| 3 | **Market Growth** | Expected growth rate | CAGR forecasts, YoY review growth, new-vendor formation rate | 15% |
| 4 | **AI-Driven Opening** | Degree to which generative/agentic AI can reinvent or newly enable the product | Automation potential of workflows, document/unstructured-data intensity, incumbents' AI maturity, AI-native entrants count | 15% |
| 5 | **Competitive Intensity (inverse)** | Weakness/absence of competition | Output of F4 Supply Gap Score | 15% |
| 6 | **Customer Pain / Dissatisfaction** | Unmet needs of current users | Review sentiment on incumbents, recurring complaint themes, low satisfaction subscores | 5% |
| 7 | **Buyer Urgency & Budget** | Likelihood of purchase soon | Search/intent trends, RFP volume, hiring signals, budget ownership | 5% |
| 8 | **Ease of Entry** | Barriers to building and selling | Data/integration moat, certification needs, sales-cycle length, switching costs, capital intensity | 5% |
| 9 | **Funding & Momentum** | Capital and attention flowing in | VC rounds, M&A activity, new-entrant count | 5% |
|   | **Total** |  | | **100%** |

*Criteria are pluggable: administrators may add, remove, rename, or re-weight criteria without code changes (see F5.6). Weights are normalized to sum to 100% automatically.*

#### F5.2 Scoring methodology

For each criterion `c` and sub-category `s`:

1. **Raw metric(s)** are gathered from sources (F5.5).
2. Raw values are converted to a **criterion score** `S(c,s)` in 0–100 via a documented rule: percentile rank across all eligible sub-categories, banded thresholds, or LLM-assisted rubric scoring (1–5 mapped to 0–100) for qualitative criteria such as AI opening and regulation impact.
3. Each score carries a **confidence** `K(c,s)` in 0–1 reflecting data completeness, recency, and source reliability.
4. **Weighted score:**
   `Attractiveness(s) = Σ w(c) · S(c,s) / Σ w(c)` (over criteria with available data; missing criteria handled per F5.7)
5. **Confidence-adjusted score (optional):**
   `Adjusted(s) = Attractiveness(s) · (α + (1 − α) · avg K)` with α configurable (default 0.7).

#### F5.3 Rubrics for qualitative criteria

| Criterion | 0–20 | 40–60 | 80–100 |
|---|---|---|---|
| Compliance / Regulation Impact | No relevant regulation | Guidance or recommended frameworks; sector-specific rules | Binding law with near-term deadlines, penalties, broad scope |
| AI-Driven Opening | Workflow is physical/relationship-led, AI adds little | AI adds incremental productivity | Core workflow can be automated/reinvented; incumbents lack AI; new data modalities unlocked |

Rubrics are stored with versions; LLM-scored values store the rubric version, prompt, model, rationale, and evidence references.

#### F5.4 Opportunity score and composite

```
Opportunity(s) = w_A · Attractiveness(s) + w_G · SupplyGap(s)
```
Defaults: `w_A = 0.6`, `w_G = 0.4` (configurable). Because criterion #5 overlaps with Supply Gap, the engine prevents double counting: if Supply Gap is included as a composite component, criterion #5's weight is set to 0 (setting `double_count_guard = true`, default).

#### F5.5 Data inputs and sources

| Criterion | Preferred sources | Fallback |
|---|---|---|
| Revenue pool | Gartner / IDC market sizing (imported), public filings | Bottom-up: #target firms × estimated ACV × penetration; LLM-estimated with flagged low confidence |
| Growth | Analyst CAGR, G2 review velocity | Search-trend and new-vendor growth proxies |
| Compliance | Regulatory registries (EUR-Lex, Federal Register, NIST, ISO), curated regulation→category mapping | LLM-assisted mapping with human review |
| AI opening | Product feature scans, AI-feature adoption among incumbents, job postings | LLM rubric assessment |
| Pain | G2 review text (themes, sentiment) | Rating sub-scores (ease of use, support, etc.) |
| Funding | Crunchbase / PitchBook (licensed) | News feeds |

| ID | Requirement | Pri |
|---|---|---|
| F5.5.1 | Every score shall store its underlying raw metrics, source, retrieval date, and confidence. | M |
| F5.5.2 | Source connectors shall be modular, with per-source enablement and credentials. | M |

#### F5.6 Configurability requirements

| ID | Requirement | Pri |
|---|---|---|
| F5.6.1 | Weights shall be editable in the UI (sliders with live re-ranking), via API, and via config file; edits are versioned. | M |
| F5.6.2 | Admin can add custom criteria with a name, description, data source or manual input, scoring rule, and weight. | S |
| F5.6.3 | Preset weighting profiles: *Balanced* (default), *Regulation-led*, *AI-led*, *Revenue-led*, *Low-competition*. | S |
| F5.6.4 | Per-category weight overrides (e.g., higher compliance weight in Finance/Security categories). | C |

#### F5.7 Handling missing data

| ID | Requirement | Pri |
|---|---|---|
| F5.7.1 | Missing criteria shall not silently score 0. Configurable policy: *re-normalize weights over available criteria* (default), *impute category median*, or *exclude sub-category*. | M |
| F5.7.2 | Sub-categories with data completeness below a threshold (default 60% of weight) are flagged **Low Confidence** and optionally excluded from recommendations. | M |

#### F5.8 Score presentation (per white-space sub-category)

For each white-space sub-category, the report shall display:

1. **Header:** category path, canonical name, cross-reference badges (G2 / Gartner / IDC), tier, archetype.
2. **Opportunity score** (0–100) with rank and confidence indicator.
3. **Score breakdown table:**

   | Criterion | Raw metric | Score (0–100) | Weight | Weighted contribution | Confidence | Source |
   |---|---|---|---|---|---|---|

4. **Visuals:** radar chart of criteria, stacked bar of weighted contributions, and a 2×2 positioning matrix (Market Attractiveness vs. Supply Gap) with bubble size = revenue pool.
5. **Incumbent snapshot:** top ≤5 qualified products (name, vendor, rating, review count) and shortfall note.
6. **Narrative recommendation:** why now, buyer, suggested wedge, key risks, validation next steps.
7. **Evidence panel:** linked data points and sources with timestamps.
8. **What-would-change-this:** sensitivity of rank to weight changes (F4.8).

**Example output (illustrative only, not real data):**

| Sub-category | Opportunity | Compliance (15%) | Revenue Pool (20%) | Growth (15%) | AI Opening (15%) | Supply Gap (40% composite) | Tier |
|---|---|---|---|---|---|---|---|
| AI Governance & Model Risk Mgmt | 84 | 92 | 55 | 90 | 85 | 80 | White Space |
| Sub-category B | 76 | 60 | 85 | 70 | 78 | 74 | White Space |
| Sub-category C | 62 | 40 | 70 | 55 | 60 | 66 | Emerging |

---

## 5. Data Model (logical)

| Entity | Key fields |
|---|---|
| `Snapshot` | id, source, started_at, completed_at, status, coverage_pct |
| `Category` | id, g2_id, name, slug, parent_id, depth, snapshot_id |
| `SubCategory` | id, category_id, canonical_id, name, product_count, snapshot_id |
| `CanonicalSubCategory` | id, name, mappings[{source, node_id, confidence, status}] |
| `Product` | id, g2_id, name, vendor_id, url, description |
| `ProductListing` | product_id, subcategory_id, snapshot_id, rating, review_count, rank, qualified, excluded_reason |
| `Classification` | product_id, is_b2b, is_software, confidence, method, rationale, overridden_by |
| `Vendor` | id, name, hq, size, funding, founded |
| `CriterionDefinition` | id, name, description, rubric_version, scoring_rule, active |
| `WeightProfile` | id, name, weights{criterion→%}, version, owner |
| `Metric` | subcategory_id, criterion_id, raw_value, source, retrieved_at, confidence |
| `Score` | subcategory_id, criterion_id, profile_id, score, confidence, snapshot_id |
| `Opportunity` | subcategory_id, profile_id, attractiveness, supply_gap, opportunity, tier, archetype, rank |
| `Recommendation` | subcategory_id, profile_id, narrative, evidence_refs[], generated_at, model_version |
| `AuditLog` | actor, action, entity, before, after, timestamp |

---

## 6. Configuration Reference

| Parameter | Default | Scope |
|---|---|---|
| `top_n_products` | 5 | global / category |
| `min_top_product_score` | 4.0 | global / category |
| `score_comparison_operator` | `>` | global |
| `min_review_count` | 10 | global / category |
| `ranking_basis` | rating → reviews → satisfaction | global |
| `classification_confidence_threshold` | 0.8 | global |
| `pending_review_policy` | exclude | global |
| `xref_match_confidence_threshold` | 0.75 | global |
| `scan_schedule` | monthly | global |
| `normalization_method` | percentile | global |
| `white_space_threshold` | 70 | profile |
| `emerging_threshold` | 50 | profile |
| `missing_data_policy` | renormalize | profile |
| `min_data_completeness` | 60% | profile |
| `confidence_alpha` | 0.7 | profile |
| `attractiveness_vs_gap_weights` | 0.6 / 0.4 | profile |
| `criterion_weights` | See §F5.1 | profile |

---

## 7. User Interface

| Screen | Description |
|---|---|
| **Dashboard** | Latest snapshot, scan health, top 10 white-space sub-categories, tier distribution, recent taxonomy changes |
| **Taxonomy Explorer** | Expandable tree of categories → sub-categories with top-5 products inline; filters (category, rating, source coverage); snapshot selector; export |
| **Cross-reference Review** | Queue of low-confidence G2↔Gartner/IDC matches; approve/reject/remap |
| **Classification Review** | Products pending B2B/software classification; override with reason |
| **White Space Ranking** | Sortable, filterable table of sub-categories with scores, tier, archetype; compare up to 4 side by side |
| **Sub-category Detail** | Full per-sub-category score view (§F5.8) |
| **Scoring Studio** | Weight sliders, profile management, live re-ranking, sensitivity view |
| **Admin** | Sources, credentials, schedules, thresholds, criteria, users and roles |

---

## 8. Non-Functional Requirements

| Area | Requirement |
|---|---|
| **Performance** | Full G2 scan completes within 24 h; scoring recompute for all sub-categories (<~2,000) in < 2 min on weight change; UI pages load < 3 s |
| **Scalability** | Handle ≥ 100 categories, ≥ 2,500 sub-categories, ≥ 100,000 products, ≥ 24 retained snapshots |
| **Reliability** | Scan jobs idempotent and resumable; partial failures don't corrupt snapshots |
| **Explainability** | Every score, classification, and narrative traces to evidence; LLM outputs stored with prompt/model version |
| **Reproducibility** | Same snapshot + profile always yields the same scores (deterministic scoring; LLM outputs cached and versioned) |
| **Security** | SSO / role-based access (Viewer, Analyst, Admin); secrets in a vault; audit log for config and override changes |
| **Observability** | Metrics and alerts for scan failures, source drift (layout/API changes), classification confidence drift |
| **Data quality** | Automated checks: rating range 0–5, duplicate products, orphan nodes, stale data (> configurable days) |
| **Accessibility** | WCAG 2.1 AA for UI |

---

## 9. Compliance, Legal, and Ethical Considerations

- **Terms of service:** G2, Gartner, and IDC content is licensed and protected. Prefer official APIs, partner data feeds, or licensed exports. The system must not circumvent access controls. Gartner/IDC ingestion is limited to content the organization is licensed to use.
- **Attribution and redistribution:** Exports and shared reports must respect source licensing (e.g., hide paid analyst figures from unlicensed viewers).
- **Personal data:** Review text may contain personal data; store only what is needed, avoid collecting reviewer identity, and honor retention limits.
- **Rate limiting and politeness:** Respect source rate limits and robots directives where applicable.
- **AI transparency:** Label LLM-generated classifications, scores, and narratives; keep a human-in-the-loop path for overrides.
- **Disclaimer:** Outputs are decision-support hypotheses, not investment or legal advice.

---

## 10. Technical Considerations (non-binding)

- **Pipeline:** orchestrated batch jobs (ingest → normalize → classify → map → score → recommend).
- **Storage:** relational DB for the structured model, object store for raw snapshots, optional vector index for taxonomy matching and review-theme clustering.
- **LLM usage:** classification, semantic category matching, qualitative rubric scoring, review-theme extraction, narrative generation; always schema-constrained outputs with confidence and rationale.
- **API:** REST + export endpoints (`/snapshots`, `/categories`, `/subcategories/{id}`, `/opportunities`, `/profiles`).
- **Extensibility:** source connectors and criteria implemented as plugins.

---

## 11. Acceptance Criteria (key scenarios)

1. **Taxonomy scan:** Given valid G2 access, a scan produces a snapshot capturing ≥ 95% of the category tree, with a coverage report listing any failures.
2. **Top-5 + threshold:** Given a sub-category with 8 products where 3 are rated < 4.0, the map lists at most 5 qualifying products; if only 3 qualify, it lists 3 and records a shortfall of 2. Changing `min_top_product_score` to 4.3 updates the map without a rescan.
3. **Filter:** A consumer-oriented app and a staffing-services listing are both excluded, appear in the filter report with reasons, and do not influence scores.
4. **Cross-reference:** A G2 sub-category with a ≥ 0.75-confidence Gartner match shows both source badges; a 0.6-confidence match appears in the review queue and not in the badge list.
5. **White space output:** The ranking lists only sub-categories at or above the white-space threshold, each with the full breakdown in §F5.8.
6. **Weights:** Changing the weights in Scoring Studio re-ranks the list live; the weights sum is normalized to 100%; the change is versioned in the audit log.
7. **Missing data:** A sub-category lacking revenue-pool data is scored on remaining criteria with re-normalized weights and displays a Low Confidence flag if completeness is under 60%.
8. **Traceability:** Clicking any score opens the raw metrics and sources used; the narrative contains no claim without an evidence link.

---

## 12. Release Plan

| Phase | Scope |
|---|---|
| **MVP** | F1.1–F1.3, F1.9; F2 (all Must); F3 (rules + LLM classification, overrides); F4 core (supply gap, tiers, ranking); F5 with 5 criteria (Revenue Pool, Growth, Compliance, AI Opening, Competitive Intensity), configurable weights; CSV/XLSX export |
| **Phase 2** | Gartner/IDC cross-referencing and review queue; remaining criteria; sensitivity analysis; scenarios; snapshot comparison; REST API |
| **Phase 3** | Regional views, additional sources (Capterra, Forrester, Crunchbase), custom criteria, per-category weights, scheduled alerts when sub-categories enter white-space tier |

---

## 13. Risks and Open Questions

| # | Item | Impact / Mitigation |
|---|---|---|
| R1 | G2 has no fully open API for taxonomy and rankings; scraping may violate ToS | Use G2's official API / data partnership; confirm licensing before build |
| R2 | Gartner/IDC taxonomies are paywalled and not structurally aligned with G2 | Import-based integration; accept partial coverage; show coverage matrix |
| R3 | Star ratings are biased (self-selection, vendor-solicited reviews, review-count disparity) | Review-count minimums; confidence scoring; show rating alongside volume |
| R4 | Revenue pool/growth figures differ across analysts and are often unavailable at sub-category level | Store ranges and sources; bottom-up fallback with low-confidence flag |
| R5 | LLM-scored criteria may be inconsistent | Versioned rubrics, caching, human spot-checks, calibration set |
| R6 | A sub-category with *no* products may be empty because there is no market, not because there is opportunity | Require demand-side evidence (revenue pool/growth/pain) before tiering as White Space |
| R7 | Taxonomy churn on G2 breaks historical comparisons | Canonical IDs and mapping history |

**Open questions for the product owner**
1. Is "above 4" strictly `> 4.0`, or `≥ 4.0`? (Spec defaults to `>`, configurable.)
2. What is the licensing situation for G2, Gartner, and IDC data (API keys, exports, or none)?
3. Should the Top-5 be per leaf sub-category only, or also rolled up to parent categories?
4. Target buyer of the output (founders vs. investors) — affects narrative style and the "Ease of Entry" weighting.
5. Is a regional or company-size lens required in the MVP?
6. Preferred delivery form: web app, scheduled report, or both?

---

## 14. Glossary

- **HHI:** Herfindahl–Hirschman Index; measure of vendor concentration.
- **CAGR:** Compound annual growth rate.
- **ACV:** Annual contract value.
- **Canonical sub-category:** A cross-source merged taxonomy node.
- **Snapshot:** An immutable capture of source data at a point in time.
