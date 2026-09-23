# InsuranceAgent — Fabric data agent setup

The data agent sits on **three sources over the same data** and routes between them
by capability. That routing is the whole point of the demo: each source can do
something the others can't.

| Source | Type | Good at | Can't do |
|---|---|---|---|
| `InsuranceSM` | Semantic model (DAX) | Metrics, aggregations, rankings, trends | Fields not in the model |
| `RF_Lakehouse` | Lakehouse SQL endpoint (T-SQL) | CTEs, window functions, data-quality checks, row-level lists | — |
| `InsuranceOntology_ManualGen` | Ontology (graph) | Multi-hop traversal on specific records | Aggregation |

## Prerequisites

Build these first, in order:

1. `notebooks/01_create_lakehouse_data.ipynb` → `RF_Lakehouse` with 8 tables
2. `notebooks/02_create_ontology.ipynb` → `InsuranceOntology_ManualGen`
3. `semantic-model/` → `InsuranceSM`

## Setup

### 1. Create the agent

Workspace → **New item** → **Data agent**. Name it `InsuranceAgent`.

### 2. Add agent instructions

Paste [`agent-instructions.md`](agent-instructions.md) into the agent's
**AI instructions** box.

This is the routing brain. It tells the agent which source to reach for, how to
fall back, and the domain rules that stop it from producing confidently wrong
numbers — `approved_amount` NULL handling, the "no closed status" rule, and the
loss-ratio definition.

### 3. Add the three data sources

Add each source, then paste in its configuration from
[`data-sources/`](data-sources/):

```
data-sources/
├── 1-lakehouse-RF_Lakehouse/
│   ├── description.md          -> data source Description
│   ├── instructions.md         -> data source Instructions
│   ├── schema-descriptions.md  -> per-table/column descriptions
│   ├── topics.md               -> Topics (6 topics, each with instructions)
│   └── example-queries.json    -> Example queries (15 few-shot SQL pairs)
├── 2-semantic-model-InsuranceSM/
│   └── instructions.md         -> data source Instructions
└── 3-ontology-InsuranceOntology_ManualGen/
    └── instructions.md         -> data source Instructions
```

Select all 8 tables on the lakehouse and all 8 entity types on the ontology.

### 4. Publish

Publish the agent. Test with the questions below before demoing.

## Why the few-shots matter

`example-queries.json` holds 15 question/SQL pairs. They are the single highest-leverage
part of this configuration — they teach the agent the join paths and the traps:

- Aggregate `claim_events` in a CTE **before** joining to `claims`, or counts inflate
- `NULLIF` in every denominator
- `claims.adjuster_id` (assigned) vs `claim_events.adjuster_id` (performed the work)
- `SUM(approved_amount)` silently drops unpaid claims

Without these the agent will fan out rows on joins and quietly report inflated totals.

## Demo questions

Each maps to an edge case deliberately planted by notebook 01, so all return real results.

**Routes to the semantic model:**
- What are total estimated loss, approved payout, and loss ratio by claim type?
- What's the monthly trend in claims filed in Q1 2026?
- Which offices have the highest approved payouts?

**Routes to the lakehouse (needs SQL the model can't express):**
- Which claims have estimated losses greater than their policy coverage amount?
- Which adjusters are closest to or over their maximum active-claim capacity?
- Which insured assets have no recorded last inspection date?

**Routes to the ontology (single-record traversal):**
- Which adjuster handled claim CN500042, and what policy covers the asset involved?
- What events occurred on claim CN500107?

**Good stress test:**
- How many claims are closed?

There is no `closed` status. A correctly configured agent says so and offers
paid/denied instead of inventing a number.

## Note on the instructions

`agent-instructions.md` ends with *"Data is Confidential — summarize, don't dump."*
That line is deliberate: it shapes response behaviour. The data in this repo is
entirely synthetic, so nothing here is actually confidential. Keep or drop the line
depending on whether you want that behaviour in your own deployment.
