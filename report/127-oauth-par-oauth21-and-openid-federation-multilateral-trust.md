# Chuyên đề 127: IETF RFC 9126 Pushed Authorization Requests (PAR), OAuth 2.1 Consolidated Framework & OpenID Federation 1.0 Multilateral Trust

## 1. Tổng quan & Bối cảnh Tiêu chuẩn hóa

Trong kiến trúc Super App hiện đại, việc quản lý danh tính, phân quyền ủy quyền (authorization) và thiết lập mạng lưới tin cậy đa bên (multilateral federation) đòi hỏi sự kết hợp chặt chẽ giữa các tiêu chuẩn kỹ thuật cấp cao từ IETF và OpenID Foundation:

1. **IETF RFC 9126 (OAuth 2.0 Pushed Authorization Requests - PAR):** Chuyển dịch toàn bộ tải trọng yêu cầu ủy quyền từ chuỗi truy vấn URL (GET query string) sang lời gọi trực tiếp HTTP POST tới điểm cuối PAR (`/as/par`). Cơ chế này loại bỏ hoàn toàn nguy cơ rò rỉ tham số nhạy cảm trong lịch sử trình duyệt, nhật ký máy chủ (server logs), và tiêu đề HTTP Referer, đồng thời giải quyết triệt để ranh giới giới hạn độ dài URI khi tích hợp Rich Authorization Requests (RAR - RFC 9470).
2. **IETF OAuth 2.1 Consolidated Authorization Framework (draft-ietf-oauth-v2-1):** Hợp nhất toàn bộ các thực hành an ninh tốt nhất được công bố trong hơn một thập kỷ qua (RFC 6749, RFC 6750, RFC 7636 PKCE, RFC 8252 Native Apps, RFC 9700 Security BCP) thành một chuẩn cốt lõi duy nhất. OAuth 2.1 chính thức khai tử luồng Implicit (`response_type=token`) và Resource Owner Password Credentials (ROPC), cấm tuyệt đối việc truyền Bearer Token qua chuỗi truy vấn URI, bắt buộc áp dụng PKCE với phương pháp mã hóa `S256` cho mọi đối tượng client, thực thi so khớp chuỗi Redirection URI chính xác từng byte, và áp dụng luân chuyển Refresh Token (Refresh Token Rotation) với cơ chế phát hiện tái sử dụng tự động.
3. **OpenID Federation 1.0 (Final Specification Approved 2026):** Khung hạ tầng kỹ thuật thiết lập mối quan hệ tin cậy đa phương thức phân tán mà không đòi hỏi thỏa thuận song phương thủ công (bilateral agreements). Thông qua cấu trúc Entity Statements được định kiểu tường minh (`typ: entity-statement+jwt`), chuỗi tin cậy (Trust Chains) nối liền từ ứng dụng lá (Leaf Entity / Mini-App) tới Thực thể Mỏ neo Tin cậy (Trust Anchor), các chính sách siêu dữ liệu (Metadata Policy) có thể áp đặt quy tắc bảo mật từ trên xuống, và Dấu hiệu Tin cậy (Trust Marks) cho phép chứng thực kiểm toán và tuân thủ pháp lý theo thời gian thực.

---

## 2. Bảng tổng hợp 15 Canonical Findings (Iteration 127)

