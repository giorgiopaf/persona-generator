---
name: persona-generator
description: >-
  Generate specific, differentiated, commercially-useful buyer personas for ANY target
  demographic or category — the Persona stage of a Persona → Angle → Offer system. Use this
  whenever the user wants customer personas, buyer personas, audience broken into distinct
  personas, "who are our customers / who's actually buying X," a persona map, or ICP
  segmentation — even if they don't say the word "persona." Also trigger on "persona agent,"
  "/persona-generator," "build personas for [category]," or "generate personas for [audience]."
  Works for any market (DTC consumers, gift-buyers, trade/wholesale, B2B, hospitality/contract),
  not just one vertical. Produces personas ONLY — never angles, offers, hooks, headlines, or
  campaigns (those are downstream agents). Optionally grounds personas against BigQuery for
  demand signals; archives each master persona to the Notion "Persona Database" and every
  sub-persona to the linked "Persona Library."
---

# Persona Generator

Stage 1 of a **Persona → Angle → Offer** pipeline. This skill turns a target demographic into a
set of sharp, differentiated buyer personas, then archives them to Notion. It is deliberately
category-agnostic: the same run works for interior designers, gift-buyers, or wholesale buyers.

**Read `references/methodology.md` in full before generating** — it is the craft spec (persona
definition, discovery process, differentiation test, output format, and the ten rules). This
SKILL.md is only the workflow around it.

## Step 1 — Gather the inputs

Collect these before generating. Two are required; the rest have sensible defaults.

| Input | Required? | Default if omitted |
|-------|-----------|--------------------|
| **Target demographic** | Yes | — ask if missing |
| **Number of personas** | Yes | — ask if missing |
| **Product / category context** | No | Americanflat frames, wall art & home decor |
| **Segmentation split** (e.g. residential vs. commercial) | No | none — let personas fall out naturally |
| **Additional research** (reviews, interviews, data) | No | none |

If a required input is missing, ask for it concisely — don't guess a demographic or a count.
Tighter demographics produce sharper personas: nudge the user from "women 25–54" toward a
specific situation ("parents decorating a nursery") when their target is vague. A quick
multiple-choice question to pin the count and confirm the product context is usually worth it.

## Step 2 — Generate the personas

Follow `references/methodology.md` exactly. In short: run the discovery process internally,
build scenario-specific personas (never broad labels), apply the differentiation test to every
pair, and **do not manufacture differences to hit the requested number** — if the market
genuinely supports fewer, say so.

Output in chat using the per-persona format from the methodology, then always close with the
**Final Persona Map** table plus the one-line "what makes each different" notes.

State up front that personas are **hypotheses** unless research or BigQuery grounding was used —
never invent statistics or claim evidence that wasn't provided.

## Step 3 — Assign permanent persona IDs (SKUs)

Every persona gets a permanent identifier the **moment it is created** — assigning IDs here,
upstream, is what lets the downstream Angle → Offer → Creative skills build a stable
`P### → A## → O## → S###` lineage. (If IDs were assigned later at script time, "P01" could point
at a different persona on every run — the opposite of stable.)

**Format:** `P` + a **master-type letter** + a 2-digit number — e.g. `PI01` for Interior
Designers, `PC01` for Commercial Artists. The letter identifies the master persona: by default
its first letter; if two masters would collide on a letter, pick a distinct unused letter and
note it on the master row. Numbering **restarts per master** (each master runs 01, 02, …), so
`PI07` and `PC07` are different personas under different masters.

**Assigning the next IDs:**
1. Determine the master's letter — an existing master reuses its letter; a new master takes its
   first free letter.
2. In the **Persona Library**, find the highest existing `Persona ID` beginning `P<letter>` and
   continue from there. E.g. if `PI18` exists, the next Interior-Designer persona is `PI19`; a
   brand-new Commercial-Artists master starts at `PC01`.

**Immutability — this is the whole point, never violate it:**
- Once assigned, a persona's ID is **never changed, reused, or renumbered** — not when you edit
  the persona, not when you re-run the master, not when personas are added or removed.
- Re-running a master does **not** reset its numbering — always continue from that master's
  highest existing number.
- To remove a persona, set its **Status** to `Retired`. The ID stays burned forever — never
  reassign it to a different persona.
- If you rewrite a persona into a genuinely different one, **retire the old ID and mint a new
  one** rather than repurposing the old number.

Surface each ID inline in chat (heading format `## PI19 · [Name]`) and add an **ID** column as
the first column of the Final Persona Map.

## Step 4 — (Optional) Ground against BigQuery

