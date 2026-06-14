# How to get started in your city

*A step-by-step walkthrough for launching this app in a new city. Written for a
motivated admin who is **not** necessarily a software developer.*

> **Status: DRAFT / aspirational.** This file describes the experience we're
> designing toward (the "City-in-a-box" and "Updates are one tap" principles in
> [helpvine-design.md](helpvine-design.md) §2). Steps below are a target, not yet
> a tested procedure. We'll tighten it as the deployment path is built.

---

## What you'll end up with

A live website at your own address (e.g. `letstalk<yourcity>.com`) running the
full app — chat, directory, community exchange, notifications — seeded with your
city's services and branding, that you can keep up to date yourself.

## What you'll need before you start

- About an afternoon.
- A credit card (for hosting + the AI and SMS services the app uses — small, usage-based).
- An email address to be the admin.
- *(Optional)* A domain name, if you want a custom address rather than the free one hosting gives you.

---

## The steps (target experience)

> Design goal: each numbered step is a few clicks, not a config file. Where a
> decision is still open, it's marked **[TBD]**.

1. **Get hosting + a URL, wired together in one click.**
   The plan is a "Deploy" button that hands off to a managed hosting platform
   **[TBD: Render / Railway / Fly.io / DigitalOcean App Platform]**. You log in,
   confirm, and the platform provisions the app *and* gives you a working URL
   automatically (e.g. `yourcity.onrender.com`). No servers to manage.

2. **Add your own LLM API key.** *(The main one — your city pays its own AI bill.)*
   The setup wizard links you to the provider's sign-up, you paste your key, and
   it runs a test call to confirm it works before continuing. **[TBD: default
   provider + model.]** This is the one cost every instance must cover itself —
   there is no shared/central key.
   - *(If you enable text/email notifications)* add an SMS key **[TBD: Twilio?]**
     and email sending **[TBD]** the same way. Optional to start.

3. **Tell the app about your city.**
   A first-run setup screen captures the city-specific config that's kept
   separate from the code:
   - City name, region/bounding box (for "within walking distance").
   - Branding: name, logo, colors.
   - Admin account (your email).
   - Local defaults: timezone, languages, emergency/essential-alert sources.

4. **Seed the service directory.**
   Get your city's services in. The preferred path is to **ingest your local 211's
   data** (most cities have one, run by a United Way or affiliate, already classified):
   - **Request access** to your local 211 / United Way data (API or data-sharing
     agreement) — in the US via the 211 National Data Platform, in Canada via your
     provincial 211. The app ingests it through the **Open Referral HSDS** standard.
   - **Then scrape the gaps** with the build-1 search+scrape pipeline for anything the
     211 hasn't indexed. See [directory-data-sources.md](directory-data-sources.md).
   The freshness pipeline keeps it current after launch (design.md §6).
   **[TBD: how much of this can be one-click vs. requires the admin to sign an agreement.]**

5. **(Optional) Point your own domain at it.**
   If you bought a domain, add it in the hosting dashboard and follow the DNS
   instructions. Skippable — the free URL works fine to start.

6. **Go live & invite your community.**
   Share the URL. Recruit one or two optional volunteers if you have them (the
   app runs unattended without them — design.md §6).

---

## Keeping it up to date

Per the **"Updates are one tap"** principle:

- When a new version is published upstream, you get a notification (in the admin
  UI and/or by email).
- Updating is a single confirm from the admin UI — no command line.
- Updates are safe and reversible **[TBD: rollback mechanism]**.

---

## Open questions to resolve as we build

- Which hosting platform gives the smoothest non-developer "click → hosted + URL" path?
- How is city config + data packaged so one codebase serves any city cleanly?
- Container image? Deploy template format (`render.yaml`, Railway template, etc.)?
- How does the update notification reach admins, and how does one-click update + rollback work?
- What's the cheapest viable footprint so a small city can afford to run it?
- Directory seeding: import template vs. assisted scraping vs. manual.
