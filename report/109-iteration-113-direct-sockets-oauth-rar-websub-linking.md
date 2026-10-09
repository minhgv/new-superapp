# Iteration 113: WICG Direct Sockets & Fenced Frames Sandboxing, IETF RFC 9396 RAR & RFC 9470 Step-Up Authentication, and W3C WebSub & RFC 8288 Web Linking

## Executive Summary
Iteration 113 expands the Super App Mini App Store Standard across three mission-critical architectural frontiers:
1. **WICG Direct Sockets & HTML Fenced Frames Sandboxing**: Low-level networking governance establishing strict origin confinement, Permissions Policy gating, port blocklists against intranet traversal, and cryptographically isolated `<fencedframe>` embedding boundaries that replace legacy, leaky iframes for third-party widgets and ads.
2. **IETF RFC 9396 Rich Authorization Requests & RFC 9470 Step-Up Authentication**: Modern OAuth 2.0 security extensions replacing coarse, overloaded scope strings with structured JSON `authorization_details` for fine-grained mini-app transactions, alongside dynamic `WWW-Authenticate` step-up challenges (`acr_values`, `max_age`) requiring biometric or hardware token verification for elevated operations.
3. **W3C WebSub & W3C ActivityPub / RFC 8288 Web Linking**: Decentralized hypermedia discovery and real-time event syndication frameworks, utilizing standardized HTTP `Link` header semantics (`rel="hub"`, `rel="self"`, `rel="next"`), cryptographically verified WebSub webhook delivery (`hub.challenge`), and ActivityStreams 2.0 JSON-LD vocabularies for inter-mini-app federated audit trails.

---

## 1. WICG Direct Sockets & HTML Fenced Frames Sandboxing

### 1.1 Direct Sockets Architecture & Confinement Model
The WICG Direct Sockets specification enables web applications in highly secure, isolated contexts to establish raw bidirectional TCP and UDP socket connections without requiring intermediate WebSocket or HTTP gateways.

* **Confinement Prerequisites**: Direct Sockets are strictly prohibited in standard web origins and cross-origin iframes. In super-app platforms, Direct Sockets require an **Isolated Context**—specifically, an application package signed by a trusted platform certificate where all subresources are served with cryptographic integrity guarantees (matching Isolated Web App paradigms).
* **Port Blocklisting & Intranet Defense**: To defend internal enterprise intranets against SSRF, cross-protocol abuse (e.g., targeting internal Redis, memcached, or SMTP servers), and printer exploitation, the runtime blocks standard reserved ports:
  * Port 21 (FTP), 22 (SSH), 23 (Telnet), 25 (SMTP), 53 (DNS), 80/443 (HTTP/HTTPS), 445 (SMB), 6379 (Redis), 11211 (Memcached).
* **Permissions Policy & Transient Activation**: Governed by the `direct-sockets` Permissions Policy directive. Any call to `navigator.serial`, `TCPSocket`, or `UDPSocket` requires transient user activation (trusted physical gesture).

| Standard / API | Scope & Interface | Security Sandbox Boundary | Super App Store Governance Rule |
| :--- | :--- | :--- | :--- |
| **WICG Direct Sockets** | `TCPSocket`, `UDPSocket` | Isolated Web App context, IP literal checks, restricted port blocklist | Strictly quarantined from public mini-apps; reserved for certified POS hardware extensions |
| **Chrome Status (Feature 6398297361088512)** | Origin isolation & signed code packages | Transient user activation required; zero outbound privilege for unverified origins | Container-level permission prompt; automated teardown upon app backgrounding |
| **Direct Sockets Explainer** | Threat modeling & DNS rebinding defense | Credentials-free byte streams; platform resolver anti-rebinding | Real-time bandwidth throttling and socket audit logging in container gateway |

### 1.2 HTML <fencedframe> & FencedFrameConfig
The HTML `<fencedframe>` element represents an embedded top-level browsing context engineered to prevent cross-site identity leakage and side-channel communication.

* **Opaque URL Containment**: Instead of exposing destination URLs to JavaScript, the super-app embedder provides an opaque token (`FencedFrameConfig` or `urn:uuid`). The embedding document cannot introspect the rendered content, frame hierarchy, or destination endpoints.
* **Communication Barrier**: Fenced frames cannot communicate with the parent window via `window.postMessage`, `window.parent`, or shared DOM nodes. Network communication can be revoked upon accessing cross-site identity state, preventing user-tracking side channels.
* **Super App Application**: Ideal for embedding third-party sponsored catalog widgets, merchant affiliate banners, and personalized recommendation units within mini-apps without leaking host user profile attributes or financial identifiers.

---

## 2. IETF RFC 9396 Rich Authorization Requests & RFC 9470 Step-Up Authentication

### 2.1 RFC 9396 Rich Authorization Requests (RAR)
Legacy OAuth 2.0 implementations rely on flat, space-delimited string scopes (e.g., `scope="payment read write"`), which fail to capture multidimensional transactional constraints.

