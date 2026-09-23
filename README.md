# Fabric data agent — insurance claims

A working **Microsoft Fabric data agent** configured over three query surfaces on the
same dataset, built to show how an agent routes a question to the right engine.

```
                        RF_Lakehouse  (8 tables, synthetic claims data)
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
   InsuranceSM      InsuranceOntology_       RF_Lakehouse
  (semantic model)       ManualGen            SQL endpoint
   aggregate, DAX     traverse, graph        anything, T-SQL
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              │
                        InsuranceAgent
                     routes by capability
```

Most data agent demos have a single source, so routing never comes up. Here each
surface can do something the others genuinely can't — the semantic model aggregates,
the ontology walks relationships, raw SQL handles what neither can express — and the
agent picks per question.

**Start here once it's running:** [`data-agent/exploring.md`](data-agent/exploring.md)

All data is synthetic, spanning Jan–Mar 2026.

---

## Prerequisites

- A Fabric workspace on an F SKU or trial capacity
- **Fabric IQ / Ontology** enabled (preview) — needed for the ontology source
- Permission to create lakehouses, notebooks, semantic models, and data agents

---

## Setup

### Step 1 — Get data into a lakehouse

Two options. Either works; the agent configuration is identical.

**Option A — quickstart (self-contained)**

Create a lakehouse named `RF_Lakehouse`, attach it to
[`notebooks/01_create_lakehouse_data.ipynb`](notebooks/01_create_lakehouse_data.ipynb),
run all cells.

Generates 8 Delta tables from a fixed seed, and deliberately plants the edge cases the
demo questions rely on — NULL payouts, claims above coverage, uninspected assets,
over-capacity adjusters, incomplete events. The last cell asserts every one is present,
so you know before a demo whether any question will come back empty.

**Option B — the full workshop dataset**

The broader Fabric IQ insurance workshop this demo grew out of lives at
**[alipouw13/insurance-FabricIQ](https://github.com/alipouw13/insurance-FabricIQ)** and
includes the original reference data plus an Eventhouse / Real-Time Intelligence track.

Use its `reference_data/*.jsonl` and `01_load_reference_data.ipynb` to load the tables,
then come back here for the ontology, semantic model, and agent.

> Two caveats if you take this path. Its lakehouse is named `lh_insurance` — either
> rename to `RF_Lakehouse` or adjust the notebooks and agent config. And in that
> dataset every asset has a `last_inspection_date` and no adjuster is near capacity, so
> two of the lakehouse-routed questions return nothing. Everything else works.

### Step 2 — Ontology

Run [`notebooks/02_create_ontology.ipynb`](notebooks/02_create_ontology.ipynb) in the
same workspace with the lakehouse attached.

Creates `InsuranceOntology_ManualGen` — 8 entity types, 10 relationship types, built via
the REST API **with data bindings included**.

> That's what "ManualGen" means. The portal's generate button leaves data bindings
> *Unbound*, so no instances materialize and the graph comes back empty. Defining the
> bindings in code is what makes it queryable.

### Step 3 — Semantic model

See [`semantic-model/README.md`](semantic-model/README.md). Import the TMDL or build a
Direct Lake model over the 8 tables. Name it `InsuranceSM`.

### Step 4 — Data agent

See [`data-agent/README.md`](data-agent/README.md). Create `InsuranceAgent`, attach all
three sources, paste in the supplied instructions, topics, and example queries.

### Step 5 — Explore

[`data-agent/exploring.md`](data-agent/exploring.md) — routing walkthrough, ways to
break it, and an experiment showing exactly what the configuration is worth.

---

## Layout

```
notebooks/
  01_create_lakehouse_data.ipynb    8 Delta tables + planted edge cases
  02_create_ontology.ipynb          ontology via REST API, fully bound
semantic-model/
  README.md                         relationships, measures, deployment
  InsuranceSM.SemanticModel/        TMDL definition
data-agent/
  README.md                         setup
  exploring.md                      routing guide + experiments
  agent-instructions.md             routing brain
  data-sources/
    1-lakehouse-RF_Lakehouse/       description, instructions, schema, 6 topics, 15 few-shots
    2-semantic-model-InsuranceSM/   instructions
    3-ontology-InsuranceOntology_ManualGen/  instructions
```

---

## Data model

`claims` is the central fact table.

| Table | Grain | Notes |
|---|---|---|
| `claims` | one filed claim | `approved_amount` NULL until paid |
| `claim_events` | one action on a claim | `completed_at` NULL = incomplete; `cost_usd` is handling cost, **not** payout |
| `policies` | one policy | `coverage_amount` is the ceiling |
| `policyholders` | one person/org | |
| `insured_assets` | one asset | `last_inspection_date` NULL = never inspected |
| `adjusters` | claims staff | `max_active_claims` is a ceiling, not a count |
| `offices` | branch office | |
| `asset_inspections` | one inspection | |

The traps the configuration teaches the agent to avoid:

- **`approved_amount` is NULL, not 0.** `SUM` skips NULLs, so a raw payout total
  silently excludes unpaid claims.
- **There is no `closed` status.** Open = `NOT IN ('paid','denied')`.
- **`claims` → `claim_events` fans out rows.** Aggregate in a CTE first or counts inflate.
- **Two adjuster paths.** `claims.adjuster_id` assigned, `claim_events.adjuster_id` performed.
- **Three `status` columns** — on `claims`, `claim_events`, and `policies`.

---

## Credits

The insurance scenario, schema, and original reference data come from the Fabric IQ
insurance workshop at
[alipouw13/insurance-FabricIQ](https://github.com/alipouw13/insurance-FabricIQ).

This repo narrows that to the data agent layer: routing across three surfaces, the
configuration that makes it behave, and a guide to exploring it.

## Notes

Synthetic data only — no real policyholders, claims, or personal information. Names and
identifiers are generated from fixed seeds; any resemblance to real entities is
coincidental.

Ontology is a Fabric preview feature and its API surface may change.
