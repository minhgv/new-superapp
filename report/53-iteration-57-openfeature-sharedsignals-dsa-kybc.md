# Chuyên đề 53 (Iteration 57): CNCF OpenFeature, OpenID Shared Signals (CAEP/RISC) & DSA Trader Traceability (KYBC)

## 1. Tổng quan & Phạm vi nghiên cứu Iteration 57

Trong khuôn khổ phát triển tiêu chuẩn Mini App Store cho Super App (tham chiếu POC Viettel / Alibaba Cloud WindVane Mini App), Iteration 57 tập trung giải quyết ba trụ cột kiến trúc và pháp lý sống còn của hệ sinh thái siêu ứng dụng hiện đại:
1. **CNCF OpenFeature Specification & Dynamic Runtime Feature Flag Sandboxing**: Chuẩn hóa API phân giải tính năng động, cấu trúc đánh giá ngữ cảnh phân cấp bảo vệ quyền riêng tư, vòng đời nhà cung cấp (Providers) kèm giao thức đánh giá biên OFREP, đường ống can thiệp 4 giai đoạn Hooks và cơ chế theo dõi sự kiện phục vụ tự động hóa rollback canary release.
2. **OpenID Shared Signals Framework (SSF 1.0), CAEP 1.0 & RISC 1.0 Zero-Trust Security Event Propagation**: Ứng dụng Security Event Tokens (RFC 8417) để phát tín hiệu bảo mật thời gian thực giữa Super-App Identity Provider và các mini-app; thu hồi phiên tức thì (session revocation), giám sát trạng thái thiết bị rủi ro (root/jailbreak) và phối hợp phòng chống chiếm đoạt tài khoản mà không cần chờ hết hạn token OAuth; triển khai giao thức truyền tải đẩy HTTP (RFC 8935) và kéo dự phòng cho mạng doanh nghiệp cô lập (RFC 8936).
3. **EU Digital Services Act (DSA - Quy định EU 2022/2065) Điều 30 & 31: Trader Traceability & KYBC Governance**: Nghĩa vụ pháp lý xác minh danh tính doanh nghiệp thương mại (Know Your Business Customer - KYBC) trước khi cấp quyền phân phối mini-app; quy trình kiểm tra chéo cơ sở dữ liệu quốc gia (VIES, đăng ký kinh doanh); thời hạn khắc phục và đình chỉ bắt buộc theo Điều 30(3); thiết kế giao diện minh bạch danh tính nhà phát triển theo Điều 31; đối sánh thực tiễn triển khai Apple App Store (D-U-N-S) và Google Play Console (kiểm tra ngẫu nhiên định kỳ Điều 30(5)).

---

## 2. CNCF OpenFeature & Dynamic Runtime Feature Gating

### 2.1. Đánh giá tính năng kiểu mạnh và cấu trúc phản hồi chi tiết (Flag Evaluation API)
- **Tiêu chuẩn**: *OpenFeature Specification v1.0 Section 1 (Flag Evaluation API) & Types*.
- **Cơ chế cốt lõi**:
  - OpenFeature cung cấp giao diện lập trình trung lập với nhà cung cấp (vendor-neutral), phân tách logic mini-app khỏi hạ tầng cờ tính năng bên dưới.
  - Hỗ trợ các hàm phân giải kiểu mạnh: `getBooleanValue`, `getStringValue`, `getNumberValue`, `getObjectValue` cùng các phương thức trả về chi tiết `get*Details`.
  - Cấu trúc phản hồi mở rộng bao gồm: `value` (giá trị cờ), `variant` (nhãn phiên bản ngữ nghĩa, vd: `canary-v2`, `control`), `reason` (nguyên nhân kích hoạt: `TARGETING_MATCH`, `DEFAULT`, `STATIC`, `SPLIT`, `CACHED`, `ERROR`) và `errorCode` (`FLAG_NOT_FOUND`, `TYPE_MISMATCH`, `PARSE_ERROR`, `TARGETING_KEY_MISSING`).
