# Build 1 — Admin control panel: organization & service ingestion

*The first thing we build. It's the concrete realization of "directory + freshness
pipeline first" from [helpvine-design.md](helpvine-design.md) §11, and it lives
inside the city admin's control panel.*

> **Companion:** [build-1.5-hsds-ingester.md](build-1.5-hsds-ingester.md) seeds the
> directory from a city's 211 (HSDS) and shares this build's extraction → chips → index
> backend. Division of labor: **211 for breadth + classification, this scrape pipeline for
> gaps + freshness, crowdsource for ground truth.**

> **LLM choice:** initially built on **Gemini** (this pipeline uses **Gemini Flash**
> — noted as "Gemini Flash 3.5"). We'll try other models later as we refine.
> Confirm the exact model id + current pricing against Google's docs before building.

---

## Where it fits the design principles

- This is **admin-seeded** directory data — an authoritative source, not an anonymous claim. So it skips the "claims, not facts" weighing (§6) and lands as high-confidence data, but it is **still versioned and provenance-stamped** ("scraped from <url> on <date>," "admin correction on <date>"). Never hard-delete (§6).
- It's **per-instance** (§2 City-in-a-box): the scrape/search/extract jobs run on the city's own deployment using the city's own Gemini key (§2 self-funded). The 7-day re-scrape is that instance's own scheduled job — no central service.
- The eligibility/client flags produced here **are the intersection axes** from §2 "meet the whole person" — see *Taxonomies* below.

---

## The control panel (first surface)

A city-admin-only web UI (mobile-first still applies, but admins are likely on a
laptop — desktop is fine here). First feature inside it: the ingestion box.

---

## Feature: "Add organizations, programs, and services"

A plain-text input box. **Indentation encodes hierarchy:**
- **Unindented line** → a top-level item (usually an organization).
- **Indented line** → a program, location, or sub-part of the unindented line above it.
- Lines may contain **phone numbers, URLs, or addresses** inline.
- Sub-items may also appear **in parentheses** on one line, e.g.
  `The Salvation Army (Connection Point, Langford, ARC – addiction rehabilitation centre)`
  — the parser must handle both indentation *and* parenthetical enumeration.

Example input (verbatim from design discussion):

```
Our Place
	- Our Place Society Day Labour Pool
	- Our Place food pantry
	- Our Place street outreach team
Cool Aid
	- Cool Aid mobile health clinic
	- Living Edge mobile markets
Pacifica Housing / BC Housing
Victoria Homeless Coalition
Heaterblocvicbc
Foodnotbombs_vic
YWCA
The Salvation Army (Connection Point, Langford, ARC – addiction rehabilitation centre)
Victoria Native Friendship Centre (VNFC)
Anawin Companion Society (Anawin House)
Mustard Seed
	- Essential Goods Program
```

### Processing loop (one line at a time)

For each parsed item:

**1. Find it on the web → URL + contacts.**
Goal: at minimum a URL; ideally phone, email, address too.
**Recommendation:** use **Gemini Flash with Google Search grounding** — one call per
item ("find the official website, phone, email, and address for `<item>` in `<city>`"),
returning structured fields. This reuses the city's single Gemini key (no extra search
API key — aligns with §2 self-funded/BYO-key). *Fallback if grounding quality is weak:*
a dedicated search API (Brave Search / Tavily / SerpAPI), accepting the extra key.
Ambiguous names (e.g. "YWCA") are disambiguated by the city context and caught in admin
review.

**2. Scrape the site → build a same-domain page tree.**
**Recommendation:** tiered, not Playwright-for-everything:
- Default: a plain HTTP fetch (`httpx`/`requests`) + an HTML parser (BeautifulSoup).
  Most charity/nonprofit sites are static or WordPress — this is fast and cheap.
- Escalate to **Playwright (headless browser)** *only* when a page returns an empty/JS-shell
  body (client-rendered SPA). Detect and fall back per-page.
- Crawl rules: **same-domain links only** (external links may be *used* but don't extend
  the tree); **respect robots.txt**; set a descriptive User-Agent; rate-limit politely;
  **cap depth and page count** to avoid runaway crawls on huge sites.

