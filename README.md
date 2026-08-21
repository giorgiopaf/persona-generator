# persona-generator

A Claude Code skill that generates specific, differentiated, commercially-useful **buyer personas**
for any target demographic or category. It is stage 1 of a **Persona → Angle → Offer** system —
it produces personas only, never angles, offers, hooks, or campaigns.

## What it does

- Takes a target demographic + number of personas (product context, segmentation split, and
  research are optional).
- Runs a discovery process and a differentiation test to produce scenario-specific personas
  (not broad labels), each with situation, job-to-be-done, pains, desired outcome, emotional
  driver, trigger, buying priorities, objections, and customer-voice quotes.
- Ends with a Final Persona Map and a "what makes each different" summary.
- Optionally grounds personas against BigQuery (project `americanflat`) for demand/format signals.
- Archives each finished set to the Notion **Persona Library** database.

## How to use

Invoke on demand from Claude Code:

```
generate personas for [audience]
/persona-generator
build personas for [category]
```

If the demographic or count is missing, the skill asks before generating.

## Persona IDs (SKUs)

Every persona is assigned a **permanent identifier** at creation — format `P` + a master-type
letter + a 2-digit number (e.g. `PI01` for Interior Designers, `PC01` for Commercial Artists),
numbering restarting per master, never reused or renumbered. This is the SKU that the downstream
Angle → Offer → Creative skills reference to keep a stable `P##→ A## → O## → S###` lineage. Retiring a persona sets its status to `Retired`; the ID stays burned forever.

Personas live in two linked Notion databases:

- **Persona Database** — one row per *master persona* (the umbrella category, e.g. "Interior
  Designers"), with the demographic and product context.
- **Persona Library** — one row per *sub-persona* (keyed by Persona ID, full detail in the page
  body), each linked back to its master.

## Structure

```
persona-generator/
├── SKILL.md                    # workflow: gather inputs → generate → BQ grounding → Notion
└── references/
    └── methodology.md          # the persona craft spec (definition, discovery, rules)
```

## Notes

- Personas are labeled **hypotheses** unless backed by supplied research or BigQuery grounding —
  the skill never invents statistics.
- Category-agnostic: works for DTC consumers, gift-buyers, trade/wholesale, and B2B/contract
  markets, not just one vertical.