- **Yêu cầu Super-App**:
  - Vỏ bọc mini-app (host container) tiêm một OpenFeature client độc lập vào từng ngữ cảnh sandbox.
  - Toàn bộ các JSAPI hoặc cầu nối native thử nghiệm (experimental bridges) như WebTransport, sinh trắc học nâng cao, truy cập thiết bị ngoại vi bắt buộc phải được bọc sau cờ tính năng OpenFeature.
  - Bảng điều khiển vận hành store cung cấp kill-switch khẩn cấp có khả năng vô hiệu hóa tính năng mini-app bị lỗi hoặc khai thác bảo mật trong thời gian dưới 1 giây mà không cần biên dịch hay tải lại toàn bộ gói ứng dụng.

### 2.2. Ngữ cảnh đánh giá và thứ tự hợp nhất bảo mật (EvaluationContext)
- **Tiêu chuẩn**: *OpenFeature Specification v1.0 Section 3 (Evaluation Context)*.
- **Cơ chế cốt lõi**:
  - `EvaluationContext` chứa các thuộc tính bổ trợ phục vụ thuật toán phân chia tỷ lệ (fractional split) và nhắm mục tiêu (cohort targeting).
  - Quy tắc ưu tiên hợp nhất 5 cấp bắt buộc: **Global API** (cấp container, ưu tiên thấp nhất) -> **Transaction** -> **Client** (cấp mini-app cụ thể) -> **Invocation** (cấp lời gọi hàm) -> **Before Hooks** (ưu tiên cao nhất; ghi đè mọi thuộc tính trùng tên).
- **Yêu cầu Super-App**:
  - Ngăn chặn triệt để việc rò rỉ PII (tên, số điện thoại, định danh người dùng thật) từ host sang mini-app: trường `targetingKey` bắt buộc phải là mã giả danh theo cặp phù du (Pairwise Pseudonymous Identifier - PPID).
  - Ngữ cảnh toàn cục chỉ chia sẻ các thông tin môi trường không nhạy cảm (`os_version`, `app_channel`, `locale`, `network_type`).
  - Container host khóa cứng các thuộc tính tuân thủ pháp lý và khu vực tài phán, không cho phép mini-app ghi đè ở các tầng ngữ cảnh con.

### 2.3. Vòng đời Provider & Giao thức đánh giá từ xa OFREP (OpenFeature Remote Evaluation Protocol)
- **Tiêu chuẩn**: *OpenFeature Specification v1.0 Section 2 & Appendix C (OFREP)*.
- **Cơ chế cốt lõi**:
  - Provider tuân thủ máy trạng thái nghiêm ngặt: `NOT_READY` (khởi tạo), `READY` (sẵn sàng, dữ liệu tươi), `STALE` (mất mạng hoặc cache quá hạn), `ERROR` (lỗi tạm thời), `FATAL` (lỗi xác thực/cấu hình nghiêm trọng).
  - Chuẩn hóa giao thức OFREP trên HTTP (`/ofrep/v1/evaluate/flags`) cho phép container biên (Edge) hoặc máy chủ vùng phân giải hàng trăm quy tắc trong một lượt gọi duy nhất kèm thẻ ETag tối ưu băng thông.
- **Yêu cầu Super-App**:
  - Container client triển khai In-Memory Provider backed bởi bộ nhớ cache cục bộ (SQLite Wasm hoặc OPFS), đảm bảo thời gian phân giải cờ tức thì dưới 1ms mà không chặn UI thread.
  - Đồng bộ cập nhật cấu hình theo thời gian thực qua kênh WebSocket hoặc Server-Sent Events (SSE).
  - Khi ngoại tuyến (offline) hoặc khởi động lạnh, Provider duy trì trạng thái `STALE`/`READY` với giá trị mặc định đã chứng nhận an toàn, không làm gián đoạn trải nghiệm người dùng.

