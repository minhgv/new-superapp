# Module 126: OpenID Connect Core 1.0 Identity Federation, Coordinated Logout & OAuth Device Authorization Governance

## 1. Executive Summary & Normative Scope

This module formalizes the core federated identity, session lifecycle management, coordinated multi-client logout, and cross-device authorization governance standards for the super-app and mini-app runtime ecosystem. Modern super-apps act as distributed identity providers and application marketplaces hosting hundreds or thousands of heterogeneous third-party mini-apps across diverse client platforms—ranging from iOS and Android smartphones to desktop tablets, smart TVs, and interactive retail kiosk terminals.

In such a complex, multi-tenant environment, basic authentication models present severe architectural and security deficiencies:
1. **Cross-Application Tracking & PII Leakage**: Sharing static user identifiers (such as raw phone numbers, national IDs, or global internal account IDs) across independent mini-apps enables third-party developers to correlate user behavioral profiles without consent, directly violating GDPR Article 25 (Data Protection by Design) and sovereign data privacy regulations.
2. **Session Drift & Dangling Access on Logout**: When a user logs out of the host super-app or revokes access to a compromised account, third-party mini-app WebViews and backend servers often retain active access tokens, refresh tokens, and cached sessions indefinitely unless an authenticated, coordinated logout standard is enforced.
3. **Constrained Terminal Input Bottlenecks**: Smart TV applications, point-of-sale (POS) hardware, and unattended retail kiosk displays lack standard physical keyboards, making credential entry cumbersome and vulnerable to shoulder-surfing attacks.
4. **Broad Audience Token Replay Attacks**: Issuing broad-scoped tokens that lack specific microservice resource binding allows a compromised auxiliary mini-app service to replay credentials against high-value payment or administrative endpoints.

To resolve these vulnerabilities and establish an enterprise-grade identity fabric, this module standardizes three foundational OpenID and IETF specifications:
- **OpenID Connect Core 1.0 (incorporating Errata Set 2)**: Standardizing ID Token claims (`iss`, `sub`, `aud`, `exp`, `iat`, `auth_time`, `nonce`, `acr`, `amr`), cryptographic signature verification, Pairwise Pseudonymous Identifiers (PPID) salt derivation to eliminate cross-app tracking, and scoped UserInfo endpoint sandboxing.
- **OpenID Connect Coordinated Session & Logout Suite**: Incorporating **RP-Initiated Logout 1.0** (standardizing `id_token_hint` and `post_logout_redirect_uri` validation), **Back-Channel Logout 1.0** (direct server-to-server `logout_token` JWT delivery for deterministic session invalidation), **Front-Channel Logout 1.0** (browser/WebView mediated lifecycle teardown), and **Session Management 1.0** (`check_session_iframe` status telemetry).
- **IETF RFC 8628 (OAuth 2.0 Device Authorization Grant) & Token Lifecycle Governance**: Specifying the RFC 8628 flow (`device_code`, `user_code`, `verification_uri_complete` QR-code authentication for constrained terminals), **RFC 7662 OAuth 2.0 Token Introspection** (real-time active metadata querying for internal gateways), **RFC 7009 OAuth 2.0 Token Revocation** (instant client kill-switch execution on uninstall/takedown), and **RFC 8707 Resource Indicators for OAuth 2.0** (strict single-audience downscoping).

---

## 2. Technical Control Matrix: 15 Concrete Architectural Findings

