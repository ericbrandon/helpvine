# letstalkvictoria.com — Design Notes

*A natural-language assistant connecting people experiencing homelessness, and the wider community, to services and to each other in Victoria, BC.*

---

## 1. Vision

A single conversational surface where anyone can type plain language — "I need a place to store my papers," "I have size 10 shoes to give away," "where can I get a hot meal tonight?" — and get back a ranked stack of expandable **chips**: some pointing to services, some to people, some to actions. Behind it sits a continually maintained knowledge base of every service from every level of government, plus charities, churches, and community organizations, alongside a community exchange where people offer and request help directly.

Two guiding principles run through everything:

- **Anonymous read, identified write.** Finding help requires no account. Contributing, posting, messaging, or earning standing requires only a simple account (a username + password). Verifying a contact is optional — it boosts trust and enables recovery and notifications, but is never a gate to participating. Lookup is lifesaving and low-risk; the social/trust surface is where identity matters.
- **Dignity first.** The people the app serves are also its most valuable contributors. Asking for help is never a low-status act.

---

## 2. Design Principles

The canonical list of rules every other decision answers to. Sections 3–12 are where they play out in detail. Some principles here are newer than the sections below and may not yet be reflected everywhere.

**Foundational**
- **Dignity first** — asking for help is never a low-status act; the helped and the helper sit side by side. (Runs through everything; see §1, §7.)
- **Anonymous read, identified write** — finding help requires no account; contributing, posting, or messaging requires only a simple username + password account. Verifying a contact is an optional trust booster, never a gate. (§1, §8.)

**One conversational surface**
- **One surface, not modes** — every input is simultaneously a query and a write; there is no "directory mode" vs "exchange mode." (§3.)
- **Hard constraints are filters, not vibes** — open-now, no-ID, pets-OK, women-only and the like are structured filters; embeddings only handle the fuzzy part. (§5.)
- **Meet the whole person — match the intersection** — a person's situation is rarely one tag. The app extracts *every* relevant attribute (e.g. *elderly* **and** *at risk of eviction*), surfaces help for each axis, and especially ranks up organizations that serve the *specific intersection* (seniors' eviction-prevention), above generic single-axis matches. Inferred attributes are sensitive and follow the same consent/privacy rules as profiles. (§4, §5, §8.)

**Trust & data integrity**
- **Claims, not facts** — no single unverified message ever directly edits the database; incoming info is a weighed claim. (§6.)
- **Show confidence, don't fake certainty** — surface uncertainty and let the user judge; hiding it to look polished removes the safety net. (§6, §10.)
- **Provenance is always inspectable** — any fact can show where it came from and when it was last confirmed. (§6.)
- **Never hard-delete; version everything** — soft-delete with provenance, fully reversible, full audit trail. (§6.)
- **Confidence decays** — unconfirmed facts lose weight over time and re-prompt for verification, so stale data can't ossify. (§6.)
- **Karma tracks accuracy, not popularity** — standing is earned when reality later confirms a contribution, not from volume or applause. (§7.)
- **Work with agencies, not around them** — augment the orgs already doing the work; never position as their replacement. Concretely, a verified org owns and edits its own page (§8).
- **Authority is scoped** — power is confined to the smallest sensible boundary: a user acts as themselves, an org edits only its own page, a city admin/moderator acts only within their city; cross-city (super-admin) access is the rare, deliberate exception, never a default. (§8.)

**People & ethics**
- **Contact info stays private, always** — the pseudonym is the only public face; real phone/email is never exposed. (§8.)
- **Consent-gated people surfacing** — a person is only ever suggested as a people-chip with their consent. (§3, §12.)
- **Never coercive** — opt-in notifications, essential alerts that can still be declined, zero dark patterns. (§9.)