Offer this whenever the category plausibly maps to Americanflat sales data. Be honest about what
it can and can't do:

- **Can** validate *demand and format* signals — which sizes, gallery/multi-packs, bulk
  quantities, price points, and custom-print SKUs actually sell; reorder behavior; return/damage
  rates; geographic concentration.
- **Usually cannot** identify *who* a buyer is (designer vs. consumer, or a specific firm) —
  marketplace buyers are anonymous to the vendor. The exception is a wholesale/B2B/trade dataset;
  **check whether one exists first** (`bq ls americanflat:`), because that's the only path to real
  firmographic validation.

Query per the user's global BigQuery rules (`bq query --use_legacy_sql=false`, read-only,
backtick-escaped `\`americanflat.dataset.table\``, always LIMIT/WHERE). Then mark each persona
**Validated / Unclear / Refuted** so only personas with a pulse move downstream. If BQ auth
fails, tell the user to run `gcloud auth login` and proceed with clearly-labeled hypotheses.

## Step 5 — Archive to Notion (Persona Database + Persona Library)

**Two linked databases.** The **Persona Database** holds one row per *master persona* (the
umbrella category, e.g. `Interior Designers`); the **Persona Library** holds one row per
*sub-persona* (`P###`), each linked back to its master. Both live under the **Persona System**
hub page (https://app.notion.com/p/3c38555c2abc81a4a50edd3cc150b5e5). If a data source ID below
ever fails to resolve, find the database by searching Notion for its name and use the current
data source; if it truly doesn't exist, recreate it with the same schema.

**A. Persona Database (masters)** — one row per umbrella category.
- Database: **Persona Database** — https://app.notion.com/p/159db9b455774311bbb525ddddfbdb91
- Data source ID: `collection://21bc8a5f-f6a9-4152-a250-78b05a60cd73`
- **Find-or-create** the master row for this run's category (match on **Master Persona** = the
  category name; only create it if it doesn't already exist — don't duplicate a master):
  - **Master Persona** (title) — the umbrella name, e.g. `Interior Designers`
  - **Demographic**, **Product Context** — the run's inputs
  - **Status** — `Draft` / `Active` / `Retired`
- Keep the returned master-row URL; you link each sub-persona to it below. (The **Sub-personas**
  relation back-fills automatically from the Library side.)
- So opening a master shows its sub-personas, give the **master row's page body** a view of them:
  ideally an inline **linked view of the Persona Library filtered to this master** (added in the
  Notion UI via `/Link to database`), or — as an API-createable fallback — a grouped linked list
  (by segment, each linking its `P##` page).

**B. Persona Library (sub-personas)** — one row per persona.
- Database: **Persona Library** — https://app.notion.com/p/4dea28d70b2c49bbaf6de53eefbcd530
- Data source ID: `collection://c4c9eada-83c5-40ad-8f21-6c748faef00e`
- Create one page per persona (`notion-create-pages`, parent = the data source ID):
  - **Persona ID** (title) — the permanent `P###` from Step 3
  - **Name** — the persona's descriptive name
  - **Segment** — e.g. Residential, Hotels, Healthcare
  - **Master** — relation to the master row in Persona Database (pass its URL in an array)
  - **One-liner** — the persona-in-one-sentence
  - **Status** — `Draft` until validated/approved, then `Active`; `Retired` when burned (the ID
    stays burned — never reassigned)
  - **Page body** — the persona's **full detail**, laid out with the clean callout template (not
    stacked headings): a **summary callout** (the one-sentence) → a **Segment** line → a
    **Job-to-be-done** callout and a **Core-problem** callout → `## Pain points` → `## Buying
    priorities` → a **snapshot callout** grouping Desired outcome / Emotional driver / Trigger /
    Current solution → `## Objections` → `## In their words` (quotes). **Write real line breaks in
    the content — never the literal characters `\n` or `\t`**, which render as a garbled single
    paragraph.

After writing, give the user links to the master row and the Library (filtered to this master),
plus the assigned ID range (e.g. P019–P023). In chat, still present the full personas plus a
Final Persona Map for review.

## Guardrails (from the methodology — worth repeating)

- **Personas only.** No angles, offers, hooks, headlines, or campaign ideas — those are the next
  agents' jobs, and producing them here degrades the input they rely on.
- **No invented statistics.** Label anything not backed by supplied research or BigQuery as a
  hypothesis.
- **Don't pad to a number.** Fewer real personas beats more fake ones — say so when the market
  supports fewer.
- **Customer voice, not marketing voice**, especially in the Natural Language quotes.