| Finding ID | Standard Reference | Focus Domain | Key Architecture & Protocol Mechanisms | Super-App Host Implementation Requirements |
|---|---|---|---|---|
| `oidc_core_126_01` | **OpenID Connect Core 1.0** Sec 2 & 3.1.3.6 | ID Token Architecture, Core Claims & Temporal Boundaries | Cryptographically signed JWT assertion; claims: `iss`, `sub`, `aud`, `exp`, `iat`, `auth_time`, `nonce`, `acr`, `amr`, and `azp`; strict separation between authentication assertion and authorization credentials. | Issue signed ID Tokens from super-app identity broker upon user consent; enforce strict validation of temporal boundaries and client audience in mini-app SDK. |
| `oidc_core_126_02` | **OpenID Connect Core 1.0** Sec 3.1.3.7 | Normative ID Token Cryptographic Verification | Mandatory public key retrieval via published JWKS URI indexed by `kid`; enforcement of asymmetric algorithms (ES256/RS256); prohibition of 'none'; clock skew validation <= 30s. | Mandate asymmetric digital signatures for all mini-app ID Tokens; prohibit symmetric HS256 on public clients; automate key rotation via `/.well-known/jwks.json`. |
| `oidc_core_126_03` | **OpenID Connect Core 1.0** Sec 8 & 8.1 | Pairwise Pseudonymous Identifiers (PPID) & Anti-Correlation | Sector-specific pseudonymous subject derivation: `sub = SHA256(Sector_ID + User_ID + Salt)`; prevents cross-app profiling across unrelated third-party mini-apps; GDPR Art 25 alignment. | Mandate `subject_type: pairwise` as default for third-party mini-apps; reserve `public` subject types exclusively for certified first-party enterprise suites. |
| `oidc_core_126_04` | **OpenID Connect Core 1.0** Sec 5.3 & 5.4 | UserInfo Endpoint Architecture & Scoped Claims | OAuth 2.0 Protected Resource accessed via Bearer or DPoP; standard scopes (`profile`, `email`, `phone`, `address`); exact matching between UserInfo `sub` and ID Token `sub`. | Expose rate-limited UserInfo gateway via native bridge; verify `sub` match prior to user profile hydration; return signed JWTs (`application/jwt`) for enterprise tiers. |
| `oidc_core_126_05` | **OpenID Connect Core 1.0** Sec 16 & 16.11 | Security Considerations & Nonce Binding | Defense against token substitution, cut-and-paste, and CSRF; mandatory cryptographic `nonce` parameter in requests; URL fragment scrubbing via `history.replaceState`. | Enforce 128-bit entropy `nonce` parameter for all mini-app auth flows; isolate tokens in native container memory or encrypted OPFS; scrub tokens from WebView history. |
| `oidc_logout_126_06` | **OpenID Connect RP-Initiated Logout 1.0** Sec 2 & 3 | End-Session Endpoint, `id_token_hint` & Redirection | Standardized client-driven logout via `end_session_endpoint`; mandatory `id_token_hint` preventing logout DoS; pre-registered `post_logout_redirect_uri` validation; CSRF `state`. | Implement RFC-compliant end-session endpoint on super-app broker; mandate `id_token_hint` for bridge logout calls; reject unregistered post-logout redirect destinations. |
| `oidc_logout_126_07` | **OpenID Connect Back-Channel Logout 1.0** Sec 2.4 & 2.5 | Direct Server-to-Server `logout_token` JWT | Out-of-band HTTP POST webhooks to mini-app backends; signed `logout_token` JWT with `events` claim; omission of `nonce`; deterministic revocation independent of client state. | Deploy Back-Channel Logout webhooks on identity platform; require third-party backends to invalidate server sessions and refresh tokens on verified `logout_token` receipt. |
| `oidc_logout_126_08` | **OpenID Connect Front-Channel Logout 1.0** Sec 2 & 3 | Browser-Mediated Logout URIs & WebView Sandboxing | OP rendering hidden HTML iframes or native bridge triggers targeting `frontchannel_logout_uri`; passing `iss` and `sid`; client-side storage and session cookie cleanup. | Support Front-Channel Logout in mini-app runtime via container bridge event hooks (`onMiniAppLogoutRequested`); evict local cache and session state synchronously. |
| `oidc_logout_126_09` | **OpenID Connect Session Management 1.0** Sec 3 & 4 | Real-Time Session Status Polling & Native Parity | Monitoring OP login status without network overhead via `check_session_iframe` / postMessage hash verification; detecting 'changed' status; triggers step-up or teardown. | Replace polling iframes with native bridge telemetry (`onSessionStateChanged`); notify mini-apps immediately when host account status transitions or locks. |
| `oidc_logout_126_10` | **OpenID Connect Logout Specifications** Sec 4 | Security Considerations & DoS Defense | Defense against forged logout injection; requiring `id_token_hint` or user confirmation; verifying JWT signatures and audience; enforcing 60s webhook freshness windows. | Enforce signature and freshness checks on logout webhooks; provide host-level confirmation dialogs for unauthenticated logout requests; apply bounded delivery queues. |
| `oauth_gov_126_11` | **IETF RFC 8628** Sec 3 & 3.1 | OAuth 2.0 Device Authorization Grant | Cross-device authorization for constrained hardware (smart TVs, kiosks, POS); `/device/code` endpoint issuing `device_code`, `user_code`, `verification_uri_complete`; QR-code flow. | Implement RFC 8628 device flow for TV/kiosk mini-apps; render `verification_uri_complete` as scanned QR codes; enforce backoff penalties (`slow_down`) and 300s code expiry. |
| `oauth_gov_126_12` | **IETF RFC 7662** Sec 2 & 2.2 | OAuth 2.0 Token Introspection & Dynamic Verification | Standardized HTTP POST introspection endpoint; returns boolean `active` and metadata (`scope`, `client_id`, `sub`, `aud`, `exp`); internal API gateway verification of opaque tokens. | Deploy RFC 7662 introspection endpoint protected by mTLS; mandate gateway introspection for opaque mini-app tokens; verify audience match before authorizing API requests. |
| `oauth_gov_126_13` | **IETF RFC 7009** Sec 2 & 2.1 | OAuth 2.0 Token Revocation Protocol & Kill-Switch | Explicit HTTP POST token revocation endpoint; invalidating refresh tokens and cascading across associated access tokens; constant HTTP 200 response preventing scanning attacks. | Integrate RFC 7009 revocation with mini-app lifecycle; trigger instant token revocation upon mini-app uninstallation, permission revoking, or marketplace takedown. |
| `oauth_gov_126_14` | **IETF RFC 8707** Sec 2 & 2.2 | Resource Indicators for OAuth 2.0 & Audience Downscoping | Adding `resource` URI parameter to authorization/token requests; restricts token audience (`aud`) strictly to specified API endpoint; prevents cross-service token replay. | Require mini-apps to declare target APIs via RFC 8707 `resource` parameters; enforce single-audience restrictions on issued JWTs; support progressive token downscoping. |
| `oauth_gov_126_15` | **OAuth 2.0 Security Architecture** BCP Synthesis | Comprehensive Token Lifecycle Governance & Revocation | End-to-end architectural synthesis: constrained device flow + introspection + instant revocation + audience downscoping + sender-constrained DPoP/mTLS binding. | Establish unified Token Lifecycle Governance policy; broadcast revocation events across distributed edge gateways via Redis/Kafka; require MFA confirmation for device grants. |

