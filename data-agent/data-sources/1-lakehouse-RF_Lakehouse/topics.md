## Claims and status
Claim counts and lifecycle position.

**Instructions:**
Use `claims.status` (filed, under_review, investigation, approved, paid,
denied). "Open" = NOT IN ('paid','denied'). There is no "closed" — if asked,
treat as paid or denied and say so.

---

## Payouts and loss ratio
Approved money versus estimated money.

**Instructions:**
`approved_amount` is NULL until paid. SUM skips NULLs, so a raw payout total
silently excludes unpaid claims.
Loss ratio = SUM(approved_amount) / SUM(estimated_loss), both sides
restricted to approved_amount IS NOT NULL. Otherwise label it "unpaid claims
in the denominator". NULLIF the denominator.

---

## Adjuster workload and capacity
Who handles how much, and who is over their limit.

**Instructions:**
`claims.adjuster_id` = ASSIGNED adjuster. `claim_events.adjuster_id` = who
performed the work. Default to assigned; state which you used.
`max_active_claims` is a ceiling, not a count. Over capacity = open claims >
max_active_claims. LEFT JOIN so adjusters with zero open claims still appear.
`adjusters.home_office_id` ≠ `claims.office_id` — base office vs filing
office.

---

## Claim events
Work performed on a claim.

**Instructions:**
Incomplete = `completed_at IS NULL`. `claim_events.status` is the event's,
not the claim's — qualify it. `cost_usd` is not payout; never add it to
approved_amount.
Joining claims to claim_events fans out rows. Aggregate events in a CTE
first or counts and sums inflate.

---

## Policies and assets
Coverage, terms, and insured property.

**Instructions:**
"Exceeds coverage" = `claims.estimated_loss > policies.coverage_amount`.
`claims.policyholder_id` (claimant) may differ from
`policies.policyholder_id` (holder of record) — prefer the claim's, say
which.
`insured_assets.estimated_value` is asset worth, not `estimated_loss`.
`last_inspection_date IS NULL` = never inspected, not overdue.
Data spans Jan–Mar 2026 — read relative dates against that, not today.

---

## Data quality
Missing, orphaned, or inconsistent records.

**Instructions:**
The NULLs are the measurement — never report them as zero or filter them
out. Give the affected count alongside the total row count.


---