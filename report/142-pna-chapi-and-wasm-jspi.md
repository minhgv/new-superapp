# Chuyên đề 142: WICG Private Network Access (PNA), W3C CCG Credential Handler API (CHAPI) & WebAssembly JavaScript Promise Integration (JSPI)

## 1. Tóm tắt điều hành & Bối cảnh kỹ thuật

Vòng nghiên cứu 142 của tiêu chuẩn Mini App Store trên Super App tập trung giải quyết ba bài toán cốt lõi về bảo mật mạng nội bộ, quản lý danh tính phi tập trung và tối ưu hóa hiệu năng thực thi nhị phân cho các ứng dụng siêu phức tạp:

1. **WICG Private Network Access (PNA) / Local Network Access:** Đặc tả bảo mật mạng định nghĩa phân loại không gian địa chỉ IP (`public`, `private`, `local`) và cơ chế preflight bắt buộc (`Access-Control-Request-Private-Network` / `Access-Control-Allow-Private-Network: true`). PNA loại bỏ hoàn toàn các cuộc tấn công Cross-Site Request Forgery (CSRF) và Server-Side Request Forgery (SSRF) xuất phát từ mini-app nhắm vào router Wi-Fi gia đình, cổng quản trị nội bộ doanh nghiệp (192.168.x.x, 10.x.x.x) và dịch vụ loopback cục bộ (127.0.0.1).
2. **W3C CCG Credential Handler API (CHAPI):** Chuẩn kiến trúc trung gian mở mở rộng giao diện `navigator.credentials` của W3C, cho phép mini-app yêu cầu (`credential.get()`) và lưu trữ (`credential.store()`) Verifiable Credentials (VC) và Decentralized Identifiers (DID) qua các ví số độc lập. CHAPI sử dụng các sự kiện Service Worker chuyên biệt (`credentialrequest`, `credentialstore`) và container bottom sheet bảo mật nhằm loại bỏ việc khóa chặt nhà cung cấp ví (vendor lock-in) và ngăn ngừa theo dõi chéo (cross-app tracking) qua định danh theo cặp (pairwise pseudonymous DIDs).
3. **WebAssembly JavaScript Promise Integration (JSPI):** Chuẩn chuyển đổi ngữ cảnh stack-switching cấp máy ảo (V8/Chromium) cho phép các đoạn mã WebAssembly đồng bộ (C/C++, Rust, Go) gọi các Web API bất đồng bộ trả về Promise (như `fetch()`, IndexedDB, OPFS, WebCrypto, audio) mà không cần dùng kỹ thuật Asyncify cồng kềnh. JSPI giúp giảm kích thước gói nhị phân `.wasm` từ 45-60%, loại bỏ chi phí CPU không cần thiết, và duy trì an toàn bộ nhớ tuyệt đối với trần ngăn xếp (stack ceilings) và cơ chế chống re-entrancy.

---

## 2. Danh mục 15 Findings chi tiết (Milestone 142)

