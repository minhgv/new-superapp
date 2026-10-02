# Iteration 2 — Launch, Offline Resilience and Trust Controls

This section preserves all 36 validated findings added in Deli Deep iteration 2. Source facts are separated from design proposals inside each finding; vendor/platform and jurisdiction-specific evidence is not treated as a universal rule. Every URL below returned HTTP 200 during parent recheck on 2026-10-02. Credentials from the input sheet were not accessed or copied.

## A. Interoperable discovery and launch contracts

### 1. `manifest_identity_scope_and_deep_launch` — official-normative
- **Source:** W3C Web Application Manifest Working Draft (13 August 2026)
- **Topic:** `app_identity_scope_launch_context`
- **Detail:** [NORMATIVE FACT] The Web App Manifest defines `id` as a URL same-origin with `start_url`, used by user agents to identify an application; processing removes the id fragment. `scope` is the navigation scope, its processing removes query and fragment, and a declared scope is rejected if `start_url` is outside it. `start_url` is advisory and may be changed by the user agent or user. When an application context is created from a deep link, the specification requires immediate navigation to that deep link with history replacement; otherwise it navigates to the start URL. [STORE PROPOSAL] Require a stable manifest_id/app_id pair, same-origin id/start_url, an explicit scope containing every approved entrypoint, and a versioned deep-link allowlist. Treat id changes as a new app or an explicit migration; never use display name or mutable start_url as identity; record raw and validated launch targets before dispatch.
- **URL(s):** [https://www.w3.org/TR/appmanifest/](https://www.w3.org/TR/appmanifest/)

### 2. `url_parsing_and_origin_boundary` — official-normative
- **Source:** WHATWG URL Standard
- **Topic:** `url_canonicalization_launch_integrity`
- **Detail:** [NORMATIVE FACT] The URL Standard defines interoperable parsing and serialization; URL equality compares serialized URLs, with fragment exclusion optional. It says specifications should prefer the origin concept for security decisions and warns that naive string-prefix comparisons do not establish URL-path containment. [STORE PROPOSAL] Use one WHATWG-compatible parser/serializer for catalog, manifest, and launch inputs; retain the original input plus canonical serialization; authorize by parsed scheme/host/port and whole path segments, not string prefixes; allow only explicitly registered HTTPS origins for public launches and apply an app-specific allowlist to query and fragment data.
- **URL(s):** [https://url.spec.whatwg.org/](https://url.spec.whatwg.org/)

### 3. `uri_normalization_profile` — official-normative
- **Source:** IETF RFC 3986 URI Generic Syntax
- **Topic:** `canonical_urls_uri_equivalence`
- **Detail:** [NORMATIVE FACT] RFC 3986 defines a comparison ladder and distinguishes syntax-, scheme-, and protocol-based normalization. Scheme and host are case-insensitive and may be normalized to lowercase; percent-encoded octets for unreserved characters may be decoded, while other generic components are assumed case-sensitive unless the scheme defines otherwise. [STORE PROPOSAL] Publish a versioned canonicalization profile instead of lowercasing or decoding every URL. Keep raw URL, parsed components, canonical key, and normalization profile; use canonical keys for deduplication only after applying the profile, and preserve query semantics unless the app contract explicitly defines them.
- **URL(s):** [https://www.rfc-editor.org/rfc/rfc3986.html](https://www.rfc-editor.org/rfc/rfc3986.html)

### 4. `canonical_listing_aliases_not_identity` — proposal
- **Source:** IETF RFC 6596 The Canonical Link Relation
- **Topic:** `canonical_urls_discovery_identity`
- **Detail:** [SOURCE STATUS] RFC 6596 is an IETF Informational RFC, not an Internet Standards Track specification. [NORMATIVE FACT IN THE RFC] The `canonical` link relation designates a preferred IRI among resources with duplicative content; its target MUST identify duplicative or superset content and may be relative, self-referential, or on a different hostname. [STORE PROPOSAL] Treat rel=canonical as a discovery/deduplication and display hint, never as proof of publisher ownership, package identity, or release authority. Store canonical_url separately from aliases and require domain/app association or publisher verification before trusting a launch target.
- **URL(s):** [https://www.rfc-editor.org/rfc/rfc6596.html](https://www.rfc-editor.org/rfc/rfc6596.html)

### 5. `well_known_discovery_contract` — official-normative
- **Source:** IETF RFC 8615 Well-Known URIs
- **Topic:** `well_known_discovery_domain_association`
- **Detail:** [NORMATIVE FACT] RFC 8615 reserves the `/.well-known/` path prefix for registered well-known URIs on schemes that support them. New names MUST be registered, use a segment-nz name, and reference a specification for the returned format and media type; RFC 8615 does not itself define the hostname-selection rule, metadata scope, format, or media type. [STORE PROPOSAL] If the mini-app ecosystem needs machine discovery, define a separately versioned association/manifest resource under a precisely named well-known path, with HTTPS, exact-host checking, bounded caching, expiry, and integrity metadata. Do not treat presence of any well-known file alone as publisher ownership.
- **URL(s):** [https://www.rfc-editor.org/rfc/rfc8615.html](https://www.rfc-editor.org/rfc/rfc8615.html)

### 6. `claimed_https_callback_binding` — official-normative
- **Source:** IETF RFC 8252 OAuth 2.0 for Native Apps
- **Topic:** `launch_context_auth_integrity`
- **Detail:** [NORMATIVE FACT] RFC 8252 is an IETF Best Current Practice: native-app authorization requests use an external user-agent; claimed HTTPS redirect URIs can provide operating-system ownership proof and SHOULD be preferred where possible. It recommends high-entropy state for cross-app request-forgery protection and requires storing and exactly matching the redirect URI used by each authorization session; PKCE mitigates intercepted authorization codes. [STORE PROPOSAL] Use claimed HTTPS/verified-domain callbacks for privileged mini-app authentication, not unowned custom schemes; bind app_id, issuer, redirect URI, state/nonce, and session to one transaction; reject mismatches, replay, and tokens embedded in launch URLs, and return authorization codes for redemption inside the host.
- **URL(s):** [https://www.rfc-editor.org/rfc/rfc8252.html](https://www.rfc-editor.org/rfc/rfc8252.html)

### 7. `android_app_links_domain_proof` — primary-platform
- **Source:** Android Developers — About App Links
- **Topic:** `android_deep_links_domain_verification`
- **Detail:** [PLATFORM FACT] Android App Links use an intent filter with `android:autoVerify="true"`; Android retrieves the website's `assetlinks.json`, verifies the association and the app signing certificate fingerprint, and routes matching HTTP links directly to the app. If the app is absent, the same HTTP URL opens website content. [STORE PROPOSAL] A listing's Android launch contract should include exact HTTPS hosts/paths, package name, signing-certificate fingerprints, and verification evidence per release. The host should dispatch only after verification and retain the canonical web fallback URL for users or platforms without the app.
- **URL(s):** [https://developer.android.com/training/app-links/about](https://developer.android.com/training/app-links/about)

### 8. `android_https_over_custom_scheme` — primary-platform
- **Source:** Android Developers — Create deep links
- **Topic:** `android_deep_link_identity_fallback`
- **Detail:** [PLATFORM FACT] Android deep links are routed through the Intents system. Custom URI schemes can be registered by multiple apps and may trigger a disambiguation dialog; App Links are verified web links for domains the app controls. [STORE PROPOSAL] Use stable HTTPS App Links as the public mini-app discovery/launch identifier and reserve custom schemes for tightly controlled internal handoffs. Map the HTTPS URL to an immutable app_id plus an approved route, keep a web fallback, and do not treat a scheme-only match as proof of app identity.
- **URL(s):** [https://developer.android.com/training/app-links/create-deeplinks](https://developer.android.com/training/app-links/create-deeplinks)

### 9. `android_dynamic_link_scope` — primary-platform
- **Source:** Android Developers — Configure website associations and dynamic rules
- **Topic:** `android_dynamic_launch_policy`
- **Detail:** [PLATFORM FACT] On Android 15 and later devices with Google services, the `assetlinks.json` file can carry dynamic App Links rules for path, fragment, and query matching; Android periodically retrieves the file and merges dynamic configuration with static manifest configuration. The file publicly declares which apps are authorized to handle a domain's links. [STORE PROPOSAL] Model effective launch scope as a versioned combination of approved static and dynamic rules. Re-review any path/query/fragment expansion, monitor retrieval freshness and parse errors, record rule diffs, and fail closed to the web fallback when the effective association is stale or unverified.
- **URL(s):** [https://developer.android.com/training/app-links/configure-assetlinks](https://developer.android.com/training/app-links/configure-assetlinks)

### 10. `android_per_host_verification_evidence` — primary-platform
- **Source:** Android Developers — Verify App Links
- **Topic:** `android_domain_verification_lifecycle`
- **Detail:** [PLATFORM FACT] For each unique hostname in the app's intent filters, Android queries `https://hostname/.well-known/assetlinks.json`; the documented verification output distinguishes a `verified` domain from other states, and older Android versions require a matching Digital Asset Links file for all declared hosts. [STORE PROPOSAL] Make host verification an explicit acceptance test: check every declared host over HTTPS, compare package and signing fingerprint, persist status/timestamp/evidence, and do not label an app globally verified while a required host remains unverified.
- **URL(s):** [https://developer.android.com/training/app-links/verify-applinks](https://developer.android.com/training/app-links/verify-applinks)

### 11. `apple_universal_link_context_and_fallback` — primary-platform
- **Source:** Apple Developer Documentation — Allowing apps and websites to link to your content
- **Topic:** `apple_universal_links_launch_context`
- **Detail:** [PLATFORM FACT] Apple Universal Links are standard HTTP/HTTPS links: the same URL can serve the website or open the app, and an uninstalled app falls back to the default web browser. The system verifies a server-hosted association file and supplies a user-activity object when a universal link routes to the app; the URL can carry declared query parameters for app-to-app context. [STORE PROPOSAL] Define an explicit launch envelope containing app_id, canonical HTTPS URL, source surface, route, and an allowlisted parameter schema. Preserve the original URL for audit and fallback, reject undeclared parameters or credentials, and let the app render only routes covered by its association contract.
- **URL(s):** [https://developer.apple.com/documentation/xcode/allowing-apps-and-websites-to-link-to-your-content.md](https://developer.apple.com/documentation/xcode/allowing-apps-and-websites-to-link-to-your-content.md)

### 12. `apple_association_file_integrity` — primary-platform
- **Source:** Apple Developer Documentation — Supporting associated domains
- **Topic:** `apple_universal_links_domain_binding`
- **Detail:** [PLATFORM FACT] Apple requires an `apple-app-site-association` file and matching Associated Domains entitlement; the file lists app identifiers and URL components, is served at `https://<fully-qualified-domain>/.well-known/apple-app-site-association` without an extension, valid certificate, or redirects. Each subdomain needs its own entitlement entry and association file. [STORE PROPOSAL] Require per-host association verification during submission and before promotion, store the appID/entitlement evidence, treat path/query/fragment component changes as re-review triggers, and fall back to the browser rather than silently opening an unverified app.
- **URL(s):** [https://developer.apple.com/documentation/xcode/supporting-associated-domains.md](https://developer.apple.com/documentation/xcode/supporting-associated-domains.md)

## B. Offline, cache and performance contracts

### 13. `cache_policy_requirement` — official-normative
- **Source:** IETF RFC 9111 HTTP Caching
- **Topic:** `http_caching_cache_invalidation`
- **Detail:** [SOURCE FACT] RFC 9111 is an Internet Standards Track document for HTTP cache behavior. A cache MUST NOT store a response when no-store applies; shared-cache reuse is restricted for private responses and requests carrying Authorization unless explicitly allowed; a stored response may satisfy a request only when fresh, allowed to be stale, or successfully validated; unsafe requests must be written through and may invalidate stored responses; Vary controls whether a stored response matches a later request. [PROPOSAL] Define a catalog/runtime cache matrix: digest-addressed package blobs may use a long-lived policy because their URL identifies immutable content; mutable catalog/index responses must carry an explicit freshness/validation policy; user, tenant, entitlement, and revocation responses must be no-store or tightly private. Include cache-key dimensions and Vary behavior in the contract, and version catalog pointers so release changes do not depend on ad-hoc purge timing. [LIMIT] RFC 9111 governs HTTP caches and headers; it does not define a mini-app manifest, package lifecycle, host bridge, or offline execution policy.
- **URL(s):** [https://www.rfc-editor.org/rfc/rfc9111.html](https://www.rfc-editor.org/rfc/rfc9111.html)

### 14. `stale_response_resilience` — official-normative
- **Source:** IETF RFC 5861 HTTP Cache-Control stale controls
- **Topic:** `poor_network_stale_fallback`
- **Detail:** [SOURCE FACT] RFC 5861 defines independent stale-while-revalidate and stale-if-error Cache-Control extensions. stale-while-revalidate permits a cache to serve a response after it becomes stale for a bounded delta while revalidation runs without blocking; stale-if-error permits stale use after an error such as a 500 response, network-segment failure, or DNS failure. Its example uses max-age=600 and stale-while-revalidate=30, and says the stale window should stop after the configured delta absent other information. [PROPOSAL] Permit bounded stale serving for non-critical catalog metadata, thumbnails, and discovery facets; show freshness/last-sync state and record stale-hit, revalidation-success, and stale-age metrics. Do not apply stale fallback to package revocation, permission policy, entitlements, or security advisories unless the operator has an explicit risk decision. [LIMIT] RFC 5861 is an IETF Informational RFC and an HTTP cache extension, not a mini-app standard; its example values are not universal SLOs.
- **URL(s):** [https://www.rfc-editor.org/rfc/rfc5861.html](https://www.rfc-editor.org/rfc/rfc5861.html)

### 15. `offline_runtime_fallback` — official-normative
- **Source:** W3C Service Workers specification
- **Topic:** `service_worker_offline_fallback`
- **Detail:** [SOURCE FACT] The W3C Service Workers document describes a fetch event and a request/response store similar in design to the HTTP cache to help build offline-enabled web applications. It defines install and activate lifecycle events plus fetch functional events; service workers are event-driven, asynchronous, time-limited contexts that a user agent may terminate at any time. [PROPOSAL] A web-based mini-app runtime, or a native equivalent, should pre-cache only the verified shell, catalog snapshot, and last-known-good app entrypoint; route navigation/resource failures to an explicit offline fallback; make install/activate/update idempotent; and persist required state rather than relying on a worker remaining alive. Record fallback reason and age so users can distinguish cached content from live content. [LIMIT] This W3C page is a Working Draft/nightly document and models web origins/service-worker clients; it does not define a mini-app package manifest, host-native capability policy, store review flow, or cross-runtime conformance target.
- **URL(s):** [https://www.w3.org/TR/service-workers/](https://www.w3.org/TR/service-workers/)

### 16. `cache_layer_strategy_matrix` — official-vendor
- **Source:** Chrome Developers Workbox caching strategies guidance
- **Topic:** `cache_layers_strategy_offline`
- **Detail:** [SOURCE FACT] Chrome's Workbox guidance states that the JavaScript Cache interface is separate from the browser HTTP cache, so HTTP Cache-Control directives do not control what is stored in the Cache interface. It presents cache-first for static/hash-versioned assets, network-first with cache fallback for HTML or API data, and stale-while-revalidate for resources where fast access matters more than immediate freshness. [PROPOSAL] Make cache layer and strategy explicit per artifact class: cache-first for digest-pinned package chunks and immutable assets; network-first then last-known-good for catalog/index/API reads; stale-while-revalidate only for non-critical metadata. Add cache namespace, schema version, maximum age, eviction rule, fallback eligibility, and metrics to the runtime contract, and test offline after a cold install and after an interrupted update. [LIMIT] This is Google/Chrome vendor guidance for service-worker web apps, not a normative mini-app standard; the Cache interface and strategy names may not exist in a native container.
- **URL(s):** [https://developer.chrome.com/docs/workbox/caching-strategies-overview](https://developer.chrome.com/docs/workbox/caching-strategies-overview)

### 17. `cached_artifact_integrity` — official-normative
- **Source:** W3C Subresource Integrity specification
- **Topic:** `integrity_of_cached_artifacts`
- **Detail:** [SOURCE FACT] W3C Subresource Integrity defines integrity metadata as a cryptographic hash and digest supplied with a request so a user agent can verify the fetched representation before execution. Conformant user agents must support SHA-256, SHA-384, and SHA-512 for integrity metadata; the document defines integrity for HTML script/link subresources and is currently a Working Draft. [PROPOSAL] Extend the same invariant to mini-app artifacts: bind every manifest, package chunk, and hosted executable subresource to a digest; verify bytes before committing them to the cache and again before launch; quarantine and delete mismatches; keep the expected digest with the release record; and surface verification failure as a hard offline-safe error rather than executing a previous unverified download. [LIMIT] SRI itself does not define arbitrary package/archive verification, publisher key trust, revocation, catalog signing, or mini-app cache invalidation; the runtime needs a package-level digest/signature contract in addition to SRI where applicable.
- **URL(s):** [https://www.w3.org/TR/SRI/](https://www.w3.org/TR/SRI/)

### 18. `startup_timing_contract` — official-normative
- **Source:** W3C Navigation Timing Level 2
- **Topic:** `startup_performance_measurement`
- **Detail:** [SOURCE FACT] Navigation Timing exposes workerStart immediately before service-worker activation/start, fetchStart immediately before FetchEvent dispatch when a service worker handles navigation, responseStart/responseEnd covering the fetch process including HTTP-cache work, and DOM-content-loaded/load timestamps. [PROPOSAL] Define startup telemetry with worker_activation_ms = fetchStart - workerStart, response_wait_ms = responseStart - requestStart, document_ready_ms = domContentLoadedEventEnd - startTime, and load_complete_ms = loadEventEnd - startTime; report p50/p75/p95 by network class, cache state, host version, device class, and package digest. Gate rollout on measured regression budgets rather than a guessed universal threshold, and separate cold-cache, warm-cache, and offline-fallback cohorts. [LIMIT] This W3C page is a Working Draft and supplies timing interfaces, not target SLO values or a mini-app lifecycle; native runtimes must document how their equivalent timestamps map to these fields.
- **URL(s):** [https://www.w3.org/TR/navigation-timing-2/](https://www.w3.org/TR/navigation-timing-2/)

### 19. `resource_transfer_budget_telemetry` — official-normative
- **Source:** W3C Resource Timing
- **Topic:** `resource_cache_performance_metrics`
- **Detail:** [SOURCE FACT] W3C Resource Timing includes resources retrieved from the HTTP cache and aborted network-error fetches in the performance timeline. It exposes deliveryType, response timing, worker/cache lookup timing, transferSize, encodedBodySize, decodedBodySize, and responseStatus; the specification defines deliveryType=cache for non-empty cache mode and transferSize=0 for local cache and 300 for validated cache, while cross-origin data can be zeroed without the required timing permission. [PROPOSAL] Require the mini-app runtime to emit an equivalent per-artifact record: request class, cache layer, hit/miss/revalidated state, response status, encoded/decoded bytes, transfer bytes, worker/cache lookup duration, and failure reason. Use these fields to enforce package-size and network-byte budgets by cohort, and require a Timing-Allow-Origin-equivalent declaration or host-side instrumentation when resources are cross-origin. [LIMIT] This is a browser performance API in a W3C draft, not a native mini-app telemetry standard; exact byte semantics and visibility cannot be assumed across WebView, native, and custom runtimes.
- **URL(s):** [https://www.w3.org/TR/resource-timing/](https://www.w3.org/TR/resource-timing/)

### 20. `on_demand_package_splitting` — primary-platform
- **Source:** Android Developers Play Feature Delivery
- **Topic:** `package_splitting_on_demand_delivery`
- **Detail:** [SOURCE FACT] Android App Bundles generate device-specific delivery so users download only the code and resources needed; Play Feature Delivery supports conditional and on-demand feature modules separated from the base module. Android's guidance warns that 50 or more feature modules may cause performance issues, recommends 10 or fewer removable install-time modules, requires API level 21 or higher for on-demand installation, and says callers should confirm a feature is downloaded before accessing its code/resources. [PROPOSAL] Use a mini-app split manifest with a mandatory base shell plus optional feature chunks, dependency edges, install mode, byte size, minimum runtime, and offline availability. Resolve dependencies before launch, refuse partial execution, expose download/install progress and retry state, and measure cold-start bytes, chunk download time, and launch failure by network class. [LIMIT] These module counts and API-level constraints are Android-specific platform guidance, not transferable mini-app limits; Android App Bundles do not define a cross-platform catalog or mini-app package format.
- **URL(s):** [https://developer.android.com/guide/playcore/feature-delivery](https://developer.android.com/guide/playcore/feature-delivery)

### 21. `main_thread_startup_budget` — official-normative
- **Source:** W3C Long Tasks API
- **Topic:** `startup_main_thread_responsiveness`
- **Detail:** [SOURCE FACT] The W3C Long Tasks API identifies tasks that monopolize the UI thread and defines a long task as exceeding 50 ms. Its motivation connects long tasks to delayed interactivity, input latency, and janky scrolling; it cites the RAIL model's under-100-ms input response goal and exposes buffered longtask entries through PerformanceObserver. [PROPOSAL] Treat longtask count, total duration, and maximum duration during mini-app startup as release metrics; fail a performance gate when a tested device/network cohort regresses beyond a recorded baseline, and use code-splitting/deferred initialization to keep non-critical work out of the launch path. The 50-ms observation threshold can be a diagnostic boundary, while launch SLOs must be calibrated from real devices. [LIMIT] This W3C page is a Working Draft for browser UI threads, and the RAIL goal is guidance rather than a mini-app store requirement; native runtimes need an equivalent main-thread or event-loop metric.
- **URL(s):** [https://www.w3.org/TR/longtasks-1/](https://www.w3.org/TR/longtasks-1/)

## C. Payments, notifications and marketplace trust

### 22. `payment_token_scope` — official-normative
- **Source:** EMVCo — Payment Tokenisation
- **Topic:** `payments/tokenization/host-boundary`
- **Detail:** EMVCo states that payment tokenisation removes the primary account number (PAN) and replaces it with a unique alternative value; an EMV Payment Token is constrained to a use context such as a specific merchant, device, or payment scenario. This is EMVCo payment-token evidence, not a universal mini-app rule. Boundary proposal: a mini-app should receive only a host/PSP-controlled, least-scope payment token or transaction reference, while PAN handling and token lifecycle remain outside the mini-app runtime; do not treat a payment token as proof of entitlement by itself.
- **URL(s):** [https://www.emvco.com/emv-technologies/payment-tokenisation/](https://www.emvco.com/emv-technologies/payment-tokenisation/)

### 23. `tokenization_system_boundary` — official-normative
- **Source:** PCI Security Standards Council — PCI DSS Tokenization Guidelines
- **Topic:** `payments/tokenization/pci-scope`
- **Detail:** The PCI SSC supplement says tokenization and de-tokenization should occur only inside a clearly defined tokenization system; PAN retrieval from a token must be restricted to specifically authorized individuals, applications, or systems, and token-generation/de-tokenization keys must not be available outside that secure system. It also states that tokenization of sensitive authentication data such as CVV/CVC/PIN data is not permitted under PCI DSS Requirement 3.2, and that the supplement does not replace the PCI DSS. For a mini-app store, use this as payment-data boundary guidance: mini-app code should not receive PAN, CVV, or de-tokenization authority; the host/payment service returns a result or opaque reference.
- **URL(s):** [https://www.pcisecuritystandards.org/documents/Tokenization_Guidelines_Info_Supplement.pdf](https://www.pcisecuritystandards.org/documents/Tokenization_Guidelines_Info_Supplement.pdf)

### 24. `entitlement_server_authority` — primary-platform
- **Source:** Google Play Billing official documentation — Integrate the Google Play Billing Library
- **Topic:** `payments/purchase-entitlement/server-authority`
- **Detail:** Google Play documents a purchase flow in which the app verifies the purchase on a server before granting benefits. It describes the purchase token as identifying the user and product, recommends sending it to a secure backend for verification and fraud protection, requires checking PURCHASED rather than PENDING before granting entitlement, and says acknowledgement within three days prevents automatic refund and entitlement revocation. This is Google Play-specific evidence, not a universal rule; the reusable boundary is to make the host/backend the entitlement authority and expose only a verified entitlement result to a mini-app.
- **URL(s):** [https://developer.android.com/google/play/billing/integrate](https://developer.android.com/google/play/billing/integrate)

### 25. `entitlement_event_idempotency` — primary-platform
- **Source:** Google Play Billing official documentation — Purchase lifecycle and RTDNs
- **Topic:** `payments/entitlements/webhook-idempotency`
- **Detail:** Google Play recommends a backend purchase-status management system for one-time purchases and subscriptions. Its RTDN flow sends entitlement-state changes to the backend and instructs the consumer to check messageId uniqueness so duplicate notifications are not processed. This is a provider-specific lifecycle pattern, but it supports a portable control: keep an append-only entitlement-event ledger, deduplicate by provider event ID, reconcile against the provider API, and make grant/revoke handlers idempotent.
- **URL(s):** [https://developer.android.com/google/play/billing/lifecycle](https://developer.android.com/google/play/billing/lifecycle)

### 26. `refund_dispute_event_ownership` — primary-platform
- **Source:** Google Play Billing official documentation — Real-time developer notifications reference
- **Topic:** `payments/refunds/chargebacks/ownership`
- **Detail:** Google Play's notification reference distinguishes voided-purchase notifications from pending-refund-review notifications; the latter represents a chargeback request for which the recipient can suggest a resolution through the ReviewRefund API. This is a Google Play-specific ownership model, not a universal marketplace rule. A mini-app standard should explicitly name the payment provider as event authority where applicable, define who owns refund/dispute response and entitlement revocation, and expose status/support routing to the user instead of making the mini-app infer settlement state.
- **URL(s):** [https://developer.android.com/google/play/billing/rtdn-reference](https://developer.android.com/google/play/billing/rtdn-reference)

### 27. `webhook_authenticity_raw_payload` — official-vendor
- **Source:** Stripe official documentation — Webhook signature verification
- **Topic:** `payments/webhooks/authenticity`
- **Detail:** Stripe's official guidance says webhook consumers should verify that an event came from Stripe using the signature header, the unchanged request body, and the endpoint-specific verification credential. This is Stripe practice, not a universal protocol. The reusable boundary is to authenticate and integrity-check payment, refund, and entitlement events before state mutation; reject altered or unauthenticated payloads, then deduplicate the provider event ID and record the verification result.
- **URL(s):** [https://docs.stripe.com/webhooks/signature](https://docs.stripe.com/webhooks/signature)

### 28. `webhook_signature_replay_boundary` — official-normative
- **Source:** IETF RFC 9421 — HTTP Message Signatures
- **Topic:** `payments/webhooks/message-signatures/replay`
- **Detail:** RFC 9421 defines signing and verification of selected HTTP message components. It warns that unsigned components can be modified, says TLS remains necessary, and describes nonce, creation-time, and expiry parameters to limit replay; verifiers should accept only expected signers. This is a protocol standard rather than a mini-app marketplace rule. Apply it as a high-risk webhook option: sign the event identity, target, relevant headers/body digest, issuer, and time window; enforce expected-key and replay checks before granting or revoking entitlements.
- **URL(s):** [https://www.rfc-editor.org/rfc/rfc9421.html](https://www.rfc-editor.org/rfc/rfc9421.html)

### 29. `notification_permission_user_control` — official-normative
- **Source:** W3C Permissions specification
- **Topic:** `notifications/opt-in/consent`
- **Detail:** The W3C Permissions model treats permission as the user's decision to allow or deny a powerful feature; its explanatory text says user agents should deny access until express permission, retain user control through grant/deny preferences, and may hide nuisance prompts or expire grants. It exposes permission states such as granted, prompt, and denied. This is web-platform evidence, not a universal super-app mandate. A host should maintain per-mini-app/per-purpose notification state, provide revoke/quiet controls, and never convert a catalog install into blanket notification consent.
- **URL(s):** [https://www.w3.org/TR/permissions/](https://www.w3.org/TR/permissions/)

### 30. `push_subscription_scope_and_quota` — official-normative
- **Source:** W3C Push API specification
- **Topic:** `notifications/push/abuse-controls`
- **Detail:** The W3C Push API requires subscription creation to request push permission and reject the operation when permission is denied; a push subscription has a URL scope and a unique push endpoint. The specification also notes that push services commonly limit message size and quantity. This is web-platform evidence, not a marketplace-specific policy. For mini-app notifications, bind each subscription to app/tenant/topic, support explicit unsubscribe and expiry, apply provider and host rate/volume quotas, and keep promotional pushes opt-in rather than treating delivery capability as an entitlement.
- **URL(s):** [https://www.w3.org/TR/push-api/](https://www.w3.org/TR/push-api/)

### 31. `notification_opt_in_and_abuse_controls` — primary-platform
- **Source:** Android Developers — Notification runtime permission
- **Topic:** `notifications/opt-in/abuse-controls`
- **Detail:** Android documents POST_NOTIFICATIONS as a runtime permission on Android 13 and higher: newly installed apps have notifications off by default and must wait for the user to grant permission; a denial blocks notifications except for exemptions, and users can see daily notification counts and revoke permission. This is Android-specific evidence. A super-app host can generalize the boundary by separating transactional from marketing topics, requiring explicit opt-in per mini-app, exposing aggregate volume and quiet controls, and enforcing rate limits before a mini-app can send.
- **URL(s):** [https://developer.android.com/develop/ui/views/notifications/notification-permission](https://developer.android.com/develop/ui/views/notifications/notification-permission)

### 32. `deep_link_association_and_fallback` — primary-platform
- **Source:** Apple Developer — Supporting Universal Links
- **Topic:** `deep-links/domain-association/fallback`
- **Detail:** Apple documents Universal Links as standard HTTP/HTTPS links that other apps cannot claim, with a secure association checked through an apple-app-site-association file. The file must be served over HTTPS without redirects and can allow or exclude specific paths; Apple also says unsupported URLs should fail gracefully. This is Apple-specific evidence. A cross-platform store should require signed/verified domain association, explicit path allowlists, safe web fallback, and host-side validation before opening a mini-app or payment-return link.
- **URL(s):** [https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/UniversalLinks.html](https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/UniversalLinks.html)

### 33. `ranking_parameter_disclosure` — official-normative
- **Source:** European Commission — Ranking transparency guidelines explainer
- **Topic:** `marketplace/ranking/disclosure`
- **Detail:** The European Commission explains that Regulation (EU) 2019/1150 covers online intermediation services including app stores and requires the main ranking parameters and the reasons for their relative importance to be set out in terms and conditions. The Commission expressly says these guidelines are not legally binding. This is EU-scoped official guidance, not a universal rule. It supports a marketplace control requiring a high-level ranking-parameter disclosure, change history, and separate labels for organic ranking, editorial merchandising, advertising, and sponsored placement.
- **URL(s):** [https://digital-strategy.ec.europa.eu/en/library/ranking-transparency-guidelines-framework-eu-regulation-platform-business-relations-explainer](https://digital-strategy.ec.europa.eu/en/library/ranking-transparency-guidelines-framework-eu-regulation-platform-business-relations-explainer)

### 34. `review_ranking_integrity` — primary-platform
- **Source:** Google Play Developer Policy — User ratings, reviews and installs
- **Topic:** `marketplace/reviews/ranking-integrity`
- **Detail:** Google Play policy prohibits attempts to manipulate app placement, including inflating ratings, reviews, or install counts through fraudulent or incentivized reviews/ratings; it also lists forced pop-ups and paying for fake reviews as prohibited examples and says users depend on reviews being authentic and relevant. This is Google Play policy, not a universal marketplace law. A mini-app store can reuse the control pattern: prohibit sentiment-conditioned incentives and synthetic activity, preserve review provenance/moderation records, and keep paid/editorial placement separate from user-review signals.
- **URL(s):** [https://support.google.com/googleplay/android-developer/answer/9898684?hl=en](https://support.google.com/googleplay/android-developer/answer/9898684?hl=en)

### 35. `review_incentive_disclosure_and_suppression` — official-normative
- **Source:** U.S. Federal Trade Commission — Consumer Reviews and Testimonials Rule Q&A
- **Topic:** `marketplace/reviews/incentives/disclosure`
- **Detail:** FTC guidance says the Consumer Reviews and Testimonials Rule took effect on 21 October 2024 and addresses fake, false, or deceptive reviews/testimonials. It explains that incentivized reviews can also be testimonials, that incentives conditioned on a particular sentiment are prohibited, and that paying consumers to change or remove truthful negative reviews may violate the FTC Act. The page states its staff guidance is context-specific and not a safe harbor. For a marketplace, use this as U.S.-scoped evidence for transparent incentive disclosure, no sentiment gating, anti-suppression controls, and an auditable moderation/appeal trail; do not infer global legal coverage.
- **URL(s):** [https://www.ftc.gov/business-guidance/resources/consumer-reviews-testimonials-rule-questions-answers](https://www.ftc.gov/business-guidance/resources/consumer-reviews-testimonials-rule-questions-answers)

### 36. `refund_merchant_of_record_boundary` — official-vendor
- **Source:** Stripe official documentation — Refunds
- **Topic:** `payments/refunds/merchant-of-record/ownership`
- **Detail:** Stripe's refund documentation says Connect refund responsibility depends on charge type: direct-charge refunds are charged to the connected account, while destination charges or separate charges/transfers charge the platform; refunds go only to the original payment method, and refund status is exposed through events such as refund.created, refund.updated, refund.failed, and charge.refunded. This is Stripe-specific practice, not a universal ownership rule. A marketplace contract and ledger should name the merchant of record, payment processor, refund/dispute owner, funding account, user-support route, and entitlement-revocation trigger before a mini-app can sell.
- **URL(s):** [https://docs.stripe.com/refunds](https://docs.stripe.com/refunds)

## Standard implications from this iteration

- **Launch identity:** use a stable app identity separate from mutable display metadata; require verified HTTPS associations, explicit route/parameter allowlists, canonicalization rules, safe web fallback, and transaction-bound authentication context.
- **Offline behavior:** define cache layer, freshness, stale fallback, invalidation, integrity, eviction, and offline eligibility per artifact class. Never serve stale entitlement, revocation, permission, or security-policy state without an explicit risk decision.
- **Performance evidence:** measure cold/warm/offline startup and resource transfer by package digest, host version, device and network cohort; use regression budgets calibrated from pilot telemetry rather than importing platform-specific limits as universal requirements.
- **Payments and notifications:** keep payment credentials and entitlement authority in the host/backend; authenticate, deduplicate and reconcile provider events; separate transactional notifications from marketing opt-in and enforce quotas.
- **Marketplace trust:** disclose ranking parameters at the applicable jurisdictional scope, separate organic/editorial/paid placement, and maintain review provenance, incentive disclosure, anti-manipulation and appeal controls.
