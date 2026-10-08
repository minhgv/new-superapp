# Chuyên đề Iteration 42: Financial-Grade API Security, Client Privacy Signals, Content Indexing, Payment Handler & Multi-Screen Window Governance

Tài liệu này ghi nhận kết quả nghiên cứu chuẩn kỹ thuật chuyên sâu tại **Vòng 42 (Iteration 42)** của dự án Chuẩn hóa Mini App Store trên Super App (Deli Deep Research). Vòng nghiên cứu này tập trung vào 3 trụ cột kỹ thuật cốt lõi:
1. **Financial-Grade API Security & Cryptographic Token Governance**: Chuẩn bảo mật cấp tài chính/ngân hàng OpenID FAPI 2.0, mTLS token binding (RFC 8705), Token Exchange phân quyền (RFC 8693), mã hóa payload JWE (RFC 7516), và chuẩn token nhị phân cho IoT/Wearable (RFC 9200 ACE-OAuth / RFC 8392 CWT).
2. **Client Privacy Signals, Content Indexing & Media Transformations**: Quản trị privacy budget và ngăn chặn fingerprinting qua WICG User-Agent Client Hints, tích hợp feed nội dung offline qua WICG Content Indexing API, xử lý video/audio luồng trực tiếp không trễ qua W3C MediaStreamTrack Insertable Media (Breakout Box), phân tầng tài nguyên RAM qua W3C Device Memory, và trích xuất màu an toàn qua WICG EyeDropper API.
3. **Standardized Payment Handling, Multi-Screen Windowing & Worklet Rendering**: Chuẩn xử lý thanh toán nền tảng qua W3C Payment Handler API, điều phối hiển thị đa màn hình POS bán lẻ qua W3C Window Management API, trình chiếu không dây qua W3C Presentation API, quản trị phông chữ doanh nghiệp qua WICG Local Font Access API, và tăng tốc giao diện đồ họa không chiếm DOM qua W3C CSS Paint API Level 1 (PaintWorklet).

---

## 1. Financial-Grade API Security & Cryptographic Token Governance

### 1.1. OpenID FAPI 2.0 Security Profile & Attacker Model
- **Nguồn chuẩn**: OpenID Foundation FAPI 2.0 Security Profile (`https://openid.net/specs/fapi-2_0-security-profile.html`), RFC 9101 (JAR), RFC 9449 (DPoP).
- **Mã định danh finding**: `fapi_token_042_01`.
- **Cơ chế kỹ thuật**:
  - Thiết lập mô hình phòng thủ nâng cao (Attacker Model) cho các mini app tài chính, ngân hàng số và cổng dịch vụ công tích hợp trên Super App.
  - **Triệt tiêu Bearer Token**: Tuyệt đối cấm sử dụng bearer token truyền thống cho các giao dịch có độ rủi ro cao. Bắt buộc sender-constraining thông qua DPoP (RFC 9449) hoặc Mutual TLS (RFC 8705). Token bị lộ qua log mạng hoặc proxy trung gian không thể tái sử dụng từ bất kỳ IP/thiết bị nào khác.
  - **Bảo mật Request/Response**: Áp dụng PKCE (RFC 7636) với phương thức `S256` kết hợp JARM (JWT Secured Authorization Response Mode). Mọi tham số phản hồi ủy quyền đều được ký số và đóng gói trong JWT, loại bỏ hoàn toàn việc truyền lộ token/code trên query parameters hoặc URL fragments.
  - **Cơ chế Store Gate**: Mini app thuộc danh mục Fintech/Banking phải vượt qua bộ kiểm thử tự động OpenID FAPI Conformance Suite trước khi được cấp phép release lên store.

### 1.2. RFC 8705 mTLS Client Authentication & Certificate-Bound Tokens
- **Nguồn chuẩn**: IETF RFC 8705 (`https://datatracker.ietf.org/doc/html/rfc8705`), RFC 6749, RFC 8446 (TLS 1.3).
- **Mã định danh finding**: `fapi_token_042_02`.
- **Cơ chế kỹ thuật**:
  - Định nghĩa cơ chế xác thực client mTLS (`tls_client_auth` và `self_signed_tls_client_auth`) giữa backend của mini app đối tác doanh nghiệp và Super App API Gateway.
  - **Certificate-Bound Access Tokens**: Gateway nhúng mã băm SHA-256 của chứng chỉ X.509 (`cnf.x5t#S256`) vào payload của access token.
  - Khi mini app gọi các API core banking hoặc thanh toán, Resource Server đối soát trực tiếp thumbprint chứng chỉ trong handshake TLS với giá trị `cnf` trong token. Không cần thêm roundtrip truy vấn introspection server, đảm bảo hiệu năng xử lý cao và miễn nhiễm với tấn công trộm token trong bộ nhớ WebView.