**Access & usability**
- **Mobile-first, responsive second** — most users are on a phone, often a modest one, so design every screen for the phone first. The layout is responsive so it also works well on tablet and desktop, but those are secondary targets, not the design baseline. Touch-sized targets, single-column flows, and low-bandwidth tolerance are the default.
- **Usability beats security** *(bedrock for this population)* — the app must be maximally usable by people who are elderly, homeless, in crisis, mentally ill, neurodivergent, or simply disorganized. When usability and security pull against each other, **choose usability.** Concretely: plain username + password, *any* password allowed (even "password"); no passkeys, no Google/third-party login, no email or SMS codes and no magic links to sign in; stay logged in indefinitely, logging out only on a deliberate tap. This is affordable because the app is a low-value target — no bank accounts, payments, or crypto, just people finding services — and because abuse is absorbed by the trust layer (claims-not-facts, karma-gated power, anomaly tripwires; §6–§7), not by hardening the front door. (Auth specifics in §8.)
- **Invisible security is mandatory** — "usability beats security" governs *tradeoffs*; it is never an excuse to skip security that costs the user nothing. Anything the user never feels gets done: hashed passwords, TLS everywhere, PII encrypted at rest, keys in a secret store, parameterized queries, output encoding, CSRF + security headers, rate limiting, patched dependencies, audit logs, and prompt-injection defenses. (Checklist in §10.)
- **Security scales with privilege** — ordinary users get the lowest-friction path; higher-power accounts (org editors, city admins, the parent-org super-admin) can justify more hardening (e.g. optional/required 2FA) because they can affect many people's data and are higher-value targets. Friction lands on the few who hold power, not the vulnerable many. (§8, §10.)

**Portability & operations**
- **City-in-a-box** *(bedrock)* — standing up a new city should be a guided afternoon, not a software project. A non-technical admin must be able to get hosting, get a URL, and wire them together in a few steps. The app ships as a single deployable artifact (likely a container) with a templated one-click deploy, and city-specific data and configuration are cleanly separated from code so the same codebase serves any city. This constrains architecture everywhere. Walkthrough: [how-to-get-started-in-your-city.md](how-to-get-started-in-your-city.md).
- **Every city is self-funded (bring your own key)** — each instance is financially independent: its own hosting account, its own bill, and above all **its own LLM API key** (the main usage cost; SMS/email keys too where used). There is no shared/central service the cities depend on to pay for resources. So setup must make obtaining and entering the LLM key *easy* — a guided wizard that links out to the provider's sign-up, accepts the key, and validates it with a test call before moving on. Keys are stored securely per-instance and never travel upstream.
- **Updates are one tap** — staying current is as easy as starting. Admins are notified when a new version is available and can update from the admin UI without touching a command line; updates are safe and reversible.
- **Degrades gracefully** — runs unattended with zero volunteers; one or two only raise throughput and relax thresholds. No single point of human failure. (§6.)

---

## 3. The core loop: one unified surface

There is no "directory mode" and "exchange mode." A single natural-language input fans out across every source and comes back as a mixed, ranked stack of chips.

**Every input is simultaneously a write and a query.** What differs is which pool it queries against. The chat's first job is **intent routing**:

| Input type | Example | Queries against | Typical result |
|---|---|---|---|
| Question | "Where can I get shoes?" | Service directory | Info chips |
| Offer | "I have size 10 shoes" | Existing needs + profiles | People chips ("DM these people") |
| Need | "I need size 10 shoes" | Offers + profiles **and** directory | Mixed: people chips + info chips |

The elegant case is a **need**, which can be satisfied two ways at once — a people-chip ("Dave two blocks over is giving away size 10s") sitting next to an info-chip ("Our Place hands out shoes Tuesdays, no ID needed"), ranked together.

### Chips
Expandable cards the user can open and keep talking to. At least three kinds:
- **Info chips** — service details (hours, location, eligibility, contact).
- **People chips** — a suggested person to DM (consent-gated; see §7).
- **Action chips** — call, directions, save, set a reminder.

---

## 4. Knowledge sources

Two sources with very different properties, both indexed the same way:

1. **Curated service directory** — semi-static, authoritative reference data with structure: services, eligibility, requirements, hours, contact. Accuracy is paramount.
2. **Community content** — dynamic, user-generated, often ephemeral: offers, needs, profiles, and forum threads. Trust and safety concerns dominate.

