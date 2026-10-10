# Chuyên đề 125 (Iteration 129): Decentralized Identity, Credential Issuance (OID4VCI), Cryptographic Revocation Lists (OAuth Status List) & Selective Disclosure (SD-JWT)

## 1. Tổng quan & Bối cảnh Chiến lược trong Super-App Mini-App Store

Khi Super-App mở rộng thành nền tảng hạ tầng số quốc gia và trung tâm dịch vụ công dân – tích hợp thanh toán tài chính, viễn thông, thẻ bảo hiểm y tế, giấy phép lái xe (mDL) và vé dịch vụ doanh nghiệp – việc bảo vệ quyền riêng tư dữ liệu cá nhân (GDPR/PDPA) và cung cấp cơ chế định danh số phi tập trung (Decentralized Identity) trở thành yêu cầu sống còn. Nếu tiếp tục sử dụng các cơ chế xác thực danh tính truyền thống (truyền nguyên vẹn profile JSON chứa toàn bộ CCCD, số điện thoại, ngày sinh và địa chỉ cho mọi mini-app), người dùng sẽ đối mặt nguy cơ lộ lọt dữ liệu quy mô lớn và bị theo dõi chéo trái phép.

Hơn nữa, các cơ chế kiểm tra thu hồi chứng chỉ/token truyền thống như CRL (Certificate Revocation List quá cồng kềnh) hoặc OCSP (gây rò rỉ quyền riêng tư vì máy chủ phát hành biết chính xác người dùng đang xác thực ở đâu) không thể đáp ứng được môi trường di động và mạng tế bào băng thông thấp.

Milestone 129 chuẩn hóa 3 công nghệ định danh số, phát hành chứng chỉ và bảo mật thu hồi hiện đại bậc nhất:
1. **OpenID for Verifiable Credential Issuance (OID4VCI 1.0)** (`openid-4-verifiable-credential-issuance-1_0`): Đặc tả chuẩn của OpenID Foundation định nghĩa toàn diện kiến trúc phát hành chứng chỉ xác thực (Verifiable Credentials) cho ví di động (wallet) dựa trên nền tảng OAuth 2.0; chuẩn hóa endpoint khám phá siêu dữ liệu tại `/.well-known/openid-credential-issuer`, danh mục chứng chỉ hỗ trợ `credential_configurations_supported`, luồng cấp phát mã ủy quyền trước (Pre-Authorized Code flow) kết hợp mã giao dịch bảo vệ `tx_code`, cơ chế chứng minh sở hữu khóa mật mã (Proof of Possession - PoP) gắn với khóa phần cứng HSM và số ngẫu nhiên dùng một lần `c_nonce`, cùng các endpoint phát hành đồng bộ, phát hành trễ (Deferred Issuance) và phát hành theo lô (`/batch_credential`), tích hợp vòng đời thông báo kiểm toán `/notification`.
2. **IETF OAuth Token Status List** (`draft-ietf-oauth-status-list`): Đặc tả chuẩn IETF cho danh sách trạng thái token và chứng chỉ dạng bitstring nén siêu nhỏ gọn; biểu diễn trạng thái của hàng triệu chứng chỉ bằng chuỗi bit `lst` (hỗ trợ phân bổ 1 bit cho nhị phân hoặc 2 bit cho 4 trạng thái: hợp lệ, thu hồi, tạm ngưng); cơ chế tham chiếu trạng thái `status.status_list` trong token với con trỏ chỉ mục `idx` và đường dẫn URI; kỹ thuật nén Deflate (`c: 'DEF'`) giúp danh sách 1 triệu token chỉ nặng ~120KB; tận dụng HTTP Caching (RFC 9111) loại bỏ hoàn toàn khả năng theo dõi người dùng của đơn vị phát hành (Verifier Privacy Preservation); và kiến trúc phân mảnh danh sách (sharding) theo từng danh mục mini-app để tránh nghẽn hạ tầng ký số.
3. **IETF Selective Disclosure for JSON Web Tokens (SD-JWT)** (`draft-ietf-oauth-selective-disclosure-jwt`): Chuẩn hóa cơ chế tiết lộ có chọn lọc (Selective Disclosure) cho JWT; biến đổi từng thuộc tính danh tính nhạy cảm thành một cấu trúc Disclosure chứa muối ngẫu nhiên (salt >= 128-bit) và mã băm SHA-256 đưa vào mảng `_sd` trong JWT; cơ chế ràng buộc khóa Key Binding JWT (KB-JWT) chống đánh cắp và phát lại chứng chỉ bằng chữ ký số tức thời gắn với khóa phần cứng và nonce của bên xác minh; hỗ trợ cấu trúc dữ liệu đệ quy lồng nhau và tiết lộ từng phần tử mảng JSON thông qua ký hiệu `...`; cùng quy trình 7 bước xác minh toàn vẹn mật mã và giao diện cấp phép (Consent UI) trên Super-App nhằm ngăn chặn mini-app thu thập dữ liệu dư thừa.

