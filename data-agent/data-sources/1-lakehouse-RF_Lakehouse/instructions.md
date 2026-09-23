Dialect: T-SQL (Fabric SQL analytics endpoint).
- SELECT TOP (n), never LIMIT
- COUNT(*), never COUNT()
- NULLIF(x, 0) in any denominator
- CONCAT() for names, YEAR()/MONTH() for date parts

Semantics:
- claims.status: filed, under_review, investigation, approved, paid, denied.
  Open = status NOT IN ('paid','denied'). There is no 'closed' status.
- claims, claim_events, and policies each have a status column. Qualify it
  and state which one you used.
- claim_events.completed_at IS NULL means the event is incomplete.
- claims.approved_amount is NULL when unpaid. NULL is not 0, and SUM skips
  NULLs — so a payout total silently excludes unpaid claims.
- Two adjuster paths: claims.adjuster_id is the ASSIGNED adjuster;
  claim_events.adjuster_id is who performed the work. State which you used,
  or ask if the question is ambiguous.
- claim_type: fire, property_damage, liability, auto_collision, weather,
  auto_theft, water_damage.
- Loss ratio: restrict BOTH sides to claims where approved_amount IS NOT
  NULL, or label that unpaid claims sit in the denominator.
- Adjuster capacity: open claim count vs adjusters.max_active_claims.

Joining claims to claim_events fans out rows. Aggregate events in a CTE or
subquery before joining, or counts and sums will be inflated.