---

## 3. Normative Standards Breakdown

### 3.1 OpenID Connect Core 1.0 Identity Federation & PPID Privacy
OpenID Connect Core 1.0 builds upon OAuth 2.0 to provide a standardized, cryptographically verifiable identity layer. In a super-app ecosystem hosting third-party mini-apps:
- **ID Token Issuance**: When a mini-app requests user sign-in, the super-app identity broker issues an ID Token signed with its private key (using ES256 or RS256). The token contains standard claims confirming the authentication event:
  - `iss`: Canonical issuer URL matching the discovery document (e.g. `https://id.superapp.vn`).
  - `sub`: Unique subject identifier. To preserve privacy across multi-tenant applications, the host enforces **Pairwise Pseudonymous Identifiers (PPID)**. The subject is calculated deterministically as:
    $$	ext{sub} = 	ext{HMAC-SHA256}(	ext{Developer\_Sector\_ID}, 	ext{Internal\_User\_UUID} \parallel 	ext{Master\_Salt})$$
    Consequently, Mini-App A and Mini-App B receive completely different `sub` values for the same physical user, making cross-app user tracking mathematically impossible.
  - `aud`: Pre-registered mini-app `client_id`.
  - `nonce`: Cryptographic challenge sent by the mini-app client in the authorization request, copied verbatim into the ID Token to prevent replay attacks.
  - `auth_time` & `acr`: Timestamps and assurance class references allowing mini-apps to enforce step-up authentication if the original login ceremony is older than required policy limits.

