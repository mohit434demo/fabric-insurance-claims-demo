Sample Descriptions for schema objects in RF_Lakehouse lakehouse tables

dbo:Insurance claims operations — claims, policies, policyholders, adjusters,
offices, insured assets, and inspection activity. Jan–Mar 2026.

adjusters: Claims staff. max_active_claims is each adjuster's capacity ceiling, not a
current workload count. home_office_id is their base office, which may
differ from the office a claim was filed under.

claims: One row per filed claim. Central fact table linking policy, policyholder,
assigned adjuster, and filing office. Holds claim_type, status,
estimated_loss, and approved_amount (NULL until paid). Open = status not
paid or denied.