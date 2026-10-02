# Store UX, Benchmark and Operating Model

## 1. Catalog and discovery

Minimum listing metadata: name, icon, localized short/long description, category/tags, publisher identity/trust state, version, last update, supported host/platform/locale, capabilities, data-use summary, privacy/support links, screenshots, content rating, install/activation entitlement, known limitations and release notes.

Search and ranking should separate relevance, quality, freshness, trust and merchandising. Ranking signals must be explainable to operators and resistant to click/install fraud. A policy/security block must override ranking. Featured placement is a distribution decision, not a security decision.

## 2. Trust UX

Display: verified publisher, review tier/date, capability badges, data-use summary, permission prompts, update notes, support contact, incident/deprecation banner and compatibility reason. Make “why cannot install” actionable: host version, missing capability, region/tenant policy, suspended release or required update.

## 3. Benchmark synthesis

- **WeChat/Alipay:** strong platform-controlled registration, permission/consent and review patterns; useful for capability declarations, developer governance and controlled release.
- **LINE:** useful evidence for app lifecycle/deprecation and developer-facing APIs; explicit public deprecation guidance is stronger than many peers.
- **Microsoft Teams Store/org distribution:** useful enterprise pattern for admin-controlled availability, organizational distribution and app governance.
- **Grab:** useful partner/super-app SDK and integration pattern, but public evidence does not expose a complete catalog/ranking/review specification.
- **Gojek/Kakao:** public materials validate API/partner/permission/telemetry patterns but do not establish a public mini-app marketplace model.

Do not infer missing marketplace behavior from platform marketing. Where public evidence did not show ranking, reviews, catalog taxonomy or analytics, the report records a gap.

## 4. Operating model

| Function | Owner | Key outputs |
|---|---|---|
| Ecosystem policy | Product + Legal + Security | eligibility, content/privacy/security policy, risk tiers |
| Publisher operations | Developer relations | onboarding, namespace, support, appeals, SLA |
| Review | Trust & Safety + Security + Privacy | decision, findings, evidence, remediation |
| Release engineering | Platform/SRE | artifact, rollout, health gates, rollback |
| Store merchandising | Product | taxonomy, search, ranking, featured policy |
| Incident response | SOC/SRE/Policy | quarantine, notice, recovery, postmortem |
| Data governance | Privacy/Data Office | telemetry minimization, retention, regional controls |

## 5. Analytics

Track funnel and reliability without over-collecting user identity: impression → detail view → eligibility check → install/download → verification → launch → activation → task success; plus startup latency, crash/error rate, API failures, permission denial, rollback, quarantine and support contacts. Segment by app/release/digest/cohort/region/host SDK/device class.

## 6. Monetization and entitlements

Treat paid access, subscriptions, partner revenue share and entitlements as a separate policy domain. The store needs clear owner, price/currency/tax/refund/renewal, entitlement state, server-side verification and deactivation behavior. Do not let payment status become a client-only capability.

## 7. Localization and support

Localize listing metadata, permission explanations, policy/error messages, support and legal notices. Keep locale availability in compatibility resolution. Define support ownership for host, miniapp, backend, payment and data incidents; expose a correlation/release ID so support can reproduce a version-specific issue.
