# Topic Report 136: Oblivious HTTP (OHTTP), DAP Multi-Party Telemetry, Private State Tokens (PST) & W3C MiniApp Core Architecture

## Executive Summary

Milestone 136 establishes the foundational cryptographic standards for **zero-leakage telemetry**, **privacy-preserving anti-fraud attestation**, and **W3C MiniApp container architecture** for modern enterprise super-apps:

1. **IETF RFC 9458 Oblivious HTTP (OHTTP), RFC 9292 Binary HTTP (BHTTP) & IETF DAP (draft-ietf-ppm-dap)**: Breaks the fundamental linkage between user network identity (IP address, TLS fingerprints) and telemetry payloads through cryptographic gateway/relay separation, Hybrid Public Key Encryption (HPKE, RFC 9180), and Verifiable Distributed Aggregation Functions (VDAF Prio3 FLP proofs).
2. **IETF RFC 9497 Oblivious Pseudorandom Functions (OPRFs/VOPRF) & WICG Private State Tokens (PST)**: Provides mathematical foundations for blind-issued, unlinkable anti-fraud trust tokens. The super-app host issues signed trust tokens (`operation: 'token-request'`) that mini-apps redeem (`operation: 'token-redemption'`) to obtain cryptographic Redemption Records (RR), eliminating bot manipulation, ticket scalping, and ad fraud without tracking or sharing user identifiers.
3. **W3C MiniApp Working Group Standards Suite (MiniApp White Paper v2, MiniApp Manifest CR, MiniApp Packaging CR, MiniApp Lifecycle, and MiniApp Addressing)**: Codifies the official dual-process View/Logic runtime isolation model, declarative `subpackages` lazy loading, multi-origin packaging with per-file SHA-256 digest manifests and developer/platform dual-signatures, deterministic 5-stage lifecycle state machines with background resource suspension, and `miniapp://` deep-linking.

---

## 1. IETF RFC 9458 Oblivious HTTP (OHTTP) & IETF DAP Architecture

### 1.1 Non-Colluding Gateway-Relay Cryptographic Model
- **Architectural Entities**:
  - **Client**: Mini-app runtime or super-app native container wanting to submit telemetry or queries without exposing network identifiers.
  - **Oblivious Relay**: Receives encrypted requests from the client, terminates the outer client transport connection, strips client IP addresses, and forwards the ciphertext to the Gateway.
  - **Oblivious Gateway**: Decrypts the request using its private HPKE key, verifies configuration validity, and passes the inner Binary HTTP message to the Target Server.
  - **Target Server**: Evaluates business logic and responds with a Binary HTTP response, which the Gateway encrypts back to the client.
- **Security Invariants**:
  - Relay knows client IP address but *cannot* read message plaintext.
  - Gateway/Target knows message plaintext but *cannot* determine client IP address.
  - Non-collusion guarantee is enforced via administrative, physical, and legal isolation between Relay and Gateway operators.

### 1.2 Binary HTTP Framing & HPKE Encapsulation
- **Message Serialization**:
  - Outer encapsulation uses media types `message/ohttp-req` and `message/ohttp-res`.
  - Inner payloads are formatted strictly using RFC 9292 Binary HTTP (`message/bhttp`), eliminating ASCII delimiter parsing vulnerabilities.
- **Cryptographic Primitives**:
  - HPKE (RFC 9180) single-shot encapsulation: Client extracts Gateway Key Configuration (`key_id`, `kem_id`, `kdf_id`, `aead_id`, and `public_key`).
  - Ephemeral key encapsulation generates `enc` public key and AEAD ciphertext.
  - Response key material is derived via the HPKE exporter context, ensuring full end-to-end forward secrecy and replay resistance.

### 1.3 Distributed Aggregation Protocol (DAP) & Prio3 VDAF
- **Multi-Party Computation Model**:
  - Clients compute measurements (e.g., feature clicks, crash counters, latency buckets) and split inputs into additive secret shares distributed to two independent aggregators: **Leader** and **Helper**.
  - Aggregators execute zero-knowledge Fully Linear Proofs (FLP) to verify that client inputs conform to valid domain constraints (e.g., booleans or bounded integers) without reconstructing cleartext data.
