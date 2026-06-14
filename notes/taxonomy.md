# Taxonomies — controlled vocabularies

*The fixed category lists that power the on/off flags from
[build-1-org-ingestion.md](build-1-org-ingestion.md) and the intersectional matching
of [helpvine-design.md](helpvine-design.md) §2/§5.*

> **Status:** v0. These will grow after we scrape real sites. Treat the lists as a
> starting controlled vocabulary, plus a sanctioned path for *proposing* new tags
> (a §6 Tier-1, additive, low-harm edit) so the vocabulary expands without chaos.

---

## Why two axes

Two parallel vocabularies, deliberately mirrored:

- **Axis A — Service & Program types** ("what is offered"): tag on every org/program/listing.
- **Axis B — Client / Situation types** ("who it's for" / "who the person is"): tag on
  listings (eligibility) **and** on user situations.

**Axis B *is* the set of intersection axes** from principle §2 "meet the whole person."
The whole intersectional-matching idea only works if the *same* Axis-B vocabulary is used
to tag listings and to tag people — so "elderly + at-risk-of-eviction" can match an org
flagged for both. Keep them one list, never two that drift.

## Backbone: Open Eligibility — and why, vs. AIRS/211

There are two standards. We use **Open Eligibility** as the product backbone but
**crosswalk to AIRS/211** to ingest existing data. The distinction matters:

| | **AIRS/211 Taxonomy** | **Open Eligibility** |
|---|---|---|
| What | The North American I&R standard; 9,000+ terms, deeply hierarchical | A simple, consumer-facing taxonomy (Human Services + Human Situations) |
| Maintained by | 211 LA County / Inform USA | findhelp (Aunt Bertha), on GitHub |
| Canada | **Canadian version, EN + FR; BC211 classifies Victoria services with it** | Mostly US / findhelp ecosystem; little local awareness |
| Who classifies | **Trained 211 resource specialists** — *not* the orgs themselves | Simple enough for orgs/laypeople to self-select |
| License | **Proprietary, paid subscription** (~$200–650/yr); restricted | **CC BY-SA 3.0 — free, redistributable** with attribution + share-alike |
| Fit for us | Source-data lingua franca; too heavy for our UI | Our org-facing UI and user-situation flags |

**Key reality:** *neither* standard means "every org already knows which boxes it checks."
AIRS is expert-indexed by 211 staff; Open Eligibility is barely known here. So the
build-1 **LLM classification is still required** — we are automating what a 211 indexer does.