* **`authorization_details` JSON Structure**: RFC 9396 standardizes a structured JSON array containing typed authorization objects:
  ```json
  {
    "type": "mini_app_payment",
    "actions": ["transfer"],
    "locations": ["https://wallet.superapp.vn/api/v1/"],
    "instructed_amount": {
      "currency": "VND",
      "amount": "1500000"
    },
    "beneficiary": {
      "merchant_id": "MERC-98234-VN",
      "merchant_name": "Highlands Coffee Flagship"
    }
  }
  ```
* **Consent Integrity & Token Introspection**: The super-app authorization server parses the `authorization_details` schema, renders an unambiguous, human-auditable consent screen to the user, and binds the approved transaction payload directly into a cryptographically signed token (DPoP or MTLS-bound). The mini-app cannot reuse the issued token for secondary unapproved debits.

### 2.2 RFC 9470 Step-Up Authentication Challenge Protocol
When a mini-app attempts an elevated or high-risk transaction (e.g., high-value funds transfer, biometrics update, contract e-signing), the protected resource enforces on-demand authentication step-up without terminating the general user session.

* **Challenge Syntax**:
  ```http
  HTTP/1.1 401 Unauthorized
  WWW-Authenticate: Bearer error="insufficient_user_authentication",
                           error_description="Transaction requires biometric confirmation",
                           acr_values="urn:superapp:auth:fido2:biometric",
                           max_age="300"
  ```
* **Protocol Flow**:
  1. Mini-app client presents existing bearer/DPoP access token to the resource server.
  2. Resource server detects insufficient ACR (Authentication Context Class Reference) or stale authentication age (`max_age` exceeded).
  3. Resource server returns HTTP 401 with RFC 9470 `WWW-Authenticate` header.
  4. Mini-app runtime invokes the super-app host identity bridge to perform on-device WebAuthn/FIDO2 biometric verification.
  5. Authorization server issues an elevated access token reflecting the new `auth_time` and `acr`, enabling the mini-app to execute the transaction.

---

## 3. W3C WebSub & W3C ActivityPub / RFC 8288 Web Linking

### 3.1 W3C WebSub (Pub/Sub Syndication)
W3C WebSub defines a standardized, decentralized publish-subscribe mechanism built on HTTP to eliminate costly polling for dynamic content.

* **Hub Discovery**: Publishers advertise their associated hub and canonical topic URLs using RFC 8288 Link headers:
  ```http
  Link: <https://hub.superapp.vn/endpoint>; rel="hub",
        <https://catalog.superapp.vn/miniapps/updates>; rel="self"
  ```
* **Cryptographic Subscription Handshake**:
  * The subscriber issues a subscription request to the hub specifying `hub.callback`, `hub.mode=subscribe`, `hub.topic`, and `hub.secret`.
  * The hub validates subscriber ownership by issuing an asynchronous GET request to the callback URL containing `hub.challenge`. The subscriber must echo back the challenge string.
  * Update notifications are pushed via HTTP POST with `X-Hub-Signature: sha256=...` HMAC verification.
* **Super App Store Deployment**: Provides real-time catalog invalidation, immediate price update broadcasts, and emergency mini-app kill-switch dissemination to edge gateways and distributed mini-app instances.

### 3.2 RFC 8288 Web Linking Framework
RFC 8288 defines the formal hypermedia linking model and strict parsing algorithms for the HTTP `Link` header:
* Standardizes typed relations across mini-app store discovery endpoints:
  * `rel="next"` and `rel="prev"` for deterministic catalog pagination.
  * `rel="privacy-policy"` and `rel="terms-of-service"` for automated legal compliance crawlers.
  * `rel="preload"` and `rel="modulepreload"` for edge-accelerated runtime script and WebAssembly prefetching.

### 3.3 W3C ActivityPub & Activity Streams 2.0 Core
Provides a standardized JSON-LD object model for cross-mini-app social feeds, developer release announcements, and federated audit logging.
* Core primitives: `Actor` (mini-app or user entity), `Activity` (Create, Update, Suspend, Review), and `Object` (mini-app package, review comment, order).
* Standardizes administrative event feeds across enterprise tenants without proprietary lock-in.

---

## 4. Normative Implementation Requirements for Super App Mini App Store

1. **Direct Sockets Quarantining**: Public third-party mini-apps MUST NOT be granted `direct-sockets` permissions. Direct Sockets MUST be reserved exclusively for verified enterprise hardware extensions operating in Isolated Contexts with strict port blocklists.
2. **Third-Party Ad Sandboxing via Fenced Frames**: Mini-app developer guidelines MUST mandate the use of `<fencedframe>` and `FencedFrameConfig` for third-party programmatic ad units and affiliate widgets, completely forbidding leaky cross-origin iframes.
3. **Structured Financial Authorizations via RFC 9396**: All transactional payment requests between mini-apps and the host super-app wallet MUST utilize RFC 9396 `authorization_details` JSON schemas, strictly outlawing ambiguous string scopes.
4. **Biometric Elevation via RFC 9470**: Mini-app gateways exposing sensitive APIs MUST implement RFC 9470 `WWW-Authenticate: Bearer error="insufficient_user_authentication"` challenges bound to WebAuthn ACR levels.
5. **Standardized Edge Catalog Discovery via RFC 8288 & WebSub**: Catalog APIs MUST expose RFC 8288 typed Link headers for pagination and schema discovery, and provide a WebSub hub for real-time lifecycle event distribution.
