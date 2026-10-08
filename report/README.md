# Mini App Store Standard for a Super App

**Research status:** Deli Deep iteration 25 (Major Milestone)
**Evidence records:** 326
**Iteration 25 additions:** 15 concurrency synchronization, background execution & device power governance findings / 11 unique URLs rechecked HTTP 200; 0 candidate URL overlap with canonical findings
**Credential exclusion:** true; Google Sheet credentials and raw sheet content are not included
**Input context:** Alibaba Cloud SuperApp/WindVane POC resource sheet, tab `Software Info` (`gid=58086397`)

## Reading order

| # | File | Purpose |
|---:|---|---|
| 1 | [Executive summary](01-executive-summary.md) | Decision-level conclusions and standard shape |
| 2 | [Reference architecture and lifecycle](02-reference-architecture-lifecycle.md) | Control/data planes, catalog model, release state machine |
| 3 | [Security, privacy and control matrix](03-security-privacy-control-matrix.md) | P0/P1/P2 controls and evidence required |
| 4 | [Store UX, benchmark and operating model](04-store-ux-benchmark-operating-model.md) | Discovery, trust, analytics, monetization and support |
| 5 | [Evidence appendix](05-evidence-appendix.md) | All validated findings and URLs, without omission |
| 6 | [Research gaps and next validation](06-gaps-validation-plan.md) | Unknowns, pilot tests and decisions still required |
| 7 | [Iteration 2: launch, offline and trust](07-iteration-2-launch-offline-trust.md) | 36 detailed findings on deep links, resilience, payments, notifications and marketplace trust |
| 8 | [Iteration 3: accessibility, privacy and governance](08-iteration-3-accessibility-privacy-governance.md) | 20 normalized findings on inclusive UX, data governance, moderation, appeals and procedural controls |
| 9 | [Iteration 4: reliability, SLO and operations](09-iteration-4-reliability-slo-operations.md) | 10 findings on SLOs, error budgets, canary evidence, incident learning, tenant isolation, recovery objectives and telemetry schema governance |
| 10 | [Iteration 5: portable catalog and evidence federation](10-iteration-5-portable-catalog-and-evidence-federation.md) | 12 findings on W3C/Schema.org catalog projections, SPDX/CycloneDX/OpenVEX/in-toto evidence exchange, and OAuth publisher capability negotiation |
| 11 | [Iteration 6: runtime resource and cost governance](11-iteration-6-resource-cost-governance.md) | 13 findings on quotas, tenant isolation, abuse resistance, cost attribution and shared/idle cost visibility |
| 12 | [Iteration 7: localization and regionalization](12-iteration-7-localization-regionalization.md) | 10 findings on BCP 47/CLDR locale contracts, HTTP language negotiation, regional formatting and privacy-aware fallback |
| 13 | [Iteration 14: age rating and child safety](13-iteration-14-age-rating-child-safety.md) | 5 findings on audience declarations, content ratings, child/mixed-audience capability policies, release re-review and host-only age signals |
| 14 | [Iteration 15: commercial settlement governance](14-iteration-15-commercial-settlement-governance.md) | 17 findings on commerce eligibility/product identity, subscription/refund reconciliation, publisher payout evidence, reserves/disputes and provider-to-bank settlement |
| 15 | [Iteration 18: identity, authentication and session contracts](15-iteration-18-identity-authentication-session.md) | 9 findings on OAuth authorization integrity, issuer/account binding, WebAuthn host boundaries, passkey step-up, token exchange, JWT audience validation, DPoP replay resistance and revocation |
| 16 | [Iteration 19: host bridge and cross-context messaging](16-iteration-19-host-bridge-cross-context.md) | 12 findings on origin/recipient binding, structured clone limits, structured errors, JSON-RPC correlation, Android WebView bridge hardening, and Apple WKWebView content-world/reply/navigation controls |
| 17 | [Iteration 20: resource governance and abuse resistance](17-iteration-20-resource-governance-abuse-resistance.md) | 14 findings on graded overload/fairness, storage partition/quota/eviction, adaptive abuse controls, provider idempotency, and hard-spend boundaries |
| 18 | [Iteration 22: W3C MiniApp, TUF & device governance](18-iteration-22-w3c-miniapp-tuf-device-governance.md) | 16 findings on W3C MiniApp core standards (Manifest, Packaging, Addressing, Lifecycle, Dual-Thread Architecture), TUF/SLSA/Sigstore package supply-chain security, and W3C Permissions Policy device capabilities governance |
| 19 | [Iteration 23: sandbox storage, network egress CSP and OS integration](19-iteration-23-sandbox-storage-network-csp-os-integration.md) | 15 findings on WHATWG OPFS filesystem sandbox, WeChat storage tiers, W3C CSP3 connect-src/script-src, WeChat network domain allowlist, OWASP MASVS-STORAGE-1/NETWORK-1, and Web Share/Badging/Contact Picker APIs |
| 20 | [Iteration 24: cryptographic hardware security, client payment mediation & runtime telemetry](20-iteration-24-crypto-hardware-payment-telemetry.md) | 14 findings on WebCrypto non-extractable keys, Android Keystore StrongBox/KeyMint attestation, Apple Secure Enclave Keychain access, OWASP MASVS-CRYPTO-1/2, W3C Payment Request lifecycle, Permissions Policy payment delegation, Secure Payment Confirmation (SPC), Payment Method Manifest, Paint Timing (FP/FCP), Largest Contentful Paint (LCP), Event Timing (INP), Reporting API, and Network Error Logging (NEL) |
| 21 | [Iteration 25: concurrency synchronization, background execution & device power governance](21-iteration-25-concurrency-background-power-governance.md) | 15 findings on W3C Web Locks API (exclusive/shared arbitration, steal, ifAvailable), WHATWG Web Messaging (origin binding, MessageChannel/MessagePort transferability), WICG Background Sync/Periodic Sync/Background Fetch, W3C Media Session API, Screen Wake Lock API, and Battery Status/Power Governance |

## Evidence rules

- `primary-standard`, `primary-government`, and `primary-official` are normative or official primary evidence.
- `official-vendor`, `official-platform-documentation`, and related labels describe vendor/platform practice, not a universal standard.
- `proposal` is a design recommendation derived from the evidence; it is not presented as a source requirement.
- `gap` means the public evidence was insufficient; it is not filled with a guess.
- Source credentials from the input spreadsheet were deliberately excluded.

## State

The append-only research state is in `../code/deli/super-app-mini-app-store-standard/state/` on the host. The repository publication should include a sanitized copy only.
