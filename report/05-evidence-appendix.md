# Evidence Appendix

Total validated records: **108**. Every finding below is retained from the append-only state.

## accessibility
### 1. keyboard_accessibility — proposal
- **Source:** W3C WCAG 2.2 SC 2.1.1 Keyboard
- **Detail:** WCAG 2.2 Success Criterion 2.1.1 requires all functionality to be operable through a keyboard interface, except path-dependent freehand movement. Proposal: make catalogue browse/search/filter, package details, install/update, permission consent, accessibility settings, publisher review, and incident-report workflows keyboard-operable and testable without timing traps.
- **URL(s):** [https://www.w3.org/WAI/WCAG22/Understanding/keyboard](https://www.w3.org/WAI/WCAG22/Understanding/keyboard)

### 2. accessible_component_semantics — proposal
- **Source:** W3C WCAG 2.2 SC 4.1.2 Name Role Value
- **Detail:** WCAG 2.2 Success Criterion 4.1.2 requires names and roles to be programmatically determinable, user-settable states/properties/values to be programmatically set, and changes to be available to assistive technology. Proposal: require accessible names, roles, states, values, and live announcements for custom app cards, filters, permission dialogs, progress/status indicators, errors, and install/update results, verified with accessibility-tree and screen-reader tests.
- **URL(s):** [https://www.w3.org/WAI/WCAG22/Understanding/name-role-value](https://www.w3.org/WAI/WCAG22/Understanding/name-role-value)

### 3. accessible_contrast — proposal
- **Source:** W3C WCAG 2.2 SC 1.4.3 Contrast Minimum
- **Detail:** WCAG 2.2 Success Criterion 1.4.3 sets a minimum contrast ratio of 4.5:1 for normal text and 3:1 for large text, with stated exceptions. Proposal: enforce these thresholds for catalogue text, metadata, focus/error states, permission warnings, and update/takedown notices across themes; do not use color alone to communicate risk or status; include automated contrast checks and manual review.
- **URL(s):** [https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum)

## analytics_operations
### 1. observed_capability — primary_official
- **Source:** Kakao Developers official documentation
- **Detail:** Observed capability: Kakao's app-management Statistics page provides real-time API usage statistics with optional auto-refresh every minute, including successful and failed calls by category and by API. Recommended standard (separate): expose freshness and success/failure dimensions for operator analytics, but do not treat API telemetry as a proxy for user ratings or discovery ranking.
- **URL(s):** [https://developers.kakao.com/docs/en/getting-started/stat](https://developers.kakao.com/docs/en/getting-started/stat)

## analytics_quality_lifecycle
### 1. observed_capability — primary_official
- **Source:** WeChat official developer docs
- **Detail:** Observed capability: Weixin Experience Rating checks mini-program experience in real time, analyzes likely causes, locates problems, and gives optimization suggestions; automated runs warn developers when the score is below 70. Recommended standard (separate): expose pre-release and runtime quality signals, but label them as operational quality analytics rather than user-review or store-ranking data.
- **URL(s):** [https://developers.weixin.qq.com/miniprogram/en/dev/framework/audits/audits.html](https://developers.weixin.qq.com/miniprogram/en/dev/framework/audits/audits.html)

## analytics_support_lifecycle
### 1. observed_capability — primary_official
- **Source:** Microsoft official Teams documentation
- **Detail:** Observed capability: after approval, Microsoft provides Teams app usage reporting with monthly, daily, and weekly active users plus retention and intensity charts; submissions receive validation reports, remediation guidance, and concierge support, and can be resubmitted until required issues are resolved. Recommended standard (separate): combine lifecycle/support telemetry with adoption and retention metrics, and make remediation and revalidation first-class states.
- **URL(s):** [https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/deploy-and-publish/appsource/publish](https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/deploy-and-publish/appsource/publish)

## api_authentication
### 1. oauth_pkce_for_miniapps — proposal
- **Source:** RFC 9700 OAuth 2.0 Security BCP
- **Detail:** RFC 9700 states that public clients must use PKCE, recommends S256 because it does not expose the verifier, and requires transaction-specific binding; authorization servers must support and enforce correct verifier use. Proposal: require mini-app public clients to use the authorization-code flow with S256 PKCE, one-time transaction state, exact client binding, scoped/audience-limited tokens, and no implicit grant for store APIs.
- **URL(s):** [https://www.rfc-editor.org/rfc/rfc9700.html](https://www.rfc-editor.org/rfc/rfc9700.html)

### 2. oauth_redirect_uri_integrity — proposal
- **Source:** RFC 9700 OAuth 2.0 Security BCP
- **Detail:** RFC 9700 requires exact string matching against pre-registered redirect URIs, except for localhost ports in native apps, and says clients and authorization servers must not expose open redirectors. Proposal: register redirect URIs per mini-app and issuer, reject wildcard or mismatch cases, prohibit query-parameter forwarding to arbitrary destinations, and audit redirect failures.
- **URL(s):** [https://www.rfc-editor.org/rfc/rfc9700.html](https://www.rfc-editor.org/rfc/rfc9700.html)

### 3. sender_constrained_tokens — proposal
- **Source:** RFC 9449 OAuth 2.0 Demonstrating Proof of Possession
- **Detail:** RFC 9449 explains that DPoP sender-constrains an access token to the party holding the private key, reducing replay impact compared with bearer tokens, while requiring HTTPS. Proposal: use DPoP or an equivalent sender-constrained mechanism for publisher, reviewer, administrator, and other high-risk mini-app APIs; verify proofs, nonce/replay protections, key binding, and HTTPS at the gateway.
- **URL(s):** [https://www.rfc-editor.org/rfc/rfc9449.html](https://www.rfc-editor.org/rfc/rfc9449.html)

## api_authorization
### 1. object_level_authorization — proposal
- **Source:** OWASP API Security Top 10 API1:2023
- **Detail:** OWASP API1:2023 says the server must check whether the logged-in user may perform the requested action on the referenced record in every function, and recommends authorization tests. Proposal: enforce server-side tenant, organization, publisher, reviewer, and end-user authorization for every package, release, review, install, and analytics object; use opaque identifiers and negative ID-tampering tests.
- **URL(s):** [https://api-security.owasp.org/editions/2023/en/0xa1-broken-object-level-authorization/](https://api-security.owasp.org/editions/2023/en/0xa1-broken-object-level-authorization/)

## api_contract
### 1. sourced_finding — primary-standard
- **Source:** OpenAPI Initiative, OpenAPI Specification v3.1.1
- **Detail:** OpenAPI defines a standard, programming-language-agnostic interface description for HTTP APIs, enabling humans and computers to discover and understand service capabilities without source-code access or network inspection. The store control plane should publish its catalog, submission, review, release, and publisher APIs as an OpenAPI contract.
- **URL(s):** [https://spec.openapis.org/oas/v3.1.1.html](https://spec.openapis.org/oas/v3.1.1.html)

## api_data_minimization
### 1. property_level_authorization — proposal
- **Source:** OWASP API Security Top 10 API3:2023
- **Detail:** OWASP API3:2023 recommends exposing only properties the user may access, avoiding generic serialization and mass assignment, validating responses against schemas, and keeping returned structures minimal. Proposal: define field-level allowlists for store APIs, reject unknown or immutable write fields, validate responses, and return only the package metadata needed by each role and tenant.
- **URL(s):** [https://api-security.owasp.org/editions/2023/en/0xa3-broken-object-property-level-authorization/](https://api-security.owasp.org/editions/2023/en/0xa3-broken-object-property-level-authorization/)

## api_event_contract
### 1. design_proposal — proposal
- **Source:** Reference architecture synthesis
- **Detail:** [PROPOSAL] Publish a versioned OpenAPI control-plane contract with cursor pagination and ETags for catalog reads; idempotency keys for submissions and release actions; explicit problem details and state-transition errors; and role-scoped OAuth/OIDC authorization. Minimum resources: /publishers, /apps, /apps/{id}/versions, /submissions, /reviews, /releases, /compatibility, /advisories, /incidents, and /deprecations. Emit CloudEvents for submission.accepted, review.completed, release.promoted, release.paused, release.aborted, package.quarantined, advisory.published, and app.deprecated.
- **URL(s):** [https://spec.openapis.org/oas/v3.1.1.html](https://spec.openapis.org/oas/v3.1.1.html), [https://raw.githubusercontent.com/cloudevents/spec/v1.0.2/cloudevents/spec.md](https://raw.githubusercontent.com/cloudevents/spec/v1.0.2/cloudevents/spec.md)

## api_governance
### 1. api_hardening_baseline — proposal
- **Source:** OWASP API Security Top 10 API8:2023
- **Detail:** OWASP API8:2023 calls for repeatable hardening, continuous assessment, TLS for client, upstream, and downstream communication, explicit HTTP-method allowlists, correct CORS and security headers, restricted content types, and uniform request handling. Proposal: enforce a versioned API-gateway baseline, continuously scan configuration drift, require TLS and enterprise mTLS where appropriate, disable unused verbs, restrict origins/content types, and standardize error responses.
- **URL(s):** [https://api-security.owasp.org/editions/2023/en/0xa8-security-misconfiguration/](https://api-security.owasp.org/editions/2023/en/0xa8-security-misconfiguration/)

### 2. api_inventory_version_retirement — proposal
- **Source:** OWASP API Security Top 10 API9:2023
- **Detail:** OWASP API9:2023 calls for an inventory of API hosts, environments, versions, access, integrated services, data flows, documentation, and retirement strategy; it warns against leaving old versions exposed and against production data in non-production deployments. Proposal: maintain a signed API catalog with owners, data classification, scopes, versions, deprecation dates, and test/staging boundaries; block undocumented or retired endpoints.
- **URL(s):** [https://api-security.owasp.org/editions/2023/en/0xa9-improper-inventory-management/](https://api-security.owasp.org/editions/2023/en/0xa9-improper-inventory-management/)

### 3. api_input_integrity — proposal
- **Source:** OWASP ASVS v4.0.3 V13.2.2/V13.2.6
- **Detail:** ASVS V13.2.2 requires JSON schema validation before accepting input; V13.2.6 requires message headers and payloads to be trustworthy and not modified in transit, with TLS generally sufficient and signatures an option for high-security cases. Proposal: validate package, permission, review, and publisher API payloads against schemas before processing; require TLS everywhere and message signing for high-risk administrative or release actions.
- **URL(s):** [https://raw.githubusercontent.com/OWASP/ASVS/v4.0.3/4.0/en/0x21-V13-API.md](https://raw.githubusercontent.com/OWASP/ASVS/v4.0.3/4.0/en/0x21-V13-API.md)

## architecture
### 1. design_proposal — proposal
- **Source:** Reference architecture synthesis
- **Detail:** [PROPOSAL] Use two planes and explicit trust boundaries. Control plane: publisher identity/RBAC, submission API, policy engine, review queue, OCI-compatible registry, signed catalog/index, release orchestrator, audit ledger, incident/deprecation service. Data plane: host SDK/runtime, compatibility resolver, signature/provenance verifier, cache/CDN, capability broker, and telemetry exporter. Treat publisher CI, registry, catalog signer, CDN/cache, host runtime, and third-party APIs as separate trust boundaries; never let a package call native capabilities without host-mediated policy.
- **URL(s):** [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp), [https://raw.githubusercontent.com/opencontainers/distribution-spec/main/spec.md](https://raw.githubusercontent.com/opencontainers/distribution-spec/main/spec.md), [https://mas.owasp.org/MASVS/](https://mas.owasp.org/MASVS/)

### 2. control_plane_separation — official-vendor
- **Source:** Alibaba Cloud Documentation
- **Detail:** Alibaba distinguishes SuperApp Open Platform (core capabilities/content/data/services), Application Open Platform (miniapp developer onboarding and lifecycle), and Miniapp Backend (data APIs integrated with the open platform). A reusable architecture should separate these control planes and responsibilities.
- **URL(s):** [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/basic-concepts](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/basic-concepts)

## artifact_integrity
### 1. sourced_finding — primary-standard
- **Source:** Open Container Initiative, Image Manifest Specification
- **Detail:** The OCI image manifest is designed for content-addressable artifacts and uses descriptors for configuration and layers. Its optional subject links a related manifest and is used by the referrers API, enabling signatures, attestations, SBOMs, and other metadata to be attached without mutating the released artifact.
- **URL(s):** [https://raw.githubusercontent.com/opencontainers/image-spec/main/manifest.md](https://raw.githubusercontent.com/opencontainers/image-spec/main/manifest.md)

## auditability
### 1. audit_logging_and_redaction — proposal
- **Source:** OWASP ASVS v4.0.3 V7
- **Detail:** ASVS V7 says not to log credentials, payment details, or sensitive data; to log security-relevant authentication, access-control, deserialization, and validation events with investigation-ready timeline data; and to protect logs from unauthorized access or modification. Proposal: emit tamper-evident audit events for submission, review, approval, install, update, permission grant/revoke, quarantine, and takedown with actor, tenant, package/version, timestamp, result, and correlation ID, while redacting secrets and retaining logs in restricted remote storage.
- **URL(s):** [https://raw.githubusercontent.com/OWASP/ASVS/v4.0.3/4.0/en/0x15-V7-Error-Logging.md](https://raw.githubusercontent.com/OWASP/ASVS/v4.0.3/4.0/en/0x15-V7-Error-Logging.md)

## catalog_api
### 1. sourced_finding — primary-vendor
- **Source:** Alibaba Cloud, miniapp SDK operations
- **Detail:** The SDK documentation exposes list, search, open, preview, and preload operations. Listing is paginated with an anchor and defaults to a maximum of 10 results, while advanced requirements can use the SuperApp OpenAPI. This is a concrete catalog/runtime interaction pattern, not a reusable normative requirement.
- **URL(s):** [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/small-program-operation-2](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/small-program-operation-2)

## catalog_categories_permissions_analytics_monetization_lifecycle
### 1. observed_capability — primary_official
- **Source:** Alipay+ official platform documentation
- **Detail:** Observed capability: Alipay+ Mini Program Platform describes a centralized Service Market where ISVs publish services for merchants to browse, order, and select. It separates merchant and ISV roles/permissions; ISVs can publish services, manage authorizations, and handle the development lifecycle, while merchants manage create/review/release/update/takedown and receive performance analysis covering user experience and exceptions. Recommended standard (separate): model catalog, access control, lifecycle, and merchant/ISV analytics as separate surfaces.
- **URL(s):** [https://miniprogram.alipay.com/docs-alipayconnect/miniprogram_alipayconnect/platform/overview](https://miniprogram.alipay.com/docs-alipayconnect/miniprogram_alipayconnect/platform/overview)

## catalog_metadata_review_release_lifecycle
### 1. observed_capability — primary_official
- **Source:** Alipay official developer documentation
- **Detail:** Observed capability: Alipay's quick-start lifecycle asks an admin to submit mini-program particulars including name, description, and logo; the platform assigns a unique identifier, then runs review and release with workspace-admin approval. Marketing capabilities can be enabled, and the lifecycle ends when the mini-program is removed. Recommended standard (separate): require stable identity plus descriptive metadata before review, explicit approval ownership, marketing opt-in, and a terminal takedown state.
- **URL(s):** [https://miniprogram.alipay.com/docs/miniprogram/mpdev/quick-start_overview](https://miniprogram.alipay.com/docs/miniprogram/mpdev/quick-start_overview)

## catalog_search_ranking_reviews_analytics_lifecycle_gap
### 1. evidence_gap — primary_official_gap
- **Source:** Gojek official public developer pages
- **Detail:** Research gap: the fetched official GoSend API pages document partner API integration and operational support, but do not disclose a public mini-app catalog, category metadata, search/ranking rules, ratings/reviews, marketplace analytics, or an app submission/review lifecycle. This is a gap in publicly verified evidence, not proof that no internal capability exists; do not infer those fields for a standard without partner access or further primary documentation.
- **URL(s):** [https://www.gojek.com/en-id/gosend/api](https://www.gojek.com/en-id/gosend/api)

## catalog_search_ranking_reviews_monetization_gap
### 1. evidence_gap — primary_official_gap
- **Source:** Kakao Developers official documentation
- **Detail:** Research gap: Kakao's public developer documentation verifies app/API management, permission review, consent controls, and API statistics, but the reviewed public pages do not verify a KakaoTalk mini-app marketplace with public catalog metadata, category navigation, search/ranking rules, ratings/reviews, or marketplace monetization. Treat Kakao as an integration-platform benchmark for these fields until primary marketplace evidence is obtained.
- **URL(s):** [https://developers.kakao.com/docs/en](https://developers.kakao.com/docs/en)

## catalog_update_security
### 1. sourced_finding — primary-standard
- **Source:** The Update Framework, Specification 1.0.36
- **Detail:** TUF secures software update systems with signed metadata and distinct root, targets, snapshot, and timestamp roles, supporting delegated trust and threshold signatures. Its stated threat goals include detection/prevention of rollback, freeze, mix-and-match, wrong-software, and below-threshold key-compromise attacks. Target files remain opaque to TUF, so the pattern applies to miniapp packages and catalogs.
- **URL(s):** [https://theupdateframework.github.io/specification/latest/](https://theupdateframework.github.io/specification/latest/)

## ci_cd_security_review
### 1. design_proposal — proposal
- **Source:** Reference architecture synthesis
- **Detail:** [PROPOSAL] CI/CD acceptance gates should be deterministic and recorded per digest: manifest/schema validation; size/file/path/entrypoint limits; dependency and license policy; unit/integration/UI tests across the supported host SDK matrix; static, malware, secret, and dynamic sandbox checks; capability/network allowlist checks; SBOM generation; provenance generation; signature and identity verification; vulnerability policy; and human review for high-risk capabilities. A failed gate produces a non-publishable immutable submission, never a partially promoted package.
- **URL(s):** [https://csrc.nist.gov/pubs/sp/800/218/final](https://csrc.nist.gov/pubs/sp/800/218/final), [https://mas.owasp.org/MASVS/](https://mas.owasp.org/MASVS/), [https://slsa.dev/spec/v1.2/build-requirements](https://slsa.dev/spec/v1.2/build-requirements), [https://spdx.dev/use/specifications/](https://spdx.dev/use/specifications/)

## ci_cd_supply_chain
### 1. sourced_finding — primary-standard
- **Source:** SLSA, Version 1.2 specification and build requirements
- **Detail:** SLSA v1.2 describes provenance as verifiable information about where, when, and how an artifact was produced. Its build requirements assign responsibilities to the producer and build platform: consistent build process and distributed provenance; increasing levels require provenance existence, authenticity, and stronger isolation. Provenance must identify the output by cryptographic digest and describe production.
- **URL(s):** [https://slsa.dev/spec/v1.2/build-requirements](https://slsa.dev/spec/v1.2/build-requirements)

## comparable-ecosystem/runtime/auth/discovery
### 1. bot_webapp_runtime_auth_and_discovery — official-platform-documentation
- **Source:** Telegram Core Documentation — Telegram Mini Apps
- **Detail:** Telegram Mini Apps are web apps launched from bot web_app buttons, a bot menu button, direct links, and a configured Main Mini App in BotFather; the official documentation also describes media previews in the Telegram Mini App Store. The runtime is initialized with telegram-web-app.js. Telegram exposes raw initData for server validation and warns that initDataUnsafe must not be trusted. This is a comparable bot-mediated mini-app and discovery/authentication pattern, not a claim that Telegram uses the same package model as WeChat or Alipay.
- **URL(s):** [https://core.telegram.org/bots/webapps](https://core.telegram.org/bots/webapps)

## consent
### 1. consent_and_data_control — proposal
- **Source:** OWASP MASVS-PRIVACY-4
- **Detail:** MASVS-PRIVACY-4 calls for user mechanisms to manage, delete, and modify data and change privacy settings, including revoking consent; it also calls for re-prompting and updating disclosures when more data is needed. Proposal: provide a host permission center with per-scope revoke, deletion/export workflows, and mandatory re-review when a release expands data use.
- **URL(s):** [https://mas.owasp.org/MASVS/controls/MASVS-PRIVACY-4/](https://mas.owasp.org/MASVS/controls/MASVS-PRIVACY-4/)

### 2. consent_withdrawal_ux — proposal
- **Source:** EDPB Guidelines 4/2019 on Article 25
- **Detail:** The EDPB guidance describes consent as freely given, specific, informed, and unambiguous, and says withdrawal should be as easy as giving consent; it also warns against dark patterns and recommends equally visible consent/abstain choices. Proposal: require per-purpose, non-preselected consent with equal accept/decline controls, an always-available withdrawal path, propagation of revocation to SDKs and data stores, and evidence of the consent state.
- **URL(s):** [https://www.edpb.europa.eu/sites/default/files/files/file1/edpb_guidelines_201904_dataprotection_by_design_and_by_default_v2.0_en.pdf](https://www.edpb.europa.eu/sites/default/files/files/file1/edpb_guidelines_201904_dataprotection_by_design_and_by_default_v2.0_en.pdf)

## control_matrix
### 1. control_matrix — proposal
- **Source:** Prioritized control matrix
- **Detail:** [PROPOSAL] MVP/P0 must-have: verified publisher account plus MFA and RBAC; immutable versioned package and digest; signed artifact verification; manifest/schema/size/capability checks; basic CI tests and malware/secret scan; manual review; preview plus percentage/cohort rollout; one-click pause/disable/rollback to last-known-good; OpenAPI catalog/submission/release APIs; append-only audit log; basic launch/error/install telemetry; security contact and deprecation metadata. Enterprise/P1 adds: SSO/SCIM, legal-entity verification, maker-checker approvals, OIDC workload identity, SLSA provenance, SPDX SBOM, Sigstore transparency verification, TUF threshold/delegated catalog trust, risk-based manual review, region/tenant cohorts, policy-as-code, WORM audit retention, OpenTelemetry/SLO gates, incident command/runbooks, advisories, data residency/DPA controls, and periodic access recertification.
- **URL(s):** [https://docs.github.com/en/actions/concepts/security/openid-connect](https://docs.github.com/en/actions/concepts/security/openid-connect), [https://slsa.dev/spec/v1.2/build-requirements](https://slsa.dev/spec/v1.2/build-requirements), [https://docs.sigstore.dev/](https://docs.sigstore.dev/), [https://theupdateframework.github.io/specification/latest/](https://theupdateframework.github.io/specification/latest/), [https://csrc.nist.gov/pubs/sp/800/218/final](https://csrc.nist.gov/pubs/sp/800/218/final), [https://spdx.dev/use/specifications/](https://spdx.dev/use/specifications/)

### 2. control_matrix — proposal
- **Source:** Prioritized control matrix
- **Detail:** [PROPOSAL] Priority acceptance evidence: P0 release is publishable only when the package digest, publisher identity, host compatibility result, capability policy, automated-check results, reviewer decision, rollout target, and rollback target are queryable by release_id. Enterprise release is publishable only when the same record also links provenance/SBOM/signature verification, separation-of-duties approvals, cohort metrics/SLO decision, incident owner/runbook, retention class, and deprecation/withdrawal plan.
- **URL(s):** [https://spec.openapis.org/oas/v3.1.1.html](https://spec.openapis.org/oas/v3.1.1.html), [https://raw.githubusercontent.com/cloudevents/spec/v1.0.2/cloudevents/spec.md](https://raw.githubusercontent.com/cloudevents/spec/v1.0.2/cloudevents/spec.md), [https://opentelemetry.io/docs/concepts/observability-primer/](https://opentelemetry.io/docs/concepts/observability-primer/)

### 3. control_matrix — proposal
- **Source:** Prioritized control matrix
- **Detail:** [PROPOSAL] P2 scale controls after the MVP is stable: multi-region registry mirrors with signed metadata; offline/poor-network cache policy; delegated namespaces; HSM-backed catalog keys; reproducible-build verification; automated re-review on dependency/CVE changes; tenant-private catalogs; ranking/merchandising with abuse resistance; quotas/cost attribution; and publisher quality/reputation signals. Do not make these prerequisites for the first end-to-end store unless the deployment risk model requires them.
- **URL(s):** [https://theupdateframework.github.io/specification/latest/](https://theupdateframework.github.io/specification/latest/), [https://slsa.dev/spec/v1.2/build-requirements](https://slsa.dev/spec/v1.2/build-requirements), [https://raw.githubusercontent.com/opencontainers/distribution-spec/main/spec.md](https://raw.githubusercontent.com/opencontainers/distribution-spec/main/spec.md)

## deprecation
### 1. sourced_finding — primary-standard
- **Source:** IETF RFC 9745 and RFC 8594
- **Detail:** RFC 9745 defines a Deprecation response header and deprecation link relation to signal that a resource will be or has been deprecated and to point to migration information; deprecation itself does not change behavior. RFC 8594 defines Sunset for a URI likely to become unresponsive at a specified future time and explicitly distinguishes deprecation from final decommissioning.
- **URL(s):** [https://www.rfc-editor.org/rfc/rfc9745.html](https://www.rfc-editor.org/rfc/rfc9745.html), [https://www.rfc-editor.org/rfc/rfc8594.html](https://www.rfc-editor.org/rfc/rfc8594.html)

### 2. design_proposal — proposal
- **Source:** Reference architecture synthesis
- **Detail:** [PROPOSAL] Deprecation should be explicit and staged: active -> deprecated (new installs discouraged, successor/migration guide published) -> sunset_scheduled (date and scope visible) -> blocked (no new execution/installs) -> withdrawn/deleted only when legal, security, or retention policy permits. Include replacement, owner, notice date, sunset date, data-export/retention behavior, and compatibility impact in catalog metadata; expose HTTP Deprecation/Link signals for API resources and Sunset only when the resource will become unresponsive.
- **URL(s):** [https://www.rfc-editor.org/rfc/rfc9745.html](https://www.rfc-editor.org/rfc/rfc9745.html), [https://www.rfc-editor.org/rfc/rfc8594.html](https://www.rfc-editor.org/rfc/rfc8594.html), [https://semver.org/spec/v2.0.0.html](https://semver.org/spec/v2.0.0.html)

## developer_onboarding
### 1. developer_flow — official-vendor
- **Source:** Alibaba Cloud Documentation
- **Detail:** The documented WindVane flow separates native-app/container integration from miniapp development and publishing. The miniapp developer links a project to an Application Open Platform application before build/package/publish.
- **URL(s):** [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/getting-started/rapid-development-of-windvane-applets-or-h5-applications](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/getting-started/rapid-development-of-windvane-applets-or-h5-applications)

## discovery_catalog_metadata_categories_search
### 1. observed_capability — primary_official
- **Source:** Microsoft official Teams documentation
- **Detail:** Observed capability: Microsoft Teams Store supports both search and category browsing. Search matches developer-provided app name, publisher, short/long descriptions, keywords, and category names in the manifest or Partner Center. Recommended standard (separate): define a normalized listing schema for name, publisher, short/long descriptions, keywords, and controlled categories, and make the match fields inspectable.
- **URL(s):** [https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/deploy-and-publish/appsource/publish](https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/deploy-and-publish/appsource/publish)

## discovery_catalog_search
### 1. observed_capability — primary_official
- **Source:** WeChat official developer docs
- **Detail:** Observed capability: Weixin's official search-optimization guide says crawler-discoverable URLs are an important source for finding pages, search-result URLs must open directly independent of context, and page titles plus thumbnail images affect understanding and exposure conversion. Recommended standard (separate): require a canonical deep link, explicit title/thumbnail metadata, and a crawlable landing page for every listed mini-app.
- **URL(s):** [https://developers.weixin.qq.com/miniprogram/en/dev/framework/search/seo.html](https://developers.weixin.qq.com/miniprogram/en/dev/framework/search/seo.html)

## discovery_monetization_security_localization
### 1. observed_capability — primary_official_press
- **Source:** Grab official press release
- **Detail:** Observed capability: Grab's Partner Apps launch makes third-party services available inside Grab; Grab states that partners can use its app interface/infrastructure plus payments and security infrastructure, while contextual advertising can cross-sell and boost service discoverability. The announcement names initial availability in Singapore and Malaysia and plans for other markets. Recommended standard (separate): support contextual placement and regional rollout controls, while keeping placement/ads distinct from organic ranking.
- **URL(s):** [https://www.grab.com/sg/press/others/grab-launches-third-party-partner-apps-within-grab-app-offering-more-everyday-services-for-everyday-needs/](https://www.grab.com/sg/press/others/grab-launches-third-party-partner-apps-within-grab-app-offering-more-everyday-services-for-everyday-needs/)

## discovery_trust_review
### 1. observed_capability — primary_official
- **Source:** LINE official developer documentation
- **Detail:** Observed capability: LINE separates unverified and verified MINI Apps; verification adds a verified badge, and LINE search is available only for verified MINI Apps. The submission flow separately controls when search is enabled after approval. Recommended standard (separate): make verification state a visible trust signal and gate public search/discovery on approval status.
- **URL(s):** [https://developers.line.biz/en/docs/line-mini-app/discover/introduction/](https://developers.line.biz/en/docs/line-mini-app/discover/introduction/)

## ecosystem_integration_support_operations
### 1. observed_capability — primary_official
- **Source:** Gojek official GoSend API FAQ
- **Detail:** Observed capability: Gojek's public GoSend integration documentation describes an API call that searches for available drivers, webhooks that push real-time order-status updates to a partner endpoint, and a claims process with a seven-day limit and agreement-based support. This is verified partner-service integration evidence, not evidence of a public mini-app store. Recommended standard (separate): reuse webhook status, operational support, and claims-SLA patterns for embedded services.
- **URL(s):** [https://www.gojek.com/en-id/gosend/api/faq](https://www.gojek.com/en-id/gosend/api/faq)

## enterprise_internal_permissions_validation_lifecycle
### 1. observed_capability — primary_official
- **Source:** Microsoft official Teams documentation
- **Detail:** Observed capability: Teams Developer Portal exposes device, team, chat/meeting, and user permissions; ownership roles include Administrator and Operative; and publishers can publish to their organization or to the Teams Store. Its validation tool checks Microsoft review test cases and requires errors to be resolved before publishing. Recommended standard (separate): support org-private distribution, least-privilege permission review, role-based administration, and deterministic pre-submission validation.
- **URL(s):** [https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/build-and-test/manage-your-apps-in-developer-portal](https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/build-and-test/manage-your-apps-in-developer-portal)

## event_contract
### 1. sourced_finding — primary-standard
- **Source:** CloudEvents Specification v1.0.2
- **Detail:** CloudEvents is a vendor-neutral event format intended to make event data interoperable across services and platforms. Events carry required context attributes plus event data; the specification requires JSON support and defines producers, consumers, and intermediaries. It is suitable for release, review, security, incident, and deprecation events.
- **URL(s):** [https://raw.githubusercontent.com/cloudevents/spec/v1.0.2/cloudevents/spec.md](https://raw.githubusercontent.com/cloudevents/spec/v1.0.2/cloudevents/spec.md)

## gaps
### 1. gap — gap
- **Source:** Research gap register
- **Detail:** [GAP] Generic standards do not define the WindVane/uni-app package manifest, exact JSAPI permission taxonomy, or host SDK compatibility matrix. These must be obtained from the POC vendor documentation and validated experimentally before implementation-specific conformance tests are frozen.
- **URL(s):** [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp)

### 2. gap — gap
- **Source:** Research gap register
- **Detail:** [GAP] Publisher KYC, payout, tax, content-policy, privacy, data-residency, and retention obligations are jurisdiction- and business-model-specific; this architecture intentionally does not invent a legal control set or thresholds. Legal/compliance owners must bind those fields and review cadences to the target market.
- **URL(s):** [https://csrc.nist.gov/pubs/sp/800/218/final](https://csrc.nist.gov/pubs/sp/800/218/final)

### 3. gap — gap
- **Source:** Research gap register
- **Detail:** [GAP] Exact rollout percentages, SLO thresholds, package-size ceilings, vulnerability severity cutoffs, and rollback timing need calibration from pilot telemetry and threat modeling. The proposal deliberately specifies control points and evidence, not unsupported numeric targets.
- **URL(s):** [https://sre.google/sre-book/monitoring-distributed-systems/](https://sre.google/sre-book/monitoring-distributed-systems/), [https://argo-rollouts.readthedocs.io/en/stable/features/canary/](https://argo-rollouts.readthedocs.io/en/stable/features/canary/)

## incident_response
### 1. sourced_finding — engineering-reference
- **Source:** Google SRE, Monitoring Distributed Systems and Managing Incidents
- **Detail:** Google SRE guidance emphasizes low-noise monitoring, simple human-page rules, and distinguishing symptoms from causes. Its incident-management guidance uses separated Incident Command, Operations, Communication, and Planning roles, a recognized command post, a live incident state document, and explicit handoffs; its postmortem guidance expects documented impact, causes, mitigation, and follow-up actions for significant incidents.
- **URL(s):** [https://sre.google/sre-book/managing-incidents/](https://sre.google/sre-book/managing-incidents/)

### 2. sourced_finding — primary-government
- **Source:** NIST SP 800-61 Rev. 3
- **Detail:** NIST SP 800-61 Rev. 3 recommends incorporating incident-response considerations throughout cybersecurity risk management to prepare for incidents, reduce their number and impact, and improve detection, response, and recovery efficiency. It supersedes Rev. 2 and is the current NIST incident-response publication used here.
- **URL(s):** [https://csrc.nist.gov/pubs/sp/800/61/r3/final](https://csrc.nist.gov/pubs/sp/800/61/r3/final)

### 3. design_proposal — proposal
- **Source:** Reference architecture synthesis
- **Detail:** [PROPOSAL] Incident controls: freeze promotions; mark the digest quarantined; stop new installs; revoke or constrain publisher/CI trust; serve the last-known-good digest; invalidate unsafe cache entries; preserve signed artifacts, logs, review evidence, and telemetry; notify publisher/operators/users according to severity; then run a blameless postmortem with corrective actions. Use named Incident Commander, Operations, Communications, and Planning roles, with a tested kill switch and rollback drill.
- **URL(s):** [https://sre.google/sre-book/managing-incidents/](https://sre.google/sre-book/managing-incidents/), [https://csrc.nist.gov/pubs/sp/800/61/r3/final](https://csrc.nist.gov/pubs/sp/800/61/r3/final), [https://theupdateframework.github.io/specification/latest/](https://theupdateframework.github.io/specification/latest/)

## lifecycle
### 1. lifecycle_control — official-vendor
- **Source:** Alibaba Cloud Documentation
- **Detail:** The Application Open Platform is described as a developer platform for full-lifecycle management, including developer registration, miniapp review, and distribution. This supports making onboarding, review, and distribution first-class store capabilities.
- **URL(s):** [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp)

## lifecycle_review_localization
### 1. observed_capability — primary_official
- **Source:** LINE official developer documentation
- **Detail:** Observed capability: LINE creates Developing, Review, and Published internal channels for each MINI App; review copies the developing settings, and publishing copies approved settings to the live channel. Changes to privacy-policy URL, localization/multilingual support, and published endpoint require re-review for verified apps. Recommended standard (separate): use isolated dev/review/prod channels and classify high-risk metadata, localization, and endpoint changes as re-review triggers.
- **URL(s):** [https://developers.line.biz/en/docs/line-mini-app/discover/console-guide/](https://developers.line.biz/en/docs/line-mini-app/discover/console-guide/)

## monetization_entitlements_lifecycle
### 1. observed_capability — primary_official
- **Source:** Microsoft official Teams documentation
- **Detail:** Observed capability: Teams SaaS offers can use the SaaS Fulfillment API to manage subscription-plan lifecycle and the usageRights Graph API to check whether a licensed user may access the app; Microsoft-facilitated offers are transactable through Partner Center and undergo automated validation plus preview before publication. Recommended standard (separate): model subscription entitlement, user authorization, transaction ownership, and pre-live validation as explicit lifecycle stages.
- **URL(s):** [https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/deploy-and-publish/appsource/prepare/include-saas-offer](https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/deploy-and-publish/appsource/prepare/include-saas-offer)

## monetization_review_localization
### 1. observed_capability — primary_official
- **Source:** LINE official developer documentation
- **Detail:** Observed capability: LINE MINI Apps support payments through an integrated payment system, while ad monetization uses LY Ads Network Display Ads (Web) for services in Japan and requires LY Corporation's ad/site review; only reviewed and approved ads are displayed. Recommended standard (separate): separate transaction capability from advertising, require ad-network review, and scope monetization by market.
- **URL(s):** [https://developers.line.biz/en/docs/line-mini-app/service/line-mini-app-ads/](https://developers.line.biz/en/docs/line-mini-app/service/line-mini-app-ads/)

## observability
### 1. sourced_finding — primary-project
- **Source:** OpenTelemetry, Observability primer
- **Detail:** OpenTelemetry describes observability as instrumentation that emits traces, metrics, and logs, and distinguishes SLIs from SLOs. Distributed traces connect a request across services through spans and attributes. The store and host SDK should carry app identity, version/digest, release cohort, and host/runtime version as non-PII correlation attributes.
- **URL(s):** [https://opentelemetry.io/docs/concepts/observability-primer/](https://opentelemetry.io/docs/concepts/observability-primer/)

### 2. design_proposal — proposal
- **Source:** Reference architecture synthesis
- **Detail:** [PROPOSAL] Instrument store and host with OpenTelemetry traces, metrics, and logs. Required dimensions: app_id, package_digest, release_id, cohort, region, host_sdk, device/platform class, and policy decision; exclude user identifiers by default and apply retention/redaction rules. Track install/download success, verification failures, launch success, startup latency, crash/error rate, bridge-call failures, permission denials, API latency, catalog freshness, and rollback rate as SLIs; define SLOs before enabling automated promotion.
- **URL(s):** [https://opentelemetry.io/docs/concepts/observability-primer/](https://opentelemetry.io/docs/concepts/observability-primer/), [https://sre.google/sre-book/monitoring-distributed-systems/](https://sre.google/sre-book/monitoring-distributed-systems/)

## package-model/limits/on-demand-loading
### 1. on_demand_subpackage_runtime — official-platform-documentation
- **Source:** Weixin public documentation — Subcontract loading
- **Detail:** Weixin packages can be split into a main package and one or more subpackages at build time. The main package starts first; when a user enters a subpackage page, the client downloads that subpackage and displays it after download. The current documented limits are 30 MB for all subpackages together (20 MB for service-provider-developed mini programs) and 2 MB for a single main or subpackage. This is an official platform rule and a direct pattern for on-demand package delivery.
- **URL(s):** [https://developers.weixin.qq.com/miniprogram/en/dev/framework/subpackages.html](https://developers.weixin.qq.com/miniprogram/en/dev/framework/subpackages.html)

## package_signing
### 1. package_code_integrity — proposal
- **Source:** OWASP ASVS v4.0.3 V10.3.2
- **Detail:** ASVS V10.3.2 requires integrity protections such as code signing or subresource integrity and says applications must not load or execute code from untrusted sources. Proposal: reject unsigned or tampered mini-app packages, pin hashes for static assets, require SRI for hosted resources, and prohibit arbitrary remote JavaScript/modules at runtime.
- **URL(s):** [https://raw.githubusercontent.com/OWASP/ASVS/v4.0.3/4.0/en/0x18-V10-Malicious.md](https://raw.githubusercontent.com/OWASP/ASVS/v4.0.3/4.0/en/0x18-V10-Malicious.md)

### 2. release_integrity_verification — proposal
- **Source:** NIST SP 800-218 SSDF PS.2.1
- **Detail:** SSDF PS.2.1 says producers should make release-integrity verification information available, including cryptographic hashes and code signing, and periodically review certificate renewal, rotation, revocation, and protection. Proposal: require a publisher-signed manifest containing package digest and version; verify it before install/update; maintain a trusted-key registry with rotation and emergency revocation; and show verification status to operators.
- **URL(s):** [https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-218.pdf](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-218.pdf)

## performance
### 1. cache_tradeoff — official-vendor
- **Source:** Alibaba Cloud Documentation
- **Detail:** Alibaba documents ZCache as a preload/cache layer for H5 and mini programs and warns that preloading increases package size, recommending selective use and measurement by cache hit rate. Store quality gates should include package size, cache behavior, and performance metrics.
- **URL(s):** [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/basic-concepts](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/basic-concepts)

## permissions
### 1. authorization — official-vendor
- **Source:** Alibaba Cloud Documentation
- **Detail:** Alibaba's IDE documentation exposes a Need Auth From App setting: invoking client capabilities can trigger an authorization prompt. Store metadata and review should therefore declare native capabilities and authorization behavior.
- **URL(s):** [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/installing-the-plug-in](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/installing-the-plug-in)

## permissions/authorization/apis
### 1. user_permission_authorization — official-platform-documentation
- **Source:** Weixin public documentation — wx.authorize
- **Detail:** wx.authorize requests a named scope and immediately prompts the user to authorize a function or data access; if the user already agreed, it returns success without another popup. The official API requires the scope parameter and points developers to the platform scope list. This is platform behavior for user-mediated permissions, not a proposal for host-side capability policy.
- **URL(s):** [https://developers.weixin.qq.com/miniprogram/en/dev/api/open-api/authorize/wx.authorize.html](https://developers.weixin.qq.com/miniprogram/en/dev/api/open-api/authorize/wx.authorize.html)

## permissions_lifecycle
### 1. observed_capability — primary_official
- **Source:** WeChat official developer docs
- **Detail:** Observed capability: Weixin's privacy workflow is version-aware: after a privacy configuration update, newly added interfaces/components require users to resynchronize consent; removing a mini-program from the recent list clears the synchronization state; undeclared privacy APIs can be disabled by the platform. Recommended standard (separate): version permission changes must trigger migration/re-consent rules and an auditable deprecation path.
- **URL(s):** [https://developers.weixin.qq.com/miniprogram/en/dev/framework/user-privacy/PrivacyAuthorize.html](https://developers.weixin.qq.com/miniprogram/en/dev/framework/user-privacy/PrivacyAuthorize.html)

## permissions_privacy_ux
### 1. observed_capability — primary_official
- **Source:** LINE official developer documentation
- **Detail:** Observed capability: LINE's channel-consent simplification grants only the openid scope up front; profile and message scopes require an in-context verification screen listing the additional permissions when they are needed. Recommended standard (separate): minimize first-run consent, disclose incremental scopes at point of use, and record the consent decision per capability.
- **URL(s):** [https://developers.line.biz/en/docs/line-mini-app/develop/channel-consent-simplification/](https://developers.line.biz/en/docs/line-mini-app/develop/channel-consent-simplification/)

## permissions_review_trust
### 1. observed_capability — primary_official
- **Source:** Kakao Developers official documentation
- **Detail:** Observed capability: Kakao treats permissions as app qualifications for protected API features, consent items, response data, and configuration; developers apply through a Request additional features review/grant system and must follow feature-specific guidelines, approval criteria, and operating policies. Permission failures return a documented denial error. Recommended standard (separate): use capability-based scopes with policy review and clear denial states rather than one broad app permission.
- **URL(s):** [https://developers.kakao.com/docs/en/getting-started/permission](https://developers.kakao.com/docs/en/getting-started/permission)

## permissions_testing_lifecycle_privacy
### 1. observed_capability — primary_official
- **Source:** Kakao Developers official documentation
- **Detail:** Observed capability: Kakao Login documents required, optional, and consent-during-use levels; restricted consent levels require permission, and test-app permissions let teams develop changes before submitting a review request. The same documentation requires services to handle user-information deletion/account deletion according to privacy obligations. Recommended standard (separate): provide test/staging channels, consent-level controls, and deletion handling before production approval.
- **URL(s):** [https://developers.kakao.com/docs/en/kakaologin/utilize](https://developers.kakao.com/docs/en/kakaologin/utilize)

## platform_scope
### 1. platform_scope — official-vendor
- **Source:** Alibaba Cloud Documentation
- **Detail:** Alibaba SuperApp Business Application Platform defines a superapp as a platform/ecosystem for miniapps and provides a full-stack model with miniapp containers, IDE plugins, and an Application Open Platform.
- **URL(s):** [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp)

## privacy_governance
### 1. privacy_by_design_default — proposal
- **Source:** EDPB Guidelines 4/2019 on Article 25
- **Detail:** The EDPB guidance explains that data protection by design/default requires appropriate measures and safeguards, only necessary data by default, documented effectiveness, regular review, and risk-sensitive design. Proposal: make privacy review a store approval gate with a data inventory, purpose and retention map, default-off telemetry/collection, access limits, DPIA escalation for high-risk apps, and evidence that controls remain effective after updates.
- **URL(s):** [https://www.edpb.europa.eu/sites/default/files/files/file1/edpb_guidelines_201904_dataprotection_by_design_and_by_default_v2.0_en.pdf](https://www.edpb.europa.eu/sites/default/files/files/file1/edpb_guidelines_201904_dataprotection_by_design_and_by_default_v2.0_en.pdf)

## privacy_permissions
### 1. privacy_permission_minimization — proposal
- **Source:** OWASP MASVS-PRIVACY-1
- **Detail:** MASVS-PRIVACY-1 says an app should minimize access to sensitive data/resources, request only what it absolutely needs with informed consent, and prevent third-party SDKs from collecting before consent. Proposal: require every mini-app manifest to enumerate data and capability scopes, purpose, and third-party SDKs; default-deny unlisted scopes and block review when collection is unexplained or begins before consent.
- **URL(s):** [https://mas.owasp.org/MASVS/controls/MASVS-PRIVACY-1/](https://mas.owasp.org/MASVS/controls/MASVS-PRIVACY-1/)

## publisher_identity_signing
### 1. sourced_finding — primary-project
- **Source:** Sigstore documentation
- **Detail:** Sigstore supports identity-based artifact signing with ephemeral keys, an OIDC-bound certificate from Fulcio, and an append-only transparency log in Rekor. Verification checks the artifact signature, expected signer identity, certificate trust, and proof of log inclusion; Cosign can sign OCI artifacts and attestations.
- **URL(s):** [https://docs.sigstore.dev/](https://docs.sigstore.dev/)

## publisher_onboarding
### 1. sourced_finding — primary-vendor
- **Source:** GitHub Actions, OpenID Connect
- **Detail:** GitHub documents exchanging workflow identity for short-lived cloud access tokens instead of storing long-lived cloud credentials as repository secrets. Tokens are unique to a workflow job, carry claims such as repository/workflow/environment, and are validated against trust conditions. This supports auditable publisher CI identity without copying registry credentials into builds.
- **URL(s):** [https://docs.github.com/en/actions/concepts/security/openid-connect](https://docs.github.com/en/actions/concepts/security/openid-connect)

### 2. design_proposal — proposal
- **Source:** Reference architecture synthesis
- **Detail:** [PROPOSAL] Publisher onboarding should be a state machine: invited -> identity_verified -> organization_verified -> namespace_reserved -> CI_trust_bound -> sandbox_enabled -> eligible_for_submission -> active. Require verified email/domain or enterprise SSO, MFA, terms/privacy acceptance, abuse contact, legal/payout data where applicable, least-privilege publisher roles, namespace ownership, and a CI trust condition bound to repository/workflow/environment. For enterprise, add maker-checker separation and periodic access recertification.
- **URL(s):** [https://docs.github.com/en/actions/concepts/security/openid-connect](https://docs.github.com/en/actions/concepts/security/openid-connect), [https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/manage-environments](https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/manage-environments), [https://docs.sigstore.dev/](https://docs.sigstore.dev/)

## registration/removal/version-retention
### 1. app_registration_and_removal — official-vendor-practice
- **Source:** Alibaba Cloud SuperApp Documentation — Create a miniapp
- **Detail:** The Application Open Platform console guide says only admins can create or delete miniapps. Creation starts from the Miniapp List; the detail view exposes a Versions tab. Deletion is permitted only for a miniapp with no added versions. This is Alibaba Cloud console practice, not a universal normative rule; a store standard may use it as evidence for admin-gated registration and a version-retention removal gate.
- **URL(s):** [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/user-guide/application-development-1](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/user-guide/application-development-1)

## registration/roles/submission/review
### 1. workspace_registration_roles_and_approval — official-vendor-practice
- **Source:** Alipay International Mini Program Development Platform — Workflow procedures
- **Detail:** Alipay's workflow requires a developer account before development, then uses workspace roles such as Workspace Admin, Workspace Developer, and Workspace Reviewer. The guide describes app-level access/upload keys and signing/uploading a mini-program to the workspace portal; publishing and removal are approval items where a Mini Program Admin initiates and a Workspace Admin approves or rejects. These are Alipay platform governance practices, not universal requirements.
- **URL(s):** [https://miniprogram.alipay.com/docs-besuperapp/miniprogram_besuperapp/platform/workflow-procedures](https://miniprogram.alipay.com/docs-besuperapp/miniprogram_besuperapp/platform/workflow-procedures)

## registration/runtime/permissions/endpoint
### 1. liff_registration_endpoint_and_scopes — official-platform-documentation
- **Source:** LINE Developers — Adding a LIFF app to your channel
- **Detail:** A LIFF app is added to a LINE Login channel in the LINE Developers Console and can run in LINE or an external browser. The official registration guide allows up to 30 LIFF apps per channel, requires an HTTPS endpoint URL with no URL fragment, creates a LIFF ID and LIFF URL, and offers Compact/Tall/Full view sizes. Registration scopes include openid, email, profile, and chat_message.write, each tied to specific SDK capabilities and the permission-consent screen; the console also supports editing or deleting a LIFF app. These are LINE-specific platform rules.
- **URL(s):** [https://developers.line.biz/en/docs/liff/registering-liff-apps/](https://developers.line.biz/en/docs/liff/registering-liff-apps/)

## registry_catalog
### 1. sourced_finding — primary-standard
- **Source:** Open Container Initiative, Distribution Specification
- **Detail:** OCI Distribution defines a content-type-agnostic API. It models repositories as collections of manifests, blobs, and tags; blobs are addressed by digest; tags are human-readable pointers; subjects/referrers associate related manifests. Pull, push, content discovery, and content management are distinct API categories, with pull required for conformance and the other categories recommended.
- **URL(s):** [https://raw.githubusercontent.com/opencontainers/distribution-spec/main/spec.md](https://raw.githubusercontent.com/opencontainers/distribution-spec/main/spec.md)

### 2. design_proposal — proposal
- **Source:** Reference architecture synthesis
- **Detail:** [PROPOSAL] Store each release as an immutable content-addressed artifact: app_id, semver, package_digest, size, media_type, entrypoint, publisher_id, capabilities, data declarations, supported locales/platforms, min_host_version, max_host_version or tested range, required_host_apis, SBOM/provenance/signature references, release notes, and policy decision. Use tags only as mutable aliases; pin downloads and rollout targets to digests. Attach signatures, SLSA provenance, SBOM, review results, and security advisories as OCI referrers or equivalent signed metadata.
- **URL(s):** [https://raw.githubusercontent.com/opencontainers/distribution-spec/main/spec.md](https://raw.githubusercontent.com/opencontainers/distribution-spec/main/spec.md), [https://raw.githubusercontent.com/opencontainers/image-spec/main/manifest.md](https://raw.githubusercontent.com/opencontainers/image-spec/main/manifest.md), [https://slsa.dev/spec/v1.2/build-requirements](https://slsa.dev/spec/v1.2/build-requirements), [https://docs.sigstore.dev/cosign/signing/other_types/](https://docs.sigstore.dev/cosign/signing/other_types/)

## release
### 1. progressive_release — official-vendor
- **Source:** Alibaba Cloud Documentation
- **Detail:** The documented release path requires a gray-scale/gradual rollout to a targeted user group for validation, followed by an official release to all users. This is a concrete pattern for staged rollout and rollback gates in a miniapp store.
- **URL(s):** [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/getting-started/rapid-development-of-windvane-applets-or-h5-applications](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/getting-started/rapid-development-of-windvane-applets-or-h5-applications)

## release_governance
### 1. sourced_finding — primary-vendor
- **Source:** GitHub Actions, managing environments for deployment
- **Detail:** A job referencing an environment must satisfy its protection rules before it runs or accesses environment secrets. GitHub supports required reviewers, wait timers, deployment branch/tag restrictions, optional prevention of self-review, and deployment status objects/webhooks. These are reusable approval-gate patterns, not a requirement to use GitHub.
- **URL(s):** [https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/manage-environments](https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/manage-environments)

## release_lifecycle
### 1. sourced_finding — primary-vendor
- **Source:** Alibaba Cloud, Develop and publish a WindVane MiniApp using the VS Code extension
- **Detail:** The documented sequence is project scaffolding, link to the open platform, preview/debug, build and submit for review, gradual gray-scale rollout to a targeted group, official release, and post-release scan-to-preview verification. The guide also states a miniapp requires a host native app with the container integrated.
- **URL(s):** [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/getting-started/rapid-development-of-windvane-applets-or-h5-applications](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/getting-started/rapid-development-of-windvane-applets-or-h5-applications)

### 2. design_proposal — proposal
- **Source:** Reference architecture synthesis
- **Detail:** [PROPOSAL] Release states should be draft -> submitted -> automated_checks_passed -> human_review -> approved -> internal -> preview -> canary -> staged -> active, with rejected, quarantined, paused, aborted, withdrawn, and deprecated terminal/side states. Each transition requires an authenticated actor or policy, timestamp, evidence references, and an append-only audit event. Promote by cohort/region/host version; keep the last-known-good digest available and make abort/disable/rollback idempotent.
- **URL(s):** [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/getting-started/rapid-development-of-windvane-applets-or-h5-applications](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/getting-started/rapid-development-of-windvane-applets-or-h5-applications), [https://argo-rollouts.readthedocs.io/en/stable/features/canary/](https://argo-rollouts.readthedocs.io/en/stable/features/canary/), [https://support.google.com/googleplay/android-developer/answer/6346149?hl=en-IN](https://support.google.com/googleplay/android-developer/answer/6346149?hl=en-IN)

## review
### 1. release_gate — official-vendor
- **Source:** Alibaba Cloud Documentation
- **Detail:** After preview/debug, a built package is submitted to the Open Platform for review and publishing. The store pipeline should model submission and review as a gate before distribution.
- **URL(s):** [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/getting-started/rapid-development-of-windvane-applets-or-h5-applications](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/getting-started/rapid-development-of-windvane-applets-or-h5-applications)

## runtime
### 1. runtime_model — official-vendor
- **Source:** Alibaba Cloud Documentation
- **Detail:** Miniapps run inside a host superapp/container; WindVane uses a WebView container and JavaScript APIs to mediate web-to-native interaction and hardware capabilities. A store standard therefore needs explicit host/runtime compatibility metadata.
- **URL(s):** [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp)

## runtime/package/permissions/apis
### 1. runtime_package_and_api_model — official-vendor-practice
- **Source:** Alibaba Cloud SuperApp Documentation — Mini Program Overview
- **Detail:** Alibaba describes mini programs as JavaScript applications running in a mobile mini-program container, with access to network status, data caching, sensors, and other device capabilities. WindVane mini programs are web apps that integrate windvane.js and run in the WindVane container; WindVane JSAPIs expose system permissions/device features and support custom JSAPI extensions. The comparison identifies the default WindVane renderer as WebView and WindVane package size as KB~MB. These are vendor-defined runtime and package characteristics, not a cross-platform standard.
- **URL(s):** [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/latest](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/latest)

## runtime/package/review
### 1. native_vs_html5_package_model — official-vendor-practice
- **Source:** Alipay International Mini Program Development Platform — Mini Program types
- **Detail:** Alipay documents two mini-program types: DSL (the default native mini program) and HTML5. For HTML5 mini programs, publication steps are simpler because the package-build process is not required; the publishing application still requires mini-program information review, and the guide says workspace-admin approval does not support the mini-program quality-review step for HTML5. This is a vendor-specific runtime/review distinction, not a general mini-app standard.
- **URL(s):** [https://miniprogram.alipay.com/docs-besuperapp/miniprogram_besuperapp/platform/miniprogramtype](https://miniprogram.alipay.com/docs-besuperapp/miniprogram_besuperapp/platform/miniprogramtype)

## runtime_compatibility
### 1. sourced_finding — primary-standard
- **Source:** Open Container Initiative, Image Index Specification
- **Detail:** An OCI image index points to platform-specific manifests. Each entry may describe minimum runtime requirements such as CPU architecture, operating system, OS version, and features; consumers should be prepared to process indexes and use the first matching entry. This is a useful precedent for explicit host/runtime compatibility metadata.
- **URL(s):** [https://raw.githubusercontent.com/opencontainers/image-spec/main/image-index.md](https://raw.githubusercontent.com/opencontainers/image-spec/main/image-index.md)

### 2. sourced_finding — primary-standard
- **Source:** Semantic Versioning 2.0.0
- **Detail:** SemVer assigns MAJOR to incompatible API changes, MINOR to backward-compatible functionality, and PATCH to backward-compatible fixes. Once a version is released, its contents must not be modified; modifications are released as a new version. This provides a clear immutability rule for miniapp package versions and host API compatibility.
- **URL(s):** [https://semver.org/spec/v2.0.0.html](https://semver.org/spec/v2.0.0.html)

### 3. design_proposal — proposal
- **Source:** Reference architecture synthesis
- **Detail:** [PROPOSAL] The host compatibility resolver must reject before download when platform, host SDK, bridge/API range, required capability, locale, package format, or policy version is incompatible. Use SemVer for host/API contracts, explicit min_host_version and required_host_apis, and a capability declaration that is intersected with host policy. Cache and execute only a verified digest; on update failure retain the previous verified version and expose a safe fallback/offline state.
- **URL(s):** [https://semver.org/spec/v2.0.0.html](https://semver.org/spec/v2.0.0.html), [https://raw.githubusercontent.com/opencontainers/image-spec/main/image-index.md](https://raw.githubusercontent.com/opencontainers/image-spec/main/image-index.md), [https://mas.owasp.org/MASVS/](https://mas.owasp.org/MASVS/)

## runtime_permissions_localization_analytics_lifecycle_monetization
### 1. observed_capability — primary_official_repo
- **Source:** Grab official GitHub repository documentation
- **Detail:** Observed capability: Grab's official SuperApp SDK README defines MiniApps running in the Grab SuperApp WebView with type-safe modular APIs, standardized responses, streaming, and fallbacks. Modules include container lifecycle/analytics/connection verification, device locale, GrabID OAuth2/OIDC identity, location, checkout, and explicit permission scopes. Recommended standard (separate): require a documented host bridge with scoped permissions, locale access, payment boundary, lifecycle callbacks, and graceful fallback behavior.
- **URL(s):** [https://raw.githubusercontent.com/grab/superapp-sdk/master/README.md](https://raw.githubusercontent.com/grab/superapp-sdk/master/README.md)

## runtime_sandbox
### 1. webview_bridge_isolation — proposal
- **Source:** OWASP MASVS-PLATFORM-2
- **Detail:** MASVS-PLATFORM-2 says WebViews must be configured to prevent sensitive-data leakage and exposure of sensitive native functionality through JavaScript bridges and untrusted content. Proposal: restrict navigation to declared origins, disable unnecessary local-file access and debug interfaces, expose only allowlisted bridge methods, and run untrusted content without native capabilities.
- **URL(s):** [https://mas.owasp.org/MASVS/controls/MASVS-PLATFORM-2/](https://mas.owasp.org/MASVS/controls/MASVS-PLATFORM-2/)

### 2. miniapp_runtime_sandbox — proposal
- **Source:** OWASP ASVS v4.0.3 V1.14.5
- **Detail:** ASVS V1.14.5 calls for deployments to sandbox, containerize, and/or isolate at the network level to deter attacks against other applications, especially during sensitive or dangerous actions. Proposal: give each mini-app an isolated process/container or equivalent, deny lateral network access and host filesystem access by default, apply egress allowlists, and reset the runtime between tenants or sessions.
- **URL(s):** [https://raw.githubusercontent.com/OWASP/ASVS/v4.0.3/4.0/en/0x10-V1-Architecture.md](https://raw.githubusercontent.com/OWASP/ASVS/v4.0.3/4.0/en/0x10-V1-Architecture.md)

## search_ranking_trust_reviews
### 1. observed_capability — primary_official
- **Source:** Microsoft official Teams documentation
- **Detail:** Observed capability: Microsoft publicly discloses that Teams Store ranking uses relevance parameters; higher app quality/value, ratings, and genuine reviews improve ranking, while the editorial team uses ranking parameters for prominence and promo placement. Recommended standard (separate): publish ranking inputs at a high level, distinguish organic rank from editorial placement, and protect review authenticity.
- **URL(s):** [https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/deploy-and-publish/appsource/post-publish/teams-store-ranking-parameters](https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/deploy-and-publish/appsource/post-publish/teams-store-ranking-parameters)

## security_review
### 1. sourced_finding — primary-government
- **Source:** NIST SP 800-218, Secure Software Development Framework v1.1
- **Detail:** NIST recommends integrating a core set of secure software development practices into each SDLC to reduce released vulnerabilities, reduce exploitation impact, and address root causes. The framework also provides a common vocabulary for communicating security expectations to suppliers and consumers.
- **URL(s):** [https://csrc.nist.gov/pubs/sp/800/218/final](https://csrc.nist.gov/pubs/sp/800/218/final)

### 2. sourced_finding — primary-project
- **Source:** OWASP Mobile Application Security Verification Standard
- **Detail:** OWASP MASVS is an industry standard for mobile application security testing and architecture. Its control groups cover storage, cryptography, authentication/authorization, network communication, platform interaction, code quality, resilience against tampering, and privacy. A miniapp host bridge should map its risk controls to the applicable groups.
- **URL(s):** [https://mas.owasp.org/MASVS/](https://mas.owasp.org/MASVS/)

## software_supply_chain
### 1. sbom_provenance_retention — proposal
- **Source:** NIST SP 800-218 SSDF PS.3.2
- **Detail:** SSDF PS.3.2 says to collect, safeguard, maintain, and share provenance data for every release component, such as an SBOM; update it whenever components change; and protect its integrity. Proposal: require an SBOM and signed provenance per mini-app release, retain them with the immutable package record, expose them to enterprise operators, and trigger re-review on dependency changes.
- **URL(s):** [https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-218.pdf](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-218.pdf)

### 2. build_provenance_verification — proposal
- **Source:** SLSA v1.2 Build Track
- **Detail:** SLSA v1.2 requires producers to distribute provenance and describes provenance as verifiable information tying an artifact to where, when, and how it was produced; higher build levels add authenticity and isolation guarantees. Proposal: require a signed attestation bound to the package digest with source, builder, build parameters, and dependency data; verify it at ingestion, and target stronger SLSA levels for high-risk enterprise mini-apps.
- **URL(s):** [https://slsa.dev/spec/v1.2/build-requirements](https://slsa.dev/spec/v1.2/build-requirements)

## staged_release_rollback
### 1. sourced_finding — engineering-reference
- **Source:** Argo Rollouts progressive delivery documentation
- **Detail:** Argo Rollouts models canary delivery with weighted traffic steps and pauses, and can evaluate analysis before promotion. Its blue-green strategy separates preview and active services; pre-promotion analysis can block the switch, while failed post-promotion analysis aborts and returns traffic to the prior stable ReplicaSet. The controller explicitly supports automated promotion and rollback.
- **URL(s):** [https://argo-rollouts.readthedocs.io/en/stable/features/bluegreen/](https://argo-rollouts.readthedocs.io/en/stable/features/bluegreen/)

### 2. sourced_finding — primary-vendor
- **Source:** Google Play Console Help, staged roll-outs
- **Detail:** Google Play staged roll-outs expose an update to a percentage of users and allow the percentage to increase over time. Users are selected for each new release rollout; halting and resuming affects the same set of users. The guidance recommends monitoring crash reports and user feedback during a staged rollout. This is a mature store pattern, not a miniapp-specific standard.
- **URL(s):** [https://support.google.com/googleplay/android-developer/answer/6346149?hl=en-IN](https://support.google.com/googleplay/android-developer/answer/6346149?hl=en-IN)

## storage
### 1. sensitive_data_leakage_prevention — proposal
- **Source:** OWASP MASVS-STORAGE-2
- **Detail:** MASVS-STORAGE-2 says the app prevents leakage of sensitive data, including unintended exposure through APIs, backups, logs, and other public locations. Proposal: run package/runtime checks for secrets or personal data in logs, backups, public storage, clipboard, and screenshots; isolate mini-app storage from the host and enforce host-side redaction.
- **URL(s):** [https://mas.owasp.org/MASVS/controls/MASVS-STORAGE-2/](https://mas.owasp.org/MASVS/controls/MASVS-STORAGE-2/)

## store-apis/preview/preload/versioning
### 1. store_sdk_operations_and_cache_versions — official-vendor-practice
- **Source:** Alibaba Cloud SuperApp Documentation — Miniapp operations
- **Detail:** After a native app integrates the WindVane or uni-app miniapp container, the SDK provides list, search, open, preview, and preload operations; advanced requirements can use the SuperApp OpenAPI. Opening uses a miniapp ID, preview uses a miniapp ID plus publishId, and the SDK exposes cached and preloaded package version metadata and cache-removal operations. This is a concrete host-store API pattern, not evidence that every store must expose the same calls.
- **URL(s):** [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/small-program-operation-2](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/small-program-operation-2)

## submission/review/versioning/rollback
### 1. submission_review_and_version_lifecycle — official-platform-documentation
- **Source:** Weixin public documentation — Preparations Before Launching
- **Detail:** Weixin distinguishes preview from code upload: upload submits code for trial or audit and records a version number and project notes. The development version keeps the latest upload per person; only one code copy can be under audit; after audit it can go online or be resubmitted by covering the audit version. The online version is what users run and is overwritten/updated after publishing. Team development permissions include submitting for review, release, rollback, and suspending online service.
- **URL(s):** [https://developers.weixin.qq.com/miniprogram/en/dev/quickstart/basic/role.html](https://developers.weixin.qq.com/miniprogram/en/dev/quickstart/basic/role.html)

## supply_chain_runtime
### 1. third_party_component_encapsulation — proposal
- **Source:** OWASP ASVS v4.0.3 V14.2.6
- **Detail:** ASVS V14.2.6 says to reduce attack surface by sandboxing or encapsulating third-party libraries so only required behavior is exposed. Proposal: place SDKs and mini-app libraries behind capability adapters; prohibit direct access to host secrets, filesystem, network, and identity APIs; grant and test only the declared calls.
- **URL(s):** [https://raw.githubusercontent.com/OWASP/ASVS/v4.0.3/4.0/en/0x22-V14-Config.md](https://raw.githubusercontent.com/OWASP/ASVS/v4.0.3/4.0/en/0x22-V14-Config.md)

## trust_permissions_review_support
### 1. observed_capability — primary_official
- **Source:** WeChat official developer docs
- **Detail:** Observed capability: Weixin requires developers to declare processed user information; mandatory entries are displayed based on privacy-interface calls, the platform reviews the stated processing purpose, and developers provide contact details for access/copy/correction/deletion rights. Supplemental documents may be uploaded and are reviewed. Recommended standard (separate): make a permission manifest, purpose statement, user-rights contact, and review evidence mandatory catalog metadata.
- **URL(s):** [https://developers.weixin.qq.com/miniprogram/en/dev/framework/user-privacy/miniprogram-intro.html](https://developers.weixin.qq.com/miniprogram/en/dev/framework/user-privacy/miniprogram-intro.html)

## vendor_reference
### 1. sourced_finding — primary-vendor
- **Source:** Alibaba Cloud, SuperApp Business Application Platform miniapp overview
- **Detail:** The platform separates a native miniapp container, an Application Open Platform for developer registration/review/distribution, and an IDE plugin for creation, debugging, preview, packaging, and release. It describes the super app as a platform/ecosystem in which third-party developers create and publish miniapps.
- **URL(s):** [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp)

## versioning/deprecation/migration
### 1. sdk_deprecation_and_migration — official-platform-change-notice
- **Source:** LINE Developers — LIFF release notes
- **Detail:** LINE's official release notes state that LIFF v1 will be discontinued and recommend LIFF v2. The same primary source records liff.scanCode() as deprecated and recommends liff.scanCodeV2(). This is explicit vendor deprecation evidence: a store standard should require deprecation notices, replacement APIs, and migration timelines rather than assuming old mini-app APIs remain available.
- **URL(s):** [https://developers.line.biz/en/docs/liff/release-notes/](https://developers.line.biz/en/docs/liff/release-notes/)

## versioning/review/pilot/grayscale/full-release
### 1. version_review_pilot_and_rollout — official-vendor-practice
- **Source:** Alipay International Mini Program Development Platform — Release mini programs
- **Detail:** Alipay's release workflow starts after an IDE-uploaded version is created in the workspace. A version can be released to the current app or another configured app; the release request is submitted to the app for review, then approved versions proceed to pilot testing, followed by a grayscale release or full release. This is a directly documented staged-rollout pattern; the sequencing is Alipay practice rather than a universal rule.
- **URL(s):** [https://miniprogram.alipay.com/docs-besuperapp/miniprogram_besuperapp/platform/release](https://miniprogram.alipay.com/docs-besuperapp/miniprogram_besuperapp/platform/release)

## vulnerability_management
### 1. dependency_vulnerability_lifecycle — proposal
- **Source:** NIST SP 800-218 SSDF PW.4.4
- **Detail:** SSDF PW.4.4 calls for lifecycle verification of commercial, open-source, and other third-party components, automated detection of known vulnerabilities, end-of-life checks, and integrity confirmation through signatures or equivalent mechanisms. Proposal: continuously scan each mini-app SBOM and package, flag end-of-life dependencies, quarantine releases that cross enterprise severity policy, and require a documented exception or remediation plan.
- **URL(s):** [https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-218.pdf](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-218.pdf)

## vulnerability_response
### 1. vulnerability_disclosure_response — proposal
- **Source:** NIST SP 800-218 SSDF RV.1.3
- **Detail:** SSDF RV.1.3 calls for a vulnerability-disclosure policy, roles and processes, a product security incident response team, communication plans, playbooks, and exercises. Proposal: operate a store disclosure channel with publisher ownership, severity/SLA fields, PSIRT escalation, signed advisories, emergency takedown/quarantine, and an auditable drill record.
- **URL(s):** [https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-218.pdf](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-218.pdf)
