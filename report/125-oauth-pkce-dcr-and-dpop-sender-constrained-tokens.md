# Module 125: OAuth 2.0 PKCE, Discovery, Dynamic Client Registration & DPoP Sender-Constrained Architecture

## 1. Executive Summary & Normative Scope

This module formalizes the core identity, discovery, automated client lifecycle, and sender-constrained token security standard for the super-app and mini-app runtime ecosystem. Modern super-apps function as distributed execution engines hosting hundreds or thousands of heterogeneous mini-apps developed by independent organizations. In such multi-tenant, untrusted environments, legacy OAuth 2.0 implementations suffer from acute vulnerabilities:
1. **Authorization Code Interception**: Public clients running in mobile WebViews or custom schemes can have redirected authorization codes stolen by malicious apps or adjacent cross-context scripts.
2. **Configuration Drift & Impersonation**: Hardcoding endpoints and manually provisioning client secrets leads to credential leakage, operational bottlenecks, and vulnerability to Mix-Up attacks.
3. **Bearer Token Theft & Replay**: Intercepted bearer tokens can be used indiscriminately by attackers across disparate API gateways without proof of caller identity.

To establish zero-trust identity isolation, this module synthesizes four authoritative IETF standards:
- **IETF RFC 7636**: *Proof Key for Code Exchange by OAuth Public Clients (PKCE)*, mandating SHA-256 (`S256`) code challenge derivation and eliminating authorization code interception.
- **IETF RFC 8414 & RFC 9207**: *OAuth 2.0 Authorization Server Metadata* and *OAuth 2.0 Authorization Server Issuer Identification in Authorization Responses*, standardizing well-known configuration discovery and neutralizing authorization mix-up attacks.
- **IETF RFC 7591 & RFC 7592**: *OAuth 2.0 Dynamic Client Registration Protocol* and *Dynamic Client Registration Management*, establishing automated client onboarding, marketplace review authority software statements, and HTTP DELETE emergency kill-switch revocation.
- **IETF RFC 9449**: *OAuth 2.0 Demonstrating Proof of Possession (DPoP) at the Application Layer*, binding access and refresh tokens to asymmetric client key pairs via `dpop+jwt` proofs, access token hash (`ath`) validation, server-provided nonces, and hardware-backed WebCrypto isolation.

---

## 2. Technical Control Matrix: 15 Concrete Architectural Findings

