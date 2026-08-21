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
  demand signals and archives each finished set to the Notion "Persona Library" database.
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

**Format:** `P###` — a capital `P` plus a zero-padded, **globally sequential** number
(P001, P002, …). Global, not per-category: `P007` refers to one specific persona forever, no
matter which category run produced it. Category/segment is captured in separate fields, so you
lose no grouping.

**Assigning the next IDs:**
1. Open the **Persona Registry** (coordinates in Step 5) and find the highest existing `Persona ID`.
2. Assign the next numbers in sequence to the new personas, in the order they appear. E.g. if the
   registry's max is `P018`, a fresh run of 5 personas gets `P019`–`P023`.

**Immutability — this is the whole point, never violate it:**
- Once assigned, a persona's ID is **never changed, reused, or renumbered** — not when you edit
  the persona, not when you re-run the category, not when personas are added or removed.
- Re-running a category does **not** reset numbering — always continue from the global max.
- To remove a persona, set its Registry **Status** to `Retired`. The ID stays burned forever —
  never reassign it to a different persona.
- If you rewrite a persona into a genuinely different one, **retire the old ID and mint a new
  one** rather than repurposing the old number.

Surface each ID inline in chat (heading format `## P019 · [Name]`) and add an **ID** column as
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

## Step 5 — Archive to Notion (Library + Registry)

Two databases, always written together. The **Library** holds the full narrative set; the
**Registry** is the SKU master that assigns/holds the permanent IDs. If a data source ID below
ever fails to resolve, find the database by searching Notion for its name and use the current
data source; if it truly doesn't exist, recreate it with the same schema.

**A. Persona Library** — one row per *set/run* (the full document).
- Database: https://app.notion.com/p/15afe072d91249e9be960afa8c949ce7
- Data source ID: `collection://174e9036-3ab2-489d-88d1-5c25f1c4f5bc`
- Create one page (`notion-create-pages`, parent = the data source ID):
  - **Name** — e.g. `Interior Designers — Frames & Gallery Walls (18)`
  - **Demographic**, **Product Context** — the inputs used
  - **Personas** — the count (number) · **Split** — the segmentation split, if any
  - **Status** — `Draft` (hypothesis-only), `BQ-Validated`, or `Final` (user-approved)
  - **BQ Grounded** — checkbox
  - **Page body** — the full content: every persona block (headed `## P### · [Name]`) **and** the
    Final Persona Map (with its ID column). This is the durable artifact — include all of it.
- Keep the returned set-page URL; you need it for the Registry relation.

**B. Persona Registry** — one row per *individual persona* (the SKU master).
- Database: https://app.notion.com/p/4dea28d70b2c49bbaf6de53eefbcd530
- Data source ID: `collection://c4c9eada-83c5-40ad-8f21-6c748faef00e`
- Create one page per persona (`notion-create-pages`, parent = the registry data source ID):
  - **Persona ID** (title) — the permanent `P###` from Step 3
  - **Name**, **Segment**, **One-liner** (the persona-in-one-sentence)
  - **Status** — `Draft` until validated/approved, then `Active`; `Retired` when burned
  - **Set** — relation to the Library set page (pass its URL in an array)

After writing both, give the user the Library page link and the assigned ID range (e.g. P019–P023).

## Guardrails (from the methodology — worth repeating)

- **Personas only.** No angles, offers, hooks, headlines, or campaign ideas — those are the next
  agents' jobs, and producing them here degrades the input they rely on.
- **No invented statistics.** Label anything not backed by supplied research or BigQuery as a
  hypothesis.
- **Don't pad to a number.** Fewer real personas beats more fake ones — say so when the market
  supports fewer.
- **Customer voice, not marketing voice**, especially in the Natural Language quotes.
