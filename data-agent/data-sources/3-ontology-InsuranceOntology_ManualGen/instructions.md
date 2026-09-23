Use the ontology **only** for relationship traversal on specific records —
following edges from one known entity to another.

It CANNOT aggregate. Never use it for counts, sums, averages, rankings,
"top N", or group-by. Those go to InsuranceSM.

Good uses:
- Which adjuster handled claim CN500042?
- What policy covers the asset involved in claim X, and who holds it?
- What events occurred on claim Y, and who performed them?
- Which office is adjuster Z based at?

Entity types (8):
Office, Policyholder, InsuredAsset, Adjuster, Policy, Claim, ClaimEvent,
AssetInspection

Relationship types (10):
- PolicyCoversPolicyholder   Policy          -> Policyholder
- PolicyCoversAsset          Policy          -> InsuredAsset
- ClaimUnderPolicy           Claim           -> Policy
- ClaimInvolvesAsset         Claim           -> InsuredAsset
- ClaimAssignedToAdjuster    Claim           -> Adjuster
- ClaimFiledByPolicyholder   Claim           -> Policyholder
- ClaimEventForClaim         ClaimEvent      -> Claim
- AdjusterAtOffice           Adjuster        -> Office
- InspectionForAsset         AssetInspection -> InsuredAsset
- InspectionAtOffice         AssetInspection -> Office

Identify entities by their display property, not their GUID:
Claim by claim_number, Policy by policy_number, Adjuster by last_name,
InsuredAsset by asset_number, Policyholder by full_name, Office by name,
ClaimEvent by event_number.

Note ClaimAssignedToAdjuster is the ASSIGNED adjuster. Who actually performed a
given piece of work is on ClaimEvent, via its own adjuster reference — state
which one you used.

If a question needs an aggregate, switch to InsuranceSM rather than walking the
graph and counting.