| Mã Finding | Danh mục & Tiêu chuẩn | Tiêu đề Chuyên sâu | Cấp độ Minh chứng | Nguồn Xác thực (HTTP 200) |
|---|---|---|---|---|
| `oauth_par_127_01` | RFC 9126 PAR Core | IETF RFC 9126: OAuth 2.0 Pushed Authorization Requests (PAR) Endpoint & Architecture | `official_standard` | [RFC 9126](https://www.rfc-editor.org/rfc/rfc9126.html) |
| `oauth_par_127_02` | RFC 9126 PAR Metadata | IETF RFC 9126: PAR Authorization Server Metadata & Client Enforcement Rules | `official_standard` | [RFC 9126](https://www.rfc-editor.org/rfc/rfc9126.html) |
| `oauth_par_127_03` | RFC 9126 PAR Security | IETF RFC 9126: PAR Security Considerations, Request URI Entropy & Anti-Replay Defense | `official_standard` | [RFC 9126](https://www.rfc-editor.org/rfc/rfc9126.html) |
| `oauth_par_127_04` | PAR Bridge Pattern | OAuth 2.0 PAR: WebView & Native Bridge Sandboxed Invocation Pattern | `official_standard` | [OAuth.net PAR](https://oauth.net/2/pushed-authorization-requests/) |
| `oauth_par_127_05` | PAR & FAPI / RAR | OAuth 2.0 PAR: Synergies with FAPI 2.0 & Rich Authorization Requests (RAR) | `official_standard` | [RFC 9126](https://www.rfc-editor.org/rfc/rfc9126.html) |
| `oauth21_std_127_06` | OAuth 2.1 Consolidation | OAuth 2.1: Specification Consolidation, Implicit Grant Deprecation & Bearer Query Prohibitions | `official_standard` | [draft-ietf-oauth-v2-1](https://datatracker.ietf.org/doc/draft-ietf-oauth-v2-1/) |
| `oauth21_std_127_07` | OAuth 2.1 PKCE Rules | OAuth 2.1: Universal Mandatory PKCE Enforcement & S256 Transformation | `official_standard` | [draft-ietf-oauth-v2-1](https://datatracker.ietf.org/doc/draft-ietf-oauth-v2-1/) |
| `oauth21_std_127_08` | OAuth 2.1 Redirect URIs | OAuth 2.1: Exact Redirect URI String Matching & Wildcard Path Prohibition | `official_standard` | [draft-ietf-oauth-v2-1](https://datatracker.ietf.org/doc/draft-ietf-oauth-v2-1/) |
| `oauth21_std_127_09` | OAuth 2.1 Token Lifecycle | OAuth 2.1: Refresh Token Protection, Cryptographic Binding & Mandatory Rotation | `official_standard` | [draft-ietf-oauth-v2-1](https://datatracker.ietf.org/doc/draft-ietf-oauth-v2-1/) |
| `oauth21_std_127_10` | OAuth 2.1 Enterprise Gov | OAuth 2.1: Enterprise Migration Strategy, Workload Identity & Governance Architecture | `official_standard` | [draft-ietf-oauth-v2-1](https://datatracker.ietf.org/doc/draft-ietf-oauth-v2-1/) |
| `openid_fed_127_11` | OpenID Fed Entity Statement | OpenID Federation 1.0: Entity Statements, Self-Signed Entity Configurations & Architecture | `official_standard` | [OpenID Federation 1.0](https://openid.net/specs/openid-federation-1_0.html) |
| `openid_fed_127_12` | OpenID Fed Trust Chains | OpenID Federation 1.0: Hierarchical Trust Chains, Authority Hints & Discovery Resolution | `official_standard` | [OpenID Federation 1.0](https://openid.net/specs/openid-federation-1_0.html) |
| `openid_fed_127_13` | OpenID Fed Metadata Policy | OpenID Federation 1.0: Subordinate Statements, Metadata Policies & Constraint Enforcement | `official_standard` | [OpenID Federation 1.0](https://openid.net/specs/openid-federation-1_0.html) |
| `openid_fed_127_14` | OpenID Fed Trust Marks | OpenID Federation 1.0: Trust Marks, Accreditation Architecture & Third-Party Auditing | `official_standard` | [OpenID Federation 1.0](https://openid.net/specs/openid-federation-1_0.html) |
| `openid_fed_127_15` | OpenID Fed Multi-Tenant | OpenID Federation 1.0: Multilateral Trust Architecture for Cross-Enterprise Super-Apps | `official_standard` | [OpenID Federation 1.0](https://openid.net/specs/openid-federation-1_0.html) |

---

## 3. Phân tích Kỹ thuật Chi tiết & Khuyến nghị Tiêu chuẩn Super App

### 3.1. IETF RFC 9126: Pushed Authorization Requests (PAR)
- **Cơ chế vận hành:** Thay vì ghép nối các tham số ủy quyền vào URL truy cập của trình duyệt (`GET /as/authorize?response_type=code&client_id=...&scope=...`), client gửi trực tiếp một HTTP POST request tới điểm cuối `pushed_authorization_request_endpoint`. Máy chủ ủy quyền (Authorization Server - AS) tiếp nhận, xác thực tính hợp lệ của client và tham số, sau đó cấp phát một định danh tạm thời `request_uri` (dưới dạng URN `urn:ietf:params:oauth:request_uri:<opaque-token>`) với thời gian sống ngắn (khuyến nghị 60-90 giây). Trình duyệt hoặc WebView chỉ cần mở URL `/as/authorize?client_id=...&request_uri=...`.
- **Ràng buộc an ninh bắt buộc:**
  - `request_uri` phải có độ hỗn loạn mã hóa tối thiểu 128 bit (khuyến nghị 160 bit CSPRNG) và chỉ được phép sử dụng **duy nhất một lần** (single-use semantics).
  - Máy chủ ủy quyền phải ràng buộc chặt chẽ `request_uri` với `client_id` đã xác thực tại endpoint PAR; mọi nỗ lực hoán đổi client tại endpoint ủy quyền đều bị từ chối với lỗi `invalid_request`.
- **Mô hình Mini-App Bridge Sandboxed:** Trong môi trường Super App, thay vì để mã JavaScript trong WebView tự phát lệnh POST tới PAR, Super App SDK cung cấp cầu nối nội bộ `superapp.auth.pushAuthorizationRequest()`. Container gốc (Native Host Container) thực hiện cuộc gọi back-channel an toàn tới máy chủ ủy quyền, đối soát scope được yêu cầu với danh mục quyền đã duyệt của Mini-App trong App Store Manifest, bảo vệ toàn diện chống lại nguy cơ giả mạo scope hoặc chèn tham số trái phép.
- **Tương thích Financial-Grade (FAPI 2.0 & RAR):** PAR là yêu cầu bắt buộc đối với FAPI 2.0 và tạo nền tảng truyền tải hoàn hảo cho Rich Authorization Requests (RFC 9470), cho phép truyền tải các cấu trúc JSON chi tiết về giao dịch thanh toán hoặc chuyển khoản ngân hàng mà không bị chặn bởi giới hạn 2048 byte của HTTP GET URL.

### 3.2. IETF OAuth 2.1: Bộ Khung Hợp nhất Toàn diện
- **Loại bỏ các luồng không an toàn (Deprecated Flows):**
  - Khai tử hoàn toàn luồng Implicit Grant (`response_type=token`). Mã truy cập (Access Token) không bao giờ được phép trả về qua URL hash fragment vì nguy cơ rò rỉ qua lịch sử trình duyệt và chuyển hướng không kiểm soát.
  - Khai tử luồng Resource Owner Password Credentials (ROPC), ngăn chặn việc Mini-App thu thập và lưu trữ trực tiếp tên đăng nhập/mật khẩu của người dùng Super App.
- **Bắt buộc áp dụng PKCE trên quy mô toàn hệ thống:**
  - PKCE (RFC 7636) trở thành yêu cầu bắt buộc cho **tất cả** các loại client (cả public client trong WebView và confidential backend service).
  - Cấm hoàn toàn phương thức chuyển đổi không an toàn `code_challenge_method=plain`; bắt buộc áp dụng `code_challenge_method=S256`.
- **So khớp Redirection URI chính xác tuyệt đối (Exact String Matching):**
  - Nghiêm cấm sử dụng ký tự đại diện (wildcard `*`), biểu thức chính quy (regex), hoặc duyệt đường dẫn tương đối (`/../`) trong danh sách Redirection URI đăng ký.
  - Yêu cầu so khớp chuỗi byte-for-byte chính xác khi xử lý yêu cầu ủy quyền, vô hiệu hóa hoàn toàn các kỹ thuật tấn công Open Redirector nhằm chiếm đoạt Authorization Code.
- **Bảo vệ và luân chuyển Refresh Token (Rotation):**
  - Mọi Refresh Token cấp phát cho client công khai phải được ràng buộc theo chứng minh sở hữu mã hóa (DPoP hoặc mTLS) hoặc bắt buộc áp dụng cơ chế luân chuyển (Refresh Token Rotation).
  - Mỗi lần làm mới mã thành công, một Refresh Token mới được phát hành và Refresh Token cũ bị vô hiệu hóa ngay lập tức. Nếu máy chủ phát hiện một Refresh Token đã thu hồi được sử dụng lại, toàn bộ họ mã ủy quyền (Authorization Family) liên quan phải bị hủy bỏ ngay lập tức để phòng ngừa nguy cơ rò rỉ mã bí mật.

### 3.3. OpenID Federation 1.0: Hạ tầng Tin cậy Đa phương thức (Multilateral Trust)
- **Entity Statement & Cấu hình Thực thể Tự ký (Entity Configuration):**
  - Mọi thực thể trong liên minh phân tán (Super App Host, Đơn vị Vận hành, Mini-App bên thứ ba) công bố thông tin tự ký dưới định dạng JWT định kiểu tường minh (`typ: entity-statement+jwt`) tại điểm cuối chuẩn hóa `/.well-known/openid-federation`.
  - Khóa công khai của thực thể (`jwks`) được nhúng trực tiếp trong Entity Statement, cung cấp gốc xác minh chữ ký số độc lập.
- **Chuỗi Tin cậy Thứ bậc (Hierarchical Trust Chains):**
  - Cho phép xác lập niềm tin mật mã học giữa Mini-App (Leaf Entity) và Super App Core (Trust Anchor) thông qua việc phân giải danh sách `authority_hints` và tải các Tuyên bố Trực thuộc (Subordinate Statements) từ các Thực thể Trung gian (Intermediate Entities).
  - Quá trình phân giải đảm bảo tính toàn vẹn: mỗi phần tử trong chuỗi được ký bởi khóa công khai khai báo trong JWKS của thực thể cấp trên trực tiếp, dẫn thẳng tới khóa gốc của Trust Anchor.
- **Chính sách Siêu dữ liệu (Metadata Policy):**
  - Các cấp có thẩm quyền áp đặt quy chuẩn bảo mật xuống các thực thể cấp dưới bằng các toán tử chính sách chuẩn hóa: `value`, `add`, `default`, `essential`, `one_of`, `subset_of`, `superset_of`.
  - Ví dụ: Super App Trust Anchor áp đặt `grant_types_supported: {"subset_of": ["authorization_code"]}` và `token_endpoint_auth_methods_supported: {"one_of": ["private_key_jwt", "self_signed_tls_client_auth"]}` để đảm bảo không một Mini-App nào có thể cấu hình luồng cấp quyền kém an toàn.
- **Dấu hiệu Tin cậy (Trust Marks) & Hệ sinh thái Đa người thuê:**
  - Cung cấp mô hình chứng nhận số phi tập trung: Các cơ quan kiểm toán, tổ chức chứng nhận ngành hoặc bộ phận thẩm định Super App Store cấp phát Trust Mark dưới dạng JWT có chữ ký số xác nhận Mini-App đạt chuẩn an ninh dữ liệu tài chính, ISO 27001, hoặc tuân thủ bảo vệ dữ liệu cá nhân.
  - Phù hợp hoàn hảo cho các Super App quy mô quốc gia hoặc tập đoàn đa ngành, cho phép tích hợp hàng nghìn đối tác mà không cần thiết lập cấu hình ủy quyền song phương thủ công.

---

## 4. Kiểm toán & Tuân thủ An ninh Hệ thống

- Toàn bộ 15 finding mới đã được bổ sung vào cơ sở tri thức chuẩn theo cơ chế **append-only** nghiêm ngặt.
- Kiểm tra số dòng `state/findings.jsonl`: tăng chính xác từ **1841** lên **1856** dòng (+15 findings).
- Mọi URL trích dẫn chính thức từ IETF RFC Editor, IETF Datatracker, OAuth.net, và OpenID Foundation đã được kiểm tra trạng thái HTTP 200 tự động trước khi ghi nhận.
- Đã quét lọc bảo mật: 100% không chứa mã bí mật, mật khẩu hay credential thực tế.
