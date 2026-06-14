# Build 1.5 — HSDS ingester + AIRS→Open Eligibility crosswalk

*Seed a city's directory from its local 211 via the **Open Referral HSDS** standard, and
normalize its **AIRS** classifications into our **Open Eligibility** model. Complements
[build-1-org-ingestion.md](build-1-org-ingestion.md) (scrape): **211 for breadth +
classification, scrape for gaps + freshness, crowdsource for ground truth.** Strategy and
access terms in [directory-data-sources.md](directory-data-sources.md); taxonomies in
[taxonomy.md](taxonomy.md).*

> **Built on Gemini** like the rest (LLM-assisted crosswalk + eligibility extraction). The
> ingester itself is plain code; the LLM only does the fuzzy mapping/extraction steps.

---

## Why this exists

The directory is a *reuse* problem, not a rebuild (see landscape.md). Most cities' service
data already lives in a 211, classified with AIRS, increasingly exposed via HSDS/HSDA. This
build turns that into a cheap, structured **front door** that lands records straight into
our internal model — far cheaper and more accurate than scraping cold, and it generalizes:
"add a city" = point at that city's 211 endpoint + reuse the shared crosswalk.

## Source modes (per city, configurable)

1. **HSDA API** (HSDS-over-HTTP) — preferred; live, paginated, supports incremental sync.
2. **HSDS bulk export** — CSV/JSON Tabular Data Package import.
3. **US National 211 Data Platform** (United Way) — Search + Export APIs that map to/from HSDS.

Auth is per-city: API key / data-sharing agreement with the local 211 (directory-data-sources.md).
**Where a source carries no usable taxonomy or uses free-text eligibility, fall back to the
build-1 Gemini extractor** to derive flags — so both builds share one extraction backend.

---

## HSDS 3.0 data model we consume

HSDS 3.0 is API-first. Core objects and how we read them:

| HSDS object | Holds | Maps to our model |
|---|---|---|
| `organization` | name, description, url, email | **Org** record |
| `service` | name, description, status, fees, eligibility ref, schedules | **Program/Service** record |
| `location` + `service_at_location` | physical places; links service↔location | **Locations** (incl. multiple) |
| `address` (physical/postal) | street, city, postal | Address fields |
| `phone`, `contact` | numbers, named contacts | Phone/contact fields |
| `schedule` (regular/opening hours) | recurring hours | **Hours per day of week** |
| `service_area` | geographic coverage | Service-area / "within walking distance" support |
| `eligibility`, `required_document` | who qualifies, what to bring | **Axis B** + requirements |
| `accessibility` | access features | accessibility flags |
| `taxonomy`, `taxonomy_term` | the source's category vocabulary (AIRS), hierarchical via `parent_id` | source codes to crosswalk |
| `service_taxonomy` / `link_taxonomy` | links a term to a service / to other entities | **the classification edges** |

**Classification mechanism (the crux):** in HSDS, "an attribute is asserted by applying a
**taxonomy_term** to an object" — via `service_taxonomy` for services, `link_taxonomy` for
everything else. HSDS is **taxonomy-agnostic**: it carries whatever vocabulary the source
uses (for BC211: **AIRS**). So our job is to translate those carried terms into our model.

**The org→service→location hierarchy maps directly onto build-1's indentation tree**
(org = unindented, services/locations = indented), so both builds feed the same internal shape.

---

## The AIRS → Open Eligibility crosswalk (build once, ship everywhere)

This is the heart of build-1.5.

- HSDS delivers source `taxonomy_term`s (AIRS codes/labels). We maintain a **crosswalk
  table**: `AIRS term/code → Open Eligibility term(s)`, each tagged **Axis A (service)** or
  **Axis B (situation/eligibility)** per taxonomy.md.
- **Direction is collapsing:** AIRS is ~9,000 granular terms; Open Eligibility is coarse →
  mostly **many-to-one**. AIRS also separates service terms from **target-population facets**
  — population/target terms feed **Axis B**, service terms feed **Axis A**.
