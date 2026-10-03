# Iteration 18: Identity, Authentication, Session and Account-Linking Contracts

## Scope and evidence policy

This section records nine parent-validated findings from a structurally new direction. The evidence is official IETF or W3C material fetched and rechecked with HTTP 200 on 2026-10-03. Standard facts and store design proposals are separated inside each finding. The W3C passkey-endpoints document is identified as a working draft where applicable; it is not treated as a final universal requirement.

## Findings

### 1. `oauth_par_authorization_request_integrity`

- **Event:** `par_request_integrity_and_single_use_binding`
- **Evidence level:** `official-normative`
- **Source:** IETF RFC 9126 — OAuth 2.0 Pushed Authorization Requests
- **Detail:** [NORMATIVE RFC FACT] RFC 9126 is an Internet Standards Track document. Its pushed-authorization-request (PAR) endpoint MUST use HTTPS; the authorization server MUST authenticate the client, validate the pushed request as an authorization request (including the configured redirect URI and authorized scope), and on success return a cryptographically unpredictable request_uri bound to the client, with a positive expiration and single-use semantics. [STORE DESIGN PROPOSAL] For high-risk publisher onboarding, publisher administration, and end-user account-linking flows, require PAR before browser consent: bind each request to the store tenant, mini-app ID, publisher ID, issuer, exact redirect URI, requested scopes/resources, and any PKCE/state transaction values used by the flow; persist a request digest and consume request_uri once. Reject client, issuer, redirect, scope, and replay mismatches, and expose the end-user consent screen only after host pre-validation. PAR is an authorization-request transport and integrity pattern; it does not itself define publisher verification, consent wording, session lifetime, or account-linking policy.
- **URL:** [https://www.rfc-editor.org/rfc/rfc9126.html](https://www.rfc-editor.org/rfc/rfc9126.html)

### 2. `oauth_issuer_binding_account_linking`

- **Event:** `authorization_response_issuer_binding`
- **Evidence level:** `official-normative`
- **Source:** IETF RFC 9207 — OAuth 2.0 Authorization Server Issuer Identification
- **Detail:** [NORMATIVE RFC FACT] RFC 9207 defines the OAuth authorization-response iss parameter. An authorization server supporting the specification MUST include iss in authorization and error responses; the value MUST be an HTTPS URL without query or fragment components. A supporting client MUST compare the decoded iss value with the expected issuer using simple string comparison and MUST reject a mismatch; it must also reject a missing iss from a server configured as supporting the parameter, and accepted issuer identifiers must be unique per authorization server. [STORE DESIGN PROPOSAL] Maintain an issuer registry per mini-app/client and bind the callback response, token endpoint, metadata, and runtime API audience to that exact issuer string. Use a namespaced external identity key such as (issuer, subject) rather than a username or email, require explicit end-user confirmation before linking a second issuer, and never merge accounts solely on matching profile fields. RFC 9207 addresses authorization-server mix-up protection; it does not establish human identity, publisher legal identity, or account-merging rules.
- **URL:** [https://www.rfc-editor.org/rfc/rfc9207.html](https://www.rfc-editor.org/rfc/rfc9207.html)

### 3. `oauth_introspection_session_liveness_identity`

- **Event:** `introspection_identity_audience_and_session_liveness`
- **Evidence level:** `official-normative`
- **Source:** IETF RFC 7662 — OAuth 2.0 Token Introspection
- **Detail:** [NORMATIVE RFC FACT] RFC 7662 defines an introspection endpoint that returns JSON metadata with a required active indicator; active=true generally means the token was issued by that authorization server, is not revoked, is within its validity window, and is valid for the protected resource. The endpoint MUST use transport security and MUST require authorization to prevent token-scanning attacks. For an inactive, unknown, or unauthorized token, the server MUST return active=false and SHOULD avoid disclosing why. Optional metadata can include client_id, sub, aud, iss, scope, exp, iat, and jti. [STORE DESIGN PROPOSAL] Use introspection at host API boundaries for high-risk runtime, publisher, and administrator calls or whenever immediate revocation matters. Validate active together with expected issuer, audience/resource, scopes, client/app, tenant, and end-user subject; use a namespaced identity mapping such as (issuer, sub) rather than a username or display name; cache only with a risk-tiered short TTL and fail closed on inactive or mismatched results; never log raw credential values. RFC 7662 permits caching but leaves the liveness trade-off, identity linking, and authorization policy to the deployment.
- **URL:** [https://www.rfc-editor.org/rfc/rfc7662.html](https://www.rfc-editor.org/rfc/rfc7662.html)

### 4. `authentication/rp-id-origin/host-boundary`

- **Event:** `host_mediated_rp_id_origin_scope`
- **Evidence level:** `official-w3c-recommendation+design-proposal`
- **Source:** W3C Web Authentication: An API for accessing Public Key Credentials — Level 3
- **Detail:** [STANDARD FACT — W3C RECOMMENDATION] WebAuthn credentials are scoped by the RP ID: the client MUST verify that the RP origin matches the RP ID scope, the authenticator MUST require the RP ID to exactly equal the credential's rpId, and the RP server MUST validate clientData.origin and authenticatorData.rpIdHash. WebAuthn is disabled by default in cross-origin iframes; an embedded flow needs the relevant Permissions Policy and the RP must validate expected cross-origin/topOrigin context. Related-origin use requires one common RP ID plus an HTTPS /.well-known/webauthn JSON document listing approved origins; clients fetch it without credentials or referrer and require a 200 response and HTTPS redirects. [MINI-APP STORE DESIGN PROPOSAL] Make the store host the sole WebAuthn RP for user and publisher identities, pin a stable host RP ID and exact approved origins/top origins, and reject mini-app-supplied rpId, origin, challenge, or related-origin values. Expose a capability-gated host authentication API rather than navigator.credentials or credential material to mini-app code; the host performs the ceremony and verification, and the mini-app receives only a host-issued authorization result. Do not broaden the RP ID to an untrusted publisher/content domain; approve any related-origin entry through host review and treat the related-origin document as a security-controlled configuration.
- **URL:** [https://www.w3.org/TR/webauthn-3/](https://www.w3.org/TR/webauthn-3/)

### 5. `authentication/user-verification/step-up/opaque-contract`

- **Event:** `host_mediated_user_verification_step_up`
- **Evidence level:** `official-w3c-recommendation+working-draft+design-proposal`
- **Source:** W3C WebAuthn Level 3 and A Well-Known URL for Relying Party Passkey Endpoints
- **Detail:** [STANDARD FACT — W3C RECOMMENDATION] For an operation requiring user verification, WebAuthn userVerification=required makes the client fail if verification cannot be performed and the RP must reject an assertion whose UV flag is not set. WebAuthn challenges MUST be randomly generated in a trusted RP environment, retained until completion, matched against the response, and SHOULD contain at least 16 bytes of entropy. The specification also says biometric data is used locally and is not revealed to the RP. [WORKING-DRAFT FACT — W3C PASSKEY ENDPOINTS] The passkey-endpoints Working Draft defines an HTTPS /.well-known/passkey-endpoints JSON document with optional direct enroll and manage URLs; it is a work in progress, not a final Recommendation. [MINI-APP STORE DESIGN PROPOSAL] Define a host step-up contract for publisher release, payout, credential, permission, and account-recovery actions: the host creates a one-time challenge bound to session, publisher/account, app_id, action name, action-payload digest, audience, and expiry; verifies origin, RP ID, signature, and required UV; then returns a short-lived, single-use, signed host token or an approved/denied decision. The mini-app must not receive the WebAuthn assertion, credential ID, public key, user handle, attestation, authenticator metadata, or any other credential material; it gets only the action-scoped result needed to call the host API. Keep the host as the verifier and policy engine, and require explicit user consent before enrollment or destructive actions.
- **URL:** [https://www.w3.org/TR/passkey-endpoints/](https://www.w3.org/TR/passkey-endpoints/)

### 6. `session-security/dpop/nonce-replay/transaction-binding`

- **Event:** `dpop_nonce_jti_transaction_binding`
- **Evidence level:** `official-normative+design-proposal`
- **Source:** IETF RFC 9449 — OAuth 2.0 Demonstrating Proof of Possession (DPoP)
- **Detail:** [NORMATIVE RFC FACT] RFC 9449 defines DPoP as an application-layer proof that sender-constrains an OAuth token to a public key. A protected-resource request carries a DPoP proof plus the access token; the proof binds the HTTP method (htm), target URI without query/fragment (htu), creation time (iat), a unique proof identifier (jti), and the SHA-256 hash of the presented token (ath). The access-token JWT can carry cnf.jkt, the JWK thumbprint of the key, and the resource server compares it with the proof key. RFC 9449 says servers MUST accept proofs only for a limited lifetime and can store jti values during that window to reject reuse; server-provided DPoP-Nonce values are unpredictable, and a server MUST NOT accept a proof without its nonce after it has issued a nonce. [MINI-APP HOST CONTROL] Put this check in the store/API gateway, not in untrusted mini-app JavaScript: maintain a bounded replay cache keyed by issuer, DPoP key thumbprint, canonical route, and jti; enforce nonce ownership per authorization/resource server; canonicalize the route after proxy normalization; and bind the accepted proof to the registered mini-app, session handle, and one high-risk transaction. Return an idempotent prior result rather than re-running a purchase, permission grant, install, or release action when the transaction handle or jti has already been consumed. [TRANSACTION BINDING LIMIT] DPoP binds the proof to method, target, token, and key but does not sign arbitrary request-body fields; the host therefore must bind transaction ID, amount/capability version, and other action semantics in server-side state or an additional application signature rather than treating DPoP alone as a body-level authorization.
- **URL:** [https://datatracker.ietf.org/doc/html/rfc9449](https://datatracker.ietf.org/doc/html/rfc9449)

### 7. `transaction-authorization/token-exchange/scoped-capability/opaque-result-passing`

- **Event:** `host_token_exchange_scoped_capability_and_opaque_result`
- **Evidence level:** `official-normative+design-proposal`
- **Source:** IETF RFC 8693 — OAuth 2.0 Token Exchange
- **Detail:** [NORMATIVE RFC FACT] RFC 8693 defines an HTTP/JSON security-token service for requesting a new token using a subject_token and, when delegation is involved, an actor_token. The subject identifies the party on whose behalf access is requested; the actor identifies the party to which rights are delegated. The request can name resource, audience, and scope, and the authorization server can return invalid_target when the target is unacceptable. The response exposes expires_in for the issued token's validity lifetime; a refresh token is typically not issued when one temporary credential is exchanged for another temporary credential. For JWT representations, the act claim records the current actor and can express a delegation chain. [MINI-APP STORE/RUNTIME PROPOSAL] Keep the end-user or host session credential inside the trusted host. At each sensitive mini-app operation, exchange it at the host STS for a short-lived capability whose audience/resource is the exact mini-app API and whose scope is the one approved action; preserve host/mini-app delegation in actor metadata when the chosen token representation supports it. Do not hand the mini-app a broad user token or refresh token. [OPAQUE RESULT PASSING PROPOSAL] After the action, return only a random, one-time result_handle to the mini-app bridge; redeem it through the host with the same session handle and transaction ID, and let the host fetch the provider/user result. Never put access tokens, identity claims, payment data, or raw provider responses in mini-app storage, URLs, deep links, logs, or cross-app intents. RFC 8693 standardizes the exchange boundary, not this result-handle format, so the handle's one-use, audience, expiry, and storage rules are host controls.
- **URL:** [https://www.rfc-editor.org/rfc/rfc8693.html](https://www.rfc-editor.org/rfc/rfc8693.html)

### 8. `audience-resource-binding/jwt-validation/mini-app-api-boundaries`

- **Event:** `jwt_access_token_audience_expiry_and_jti_validation`
- **Evidence level:** `official-normative+design-proposal`
- **Source:** IETF RFC 9068 — JSON Web Token (JWT) Profile for OAuth 2.0 Access Tokens
- **Detail:** [NORMATIVE RFC FACT] RFC 9068 defines an interoperable JWT access-token profile and the claims/validation rules for resource servers. Its JWT data structure requires iss, exp, aud, sub, client_id, iat, and jti; aud identifies the intended service, while exp and iat give the validity window and issuance time. The profile also explains how authorization-request context determines the issued token and how a resource server should validate incoming JWT access tokens, rather than treating a vendor-specific JWT layout as sufficient. [MINI-APP STORE/RUNTIME PROPOSAL] Register each mini-app API as a distinct resource URI with an issuer and accepted signing keys, and make the host gateway reject a token with the wrong issuer, audience, signature, time window, or client/mini-app binding before forwarding the call. Keep audience/resource policy at the gateway so a token for the store catalog, artifact registry, review API, or payment adapter cannot be replayed at a different service. Use jti as a correlation and transaction-reuse key only when the host defines that replay policy; RFC 9068 requires the claim but does not by itself make a JWT single-use. If the gateway chooses JWTs for internal scale, the mini-app should still receive an opaque host session/result handle, not a parseable user token; the store should re-review any release that changes its resource or capability audience.
- **URL:** [https://www.rfc-editor.org/rfc/rfc9068.html](https://www.rfc-editor.org/rfc/rfc9068.html)

### 9. `session-lifecycle/token-revocation/logout/uninstall/takedown`

- **Event:** `session_token_revocation_on_logout_uninstall_and_takedown`
- **Evidence level:** `official-normative+design-proposal`
- **Source:** IETF RFC 7009 — OAuth 2.0 Token Revocation
- **Detail:** [NORMATIVE RFC FACT] RFC 7009 defines a revocation endpoint for refresh and access tokens. Revocation invalidates the submitted token and, depending on policy, related tokens based on the same authorization grant; implementations MUST support refresh-token revocation and SHOULD support access-token revocation. After successful revocation, the token cannot be used again, although distributed deployments should minimize propagation delay. The RFC explicitly connects revocation with logout, identity changes, and application uninstall, and says a client must be prepared for unexpected token invalidation. [MINI-APP STORE/RUNTIME PROPOSAL] Treat logout, end-user permission withdrawal, publisher disablement, package quarantine, uninstall, and high-risk capability downgrade as revocation events: atomically mark the host session_handle and outstanding result_handles unusable, revoke exchanged access/refresh tokens upstream where applicable, evict DPoP nonce/jti state, and deny new exchanges until policy re-authorizes the app. Keep revocation status at the host gateway rather than relying only on token expiry; record actor, app/package version, grant, transaction, reason, provider response, and propagation state for audit and recovery. Because RFC 7009 allows implementation-specific propagation delay and cascading policy, the store should expose a fail-closed grace window for sensitive actions and should not claim global revocation has completed until every resource adapter reports the intended state.
- **URL:** [https://www.rfc-editor.org/rfc/rfc7009.html](https://www.rfc-editor.org/rfc/rfc7009.html)

## Standard implications for the mini-app store

1. Keep authorization and authentication ceremonies in the trusted host. Mini-app code should receive only an action-scoped result or opaque capability, not provider tokens, passkey assertions, raw identity claims, or credential material.
2. Bind every identity and authorization transaction to issuer, client/mini-app, tenant, exact redirect/origin, audience/resource, requested scopes, action, payload digest, expiry, and one-time state where the risk requires it.
3. Treat account linking, logout, uninstall, permission withdrawal, quarantine and takedown as revocation events. The host gateway must fail closed for sensitive calls while provider-side revocation is pending.
4. Use sender-constrained proofs and bounded replay state for high-risk operations. DPoP does not sign arbitrary request bodies, so transaction semantics still require server-side binding or an additional application signature.
5. Add a dedicated conformance fixture for cold-start authentication return, issuer mix-up, redirect mismatch, passkey origin/RP mismatch, challenge replay, DPoP nonce/jti replay, audience confusion, result-handle reuse, logout propagation and uninstall/takedown propagation.

## Remaining validation gaps

- The RFCs and W3C specifications do not select a universal identity provider, publisher KYC model, recovery SLA, session TTL, or legal account-merging policy.
- The Alibaba/WindVane POC must be tested for host bridge behavior, browser authorization return, native/passkey availability, callback replay handling and whether opaque host handles can be enforced without exposing raw credentials.
- Any deployment-specific issuer registry, resource/audience catalog, key rotation, DPoP support, recovery policy and revocation propagation target remains an implementation decision requiring security and privacy sign-off.
