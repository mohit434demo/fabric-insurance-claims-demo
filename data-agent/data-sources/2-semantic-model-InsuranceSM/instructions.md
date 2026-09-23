Dialect: DAX. This model has **no predefined measures** — compute everything
explicitly with SUM, COUNTROWS, AVERAGE, and DIVIDE.

Use InsuranceSM as the default for any metric, count, total, average, ratio,
ranking, "top N", group-by, or time trend.

Standard metrics:
- Total Claims   = COUNTROWS(claims)
- Estimated Loss = SUM(claims[estimated_loss])
- Approved Payout= SUM(claims[approved_amount])
- Loss Ratio     = DIVIDE(Approved Payout, Estimated Loss), shown as a percentage
- Open Claims    = count of claims where status NOT IN ("paid", "denied")

Semantics:
- claims[status]: filed, under_review, investigation, approved, paid, denied.
  Open = NOT IN ("paid","denied"). There is no "closed" status.
- claims, claim_events, and policies each have a status column. Default to
  claims[status] and name which one you used.
- approved_amount is BLANK/NULL until paid. SUM skips it, so a raw payout total
  silently excludes unpaid claims. Use DIVIDE for ratios and state how NULLs
  were handled.
- claim_type: fire, property_damage, liability, auto_collision, weather,
  auto_theft, water_damage. Map user terms (e.g. "car crash" = auto_collision);
  ask if ambiguous.
- Dates are datetimes spanning Jan–Mar 2026. Derive month/quarter/year from
  them; read relative dates against that window, not today.

Relationship notes:
- claim_events → adjusters is INACTIVE. Activate it with USERELATIONSHIP only
  when the question is explicitly about who performed the work, and say so.
  Otherwise claims → adjusters (the assigned adjuster) is the active path.
- policies → policyholders and policies → insured_assets are also inactive;
  the active paths run through claims.

If a question needs a column not in this model, data-quality checks, or
row-level export-style lists, fall back to RF_Lakehouse.
