# Chuyên đề 124 (Iteration 128): Next-Gen Grant Negotiation (GNAP), FAPI 2.0 Message Signing & Distributed Security Event Tokens (RISC / SET)

## 1. Tổng quan & Bối cảnh Chiến lược trong Super-App Mini-App Store

Khi hệ sinh thái Super-App phát triển lên quy mô hàng chục triệu người dùng, hàng nghìn mini-app từ đối tác viễn thông, tài chính, bảo hiểm, chuỗi bán lẻ và chính phủ điện tử, mô hình phân quyền và bảo mật truyền thống dựa trên OAuth 2.0/2.1 (chuyển hướng trình duyệt thô, phạm vi quyền hạn tĩnh coarse-grained scopes, và token dạng Bearer dễ bị đánh cắp) bộc lộ các điểm nghẽn kỹ thuật nghiêm trọng. Đồng thời, các giao dịch tài chính nhạy cảm và sự cố an ninh (chiếm quyền tài khoản, rò rỉ thông tin xác thực) đòi hỏi cơ chế ký số phi thừa nhận (non-repudiation) và phối hợp ứng phó sự cố theo thời gian thực giữa các miền quản trị độc lập.

Milestone 128 tập trung chuẩn hóa 3 trụ cột kỹ thuật tối tân trong kiến trúc định danh, kiểm soát phân quyền và điều phối an ninh phân tán:
1. **IETF Grant Negotiation and Authorization Protocol (GNAP)** (`draft-ietf-gnap-core-protocol` & `draft-ietf-gnap-resource-servers`): Giao thức đàm phán phân quyền thế hệ mới thay thế OAuth 2.0/2.1, hỗ trợ đàm phán quyền hạn chi tiết (fine-grained access negotiation), ràng buộc khóa mật mã client ngay từ bước khởi tạo, cấp phát đa token trong một lượt phản hồi, điều phối tương tác người dùng đa kênh (không chỉ riêng chuyển hướng URL trình duyệt mà hỗ trợ cả ứng dụng native, mã xác thực thiết bị, push notification), API tiếp tục giao dịch (`continue`) để nâng cấp quyền hạn động (step-up authorization) và cơ chế phân tách máy chủ tài nguyên (Resource Server decoupling).
2. **OpenID FAPI 2.0 Message Signing & OAuth 2.0 Grant Management** (`fapi-message-signing-2_0` & `oauth-v2-grant-management`): Chuẩn hóa ký số thông điệp HTTP theo **IETF RFC 9421** cho các giao dịch cấp độ tài chính, bắt buộc bảo vệ tính toàn vẹn của `@method`, `@target-uri`, `@status` và `content-digest` (RFC 9530), loại bỏ thuật toán yếu để áp dụng độc quyền PS256/ES256/Ed25519; kết hợp đặc tả Quản lý Quyền (Grant Management) với định danh `grant_id` duy nhất, cho phép mini-app và người dùng kiểm tra trạng thái quyền hạn đã cấp hoặc thu hồi quyền hạn ngay lập tức (`HTTP DELETE /grants/{grant_id}`) đáp ứng các tiêu chuẩn bảo vệ dữ liệu cá nhân (GDPR/PDPA).
3. **OpenID RISC 1.0 & IETF Security Event Token (SET) Delivery** (`openid-risc-profile-1_0`, **IETF RFC 8417**, **RFC 8935**, **RFC 8936**): Hệ thống bus tín hiệu an ninh phân tán dựa trên định dạng SET (`typ: secevent+jwt`), định nghĩa các loại sự kiện an ninh tài khoản chuẩn hóa (`account-credential-change-required`, `account-purged`, `sessions-revoked`), định danh chủ thể bảo vệ quyền riêng tư theo **RFC 9493** (sử dụng PPID `iss_sub` tránh liên kết chéo người dùng giữa các bên thứ ba), cùng hai phương thức truyền tải sự kiện tối ưu: cơ chế Push qua HTTP POST Webhook bảo vệ bằng mTLS (RFC 8935) và cơ chế Polling với xác nhận theo lô (batch acknowledgements) cho các máy chủ doanh nghiệp nằm sau tường lửa nghiêm ngặt (RFC 8936).

---

## 2. Bảng Tổng Hợp 15 Chuẩn Mực Nghiên Cứu Chi Tiết

