# Research Gaps and Next Validation Plan

## Evidence gaps

- No single cross-platform normative standard covers mini-app registration, store review, rollout, ranking and deprecation.
- Public documentation is incomplete for Gojek marketplace taxonomy/search/ranking/reviews/analytics and for Kakao/Grab public marketplace behavior.
- Vendor-specific WindVane/uni-app manifest fields, JSAPI permission taxonomy, host SDK compatibility matrix and package limits require direct POC validation.
- Publisher KYC, payout, tax, content policy, privacy residency and retention depend on jurisdiction and business model.
- Exact rollout percentages, SLOs, package-size ceilings, vulnerability severity cutoffs and rollback timing require telemetry/threat-model calibration.

## Pilot validation backlog

1. Export the POC's actual app/version/manifest/release/API schema without copying secrets.
2. Create a harmless test miniapp and record every lifecycle transition from registration to gray release and rollback.
3. Capture the exact host SDK, JSAPI and capability matrix for Android, iOS and Flutter.
4. Test package tamper, signature failure, incompatible host, denied capability, revoked release and stale cache behavior.
5. Measure install/download/verification/launch/startup failure rates by cohort and host version.
6. Verify audit events are immutable, queryable by release ID and sufficient to reconstruct who approved/promoted/paused/rolled back a release.
7. Run accessibility checks on catalog, search, filters, permission summaries, review status, error states and support flows.
8. Threat-model one sensitive-data miniapp and one partner-owned miniapp; calibrate risk tiers and review SLAs.
9. Define regional data/retention/notice policy with Legal and Privacy before onboarding external publishers.
10. Revisit ranking/featured/monetization only after P0 trust and release controls pass.

## Decision log

- The report preserves vendor practice as vendor evidence, not a universal standard.
- Design proposals are explicitly labeled and do not claim normative status.
- Credentials in the input spreadsheet were not copied to state, report or repository.