### 2.4. Đường ống can thiệp vòng đời 4 giai đoạn (Hooks Pipeline)
- **Tiêu chuẩn**: *OpenFeature Specification v1.0 Section 4 (Hooks)*.
- **Cơ chế cốt lõi**:
  - Mô hình vòng lặp đón đầu (onion model) can thiệp 4 giai đoạn: `before` (chạy trước khi phân giải, kiểm tra ngữ cảnh), `after` (chạy sau khi thành công, thu thập telemetry), `error` (xử lý ngoại lệ, kích hoạt fallback), `finally` (dọn dẹp tài nguyên).
  - Giai đoạn `before` thực thi theo thứ tự đăng ký; `after`, `error`, `finally` thực thi theo thứ tự đảo ngược.
- **Yêu cầu Super-App**:
  - Triển khai `SecurityAuditHook` tự động quét và loại bỏ các token hoặc secret vô tình lọt vào ngữ cảnh.
  - Triển khai `MetricsTelemetryHook` ghi nhận độ trễ phân giải, tỷ lệ cache hit/miss vào OpenTelemetry.
  - Triển khai `ResilienceFallbackHook` đảm bảo khi cổng cờ tính năng sập, mini-app lập tức chuyển sang chế độ suy giảm nhẹ (graceful degradation) thay vì treo ứng dụng.

### 2.5. Sự kiện và liên kết đo lường Canary Rollout (Events & Tracking)
- **Tiêu chuẩn**: *OpenFeature Specification v1.0 Section 5 (Events) & Section 6 (Tracking)*.
- **Cơ chế cốt lõi**:
  - Giao diện `client.track(trackingEventName, evaluationContext, trackingEventDetails)` thiết lập cầu nối đo lường tiêu chuẩn giữa biến thể cờ được bật và các sự kiện độ tin cậy/nghiệp vụ (crash, tỷ lệ thanh toán thành công, độ trễ API).
- **Yêu cầu Super-App**:
  - Toàn bộ các phiên bản mini-app phát hành theo tiến trình canary (5% -> 25% -> 100%) bắt buộc phải gắn kết với OpenFeature tracking.
  - Nếu nhóm canary xuất hiện tỷ lệ crash tăng vượt ngưỡng (delta > 0.1%) hoặc độ trễ API tăng quá 200ms, hệ thống CI/CD tự động kích hoạt cờ lùi về biến thể an toàn (`control`) trong vòng 30 giây.

---

## 3. OpenID Shared Signals Framework (SSF 1.0), CAEP & RISC Zero-Trust Architecture

### 3.1. Cấu trúc Security Event Token (RFC 8417) và SSF 1.0
- **Tiêu chuẩn**: *OpenID Shared Signals Framework 1.0 & IETF RFC 8417*.
- **Cơ chế cốt lõi**:
  - SSF 1.0 định hình việc trao đổi tín hiệu rủi ro bất đồng bộ giữa các hệ thống tin cậy thông qua Security Event Token (SET) - một dạng JWT chuyên biệt mang trường claim `events`.
  - Cung cấp cơ chế tự động khám phá cấu hình máy phát (Transmitter Discovery tại `/.well-known/sse-configuration`), API quản lý luồng sự kiện (Stream Management API) và định danh đối tượng tiêu chuẩn (`iss_sub`, `email`, `phone`, `jwt_id`).
- **Yêu cầu Super-App**:
  - Super-App Identity Provider đóng vai trò Máy phát SSF (Transmitter), phát tín hiệu đến tất cả các backend mini-app đang hoạt động.
  - Mọi SET bắt buộc phải được ký số bất đối xứng (JWS RFC 7515) bằng khóa được công bố trên endpoint JWKS của siêu ứng dụng.