### 1.3. RFC 8693 OAuth 2.0 Token Exchange & Authority Downscoping
- **Nguồn chuẩn**: IETF RFC 8693 (`https://datatracker.ietf.org/doc/html/rfc8693`), RFC 7519 (JWT).
- **Mã định danh finding**: `fapi_token_042_03`.
- **Cơ chế kỹ thuật**:
  - Chuẩn hóa luồng trao đổi token thông qua grant type `urn:ietf:params:oauth:grant-type:token-exchange`.
  - Phân tách tường minh giữa **Subject Token** (đại diện cho người dùng đăng nhập trên Super App) và **Actor Token** (đại diện cho mini app được ủy quyền).
  - **Thu hẹp quyền hạn (Downscoping)**: Super App Identity Provider chỉ cấp phát token có thời hạn ngắn (TTL 5-15 phút), phạm vi scope tối thiểu và audience (`aud`) đích danh cho backend của mini app đó. Nếu backend đối tác bị xâm nhập, kẻ tấn công không thể leo thang đặc quyền sang các dịch vụ khác của Super App.

### 1.4. RFC 7516 JSON Web Encryption (JWE) Payload Confidentiality
- **Nguồn chuẩn**: IETF RFC 7516 (`https://datatracker.ietf.org/doc/html/rfc7516`), RFC 7518 (JWA), NIST SP 800-38D.
- **Mã định danh finding**: `fapi_token_042_04`.
- **Cơ chế kỹ thuật**:
  - Cung cấp cơ chế mã hóa xác thực mức ứng dụng (Application-Layer Authenticated Encryption) cho các luồng dữ liệu nhạy cảm (thông tin CCCD/hộ chiếu, số thẻ, hồ sơ bệnh án) truyền qua cầu nối IPC giữa host native và WebView container.
  - Sử dụng cấu trúc JWE Compact Serialization 5 phần: `Protected Header . Encrypted Key . IV . Ciphertext . Authentication Tag`.
  - Áp dụng thuật toán bọc khóa bất đối xứng `ECDH-ES` (đường cong P-256/P-384) và mã hóa đối xứng `A256GCM`, đảm bảo tính bảo mật và toàn vẹn dữ liệu ngay cả khi host bị can thiệp bởi phần mềm gián điệp hoặc OS telemetry logging.

### 1.5. RFC 9200 ACE-OAuth & RFC 8392 CBOR Web Tokens (CWT) cho IoT/Mobility
- **Nguồn chuẩn**: IETF RFC 9200 (`https://datatracker.ietf.org/doc/html/rfc9200`), RFC 8392 (CWT), RFC 9052 (COSE), RFC 8949 (CBOR).
- **Mã định danh finding**: `fapi_token_042_05`.
- **Cơ chế kỹ thuật**:
  - Giải quyết bài toán tiêu hao băng thông và bộ nhớ khi mini app mở rộng sang các thiết bị IoT, thiết bị đeo thông minh (wearables), và màn hình điều khiển ô tô (Automotive In-Vehicle Infotainment).
  - Chuyển đổi toàn bộ kiến trúc OAuth 2.0 sang định dạng nhị phân CBOR/COSE, giảm 70-80% kích thước gói tin so với JWT/JSON truyền thống.
  - Hỗ trợ xác thực phần cứng ngoại tuyến thông qua khóa đối xứng hoặc cặp khóa bất đối xứng Ed25519 được lưu trong chip bảo mật của thiết bị ngoại vi, cho phép mini app thực thi lệnh điều khiển BLE an toàn mà không phụ thuộc vào kết nối 4G/5G liên tục.

---

## 2. Client Privacy Signals, Content Indexing & Media Transformations

