# Research Gaps and Next Validation Plan

## Evidence gaps

- No single cross-platform normative standard covers mini-app registration, store review, rollout, ranking and deprecation.
- Public documentation is incomplete for Gojek marketplace taxonomy/search/ranking/reviews/analytics and for Kakao/Grab public marketplace behavior.
- Vendor-specific WindVane/uni-app manifest fields, JSAPI permission taxonomy, host SDK compatibility matrix and package limits require direct POC validation.
- Publisher KYC, payout, tax, content policy, privacy residency and retention depend on jurisdiction and business model.
- Exact rollout percentages, SLOs, package-size ceilings, vulnerability severity cutoffs and rollback timing require telemetry/threat-model calibration.
- The cited web/cache APIs provide primitives but not a cross-runtime mini-app offline contract; native WindVane/uni-app cache layers, invalidation, stale serving and offline launch behavior remain POC-specific unknowns.
- Payment provider, merchant-of-record, entitlement and refund/chargeback ownership are not determined by generic mini-app standards; they require a jurisdiction/provider contract and reconciliation tests.
- Commercial settlement remains provider- and jurisdiction-dependent: merchant-of-record/principal-agent role, tax and FX policy, payout cadence, reserve duration, dispute liability, accounting basis, and bank-statement mapping need Finance/Tax/Legal decisions. Apple/Google/Stripe/Adyen evidence is adapter input, not a universal contract.
- Identity/session policy is not universal: the RFCs and W3C specifications do not select an identity provider, publisher KYC model, recovery SLA, session TTL, legal account-merging policy, or revocation-propagation target. These require Security, Privacy, Legal and product decisions.
- The POC must test browser authorization return, issuer/redirect mismatch rejection, native/passkey availability, callback replay handling, DPoP nonce/jti replay, audience confusion, opaque result-handle reuse, logout propagation and uninstall/takedown propagation on supported host versions.
- The host bridge is not yet mapped to the actual WindVane/Alibaba JSAPI contract: origin/frame metadata, handler/port lifecycle, request/reply/error format, cancellation and timeout semantics, navigation teardown, renderer failure, and Android/iOS/Flutter parity require real-host tests. WHATWG, Android and Apple sources provide primitives and platform behavior, not a universal bridge standard.
- The POC's deep-link association, route/parameter allowlist and Android/iOS fallback behavior must be tested on real host versions; public web/app-link specifications do not prove WindVane support.
- Ranking transparency, incentivized-review rules and notification consent/volume limits are jurisdiction- and channel-specific; Legal, Privacy and Trust & Safety must bind the applicable policy profile.
- Accessibility evidence is still a control profile, not proof that the Alibaba/WindVane host bridge and every partner mini-app meet WCAG; native accessibility-tree, keyboard, screen-reader, text-scale and reduced-motion tests are required.
- Privacy obligations, controller/processor roles, retention periods, child-directed handling and age assurance remain jurisdiction- and product-specific; EDPB/ICO/NIST sources provide controls and decision points, not a universal legal answer.
- DSA/P2B notice, appeal, mediation and transparency duties depend on EU scope, provider category and exemptions; Google Play enforcement is platform-specific. Legal must map the target store model before adopting thresholds or SLA claims.
- Age-rating and child-safety evidence was previously Google Play/Android-specific; Iteration 26 established full cross-platform parity by integrating Apple App Store Review Guidelines §4.7/§1.3/§5.1.4, UK ICO 15 Children's Code statutory standards, and the International Age Rating Coalition (IARC) global multi-authority federation.
- Resource-governance evidence is a synthesis of SIP overload-control precedents, browser storage primitives, provider guidance and security/accessibility guidance; it does not select universal mini-app quotas, fairness weights, eviction byte limits, retry thresholds, spend caps, or abuse-detector accuracy targets.
- The host must validate storage partition identity, quota accounting, eviction order, clear-data semantics, transaction rollback and power-loss durability across WindVane/uni-app Android, iOS and Flutter implementations; browser-origin behavior is not proof of native-host behavior.
- Abuse-control pilots must measure goodput, queue age, starvation, retry amplification, false positives, accessibility-equivalent paths, shared-device/NAT fairness, provider timeout ambiguity, idempotency-key expiry and reconciliation lag before hard enforcement.
- The host must test child-safe runtime enforcement, parental approval/revocation propagation, and direct-launch bypass resistance; catalog declarations alone do not prove that capabilities, SDKs, social flows, ads, or precise location are actually blocked.

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
14. Build the settlement close fixture: catalog/product diff, provider event replay, daily estimates, monthly report, fees/taxes/refunds, reserves/disputes, publisher payout and bank statement; verify UTC cut-off, watermarks, control totals, late-data exceptions and compensating entries.
15. Test notification opt-in, revoke, quiet hours, transactional/marketing separation and volume quotas for multiple mini-apps and tenants.
16. Build a ranking/review policy fixture with paid placement, editorial placement, incentivized feedback, fraud signals and appeal records; obtain Legal/Trust & Safety sign-off for target markets.
17. Build the identity/session conformance fixture: PAR one-time use, issuer-bound callback, explicit account-link confirmation, host-only WebAuthn ceremony, step-up challenge binding, scoped token exchange, audience/resource validation, DPoP replay, opaque handle redemption, and fail-closed revocation across logout/uninstall/quarantine/takedown.
18. Build the host-bridge conformance fixture: wrong-origin/source/frame/handler/world rejection, wildcard rejection for sensitive calls, malformed/oversized/unsupported payloads, structured-clone failure, duplicate/late reply, timeout/cancel race, transferred-port reuse, Android legacy bridge exposure, Android renderer/SSL/Safe Browsing failure, Apple handler removal/content-world mismatch, navigation teardown, and exactly-once side effects under retry.
19. Build the resource-governance fixture: graded overload levels, weighted/fair-share principals, legacy/non-participating clients, health-checked redirect, retry amplification, queue aging/starvation, storage partition/quota/eviction/persistence behavior, atomic quota-failure rollback, adaptive friction and accessible alternatives, provider idempotency/replay/timeout, spend reserve exhaustion, alerts-versus-enforcement, appeal and reconciliation paths.

## Decision log

- The report preserves vendor practice as vendor evidence, not a universal standard.
- Design proposals are explicitly labeled and do not claim normative status.
- Credentials in the input spreadsheet were not copied to state, report or repository.