### 3.2. Đánh giá truy cập liên tục CAEP 1.0 (Continuous Access Evaluation Profile)
- **Tiêu chuẩn**: *OpenID Continuous Access Evaluation Profile (CAEP) Specification 1.0*.
- **Cơ chế cốt lõi**:
  - Thu hẹp khoảng trống bảo mật giữa thời gian sống tĩnh của OAuth token (thường từ 15-60 phút) và rủi ro thời gian thực.
  - Định nghĩa các loại sự kiện chuẩn hóa:
    1. `https://schemas.openid.net/secevent/caep/event-type/session-revoked`: Hủy bỏ tức thì phiên đăng nhập của người dùng.
    2. `https://schemas.openid.net/secevent/caep/event-type/device-compliance-change`: Cảnh báo khi thiết bị chuyển sang trạng thái không an toàn (phát hiện root, bẻ khóa bootloader, gỡ mã hóa phần cứng).
    3. `https://schemas.openid.net/secevent/caep/event-type/assurance-level-change`: Báo hiệu nâng cấp hoặc hạ cấp cấp độ xác thực.
    4. `https://schemas.openid.net/secevent/caep/event-type/credential-change`: Thông báo đổi mật khẩu hoặc xóa passkey.
- **Yêu cầu Super-App**:
  - Khi nhận tín hiệu `session-revoked` hoặc `device-compliance-change: non-compliant`, native container của siêu ứng dụng lập tức đóng băng WebView, xóa token lưu tạm và thu hồi quyền truy cập của mini-app trong vòng dưới 500ms.
  - Backend mini-app đăng ký webhook để vô hiệu hóa ngay lập tức refresh token tương ứng.

### 3.3. Phối hợp ngăn ngừa tấn công chiếm đoạt tài khoản RISC 1.0 (Risk Incident Sharing)
- **Tiêu chuẩn**: *OpenID Risk Incident Sharing and Coordination (RISC) Profile 1.0*.
- **Cơ chế cốt lõi**:
  - Tiêu chuẩn hóa các tín hiệu rủi ro liên tổ chức: `account-credential-change-required` (buộc đổi mật khẩu do lộ lọt cơ sở dữ liệu), `account-purged` (xóa vĩnh viễn tài khoản), `account-disabled` (tạm khóa tài khoản do hành vi gian lận).
- **Yêu cầu Super-App**:
  - Khi Trung tâm Giám sát An ninh (SOC) của siêu ứng dụng phát hiện người dùng bị tấn công credential stuffing, hệ thống phát tín hiệu RISC `account-credential-change-required` đến toàn bộ mini-app tài chính/thương mại liên kết.
  - Mini-app bắt buộc phải kích hoạt xác thực sinh trắc học hoặc OTP bổ sung trước khi cho phép thực hiện các giao dịch chuyển tiền hoặc đổi thông tin nhạy cảm.

### 3.4. Phương thức truyền tải SET: Đẩy HTTP (RFC 8935) vs Kéo định kỳ (RFC 8936)
- **Tiêu chuẩn**: *IETF RFC 8935 & IETF RFC 8936*.
- **So sánh kiến trúc**:
  | Tiêu chí | RFC 8935 Push Delivery | RFC 8936 Poll Delivery |
  | :--- | :--- | :--- |
  | **Cơ chế** | Transmitter gửi HTTP POST đến Receiver endpoint | Receiver chủ động HTTP POST kéo sự kiện (Long-Polling) |
  | **Môi trường phù hợp** | Cloud-native mini-apps, máy chủ có IP public | Doanh nghiệp nội bộ, Core Banking phía sau Firewall/NAT |
  | **Độ trễ** | Thời gian thực (<500ms) | Phụ thuộc chu kỳ poll (1s - 30s) |
  | **Xác thực** | mTLS hoặc OAuth Bearer Token | Mutual TLS hoặc Client Credentials |
  | **Cơ chế phản hồi** | HTTP 202 Accepted hoặc lỗi JSON chuẩn hóa | Xác nhận theo lô qua mảng `ack: ["jti-1", "jti-2"]` |
- **Yêu cầu Super-App**:
  - Hỗ trợ song song cả hai giao thức: RFC 8935 cho các đối tác SaaS công cộng và RFC 8936 cho các mini-app doanh nghiệp/ngân hàng chạy trong mạng nội bộ.

---

## 4. Tuân thủ Quy định EU Digital Services Act (DSA): Minh bạch Trader & KYBC

