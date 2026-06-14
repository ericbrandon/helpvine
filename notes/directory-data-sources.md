# Directory data sources — seeding a city's directory

*Answers two questions: (1) can we ingest BC211 for Victoria? (2) will another
Canadian/US city have equivalent data to ingest? Short version: **yes to both, via the
211 network — but it's access-by-agreement, not open download.** Build the ingester
against the **Open Referral HSDS** standard so it generalizes, and keep the build-1 scrape
pipeline as the fallback where no cooperative 211 exists.*

---

## The key realization

Every city's "directory" already exists inside its local **211** service (operated by a
United Way or affiliate), classified with the **AIRS taxonomy** (see [taxonomy.md](taxonomy.md)).
The 211 network is increasingly exposing that data through **Open Referral HSDS/HSDA**
APIs and United Way data platforms. So the City-in-a-box seeding strategy is:

> **Ingest the local 211's data via HSDS/API where available → scrape (build-1) only the gaps.**

This makes the directory a *reuse* problem, not a *rebuild* problem — and it's textbook
"work with agencies, not around them" (§2).

---

## Victoria → BC211

- **Operated by United Way BC** (`bc211.ca` redirects to `uwbc.ca`; public site at `bc.211.ca`).
  Service directory of **12,000+ entries**, AIRS-classified.
- **Data is proprietary, shared by agreement** — not a free open-data download. There's a
  Terms of Service (`bc.211.ca/terms-of-service`) and the broader 211 model grants
  third-party access via **API keys / data-sharing agreements**, with the **local 211
  retaining ownership** of its dataset.
- Some slices already appear as **open data** (e.g. the HealthLinkBC mental-health/substance-use
  AIRS-classified dataset on Canada's open government portal).
- **Action:** approach **United Way BC / BC211** for a data-sharing agreement + API access.
  Start the relationship early; it's also a credibility/partnership win.

## Does it generalize to other cities? Mostly yes.

### United States
- **United Way Worldwide National 211 Data Platform (NDP)** aggregates 211 data covering
  **~95% of the US population** into a read-only repository with **Search** and **Export**
  APIs.
- **Terms:** *no cost* to use for non-revenue purposes; **5% of "data valuation" fee if the
  use generates revenue.** Access via an **API Request Form**; **per-local-211 ownership**,
  so a city instance needs that 211's authorization.
- → A US city instance can likely seed from the NDP, subject to per-211 sign-off.

### Canada
- 211 is organized **provincially under United Way** (BC211, Ontario 211, etc.). No single
  national open dump, but consistent operators and an emerging standard:
- **Ontario 211's Open211** initiative exposes **HSDS-compliant APIs** — a model for what
  other provinces are moving toward.

### The standard to build against
- **Open Referral — HSDS** (Human Services Data Specification) + **HSDA** (the API protocols).
  Michigan 211, Ontario Open211, and the UWW NDP export all speak (or map to) HSDS.
- **Build our ingester to consume HSDS.** Then "add a new city" = point it at that city's
  211 HSDS/NDP endpoint + run the **AIRS → Open Eligibility crosswalk** (taxonomy.md).
- **Open-source prior art to study/possibly reuse:** `211-Connect/open-resource-api`
  (a community-resource search back end); United Ways of California are building a "Social
  Care Data Exchange Gateway."

---

## Demo / prototype data — no agreement needed

You can build a working demo **before** any BC211/United Way agreement, on a 3-rung ladder
from zero-friction to real local data. Strategic point: show up to the United Way meeting
with **working software that already ingests 211 data**, not a concept.

1. **Build the plumbing today (zero signup):** the **Open Referral sample HSDS datapackage**
   (GitHub) — fictional but real-structured data, CSV + `datapackage.json`. Point the ingester
   at it immediately. *Caveat:* HSDS **1.1**, limited fields, confirm license in the repo files.
2. **Real provincial data, self-serve key:** **Ontario 211's public API portal**
   (`api211.portal.azure-api.net`). Ontario 211 was the first Open211/HSDS API; developers
   **self-register for a key and use the sample/public API** — *not* a formal data-sharing
   agreement. Gives real, HSDS-format, AIRS-classified data to exercise the crosswalk for
   real. (Full/production feed likely still needs partnership; verify portal terms.)
3. **BC-flavored, real, openly licensed:** the **HealthLinkBC** mental-health/substance-use
   dataset on Canada's open government portal — actual BC services, AIRS-classified, open
   licence. A Victoria-relevant slice with no agreement (subset; may not be full HSDS).

**Version heads-up:** sample is HSDS 1.1, Ontario may differ, build-1.5 targets 3.0 → make
the reader **version-tolerant** or run inputs through the Open Referral **HSDS Transformer**.

## Caveats for the City-in-a-box vision

- **Not uniform open data.** Each new instance's admin will likely need to **request access
  / sign a data-sharing agreement** with the local 211. → This becomes a step in
  [how-to-get-started-in-your-city.md](how-to-get-started-in-your-city.md).
- **Licensing vs. our open model.** 211 data is proprietary and access is read-only by
  agreement. Re-exposing it, letting it flow between instances, or our crowdsourced edits
  writing "back" to it may be **restricted**. The US NDP **5%-of-revenue** clause is a
  non-issue for a free instance but flags that "data valuation" terms exist — **review
  redistribution rights before depending on it.**
- **Coverage & quality vary** by city and by how active the local 211 is.
- **Ingested 211 data carries a verification cadence** — the AIRS baseline is *annual*
  re-verification (some centers more often). So a record can be months stale on volatile
  fields (hours, "open now"). Capture each record's last-verified date and let our
  scrape + crowd layer (§6) own volatile fields between 211 cycles. See landscape.md
  "How 211 keeps data fresh."
- **Always keep the scrape fallback.** Where there's no cooperative 211 (or for orgs/programs
  the 211 hasn't indexed), the build-1 search+scrape+LLM pipeline fills the gap. The two are
  complementary: **211 for breadth & classification, scrape for freshness & gaps,
  crowdsource for ground truth.**

---

## Open / to do

- Contact United Way BC / BC211 re: data-sharing agreement + API for Victoria.
- Confirm we may use 211 data in a **free, open-source, self-hostable** product, and what
  redistribution between instances is allowed.
- Prototype an **HSDS ingester** + **AIRS→Open Eligibility crosswalk** — specced in [build-1.5-hsds-ingester.md](build-1.5-hsds-ingester.md).
- Evaluate `211-Connect/open-resource-api` for reuse.

---

## Sources

- [Open Referral — HSDS / HSDA](https://openreferral.org/the-open-referral-api-project/)
- [Open Referral in Ontario (Open211, HSDS)](https://openreferral.org/open-referral-in-ontario-a-big-step-forward/)
- [United Way Worldwide National 211 Data Platform](https://openreferral.org/united-way-worldwides-national-211-data-platform-bringing-people-and-services-together/)
- [211 National Data Platform — FAQs (terms, APIs, fees)](https://register.211.org/Home/FAQs)
- [BC211 / United Way BC helpline services](https://uwbc.ca/helpline-services/)
- [211-Connect open-resource-api (GitHub)](https://github.com/211-Connect/open-resource-api)