### Finding 1: WICG Private Network Access (PNA): Architecture, Security Context & IP Address Space Taxonomy
- **ID:** `pna_local_net_142_01`
- **Chủ đề (Topic):** `pna_specification_and_ip_address_space_taxonomy`
- **Phân loại (Category):** Network Security, Intranet Isolation & Host Protection
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** WICG Private Network Access Specification, Section 2 & 3
- **URL chính thức:** [https://wicg.github.io/private-network-access/](https://wicg.github.io/private-network-access/)
- **URL phụ trợ:** [https://wicg.github.io/private-network-access/#ip-address-space](https://wicg.github.io/private-network-access/#ip-address-space), [https://chromestatus.com/feature/5436853517811712](https://chromestatus.com/feature/5436853517811712)

#### Các điểm cốt lõi (Key Takeaways):
- WICG Private Network Access (PNA, formerly CORS-RFC1918) restricts the capability of websites and web-embedded contexts to make requests to servers on private networks and localhost.
- Classifies all network endpoints into three discrete, strictly ordered IP address spaces: 'local' (loopback addresses 127.0.0.0/8, ::1, and link-local addresses), 'private' (RFC 1918 IPv4 ranges 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, and IPv6 Unique Local Addresses fc00::/7), and 'public' (globally routable Internet addresses).
- Establishes the fundamental security invariant: requests originating from a less private IP address space to a more private IP address space ('public' -> 'private', 'public' -> 'local', or 'private' -> 'local') are prohibited by default unless explicitly authorized.
- Requires that the initiating origin MUST be an authenticated Secure Context (HTTPS or localhost); plain HTTP web contexts are unconditionally blocked from initiating any private network requests.
- Prevents Cross-Site Request Forgery (CSRF) and Server-Side Request Forgery (SSRF) pivot attacks where a malicious external web resource weaponizes an end-user's browser/super-app as an HTTP proxy into home/enterprise intranets.

#### Khuyến nghị triển khai trên Super App:
- Enforce WICG PNA address space classification in the super-app WebView container network stack to prevent untrusted mini-apps from probing local WiFi routers or intranet endpoints.
- Mandate that mini-app packages be served exclusively over Secure Contexts (custom secure scheme or HTTPS) to comply with PNA baseline security requirements.

---

### Finding 2: PNA Preflight Handshake: 'Access-Control-Request-Private-Network' & Explicit Target Authorization
- **ID:** `pna_local_net_142_02`
- **Chủ đề (Topic):** `pna_preflight_handshake_and_cors_headers`
- **Phân loại (Category):** Protocol Security, CORS Governance & Access Control
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** WICG Private Network Access, Section 4 (Preflight Requests)
- **URL chính thức:** [https://developer.chrome.com/blog/private-network-access-preflight/](https://developer.chrome.com/blog/private-network-access-preflight/)
- **URL phụ trợ:** [https://wicg.github.io/private-network-access/#cors-preflight](https://wicg.github.io/private-network-access/#cors-preflight), [https://developer.chrome.com/blog/private-network-access-update/](https://developer.chrome.com/blog/private-network-access-update/)

#### Các điểm cốt lõi (Key Takeaways):
- Before sending any non-simple request or subresource request targeting a more private IP space, the user agent dispatches an explicit CORS preflight OPTIONS request containing the header 'Access-Control-Request-Private-Network: true'.
- The private target server MUST explicitly acknowledge and authorize private network cross-origin access by returning the response header 'Access-Control-Allow-Private-Network: true' alongside standard CORS allow headers.
- Unlike standard CORS where 'simple requests' (e.g. GET/POST with simple headers) bypass preflights, PNA extends preflight verification to ALL subresource fetches (including <img>, <script>, and simple GETs) crossing from public to private/local spaces.
- If the private target server fails to respond with 'Access-Control-Allow-Private-Network: true', the user agent immediately terminates the fetch with a NetworkError, preventing malicious payload transmission.
- Protects legacy embedded devices, routers, printers, and IoT gateways that lack internal CSRF token validation and can be exploited or reconfigured via simple blind GET/POST requests.

#### Khuyến nghị triển khai trên Super App:
- Implement the PNA preflight filter in the native network bridge so that any mini-app `fetch()` targeting internal LAN services must undergo preflight validation.
- Log PNA preflight failures to the developer console to alert enterprise mini-app developers when internal microservices lack required CORS PNA headers.

---

### Finding 3: Mitigating Intranet SSRF, Loopback Probing & IoT Peripheral Exploits in Mini-App Stores
- **ID:** `pna_local_net_142_03`
- **Chủ đề (Topic):** `pna_mini_app_intranet_ssrf_and_iot_defense`
- **Phân loại (Category):** Application Security, Vulnerability Mitigation & Container Hardening
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** WICG Private Network Access Security Considerations & Chrome PNA Implementation Roadmap
- **URL chính thức:** [https://developer.chrome.com/blog/private-network-access-update/](https://developer.chrome.com/blog/private-network-access-update/)
- **URL phụ trợ:** [https://wicg.github.io/private-network-access/](https://wicg.github.io/private-network-access/), [https://chromestatus.com/feature/5436853517811712](https://chromestatus.com/feature/5436853517811712)

#### Các điểm cốt lõi (Key Takeaways):
- Super-apps running on mobile devices often connect to sensitive enterprise internal Wi-Fi networks or home LANs containing unauthenticated admin interfaces (192.168.1.1, 10.0.0.1, localhost daemon ports 8080/3000).
- Malicious third-party mini-apps can execute network port scanning, internal topology discovery, DNS rebinding, or firmware exploit injection by making automated background fetches to internal IPs.
- PNA eliminates blind intranet scanning by aborting requests before TCP connection establishment if the origin lacks explicit permission or preflight approval.
- Distinguishes between worker contexts: Web Workers and Service Workers inherit their owner document's IP address space and are held to identical PNA constraints, preventing background thread bypasses.
- DNS Rebinding mitigation: PNA verifies the resolved IP address of the target host during DNS resolution; if a public domain resolves to a private IP, the request is subjected to strict private network checks.

#### Khuyến nghị triển khai trên Super App:
- Enforce strict DNS resolution tracking in the super-app core network client: if any domain resolves to RFC 1918 or loopback addresses, automatically apply PNA security checks.
- Default all third-party marketplace mini-apps to 'public' address space classification regardless of whether the mobile device is on corporate VPN or internal Wi-Fi.

---

### Finding 4: Permissions Policy Gating: 'local-network-access' Delegation & Manifest Entitlements
- **ID:** `pna_local_net_142_04`
- **Chủ đề (Topic):** `pna_permissions_policy_and_developer_console_gating`
- **Phân loại (Category):** Governance, Permissions Policy & Marketplace Scoping
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** Chrome Platform Status Feature 5436853517811712 & W3C Permissions Policy Integration
- **URL chính thức:** [https://chromestatus.com/feature/5436853517811712](https://chromestatus.com/feature/5436853517811712)
- **URL phụ trợ:** [https://wicg.github.io/private-network-access/](https://wicg.github.io/private-network-access/), [https://developer.chrome.com/blog/private-network-access-preflight/](https://developer.chrome.com/blog/private-network-access-preflight/)

#### Các điểm cốt lõi (Key Takeaways):
- To provide administrative governance over intranet communication, PNA integrates with the W3C Permissions Policy framework via the proposed `local-network-access` directive.
- Containers can restrict or delegate local network access using Permissions-Policy HTTP headers or iframe `allow='local-network-access <origin>'` attributes.
- Legitimate mini-apps requiring local network access (e.g. smart home device provisioning, local POS printer connectivity, Wi-Fi router setup utilities) must declare an explicit capability entitlement.
- User consent dialogs: When a mini-app requests access to a local network endpoint, the host platform can trigger an explicit, one-time permission prompt informing the user of the local device access.
- Developer console declaration: Super-app store submission pipelines require developers to list all intended local IP ranges and service types, subjecting the mini-app to enhanced security review.

#### Khuyến nghị triển khai trên Super App:
- Implement a `permissions: ['local-network-access']` declaration in the mini-app manifest, gating access behind operator manual review.
- Present a system permission prompt: '[Mini-App Name] wants to find and connect to devices on your local network' before granting PNA access on mobile devices.

---

### Finding 5: PNA Telemetry, Violation Reporting & Gradual Enforcement via Reporting API
- **ID:** `pna_local_net_142_05`
- **Chủ đề (Topic):** `pna_telemetry_violation_reporting_and_transitional_modes`
- **Phân loại (Category):** Telemetry, Store Operations & Zero-Trust Auditing
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** WICG Private Network Access, Section 5 (Reporting & Telemetry)
- **URL chính thức:** [https://wicg.github.io/private-network-access/#ip-address-space](https://wicg.github.io/private-network-access/#ip-address-space)
- **URL phụ trợ:** [https://developer.chrome.com/blog/private-network-access-update/](https://developer.chrome.com/blog/private-network-access-update/), [https://developer.chrome.com/blog/private-network-access-preflight/](https://developer.chrome.com/blog/private-network-access-preflight/)

#### Các điểm cốt lõi (Key Takeaways):
- PNA integrates with the W3C Reporting API to generate structured out-of-band violation reports whenever a private network request is blocked or fails preflight validation.
- Report payload details: includes the initiating URL, target URL, source IP address space, target IP address space, and specific failure reason (e.g., 'no-preflight-response', 'preflight-missing-allow-header', 'insecure-context').
- Phased rollout strategy: Supports a 'warning-only' (audit) mode where preflight failures emit console warnings and telemetry reports to super-app operations without terminating the network request.
- Transition to enforcement: Once telemetry shows legacy intranet services have added `Access-Control-Allow-Private-Network: true`, the container flips the switch to hard enforcement.
- Automated Store Audit: Pre-release sandbox automated scanners test mini-apps in synthetic networks to verify that any local network calls comply with PNA requirements before public release.

#### Khuyến nghị triển khai trên Super App:
- Configure the super-app Reporting API endpoint to aggregate PNA violation reports, flagging unapproved intranet probing in real time.
- Establish an automated CI/CD security test stage in the mini-app review pipeline that intercepts outbound sockets and rejects packages that attempt unauthorized local IP requests.

---

### Finding 6: W3C CCG Credential Handler API (CHAPI): Architecture & Polyfill-Mediated Credential Exchange
- **ID:** `chapi_identity_142_06`
- **Chủ đề (Topic):** `chapi_specification_and_browser_mediation_architecture`
- **Phân loại (Category):** Decentralized Identity, Verifiable Credentials & Wallet Mediation
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** W3C Credentials Community Group (CCG) Credential Handler API (CHAPI) Specification
- **URL chính thức:** [https://w3c-ccg.github.io/credential-handler-api/](https://w3c-ccg.github.io/credential-handler-api/)
- **URL phụ trợ:** [https://chapi.io/](https://chapi.io/), [https://w3c.github.io/vc-data-model/](https://w3c.github.io/vc-data-model/)

#### Các điểm cốt lõi (Key Takeaways):
- CHAPI specifies an open, decentralized mediation layer enabling web applications to request, store, and exchange Verifiable Credentials (VCs) and digital identity tokens across independent wallet providers without vendor lock-in.
- Extends the W3C Credential Management API (`navigator.credentials.get()` and `navigator.credentials.store()`) by introducing custom credential query types specifically designed for Verifiable Presentations and Decentralized Identifiers (DIDs).
- Mediation pattern: Acts as a neutral broker between a Relying Party (mini-app requesting identity proof) and a Credential Handler (digital wallet web app or native wallet module) chosen freely by the user.
- Polyfill and native duality: Can operate either via a lightweight client-side JavaScript polyfill (`credential-handler-polyfill`) interfacing with an iframe mediator or as a native container service built directly into the super-app platform.
- Eliminates hard-coded proprietary SDK integrations for individual identity vendors, fostering an open ecosystem where users control their personal credentials.

#### Khuyến nghị triển khai trên Super App:
- Implement a native CHAPI mediation bridge in the super-app container, allowing mini-apps to invoke `navigator.credentials.get({digital: ...})` to request verifiable attributes.
- Embed a super-app native wallet provider that registers as a default Credential Handler while allowing users to connect external decentralized wallets if desired.

---

### Finding 7: Credential Lifecycle: 'credential.get()', 'credential.store()' & Container Sheet Mediation
- **ID:** `chapi_identity_142_07`
- **Chủ đề (Topic):** `chapi_credential_lifecycle_and_ui_mediation`
- **Phân loại (Category):** Identity Governance, User Consent & UX Sandboxing
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** CHAPI Architecture & Integration Guide (chapi.io) / W3C CCG
- **URL chính thức:** [https://chapi.io/](https://chapi.io/)
- **URL phụ trợ:** [https://w3c-ccg.github.io/credential-handler-api/](https://w3c-ccg.github.io/credential-handler-api/), [https://identity.foundation/](https://identity.foundation/)

#### Các điểm cốt lõi (Key Takeaways):
- Storage lifecycle (`credential.store()`): An issuer mini-app (e.g. university diploma portal, health authority, bank) issues a Verifiable Credential and requests storage; the user agent presents a native modal prompt allowing the user to select their destination wallet and confirm storage.
- Retrieval lifecycle (`credential.get()`): A relying party mini-app presents a query descriptor specifying required credential schemas (e.g. age verification > 18, driver license status); the browser/super-app queries installed credential handlers.
- Container mediation sheet: The super-app renders a secure, unforgeable native bottom sheet displaying matching credentials discovered across handlers, completely shielding handler selection from relying party snooping.
- User confirmation barrier: No credential data or identity attributes are disclosed to the relying party until the user explicitly selects a credential card and confirms via biometric authentication (passkey/FaceID).
- Empty state handling: If no matching credential exists in the user's wallet, CHAPI returns `null`, enabling the relying party mini-app to offer an inline onboarding/issuance link gracefully.

#### Khuyến nghị triển khai trên Super App:
- Design an unforgeable native container UI sheet for CHAPI requests that clearly displays the requesting mini-app's identity, verified badge, and exact requested attributes.
- Enforce biometric re-authentication (system PIN/fingerprint) inside the CHAPI bottom sheet before releasing cryptographically signed credential proofs.

---

### Finding 8: CHAPI Event Architecture: 'CredentialRequestEvent', 'CredentialStoreEvent' & Service Worker Handling
- **ID:** `chapi_identity_142_08`
- **Chủ đề (Topic):** `chapi_event_architecture_and_worker_handling`
- **Phân loại (Category):** Web APIs, Service Worker Architecture & Event Orchestration
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** W3C CCG CHAPI Specification, Section 4 (CredentialEvent & Service Worker Interfaces)
- **URL chính thức:** [https://w3c-ccg.github.io/credential-handler-api/#credentialevent](https://w3c-ccg.github.io/credential-handler-api/#credentialevent)
- **URL phụ trợ:** [https://w3c-ccg.github.io/credential-handler-api/](https://w3c-ccg.github.io/credential-handler-api/), [https://chapi.io/](https://chapi.io/)

#### Các điểm cốt lõi (Key Takeaways):
- CHAPI introduces specialized Service Worker events: `credentialrequest` (invoked when a relying party calls `credential.get()`) and `credentialstore` (invoked when an issuer calls `credential.store()`).
- Decentralized wallet mini-apps register a background Service Worker that listens for `credentialrequest` events, inspects the requested query protocols, and checks local encrypted storage for matching Verifiable Credentials.
- Event response methods: The handler responds via `event.respondWith(promise)` delivering a structured `CredentialResponse` containing the signed Verifiable Presentation, or opens a custom modal window via `event.openWindow(url)` to collect user pin/signature.
- Asynchronous resolution: Allows complex cryptographic proof generation (e.g. BBS+ zero-knowledge proofs or Ed25519 signature computation) to execute asynchronously off the main DOM thread.
- Sandboxed execution: Service Workers executing credential events run in an isolated origin boundary, preventing the relying party from inspecting the wallet's internal cryptographic keys or key store.

#### Khuyến nghị triển khai trên Super App:
- Support the `credentialrequest` and `credentialstore` Service Worker event pipeline inside the super-app background worker execution subsystem.
- Provide hardware-accelerated cryptographic signing primitives to credential handling service workers via the WebCrypto native bridge.

---

### Finding 9: W3C Verifiable Credentials & Decentralized Ecosystem Interoperability via CHAPI
- **ID:** `chapi_identity_142_09`
- **Chủ đề (Topic):** `chapi_verifiable_credentials_and_decentralized_trust`
- **Phân loại (Category):** Data Standards, Cryptographic Proofs & Verifiable Credentials
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** W3C Verifiable Credentials Data Model v2.0 & Decentralized Identity Foundation (DIF)
- **URL chính thức:** [https://w3c.github.io/vc-data-model/](https://w3c.github.io/vc-data-model/)
- **URL phụ trợ:** [https://identity.foundation/](https://identity.foundation/), [https://w3c-ccg.github.io/credential-handler-api/](https://w3c-ccg.github.io/credential-handler-api/)

#### Các điểm cốt lõi (Key Takeaways):
- CHAPI standardizes payload transport for the W3C Verifiable Credentials Data Model, facilitating interoperability between diverse cryptographic proof types (Data Integrity Proofs, JWT-VC, SD-JWT-VC).
- Decentralized trust model: Relies on verifiable issuers and decentralized identifiers (DIDs) rather than proprietary centralized identity federation silos, enabling multi-issuer enterprise ecosystems.
- Selective disclosure integration: Handlers can parse requested JSON-LD contexts or SD-JWT disclosure claims and dynamically redact non-essential attributes (e.g. proving age > 21 without revealing birthdate or legal name).
- Cross-border and cross-platform utility: Mini-apps across ride-hailing, e-commerce, banking, and government utilities can verify verified digital identity artifacts without maintaining direct database integrations with government registries.
- Revocation checking: Mini-apps receiving CHAPI credential presentations verify real-time status via decentralized Status List 2021 or Bitstring Status List tokens embedded in the credential metadata.

#### Khuyến nghị triển khai trên Super App:
- Mandate W3C VC Data Model compliance for all identity verification workflows in the super-app mini-app store.
- Integrate an automated cryptographic proof verification service in the super-app API gateway to validate CHAPI presentations on behalf of mini-apps.

---

### Finding 10: CHAPI Security Model: Origin Scoping, Cross-App Tracking Prevention & Key Isolation
- **ID:** `chapi_identity_142_10`
- **Chủ đề (Topic):** `chapi_security_origin_scoping_and_anti_correlation`
- **Phân loại (Category):** Privacy Engineering, Anti-Fingerprinting & Origin Security
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** Decentralized Identity Foundation (DIF) Security & Privacy Best Practices / W3C CCG CHAPI Security
- **URL chính thức:** [https://identity.foundation/](https://identity.foundation/)
- **URL phụ trợ:** [https://w3c-ccg.github.io/credential-handler-api/](https://w3c-ccg.github.io/credential-handler-api/), [https://chapi.io/](https://chapi.io/)

#### Các điểm cốt lõi (Key Takeaways):
- Origin Scoping: CHAPI enforces strict origin verification; the browser mediator binds every credential request to the verified origin of the requesting mini-app, preventing origin spoofing or man-in-the-middle impersonation.
- Anti-Correlation protections: To prevent distinct relying mini-apps from colluding to track users across the super-app catalog, credential handlers should generate pairwise pseudonymous DIDs (did:peer or did:key) per relying party origin.
- Silent probing defense: Relying mini-apps cannot silently enumerate which wallets or credentials a user possesses; all query results are withheld by the mediator unless the user actively selects a credential in the native UI.
- Secure key isolation: Private signing keys used by credential handlers must reside in isolated hardware security enclaves (Android Keystore / iOS Secure Enclave) and never be accessible to relying mini-apps or the mediator script.
- ReDoS and payload validation: Mediation envelopes and credential query parameters are subjected to strict schema checks and size limits (max 64KB per presentation) to prevent denial of service.

#### Khuyến nghị triển khai trên Super App:
- Require that native wallet handlers generate unique pairwise cryptographic identifiers for each requesting mini-app to eliminate cross-app tracking.
- Establish strict payload validation in the CHAPI container bridge, rejecting any credential query exceeding 64KB or containing invalid JSON-LD contexts.

---

### Finding 11: WebAssembly JSPI: JavaScript Promise Integration Specification & Stack-Switching Architecture
- **ID:** `wasm_jspi_142_11`
- **Chủ đề (Topic):** `wasm_jspi_specification_and_stack_switching_architecture`
- **Phân loại (Category):** WebAssembly Standards, Concurrency & Virtual Machine Architecture
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** WebAssembly JavaScript Promise Integration (JSPI) Proposal (Phase 3 W3C Wasm WG) & V8 JSPI Architecture
- **URL chính thức:** [https://v8.dev/blog/jspi](https://v8.dev/blog/jspi)
- **URL phụ trợ:** [https://chromestatus.com/feature/5196961445773312](https://chromestatus.com/feature/5196961445773312), [https://webassembly.org/](https://webassembly.org/)

#### Các điểm cốt lõi (Key Takeaways):
- WebAssembly JSPI defines an API and runtime mechanism that seamlessly bridges synchronous WebAssembly code with asynchronous JavaScript APIs returning Promises, without requiring blocking event loops or busy-waiting.
- Traditional WebAssembly execution models are strictly synchronous; calling asynchronous Web APIs (e.g. `fetch()`, IndexedDB, WebCrypto, audio decoding) previously required compilers to transform code using Asyncify (instrumenting every function call with state save/restore machinery).
- Asyncify disadvantages: Increases Wasm binary size by 40-100%, introduces 20-50% CPU execution overhead, and severely degrades garbage collection and memory cache locality.
- JSPI solves this by leveraging engine-level stack switching (fibers/coroutines): when WebAssembly calls an asynchronous JavaScript function, the WebAssembly execution stack is cleanly suspended, control returns to the JavaScript event loop, and the stack is resumed automatically once the Promise resolves.
- Introduces `WebAssembly.promising()` (wraps an exported Wasm function to return a JavaScript Promise) and `WebAssembly.suspending()` (wraps an async JavaScript function so it can be called synchronously from inside Wasm).

#### Khuyến nghị triển khai trên Super App:
- Enable WebAssembly JSPI runtime flags in the super-app WebView container (Chromium 123+ / Android System WebView) to unlock zero-overhead asynchronous I/O for compiled C/C++/Rust mini-apps.
- Recommend JSPI as the standard compilation target for mini-app game engines, media processing libraries, and cryptography modules instead of legacy Asyncify.

---

### Finding 12: V8 Engine Fiber Coroutines: Elimination of Asyncify Bloat & Zero-Overhead Stack Resumption
- **ID:** `wasm_jspi_142_12`
- **Chủ đề (Topic):** `wasm_jspi_vm_coroutine_and_memory_efficiency`
- **Phân loại (Category):** Virtual Machine Optimization, Binary Efficiency & Memory Footprint
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** Chromium V8 Engine WebAssembly Stack-Switching Implementation & Liftoff Baseline Compiler
- **URL chính thức:** [https://chromium.googlesource.com/v8/v8/+/refs/heads/main/src/wasm/baseline/liftoff-compiler.cc](https://chromium.googlesource.com/v8/v8/+/refs/heads/main/src/wasm/baseline/liftoff-compiler.cc)
- **URL phụ trợ:** [https://v8.dev/blog/jspi](https://v8.dev/blog/jspi), [https://chromestatus.com/feature/5196961445773312](https://chromestatus.com/feature/5196961445773312)

#### Các điểm cốt lõi (Key Takeaways):
- Stack-switching mechanics: In V8, JSPI manages lightweight stack segments (fibers) allocated dynamically from the operating system memory pool.
- Suspension cost: When `suspending()` is triggered, V8 simply saves registers to the current stack segment header and switches the stack pointer (RSP/ESP) back to the main JavaScript stack frame, executing in nanoseconds (order of ~50-100 CPU cycles).
- Resumption cost: When the awaited Promise settles, the microtask runner restores the stack pointer and register context, allowing the Wasm function to continue execution immediately after the call instruction.
- Binary size reduction: Benchmarks on real-world C++ game engines and SQLite Wasm show that replacing Asyncify with JSPI shrinks `.wasm` bundle sizes by an average of 45-60%, drastically accelerating mini-app package downloads.
- CPU efficiency: Eliminates the CPU instructions required to unwind and rewind the call stack on every asynchronous boundary, improving frame rates and battery life on mobile devices.

#### Khuyến nghị triển khai trên Super App:
- Configure mini-app packaging guidelines to validate that WebAssembly modules compiled for high-performance use cases take advantage of native stack switching.
- Track container battery consumption and CPU metrics, noting measurable power savings on JSPI-based mini-app execution compared to Asyncify workloads.

---

### Finding 13: Porting Legacy Native Systems: Synchronous C/C++ I/O Binding to Async Super-App Web APIs
- **ID:** `wasm_jspi_142_13`
- **Chủ đề (Topic):** `wasm_jspi_legacy_c_cpp_rust_porting_and_apis`
- **Phân loại (Category):** Developer Tooling, Native Portability & Emscripten Ecosystem
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** MDN WebAssembly Concepts & Emscripten JSPI Toolchain Integration (`-sJSPI=1`)
- **URL chính thức:** [https://developer.mozilla.org/en-US/docs/WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly)
- **URL phụ trợ:** [https://v8.dev/blog/jspi](https://v8.dev/blog/jspi), [https://webassembly.org/](https://webassembly.org/)

#### Các điểm cốt lõi (Key Takeaways):
- Enables straightforward porting of massive legacy desktop/mobile C, C++, and Rust codebases (CAD software, GIS engines, 3D games, proprietary audio synthesizers, database engines like SQLite) directly into mini-apps without rewriting synchronous architecture.
- Synchronous filesystem simulation: C standard library calls (`fread()`, `fwrite()`, `open()`, `sqlite3_step()`) can remain synchronous in C++ source while transparently resolving against asynchronous browser storage (Origin Private File System / OPFS or IndexedDB) under the hood.
- Network I/O: Synchronous socket or HTTP client abstractions in native libraries can map directly to asynchronous `fetch()` calls or native super-app bridge APIs without complex callback spaghetti or state machines.
- Emscripten support: Standardized via the `-sJSPI=1` compiler flag in Emscripten toolchains, automatically wrapping imported async JS functions and exported entry points.
- Seamless interoperability: Native C++ code can yield to the UI thread during expensive operations, preventing the mini-app UI from freezing during heavy disk or network operations.

#### Khuyến nghị triển khai trên Super App:
- Provide an official Emscripten build profile in the mini-app developer documentation featuring `-sJSPI=1` and OPFS integration for enterprise developers porting C++ libraries.
- Expose super-app native bridge APIs (e.g. payment sheet presentation, biometric auth) in an asynchronous format that JSPI can easily consume synchronously.

---

### Finding 14: JSPI Security Governance: Call Stack Ceilings, Memory Safety & Re-Entrancy Mitigations
- **ID:** `wasm_jspi_142_14`
- **Chủ đề (Topic):** `wasm_jspi_security_call_stack_limits_and_reentrancy`
- **Phân loại (Category):** Security Engineering, Memory Safety & Re-Entrancy Hardening
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** WebAssembly Core Security Principles (webassembly.org) & JSPI Re-Entrancy Security Model
- **URL chính thức:** [https://webassembly.org/](https://webassembly.org/)
- **URL phụ trợ:** [https://v8.dev/blog/jspi](https://v8.dev/blog/jspi), [https://chromestatus.com/feature/5196961445773312](https://chromestatus.com/feature/5196961445773312)

#### Các điểm cốt lõi (Key Takeaways):
- Re-entrancy hazard: While a WebAssembly stack is suspended awaiting a Promise, incoming user events or timer callbacks can invoke another exported WebAssembly function on the same instance, potentially mutating shared linear memory (`WebAssembly.Memory`) and corrupting state.
- State guardrails: Developers and container runtimes must implement re-entrancy locks or state validation flags around suspended Wasm instances to reject unexpected concurrent invocations during active suspensions.
- Stack exhaustion protection: Each suspended fiber consumes dedicated stack memory; uncontrolled recursion or launching thousands of unfulfilled suspended Promises can cause memory exhaustion (Out-Of-Memory) or stack overflow.
- Engine limits: Wasm engines enforce strict upper limits on the number of concurrently suspended stacks and maximum stack depth per fiber, throwing a `RangeError` if limits are exceeded.
- Memory isolation: JSPI does not violate WebAssembly's core security boundary: memory accesses remain strictly bounds-checked within the instance's declared `WebAssembly.Memory` buffer, preventing arbitrary memory corruption outside the sandbox.

#### Khuyến nghị triển khai trên Super App:
- Establish guidelines for mini-app developers requiring re-entrancy guards on all Wasm exports that trigger JSPI suspension.
- Enforce container memory limits (max 512MB linear memory and max 256 concurrent suspended fibers) during automated store runtime security checks.

---

### Finding 15: Container Runtime Capability Detection, Feature Flags & Store Packaging Policy for JSPI
- **ID:** `wasm_jspi_142_15`
- **Chủ đề (Topic):** `wasm_jspi_platform_compatibility_and_store_review`
- **Phân loại (Category):** Platform Governance, Runtime Detection & Store Policy
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** Chrome Platform Status Feature 5196961445773312 (WebAssembly JavaScript Promise Integration)
- **URL chính thức:** [https://chromestatus.com/feature/5196961445773312](https://chromestatus.com/feature/5196961445773312)
- **URL phụ trợ:** [https://v8.dev/blog/jspi](https://v8.dev/blog/jspi), [https://developer.mozilla.org/en-US/docs/WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly)

#### Các điểm cốt lõi (Key Takeaways):
- Progressive feature detection: Mini-apps should perform programmatic capability checking via `typeof WebAssembly.promising === 'function'` and `typeof WebAssembly.suspending === 'function'` before invoking JSPI workflows.
- Platform compatibility: JSPI is fully standardized in V8/Chromium and actively supported in modern Android System WebView, Node.js, and desktop environments, with ongoing implementation across JavaScriptCore (WebKit) and SpiderMonkey (Firefox).
- Fallback architecture: For legacy webview runtimes lacking native JSPI, mini-app packaging pipelines can maintain dual-compiled artifacts (JSPI binary for modern engines and an Asyncify fallback binary for legacy clients) dynamically selected at launch.
- Container configuration: Super-app native shell applications on Android must ensure their WebView instances are initialized with modern Chromium settings allowing advanced Wasm features.
- Store review validation: Pre-submission automated scanners verify that `.wasm` binaries declaring stack-switching opcodes correctly declare their minimum engine version requirements in the mini-app manifest.

#### Khuyến nghị triển khai trên Super App:
- Implement a capability check in the mini-app launcher: if the host device engine supports JSPI, load the optimized JSPI package; otherwise, gracefully load the compatibility build.
- Include JSPI validation in the super-app store automated submission linter, verifying that mini-apps declare accurate runtime engine dependencies.

---

## 3. Kiến trúc tích hợp & Khuyến nghị kỹ thuật cho Super App Store

### 3.1. Ma trận phân lớp bảo mật mạng nội bộ (WICG PNA)
- Mọi mini-app trong catalog mặc định chạy trong không gian `public`.
- Bất kỳ yêu cầu kết nối nào đến dải IP RFC 1918 (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) hoặc loopback (127.0.0.0/8, ::1) bắt buộc phải:
  1. Được khai báo trong manifest qua quyền `local-network-access`.
  2. Vượt qua bước CORS preflight với header `Access-Control-Allow-Private-Network: true` từ thiết bị đích.
  3. Được người dùng xác nhận thông qua hộp thoại quyền hệ thống trên Super App.

### 3.2. Trung gian xác thực danh tính & Ví Verifiable Credentials (CHAPI)
- Cung cấp native bridge cho `navigator.credentials.get({digital: ...})`.
- Thiết kế container sheet độc lập chống giả mạo, cho phép hiển thị thẻ xác thực từ ví nội bộ Super App hoặc ví phi tập trung do người dùng cài đặt.
- Tạo định danh theo cặp (pairwise pseudonymous DIDs) để triệt tiêu nguy cơ liên kết danh tính người dùng giữa các bên thứ ba.

### 3.3. Tối ưu hóa thực thi nhị phân đa nền tảng (Wasm JSPI)
- Kích hoạt cờ hỗ trợ JSPI trên WebView container (Chromium 123+).
- Khuyến nghị nhà phát triển mini-app biên dịch module C/C++/Rust với `-sJSPI=1` trong Emscripten, giảm 45-60% dung lượng tải ban đầu so với Asyncify.
- Thiết lập trần bộ nhớ ngăn xếp (tối đa 256 suspended fibers) và cơ chế khóa chống re-entrancy để bảo vệ tính toàn vẹn của tuyến luồng chính.
