Claims operations data for a property & casualty insurer — the physical
source of record. Tables: claims, claim_events, policies, policyholders,
adjusters, offices, insured_assets. Data spans Jan–Mar 2026.

Use RF_Lakehouse when a question needs full SQL expressiveness:
- data-quality checks (NULLs, orphans, duplicates, missing dates)
- CTEs, window functions, correlated subqueries, HAVING with a subquery
- comparing a row against a threshold on a related table
  (e.g. estimated_loss vs policies.coverage_amount)
- columns not surfaced in the InsuranceSM semantic model
- row-level detail and export-style lists

InsuranceSM is built over these same tables and is preferred for standard
business metrics and aggregations. Prefer RF_Lakehouse when the question
needs logic the semantic model can't express, or fields it doesn't surface.
