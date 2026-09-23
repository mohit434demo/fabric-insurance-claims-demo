# InsuranceSM semantic model

Direct Lake model over the 8 `RF_Lakehouse` tables. Run notebook 01 first.

## Option A — build it in the portal (simplest)

`RF_Lakehouse` → **New semantic model** → select all 8 tables → name it `InsuranceSM`.

Then add the relationships in the table below. The portal will auto-detect some of
them; verify each one, and make sure the three marked inactive stay inactive.

## Option B — import the TMDL definition

`InsuranceSM.SemanticModel/` is the exported definition. Deploy it with
[fabric-cli](https://github.com/microsoft/fabric-cli), the Fabric REST API
(`createItem` with the definition parts), or by connecting the folder to your own
workspace via Fabric Git integration.

**Before you deploy**, edit `definition/expressions.tmdl` and replace the placeholder
with your own lakehouse SQL analytics endpoint:

```
database = Sql.Database("<YOUR-SQL-ANALYTICS-ENDPOINT>.datawarehouse.fabric.microsoft.com", "RF_Lakehouse")
```

Find it in the portal: `RF_Lakehouse` → **SQL analytics endpoint** → Settings →
**SQL connection string**.

## Relationships

| From | To | Active |
|---|---|---|
| `claims.policy_id` | `policies.policy_id` | yes |
| `claims.policyholder_id` | `policyholders.policyholder_id` | yes |
| `claims.asset_id` | `insured_assets.asset_id` | yes |
| `claims.adjuster_id` | `adjusters.adjuster_id` | yes |
| `claims.office_id` | `offices.office_id` | yes |
| `claim_events.claim_id` | `claims.claim_id` | yes |
| `asset_inspections.asset_id` | `insured_assets.asset_id` | yes |
| `asset_inspections.office_id` | `offices.office_id` | yes |
| `claim_events.adjuster_id` | `adjusters.adjuster_id` | **inactive** |
| `policies.asset_id` | `insured_assets.asset_id` | **inactive** |
| `policies.policyholder_id` | `policyholders.policyholder_id` | **inactive** |

The inactive ones are deliberate — they'd create ambiguous filter paths alongside the
active routes through `claims`. Reach them with `USERELATIONSHIP` when a question is
specifically about who *performed* work rather than who was assigned.

## Measures

**None.** The model ships without predefined measures on purpose — the data agent is
instructed to write explicit DAX (`SUM`, `COUNTROWS`, `AVERAGE`, `DIVIDE`). That keeps
the demo honest about what the agent is actually generating rather than hiding the work
behind pre-built measures.

Add measures if you want, but update
`data-agent/data-sources/2-semantic-model-InsuranceSM/instructions.md` to match, or the
agent will keep rolling its own.