### 3.2 OpenID Connect Coordinated Session & Logout Lifecycle
Terminating user sessions in a federated super-app environment requires synchronizing the host mobile container, embedded mini-app WebViews, and remote third-party developer cloud backends:
1. **RP-Initiated Logout 1.0**: A mini-app can programmatically request user logout by invoking the host bridge or redirecting to the OP's `end_session_endpoint`. The request passes `id_token_hint` (validating that the caller owns the active session) and an optional `post_logout_redirect_uri` (which must be pre-registered during client provisioning). If `id_token_hint` is missing, the host must present an explicit user confirmation dialog before invalidating the session.
2. **Back-Channel Logout 1.0**: When the user signs out of the super-app account globally, the identity platform queries all registered mini-apps that hold active sessions for that user. The platform dispatches an asynchronous HTTP POST to each mini-app's registered `backchannel_logout_uri` containing a signed `logout_token` JWT. The `logout_token` contains:
   ```json
   {
     "iss": "https://id.superapp.vn",
     "sub": "ppid_user_hash_12345",
     "aud": "mini_app_client_id_888",
     "iat": 1728518400,
     "jti": "b0a45e7f-7123-4c92-bce4-672199bcf891",
     "events": {
       "http://schemas.openid.net/event/backchannel-logout": {}
     },
     "sid": "sess_8941_abcdef"
   }
   ```
   Crucially, RFC standards mandate that the `logout_token` **MUST NOT** include a `nonce` claim, ensuring it cannot be intercepted and submitted to user authentication endpoints. The third-party backend verifies the signature against the super-app JWKS and purges the corresponding local session and refresh tokens.
3. **Front-Channel Logout 1.0 & Bridge Parity**: For currently active WebView instances, the super-app container triggers the `onMiniAppLogoutRequested` native bridge event, allowing running client scripts to flush local caches, purge IndexedDB vaults, and reset DOM state before the WebView container is torn down.

### 3.3 IETF RFC 8628 Device Authorization Grant for Smart TV & Kiosk Terminals
Mini-apps deployed on smart displays, connected vehicle consoles, and retail POS kiosks operate under severe input constraints:
- **Protocol Flow**: The terminal mini-app calls the `/device/code` endpoint declaring its `client_id` and requested `scope`. The server returns:
  - `device_code`: High-entropy polling identifier kept confidential by the terminal.
  - `user_code`: Short, formatted alphanumeric code (e.g. `WDJB-MJHT`) designed for human entry.
  - `verification_uri`: Base URL for user entry on mobile browsers (e.g. `https://superapp.vn/device`).
  - `verification_uri_complete`: Full URL embedding the `user_code` as a query parameter.
- **QR-Code Optimization**: The terminal mini-app renders `verification_uri_complete` as an on-screen QR code. The user scans the QR code using their primary smartphone super-app, which displays an authorization prompt containing the terminal identity, location, and requested permissions.
- **Polling & Backoff**: The terminal polls the token endpoint at intervals defined by `interval` (default 5s). If the device polls too rapidly, the server returns HTTP 400 with `error: slow_down`, forcing a +5s backoff. Once authorized on mobile, the terminal's next poll returns access and refresh tokens, completing authentication without manual credential entry.

### 3.4 Token Introspection (RFC 7662), Revocation (RFC 7009) & Resource Indicators (RFC 8707)
Enterprise API gateways and internal service meshes require real-time token governance:
- **Token Introspection (RFC 7662)**: Protects internal microservices processing opaque tokens. Gateways post tokens to `/oauth/introspect` using mutual TLS (mTLS). If active, the response returns the authorized user identity, granting scopes, and client ID. If inactive or expired, it returns `{"active": false}` without leaking token metadata.
- **Instant Revocation (RFC 7009)**: When a mini-app is uninstalled by the user or quarantined by marketplace operators, the container issues an RFC 7009 revocation call. Revoking a refresh token immediately cascades across distributed cache clusters, terminating all derived access tokens.
- **Resource Indicators (RFC 8707)**: When requesting authorization or token exchange, mini-apps MUST supply a `resource` URI parameter targeting the specific API cluster (e.g. `https://finance-api.superapp.vn/v1`). The issued access token's audience (`aud`) is strictly clamped to this URI, guaranteeing that an access token issued for an e-commerce mini-app cannot be replayed against high-privilege banking microservices.