| ID | Tiêu đề Chuẩn Mực | Danh mục | Cấp độ Bằng chứng | Nguồn / Đặc tả Tham chiếu |
|---|---|---|---|---|
| `gnap_std_128_01` | **IETF GNAP: Fine-Grained Grant Request Model, Client Key Presentation & Multi-Token Responses** | Identity, Authentication & Cryptographic Tokens | `official_standard` | [draft-ietf-gnap-core-protocol Section 2 (Requesting Access) & Section 2.1 (Client Instance Identification and Key Presentation)](https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol) |
| `gnap_std_128_02` | **IETF GNAP: Interaction Lifecycle, Interaction Handles & Non-Browser Multi-Channel Orchestration** | Identity, Authentication & Cryptographic Tokens | `official_standard` | [draft-ietf-gnap-core-protocol Section 2.5 (User Interaction) & Section 4 (Interacting with the End User)](https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol) |
| `gnap_std_128_03` | **IETF GNAP: Proof-of-Possession Token Binding & RFC 9421 HTTP Message Signatures** | Identity, Authentication & Cryptographic Tokens | `official_standard` | [draft-ietf-gnap-core-protocol Section 7 (Key Proofing) & Section 7.3 (HTTP Message Signatures)](https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol) |
| `gnap_std_128_04` | **IETF GNAP: Continuation API, Dynamic Privilege Step-Up & Grant Lifecycle Management** | Identity, Authentication & Cryptographic Tokens | `official_standard` | [draft-ietf-gnap-core-protocol Section 5 (Continuing a Grant Request) & Section 5.3 (Modifying and Revoking Grants)](https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol) |
| `gnap_std_128_05` | **IETF GNAP Resource Servers: RS-to-AS Decoupling, Capability Delegation & Token Introspection** | Identity, Authentication & Cryptographic Tokens | `official_standard` | [draft-ietf-gnap-resource-servers Section 2 (Resource Server and AS Interaction) & Section 3 (Token Verification)](https://datatracker.ietf.org/doc/html/draft-ietf-gnap-resource-servers) |
| `fapi_msg_128_06` | **OpenID FAPI 2.0: Message Signing Architecture, End-to-End Non-Repudiation & RFC 9421 Profiles** | Financial-Grade API & Consent Governance | `official_standard` | [FAPI 2.0 Message Signing Section 3 (Message Signing Profile) & Section 4 (Signature Verification)](https://openid.net/specs/fapi-message-signing-2_0.html) |
| `fapi_msg_128_07` | **OpenID FAPI 2.0: Mandatory Component Coverage, Content-Digest Integrity & Anti-Tampering** | Financial-Grade API & Consent Governance | `official_standard` | [FAPI 2.0 Message Signing Section 3.2 (HTTP Request Signing) & Section 3.3 (HTTP Response Signing)](https://openid.net/specs/fapi-message-signing-2_0.html) |
| `fapi_msg_128_08` | **OpenID FAPI 2.0: Algorithmic Rigor, Asymmetric Keys & Anti-Confusion Defenses** | Financial-Grade API & Consent Governance | `official_standard` | [FAPI 2.0 Message Signing Section 5 (Cryptographic Considerations) & FAPI 2.0 Security Profile Section 5.2.2](https://openid.net/specs/fapi-message-signing-2_0.html) |
| `fapi_msg_128_09` | **OAuth 2.0 Grant Management: Grant Query, Consent Inspection & Enterprise Auditability** | Financial-Grade API & Consent Governance | `official_standard` | [OAuth 2.0 Grant Management for OAuth 2.0 Section 3 (Grant Management Endpoint) & Section 4 (Query Grant)](https://openid.net/specs/oauth-v2-grant-management.html) |
| `fapi_msg_128_10` | **OAuth 2.0 Grant Management: Granular Consent Revocation, Action Lifecycle & User Privacy Centers** | Financial-Grade API & Consent Governance | `official_standard` | [OAuth 2.0 Grant Management for OAuth 2.0 Section 5 (Revoke Grant) & Section 2 (Grant Management Actions)](https://openid.net/specs/oauth-v2-grant-management.html) |
| `risc_set_128_11` | **IETF RFC 8417: Security Event Token (SET) Architecture, JSON Profiling & Asynchronous Signaling** | Security Events, Incident Response & Distributed Governance | `official_standard` | [RFC 8417 Section 2 (Security Event Token Format) & Section 3 (Security Event Claims)](https://www.rfc-editor.org/rfc/rfc8417.html) |
| `risc_set_128_12` | **OpenID RISC 1.0: Account Security Event Types, Credential Compromise & Incident Coordination** | Security Events, Incident Response & Distributed Governance | `official_standard` | [OpenID RISC Profile of IETF Security Events 1.0 Section 2 (Event Types) & Section 3 (RISC Event Definitions)](https://openid.net/specs/openid-risc-profile-1_0.html) |
| `risc_set_128_13` | **OpenID RISC 1.0: Privacy-Preserving Subject Identifiers, RFC 9493 & Anti-Correlation Protections** | Security Events, Incident Response & Distributed Governance | `official_standard` | [OpenID RISC Profile 1.0 Section 2.2 (Subject Identifiers) & IETF RFC 9493 (Subject Identifiers for Security Event Tokens)](https://openid.net/specs/openid-risc-profile-1_0.html) |
| `risc_set_128_14` | **IETF RFC 8935: Push-Based Security Event Token (SET) Delivery Over HTTP & Webhook Resiliency** | Security Events, Incident Response & Distributed Governance | `official_standard` | [RFC 8935 Section 2 (Delivery Protocol) & Section 3 (Error Handling)](https://www.rfc-editor.org/rfc/rfc8935.html) |
| `risc_set_128_15` | **IETF RFC 8936: Poll-Based Security Event Token (SET) Delivery Over HTTP & Air-Gapped Resiliency** | Security Events, Incident Response & Distributed Governance | `official_standard` | [RFC 8936 Section 2 (Polling Interface) & Section 3 (Event Acknowledgement)](https://www.rfc-editor.org/rfc/rfc8936.html) |

---

## 3. Phân Tích Kỹ Thuật Chi Tiết Từng Chuẩn Mực

### 3.1. IETF GNAP: Fine-Grained Grant Request Model, Client Key Presentation & Multi-Token Responses (`gnap_std_128_01`)

- **Chủ đề**: `gnap_request_and_grant_negotiation_model`
- **Danh mục**: Identity, Authentication & Cryptographic Tokens
- **Cấp độ bằng chứng**: `official_standard`
- **Chuẩn tham chiếu**: draft-ietf-gnap-core-protocol Section 2 (Requesting Access) & Section 2.1 (Client Instance Identification and Key Presentation)
- **URL chuẩn**: [https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol](https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol)
- **Tài liệu bổ trợ**: [https://datatracker.ietf.org/doc/html/draft-ietf-gnap-resource-servers](https://datatracker.ietf.org/doc/html/draft-ietf-gnap-resource-servers), [https://www.rfc-editor.org/rfc/rfc9396.html](https://www.rfc-editor.org/rfc/rfc9396.html)

#### Các Điểm Cốt Lõi Kỹ Thuật (Key Takeaways):
- GNAP (Grant Negotiation and Authorization Protocol) replaces OAuth 2.0's rigid redirect-and-scope model with direct, programmatic negotiation between a client instance and the Authorization Server (AS).
- Clients initiate requests by posting a structured JSON document to the AS grant endpoint containing explicit resource access arrays (`access_token` with fine-grained types, actions, locations, and data types).
- The client identifies its instance and presents its cryptographic public key (`client.key`) at initial negotiation, ensuring all subsequent requests and token usage are cryptographically bound to that key.
- GNAP natively supports issuing multiple distinct access tokens within a single transaction response, each targeted with specific rights and audiences for different backend services.
- Eliminates ambient authority and broad scope escalation by establishing exact, declaratively negotiated privileges before user interaction is initiated.

#### Khuyến Nghị Thực Thi Cho Chuẩn Super-App Mini-App Store:
- Adopt GNAP request structures in the super-app JS bridge SDK (`superapp.auth.negotiateGrant`), allowing mini-apps to request specific API actions without requiring broad coarse-grained scopes.
- Require mini-app WebViews to generate an ephemeral or device-bound WebCrypto public key passed in `client.key` during initial grant negotiation.
- Issue distinct, segmented tokens when a mini-app requires access to separate backend domains (e.g., telco billing vs catalog metadata) in a single authorization turn.

---

### 3.2. IETF GNAP: Interaction Lifecycle, Interaction Handles & Non-Browser Multi-Channel Orchestration (`gnap_std_128_02`)

- **Chủ đề**: `gnap_interaction_lifecycle_and_non_browser_orchestration`
- **Danh mục**: Identity, Authentication & Cryptographic Tokens
- **Cấp độ bằng chứng**: `official_standard`
- **Chuẩn tham chiếu**: draft-ietf-gnap-core-protocol Section 2.5 (User Interaction) & Section 4 (Interacting with the End User)
- **URL chuẩn**: [https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol](https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol)
- **Tài liệu bổ trợ**: [https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol#section-2.5](https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol#section-2.5), [https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol#section-4](https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol#section-4)

#### Các Điểm Cốt Lõi Kỹ Thuật (Key Takeaways):
- GNAP decouples client initiation from user interaction, supporting diverse interaction modes including redirect, user code display, app launching, and out-of-band push notifications.
- The client declares supported interaction modes (`start: ["redirect", "app", "user_code"]`) and completion mechanisms (`finish: {"method": "redirect", "uri": "...", "hash": "..."}`).
- The AS returns interaction directives (`interact` object with `redirect` URI or `finish` handle) along with a continuation access token and continuation URI for the client.
- Supports headless, voice-activated, smart TV, or in-store POS mini-apps where authentication occurs on a secondary trusted smartphone device while the client session remains active.
- The `finish` hash parameter cryptographically binds the client's original interaction request with the callback, preventing authorization session injection and cross-site redirection attacks.

#### Khuyến Nghị Thực Thi Cho Chuẩn Super-App Mini-App Store:
- Standardize native super-app modal consent sheets as the primary GNAP interaction mode (`start: ["app"]`), bypassing WebView URL redirections entirely.
- Implement out-of-band user approval for cross-device mini-app sessions (such as kiosk or TV apps) via push notifications delivered to the user's primary mobile super-app.
- Enforce verification of the GNAP `finish` interaction hash inside the container bridge before resuming mini-app script execution.

---

### 3.3. IETF GNAP: Proof-of-Possession Token Binding & RFC 9421 HTTP Message Signatures (`gnap_std_128_03`)

- **Chủ đề**: `gnap_cryptographic_key_binding_and_http_signatures`
- **Danh mục**: Identity, Authentication & Cryptographic Tokens
- **Cấp độ bằng chứng**: `official_standard`
- **Chuẩn tham chiếu**: draft-ietf-gnap-core-protocol Section 7 (Key Proofing) & Section 7.3 (HTTP Message Signatures)
- **URL chuẩn**: [https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol](https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol)
- **Tài liệu bổ trợ**: [https://www.rfc-editor.org/rfc/rfc9421.html](https://www.rfc-editor.org/rfc/rfc9421.html), [https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol#section-7](https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol#section-7)

#### Các Điểm Cốt Lõi Kỹ Thuật (Key Takeaways):
- GNAP enforces proof-of-possession (PoP) for all access tokens and continuation requests, strictly preventing bearer token replay if a token is intercepted.
- Every access token issued by the AS is cryptographically associated with a client key; presentation of the token to a Resource Server (RS) requires signing the request with that corresponding private key.
- Specifies RFC 9421 HTTP Message Signatures as the primary key-proofing mechanism, attaching `Signature` and `Signature-Input` headers covering method, target URI, timestamp, and body digest.
- Eliminates shared client secrets and static credentials, utilizing asymmetric public keys (ECDSA P-256/Ed25519) registered or dynamically presented during negotiation.
- Provides hardware-backed non-repudiation when keys are generated within mobile device secure enclaves or Keystore.

#### Khuyến Nghị Thực Thi Cho Chuẩn Super-App Mini-App Store:
- Mandate RFC 9421 HTTP Message Signing for all mini-app communications carrying GNAP access tokens across the super-app API gateway.
- Provide automatic transparent signing in the native bridge layer using container-managed hardware keys so mini-app web code cannot leak the private key.
- Reject unsigned or bearer-only access requests at all enterprise resource servers supporting the mini-app store.

---

### 3.4. IETF GNAP: Continuation API, Dynamic Privilege Step-Up & Grant Lifecycle Management (`gnap_std_128_04`)

- **Chủ đề**: `gnap_continuation_api_and_dynamic_grant_modification`
- **Danh mục**: Identity, Authentication & Cryptographic Tokens
- **Cấp độ bằng chứng**: `official_standard`
- **Chuẩn tham chiếu**: draft-ietf-gnap-core-protocol Section 5 (Continuing a Grant Request) & Section 5.3 (Modifying and Revoking Grants)
- **URL chuẩn**: [https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol](https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol)
- **Tài liệu bổ trợ**: [https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol#section-5](https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol#section-5), [https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol#section-5.2](https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol#section-5.2)

#### Các Điểm Cốt Lõi Kỹ Thuật (Key Takeaways):
- GNAP introduces a stateful Grant Continuation API via a dedicated `continue` endpoint and continuation access token.
- After user interaction finishes or when an app requires additional rights mid-session, the client submits a continuation request without restarting the entire authentication handshake.
- Supports dynamic grant modification: clients can request incremental privileges (step-up authorization) or release unused capabilities during a running session.
- The client can explicitly delete the grant via HTTP DELETE on the continuation URI, instantly revoking all associated access tokens and session handles across the ecosystem.
- Enables just-in-time privilege escalation tailored for micro-transactions, high-value checkouts, or accessing sensitive biometric APIs.

#### Khuyến Nghị Thực Thi Cho Chuẩn Super-App Mini-App Store:
- Utilize GNAP continuation endpoints to implement progressive disclosure and incremental permission prompts as users navigate into deep mini-app features.
- Trigger automated HTTP DELETE on the mini-app's continuation URI when the user closes the mini-app container or navigates back to the super-app home screen.
- Log all continuation requests and grant modification events in the platform audit trail for security compliance and fraud detection.

---

### 3.5. IETF GNAP Resource Servers: RS-to-AS Decoupling, Capability Delegation & Token Introspection (`gnap_std_128_05`)

- **Chủ đề**: `gnap_resource_servers_and_token_introspection`
- **Danh mục**: Identity, Authentication & Cryptographic Tokens
- **Cấp độ bằng chứng**: `official_standard`
- **Chuẩn tham chiếu**: draft-ietf-gnap-resource-servers Section 2 (Resource Server and AS Interaction) & Section 3 (Token Verification)
- **URL chuẩn**: [https://datatracker.ietf.org/doc/html/draft-ietf-gnap-resource-servers](https://datatracker.ietf.org/doc/html/draft-ietf-gnap-resource-servers)
- **Tài liệu bổ trợ**: [https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol](https://datatracker.ietf.org/doc/html/draft-ietf-gnap-core-protocol), [https://www.rfc-editor.org/rfc/rfc7662.html](https://www.rfc-editor.org/rfc/rfc7662.html)

#### Các Điểm Cốt Lõi Kỹ Thuật (Key Takeaways):
- The GNAP Resource Server (RS) specification formalizes the protocol interface between independent resource servers and the centralized GNAP Authorization Server.
- Allows the RS to introspect incoming proof-of-possession tokens, verify client key bindings, and retrieve fine-grained capability metadata (authorized actions, data paths, user attributes).
- Supports both reference tokens (resolved via real-time RS-to-AS backchannel queries) and structured cryptographic tokens (self-contained signed JWTs).
- Defines standard RS error signaling (`GNAP-Error` header or JSON response) communicating insufficient privileges, expired keys, or required step-up authorization parameters.
- Decouples microservices from identity logic, allowing third-party enterprise mini-app backends to integrate securely with the super-app authorization fabric.

#### Khuyến Nghị Thực Thi Cho Chuẩn Super-App Mini-App Store:
- Deploy standard GNAP RS introspection endpoints on all super-app core service gateways (payments, user profile, messaging, device sensors).
- Provide backend SDKs for third-party mini-app developers implementing GNAP RS token validation and client signature verification.
- Standardize RS error codes to trigger automatic client-side continuation flows when step-up authentication is demanded by downstream APIs.

---

### 3.6. OpenID FAPI 2.0: Message Signing Architecture, End-to-End Non-Repudiation & RFC 9421 Profiles (`fapi_msg_128_06`)

- **Chủ đề**: `fapi2_message_signing_architecture_rfc9421`
- **Danh mục**: Financial-Grade API & Consent Governance
- **Cấp độ bằng chứng**: `official_standard`
- **Chuẩn tham chiếu**: FAPI 2.0 Message Signing Section 3 (Message Signing Profile) & Section 4 (Signature Verification)
- **URL chuẩn**: [https://openid.net/specs/fapi-message-signing-2_0.html](https://openid.net/specs/fapi-message-signing-2_0.html)
- **Tài liệu bổ trợ**: [https://www.rfc-editor.org/rfc/rfc9421.html](https://www.rfc-editor.org/rfc/rfc9421.html), [https://openid.net/specs/fapi-2_0-security-profile.html](https://openid.net/specs/fapi-2_0-security-profile.html)

#### Các Điểm Cốt Lõi Kỹ Thuật (Key Takeaways):
- FAPI 2.0 Message Signing defines end-to-end cryptographic non-repudiation and payload integrity for high-value financial, banking, and e-commerce transactions.
- Standardizes on RFC 9421 HTTP Message Signatures, attaching detached cryptographic signatures over both HTTP requests and HTTP responses.
- Signatures cryptographically bind the sender identity, HTTP method, target URI, request headers, and content digest, preventing tampering by intermediary proxies or compromised network nodes.
- Mandates deterministic canonicalization and strict verification of signature metadata parameters including `keyid`, `alg`, `created`, and `expires`.
- Replaces deprecated custom JWT payload wrapping (e.g. JWS in HTTP body) with native, protocol-level HTTP header signatures that work transparently across diverse payload types (JSON, binary, streams).

#### Khuyến Nghị Thực Thi Cho Chuẩn Super-App Mini-App Store:
- Enforce FAPI 2.0 Message Signing for all mini-app payment checkouts, fund transfers, and digital contract signing operations exceeding high-risk financial thresholds.
- Implement native RFC 9421 signature generation within the super-app container core using the mobile device's Hardware Security Module (HSM) or Secure Enclave.
- Require all partner banking and payment gateways to return FAPI 2.0 signed HTTP responses, verifying the server signature before displaying payment success screens in the mini-app.

---

### 3.7. OpenID FAPI 2.0: Mandatory Component Coverage, Content-Digest Integrity & Anti-Tampering (`fapi_msg_128_07`)

- **Chủ đề**: `fapi2_mandatory_header_signing_and_content_digest`
- **Danh mục**: Financial-Grade API & Consent Governance
- **Cấp độ bằng chứng**: `official_standard`
- **Chuẩn tham chiếu**: FAPI 2.0 Message Signing Section 3.2 (HTTP Request Signing) & Section 3.3 (HTTP Response Signing)
- **URL chuẩn**: [https://openid.net/specs/fapi-message-signing-2_0.html](https://openid.net/specs/fapi-message-signing-2_0.html)
- **Tài liệu bổ trợ**: [https://www.rfc-editor.org/rfc/rfc9421.html#section-2](https://www.rfc-editor.org/rfc/rfc9421.html#section-2), [https://www.rfc-editor.org/rfc/rfc9530.html](https://www.rfc-editor.org/rfc/rfc9530.html)

#### Các Điểm Cốt Lõi Kỹ Thuật (Key Takeaways):
- Mandates that signed HTTP requests MUST cover at least `@method`, `@target-uri`, and `content-digest` (if a body is present), as well as authorization headers.
- The `content-digest` component utilizes SHA-256 or SHA-512 hashing (per RFC 9530 / RFC 9421) to ensure the request/response body cannot be modified or truncated in transit.
- For responses, the signature MUST cover `@status` and `content-digest`, linking the specific HTTP status code cryptographically to the returned data body.
- Prohibits selective or partial field validation: if any covered component fails signature verification or hash matching, the entire transaction MUST fail immediately with an HTTP 401/400.
- Protects against cross-endpoint replay attacks by binding the exact request target URI and path directly into the cryptographic signature envelope.

#### Khuyến Nghị Thực Thi Cho Chuẩn Super-App Mini-App Store:
- Configure the super-app API gateway to validate `@method`, `@target-uri`, `@status`, and `content-digest` on all signed API endpoints used by financial mini-apps.
- Provide automated client-side hashing in the mini-app runtime bridge so web developers do not manually construct `content-digest` strings.
- Log signature verification failures with component-level granularity in the security monitoring pipeline to detect network manipulation attempts.

---

### 3.8. OpenID FAPI 2.0: Algorithmic Rigor, Asymmetric Keys & Anti-Confusion Defenses (`fapi_msg_128_08`)

- **Chủ đề**: `fapi2_cryptographic_algorithms_and_key_hygiene`
- **Danh mục**: Financial-Grade API & Consent Governance
- **Cấp độ bằng chứng**: `official_standard`
- **Chuẩn tham chiếu**: FAPI 2.0 Message Signing Section 5 (Cryptographic Considerations) & FAPI 2.0 Security Profile Section 5.2.2
- **URL chuẩn**: [https://openid.net/specs/fapi-message-signing-2_0.html](https://openid.net/specs/fapi-message-signing-2_0.html)
- **Tài liệu bổ trợ**: [https://openid.net/specs/fapi-2_0-security-profile.html#section-5.2.2](https://openid.net/specs/fapi-2_0-security-profile.html#section-5.2.2), [https://www.rfc-editor.org/rfc/rfc8725.html](https://www.rfc-editor.org/rfc/rfc8725.html)

#### Các Điểm Cốt Lõi Kỹ Thuật (Key Takeaways):
- FAPI 2.0 Message Signing strictly restricts cryptographic algorithms to strong asymmetric ciphers: PS256 (RSASSA-PSS with SHA-256), ES256 (ECDSA with P-256 and SHA-256), or Ed25519.
- Explicitly forbids the use of RSA PKCS#1 v1.5 (`RS256`), symmetric HMAC algorithms (`HS256`), and the `none` algorithm to eliminate signature forgery vulnerabilities.
- Requires that signing keys are registered and verifiable via public JWKS endpoints or X.509 certificate chains bound to verified enterprise developer identities.
- Mandates distinct key separation: signing keys used for FAPI message signing MUST NOT be reused for TLS client authentication (mTLS) or token encryption.
- Enforces strict signature lifetime checks: signatures MUST include `created` and `expires` tags with maximum validity windows (typically <= 300 seconds) to prevent replay of captured signed messages.

#### Khuyến Nghị Thực Thi Cho Chuẩn Super-App Mini-App Store:
- Enforce PS256 or ES256 exclusively for mini-app message signing across all super-app developer tiers, rejecting legacy RSA PKCS#1 v1.5 algorithms during app validation.
- Implement automated cryptographic key rotation mechanisms in the developer portal, validating developer JWKS configurations against FAPI 2.0 strict requirements.
- Enforce a maximum signature time-to-live of 60 seconds on all native bridge financial invocations to eliminate message replay windows.

---

### 3.9. OAuth 2.0 Grant Management: Grant Query, Consent Inspection & Enterprise Auditability (`fapi_msg_128_09`)

- **Chủ đề**: `oauth_grant_management_query_and_inspection`
- **Danh mục**: Financial-Grade API & Consent Governance
- **Cấp độ bằng chứng**: `official_standard`
- **Chuẩn tham chiếu**: OAuth 2.0 Grant Management for OAuth 2.0 Section 3 (Grant Management Endpoint) & Section 4 (Query Grant)
- **URL chuẩn**: [https://openid.net/specs/oauth-v2-grant-management.html](https://openid.net/specs/oauth-v2-grant-management.html)
- **Tài liệu bổ trợ**: [https://www.rfc-editor.org/rfc/rfc9396.html](https://www.rfc-editor.org/rfc/rfc9396.html), [https://openid.net/specs/openid-federation-1_0.html](https://openid.net/specs/openid-federation-1_0.html)

#### Các Điểm Cốt Lõi Kỹ Thuật (Key Takeaways):
- The OAuth 2.0 Grant Management specification introduces a dedicated Grant Management Endpoint and a persistent `grant_id` handle representing user consent.
- Allows clients (or the super-app platform on behalf of users) to query active grants via `HTTP GET /grants/{grant_id}` to inspect currently authorized scopes, claims, and RAR authorization details.
- Enables mini-apps to perform programmatic self-auditing before initiating workflows, confirming whether necessary permissions remain valid without triggering disruptive error handling.
- Provides standardized JSON metadata detailing authorization status, resource boundaries, expiration timestamps, and delegating user identities.
- Crucial for enterprise compliance (GDPR Article 15/17, PDP regulations) requiring transparent consent visualization and verification.

#### Khuyến Nghị Thực Thi Cho Chuẩn Super-App Mini-App Store:
- Implement the Grant Management API in the super-app consent subsystem, assigning a unique `grant_id` to each user/mini-app relationship.
- Expose a client SDK method (`superapp.auth.getGrantStatus`) querying the grant endpoint to let mini-apps dynamically adapt UI states based on confirmed active permissions.
- Integrate grant inspection into the super-app account dashboard, allowing users to view real-time granular privileges granted to each installed mini-app.

---

### 3.10. OAuth 2.0 Grant Management: Granular Consent Revocation, Action Lifecycle & User Privacy Centers (`fapi_msg_128_10`)

- **Chủ đề**: `oauth_grant_management_revocation_and_lifecycle`
- **Danh mục**: Financial-Grade API & Consent Governance
- **Cấp độ bằng chứng**: `official_standard`
- **Chuẩn tham chiếu**: OAuth 2.0 Grant Management for OAuth 2.0 Section 5 (Revoke Grant) & Section 2 (Grant Management Actions)
- **URL chuẩn**: [https://openid.net/specs/oauth-v2-grant-management.html](https://openid.net/specs/oauth-v2-grant-management.html)
- **Tài liệu bổ trợ**: [https://openid.net/specs/oauth-v2-grant-management.html#section-5](https://openid.net/specs/oauth-v2-grant-management.html#section-5), [https://www.rfc-editor.org/rfc/rfc7009.html](https://www.rfc-editor.org/rfc/rfc7009.html)

#### Các Điểm Cốt Lõi Kỹ Thuật (Key Takeaways):
- Specifies standardized grant management actions: `create`, `query`, `replace`, and `revoke` passed via the `grant_management_action` parameter.
- Clients or users can revoke a specific grant by issuing an `HTTP DELETE /grants/{grant_id}` request to the grant management endpoint.
- Revoking a grant immediately invalidates the grant handle, all associated access tokens, refresh tokens, and downscoped authorization artifacts across all resource servers.
- Supports partial consent replacement: clients can update an existing grant (`grant_management_action=replace`) to drop unneeded scopes when a user disables specific features, minimizing data liability.
- Harmonizes consent management across multiple devices, so revoking a grant on the user's phone immediately terminates authorization for corresponding sessions on tablets, smart TVs, or web instances.

#### Khuyến Nghị Thực Thi Cho Chuẩn Super-App Mini-App Store:
- Provide a one-click 'Revoke Permissions' button in the mini-app settings sheet that executes standard `HTTP DELETE` against the Grant Management endpoint.
- Require mini-apps to execute `grant_management_action=replace` to shed sensitive privileges when users deactivate optional features within the mini-app.
- Synchronize grant revocation events across the super-app event bus to notify resource servers and terminate active WebSocket connections instantaneously.

---

### 3.11. IETF RFC 8417: Security Event Token (SET) Architecture, JSON Profiling & Asynchronous Signaling (`risc_set_128_11`)

- **Chủ đề**: `rfc8417_security_event_token_architecture`
- **Danh mục**: Security Events, Incident Response & Distributed Governance
- **Cấp độ bằng chứng**: `official_standard`
- **Chuẩn tham chiếu**: RFC 8417 Section 2 (Security Event Token Format) & Section 3 (Security Event Claims)
- **URL chuẩn**: [https://www.rfc-editor.org/rfc/rfc8417.html](https://www.rfc-editor.org/rfc/rfc8417.html)
- **Tài liệu bổ trợ**: [https://www.rfc-editor.org/rfc/rfc7519.html](https://www.rfc-editor.org/rfc/rfc7519.html), [https://openid.net/specs/openid-risc-profile-1_0.html](https://openid.net/specs/openid-risc-profile-1_0.html)

#### Các Điểm Cốt Lõi Kỹ Thuật (Key Takeaways):
- RFC 8417 defines the Security Event Token (SET), a standardized data structure based on JSON Web Tokens (JWT) for sharing asynchronous security event signals between administrative domains.
- SETs use the mandatory explicit media type `typ: secevent+jwt` in the JWS header to eliminate cross-JWT token confusion and substitution attacks.
- Introduces the `events` claim: a JSON object whose member keys are URIs identifying specific security event types (e.g. RISC or CAEP events) and whose values are event-specific metadata objects.
- Mandates core tracking claims including `iss` (event transmitter), `iat` (issuance time), `jti` (unique event identifier for deduplication), and `sub` or `sub_id` (affected security subject).
- Establishes a cryptographically signed, verifiable ledger of security incidents that can be exchanged between cloud services, platform operators, and federated applications.

#### Khuyến Nghị Thực Thi Cho Chuẩn Super-App Mini-App Store:
- Adopt RFC 8417 SETs as the universal wire format for all security and lifecycle telemetry distributed between the super-app core, identity providers, and mini-app enterprise backends.
- Enforce strict validation of `typ: secevent+jwt` and cryptographic JWS signatures before routing security event tokens through the platform message bus.
- Maintain an in-memory distributed cache of SET `jti` identifiers with a 24-hour expiration window to prevent event replay and duplicate processing.

---

### 3.12. OpenID RISC 1.0: Account Security Event Types, Credential Compromise & Incident Coordination (`risc_set_128_12`)

- **Chủ đề**: `openid_risc_event_types_and_incident_coordination`
- **Danh mục**: Security Events, Incident Response & Distributed Governance
- **Cấp độ bằng chứng**: `official_standard`
- **Chuẩn tham chiếu**: OpenID RISC Profile of IETF Security Events 1.0 Section 2 (Event Types) & Section 3 (RISC Event Definitions)
- **URL chuẩn**: [https://openid.net/specs/openid-risc-profile-1_0.html](https://openid.net/specs/openid-risc-profile-1_0.html)
- **Tài liệu bổ trợ**: [https://www.rfc-editor.org/rfc/rfc8417.html](https://www.rfc-editor.org/rfc/rfc8417.html), [https://openid.net/specs/openid-caep-specification-1_0.html](https://openid.net/specs/openid-caep-specification-1_0.html)

#### Các Điểm Cốt Lõi Kỹ Thuật (Key Takeaways):
- OpenID RISC (Risk and Incident Sharing and Coordination) defines standard security event types enabling coordinated incident response across ecosystem participants.
- Core standardized event types include `https://schemas.openid.net/secevent/risc/event-type/account-credential-change-required`, `account-purged`, `account-disabled`, `account-enabled`, and `sessions-revoked`.
- When the super-app or an identity provider detects a credential leak, account takeover attempt, or brute-force attack, it transmits a RISC event to all relying mini-apps.
- Allows mini-apps to immediately terminate local user sessions, clear sensitive cached documents, or force step-up biometric re-authentication without waiting for access token expiration.
- Crucial for containing cross-application contamination in multi-tenant super-app ecosystems where one compromised third-party service could otherwise compromise user trust.

#### Khuyến Nghị Thực Thi Cho Chuẩn Super-App Mini-App Store:
- Implement automated RISC event emission from the super-app Security Operations Center (SOC) to all active mini-app backends upon detecting compromised user credentials or device malware.
- Require enterprise mini-apps handling financial or healthcare transactions to subscribe to RISC feeds and immediately invalidate active session tokens upon receiving `sessions-revoked` events.
- Broadcast `account-purged` events when a user exercises their GDPR/PDPA right-to-erasure in the super-app, mandating that mini-apps purge local persistent records within 72 hours.

---

### 3.13. OpenID RISC 1.0: Privacy-Preserving Subject Identifiers, RFC 9493 & Anti-Correlation Protections (`risc_set_128_13`)

- **Chủ đề**: `risc_subject_identifiers_and_privacy_preservation`
- **Danh mục**: Security Events, Incident Response & Distributed Governance
- **Cấp độ bằng chứng**: `official_standard`
- **Chuẩn tham chiếu**: OpenID RISC Profile 1.0 Section 2.2 (Subject Identifiers) & IETF RFC 9493 (Subject Identifiers for Security Event Tokens)
- **URL chuẩn**: [https://openid.net/specs/openid-risc-profile-1_0.html](https://openid.net/specs/openid-risc-profile-1_0.html)
- **Tài liệu bổ trợ**: [https://www.rfc-editor.org/rfc/rfc9493.html](https://www.rfc-editor.org/rfc/rfc9493.html), [https://openid.net/specs/openid-connect-core-1_0.html#SubjectIDTypes](https://openid.net/specs/openid-connect-core-1_0.html#SubjectIDTypes)

#### Các Điểm Cốt Lõi Kỹ Thuật (Key Takeaways):
- RISC leverages RFC 9493 Subject Identifiers to reference affected user accounts without exposing unnecessary Personally Identifiable Information (PII) to unauthorized third parties.
- Supports diverse standardized format identifiers: `iss_sub` (issuer and subject URI), `email`, `phone_number`, `opaque_id`, and `did` (decentralized identifier).
- Mandates the use of Pairwise Pseudonymous Identifiers (PPID) in `iss_sub` format for third-party mini-apps, preventing different mini-app vendors from colluding and correlating user activities via security events.
- Transmitters MUST evaluate recipient permissions and only send subject formats the specific receiver is authorized to process, upholding privacy-by-design standards.
- Ensures that broadcasted security signals coordinate incident response across hundreds of ecosystem participants while strictly complying with global data protection laws.

#### Khuyến Nghị Thực Thi Cho Chuẩn Super-App Mini-App Store:
- Enforce pairwise pseudonymous subject identifiers (`iss_sub` with app-scoped salt) when broadcasting RISC events to third-party mini-app developers.
- Strictly prohibit transmitting plaintext email or phone subject identifiers to untrusted or low-tier mini-apps, using opaque alias mappings maintained by the super-app platform.
- Audit all subject identifier schemas against RFC 9493 during mini-app store security compliance reviews.

---

### 3.14. IETF RFC 8935: Push-Based Security Event Token (SET) Delivery Over HTTP & Webhook Resiliency (`risc_set_128_14`)

- **Chủ đề**: `rfc8935_push_based_set_delivery`
- **Danh mục**: Security Events, Incident Response & Distributed Governance
- **Cấp độ bằng chứng**: `official_standard`
- **Chuẩn tham chiếu**: RFC 8935 Section 2 (Delivery Protocol) & Section 3 (Error Handling)
- **URL chuẩn**: [https://www.rfc-editor.org/rfc/rfc8935.html](https://www.rfc-editor.org/rfc/rfc8935.html)
- **Tài liệu bổ trợ**: [https://www.rfc-editor.org/rfc/rfc8417.html](https://www.rfc-editor.org/rfc/rfc8417.html), [https://www.rfc-editor.org/rfc/rfc8705.html](https://www.rfc-editor.org/rfc/rfc8705.html)

#### Các Điểm Cốt Lõi Kỹ Thuật (Key Takeaways):
- RFC 8935 specifies the standardized push delivery protocol for transmitting SETs from an event transmitter to a receiver via HTTP POST webhooks.
- The transmitter issues an `HTTP POST` request to the receiver's delivery endpoint with `Content-Type: application/secevent+jwt` carrying the compact serialized SET in the request body.
- Receivers validate the SET and respond with `HTTP 202 Accepted` on success, or an explicit JSON error document (e.g. `err: invalid_key`, `invalid_issuer`, `authentication_failed`).
- Mandates strong mutual authentication: transmitters and receivers must authenticate via mutual TLS (mTLS) or OAuth 2.0 Bearer/DPoP tokens to prevent spoofed event injection.
- Defines resilient delivery behavior: transmitters MUST implement exponential backoff and persistent retry queuing to handle temporary network partitions or receiver downtime.

#### Khuyến Nghị Thực Thi Cho Chuẩn Super-App Mini-App Store:
- Deploy RFC 8935 push-based webhooks as the default distribution mechanism for delivering real-time security events to tier-1 enterprise and financial mini-app cloud backends.
- Require mTLS certificate authentication (per RFC 8705) on all push delivery webhook endpoints registered in the mini-app developer console.
- Implement a resilient dead-letter queue (DLQ) with exponential backoff retries for up to 72 hours when delivering critical account compromise SETs.

---

### 3.15. IETF RFC 8936: Poll-Based Security Event Token (SET) Delivery Over HTTP & Air-Gapped Resiliency (`risc_set_128_15`)

- **Chủ đề**: `rfc8936_poll_based_set_delivery`
- **Danh mục**: Security Events, Incident Response & Distributed Governance
- **Cấp độ bằng chứng**: `official_standard`
- **Chuẩn tham chiếu**: RFC 8936 Section 2 (Polling Interface) & Section 3 (Event Acknowledgement)
- **URL chuẩn**: [https://www.rfc-editor.org/rfc/rfc8936.html](https://www.rfc-editor.org/rfc/rfc8936.html)
- **Tài liệu bổ trợ**: [https://www.rfc-editor.org/rfc/rfc8417.html](https://www.rfc-editor.org/rfc/rfc8417.html), [https://www.rfc-editor.org/rfc/rfc8935.html](https://www.rfc-editor.org/rfc/rfc8935.html)

#### Các Điểm Cốt Lõi Kỹ Thuật (Key Takeaways):
- RFC 8936 defines a poll-based delivery protocol allowing receivers behind NATs, firewalls, or strict inbound security perimeters to pull SETs from an event transmitter.
- Receivers issue `HTTP POST` requests to the polling endpoint with JSON requests specifying `maxEvents` and `returnImmediately` flags, supporting long-polling patterns for low-latency event delivery.
- Features built-in acknowledgement tracking: poll requests include `ack` arrays confirming receipt of previous `jti` event IDs, enabling the transmitter to reliably advance the queue.
- Provides error-signaling mechanisms (`setErrs`) allowing receivers to report specific validation failures (e.g. signature error, unknown subject) without stalling overall queue processing.
- Essential for enterprise, government, or on-premise partner mini-apps that cannot expose public inbound HTTP webhook endpoints to the super-app cloud.

#### Khuyến Nghị Thực Thi Cho Chuẩn Super-App Mini-App Store:
- Provide an RFC 8936 polling endpoint on the super-app event gateway for enterprise mini-app backends operating behind corporate firewalls or strict security zones.
- Support HTTP long-polling with a 30-second hold window to minimize network overhead while ensuring sub-second delivery of security alerts.
- Mandate explicit `ack` confirmation from polling clients before purging security events from the transmitter's outbound buffer.

---

## 4. Kiến Trúc Tích Hợp Hệ Thống Thực Tế

### 4.1. Sơ Đồ Phân Tầng Phân Quyền & Bảo Vệ Giao Dịch

```
+---------------------------------------------------------------------------------------+
|                                  SUPER-APP CLIENT LAYER                               |
|                                                                                       |
|  +-----------------------------------+     +---------------------------------------+  |
|  |   Mini-App WebView Container      |     |     Super-App Native Host Shell       |  |
|  |                                   |     |                                       |  |
|  | - JS Bridge Client Key Generation |<--->| - Hardware Keystore / Secure Enclave  |  |
|  | - GNAP Request Formulation        |     | - FAPI RFC 9421 HTTP Message Signer   |  |
|  | - Transparent Signature Headers   |     | - Native User Consent Bottom Sheet    |  |
|  +-----------------------------------+     +---------------------------------------+  |
+------------------------------------------+--------------------------------------------+
                                           | HTTPS / mTLS (RFC 8705)
                                           v
+---------------------------------------------------------------------------------------+
|                             SUPER-APP CLOUD & API GATEWAY                             |
|                                                                                       |
|  +-----------------------------------+     +---------------------------------------+  |
|  |    GNAP Authorization Server      |     |      FAPI 2.0 Security Gateway        |  |
|  |                                   |     |                                       |  |
|  | - Grant Negotiation & Continuation|     | - RFC 9421 HTTP Signature Verifier    |  |
|  | - Multi-Token Issuance & PoP Bind |     | - RFC 9530 Content-Digest Hash Check  |  |
|  | - Grant Management (grant_id CRUD)|     | - Algorithmic Rigor (PS256 / ES256)   |  |
|  +-----------------------------------+     +---------------------------------------+  |
|                                          |                                            |
|                                          v                                            |
|  +---------------------------------------------------------------------------------+  |
|  |          Security Operations Center (SOC) & Distributed SET Event Bus           |  |
|  |                                                                                 |  |
|  | - Incident Detection & Account Anomaly Analysis Engine                         |  |
|  | - RFC 8417 Security Event Token (SET) Generator (`typ: secevent+jwt`)          |  |
|  | - RFC 9493 Privacy-Preserving Subject ID Mapper (Pairwise Pseudonyms `iss_sub`)|  |
|  +---------------------------------------------------------------------------------+  |
+--------------------------+-------------------------------------+----------------------+
                           |                                     |
           RFC 8935 Push   |                     RFC 8936 Poll   |
           Webhook (mTLS)  |                     Long-Poll Queue |
                           v                                     v
+--------------------------------------+     +------------------------------------------+
| Enterprise Partner Cloud Backends    |     | On-Premise / Firewalled Mini-App Servers |
| (Banking / Telco / Healthcare)       |     | (Government / Regulated Enterprise)      |
|                                      |     |                                          |
| - Validate Inbound SETs              |     | - Pull Security Signals with Batch ACK   |
| - Invalidate Local App Sessions      |     | - Local Revocation & Credential Rotation |
+--------------------------------------+     +------------------------------------------+
```

### 4.2. Ma Trận Quy Chuẩn Triển Khai (Normative Requirements vs Vendor Best Practices)

| Lĩnh vực | Yêu cầu Bắt buộc (Normative) | Khuyến nghị Nâng cao (Best Practice) | Lộ trình Đề xuất |
|---|---|---|---|
| **Cơ chế Phân quyền Mini-App** | Triển khai Proof-of-Possession bắt buộc cho mọi token truy cập; loại bỏ Bearer token không ràng buộc khóa. | Chuyển đổi từ OAuth 2.0 scopes thô sang mô hình GNAP fine-grained access request với client key binding. | Áp dụng ngay cho các mini-app cấp 1 (Tài chính/Ví); mở rộng toàn sàn trong 6 tháng. |
| **Ký số Giao dịch Tài chính** | Tuân thủ FAPI 2.0 Message Signing (RFC 9421); bắt buộc ký `@method`, `@target-uri`, `@status` và `content-digest`. | Khóa bí mật ký số được tạo và lưu trữ trong Secure Enclave / Android Keystore của thiết bị di động. | Bắt buộc đối với mọi giao dịch chuyển tiền hoặc thanh toán vượt hạn mức 5.000.000 VND. |
| **Quản lý & Thu hồi Quyền (Consent)** | Cung cấp định danh `grant_id` duy nhất; hỗ trợ endpoint thu hồi quyền qua `HTTP DELETE /grants/{grant_id}`. | Cho phép thay thế quyền bán phần (`grant_management_action=replace`) khi người dùng tắt tính năng phụ. | Đưa vào bộ SDK v3.0 của super-app và giao diện trung tâm quyền riêng tư (Privacy Center). |
| **Phát tán Tín hiệu An ninh** | Định dạng tín hiệu an ninh chuẩn hóa theo IETF RFC 8417 SET (`typ: secevent+jwt`). | Hỗ trợ song song RFC 8935 (Push cho đối tác đám mây) và RFC 8936 (Poll cho đối tác sau firewall). | Triển khai tích hợp vào hệ thống SIEM/SOC của Super-App. |
| **Bảo vệ Danh tính Người dùng** | Sử dụng Subject Identifiers dạng Pairwise Pseudonymous (`iss_sub`) theo RFC 9493 khi phát SET ra ngoài. | Cấm hoàn toàn việc phát tán số điện thoại hoặc email dạng plain-text trong các bản tin cảnh báo an ninh. | Kiểm tra tự động tại vòng kiểm duyệt bảo mật API gateway. |

---

## 5. Kết Luận & Định Hướng Iteration 129

Milestone 128 đã bổ sung 15 phát hiện chuẩn hóa có giá trị pháp lý và kỹ thuật cao nhất vào kho tri thức Canonical Findings (nâng tổng số lên **1871 findings**). Việc tích hợp IETF GNAP, OpenID FAPI 2.0 Message Signing và OpenID RISC / RFC 8417 SET thiết lập một chuẩn mực bảo mật toàn diện, vững chắc ở cấp độ hạ tầng tài chính và chính phủ số cho hệ sinh thái Super-App Mini-App Store.

Trong vòng tiếp theo (Iteration 129), kiến trúc sẽ tiếp tục hoàn thiện các chuẩn mực về quản lý phiên đăng nhập liên minh (OIDC Identity Federation), cơ chế xác thực không mật khẩu cấp độ phần cứng FIDO2 / WebAuthn L3 cải tiến, và quy trình chứng thực tính toàn vẹn ứng dụng di động (Play Integrity / App Attest) phục vụ trực tiếp cho việc vận hành kho ứng dụng an toàn.