### 4.1. Nghĩa vụ xác minh danh tính nhà phát triển thương mại (DSA Điều 30)
- **Tiêu chuẩn**: *Regulation (EU) 2022/2065 (Digital Services Act), Article 30 (Traceability of traders)*.
- **Nghĩa vụ pháp lý**:
  - Nền tảng trung gian trực tuyến chỉ được phép cấp quyền cung cấp dịch vụ/sản phẩm cho người tiêu dùng sau khi đã thu thập và xác minh thông tin KYBC:
    1. Tên pháp lý, địa chỉ trụ sở, số điện thoại và email chính thức.
    2. Bản sao giấy tờ tùy thân hợp pháp hoặc định danh điện tử doanh nghiệp.
    3. Tài khoản thanh toán (IBAN/Bank Account).
    4. Cơ quan đăng ký kinh doanh và mã số đăng ký kinh doanh.
    5. Cam kết tự chứng nhận tuân thủ pháp luật và an toàn sản phẩm.
  - Nền tảng có nghĩa vụ "nỗ lực cao nhất" (best efforts) kiểm tra chéo tính xác thực qua các cơ sở dữ liệu công khai (VIES, cổng đăng ký doanh nghiệp quốc gia).

### 4.2. Quy trình cảnh báo, thời hạn khắc phục và đình chỉ bắt buộc (DSA Điều 30(3))
- **Tiêu chuẩn**: *Regulation (EU) 2022/2065 (Digital Services Act), Article 30(3) & 30(4)*.
- **Quy trình xử lý**:
  - Khi có căn cứ cho thấy thông tin nhà phát triển không chính xác, hết hạn hoặc bị làm giả, nền tảng phải gửi thông báo yêu cầu khắc phục ngay lập tức hoặc trong thời hạn ấn định (thường từ 14 đến 30 ngày).
  - Nếu nhà phát triển không hoàn thành việc bổ sung/hiệu chỉnh thông tin trong thời hạn, nền tảng **bắt buộc phải đình chỉ** cung cấp dịch vụ của nhà phát triển đó trên store.
  - Dữ liệu định danh trader phải được lưu trữ an toàn trong suốt thời gian hợp đồng và duy trì thêm đúng 6 tháng sau khi chấm dứt hợp đồng để phục vụ thanh tra.

### 4.3. Thiết kế giao diện tuân thủ quy chuẩn (DSA Điều 31: Compliance by Design)
- **Tiêu chuẩn**: *Regulation (EU) 2022/2065 (Digital Services Act), Article 31*.
- **Quy tắc thiết kế UI**:
  - Giao diện cửa hàng (storefront) phải được thiết kế để hiển thị rõ ràng, dễ tiếp cận các thông tin định danh của trader: tên công ty, địa chỉ, số điện thoại, email hỗ trợ, mã số thuế và quyền hủy hợp đồng/hoàn tiền của người tiêu dùng trước khi người dùng thực hiện giao dịch hoặc tải mini-app.

### 4.4. Đối sánh thực tế: Mô hình triển khai của Apple App Store & Google Play
- **Apple App Store**:
  - Yêu cầu xác thực tài khoản tổ chức thông qua mã số D-U-N-S (Dun & Bradstreet) toàn cầu.
  - Xác thực hai bước qua SMS và email công khai.
  - Khai báo tình trạng Trader/Non-Trader; tự động ẩn ứng dụng tại 27 quốc gia EU nếu không hoàn tất xác thực DSA trước hạn chót.
- **Google Play Console**:
  - Bắt buộc xác minh danh tính tổ chức (giấy phép kinh doanh, D-U-N-S, địa chỉ thực tế).
  - Thực thi Điều 30(5) DSA: triển khai hệ thống quét và kiểm tra ngẫu nhiên (random sampling) định kỳ tối thiểu 5% danh mục ứng dụng thương mại, đối soát với cổng cảnh báo an toàn người tiêu dùng (EU Safety Gate / RAPEX).

---