### 2.1. WICG User-Agent Client Hints & Anti-Fingerprinting
- **Nguồn chuẩn**: WICG User-Agent Client Hints (`https://wicg.github.io/ua-client-hints/`), W3C Permissions Policy, RFC 8942.
- **Mã định danh finding**: `ch_media_042_01`.
- **Cơ chế kỹ thuật**:
  - Thay thế chuỗi User-Agent tĩnh truyền thống bằng cơ chế kiểm soát ngân sách quyền riêng tư (Privacy Budget).
  - Phân tách thành **Low-Entropy Hints** (Sec-CH-UA, Sec-CH-UA-Mobile, Sec-CH-UA-Platform) được gửi mặc định và **High-Entropy Hints** (Sec-CH-UA-Model, Sec-CH-UA-Platform-Version) bắt buộc server phải yêu cầu qua header `Accept-CH` và được cấp quyền qua `Permissions-Policy`.
  - Container WebView chuẩn hóa các giá trị trả về để ngăn chặn mini app thu thập cấu hình phần cứng tạo mã fingerprinting định danh người dùng trái phép.

### 2.2. WICG Content Indexing API & Offline Discovery
- **Nguồn chuẩn**: WICG Content Indexing API (`https://wicg.github.io/content-index/spec/`), W3C Service Workers.
- **Mã định danh finding**: `ch_media_042_02`.
- **Cơ chế kỹ thuật**:
  - Cung cấp API trực tiếp trong Service Worker (`index.add()`, `index.delete()`, `index.getAll()`) cho phép mini app đăng ký siêu dữ liệu có cấu trúc (tiêu đề, tóm tắt, icon, danh mục, URL khởi chạy) của các tài nguyên đã lưu trong Cache Storage.
  - Super App Store và hub ngoại tuyến của OS có thể truy vấn chỉ mục này để hiển thị danh sách bài báo, vé xe, video đã tải sẵn trực tiếp trên màn hình chủ hoặc feed khám phá mà không cần khởi chạy toàn bộ WebView container.

### 2.3. W3C MediaStreamTrack Insertable Media (Breakout Box)
- **Nguồn chuẩn**: W3C MediaStreamTrack Insertable Media (`https://w3c.github.io/mediacapture-transform/`), W3C WebCodecs.
- **Mã định danh finding**: `ch_media_042_03`.
- **Cơ chế kỹ thuật**:
  - Giới thiệu `MediaStreamTrackProcessor` và `MediaStreamTrackGenerator`, cho phép chuyển đổi luồng camera/microphone thành các luồng `ReadableStream` và `WritableStream` của các đối tượng `VideoFrame` hoặc `AudioData`.
  - Cho phép DedicatedWorker xử lý khung hình thời gian thực (nhận diện mã QR, làm mờ hậu cảnh AI, chèn watermark chống rò rỉ, mã hóa luồng E2EE) với độ trễ dưới 16ms mà không gây nghẽn luồng UI chính (Main Thread) của Super App.

### 2.4. W3C Device Memory API & Phân tầng tài nguyên
- **Nguồn chuẩn**: W3C Device Memory (`https://www.w3.org/TR/device-memory/`), RFC 8942.
- **Mã định danh finding**: `ch_media_042_04`.
- **Cơ chế kỹ thuật**:
  - Cung cấp thuộc tính `navigator.deviceMemory` và header HTTP `Device-Memory` với các giá trị được lượng tử hóa (0.25, 0.5, 1, 2, 4, 8 GiB) nhằm triệt tiêu dấu vân tay phần cứng.
  - Hệ thống đóng gói và phân phối của Super App tự động phục vụ các gói tài nguyên đồ họa phù hợp (ảnh nén WebP/AVIF độ phân giải thấp, giảm hiệu ứng hạt trong WebGL canvas) cho các thiết bị có RAM <= 2 GiB, ngăn chặn triệt để sự cố crash sập ứng dụng do tràn bộ nhớ (Out-Of-Memory).

### 2.5. WICG EyeDropper API & Bảo vệ quyền riêng tư điểm ảnh
- **Nguồn chuẩn**: WICG EyeDropper API (`https://wicg.github.io/eyedropper-api/`).
- **Mã định danh finding**: `ch_media_042_05`.
- **Cơ chế kỹ thuật**:
  - Hỗ trợ công cụ lấy màu pixel trên màn hình (`eyeDropper.open({ signal })`) cho các mini app đồ họa, thiết kế và chỉnh sửa ảnh.
  - **Mô hình bảo mật User-Mediated**: JavaScript trong mini app hoàn toàn không thể đọc dữ liệu pixel khi người dùng đang di chuột qua các ứng dụng khác. Dữ liệu màu chỉ được trả về khi người dùng nhấp chọn điểm ảnh một cách chủ động. Trình thu phóng kính lúp do native container hiển thị độc lập, loại bỏ nguy cơ chụp lén màn hình hoặc đánh cắp mật khẩu hiển thị.