---

## 4. Architectural Implementation Blueprint

```
+---------------------------------------------------------------------------------------------------------+
|                                    SUPER-APP IDENTITY & RUNTIME CONTROL PLANE                           |
|                                                                                                         |
|   +-----------------------+     +-----------------------+     +-------------------------------------+   |
|   |  OpenID Provider (OP) |     | Dynamic Registration  |     |   Distributed Token Store & Cache   |   |
|   |   - Discovery (.well) |     |   - Software Stmt     |     |   - Distributed Revocation (Redis)  |   |
|   |   - JWKS (ES256/RS256)|     |   - DCR (RFC 7591)    |     |   - Active Session Index (sid)      |   |
|   |   - PPID Generator    |     |   - DCRM (RFC 7592)   |     |   - Token Introspection (RFC 7662)  |   |
|   +-----------+-----------+     +-----------+-----------+     +------------------+------------------+   |
|               |                             |                                    |                      |
+---------------+-----------------------------+------------------------------------+----------------------+
                |                                                                  |
                | (ID Token / PPID)                                                | (Back-Channel Logout)
                v                                                                  v
+----------------------------------------------------+          +-----------------------------------------+
|       MINI-APP CONTAINER RUNTIME (WebView / OS)    |          |   THIRD-PARTY MINI-APP CLOUD BACKEND    |
|                                                    |          |                                         |
|   +--------------------------------------------+   |          |   +---------------------------------+   |
|   | Native Container Bridge (Isolated Security)|   |          |   |  Back-Channel Webhook Receiver  |   |
|   |  - Nonce & PKCE S256 Verifier Generator    |   |          |   |   - Validates 'logout_token' JWT|   |
|   |  - Ephemeral Key Pair (WebCrypto non-ext)  |   |          |   |   - Invalidate Local Session    |   |
|   |  - DPoP Proof Generator (RFC 9449)         |   |          |   |   - Invalidate Refresh Tokens   |   |
|   +---------------------+----------------------+   |          |   +---------------------------------+   |
|                         |                          |          |                                         |
|   +---------------------v----------------------+   |          |   +---------------------------------+   |
|   | Mini-App Context (Sandboxed DOM Execution) |   |          |   |  Resource Server / API Adapter  |   |
|   |  - Receives Pairwise PPID (sub)            |   |          |   |   - Validates DPoP 'ath' & 'cnf'|   |
|   |  - Consumes Scoped UserInfo Claims         |   |          |   |   - Checks RFC 8707 'resource'  |   |
|   |  - Subscribes to 'onSessionStateChanged'   |   |          |   |   - Introspects Tokens (RFC 7662|   |
|   +--------------------------------------------+   |          |   +---------------------------------+   |
+----------------------------------------------------+          +-----------------------------------------+
```

---

## 5. Security & Verification Directives

1. **Zero Credential Exposure**: Client credentials, user passwords, and authorization codes must never appear in application logs, URI query parameters, or client-side storage. Authorization codes delivered via redirects must be purged immediately using `history.replaceState`.
2. **Mandatory Asymmetric Signing**: All ID Tokens and Back-Channel `logout_token` assertions must be signed using asymmetric cryptographic algorithms (ES256 or RS256). Symmetrical HMAC signatures on public clients and insecure `none` algorithms are rejected at the store review security gate.
3. **PPID Enforcement**: Store certification gates must verify that third-party mini-apps declare `subject_type: pairwise`. Any attempt to harvest or request unhashed global user IDs must be flagged as a critical privacy violation during automated code review.
4. **Idempotent Revocation Handshake**: To prevent enumeration attacks, RFC 7009 token revocation endpoints must return HTTP 200 OK for all structurally valid requests, regardless of whether the token was already revoked or expired.
5. **Circuit Breaking on Webhooks**: Super-app back-channel logout dispatch queues must enforce strict 5-second HTTP request timeouts and maximum retry counts (3 attempts with exponential backoff) to safeguard host infrastructure against uncooperative or slow third-party developer webhooks.