A third source emerged from the matching design:

3. **Profiles** — persistent attributes about a person (size 10 feet, has a dog, needs reading glasses), distinct from transient posts. Indexed and matchable like everything else, but only surfaced with the user's consent.

---

## 5. Processing pipeline & retrieval

For every chunk of every source: **ingest → chunk → generate tags + extract structured fields → embed → index.**

### Hybrid search is non-negotiable
Embeddings alone quietly fail on the constraints that matter most. Semantic search is great for "something about food" but bad at hard constraints:

- open right now
- no ID required
- women only / under 25 / families only
- accepts pets
- within walking distance
- beds left tonight
- referral needed

These are make-or-break facts for someone in crisis and exactly what vector search handles worst. So the system combines **structured fields as filters** (the purpose of the tagging layer) with **embeddings for the fuzzy part**. This is the difference between an app that's charming and one that doesn't send someone across town to a locked door.

### Intersectional matching & ranking
A person's situation is rarely a single category, and the most useful help is often the org built for the *specific combination*. So:

- **Multi-tag the situation.** Parse an input (or profile) into *all* its relevant axes, not one — "I'm 74 and my landlord is evicting me" → `elderly` **+** `at-risk-of-losing-housing` (+ any others, e.g. `fixed-income`, `mobility`).
- **Answer every axis.** Surface help for each tag on its own (general eviction-prevention; general seniors' services), so nothing is dropped.
- **Reward the intersection in ranking.** A listing that matches *multiple* of the user's axes ranks above one matching only a single axis — an org that does **seniors' eviction prevention** outranks a generic tenancy hotline *and* a generic seniors' centre. The more of the person's situation a service covers, the higher it sits.
- **Tag listings on the same axes** so intersections can actually be computed — orgs carry the populations and needs they serve as structured tags, matched against the user's.
- **Privacy & consent.** Inferred attributes (age, housing risk, disability, status) are sensitive; they follow profile consent rules (§4) and the privacy posture in §8 — used to help, never to expose or profile a user publicly.

### Matching engine
Matching is symmetric and has two timings:
- **Immediate** — at post time, search existing counterparts and show chips now.
- **Standing** — the post persists and pings the user later when a new counterpart appears ("someone just posted size 10 shoes"). This is delivered through the notification layer (§9).

---

## 6. Data freshness — "Tell the app something it should know"

The hardest problem in the whole system. Hours change, programs end, a soup kitchen moves; stale info is worse than none. The goal is **near-unattended** operation.

### The core reframe: claims, not facts
A single unverified message must never directly edit the database — that turns the app into a vector for poisoning (marking a real shelter "permanently closed," injecting a fake payment scam). Instead, incoming info is a **claim** that the system weighs. Every record carries **trust metadata**: who said it, when last confirmed, a confidence score, and a status (active / proposed / disputed / removed).

### The pipeline (mostly automated)
1. **Parse** — AI turns a freeform message into a structured proposed edit (which org, which field, old → new).
2. **Match** — does it corroborate or conflict with existing data?
3. **Weigh** — score by source tier and corroboration. One anonymous report = weak; three independent reports in a day, or one verified org admin = strong.
4. **Verify** (automatable) — re-fetch the org's official page, or ping the org's verified contact for a one-tap confirm.
5. **Apply or label** — high confidence applies; low/conflicting is marked "disputed" and both versions are held.

### Field tiers — what flows freely vs. what's gated
Sorted by *harm if wrong* × *reversibility/verifiability*:

**Tier 1 — Open (flows freely, additive, low harm)**
- Adding a new listing (starts low-confidence until corroborated)
- Adding a service to an existing org
- Notes, tips, descriptions (still pass the safety classifier)
- Suggested tags

**Tier 2 — Confirm before replace (accept, but label until backed up)**
- Hours
- "Open/closed right now" — a fast-decaying labeled signal shown beside official hours, never a permanent overwrite
- Eligibility / requirements
- Removing a service

**Tier 3 — Hard-gated (needs high-tier source or strong corroboration)**
- Address / location
- Contact info (a classic scam swap)
- Payment / financial details (only a verified org owner should ever touch these)
- Deleting a listing or marking it **permanently closed** (never from a single report — the favourite vandalism move)

### Safety backstops for unattended operation
- **The end user is the real human in the loop.** The app shows confidence and lets the user judge ("3 people say Wednesday; official page says Tuesday — here's both") rather than silently picking a winner.
- **Hard-block list** — a safety classifier refuses to auto-publish certain things regardless of confidence: money/e-transfer requests, anything steering someone somewhere unsafe.
- **Anomaly tripwires** — bursts of edits from one account, waves of "closed" reports, injected links → auto-quarantine and escalate.
- **Never hard-delete; version everything** — soft-delete with provenance, fully reversible, full audit trail, auto-revert when contradicted by a re-confirmed official source.
- **Confidence decays over time** so stale auto-applied data doesn't ossify; a fact unconfirmed for months re-prompts for verification.

### Volunteers (optional, 1–2)
The system runs unattended by default; volunteers raise throughput and let thresholds relax. They plug in at three points:
1. Clearing the anomaly/quarantine queue (~1% of traffic).
2. Unsticking middle-ground edits that have some but not enough corroboration.
3. Granting trust — verifying org ownership, promoting reliable contributors.

Crucially, the design **degrades gracefully**: with zero volunteers it still works; with two it works faster.

---

## 7. Trust & reputation (karma)

Karma is both the motivator *and* the trust ladder — the same number that makes the game fun decides how much the system trusts an edit unattended.

**Anchor rule: karma tracks accuracy, not popularity.** It is earned mostly from contributions that **later get confirmed true**, not from posting volume or applause. The lag kills drive-by gaming — you can't cash out until reality backs you up.

- **Earns karma:** an edit later corroborated/officially confirmed; a verified new listing; a "closed now" report matching others; a confirmed successful match in the exchange.
- **Loses karma:** an edit contradicted by an official source or reverted; flagged content. Asymmetric — easy to lose, slow to earn — with bigger penalties for high-impact (Tier 3) fields.

**Karma maps to power** — it is literally a dial on the corroboration thresholds:

| Standing | Effect |
|---|---|
| New / anonymous | Tier-1 flows; Tier-2/3 are weak signals needing heavy corroboration |
| Some karma | Edits count for more; fewer confirmations needed; reports weighted higher |
| High karma | Edits auto-apply at lower corroboration; may reach a Tier-3 field |
| Top / human-vouched | Effectively a trusted contributor, near org-level for general fields |

The top rung **automates** the trusted-contributor status volunteers would otherwise grant by hand, so the system leans less on both gates and volunteers as it matures.

**Anti-gaming:** karma decays with inactivity (a dormant compromised account isn't a super-weapon); diminishing returns so volume can't dominate; new accounts climb slowly so trust can't be minted by spawning accounts; karma is only *one* input to confidence and never overrides a verified org or official source.

**Dignity:** people experiencing homelessness are the best real-time sensors — they're the ones at the locked door at 8pm. Gamifying accurate ground-truth reporting turns them into the most valuable contributors and earns them standing. Keep the game on the contribution side; "only ever asked for help" must never read as low status.

---

## 8. Identity & accounts

- **Pseudonymous ID is the stable identity.** A username (the pseudonym) + password is the login. Email and/or phone are *optional* additions attached to the ID — for notifications, recovery, and trust — and can change without losing who you are.
- **Dead-simple auth (usability beats security).** Any password is accepted — no complexity rules, no forced rotation; "password" is fine. No passkeys, no Google/third-party login, no email or SMS codes, no magic links. Sessions are long-lived: a user stays logged in until they deliberately tap "log out." *The one real caution* is the shared/public device — a long session there exposes the account to the next person — so the answer is a prominent, dignified "this isn't my device — log out," **not** idle timeouts that would punish the majority on personal phones. (See the §2 principle; relates to shared-phone reality below and discreet-notification cautions in §9.)
- **Verification is an optional trust booster, never a gate or a login step.** A user who confirms a phone or email once (not repeated at sign-in) has their contributions weighted higher — phone-verified outranks email-only (email is free and infinite; phone numbers are costlier to acquire in bulk); org-vouched and org-owner sit higher still. Skipping verification means lower trust/power, not exclusion.
- **Anonymous read, identified write** (see §1) resolves the inclusion tension — contributing needs only a username+password account, not contact info. The anti-abuse burden sits on the trust layer (claims-not-facts, karma-gated power, anomaly tripwires; §6–§7), **not** on a strong identity gate — which is exactly what lets the front door stay this easy.
- **Contact info stays private, always.** Pseudonym is the only public face; DMs route through it and never expose the real phone/email. Store PII minimally and securely — a phone tied to someone's posts could be used to locate them.
- **Plan for churn & recovery.** This population changes numbers often; encourage adding both email and phone, and let outreach workers assist with setup/recovery.
- **Don't assume one phone = one person.** Shared shelter phones and families on one number are common; tolerate a reused contact with mild friction rather than a hard block.

### Roles & special users
Beyond anonymous readers and ordinary registered users, several privileged roles exist. Authority is **scoped** (§2) — each role acts only within its boundary.

| Role | Scope | What they can do |
|---|---|---|
| **Anonymous reader** | — | Search and read; no account. |
| **Registered user** | Self | Contribute, post, message, earn karma (username + password). |
| **Org** (verified owner/editor) | Their own org page | View and edit everything the app knows about their organization, including gated Tier-3 fields for *their own* listing; broadcast to followers (§9). The concrete form of "work with agencies, not around them." |
| **City moderator** (city-scoped superuser, moderator powers) | One city | Clear the anomaly/quarantine queue, resolve disputed edits, action reports/abuse, grant trust, hide content. Formalizes the optional "volunteers" of §6. |
| **City admin** | One city | Full superuser *for that city*: everything a moderator can do, plus configure the instance (branding, keys, data sources), verify orgs, and manage roles/moderators. |
| **Super-admin** (parent org) | One account per instance, on every city | A superuser account the **installer provisions at setup time** on every deployment, held by the parent organization. Lets the parent org comply with authorities/legal requests and intervene in emergencies *without* depending on the local admin to act. Mechanically just a local account per instance — no central service. |

**How the cross-city super-admin works (resolved: a per-instance account).** Each city is a separate, self-funded deployment with no central service it depends on (§2 "City-in-a-box," "self-funded") — so there is no global super-admin *service*. Instead, the installer provisions a **parent-organization super-admin account on each instance at install time.** It is cross-city *in effect* (the parent org can act in any city) but mechanically just one standing local account per deployment, so the no-central-service principle holds.

*Primary use case:* authorities or a legal process contact the parent organization for information or a change, and the parent org must be able to comply even if the local city admin is unavailable or unwilling to act.

*Safeguards to bake in,* since this is standing parent-org access to every instance: keep it least-privilege, log every super-admin action to the audit trail (§6 versioning/provenance), and make its existence transparent to local admins rather than a hidden backdoor.

---

## 9. Notifications & reminders

Not a separate feature — the **delivery layer** several pieces already depend on (standing matches, edit-confirmation loops). One unified system, three kinds of producer:

- **App-generated** (events): you have a DM; a match fired; your edit was confirmed (+karma); something you reported needs confirming.
- **Org-generated** (broadcast): "dinner cancelled tonight." Publish/subscribe — users *follow* orgs or services; orgs don't blast everyone — and rate-limited, since broadcast is a spam vector.
- **User-requested** (two flavours): recurring time reminders ("renew paperwork every April") and natural-language *watches* ("notify me when the bottle drive is"). Watches reuse the core engine — the AI parses the request into a condition matched against incoming claims/posts.

**Channels:** email, SMS, web/app push — granular per-type, per-channel opt-in.

**Cautions to bake in:**
- SMS costs money and intrudes → reserve for urgent/time-sensitive items (bed available, dinner cancelled); use email/push for the rest.
- Digests, quiet hours, frequency caps, one-tap unsubscribe (STOP for SMS) to avoid the notification fatigue that kills these apps.
- Discreet previews — no sensitive content on a lockscreen or shared device.
- Consider an "essential alerts" class (e.g. extreme-cold warnings) defaulted on but never coercive.

---

## 10. Cross-cutting safety & ethics

- **Scam vectors** concentrate in contact info and payment details (§6) — gated hardest.
- **Privacy** for a vulnerable population — minimal PII, pseudonymous surfaces, discreet notifications.
- **Residual risk is accepted honestly.** Near-unattended operation means some wrong data will occasionally reach users before the system catches it. The thing that keeps that from being dangerous is confidence-labeling — users see "unconfirmed" rather than trusting blindly. Hiding uncertainty to look polished would remove the safety net.

### Security hygiene (the invisible kind)
Everything here is invisible to the user, so per the §2 "invisible security is mandatory" principle, it's all in scope. None of it conflicts with "usability beats security" — that principle only governs choices the user would actually *feel*.

**Credentials & sessions**
- Store passwords as salted hashes (argon2/bcrypt/scrypt), never plaintext.
- Brute-force defense via rate limiting + slow hashing, **not** account lockout (lockout hurts UX and lets an attacker lock a user out of their own account).
- Secure cookies (httpOnly, secure, SameSite) and **server-side revocable sessions** — so even with indefinitely long sessions (§8), a lost or shared device can be logged out remotely.

**Transport & storage**
- TLS/HTTPS everywhere (HSTS) — nearly free from the hosting platform.
- Encrypt PII (phone/email) at rest, and store the minimum (§8) — less data is less to leak.
- Keep API keys (especially the LLM key) in a secret store, never in code, repo, or logs; scrub secrets and PII from logs.

**App surface**
- Parameterized queries / ORM (no SQL injection), output-encoding (no XSS), CSRF protection, and security headers — CSP especially, since LLM output is rendered.
- Rate-limit auth, posting, and API endpoints — doubles as part of the §6 anomaly tripwires.

**LLM-specific** (this is an AI app)
- Treat all retrieved/community/scraped content as **data, not instructions** — guard against prompt injection, since untrusted claims and org pages flow into the model's context. Validate model output before it acts; the §6 safety classifier / hard-block list is part of this.

**Operations**
- Patch dependencies (automated scanning); ship security fixes through the "updates are one tap" channel (§2).
- Encrypted backups; audit-log privileged actions (§8 super-admin).

**Privileged accounts** (per "security scales with privilege," §2)
- Offer optional 2FA to org editors and city admins; consider **requiring** it for the parent-org super-admin given its standing cross-city power.

---

## 11. Suggested sequencing

1. **Directory + chat retrieval** first — a data-quality problem, low safety risk. Nail hybrid search and the "tell the app something" freshness pipeline.
2. **Community exchange + DMs** next — matching plus the harder safety/moderation work, once there's trust and moderation muscle.
3. **Notifications** layered across both as the delivery backbone.

> **First build:** the city-admin control panel and its "add organizations, programs, and services" ingestion box — full spec in [build-1-org-ingestion.md](build-1-org-ingestion.md).

---

## 12. Still to design

- **DM / connection safety layer** — pseudonymous messaging between strangers, one party often vulnerable: reporting, blocking, abuse detection, consent for people-chip surfacing. *(Next conversation.)*
- Data model / schema details (records, claims, trust metadata, karma ledger).
- Ranking algorithm for mixed chip stacks — including how much to boost intersectional matches (multi-axis overlap) over single-axis ones (§5).
- Org onboarding and listing-claim flow.
- Natural-language watch parsing and matching specifics.
- Success metrics (did people actually get helped?).
- **Multi-city deployment & update mechanism** — packaging, one-click deploy, config/data separation, admin update flow (see "City-in-a-box" and "Updates are one tap" in §2; walkthrough in how-to-get-started-in-your-city.md).