---

## 3. Standardized Payment Handling, Multi-Screen Windowing & Worklet Rendering

### 3.1. W3C Payment Handler API & Trung gian thanh toán nền tảng
- **Nguồn chuẩn**: W3C Payment Handler API (`https://www.w3.org/TR/payment-handler/`), W3C Payment Request API Level 1.
- **Mã định danh finding**: `pay_window_042_01`.
- **Cơ chế kỹ thuật**:
  - Chuẩn hóa kiến trúc cho phép các mini app ví điện tử hoặc cổng thanh toán bên thứ ba đăng ký công cụ thanh toán (`PaymentManager.instruments`) trong Service Worker.
  - Khi mini app thương mại gọi `PaymentRequest.show()`, container Super App đóng vai trò điều phối hiển thị giao diện xác thực thanh toán bảo mật với banner định danh nguồn gốc đáng tin cậy, triệt tiêu hoàn toàn rủi ro chuyển hướng lừa đảo (phishing) qua WebView không kiểm duyệt.

### 3.2. W3C Window Management API & Điều phối hiển thị đa màn hình POS
- **Nguồn chuẩn**: W3C Window Management API (`https://w3c.github.io/window-management/`), W3C Permissions Policy.
- **Mã định danh finding**: `pay_window_042_02`.
- **Cơ chế kỹ thuật**:
  - Cung cấp phương thức `window.getScreenDetails()` trả về thông số hình học chi tiết, độ phân giải và vị trí của tất cả các màn hình vật lý được kết nối với thiết bị.
  - Phục vụ trực tiếp cho các mini app bán hàng (POS) tại quầy thu ngân: mini app có thể mở cửa sổ hiển thị hóa đơn và mã thanh toán VietQR động trên màn hình phụ quay về phía khách hàng, trong khi màn hình chính dành riêng cho giao diện thao tác của nhân viên thu ngân. Được bảo vệ nghiêm ngặt bởi quyền `window-management`.

### 3.3. W3C Presentation API & Trình chiếu không dây màn hình phụ
- **Nguồn chuẩn**: W3C Presentation API (`https://www.w3.org/TR/presentation-api/`), WHATWG HTML.
- **Mã định danh finding**: `pay_window_042_03`.
- **Cơ chế kỹ thuật**:
  - Tách biệt rõ ràng giữa Controller Context (giao diện điều khiển trên điện thoại) và Receiver Context (nội dung trình chiếu trên Smart TV, máy chiếu hoặc màn hình quảng cáo qua Google Cast, AirPlay, Miracast hoặc cáp HDMI).
  - Thiết lập kênh thông điệp hai chiều `PresentationConnection` truyền dữ liệu text và binary (`ArrayBuffer`), phục vụ cho các mini app hội nghị truyền hình, trình chiếu tài liệu doanh nghiệp và kiosk tương tác.

### 3.4. WICG Local Font Access API & Kiểm soát phông chữ doanh nghiệp
- **Nguồn chuẩn**: WICG Local Font Access API (`https://wicg.github.io/local-font-access/`), ISO/IEC 14496-22 (OpenType).
- **Mã định danh finding**: `pay_window_042_04`.
- **Cơ chế kỹ thuật**:
  - Cung cấp hàm `window.queryLocalFonts()` cho phép các mini app soạn thảo văn bản, ký số hợp đồng và thiết kế đồ họa truy cập trực tiếp vào luồng nhị phân SFNT (`fontData.blob()`) của phông chữ hệ thống.
  - Ngăn ngừa nguy cơ fingerprinting danh sách phông chữ bằng cơ chế ảo hóa phông chữ (Font Virtualization) và yêu cầu quyền rõ ràng `local-fonts` trong Secure Context (HTTPS).

