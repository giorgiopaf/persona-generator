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

## Step 3 — (Optional) Ground against BigQuery

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

## Step 4 — Archive to the Notion Persona Library

Write every finished set to the **Persona Library** database so the work is reusable and the
Angle Agent can pull from it.

- Database: **Persona Library** — https://app.notion.com/p/15afe072d91249e9be960afa8c949ce7
- Data source ID: `collection://174e9036-3ab2-489d-88d1-5c25f1c4f5bc`
- If that ID ever fails to resolve, find the database by searching Notion for "Persona Library"
  and use the current data source; if it truly doesn't exist, recreate it with the same schema.

Create one page per persona set with `notion-create-pages` (parent = the data source ID above):

- **Name** — descriptive title, e.g. `Interior Designers — Frames & Gallery Walls (18)`
- **Demographic** — the target demographic string
- **Product Context** — the category context used
- **Personas** — the count (number)
- **Split** — the segmentation split, if any (else blank)
- **Status** — `Draft` (hypothesis-only), `BQ-Validated` (grounded), or `Final` (user-approved)
- **BQ Grounded** — checkbox
- **Page body** — the full persona content: every persona block **and** the Final Persona Map
  table. This is the durable artifact, so include the whole thing, not a summary.

After writing, give the user the Notion page link.

## Guardrails (from the methodology — worth repeating)

- **Personas only.** No angles, offers, hooks, headlines, or campaign ideas — those are the next
  agents' jobs, and producing them here degrades the input they rely on.
- **No invented statistics.** Label anything not backed by supplied research or BigQuery as a
  hypothesis.
- **Don't pad to a number.** Fewer real personas beats more fake ones — say so when the market
  supports fewer.
- **Customer voice, not marketing voice**, especially in the Natural Language quotes.
