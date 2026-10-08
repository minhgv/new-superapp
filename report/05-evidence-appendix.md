# Evidence Appendix: Canonical Findings Index

This appendix catalogues all **416** verified findings accumulated across Iterations 1 through 31.
Every finding represents a source-grounded fact, standard specification, platform practice, or control proposal with verified URLs.

| # | ID / Topic | Category | Evidence Level | Title & Summary | Source URL |
|---|---|---|---|---|---|
| 1 | `vendor_reference` | vendor_reference | `primary-vendor` | **sourced_finding**<br>The platform separates a native miniapp container, an Application Open Platform for developer registration/review/distri... | [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp) |
| 2 | `release_lifecycle` | release_lifecycle | `primary-vendor` | **sourced_finding**<br>The documented sequence is project scaffolding, link to the open platform, preview/debug, build and submit for review, g... | [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/getting-started/rapid-development-of-windvane-applets-or-h5-applications](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/getting-started/rapid-development-of-windvane-applets-or-h5-applications) |
| 3 | `catalog_api` | catalog_api | `primary-vendor` | **sourced_finding**<br>The SDK documentation exposes list, search, open, preview, and preload operations. Listing is paginated with an anchor a... | [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/small-program-operation-2](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/small-program-operation-2) |
| 4 | `registry_catalog` | registry_catalog | `primary-standard` | **sourced_finding**<br>OCI Distribution defines a content-type-agnostic API. It models repositories as collections of manifests, blobs, and tag... | [https://raw.githubusercontent.com/opencontainers/distribution-spec/main/spec.md](https://raw.githubusercontent.com/opencontainers/distribution-spec/main/spec.md) |
| 5 | `artifact_integrity` | artifact_integrity | `primary-standard` | **sourced_finding**<br>The OCI image manifest is designed for content-addressable artifacts and uses descriptors for configuration and layers. ... | [https://raw.githubusercontent.com/opencontainers/image-spec/main/manifest.md](https://raw.githubusercontent.com/opencontainers/image-spec/main/manifest.md) |
| 6 | `runtime_compatibility` | runtime_compatibility | `primary-standard` | **sourced_finding**<br>An OCI image index points to platform-specific manifests. Each entry may describe minimum runtime requirements such as C... | [https://raw.githubusercontent.com/opencontainers/image-spec/main/image-index.md](https://raw.githubusercontent.com/opencontainers/image-spec/main/image-index.md) |
| 7 | `ci_cd_supply_chain` | ci_cd_supply_chain | `primary-standard` | **sourced_finding**<br>SLSA v1.2 describes provenance as verifiable information about where, when, and how an artifact was produced. Its build ... | [https://slsa.dev/spec/v1.2/build-requirements](https://slsa.dev/spec/v1.2/build-requirements) |
| 8 | `publisher_identity_signing` | publisher_identity_signing | `primary-project` | **sourced_finding**<br>Sigstore supports identity-based artifact signing with ephemeral keys, an OIDC-bound certificate from Fulcio, and an app... | [https://docs.sigstore.dev/](https://docs.sigstore.dev/) |
| 9 | `catalog_update_security` | catalog_update_security | `primary-standard` | **sourced_finding**<br>TUF secures software update systems with signed metadata and distinct root, targets, snapshot, and timestamp roles, supp... | [https://theupdateframework.github.io/specification/latest/](https://theupdateframework.github.io/specification/latest/) |
| 10 | `security_review` | security_review | `primary-government` | **sourced_finding**<br>NIST recommends integrating a core set of secure software development practices into each SDLC to reduce released vulner... | [https://csrc.nist.gov/pubs/sp/800/218/final](https://csrc.nist.gov/pubs/sp/800/218/final) |
| 11 | `security_review` | security_review | `primary-project` | **sourced_finding**<br>OWASP MASVS is an industry standard for mobile application security testing and architecture. Its control groups cover s... | [https://mas.owasp.org/MASVS/](https://mas.owasp.org/MASVS/) |
| 12 | `publisher_onboarding` | publisher_onboarding | `primary-vendor` | **sourced_finding**<br>GitHub documents exchanging workflow identity for short-lived cloud access tokens instead of storing long-lived cloud cr... | [https://docs.github.com/en/actions/concepts/security/openid-connect](https://docs.github.com/en/actions/concepts/security/openid-connect) |
| 13 | `release_governance` | release_governance | `primary-vendor` | **sourced_finding**<br>A job referencing an environment must satisfy its protection rules before it runs or accesses environment secrets. GitHu... | [https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/manage-environments](https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/manage-environments) |
| 14 | `staged_release_rollback` | staged_release_rollback | `engineering-reference` | **sourced_finding**<br>Argo Rollouts models canary delivery with weighted traffic steps and pauses, and can evaluate analysis before promotion.... | [https://argo-rollouts.readthedocs.io/en/stable/features/bluegreen/](https://argo-rollouts.readthedocs.io/en/stable/features/bluegreen/) |
| 15 | `staged_release_rollback` | staged_release_rollback | `primary-vendor` | **sourced_finding**<br>Google Play staged roll-outs expose an update to a percentage of users and allow the percentage to increase over time. U... | [https://support.google.com/googleplay/android-developer/answer/6346149?hl=en-IN](https://support.google.com/googleplay/android-developer/answer/6346149?hl=en-IN) |
| 16 | `api_contract` | api_contract | `primary-standard` | **sourced_finding**<br>OpenAPI defines a standard, programming-language-agnostic interface description for HTTP APIs, enabling humans and compu... | [https://spec.openapis.org/oas/v3.1.1.html](https://spec.openapis.org/oas/v3.1.1.html) |
| 17 | `event_contract` | event_contract | `primary-standard` | **sourced_finding**<br>CloudEvents is a vendor-neutral event format intended to make event data interoperable across services and platforms. Ev... | [https://raw.githubusercontent.com/cloudevents/spec/v1.0.2/cloudevents/spec.md](https://raw.githubusercontent.com/cloudevents/spec/v1.0.2/cloudevents/spec.md) |
| 18 | `observability` | observability | `primary-project` | **sourced_finding**<br>OpenTelemetry describes observability as instrumentation that emits traces, metrics, and logs, and distinguishes SLIs fr... | [https://opentelemetry.io/docs/concepts/observability-primer/](https://opentelemetry.io/docs/concepts/observability-primer/) |
| 19 | `incident_response` | incident_response | `engineering-reference` | **sourced_finding**<br>Google SRE guidance emphasizes low-noise monitoring, simple human-page rules, and distinguishing symptoms from causes. I... | [https://sre.google/sre-book/managing-incidents/](https://sre.google/sre-book/managing-incidents/) |
| 20 | `incident_response` | incident_response | `primary-government` | **sourced_finding**<br>NIST SP 800-61 Rev. 3 recommends incorporating incident-response considerations throughout cybersecurity risk management... | [https://csrc.nist.gov/pubs/sp/800/61/r3/final](https://csrc.nist.gov/pubs/sp/800/61/r3/final) |
| 21 | `runtime_compatibility` | runtime_compatibility | `primary-standard` | **sourced_finding**<br>SemVer assigns MAJOR to incompatible API changes, MINOR to backward-compatible functionality, and PATCH to backward-comp... | [https://semver.org/spec/v2.0.0.html](https://semver.org/spec/v2.0.0.html) |
| 22 | `deprecation` | deprecation | `primary-standard` | **sourced_finding**<br>RFC 9745 defines a Deprecation response header and deprecation link relation to signal that a resource will be or has be... | [https://www.rfc-editor.org/rfc/rfc9745.html](https://www.rfc-editor.org/rfc/rfc9745.html) |
| 23 | `architecture` | architecture | `proposal` | **design_proposal**<br>[PROPOSAL] Use two planes and explicit trust boundaries. Control plane: publisher identity/RBAC, submission API, policy ... | [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp) |
| 24 | `registry_catalog` | registry_catalog | `proposal` | **design_proposal**<br>[PROPOSAL] Store each release as an immutable content-addressed artifact: app_id, semver, package_digest, size, media_ty... | [https://raw.githubusercontent.com/opencontainers/distribution-spec/main/spec.md](https://raw.githubusercontent.com/opencontainers/distribution-spec/main/spec.md) |
| 25 | `publisher_onboarding` | publisher_onboarding | `proposal` | **design_proposal**<br>[PROPOSAL] Publisher onboarding should be a state machine: invited -> identity_verified -> organization_verified -> name... | [https://docs.github.com/en/actions/concepts/security/openid-connect](https://docs.github.com/en/actions/concepts/security/openid-connect) |
| 26 | `ci_cd_security_review` | ci_cd_security_review | `proposal` | **design_proposal**<br>[PROPOSAL] CI/CD acceptance gates should be deterministic and recorded per digest: manifest/schema validation; size/file... | [https://csrc.nist.gov/pubs/sp/800/218/final](https://csrc.nist.gov/pubs/sp/800/218/final) |
| 27 | `release_lifecycle` | release_lifecycle | `proposal` | **design_proposal**<br>[PROPOSAL] Release states should be draft -> submitted -> automated_checks_passed -> human_review -> approved -> interna... | [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/getting-started/rapid-development-of-windvane-applets-or-h5-applications](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/getting-started/rapid-development-of-windvane-applets-or-h5-applications) |
| 28 | `runtime_compatibility` | runtime_compatibility | `proposal` | **design_proposal**<br>[PROPOSAL] The host compatibility resolver must reject before download when platform, host SDK, bridge/API range, requir... | [https://semver.org/spec/v2.0.0.html](https://semver.org/spec/v2.0.0.html) |
| 29 | `api_event_contract` | api_event_contract | `proposal` | **design_proposal**<br>[PROPOSAL] Publish a versioned OpenAPI control-plane contract with cursor pagination and ETags for catalog reads; idempo... | [https://spec.openapis.org/oas/v3.1.1.html](https://spec.openapis.org/oas/v3.1.1.html) |
| 30 | `observability` | observability | `proposal` | **design_proposal**<br>[PROPOSAL] Instrument store and host with OpenTelemetry traces, metrics, and logs. Required dimensions: app_id, package_... | [https://opentelemetry.io/docs/concepts/observability-primer/](https://opentelemetry.io/docs/concepts/observability-primer/) |
| 31 | `incident_response` | incident_response | `proposal` | **design_proposal**<br>[PROPOSAL] Incident controls: freeze promotions; mark the digest quarantined; stop new installs; revoke or constrain pub... | [https://sre.google/sre-book/managing-incidents/](https://sre.google/sre-book/managing-incidents/) |
| 32 | `deprecation` | deprecation | `proposal` | **design_proposal**<br>[PROPOSAL] Deprecation should be explicit and staged: active -> deprecated (new installs discouraged, successor/migratio... | [https://www.rfc-editor.org/rfc/rfc9745.html](https://www.rfc-editor.org/rfc/rfc9745.html) |
| 33 | `control_matrix` | control_matrix | `proposal` | **control_matrix**<br>[PROPOSAL] MVP/P0 must-have: verified publisher account plus MFA and RBAC; immutable versioned package and digest; signe... | [https://docs.github.com/en/actions/concepts/security/openid-connect](https://docs.github.com/en/actions/concepts/security/openid-connect) |
| 34 | `control_matrix` | control_matrix | `proposal` | **control_matrix**<br>[PROPOSAL] Priority acceptance evidence: P0 release is publishable only when the package digest, publisher identity, hos... | [https://spec.openapis.org/oas/v3.1.1.html](https://spec.openapis.org/oas/v3.1.1.html) |
| 35 | `control_matrix` | control_matrix | `proposal` | **control_matrix**<br>[PROPOSAL] P2 scale controls after the MVP is stable: multi-region registry mirrors with signed metadata; offline/poor-n... | [https://theupdateframework.github.io/specification/latest/](https://theupdateframework.github.io/specification/latest/) |
| 36 | `gaps` | gaps | `gap` | **gap**<br>[GAP] Generic standards do not define the WindVane/uni-app package manifest, exact JSAPI permission taxonomy, or host SD... | [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp) |
| 37 | `gaps` | gaps | `gap` | **gap**<br>[GAP] Publisher KYC, payout, tax, content-policy, privacy, data-residency, and retention obligations are jurisdiction- a... | [https://csrc.nist.gov/pubs/sp/800/218/final](https://csrc.nist.gov/pubs/sp/800/218/final) |
| 38 | `gaps` | gaps | `gap` | **gap**<br>[GAP] Exact rollout percentages, SLO thresholds, package-size ceilings, vulnerability severity cutoffs, and rollback tim... | [https://sre.google/sre-book/monitoring-distributed-systems/](https://sre.google/sre-book/monitoring-distributed-systems/) |
| 39 | `discovery_catalog_search` | discovery_catalog_search | `primary_official` | **observed_capability**<br>Observed capability: Weixin's official search-optimization guide says crawler-discoverable URLs are an important source ... | [https://developers.weixin.qq.com/miniprogram/en/dev/framework/search/seo.html](https://developers.weixin.qq.com/miniprogram/en/dev/framework/search/seo.html) |
| 40 | `trust_permissions_review_support` | trust_permissions_review_support | `primary_official` | **observed_capability**<br>Observed capability: Weixin requires developers to declare processed user information; mandatory entries are displayed b... | [https://developers.weixin.qq.com/miniprogram/en/dev/framework/user-privacy/miniprogram-intro.html](https://developers.weixin.qq.com/miniprogram/en/dev/framework/user-privacy/miniprogram-intro.html) |
| 41 | `analytics_quality_lifecycle` | analytics_quality_lifecycle | `primary_official` | **observed_capability**<br>Observed capability: Weixin Experience Rating checks mini-program experience in real time, analyzes likely causes, locat... | [https://developers.weixin.qq.com/miniprogram/en/dev/framework/audits/audits.html](https://developers.weixin.qq.com/miniprogram/en/dev/framework/audits/audits.html) |
| 42 | `permissions_lifecycle` | permissions_lifecycle | `primary_official` | **observed_capability**<br>Observed capability: Weixin's privacy workflow is version-aware: after a privacy configuration update, newly added inter... | [https://developers.weixin.qq.com/miniprogram/en/dev/framework/user-privacy/PrivacyAuthorize.html](https://developers.weixin.qq.com/miniprogram/en/dev/framework/user-privacy/PrivacyAuthorize.html) |
| 43 | `catalog_categories_permissions_analytics_monetization_lifecycle` | catalog_categories_permissions_analytics_monetization_lifecycle | `primary_official` | **observed_capability**<br>Observed capability: Alipay+ Mini Program Platform describes a centralized Service Market where ISVs publish services fo... | [https://miniprogram.alipay.com/docs-alipayconnect/miniprogram_alipayconnect/platform/overview](https://miniprogram.alipay.com/docs-alipayconnect/miniprogram_alipayconnect/platform/overview) |
| 44 | `catalog_metadata_review_release_lifecycle` | catalog_metadata_review_release_lifecycle | `primary_official` | **observed_capability**<br>Observed capability: Alipay's quick-start lifecycle asks an admin to submit mini-program particulars including name, des... | [https://miniprogram.alipay.com/docs/miniprogram/mpdev/quick-start_overview](https://miniprogram.alipay.com/docs/miniprogram/mpdev/quick-start_overview) |
| 45 | `runtime_permissions_localization_analytics_lifecycle_monetization` | runtime_permissions_localization_analytics_lifecycle_monetization | `primary_official_repo` | **observed_capability**<br>Observed capability: Grab's official SuperApp SDK README defines MiniApps running in the Grab SuperApp WebView with type... | [https://raw.githubusercontent.com/grab/superapp-sdk/master/README.md](https://raw.githubusercontent.com/grab/superapp-sdk/master/README.md) |
| 46 | `discovery_monetization_security_localization` | discovery_monetization_security_localization | `primary_official_press` | **observed_capability**<br>Observed capability: Grab's Partner Apps launch makes third-party services available inside Grab; Grab states that partn... | [https://www.grab.com/sg/press/others/grab-launches-third-party-partner-apps-within-grab-app-offering-more-everyday-services-for-everyday-needs/](https://www.grab.com/sg/press/others/grab-launches-third-party-partner-apps-within-grab-app-offering-more-everyday-services-for-everyday-needs/) |
| 47 | `ecosystem_integration_support_operations` | ecosystem_integration_support_operations | `primary_official` | **observed_capability**<br>Observed capability: Gojek's public GoSend integration documentation describes an API call that searches for available d... | [https://www.gojek.com/en-id/gosend/api/faq](https://www.gojek.com/en-id/gosend/api/faq) |
| 48 | `catalog_search_ranking_reviews_analytics_lifecycle_gap` | catalog_search_ranking_reviews_analytics_lifecycle_gap | `primary_official_gap` | **evidence_gap**<br>Research gap: the fetched official GoSend API pages document partner API integration and operational support, but do not... | [https://www.gojek.com/en-id/gosend/api](https://www.gojek.com/en-id/gosend/api) |
| 49 | `permissions_review_trust` | permissions_review_trust | `primary_official` | **observed_capability**<br>Observed capability: Kakao treats permissions as app qualifications for protected API features, consent items, response ... | [https://developers.kakao.com/docs/en/getting-started/permission](https://developers.kakao.com/docs/en/getting-started/permission) |
| 50 | `permissions_testing_lifecycle_privacy` | permissions_testing_lifecycle_privacy | `primary_official` | **observed_capability**<br>Observed capability: Kakao Login documents required, optional, and consent-during-use levels; restricted consent levels ... | [https://developers.kakao.com/docs/en/kakaologin/utilize](https://developers.kakao.com/docs/en/kakaologin/utilize) |
| 51 | `analytics_operations` | analytics_operations | `primary_official` | **observed_capability**<br>Observed capability: Kakao's app-management Statistics page provides real-time API usage statistics with optional auto-r... | [https://developers.kakao.com/docs/en/getting-started/stat](https://developers.kakao.com/docs/en/getting-started/stat) |
| 52 | `catalog_search_ranking_reviews_monetization_gap` | catalog_search_ranking_reviews_monetization_gap | `primary_official_gap` | **evidence_gap**<br>Research gap: Kakao's public developer documentation verifies app/API management, permission review, consent controls, a... | [https://developers.kakao.com/docs/en](https://developers.kakao.com/docs/en) |
| 53 | `discovery_trust_review` | discovery_trust_review | `primary_official` | **observed_capability**<br>Observed capability: LINE separates unverified and verified MINI Apps; verification adds a verified badge, and LINE sear... | [https://developers.line.biz/en/docs/line-mini-app/discover/introduction/](https://developers.line.biz/en/docs/line-mini-app/discover/introduction/) |
| 54 | `permissions_privacy_ux` | permissions_privacy_ux | `primary_official` | **observed_capability**<br>Observed capability: LINE's channel-consent simplification grants only the openid scope up front; profile and message sc... | [https://developers.line.biz/en/docs/line-mini-app/develop/channel-consent-simplification/](https://developers.line.biz/en/docs/line-mini-app/develop/channel-consent-simplification/) |
| 55 | `lifecycle_review_localization` | lifecycle_review_localization | `primary_official` | **observed_capability**<br>Observed capability: LINE creates Developing, Review, and Published internal channels for each MINI App; review copies t... | [https://developers.line.biz/en/docs/line-mini-app/discover/console-guide/](https://developers.line.biz/en/docs/line-mini-app/discover/console-guide/) |
| 56 | `monetization_review_localization` | monetization_review_localization | `primary_official` | **observed_capability**<br>Observed capability: LINE MINI Apps support payments through an integrated payment system, while ad monetization uses LY... | [https://developers.line.biz/en/docs/line-mini-app/service/line-mini-app-ads/](https://developers.line.biz/en/docs/line-mini-app/service/line-mini-app-ads/) |
| 57 | `discovery_catalog_metadata_categories_search` | discovery_catalog_metadata_categories_search | `primary_official` | **observed_capability**<br>Observed capability: Microsoft Teams Store supports both search and category browsing. Search matches developer-provided... | [https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/deploy-and-publish/appsource/publish](https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/deploy-and-publish/appsource/publish) |
| 58 | `search_ranking_trust_reviews` | search_ranking_trust_reviews | `primary_official` | **observed_capability**<br>Observed capability: Microsoft publicly discloses that Teams Store ranking uses relevance parameters; higher app quality... | [https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/deploy-and-publish/appsource/post-publish/teams-store-ranking-parameters](https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/deploy-and-publish/appsource/post-publish/teams-store-ranking-parameters) |
| 59 | `enterprise_internal_permissions_validation_lifecycle` | enterprise_internal_permissions_validation_lifecycle | `primary_official` | **observed_capability**<br>Observed capability: Teams Developer Portal exposes device, team, chat/meeting, and user permissions; ownership roles in... | [https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/build-and-test/manage-your-apps-in-developer-portal](https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/build-and-test/manage-your-apps-in-developer-portal) |
| 60 | `analytics_support_lifecycle` | analytics_support_lifecycle | `primary_official` | **observed_capability**<br>Observed capability: after approval, Microsoft provides Teams app usage reporting with monthly, daily, and weekly active... | [https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/deploy-and-publish/appsource/publish](https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/deploy-and-publish/appsource/publish) |
| 61 | `monetization_entitlements_lifecycle` | monetization_entitlements_lifecycle | `primary_official` | **observed_capability**<br>Observed capability: Teams SaaS offers can use the SaaS Fulfillment API to manage subscription-plan lifecycle and the us... | [https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/deploy-and-publish/appsource/prepare/include-saas-offer](https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/deploy-and-publish/appsource/prepare/include-saas-offer) |
| 62 | `platform_scope` | platform_scope | `official-vendor` | **platform_scope**<br>Alibaba SuperApp Business Application Platform defines a superapp as a platform/ecosystem for miniapps and provides a fu... | [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp) |
| 63 | `lifecycle` | lifecycle | `official-vendor` | **lifecycle_control**<br>The Application Open Platform is described as a developer platform for full-lifecycle management, including developer re... | [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp) |
| 64 | `runtime` | runtime | `official-vendor` | **runtime_model**<br>Miniapps run inside a host superapp/container; WindVane uses a WebView container and JavaScript APIs to mediate web-to-n... | [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/introduction-to-superapp) |
| 65 | `developer_onboarding` | developer_onboarding | `official-vendor` | **developer_flow**<br>The documented WindVane flow separates native-app/container integration from miniapp development and publishing. The min... | [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/getting-started/rapid-development-of-windvane-applets-or-h5-applications](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/getting-started/rapid-development-of-windvane-applets-or-h5-applications) |
| 66 | `review` | review | `official-vendor` | **release_gate**<br>After preview/debug, a built package is submitted to the Open Platform for review and publishing. The store pipeline sho... | [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/getting-started/rapid-development-of-windvane-applets-or-h5-applications](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/getting-started/rapid-development-of-windvane-applets-or-h5-applications) |
| 67 | `release` | release | `official-vendor` | **progressive_release**<br>The documented release path requires a gray-scale/gradual rollout to a targeted user group for validation, followed by a... | [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/getting-started/rapid-development-of-windvane-applets-or-h5-applications](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/getting-started/rapid-development-of-windvane-applets-or-h5-applications) |
| 68 | `permissions` | permissions | `official-vendor` | **authorization**<br>Alibaba's IDE documentation exposes a Need Auth From App setting: invoking client capabilities can trigger an authorizat... | [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/installing-the-plug-in](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/installing-the-plug-in) |
| 69 | `architecture` | architecture | `official-vendor` | **control_plane_separation**<br>Alibaba distinguishes SuperApp Open Platform (core capabilities/content/data/services), Application Open Platform (minia... | [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/basic-concepts](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/basic-concepts) |
| 70 | `performance` | performance | `official-vendor` | **cache_tradeoff**<br>Alibaba documents ZCache as a preload/cache layer for H5 and mini programs and warns that preloading increases package s... | [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/basic-concepts](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/basic-concepts) |
| 71 | `registration/removal/version-retention` | registration/removal/version-retention | `official-vendor-practice` | **app_registration_and_removal**<br>The Application Open Platform console guide says only admins can create or delete miniapps. Creation starts from the Min... | [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/user-guide/application-development-1](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/user-guide/application-development-1) |
| 72 | `runtime/package/permissions/apis` | runtime/package/permissions/apis | `official-vendor-practice` | **runtime_package_and_api_model**<br>Alibaba describes mini programs as JavaScript applications running in a mobile mini-program container, with access to ne... | [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/latest](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/product-overview/latest) |
| 73 | `store-apis/preview/preload/versioning` | store-apis/preview/preload/versioning | `official-vendor-practice` | **store_sdk_operations_and_cache_versions**<br>After a native app integrates the WindVane or uni-app miniapp container, the SDK provides list, search, open, preview, a... | [https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/small-program-operation-2](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/small-program-operation-2) |
| 74 | `package-model/limits/on-demand-loading` | package-model/limits/on-demand-loading | `official-platform-documentation` | **on_demand_subpackage_runtime**<br>Weixin packages can be split into a main package and one or more subpackages at build time. The main package starts firs... | [https://developers.weixin.qq.com/miniprogram/en/dev/framework/subpackages.html](https://developers.weixin.qq.com/miniprogram/en/dev/framework/subpackages.html) |
| 75 | `submission/review/versioning/rollback` | submission/review/versioning/rollback | `official-platform-documentation` | **submission_review_and_version_lifecycle**<br>Weixin distinguishes preview from code upload: upload submits code for trial or audit and records a version number and p... | [https://developers.weixin.qq.com/miniprogram/en/dev/quickstart/basic/role.html](https://developers.weixin.qq.com/miniprogram/en/dev/quickstart/basic/role.html) |
| 76 | `permissions/authorization/apis` | permissions/authorization/apis | `official-platform-documentation` | **user_permission_authorization**<br>wx.authorize requests a named scope and immediately prompts the user to authorize a function or data access; if the user... | [https://developers.weixin.qq.com/miniprogram/en/dev/api/open-api/authorize/wx.authorize.html](https://developers.weixin.qq.com/miniprogram/en/dev/api/open-api/authorize/wx.authorize.html) |
| 77 | `registration/roles/submission/review` | registration/roles/submission/review | `official-vendor-practice` | **workspace_registration_roles_and_approval**<br>Alipay's workflow requires a developer account before development, then uses workspace roles such as Workspace Admin, Wo... | [https://miniprogram.alipay.com/docs-besuperapp/miniprogram_besuperapp/platform/workflow-procedures](https://miniprogram.alipay.com/docs-besuperapp/miniprogram_besuperapp/platform/workflow-procedures) |
| 78 | `runtime/package/review` | runtime/package/review | `official-vendor-practice` | **native_vs_html5_package_model**<br>Alipay documents two mini-program types: DSL (the default native mini program) and HTML5. For HTML5 mini programs, publi... | [https://miniprogram.alipay.com/docs-besuperapp/miniprogram_besuperapp/platform/miniprogramtype](https://miniprogram.alipay.com/docs-besuperapp/miniprogram_besuperapp/platform/miniprogramtype) |
| 79 | `versioning/review/pilot/grayscale/full-release` | versioning/review/pilot/grayscale/full-release | `official-vendor-practice` | **version_review_pilot_and_rollout**<br>Alipay's release workflow starts after an IDE-uploaded version is created in the workspace. A version can be released to... | [https://miniprogram.alipay.com/docs-besuperapp/miniprogram_besuperapp/platform/release](https://miniprogram.alipay.com/docs-besuperapp/miniprogram_besuperapp/platform/release) |
| 80 | `registration/runtime/permissions/endpoint` | registration/runtime/permissions/endpoint | `official-platform-documentation` | **liff_registration_endpoint_and_scopes**<br>A LIFF app is added to a LINE Login channel in the LINE Developers Console and can run in LINE or an external browser. T... | [https://developers.line.biz/en/docs/liff/registering-liff-apps/](https://developers.line.biz/en/docs/liff/registering-liff-apps/) |
| 81 | `versioning/deprecation/migration` | versioning/deprecation/migration | `official-platform-change-notice` | **sdk_deprecation_and_migration**<br>LINE's official release notes state that LIFF v1 will be discontinued and recommend LIFF v2. The same primary source rec... | [https://developers.line.biz/en/docs/liff/release-notes/](https://developers.line.biz/en/docs/liff/release-notes/) |
| 82 | `comparable-ecosystem/runtime/auth/discovery` | comparable-ecosystem/runtime/auth/discovery | `official-platform-documentation` | **bot_webapp_runtime_auth_and_discovery**<br>Telegram Mini Apps are web apps launched from bot web_app buttons, a bot menu button, direct links, and a configured Mai... | [https://core.telegram.org/bots/webapps](https://core.telegram.org/bots/webapps) |
| 83 | `privacy_permissions` | privacy_permissions | `proposal` | **privacy_permission_minimization**<br>MASVS-PRIVACY-1 says an app should minimize access to sensitive data/resources, request only what it absolutely needs wi... | [https://mas.owasp.org/MASVS/controls/MASVS-PRIVACY-1/](https://mas.owasp.org/MASVS/controls/MASVS-PRIVACY-1/) |
| 84 | `consent` | consent | `proposal` | **consent_and_data_control**<br>MASVS-PRIVACY-4 calls for user mechanisms to manage, delete, and modify data and change privacy settings, including revo... | [https://mas.owasp.org/MASVS/controls/MASVS-PRIVACY-4/](https://mas.owasp.org/MASVS/controls/MASVS-PRIVACY-4/) |
| 85 | `storage` | storage | `proposal` | **sensitive_data_leakage_prevention**<br>MASVS-STORAGE-2 says the app prevents leakage of sensitive data, including unintended exposure through APIs, backups, lo... | [https://mas.owasp.org/MASVS/controls/MASVS-STORAGE-2/](https://mas.owasp.org/MASVS/controls/MASVS-STORAGE-2/) |
| 86 | `runtime_sandbox` | runtime_sandbox | `proposal` | **webview_bridge_isolation**<br>MASVS-PLATFORM-2 says WebViews must be configured to prevent sensitive-data leakage and exposure of sensitive native fun... | [https://mas.owasp.org/MASVS/controls/MASVS-PLATFORM-2/](https://mas.owasp.org/MASVS/controls/MASVS-PLATFORM-2/) |
| 87 | `runtime_sandbox` | runtime_sandbox | `proposal` | **miniapp_runtime_sandbox**<br>ASVS V1.14.5 calls for deployments to sandbox, containerize, and/or isolate at the network level to deter attacks agains... | [https://raw.githubusercontent.com/OWASP/ASVS/v4.0.3/4.0/en/0x10-V1-Architecture.md](https://raw.githubusercontent.com/OWASP/ASVS/v4.0.3/4.0/en/0x10-V1-Architecture.md) |
| 88 | `supply_chain_runtime` | supply_chain_runtime | `proposal` | **third_party_component_encapsulation**<br>ASVS V14.2.6 says to reduce attack surface by sandboxing or encapsulating third-party libraries so only required behavio... | [https://raw.githubusercontent.com/OWASP/ASVS/v4.0.3/4.0/en/0x22-V14-Config.md](https://raw.githubusercontent.com/OWASP/ASVS/v4.0.3/4.0/en/0x22-V14-Config.md) |
| 89 | `package_signing` | package_signing | `proposal` | **package_code_integrity**<br>ASVS V10.3.2 requires integrity protections such as code signing or subresource integrity and says applications must not... | [https://raw.githubusercontent.com/OWASP/ASVS/v4.0.3/4.0/en/0x18-V10-Malicious.md](https://raw.githubusercontent.com/OWASP/ASVS/v4.0.3/4.0/en/0x18-V10-Malicious.md) |
| 90 | `package_signing` | package_signing | `proposal` | **release_integrity_verification**<br>SSDF PS.2.1 says producers should make release-integrity verification information available, including cryptographic has... | [https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-218.pdf](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-218.pdf) |
| 91 | `software_supply_chain` | software_supply_chain | `proposal` | **sbom_provenance_retention**<br>SSDF PS.3.2 says to collect, safeguard, maintain, and share provenance data for every release component, such as an SBOM... | [https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-218.pdf](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-218.pdf) |
| 92 | `vulnerability_management` | vulnerability_management | `proposal` | **dependency_vulnerability_lifecycle**<br>SSDF PW.4.4 calls for lifecycle verification of commercial, open-source, and other third-party components, automated det... | [https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-218.pdf](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-218.pdf) |
| 93 | `vulnerability_response` | vulnerability_response | `proposal` | **vulnerability_disclosure_response**<br>SSDF RV.1.3 calls for a vulnerability-disclosure policy, roles and processes, a product security incident response team,... | [https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-218.pdf](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-218.pdf) |
| 94 | `software_supply_chain` | software_supply_chain | `proposal` | **build_provenance_verification**<br>SLSA v1.2 requires producers to distribute provenance and describes provenance as verifiable information tying an artifa... | [https://slsa.dev/spec/v1.2/build-requirements](https://slsa.dev/spec/v1.2/build-requirements) |
| 95 | `api_authorization` | api_authorization | `proposal` | **object_level_authorization**<br>OWASP API1:2023 says the server must check whether the logged-in user may perform the requested action on the referenced... | [https://api-security.owasp.org/editions/2023/en/0xa1-broken-object-level-authorization/](https://api-security.owasp.org/editions/2023/en/0xa1-broken-object-level-authorization/) |
| 96 | `api_data_minimization` | api_data_minimization | `proposal` | **property_level_authorization**<br>OWASP API3:2023 recommends exposing only properties the user may access, avoiding generic serialization and mass assignm... | [https://api-security.owasp.org/editions/2023/en/0xa3-broken-object-property-level-authorization/](https://api-security.owasp.org/editions/2023/en/0xa3-broken-object-property-level-authorization/) |
| 97 | `api_governance` | api_governance | `proposal` | **api_hardening_baseline**<br>OWASP API8:2023 calls for repeatable hardening, continuous assessment, TLS for client, upstream, and downstream communic... | [https://api-security.owasp.org/editions/2023/en/0xa8-security-misconfiguration/](https://api-security.owasp.org/editions/2023/en/0xa8-security-misconfiguration/) |
| 98 | `api_governance` | api_governance | `proposal` | **api_inventory_version_retirement**<br>OWASP API9:2023 calls for an inventory of API hosts, environments, versions, access, integrated services, data flows, do... | [https://api-security.owasp.org/editions/2023/en/0xa9-improper-inventory-management/](https://api-security.owasp.org/editions/2023/en/0xa9-improper-inventory-management/) |
| 99 | `api_governance` | api_governance | `proposal` | **api_input_integrity**<br>ASVS V13.2.2 requires JSON schema validation before accepting input; V13.2.6 requires message headers and payloads to be... | [https://raw.githubusercontent.com/OWASP/ASVS/v4.0.3/4.0/en/0x21-V13-API.md](https://raw.githubusercontent.com/OWASP/ASVS/v4.0.3/4.0/en/0x21-V13-API.md) |
| 100 | `api_authentication` | api_authentication | `proposal` | **oauth_pkce_for_miniapps**<br>RFC 9700 states that public clients must use PKCE, recommends S256 because it does not expose the verifier, and requires... | [https://www.rfc-editor.org/rfc/rfc9700.html](https://www.rfc-editor.org/rfc/rfc9700.html) |
| 101 | `api_authentication` | api_authentication | `proposal` | **oauth_redirect_uri_integrity**<br>RFC 9700 requires exact string matching against pre-registered redirect URIs, except for localhost ports in native apps,... | [https://www.rfc-editor.org/rfc/rfc9700.html](https://www.rfc-editor.org/rfc/rfc9700.html) |
| 102 | `api_authentication` | api_authentication | `proposal` | **sender_constrained_tokens**<br>RFC 9449 explains that DPoP sender-constrains an access token to the party holding the private key, reducing replay impa... | [https://www.rfc-editor.org/rfc/rfc9449.html](https://www.rfc-editor.org/rfc/rfc9449.html) |
| 103 | `privacy_governance` | privacy_governance | `proposal` | **privacy_by_design_default**<br>The EDPB guidance explains that data protection by design/default requires appropriate measures and safeguards, only nec... | [https://www.edpb.europa.eu/sites/default/files/files/file1/edpb_guidelines_201904_dataprotection_by_design_and_by_default_v2.0_en.pdf](https://www.edpb.europa.eu/sites/default/files/files/file1/edpb_guidelines_201904_dataprotection_by_design_and_by_default_v2.0_en.pdf) |
| 104 | `consent` | consent | `proposal` | **consent_withdrawal_ux**<br>The EDPB guidance describes consent as freely given, specific, informed, and unambiguous, and says withdrawal should be ... | [https://www.edpb.europa.eu/sites/default/files/files/file1/edpb_guidelines_201904_dataprotection_by_design_and_by_default_v2.0_en.pdf](https://www.edpb.europa.eu/sites/default/files/files/file1/edpb_guidelines_201904_dataprotection_by_design_and_by_default_v2.0_en.pdf) |
| 105 | `auditability` | auditability | `proposal` | **audit_logging_and_redaction**<br>ASVS V7 says not to log credentials, payment details, or sensitive data; to log security-relevant authentication, access... | [https://raw.githubusercontent.com/OWASP/ASVS/v4.0.3/4.0/en/0x15-V7-Error-Logging.md](https://raw.githubusercontent.com/OWASP/ASVS/v4.0.3/4.0/en/0x15-V7-Error-Logging.md) |
| 106 | `accessibility` | accessibility | `proposal` | **keyboard_accessibility**<br>WCAG 2.2 Success Criterion 2.1.1 requires all functionality to be operable through a keyboard interface, except path-dep... | [https://www.w3.org/WAI/WCAG22/Understanding/keyboard](https://www.w3.org/WAI/WCAG22/Understanding/keyboard) |
| 107 | `accessibility` | accessibility | `proposal` | **accessible_component_semantics**<br>WCAG 2.2 Success Criterion 4.1.2 requires names and roles to be programmatically determinable, user-settable states/prop... | [https://www.w3.org/WAI/WCAG22/Understanding/name-role-value](https://www.w3.org/WAI/WCAG22/Understanding/name-role-value) |
| 108 | `accessibility` | accessibility | `proposal` | **accessible_contrast**<br>WCAG 2.2 Success Criterion 1.4.3 sets a minimum contrast ratio of 4.5:1 for normal text and 3:1 for large text, with sta... | [https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum) |
| 109 | `app_identity_scope_launch_context` | app_identity_scope_launch_context | `official-normative` | **manifest_identity_scope_and_deep_launch**<br>[NORMATIVE FACT] The Web App Manifest defines `id` as a URL same-origin with `start_url`, used by user agents to identif... | [https://www.w3.org/TR/appmanifest/](https://www.w3.org/TR/appmanifest/) |
| 110 | `url_canonicalization_launch_integrity` | url_canonicalization_launch_integrity | `official-normative` | **url_parsing_and_origin_boundary**<br>[NORMATIVE FACT] The URL Standard defines interoperable parsing and serialization; URL equality compares serialized URLs... | [https://url.spec.whatwg.org/](https://url.spec.whatwg.org/) |
| 111 | `canonical_urls_uri_equivalence` | canonical_urls_uri_equivalence | `official-normative` | **uri_normalization_profile**<br>[NORMATIVE FACT] RFC 3986 defines a comparison ladder and distinguishes syntax-, scheme-, and protocol-based normalizati... | [https://www.rfc-editor.org/rfc/rfc3986.html](https://www.rfc-editor.org/rfc/rfc3986.html) |
| 112 | `canonical_urls_discovery_identity` | canonical_urls_discovery_identity | `proposal` | **canonical_listing_aliases_not_identity**<br>[SOURCE STATUS] RFC 6596 is an IETF Informational RFC, not an Internet Standards Track specification. [NORMATIVE FACT IN... | [https://www.rfc-editor.org/rfc/rfc6596.html](https://www.rfc-editor.org/rfc/rfc6596.html) |
| 113 | `well_known_discovery_domain_association` | well_known_discovery_domain_association | `official-normative` | **well_known_discovery_contract**<br>[NORMATIVE FACT] RFC 8615 reserves the `/.well-known/` path prefix for registered well-known URIs on schemes that suppor... | [https://www.rfc-editor.org/rfc/rfc8615.html](https://www.rfc-editor.org/rfc/rfc8615.html) |
| 114 | `launch_context_auth_integrity` | launch_context_auth_integrity | `official-normative` | **claimed_https_callback_binding**<br>[NORMATIVE FACT] RFC 8252 is an IETF Best Current Practice: native-app authorization requests use an external user-agent... | [https://www.rfc-editor.org/rfc/rfc8252.html](https://www.rfc-editor.org/rfc/rfc8252.html) |
| 115 | `android_deep_links_domain_verification` | android_deep_links_domain_verification | `primary-platform` | **android_app_links_domain_proof**<br>[PLATFORM FACT] Android App Links use an intent filter with `android:autoVerify="true"`; Android retrieves the website's... | [https://developer.android.com/training/app-links/about](https://developer.android.com/training/app-links/about) |
| 116 | `android_deep_link_identity_fallback` | android_deep_link_identity_fallback | `primary-platform` | **android_https_over_custom_scheme**<br>[PLATFORM FACT] Android deep links are routed through the Intents system. Custom URI schemes can be registered by multip... | [https://developer.android.com/training/app-links/create-deeplinks](https://developer.android.com/training/app-links/create-deeplinks) |
| 117 | `android_dynamic_launch_policy` | android_dynamic_launch_policy | `primary-platform` | **android_dynamic_link_scope**<br>[PLATFORM FACT] On Android 15 and later devices with Google services, the `assetlinks.json` file can carry dynamic App L... | [https://developer.android.com/training/app-links/configure-assetlinks](https://developer.android.com/training/app-links/configure-assetlinks) |
| 118 | `android_domain_verification_lifecycle` | android_domain_verification_lifecycle | `primary-platform` | **android_per_host_verification_evidence**<br>[PLATFORM FACT] For each unique hostname in the app's intent filters, Android queries `https://hostname/.well-known/asse... | [https://developer.android.com/training/app-links/verify-applinks](https://developer.android.com/training/app-links/verify-applinks) |
| 119 | `apple_universal_links_launch_context` | apple_universal_links_launch_context | `primary-platform` | **apple_universal_link_context_and_fallback**<br>[PLATFORM FACT] Apple Universal Links are standard HTTP/HTTPS links: the same URL can serve the website or open the app,... | [https://developer.apple.com/documentation/xcode/allowing-apps-and-websites-to-link-to-your-content.md](https://developer.apple.com/documentation/xcode/allowing-apps-and-websites-to-link-to-your-content.md) |
| 120 | `apple_universal_links_domain_binding` | apple_universal_links_domain_binding | `primary-platform` | **apple_association_file_integrity**<br>[PLATFORM FACT] Apple requires an `apple-app-site-association` file and matching Associated Domains entitlement; the fil... | [https://developer.apple.com/documentation/xcode/supporting-associated-domains.md](https://developer.apple.com/documentation/xcode/supporting-associated-domains.md) |
| 121 | `http_caching_cache_invalidation` | http_caching_cache_invalidation | `official-normative` | **cache_policy_requirement**<br>[SOURCE FACT] RFC 9111 is an Internet Standards Track document for HTTP cache behavior. A cache MUST NOT store a respons... | [https://www.rfc-editor.org/rfc/rfc9111.html](https://www.rfc-editor.org/rfc/rfc9111.html) |
| 122 | `poor_network_stale_fallback` | poor_network_stale_fallback | `official-normative` | **stale_response_resilience**<br>[SOURCE FACT] RFC 5861 defines independent stale-while-revalidate and stale-if-error Cache-Control extensions. stale-whi... | [https://www.rfc-editor.org/rfc/rfc5861.html](https://www.rfc-editor.org/rfc/rfc5861.html) |
| 123 | `service_worker_offline_fallback` | service_worker_offline_fallback | `official-normative` | **offline_runtime_fallback**<br>[SOURCE FACT] The W3C Service Workers document describes a fetch event and a request/response store similar in design to... | [https://www.w3.org/TR/service-workers/](https://www.w3.org/TR/service-workers/) |
| 124 | `cache_layers_strategy_offline` | cache_layers_strategy_offline | `official-vendor` | **cache_layer_strategy_matrix**<br>[SOURCE FACT] Chrome's Workbox guidance states that the JavaScript Cache interface is separate from the browser HTTP cac... | [https://developer.chrome.com/docs/workbox/caching-strategies-overview](https://developer.chrome.com/docs/workbox/caching-strategies-overview) |
| 125 | `integrity_of_cached_artifacts` | integrity_of_cached_artifacts | `official-normative` | **cached_artifact_integrity**<br>[SOURCE FACT] W3C Subresource Integrity defines integrity metadata as a cryptographic hash and digest supplied with a re... | [https://www.w3.org/TR/SRI/](https://www.w3.org/TR/SRI/) |
| 126 | `startup_performance_measurement` | startup_performance_measurement | `official-normative` | **startup_timing_contract**<br>[SOURCE FACT] Navigation Timing exposes workerStart immediately before service-worker activation/start, fetchStart immed... | [https://www.w3.org/TR/navigation-timing-2/](https://www.w3.org/TR/navigation-timing-2/) |
| 127 | `resource_cache_performance_metrics` | resource_cache_performance_metrics | `official-normative` | **resource_transfer_budget_telemetry**<br>[SOURCE FACT] W3C Resource Timing includes resources retrieved from the HTTP cache and aborted network-error fetches in ... | [https://www.w3.org/TR/resource-timing/](https://www.w3.org/TR/resource-timing/) |
| 128 | `package_splitting_on_demand_delivery` | package_splitting_on_demand_delivery | `primary-platform` | **on_demand_package_splitting**<br>[SOURCE FACT] Android App Bundles generate device-specific delivery so users download only the code and resources needed... | [https://developer.android.com/guide/playcore/feature-delivery](https://developer.android.com/guide/playcore/feature-delivery) |
| 129 | `startup_main_thread_responsiveness` | startup_main_thread_responsiveness | `official-normative` | **main_thread_startup_budget**<br>[SOURCE FACT] The W3C Long Tasks API identifies tasks that monopolize the UI thread and defines a long task as exceeding... | [https://www.w3.org/TR/longtasks-1/](https://www.w3.org/TR/longtasks-1/) |
| 130 | `payments/tokenization/host-boundary` | payments/tokenization/host-boundary | `official-normative` | **payment_token_scope**<br>EMVCo states that payment tokenisation removes the primary account number (PAN) and replaces it with a unique alternativ... | [https://www.emvco.com/emv-technologies/payment-tokenisation/](https://www.emvco.com/emv-technologies/payment-tokenisation/) |
| 131 | `payments/tokenization/pci-scope` | payments/tokenization/pci-scope | `official-normative` | **tokenization_system_boundary**<br>The PCI SSC supplement says tokenization and de-tokenization should occur only inside a clearly defined tokenization sys... | [https://www.pcisecuritystandards.org/documents/Tokenization_Guidelines_Info_Supplement.pdf](https://www.pcisecuritystandards.org/documents/Tokenization_Guidelines_Info_Supplement.pdf) |
| 132 | `payments/purchase-entitlement/server-authority` | payments/purchase-entitlement/server-authority | `primary-platform` | **entitlement_server_authority**<br>Google Play documents a purchase flow in which the app verifies the purchase on a server before granting benefits. It de... | [https://developer.android.com/google/play/billing/integrate](https://developer.android.com/google/play/billing/integrate) |
| 133 | `payments/entitlements/webhook-idempotency` | payments/entitlements/webhook-idempotency | `primary-platform` | **entitlement_event_idempotency**<br>Google Play recommends a backend purchase-status management system for one-time purchases and subscriptions. Its RTDN fl... | [https://developer.android.com/google/play/billing/lifecycle](https://developer.android.com/google/play/billing/lifecycle) |
| 134 | `payments/refunds/chargebacks/ownership` | payments/refunds/chargebacks/ownership | `primary-platform` | **refund_dispute_event_ownership**<br>Google Play's notification reference distinguishes voided-purchase notifications from pending-refund-review notification... | [https://developer.android.com/google/play/billing/rtdn-reference](https://developer.android.com/google/play/billing/rtdn-reference) |
| 135 | `payments/webhooks/authenticity` | payments/webhooks/authenticity | `official-vendor` | **webhook_authenticity_raw_payload**<br>Stripe's official guidance says webhook consumers should verify that an event came from Stripe using the signature heade... | [https://docs.stripe.com/webhooks/signature](https://docs.stripe.com/webhooks/signature) |
| 136 | `payments/webhooks/message-signatures/replay` | payments/webhooks/message-signatures/replay | `official-normative` | **webhook_signature_replay_boundary**<br>RFC 9421 defines signing and verification of selected HTTP message components. It warns that unsigned components can be ... | [https://www.rfc-editor.org/rfc/rfc9421.html](https://www.rfc-editor.org/rfc/rfc9421.html) |
| 137 | `notifications/opt-in/consent` | notifications/opt-in/consent | `official-normative` | **notification_permission_user_control**<br>The W3C Permissions model treats permission as the user's decision to allow or deny a powerful feature; its explanatory ... | [https://www.w3.org/TR/permissions/](https://www.w3.org/TR/permissions/) |
| 138 | `notifications/push/abuse-controls` | notifications/push/abuse-controls | `official-normative` | **push_subscription_scope_and_quota**<br>The W3C Push API requires subscription creation to request push permission and reject the operation when permission is d... | [https://www.w3.org/TR/push-api/](https://www.w3.org/TR/push-api/) |
| 139 | `notifications/opt-in/abuse-controls` | notifications/opt-in/abuse-controls | `primary-platform` | **notification_opt_in_and_abuse_controls**<br>Android documents POST_NOTIFICATIONS as a runtime permission on Android 13 and higher: newly installed apps have notific... | [https://developer.android.com/develop/ui/views/notifications/notification-permission](https://developer.android.com/develop/ui/views/notifications/notification-permission) |
| 140 | `deep-links/domain-association/fallback` | deep-links/domain-association/fallback | `primary-platform` | **deep_link_association_and_fallback**<br>Apple documents Universal Links as standard HTTP/HTTPS links that other apps cannot claim, with a secure association che... | [https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/UniversalLinks.html](https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/UniversalLinks.html) |
| 141 | `marketplace/ranking/disclosure` | marketplace/ranking/disclosure | `official-normative` | **ranking_parameter_disclosure**<br>The European Commission explains that Regulation (EU) 2019/1150 covers online intermediation services including app stor... | [https://digital-strategy.ec.europa.eu/en/library/ranking-transparency-guidelines-framework-eu-regulation-platform-business-relations-explainer](https://digital-strategy.ec.europa.eu/en/library/ranking-transparency-guidelines-framework-eu-regulation-platform-business-relations-explainer) |
| 142 | `marketplace/reviews/ranking-integrity` | marketplace/reviews/ranking-integrity | `primary-platform` | **review_ranking_integrity**<br>Google Play policy prohibits attempts to manipulate app placement, including inflating ratings, reviews, or install coun... | [https://support.google.com/googleplay/android-developer/answer/9898684?hl=en](https://support.google.com/googleplay/android-developer/answer/9898684?hl=en) |
| 143 | `marketplace/reviews/incentives/disclosure` | marketplace/reviews/incentives/disclosure | `official-normative` | **review_incentive_disclosure_and_suppression**<br>FTC guidance says the Consumer Reviews and Testimonials Rule took effect on 21 October 2024 and addresses fake, false, o... | [https://www.ftc.gov/business-guidance/resources/consumer-reviews-testimonials-rule-questions-answers](https://www.ftc.gov/business-guidance/resources/consumer-reviews-testimonials-rule-questions-answers) |
| 144 | `payments/refunds/merchant-of-record/ownership` | payments/refunds/merchant-of-record/ownership | `official-vendor` | **refund_merchant_of_record_boundary**<br>Stripe's refund documentation says Connect refund responsibility depends on charge type: direct-charge refunds are charg... | [https://docs.stripe.com/refunds](https://docs.stripe.com/refunds) |
| 145 | `accessibility_inclusive_ux` | accessibility_inclusive_ux | `official-normative` | **catalog_discovery_reflow**<br>[SOURCE FACT] WCAG 2.2 SC 1.4.10 (Level AA) requires content to be presented without loss of information or functionalit... | [https://www.w3.org/TR/WCAG22/#reflow](https://www.w3.org/TR/WCAG22/#reflow) |
| 146 | `accessibility_inclusive_ux` | accessibility_inclusive_ux | `official-normative` | **app_detail_text_resize**<br>[SOURCE FACT] WCAG 2.2 SC 1.4.4 (Level AA) requires text, except captions and images of text, to be resizable up to 200 ... | [https://www.w3.org/TR/WCAG22/#resize-text](https://www.w3.org/TR/WCAG22/#resize-text) |
| 147 | `accessibility_inclusive_ux` | accessibility_inclusive_ux | `official-primary-guidance` | **catalog_search_filter_screen_reader_semantics**<br>[SOURCE FACT] The WAI-ARIA Authoring Practices Guide describes a combobox as an input with an associated popup and speci... | [https://www.w3.org/WAI/ARIA/apg/patterns/combobox/](https://www.w3.org/WAI/ARIA/apg/patterns/combobox/) |
| 148 | `accessibility_inclusive_ux` | accessibility_inclusive_ux | `official-normative` | **install_update_progress_and_result_announcements**<br>[SOURCE FACT] WCAG 2.2 SC 4.1.3 (Level AA) requires status messages to be programmatically determinable through roles or... | [https://www.w3.org/TR/WCAG22/#status-messages](https://www.w3.org/TR/WCAG22/#status-messages) |
| 149 | `accessibility_inclusive_ux` | accessibility_inclusive_ux | `official-normative+official-primary-guidance` | **permission_error_and_takedown_dialogs**<br>[SOURCE FACT - WCAG] When an input error is automatically detected, WCAG 2.2 SC 3.3.1 requires the item in error to be i... | [['https://www.w3.org/TR/WCAG22/#error-identification', 'https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/']](['https://www.w3.org/TR/WCAG22/#error-identification', 'https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/']) |
| 150 | `accessibility_inclusive_ux` | accessibility_inclusive_ux | `official-normative` | **contrast_and_keyboard_focus**<br>[SOURCE FACT] WCAG 2.2 SC 1.4.11 (Level AA) requires visual information needed to identify user-interface components and... | [['https://www.w3.org/TR/WCAG22/#non-text-contrast', 'https://www.w3.org/TR/WCAG22/#focus-visible']](['https://www.w3.org/TR/WCAG22/#non-text-contrast', 'https://www.w3.org/TR/WCAG22/#focus-visible']) |
| 151 | `accessibility_inclusive_ux` | accessibility_inclusive_ux | `official-normative` | **motion_and_time_limit_controls**<br>[SOURCE FACT] WCAG 2.2 SC 2.2.1 requires users to be able to turn off or adjust content-set time limits, or receive a wa... | [['https://www.w3.org/TR/WCAG22/#timing-adjustable', 'https://www.w3.org/TR/WCAG22/#pause-stop-hide', 'https://www.w3.org/TR/WCAG22/#animation-from-interactions']](['https://www.w3.org/TR/WCAG22/#timing-adjustable', 'https://www.w3.org/TR/WCAG22/#pause-stop-hide', 'https://www.w3.org/TR/WCAG22/#animation-from-interactions']) |
| 152 | `accessibility_inclusive_ux` | accessibility_inclusive_ux | `official-normative` | **localized_catalog_and_lifecycle_messages**<br>[SOURCE FACT] WCAG 2.2 requires the default human language of a page to be programmatically determinable (SC 3.1.1 Level... | [['https://www.w3.org/TR/WCAG22/#language-of-page', 'https://www.w3.org/TR/WCAG22/#language-of-parts']](['https://www.w3.org/TR/WCAG22/#language-of-page', 'https://www.w3.org/TR/WCAG22/#language-of-parts']) |
| 153 | `data_minimization_purpose_limitation` | data_minimization_purpose_limitation | `official-guidance` | **purpose_minimization_and_storage_baseline**<br>[SOURCE FACT] The EDPB identifies purpose limitation, data minimisation, storage limitation, integrity and confidentiali... | [https://www.edpb.europa.eu/topics/key-gdpr-concepts/basic-principles_en](https://www.edpb.europa.eu/topics/key-gdpr-concepts/basic-principles_en) |
| 154 | `consent_withdrawal` | consent_withdrawal | `official-guidance` | **consent_withdrawal_and_granularity**<br>[SOURCE FACT] The EDPB describes consent as freely given, specific, informed, and unambiguous; controllers must demonstr... | [https://www.edpb.europa.eu/system/files/documents/files/file1/edpb_guidelines_202005_consent_en.pdf](https://www.edpb.europa.eu/system/files/documents/files/file1/edpb_guidelines_202005_consent_en.pdf) |
| 155 | `deletion_data_portability` | deletion_data_portability | `official-guidance` | **deletion_export_and_propagation**<br>[SOURCE FACT] The EDPB guide lists access, erasure, and data portability rights; says controllers must facilitate reques... | [https://www.edpb.europa.eu/sme/be-compliant/respect-individuals-rights_en](https://www.edpb.europa.eu/sme/be-compliant/respect-individuals-rights_en) |
| 156 | `processor_subprocessor_transparency` | processor_subprocessor_transparency | `official-guidance` | **processor_subprocessor_registry_and_change_notice**<br>[SOURCE FACT] The EDPB says a processor acts on documented controller instructions; another processor requires prior wri... | [https://www.edpb.europa.eu/system/files/2023-10/EDPB_guidelines_202007_controllerprocessor_final_en.pdf](https://www.edpb.europa.eu/system/files/2023-10/EDPB_guidelines_202007_controllerprocessor_final_en.pdf) |
| 157 | `telemetry_governance` | telemetry_governance | `official-standard` | **telemetry_profile_and_lifecycle_governance**<br>[SOURCE FACT] NIST describes Current and Target Profiles for privacy outcomes, using Identify-P and Govern-P to identify... | [https://www.nist.gov/privacy-framework/using-privacy-framework-11](https://www.nist.gov/privacy-framework/using-privacy-framework-11) |
| 158 | `retention_storage_limitation` | retention_storage_limitation | `official-regulator-guidance` | **retention_schedule_and_deletion_verification**<br>[SOURCE FACT] The ICO says personal data must not be kept longer than needed, retention duration must be justified by th... | [https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/data-protection-principles/a-guide-to-the-data-protection-principles/storage-limitation/](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/data-protection-principles/a-guide-to-the-data-protection-principles/storage-limitation/) |
| 159 | `high_risk_dpia_re_review` | high_risk_dpia_re_review | `official-guidance` | **high_risk_privacy_review_trigger**<br>[SOURCE FACT] The EDPB says DPIAs help identify and manage risks to people’s personal data and must be carried out befor... | [https://www.edpb.europa.eu/topics/accountability-and-compliance-tools/data-protection-impact-assessment_en](https://www.edpb.europa.eu/topics/accountability-and-compliance-tools/data-protection-impact-assessment_en) |
| 160 | `children_privacy` | children_privacy | `official-guidance` | **child_data_and_age_assurance_safeguards**<br>[SOURCE FACT] The EDPB states that children receive specific protection under the GDPR because they are particularly vul... | [https://www.edpb.europa.eu/topics/key-gdpr-concepts/children_en](https://www.edpb.europa.eu/topics/key-gdpr-concepts/children_en) |
| 161 | `marketplace/moderation/notice-and-action; marketplace/moderation/publisher-user-appeals/procedural-fairness; marketplace/moderation/suspension/takedown/misuse-controls; marketplace/advertising/sponsored-placement/transparency` | marketplace/moderation/notice-and-action; marketplace/moderation/publisher-user-appeals/procedural-fairness; marketplace/moderation/suspension/takedown/misuse-controls; marketplace/advertising/sponsored-placement/transparency | `official-normative` | **notice_and_action_intake**<br>EU legal control: Article 16 requires hosting services to provide easy-to-access, user-friendly electronic notice mechan... | [https://publications.europa.eu/resource/celex/32022R2065.ENG.xhtml.L_2022277EN.01000101.doc.html](https://publications.europa.eu/resource/celex/32022R2065.ENG.xhtml.L_2022277EN.01000101.doc.html) |
| 162 | `marketplace/publisher-appeals/complaints/mediation` | marketplace/publisher-appeals/complaints/mediation | `official-normative` | **publisher_complaint_handling_and_mediation**<br>EU publisher/business-user redress control: the P2B Regulation covers online intermediation services and search engines ... | [https://publications.europa.eu/resource/celex/32019R1150.ENG.xhtml.L_2019186EN.01005701.doc.html](https://publications.europa.eu/resource/celex/32019R1150.ENG.xhtml.L_2019186EN.01005701.doc.html) |
| 163 | `marketplace/transparency-reporting/auditability/moderation-records` | marketplace/transparency-reporting/auditability/moderation-records | `official-guidance` | **moderation_transparency_and_auditability**<br>The Commission explains that DSA Article 17 statements of reasons cover hosting-service restrictions, and Article 24(5) ... | [https://digital-strategy.ec.europa.eu/en/faqs/dsa-transparency-database-questions-and-answers](https://digital-strategy.ec.europa.eu/en/faqs/dsa-transparency-database-questions-and-answers) |
| 164 | `marketplace/publisher-moderation/enforcement/appeals/takedown` | marketplace/publisher-moderation/enforcement/appeals/takedown | `primary-platform` | **publisher_enforcement_ladder_and_appeal**<br>Google Play-specific practice: Google says enforcement decisions may consider app metadata, in-app experience, account h... | [https://support.google.com/googleplay/android-developer/answer/9899234?hl=en](https://support.google.com/googleplay/android-developer/answer/9899234?hl=en) |
| 165 | `reliability/slo/error-budget/release-gating` | reliability/slo/error-budget/release-gating | `primary-official-guidance` | **slo_error_budget_governance**<br>[SOURCE FACT] Google defines an SLI as a carefully defined quantitative measure of a service level and an SLO as a targe... | [https://sre.google/sre-book/service-level-objectives/](https://sre.google/sre-book/service-level-objectives/) |
| 166 | `change-management/reproducible-builds/canary/rollback` | change-management/reproducible-builds/canary/rollback | `primary-official-guidance` | **release_reproducibility_canary_rollback**<br>[SOURCE FACT] Google SRE states that reliable services require reliable release processes: binaries and configurations s... | [https://sre.google/sre-book/release-engineering/](https://sre.google/sre-book/release-engineering/) |
| 167 | `incident-operations/postmortem/learning/action-tracking` | incident-operations/postmortem/learning/action-tracking | `primary-official-guidance` | **blameless_postmortem_action_tracking**<br>[SOURCE FACT] Google SRE describes a postmortem as a written record of an incident, its impact, mitigation or resolution... | [https://sre.google/sre-book/postmortem-culture/](https://sre.google/sre-book/postmortem-culture/) |
| 168 | `observability/trace-context/cross-service-correlation` | observability/trace-context/cross-service-correlation | `primary-standard` | **cross_service_telemetry_context**<br>[SOURCE FACT] The OpenTelemetry Context specification is marked Stable and defines Context as a propagation mechanism ca... | [https://opentelemetry.io/docs/specs/otel/context/](https://opentelemetry.io/docs/specs/otel/context/) |
| 169 | `operations/sli-slo-contract` | operations/sli-slo-contract | `open-specification` | **slo_schema_and_budgeting**<br>[SOURCE FACT] OpenSLO is an open, vendor-agnostic specification for defining SLOs; an SLO is a target or range described... | [https://raw.githubusercontent.com/OpenSLO/OpenSLO/main/website/docs/specification.md](https://raw.githubusercontent.com/OpenSLO/OpenSLO/main/website/docs/specification.md) |
| 170 | `operations/error-budget-release-gate` | operations/error-budget-release-gate | `engineering-reference` | **error_budget_release_gate**<br>[SOURCE FACT] Google's example policy says releases proceed when the service is at or above its SLO; when the preceding ... | [https://sre.google/workbook/error-budget-policy/](https://sre.google/workbook/error-budget-policy/) |
| 171 | `operations/tenant-and-runtime-isolation` | operations/tenant-and-runtime-isolation | `primary-platform` | **tenant_isolation_and_noisy_neighbor_control**<br>[SOURCE FACT] Kubernetes documents shared-cluster multi-tenancy as a tradeoff among security, fairness, noisy-neighbor c... | [https://kubernetes.io/docs/concepts/security/multi-tenancy/](https://kubernetes.io/docs/concepts/security/multi-tenancy/) |
| 172 | `operations/recovery-rto-rpo` | operations/recovery-rto-rpo | `primary-government-guidance` | **recovery_objectives_rto_rpo**<br>[SOURCE FACT] NIST SP 800-34 Rev. 1 defines Maximum Tolerable Downtime as the total outage time an owner is willing to a... | [https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-34r1.pdf](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-34r1.pdf) |
| 173 | `telemetry_schema_versioning` | telemetry_schema_versioning | `primary-project` | **telemetry_schema_compatibility_gate**<br>[SOURCE FACT] OpenTelemetry says telemetry schemas are versioned, each version is identified by a unique Schema URL, sou... | [https://opentelemetry.io/docs/specs/otel/schemas/](https://opentelemetry.io/docs/specs/otel/schemas/) |
| 174 | `release_cohort_monitoring` | release_cohort_monitoring | `engineering-reference` | **cohort_differential_release_gate**<br>[SOURCE FACT] Google SRE describes canarying as testing a new release on a small subset of a typical workload and says c... | [https://sre.google/sre-book/embracing-risk/](https://sre.google/sre-book/embracing-risk/) |
| 175 | `portable_catalog_manifest_federation` | portable_catalog_manifest_federation | `primary-standard` | **manifest_discovery_and_federated_export**<br>[SOURCE FACT] The W3C Web Application Manifest is a JSON document containing startup parameters and application defaults... | [https://www.w3.org/TR/2026/WD-appmanifest-20260813/#link-relation-type-registration](https://www.w3.org/TR/2026/WD-appmanifest-20260813/#link-relation-type-registration) |
| 176 | `portable_localization_manifest_projection` | portable_localization_manifest_projection | `primary-standard` | **localized_catalog_labels_and_icons**<br>[SOURCE FACT] The manifest's `lang` is a BCP 47 language tag for localizable member values (unknown if absent), and `dir... | [https://www.w3.org/TR/2026/WD-appmanifest-20260813/#x_localized-members](https://www.w3.org/TR/2026/WD-appmanifest-20260813/#x_localized-members) |
| 177 | `portable_icons_display_capability_projection` | portable_icons_display_capability_projection | `primary-standard` | **portable_icons_presentation_and_capability_projection**<br>[SOURCE FACT] The manifest's `icons` member supplies images representing the web application in contexts such as applica... | [https://www.w3.org/TR/2026/WD-appmanifest-20260813/#icons-member](https://www.w3.org/TR/2026/WD-appmanifest-20260813/#icons-member) |
| 178 | `compatibility_and_federated_schema_export` | compatibility_and_federated_schema_export | `primary-official` | **schema_software_application_compatibility_and_catalog_projection**<br>[SOURCE FACT] Schema.org's official `SoftwareApplication` type page lists reusable application/catalog properties includ... | [https://schema.org/SoftwareApplication](https://schema.org/SoftwareApplication) |
| 179 | `sbom_vex_portability` | sbom_vex_portability | `primary-standard` | **sbom_vex_profile_portability**<br>[SOURCE FACT] SPDX 3.0.1 defines an RDF-based data model for representing and exchanging information about systems with ... | [https://spdx.github.io/spdx-spec/v3.0.1/](https://spdx.github.io/spdx-spec/v3.0.1/) |
| 180 | `sbom_vex_signature_portability` | sbom_vex_signature_portability | `primary-standard` | **sbom_vex_signature_portability**<br>[SOURCE FACT] The CycloneDX v1.6 JSON reference requires bomFormat=CycloneDX and specVersion=1.6; it recommends a unique... | [https://cyclonedx.org/docs/1.6/json/](https://cyclonedx.org/docs/1.6/json/) |
| 181 | `attestation_bundle_portability` | attestation_bundle_portability | `primary-standard` | **attestation_bundle_portability**<br>[SOURCE FACT] The in-toto site lists the Attestation Framework as a Stable v1.0 specification, while the versioned layer... | [https://in-toto.io/docs/specs/](https://in-toto.io/docs/specs/) |
| 182 | `vex_status_portability` | vex_status_portability | `primary-project` | **vex_status_portability**<br>[SOURCE FACT] OpenVEX describes itself as a minimal, compliant, interoperable, embeddable VEX implementation that is SBO... | [https://github.com/openvex/spec](https://github.com/openvex/spec) |
| 183 | `publisher_registration_metadata` | publisher_registration_metadata | `official-normative` | **publisher_registration_metadata_negotiation**<br>[NORMATIVE RFC FACT] RFC 7591 is an IETF Standards Track specification for JSON-based dynamic client registration. A cli... | [https://www.rfc-editor.org/rfc/rfc7591.html](https://www.rfc-editor.org/rfc/rfc7591.html) |
| 184 | `publisher_registration_credential_lifecycle` | publisher_registration_credential_lifecycle | `official-experimental` | **publisher_registration_credential_lifecycle**<br>[NORMATIVE RFC FACT] RFC 7592 is an IETF Experimental specification for managing dynamic client registrations over their... | [https://www.rfc-editor.org/rfc/rfc7592.html](https://www.rfc-editor.org/rfc/rfc7592.html) |
| 185 | `authorization_server_metadata_discovery` | authorization_server_metadata_discovery | `official-normative` | **authorization_server_metadata_discovery**<br>[NORMATIVE RFC FACT] RFC 8414 is an IETF Standards Track specification for machine-readable authorization-server metadat... | [https://www.rfc-editor.org/rfc/rfc8414.html](https://www.rfc-editor.org/rfc/rfc8414.html) |
| 186 | `resource_bound_least_privilege` | resource_bound_least_privilege | `official-normative` | **resource_bound_least_privilege**<br>[NORMATIVE RFC FACT] RFC 8707 is an IETF Standards Track specification that adds the resource parameter to authorization... | [https://www.rfc-editor.org/rfc/rfc8707.html](https://www.rfc-editor.org/rfc/rfc8707.html) |
| 187 | `runtime-api/rate-limits/429` | runtime-api/rate-limits/429 | `official-normative` | **rate_limit_response_contract**<br>[SOURCE FACT] RFC 6585 defines 429 Too Many Requests for a client that has sent too many requests in a given amount of t... | [https://www.rfc-editor.org/rfc/rfc6585.html](https://www.rfc-editor.org/rfc/rfc6585.html) |
| 188 | `runtime-api/retry-semantics/overload` | runtime-api/retry-semantics/overload | `official-normative` | **retry_after_backoff_semantics**<br>[SOURCE FACT] RFC 9110 says servers send Retry-After to indicate how long a user agent ought to wait before a follow-up ... | [https://www.rfc-editor.org/rfc/rfc9110.html](https://www.rfc-editor.org/rfc/rfc9110.html) |
| 189 | `runtime-api/errors/problem-details` | runtime-api/errors/problem-details | `official-normative` | **machine_readable_api_error_contract**<br>[SOURCE FACT] RFC 9457 defines problem details as machine-readable error details in HTTP response content; JSON uses the... | [https://www.rfc-editor.org/rfc/rfc9457.html](https://www.rfc-editor.org/rfc/rfc9457.html) |
| 190 | `runtime-api/quotas/ratelimit-headers` | runtime-api/quotas/ratelimit-headers | `official-ietf-draft` | **quota_advertisement_and_partitioned_budget**<br>[SOURCE FACT] This active Internet-Draft, not a final RFC, defines structured RateLimit-Policy and RateLimit response fi... | [https://www.ietf.org/archive/id/draft-ietf-httpapi-ratelimit-headers-11.html](https://www.ietf.org/archive/id/draft-ietf-httpapi-ratelimit-headers-11.html) |
| 191 | `tenant_aggregate_quotas_object_exhaustion` | tenant_aggregate_quotas_object_exhaustion | `platform-engineering-precedent+mini-app-store-proposal` | **tenant_aggregate_quota_and_object_exhaustion**<br>[PLATFORM ENGINEERING PRECEDENT] Kubernetes defines ResourceQuota as an aggregate constraint on resource consumption per... | [https://kubernetes.io/docs/concepts/policy/resource-quotas/](https://kubernetes.io/docs/concepts/policy/resource-quotas/) |
| 192 | `per_workload_limits_runtime_admission` | per_workload_limits_runtime_admission | `platform-engineering-precedent+mini-app-store-proposal` | **per_workload_limits_and_admission_defaults**<br>[PLATFORM ENGINEERING PRECEDENT] Kubernetes LimitRange constrains resource allocations for applicable object kinds; it c... | [https://kubernetes.io/docs/concepts/policy/limit-range/](https://kubernetes.io/docs/concepts/policy/limit-range/) |
| 193 | `admission_enforcement_tenant_policy` | admission_enforcement_tenant_policy | `platform-engineering-precedent+mini-app-store-proposal` | **admission_policy_for_tenant_runtime_requests**<br>[PLATFORM ENGINEERING PRECEDENT] Kubernetes Validating Admission Policy is a declarative, in-process alternative to vali... | [https://kubernetes.io/docs/reference/access-authn-authz/validating-admission-policy/](https://kubernetes.io/docs/reference/access-authn-authz/validating-admission-policy/) |
| 194 | `layered_tenant_isolation_boundaries` | layered_tenant_isolation_boundaries | `platform-engineering-precedent+mini-app-store-proposal` | **layered_tenant_isolation_boundary**<br>[PLATFORM ENGINEERING PRECEDENT] Kubernetes frames multi-tenancy around security, fairness, and noisy neighbors, and des... | [https://kubernetes.io/docs/concepts/security/multi-tenancy/](https://kubernetes.io/docs/concepts/security/multi-tenancy/) |
| 195 | `api_resource_exhaustion_abuse_resistance` | api_resource_exhaustion_abuse_resistance | `platform-engineering-precedent+mini-app-store-proposal` | **api_resource_exhaustion_and_abuse_controls**<br>[PLATFORM/API SECURITY PRECEDENT] OWASP API4 identifies missing or inappropriate limits on execution time, memory, file ... | [https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/](https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/) |
| 196 | `cost_attribution_showback_chargeback` | cost_attribution_showback_chargeback | `official-guidance` | **cost_allocation_owner_dimensions**<br>[SOURCE FACT] The guide defines cost allocation as identifying, categorizing, and assigning cloud-resource costs to user... | [https://www.finops.org/wg/cloud-cost-allocation/](https://www.finops.org/wg/cloud-cost-allocation/) |
| 197 | `shared_costs_fair_allocation` | shared_costs_fair_allocation | `official-guidance` | **shared_cost_attribution_governance**<br>[SOURCE FACT] The guide says allocating shared costs back to the business areas that created the spend creates transpare... | [https://www.finops.org/wg/identifying-shared-costs/](https://www.finops.org/wg/identifying-shared-costs/) |
| 198 | `cost_telemetry_owner_resource_dimensions` | cost_telemetry_owner_resource_dimensions | `official-semantic-conventions` | **cost_telemetry_owner_resource_dimensions**<br>[SOURCE FACT] OpenTelemetry defines service.namespace as a required namespace that groups related services and notes tha... | [https://opentelemetry.io/docs/specs/semconv/resource/service/](https://opentelemetry.io/docs/specs/semconv/resource/service/) |
| 199 | `usage_attribution_shared_idle_costs` | usage_attribution_shared_idle_costs | `official-project-documentation` | **workload_shared_idle_cost_attribution**<br>[SOURCE FACT] The OpenCost specification supports workload cost aggregation by container, pod, deployment, statefulset, ... | [https://opencost.io/docs/specification/](https://opencost.io/docs/specification/) |
| 200 | `localization/bcp47/language-tag-validation` | localization/bcp47/language-tag-validation | `official-bcp` | **language_tag_validation_and_variants**<br>[SOURCE FACT] RFC 5646 defines BCP 47 language tags as hyphen-separated subtags with a language followed optionally by s... | [https://www.rfc-editor.org/rfc/rfc5646.html](https://www.rfc-editor.org/rfc/rfc5646.html) |
| 201 | `localization/bcp47/locale-matching-fallback` | localization/bcp47/locale-matching-fallback | `official-bcp` | **language_range_matching_and_fallback**<br>[SOURCE FACT] RFC 4647 defines Basic Filtering and Extended Filtering, which can return a possibly empty set of matching... | [https://www.rfc-editor.org/rfc/rfc4647.html](https://www.rfc-editor.org/rfc/rfc4647.html) |
| 202 | `localization/cldr/inheritance-script-region-fallback` | localization/cldr/inheritance-script-region-fallback | `official-standard` | **cldr_locale_inheritance_script_region_fallback**<br>[SOURCE FACT] UTS #35 organizes locale resources into bundles in an inheritance tree. Its example resource chain is `en_... | [https://www.unicode.org/reports/tr35/#Locale_Inheritance_and_Matching](https://www.unicode.org/reports/tr35/#Locale_Inheritance_and_Matching) |
| 203 | `localization/catalog-metadata/translation-versioning` | localization/catalog-metadata/translation-versioning | `official-project` | **cldr_release_and_translation_metadata_versioning**<br>[SOURCE FACT] The Unicode CLDR Releases/Downloads page says each CLDR release is stable, may be cited as a normative ref... | [https://cldr.unicode.org/index/downloads](https://cldr.unicode.org/index/downloads) |
| 204 | `http_language_negotiation_vary_cache_correctness` | http_language_negotiation_vary_cache_correctness | `official-normative` | **localized_representation_negotiation_and_cache_key**<br>[SOURCE FACT] RFC 9110 defines Accept-Language as a request field for preferred natural languages, with language ranges ... | [https://datatracker.ietf.org/doc/html/rfc9110#name-accept-language](https://datatracker.ietf.org/doc/html/rfc9110#name-accept-language) |
| 205 | `locale_aware_discovery_urls_user_override_sticky_selection` | locale_aware_discovery_urls_user_override_sticky_selection | `official-i18n-guidance` | **locale_sticky_discovery_urls_and_user_override**<br>[SOURCE FACT] W3C explains that HTTP language negotiation selects among language versions using the URL and browser pref... | [https://www.w3.org/International/questions/qa-when-lang-neg](https://www.w3.org/International/questions/qa-when-lang-neg) |
| 206 | `locale_region_selection_privacy_and_shared_devices` | locale_region_selection_privacy_and_shared_devices | `official-i18n-guidance` | **accept_language_not_locale_or_region_authority**<br>[SOURCE FACT] W3C says Accept-Language was originally intended to specify language, not a complete user locale; using it... | [https://www.w3.org/International/questions/qa-accept-lang-locales.en.html](https://www.w3.org/International/questions/qa-accept-lang-locales.en.html) |
| 207 | `content_language_api_metadata_html_lang_consistency` | content_language_api_metadata_html_lang_consistency | `official-i18n-guidance` | **content_language_metadata_and_text_language_separation**<br>[SOURCE FACT] W3C distinguishes HTTP Content-Language as metadata about the intended audience of the resource from HTML ... | [https://www.w3.org/International/questions/qa-http-and-lang/qa-html-language-declarations](https://www.w3.org/International/questions/qa-http-and-lang/qa-html-language-declarations) |
| 208 | `regional_number_currency_formatting` | regional_number_currency_formatting | `primary-standard` | **regional_number_currency_formatting_contract**<br>[SOURCE FACT] ECMA-402 defines Intl.NumberFormat with locale input and options for decimal, percent, currency, and unit ... | [https://402.ecma-international.org/#numberformat-objects](https://402.ecma-international.org/#numberformat-objects) |
| 209 | `time_zone_identifier_contract` | time_zone_identifier_contract | `primary-standard` | **time_zone_identifier_contract**<br>[SOURCE FACT] RFC 9557 defines an IANA Time Zone as a named zone from the IANA Time Zone Database; its rules can change,... | [https://www.rfc-editor.org/rfc/rfc9557.html#section-1.2](https://www.rfc-editor.org/rfc/rfc9557.html#section-1.2) |
| 210 | `target_audience_declaration` | target_audience_declaration | `official-guidance` | **target_audience_age_band_declaration**<br>[SOURCE FACT] Google Play requires a new app or an update to declare its target age group in the App content section; mu... | [https://support.google.com/googleplay/android-developer/answer/9867159?hl=en](https://support.google.com/googleplay/android-developer/answer/9867159?hl=en) |
| 211 | `content_rating_release_gating` | content_rating_release_gating | `official-policy` | **content_rating_questionnaire_and_release_gate**<br>[SOURCE FACT] Google Play says content ratings are assigned by separate rating authorities from the developer's question... | [https://support.google.com/googleplay/android-developer/answer/9859655?hl=en](https://support.google.com/googleplay/android-developer/answer/9859655?hl=en) |
| 212 | `child_safety_controls` | child_safety_controls | `official-policy` | **child_safety_capabilities_and_mixed_audience_controls**<br>[SOURCE FACT] Google Play requires child-inclusive apps to keep content accessible to children appropriate, keep Target ... | [https://support.google.com/googleplay/android-developer/answer/9893335?hl=en](https://support.google.com/googleplay/android-developer/answer/9893335?hl=en) |
| 213 | `release_gating_declarations` | release_gating_declarations | `official-guidance` | **app_content_declarations_and_release_readiness**<br>[SOURCE FACT] Google Play describes the App content page as the place to manage safety, policy, and legal information, i... | [https://support.google.com/googleplay/android-developer/answer/9859455?hl=en](https://support.google.com/googleplay/android-developer/answer/9859455?hl=en) |
| 214 | `age_assurance_host_adapter` | age_assurance_host_adapter | `official-platform-doc` | **optional_age_signals_safe_mode_and_parental_approval_gate**<br>[SOURCE FACT] Android's Play Age Signals documentation describes a beta API for age-related signals and for notifying Go... | [['https://developer.android.com/google/play/age-signals/overview', 'https://developer.android.com/google/play/age-signals/request-age-signals']](['https://developer.android.com/google/play/age-signals/overview', 'https://developer.android.com/google/play/age-signals/request-age-signals']) |
| 215 | `apple/mini-apps/commerce-eligibility/host-partner-boundary` | apple/mini-apps/commerce-eligibility/host-partner-boundary | `primary-platform` | **apple_mini_app_commerce_eligibility_and_host_partner_boundary**<br>[APPLE PLATFORM PATTERN] Apple defines a qualifying mini app as one put out by a person or entity not directly or indire... | [https://developer.apple.com/programs/mini-apps-partner/](https://developer.apple.com/programs/mini-apps-partner/) |
| 216 | `apple/advanced-commerce-api/mini-app-product-identity-sku` | apple/advanced-commerce-api/mini-app-product-identity-sku | `primary-platform` | **apple_mini_app_product_identity_and_sku_contract**<br>[APPLE PLATFORM PATTERN] Apple requires mini-app product display names and SKUs to fully identify the mini app product u... | [https://developer.apple.com/documentation/advancedcommerceapi/creating-skus-for-the-mini-app-partner-program.md](https://developer.apple.com/documentation/advancedcommerceapi/creating-skus-for-the-mini-app-partner-program.md) |
| 217 | `apple/advanced-commerce-api/eligibility-catalog-change-governance` | apple/advanced-commerce-api/eligibility-catalog-change-governance | `primary-platform` | **apple_advanced_commerce_eligibility_and_change_governance**<br>[APPLE PLATFORM PATTERN] Apple limits Advanced Commerce API access to apps whose core business uses Apple In-App Purchas... | [https://developer.apple.com/in-app-purchase/advanced-commerce-api/](https://developer.apple.com/in-app-purchase/advanced-commerce-api/) |
| 218 | `apple/app-store-server-api/transaction-subscription-event-reconciliation` | apple/app-store-server-api/transaction-subscription-event-reconciliation | `primary-platform` | **apple_transaction_subscription_event_reconciliation**<br>[APPLE PLATFORM PATTERN] Apple’s App Store Server API returns App Store-signed transaction and subscription-renewal info... | [https://developer.apple.com/documentation/appstoreserverapi.md](https://developer.apple.com/documentation/appstoreserverapi.md) |
| 219 | `apple/refunds/consumption-information/consent-deadline` | apple/refunds/consumption-information/consent-deadline | `primary-platform` | **apple_refund_consumption_consent_and_deadline_governance**<br>[APPLE PLATFORM PATTERN] When a customer requests a refund for any In-App Purchase type, Apple can send a CONSUMPTION_RE... | [https://developer.apple.com/documentation/appstoreserverapi/send-consumption-information.md](https://developer.apple.com/documentation/appstoreserverapi/send-consumption-information.md) |
| 220 | `apple/app-store-connect/financial-reports/publisher-payout-settlement-evidence` | apple/app-store-connect/financial-reports/publisher-payout-settlement-evidence | `primary-platform` | **apple_publisher_payout_evidence_and_settlement_reconciliation**<br>[APPLE REPORTING PATTERN] Apple says Advanced Commerce purchases appear in App Store Connect Summary Sales Reports and P... | [https://developer.apple.com/help/app-store-connect/reference/financial-report-fields/](https://developer.apple.com/help/app-store-connect/reference/financial-report-fields/) |
| 221 | `monetization_settlement_catalog` | monetization_settlement_catalog | `official-platform-doc` | **google_play_catalog_identity_and_reconciliation**<br>[SOURCE FACT] Google's Play Developer API guide separates one-time and subscription catalog models. Subscription catalog... | [https://developer.android.com/google/play/billing/manage-catalog](https://developer.android.com/google/play/billing/manage-catalog) |
| 222 | `subscription_settlement_reconciliation` | subscription_settlement_reconciliation | `official-platform-guidance` | **google_play_subscription_price_cohort_and_payout_timing**<br>[SOURCE FACT] Google defines a base plan by billing period, renewal type (auto-renewing or prepaid), and price; a subscr... | [https://support.google.com/googleplay/android-developer/answer/12154973](https://support.google.com/googleplay/android-developer/answer/12154973) |
| 223 | `voided_purchase_settlement` | voided_purchase_settlement | `official-api-reference` | **google_play_voided_purchase_rolling_reconciliation**<br>[SOURCE FACT] purchases.voidedpurchases.list returns canceled, refunded, or charged-back purchases. It supports filters ... | [https://developers.google.com/android-publisher/api-ref/rest/v3/purchases.voidedpurchases/list](https://developers.google.com/android-publisher/api-ref/rest/v3/purchases.voidedpurchases/list) |
| 224 | `financial_reporting_payout_reconciliation` | financial_reporting_payout_reconciliation | `official-help` | **google_play_financial_report_and_order_ledger**<br>[SOURCE FACT] Google Play provides monthly earnings reports and daily estimated sales reports; the latter add recently c... | [https://support.google.com/googleplay/android-developer/answer/2482017?hl=en-sg](https://support.google.com/googleplay/android-developer/answer/2482017?hl=en-sg) |
| 225 | `pricing_refund_transparency` | pricing_refund_transparency | `official-policy-and-help` | **google_play_price_refund_transparency**<br>[SOURCE FACT] Google Play's subscriptions policy requires explicit disclosure of cost, billing frequency, automatic rene... | [https://support.google.com/googleplay/android-developer/answer/9900533](https://support.google.com/googleplay/android-developer/answer/9900533) |
| 226 | `settlement-ledger/immutable-provider-transactions/payout-batch-linkage` | settlement-ledger/immutable-provider-transactions/payout-batch-linkage | `official-vendor-pattern+design-proposal` | **immutable_balance_transaction_payout_batch_reconciliation**<br>[SOURCE FACT — STRIPE-SPECIFIC] Stripe recommends automatic payouts because they preserve the association between each t... | [https://docs.stripe.com/plan-integration/get-started/reporting-reconciliation](https://docs.stripe.com/plan-integration/get-started/reporting-reconciliation) |
| 227 | `reconciliation/report-runs/watermarks/failed-payouts/ending-balance` | reconciliation/report-runs/watermarks/failed-payouts/ending-balance | `official-vendor-pattern+design-proposal` | **payout_report_watermarks_failed_payouts_ending_balance**<br>[SOURCE FACT — STRIPE-SPECIFIC] Stripe's payout reconciliation report is designed to match a bank payout with its paymen... | [https://docs.stripe.com/reports/payout-reconciliation](https://docs.stripe.com/reports/payout-reconciliation) |
| 228 | `disputes/chargebacks/reserves/negative-balances/publisher-payout-controls` | disputes/chargebacks/reserves/negative-balances/publisher-payout-controls | `official-vendor-pattern+design-proposal` | **dispute_liability_reserve_and_publisher_payout_gate**<br>[SOURCE FACT — STRIPE-SPECIFIC] Stripe says charge type and negative-balance responsibility determine who responds to a ... | [https://docs.stripe.com/connect/disputes](https://docs.stripe.com/connect/disputes) |
| 229 | `settlement-ledger/value-date/transfer-transaction-linkage/commission-payout-reconciliation` | settlement-ledger/value-date/transfer-transaction-linkage/commission-payout-reconciliation | `official-vendor-pattern+design-proposal` | **value_date_pending_posted_and_marketplace_commission_reconciliation**<br>[SOURCE FACT — ADYEN-SPECIFIC] Adyen's Balance Platform Accounting Report is a daily report of balance changes across li... | [https://docs.adyen.com/platforms/reports-and-fees/balance-platform-accounting-report/](https://docs.adyen.com/platforms/reports-and-fees/balance-platform-accounting-report/) |
| 230 | `iso20022/camt053/bank-statement/reconciliation-control-totals` | iso20022/camt053/bank-statement/reconciliation-control-totals | `official-standard-draft+design-proposal` | **bank_statement_entry_reference_booking_value_date_and_control_totals**<br>[SOURCE FACT — ISO 20022 OFFICIAL REVIEW DRAFT, NOT A FINAL UNIVERSAL MARKETPLACE SPEC] The official ISO 20022 message-d... | [https://www.iso20022.org/sites/default/files/2020-12/ISO20022_MDRPart2_BankToCustomerCashManagement_2020_2021_v1_ForSEGReview.pdf](https://www.iso20022.org/sites/default/files/2020-12/ISO20022_MDRPart2_BankToCustomerCashManagement_2020_2021_v1_ForSEGReview.pdf) |
| 231 | `accounting/refund-liability/contract-liability/publisher-payable/revenue-recognition` | accounting/refund-liability/contract-liability/publisher-payable/revenue-recognition | `official-accounting-standard+design-proposal` | **refund_liability_contract_liability_periodic_update**<br>[SOURCE FACT — ACCOUNTING STANDARD, APPLICATION DEPENDS ON THE REPORTING ENTITY AND CONTRACT] IFRS 15 paragraph 16 requi... | [https://www.ifrs.org/content/dam/ifrs/publications/html-standards/english/2026/issued/ifrs15.html](https://www.ifrs.org/content/dam/ifrs/publications/html-standards/english/2026/issued/ifrs15.html) |
| 232 | `oauth_par_authorization_request_integrity` | oauth_par_authorization_request_integrity | `official-normative` | **par_request_integrity_and_single_use_binding**<br>[NORMATIVE RFC FACT] RFC 9126 is an Internet Standards Track document. Its pushed-authorization-request (PAR) endpoint M... | [https://www.rfc-editor.org/rfc/rfc9126.html](https://www.rfc-editor.org/rfc/rfc9126.html) |
| 233 | `oauth_issuer_binding_account_linking` | oauth_issuer_binding_account_linking | `official-normative` | **authorization_response_issuer_binding**<br>[NORMATIVE RFC FACT] RFC 9207 defines the OAuth authorization-response iss parameter. An authorization server supporting... | [https://www.rfc-editor.org/rfc/rfc9207.html](https://www.rfc-editor.org/rfc/rfc9207.html) |
| 234 | `oauth_introspection_session_liveness_identity` | oauth_introspection_session_liveness_identity | `official-normative` | **introspection_identity_audience_and_session_liveness**<br>[NORMATIVE RFC FACT] RFC 7662 defines an introspection endpoint that returns JSON metadata with a required active indica... | [https://www.rfc-editor.org/rfc/rfc7662.html](https://www.rfc-editor.org/rfc/rfc7662.html) |
| 235 | `authentication/rp-id-origin/host-boundary` | authentication/rp-id-origin/host-boundary | `official-w3c-recommendation+design-proposal` | **host_mediated_rp_id_origin_scope**<br>[STANDARD FACT — W3C RECOMMENDATION] WebAuthn credentials are scoped by the RP ID: the client MUST verify that the RP or... | [https://www.w3.org/TR/webauthn-3/](https://www.w3.org/TR/webauthn-3/) |
| 236 | `authentication/user-verification/step-up/opaque-contract` | authentication/user-verification/step-up/opaque-contract | `official-w3c-recommendation+working-draft+design-proposal` | **host_mediated_user_verification_step_up**<br>[STANDARD FACT — W3C RECOMMENDATION] For an operation requiring user verification, WebAuthn userVerification=required ma... | [https://www.w3.org/TR/passkey-endpoints/](https://www.w3.org/TR/passkey-endpoints/) |
| 237 | `session-security/dpop/nonce-replay/transaction-binding` | session-security/dpop/nonce-replay/transaction-binding | `official-normative+design-proposal` | **dpop_nonce_jti_transaction_binding**<br>[NORMATIVE RFC FACT] RFC 9449 defines DPoP as an application-layer proof that sender-constrains an OAuth token to a publ... | [https://datatracker.ietf.org/doc/html/rfc9449](https://datatracker.ietf.org/doc/html/rfc9449) |
| 238 | `transaction-authorization/token-exchange/scoped-capability/opaque-result-passing` | transaction-authorization/token-exchange/scoped-capability/opaque-result-passing | `official-normative+design-proposal` | **host_token_exchange_scoped_capability_and_opaque_result**<br>[NORMATIVE RFC FACT] RFC 8693 defines an HTTP/JSON security-token service for requesting a new token using a subject_tok... | [https://www.rfc-editor.org/rfc/rfc8693.html](https://www.rfc-editor.org/rfc/rfc8693.html) |
| 239 | `audience-resource-binding/jwt-validation/mini-app-api-boundaries` | audience-resource-binding/jwt-validation/mini-app-api-boundaries | `official-normative+design-proposal` | **jwt_access_token_audience_expiry_and_jti_validation**<br>[NORMATIVE RFC FACT] RFC 9068 defines an interoperable JWT access-token profile and the claims/validation rules for reso... | [https://www.rfc-editor.org/rfc/rfc9068.html](https://www.rfc-editor.org/rfc/rfc9068.html) |
| 240 | `session-lifecycle/token-revocation/logout/uninstall/takedown` | session-lifecycle/token-revocation/logout/uninstall/takedown | `official-normative+design-proposal` | **session_token_revocation_on_logout_uninstall_and_takedown**<br>[NORMATIVE RFC FACT] RFC 7009 defines a revocation endpoint for refresh and access tokens. Revocation invalidates the su... | [https://www.rfc-editor.org/rfc/rfc7009.html](https://www.rfc-editor.org/rfc/rfc7009.html) |
| 241 | `host-bridge/postMessage/origin-recipient-binding` | host-bridge/postMessage/origin-recipient-binding | `official-normative` | **postmessage_origin_recipient_binding**<br>[SOURCE STATUS] The WHATWG HTML Standard is a Living Standard, not an IETF draft. [NORMATIVE FACT] For Window.postMessag... | [https://html.spec.whatwg.org/multipage/web-messaging.html#security-postmsg](https://html.spec.whatwg.org/multipage/web-messaging.html#security-postmsg) |
| 242 | `host-bridge/structured-clone/transfer-safety` | host-bridge/structured-clone/transfer-safety | `official-normative` | **structured_clone_transfer_profile**<br>[SOURCE STATUS] The WHATWG HTML Standard is a Living Standard, not an IETF draft. [NORMATIVE FACT] The safe-passing-of-s... | [https://html.spec.whatwg.org/multipage/structured-data.html#safe-passing-of-structured-data](https://html.spec.whatwg.org/multipage/structured-data.html#safe-passing-of-structured-data) |
| 243 | `host-bridge/problem-details/structured-errors` | host-bridge/problem-details/structured-errors | `official-proposed-standard` | **bridge_problem_details_error_profile**<br>[SOURCE STATUS] RFC 9457 is a published IETF RFC with status Proposed Standard (July 2023); it obsoletes RFC 7807 and wa... | [https://datatracker.ietf.org/doc/html/rfc9457](https://datatracker.ietf.org/doc/html/rfc9457) |
| 244 | `host-bridge/json-rpc/request-response-errors-cancellation` | host-bridge/json-rpc/request-response-errors-cancellation | `neutral-protocol-specification` | **jsonrpc_request_response_error_correlation**<br>[SOURCE STATUS] The official JSON-RPC 2.0 page records an origin date of 2010-03-26 and an update on 2013-01-04; it is a... | [https://www.jsonrpc.org/specification](https://www.jsonrpc.org/specification) |
| 245 | `android-webview/javascript-interface/native-method-exposure/origin-binding` | android-webview/javascript-interface/native-method-exposure/origin-binding | `primary-platform+store-design-proposal` | **javascript_interface_no_origin_and_minimal_facade**<br>[PLATFORM FACT — ANDROID-SPECIFIC] Android Developers documents that addJavascriptInterface injects the supplied Java ob... | [https://developer.android.com/privacy-and-security/risks/insecure-webview-native-bridges](https://developer.android.com/privacy-and-security/risks/insecure-webview-native-bridges) |
| 246 | `android-webview/safe-browsing/onSafeBrowsingHit/fail-closed` | android-webview/safe-browsing/onSafeBrowsingHit/fail-closed | `primary-platform+store-design-proposal` | **safe_browsing_enabled_and_fail_closed_hit_handling**<br>[PLATFORM FACT — ANDROID-SPECIFIC] WebSettings.setSafeBrowsingEnabled says Safe Browsing verifies links to protect again... | [https://developer.android.com/reference/android/webkit/WebSettings](https://developer.android.com/reference/android/webkit/WebSettings) |
| 247 | `android-webview/lifecycle/error/renderer-crash/ssl-error/bridge-teardown` | android-webview/lifecycle/error/renderer-crash/ssl-error/bridge-teardown | `primary-platform+store-design-proposal` | **webview_error_renderer_lifecycle_and_bridge_teardown**<br>[PLATFORM FACT — ANDROID-SPECIFIC] WebViewClient.onReceivedError(WebView, WebResourceRequest, WebResourceError) is calle... | [https://developer.android.com/reference/android/webkit/WebViewClient](https://developer.android.com/reference/android/webkit/WebViewClient) |
| 248 | `wkwebview-bridge/reply/error-contract` | wkwebview-bridge/reply/error-contract | `official-platform-documentation+design-proposal` | **wkwebview_reply_contract_and_error_boundary**<br>[APPLE PLATFORM BEHAVIOR] WKScriptMessageHandler receives a targeted JavaScript message through userContentController(_:... | [https://developer.apple.com/documentation/webkit/wkscriptmessagehandlerwithreply/usercontentcontroller(_:didreceive:replyhandler:)](https://developer.apple.com/documentation/webkit/wkscriptmessagehandlerwithreply/usercontentcontroller(_:didreceive:replyhandler:)) |
| 249 | `wkwebview-content-worlds/isolation-boundary` | wkwebview-content-worlds/isolation-boundary | `official-platform-documentation+design-proposal` | **wkwebview_content_world_namespace_boundary**<br>[APPLE PLATFORM BEHAVIOR] WKContentWorld defines a JavaScript execution scope/namespace. Apple describes separate copies... | [https://developer.apple.com/documentation/webkit/wkcontentworld](https://developer.apple.com/documentation/webkit/wkcontentworld) |
| 250 | `wkwebview-bridge/registration/message-validation/origin-metadata` | wkwebview-bridge/registration/message-validation/origin-metadata | `official-platform-documentation+design-proposal` | **wkwebview_handler_registration_and_message_provenance**<br>[APPLE PLATFORM BEHAVIOR] add(_:contentWorld:name:) requires a WKScriptMessageHandler, scopes it to the selected WKConte... | [https://developer.apple.com/documentation/webkit/wkusercontentcontroller/add(_:contentworld:name:)](https://developer.apple.com/documentation/webkit/wkusercontentcontroller/add(_:contentworld:name:)) |
| 251 | `wkwebview-bridge/handler-removal/lifecycle` | wkwebview-bridge/handler-removal/lifecycle | `official-platform-documentation+design-proposal` | **wkwebview_handler_unregistration_scope_and_lifecycle**<br>[APPLE PLATFORM BEHAVIOR] removeScriptMessageHandler(forName:contentWorld:) uninstalls a custom handler from the specifi... | [https://developer.apple.com/documentation/webkit/wkusercontentcontroller/removescriptmessagehandler(forname:contentworld:)](https://developer.apple.com/documentation/webkit/wkusercontentcontroller/removescriptmessagehandler(forname:contentworld:)) |
| 252 | `wkwebview-navigation/action-policy/restrictions` | wkwebview-navigation/action-policy/restrictions | `official-platform-documentation+design-proposal` | **wkwebview_navigation_action_policy_host_gate**<br>[APPLE PLATFORM BEHAVIOR] WKNavigationDelegate provides policy callbacks to allow or reject navigation changes; Apple sa... | [https://developer.apple.com/documentation/webkit/wknavigationdelegate/webview(_:decidepolicyfor:decisionhandler:)-2ni62](https://developer.apple.com/documentation/webkit/wknavigationdelegate/webview(_:decidepolicyfor:decisionhandler:)-2ni62) |
| 253 | `runtime/overload-control/fairness/priority/retry-stability` | runtime/overload-control/fairness/priority/retry-stability | `official-informational-rfc-requirements` | **graded_throttle_fairness_and_mixed_client_requirements**<br>[SOURCE STATUS] RFC 5390 is an Informational RFC, not a Standards Track protocol. [SOURCE FACT] REQ 4 says an overload m... | [https://www.rfc-editor.org/rfc/rfc5390.html](https://www.rfc-editor.org/rfc/rfc5390.html) |
| 254 | `runtime/overload-control/fair-share/rate-caps/stability` | runtime/overload-control/fair-share/rate-caps/stability | `official-informational-rfc-design` | **fair_share_rate_caps_and_overload_stability**<br>[SOURCE STATUS] RFC 6357 is an Informational RFC describing SIP overload-control design considerations, not a normative ... | [https://www.rfc-editor.org/rfc/rfc6357.html](https://www.rfc-editor.org/rfc/rfc6357.html) |
| 255 | `runtime/overload-control/negotiation/self-limiting/non-participating-clients` | runtime/overload-control/negotiation/self-limiting/non-participating-clients | `official-standards-track-rfc` | **negotiated_overload_state_self_limiting_and_legacy_fairness**<br>[SOURCE STATUS] RFC 7339 is an Internet Standards Track specification for SIP overload control; its described SIP entiti... | [https://www.rfc-editor.org/rfc/rfc7339.html](https://www.rfc-editor.org/rfc/rfc7339.html) |
| 256 | `runtime/overload-control/rate-governance/priority/burst-stability` | runtime/overload-control/rate-governance/priority/burst-stability | `official-standards-track-rfc` | **rate_based_upper_bound_priority_lanes_and_burst_stability**<br>[SOURCE STATUS] RFC 7415 is an Internet Standards Track document and defines an optional rate-based SIP overload scheme ... | [https://www.rfc-editor.org/rfc/rfc7415.html](https://www.rfc-editor.org/rfc/rfc7415.html) |
| 257 | `storage/origin-partitioning/accounting` | storage/origin-partitioning/accounting | `community-group-work-item` | **origin_partitioned_storage_accounting**<br>[SOURCE STATUS] This is a Privacy Community Group work item, not a final cross-browser normative specification. [SOURCE ... | [https://privacycg.github.io/storage-partitioning/](https://privacycg.github.io/storage-partitioning/) |
| 258 | `storage/quota/modes/eviction/user-control` | storage/quota/modes/eviction/user-control | `official-normative` | **storage_modes_quota_eviction_and_user_deletion**<br>[SOURCE STATUS] The WHATWG Storage Standard is a Living Standard. [SOURCE FACT] A storage shed maps storage keys to shel... | [https://storage.spec.whatwg.org/](https://storage.spec.whatwg.org/) |
| 259 | `storage/quota-exceeded/error-semantics` | storage/quota-exceeded/error-semantics | `official-normative` | **web_storage_quota_exceeded_write_contract**<br>[SOURCE STATUS] This is the current WHATWG HTML Living Standard Web Storage section. [SOURCE FACT] The `Storage` object ... | [https://html.spec.whatwg.org/multipage/webstorage.html](https://html.spec.whatwg.org/multipage/webstorage.html) |
| 260 | `storage/quota-errors/consistency/rollback` | storage/quota-errors/consistency/rollback | `w3c-editor-draft` | **indexeddb_quota_error_atomic_rollback**<br>[SOURCE STATUS] Indexed Database API 3.0 is a W3C Editor's Draft, intended to supersede the Indexed Database API 2.0 Rec... | [https://w3c.github.io/IndexedDB/](https://w3c.github.io/IndexedDB/) |
| 261 | `storage/buckets/eviction/durability/visibility` | storage/buckets/eviction/durability/visibility | `official-browser-guidance` | **bucket_eviction_priority_and_durability_disclosure**<br>[SOURCE STATUS] This is official Chrome web-platform guidance/blog material, not a cross-browser normative standard; the... | [https://developer.chrome.com/docs/web-platform/storage-buckets](https://developer.chrome.com/docs/web-platform/storage-buckets) |
| 262 | `business-flow-abuse/operation-risk/adaptive-friction/audit` | business-flow-abuse/operation-risk/adaptive-friction/audit | `official-government-guidance+mini-app-design-proposal` | **operation_risk_classification_adaptive_friction**<br>[SOURCE STATUS: official NIST government guidance; zero-trust architecture guidance, not a mini-app abuse standard] [SOU... | [https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-207.pdf](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-207.pdf) |
| 263 | `business-flow-abuse/rate-limit-partitioning/shared-device-NAT/fairness` | business-flow-abuse/rate-limit-partitioning/shared-device-NAT/fairness | `official-platform-guidance+mini-app-design-proposal` | **partitioned_rate_limit_avoids_ip_only_false_positives**<br>[SOURCE STATUS: AWS WAF platform guidance; implementation-specific, not a portable mini-app standard] [SOURCE FACT] AWS ... | [https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-type-rate-based-aggregation-options.html](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-type-rate-based-aggregation-options.html) |
| 264 | `business-flow-abuse/adaptive-friction/accessibility/false-positives/appeals` | business-flow-abuse/adaptive-friction/accessibility/false-positives/appeals | `official-w3c-guidance+mini-app-design-proposal` | **accessible_step_up_and_friction_fallback**<br>[SOURCE STATUS: official W3C WCAG 2.2 explanatory guidance for the Level AA criterion; the criterion addresses authentic... | [https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication-minimum](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication-minimum) |
| 265 | `business-flow-abuse/cost-bearing-operations/idempotency/provider-protection/replay` | business-flow-abuse/cost-bearing-operations/idempotency/provider-protection/replay | `official-provider-guidance+mini-app-design-proposal` | **provider_idempotency_and_replay_safe_cost_operation**<br>[SOURCE STATUS: Stripe API reference; provider-specific behavior, not universal payment law] [SOURCE FACT] Stripe says i... | [https://docs.stripe.com/api/idempotent_requests](https://docs.stripe.com/api/idempotent_requests) |
| 266 | `business-flow-abuse/hard-spend-ceilings/provider-protection/audit-appeal` | business-flow-abuse/hard-spend-ceilings/provider-protection/audit-appeal | `official-provider-guidance+mini-app-design-proposal` | **hard_spend_ceiling_separates_alerts_from_enforcement**<br>[SOURCE STATUS: official Google Cloud provider documentation; spend-cap behavior is service/provider-specific and the pa... | [https://docs.cloud.google.com/billing/docs/how-to/budgets](https://docs.cloud.google.com/billing/docs/how-to/budgets) |
| 267 | `w3c-miniapp-manifest-001` | manifest | `normative_standard` | **MiniApp Manifest defines the store and runtime contract for identity, routes, permissions, icons, and window/theme configuration**<br>The manifest is a JSON document that extends/profiles the Web App Manifest and Web App Manifest - Application Informatio... | [https://w3c.github.io/miniapp-manifest/](https://w3c.github.io/miniapp-manifest/) |
| 268 | `w3c-miniapp-packaging-001` | packaging | `normative_standard` | **MiniApp Packaging standardizes the ZIP artifact, reserved file layout, integrity hooks, and runtime processing while leaving split-package signing open**<br>A conformant package MUST use a root directory plus pages directory and MUST contain root manifest.json, app.js, app.css... | [https://w3c.github.io/miniapp-packaging/](https://w3c.github.io/miniapp-packaging/) |
| 269 | `w3c-miniapp-addressing-001` | addressing_navigation | `normative_standard` | **MiniApp Addressing defines deep-link syntax and host/provider resolution from OS dispatch to an in-package resource**<br>The ABNF is miniappuri = uri-prefix uri-infix "/" identify path-abempty ["?" query] ["#" fragment], with uri-prefix eith... | [https://w3c.github.io/miniapp-addressing/](https://w3c.github.io/miniapp-addressing/) |
| 270 | `w3c-miniapp-lifecycle-001` | lifecycle | `normative_standard` | **MiniApp Lifecycle supplies global and page state machines with host callbacks for foregrounding, suspension, errors, rendering readiness, and unload**<br>The global application states are launched, shown, hidden, error, and unloaded, exposed through GlobalState and onglobal... | [https://w3c.github.io/miniapp-lifecycle/](https://w3c.github.io/miniapp-lifecycle/) |
| 271 | `w3c-miniapp-architecture-001` | architecture | `industry_standard` | **MiniApp White Paper positions a super app as a host platform using hybrid rendering, a JavaScript-worker logic layer, and controlled native bridges**<br>The White Paper defines a super app as a software platform that hosts and supports other applications, and contrasts Min... | [https://w3c.github.io/miniapp-white-paper/](https://w3c.github.io/miniapp-white-paper/) |
| 272 | `tuf_roles_threshold_and_delegation` | TUF multi-role trust model for mini-app package metadata | `normative_standard` | **Separate Root, Targets, Snapshot, and Timestamp trust with threshold signatures**<br>[NORMATIVE] TUF defines four fundamental top-level roles: Root, Targets, Snapshot, and Timestamp. Root delegates trust t... | [https://theupdateframework.io/specification/latest/](https://theupdateframework.io/specification/latest/) |
| 273 | `slsa_submission_provenance` | SLSA v1.0 mini-app developer submission provenance | `normative_standard` | **Require digest-bound provenance for every submitted mini-app artifact**<br>[NORMATIVE] SLSA v1.0 requires a producer to choose a build platform capable of the desired level, follow a consistent b... | [https://slsa.dev/spec/v1.0/requirements](https://slsa.dev/spec/v1.0/requirements) |
| 274 | `slsa_hardened_store_builds` | SLSA v1.0 hosted and isolated store builds | `normative_standard` | **Use hosted, isolated, control-plane-generated provenance for multi-tenant store builds**<br>[NORMATIVE] SLSA Build L2 means builds run on a hosted platform that generates and signs provenance, with downstream val... | [https://slsa.dev/spec/v1.0/levels](https://slsa.dev/spec/v1.0/levels) |
| 275 | `sigstore_keyless_transparency_attestations` | Sigstore keyless signing Fulcio OIDC and Rekor transparency | `industry_standard` | **Bind publisher identity and build attestations to package digests with Sigstore**<br>[INDUSTRY STANDARD] Sigstore documents keyless signing as associating an identity rather than a long-lived key with an a... | [https://docs.sigstore.dev/cosign/signing/overview/](https://docs.sigstore.dev/cosign/signing/overview/) |
| 276 | `device-capabilities-permissions-policy-001` | permissions-policy/capability-delegation/permission-state | `normative_standard` | **Use Permissions Policy as the host capability envelope before user consent**<br>[NORMATIVE FACT] Permissions Policy controls policy-controlled features independently of end-user permission: the HTTP P... | [https://w3c.github.io/webappsec-permissions-policy/](https://w3c.github.io/webappsec-permissions-policy/) |
| 277 | `device-capabilities-geolocation-002` | geolocation/foreground-consent/high-accuracy/privacy | `normative_standard` | **Make location access visible, foreground-bound, and precision-limited**<br>[NORMATIVE FACT] Geolocation is a secure-context powerful feature named geolocation. Its default allowlist is self, so c... | [https://w3c.github.io/geolocation/](https://w3c.github.io/geolocation/) |
| 278 | `device-capabilities-media-capture-003` | media-capture/camera/microphone/indicators/revocation | `normative_standard` | **Gate camera and microphone per mini-app and make capture revocable**<br>[NORMATIVE FACT] Media Capture defines camera and microphone as powerful features and policy-controlled features with de... | [https://w3c.github.io/mediacapture-main/](https://w3c.github.io/mediacapture-main/) |
| 279 | `device-capabilities-bluetooth-004` | web-bluetooth/ble/gatt/service-filtering/permission | `industry_standard` | **Constrain Web Bluetooth to user-selected devices and declared GATT services**<br>[PLATFORM/WEB API FACT] Web Bluetooth is limited to secure contexts and is controlled by the bluetooth Permissions Polic... | [https://developer.mozilla.org/en-US/docs/Web/API/Web_Bluetooth_API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Bluetooth_API) |
| 280 | `device-capabilities-nfc-005` | web-nfc/ndef/read-write/visibility/host-boundary | `industry_standard` | **Keep Web NFC visible, NDEF-only, and read-before-write**<br>[SPEC/DRAFT FACT] The Web NFC draft limits the API to NDEF use cases; low-level I/O such as ISO-DEP/NFC-A/B/NFC-F and ho... | [https://w3c-cg.github.io/web-nfc/](https://w3c-cg.github.io/web-nfc/) |
| 281 | `device-capabilities-generic-sensors-006` | generic-sensor/accelerometer/gyroscope/magnetometer/frequency/privacy | `normative_standard` | **Throttle and foreground-bind motion sensors; do not promise a universal sensor rate**<br>[NORMATIVE FACT] Generic Sensor requires secure context, Permissions Policy permission for the sensor type, the Permissi... | [https://w3c.github.io/sensors/](https://w3c.github.io/sensors/) |
| 282 | `device-capabilities-wake-lock-007` | screen-wake-lock/visibility/release/battery | `normative_standard` | **Treat screen wake lock as a visible, revocable, budgeted capability**<br>[NORMATIVE FACT] Screen Wake Lock is secure-context-only and a policy-controlled feature named screen-wake-lock with def... | [https://w3c.github.io/screen-wake-lock/](https://w3c.github.io/screen-wake-lock/) |
| 283 | `storage-sandbox-origin-binding` | filesystem/origin-partitioning | `normative_standard` | **Bind the private file-system root and handles to the mini-app origin**<br>[SPEC FACT] The standard exposes navigator.storage.getDirectory() as the root of a bucket file system; its algorithm obt... | [https://fs.spec.whatwg.org/](https://fs.spec.whatwg.org/) |
| 284 | `storage-sandbox-opfs-performance-quota` | opfs/private-storage/quota/lifecycle | `official_platform_practice` | **Use OPFS-like storage for fast private app data without weakening isolation**<br>[SPEC FACT] MDN describes OPFS as private to the page origin, not visible like the regular file system, optimized for pe... | [https://developer.mozilla.org/en-US/docs/Web/API/File_System_API/Origin_private_file_system](https://developer.mozilla.org/en-US/docs/Web/API/File_System_API/Origin_private_file_system) |
| 285 | `storage-sandbox-wechat-appid-isolation` | wechat/filesystemmanager/appid-isolation | `official_platform_practice` | **Partition local files by user and mini-app appId**<br>[SPEC FACT] Weixin documents a globally unique wx.getFileSystemManager() for Mini Program filesystem operations and says... | [https://developers.weixin.qq.com/miniprogram/en/dev/framework/ability/file-system.html](https://developers.weixin.qq.com/miniprogram/en/dev/framework/ability/file-system.html) |
| 286 | `storage-sandbox-wechat-tiered-lifecycle` | wechat/filesystemmanager/tiered-storage/lifecycle/quota | `official_platform_practice` | **Separate immutable package files, ephemeral temp files, and persistent user files**<br>[SPEC FACT] Weixin classifies code-package files and local files separately: package files cannot be dynamically modifie... | [https://developers.weixin.qq.com/miniprogram/en/dev/framework/ability/file-system.html](https://developers.weixin.qq.com/miniprogram/en/dev/framework/ability/file-system.html) |
| 287 | `storage-sandbox-masvs-sensitive-at-rest` | owasp/masvs-storage-1/sensitive-data-at-rest/cross-tenant | `industry_standard` | **Protect sensitive data at rest independently of storage location and tenant**<br>[SPEC FACT] OWASP MASVS-STORAGE-1 states that an app securely stores sensitive data and explains that data may come from... | [https://mas.owasp.org/MASVS/controls/MASVS-STORAGE-1/](https://mas.owasp.org/MASVS/controls/MASVS-STORAGE-1/) |
| 288 | `network-egress-csp-connect-001` | csp/connect-src/data-exfiltration | `normative_standard` | **Enforce a default-deny CSP and explicit connect-src egress allowlist for every mini-app**<br>[SPEC FACT] CSP Level 3 says connect-src restricts URLs loaded by script interfaces, including fetch, XMLHttpRequest, Ev... | [https://w3c.github.io/webappsec-csp/](https://w3c.github.io/webappsec-csp/) |
| 289 | `network-egress-csp-script-object-002` | csp/script-src/object-src/runtime-execution | `normative_standard` | **Restrict executable sources and disable plugin content to suppress rogue bundle code**<br>[SPEC FACT] CSP Level 3 defines script-src as restricting locations from which scripts may be executed, including inline... | [https://w3c.github.io/webappsec-csp/](https://w3c.github.io/webappsec-csp/) |
| 290 | `network-egress-csp-frame-ancestors-003` | csp/frame-ancestors/embedding | `normative_standard` | **Prevent hostile embedding of mini-app documents with frame-ancestors**<br>[SPEC FACT] CSP Level 3 says frame-ancestors restricts URLs that can embed a resource through frame, iframe, object, or ... | [https://w3c.github.io/webappsec-csp/](https://w3c.github.io/webappsec-csp/) |
| 291 | `network-egress-wechat-allowlist-004` | wechat/domain-allowlist/tls-hostname | `official_platform_practice` | **Require pre-registered, exact-domain HTTPS and WSS endpoints for mini-app traffic**<br>[SPEC FACT] WeChat's official network guide says a mini program must configure a communication domain in advance and can... | [https://developers.weixin.qq.com/miniprogram/en/dev/framework/ability/network.html](https://developers.weixin.qq.com/miniprogram/en/dev/framework/ability/network.html) |
| 292 | `network-egress-owasp-host-mediation-005` | owasp/masvs-network-1/m5/pinning/egress-mediation | `industry_standard` | **Make host-side secure transport and egress mediation mandatory for WebView and third-party bundles**<br>[SPEC FACT] MASVS-NETWORK-1 requires all network traffic to follow current best practices and explains that privacy and ... | [https://mas.owasp.org/MASVS/controls/MASVS-NETWORK-1/](https://mas.owasp.org/MASVS/controls/MASVS-NETWORK-1/) |
| 293 | `os-integration-web-share-001` | web-share/user-activation/target-selection/payload-validation | `normative_standard` | **Make outbound sharing an explicit, host-mediated user action**<br>[SPEC FACT] navigator.share() requires a fully active document, an allowed web-share policy, and transient activation; i... | [https://w3c.github.io/web-share/](https://w3c.github.io/web-share/) |
| 294 | `os-integration-web-share-target-002` | web-share-target/manifest/secure-registration/incoming-data | `normative_standard` | **Require a trustworthy, in-scope manifest target and bounded incoming-share handler**<br>[SPEC FACT] A share_target manifest member declares the action URL and parameter names; manifest processing rejects miss... | [https://w3c.github.io/web-share-target/](https://w3c.github.io/web-share-target/) |
| 295 | `os-integration-web-share-target-003` | web-share-target/files/payload-scope/secure-ingestion | `normative_standard` | **Treat inbound files as an explicit extension, not an implicit Web Share Target guarantee**<br>[SPEC FACT] The Web Share Target draft's defined ShareTargetParams and launch algorithm enumerate only title, text, and ... | [https://w3c.github.io/web-share-target/](https://w3c.github.io/web-share-target/) |
| 296 | `os-integration-badging-004` | badging/lifecycle/os-state/aggregation/rate-limits | `normative_standard` | **Keep badges host-aggregated, actionable, permission-aware, and rate-limited**<br>[SPEC FACT] An installed web application's badge starts as nothing; the OS stores and manages the value while the user a... | [https://w3c.github.io/badging/](https://w3c.github.io/badging/) |
| 297 | `os-integration-contact-picker-005` | contact-picker/user-activation/one-off-selection/privacy | `normative_standard` | **Keep address-book access picker-mediated, top-level, secure, and one-off**<br>[SPEC FACT] Contact Picker provides one-off access to contact information with user control over shared data. select() r... | [https://www.w3.org/TR/contact-picker/](https://www.w3.org/TR/contact-picker/) |
| 298 | `crypto-hw-webcrypto-001` | cryptography/webcrypto/key-isolation | `normative_standard` | **Make WebCrypto keys non-extractable and capability-scoped, without treating WebCrypto as hardware-backed storage**<br>[SPEC FACT] CryptoKey is an opaque user-agent-managed reference with an extractable flag and an explicit usages set. The... | [https://www.w3.org/TR/WebCryptoAPI/](https://www.w3.org/TR/WebCryptoAPI/) |
| 299 | `crypto-hw-android-strongbox-002` | cryptography/android-keystore/strongbox/keymint | `platform_specification` | **Prefer Android StrongBox for high-value mini-app keys and attest the resulting security level**<br>[SPEC FACT] Android documents that Keystore key material is non-exportable, does not enter the application process, and ... | [https://developer.android.com/privacy-and-security/cryptography](https://developer.android.com/privacy-and-security/cryptography) |
| 300 | `crypto-hw-apple-secureenclave-003` | cryptography/apple-secure-enclave/keychain/access-control | `platform_specification` | **Broker Secure Enclave P-256 operations through Keychain access controls**<br>[SPEC FACT] Apple describes the Secure Enclave as a hardware-based key manager isolated from the main processor: the app... | [https://developer.apple.com/documentation/security/certificate_key_and_trust_services/keys/storing_keys_in_the_secure_enclave](https://developer.apple.com/documentation/security/certificate_key_and_trust_services/keys/storing_keys_in_the_secure_enclave) |
| 301 | `crypto-hw-masvs-crypto-1-004` | cryptography/algorithm-selection/key-generation | `security_standard` | **Enforce current strong cryptography and platform-backed generation at mini-app review and runtime**<br>[SPEC FACT] MASVS-CRYPTO-1 requires the app to employ current strong cryptography according to industry best practices a... | [https://mas.owasp.org/MASVS/controls/MASVS-CRYPTO-1/](https://mas.owasp.org/MASVS/controls/MASVS-CRYPTO-1/) |
| 302 | `crypto-hw-masvs-crypto-2-005` | cryptography/key-lifecycle/keystore/hardware-backed | `security_standard` | **Make key lifecycle, keystore use, access gating, rotation, and invalidation mini-app conformance requirements**<br>[SPEC FACT] MASVS-CRYPTO-2 covers cryptographic-key management across generation, storage, and protection. Its related w... | [https://mas.owasp.org/MASVS/controls/MASVS-CRYPTO-2/](https://mas.owasp.org/MASVS/controls/MASVS-CRYPTO-2/) |
| 303 | `payment-request-core-001` | Payment Request API mediation lifecycle | `normative_standard` | **Treat PaymentRequest as user-agent mediation, not authorization**<br>[SPEC FACT] Payment Request API places the user agent between the payee, payer, and payment method. show() starts user i... | [https://www.w3.org/TR/payment-request/](https://www.w3.org/TR/payment-request/) |
| 304 | `payment-request-iframe-002` | Payment Request API cross-origin mediation | `normative_standard` | **Gate cross-origin mini-app payment frames with the payment Permissions Policy**<br>[SPEC FACT] Payment Request defines payment as a policy-controlled feature whose default allowlist is self. The specific... | [https://w3c.github.io/payment-request/](https://w3c.github.io/payment-request/) |
| 305 | `payment-spc-auth-003` | Secure Payment Confirmation authenticated checkout | `normative_standard` | **Use SPC to bind user verification to the order and payee**<br>[SPEC FACT] Secure Payment Confirmation defines the standardized payment method identifier secure-payment-confirmation. ... | [https://www.w3.org/TR/secure-payment-confirmation/](https://www.w3.org/TR/secure-payment-confirmation/) |
| 306 | `payment-method-manifest-004` | Payment Method Manifest discovery and origin control | `normative_standard` | **Use Payment Method Manifests as an explicit allowlist for wallet/payment apps**<br>[SPEC FACT] A URL-based payment method identifier points to its machine-readable manifest through an HTTP Link header wi... | [https://www.w3.org/TR/payment-method-manifest/](https://www.w3.org/TR/payment-method-manifest/) |
| 307 | `telemetry-paint-timing-001` | runtime_paint_timing | `normative_standard` | **Paint Timing gives the host FP/FCP launch milestones with explicit missing-data semantics**<br>[SPEC FACT] W3C Paint Timing defines PerformancePaintTiming entries with entryType "paint", startTime, duration 0, and t... | [https://www.w3.org/TR/paint-timing/](https://www.w3.org/TR/paint-timing/) |
| 308 | `telemetry-lcp-metric-002` | runtime_lcp_candidate_tracking | `normative_standard` | **LCP candidate tracking exposes visual readiness but must be scoped to MiniApp route navigations**<br>[SPEC FACT] The Largest Contentful Paint algorithm tracks content seen so far and creates a new entry whenever a larger ... | [https://www.w3.org/TR/largest-contentful-paint/](https://www.w3.org/TR/largest-contentful-paint/) |
| 309 | `telemetry-event-timing-inp-003` | runtime_event_timing_inp | `normative_standard` | **Event Timing supplies interaction-latency primitives for host-defined INP governance**<br>[SPEC FACT] Event Timing exposes PerformanceEventTiming for selected trusted input events and excludes continuous events... | [https://www.w3.org/TR/event-timing/](https://www.w3.org/TR/event-timing/) |
| 310 | `telemetry-reporting-api-004` | runtime_out_of_band_reporting | `normative_standard` | **Reporting API enables secure out-of-band diagnostic collection but cannot be a reliable control channel**<br>[SPEC FACT] The Reporting API defines named endpoints through the Reporting-Endpoints response header; endpoint values m... | [https://www.w3.org/TR/reporting-1/](https://www.w3.org/TR/reporting-1/) |
| 311 | `telemetry-network-error-logging-005` | runtime_network_error_logging | `normative_standard` | **NEL supplies client-observed DNS/TLS/HTTP failure telemetry with sampling and privacy boundaries**<br>[SPEC FACT] Network Error Logging opts an origin into client-side network telemetry through the HTTP NEL response header... | [https://www.w3.org/TR/network-error-logging/](https://www.w3.org/TR/network-error-logging/) |
| 312 | `concurrency-locks-messaging-001` | cross-context-isolation | `normative_standard` | **Web Locks scope follows the storage bucket and must never cross origins**<br>[SPEC FACT] The W3C Web Locks draft defines resource names as global across agents sharing a storage bucket, and gives e... | [https://www.w3.org/TR/web-locks/](https://www.w3.org/TR/web-locks/) |
| 313 | `concurrency-locks-messaging-002` | concurrency-state-synchronization | `normative_standard` | **Web Locks provide exclusive/shared arbitration with callback-bounded ownership**<br>[SPEC FACT] A lock mode is either exclusive or shared. An exclusive lock prevents any other lock with the same name; sha... | [https://www.w3.org/TR/web-locks/](https://www.w3.org/TR/web-locks/) |
| 314 | `concurrency-locks-messaging-003` | lock-failure-recovery | `normative_standard` | **Web Locks define explicit non-waiting, cancellation, recovery, and diagnostic semantics**<br>[SPEC FACT] With ifAvailable true, a request does not wait for an unavailable lock and invokes the callback with null. A... | [https://www.w3.org/TR/web-locks/](https://www.w3.org/TR/web-locks/) |
| 315 | `concurrency-locks-messaging-004` | cross-context-message-isolation | `normative_standard` | **Window postMessage delivery is origin-targeted and requires origin and schema validation**<br>[SPEC FACT] HTML's window postMessage algorithm structured-serializes the message and queues delivery only when the targ... | [https://html.spec.whatwg.org/multipage/web-messaging.html](https://html.spec.whatwg.org/multipage/web-messaging.html) |
| 316 | `concurrency-locks-messaging-005` | channel-capability-isolation | `normative_standard` | **MessageChannel and MessagePort create transferable channels with explicit start and close lifecycle**<br>[SPEC FACT] MessageChannel constructs two MessagePorts and entangles them as a two-way channel; data posted through one ... | [https://html.spec.whatwg.org/multipage/web-messaging.html](https://html.spec.whatwg.org/multipage/web-messaging.html) |
| 317 | `background-sync-one-shot-001` | background-sync/one-shot-registration | `normative_standard` | **Gate one-shot Background Sync registrations per MiniApp and keep tags host-namespaced**<br>[SPEC FACT] The API is service-worker based and available only in a secure context. Each service-worker registration has... | [https://wicg.github.io/background-sync/spec/](https://wicg.github.io/background-sync/spec/) |
| 318 | `background-sync-retry-lifecycle-002` | background-sync/retry-online-reconnection/lifecycle | `normative_standard` | **Model one-shot Sync as eventual, retryable, online-gated delivery**<br>[SPEC FACT] If the registering page or worker is running, the user agent should fire the sync event as soon as network c... | [https://wicg.github.io/background-sync/spec/](https://wicg.github.io/background-sync/spec/) |
| 319 | `periodic-background-sync-scheduler-003` | periodic-background-sync/permission-min-interval/engagement | `normative_standard` | **Treat Periodic Background Sync as permissioned best-effort refresh with a lower-bound interval**<br>[SPEC FACT] register(tag, options) MUST reject without an active worker, without PermissionState granted for periodic-ba... | [https://wicg.github.io/periodic-background-sync/](https://wicg.github.io/periodic-background-sync/) |
| 320 | `periodic-background-sync-resource-budget-004` | periodic-background-sync/network-power/data-budget | `normative_standard` | **Budget periodic wakeups for battery, network state, and data consumption**<br>[SPEC FACT] The scheduler waits for the cross-origin minimum interval, may add a user-agent-defined delay to group regis... | [https://wicg.github.io/periodic-background-sync/](https://wicg.github.io/periodic-background-sync/) |
| 321 | `background-fetch-large-transfer-005` | background-fetch/large-upload-download/resume/progress | `normative_standard` | **Use Background Fetch for user-visible large transfers, with stricter upload replay controls**<br>[SPEC FACT] Background Fetch is designed to continue a multi-request large upload or download even after all windows and... | [https://wicg.github.io/background-fetch/](https://wicg.github.io/background-fetch/) |
| 322 | `media-session-platform-controls-001` | media-session/metadata/playback-state/action-handlers | `normative_standard` | **Expose one active media session with metadata, state, and lockscreen control mapping**<br>[SPEC FACT] The W3C Media Session Working Draft lets a page publish media metadata to platform UI and receive media-key/... | [https://www.w3.org/TR/mediasession/](https://www.w3.org/TR/mediasession/) |
| 323 | `wechat-background-audio-implementation-002` | wechat/background-audio/requiredBackgroundModes/lifecycle | `platform_practice` | **Require a reviewed background-audio entitlement and make playback lifecycle explicit**<br>[SPEC FACT] WeChat documents wx.getBackgroundAudioManager() as returning a globally unique background-audio manager. If ... | [https://developers.weixin.qq.com/miniprogram/dev/api/media/background-audio/wx.getBackgroundAudioManager.html](https://developers.weixin.qq.com/miniprogram/dev/api/media/background-audio/wx.getBackgroundAudioManager.html) |
| 324 | `screen-wake-lock-visibility-power-003` | screen-wake-lock/sentinel/visibility/power-saving | `normative_standard` | **Treat screen wake lock as a visible, revocable, advisory lease**<br>[SPEC FACT] The W3C Screen Wake Lock draft defines a SecureContext screen wake lock that only visible documents can acqu... | [https://www.w3.org/TR/screen-wake-lock/](https://www.w3.org/TR/screen-wake-lock/) |
| 325 | `wechat-screen-battery-power-governor-004` | wechat/setKeepScreenOn/getBatteryInfo/low-power | `platform_practice` | **Broker WeChat screen-on and battery signals through a host power governor**<br>[SPEC FACT] WeChat wx.setKeepScreenOn({keepScreenOn}) controls whether the screen stays on and is scoped only to the cur... | [https://developers.weixin.qq.com/miniprogram/dev/api/device/screen/wx.setKeepScreenOn.html](https://developers.weixin.qq.com/miniprogram/dev/api/device/screen/wx.setKeepScreenOn.html) |
| 326 | `battery-status-privacy-governance-005` | battery-status/privacy/fingerprinting/power-governance | `normative_standard` | **Expose coarse battery governance signals; do not export a raw fingerprinting surface**<br>[SPEC FACT] The W3C Battery Status draft makes getBattery() SecureContext-only and rejects it when the battery policy-co... | [https://www.w3.org/TR/battery-status/](https://www.w3.org/TR/battery-status/) |
| 327 | `apple_4_7_host_responsibility_and_moderation` | mini_apps_host_responsibility_and_safety | `platform_practice` | **The host owns compliance and safety controls for non-embedded mini-app software**<br>[SPEC FACT] Apple permits HTML5/JavaScript mini apps and mini games, streaming games, chatbots, plug-ins, and downloadab... | [https://developer.apple.com/app-store/review/guidelines/](https://developer.apple.com/app-store/review/guidelines/) |
| 328 | `apple_4_7_api_and_permission_isolation` | mini_app_api_and_permission_isolation | `platform_practice` | **Native API exposure and host permission reuse are prohibited without Apple permission or per-instance consent**<br>[SPEC FACT] Section 4.7.2 says the app may not extend or expose native platform APIs or technologies to offered software... | [https://developer.apple.com/app-store/review/guidelines/](https://developer.apple.com/app-store/review/guidelines/) |
| 329 | `apple_4_7_catalog_and_universal_links` | mini_app_catalog_discovery | `platform_practice` | **Every offered mini-app must be indexed with metadata and a working universal link**<br>[SPEC FACT] Section 4.7.4 requires an index of the software and metadata available in the app, and requires universal li... | [https://developer.apple.com/app-store/review/guidelines/](https://developer.apple.com/app-store/review/guidelines/) |
| 330 | `apple_4_7_5_age_gating_and_kids_gate` | mini_app_age_restriction_and_kids_safety | `platform_practice` | **The host must surface over-rating mini-apps and enforce age restrictions, with parental gates for Kids Category flows**<br>[SPEC FACT] Section 4.7.5 requires a way for users to identify software that exceeds the host app's age rating and an ag... | [https://developer.apple.com/app-store/review/guidelines/](https://developer.apple.com/app-store/review/guidelines/) |
| 331 | `apple_1_3_5_1_4_kids_privacy` | kids_privacy_and_advertising | `platform_practice` | **Child-directed or minor-data mini-apps need statutory privacy controls and a default ban on third-party analytics and advertising**<br>[SPEC FACT] Section 5.1.4 directs developers to review COPPA, GDPR, and other applicable children’s-privacy laws; permit... | [https://developer.apple.com/app-store/review/guidelines/](https://developer.apple.com/app-store/review/guidelines/) |
| 332 | `uk-ico-childrens-code-001` | children-code/governance-age-application-transparency | `regulatory_framework` | **Make child-first DPIA, age application, and transparency host release gates**<br>[SPEC FACT] DPA 2018 s123 requires the Commissioner to prepare guidance on age-appropriate design for relevant informati... | [https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/childrens-information/childrens-code-guidance-and-resources/age-appropriate-design-a-code-of-practice-for-online-services/code-standards/](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/childrens-information/childrens-code-guidance-and-resources/age-appropriate-design-a-code-of-practice-for-online-services/code-standards/) |
| 333 | `uk-ico-childrens-code-002` | children-code/privacy-defaults-minimisation-sharing | `regulatory_framework` | **Ship high-privacy, minimal-data, no-sharing mini-app defaults**<br>[SPEC FACT] Standards 5–9 prohibit using children’s data in ways shown to harm wellbeing; require published privacy, age... | [https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/childrens-information/childrens-code-guidance-and-resources/age-appropriate-design-a-code-of-practice-for-online-services/code-standards/](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/childrens-information/childrens-code-guidance-and-resources/age-appropriate-design-a-code-of-practice-for-online-services/code-standards/) |
| 334 | `uk-ico-childrens-code-003` | children-code/geolocation-parental-controls | `regulatory_framework` | **Make location session-bound and parental monitoring visible**<br>[SPEC FACT] Standard 10 requires geolocation off by default unless a compelling reason, an obvious sign while location t... | [https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/childrens-information/childrens-code-guidance-and-resources/age-appropriate-design-a-code-of-practice-for-online-services/code-standards/](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/childrens-information/childrens-code-guidance-and-resources/age-appropriate-design-a-code-of-practice-for-online-services/code-standards/) |
| 335 | `uk-ico-childrens-code-004` | children-code/profiling-nudges | `regulatory_framework` | **Disable child profiling for ads and block privacy-weakening nudges**<br>[SPEC FACT] Standard 12 requires profiling options off by default unless a compelling reason and appropriate safeguards ... | [https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/childrens-information/childrens-code-guidance-and-resources/age-appropriate-design-a-code-of-practice-for-online-services/code-standards/](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/childrens-information/childrens-code-guidance-and-resources/age-appropriate-design-a-code-of-practice-for-online-services/code-standards/) |
| 336 | `uk-ico-childrens-code-005` | children-code/connected-devices-online-tools | `regulatory_framework` | **Add device kill switches and prominent child rights/reporting tools**<br>[SPEC FACT] Standard 14 requires effective tools for connected toys or devices to conform to the code. ICO guidance cove... | [https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/childrens-information/childrens-code-guidance-and-resources/age-appropriate-design-a-code-of-practice-for-online-services/code-standards/](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/childrens-information/childrens-code-guidance-and-resources/age-appropriate-design-a-code-of-practice-for-online-services/code-standards/) |
| 337 | `iarc-global-architecture-001` | age_rating_federation_architecture | `consortium_standard` | **Single questionnaire, regional algorithmic ratings, and local storefront display**<br>[SPEC FACT] IARC describes a storefront-integrated SaaS solution: a developer completes one questionnaire through a part... | [https://globalratings.com/about/](https://globalratings.com/about/) |
| 338 | `iarc-global-authorities-002` | regional_rating_authorities | `consortium_standard` | **Canonical participating-authority map covers the requested regional systems**<br>[SPEC FACT] IARC's participant page lists Australian Classification Board (Australia), Classificação Indicativa (Brazil)... | [https://www.globalratings.com/participants/](https://www.globalratings.com/participants/) |
| 339 | `iarc-global-interactive-003` | content_descriptors_and_interactive_elements | `consortium_standard` | **Interactive disclosures cover purchases, user interaction, location sharing, and Internet access**<br>[SPEC FACT] IARC says the questionnaire result includes Content Descriptors and Interactive Elements. The official descr... | [https://globalratings.com/how-iarc-works/](https://globalratings.com/how-iarc-works/) |
| 340 | `iarc-global-operations-004` | post_release_rating_operations | `consortium_standard` | **Authority monitoring, API corrections, and rating-check disputes are part of the digital workflow**<br>[SPEC FACT] IARC explains that its automated process issues ratings instantly without authority review before release, s... | [https://globalratings.com/faq/](https://globalratings.com/faq/) |
| 341 | `iarc-global-portability-005` | certificate_id_portability_and_storefront_integration | `consortium_standard` | **Certificate ID enables reuse across participating storefronts, including named major stores**<br>[SPEC FACT] IARC says a developer completes the questionnaire once and can use the IARC rating certificate ID on other I... | [https://globalratings.com/how-iarc-works/](https://globalratings.com/how-iarc-works/) |
| 342 | `iter27-trusted-types-csp-sink-enforcement` | trusted-types-dom-xss | `normative_standard` | **Enforce Trusted Types at every mini-app DOM XSS sink**<br>[SPEC FACT] The specification groups the DOM XSS sinks under the 'script' sink group, including script URL/text setters,... | [https://www.w3.org/TR/trusted-types/](https://www.w3.org/TR/trusted-types/) |
| 343 | `iter27-trusted-types-policy-allowlist` | trusted-types-policy-governance | `normative_standard` | **Allowlist and review Trusted Type policy factories**<br>[SPEC FACT] TrustedHTML, TrustedScript, and TrustedScriptURL objects are created through application-defined policies; t... | [https://www.w3.org/TR/trusted-types/](https://www.w3.org/TR/trusted-types/) |
| 344 | `iter27-sri-byte-integrity-before-execution` | subresource-integrity | `normative_standard` | **Pin executable and stylesheet bytes with SRI before mini-app execution**<br>[SPEC FACT] SRI requires integrity metadata containing a hash function and digest to validate a response; conformant use... | [https://www.w3.org/TR/SRI/](https://www.w3.org/TR/SRI/) |
| 345 | `iter27-sri-cors-integrity-policy` | subresource-integrity-cross-origin | `normative_standard` | **Require CORS and integrity metadata for cross-origin mini-app resources**<br>[SPEC FACT] The SRI specification states that integrity-protected cross-origin requests require CORS and that using SRI ... | [https://www.w3.org/TR/SRI/](https://www.w3.org/TR/SRI/) |
| 346 | `iter27-dom-xss-safe-sinks` | dom-xss-safe-sinks | `security_standard` | **Default-deny dangerous DOM sinks and use text or DOM construction for untrusted mini-app data**<br>[SPEC FACT] OWASP recommends textContent for untrusted display data, says innerText or textContent should replace innerH... | [https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html) |
| 347 | `wasm-runtime-capability-memory-isolation-001` | wasm/runtime-sandboxing/linear-memory/capability-imports | `normative_standard` | **Make Wasm imports the only host capability and keep linear memory app-scoped**<br>[SPEC FACT] Core says validated code executes in a memory-safe, sandboxed environment; a module has no ambient access, a... | [https://www.w3.org/TR/wasm-core-2/](https://www.w3.org/TR/wasm-core-2/) |
| 348 | `wasm-runtime-indirect-call-table-safety-002` | wasm/tables/call-indirect/type-and-bounds-safety | `normative_standard` | **Fail closed on out-of-range, empty, or type-mismatched indirect calls**<br>[SPEC FACT] Core 1 defines a table as an array of opaque values whose funcref entries may have heterogeneous function ty... | [https://www.w3.org/TR/wasm-core-1/](https://www.w3.org/TR/wasm-core-1/) |
| 349 | `wasm-runtime-streaming-instantiation-gate-003` | wasm/web-api/instantiate-streaming/mime-cors-integrity | `normative_standard` | **Gate instantiateStreaming on exact media type, CORS status, and package identity**<br>[SPEC FACT] The WebAssembly Web API algorithm for instantiateStreaming compiles a potential WebAssembly Response and the... | [https://www.w3.org/TR/wasm-web-api-1/](https://www.w3.org/TR/wasm-web-api-1/) |
| 350 | `wasm-runtime-csp-execution-policy-004` | wasm/csp/script-src/wasm-unsafe-eval | `normative_standard` | **Allow Wasm compilation narrowly without granting JavaScript eval**<br>[SPEC FACT] CSP3 lists new WebAssembly.Module(), compile(), compileStreaming(), instantiate(), and instantiateStreaming(... | [https://www.w3.org/TR/CSP3/](https://www.w3.org/TR/CSP3/) |
| 351 | `wasm-runtime-host-memory-budget-005` | wasm/memory-ceilings/host-quota/memory-grow | `normative_standard` | **Convert Wasm page limits into enforced per-app and aggregate host memory budgets**<br>[SPEC FACT] Core 2 makes the memory minimum the initial size and an optional maximum the size ceiling to which the memor... | [https://www.w3.org/TR/wasm-core-2/](https://www.w3.org/TR/wasm-core-2/) |
| 352 | `coi_timing_027_01` | cross_origin_isolation_header_pair | `normative_standard` | **COOP same-origin plus COEP require-corp or credentialless is the cross-origin-isolation header gate**<br>[SPEC FACT] WHATWG defines same-origin-plus-COEP as the cross-origin-isolation mode produced by Cross-Origin-Opener-Poli... | [https://html.spec.whatwg.org/multipage/origin.html](https://html.spec.whatwg.org/multipage/origin.html) |
| 353 | `coi_timing_027_02` | coep_corp_resource_admission | `industry_standard` | **COEP require-corp makes CORP or CORS an explicit opt-in for cross-origin no-CORS resources**<br>[SPEC FACT] Under COEP: require-corp, a no-CORS cross-origin resource is admitted only when it is same-origin or its res... | [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Embedder-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Embedder-Policy) |
| 354 | `coi_timing_027_03` | cross_origin_isolated_runtime_status | `normative_standard` | **crossOriginIsolated is the effective Window/Worker status bit for isolated execution**<br>[SPEC FACT] Window.crossOriginIsolated returns a boolean indicating whether the document is cross-origin isolated; the c... | [https://developer.mozilla.org/en-US/docs/Web/API/crossOriginIsolated](https://developer.mozilla.org/en-US/docs/Web/API/crossOriginIsolated) |
| 355 | `coi_timing_027_04` | shared_array_buffer_memory_sharing_gate | `normative_standard` | **SharedArrayBuffer creation and message-based shared memory require the isolated app/worker boundary**<br>[SPEC FACT] MDN identifies SharedArrayBuffer as an API with reduced restrictions in a cross-origin-isolated document: it... | [https://developer.mozilla.org/en-US/docs/Web/API/crossOriginIsolated](https://developer.mozilla.org/en-US/docs/Web/API/crossOriginIsolated) |
| 356 | `coi_timing_027_05` | high_resolution_timer_coarsening | `normative_standard` | **performance.now() is coarsened for timing-attack resistance and gains finer resolution only with isolation**<br>[SPEC FACT] High Resolution Time Level 3 defines coarsen time with a default resolution of 100 microseconds or a higher ... | [https://w3c.github.io/hr-time/](https://w3c.github.io/hr-time/) |
| 357 | `push_vapid_028_01` | rfc_8030_web_push_protocol_gateway | `normative_standard` | **RFC 8030 defines HTTP/2 push delivery with TTL, Urgency headers and Topic message replacement**<br>[SPEC FACT] RFC 8030 establishes the protocol for delivering push messages from an application server to a push service.... | [https://www.rfc-editor.org/rfc/rfc8030](https://www.rfc-editor.org/rfc/rfc8030) |
| 358 | `push_vapid_028_02` | rfc_8291_web_push_payload_encryption | `normative_standard` | **RFC 8291 mandates aes128gcm single-record payload encryption using ECDH P-256 and HKDF-SHA-256**<br>[SPEC FACT] RFC 8291 specifies an end-to-end message encryption scheme ensuring confidentiality and integrity for push m... | [https://www.rfc-editor.org/rfc/rfc8291](https://www.rfc-editor.org/rfc/rfc8291) |
| 359 | `push_vapid_028_03` | rfc_8292_vapid_application_server_identification | `normative_standard` | **RFC 8292 specifies Voluntary Application Server Identification (VAPID) using ECDSA P-256 signed JWT tokens**<br>[SPEC FACT] RFC 8292 defines the VAPID authentication scheme, enabling an application server to voluntarily identify its... | [https://www.rfc-editor.org/rfc/rfc8292](https://www.rfc-editor.org/rfc/rfc8292) |
| 360 | `push_vapid_028_04` | w3c_push_api_subscription_lifecycle | `normative_standard` | **W3C Push API establishes PushManager subscription lifecycle, userVisibleOnly constraints and secure endpoint scoping**<br>[SPEC FACT] The W3C Push API defines the web platform interfaces connecting client workers to remote push services. The ... | [https://www.w3.org/TR/push-api/](https://www.w3.org/TR/push-api/) |
| 361 | `push_vapid_028_05` | superapp_push_gateway_mediation_and_rate_limiting | `platform_practice` | **Super-app push gateway mediation isolates mini-app endpoints, enforces frequency caps, and controls battery impact**<br>[SPEC FACT] In enterprise super-app architectures (such as WeChat, Alipay, and Grab), native push delivery over OS chann... | [https://developer.mozilla.org/en-US/docs/Web/API/Push_API](https://developer.mozilla.org/en-US/docs/Web/API/Push_API) |
| 362 | `host_notif_028_01` | whatwg_notifications_api_standard | `normative_standard` | **WHATWG Notifications API defines permission lifecycle, tag replacement, action buttons and silent delivery**<br>[SPEC FACT] The WHATWG Notifications API defines cross-platform standards for displaying notifications outside a web pag... | [https://notifications.spec.whatwg.org/](https://notifications.spec.whatwg.org/) |
| 363 | `host_notif_028_02` | w3c_badging_api_application_indicators | `normative_standard` | **W3C Badging API provides unread counts and status flags with anti-spoofing and platform integration**<br>[SPEC FACT] The W3C Badging API allows installed web applications to set an application badge, typically displayed along... | [https://w3c.github.io/badging/](https://w3c.github.io/badging/) |
| 364 | `host_notif_028_03` | android_notification_channels_tenant_isolation | `normative_standard` | **Android Notification Channels enable granular priority, user-controlled muting, and per-mini-app channel grouping**<br>[SPEC FACT] Since Android 8.0 (API level 26), all native notifications must be assigned to a NotificationChannel; unassi... | [https://developer.android.com/develop/ui/views/notifications/channels](https://developer.android.com/develop/ui/views/notifications/channels) |
| 365 | `host_notif_028_04` | apple_usernotifications_category_presentation | `normative_standard` | **Apple UserNotifications framework requires UNNotificationCategory registration, action handling and foreground presentation**<br>[SPEC FACT] Apple's iOS UserNotifications framework governs system notification presentation and interaction. Applicatio... | [https://developer.apple.com/documentation/usernotifications/unnotificationcategory](https://developer.apple.com/documentation/usernotifications/unnotificationcategory) |
| 366 | `host_notif_028_05` | superapp_notification_governance_and_frequency_capping | `platform_practice` | **Super-app notification governance enforces user consent, frequency caps, quiet hours and anti-phishing safeguards**<br>[SPEC FACT] In super-app ecosystems (e.g. WeChat Service Notifications / Template Messages, Alipay Message Center, Gojek... | [https://developer.mozilla.org/en-US/docs/Web/API/Notifications_API](https://developer.mozilla.org/en-US/docs/Web/API/Notifications_API) |
| 367 | `display_chrome_028_01` | w3c_manifest_display_override_fallback | `normative_standard` | **W3C Web App Manifest display_override establishes developer fallback chain across modern window modes**<br>[SPEC FACT] The standard Web Application Manifest defines the 'display' member with discrete values ('fullscreen', 'stan... | [https://www.w3.org/TR/appmanifest/](https://www.w3.org/TR/appmanifest/) |
| 368 | `display_chrome_028_02` | wicg_window_controls_overlay_geometry | `normative_standard` | **WICG Window Controls Overlay enables titlebar extension with geometrychange events and CSS env() rect variables**<br>[SPEC FACT] The Window Controls Overlay (WCO) specification allows installed web applications to take control of the tit... | [https://wicg.github.io/window-controls-overlay/](https://wicg.github.io/window-controls-overlay/) |
| 369 | `display_chrome_028_03` | wechat_capsule_button_safe_area_bounding_rect | `vendor_specification` | **WeChat mini program capsule button architecture reserves persistent host controls via wx.getMenuButtonBoundingClientRect**<br>[SPEC FACT] WeChat Mini Programs establish the industry benchmark for mobile super-app window and navigation governance.... | [https://developers.weixin.qq.com/miniprogram/dev/api/ui/menu/wx.getMenuButtonBoundingClientRect.html](https://developers.weixin.qq.com/miniprogram/dev/api/ui/menu/wx.getMenuButtonBoundingClientRect.html) |
| 370 | `display_chrome_028_04` | superapp_custom_navigation_security_boundaries | `platform_practice` | **Super-app custom navigation enforces immutable security origins, exit guarantees and anti-spoofing overlays**<br>[SPEC FACT] Allowing third-party mini-apps to control their window titlebars and navigation chrome introduces severe sec... | [https://developer.mozilla.org/en-US/docs/Web/API/Window_Controls_Overlay_API](https://developer.mozilla.org/en-US/docs/Web/API/Window_Controls_Overlay_API) |
| 371 | `display_chrome_028_05` | cross_platform_safe_area_and_notch_insets | `normative_standard` | **Cross-platform safe area and notch integration unifies CSS env(safe-area-inset) with titlebar geometry across mobile and desktop**<br>[SPEC FACT] Modern mobile and desktop operating systems exhibit diverse hardware cutouts, display notches, rounded scree... | [https://wicg.github.io/window-controls-overlay/](https://wicg.github.io/window-controls-overlay/) |
| 372 | `fedcm_029_01` | w3c_fedcm_browser_mediated_identity_federation | `normative_standard` | **W3C FedCM establishes browser-mediated identity federation mitigating third-party cookie deprecation and link tracking**<br>[SPEC FACT] The Federated Credential Management (FedCM) API defines a browser-mediated mechanism allowing users to log i... | [https://w3c-fedid.github.io/FedCM/](https://w3c-fedid.github.io/FedCM/) |
| 373 | `fedcm_029_02` | fedcm_idp_http_api_endpoints_and_metadata | `normative_standard` | **FedCM Identity Provider HTTP API enforces strict endpoint routing, /.well-known validation and account metadata isolation**<br>[SPEC FACT] The FedCM specification mandates four core HTTP endpoints on the Identity Provider to govern identity mediat... | [https://w3c-fedid.github.io/FedCM/](https://w3c-fedid.github.io/FedCM/) |
| 374 | `fedcm_029_03` | fedcm_permissions_policy_and_cross_origin_iframe_gating | `normative_standard` | **FedCM integrates with W3C Permissions Policy requiring identity-credentials-get delegation for embedded contexts**<br>[SPEC FACT] The FedCM specification defines the `identity-credentials-get` policy-controlled feature under the W3C Permi... | [https://w3c-fedid.github.io/FedCM/](https://w3c-fedid.github.io/FedCM/) |
| 375 | `fedcm_029_04` | w3c_credential_management_container_architecture | `normative_standard` | **W3C Credential Management API establishes a unified container for passwords, federated tokens, and public-key credentials**<br>[SPEC FACT] The W3C Credential Management API defines the `CredentialsContainer` interface (exposed as `navigator.creden... | [https://w3c.github.io/webappsec-credential-management/](https://w3c.github.io/webappsec-credential-management/) |
| 376 | `fedcm_029_05` | superapp_guest_identity_mediation_and_privacy_boundaries | `platform_practice` | **Super-app guest identity mediation isolates host master credentials and enforces scoped pseudonymous identifiers**<br>[SPEC FACT] In commercial super-app ecosystems (e.g. WeChat, Alipay, Grab, Gojek), mini-apps run as semi-trusted third-p... | [https://w3c-fedid.github.io/FedCM/](https://w3c-fedid.github.io/FedCM/) |
| 377 | `peripheral_029_01` | wicg_webusb_protected_interface_classes_and_landing_page | `normative_standard` | **WICG WebUSB API mandates protected interface class exclusions and device landing page validation**<br>[SPEC FACT] The WebUSB API provides direct script access to USB devices connected to the host system via `navigator.usb`... | [https://wicg.github.io/webusb/](https://wicg.github.io/webusb/) |
| 378 | `peripheral_029_02` | wicg_webhid_input_filtering_and_macro_injection_defense | `normative_standard` | **WICG WebHID API enforces top-level user gesture gating and protects top-level keyboard and mouse report collections**<br>[SPEC FACT] The WebHID API (`navigator.hid`) enables web applications to interact with Human Interface Devices using HID... | [https://wicg.github.io/webhid/](https://wicg.github.io/webhid/) |
| 379 | `peripheral_029_03` | wicg_web_serial_baud_rate_and_modem_command_mitigation | `normative_standard` | **WICG Web Serial API restricts serial communication, mitigating AT-command modem hijack and baud rate saturation**<br>[SPEC FACT] The Web Serial API (`navigator.serial`) connects web applications to serial devices, including microcontroll... | [https://wicg.github.io/serial/](https://wicg.github.io/serial/) |
| 380 | `peripheral_029_04` | hardware_sandbox_escape_and_firmware_flashing_defense | `platform_practice` | **Hardware sandbox escape mitigations counter firmware reprogramming, macro injection and persistent peripheral grants**<br>[SPEC FACT] Peer-reviewed security research by Trampert et al. (WWW '25: 'Peripheral Instinct: How External Devices Brea... | [https://wicg.github.io/webusb/](https://wicg.github.io/webusb/) |
| 381 | `peripheral_029_05` | superapp_container_hardware_policy_matrix_and_device_attestation | `platform_practice` | **Super-app container hardware policy matrix unifies device attestation, capability scoping and enterprise isolation**<br>[SPEC FACT] In modern enterprise and retail super-app deployments (e.g. logistics tracking, warehouse management, mobile... | [https://w3c.github.io/webappsec-permissions-policy/](https://w3c.github.io/webappsec-permissions-policy/) |
| 382 | `billing_029_01` | google_play_user_choice_billing_dual_screens_and_reporting | `platform_practice` | **Google Play User Choice Billing establishes dual-screen choice architecture and mandatory 24-hour transaction reporting**<br>[SPEC FACT] Google Play's User Choice Billing program permits registered developers to offer an alternative in-app billi... | [https://support.google.com/googleplay/android-developer/answer/13821247](https://support.google.com/googleplay/android-developer/answer/13821247) |
| 383 | `billing_029_02` | google_play_billing_choice_program_fee_tiers_and_external_links | `platform_practice` | **Google Play Billing Choice Program defines differentiated fee tiers for alternative billing and external web links**<br>[SPEC FACT] The Google Play Billing Choice Program (operating across the UK, EEA, and US) formalizes monetization polici... | [https://support.google.com/googleplay/android-developer/answer/17161464](https://support.google.com/googleplay/android-developer/answer/17161464) |
| 384 | `billing_029_03` | apple_eu_alternative_payment_options_and_storekit_entitlement | `platform_practice` | **Apple EU alternative payment framework enforces StoreKit external purchase entitlements, disclosure sheets and reporting**<br>[SPEC FACT] In compliance with the European Union Digital Markets Act (DMA), Apple established alternative payment optio... | [https://developer.apple.com/support/alternative-payment-options-in-the-eu/](https://developer.apple.com/support/alternative-payment-options-in-the-eu/) |
| 385 | `billing_029_04` | apple_eu_reduced_commission_and_core_technology_fee_economics | `platform_practice` | **Apple EU commercial terms establish reduced commissions, payment processing surcharges and the Core Technology Fee**<br>[SPEC FACT] Under Apple's Alternative Terms Addendum for EU Apps, commercial structures are unbundled into three distinc... | [https://developer.apple.com/support/alternative-payment-options-in-the-eu/](https://developer.apple.com/support/alternative-payment-options-in-the-eu/) |
| 386 | `billing_029_05` | superapp_unified_multi_provider_billing_abstraction_layer | `platform_practice` | **Super-app unified multi-provider billing abstraction layer reconciles host store compliance with mini-app monetization**<br>[SPEC FACT] Super apps operate in heterogeneous cross-platform environments: iOS (governed by Apple App Store Guidelines... | [https://support.google.com/googleplay/android-developer/answer/13821247](https://support.google.com/googleplay/android-developer/answer/13821247) |
| 387 | `webtrans_030_01` | Networking & Transport Standards | `normative_standard` | **WebTransport over HTTP/3 establishes low-latency multiplexed transport via extended CONNECT and QUIC datagrams**<br>W3C WebTransport and IETF draft-ietf-webtrans-http3 define an ECMAScript API and HTTP/3 transport binding supporting bidirectional/unidirectional streams and unreliable datagrams over a single QUIC connection. | [https://www.w3.org/TR/webtransport/](https://www.w3.org/TR/webtransport/) |
| 388 | `webtrans_030_02` | Networking & Transport Standards | `normative_standard` | **WebTransport datagram streams enforce MTU size boundaries, send order arbitration and stream backpressure**<br>W3C WebTransport exposes WebTransportDatagramDuplexStream with explicit maxDatagramSize constraints and WritableStream backpressure controls. | [https://developer.mozilla.org/en-US/docs/Web/API/WebTransportDatagramDuplexStream](https://developer.mozilla.org/en-US/docs/Web/API/WebTransportDatagramDuplexStream) |
| 389 | `webtrans_030_03` | Networking & Transport Standards | `normative_standard` | **WebTransport enforces strict TLS 1.3 encryption, Origin checks and 14-day pinned certificate hash authentication**<br>WebTransport prohibits cleartext transmission, mandates TLS 1.3/QUIC encryption, validates Origin headers, and supports serverCertificateHashes for scoped local development. | [https://w3c.github.io/webtransport/](https://w3c.github.io/webtransport/) |
| 390 | `webtrans_030_04` | Networking & Transport Standards | `normative_standard` | **Super-app container session pooling optimizes HTTP/3 connection re-use while isolating mini-app contexts**<br>WebTransport over HTTP/3 supports connection pooling across multiple sessions over a single QUIC connection while maintaining separate security contexts. | [https://datatracker.ietf.org/doc/html/draft-ietf-webtrans-http3](https://datatracker.ietf.org/doc/html/draft-ietf-webtrans-http3) |
| 391 | `webtrans_030_05` | Networking & Transport Standards | `normative_standard` | **WebTransport dedicated worker execution isolates real-time network processing from UI rendering threads**<br>WebTransport is exposed in DedicatedWorkerGlobalScope, enabling asynchronous packet decoding and ingestion outside the main UI event loop. | [https://developer.mozilla.org/en-US/docs/Web/API/WebTransport_API](https://developer.mozilla.org/en-US/docs/Web/API/WebTransport_API) |
| 392 | `webgpu_030_01` | Graphics & Compute Sandboxing | `candidate_recommendation` | **W3C WebGPU establishes hardware-abstracted GPUAdapter and GPUDevice architectures with explicit limit contracts**<br>W3C WebGPU Candidate Recommendation defines GPUAdapter and GPUDevice interfaces with enforceable hardware limits on buffer, texture, and compute storage. | [https://www.w3.org/TR/webgpu/](https://www.w3.org/TR/webgpu/) |
| 393 | `webgpu_030_02` | Graphics & Compute Sandboxing | `candidate_recommendation` | **WebGPU Shading Language enforces compile-time memory safety, bounds checking and deterministic execution**<br>W3C WGSL eliminates arbitrary pointer arithmetic and mandates strict array/buffer out-of-bounds safety rules. | [https://www.w3.org/TR/WGSL/](https://www.w3.org/TR/WGSL/) |
| 394 | `webgpu_030_03` | Graphics & Compute Sandboxing | `candidate_recommendation` | **WebGPU device.lost lifecycle arbitration enables resilient error recovery and background VRAM reclamation**<br>GPUDevice.lost Promise notifies applications of GPU context invalidation (OOM, driver reset, or deliberate host eviction) with reason tagging. | [https://developer.mozilla.org/en-US/docs/Web/API/GPUDevice/lost](https://developer.mozilla.org/en-US/docs/Web/API/GPUDevice/lost) |
| 395 | `webgpu_030_04` | Graphics & Compute Sandboxing | `candidate_recommendation` | **WebGPU timestamp queries enforce Cross-Origin Isolation gating and microsecond clock coarsening**<br>WebGPU restricts high-resolution timestamp queries behind Cross-Origin Isolation and applies clock coarsening to neutralize GPU cache timing attacks. | [https://www.w3.org/TR/webgpu/](https://www.w3.org/TR/webgpu/) |
| 396 | `webgpu_030_05` | Graphics & Compute Sandboxing | `candidate_recommendation` | **Super-app container GPU governance establishes capability tiers, VRAM quotas and graceful WebGL 2 fallback**<br>Super-app multi-tier container governance audits WebGPU hardware availability, bounds VRAM usage per tenant, and arbitrates fallback to WebGL 2 or 2D rendering. | [https://developer.chrome.com/docs/web-platform/webgpu/troubleshooting-tips](https://developer.chrome.com/docs/web-platform/webgpu/troubleshooting-tips) |
| 397 | `multimodal_030_01` | Multimodal & Media Standards | `normative_standard` | **W3C WebCodecs defines low-overhead hardware-accelerated video/audio encoding and decoding interfaces**<br>W3C WebCodecs Working Draft standardizes VideoEncoder, VideoDecoder, AudioEncoder, and AudioDecoder interfaces for raw frame manipulation without WebAssembly overhead. | [https://www.w3.org/TR/webcodecs/](https://www.w3.org/TR/webcodecs/) |
| 398 | `multimodal_030_02` | Multimodal & Media Standards | `normative_standard` | **WebCodecs VideoFrame lifecycle mandates explicit disposal to prevent GPU texture and memory leaks**<br>VideoFrame and AudioData objects hold underlying GPU memory and require explicit .close() invocations to avoid catastrophic memory leaks. | [https://developer.mozilla.org/en-US/docs/Web/API/VideoFrame](https://developer.mozilla.org/en-US/docs/Web/API/VideoFrame) |
| 399 | `multimodal_030_03` | Multimodal & Media Standards | `normative_standard` | **W3C WebXR Device API standardizes spatial computing sessions, reference spaces and immersive rendering loops**<br>W3C WebXR Device API Recommendation defines XRSession modes ('inline', 'immersive-vr', 'immersive-ar') and XRReferenceSpace coordinates for spatial applications. | [https://www.w3.org/TR/webxr/](https://www.w3.org/TR/webxr/) |
| 400 | `multimodal_030_04` | Multimodal & Media Standards | `normative_standard` | **WebXR xr-spatial-tracking Permissions Policy restricts physical room scanning and biometric tracking telemetry**<br>The xr-spatial-tracking Permissions-Policy directive gates access to real-world coordinate tracking, protecting user physical environments and gaze data. | [https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Permissions-Policy/xr-spatial-tracking](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Permissions-Policy/xr-spatial-tracking) |
| 401 | `multimodal_030_05` | Multimodal & Media Standards | `normative_standard` | **Super-app container multimodal governance enforces camera, GPU and audio concurrency arbitration**<br>Unified host arbitration coordinates camera hardware, hardware media decoders, and audio focus between concurrent mini-apps and host features. | [https://developer.mozilla.org/en-US/docs/Web/API/WebCodecs_API](https://developer.mozilla.org/en-US/docs/Web/API/WebCodecs_API) |
| 402 | `proc_iso_031_01` | Process Isolation & Stability | `platform_practice` | **Chromium Multi-Process Architecture and Site Isolation enforce OS-level process boundaries for untrusted origins**<br>Site Isolation uses separate OS-level renderer processes for distinct web sites and hosts out-of-process iframes (OOPIF) to protect memory across origins. | [https://chromium.googlesource.com/chromium/src/+/main/docs/process_model_and_site_isolation.md](https://chromium.googlesource.com/chromium/src/+/main/docs/process_model_and_site_isolation.md) |
| 403 | `proc_iso_031_02` | Process Isolation & Stability | `platform_practice` | **Android WebViewRenderProcessClient handles renderer process crashes, hangs and priority adjustments**<br>Android WebViewRenderProcessClient provides onRenderProcessGone and onRenderProcessUnresponsive callbacks to detect and handle out-of-process renderer crashes. | [https://developer.android.com/reference/android/webkit/WebViewRenderProcessClient](https://developer.android.com/reference/android/webkit/WebViewRenderProcessClient) |
| 404 | `proc_iso_031_03` | Process Isolation & Stability | `platform_practice` | **Apple WebKit WKProcessPool isolates web content processes and governs shared cookie/cache partitions**<br>WKProcessPool in WKWebViewConfiguration establishes isolated WebContent processes and governs cookie/storage partitions between distinct mini-apps on iOS. | [https://developer.apple.com/documentation/webkit/wkprocesspool](https://developer.apple.com/documentation/webkit/wkprocesspool) |
| 405 | `proc_iso_031_04` | Process Isolation & Stability | `platform_practice` | **Renderer crash recovery requires transactional state snapshots and automated user session reconstruction**<br>Resilient super-app architectures persist incremental navigation checkpoints to restore user state seamlessly after unexpected renderer process termination. | [https://developer.android.com/reference/android/webkit/WebViewClient](https://developer.android.com/reference/android/webkit/WebViewClient) |
| 406 | `proc_iso_031_05` | Process Isolation & Stability | `platform_practice` | **Super-App Multi-Process Topology decouples UI Host, Native Service Broker, and Sandboxed Mini-App Renderers**<br>A 3-tier multi-process architecture isolates the Host UI process, Background Service Broker, and Sandboxed Mini-App Renderer processes via IPC. | [https://developer.android.com/reference/android/webkit/WebViewClient](https://developer.android.com/reference/android/webkit/WebViewClient) |
| 407 | `delta_ota_031_01` | Package Distribution & OTA | `normative_standard` | **RFC 3284 VCDIFF standardizes byte-level differential encoding for bandwidth-efficient software updates**<br>RFC 3284 specifies the VCDIFF generic differential and linear-time compression data format utilizing ADD, COPY, and RUN instructions. | [https://www.rfc-editor.org/rfc/rfc3284](https://www.rfc-editor.org/rfc/rfc3284) |
| 408 | `delta_ota_031_02` | Package Distribution & OTA | `normative_standard` | **RFC 8878 Zstandard provides real-time decompression speeds and domain-specific custom dictionary compression**<br>RFC 8878 specifies Zstandard (zstd) combining Finite State Entropy (FSE) and custom dictionary training for high-ratio, fast-decompression payloads. | [https://www.rfc-editor.org/rfc/rfc8878](https://www.rfc-editor.org/rfc/rfc8878) |
| 409 | `delta_ota_031_03` | Package Distribution & OTA | `platform_practice` | **Super-App Differential Packaging builds directed delta update graphs between consecutive active versions**<br>Differential update pipelines generate delta packages across an N-version compatibility window and bind patch artifacts to base SHA-256 hashes. | [https://github.com/facebook/zstd](https://github.com/facebook/zstd) |
| 410 | `delta_ota_031_04` | Package Distribution & OTA | `platform_practice` | **Atomic patch reconstruction enforces pre-validation, staged assembly and automatic fallback to full archives**<br>Client-side patch reconstruction verifies base hash before patching, validates target hash post-patching, and falls back to full archives on mismatch. | [https://www.rfc-editor.org/rfc/rfc3284](https://www.rfc-editor.org/rfc/rfc3284) |
| 411 | `delta_ota_031_05` | Package Distribution & OTA | `platform_practice` | **Apple App Store Rule 4.7 governs OTA updates, prohibiting native binary execution and mandating audit logs**<br>Apple App Store Review Guideline 4.7 permits dynamic code delivery solely for interpreted web technologies while strictly forbidding native binary execution. | [https://developer.apple.com/app-store/review/guidelines/](https://developer.apple.com/app-store/review/guidelines/) |
| 412 | `trace_otel_031_01` | Observability & Telemetry | `normative_standard` | **W3C Trace Context Recommendation standardizes distributed traceparent headers across heterogeneous systems**<br>W3C Trace Context defines the traceparent HTTP header format (version-trace_id-parent_id-trace_flags) for end-to-end distributed transaction tracing. | [https://w3c.github.io/trace-context/](https://w3c.github.io/trace-context/) |
| 413 | `trace_otel_031_02` | Observability & Telemetry | `candidate_recommendation` | **W3C Baggage defines open contextual key-value propagation for distributed application telemetry**<br>W3C Baggage establishes the 'baggage' HTTP header format to transport user context, tenant IDs, and telemetry metadata across service boundaries. | [https://w3c.github.io/baggage/](https://w3c.github.io/baggage/) |
| 414 | `trace_otel_031_03` | Observability & Telemetry | `open-specification` | **OpenTelemetry Distributed Tracing API bridges asynchronous JSAPI calls and native host execution**<br>OpenTelemetry Trace API and Context Propagators inject and extract distributed trace context across JavaScript WebViews and native mobile host runtimes. | [https://opentelemetry.io/docs/specs/otel/trace/api/](https://opentelemetry.io/docs/specs/otel/trace/api/) |
| 415 | `trace_otel_031_04` | Observability & Telemetry | `platform_practice` | **End-to-End Super-App Tracing correlates client user interactions, native bridge calls, gateway hops, and microservices**<br>Unified distributed tracing captures the complete journey from user UI interaction through native bridge, API gateway, and backend microservice spans. | [https://opentelemetry.io/docs/specs/otel/trace/api/](https://opentelemetry.io/docs/specs/otel/trace/api/) |
| 416 | `trace_otel_031_05` | Observability & Telemetry | `platform_practice` | **Distributed telemetry mandates automated PII scrubbing, token redaction, and strict data minimization**<br>Telemetry pipelines must scrub personally identifiable information (PII), credentials, and sensitive headers from trace attributes and baggage before storage. | [https://opentelemetry.io/docs/specs/otel/context/api-propagators/](https://opentelemetry.io/docs/specs/otel/context/api-propagators/) |


## Iteration 35: Privacy Attribution, Anti-Abuse Private State Tokens, and On-Device Ad Auctions (Protected Audience & Fenced Frames)

### Finding 462: WICG Attribution Reporting API Architecture: Source Registration, Trigger Registration, and Event-Level vs Aggregatable Conversion Delivery
- **ID**: `attr_035_01`
- **Topic**: `attribution_reporting_api_architecture_and_registration_lifecycle`
- **Evidence Level**: `normative_standard`
- **URL**: https://wicg.github.io/attribution-reporting-api/
- **Standards**: WICG Attribution Reporting API, RFC 8941 Structured Field Values for HTTP
- **Summary**: The WICG Attribution Reporting API enables privacy-preserving measurement of user conversions across web and mini-app contexts without third-party cookies or persistent cross-origin identifiers. Navigation and event sources are registered via 'Attribution-Reporting-Register-Source' response headers, while downstream conversion triggers are recorded via 'Attribution-Reporting-Register-Trigger' headers, allowing the client runtime to correlate source clicks/views with conversion events locally.
- **Key Points**:
  - Source registration occurs on navigation clicks or fetch requests emitting the Attribution-Reporting-Eligible request header and receiving Attribution-Reporting-Register-Source response headers containing source_event_id, destination, expiry, and priority.
  - Trigger registration occurs on conversion landing pages or transaction endpoints returning Attribution-Reporting-Register-Trigger headers with event_trigger_data, aggregatable_trigger_data, and debug_reporting flags.
  - Local client correlation eliminates third-party tracking: the user agent matches source and trigger origins based on top-level site matching rules, preventing ad networks from building cross-site behavioral dossiers.

### Finding 463: Differential Privacy & Randomized Response: Event-Level Noise, Coarse Value Bucketing, and Delay-Randomized Postback Scheduling
- **ID**: `attr_035_02`
- **Topic**: `differential_privacy_and_randomized_response_in_attribution`
- **Evidence Level**: `normative_standard`
- **URL**: https://wicg.github.io/attribution-reporting-api/
- **Standards**: WICG Attribution Reporting API, Differential Privacy Framework
- **Summary**: To prevent re-identification through high-entropy conversion feedback, the Attribution Reporting specification applies ε-differential privacy mechanisms. Event-level reports are bounded to 3 bits (8 conversion states) for click sources and 1 bit (2 conversion states) for view sources, with randomized response noise injected when report thresholds are breached. Delivery is scheduled across randomized multi-day windows to prevent traffic timing correlation.
- **Key Points**:
  - Randomized response injects cryptographic noise where the client reports a completely random conversion state with probability p, providing ε-differential privacy guarantees against conversion tracking attacks.
  - Information entropy is strictly clamped: click-through attributions allow a maximum of 3 bits of conversion data (values 0-7), while view-through attributions allow only 1 bit (0 or 1).
  - Postback scheduling introduces deterministic delays partitioned into reporting windows (e.g. 2 days, 7 days, 30 days) combined with randomized jitter to defeat network packet correlation.

### Finding 464: Aggregatable Reports & TEE Aggregation Service: 128-Bit Histogram Buckets, HPKE Payload Encryption, and Centralized Differential Privacy
- **ID**: `attr_035_03`
- **Topic**: `aggregatable_reports_and_trusted_execution_environments`
- **Evidence Level**: `normative_standard`
- **URL**: https://wicg.github.io/attribution-reporting-api/
- **Standards**: WICG Attribution Reporting API, RFC 9180 HPKE, RFC 9458 Encrypted Client Hello / Privacy Pass
- **Summary**: For rich, multi-dimensional conversion analytics (e.g., cart value, SKU categories, geographic region), the specification introduces Aggregatable Reports. Clients generate 128-bit histogram bucket contributions and encrypt the payload using Hybrid Public Key Encryption (HPKE). Decryption and aggregation can only occur inside a Trusted Execution Environment (TEE) run by an attested Aggregation Service, which adds Laplace noise and enforces strict query budgets.
- **Key Points**:
  - Aggregatable reports represent conversions as key-value pairs where the key is a 128-bit histogram bucket ID and the value is a bounded integer contribution (e.g., purchase value scaled to L1 budget).
  - Payloads are encrypted on-device using HPKE with public keys published by attested key management services, rendering the report unreadable to the publisher, super-app host, and ad-tech collector.
  - The Aggregation Service operates in a hardware TEE (AWS Nitro Enclaves or GCP Confidential Space), decrypting batched reports, computing aggregate sums, adding differential privacy noise, and enforcing single-use replay prevention.

### Finding 465: Native Host Attribution Parity: Apple SKAdNetwork 4.0 / AdAttributionKit and Google Play Install Referrer API Integration
- **ID**: `attr_035_04`
- **Topic**: `cross_platform_attribution_parity_skadnetwork_and_play_install_referrer`
- **Evidence Level**: `platform_practice`
- **URL**: https://developer.apple.com/documentation/storekit/skadnetwork
- **Standards**: Apple SKAdNetwork 4.0 / AdAttributionKit, Google Play Install Referrer API v2.2
- **Summary**: Super-app mini-app containers must interoperate with native mobile OS attribution frameworks without violating App Store or Google Play tracking policies. Apple SKAdNetwork 4.0 / AdAttributionKit provides coarse conversion values (low/medium/high) and hierarchical source IDs, while Google Play Install Referrer API provides secure UTM parameter passing for mini-app campaigns launched from external channels.
- **Key Points**:
  - Apple SKAdNetwork 4.0 / AdAttributionKit utilizes hierarchical source identifiers (2 to 4 digits) and coarse conversion values (fine, low, medium, high) delivered across 3 distinct postback windows (0-2 days, 3-7 days, 8-35 days).
  - Google Play Install Referrer API provides cryptographically signed install referrer strings, click timestamps, and install begin timestamps, allowing the super-app host to attribute native app installs and mini-app activations.
  - Super-app host container policy must strictly forbid exchanging raw IDFA/AAID identifiers with mini-app third parties, mediating all external attribution through privacy-preserving OS APIs.

### Finding 466: Super-App Attribution Gateway Architecture: Cryptographic Click Attestation, Host-Mediated Redirects, and Ad Fraud Mitigations
- **ID**: `attr_035_05`
- **Topic**: `superapp_attribution_gateway_and_fraud_prevention`
- **Evidence Level**: `platform_practice`
- **URL**: https://wicg.github.io/attribution-reporting-api/
- **Standards**: WICG Attribution Reporting API, IETF RFC 8941, OWASP Mobile Top 10 M1/M4
- **Summary**: To prevent click spoofing, impression laundering, and attribution fraud in the mini-app marketplace, the super-app host must deploy an Attribution Gateway. The gateway signs campaign navigation clicks with host asymmetric keys, enforces domain allowlisting for conversion landing pages, and filters bot traffic before attributing revenue shares or campaign incentives to publishers.
- **Key Points**:
  - Host-mediated click signing injects cryptographic nonces and short-lived HMAC signatures into mini-app launch deep links, verifying that launches originated from legitimate user taps rather than background automated intents.
  - Mini-app conversion triggers are validated against registered manifest scopes and e-commerce checkout receipts, eliminating fictitious conversion claims by fraudulent affiliates.
  - Conversion postbacks to external ad-tech partners are batched, throttled, and sanitized by the super-app gateway to prevent timing-based side-channel leaks.

### Finding 467: WICG Private State Token API: Privacy Pass Verifiable Oblivious PRF (VOPRF) Primitives and Cross-Site Anti-Abuse Tokens
- **ID**: `pst_035_01`
- **Topic**: `private_state_token_architecture_and_privacy_pass_primitives`
- **Evidence Level**: `normative_standard`
- **URL**: https://wicg.github.io/trust-token-api/
- **Standards**: WICG Private State Token API, RFC 9576 Privacy Pass Architecture, RFC 9577 Two-Party Token Issuance, RFC 9578 Privacy Pass HTTP Authentication
- **Summary**: The WICG Private State Token API (formerly Trust Tokens) provides a privacy-preserving mechanism to convey trust and combat fraud across contexts without user tracking. Built upon the IETF Privacy Pass protocol (RFC 9576, RFC 9577, RFC 9578) using Verifiable Oblivious Pseudorandom Functions (VOPRF), issuers can cryptographically attest to client trustworthiness without linking the issuance context to the subsequent redemption context.
- **Key Points**:
  - Private State Tokens replace linkable tracking cookies with blinded, cryptographically unforgeable tokens that convey only 2 to 3 bits of state (a categorical value chosen from 6 enum states).
  - Issuance and redemption utilize Privacy Pass VOPRF protocols where the client blinds a token before sending it to the issuer; the issuer signs the blinded token with private keys, allowing the client to unblind it and obtain a valid signature without the issuer observing the final token payload.
  - Redemption generates a signed Redemption Record (RR) that can be cached and attached via Sec-Redemption-Record HTTP headers to prove previous verification without revealing user identity.

### Finding 468: Private State Token Protocol Lifecycle: Key Commitments, HTTP Sec-Headers, and Redemption Record Caching
- **ID**: `pst_035_02`
- **Topic**: `private_state_token_issuance_and_redemption_lifecycle`
- **Evidence Level**: `normative_standard`
- **URL**: https://wicg.github.io/trust-token-api/
- **Standards**: WICG Private State Token API, WHATWG Fetch Standard, RFC 8446 TLS 1.3
- **Summary**: Private State Token operations are integrated into the Fetch specification. Token issuers serve cryptographic public key commitments at /.well-known/ endpoints. During issuance, clients invoke fetch() with privateToken: {operation: 'token-request'} sending Sec-Private-State-Token headers. Later, top-level origins or embedded mini-apps invoke {operation: 'token-redemption'} to produce a Redemption Record with a defined Sec-Private-State-Token-Lifetime.
- **Key Points**:
  - Issuers maintain a /.well-known/pst-key-commitment HTTP endpoint providing cryptographic keys and expiry metadata, which user agents periodically fetch and validate.
  - Token issuance requests send masked tokens via the Sec-Private-State-Token request header; issuers return base64-encoded signed masked tokens via Sec-Private-State-Token response headers.
  - Redemption operations exchange a valid token for a Redemption Record cached in the browser's tokenStore, governed by Sec-Private-State-Token-Lifetime response headers (measured in seconds).

### Finding 469: IETF Privacy Pass Standards Suite: RFC 9576 Architecture, RFC 9577 Two-Party Protocols, and RFC 9578 HTTP Authentication
- **ID**: `pst_035_03`
- **Topic**: `privacy_pass_ietf_standards_rfc9576_rfc9577_rfc9578`
- **Evidence Level**: `normative_standard`
- **URL**: https://www.rfc-editor.org/rfc/rfc9576.html
- **Standards**: RFC 9576 Privacy Pass Architecture, RFC 9577 Two-Party Token Issuance, RFC 9578 HTTP Authentication Scheme, RFC 9497 VOPRF
- **Summary**: RFC 9576 defines the official architecture for privacy-preserving attestation, establishing roles for Clients, Issuers, Origins, and Attesters. RFC 9577 specifies the two-party token issuance protocols based on Prime-order elliptic curves (Curve25519 and P-256) and VOPRF construction (RFC 9497). RFC 9578 defines the HTTP authorization challenge and token transmission scheme (WWW-Authenticate: PrivateToken).
- **Key Points**:
  - RFC 9576 categorizes token types into Type 1 (batched blind RSA tokens) and Type 2 (blind VOPRF tokens), ensuring that redemption is cryptographically unlinkable to issuance.
  - RFC 9577 standardizes cryptographic primitives for blind RSA (RSABSSA-SHA384-PSS-Zeroize) and VOPRF(P-256, SHA-256), providing zero-knowledge proofs of issuer public key consistency.
  - RFC 9578 standardizes the HTTP WWW-Authenticate: PrivateToken challenge-response mechanism, enabling servers to demand cryptographic anti-abuse tokens without triggering intrusive CAPTCHAs.

### Finding 470: PST Security Limits: 2-Issuer Top-Level Origin Clamping, Permissions Policy Gating, and Identity Leakage Mitigations
- **ID**: `pst_035_04`
- **Topic**: `private_state_token_security_and_partitioning_boundaries`
- **Evidence Level**: `normative_standard`
- **URL**: https://wicg.github.io/trust-token-api/
- **Standards**: WICG Private State Token API, W3C Permissions Policy, RFC 6265bis Cookie State
- **Summary**: To prevent colluding origins from creating multi-bit tracking vectors by chaining token redemptions across multiple issuers, the Private State Token API enforces strict runtime limits. The user agent restricts the number of active token issuers to at most 2 per top-level origin. Operations are guarded by the Permissions-Policy: private-state-token-redemption directive with default allowlist 'self'.
- **Key Points**:
  - Top-level origins are restricted to associating with at most 2 token issuers, preventing malicious sites from encoding user fingerprints across an arbitrary array of trust issuers.
  - The policy-controlled feature 'private-state-token-redemption' governs 'send-redemption-record' and 'token-redemption' operations, preventing embedded third-party iframes from redeeming tokens without top-level consent.
  - Redemption Records are cleared synchronously upon user data deletion (Clear-Site-Data: 'cookies', 'storage') and are partitioned by top-level site to prevent cross-site identity joining.

### Finding 471: Super-App Anti-Abuse Trust Mesh: Host Account Verification, Mini-App Bot Mitigation, and CAPTCHA-Free Frictionless Flow
- **ID**: `pst_035_05`
- **Topic**: `superapp_anti_abuse_trust_token_mesh`
- **Evidence Level**: `platform_practice`
- **URL**: https://wicg.github.io/trust-token-api/
- **Standards**: WICG Private State Token API, RFC 9576 Privacy Pass, OWASP Automated Threat Handbook OAT-007/OAT-019
- **Summary**: Super-app ecosystems leverage Private State Tokens to issue cryptographically verifiable trust credentials to authenticated, KYC-verified users. Mini-app developers can challenge incoming requests for a Redemption Record issued by the super-app root issuer. This eliminates fraudulent bot clicks, voucher hoarding, and Sybil attacks across mini-apps without leaking the user's master account identity.
- **Key Points**:
  - The super-app host operates a Private State Token issuer that issues blinded tokens to client containers upon successful phone/biometric KYC login.
  - High-risk mini-app transactions (e.g., flash sales, limited voucher claims, ticketing) issue a PrivateToken challenge requiring the mini-app to redeem a cached token.
  - The mini-app backend verifies the Redemption Record with the host's public key commitment, confirming that the client is an authenticated human without acquiring the user's phone number or account ID.

### Finding 472: WICG Protected Audience API: On-Device Ad Auctions, Interest Groups (joinAdInterestGroup), and Isolated Buyer/Seller Bidding Logic
- **ID**: `pa_035_01`
- **Topic**: `protected_audience_api_on_device_auction_architecture`
- **Evidence Level**: `normative_standard`
- **URL**: https://wicg.github.io/turtledove/
- **Standards**: WICG Protected Audience API, W3C Permissions Policy, WHATWG HTML Worklets
- **Summary**: The WICG Protected Audience API (formerly FLEDGE) standardizes privacy-preserving remarketing and ad auctions executed entirely on the client device. Mini-apps and publishers join interest groups via navigator.joinAdInterestGroup(), storing buyer bidding signals locally. Later, ad spaces run navigator.runAdAuction() where buyer generateBid() and seller scoreAd() scripts execute in isolated worklet environments without network access.
- **Key Points**:
  - Interest groups are owned by buyer origins and store bid logic URLs, user bidding signals, and trusted bidding signals keys with an automatic 30-day lifetime expiration.
  - On-device auctions are orchestrated via navigator.runAdAuction() using an AuctionConfig that specifies sellers, buyers, scoring logic, and decision timeouts.
  - Worklet isolation ensures that buyer bidding code (generateBid) and seller scoring code (scoreAd) run in dedicated execution sandboxes without network access, prevent tracking cookies or IP leaks during bidding.

### Finding 473: WICG Fenced Frames Specification: Opaque URL Rendering, FencedFrameConfig, and Cryptographic Network Disabling
- **ID**: `pa_035_02`
- **Topic**: `fenced_frames_isolated_rendering_and_fencedframeconfig`
- **Evidence Level**: `normative_standard`
- **URL**: https://wicg.github.io/fenced-frame/
- **Standards**: WICG Fenced Frame, WICG Protected Audience API, W3C Permissions Policy
- **Summary**: WICG Fenced Frames introduce a specialized embedding element (<fencedframe>) designed to display content based on cross-site data without allowing the parent page or embedded frame to communicate and leak user identities. Auction winners return an opaque FencedFrameConfig or URN UUID that cannot be introspected by script. After initial asset loading, fenced frames can execute window.fence.disableUntrustedNetwork() to ensure no exfiltration is possible.
- **Key Points**:
  - Fenced frames prevent cross-origin side-channel communication: unlike traditional iframes, postMessage, DOM traversal, and resize observation across the boundary are prohibited.
  - Auction results produce an opaque FencedFrameConfig object mapped to an internal URN UUID, preventing the host mini-app from reading the winning ad URL or price.
  - The window.fence API provides cryptographic isolation primitives, including disableUntrustedNetwork(), locking the frame down to local-only computation and precluding data egress.

### Finding 474: WICG Shared Storage API: Unpartitioned Cross-Site Storage, Worklet Operations, and Private Aggregation Output Gates
- **ID**: `pa_035_03`
- **Topic**: `shared_storage_worklets_and_url_selection_gates`
- **Evidence Level**: `normative_standard`
- **URL**: https://wicg.github.io/shared-storage/
- **Standards**: WICG Shared Storage API, W3C Permissions Policy, WHATWG Web IDL
- **Summary**: The WICG Shared Storage API provides an intentional, unpartitioned cross-site storage repository that can only be read within restricted isolated execution environments (SharedStorageWorklet). Mini-apps write keys and values via window.sharedStorage.set(). Reading is restricted to worklets that output results exclusively through one of two gates: URL Selection (returning an opaque index into an allowlist of URLs rendered in a fenced frame) or Private Aggregation.
- **Key Points**:
  - SharedStorage provides a key-value store unpartitioned by top-level site, accessible across mini-apps belonging to the same developer or consortium origin.
  - Direct read access via window.sharedStorage.get() is prohibited in document contexts; data is only readable inside SharedStorageWorklet module scripts.
  - Worklet output is strictly constrained to URL selection (charging a top-level entropy budget in bits) or Private Aggregation histogram contributions, mathematically bounding user re-identification risks.

### Finding 475: Cross-Site Private Aggregation: contributeToHistogram(), L1 Contribution Budgets, and Filtering IDs
- **ID**: `pa_035_04`
- **Topic**: `private_aggregation_and_contribution_bounding_budgets`
- **Evidence Level**: `normative_standard`
- **URL**: https://wicg.github.io/shared-storage/
- **Standards**: WICG Shared Storage API, WICG Protected Audience API, RFC 9180 HPKE
- **Summary**: Private Aggregation allows isolated contexts (Shared Storage worklets and Protected Audience auctions) to measure cross-site reach, frequency, and demographics. Worklets call privateAggregation.contributeToHistogram({bucket, value}). User agents enforce strict L1 contribution budgets per reporting site (max 65,536 units per 10-minute/daily window). Reports are encrypted with HPKE and delayed before dispatch to reporting endpoints.
- **Key Points**:
  - Worklets invoke privateAggregation.contributeToHistogram() to register integer contributions to specific 128-bit histogram buckets.
  - The client enforces an L1 contribution budget (sum of values <= 65,536 per 10-minute time slice) to guarantee bounded differential privacy noise addition during TEE aggregation.
  - Filtering IDs (8-bit or 64-bit integer labels) enable downstream Aggregation Services to slice reports by campaign ID or conversion cadence while maintaining deterministic report batching.

### Finding 476: Super-App In-Container Ad Exchange: On-Device Personalization, Fenced Mini-App Banners, and Zero-PII Egress Governance
- **ID**: `pa_035_05`
- **Topic**: `superapp_privacy_preserving_ad_exchange_and_personalization`
- **Evidence Level**: `platform_practice`
- **URL**: https://wicg.github.io/turtledove/
- **Standards**: WICG Protected Audience API, WICG Fenced Frame, WICG Shared Storage API
- **Summary**: Super-apps hosting hundreds of third-party merchants require on-device ad serving that delivers contextual personalization without exposing user browse history or transaction habits. Implementing Protected Audience and Shared Storage within the container SDK allows local ad auctions, renders promotional banners inside fenced frames, and verifies viewability without leaking user identifiers outside the super-app boundary.
- **Key Points**:
  - The super-app native container implements an on-device auction coordinator where mini-apps act as publishers and merchants act as interest group buyers.
  - Promotional banners and sponsored recommendations are rendered strictly inside <fencedframe> containers, blocking merchant script from accessing the hosting mini-app's DOM.
  - Ad impressions, clicks, and conversions are reported via Private Aggregation and Attribution Reporting, ensuring zero PII egress to external analytics vendors.

### Finding 477: W3C WebRTC 1.0: RTCPeerConnection Architecture, RTCDataChannel Isolation, and Host ICE Filtering
- **ID**: `webrtc_036_01`
- **Topic**: `webrtc_peer_connection_and_container_network_governance`
- **Evidence Level**: `normative_standard`
- **URL**: https://www.w3.org/TR/webrtc/
- **Standards**: W3C WebRTC 1.0, RFC 8829 JSEP, W3C Permissions Policy
- **Summary**: The W3C WebRTC 1.0 specification standardizes real-time audio, video, and arbitrary bidirectional data exchange between peers. In a super-app mini-app container, RTCPeerConnection and RTCDataChannel execution must be mediated by host security controls. The super-app host enforces ICE candidate filtering to prevent local IP leakage and mandates STUN/TURN credential injection via secure host proxies, restricting untrusted mini-apps from bypassing host egress network policies.
- **Key Points**:
  - RTCPeerConnection manages session state, SDP offer/answer negotiation, and secure media/data transport over SRTP/SCTP.
  - RTCDataChannel provides low-latency, ordered or unordered datagram transmission with configurable maxPacketLifeTime and maxRetransmits.
  - Container ICE candidate filtering strips internal private IPv4/IPv6 addresses from ICE gathering results, mitigating host intranet port scanning and device fingerprinting.
  - Super-app host injects ephemeral, short-lived TURN credentials into RTCConfiguration to enforce traffic inspection and policy compliance.

### Finding 478: W3C Media Capture and Streams: getUserMedia Constraints, Track Lifecycle, and Container Indicator Bridging
- **ID**: `webrtc_036_02`
- **Topic**: `mediacapture_streams_lifecycle_and_active_indicator_bridging`
- **Evidence Level**: `normative_standard`
- **URL**: https://www.w3.org/TR/mediacapture-streams/
- **Standards**: W3C Media Capture and Streams, W3C Permissions Policy, Android Camera2 / iOS AVCaptureSession
- **Summary**: The W3C Media Capture and Streams specification defines navigator.mediaDevices.getUserMedia() and the MediaStreamTrack object model for capturing audio and video from local devices. Mini-app containers must bind MediaStream access to W3C Permissions Policy ('camera', 'microphone'), present fine-grained native consent dialogues, and bridge active capture states directly to the OS status bar and super-app chrome active recording indicators.
- **Key Points**:
  - getUserMedia accepts MediaStreamConstraints specifying exact, ideal, and min/max capabilities for resolution, frameRate, facingMode, and audio noiseSuppression.
  - MediaStreamTrack lifecycle mandates explicit stop() transitions to release hardware sensors and hardware camera/mic power states.
  - Super-app container intercepts device enumeration (enumerateDevices) to sanitize persistent deviceId hashes across distinct mini-app origins.
  - Active capture sessions trigger host-level UI badges (green/orange dots) and enable user one-tap hardware capture revocation.

### Finding 479: W3C Screen Capture API: getDisplayMedia Surface Constraints and Super-App Sensitive UI Shielding
- **ID**: `webrtc_036_03`
- **Topic**: `screen_capture_api_and_host_ui_shielding`
- **Evidence Level**: `normative_standard`
- **URL**: https://www.w3.org/TR/screen-capture/
- **Standards**: W3C Screen Capture, W3C Permissions Policy display-capture, Android WindowManager.LayoutParams.FLAG_SECURE
- **Summary**: The W3C Screen Capture API defines navigator.mediaDevices.getDisplayMedia() for streaming user display surfaces (monitor, window, or browser tab). For mini-apps running inside a super-app, screen capture introduces major security risks such as capturing banking balances or credentials. The container must restrict displaySurface selection, enforce selfCapture preferences, and apply OS-level secure surface flags (e.g., Android FLAG_SECURE) to shield host super-app overlays from mini-app capture.
- **Key Points**:
  - getDisplayMedia mandates user-driven activation (transient user activation) and forbids automated background invocation.
  - DisplayMediaStreamConstraints allow specifying displaySurface ('browser', 'window', 'monitor') to restrict capturing broader desktop or other applications.
  - Container host overlays (payment pinpads, auth prompts, biometric gates) apply native secure window flags to render black rectangles in mini-app screen streams.
  - System audio capture must be strictly opted-in with explicit user consent to prevent recording background phone calls or media.

### Finding 480: W3C Web Audio API: AudioContext Lifecycle, AudioWorklet Sandbox Isolation, and Background Power Governance
- **ID**: `webrtc_036_04`
- **Topic**: `webaudio_api_audioworklet_and_power_governance`
- **Evidence Level**: `normative_standard`
- **URL**: https://www.w3.org/TR/webaudio/
- **Standards**: W3C Web Audio API 1.1, WHATWG HTML Audio Processing, W3C Page Lifecycle
- **Summary**: The W3C Web Audio API provides high-performance audio synthesis and digital signal processing. Inside a mini-app, AudioContext creation is constrained by autoplay policies requiring user interaction. AudioWorklet allows arbitrary DSP code to run in a dedicated audio rendering thread. The super-app container must supervise AudioContext lifecycle, immediately suspending or closing audio processing when a mini-app is hidden or backgrounded, preventing CPU exhaustion and battery drain.
- **Key Points**:
  - AudioContext must remain in 'suspended' state until explicitly triggered by a user gesture event listener, respecting mobile autoplay policies.
  - AudioWorkletNode and AudioWorkletProcessor execute audio synthesis in a separate real-time thread without access to the DOM or window context.
  - Fingerprinting mitigation restricts audio buffer precision and introduces subtle phase jitter to prevent audio hardware device identification.
  - Container background policy forces audioContext.suspend() within 500ms of mini-app entering 'hidden' state unless granted specialized background-audio entitlement.

### Finding 481: Super-App Enterprise Real-Time Media Gateway: Native WebRTC Proxying, VoIP Push, and Zero-PII Call Routing
- **ID**: `webrtc_036_05`
- **Topic**: `superapp_carrier_realtime_media_and_voip_gateway`
- **Evidence Level**: `platform_practice`
- **URL**: https://www.w3.org/TR/webrtc/
- **Standards**: W3C WebRTC 1.0, Apple PushKit Framework, Android Telecom Framework, RFC 8826 WebRTC Security Architecture
- **Summary**: Enterprise super-apps host telemedicine, live customer support, and collaborative mini-apps requiring carrier-grade voice and video. Rather than embedding independent WebRTC protocol stacks in each mini-app, the super-app native container provides a standardized JSAPI bridge delegating media encoding to hardware codecs. Incoming calls leverage native VoIP push services (Apple PushKit / Android TelecomManager) with end-to-end encrypted session signaling.
- **Key Points**:
  - Native container exposes high-level JSAPI (e.g., container.createCallSession) backed by hardware-accelerated H.264/AV1/Opus encoders.
  - Inbound call notifications wake the super-app via native VoIP push channels, presenting system-native incoming call UI before launching the mini-app.
  - Media signaling exchanges end-to-end encrypted DTLS-SRTP keys via zero-knowledge signaling relays, preventing super-app servers from eavesdropping.
  - QoS telemetry aggregates jitter, packet loss, and round-trip time metrics for store SLA auditing while redacting user IP and endpoint metadata.

### Finding 482: WICG Speculation Rules API: Declarative Prefetch & Prerender Rules, Eagerness Levels, and Resource Eviction
- **ID**: `spec_036_01`
- **Topic**: `speculation_rules_declarative_prefetch_prerender`
- **Evidence Level**: `normative_standard`
- **URL**: https://html.spec.whatwg.org/multipage/speculative-loading.html
- **Standards**: WHATWG Speculative Loading, WICG Prerendering Revamped, W3C Resource Timing
- **Summary**: The Speculation Rules API replaces legacy link rel=prefetch/prerender with an expressive, JSON-based declarative syntax (<script type="speculationrules">) embedded in HTML or injected via HTTP headers. Mini-apps use speculation rules to prefetch resources or prerender entire target pages into hidden renderer contexts. Rules specify source ('list' or 'document'), target_hint ('_self' or '_blank'), and eagerness levels ('immediate', 'eager', 'moderate', 'conservative'). Super-app containers supervise memory ceilings, dynamically aborting or discarding speculative instances on device memory pressure.
- **Key Points**:
  - Speculation rules define structured JSON objects containing 'prefetch' and 'prerender' instruction blocks with URL patterns.
  - Eagerness levels govern execution triggers: 'immediate' on DOM insertion, 'eager' on idle, 'moderate' on pointer hover/touch start, and 'conservative' on mousedown.
  - Prerendered documents execute scripts in a restricted 'prerendering' state, delaying audio, camera, or disruptive side effects until activation.
  - Container memory governor limits concurrent prerendered mini-app instances to 1 on low-RAM devices (<4GB) and 2 on high-end devices, evicting oldest on memory pressure.

### Finding 483: W3C CSS View Transitions Module Level 1: document.startViewTransition(), Snapshot Tree, and Fluid Navigation Parity
- **ID**: `spec_036_02`
- **Topic**: `view_transitions_api_and_fluid_navigation_orchestration`
- **Evidence Level**: `normative_standard`
- **URL**: https://www.w3.org/TR/css-view-transitions-1/
- **Standards**: W3C CSS View Transitions Module Level 1, W3C CSS Animations Level 2, W3C Web Animations
- **Summary**: W3C CSS View Transitions Module Level 1 standardizes seamless animated transitions between DOM states in Single-Page Applications and multi-page navigations. By calling document.startViewTransition(updateCallback), the browser captures a pseudo-element tree (::view-transition, ::view-transition-group, ::view-transition-old, ::view-transition-new) and animates changes using GPU-accelerated CSS animations. In super-app mini-apps, this delivers visual fluidity equivalent to native mobile transitions without heavy JavaScript animation frameworks.
- **Key Points**:
  - document.startViewTransition() captures outgoing DOM snapshots, executes synchronous DOM mutations, and generates incoming DOM snapshots.
  - Pseudo-element tree isolation allows independent styling, timing functions, and keyframe animations per named view-transition-name element.
  - Eliminates layout thrashing and frame drops during complex mini-app catalog-to-detail transitions, offloading rendering to the compositor thread.
  - Container enforces maximum transition timeout (default 500ms) with automatic fallback to prevent UI freezing if DOM updates hang.

### Finding 484: W3C VirtualKeyboard API: navigator.virtualKeyboard.overlaysContent and Mobile Occlusion Governance
- **ID**: `spec_036_03`
- **Topic**: `virtualkeyboard_api_and_viewport_geometry_governance`
- **Evidence Level**: `normative_standard`
- **URL**: https://w3c.github.io/virtual-keyboard/
- **Standards**: W3C VirtualKeyboard API, CSS Values and Units Module Level 4, W3C Touch Events
- **Summary**: The W3C VirtualKeyboard API provides developers fine-grained control over how on-screen virtual keyboards interact with the layout viewport. By setting navigator.virtualKeyboard.overlaysContent = true, the browser refrains from resizing the visual viewport, instead firing 'geometrychange' events and exposing navigator.virtualKeyboard.boundingRect. Mini-apps use CSS environment variables (env(keyboard-inset-top, left, width, height)) to reposition inputs and action buttons without jarring layout reflows.
- **Key Points**:
  - overlaysContent = true prevents destructive visual viewport shrinking on mobile OS, eliminating white flash and content jumping when input gains focus.
  - geometrychange event broadcasts precise keyboard bounding rectangles in CSS pixels with sub-frame timing.
  - CSS env(keyboard-inset-*) allows declarative layout adjustments for floating checkout bars and fixed CTA buttons.
  - Super-app container bridges native IME window insets on Android (WindowInsetsCompat) and iOS (UIKeyboardLayoutGuide) into the VirtualKeyboard Web API.

### Finding 485: Mobile Gesture Alignment: Synchronizing View Transitions with Native iOS Swipe-Back and Android Predictive Back
- **ID**: `spec_036_04`
- **Topic**: `gesture_alignment_predictive_back_and_view_transitions`
- **Evidence Level**: `platform_practice`
- **URL**: https://www.w3.org/TR/css-view-transitions-1/
- **Standards**: Android OnBackInvokedCallback, Apple UIKit UINavigationController interactivePopGestureRecognizer, W3C CSS View Transitions Module Level 2
- **Summary**: Super-app mini-apps must reconcile web-based routing with native mobile gesture navigation. Modern operating systems (iOS edge swipe back and Android 14+ Predictive Back) require continuous gesture progress feedback. The super-app container intercepts native gesture progress, translating finger drag velocity and distance into CSS View Transition progress or History API transitions, preventing desynchronization between native container window chrome and embedded mini-app views.
- **Key Points**:
  - Android OnBackInvokedCallback provides onBackStarted, onBackProgressed, onBackInvoked, and onBackCancelled events.
  - Super-app container maps gesture progress ratio (0.0 to 1.0) directly to active view transition animation scrubbers.
  - If user aborts edge swipe, container rolls back view transition seamlessly without firing mini-app unload or navigation lifecycle events.
  - Prevents double-back bugs where both the native container and mini-app internal router handle back navigation simultaneously.

### Finding 486: Super-App Predictive Preloader: Speculation Rules, Intent Heuristics, and Sub-50ms Perceived Launch Latency
- **ID**: `spec_036_05`
- **Topic**: `superapp_predictive_mini_app_preloader_and_intent_caching`
- **Evidence Level**: `platform_practice`
- **URL**: https://html.spec.whatwg.org/multipage/speculative-loading.html
- **Standards**: WHATWG Speculative Loading, W3C Intersection Observer, W3C Battery Status API
- **Summary**: Achieving sub-50ms cold-launch latency across large mini-app catalogs requires predictive preloading. The super-app host observes high-confidence intent signals (e.g., user hovering on a mini-app launcher icon for >150ms, viewport intersection of store promotional banners, or routine daily launch patterns). The host container dynamically injects speculation rules for the target mini-app package, pre-warming bytecode cache and renderer processes while observing strict battery and thermal constraints.
- **Key Points**:
  - Intent scoring algorithm evaluates touch trajectory, dwell time (>150ms), and time-of-day affinity to assign launch probability (0-100%).
  - Probabilities >80% trigger background package asset prefetching and metadata parsing via Speculation Rules.
  - Preloading throttles automatically when device battery is below 20%, low-power mode is enabled, or cellular data saver is active.
  - Pre-warmed renderer instances time out and are purged if not activated within 30 seconds, preventing unneeded memory retention.

### Finding 487: WICG File System Access API: File Handles, FileSystemWritableFileStream, and User Consent Workflows
- **ID**: `fs_036_01`
- **Topic**: `filesystem_access_api_handles_and_writable_streams`
- **Evidence Level**: `normative_standard`
- **URL**: https://wicg.github.io/file-system-access/
- **Standards**: WICG File System Access, W3C Streams Standard, W3C Permissions Policy
- **Summary**: The WICG File System Access API enables web applications to read and persist changes directly to files and directories on the user's device. Methods including window.showOpenFilePicker() and window.showSaveFilePicker() return FileSystemFileHandle and FileSystemDirectoryHandle instances. Modifying files utilizes FileSystemWritableFileStream for atomic chunked writes. In a super-app mini-app environment, the native container wraps this API to enforce strict path restrictions, permission lifetime limits, and user consent gates.
- **Key Points**:
  - showOpenFilePicker and showSaveFilePicker mandate transient user activation and display native OS picker dialogues.
  - FileSystemWritableFileStream creates safe atomic swap files (.crswap) during writing, preventing corruption if the stream aborts.
  - Picker options support 'types' filters with explicit MIME types and extensions, preventing executable uploads or dangerous file types.
  - Container enforces per-session handle scoping: handles granted during a mini-app session expire automatically on container closure.

### Finding 488: Native Container File System Sandboxing: Path Virtualization, Directory Traversal Defense, and Protected OS Paths
- **ID**: `fs_036_02`
- **Topic**: `container_filesystem_sandboxing_and_traversal_mitigation`
- **Evidence Level**: `platform_practice`
- **URL**: https://wicg.github.io/file-system-access/
- **Standards**: WICG File System Access, OWASP MASVS-STORAGE, Android Scoped Storage
- **Summary**: Untrusted mini-apps must never gain direct, unmediated access to host OS filesystems, super-app private app-data directories, or shared device storage. The super-app native container intercepts File System Access API invocations, virtualizing paths into isolated per-mini-app chroot sandboxes. Sensitive directories (e.g., /system, /etc, /data/data/<superapp>, ~/Library, Keychain stores) are hard-blocked by an unbypassable path deny-list, mitigating directory traversal attacks (../) and unauthorized file exfiltration.
- **Key Points**:
  - Container maps mini-app file system calls to virtual scoped roots (e.g., app_data/miniapp_{app_id}/sandbox/).
  - Path canonicalization strips symlinks, null bytes, and traversal tokens (../, ..\\) prior to OS filesystem kernel calls.
  - Host storage deny-list unconditionally rejects requests targeting super-app databases, shared preferences, auth token caches, or OS binaries.
  - Persistent directory handles require explicit multi-factor confirmation and are visible in the super-app Privacy & Security dashboard.

### Finding 489: WICG Compression Streams API: CompressionStream, DecompressionStream, and Worker Offloading
- **ID**: `fs_036_03`
- **Topic**: `compression_streams_api_and_worker_offloading`
- **Evidence Level**: `normative_standard`
- **URL**: https://wicg.github.io/compression/
- **Standards**: WICG Compression Streams, W3C Streams Standard, WHATWG Web Workers
- **Summary**: The WICG Compression Streams API provides native JavaScript stream interfaces for compressing and decompressing byte streams using standard algorithms ('gzip', 'deflate', 'deflate-raw'). Mini-apps use CompressionStream and DecompressionStream to compress network payloads, store compact cached data in IndexedDB/OPFS, and unpack remote packages. Executing compression within Dedicated Workers or Service Workers prevents UI thread blocking and eliminates massive memory spikes caused by in-memory array buffers.
- **Key Points**:
  - Native browser implementation offloads zlib/deflate processing to optimized C++/Rust engine routines, 5-10x faster than pure JS libraries.
  - Stream piping (readable.pipeThrough(new CompressionStream('gzip')).pipeTo(writable)) processes data in chunks without buffering full files in RAM.
  - Supported formats 'gzip', 'deflate', and 'deflate-raw' conform to RFC 1952, RFC 1950, and RFC 1951 standards.
  - Mandated execution in Worker threads for data streams exceeding 5MB to guarantee zero UI thread frame drops.

### Finding 490: Mini-App Streaming Document Export: Pipelining CompressionStream to FileSystemWritableFileStream
- **ID**: `fs_036_04`
- **Topic**: `streaming_document_export_and_offline_packaging`
- **Evidence Level**: `platform_practice`
- **URL**: https://wicg.github.io/compression/
- **Standards**: WICG Compression Streams, WICG File System Access, W3C Streams Standard
- **Summary**: Enterprise and e-commerce mini-apps frequently export large documents, financial reports, invoice archives, and transaction CSVs. Conventional approaches generate huge in-memory Blob instances that trigger out-of-memory crashes on resource-constrained mobile devices. By pipelining streaming document encoders through CompressionStream directly into FileSystemWritableFileStream, mini-apps stream gigabyte-scale exports with a bounded memory footprint under 16MB.
- **Key Points**:
  - Data generation yields ReadableStream chunks from data queries or incremental rendering engines.
  - Chunk pipeline: DataSource -> JSON/CSV TextEncoder -> CompressionStream('gzip') -> FileSystemWritableFileStream.
  - Memory residency remains constant throughout multi-megabyte export jobs, completely eliminating mobile container OOM crash loops.
  - Enables background export completion with user progress indication and cancelation hooks via AbortController.

### Finding 491: Super-App Safe File Exchange & Malware Quarantine Gateway: Stream Inspection and Size Quotas
- **ID**: `fs_036_05`
- **Topic**: `superapp_file_exchange_malware_quarantine_gateway`
- **Evidence Level**: `platform_practice`
- **URL**: https://wicg.github.io/file-system-access/
- **Standards**: WICG File System Access, WICG Compression Streams, OWASP Application Security Verification Standard
- **Summary**: When mini-apps import external files from user storage or peer applications, the super-app host must guard against malicious payload ingestion, script injection, and zip-bomb decompression attacks. The super-app gateway injects an inline streaming inspection filter: files are buffered in a temporary quarantine sandbox, checked against magic-byte signatures and malware hashes, subjected to strict decompression expansion ratio limits (max 10:1 ratio), and capped at a maximum file size quota (e.g., 50MB per stream).
- **Key Points**:
  - Quarantine sandbox holds incoming imported streams until virus/malware signatures and MIME-type integrity checks pass.
  - Decompression bomb mitigation tracks total uncompressed bytes against compressed stream length, aborting streams exceeding 10x ratio.
  - File metadata sanitization removes control characters, null bytes, and script tags from user-provided file names.
  - Enforces a hard ceiling of 50MB per single file write transaction and 250MB total storage allocation per mini-app tenant.

### Finding 492: WICG Network Information API: Effective Connection Types, Downlink Benchmarks, and Data-Saver Negotiation
- **ID**: `net_037_01`
- **Topic**: `network_information_api_and_connection_adaptation`
- **Evidence Level**: `normative_standard`
- **URL**: https://wicg.github.io/netinfo/
- **Standards**: WICG Network Information API, W3C Resource Hints, RFC 8942 HTTP Client Hints
- **Summary**: The WICG Network Information API standardizes client-side exposure of underlying network quality attributes via navigator.connection (NetworkInformation interface), including effectiveType ('slow-2g', '2g', '3g', '4g'), downlink, rtt, and saveData boolean flag. Mini-app runtimes consume NetworkInformation telemetry to dynamically modulate client resource loading pipelines, substituting high-resolution assets with low-bandwidth vector/CSS alternatives and throttling non-essential background network requests under degraded cellular connectivity.
- **Key Points**:
  - NetworkInformation exposes effectiveType, downlink (Mb/s estimate), rtt (round-trip time ms estimate), and change event listeners for dynamic connection tracking.
  - saveData attribute reflects system/user-level data-saver preferences, instructing mini-app asset loaders to halt speculative prefetching and autoplay media.
  - Container-level privacy defense clamps round-trip time (rtt) and downlink estimates to discrete quantization buckets to prevent network-based device fingerprinting.
  - Mini-app asset loading engine implements automated quality step-down curves (WebP/AVIF compression tiers, audio bitrates) keyed directly to effectiveType transitions.

### Finding 493: WICG Priority Hints: Declarative Resource Fetch Priorities (importance/fetchpriority) and Super-App Pipeline Arbitration
- **ID**: `net_037_02`
- **Topic**: `priority_hints_and_network_scheduling`
- **Evidence Level**: `normative_standard`
- **URL**: https://wicg.github.io/priority-hints/
- **Standards**: WICG Priority Hints, WHATWG Fetch Standard, W3C Resource Timing
- **Summary**: The WICG Priority Hints specification defines the 'fetchpriority' attribute (and legacy 'importance' attribute) across HTML elements (<script>, <link>, <img>, <iframe>) and JavaScript RequestInit objects, accepting values 'high', 'low', or 'auto'. Inside a multi-tenant super-app container, Priority Hints enable mini-apps and host containers to fine-tune the browser network scheduler, ensuring critical business RPC calls and above-the-fold interface resources bypass background logging, marketing analytics, and non-blocking subresources.
- **Key Points**:
  - fetchpriority attribute allows fine-grained priority hints ('high', 'low', 'auto') overriding default heuristics of the browser HTTP/2 and HTTP/3 multiplexing scheduler.
  - fetch(url, { priority: 'high' }) guarantees immediate stream priority for critical payment confirmations and core catalog JSON fetches.
  - Non-critical background analytics beacons and offline asset prefetching explicitly assign fetchpriority='low' to avoid bandwidth starvation on 3G/4G networks.
  - Host container inspects and arbitrates resource priorities, preventing rogue mini-apps from monopolizing network pipelines with universal 'high' priority requests.

### Finding 494: IETF RFC 8297: HTTP 103 Early Hints for Edge-Accelerated Mini-App Critical Resource Preloading
- **ID**: `net_037_03`
- **Topic**: `rfc_8297_early_hints_and_speculative_preloading`
- **Evidence Level**: `normative_standard`
- **URL**: https://datatracker.ietf.org/doc/html/rfc8297
- **Standards**: IETF RFC 8297 (HTTP 103 Early Hints), RFC 9110 HTTP Semantics, W3C Preload
- **Summary**: IETF RFC 8297 defines the HTTP 103 Early Hints informational status code, enabling edge servers and API gateways to transmit speculative Link headers containing preload and preconnect directives while the backend origin generates the final HTTP response. For super-app mini-apps, edge CDNs return 103 Early Hints with critical package dependencies, stylesheet bundles, and host API schemas, shaving 100-300ms off mobile Time to First Contentful Paint (FCP) over high-latency cellular connections.
- **Key Points**:
  - HTTP 103 status code transmits Link headers (rel=preload, rel=preconnect) prior to the terminal 200 OK header block.
  - Super-app edge gateway intercepts mini-app entry HTML/package requests and streams 103 Early Hints for common container runtime scripts and font subsets.
  - Client WebView initiates TLS handshakes and resource streaming in parallel with server-side database querying and SSR template assembly.
  - Eliminates mobile cellular connection ramp-up latency without modifying mini-app application code or cache headers.

### Finding 495: W3C Network Error Logging (NEL): Automated Client-Side Network Failure Diagnostic Telemetry
- **ID**: `net_037_04`
- **Topic**: `network_error_logging_and_resilience_reporting`
- **Evidence Level**: `normative_standard`
- **URL**: https://w3c.github.io/network-error-logging/
- **Standards**: W3C Network Error Logging, W3C Reporting API, RFC 8942 HTTP Client Hints
- **Summary**: The W3C Network Error Logging specification defines a standardized declarative framework for user agents to collect and asynchronously report network reliability and connectivity failures (DNS resolution timeouts, TCP reset, TLS certificate errors, HTTP response timeouts) directly to pre-configured reporting endpoints via the W3C Reporting API. Super-app platforms utilize NEL headers to capture edge connectivity degradation across diverse ISP and mobile carrier topologies without requiring mini-app JavaScript runtime execution.
- **Key Points**:
  - NEL HTTP response header configures sampling rate, max_age, failure_fraction, and reporting endpoint groups in conjunction with Report-To header.
  - Captures deep network-layer failures occurring before DOM execution: DNS lookup timeouts, TCP handshake rejections, TLS negotiation drops, and 5xx gateway faults.
  - User agent buffers failed request reports across offline intervals and transmits batched, compressed diagnostic payloads upon network reconnection.
  - Platform telemetry pipeline ingests NEL streams to detect regional carrier outages, edge CDN degradations, and ASN-specific routing disruptions.

### Finding 496: Super-App Adaptive Network Governance Engine: Bandwidth-Aware Quality Tiering and Resilient Offline Request Queueing
- **ID**: `net_037_05`
- **Topic**: `superapp_adaptive_network_governance_and_offline_sync`
- **Evidence Level**: `platform_practice`
- **URL**: https://wicg.github.io/netinfo/
- **Standards**: WICG Network Information API, W3C Network Error Logging, WICG Priority Hints, RFC 8297
- **Summary**: Enterprise super-apps operating in developing markets (Southeast Asia, Latin America) face fluctuating carrier bandwidth and frequent offline transitions. The super-app host container embeds an Adaptive Network Governance Engine that synthesizes WICG Network Information signals, HTTP 103 preloads, and priority hints into an automated traffic controller. When network health drops below 2G thresholds or triggers offline states, non-essential data synchronization is routed into an encrypted durable SQLite/IndexedDB queue with exponential backoff and jittered retry algorithms.
- **Key Points**:
  - Container-level connection monitor continuously computes weighted network quality scores from RTT, downlink, and NEL fault frequencies.
  - Dynamic asset quality tiering automatically substitutes WebP/AVIF assets, disables non-critical image carousels, and restricts video streams to audio-only previews under 2G/3G conditions.
  - Offline request queue serializes failed mini-app POST/PUT mutations into encrypted local persistence with cryptographic idempotency keys.
  - Reconnection manager employs exponential backoff with decorrelated jitter to prevent thundering-herd server overloads when mobile devices emerge from dead zones.

### Finding 497: W3C IndexedDB Edition 3: Multi-Store Read-Write Transactions, KeyRange Queries, and Schema Versioning
- **ID**: `idx_037_01`
- **Topic**: `indexeddb_edition_3_transactional_isolation`
- **Evidence Level**: `normative_standard`
- **URL**: https://www.w3.org/TR/IndexedDB-3/
- **Standards**: W3C Indexed Database API 3.0, WHATWG Web IDL, W3C Web Storage
- **Summary**: The W3C Indexed Database API (IndexedDB) Edition 3 defines an asynchronous transactional object store for structured client-side persistence. Mini-apps utilize IDBDatabase, IDBObjectStore, and IDBIndex to manage complex relational datasets, offline shopping catalogs, and cached user states. IndexedDB 3 introduces relaxed transaction durability hints ('default', 'strict', 'relaxed') and promises-based lifecycle ergonomics, providing high-throughput local storage without blocking the WebView rendering pipeline.
- **Key Points**:
  - IndexedDB 3 formalizes transaction durability options ('relaxed' for high-throughput batch operations, 'strict' for financial audit logs flushed to non-volatile disk).
  - Multi-store readwrite transactions allow atomic state mutations across distinct entity stores (e.g., cart, orders, inventory) with ACID isolation.
  - IDBKeyRange boundaries (bound, lowerBound, upperBound, only) support binary search indexing and efficient composite key querying.
  - Schema migration lifecycle mediated by upgradeneeded events ensures deterministic, versioned table alterations and index rebuilds.

### Finding 498: IETF RFC 6902 & RFC 7396: JSON Patch and JSON Merge Patch for Bandwidth-Efficient Offline State Deltas
- **ID**: `idx_037_02`
- **Topic**: `json_patch_and_merge_patch_state_reconciliation`
- **Evidence Level**: `normative_standard`
- **URL**: https://datatracker.ietf.org/doc/html/rfc6902
- **Standards**: IETF RFC 6902 (JSON Patch), IETF RFC 7396 (JSON Merge Patch), RFC 8259 JSON
- **Summary**: IETF RFC 6902 (JSON Patch) and IETF RFC 7396 (JSON Merge Patch) define standardized formats for describing modifications to JSON documents. Super-app mini-apps synchronizing state between client-side IndexedDB and cloud backends utilize JSON Patch operations ('add', 'remove', 'replace', 'move', 'copy', 'test') to transmit granular delta diffs instead of re-transmitting entire monolithic documents, reducing network payload sizes by 80-95% over mobile networks.
- **Key Points**:
  - RFC 6902 defines an atomic sequence of patch operations targetable by RFC 6901 JSON Pointers.
  - The 'test' operation provides optimistic concurrency control, verifying pre-conditions (e.g., document version hash) before applying mutations.
  - RFC 7396 Merge Patch provides an intuitive, lightweight alternative for partial resource updates where null values represent field deletions.
  - Mini-app offline sync engines apply remote JSON Patches directly onto local IndexedDB entities, preserving client performance and bandwidth.

### Finding 499: IETF Draft: The Idempotency-Key HTTP Header Field for Replay-Resilient Mutation State Synchronization
- **ID**: `idx_037_03`
- **Topic**: `ietf_http_idempotency_key_specification`
- **Evidence Level**: `normative_standard`
- **URL**: https://datatracker.ietf.org/doc/html/draft-ietf-httpapi-idempotency-key-header-06
- **Standards**: IETF draft-ietf-httpapi-idempotency-key-header-06, RFC 9110 HTTP Semantics, RFC 4122 UUID
- **Summary**: The IETF HTTPAPI Working Group draft specification 'The Idempotency-Key HTTP Header Field' defines a standardized HTTP request header (Idempotency-Key) used by clients to ensure that identical mutation requests (POST/PATCH) can be retried safely across transient network drops without duplicate execution on the server. Super-app mini-apps executing financial transactions, order placements, or ticket reservations generate UUIDv4 idempotency keys stored in IndexedDB before initiating network requests.
- **Key Points**:
  - Clients attach 'Idempotency-Key: <unique-token>' to mutative HTTP requests; servers cache and return the exact original response for identical keys.
  - Prevents duplicate ledger entries, double-charging, and duplicated orders during mobile network timeouts and automatic client retries.
  - Idempotency lifecycle states (in-flight, completed, failed) govern server-side processing locks and concurrent replay rejection.
  - Super-app host bridge intercepts mini-app payment and commerce mutations, enforcing cryptographically verifiable UUIDv4 idempotency headers.

### Finding 500: Client Storage Quota Arbitration: IndexedDB Persistent Storage Grants (navigator.storage.persist) and LRU Eviction Defense
- **ID**: `idx_037_04`
- **Topic**: `indexeddb_quota_arbitration_and_eviction_protection`
- **Evidence Level**: `normative_standard`
- **URL**: https://w3c.github.io/IndexedDB/
- **Standards**: WHATWG Storage Living Standard, W3C IndexedDB 3.0, W3C Permissions Policy
- **Summary**: The Storage Living Standard (WHATWG) and IndexedDB 3 specification define how browser runtimes arbitrate disk storage quotas across web origins. Under device disk space pressure, browsers automatically evict 'best-effort' storage origins. Mini-apps handling mission-critical offline workflows (field inspections, offline point-of-sale) invoke navigator.storage.persist() to request 'persistent' storage privileges, which guarantees immunity from automatic operating system background cleanup algorithms.
- **Key Points**:
  - Storage buckets are classified by the browser into 'best-effort' (subject to LRU eviction) and 'persistent' (immune to automatic cleanup).
  - navigator.storage.persist() requests persistent storage status; navigator.storage.persisted() verifies the active privilege state.
  - navigator.storage.estimate() returns accurate usage and quota limits, allowing mini-apps to monitor local disk footprint.
  - Super-app host container acts as the policy authority, automatically granting persistent status to whitelisted enterprise mini-apps while enforcing hard quotas.

### Finding 501: Super-App Two-Way Offline Synchronization Engine: IndexedDB Local-First Journaling and Conflict-Free Delta Resolution
- **ID**: `idx_037_05`
- **Topic**: `superapp_two_way_offline_state_synchronization_engine`
- **Evidence Level**: `platform_practice`
- **URL**: https://www.w3.org/TR/IndexedDB-3/
- **Standards**: W3C IndexedDB 3.0, IETF RFC 6902 JSON Patch, IETF Idempotency-Key Draft, WHATWG Web Workers
- **Summary**: Enterprise super-apps host distributed mini-apps that must function seamlessly across intermittent cellular coverage. The super-app runtime embeds an enterprise Two-Way Offline State Synchronization Engine. Mutations are immediately written to local IndexedDB transaction journals tagged with monotonic timestamps and idempotency keys, updating UI state optimistically. Upon network restoration, an asynchronous worker daemon flushes delta patches via RFC 6902 JSON Patch, employing Last-Write-Wins (LWW) or operational transformation algorithms to reconcile server conflicts.
- **Key Points**:
  - Local-first architecture commits user operations to IndexedDB transaction journals within 5ms, ensuring instant UI responsiveness.
  - Outbox pattern buffers mutation events in dedicated object stores, surviving mini-app crashes and device power cycles.
  - Delta sync engine calculates differential updates using RFC 6902 operations, minimizing cellular data transfer overhead.
  - Deterministic conflict resolution engine evaluates vector clocks or monotonic revision sequence numbers to arbitrate concurrent multi-device updates.

### Finding 502: WHATWG HTML & ECMAScript: window.onerror, unhandledrejection Events, and Container Error Scoping
- **ID**: `diag_037_01`
- **Topic**: `global_error_handling_and_unhandled_rejection_tracking`
- **Evidence Level**: `normative_standard`
- **URL**: https://html.spec.whatwg.org/multipage/webappapis.html
- **Standards**: WHATWG HTML Living Standard Web Application APIs, ECMA-262 ECMAScript Language Specification
- **Summary**: The WHATWG HTML Living Standard and ECMA-262 specifications standardize client runtime exception propagation via window.onerror (ErrorEvent) and window.onunhandledrejection (PromiseRejectionEvent). In a super-app container hosting multi-tenant mini-apps, unhandled JavaScript errors and rejected async promises must be intercepted by host container guard rails to prevent silent application freezes, white-screen hangs, and unhandled memory leaks while securely isolating error telemetry from cross-tenant leakage.
- **Key Points**:
  - window.addEventListener('error') captures synchronous runtime exceptions, yielding message, filename, lineno, colno, and error object.
  - window.addEventListener('unhandledrejection') captures unhandled Promise rejections, extracting reason and promise references.
  - Host container error interceptor wraps mini-app lifecycle hooks, mapping uncaught exceptions to structured diagnostic crash events.
  - Cross-origin script errors ('Script error.') are mitigated by enforcing CORS and proper crossorigin attributes on CDN-delivered mini-app chunks.

### Finding 503: Source Map Revision 3 Proposal: Bidirectional Stack Symbolication, Mappings VLQ Decoding, and Source Root Privacy
- **ID**: `diag_037_02`
- **Topic**: `source_map_v3_specification_and_stack_symbolication`
- **Evidence Level**: `normative_standard`
- **URL**: https://tc39.es/source-map-spec/
- **Standards**: Source Map Revision 3 Proposal (TC39), ECMA-262 Specification
- **Summary**: The Source Map Revision 3 Proposal (TC39 / source-map-spec) specifies an open format for mapping minified and bundled client-side JavaScript back to original source code files. Super-app developer consoles and crash diagnostic pipelines utilize Source Map v3 mappings (Base64 Variable-Length Quantity / VLQ encoding) to translate minified production crash stack traces into readable file paths and line numbers without exposing proprietary source maps to client devices.
- **Key Points**:
  - Source Map v3 JSON format includes version, file, sources, names, mappings (Base64-VLQ encoded strings), and sourceRoot.
  - VLQ encoding achieves compact representation of column, line, source, and symbol index deltas across large bundles.
  - Server-side crash symbolication service accepts minified error traces and decodes stack frames against securely stored source maps.
  - Super-app build toolchain strips sourceMappingURL directives from client distribution packages to prevent reverse-engineering of commercial logic.

### Finding 504: Client Runtime Health Monitoring: Performance Memory Telemetry, Long Frame Stalls, and ANR Watchdog Timers
- **ID**: `diag_037_03`
- **Topic**: `performance_memory_and_anr_watchdog_mitigation`
- **Evidence Level**: `platform_practice`
- **URL**: https://html.spec.whatwg.org/multipage/webappapis.html
- **Standards**: W3C Performance Timeline, WHATWG Web Workers, Android ApplicationExitInfo, Apple MetricKit
- **Summary**: Mobile WebView runtimes are highly vulnerable to Application Not Responding (ANR) lockups and Out-Of-Memory (OOM) kills when mini-app JavaScript loops execute unbounded computations or leak DOM references. The host container deploys an asynchronous ANR watchdog timer on a background worker thread that monitors main-thread heartbeat pings every 500ms. If the main thread fails to respond within 3000ms, the watchdog captures a thread dump and initiates a graceful container recovery flow.
- **Key Points**:
  - Watchdog timer runs in a separate DedicatedWorker or native container thread, pinging the WebView main thread at 500ms intervals.
  - Three consecutive missed heartbeats (1500ms) triggers 'severe_stutter' warnings; five missed heartbeats (2500ms) flags imminent ANR.
  - Container-level heap memory monitor samples window.performance.memory (or native WebKit/Chromium memory metrics) to catch memory leaks.
  - Upon unrecoverable main thread deadlock, container halts the hung script execution and presents user-friendly reload or fallback options.

### Finding 505: Forensic Diagnostic Instrumentation: Breadcrumb Ring Buffers, State Snapshots, and Zero-PII Crash Payloads
- **ID**: `diag_037_04`
- **Topic**: `crash_breadcrumb_and_forensic_telemetry_pipeline`
- **Evidence Level**: `platform_practice`
- **URL**: https://html.spec.whatwg.org/multipage/webappapis.html
- **Standards**: W3C Reporting API, OWASP MASVS-STORAGE, ISO/IEC 27001 Data Sanitization
- **Summary**: When a mini-app experiences an abnormal termination or crash, effective post-mortem analysis requires contextual execution history. The super-app SDK maintains an in-memory, fixed-capacity circular ring buffer (e.g., 50 items) recording recent user interactions, navigation breadcrumbs, network request status codes, and bridge invocation events. Upon crash capture, the breadcrumb trail is bundled with sanitized device metadata and dispatched to the platform telemetry gateway.
- **Key Points**:
  - Circular ring buffer records last 50 events: route changes, bridge API calls, UI clicks, and network error codes.
  - Client-side PII scrubber inspects breadcrumb data payloads, masking phone numbers, email addresses, auth tokens, and financial values.
  - Crash payload bundles stack trace, breadcrumb buffer, heap allocation snapshot, and device state (battery, network, free RAM).
  - Emergency crash telemetry uses navigator.sendBeacon or native OS background transfer service to guarantee delivery during app exit.

### Finding 506: Super-App Developer Crash Diagnostic Portal: Automated Symbolication, Anomaly Detection, and Release Gating
- **ID**: `diag_037_05`
- **Topic**: `superapp_developer_crash_diagnostics_and_symbolication_portal`
- **Evidence Level**: `platform_practice`
- **URL**: https://tc39.es/source-map-spec/
- **Standards**: Source Map Revision 3, W3C Network Error Logging, SRE Site Reliability Engineering SLO/SLI
- **Summary**: Enterprise mini-app stores require centralized developer diagnostics to ensure ecosystem stability. The super-app platform operates a Developer Crash Diagnostic Portal that ingests anonymized crash telemetry, automatically symbolicates stack traces against publisher-uploaded source maps, groups similar exceptions into fingerprint clusters, and monitors crash rate anomalies. Mini-apps exceeding established crash SLO thresholds (e.g., crash-free sessions < 99.5%) face automated rollout pauses or marketplace quarantine.
- **Key Points**:
  - Automated crash clustering algorithms group exceptions by top stack frames and normalized error messages.
  - Secure server-side symbolication pipeline resolves obfuscated function names and line numbers in real time.
  - Automated canary release gating immediately freezes OTA rollouts if crash rates spike > 0.5% above baseline within a 1-hour window.
  - Developer console provides interactive flamegraphs, breadcrumb playback, and regression impact metrics for every release version.

### Finding 507: W3C Web Neural Network API (WebNN): On-Device ML Inference, Graph Execution Sandboxing, and NPU/GPU Device Isolation
- **ID**: `ai_compute_038_01`
- **Topic**: `w3c_webnn_api_sandboxing_and_hardware_acceleration`
- **Evidence Level**: `normative_standard`
- **URL**: https://webmachinelearning.github.io/webnn/
- **Standards**: W3C Web Neural Network API (WebNN), W3C WebGPU, Permissions Policy compute-pressure
- **Summary**: The W3C Web Neural Network API (WebNN) specifies a dedicated low-level specification for hardware-accelerated on-device machine learning inference (CPU, GPU, and NPU). In a super-app container, allowing mini-apps direct access to high-performance ML hardware creates side-channel timing risks and resource starvation. The host container mediates navigator.ml.createContext() via container-level capability policies, enforcing quantized graph validation, strict tensor dimension ceilings, and compute timeout limits.
- **Key Points**:
  - WebNN navigator.ml.createContext() creates an isolated ML execution context bound to CPU, GPU, or NPU backends.
  - MLGraphBuilder constructs a validated directed acyclic graph of tensor operations prior to asynchronous compilation.
  - Super-app container enforces tensor memory caps (e.g., max 256MB per mini-app model) and prevents malicious micro-architectural timing attacks.
  - ML execution timeouts clamp continuous inference to 500ms bursts to preserve device responsiveness and prevent thermal runaway.
  - Mini-app store review verifies model manifest hashes and prohibits dynamic uncompiled binary model loading.

### Finding 508: W3C Compute Pressure API: Real-Time CPU/Thermal Pressure Telemetry, Dynamic Load Throttling, and Host Thermal Defense
- **ID**: `ai_compute_038_02`
- **Topic**: `w3c_compute_pressure_api_and_thermal_state_governance`
- **Evidence Level**: `normative_standard`
- **URL**: https://w3c.github.io/compute-pressure/
- **Standards**: W3C Compute Pressure API, Android PowerManager.OnThermalStatusChangedListener, Apple ProcessInfo.ThermalState
- **Summary**: Intensive mini-app execution (ML inference, canvas rendering, crypto) can induce severe hardware thermal throttling and degrade host OS stability. The W3C Compute Pressure API introduces PressureObserver, delivering real-time telemetry on system pressure states ('nominal', 'fair', 'serious', 'critical'). The super-app runtime integrates native OS thermal callbacks (Android PowerManager thermal listeners, Apple ProcessInfo.thermalState) to dynamically throttle mini-app frame rates, background compute, and ML model precision.
- **Key Points**:
  - PressureObserver monitors 'cpu' pressure states across 4 standardized levels: 'nominal', 'fair', 'serious', and 'critical'.
  - The Permissions Policy 'compute-pressure' gates observer creation, restricting observation to top-level active documents.
  - Super-app bridge bridges native thermal state changes into container-level degradation events (e.g. lowering canvas FPS from 60 to 30 at 'serious').
  - At 'critical' state, background execution tasks and WebNN graph dispatches are immediately suspended to allow thermal recovery.
  - Container telemetry gateway logs high-thermal incidents to identify power-inefficient mini-apps in store analytics.

### Finding 509: WebNN Graph Compilation Security: Operator Whitelisting, Zero-Division Defenses, and Micro-Architectural Timing Jitter
- **ID**: `ai_compute_038_03`
- **Topic**: `webnn_operator_validation_and_side_channel_timing_mitigation`
- **Evidence Level**: `normative_standard`
- **URL**: https://webmachinelearning.github.io/webnn/
- **Standards**: W3C WebNN §8 Safety Considerations, OWASP Client-Side Security, W3C WebGPU WGSL Safety
- **Summary**: Compiling adversarial neural network graphs directly to hardware drivers exposes underlying GPU/NPU kernels to buffer overflows, integer wraparounds, and memory corruption. The super-app WebNN runtime enforces rigorous pre-compilation AST inspection, verifying operator parameters (convolution strides, dilation, padding, pooling) against mathematical safety bounds and injecting constant-time execution guards to neutralize cache-timing side channels.
- **Key Points**:
  - Pre-compilation validator inspects MLGraphBuilder operands against strict dimension ranges, preventing integer overflow exploits.
  - Division and normalization operators are checked for epsilon zero-division guards to prevent hardware floating-point exceptions.
  - Micro-architectural side-channel mitigations coarsen execution timestamps and insert randomized jitter into graph completion promises.
  - Host WebNN execution is sandboxed within a dedicated GPU/ML utility process isolated from main WebView memory.
  - Disallowed ML operators (such as raw memory access or arbitrary kernel injection) trigger immediate compilation rejection.

### Finding 510: Cross-Platform Thermal Management Architecture: Android Thermal Status, Apple Thermal State, and Graceful Model Degradation
- **ID**: `ai_compute_038_04`
- **Topic**: `cross_platform_thermal_mitigation_and_battery_preservation`
- **Evidence Level**: `platform_practice`
- **URL**: https://developer.apple.com/documentation/foundation/processinfo/thermalstate
- **Standards**: Android PowerManager Thermal API, Apple NSProcessInfoThermalState, W3C Battery Status API
- **Summary**: Heterogeneous mobile devices feature divergent thermal management architectures. The super-app abstraction layer unifies Android 10+ PowerManager thermal listeners (THERMAL_STATUS_LIGHT through SHUTDOWN) and Apple Foundation ProcessInfo.thermalState (nominal, fair, serious, critical) into a unified cross-platform event stream. Mini-apps utilize this stream to transition between heavy neural models, lightweight quantizations, or remote cloud offloading.
- **Key Points**:
  - Unified native thermal listener maps Android 7-stage thermal status and iOS 4-stage thermal state into normalized container states.
  - When thermal state transitions to 'serious', mini-app ML inference automatically switches from FP32 to INT8 quantized models.
  - Frame-rate caps (30Hz/15Hz) are enforced at the WebView display layer to curb GPU heat generation during prolonged sessions.
  - Native host preserves device battery life by denying concurrent NPU/GPU model execution across multiple background tabs.
  - Store certification requires mini-apps with intensive compute features to provide verifiable automated fallback mechanisms.

### Finding 511: Enterprise On-Device AI Governance: Mini-App Model Registry, Dynamic Weight Auditing, and Zero-Exfiltration Isolation
- **ID**: `ai_compute_038_05`
- **Topic**: `superapp_on_device_ai_governance_and_model_registry`
- **Evidence Level**: `platform_practice`
- **URL**: https://webmachinelearning.github.io/webnn/
- **Standards**: W3C MiniApp Packaging, NIST AI Risk Management Framework (AI RMF), W3C Content Security Policy Level 3
- **Summary**: Deploying client-side AI in enterprise super-apps necessitates strict provenance, intellectual property verification, and data exfiltration defense. The super-app store operates a centralized Mini-App Model Registry where publishers register on-device model weights (ONNX, TFLite, WebNN). The host container isolates model weight storage in encrypted OPFS buckets, enforcing strict CSP connect-src boundaries to prevent inference inputs/outputs from leaking to unauthorized endpoints.
- **Key Points**:
  - Publishers must declare all on-device ML models and SHA-256 weight checksums in the mini-app manifest (app.json).
  - Host container loads model weights into origin-private encrypted storage inaccessible to other mini-apps or external apps.
  - Strict runtime CSP disallows model weight fetching from arbitrary third-party CDNs; weights must reside in verified app packages.
  - Inference pipeline privacy guard verifies that user input features (e.g. camera embeddings) are not exfiltrated via background beacons.
  - Super-app developer portal provides on-device inference benchmarking tools measuring model latency, memory peak, and battery drain.

### Finding 512: WICG Barcode Detection API: High-Performance On-Device Optical Scanning, Camera Stream Decoupling, and Native Vision Offloading
- **ID**: `vision_posture_038_01`
- **Topic**: `wicg_barcode_detection_api_hardware_acceleration`
- **Evidence Level**: `normative_standard`
- **URL**: https://developer.mozilla.org/en-US/docs/Web/API/Barcode_Detection_API
- **Standards**: WICG Shape Detection API - Barcode Detection, W3C Media Capture and Streams, Permissions Policy camera
- **Summary**: Mini-apps frequently require optical code recognition for payments, product lookup, and authentication. Embedding heavy JavaScript barcode parsers bloats bundle sizes, degrades frame rates, and introduces memory leaks. The WICG Barcode Detection API provides a standardized BarcodeDetector interface backed by platform vision engines (Android MLKit, Apple Vision framework). The super-app runtime mediates BarcodeDetector instance creation under Permissions Policy camera constraints, processing ImageBitmap or VideoFrame streams without exposing raw camera byte buffers to mini-app memory.
- **Key Points**:
  - BarcodeDetector.getSupportedFormats() enumerates supported symbologies ('qr_code', 'aztec', 'data_matrix', 'ean_13', 'code_128').
  - BarcodeDetector.detect() analyzes ImageBitmap, HTMLVideoElement, or OffscreenCanvas sources asynchronously on native background threads.
  - Raw camera frames remain within the hardware-accelerated graphics pipeline, preventing uncompressed frame dumping into JavaScript memory.
  - Permissions Policy 'camera' strictly controls BarcodeDetector instantiation, disallowing scanning inside unauthorized cross-origin iframes.
  - Processing throughput is throttled to 10 FPS to prevent battery drain while maintaining instant code recognition.

### Finding 513: W3C Device Posture API: Foldable Hardware Topology, Posture Change Events, and Adaptive Super-App Split Viewports
- **ID**: `vision_posture_038_02`
- **Topic**: `w3c_device_posture_api_foldable_governance`
- **Evidence Level**: `normative_standard`
- **URL**: https://w3c.github.io/device-posture/
- **Standards**: W3C Device Posture API, W3C Screen Orientation API, CSS Viewport Segments
- **Summary**: Next-generation foldable and dual-screen mobile devices introduce dynamic form factors (unfolded, folded, book, tent posture). The W3C Device Posture API exposes navigator.devicePosture, reporting posture types ('continuous', 'folded') and hinge orientation changes. The super-app container intercepts posture transitions to arbitrate mini-app responsive layouts, preventing UI clipping across physical screen hinges and coordinating multi-pane mini-app workflows.
- **Key Points**:
  - navigator.devicePosture.type exposes standardized postures: 'continuous' (flat display) and 'folded' (hinged display).
  - DevicePosture 'change' event dispatches real-time topology transitions without leaking fine-grained mechanical sensor angles.
  - CSS media query @media (device-posture: folded) allows declarative styling of split-pane user interfaces.
  - Super-app container maps physical hinge coordinates to safe layout insets, ensuring critical action buttons avoid the physical crease.
  - Host multi-window manager coordinates dual-pane mode, allowing mini-apps to display master-detail views across folded displays.

### Finding 514: Host-Mediated Scanner Delegation vs Embedded Vision Sandboxing: Malicious Payload Sanitization and QR Anti-Phishing Defenses
- **ID**: `vision_posture_038_03`
- **Topic**: `host_mediated_scanner_vs_embedded_vision_sandboxing`
- **Evidence Level**: `platform_practice`
- **URL**: https://wicg.github.io/shape-detection-api/
- **Standards**: WICG Shape Detection API, OWASP Mobile Top 10: Insecure Communication, WeChat Mini Program wx.scanCode
- **Summary**: Allowing mini-apps uncontrolled access to camera viewfinders creates camera hoarding and background snooping risks. The super-app standard defines a dual-mode optical architecture: (1) Host-mediated scanner intent (WeChat wx.scanCode / Alipay scan parity) that launches the trusted native super-app scanner and returns verified string payloads; (2) Embedded BarcodeDetector sandboxing for in-app AR/scanner components with mandatory on-screen active recording indicators and cryptographic destination URL pre-flight validation.
- **Key Points**:
  - Host-mediated scanner modal takes precedence for financial and authentication QR flows, isolating camera permissions from the mini-app.
  - Scanned URLs undergo host-side anti-phishing inspection and domain allowlisting before navigation or bridge dispatch.
  - Embedded BarcodeDetector usage requires active top-level document visibility and display of persistent container camera icons.
  - Malicious QR payloads containing binary exploits, CRLF injection, or unescaped URI schemes (e.g. javascript:, intent:) are sanitized.
  - Container enforces automated 60-second timeouts on continuous embedded camera scanning sessions.

### Finding 515: Foldable Screen Spanning Architecture: CSS Viewport Segments, Hinge Avoidance, and Screen Orientation Lock Governance
- **ID**: `vision_posture_038_04`
- **Topic**: `foldable_screen_spanning_and_viewport_segments_coordination`
- **Evidence Level**: `normative_standard`
- **URL**: https://w3c.github.io/device-posture/
- **Standards**: W3C Device Posture API, W3C Screen Orientation API, CSS Environmental Variables Module Level 1
- **Summary**: Foldable mobile devices present discontinuous viewports separated by physical seams. The super-app standard mandates integration between W3C Device Posture and CSS Viewport Segments environment variables (env(viewport-segment-width), env(viewport-segment-top)), enabling mini-apps to render dual-pane shopping, navigation, and document inspection views. Concurrently, the container arbitrates W3C Screen Orientation API lock requests (screen.orientation.lock()), preventing mini-apps from overriding user orientation settings during posture transitions.
- **Key Points**:
  - Container exposes CSS env(viewport-segment-*) variables representing discrete logical display segments across the hinge.
  - Mini-app UI components adopt flexbox or grid spanning models to cleanly bifurcate navigation and content panes.
  - Screen orientation lock requests (screen.orientation.lock('landscape')) require explicit host manifest capability declarations.
  - Host container suppresses orientation lock attempts when device is placed in tabletop or tent folded postures.
  - Automated store validation tests mini-app rendering across virtual foldable viewports to prevent UI overlap and unclickable zones.

### Finding 516: Super-App Payment QR & Optical Standard Conformance: EMVCo QR Parsing, VietQR Standards, and Strict Schema Enforcement
- **ID**: `vision_posture_038_05`
- **Topic**: `superapp_payment_qr_and_national_standard_conformance`
- **Evidence Level**: `platform_practice`
- **URL**: https://wicg.github.io/shape-detection-api/
- **Standards**: EMVCo QR Code Specification for Payment Systems, SBV Decision 2345/QD-NHNN, WICG Shape Detection API
- **Summary**: In enterprise super-apps (such as Southeast Asian telco and banking super-apps), optical scanning is tightly coupled to interoperable payment networks (EMVCo Merchant-Presented / Consumer-Presented QR and VietQR / NAPAS specifications). The super-app optical engine provides built-in, hardware-isolated parsing of EMVCo TLV (Tag-Length-Value) structures, verifying merchant identifier integrity, checksums (CRC16), and transaction amounts before delegating payment confirmation to the secure payment sheet.
- **Key Points**:
  - Native scanner automatically identifies and validates EMVCo Tag 00 (Format Indicator), Tag 26-51 (Merchant Account Info), and Tag 63 (CRC16).
  - VietQR and regional QR standards (NAPAS247, PromptPay, KHQR) are parsed natively in isolated container memory.
  - Mini-apps receive structured, verified payment parameters rather than unparsed raw text strings, preventing injection attacks.
  - Optical scan events are signed with host cryptographic attestation tokens before handoff to payment checkout sheets.
  - Store security review verifies that mini-apps do not attempt to bypass official payment gateways via unverified raw QR redirection.

### Finding 517: WICG WebOTP API: Origin-Bound SMS Verification, Zero-Permission Phone Number Masking, and Phishing Defenses
- **ID**: `input_a11y_038_01`
- **Topic**: `wicg_web_otp_api_and_origin_bound_sms_verification`
- **Evidence Level**: `normative_standard`
- **URL**: https://wicg.github.io/web-otp/
- **Standards**: WICG WebOTP API, W3C Credential Management API, Google Play SMS Retriever API
- **Summary**: Authenticating users via SMS one-time passwords within mini-apps traditionally requires intrusive READ_SMS native permissions or error-prone manual input. The WICG WebOTP API specifies navigator.credentials.get({otp: {transport: ['sms']}}), binding SMS verification codes directly to the origin of the super-app or mini-app. The host platform intercepts SMS messages adhering to the standardized format (@domain #123456), prompting the user with an explicit one-tap authorization sheet without granting mini-apps direct access to the device inbox.
- **Key Points**:
  - navigator.credentials.get({otp: {transport: ['sms']}}) triggers browser/host-level SMS monitoring without READ_SMS permissions.
  - SMS format requires standardized origin binding (@origin #code) to prevent cross-origin credential interception.
  - Super-app container presents a native confirmation modal showing the originating mini-app name and the masked verification code.
  - Host suppresses OTP delivery if the SMS origin tag does not match the authenticated publisher domain in the mini-app manifest.
  - AbortSignal integration provides deterministic timeout cancellation (e.g. 60-second OTP countdown timers).

### Finding 518: WICG Accessibility Object Model (AOM): Virtual Accessibility Nodes, Non-DOM Assistive Semantics, and Screen Reader Telemetry
- **ID**: `input_a11y_038_02`
- **Topic**: `wicg_accessibility_object_model_programmatic_inclusion`
- **Evidence Level**: `normative_standard`
- **URL**: https://wicg.github.io/aom/spec/
- **Standards**: WICG Accessibility Object Model, W3C WCAG 2.2 SC 4.1.2 Name, Role, Value, Android AccessibilityNodeInfo
- **Summary**: High-performance mini-apps frequently employ custom canvas rendering engines, virtualized lists, or WebGL where native HTML elements are absent, blinding screen readers and assistive technologies. The WICG Accessibility Object Model (AOM) Phase 1-4 introduces AccessibleNode and virtual accessibility trees. The super-app standard requires mini-apps utilizing non-DOM rendering to project an explicit, navigable virtual accessibility tree into the host native accessibility framework (Android AccessibilityNodeInfo / Apple UIAccessibilityElement).
- **Key Points**:
  - AOM virtual accessibility nodes project semantic labels, roles, and states for canvas-rendered UI components.
  - AccessibleNode programmatic interfaces support screen reader navigation without polluting the DOM tree.
  - Host container bridges virtual nodes into native OS accessibility services (TalkBack, VoiceOver) with accurate screen coordinates.
  - Mini-app store compliance verifies that all non-DOM interactive elements map to valid accessibility nodes.
  - Host accessibility auditor detects canvas-heavy mini-apps lacking accessibility bindings and flags them during submission review.

### Finding 519: W3C Vibration API: Haptic Feedback Orchestration, User Gesture Gating, and Hardware Abuse Prevention
- **ID**: `input_a11y_038_03`
- **Topic**: `w3c_vibration_api_haptic_feedback_governance`
- **Evidence Level**: `normative_standard`
- **URL**: https://w3c.github.io/vibration/
- **Standards**: W3C Vibration API, W3C User Activation Living Standard, Android Vibrator / CombinedVibrator API
- **Summary**: Haptic feedback significantly enriches mobile interactions, yet uncontrolled vibration can cause severe battery depletion, physical device slippage, and user annoyance. The W3C Vibration API specifies navigator.vibrate(pattern). The super-app runtime enforces strict security and ergonomic constraints: vibration calls require transient user activation, continuous vibration is clamped to maximum 200ms bursts, and background or inactive tabs are strictly prohibited from triggering haptics.
- **Key Points**:
  - navigator.vibrate() requires transient user activation (user gesture); background calls fail silently.
  - Container clamps vibration duration to maximum 200ms per pulse and caps composite patterns to 1000ms total duration.
  - Host container suppresses all haptic requests when the device is in Silent/Do Not Disturb mode or battery saver state.
  - Mini-apps must declare haptic intent in app.json; unlisted calls are ignored by the host runtime.
  - Haptic motor access is revoked immediately when the mini-app loses window focus or document visibility becomes hidden.

### Finding 520: WebOTP Enterprise Security Architecture: Cross-Site Scripting Isolation, Phishing Resistance, and Telco SIM Binding
- **ID**: `input_a11y_038_04`
- **Topic**: `webotp_phishing_mitigation_and_two_factor_auth_security`
- **Evidence Level**: `platform_practice`
- **URL**: https://developer.mozilla.org/en-US/docs/Web/API/WebOTP_API
- **Standards**: WICG WebOTP Specification §4 Security Considerations, OWASP MASVS-AUTH, SBV Circular 50/2024/TT-NHNN
- **Summary**: In enterprise super-apps with embedded banking and e-commerce mini-apps, OTP codes represent high-value authentication credentials susceptible to clickjacking, iframe phishing, and SIM swap exploits. The super-app WebOTP architecture mandates that OTP credential retrieval is only accessible to top-level secure contexts, strictly forbids credential delegation to embedded third-party iframes, and cryptographically cross-references the receiving SIM identifier with the super-app account registry.
- **Key Points**:
  - WebOTP requests are strictly restricted to top-level browsing contexts with active origin cryptographic verification.
  - Permissions Policy 'otp-credentials' defaults to 'self', completely disabling OTP retrieval in cross-origin frames.
  - Telco super-apps verify device IMSI/SIM serial against account profile to detect and block unauthorized SIM-swap OTPs.
  - OTP payload is never logged in unencrypted crash reports, telemetry ring buffers, or developer console logs.
  - Repeated OTP delivery failures (>3 within 5 minutes) trigger step-up biometric challenge via WebAuthn.

### Finding 521: Super-App Comprehensive Accessibility Auditing: Automated Screen Reader Verification, Contrast Ratios, and Store Compliance Gating
- **ID**: `input_a11y_038_05`
- **Topic**: `superapp_comprehensive_accessibility_auditing_and_wcag_conformance`
- **Evidence Level**: `platform_practice`
- **URL**: https://wicg.github.io/aom/spec/
- **Standards**: W3C WCAG 2.2 Level AA, WICG AOM, Section 508 / EN 301 549
- **Summary**: Achieving universal usability across diverse user demographics requires stringent accessibility standards across all catalog mini-apps. The super-app standard mandates an automated accessibility audit pipeline during app submission, evaluating WCAG 2.2 Level AA compliance: semantic HTML elements, AOM virtual node coverage, minimum 4.5:1 text contrast ratios, minimum 48x48dp touch targets, and assistive keyboard/focus traversal.
- **Key Points**:
  - Automated store pipeline runs axe-core and native accessibility scanners against mini-app submission packages.
  - Touch target dimensions are enforced at minimum 48x48 CSS pixels for all interactive controls.
  - Color contrast algorithms check text and interactive iconography against 4.5:1 (normal text) and 3:1 (large text) thresholds.
  - Focus trapping and keyboard navigation order are validated to ensure smooth screen reader swipe traversal.
  - Mini-apps failing Level AA baseline criteria receive actionable audit reports and cannot be published to the public catalog.

### Finding 522: W3C Accelerometer Specification: High-Frequency Motion Sandboxing, Sampling Rate Clamping, and Device Acoustic Side-Channel Mitigation
- **ID**: `motion_sensor_039_01`
- **Topic**: `w3c_generic_sensor_framework_and_accelerometer_governance`
- **Evidence Level**: `normative_standard`
- **URL**: https://w3c.github.io/accelerometer/
- **Standards**: W3C Accelerometer, W3C Generic Sensor API, Permissions Policy accelerometer
- **Summary**: Physical accelerometers report 3-axis linear acceleration (including or excluding gravity via LinearAccelerationSensor). In a super-app hosting third-party mini-apps, unrestricted high-frequency sensor readings (e.g. 100-200 Hz) expose end users to acoustic side-channel attacks, device fingerprinting, keystroke inference, and gyroscope-assisted speech eavesdropping. The W3C Generic Sensor / Accelerometer specifications mandate strict Permissions Policy integration. The super-app runtime enforces hardware-level sampling rate clamping (e.g., maximum 20-30 Hz for standard mini-apps) and automatically suspends sensor callbacks when document visibility is hidden or when the WebView loses window focus.
- **Key Points**:
  - Accelerometer and LinearAccelerationSensor expose 3-axis motion (x, y, z) in m/s^2 relative to device coordinate frames.
  - Container clamps sampling frequency (frequency parameter) to a maximum safe threshold (e.g. 20 Hz) to eliminate acoustic keylogger side channels.
  - Permissions Policy 'accelerometer' header governs access; nested cross-origin iframes receive 'none' by default.
  - Container bridges to Android SensorManager.SENSOR_DELAY_UI / iOS CoreMotion with automatic lifecycle pauses on app blur.
  - Mini-apps must explicitly declare motion sensor intent in app.json; unapproved sensor instantiations throw SecurityError.

### Finding 523: W3C Gyroscope Specification: Angular Velocity Tracking, Drift Compensation, and Rotational Privacy Boundaries
- **ID**: `motion_sensor_039_02`
- **Topic**: `w3c_gyroscope_and_rotational_velocity_governance`
- **Evidence Level**: `normative_standard`
- **URL**: https://w3c.github.io/gyroscope/
- **Standards**: W3C Gyroscope, W3C Generic Sensor API, Permissions Policy gyroscope
- **Summary**: The W3C Gyroscope API delivers real-time angular velocity around three physical axes (rad/s). While essential for interactive canvas rendering, 3D product previews, and casual gaming, high-precision gyroscope data can be combined with accelerometer data to reconstruct user touch taps and PIN entries. The super-app architecture implements strict permission gating, sensor data rounding (quantizing angular velocities to prevent micro-vibration analysis), and immediate hardware disconnection whenever background execution or overlay dialogs are active.
- **Key Points**:
  - Gyroscope interface reports angular velocity (x, y, z) in radians per second conforming to the right-hand coordinate rule.
  - Angular velocity values are quantized (rounded to 3 decimal places) to neutralize micro-vibration keystroke inference exploits.
  - Permissions Policy 'gyroscope' requires top-level active secure context and explicit user permission for high-frequency modes.
  - The super-app host container automatically unregisters native sensor listeners immediately when the mini-app enters 'hidden' or 'frozen' lifecycle states.
  - Store submission scanner flags mini-apps requesting gyroscope access without corresponding interactive UI/gameplay declarations.

### Finding 524: W3C Magnetometer Specification: Geomagnetic Field Sensing, Indoor Location Sandboxing, and Compass Privacy Isolation
- **ID**: `motion_sensor_039_03`
- **Topic**: `w3c_magnetometer_and_geomagnetic_field_sandboxing`
- **Evidence Level**: `normative_standard`
- **URL**: https://w3c.github.io/magnetometer/
- **Standards**: W3C Magnetometer, W3C Generic Sensor API, Permissions Policy magnetometer
- **Summary**: The W3C Magnetometer API measures magnetic field vectors (in microteslas) relative to local magnetic coordinates. Because magnetic field anomalies map uniquely to indoor structural steel profiles, unconstrained magnetometer access allows mini-apps to pinpoint indoor user locations without requesting coarse or fine GPS permissions. The super-app standard mandates that Magnetometer and UncalibratedMagnetometer instances require explicit runtime user consent, undergo coordinate noise injection, and are strictly prohibited in non-navigation mini-apps.
- **Key Points**:
  - Magnetometer measures magnetic flux density along x, y, and z axes in microteslas (uT).
  - Indoor geomagnetic fingerprinting is mitigated by injecting controlled gaussian noise into magnetic field measurements.
  - UncalibratedMagnetometer (exposing hard iron bias) is restricted to privileged system and indoor-navigation certified mini-apps.
  - Permissions Policy 'magnetometer' defaults to 'self' and cannot be delegated to untrusted embedded third-party scripts.
  - Sensor polling is suspended during payment dialogs, keyboard input, and authentication modals to protect PIN input privacy.

### Finding 525: W3C Orientation Sensor: Absolute vs Relative Spatial Posture, Quaternion Math, and Cross-Platform Sensor Fusion
- **ID**: `motion_sensor_039_04`
- **Topic**: `w3c_orientation_sensor_and_quaternion_coordinate_isolation`
- **Evidence Level**: `normative_standard`
- **URL**: https://w3c.github.io/orientation-sensor/
- **Standards**: W3C Orientation Sensor, W3C Generic Sensor API, Android RotationVector / Apple CMAttitude
- **Summary**: The W3C Orientation Sensor specification defines AbsoluteOrientationSensor and RelativeOrientationSensor, utilizing sensor fusion across accelerometers, gyroscopes, and magnetometers to produce 4D unit quaternions ([x, y, z, w]) and 3D rotation matrices. In enterprise super-apps (supporting AR product viewing and logistics scanning), AbsoluteOrientationSensor requires geomagnetic alignment, whereas RelativeOrientationSensor avoids magnetometer indoor tracking risks. The super-app container provides unified cross-platform sensor fusion bridges across Android AHRS and iOS CMAttitude while preventing device orientation fingerprinting.
- **Key Points**:
  - AbsoluteOrientationSensor provides orientation relative to the Earth's reference frame (East-North-Up coordinate system).
  - RelativeOrientationSensor operates without geomagnetic references, providing drift-free local rotational tracking with zero indoor location leakage.
  - Quaternions (quaternion attribute) and rotation matrices (populateMatrix()) are computed via host-level hardware fusion filters (Kalman/Madgwick).
  - Orientation updates are rate-limited to 30Hz and synchronized to requestAnimationFrame to prevent rendering stutter.
  - Super-app security policy requires mini-apps to favor RelativeOrientationSensor unless absolute compass heading is demonstrably required.

### Finding 526: W3C Ambient Light Sensor API: Environmental Illuminance Quantization, Dark Mode Automation, and Reflection Attack Prevention
- **ID**: `motion_sensor_039_05`
- **Topic**: `w3c_ambient_light_sensor_and_display_reflection_mitigation`
- **Evidence Level**: `normative_standard`
- **URL**: https://w3c.github.io/ambient-light/
- **Standards**: W3C Ambient Light Sensor, W3C Generic Sensor API §7.2 Privacy and Security, Permissions Policy ambient-light-sensor
- **Summary**: The W3C Ambient Light Sensor API provides real-time ambient illuminance measurements in LUX. While designed to allow mini-apps to adapt UI themes and contrast dynamically, high-precision ambient light sensors can measure screen reflection off nearby surfaces or user clothing, enabling malicious scripts to infer sensitive rendered content or cross-tab browsing activity. The super-app runtime quantizes LUX values into discrete categorical buckets (e.g. 'dim', 'normal', 'bright') or 50-LUX increments and enforces Permissions Policy ambient-light-sensor gating.
- **Key Points**:
  - AmbientLightSensor exposes the current illuminance level in lux via the illuminance attribute.
  - To prevent screen reflection timing attacks (reading screen luminance changes), the host quantizes illuminance readings into 50-lux steps.
  - Permissions Policy 'ambient-light-sensor' requires explicit manifest configuration; unauthorized access triggers immediate rejection.
  - Light sensor updates are restricted to a maximum frequency of 5 Hz, preventing optical communication side channels.
  - Host container intercepts ambient sensor events to manage global super-app dark/light theme switching automatically without passing raw values.

### Finding 527: W3C Pointer Events Level 3: Coalesced Micro-Jitter Sampling, Predicted Gesture Trajectories, and Pointer Capture Security
- **ID**: `input_ctrl_039_01`
- **Topic**: `w3c_pointer_events_level_3_and_gesture_sandboxing`
- **Evidence Level**: `normative_standard`
- **URL**: https://w3c.github.io/pointerevents/
- **Standards**: W3C Pointer Events Level 3, W3C UI Events, Android MotionEvent Historical Batches
- **Summary**: The W3C Pointer Events Level 3 specification unifies mouse, touch, and digital stylus inputs under a unified event model (PointerEvent). In high-performance super-app mini-apps (digital signature pads, drawing canvases, CAD tools), default event dispatch can introduce latency and missed sample points. PointerEvent.getCoalescedEvents() exposes raw hardware-rate coordinate updates delivered within a single display frame, while getPredictedEvents() utilizes device Kalman filters to extrapolate touch trajectories for zero perceived latency. The super-app container mediates pointer capture (setPointerCapture / releasePointerCapture) to prevent rogue mini-apps from locking pointer events across native navigation chrome or back gesture trigger zones.
- **Key Points**:
  - PointerEvent unifies mouse, touch, and stylus pen inputs with tiltX, tiltY, twist, tangentialPressure, and pressure attributes.
  - getCoalescedEvents() retrieves sub-frame intermediate coordinate samples, ensuring precise curves in digital signature and drawing mini-apps.
  - getPredictedEvents() allows mini-apps to render predictive stroke points ahead of actual display refresh, reducing perceived latency to near zero.
  - Container enforces strict boundaries on setPointerCapture(), releasing capture automatically if touch coordinates cross the host navigation header or gesture edge.
  - Pen/stylus hardware identifiers (pointerId) are scoped per-mini-app session to eliminate cross-app physical device fingerprinting.

### Finding 528: W3C Pointer Lock API: Relative Mouse Coordinate Tracking, Canvas Confinement, and Host Escape Gesture Guarantees
- **ID**: `input_ctrl_039_02`
- **Topic**: `w3c_pointer_lock_api_and_first_person_viewport_governance`
- **Evidence Level**: `normative_standard`
- **URL**: https://w3c.github.io/pointerlock/
- **Standards**: W3C Pointer Lock API, W3C Fullscreen API, OWASP Client-Side Security
- **Summary**: The W3C Pointer Lock API provides raw, unconstrained mouse movement delta coordinates (movementX, movementY) by hiding the cursor and locking pointer movement to a specific DOM canvas element. Essential for 3D visualization, virtual tours, and first-person simulation mini-apps, pointer lock poses severe UI entrapment risks if a malicious mini-app traps the user. The super-app standard mandates that Element.requestPointerLock() requires transient user activation, binds strictly to an active fullscreen container, and guarantees an untouchable host emergency escape hatch (physical ESC key or hardware back button) that cannot be intercepted or suppressed by mini-app scripts.
- **Key Points**:
  - requestPointerLock() locks cursor coordinates and delivers raw movementX/movementY delta streams without desktop bounds.
  - Pointer lock invocation is strictly gated on transient user activation (user click or tap) within a secure context.
  - Host container renders an un-spoofable overlay notification informing the user how to exit pointer lock.
  - Host reserve key combinations (ESC key, two-finger long-press, or device back gesture) terminate pointer lock immediately via native container dispatch.
  - Pointer lock is automatically released upon window blur, visibility change, or invocation of host payment and permission modals.

### Finding 529: W3C Input Events Level 2: BeforeInput Action Cancellation, Rich Text Editing Taxonomy, and Virtual Keyboard Sandboxing
- **ID**: `input_ctrl_039_03`
- **Topic**: `w3c_input_events_level_2_and_ime_composition_security`
- **Evidence Level**: `normative_standard`
- **URL**: https://w3c.github.io/input-events/
- **Standards**: W3C Input Events Level 2, W3C UI Events, WHATWG HTML Forms
- **Summary**: The W3C Input Events Level 2 specification enhances user input control through the beforeinput and input events, exposing fine-grained inputType classifications (such as insertText, deleteContentBackward, formatBold, insertReplacementText) and the getTargetRanges() method. In multilingual and mobile super-apps, virtual keyboard input and complex Input Method Editors (IMEs) (e.g. Vietnamese Telex, Chinese Pinyin) require reliable text modification intercepts before DOM mutation. The super-app container enforces strict inputType validation, protects sensitive form fields against covert IME composition logging, and provides seamless virtual keyboard integration.
- **Key Points**:
  - beforeinput event fires prior to DOM changes and is cancelable via event.preventDefault() for custom rich text architectures.
  - inputType taxonomy standardizes over 30 distinct editing operations across desktop, mobile, and software keyboards.
  - getTargetRanges() identifies exact DOM StaticRange boundaries intended for replacement, simplifying collaborative editor engines.
  - Container bridges IME composition lifecycle (compositionstart, compositionupdate, compositionend) to prevent broken text node states in Asian languages.
  - Sensitive input contexts (credentials, PINs, card numbers) automatically activate host-secure virtual keyboards, disabling third-party IME snooping.

### Finding 530: W3C Gamepad API & Extensions: Hardware Controller Mapping, Dual-Rumble Haptic Actuation, and Controller Privacy Boundaries
- **ID**: `input_ctrl_039_04`
- **Topic**: `w3c_gamepad_api_and_controller_haptic_actuation`
- **Evidence Level**: `normative_standard`
- **URL**: https://w3c.github.io/gamepad/
- **Standards**: W3C Gamepad API, W3C Gamepad Extensions, Permissions Policy gamepad
- **Summary**: The W3C Gamepad API and its companion Gamepad Extensions specification provide programmatic access to connected gaming controllers (Bluetooth gamepads, USB sticks, console controllers). Mini-apps query navigator.getGamepads() to read button states and analog axes mapped to the standard layout. The super-app runtime isolates connected controller metadata (masking vendorId/productId hashes to prevent cross-origin device tracking), limits polling rates to the active rendering loop, and mediates GamepadHapticActuator dual-rumble vibration to prevent excessive motor power draw and device slippage.
- **Key Points**:
  - Gamepad objects provide standard mapping for 16 digital buttons and 4 analog axes (thumbsticks and analog triggers).
  - GamepadHapticActuator.playEffect('dual-rumble') enables realistic vibration feedback with weakMagnitude and strongMagnitude parameters.
  - Host container masks raw hardware string descriptors and USB hardware IDs, returning generic 'Standard Gamepad' identifiers to prevent fingerprinting.
  - Gamepad polling via navigator.getGamepads() returns null/empty arrays unless the mini-app has received explicit user interaction and holds window focus.
  - Permissions Policy 'gamepad' restricts controller access to authorized top-level origins; embedded iframes cannot poll gamepads without manifest delegation.

### Finding 531: WICG Keyboard Map API: Physical Scancode Layout Independence, Hotkey Collision Mitigation, and Platform Safety Intercepts
- **ID**: `input_ctrl_039_05`
- **Topic**: `wicg_keyboard_map_and_gaming_navigation_governance`
- **Evidence Level**: `normative_standard`
- **URL**: https://wicg.github.io/keyboard-map/
- **Standards**: WICG Keyboard Map API, W3C UI Events (KeyboardEvent.code), W3C Keyboard Lock API
- **Summary**: On physical hardware keyboards (Bluetooth keyboards connected to tablets or desktop super-app clients), different international keyboard layouts (QWERTY, AZERTY, QWERTZ, Dvorak) map identical physical keys to conflicting character outputs. The WICG Keyboard Map API provides navigator.keyboard.getLayoutMap(), allowing mini-apps to translate physical key codes (e.g., 'KeyW', 'KeyA', 'KeyS', 'KeyD') to their localized glyphs. The super-app standard mandates that mini-apps utilize Keyboard Map for layout-independent navigation while strictly reserving host shortcut chords (Ctrl+W / Cmd+W, back navigation, app switcher) to prevent malicious mini-apps from trapping the user.
- **Key Points**:
  - navigator.keyboard.getLayoutMap() returns a promise resolving to a KeyboardLayoutMap mapping physical key codes to layout glyphs.
  - Enables layout-agnostic navigation and gaming controls (e.g. mapping WASD to ZQSD on French AZERTY layouts seamlessly).
  - Permissions Policy 'keyboard-map' requires top-level secure contexts, preventing covert key mapping inspection in cross-origin frames.
  - The super-app host container enforces an immutable deny-list of system shortcuts (e.g., Alt+F4, Cmd+Q, Esc, Host Back Key) that cannot be hijacked.
  - Store submission review audits mini-apps requesting low-level keyboard bindings to ensure compliance with host accessibility guidelines.

### Finding 532: W3C Intersection Observer: Off-Main-Thread Viewport Visibility Tracking, Ad Impression Verification, and Sub-View Sandboxing
- **ID**: `view_clip_039_01`
- **Topic**: `w3c_intersection_observer_and_viewability_telemetry`
- **Evidence Level**: `normative_standard`
- **URL**: https://w3c.github.io/IntersectionObserver/
- **Standards**: W3C Intersection Observer, IAB Viewable Ad Impression Measurement Guidelines, W3C Web Performance
- **Summary**: Tracking the visual intersection of DOM elements with the viewport traditionally required costly scroll and resize event polling on the main UI thread, causing severe scroll jank. The W3C IntersectionObserver specification enables asynchronous observation of element visibility relative to the top-level viewport or an ancestor element. In a super-app container, IntersectionObserver is foundational for lazy-loading images, virtualizing long product catalogs, and certifying ad/widget impressions without main-thread overhead. The container sandbox clamps observer thresholds and isolates root bounds within the sub-view frame.
- **Key Points**:
  - IntersectionObserver computes intersection ratios asynchronously in the browser compositor process, eliminating scroll jank.
  - threshold array and rootMargin configure precise visibility trigger boundaries for catalog items and carousel banners.
  - Super-app ad verification pipeline leverages IntersectionObserver (intersectionRatio >= 0.5 for continuous 1000ms) to certify viewable ad impressions.
  - Container boundaries enforce root isolation: mini-apps cannot configure roots outside their sandboxed WebView DOM boundary.
  - Host container pauses observer evaluation when the mini-app moves off-screen, conserving CPU cycles on battery-constrained devices.

### Finding 533: CSSWG Resize Observer: Element Content-Box Geometry Telemetry, Responsive Layout Virtualization, and Layout Loop Defense
- **ID**: `view_clip_039_02`
- **Topic**: `csswg_resize_observer_and_responsive_container_embedding`
- **Evidence Level**: `normative_standard`
- **URL**: https://drafts.csswg.org/resize-observer/
- **Standards**: CSSWG Resize Observer, W3C CSS Box Model, W3C CSS Device Adaptation
- **Summary**: Modern super-apps support dynamic split-screen multitasking, foldable viewport transformations, and embedded widget cards. The CSSWG Resize Observer specification enables mini-apps to observe element geometry changes (contentBoxSize, borderBoxSize, and devicePixelContentBoxSize) without window resize polling. In high-density canvas rendering and data visualization, ResizeObserver provides pixel-perfect backing store alignment. To preserve host stability, the super-app runtime enforces loop-limit heuristics, suppressing cascade loops when mini-app observers recursively trigger element mutations.
- **Key Points**:
  - ResizeObserver monitors element box dimensions (borderBoxSize, contentBoxSize, and devicePixelContentBoxSize) with sub-pixel precision.
  - devicePixelContentBoxSize provides exact device physical pixel dimensions, preventing canvas rendering blur and anti-aliasing artifacts on high-DPI screens.
  - The specification includes an intrinsic loop-delivery algorithm that terminates processing if notifications trigger deeper layout iterations, preventing infinite loops.
  - Host container leverages ResizeObserver on container webview frames to notify mini-apps smoothly during fold/unfold or split-screen transitions.
  - Store linting scanners detect and flag unmanaged ResizeObserver instances that fail to call disconnect() on view teardown.

### Finding 534: W3C Clipboard API: Sanitized Asynchronous Data Transfer, Transient Activation Gating, and Covert Scraping Defense
- **ID**: `view_clip_039_03`
- **Topic**: `w3c_clipboard_api_and_data_loss_prevention`
- **Evidence Level**: `normative_standard`
- **URL**: https://w3c.github.io/clipboard-apis/
- **Standards**: W3C Clipboard API and Events, Permissions Policy clipboard-read / clipboard-write, OWASP MASVS-STORAGE
- **Summary**: System clipboard access represents a critical data leakage vector on mobile devices, where clipboard contents often contain sensitive user authentication phrases, one-time verification codes, or personal messages. The W3C Clipboard API standardizes asynchronous clipboard read and write operations via navigator.clipboard.read(), readText(), write(), and writeText(). The super-app security architecture enforces strict transient user activation for all clipboard interactions, requires explicit user confirmation before allowing clipboard reads, sanitizes rich text payloads against HTML injection, and prohibits background clipboard monitoring.
- **Key Points**:
  - navigator.clipboard.readText() and read() require transient user activation (direct user click/tap) and explicit runtime user consent.
  - Permissions Policy 'clipboard-read' and 'clipboard-write' govern access, strictly forbidding background and cross-origin iframe read operations.
  - Host container displays a transient toast notification whenever a mini-app successfully accesses clipboard data, providing visual transparency.
  - Clipboard write payloads are sanitized to prevent malicious script injection, allowing only recognized MIME types (text/plain, text/html, image/png).
  - Automatic clipboard scraping upon app launch or view activation is strictly prohibited and enforced via store rejection gating.

### Finding 535: W3C MediaStream Recording API: Audio/Video Chunked Compression, Memory Ceiling Buffering, and Hardware Codec Sandboxing
- **ID**: `view_clip_039_04`
- **Topic**: `w3c_mediastream_recording_and_hardware_encoder_lifecycle`
- **Evidence Level**: `normative_standard`
- **URL**: https://w3c.github.io/mediacapture-record/
- **Standards**: W3C MediaStream Recording API, W3C Media Capture and Streams, W3C File System Access API
- **Summary**: Generating user audio messages, video reviews, or customer service recordings in super-app mini-apps requires client-side media compression. The W3C MediaStream Recording API (MediaRecorder) provides real-time encoding of MediaStream tracks into container formats (WebM, MP4). In resource-constrained mobile containers, continuous in-memory blob buffering can cause fatal Out-Of-Memory (OOM) crashes. The super-app standard mandates timeslice chunking (e.g. 1000ms intervals), offloads chunk streaming to OPFS or container file streams, enforces strict file size ceilings, and immediately terminates recording sessions when the mini-app loses foreground focus.
- **Key Points**:
  - MediaRecorder encodes live audio and video streams into standardized formats (audio/webm, video/mp4) using hardware codecs.
  - ondataavailable event delivers recorded data chunks configured via the timeslice parameter, avoiding large monolithic RAM allocations.
  - Container enforces an in-flight recording memory ceiling (e.g. max 50MB); exceeding the ceiling forces streaming flush to sandbox storage.
  - Hardware encoder sessions are automatically paused and finalized if the device camera/microphone hardware indicator is revoked.
  - Host container terminates active MediaRecorder instances immediately upon document visibility change to 'hidden' to prevent surreptitious recording.

### Finding 536: W3C Resource Timing: Fine-Grained Subresource Waterfalls, Timing-Allow-Origin Gating, and Cache-Timing Defense
- **ID**: `view_clip_039_05`
- **Topic**: `w3c_resource_timing_and_subresource_performance_auditing`
- **Evidence Level**: `normative_standard`
- **URL**: https://w3c.github.io/resource-timing/
- **Standards**: W3C Resource Timing Level 2, W3C High Resolution Time Level 3, W3C Performance Timeline
- **Summary**: Optimizing mini-app load performance and identifying slow CDN endpoints requires granular network telemetry. The W3C Resource Timing specification exposes detailed network timestamps (DNS lookup, TCP handshake, TLS negotiation, request start, response start, response end) via PerformanceResourceTiming objects. In multi-tenant super-apps, cross-origin resource timings can expose sensitive user browsing history via cache timing side channels. The container enforces Timing-Allow-Origin (TAO) validation, coarsens clock resolution, and integrates resource timing metrics into centralized store performance telemetry.
- **Key Points**:
  - PerformanceResourceTiming provides network phase timings (domainLookup, connect, secureConnection, responseEnd) and size attributes (transferSize, encodedBodySize).
  - Cross-origin subresources must serve the 'Timing-Allow-Origin' HTTP header; without it, detailed timing phases are zeroed to prevent history sniffing.
  - Container high-resolution clock (performance.now()) is coarsened to 5 microseconds to mitigate micro-architectural Spectre and cache-timing attacks.
  - Resource timing buffer size is capped (setResourceTimingBufferSize(250)) to prevent memory leaks from long-running mini-app sessions.
  - Aggregated subresource load latencies are sampled and transmitted to the super-app observability gateway to detect degraded third-party CDNs.

### Finding 537: WICG Prioritized Task Scheduling: scheduler.postTask Priority Levels, Main-Thread Orchestration, and Microtask Gating
- **ID**: `task_sched_040_01`
- **Topic**: `wicg_prioritized_task_scheduling_and_posttask`
- **Evidence Level**: `normative_standard`
- **URL**: https://wicg.github.io/scheduling-apis/
- **Standards**: WICG Prioritized Task Scheduling, W3C Long Tasks API 1.0, HTML Living Standard Event Loop
- **Summary**: In complex mini-apps and super-app host runtimes, uncoordinated script execution on the main thread leads to long tasks, input delay, and frame drops. The WICG Prioritized Task Scheduling API standardizes 'scheduler.postTask' with three distinct priority levels ('user-blocking', 'user-visible', and 'background'). The super-app mini-app container exposes this scheduling primitive to decouple latency-sensitive user interactions (such as touch gestures and UI animations) from asynchronous data processing, logging, and non-critical DOM updates, ensuring consistent 60fps/120fps responsiveness.
- **Key Points**:
  - scheduler.postTask() schedules asynchronous tasks according to explicit priorities: 'user-blocking' (highest, immediate UI rendering), 'user-visible' (default, rendering tasks), and 'background' (lowest, background sync, non-critical telemetry).
  - Returns a Promise resolving with the return value of the scheduled task callback, integrating seamlessly with async/await patterns.
  - Enables mini-apps to break monolithic initializations into prioritized slices without starving the browser UI event loop.
  - Container injects scheduler polyfills or native bridges for older WebView runtimes while binding default background tasks to idle slots.
  - Store performance audit profile measures INP (Interaction to Next Paint) and flags mini-apps that run long tasks (>50ms) within 'user-blocking' queues.

### Finding 538: TaskController and TaskSignal: Dynamic Task Prioritization, Priority Inheritance, and Coordinated Abort Signaling
- **ID**: `task_sched_040_02`
- **Topic**: `taskcontroller_and_dynamic_priority_cancellation`
- **Evidence Level**: `normative_standard`
- **URL**: https://developer.mozilla.org/en-US/docs/Web/API/Scheduler/postTask
- **Standards**: WICG Prioritized Task Scheduling, WHATWG DOM Standard AbortController, WICG Page Lifecycle
- **Summary**: Mini-app user journeys frequently change state before pending asynchronous tasks complete (e.g., navigating away from a catalog tab while data fetching and image decoding are in progress). The WICG Scheduling API defines 'TaskController' (extending AbortController) and 'TaskSignal' to support dynamic priority mutation ('controller.setPriority()') and graceful abort signaling. The super-app container utilizes TaskController to propagate lifecycle-driven priority shifts, automatically downgrading or aborting background tasks when a mini-app is hidden or suspended.
- **Key Points**:
  - TaskController allows dynamic runtime adjustments to task priority via 'setPriority()' without rescheduling or recreating promises.
  - TaskSignal dispatches 'prioritychange' events, allowing worker threads and internal task queues to reorder pending operations dynamically.
  - Cooperative task aborting is supported via 'signal.aborted' and 'signal.throwIfAborted()', instantly terminating obsolete rendering or data pipelines.
  - Container automatically triggers abort on non-critical TaskSignals when the mini-app receives lifecycle unload or background freeze events.
  - Allows developer frameworks (React, Vue, MiniProgram DSL) to align component unmount lifecycle with pending asynchronous task disposal.

### Finding 539: Cooperative Task Chunking via scheduler.yield(): Event Loop Yielding with Preserved Priority and Continuation Context
- **ID**: `task_sched_040_03`
- **Topic**: `cooperative_yielding_via_scheduler_yield`
- **Evidence Level**: `normative_standard`
- **URL**: https://developer.chrome.com/blog/use-scheduler-yield
- **Standards**: WICG Scheduling APIs (Yield and Continuation), Prioritized Task Scheduling API, HTML Event Loop Processing Model
- **Summary**: Breaking up CPU-intensive computations (such as complex JSON parsing, local search indexing, or canvas scene setup) traditionally relied on setTimeout(0) or requestAnimationFrame, which introduce unpredictable queuing delays and priority inversion. The 'scheduler.yield()' API provides an ergonomic awaitable mechanism that yields control back to the browser event loop for rendering and input handling, and immediately resumes the task continuation with its original priority intact.
- **Key Points**:
  - 'await scheduler.yield()' temporarily pauses task execution, allowing the browser to process queued user input and paint intermediate frames.
  - Unlike setTimeout(0), scheduler.yield() preserves the caller's execution priority and places the continuation at the head of the task queue rather than behind unrelated third-party macrotasks.
  - Eliminates priority inversion where low-priority setTimeout callbacks interleave ahead of critical UI continuation work.
  - Crucial for mini-app boot sequences: heavy component hydration can yield periodically to guarantee first-input latency <16ms.
  - Super-app developer guidelines mandate scheduler.yield() chunking for any synchronous algorithm exceeding 15ms execution time.

### Finding 540: TC39 ECMAScript Atomics: SharedArrayBuffer Synchronization, Lock-Free Queues, and Low-Latency Worker Concurrency
- **ID**: `task_sched_040_04`
- **Topic**: `tc39_atomics_and_sharedarraybuffer_concurrency`
- **Evidence Level**: `normative_standard`
- **URL**: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Atomics
- **Standards**: ECMAScript 2026 (ECMA-262) Atomics, SharedArrayBuffer Living Standard, W3C Web Workers
- **Summary**: High-performance mini-apps (such as game engines, audio synthesizers, and real-time data visualizers) require multi-threaded computation without serialization overhead. The TC39 ECMAScript standard defines the 'Atomics' object and 'SharedArrayBuffer' to enable atomic operations (add, and, compareExchange, load, store, wait, notify) across dedicated Web Workers. The super-app host container enforces cross-origin isolation and memory boundaries to permit safe shared-memory concurrency while mitigating thread race conditions.
- **Key Points**:
  - Atomics methods (Atomics.compareExchange, Atomics.load, Atomics.store) provide sequential consistency and memory fencing for shared TypedArrays.
  - Atomics.wait() and Atomics.notify() implement thread suspension and signaling, enabling condition variables and lock-free ring buffers between Workers.
  - Atomics.wait() is forbidden on the main thread to prevent UI freezing and ANR (Application Not Responding) timeouts; it is strictly restricted to DedicatedWorker contexts.
  - Super-app sandbox allocates dedicated SharedArrayBuffer memory pools capped per mini-app instance (e.g. 64MB ceiling) to prevent host memory exhaustion.
  - Store code scanner analyzes WebAssembly and Worker script bundles to verify safe memory alignment and validate that shared buffers are properly released on mini-app teardown.

### Finding 541: Worker Thread-Pool Governance: Cross-Origin Isolation Enforcement (COOP/COEP) and Thread Watchdog Deadlock Defense
- **ID**: `task_sched_040_05`
- **Topic**: `cross_origin_isolation_and_worker_concurrency_watchdog`
- **Evidence Level**: `normative_standard`
- **URL**: https://tc39.es/ecma262/
- **Standards**: HTML Living Standard Cross-Origin Isolation, W3C Web Workers, OWASP Mobile Application Security
- **Summary**: SharedArrayBuffer and high-resolution concurrency APIs expose high-precision timers that could be exploited by Spectre-class side-channel attacks unless strict isolation is maintained. The super-app container enforces Cross-Origin Opener Policy ('same-origin') and Cross-Origin Embedder Policy ('require-corp'), alongside hardware concurrency throttling and a worker watchdog timer. This ensures that multi-threaded mini-apps cannot compromise neighboring containers, deadlock the host, or deplete mobile CPU core resources.
- **Key Points**:
  - Cross-Origin Isolation via 'Cross-Origin-Opener-Policy: same-origin' and 'Cross-Origin-Embedder-Policy: require-corp' is mandatory for SharedArrayBuffer availability.
  - Container clamps 'navigator.hardwareConcurrency' exposed to mini-app workers (e.g. capped at 2-4 cores on mobile devices) to prevent background CPU starvation.
  - Host container runtime maintains an external watchdog thread that monitors Worker heartbeat signals and terminates deadlocked or runaway Worker instances after 5 seconds of unresponsiveness.
  - Mini-apps must declare 'multi_thread_compute' capability in manifest; unauthorized worker instantiations are intercepted and throttled.
  - Worker memory consumption is tracked in real-time, enforcing automatic garbage collection and context destruction upon mini-app blur or termination.

### Finding 542: W3C Fetch Metadata Request Headers: Sec-Fetch-* Classification and Origin-Bound API Gateway Isolation
- **ID**: `fetch_meta_040_01`
- **Topic**: `w3c_fetch_metadata_request_headers_and_gateway_isolation`
- **Evidence Level**: `normative_standard`
- **URL**: https://www.w3.org/TR/fetch-metadata/
- **Standards**: W3C Fetch Metadata Request Headers, IETF RFC 9110 HTTP Semantics, OWASP Cross-Site Request Forgery Prevention Cheat Sheet
- **Summary**: In a super-app ecosystem hosting hundreds of heterogeneous mini-apps, host backend APIs and partner services are vulnerable to Cross-Site Request Forgery (CSRF), Cross-Site Script Inclusion (XSSI), and XS-Leaks. The W3C Fetch Metadata Request Headers specification standardizes four browser-enforced headers: 'Sec-Fetch-Site', 'Sec-Fetch-Mode', 'Sec-Fetch-Dest', and 'Sec-Fetch-User'. Because these are forbidden request headers that JavaScript cannot tamper with, the super-app API gateway uses them to deterministically distinguish same-origin, same-site, and cross-site requests and reject unauthorized requests before business logic execution.
- **Key Points**:
  - 'Sec-Fetch-Site' indicates origin relationship: 'same-origin', 'same-site', 'cross-site', or 'none' (user-initiated direct navigation).
  - 'Sec-Fetch-Mode' specifies request mode: 'cors', 'no-cors', 'navigate', 'websocket', or 'same-origin'.
  - 'Sec-Fetch-Dest' reveals resource destination: 'empty' (fetch/XHR), 'image', 'script', 'document', 'worker', etc.
  - 'Sec-Fetch-User' indicates whether a navigation request was triggered by explicit user gesture (present with '?1' or absent).
  - Super-app edge API gateway enforces an upfront filter: requests with 'Sec-Fetch-Site: cross-site' targeted at sensitive state-changing endpoints (payments, account profile, voucher redemption) are rejected with HTTP 403 Forbidden.

### Finding 543: Fetch Metadata Resource Isolation Policy: Neutralizing CSRF, XSSI, and Cross-Site Leakage Across Mini-App Contexts
- **ID**: `fetch_meta_040_02`
- **Topic**: `fetch_metadata_defense_in_depth_and_xs_leaks_mitigation`
- **Evidence Level**: `normative_standard`
- **URL**: https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Fetch_metadata
- **Standards**: W3C Fetch Metadata Request Headers, Fetch Living Standard, OWASP Web Security Testing Guide
- **Summary**: Mini-apps executing within WebViews or iframes could attempt to probe host internal APIs or exfiltrate private user state by embedding protected endpoints into script or image tags. Implementing standard Fetch Metadata Resource Isolation Policies allows super-app backend servers to safely serve resources while blocking cross-site probing attacks. The policy permits top-level navigations (allowing users to open legitimate deep links) while blocking unauthorized subresource embeds and cross-site API fetches.
- **Key Points**:
  - Deterministic allowlisting rule: Allow all requests with 'Sec-Fetch-Site: same-origin' and 'Sec-Fetch-Site: none'.
  - Allow top-level navigation: Requests with 'Sec-Fetch-Mode: navigate' and HTTP GET are permitted to enable user links and redirects.
  - Cross-site subresource blocking: Requests with 'Sec-Fetch-Site: cross-site' and destinations like 'script', 'image', or 'empty' against stateful APIs are dropped immediately.
  - Provides robust defense against XS-Leaks, search query sniffing, and timing side channels without relying exclusively on samesite cookie attributes.
  - Responses varying on Fetch Metadata headers include 'Vary: Sec-Fetch-Site, Sec-Fetch-Mode, Sec-Fetch-Dest' to prevent HTTP cache poisoning across distinct origins.

### Finding 544: Cross-Origin Resource Policy (CORP): 'same-origin', 'same-site', and 'cross-origin' Protection for Mini-App Assets and APIs
- **ID**: `fetch_meta_040_03`
- **Topic**: `cross_origin_resource_policy_corp_governance`
- **Evidence Level**: `normative_standard`
- **URL**: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Cross-Origin-Resource-Policy
- **Standards**: Fetch Living Standard Cross-Origin Resource Policy, W3C Cross-Origin Embedder Policy (COEP), OWASP Embedded Application Security
- **Summary**: While Cross-Origin Resource Sharing (CORS) relaxes the same-origin policy for JavaScript reads, it does not prevent speculative execution attacks or unauthorized resource loading via tags like <img> or <script>. The 'Cross-Origin-Resource-Policy' (CORP) response header enables super-app servers, CDN storage, and mini-app backends to instruct the browser or container to block cross-origin reads before data reaches the renderer process, preventing side-channel data exfiltration (Spectre).
- **Key Points**:
  - 'Cross-Origin-Resource-Policy: same-origin' restricts resource consumption strictly to the origin serving the response, preventing cross-tenant mini-app embeds.
  - 'Cross-Origin-Resource-Policy: same-site' permits subdomains within the super-app umbrella (e.g. *.superapp.com) while blocking external third-party origins.
  - 'Cross-Origin-Resource-Policy: cross-origin' explicitly authorizes shared public assets (e.g. standard UI icon sets, open fonts) across all mini-app instances.
  - CORP is a prerequisite for Cross-Origin Embedder Policy ('COEP: require-corp'), ensuring that all embedded subresources explicitly opt into being loaded in an isolated environment.
  - Store release gating scans developer CDN headers and mandates 'Cross-Origin-Resource-Policy: same-origin' on all authenticated API endpoints and sensitive user data downloads.

### Finding 545: Content-Security-Policy-Report-Only: Zero-Downtime Security Hardening, Audit Telemetry, and Policy Verification
- **ID**: `fetch_meta_040_04`
- **Topic**: `csp_report_only_and_gradual_security_policy_rollout`
- **Evidence Level**: `normative_standard`
- **URL**: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy-Report-Only
- **Standards**: W3C Content Security Policy Level 3, W3C Reporting API, OWASP Content Security Policy Guide
- **Summary**: Enforcing strict Content Security Policies (CSP) in an existing mini-app catalog risks breaking legitimate third-party functionalities if directives are misconfigured. The 'Content-Security-Policy-Report-Only' header provides an observational auditing mechanism. It evaluates incoming resources, scripts, and network connections against the target policy and emits structured violation reports without blocking resource execution. This allows super-app platform operators to validate policy compliance across thousands of mini-apps prior to strict enforcement.
- **Key Points**:
  - 'Content-Security-Policy-Report-Only' executes policy checks, captures violations, and sends report payloads to designated reporting endpoints without interrupting user execution.
  - Supports dual-header deployment: strict baseline enforced via 'Content-Security-Policy', alongside experimental stricter rules (e.g. deprecating unsafe-eval or restricting frame-ancestors) via 'Report-Only'.
  - Violation reports capture comprehensive metadata: document-uri, blocked-uri, violated-directive, original-policy, and disposition ('report').
  - Store onboarding pipeline runs developer mini-apps in Report-Only mode during sandbox QA, aggregating violation logs to generate developer feedback reports.
  - Super-app telemetry gateway deduplicates and ingests report-only events to establish baseline compatibility metrics before promoting policies to active enforcement.

### Finding 546: W3C Reporting API: Asynchronous Client Violation Delivery (Reporting-Endpoints, report-to) and Unified Observability
- **ID**: `fetch_meta_040_05`
- **Topic**: `w3c_reporting_api_and_client_violation_telemetry`
- **Evidence Level**: `normative_standard`
- **URL**: https://www.w3.org/TR/reporting-1/
- **Standards**: W3C Reporting API, W3C Network Error Logging (NEL), W3C Content Security Policy Level 3
- **Summary**: Super-app operations require unified real-time visibility into client-side security violations, deprecations, crashes, and network anomalies across heterogeneous mini-apps. The W3C Reporting API defines a standardized framework for browsers to collect and asynchronously transmit structured JSON reports to platform endpoints via the 'Reporting-Endpoints' HTTP response header. This decouples error telemetry collection from user-facing network pipelines and standardizes compliance auditing across the super-app store.
- **Key Points**:
  - 'Reporting-Endpoints: default="https://telemetry.superapp.com/reports"' establishes designated secure ingest targets for client-side diagnostic data.
  - Browser batches reports (CSP violations, COOP/COEP isolation failures, Crash/Deprecation notices) and delivers them out-of-band via HTTP POST with Content-Type 'application/reports+json'.
  - Out-of-band delivery prevents performance degradation and ensures violation reports are dispatched even when the originating page or mini-app crashes or navigates away.
  - Container injects ReportingObserver interface, enabling container-level JavaScript handlers to monitor violations locally for immediate defensive sandboxing.
  - Store telemetry pipeline aggregates reporting-endpoint streams to calculate mini-app security compliance scores and trigger automated developer warnings.

### Finding 547: IETF RFC 9162: Certificate Transparency Version 2.0 Architecture, Public Merkle Tree Auditing, and CA Surveillance
- **ID**: `cert_trans_040_01`
- **Topic**: `ietf_rfc_9162_certificate_transparency_v2_architecture`
- **Evidence Level**: `normative_standard`
- **URL**: https://www.rfc-editor.org/info/rfc9162/
- **Standards**: IETF RFC 9162 (Certificate Transparency Version 2.0), CA/Browser Forum Baseline Requirements, IETF RFC 8446 (TLS 1.3)
- **Summary**: Transport Layer Security (TLS) trust in a super-app ecosystem relies on Public Key Infrastructure (PKI). Compromised or rogue Certification Authorities (CAs) could issue fraudulent certificates for super-app host domains or partner mini-app APIs. IETF RFC 9162 defines Certificate Transparency (CT) Version 2.0, establishing an append-only, publicly verifiable Merkle tree protocol for recording all issued TLS certificates. The super-app host container enforces CT verification to detect and prevent unauthorized certificate issuance and man-in-the-middle (MitM) interception across all network egress points.
- **Key Points**:
  - RFC 9162 obsoletes RFC 6962, introducing cryptographic hash/signature agility, OID-identified log registries, and standardized TransItem data structures.
  - CAs must submit certificate pre-issuance records to multiple independent append-only CT logs, which return Signed Certificate Timestamps (SCTs) as cryptographic proof of publication.
  - Merkle tree design provides efficient cryptographic proofs: logarithmic inclusion proofs verify a certificate is in the log; consistency proofs verify the log has not truncated or altered history.
  - Super-app host container validates that TLS certificates presented by core gateway domains contain valid SCTs from at least two diverse, recognized CT logs.
  - Continuous automated CT log monitors alert super-app security teams within minutes if a public CA issues any unauthorized certificate for '*.superapp.com' or associated domains.

### Finding 548: Signed Certificate Timestamp (SCT) Ingestion: TransItem Structure, X.509v3 Extensions, and OCSP Stapling Delivery
- **ID**: `cert_trans_040_02`
- **Topic**: `sct_formats_transitem_and_x509_v3_encapsulation`
- **Evidence Level**: `normative_standard`
- **URL**: https://datatracker.ietf.org/doc/rfc9162/
- **Standards**: IETF RFC 9162, IETF RFC 5280 (X.509 PKI), IETF RFC 6066 (TLS Extensions)
- **Summary**: A Signed Certificate Timestamp (SCT) is an explicit cryptographic promise from a CT log operator that a certificate will be added to the Merkle tree within a maximum merge delay (MMD, typically 24 hours). RFC 9162 standardizes the TransItem structure to encapsulate SCTs, precertificates, and tree heads. In super-app deployments, SCTs are delivered to the client container via X.509v3 certificate extensions, TLS handshake extensions ('transparency_info'), or OCSP stapling, eliminating real-time DNS lookups during mobile TLS handshakes.
- **Key Points**:
  - TransItem structure replaces CT 1.0 MerkleTreeLeaf and SignedCertificateTimestampList, providing a unified extensible syntax for timestamps, tree heads, and inclusion proofs.
  - Three delivery mechanisms: embedded directly in the leaf certificate as an X.509v3 extension (OID 1.3.6.1.4.1.11129.2.4.2); stapled via OCSP response; or exchanged via the TLS transparency_info extension.
  - Embedded X.509v3 extension is the preferred mobile pattern, requiring zero additional network roundtrips and zero client overhead during mobile connection setup.
  - SCT payload binds log ID, timestamp, cryptographic signature algorithm (e.g. ECDSA with SHA-256 / Ed25519), and log signature over the TBSCertificate.
  - Mini-app backend validation engines require that all external HTTPS endpoints registered by third-party developers deliver at least two valid SCTs.

### Finding 549: Cryptographic Proof Verification: Merkle Tree Inclusion Proofs, Consistency Proofs, and STH Auditability
- **ID**: `cert_trans_040_03`
- **Topic**: `merkle_inclusion_and_consistency_verification_algorithms`
- **Evidence Level**: `normative_standard`
- **URL**: https://datatracker.ietf.org/doc/html/rfc9162
- **Standards**: IETF RFC 9162, NIST FIPS 180-4 (Secure Hash Standard), IETF RFC 6962
- **Summary**: RFC 9162 specifies precise cryptographic algorithms for verifying that a certificate has been incorporated into a CT log's Signed Tree Head (STH). Clients and super-app audit proxies can request inclusion proofs ('get-all-by-hash') and verify that the calculated leaf hash matches the authenticated root hash using logarithmic SHA-256 tree path traversals. Consistency proofs verify that older tree states are strictly preserved in newer tree versions, preventing split-world attacks where a rogue log presents divergent views to different clients.
- **Key Points**:
  - Inclusion proof verification computes the path from a leaf hash (TransItem) to the root hash in O(log N) operations, verifying certificate inclusion against an authenticated STH.
  - Consistency proof verification mathematically confirms that a CT log is strictly append-only: an STH of size M is a faithful prefix of a subsequent STH of size N (where N > M).
  - Super-app background network monitors periodically query CT log 'get-sth' endpoints to audit root signatures and maintain cached state of validated STHs.
  - Detects 'split-world' attacks where an adversarial log attempts to present a malicious certificate view exclusively to specific super-app mobile clients.
  - Client container logs any CT verification failure as a critical security event and triggers immediate circuit breaking for that domain.

### Finding 550: Super-App Container TLS Trust Anchoring: Certificate Pinning vs CT Enforcement and MitM Defense
- **ID**: `cert_trans_040_04`
- **Topic**: `tls_trust_anchoring_and_public_key_pinning_governance`
- **Evidence Level**: `normative_standard`
- **URL**: https://www.rfc-editor.org/info/rfc9162/
- **Standards**: IETF RFC 9162, OWASP Mobile Application Security Verification Standard (MASVS-NETWORK), Android Network Security Config / Apple App Transport Security (ATS)
- **Summary**: Securing mini-app network communication against hostile network environments (public Wi-Fi, intercepted cellular networks, government-mandated enterprise proxies) requires robust trust anchoring. While static Public Key Pinning (HPKP) has been deprecated due to risk of operational bricking and key loss, modern super-apps combine dynamic platform trust anchoring with mandatory Certificate Transparency validation and backup root pinsets. This architecture guarantees MitM defense while retaining operational agility.
- **Key Points**:
  - Super-app native container configures Android Network Security Config and iOS ATS to enforce custom trust anchors, disabling trust in user-added CA certificates for internal endpoints.
  - For host core endpoints (*.superapp.com), container enforces public key pinning with mandatory backup pins and a 30-day pin expiration policy to prevent self-bricking.
  - For third-party mini-app domain connections, container enforces strict Certificate Transparency verification rather than brittle key pinning.
  - Guarantees that third-party mini-app developers can rotate TLS certificates and CAs without requiring super-app client app updates.
  - Store submission verification suite tests developer API hostnames, flagging any endpoint lacking valid CT SCTs or serving self-signed/untrusted certificates.

### Finding 551: TLS Certificate Lifecycle Governance: OCSP Must-Staple, Automated ACME Protocols, and Revocation Verification
- **ID**: `cert_trans_040_05`
- **Topic**: `certificate_lifecycle_and_automated_revocation_governance`
- **Evidence Level**: `normative_standard`
- **URL**: https://datatracker.ietf.org/doc/rfc9162/
- **Standards**: IETF RFC 8555 (ACME), IETF RFC 6960 (OCSP), IETF RFC 7633 (TLS Feature Extension - Must-Staple)
- **Summary**: Operational security in a multi-tenant mini-app store mandates automated certificate lifecycles and real-time revocation checking. Conventional CRL and standard OCSP checks introduce significant latency and fail-open privacy leaks. Super-app network governance enforces OCSP Stapling (with X.509v3 Must-Staple extension), automated certificate renewal via ACME (RFC 8555), and short certificate lifespans (e.g. 90-day validity). This eliminates connection latency overhead and ensures rapid revocation propagation.
- **Key Points**:
  - OCSP Stapling requires the origin web server to query the CA responder periodically and attach the digitally signed OCSP response directly to the TLS handshake.
  - X.509v3 'TLS Feature' extension (Must-Staple, RFC 7633) mandates that clients abort the connection if a valid OCSP staple is omitted, neutralizing CA compromise and MitM downgrade attacks.
  - Mini-app backend operators are required to implement ACME (RFC 8555) automated certificate renewals to prevent expired certificate outages.
  - Super-app API gateway monitors certificate expiry across all registered mini-app domains and sends automated warnings at 30, 14, and 3 days before expiration.
  - Store compliance policy automatically suspends public catalog visibility for any mini-app whose registered API origin certificates have expired or been revoked.

### Finding 552: IETF WPack Bundled HTTP Responses: Binary CBOR Packaging, Subresource Aggregation, and Offline Multi-Resource Bundling
- **ID**: `webbundle_041_01`
- **Topic**: `ietf_wpack_bundled_responses_architecture`
- **Evidence Level**: `normative_standard`
- **URL**: https://datatracker.ietf.org/doc/draft-ietf-wpack-bundled-responses/
- **Standards**: IETF draft-ietf-wpack-bundled-responses, RFC 8949 (CBOR), W3C Web App Packaging
- **Summary**: Mini-app distribution requires packaging dozens or hundreds of HTML, JS, CSS, and asset subresources into a compact, atomic bundle without the latency overhead of hundreds of HTTP roundtrips. The IETF Web Packaging (WPack) Bundled HTTP Responses specification (draft-ietf-wpack-bundled-responses) standardizes an efficient binary CBOR (RFC 8949) encapsulation format for groups of HTTP exchanges. In a super-app architecture, mini-app packages can be serialized directly as Web Bundles (.wbn), enabling random-access indexing to individual resources without extracting the entire archive to local flash storage.
- **Key Points**:
  - WPack Bundled Responses encapsulates multiple HTTP request/response pairs into a single binary payload using deterministic CBOR encoding.
  - Includes a top-level section index enabling O(1) random-access offset retrieval of individual subresources without full-archive decompression.
  - Eliminates zip file vulnerability patterns (Zip Slip directory traversal) through strict URL path canonicalization and binary offset tables.
  - Native super-app container intercepts WebView resource requests and serves responses directly from memory-mapped Web Bundles.
  - Store packaging pipeline validates bundle integrity and verifies that every embedded HTTP response contains mandatory security headers (CSP, COEP).

### Finding 553: Web Packaging Binary Index Table: Random-Access Subresource Loading and Zero-Extraction Memory Mapping
- **ID**: `webbundle_041_02`
- **Topic**: `webbundle_index_section_and_random_access`
- **Evidence Level**: `normative_standard`
- **URL**: https://datatracker.ietf.org/doc/html/draft-ietf-wpack-bundled-responses-01
- **Standards**: IETF draft-ietf-wpack-bundled-responses, RFC 8949, IEEE Std 1003.1 (POSIX mmap)
- **Summary**: Traditional mini-app packaging formats (such as standard ZIP or JAR containers) require either unpacking all files to disk upon installation (causing write wear and slow launch times) or traversing a linear compression stream. Draft-ietf-wpack-bundled-responses structures the bundle into discrete sections: magic bytes, section index, primary-url metadata, responses section, and payload chunks. The super-app runtime leverages OS memory mapping (mmap) on the bundle file, resolving asset URLs via binary search in the index section and eliminating filesystem I/O during navigation.
- **Key Points**:
  - Section index maps resource URLs to byte offsets and lengths within the responses payload block.
  - Enables zero-extraction execution: mobile container opens the bundle with POSIX mmap, reading only the bytecode/CSS needed for initial render.
  - Reduces mini-app cold-start launch latency by 40-60% compared to legacy zip-decompression architectures.
  - Guarantees atomic package updates: a new version bundle is validated in temporary storage before swapping the file descriptor.
  - Container enforces maximum bundle size limits (e.g., 8MB baseline, 20MB for rich games) in the store automated ingestion gate.

### Finding 554: WICG Isolated Web Apps (IWA) & Signed Web Bundles: Origin Derivation from Ed25519 Public Keys and Tamper-Proof Storage
- **ID**: `webbundle_041_03`
- **Topic**: `isolated_web_apps_and_signed_web_bundles`
- **Evidence Level**: `consortium_specification`
- **URL**: https://datatracker.ietf.org/doc/draft-ietf-wpack-bundled-responses/
- **Standards**: WICG Isolated Web Apps, WICG Web App Packaging, RFC 8032 (Ed25519)
- **Summary**: To provide absolute security isolation for high-privilege applications (such as banking widgets and biometric authentication), WICG Isolated Web Apps utilizes Signed Web Bundles. Rather than trusting arbitrary remote DNS origins, an IWA derives its cryptographic origin directly from the Ed25519 public key that signed the bundle (e.g. 'isolated-app://<base32-encoded-pubkey>'). The super-app host container enforces this cryptographic origin isolation, ensuring that third-party mini-apps cannot tamper with or spoof the origin, and that all local storage (OPFS, IndexedDB) is hardware-bound to the signed public key.
- **Key Points**:
  - Signed Web Bundles bind an integrity block containing developer signatures and certificates over the bundle payload.
  - Isolated Web Apps assign a synthetic, tamper-proof scheme ('isolated-app://') where the origin host is the Base32-encoded public key.
  - Neutralizes DNS hijacking, BGP poisoning, and hostile Wi-Fi MitM attacks, as the code cannot be altered without invalidating the cryptographic signature.
  - Local storage partitions (IndexedDB, Cookies, OPFS) are mathematically partitioned by the signature key.
  - Super-app developer portal signs verified mini-app releases with host-approved enterprise hardware security keys (HSMs).

### Finding 555: HTTP Range Request Delivery and Block-Level Differential Streaming for Web Bundles
- **ID**: `webbundle_041_04`
- **Topic**: `webbundle_differential_updates_and_range_requests`
- **Evidence Level**: `normative_standard`
- **URL**: https://datatracker.ietf.org/doc/html/draft-ietf-wpack-bundled-responses-01
- **Standards**: RFC 9110 (HTTP Semantics §14), IETF draft-ietf-wpack-bundled-responses, RFC 7233
- **Summary**: Distributing frequent mini-app updates over cellular connections requires efficient network utilization. Because WPack Bundles are indexed with deterministic offsets, the client super-app can perform byte-range HTTP requests (RFC 9110 Range / 206 Partial Content) to download only updated asset blocks or fetch lazily loaded sub-packages on demand. Combined with server-side range caching, super-apps reduce mobile data consumption and background update overhead across millions of client devices.
- **Key Points**:
  - Client downloads the bundle header and index section (typically first 4-8 KB) to inspect package metadata and subresource manifests.
  - On-demand feature sub-packages are fetched via HTTP 206 Partial Content range requests without re-downloading existing media assets.
  - CDN edge servers cache byte-ranges, providing ultra-low latency subresource fulfillment for mobile endpoints.
  - Fallback mechanism gracefully downgrades to full-bundle streaming download if intermediate proxies strip Range headers.
  - Integrity checks compute rolling SHA-256 hashes across downloaded byte ranges against the signed package manifest.

### Finding 556: Strict CSP and Execution Boundaries for Bundled HTTP Exchanges in WebView Containers
- **ID**: `webbundle_041_05`
- **Topic**: `webbundle_content_security_policy_enforcement`
- **Evidence Level**: `normative_standard`
- **URL**: https://datatracker.ietf.org/doc/draft-ietf-wpack-bundled-responses/
- **Standards**: W3C Content Security Policy Level 3, IETF draft-ietf-wpack-bundled-responses, OWASP MASVS-CODE
- **Summary**: Packaging web exchanges into an offline bundle creates unique security boundaries. Draft-ietf-wpack-bundled-responses mandates that synthetic responses inside the bundle strictly honor origin-level security boundaries. In a super-app container, any HTML resource loaded from a Web Bundle must enforce a restrictive Content-Security-Policy (CSP) that prevents unauthorized outbound data leaks and forbids inline script evaluation ('unsafe-inline') unless explicitly attested by the store's code analysis pipeline.
- **Key Points**:
  - Embedded HTTP response headers inside the bundle define the execution policy for each individual document exchange.
  - Container enforces that the root index.html response contains 'Content-Security-Policy: default-src self; object-src none; base-uri none'.
  - Disallows bundle responses from attempting to override origin-level storage access or spoof external cross-origin domains.
  - Automated store ingestion linter scans Web Bundle CBOR structures, rejecting bundles containing unauthorized network endpoints.
  - Ensures strict parity between offline Web Bundle execution and live HTTPS mini-app runtime governance.

### Finding 557: IETF RFC 9334: Remote Attestation Procedures (RATS) Architecture and Cryptographic Device Evidence in Super-Apps
- **ID**: `rats_attest_041_01`
- **Topic**: `ietf_rfc_9334_rats_architecture_for_superapp`
- **Evidence Level**: `normative_standard`
- **URL**: https://www.rfc-editor.org/info/rfc9334
- **Standards**: IETF RFC 9334 (RATS Architecture), TCG TPM 2.0, ISO/IEC 11889
- **Summary**: In high-trust super-app ecosystems (hosting financial leasing, banking, payment, and enterprise identity mini-apps), the backend server must verify whether the client hardware and container runtime are operating in an authentic, untampered environment. IETF RFC 9334 establishes the universal Remote Attestation Procedures (RATS) architecture, defining the interaction roles between Attester (the mobile device hardware/Keystore/Secure Enclave), Relying Party (super-app server or mini-app backend), Verifier, and Endorsement Authority. Cryptographic Evidence collected from hardware roots of trust allows the super-app to cryptographically prove device integrity before releasing sensitive customer assets.
- **Key Points**:
  - RATS separates the attestation topology into Attester (mobile client), Relying Party (super-app gateway), Verifier (attestation evaluator), and Endorsement Authority.
  - Attester produces Cryptographic Evidence (claims regarding hardware state, boot chain, OS version, and app signature) signed by an Attestation Key.
  - Verifier applies Appraisals using Appraisal Policy to transform raw Evidence into verified Attestation Results.
  - Protects high-value mini-app operations (leasing contract sign-off, biometric disbursement) against rooted devices, OS tampering, and emulator attacks.
  - Standardizes conceptual data flows, enabling unified backend verification across heterogeneous mobile operating systems.

### Finding 558: Google Play Integrity API: Verdict Decryption, Device Integrity Guarantees, and App Licensing Checks
- **ID**: `rats_attest_041_02`
- **Topic**: `google_play_integrity_api_token_lifecycle`
- **Evidence Level**: `platform_standard`
- **URL**: https://developer.android.com/google/play/integrity
- **Standards**: Google Play Integrity API, Android Hardware-Backed Keystore, IETF RFC 9334
- **Summary**: For Android super-app distributions, hardware-backed environment integrity is provided by Google Play Integrity API. The client mobile SDK requests an integrity token by binding a cryptographic server-side nonce (preventing replay attacks). The super-app backend decrypts the AES-GCM encrypted integrity payload (or calls Google Play servers), evaluating three core integrity verdicts: appIntegrity (package name and certificate SHA-256), deviceIntegrity (MEETS_STRONG_INTEGRITY backed by hardware TEE vs MEETS_DEVICE_INTEGRITY), and accountDetails (licensed user status).
- **Key Points**:
  - Integrity request binds a cryptographically random, time-bounded server nonce to prevent token replay and replay interception.
  - deviceIntegrity evaluation levels: MEETS_BASIC_INTEGRITY (basic OS checks), MEETS_DEVICE_INTEGRITY (passes CTS/Play Protect), MEETS_STRONG_INTEGRITY (hardware-backed Keystore/TEE boot verification).
  - appIntegrity verdict strictly verifies appRecognitionVerdict: 'PLAY_RECOGNIZED', matching authorized release certificate hashes.
  - Enterprise financial and leasing mini-apps require MEETS_STRONG_INTEGRITY; transactions originating from emulators or modified bootloaders are rejected.
  - Super-app gateway caches integrity session tokens for up to 15 minutes to minimize backend roundtrip overhead while maintaining zero-trust posture.

### Finding 559: Apple App Attest Service (DeviceCheck): Secure Enclave Key Attestation and Cryptographic Assertion Generation
- **ID**: `rats_attest_041_03`
- **Topic**: `apple_app_attest_hardware_trust_and_assertion`
- **Evidence Level**: `platform_standard`
- **URL**: https://developer.apple.com/documentation/devicecheck/validating-apps-that-connect-to-your-server
- **Standards**: Apple App Attest Service, Apple Secure Enclave Architecture, IETF RFC 9334
- **Summary**: On iOS client runtimes, Apple DeviceCheck's DCAppAttestService provides hardware-backed proof that a client connection originates from an authentic, unmodified instance of the super-app executing on legitimate Apple hardware. The container calls generateKey() within the Secure Enclave, followed by attestKey() which submits the key and a server nonce to Apple's attestation servers. For subsequent sensitive transactions, the client generates cryptographic assertions (signData()), binding the payload with a monotonically increasing counter to prevent replay and transaction tampering.
- **Key Points**:
  - DCAppAttestService instantiates an asymmetric key pair inside the iOS Secure Enclave that cannot be extracted or exported.
  - Apple attestation certificate chain (x5c) verifies that the key was created by Apple hardware and belongs to the designated App ID.
  - Assertion objects contain a CBOR-encoded authenticator data block, signature, and an incremental counter that strictly prevents replay attacks.
  - Super-app backend verifies the counter monotonically increases on every API call, detecting and dropping cloned or replayed request signatures.
  - Required baseline for enterprise mini-apps executing financial transactions, customer PII updates, or digital signature operations.

### Finding 560: Super-App Unified Remote Attestation Bridge and Dynamic Risk Scoring Engine
- **ID**: `rats_attest_041_04`
- **Topic**: `unified_remote_attestation_bridge_and_risk_engine`
- **Evidence Level**: `industry_practice`
- **URL**: https://datatracker.ietf.org/doc/rfc9334/
- **Standards**: IETF RFC 9334, IETF RFC 7519 (JWT), IETF RFC 7517 (JWK)
- **Summary**: Because third-party mini-app developers should not be required to maintain complex native Android Keystore and iOS Secure Enclave verification pipelines independently, the super-app core architecture exposes a Unified Remote Attestation Bridge (JSAPI `superapp.security.getAttestationAssertion()`). The super-app host container evaluates native platform integrity, passes signed claims through an automated backend risk engine, and issues a standardized, short-lived JWT assertion that third-party mini-app backends can verify using the super-app's published JWKS.
- **Key Points**:
  - Exposes high-level JSAPI abstraction decoupling third-party mini-app developers from native Android Play Integrity and iOS App Attest SDK complexities.
  - Attestation tokens are signed by the super-app's central security service with RS256/ES256, referencing public JWKS at '/.well-known/jwks.json'.
  - Claims include hardwareSecurityLevel ('strongbox_tee', 'secure_enclave', 'basic'), integrityRiskScore (0-100), and clientInstanceId.
  - Mini-app backend sets risk tolerance thresholds: financial checkouts require score < 20 and hardwareSecurityLevel != 'basic'.
  - Reduces mini-app integration time from weeks of cryptographic engineering to a single standard JWT verification.

### Finding 561: Mini-App Store Anti-Tampering, Emulator Defense, and Runtime Application Self-Protection (RASP)
- **ID**: `rats_attest_041_05`
- **Topic**: `app_store_anti_tampering_and_runtime_integrity_rules`
- **Evidence Level**: `industry_practice`
- **URL**: https://developer.apple.com/documentation/devicecheck/preparing-to-use-the-app-attest-service
- **Standards**: OWASP MASVS-RESILIENCE, OWASP Mobile Security Testing Guide (MSTG), IETF RFC 9334
- **Summary**: To prevent unauthorized reverse engineering, automated bot attacks, and fraudulent automated checkout scripts within the mini-app store ecosystem, the host application implements Runtime Application Self-Protection (RASP) conforming to OWASP MASVS-RESILIENCE standards. The container continuously monitors for debuggers, dynamic hooking frameworks (such as Frida and Xposed), repackaged binaries, and rooted/jailbroken environments, cutting off sensitive mini-app execution upon threat detection.
- **Key Points**:
  - Continuous runtime checks detect ptrace attachment, Java/native function hooking (Frida/Substrate), and unauthorized debugging flags.
  - Memory integrity verification ensures mini-app bytecode and JS bundles have not been patched or intercepted in volatile RAM.
  - Detection of virtualized sandbox environments (e.g. QEMU emulators, multi-account clone spaces) flags suspicious bot-farm activity.
  - On confirmed compromise, container terminates mini-app JS execution, clears volatile cryptographic session keys, and logs security telemetry.
  - Store submission verification suite tests submitted mini-app packages for known obfuscation and anti-tampering compatibility.

### Finding 562: W3C Verifiable Credentials Data Model v2.0: Cryptographic Claims, Holder-Issuer-Verifier Triangle, and Privacy Envelopes
- **ID**: `vc_did_041_01`
- **Topic**: `w3c_verifiable_credentials_data_model_v2_0`
- **Evidence Level**: `normative_standard`
- **URL**: https://www.w3.org/TR/vc-data-model-2.0/
- **Standards**: W3C Verifiable Credentials Data Model v2.0, ISO/IEC 18013-5 (mDL), W3C Decentralized Identifiers (DIDs) v1.0
- **Summary**: In enterprise and regulated super-apps (e.g. financial leasing, telecommunications, government services), mini-apps frequently require authenticated user identity attributes (such as verified employment, credit rating scores, national ID, or driver's license status) without receiving raw permanent PII identifiers. The W3C Verifiable Credentials Data Model v2.0 defines a standard JSON/JSON-LD data model for cryptographically tamper-evident, privacy-respecting digital credentials. The super-app host acts as the secure digital wallet/Holder, allowing users to present cryptographic proofs to third-party mini-apps without centralized tracking.
- **Key Points**:
  - Establishes formal cryptographic data model: Issuer asserts claims about a Subject, held in a digital Wallet by the Holder, and presented to a Verifier.
  - Cryptographic proof mechanisms (JSON Web Signatures / Ed25519Signature2020) verify credential integrity and issuer authenticity.
  - Eliminates unnecessary PII transmission: supports zero-knowledge predicates (e.g., proving 'age >= 18' without disclosing exact birth date).
  - Super-app host container isolates credential private keys inside Android Keystore / Apple Secure Enclave.
  - Mini-apps must declare required VC claim schemas in their manifest, enabling user consent dialogs before presenting credentials.

### Finding 563: W3C Decentralized Identifiers (DIDs) v1.0: URI Architecture, DID Documents, and Cryptographic Verification Relationships
- **ID**: `vc_did_041_02`
- **Topic**: `w3c_decentralized_identifiers_did_core_v1_0`
- **Evidence Level**: `normative_standard`
- **URL**: https://www.w3.org/TR/did-core/
- **Standards**: W3C DID Core v1.0, IETF RFC 3986 (URI Generic Syntax), W3C Verifiable Credentials Data Model v2.0
- **Summary**: Decentralized Identifiers (DIDs) provide globally unique, cryptographically verifiable persistent identifiers that do not require a centralized registration authority or directory service. W3C DID Core v1.0 specifies the syntax and resolution architecture of DID documents containing public keys, authentication suites, and service endpoints. In a super-app ecosystem, pairwise pseudonymous DIDs (e.g. 'did:key' or 'did:ion') are generated per mini-app, preventing third-party mini-app developers from colluding to track users across distinct services.
- **Key Points**:
  - DID URI syntax ('did:<method>:<method-specific-id>') binds a cryptographic identifier to a resolved DID Document.
  - DID Document specifies verificationMethods (public keys) and verification relationships (authentication, assertionMethod, keyAgreement).
  - Super-app client generates unique pairwise DIDs (did:key) for each installed mini-app, preventing cross-tenant tracking and profile linkage.
  - Decentralized resolution enables offline credential verification without pinging super-app identity servers.
  - Provides foundation for portable mini-app publisher identities and cryptographically signed application manifests.

### Finding 564: IETF Selective Disclosure for JWTs (SD-JWT VC): Field-Level Disclosure, Cryptographic Salt Digests, and PII Minimization
- **ID**: `vc_did_041_03`
- **Topic**: `ietf_oauth_selective_disclosure_sd_jwt_vc`
- **Evidence Level**: `normative_standard`
- **URL**: https://datatracker.ietf.org/doc/draft-ietf-oauth-sd-jwt-vc/
- **Standards**: IETF draft-ietf-oauth-sd-jwt-vc, IETF RFC 7519 (JWT), IETF RFC 7515 (JWS)
- **Summary**: Conventional JWT credentials require sharing the entire JSON payload, leaking irrelevant personal details to verifiers. The IETF draft-ietf-oauth-sd-jwt-vc specification standardizes Selective Disclosure for JWTs (SD-JWT). In an SD-JWT, individual credential claims are salted, hashed (e.g. SHA-256), and replaced in the signed JWT body with cryptographic digests ('_sd'). When a mini-app requests user verification (such as confirming corporate leasing authorization), the super-app wallet releases only the specific disclosure strings authorized by the user, mathematically proving authenticity while concealing private salary or address fields.
- **Key Points**:
  - SD-JWT allows an issuer to issue credentials where any subset of claims can be selectively disclosed by the holder to a verifier.
  - Claims are serialized as salt-value pairs and represented in the issuer-signed JWT as SHA-256 digest arrays ('_sd').
  - Holder binds disclosures to a Verifier-specific nonce and key binding JWT (KB-JWT) to prevent token theft and replay.
  - Super-app presentation UI displays granular checkmarks, allowing the end user to review and uncheck optional claims before dispatch.
  - Drastically reduces GDPR and personal data protection compliance liabilities for both the super-app and third-party mini-apps.

### Finding 565: OpenID for Verifiable Presentations (OID4VP): Digital Credential Exchange and Presentation Definition Queries
- **ID**: `vc_did_041_04`
- **Topic**: `openid4vp_presentation_exchange_in_superapp`
- **Evidence Level**: `consortium_specification`
- **URL**: https://www.w3.org/TR/vc-data-model-2.0/
- **Standards**: OpenID for Verifiable Presentations (OID4VP), DIF Presentation Exchange 2.0, W3C VC Data Model v2.0
- **Summary**: To exchange Verifiable Credentials seamlessly between third-party mini-apps and the host super-app wallet, the ecosystem adopts the OpenID for Verifiable Presentations (OID4VP) protocol. When a mini-app requires verification, it generates a Presentation Definition query specifying the required input descriptors, schema formats (VC v2.0 or SD-JWT), and acceptable trusted issuer DIDs. The super-app host evaluates the query, requests explicit biometric authorization from the user, and securely returns the cryptographic Verifiable Presentation to the mini-app via native bridge callbacks.
- **Key Points**:
  - OID4VP defines standard HTTP and inter-app protocols for requesting and presenting verifiable credentials.
  - Presentation Definition specifies cryptographic filtering constraints (e.g. schema URI, trusted issuer public keys, required attributes).
  - Super-app host container displays a standardized, system-level credential presentation dialog preventing phishing attacks by untrusted mini-apps.
  - Biometric authorization (FaceID / Fingerprint) confirms user intent before the wallet generates the cryptographic presentation token.
  - Returned Verifiable Presentation includes holder proof-of-possession signatures, neutralizing intercepted credential reuse.

### Finding 566: Publisher Identity Governance: Cryptographically Signed Mini-App Packages via DIDs and Trust Registries
- **ID**: `vc_did_041_05`
- **Topic**: `decentralized_mini_app_publisher_governance`
- **Evidence Level**: `industry_practice`
- **URL**: https://www.w3.org/TR/did-core/
- **Standards**: W3C DID Core v1.0, W3C Verifiable Credentials Data Model v2.0, SLSA Level 3
- **Summary**: Ensuring the provenance and authenticity of mini-apps published to a decentralized or multi-operator super-app store requires verifiable publisher identities. By binding mini-app developer organizational credentials to W3C DIDs and registering them in an immutable Trust Registry, super-app operators can mathematically verify developer legal standing, business licenses, and security certifications. Furthermore, mini-app packages are signed using the publisher's DID verification keys, ensuring end-to-end cryptographic traceability from store build to mobile execution.
- **Key Points**:
  - Mini-app developers register organizational identities linked to verified enterprise DIDs backed by legal entity credentials.
  - Store submission gateway validates developer digital signatures against trusted federation registries before queuing packages for review.
  - Automated certificate revocation: compromised developer keys are revoked across the ecosystem in real-time via DID document updates.
  - Super-app store UI displays verified publisher badges (e.g., 'Verified Enterprise Issuer') backed by cryptographic proofs.
  - Eliminates publisher impersonation, clone apps, and fraudulent developer account creation across multi-tenant super-app ecosystems.