## 5. Ma trận Kiểm soát Kỹ thuật Mini App Store (Mục tiêu 57)

| Mã kiểm soát | Tên quy chuẩn kỹ thuật | Mức độ bắt buộc | Thành phần kiến trúc chịu trách nhiệm | Cơ chế xác minh tự động |
| :--- | :--- | :--- | :--- | :--- |
| **CTL-FEAT-01** | OpenFeature Typed Resolution | Mandatory (P0) | Mini-App Host SDK / OpenFeature Client | Unit test kiểm tra kiểu dữ liệu và fallback |
| **CTL-FEAT-02** | Privacy-Preserved EvaluationContext | Mandatory (P0) | Host Privacy Filter & Context Sanitizer | Quét tự động loại trừ PII; chỉ tiêm PPID |
| **CTL-FEAT-03** | In-Memory Provider & OFREP Sync | Highly Recommended (P1) | Host Local Cache (SQLite/OPFS) + Gateway | Benchmark thời gian phân giải < 1ms |
| **CTL-FEAT-04** | Automated Canary Rollback Triggers | Mandatory (P0) | Store CI/CD & OpenTelemetry Ingestion | Tự động hạ cờ khi delta crash > 0.1% |
| **CTL-SEC-01** | OpenID SSF & RFC 8417 SET Signing | Mandatory (P0) | Super-App Central IdP & JWKS Endpoint | Kiểm tra chữ ký JWS bất đối xứng hợp lệ |
| **CTL-SEC-02** | CAEP Instant Session Revocation | Mandatory (P0) | Mini-App Native Container & Storage Manager | Kiểm tra đóng băng WebView & xóa token < 500ms |
| **CTL-SEC-03** | RISC Account Takeover Coordination | Highly Recommended (P1) | Super-App SOC & Security Event Webhooks | Mô phỏng sự kiện ép xác thực sinh trắc học |
| **CTL-SEC-04** | RFC 8935/8936 Dual Event Transport | Mandatory (P0) | SSF Event Delivery Gateway | Hỗ trợ song song Push (mTLS) và Poll (queue) |
| **CTL-REG-01** | DSA Article 30 Mandatory KYBC | Mandatory (P0) | Developer Portal Onboarding Pipeline | Tích hợp API cơ sở dữ liệu doanh nghiệp |
| **CTL-REG-02** | DSA Article 30(3) Cure & Suspension | Mandatory (P0) | Catalog Lifecycle Compliance Engine | Bộ đếm tự động khóa app sau 14-30 ngày |
| **CTL-REG-03** | DSA Article 31 Storefront Disclosures | Mandatory (P0) | Storefront Mini-App PDP UI & Payment Sheet | Linter kiểm tra schema hiển thị thẻ Trader |
| **CTL-REG-04** | Continuous Random Audit Sampling | Highly Recommended (P1) | Store Security & Integrity Scanner | Quét ngẫu nhiên 5% danh mục theo tuần |

---

## 6. Kết luận & Định hướng Milestone tiếp theo

Iteration 57 đã bổ sung 15 phát hiện chuẩn hóa nguồn chính thức, nâng tổng số phát hiện của toàn bộ nghiên cứu lên **806 findings**. Nghiên cứu đã hoàn thiện trọn vẹn:
1. Cơ chế quản trị tính năng động và phát hành an toàn với CNCF OpenFeature.
2. Nền tảng chia sẻ sự kiện an ninh Zero-Trust liên tục qua OpenID SSF, CAEP và RISC.
3. Khung pháp lý và quy chuẩn kỹ thuật định danh nhà phát triển thương mại theo EU Digital Services Act Điều 30 & 31.

Vòng kế tiếp (Iteration 58) sẽ tiếp tục mở rộng sang các tiêu chuẩn mở về giao tiếp hạ tầng thanh toán thế hệ mới, chuẩn mã hóa tài liệu định danh di động (ISO/IEC 18013-7 over REST/OIDC) và quản trị vòng đời ứng dụng PWA nâng cao.