| Finding ID | Standard Reference | Focus Domain | Key Architecture & Protocol Mechanisms | Super-App Host Implementation Requirements |
|---|---|---|---|---|
| `oauth_core_125_01` | **IETF RFC 7636** Sec 4 & 4.1 | PKCE Protocol Flow & `S256` Challenge Derivation | Cryptographically random `code_verifier` (43-128 chars, >=256 bits entropy); `code_challenge` = BASE64URL(SHA256(verifier)); token exchange verification; prohibition of 'plain'. | Mandate RFC 7636 PKCE with `code_challenge_method=S256` for all mini-app OAuth authorization code flows; fail-closed on 'plain' or omitted PKCE. |
| `oauth_core_125_02` | **IETF RFC 7636** Sec 1 & 7.1 | Authorization Code Interception Defense | Mitigating custom URI scheme collision and WebView redirect interception; binding code to the client instance possessing ephemeral secret; one-time code invalidation. | Implement one-time use and strict client-instance binding for all authorization codes; invalidate pending tokens if mismatched verifier is presented. |
| `oauth_core_125_03` | **IETF RFC 8414** Sec 2 & 3 | Authorization Server Metadata Discovery | RFC 8414 `/.well-known/oauth-authorization-server` endpoint; standardized JSON document declaring endpoints, scopes, response types, and signing algorithms; strict issuer validation. | Publish super-app identity server metadata conforming to RFC 8414; enforce strict issuer URI matching in mini-app SDK discovery clients. |
| `oauth_core_125_04` | **IETF RFC 8414** Sec 2 & 7 | Capability Advertisement & Security Gating | Advertising `code_challenge_methods_supported: ["S256"]`, token auth methods, and supported JWS algorithms; TLS 1.3 transport security; signed metadata validation. | Include `S256` in metadata advertisement; reject weak legacy ciphers (none, RS128); mandate TLS 1.3 certificate validation in SDK discovery clients. |
| `oauth_core_125_05` | **IETF RFC 9207** Sec 2 & 3 | Issuer Identification & Mix-Up Attack Defense | Returning `iss` parameter in authorization response; client-side verification against target IdP issuer URI; preventing code redirection to malicious servers. | Mandate RFC 9207 `iss` in all authorization responses returned to mini-apps; enforce SDK check verifying `iss` before making token requests. |
| `oauth_dcr_125_06` | **IETF RFC 7591** Sec 2 & 3 | Dynamic Client Registration (DCR) Protocol | RESTful registration endpoint (`registration_endpoint`); automated client onboarding; standard metadata parameters (`redirect_uris`, `client_name`, `token_endpoint_auth_method`). | Expose RFC 7591 dynamic client registration endpoint on super-app identity gateway; enforce package origin validation on registered redirect URIs. |
| `oauth_dcr_125_07` | **IETF RFC 7591** Sec 2.3 & 3.1.1 | Software Statement Attestation (SSA) | JWS-signed `software_statement` issued by trusted marketplace review authority; cryptographic binding of scopes and publisher attributes; tamper-proof onboarding. | Require third-party mini-apps to present signed software statement from the Review Authority; fail-closed on invalid or unapproved statements. |
| `oauth_dcr_125_08` | **IETF RFC 7591** Sec 2 (Client Auth) | Client Authentication & JWKS Association | Client authentication methods (`none` + PKCE for public clients, `private_key_jwt` for confidential backends); JWKS endpoint association; deprecation of static secrets. | Enforce `token_endpoint_auth_method: none` + PKCE for frontend-only mini-apps; mandate `private_key_jwt` with JWKS for backend services; ban static secrets. |
| `oauth_dcr_125_09` | **IETF RFC 7592** Sec 2 & 3 | Dynamic Client Registration Management Lifecycle | Client Configuration Endpoint; HTTP GET (read), PUT (update), DELETE (revoke); authentication via scoped `registration_access_token`; unique management URIs. | Implement RFC 7592 endpoints for automated mini-app lifecycle management during version upgrades; trigger compliance re-scans upon metadata modification. |
| `oauth_dcr_125_10` | **IETF RFC 7592** Sec 2.1 & 3.3 | Registration Token Rotation & Kill-Switch Execution | Registration access token rotation on modification; immediate de-provisioning and token invalidation via HTTP DELETE; emergency kill-switch integration. | Enforce instant token revocation and client invalidation across all gateway nodes upon HTTP DELETE; integrate marketplace kill-switch directly with RFC 7592. |
| `oauth_dpop_125_11` | **IETF RFC 9449** Sec 4 & 4.2 | DPoP Proof Architecture & Application-Layer Binding | Demonstrating Proof of Possession via signed `DPoP` header; `typ: dpop+jwt`; mandatory claims (`jti`, `htm`, `htu`, `iat`); public key JWK in header; bearer vulnerability mitigation. | Mandate RFC 9449 DPoP proofs for high-privilege and payment APIs; enforce strict validation of `typ`, HTTP method (`htm`), and URI (`htu`) on gateways. |
| `oauth_dpop_125_12` | **IETF RFC 9449** Sec 4.1 & 4.2 | Access Token Hash (`ath`) & Confirmation (`cnf`) | Access token confirmation claim `cnf.jkt` (RFC 7638 SHA-256 thumbprint); DPoP proof `ath` claim (SHA-256 of access token); cross-token substitution prevention. | Issue access tokens containing `cnf.jkt`; verify that `ath` matches presented token hash before authorizing native bridge operations; reject mismatches with 401. |
| `oauth_dpop_125_13` | **IETF RFC 9449** Sec 8 & 11.1 | Server-Provided Nonces & Distributed Replay Mitigation | Server challenge via `DPoP-Nonce` header; `use_dpop_nonce` error response flow; transparent client retry with fresh nonce; milliseconds validity window. | Implement `DPoP-Nonce` challenge flows on sensitive payment endpoints; configure mini-app network adapters to auto-retry on `use_dpop_nonce`; cap nonce TTL at 60s. |
| `oauth_dpop_125_14` | **IETF RFC 9449** Sec 5 & 7 | DPoP Token Endpoint Binding & Refresh Lifecycle | `token_type: DPoP` in token responses; `Authorization: DPoP <token>` presentation; binding refresh tokens to client DPoP key pair; end-to-end sender-constraint. | Configure token endpoints to return `token_type: DPoP`; mandate `Authorization: DPoP <token>` for protected bridge calls; bind refresh tokens to client DPoP keys. |
| `oauth_dpop_125_15` | **IETF RFC 9449** Sec 11 & 11.2 | Private Key Isolation & Web Crypto Integration | Hardware-backed keystore integration (Android Keystore / iOS Keychain); WebCrypto non-extractable keys (`extractable: false`); DOM XSS exfiltration mitigation. | Mandate non-extractable CryptoKey generation in mini-app runtimes; isolate DPoP private keys per mini-app origin; encapsulate signing in core bridge SDK. |

