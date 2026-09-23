# InsuranceAgent — setup

The configuration is the interesting part of this repo. Once the agent is running,
see [`exploring.md`](exploring.md).

## Prerequisites

The agent needs its three sources to exist first:

1. `RF_Lakehouse` with 8 tables — [notebook 01](../notebooks/01_create_lakehouse_data.ipynb)
2. `InsuranceOntology_ManualGen` — [notebook 02](../notebooks/02_create_ontology.ipynb)
3. `InsuranceSM` semantic model — [`semantic-model/`](../semantic-model/)

## 1. Create the agent

Workspace → **New item** → **Data agent**. Name it `InsuranceAgent`.

## 2. Paste the agent instructions

[`agent-instructions.md`](agent-instructions.md) → the agent's **AI instructions** box.

This is the routing brain. It decides which source handles what, how to fall back when
one fails, and the domain rules that stop confidently-wrong answers — NULL handling on
`approved_amount`, the "there is no closed status" rule, and the loss-ratio definition.

## 3. Add the three sources

Add each source in the portal, then paste in its configuration:

| Source | Add as | Files |
|---|---|---|
| `RF_Lakehouse` | Lakehouse | [`1-lakehouse-RF_Lakehouse/`](data-sources/1-lakehouse-RF_Lakehouse/) — description, instructions, schema descriptions, topics, example queries |
| `InsuranceSM` | Semantic model | [`2-semantic-model-InsuranceSM/instructions.md`](data-sources/2-semantic-model-InsuranceSM/instructions.md) |
| `InsuranceOntology_ManualGen` | Ontology | [`3-ontology-InsuranceOntology_ManualGen/instructions.md`](data-sources/3-ontology-InsuranceOntology_ManualGen/instructions.md) |

Select **all 8 tables** on the lakehouse and **all 8 entity types** on the ontology.

For the lakehouse, each file maps to a field in the portal:

```
description.md          -> Data source description
instructions.md         -> Data source instructions
schema-descriptions.md  -> per-table / per-column descriptions
topics.md               -> Topics (6, each with its own instructions)
example-queries.json    -> Example queries (15 few-shot pairs)
```

## 4. Publish and test

Publish, then run the questions in [`exploring.md`](exploring.md). Confirm each one
routes to the source you expect before showing anyone.

---

## Why the example queries matter

`example-queries.json` holds 15 question/SQL pairs and is the highest-leverage part of
this configuration. They teach join paths and traps that prose instructions don't
reliably convey:

- Aggregate `claim_events` in a CTE **before** joining to `claims`, or counts inflate
- `NULLIF` in every denominator
- `claims.adjuster_id` (assigned) vs `claim_events.adjuster_id` (performed the work)
- `SUM(approved_amount)` silently drops unpaid claims

[`exploring.md`](exploring.md) has an experiment for removing them and watching the
answers degrade. It's the clearest demonstration of what configuration buys you.

## Note on the "Confidential" line

`agent-instructions.md` ends with *"Data is Confidential — summarize, don't dump."*
That's deliberate — it shapes response behaviour and keeps the agent from dumping raw
rows. The data here is entirely synthetic, so nothing is actually confidential. Keep
the line if you want that behaviour, drop it if it confuses the conversation.
