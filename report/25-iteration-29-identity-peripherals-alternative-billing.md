# 25 — Iteration 29: Identity Federation, Peripheral Isolation, and Alternative Billing Governance

## 1. Executive Context & Scope

Iteration 29 closes three foundational architecture and compliance gaps in super-app mini-app ecosystem engineering:
1. **Identity Federation & Credential Management**: The migration from third-party cookies and tracking bounces toward browser-mediated identity via the **W3C Federated Credential Management (FedCM) API** and the **W3C Credential Management API Level 1**, securing guest mini-app single sign-on (SSO) and pairwise pseudonymous identifier (PPID) privacy.
2. **Hardware Peripheral Sandbox Isolation & Exploit Mitigations**: Deep defense against hardware-based sandbox escapes over **WICG WebUSB**, **WICG WebHID**, and **WICG Web Serial**, incorporating the peer-reviewed exploit models of Trampert et al. (*WWW '25: "Peripheral Instinct: How External Devices Breach Browser Sandboxes"*), establishing protected interface class exclusions, macro injection blocks, AT-command filtering, and a 3-tier container hardware policy matrix.
3. **Alternative In-App Billing & Anti-Steering Compliance**: Operationalizing cross-platform host store regulations, including **Google Play User Choice Billing** (24h automated reporting via ExternalTransactions API), the **Google Play Billing Choice Program** (differentiated 4% discount vs 10%/20% external link commissions through June 30, 2026), **Apple EU Alternative Payment Options** (StoreKit External Purchase Entitlements, system modal disclosure sheets, monthly reporting), **Apple EU unbundled economics** (10%/17% commissions, 3% PSP fee waivers, and €0.50 Core Technology Fee), and a **Super-App Unified Multi-Provider Billing Abstraction Layer**.

---

## 2. Topic A: Identity Federation & Credential Management

### 2.1 W3C FedCM Browser Mediation
- **Standard**: W3C Federated Credential Management API (`https://w3c-fedid.github.io/FedCM/`).
- **Core Mechanism**: Initiated via `navigator.credentials.get({ identity: { providers: [{ configURL, clientId }] } })`. The user agent natively intermediates identity exchange, eliminating third-party cookies, invisible iframes, and tracking bounces.
- **Privacy Assurance**: The Relying Party (RP mini-app) cannot detect whether the end-user maintains an account with the Identity Provider (IdP) until the user explicitly selects an account on a trusted, unforgeable native host bottom sheet.
- **Super-App Control**: The container intercepts FedCM calls or provides an equivalent native identity mediation layer, ensuring guest mini-apps cannot silently enumerate user identity profiles.

### 2.2 IdP HTTP API Architecture & Metadata Validation
- **Specification**: W3C FedCM §3 Identity Provider HTTP API.
- **Endpoint Topology**:
  - `/.well-known/web-identity`: Hosted on the eTLD+1 root domain, declaring authorized config file URLs to thwart subdomain impersonation.
  - `configURL`: JSON manifest declaring `accounts_endpoint`, `id_assertion_endpoint`, `client_metadata_endpoint`, and optional `disconnect_endpoint`.
  - `accounts_endpoint`: Returns active accounts (id, name, email, avatar) signed under IdP session credentials.
  - `id_assertion_endpoint`: Receives RP clientId, accountId, and client-generated cryptographic nonce via HTTP POST, returning the signed assertion token.
  - `disconnect_endpoint`: Synchronously informs the RP when an account link is revoked.
- **Super-App Control**: Super-app identity servers host FedCM-compliant manifests, enforce client nonce binding against replay attacks, and bind de-linking events directly to host account management settings.

### 2.3 Permissions Policy Delegation (`identity-credentials-get`)
- **Standard**: W3C Permissions Policy framework & W3C FedCM §4.
- **Gating**: The `identity-credentials-get` policy-controlled feature defaults to `'self'`. Cross-origin embedded iframes cannot invoke `navigator.credentials.get({ identity })` without explicit delegation (`allow="identity-credentials-get <origin>"`).
- **Anti-Clickjacking**: Embedded contexts strictly require transient user activation (user gesture) before triggering the credential dialog.
- **Super-App Control**: Mini-app WebViews enforce `'self'` by default; embedded third-party widgets inside mini-apps cannot initiate identity requests unless declared in the mini-app store capability manifest.

### 2.4 W3C Credential Management Container Architecture
- **Standard**: W3C Credential Management Level 1 (`https://w3c.github.io/webappsec-credential-management/`).
- **Unified Interface**: `navigator.credentials` manages `PasswordCredential`, `IdentityCredential` (FedCM), `PublicKeyCredential` (WebAuthn/FIDO2), and `OTPCredential` (SMS WebOTP).
- **Session Protection**: `navigator.credentials.preventSilentAccess()` disables silent auto-login following user logout, mandating explicit user interaction on subsequent sessions to prevent session hijack on shared devices.
- **Super-App Control**: Mini-app runtimes isolate credential storage per `appId`, ensuring guest mini-apps cannot inspect or overwrite credentials belonging to peer mini-apps or the host super-app shell.

### 2.5 Pairwise Pseudonymous Identifier (PPID) Scoping
- **Standard**: OpenID Connect Core 1.0 §8 / Super-App Identity Architecture.
- **Privacy Hazard**: Direct exposure of master user identifiers (phone number, national ID, master UUID) permits cross-mini-app user tracking without consent.
- **Mitigation Matrix**:
  - **OpenID**: App-scoped identifier generated via `HMAC-SHA256(Master_UID, AppID + Host_Salt)`.
  - **UnionID**: Developer-scoped identifier granted only to verified enterprise developers managing multiple sibling mini-apps.
  - **Step-Up Scopes**: Sensitive attributes (phone, real name, email) require explicit, fine-grained host modal consent.
  - **Ephemeral Auth Codes**: Authorization codes delivered to the client WebView expire in $\le 5$ minutes and are single-use, redeemed exclusively over server-to-server TLS via developer `appSecret`.

---

## 3. Topic B: Hardware Peripheral Sandbox Isolation & Exploit Mitigations

### 3.1 WebUSB Protected Interface Classes & Firmware Verification
- **Standard**: WICG WebUSB API (`https://wicg.github.io/webusb/`).
- **Normative Blocks**: Prohibits web script from claiming interfaces across 8 protected USB classes: Audio (0x01), Communications/CDC (0x02), HID (0x03), Mass Storage (0x08), Smart Card (0x0B), Video (0x0E), Audio/Video (0x10), and Wireless Controller (0xE0).
- **Super-App Control**: The container blocks `navigator.usb` by default (`Permissions-Policy: usb=()`). POS peripherals (thermal receipt printers, barcode scanners) require verified vendorId/productId allowlisting in the store manifest; raw mass storage or communication dongles are blocked at the native driver boundary.

### 3.2 WebHID Input Filtering & Keystroke Injection Defense
- **Standard**: WICG WebHID API (`https://wicg.github.io/webhid/`).
- **Input Filtering**: Normatively excludes Generic Desktop Page Keyboard (Usage Page 0x01, Usage 0x06), Keypad (0x07), and Mouse/Pointer (0x01/0x02) top-level collections from web access.
- **Super-App Control**: `Permissions-Policy: hid=()` defaults to `()`. Guest mini-apps are prohibited from accessing composite devices that pair HID keyboard endpoints with serial or barcode scanners, eliminating covert BadUSB keystroke injection into the host OS.

### 3.3 Web Serial Baud Rate & Modem AT-Command Mitigations
- **Standard**: WICG Web Serial API (`https://wicg.github.io/serial/`).
- **Exploit Vectors**: Unfiltered serial access allows malicious scripts to issue raw Hayes AT commands (`AT+CMGS`, `AT+CPIN`) to connected cellular baseband modems, intercepting 2FA SMS tokens, making premium calls, or locking SIM cards.
- **Super-App Control**: The native container blacklists cellular modems, GNSS receivers, and system diagnostic ports; outbound serial byte streams pass through container sanitizers inspecting for `AT+` signatures.

### 3.4 Hardware Sandbox Escape Mitigations (Trampert et al., WWW '25)
- **Security Research**: Trampert et al., *"Peripheral Instinct: How External Devices Breach Browser Sandboxes"* (WWW '25, ACM).
- **Documented Exploit Chains**:
  1. *Firmware Overwrite*: Reflashing connected RF dongles (e.g. Logitech Unifying) with malicious firmware via WebHID/WebUSB vendor requests.
  2. *Macro Reprogramming*: Injecting automated attack macros into programmable gaming mice/keyboards.
  3. *Modem Takeover*: Baseband manipulation via Web Serial.
  4. *Persistent Grant Abuse*: XSS or domain takeover inheriting persistent hardware permissions without re-prompting.
- **Super-App Defenses**:
  - **Zero Persistent Grants**: All peripheral permissions are ephemeral and terminate immediately when the mini-app is closed or navigated.
  - **DFU Blockade**: Native USB/HID hooks strip Device Firmware Update (DFU) endpoints and vendor-specific firmware flashing command sets.
  - **Physical Host Confirmation**: Explicit biometric or native button confirmation required before any hardware communication initiates.

### 3.5 Super-App 3-Tier Hardware Policy Matrix
```
+-----------------------------------------------------------------------------+
|                          SUPER-APP HARDWARE TIERS                           |
+-----------------------------------------------------------------------------+
| Tier 0: Default Guest Mini-Apps                                            |
| - Policy: usb=(), hid=(), serial=(), bluetooth=()                           |
| - Result: Zero direct access to host physical buses                         |
+-----------------------------------------------------------------------------+
| Tier 1: Standard Certified Accessories (High-Level JSAPIs)                 |
| - Bridge: container.posPrinter.print(), container.barcodeScanner.scan()      |
| - Architecture: Super-app native container manages driver, buffers, and     |
|   error handling; no raw bus access exposed to guest JavaScript             |
+-----------------------------------------------------------------------------+
| Tier 2: Certified Enterprise Hardware (Industrial / IoT Partners)           |
| - Prerequisite: Hardware vendor attestation, VID/PID allowlist, manual code  |
|   audit, mutual TLS hardware authentication                                 |
| - Controls: Ephemeral session grants, DFU endpoint blocking, bus rate caps  |
+-----------------------------------------------------------------------------+
```

---

## 4. Topic C: Alternative In-App Billing & Anti-Steering Compliance

### 4.1 Google Play User Choice Billing Architecture
- **Specification**: Google Play Help Center (answer/13821247 & answer/12570971).
- **Regional Footprint**: Available in over 35 markets, including the EEA, US, UK, South Korea, India, Japan, Brazil, Australia, and Indonesia.
- **Dual-Screen Mechanism**: The container presents a standardized choice screen displaying Google Play Billing alongside the developer's registered alternative billing option with supported payment logos.
- **Fee Discount**: Transactions via alternative billing receive a **4% service fee reduction** against standard rates.
- **Mandatory Automated Reporting**: Developers must integrate Google Play's `ExternalTransactions API`, reporting authorized transactions within **24 hours** of authorization.

### 4.2 Google Play Billing Choice Program & External Web Links
- **Specification**: Google Play Help Center (answer/17161464) effective through June 30, 2026 (UK, EEA, US).
- **Monetization Paths**:
  - *Alternative In-App Billing*: 4% discount against standard tier.
  - *External Web Links*: 10% commission on auto-renewing subscriptions and first $1M USD annual revenue; 20% commission on other in-app digital content linked to web checkout.
- **Consumer Protection Obligations**: Mandatory refund workflows, order history links, subscription self-service management URLs, and dispute resolution mechanisms.

### 4.3 Apple EU Alternative Payment Framework (DMA Compliance)
- **Specification**: Apple Developer Support (`alternative-payment-options-in-the-eu`).
- **StoreKit External Purchase Entitlement**: Enables developers to integrate alternative Payment Service Providers (PSPs) in-app or link out via `StoreKit.ExternalPurchase`.
- **System Disclosure Sheet**: Native iOS modal sheet automatically displayed prior to external checkout, detailing that Apple does not handle refunds, family sharing, or purchase protection.
- **Audit & Reporting**: Mandatory monthly transaction reporting to Apple with cryptographically verifiable server reconciliation endpoints.

### 4.4 Apple EU Unbundled Commercial Economics
- **Specification**: Apple Alternative Terms Addendum for EU Apps.
- **Cost Elements**:
  1. *Reduced Base Commission*: **10%** (Small Business Program / subscriptions >1 yr) or **17%** (standard digital goods).
  2. *Payment Processing Surcharge*: Additional **3%** fee charged only if using Apple In-App Purchase (waived if using alternative PSP or external web checkout).
  3. *Core Technology Fee (CTF)*: **€0.50** for each first annual install per year exceeding 1 million installs.
- **Super-App Impact**: Because super-apps routinely exceed 1M installs, CTF costs must be factored into developer revenue-share models; using alternative PSPs captures the 3% savings to offset infrastructure costs.

### 4.5 Super-App Unified Multi-Provider Billing Abstraction Layer
```
                                 [ Guest Mini-App ]
                                         │
                                         ▼
                     container.requestPayment({ orderId, amount })
                                         │
                                         ▼
                 ┌───────────────────────────────────────────────┐
                 │  Super-App Unified Billing Abstraction Engine │
                 └───────────────────────┬───────────────────────┘
                                         │
                  Inspect: OS / Channel / Region / Compliancy
                                         │
       ┌──────────────────┬──────────────┴─────┬──────────────────┐
       ▼                  ▼                    ▼                  ▼
[Apple StoreKit    [Google Play         [Apple StoreKit    [Native Telco /
 IAP / External]    User Choice]         External Web]      Fintech Direct]
       │                  │                    │                  │
       └──────────────────┴──────────────┬─────┴──────────────────┘
                                         │
                                         ▼
                       Unified Publisher Settlement Ledger
               (Reconciles Commissions, PSP Fees, 24h Reporting,
                        and Net Developer Disbursements)
```

---

## 5. Standard Recommendations & Conformance Matrix

| Requirement | Category | Evidence Level | Specification Source |
|---|---|---|---|
| Browser-Mediated Identity Federation | Mandatory (P0) | Normative Standard | W3C FedCM §2 / MDN FedCM API |
| Identity Provider Endpoint Schemas | Mandatory (P0) | Normative Standard | W3C FedCM §3 IdP HTTP API |
| Permissions Policy `identity-credentials-get` | Mandatory (P0) | Normative Standard | W3C Permissions Policy / FedCM §4 |
| Unified Credential Container & Silent Access Guard | Mandatory (P0) | Normative Standard | W3C Credential Management Level 1 §2 |
| Pairwise Pseudonymous Identifier Scoping | Mandatory (P0) | Platform Practice | OpenID Connect Core §8 / Super-App Identity |
| WebUSB Protected Interface Class Blockade | Mandatory (P0) | Normative Standard | WICG WebUSB API §4.3 |
| WebHID Generic Keyboard/Mouse Input Filtering | Mandatory (P0) | Normative Standard | WICG WebHID API §3 / §4 |
| Web Serial Cellular Modem AT-Command Defense | Mandatory (P0) | Normative Standard | WICG Web Serial API §7 |
| Hardware Exploit Defenses (DFU / Transient Grants) | Mandatory (P0) | Platform Practice | Trampert et al., WWW '25 / W3C DAS Charter |
| 3-Tier Container Hardware Policy Matrix | Architecture Proposal | Platform Practice | Super-App Enterprise Hardware Practice |
| Google Play User Choice Billing & 24h Reporting | Mandatory (Host P0) | Platform Practice | Google Play Help Center (answer/13821247) |
| Google Play Billing Choice Fee Tiers (4% vs 10%/20%) | Operational Policy | Platform Practice | Google Play Help Center (answer/17161464) |
| Apple EU StoreKit External Purchase Entitlements | Mandatory (Host P0) | Platform Practice | Apple Developer Support (EU Alternative Payments) |
| Apple EU Unbundled Economics & CTF Attribution | Finance / Governance | Platform Practice | Apple Alternative Terms Addendum / CTF Spec |
| Unified Multi-Provider Billing Abstraction Layer | Architecture Pattern | Platform Practice | Super-App Commerce Architecture Practice |

---

## 6. Verification & Audit Trail

- **Candidate Records**: 15 records validated across `state/candidates/fedcm-credential-management.jsonl`, `peripherals-webusb-webhid-webserial.jsonl`, and `alternative-billing-anti-steering.jsonl`.
- **Secret Scanning**: Zero credentials, tokens, or private keys detected.
- **Citation Verification**: 20 unique URLs checked and confirmed with HTTP 200.
- **Append-Only Integrity**: `state/findings.jsonl` verified via backup and line count audit (grew from 371 to 386).
- **Public Repo Sync**: Preserved for iteration 30 milestone batching.