---

## 3. End-to-End Architectural Workflows

### 3.1. Automated Mini-App Registration with Software Statement Attestation (RFC 7591 / RFC 7592)

```
[Marketplace Review Authority]          [Mini-App Developer CI/CD]               [Super-App IdP Gateway]
              │                                      │                                      │
              │ 1. Validate package & certs          │                                      │
              │ 2. Issue signed software_statement   │                                      │
              ├─────────────────────────────────────>│                                      │
              │                                      │ 3. POST /register                    │
              │                                      │    { software_statement: "...",      │
              │                                      │      redirect_uris: [...] }          │
              │                                      ├─────────────────────────────────────>│
              │                                      │                                      │ 4. Verify SSA signature
              │                                      │                                      │ 5. Provision client_id
              │                                      │ 6. HTTP 201 Created                  │
              │                                      │    { client_id, reg_access_token,    │
              │                                      │      registration_client_uri }       │
              │                                      │<─────────────────────────────────────┤
```

### 3.2. Authorization Code Exchange with PKCE & Issuer Verification (RFC 7636 / RFC 9207)

```
[Mini-App Sandbox (WebView)]             [Super-App Native Host / IdP]           [Mini-App Backend / Gateway]
              │                                      │                                      │
              │ 1. Generate code_verifier (>=256-bit)│                                      │
              │ 2. Compute code_challenge (S256)     │                                      │
              │ 3. GET /authorize?code_challenge=... │                                      │
              ├─────────────────────────────────────>│                                      │
              │                                      │ 4. Authenticate user & grant consent │
              │ 5. Redirect: ?code=XYZ&iss=IDP_URI   │                                      │
              │<─────────────────────────────────────┤                                      │
              │ 6. Verify iss == target IdP URI      │                                      │
              │ 7. POST /token + code_verifier + DPoP│                                      │
              ├────────────────────────────────────────────────────────────────────────────>│
              │                                      │                                      │ 8. Verify S256(verifier)
              │                                      │                                      │ 9. Verify DPoP proof
              │ 10. HTTP 200 { token_type: "DPoP",   │                                      │ 10. Return DPoP token
              │                access_token: "..." } │                                      │
              │<────────────────────────────────────────────────────────────────────────────┤
```

### 3.3. DPoP Sender-Constrained API Invocation (RFC 9449)

```
[Mini-App Runtime]                                                           [Super-App Protected API Gateway]
        │                                                                                   │
        │ 1. Sign DPoP Proof: { typ: "dpop+jwt", htm: "POST", htu: "/v1/pay", ath: ... }    │
        │ 2. POST /v1/pay                                                                   │
        │    Authorization: DPoP <access_token>                                             │
        │    DPoP: <dpop_proof_jwt>                                                         │
        ├──────────────────────────────────────────────────────────────────────────────────>│
        │                                                                                   │ 3. Check nonce freshness
        │                                                                                   │ 4. Verify cnf.jkt == thumbprint(jwk)
        │                                                                                   │ 5. Verify ath == SHA256(access_token)
        │ 6. HTTP 200 OK { status: "processed" }                                            │ 6. Execute transaction
        │<──────────────────────────────────────────────────────────────────────────────────┤
```

---

## 4. Normative Conformance Checklist for Enterprise & Regulated Super-Apps

1. **PKCE Enforcement**: All authorization code requests originating from mini-apps MUST supply `code_challenge` derived via `S256`. The authorization server MUST reject requests with missing PKCE or `code_challenge_method=plain`.
2. **Mix-Up Protection**: All authorization responses MUST include the `iss` parameter as defined in RFC 9207. Mini-app client runtimes MUST validate that the returned `iss` matches the configured issuer identifier.
3. **Dynamic Client Governance**: Client onboarding MUST utilize RFC 7591 dynamic client registration accompanied by a verifiable, cryptographically signed `software_statement`. Direct modification or deletion of client configurations MUST strictly adhere to RFC 7592.
4. **Sender-Constrained DPoP Binding**: Sensitive operations (financial payments, personal data export, native device control) MUST require RFC 9449 DPoP proofs. Bearer tokens MUST NOT be permitted on high-privilege endpoints.
5. **Private Key Hardening**: Client DPoP signing keys MUST be generated using `crypto.subtle.generateKey` with `extractable: false` and isolated per mini-app origin, backed by hardware security modules (TEE / StrongBox / Secure Enclave) where available.