**Decisions:**
- **Backbone = Open Eligibility** (free CC BY-SA 3.0 license — resolves the earlier license
  question; the "Proprietary" stamp was on findhelp's *handout*, not the taxonomy). It's
  simple enough for the org-facing spreadsheet UI and for tagging user situations.
- **Crosswalk to AIRS/211** so we can ingest 211-classified data. **Pursue BC211 as a seed
  source and partner** — they already maintain an AIRS-classified directory of Victoria-area
  services. This is "work with agencies, not around them" (§2) and could skip cold-scraping
  for already-classified orgs. (Some BC AIRS-classified data appears as open data, e.g. the
  HealthLinkBC MHSU dataset on Canada's open government portal — confirm scope/terms.)
- We **curate** Open Eligibility: adopt its structure, prune depth we don't need (e.g. its
  deep clinical oncology subtree), and **add (+)** the street-homelessness / Canadian items
  it lacks (below).
- **Localization:** Victoria, BC → Canadian terms. Most notably **Indigenous (First Nations
  / Métis / Inuit)** instead of "Native American," and BC/Canada government-assistance
  categories (generic flags here; specific program names live in listing data).

---

## Axis A — Service & Program types

Adopted top levels (from Open Eligibility) with the subcategories most relevant here, plus
**(+)** helpvine additions for street survival. (Full OE depth is available upstream if a
listing needs a finer term.)

- **Food** — Meals (prepared), Food Pantry, Emergency Food, Food Delivery, Community Gardens, Help Pay for Food / Government Food Benefits, Nutrition.
- **Housing** — Temporary Shelter (+ **Weather Relief / warming & cooling**), Help Find Housing, Help Pay for Housing (Utilities, Internet/Phone), Housing Vouchers, Housing Advice, **Tenancy & eviction help (+)**, Supportive/Residential Housing (Assisted/Independent/Public/Sober Living), Safe Housing.
- **Goods** — Clothing (incl. work/school, vouchers), Baby Supplies (Diapers & Formula), Home Goods (Blankets & Fans, Furniture, **Home Fuels**), Personal Care Items, Medical Supplies (Assistive Tech, Prosthesis), Toys & Gifts.
- **Hygiene & facilities (+)** — **Showers, Laundry, Restrooms, Drinking water.** (OE buries "Personal Hygiene" under Health; for this population it deserves to be first-class.)
- **Storage (+)** — **Belongings storage / lockers, document & ID storage** (ties to the vision's "store my papers").
- **Transit** — Bus Passes, Help Pay for Gas/Car, Transportation for Healthcare/School.
- **Health** — Primary/Medical Care, Dental, Vision, Mental Health Care, **Addiction & Recovery**, **Harm Reduction (+)** (supervised consumption, needle/supply exchange, naloxone, drug checking — BC-critical), Personal Hygiene, Sexual & Reproductive Health, Help Pay for Healthcare (Prescription Assistance, Health Insurance), Help Find Healthcare, **Support & Service Animals**.
- **Money** — Financial Assistance (Help Pay for Utilities/Housing/Food/Healthcare/Childcare), Government Benefits (incl. Disability, Unemployment), **specific income assistance e.g. energy/utility bills (+)**, Tax Preparation, Financial Education, Insurance, Loans.
- **Care** — Community Support Services (**Computer/Internet Access**, Recreation, **Weather Relief**), **Outreach & drop-in (+)** (street outreach teams, drop-in centres), Daytime Care/Childcare, Physical Safety (Disaster Response, Emergency Food, Temporary Shelter, **Help Escape Violence**, Immediate Safety), Support Network (Help Hotlines, Peer Support, Case Management, **Navigating the System**, Help Fill out Forms), **Mail receiving / address & phone/charging (+)**.
- **Education** — Help Find/Pay for School, Tutoring, Basic Literacy, Computer Class, GED/High-School Equivalency, ESL, Skills & Training, Youth Development, Special Education.
- **Work** — Help Find Work (Job Placement, Supported Employment), Skills & Training (Resume, Interview, Assessment), Help Pay for Work Expenses, Workplace Rights.
- **Legal** — Advocacy & Legal Aid, **Identification Recovery (ID replacement)**, Citizenship & Immigration, Discrimination & Civil Rights, Guardianship, Mediation, Notary, Representation, **Translation & Interpretation**, Workplace Rights, **tenancy/eviction (cross-ref Housing)**.

---

## Axis B — Client / Situation types  *(= the intersection axes)*

Adopted from Open Eligibility *Human Situations*, merged with the user's list, **(+)** = added.

- **Age Group** — coarse bands for matching (Infant/Child/Teen/Young Adult/Adult/Senior, All Ages). *Design note:* a person's situation carries a band, but **programs store real numeric min/max ages** as structured fields — the band is for fuzzy matching, the numbers gate eligibility.
- **Gender & Identity** — Female, Male, Transgender or Non-Binary, **Prefer not to say (+)**.
- **Sexual Orientation & Identity** — LGBTQ+, Transgender.
- **Race / Ethnicity** — **Indigenous: First Nations / Métis / Inuit (+ Canadian localization)**, plus general POC / racialized categories. (Many local orgs are Indigenous-specific, e.g. VNFC — so this axis matters for intersectional routing.)
- **Housing status** — Homeless, **Near/Hidden Homeless (car, couch-surfing) (+)**, At Risk of Homelessness, Renter, Home Owner, Runaway.
- **Income & Benefits** — Low-Income, Benefit Recipients (on government assistance), **No Income (+)**.
- **Employment** — Employed, **Part-time / Gig worker (+)**, Unemployed, Retired.
- **Education status** — Student, Dropout / not in school.
- **Household & Role** — Individual, Family, Family With Children, Single Parent, **Parent of minor children**, Pregnant, Caregiver, Spouse, **Marital status: married/single (+)**, **Foster Youth / aging out of care (+)**.
- **Disability** — All / Physical / Intellectual / Learning (**ADD/ADHD**) / Mobility / Deaf or Hard of Hearing / Visual / Brain Injury.
- **Mental Health** — All Mental Health, Anxiety, Depression, Bipolar, PTSD/trauma, Suicidal Thoughts, Eating Disorder (kept moderate; not a clinical registry).
- **Substance Dependency** — Alcohol, Opioid, Dual Diagnosis, Smoker, **general substance-use issues**.
- **Health conditions** — Chronic Illness, HIV/AIDS, Diabetes, Terminal Illness, Infectious Disease. *(OE's deep cancer/condition tree pruned — out of scope for this app.)*
- **Justice Involvement** — Criminal Justice History, Currently Incarcerated, **Recently Released / reintegrating (+)**.
- **Citizenship / Immigration** — Immigrant, Refugee, Undocumented, **Newcomer (+)**.
- **Armed Forces** — Veterans, Active Duty.
- **Survivors** — Domestic Violence, Abuse/Neglect, Human Trafficking, Sexual Assault, Trauma.
- **Has Pets (+)** — *not in OE as a situation, but make-or-break for shelter eligibility (pets-OK) and ties directly to the §5 "pets-OK" hard filter.*
- **Language** — Limited English; **French; Indigenous languages (+ Canadian)**.
- **Urgency** — In Crisis, In Danger, Emergency. *(Connects to the §2 "time- and crisis-aware" idea — urgency should lift ranking.)*

---

## Design notes & rules

- **Multi-select, always.** Both axes are sets, not single picks — that's what makes intersection possible.
- **Same vocabulary, both sides.** A listing flagged `women` + `housing/tenancy` must use the *same* Axis-B/Axis-A terms a user situation does, or matching silently breaks.
- **Age is special:** band on the person, numeric min/max on the program (above).
- **Hard filters vs. boosts:** some Axis-B tags are hard eligibility gates ("women only," "under 25," "pets-OK" → §5 filters); others are soft boosts (intersection overlap → ranking). Mark which is which per category — **[TBD]**.
- **Open-world growth:** the LLM may encounter a service/situation with no good tag. Allow it to *propose* a new tag (Tier-1 additive edit, §6) into a review queue rather than silently mis-filing — this is how the vocabulary grows from real data without becoming a free-for-all.
- **Sensitivity:** several Axis-B tags are highly sensitive (immigration status, substance use, DV survivor, mental health). Inferred tags follow profile consent + privacy rules (§4, §8) — used to help, never exposed.

---

## Open questions

- ~~License to redistribute Open Eligibility~~ → **resolved: CC BY-SA 3.0** (free, attribution + share-alike).
- **BC211 partnership/data:** available by data-sharing agreement (United Way BC), not open download — see [directory-data-sources.md](directory-data-sources.md). Generalizes to other cities via 211/HSDS.
- **AIRS ↔ Open Eligibility crosswalk:** build a mapping so 211-classified data flows into our OE-backed model.
- Per-category: hard-filter vs. soft-boost designation.
- How granular to go on health/disability/mental-health without turning into a clinical taxonomy.
- Mapping table between Axis A and Axis B (e.g. service `Help Escape Violence` ↔ situation `Domestic Violence survivor`) for auto-suggesting intersections.
- Canadian/BC government-assistance program list (income assistance types, e.g. BC programs) — data, not taxonomy, but needs a home.

---

## Sources

- [The Open Eligibility Project — findhelp](https://company.findhelp.com/the-open-eligibility-project/) (full Human Services + Human Situations lists)
- [Open Eligibility Taxonomy — HL7 Terminology](https://terminology.hl7.org/CodeSystem-OpenEligibilityTaxonomy.html)
- [AIRS/211 LA County Taxonomy of Human Services](https://211la.wordpress.com/) (the larger, license-restricted standard)
- [HUD HMIS Universal Data Elements](https://www.hudexchange.info/programs/hmis/hmis-data-standards/standards/universal-data-elements/) (homelessness subpopulation/demographic standards)