**3. Hash every page; download documents.**
- Store a content hash per page (SHA-256 of **normalized text**, not raw HTML — strip
  boilerplate/timestamps/ads so re-scrapes don't false-positive on noise).
- Download all downloadable docs (PDFs etc.) and extract their text.

**4. Classify the item.**
An LLM call decides: organization with one program / organization with multiple programs
/ a program / **other (categories TBD)**.

**5. LLM structured extraction.**
**Recommendation:** Gemini structured output (a JSON `responseSchema`) so the model
returns validated fields, not free text. Run extraction over the scraped tree + docs.
Likely only a subset of pages are extraction-relevant — classify pages first to control
cost rather than full-extracting every page.

Extract for the **organization**:
- Name
- Websites
- Phone numbers
- Emails
- Physical addresses (including multiple locations)
- Hours, per day of week
- Mission
- Services provided — with **on/off flags per service type** (legal, food, clothing, guidance, …)
- Programs offered — with **on/off flags per program type**
- Eligibility & requirements — with **on/off flags per client type**
- Does the site list events / have a calendar?

Then, for **each program / location**, the *same field set* (name, websites, phones,
emails, addresses, hours, mission, service flags, program flags, eligibility flags,
events/calendar?).

### The org page (spreadsheet-like UI)

The extracted structured data is displayed on the org's page in a spreadsheet-like grid.

Below it:
1. A **plain-text corrections box** — the admin (or, later, the verified org) types
   corrections in natural language: *"We're actually not open on Wednesdays,"*
   *"We no longer offer that program."*
2. Below that, a **diff grid** showing the items that *changed* after an LLM applies the
   corrections — review-before-commit. Corrections are provenance-stamped as a
   high-trust admin/org override.

### Chips

An LLM generates **chips** for the org/programs/locations:
- A **one-sentence chip** — the short response shown to a user querying the app (the
  info-chips of §3).
- A pre-written **~paragraph description** behind each chip, plus contact info.
- On the org's page, the admin/org sees **all** chips describing their services and can
  click any to read its full paragraph.

> Open: chips + structured fields are what actually get embedded and indexed for the
> user-facing hybrid search (§5). Decide chip granularity (per service? per program?) and
> wire chip generation into the indexing step.

---

## Freshness: 7-day re-scrape

Every 7 days the instance re-scrapes each org's site and compares page hashes. On a
change, it diffs old vs. new and updates the data. This is the **automated "re-fetch the
official page" verify step** of §6, running per-instance on a schedule (hosting cron /
background worker).
- Changes to **gated fields** (address, contact, payment — §6 Tier 3) should still flag
  for admin review even from the official site, since a diff there could mean the site was
  edited maliciously or compromised.

---

## Taxonomies (controlled vocabularies)

The on/off flags only work against fixed category lists, shared across all orgs and matched
against user queries. **These are specified in [taxonomy.md](taxonomy.md)** — a v0 built on
the **Open Eligibility** standard (Human Services + Human Situations), curated and extended
for street homelessness in Victoria (storage, showers/laundry, harm reduction, mail/address,
pets-as-a-situation, Indigenous localization).

Two mirrored axes:
- **Axis A — Service & Program types** (what's offered).
- **Axis B — Client / Situation types** (who it's for) — **these ARE the intersection axes**
  from §2 "meet the whole person." The same vocabulary tags listings *and* user situations,
  so an *elderly + at-risk-of-eviction* user matches an org flagged for both. See taxonomy.md.

---

## Cost optimization

Keeping per-city LLM spend low matters (§2 self-funded) — every city pays its own bill.

- **Cache the content once, ask many questions against it.** A single item triggers
  several LLM calls over the *same* scraped site — classify, org-level extraction,
  per-program/location extraction, chip generation. Upload/cache the scraped pages + docs
  **once** (Gemini context caching) and run each subsequent prompt against the cached
  context, so the large input is paid for once instead of re-sent per call.
- **Real-time in dev, batch in production.** Interactive ingestion while developing or
  while an admin is actively adding orgs needs to be real-time (standard calls). But the
  scheduled 7-day re-scrape, bulk re-extraction, and large initial imports are **not
  latency-sensitive** → run them through Gemini's **batch mode** for the discount. Let
  these run overnight.

## Open decisions / TBD

- Exact Gemini model id + pricing; cost ceiling per ingest (search grounding + per-page
  LLM calls add up on large sites — cap pages, extract only relevant ones).
- Classification "other" categories.
- Chip granularity and how chips feed the retrieval index (§5).
- Taxonomy v0 drafted in [taxonomy.md](taxonomy.md); remaining: Open Eligibility license, per-category hard-filter-vs-soft-boost, Axis A↔B mapping table.
- Deduplication/reconciliation: an item may already exist, and indentation-declared
  programs may overlap with programs discovered by scraping — how to merge.
- Robots.txt / ToS edge cases; crawl depth + page caps; PDF parsing quality.
- Hash normalization strategy (what to strip so diffs are meaningful).
- Error/empty-result handling when web search finds nothing or the wrong entity.
