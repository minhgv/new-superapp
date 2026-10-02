# Research Gaps and Next Validation Plan

## Evidence gaps

- No single cross-platform normative standard covers mini-app registration, store review, rollout, ranking and deprecation.
- Public documentation is incomplete for Gojek marketplace taxonomy/search/ranking/reviews/analytics and for Kakao/Grab public marketplace behavior.
- Vendor-specific WindVane/uni-app manifest fields, JSAPI permission taxonomy, host SDK compatibility matrix and package limits require direct POC validation.
- Publisher KYC, payout, tax, content policy, privacy residency and retention depend on jurisdiction and business model.
- Exact rollout percentages, SLOs, package-size ceilings, vulnerability severity cutoffs and rollback timing require telemetry/threat-model calibration.
- The cited web/cache APIs provide primitives but not a cross-runtime mini-app offline contract; native WindVane/uni-app cache layers, invalidation, stale serving and offline launch behavior remain POC-specific unknowns.
- Payment provider, merchant-of-record, entitlement and refund/chargeback ownership are not determined by generic mini-app standards; they require a jurisdiction/provider contract and reconciliation tests.
- The POC's deep-link association, route/parameter allowlist and Android/iOS fallback behavior must be tested on real host versions; public web/app-link specifications do not prove WindVane support.
- Ranking transparency, incentivized-review rules and notification consent/volume limits are jurisdiction- and channel-specific; Legal, Privacy and Trust & Safety must bind the applicable policy profile.
- Accessibility evidence is still a control profile, not proof that the Alibaba/WindVane host bridge and every partner mini-app meet WCAG; native accessibility-tree, keyboard, screen-reader, text-scale and reduced-motion tests are required.
- Privacy obligations, controller/processor roles, retention periods, child-directed handling and age assurance remain jurisdiction- and product-specific; EDPB/ICO/NIST sources provide controls and decision points, not a universal legal answer.
- DSA/P2B notice, appeal, mediation and transparency duties depend on EU scope, provider category and exemptions; Google Play enforcement is platform-specific. Legal must map the target store model before adopting thresholds or SLA claims.

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
11. Test canonical deep links on Android/iOS host builds: verified association, malformed route, undeclared parameter, cold start, installed/uninstalled fallback and authentication-return replay.
12. Test cache behavior by artifact class under online, stale, error, DNS-failure and airplane-mode conditions; verify digest checks before cache commit and launch.
13. Run payment sandbox scenarios for pending/success/refund/chargeback/duplicate/out-of-order events; verify signature, deduplication, reconciliation and entitlement revocation.
14. Test notification opt-in, revoke, quiet hours, transactional/marketing separation and volume quotas for multiple mini-apps and tenants.
15. Build a ranking/review policy fixture with paid placement, editorial placement, incentivized feedback, fraud signals and appeal records; obtain Legal/Trust & Safety sign-off for target markets.

## Decision log

- The report preserves vendor practice as vendor evidence, not a universal standard.
- Design proposals are explicitly labeled and do not claim normative status.
- Credentials in the input spreadsheet were not copied to state, report or repository.
