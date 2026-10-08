# Executive Summary

**Current evidence base:** 401 validated append-only findings through Deli Deep iteration 30 (Milestone 30). Iteration 30 adds 15 next-generation transport, WebGPU compute sandboxing, and multimodal spatial computing findings from 23 HTTP-200-verified primary URLs; all previous iteration findings remain intact in their respective sections.

## Decision

A mini app store in a super app should be treated as a **trusted software supply-chain and lifecycle control plane**, not as a collection of marketing screens. The minimum viable standard must control publisher identity, immutable packages, capability declarations, automated checks, human review, staged rollout, rollback, audit evidence, and deprecation. Search, categories and ranking matter, but they must never outrank trust, compatibility or policy state.

The Alibaba POC context is a good fit for this model: its documentation separates the miniapp container, Application Open Platform, and Miniapp Backend; its documented flow includes build/package, review, gray-scale rollout and official release. That is a useful platform baseline, not by itself a complete enterprise standard.

## Five conclusions

1. **Lifecycle is the backbone.** Model publisher → app → version → submission → review → release → rollout → active/deprecated/withdrawn as explicit resources with immutable IDs and append-only audit events.
2. **Trust must be machine-readable.** Every release needs a digest, signature/provenance/SBOM references, host compatibility, capabilities, data declarations, review decision and rollback target.
3. **Permissions are store metadata.** The catalog must expose native capabilities and data access before installation/activation; runtime authorization must be enforced by the host, not by a listing promise.
4. **Progressive delivery is mandatory for enterprise.** Preview, canary/gray release, cohort targeting, pause and last-known-good rollback are release controls, not optional DevOps enhancements.
5. **No universal public mini-app-store standard exists.** Vendor ecosystems provide patterns, while OWASP/NIST/W3C/IETF/OCI/SLSA/Sigstore/TUF provide reusable control foundations. The organization must define its own profile and test it in a pilot.

## Recommended target profile

### P0 — before any external or broad release

- Verified publisher identity, MFA, RBAC and abuse/security contact.
- Host-owned, capability-scoped bridge contract: exact origin/recipient/frame binding, versioned bounded envelopes, structured machine-readable errors, request correlation/idempotency, deadlines/cancellation, and fail-closed teardown on navigation, renderer, quarantine or revocation events.
- Immutable, content-addressed package with declared app ID, version, digest, entry point, host/runtime range and capabilities.
- Signature verification and basic provenance/SBOM/secret/malware checks.
- Manifest/schema/size/path validation and deterministic compatibility rejection before download.
- Manual policy/security review with decision evidence and appeal/rework path.
- Preview plus cohort/percentage rollout, pause, and rollback to a last-known-good digest.
- Catalog/submission/release APIs with idempotency keys, cursor pagination and audit events.
- Privacy notice, consent/data-use declarations, retention owner and accessible core store flows.
- Accessibility acceptance evidence for catalog/search/detail/install/update/permission/error/takedown flows: reflow, text resizing, keyboard and assistive-technology semantics, visible focus, non-color status, reduced motion and localized language metadata.
- Versioned data/telemetry profile, purpose/minimization map, consent withdrawal where applicable, deletion/export propagation, processor registry and high-risk privacy review decision.
- Versioned target-audience and content-rating declarations, child/mixed-audience capability profile, host-enforced safe mode for unresolved age state, and mandatory re-review when content, ads, social features, permissions, SDKs or data practices change.
- Stable app identity and verified HTTPS launch contract: explicit route/parameter allowlists, safe web fallback, and transaction-bound authentication context.
- Host/backend-controlled payment and entitlement boundary; mini-apps receive only opaque or verified results, never payment credentials or de-tokenization authority.
- Per-mini-app/per-purpose notification consent, unsubscribe/quiet controls and abuse rate limits; notification delivery must not be implied by install.

### P1 — enterprise scale

- Reproducible-build verification, HSM-backed keys, delegated namespaces, multi-region signed mirrors and tenant-private catalogs.
- Automated re-review on dependency/CVE/provenance changes.
- OpenTelemetry-based observability with redaction and retention controls.
- Incident quarantine, trust revocation, cache invalidation, user/operator notice and postmortem evidence.
- Localization and regionalization contract: validate BCP 47 tags, pin CLDR/translation revisions, publish deterministic fallback and locale coverage, keep language/script/region/currency/market separate, make HTTP cache variation explicit, and retain typed machine values for currency/time zones.
- Accessibility conformance evidence, support SLAs and entitlement/monetization controls.
- Per-artifact cache/freshness/invalidation policy with explicit offline fallback eligibility; measure cold/warm/offline performance by cohort and digest.
- Commercial settlement contract: immutable host/provider product identity mapping, provider-event reconciliation, explicit refund/void/dispute/reserve states, publisher payout evidence, report-period close and bank-settlement control totals; keep provider-specific rules in adapters.
- Identity and session contract: issuer-bound authorization requests, host-owned WebAuthn RP/origin policy, explicit account-linking confirmation, audience/resource-bound capabilities, sender-constrained high-risk sessions, opaque result handles, replay detection and revocation on logout/uninstall/quarantine/takedown.
- Ranking/review integrity controls: disclose applicable ranking parameters, separate editorial/paid placement, preserve review provenance and provide moderation/appeal evidence.
- Market-specific moderation governance: notice-and-action intake, reasoned decisions, proportionate suspension, publisher complaint/mediation, and transparent appeal records; label EU DSA/P2B and platform-policy scope rather than treating them as universal law.
- Operational reliability and resource-governance contract: versioned SLO/SLI definitions, error-budget release policy, immutable cohort evidence, rollback target, incident command/postmortem workflow, telemetry schema compatibility, tenant/app quotas and layered isolation, graded fair-share overload control, storage durability/eviction classes, operation-risk friction, provider idempotency, hard-spend reserves, and tested MTD/RTO/RPO by criticality tier.
- Portable federation contract: versioned manifest/Schema.org projections, digest-bound SPDX/CycloneDX/VEX/attestation references, explicit freshness/trust/review state, and machine-readable publisher registration/resource-capability negotiation.

### P2 — optimization

- Ranking/merchandising with abuse resistance and explainable signals.
- Offline/poor-network cache policy, quotas/cost attribution and advanced ecosystem analytics.

## Fit to the Alibaba POC

The POC should be assessed against these gates:

- **Keep:** Application Open Platform as lifecycle/control plane; WindVane container as runtime; build/package/review/gray/full release flow; native capability authorization.
- **Add around it:** signed catalog/artifact verification, publisher/RBAC/KYC policy, capability/data manifest, independent security gates, immutable audit ledger, explicit rollback/quarantine/deprecation, compatibility resolver, store analytics and accessibility evidence.
- **Validate with vendor:** exact manifest fields, JSAPI permission taxonomy, host SDK compatibility matrix, package limits, release API semantics, cache invalidation and rollback behavior.

## Recommendation

Approve a pilot that implements the P0 profile for 3–5 representative miniapps, including one sensitive-data app and one partner-owned app. Do not standardize ranking or monetization until trust, compatibility, review and rollback evidence is measurable. For identity-sensitive mini-apps, add the iteration-18 authentication/session conformance fixture before external onboarding. For any WebView/native mini-app, add the iteration-19 bridge conformance fixture before granting privileged host capabilities.
