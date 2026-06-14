# Landscape — prior art & differentiation

*Honest answer to "is someone already building this?" Short version: the **directory**
half is solved at scale by others; the **community-exchange + crowdsourced-dignity-freshness
+ free self-hostable + AI-native** half is not. Don't reinvent the directory — ingest it,
and spend our novel energy on the part nobody else does.*

---

## ⚠️ Two unrelated orgs both called "findhelp"

- **findhelp.com / findhelp.org** — *formerly Aunt Bertha.* US company (Austin, TX),
  founded 2010 by Erine Gray. Created the **Open Eligibility** taxonomy. This is the one
  that resembles our directory.
- **findhelp.ca / Findhelp Information Services** — a *separate* Toronto nonprofit that
  operates **211 Ontario**. Same name, different org, different country. Don't conflate.

Below, "findhelp" = the US Aunt Bertha company unless stated.

---

## findhelp (Aunt Bertha, US)

**What it is:** the leading US "social care network" — a searchable directory of free/
reduced-cost services (food, housing, goods, transit, health, money, care, education, work,
legal — *the same domains as us*). 552k+ human-verified program locations, 10M+ users.

**Business model:** **B2B SaaS.** They sell the platform to **health systems, governments,
payers, employers** to manage social-determinants-of-health (SDoH) referrals and track
outcomes. End-user search is free; the money is institutional. **US-only.**

**Strengths we should respect:** huge verified directory, the Open Eligibility taxonomy,
eligibility filtering, real outcome-tracking, dignity-oriented mission.

**What it is *not* (our whitespace):**
- **Not peer-to-peer.** It connects people to *organizations*, not to *each other*. There's
  no "I have size 10 shoes" person-to-person exchange — our entire **Connect** pillar.
- **Institution-centered.** The workflow assumes a caseworker/hospital refers a client.
  Ours is individual- and community-centered.
- **Not crowdsourced by the people served.** "Human-verified" means staff/process, not the
  homeless person reporting the locked door at 8pm and earning standing. No karma / dignity
  contributor ladder.
- **B2B, proprietary, US-only** — not free, open-source, self-hostable per city.
- **Search/filter + referral**, not a conversational AI-native single surface. *(They may be
  adding AI — verify; our bet is AI-native from the ground up.)*

## 211 / BC211 (the Canadian directory incumbent)

- The **211** helpline + online directory covers Canada; **BC211** covers Victoria, indexed
  with the **AIRS/211 taxonomy** by trained specialists (see [taxonomy.md](taxonomy.md)).
- This is the closest local analog for our **directory** piece — authoritative, professional,
  already exists for Victoria.
- Same gap as findhelp: it's a *directory/referral* service, **not** a community exchange,
  not crowdsourced from the served population, not AI-native, not self-hostable.

---

## How 211 keeps data fresh (and where it leaves a gap)

211's freshness model is **professional, manual, and periodic**:
- Paid **Resource Specialists contact every agency directly** to verify records.
- The **AIRS standard is annual re-verification** (100% within ~12 months baseline).
- Plus **agency self-update** portals and a **caller-feedback** loop (a caller reports a
  closed program → a specialist follows up → corrects it). Cadence ranges daily→annual by
  center; the US National Data Platform recommends a **weekly minimum**.

**The gap:** it's authoritative but **lagging and labor-bound** — a change in February may
not be caught until the annual review unless someone calls it in, and capacity is gated by
paid staff. The caller-feedback loop is a **mediated, un-gamified cousin** of our
crowdsourcing: a human relays it, a specialist verifies, nothing flows back to the reporter.

**helpvine's §6 bet against this:** ingest 211's verified baseline as high-confidence seed,
then layer a **continuous, direct, gamified crowd + automated scrape** loop that catches the
drift *between* 211's cycles (the 8pm locked door). **Confidence-decay (§6) is the automated,
per-record analog of their annual re-verification.** Bonus: since 211 already uses caller
feedback, our verified crowd-reports could be **valuable back to them** — a two-way value
prop for partnership (subject to redistribution terms).

## The gap = our opportunity

What every incumbent stops short of, and what we're actually for:

1. **Community peer exchange** — people helping people directly (offers ↔ needs ↔ profiles),
   consent-gated DMs. *No incumbent does this.*
2. **Crowdsourced, gamified, dignity-centered freshness** — the people served are the
   real-time sensors; accurate ground-truth reporting earns karma/standing. *No incumbent
   does this.*
3. **Free, open-source, self-hostable, per-city** — "City-in-a-box," community-owned, not
   institutional SaaS. *No incumbent does this.*
4. **AI-native single conversational surface** — plain language in, mixed ranked chips out.
   *Incumbents are search/filter + referral workflows.*

## Strategic implication

- **Don't out-build the directory.** findhelp and BC211 have spent a decade + professional
  staff on it. Competing head-on there is a losing battle and violates "work with agencies,
  not around them" (§2).
- **Ingest, then layer.** Treat BC211 / Open Eligibility as *sources* (seed + crosswalk,
  per taxonomy.md), and put our novel energy into the **Connect** pillar, the
  **crowdsourced-freshness** loop, the **dignity/karma** model, and the **conversational
  surface** — the things that don't exist yet.
- This reframes the directory work (build-1) as *"get to parity cheaply by reusing existing
  classified data,"* not *"build a better directory than 211."*

---

## To verify / open

- Does findhelp (US) now offer an AI/conversational layer? (Affects how novel "AI-native" is.)
- BC211 data availability for ingest (the strong lead from taxonomy.md).
- Any existing peer-to-peer mutual-aid app in the homelessness space we should study
  (e.g. mutual-aid networks, Trash Nothing/Buy-Nothing-style for needs) — scan before building Connect.

---

## Sources

- [findhelp (Aunt Bertha) — about / 10M users](https://company.findhelp.com/about/)
- [Aunt Bertha → findhelp rebrand](https://company.findhelp.com/blog/2021/11/21/aunt-bertha-is-now-findhelp/)
- [findhelp for organizations (B2B/SDoH)](https://company.findhelp.com/solutions/)
- [findhelp.ca — separate Ontario 211 org](https://findhelp.ca/about-us/)