- **It's shared reference data, not per-city.** AIRS is the same standard in every city, so
  the crosswalk is built **once** and **ships in the repo** with the app. (Only the 211
  *data* is per-city; the mapping is universal — modulo US vs Canadian-EN/FR AIRS variants.)
- **How to build it:** Gemini proposes `AIRS→OE` mappings in bulk; a human reviews once;
  store as a **versioned mapping file**. Unmapped/ambiguous AIRS terms → a **review queue**
  (a §6 Tier-1 additive edit) rather than silent mis-filing — same open-world growth rule as
  build-1.
- **Eligibility free-text:** HSDS `eligibility` is partly prose → run the build-1 Gemini
  extractor to derive Axis-B flags where structured terms are absent.

---

## Trust, provenance & merge

- **211-sourced records are authoritative-but-static:** high confidence, provenance stamped
  `"<source> via HSDS on <date>"`, versioned, never hard-deleted (§6).
- **Dedup/merge** with build-1 scrape results and admin-added items: match on stable HSDS
  `id` + name/address/url/phone; keep **source lineage** per field so we know who said what.
- **Reconcile by trust tier, don't blind-overwrite:** 211 gives breadth + classification;
  **"open right now" still comes from the crowd** (§6 fast-decaying signal), and scrape-diffs
  give freshness. The trust layer (§6) decides the winner per field; labels show disagreement.

## Sync & freshness

- Scheduled re-pull from HSDS; use record `metadata`/`last_modified` for **incremental sync**
  (don't refetch everything). Cadence configurable; can align with build-1's 7-day scrape.
- Production sync runs as a non-latency-sensitive job → Gemini **batch mode** for any
  re-extraction (cost note from build-1).

## Licensing guardrail (must honor)

211 data is **read-only, by agreement, possibly no-redistribution** (directory-data-sources.md).
So:
- Keep **source-licensed data partitioned/tagged** by license, so we can honor terms.
- **Do not assume 211-sourced data may sync between instances or be re-exported** — the
  crosswalk (our reference data) is shareable; the *ingested 211 records* may not be. Confirm
  per agreement before any cross-instance feature touches them.

## Downstream (shared with build-1)

After normalization, the same pipeline applies: **chip generation** (one-sentence chips +
paragraph + contact) and **indexing** into the §5 hybrid-search store. build-1.5 only adds a
structured front door; everything after the internal model is shared.

---

## Build plan (phases)

1. **HSDS reader** (HSDA API + export import) → normalized internal records, one source (BC211).
2. **AIRS→OE crosswalk** file (Gemini-assisted, human-reviewed, versioned, in-repo).
3. **Merge/dedup + provenance/trust** against scrape + admin items.
4. **Scheduled incremental sync**.
5. **Generalize**: per-city config (endpoint, auth, AIRS variant) so a new city just plugs in.

## Open questions / TBD

- BC211's actual access mechanism (HSDA vs export vs custom) — depends on the agreement.
- Which AIRS variant BC211 uses (Canadian EN/FR) and whether it's pure AIRS or customized.
- Redistribution rights: can crosswalked/ingested 211 data move between instances?
- Coverage overlap rules: when 211, scrape, and crowd describe the same org, the field-level
  precedence table (ties into §6 tiers).
- Whether to adopt HSDS as our **internal** model too, or only as the import format.

---

## Sources

- [HSDS 3.0 — Classifications, Attributes & Taxonomies](http://docs.openreferral.org/en/3.0/hsds/classifications.html)
- [HSDS 3.0 — Schema Reference](https://docs.openreferral.org/en/latest/hsds/schema_reference.html)
- [HSDS 3.0 — API Reference (HSDA)](https://docs.openreferral.org/en/latest/hsds/api_reference.html)
- [Introducing HSDS 3.0 (API-first)](https://openreferral.org/introducing-version-3-0-of-the-human-service-data-specifications/)