- **Prio3 Algorithms**:
  - `Prio3Count`: Validates boolean inputs $\in \{0, 1\}$.
  - `Prio3Sum`: Validates bounded integer inputs $\in [0, 2^k - 1]$.
  - `Prio3Histogram`: Validates categorical selections $\in \{0, \dots, N-1\}$.
- **Privacy Guarantees**:
  - Collector receives only aggregate statistics ($\sum x_i$) over batches with size $K \ge 1000$.
  - Differential privacy loss ($\epsilon, \delta$) is bounded and tracked per task.

---

## 2. IETF RFC 9497 Oblivious PRFs & WICG Private State Tokens (PST)

### 2.1 Cryptographic Foundations: VOPRF & POPRF
- **OPRF Definition**: Two-party protocol allowing Client to learn $f(k, x)$ without revealing $x$ to Server or $k$ to Client.
- **Hash-to-Curve & Blinding**:
  - Client blinds input $x$: $P = \text{HashToGroup}(x)$, $A = r \cdot P$.
  - Server evaluates blinded point: $B = k \cdot A$.
  - Client unblinds: $C = r^{-1} \cdot B = k \cdot P$.
- **Verifiable OPRF (VOPRF)**:
  - Server returns a Discrete Logarithm Equality (DLEQ) proof verifying that $B$ was computed using the server's committed public key $Y = k \cdot G$.
- **Partially Oblivious PRF (POPRF)**:
  - Incorporates public metadata (token epoch, trust tier) into the evaluation point without breaking blindness.

### 2.2 Private State Token Issuance & Redemption Lifecycle
- **Issuance Protocol**:
  ```javascript
  // Super-app host or verified context triggers token issuance
  await fetch("https://trust.superapp.internal/issue-token", {
    privateToken: {
      version: 1,
      operation: "token-request"
    }
  });
  ```
  - Client transmits `Sec-Private-State-Token` request header.
  - Issuer evaluates blinded tokens, validates KYC/device integrity status, and returns unblindable signed tokens.
- **Redemption Protocol**:
  ```javascript
  // Mini-app requests redemption record before sensitive action
  await fetch("https://trust.superapp.internal/redeem-token", {
    privateToken: {
      version: 1,
      operation: "token-redemption",
      refreshPolicy: "none"
    }
  });
  ```
  - Issuer verifies token signature, checks double-spend database, and returns a cryptographically signed **Redemption Record (RR)**.
- **Sharing Redemption Records**:
  ```javascript
  // Mini-app conveys RR to third-party merchant backend
  await fetch("https://merchant.mini-app.com/checkout", {
    privateToken: {
      version: 1,
      operation: "send-redemption-record",
      issuers: ["https://trust.superapp.internal"]
    }
  });
  // Browser automatically injects: Sec-Redemption-Record: ...
  ```

### 2.3 Anti-Tracking & Rate-Limiting Guarantees
- **Partitioned Storage**: PST caches are strictly partitioned by top-level site and issuer origin.
- **Rate-Limiting & Low-Entropy Signals**:
  - Maximum 2-3 bits of metadata per token (e.g. `Tier_Basic`, `Tier_Verified`, `Tier_VIP`), preventing high-entropy user tracking.
  - Redemption attempts are throttled to prevent side-channel timing deanonymization.

---

## 3. W3C MiniApp Working Group Core Architecture

### 3.1 Dual-Process Architecture (W3C MiniApp White Paper v2)
- **View Thread (UI Component)**:
  - Sandboxed WebView rendering DOM, CSS layout, animations, and processing touch/keyboard input.
  - Strictly isolated from device hardware APIs and native execution capabilities.
- **Logic Thread (Worker Component)**:
  - Sandboxed JavaScript runtime (V8 Isolate or JavaScriptCore).
  - Executes application business logic, state machines, and data processing.
  - *No direct access to DOM objects* (`window`, `document` are undefined).
- **Native Host Container IPC Bridge**:
  - Mediates all communication between View and Logic asynchronously via serialized JSON/ArrayBuffer messages.
  - Completely eliminates DOM XSS privilege escalation into native device interfaces.

### 3.2 W3C MiniApp Manifest CR (`manifest.json`)
- **Core Declarations**:
  - `pages`: Declarative routing table specifying available views (e.g. `["pages/index/index", "pages/detail/detail"]`).
  - `window`: Global viewport presentation rules (title bar background, navigation text color, pull-to-refresh).
  - `tabBar`: Native bottom/top tab bar configuration with icon and route associations.
  - `permissions`: Pre-declared capability scopes required by the mini-app (e.g. `scope.camera`, `scope.userLocation`).
  - `subpackages`: Modular packaging configuration for dynamic on-demand chunk loading.