---

## 2. Bảng Tổng Hợp 15 Chuẩn Mực Nghiên Cứu Chi Tiết

| ID | Tiêu đề Chuẩn Mực | Danh mục | Cấp độ Bằng chứng | Nguồn / Đặc tả Tham chiếu |
|---|---|---|---|---|
| `oid4vci_129_01` | **OID4VCI 1.0: Architecture, Discovery Metadata & Supported Credential Configurations** | Decentralized Identity, Credential Issuance & Verifiable State | `official_standard` | [OID4VCI 1.0 Section 11.2 (Credential Issuer Metadata) & Section 5](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html) |
| `oid4vci_129_02` | **OID4VCI 1.0: Credential Offer Protocol, Pre-Authorized Code Flow & Transaction Code (tx_code)** | Decentralized Identity, Credential Issuance & Verifiable State | `official_standard` | [OID4VCI 1.0 Section 4 (Credential Offer) & Section 4.1.1 (Pre-Authorized Code Grant)](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html) |
| `oid4vci_129_03` | **OID4VCI 1.0: Cryptographic Proof of Possession (PoP), Ephemeral c_nonce & Secure Hardware Binding** | Decentralized Identity, Credential Issuance & Verifiable State | `official_standard` | [OID4VCI 1.0 Section 7.2 (Proof Types) & Section 7.2.1 (JWT Proof)](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html) |
| `oid4vci_129_04` | **OID4VCI 1.0: Synchronous, Deferred & Batch Credential Issuance Endpoints** | Decentralized Identity, Credential Issuance & Verifiable State | `official_standard` | [OID4VCI 1.0 Section 7 (Credential Endpoint) & Section 8 (Batch Endpoint)](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html) |
| `oid4vci_129_05` | **OID4VCI 1.0: Notification Endpoint, Issuance Acknowledgement & Regulatory Audit Lifecycle** | Decentralized Identity, Credential Issuance & Verifiable State | `official_standard` | [OID4VCI 1.0 Section 10 (Notification Endpoint)](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html) |
| `token_status_list_129_06` | **IETF OAuth Token Status List: Data Model, Bitstring Representation & Multi-Bit Semantics** | Decentralized Identity, Credential Issuance & Verifiable State | `official_standard` | [draft-ietf-oauth-status-list Section 4 (Status List Data Model) & Section 4.2](https://datatracker.ietf.org/doc/draft-ietf-oauth-status-list/) |
| `token_status_list_129_07` | **IETF OAuth Token Status List: Status Claim Reference, Index Pointers & Dynamic Scope Binding** | Decentralized Identity, Credential Issuance & Verifiable State | `official_standard` | [draft-ietf-oauth-status-list Section 5 (Referencing a Status List) & Section 5.1](https://datatracker.ietf.org/doc/draft-ietf-oauth-status-list/) |
| `token_status_list_129_08` | **IETF OAuth Token Status List: Deflate Compression, HTTP Caching & Verifier Privacy Guarantees** | Decentralized Identity, Credential Issuance & Verifiable State | `official_standard` | [draft-ietf-oauth-status-list Section 4.2.1 (Compression) & Section 7 (Privacy)](https://datatracker.ietf.org/doc/draft-ietf-oauth-status-list/) |
| `token_status_list_129_09` | **IETF OAuth Token Status List: Multi-Tenant Partitioning, Sharded Buckets & Zero-Contention Updates** | Decentralized Identity, Credential Issuance & Verifiable State | `official_standard` | [draft-ietf-oauth-status-list Section 6 (Operational) & Section 8 (Security)](https://datatracker.ietf.org/doc/draft-ietf-oauth-status-list/) |
| `token_status_list_129_10` | **IETF OAuth Token Status List: Store Compliance Verification Pipeline & Offline Grace Windows** | Decentralized Identity, Credential Issuance & Verifiable State | `official_standard` | [draft-ietf-oauth-status-list Section 7.2 (Stale Status) & Section 8.3 (DoS)](https://datatracker.ietf.org/doc/draft-ietf-oauth-status-list/) |
| `sd_jwt_129_11` | **IETF SD-JWT: Data Architecture, Disclosures Anatomy & 128-Bit Cryptographic Salt Digests** | Decentralized Identity, Credential Issuance & Verifiable State | `official_standard` | [draft-ietf-oauth-selective-disclosure-jwt Section 5 (Disclosures)](https://datatracker.ietf.org/doc/draft-ietf-oauth-selective-disclosure-jwt/) |
| `sd_jwt_129_12` | **IETF SD-JWT: Key Binding JWT (KB-JWT), cnf Confirmation Claims & Replay Defense Nonces** | Decentralized Identity, Credential Issuance & Verifiable State | `official_standard` | [draft-ietf-oauth-selective-disclosure-jwt Section 6 (Key Binding)](https://datatracker.ietf.org/doc/draft-ietf-oauth-selective-disclosure-jwt/) |
| `sd_jwt_129_13` | **IETF SD-JWT: Complex Data Structures, Nested Objects & Array Element Selective Disclosure** | Decentralized Identity, Credential Issuance & Verifiable State | `official_standard` | [draft-ietf-oauth-selective-disclosure-jwt Section 5.2 (Array Elements) & Section 5.3](https://datatracker.ietf.org/doc/draft-ietf-oauth-selective-disclosure-jwt/) |
| `sd_jwt_129_14` | **IETF SD-JWT: 7-Step Verifier Processing Pipeline, Digest Verification & Anti-Tampering Rules** | Decentralized Identity, Credential Issuance & Verifiable State | `official_standard` | [draft-ietf-oauth-selective-disclosure-jwt Section 7 (Verification)](https://datatracker.ietf.org/doc/draft-ietf-oauth-selective-disclosure-jwt/) |
| `sd_jwt_129_15` | **IETF SD-JWT: Super-App Wallet Integration, User Consent UI & Minimization Store Gating** | Decentralized Identity, Credential Issuance & Verifiable State | `official_standard` | [draft-ietf-oauth-selective-disclosure-jwt Section 8 (Privacy) & Section 9 (Security)](https://datatracker.ietf.org/doc/draft-ietf-oauth-selective-disclosure-jwt/) |

---

## 3. Phân Tích Kỹ Thuật Chuyên Sâu Từng Trụ Cột

### 3.1. OpenID for Verifiable Credential Issuance (OID4VCI 1.0)
- **Cấu hình Discovery & Khám phá Issuer**: Super-App triển khai endpoint `/.well-known/openid-credential-issuer` công bố công khai cấu trúc metadata: định danh `credential_issuer`, đường dẫn `credential_endpoint`, danh sách thuật toán ký và đặc biệt là mảng `credential_configurations_supported`. Mỗi loại chứng chỉ (chứng nhận tài xế, thẻ thành viên, giấy chứng nhận tiêm chủng, thẻ ngân hàng đối tác) được định nghĩa chặt chẽ về định dạng (`format`: `jwt_vc_json`, `mso_mdoc`, `sd_jwt_vc`), các thuộc tính xác thực và thuật toán mật mã bắt buộc (`cryptographic_binding_methods_supported`: `jwk`, `did`).
- **Luồng Pre-Authorized Code & Giao dịch mã an toàn (`tx_code`)**: Đối với các mini-app cấp phát chứng chỉ trực tiếp (ví dụ: cấp vé điện tử sau khi thanh toán qua mini-app mua vé xem phim hoặc hóa đơn điện lực), OID4VCI cung cấp luồng ủy quyền trước. Khi người dùng hoàn tất giao dịch, mini-app tạo ra một `pre-authorized_code` và chuyển giao cho ví Super-App qua deep-link `credential_offer`. Để chống chặn bắt mã ủy quyền trên đường truyền hoặc trong log ứng dụng, chuẩn bắt buộc áp dụng `tx_code` (mã giao dịch OTP gửi qua SMS/thông báo hệ thống), đòi hỏi người dùng nhập đúng mã này thì ví Super-App mới có thể đổi mã lấy token truy cập tại Authorization Server.
- **Chứng minh quyền sở hữu khóa (Proof of Possession - PoP)**: Ngăn chặn tuyệt đối việc cấp chứng chỉ cho bên giả mạo. Trong yêu cầu gửi tới `/credential`, ví Super-App phải tạo một đối tượng `proof` (chữ ký số JWT) ký bởi khóa bí mật nằm trong chip bảo mật phần cứng của thiết bị di động (Secure Enclave trên iOS hoặc StrongBox Keymaster trên Android). Để chống phát lại chữ ký, đơn vị phát hành cung cấp giá trị số ngẫu nhiên `c_nonce` có hiệu lực ngắn (60 giây), bắt buộc phải được bọc trong payload của Proof.
- **Phát hành theo lô & Phát hành trễ (Deferred Issuance)**: Khi mini-app cần phát hành nhiều chứng chỉ cùng lúc (ví dụ: trọn bộ vé cho gia đình hoặc thẻ nhân viên kèm giấy phép ra vào khu vực an ninh), OID4VCI cung cấp endpoint `/batch_credential`, tối ưu hóa số vòng mạng chỉ trong một giao dịch nguyên tử. Trường hợp các chứng chỉ đòi hỏi phê duyệt thủ công từ cơ quan nhà nước hoặc ngân hàng lõi, endpoint trả về trạng thái HTTP 202 kèm `transaction_id`, cho phép ví Super-App định kỳ truy vấn tại `/deferred_credential` mà không làm treo phiên giao dịch.

### 3.2. IETF OAuth Token Status List (Danh Sách Trạng Thái Thu Hồi Mật Mã)
- **Mô hình Dữ liệu Bitstring & Phân bổ Trạng thái**: Thay vì các danh sách thu hồi cồng kềnh chứa hàng nghìn mã số định danh, IETF OAuth Status List mã hóa trạng thái dưới dạng một mảng bit liên tục `lst`. Với phân bổ 2 bit cho mỗi chứng chỉ, hệ thống biểu diễn được 4 trạng thái cốt lõi: `0x00` (Hợp lệ / Valid), `0x01` (Bị thu hồi / Revoked), `0x02` (Tạm ngưng / Suspended) và `0x03` (Dự phòng mở rộng).
- **Ràng buộc Chứng chỉ bằng Con trỏ Chỉ mục (`idx`)**: Mỗi token hoặc chứng chỉ số được cấp phát chỉ cần chứa một claim nhỏ gọn:
  ```json
  "status": {
    "status_list": {
      "idx": 45021,
      "uri": "https://identity.superapp.vn/status/fintech-2026-q4.jwt"
    }
  }
  ```
  Con trỏ `idx` trỏ trực tiếp đến vị trí bit trong chuỗi bitstring đã ký số.
- **Bảo toàn Quyền riêng tư Tuyệt đối cho Bên Xác minh (Verifier Privacy)**: Khác biệt mang tính cách mạng so với giao thức OCSP truyền thống: trong OCSP, mỗi lần mini-app kiểm tra một chứng chỉ, nó phải gửi yêu cầu lên CA kèm ID của chứng chỉ đó, khiến CA theo dõi được toàn bộ hành vi và vị trí giao dịch của người dùng. Với OAuth Status List, bên xác minh (hoặc bản thân Super-App) tải toàn bộ file danh sách trạng thái bitstring về bộ nhớ đệm cục bộ (cached). Quá trình kiểm tra trạng thái diễn ra hoàn toàn offline trên RAM của thiết bị thông qua phép toán dịch bit đơn giản:
  $$\text{byte\_index} = \lfloor \frac{\text{idx} \times \text{bits}}{8} \rfloor$$
  Do đó, đơn vị phát hành hoàn toàn không biết được thời điểm hoặc địa điểm mà chứng chỉ được xác minh.
- **Tối ưu hóa Băng thông Mạng bằng Deflate**: Chuỗi bitstring được nén bằng thuật toán Deflate (`c: 'DEF'`). Đối với một tập hợp 1.000.000 chứng chỉ (phân bổ 1 bit = 125.000 bytes), dữ liệu sau khi nén chỉ còn khoảng 120KB. Kết hợp với tiêu đề HTTP `Cache-Control: public, max-age=300, must-revalidate` và `ETag`, các mini-app có thể kiểm tra thu hồi tức thì với chi phí truyền tải mạng xấp xỉ bằng không.
- **Phân vùng Danh sách Đa Khách thuê (Multi-Tenant Sharding)**: Super-App phân vùng các danh sách trạng thái thành từng khối độc lập (65.536 mục/khối) phân bổ theo ngành dọc (tài chính, viễn thông, thương mại, bảo hiểm). Khi một đối tác viễn thông thu hồi một loạt thẻ cào hoặc tài khoản bị tấn công, sự thay đổi bitstring chỉ làm vô hiệu hóa bộ nhớ đệm của shard tương ứng, không gây ảnh hưởng tới hàng chục triệu người dùng của các dịch vụ khác.

### 3.3. IETF SD-JWT (Tiết Lộ Có Chọn Lọc Cho Token Danh Tính)
- **Giải phẫu Cấu trúc Disclosure & Muối Mật Mã (Salt)**: Mỗi thuộc tính thông tin cá nhân được đóng gói thành một bộ ba JSON:
  $$\text{Disclosure} = \text{base64url}(\text{JSON}([ \text{salt}, \text{claim\_name}, \text{claim\_value} ] ))$$
  Trong đó, `salt` là một chuỗi ngẫu nhiên có độ dài tối thiểu 128-bit được sinh bởi bộ sinh số ngẫu nhiên an toàn (`crypto.getRandomValues()`). Đơn vị phát hành chỉ tính mã băm SHA-256 của chuỗi Disclosure này và đưa mảng băm vào thuộc tính `_sd` trong JWT gốc:
  ```json
  {
    "iss": "https://identity.superapp.vn",
    "sub": "user_987654",
    "_sd": [
      "Cr0Q...a81Y",
      "jW7v...x99Q",
      "mP2k...z44T"
    ]
  }
  ```
- **Ràng buộc Khóa Người dùng (Key Binding JWT - KB-JWT)**: Nhằm ngăn chặn trường hợp kẻ tấn công đứng giữa sao chép các Disclosure được người dùng tiết lộ để đem đi giả mạo tại các mini-app khác, SD-JWT định nghĩa cơ chế KB-JWT. Đơn vị phát hành nhúng khóa công khai của ví người dùng vào claim `cnf` (confirmation) trong SD-JWT. Khi trình bày chứng chỉ cho mini-app, ví Super-App tạo ra một chữ ký KB-JWT ephemeral chứa `aud` (URL định danh mini-app), `nonce` (thách thức ngẫu nhiên từ mini-app, hiệu lực tối đa 60 giây), và `sd_hash` (mã băm SHA-256 của toàn bộ các chuỗi Disclosure được tiết lộ). Định dạng trình bày hoàn chỉnh được phân tách bằng dấu tilde (`~`):
  $$\langle \text{Issuer-Signed-SD-JWT} \rangle \sim \langle \text{Disclosure}_1 \rangle \sim \dots \sim \langle \text{Disclosure}_n \rangle \sim \langle \text{KB-JWT} \rangle$$
- **Cấu trúc Dữ liệu Phức tạp & Tiết lộ Phần tử Mảng (`...`)**: Đối với các danh sách (như danh sách quyền hạn, danh sách chức vụ hoặc các quốc tịch), SD-JWT hỗ trợ tiết lộ từng phần tử riêng lẻ mà không làm xáo trộn thứ tự mảng. Các phần tử được giấu kín được thay thế bằng một đối tượng chứa duy nhất thuộc tính dấu ba chấm `{"..." : "<hash_digest>"}`.
- **Quy trình 7 Bước Xác minh Chống Giả mạo**:
  1. Phân tách chuỗi trình bày theo dấu `~`.
  2. Xác minh chữ ký số của Issuer trên SD-JWT bằng JWKS công khai; kiểm tra thời hạn `exp`, `nbf`.
  3. Duyệt từng chuỗi Disclosure, giải mã base64url, tính mã băm SHA-256 và đối soát khớp tuyệt đối với các giá trị trong mảng `_sd` (hoặc `...` của mảng).
  4. Phát hiện và loại bỏ các Disclosure rác (không tồn tại mã băm trong SD-JWT) nhằm chống tấn công injection.
  5. Kiểm tra sự tồn tại của claim `cnf` và chữ ký số KB-JWT.
  6. Đối soát mã băm `sd_hash` trong KB-JWT với tập hợp các Disclosure được trình bày.
  7. Kiểm tra `aud` và `nonce` trong KB-JWT để đảm bảo phiên xác thực là duy nhất và hướng đến đúng mini-app hiện tại.

---

## 4. Kiến Trúc Tích Hợp Vào Nền Tảng Super-App Mini-App Store

```
   +-----------------------------------------------------------------------------------+
   |                             SUPER-APP NATIVE HOST CONTAINER                        |
   |                                                                                   |
   |   +-----------------------+     +-----------------------+     +---------------+   |
   |   | Hardware Key Enclave  |     |   Local Bitstring     |     | Privacy-First |   |
   |   | (Secure Enclave /     |     |   Status List Cache   |     | Consent Sheet |   |
   |   |  Android StrongBox)   |     |   (2-bit DEF Shards)  |     | (SD-JWT View) |   |
   |   +-----------+-----------+     +-----------+-----------+     +-------+-------+   |
   +---------------|-----------------------------|-------------------------|-----------+
                   |                             |                         |
                   v                             v                         v
   +-----------------------------------------------------------------------------------+
   |                            SUPER-APP RUNTIME SECURITY BRIDGE                      |
   |                                                                                   |
   |   * OID4VCI Bridge: superapp.wallet.requestCredential(offerUri, txCode)          |
   |   * SD-JWT Presentation: superapp.identity.presentClaims(requestedClaims, nonce)  |
   |   * Status Check: superapp.auth.verifyTokenStatus(credentialStatusClaim)          |
   +---------------------------------------------+-------------------------------------+
                                                 |
                                                 v
   +-----------------------------------------------------------------------------------+
   |                            THIRD-PARTY MINI-APP SANDBOX                           |
   |                                                                                   |
   |   - Nhận Disclosure đã người dùng đồng ý (VD: over_18=true, giấu birthdate)       |
   |   - Thực thi xác minh chữ ký Issuer + KB-JWT cục bộ trong WebAssembly sandbox    |
   |   - Tra cứu tức thì trạng thái thu hồi trong Status List Cache trên RAM           |
   +-----------------------------------------------------------------------------------+
```

---

## 5. Quy Định Cửa Hàng & Tiêu Chuẩn Gating Cho Mini-App

1. **Bắt buộc Áp dụng SD-JWT Cho Thu Thập Dữ liệu Định Danh**:
   - Nghiêm cấm mini-app yêu cầu toàn bộ hồ sơ người dùng nếu chỉ cần xác minh một điều kiện cụ thể (ví dụ: cấm đòi hỏi ngày sinh đầy đủ khi chỉ cần kiểm tra độ tuổi trên 18; cấm đòi hỏi toàn bộ CCCD khi chỉ cần xác thực số định danh khách hàng).
   - Mọi mini-app yêu cầu thuộc tính định danh phải khai báo chi tiết trong tệp `miniapp.manifest.json` và giao diện Consent Sheet của Super-App sẽ hiển thị công tắc cho phép người dùng tùy chọn tiết lộ.
2. **Quy định Về Chứng minh Quyền sở hữu Khóa (PoP)**:
   - Các chứng chỉ số có giá trị pháp lý hoặc tài chính cao phát hành qua OID4VCI bắt buộc phải được tạo cặp khóa mật mã trong Secure Enclave của thiết bị; Super-App từ chối lưu trữ các chứng chỉ không có ràng buộc khóa phần cứng.
3. **Tuân thủ Cơ chế Kiểm tra Thu hồi Bitstring**:
   - Khi kiểm tra tính hợp lệ của token hoặc giấy phép đối tác, mini-app bắt buộc phải tích hợp thư viện giải mã OAuth Status List chuẩn hóa do Super-App cung cấp.
   - Đối với các giao dịch tài chính trên 500.000 VNĐ, nếu danh sách trạng thái bị quá hạn (>15 phút) và không thể kết nối mạng tới CDN, mini-app bắt buộc phải áp dụng chính sách Fail-Closed (từ chối giao dịch).