### 3.5. W3C CSS Paint API Level 1 & PaintWorklet Sandboxing
- **Nguồn chuẩn**: W3C CSS Paint API Level 1 (`https://www.w3.org/TR/css-paint-api-1/`), W3C Worklets Level 1.
- **Mã định danh finding**: `pay_window_042_05`.
- **Cơ chế kỹ thuật**:
  - Cung cấp `PaintWorklet` và phương thức `registerPaint()` cho phép thực thi vẽ đồ họa theo thuật toán (procedural graphics) trực tiếp trong giai đoạn render của engine đồ họa, không chạy trên luồng chính JavaScript.
  - **Môi trường cách ly tuyệt đối**: Không có quyền truy cập DOM, không có đối tượng `window`/`document` toàn cục, và hoàn toàn không có quyền gọi mạng (`fetch`/`XMLHttpRequest` đều là undefined). Giúp Super App phân phối bộ design tokens mượt mà đạt chuẩn 60fps mà không rò rỉ dữ liệu qua kênh thời gian.

---

## 4. Bảng tổng hợp đối chiếu kỹ thuật Iteration 42

| ID Finding | Tiêu chuẩn / Đặc tả | Cơ chế trọng tâm | Ứng dụng trên Super App Mini App Store | Mức độ xác thực |
|---|---|---|---|---|
| `fapi_token_042_01` | OpenID FAPI 2.0 Security Profile | Sender-Constrained Tokens, JARM, PKCE S256 | Chuẩn bảo mật bắt buộc cho mini app Fintech & Ngân hàng | Normative Standard (200 OK) |
| `fapi_token_042_02` | IETF RFC 8705 | mTLS Client Auth & Certificate-Bound Tokens | Ràng buộc phiên giao dịch vào mã băm chứng chỉ client X.509 | Normative Standard (200 OK) |
| `fapi_token_042_03` | IETF RFC 8693 | OAuth 2.0 Token Exchange & Downscoping | Phân quyền an toàn từ Super App sang backend đối tác | Normative Standard (200 OK) |
| `fapi_token_042_04` | IETF RFC 7516 | JSON Web Encryption (JWE) & A256GCM AEAD | Mã hóa đầu-cuối dữ liệu PII và thanh toán qua IPC bridge | Normative Standard (200 OK) |
| `fapi_token_042_05` | IETF RFC 9200 / RFC 8392 | ACE-OAuth & CBOR Web Tokens (CWT) | Token nhị phân siêu nhẹ cho mini app IoT, Smart Home, Auto | Normative Standard (200 OK) |
| `ch_media_042_01` | WICG User-Agent Client Hints | Low/High-Entropy Client Hints & Privacy Budget | Chặn fingerprinting thiết bị, bố cục responsive chuẩn | Consortium Spec (200 OK) |
| `ch_media_042_02` | WICG Content Indexing API | Service Worker Content Indexing & Offline Hub | Đưa nội dung offline mini app lên feed khám phá của store | Consortium Spec (200 OK) |
| `ch_media_042_03` | W3C MediaStreamTrack Breakout Box | Processor/Generator & DedicatedWorker pipeline | Xử lý video/audio thời gian thực, quét mã QR độ trễ <16ms | Consortium Spec (200 OK) |
| `ch_media_042_04` | W3C Device Memory API | Quantized RAM (0.25..8 GiB) & Asset Tiering | Phân phối tài nguyên phù hợp cấu hình, chống crash OOM | Consortium Spec (200 OK) |
| `ch_media_042_05` | WICG EyeDropper API | User-Mediated Screen Pixel Color Sampling | Chọn màu an toàn, chống trộm màn hình/mật khẩu | Consortium Spec (200 OK) |
| `pay_window_042_01` | W3C Payment Handler API | Service Worker Payment Instruments & Mediation | Cổng thanh toán chuẩn hóa, loại bỏ phishing WebView | Normative Standard (200 OK) |
| `pay_window_042_02` | W3C Window Management API | Multi-Screen Details & Coordinate Placement | Giải pháp 2 màn hình cho POS thu ngân và thanh toán QR | Consortium Spec (200 OK) |
| `pay_window_042_03` | W3C Presentation API | Dual-Screen Wireless Projection & Messaging | Trình chiếu TV/kiosk tách biệt context điều khiển | Normative Standard (200 OK) |
| `pay_window_042_04` | WICG Local Font Access API | Raw SFNT Blob Access & Font Virtualization | Soạn thảo văn bản doanh nghiệp, chống font fingerprint | Consortium Spec (200 OK) |
| `pay_window_042_05` | W3C CSS Paint API Level 1 | PaintWorklet Procedural Graphics, Zero DOM | Tăng tốc đồ họa design tokens 60fps không nghẽn main-thread | Normative Standard (200 OK) |