### 3.3 W3C MiniApp Packaging CR (`.zip` Container Structure)
- **Container Structure**:
  ```
  miniapp-package.zip/
  ├── manifest.json            # W3C MiniApp Manifest metadata
  ├── app-service.js           # Compiled Logic thread bundle
  ├── page-frame.html          # View thread shell
  ├── pages/                   # Page templates, styles & view scripts
  ├── assets/                  # Images, fonts, media
  └── signature.mf             # Per-file SHA-256 digest manifest & signatures
  ```
- **Integrity Validation**:
  - Container must include per-file SHA-256 digests.
  - Dual cryptographic signatures: Developer ECDSA signature (proves origin authorship) + Super-App Store Counter-Signature (proves store certification and security scanning approval).

### 3.4 W3C MiniApp Lifecycle & State Transitions
- **Application Lifecycle**:
  - `Launch`: Initial container spin-up and logic initialization.
  - `Foreground (Active)`: Interactive state dispatching `onShow()`.
  - `Background`: Inactive state dispatching `onHide()`. Container immediately suspends media, pauses timers, and throttles sensor/network access.
  - `Destroy`: Graceful teardown dispatching `onDestroy()` before OS memory reclamation.
- **Page Lifecycle**:
  - State progression: `onLoad()` -> `onShow()` -> `onReady()` -> `onHide()` -> `onUnload()`.

### 3.5 W3C MiniApp Addressing (`miniapp://` URI Scheme)
- **URI Grammar**:
  `miniapp://<miniapp-id>/<page-path>?<query-params>#<hash-fragment>`
- **Resolution Semantics**:
  - Native OS intent filters and universal link bridges intercept `miniapp://` URIs and route execution to the super-app host container.
  - Container enforces security policies: verifies target mini-app installation, evaluates cross-app invocation permissions, and passes sanitized query parameters into lifecycle parameters.

---

## 4. Synthesis: Verified Standards Reference Table

| Standard Reference | Lead Organization | Core Specification Components | Super-App Enterprise Implementation Focus |
|---|---|---|---|
| **IETF RFC 9458** | IETF | Oblivious HTTP (OHTTP), Relay/Gateway Trust Separation, BHTTP | Privacy-preserving telemetry, crash reporting & attribution egress |
| **IETF RFC 9292** | IETF | Binary Representation of HTTP Messages (`message/bhttp`) | Deterministic binary serialization for low-overhead secure messaging |
| **draft-ietf-ppm-dap** | IETF PPM WG | Distributed Aggregation Protocol (DAP), Prio3 VDAF, MPC FLP proofs | Verifiable multi-party telemetry aggregation with differential privacy |
| **IETF RFC 9497** | IETF CFRG | Oblivious Pseudorandom Functions (OPRFs), VOPRF, POPRF | Blinded token generation and cryptographic server evaluation |
| **WICG Private State Tokens** | W3C / WICG | PST Architecture, Token Request, Token Redemption, Redemption Records | Privacy-preserving cross-mini-app anti-bot and anti-fraud attestation |
| **W3C MiniApp White Paper v2** | W3C MiniApps WG | Dual-Process Architecture, View/Logic Isolation, Host IPC Bridge | Physical separation of rendering and logic to defeat DOM XSS escalation |
| **W3C MiniApp Manifest CR** | W3C MiniApps WG | `manifest.json`, Page Routing, Window Properties, `subpackages` | Automated static manifest validation and modular on-demand loading |
| **W3C MiniApp Packaging CR** | W3C MiniApps WG | Container Zip Format, Per-File SHA-256 Digest Manifest, Dual Signatures | Tamper-proof package distribution, integrity gates & store counter-signing |
| **W3C MiniApp Lifecycle** | W3C MiniApps WG | Application & Page State Machines, Background Throttling, Memory Reclaim | Synchronous lifecycle callbacks, background resource freezing & eviction |
| **W3C MiniApp Addressing** | W3C MiniApps WG | `miniapp://` URI Scheme, Deep-Linking, Inter-App Navigation Policies | Standardized universal routing, query sanitization & capability gating |
