# Chuyên đề 138: Chuẩn Hóa OAuth 2.0 Step-Up Authentication (RFC 9470), Cô Lập Cross-Origin Embedder Policy (COEP Credentialless) & Kiến Trúc Dữ Liệu Ngữ Nghĩa W3C JSON-LD 1.1

## 1. Bối cảnh & Mục tiêu Kỹ thuật Vòng 138
Trong hệ sinh thái Super App quy mô lớn phục vụ hàng chục triệu người dùng, việc bảo vệ tài nguyên nhạy cảm (giao dịch thanh toán giá trị cao, cấu hình bảo mật tài khoản, truy cập dữ liệu nhạy cảm) đòi hỏi các cơ chế nâng cấp xác thực tức thời (Step-Up Authentication) mà không làm gián đoạn trải nghiệm người dùng. Đồng thời, việc chạy song song các Mini App phức tạp có đồ họa 3D/DSP đòi hỏi môi trường Cross-Origin Isolation với chi phí triển khai tối thiểu, và việc phân phối danh mục ứng dụng cần một kiến trúc dữ liệu ngữ nghĩa chuẩn hóa để tối ưu hóa tìm kiếm, định tuyến và liên kết ứng dụng liên nền tảng.

Vòng nghiên cứu 138 mở rộng chuẩn hóa hệ thống qua 3 trụ cột kỹ thuật nền tảng:
1. **IETF RFC 9470 OAuth 2.0 Step-Up Authentication Challenge Protocol**: Chuẩn hóa mã lỗi HTTP 401 `insufficient_user_authentication`, tham số `acr_values`, `max_age`, metadata `acrs_supported`, kiểm tra xác thực thời gian thực và tự động chặn bắt (interception) trên Native WebView Bridge để kích hoạt Biometric Prompt bản địa.
2. **Cross-Origin Embedder Policy (COEP) 'credentialless' & WHATWG HTML Iframe Credentialless**: Chuẩn hóa cơ chế tải tài nguyên không kèm ambient credentials, cho phép kích hoạt Cross-Origin Isolation (SharedArrayBuffer, High-Res Timers) mà không bị phụ thuộc vào header CORP của bên thứ ba; thẻ `iframe credentialless` cho phép nhúng widget đối tác trong sandbox ephemeral độc lập.
3. **W3C JSON-LD 1.1 Core Syntax, Processing Algorithms & Framing**: Chuẩn hóa mô hình dữ liệu ngữ nghĩa Linked Data (@context, @id, @type, @graph), thuật toán Expansion/Compaction chuẩn hóa dữ liệu, JSON-LD Framing định hình cấu trúc cây cho giao diện di động, phòng thủ SSRF/Prototype Pollution và phân phối danh mục Mini App liên nền tảng qua Schema.org.

---

## 2. Chi Tiết Kỹ Thuật Từng Trụ Cột

### Trụ cột 1: IETF RFC 9470 OAuth 2.0 Step-Up Authentication Challenge Protocol

#### 2.1. Thách thức Xác thực Động (Dynamic Authentication Challenges)
- Trong kiến trúc truyền thống, khi một Access Token hiện có không đủ cấp độ bảo mật (chẳng hạn token phát hành qua mật khẩu hoặc ghi nhớ đăng nhập, nhưng Mini App muốn thực hiện chuyển tiền > 500,000 VND hoặc đổi mật khẩu), các hệ thống thường tự định nghĩa mã lỗi hoặc bắt người dùng đăng xuất.
- **IETF RFC 9470** chuẩn hóa giao thức phản hồi lỗi từ Resource Server (RS) về Client qua header HTTP 401:
  ```http
  HTTP/1.1 401 Unauthorized
  WWW-Authenticate: Bearer error="insufficient_user_authentication",
    error_description="A higher authentication level or fresh credentials are required",
    acr_values="urn:superapp:auth:bio urn:superapp:auth:fido-hw",
    max_age=300,
    scope="transfer:execute"
  ```
- **Ý nghĩa tham số**:
  - `error="insufficient_user_authentication"`: Báo hiệu rõ ràng cho client rằng token hợp lệ nhưng cấp độ xác thực người dùng (ACR) hoặc thời gian kể từ lần xác thực cuối (`auth_time`) không đáp ứng chính sách bảo mật của endpoint này.
  - `acr_values`: Chuỗi định danh các lớp xác thực được chấp nhận (Authentication Context Class References), ví dụ: xác thực sinh trắc học thiết bị, khóa phần cứng FIDO2.
  - `max_age`: Thời gian tối đa (tính bằng giây) kể từ lần cuối người dùng xác thực chủ động. Nếu `max_age=300`, người dùng phải xác thực trong vòng 5 phút trở lại.

#### 2.2. Khám phá Metadata Authorization Server & Kiểm tra Token Claims
- **AS Metadata (`acrs_supported`)**: AS công bố tại `/.well-known/oauth-authorization-server` danh sách các ACR hỗ trợ:
  ```json
  {
    "issuer": "https://auth.superapp.internal",
    "acrs_supported": [
      "urn:superapp:auth:pwd",
      "urn:superapp:auth:sms-otp",
      "urn:superapp:auth:bio",
      "urn:superapp:auth:fido-hw"
    ]
  }
  ```
- **Kiểm tra Token Claims tại Resource Gateways**:
  - Token JWT (hoặc kết quả RFC 7662 Introspection) phải mang các claims:
    - `acr`: Chuỗi định danh cấp độ xác thực đã đạt được.
    - `auth_time`: Thời điểm Unix Epoch khi người dùng tương tác xác thực.
  - Resource Server kiểm tra:
    $$\Delta t = 	ext{current\_time} - 	ext{auth\_time}$$
    Nếu $\Delta t > 	ext{max\_age}$ hoặc `acr` không nằm trong danh mục cho phép $ightarrow$ từ chối ngay lập tức với HTTP 401 RFC 9470.

#### 2.3. Điều phối Tự động qua Native WebView Bridge
- Khi Mini App gửi request API gặp lỗi 401 `insufficient_user_authentication`, tầng SDK JavaScript trong WebView tự động phát hiện và gửi yêu cầu tới Super App Native Container:
  ```javascript
  superapp.auth.requestStepUp({
    acrValues: "urn:superapp:auth:bio",
    maxAge: 300
  }).then(freshToken => {
    // Tự động retry request ban đầu với token mới
  });
  ```
- Native Container hiển thị hộp thoại xác thực sinh trắc học hệ điều hành (FaceID / TouchID / Android BiometricPrompt) và gửi assertion lên AS để nhận Access Token mới (được ràng buộc DPoP hoặc mTLS).
- Toàn bộ luồng diễn ra mượt mà, không làm reload trang web, không làm mất giỏ hàng hay trạng thái giao diện của Mini App.

---

### Trụ cột 2: Cross-Origin Embedder Policy (COEP) 'credentialless' & Sandboxing Subresource

#### 2.1. Đột phá từ 'require-corp' sang 'credentialless'
- Trước đây, để bật tính năng **Cross-Origin Isolation** (bảo vệ phần cứng chống tấn công Spectre để sử dụng `SharedArrayBuffer` và `performance.measureUserAgentSpecificMemory()`), trình duyệt yêu cầu:
  ```http
  Cross-Origin-Opener-Policy: same-origin
  Cross-Origin-Embedder-Policy: require-corp
  ```
- Nhược điểm lớn của `require-corp`: Mọi hình ảnh, script, font từ CDN của bên thứ ba đều phải có header `Cross-Origin-Resource-Policy: cross-origin`. Nếu CDN đối tác chưa cấu hình, tài nguyên sẽ bị chặn tải hoàn toàn.
- **COEP 'credentialless'** giải quyết triệt để rào cản này:
  ```http
  Cross-Origin-Embedder-Policy: credentialless
  ```
- **Cơ chế hoạt động**: Mọi yêu cầu cross-origin phát sinh từ document sẽ được gửi đi **không mang theo thông tin xác thực** (no cookies, no client certificates, no authorization headers).
- Phía server đích phản hồi không có cookies $ightarrow$ trình duyệt cho phép hiển thị an toàn vì kẻ tấn công không thể dùng document để đọc lén tài nguyên nhạy cảm có định danh của người dùng.

#### 2.2. Thẻ `iframe credentialless` trong WHATWG HTML
- Chuẩn HTML mở rộng thuộc tính boolean `credentialless` cho phần tử `<iframe>`:
  ```html
  <iframe src="https://partner-widget.example.com" credentialless></iframe>
  ```
- Khung iframe chạy trong một ngữ cảnh lưu trữ và mạng tạm thời (ephemeral storage partition), không truy cập được cookie hay localStorage của chính domain nó trên trình duyệt chính.
- Mọi dữ liệu lưu trữ tạo ra trong iframe sẽ bị hủy ngay khi iframe bị gỡ bỏ hoặc đóng ứng dụng.
- Giúp Super App nhúng an toàn các widget đối tác, quảng cáo tương tác hoặc cổng liên kết bên thứ ba mà không sợ rò rỉ session hoặc bị theo dõi cross-site.

#### 2.3. Bộ ba CORP, COOP & COEP trong Kiến Trúc Container
- **CORP (Cross-Origin-Resource-Policy)**: Bảo vệ tài nguyên nội bộ (`same-origin`, `same-site`).
- **COOP (Cross-Origin-Opener-Policy)**: Ngăn chặn tấn công qua `window.opener` (`same-origin`).
- **COEP (Cross-Origin-Embedder-Policy)**: Chặn tấn công kênh kề Spectre (`credentialless`).
- Tạo thành pháo đài bảo mật ba lớp cho các Mini App tài chính và Mini Game 3D nặng.

---

### Trụ cột 3: W3C JSON-LD 1.1 Core Syntax, Processing Algorithms & Framing

#### 3.1. Cú pháp Cốt lõi & Semantic Graph
- **JSON-LD 1.1** (W3C Recommendation 2020) mang lại khả năng định nghĩa ngữ nghĩa rõ ràng cho dữ liệu JSON mà không làm thay đổi cấu trúc parser JSON cơ bản:
  ```json
  {
    "@context": "https://schema.org",
    "@type": "SoftwareApplication",
    "@id": "urn:superapp:miniapp:fintech-01",
    "name": "Chuyển Tiền Siêu Tốc",
    "applicationCategory": "FinanceApplication",
    "operatingSystem": "SuperApp MiniApp Runtime v2",
    "permissions": "camera, contacts, biometric",
    "offers": {
      "@type": "Offer",
      "price": "0",
      "priceCurrency": "VND"
    }
  }
  ```
- Mọi thuộc tính được ánh xạ tới IRI định danh toàn cầu qua `@context`, ngăn ngừa hoàn toàn tình trạng xung đột trường dữ liệu giữa hàng nghìn nhà phát triển độc lập.

#### 3.2. Thuật toán Expansion, Compaction & Flattening
- **Expansion**: Loại bỏ hoàn toàn `@context`, mở rộng toàn bộ thuộc tính thành IRI đầy đủ. Được sử dụng trong các pipeline kiểm duyệt bảo mật tự động tại cổng nộp ứng dụng để xác thực schema mà không bị đánh lừa bởi context injection.
- **Compaction**: Thu gọn đồ thị IRI thành JSON ngắn gọn theo `@context` đích, tối ưu hóa băng thông tải danh mục ứng dụng về thiết bị di động.
- **Flattening**: Biến đổi đồ thị lồng nhau phức tạp thành mảng các node phẳng với liên kết `@id`, loại bỏ các vòng lặp tham chiếu tuần hoàn trong bộ nhớ di động.

#### 3.3. JSON-LD Framing: Định hình Cấu trúc Cây cho Giao diện Di động
- Các component di động (React Native, Flutter, Swift UI) yêu cầu dữ liệu dạng cây phân cấp cố định thay vì một mạng lưới đồ thị (graph).
- **JSON-LD Framing** sử dụng một tài liệu khung mẫu (Frame) để trích xuất và định hình lại dữ liệu:
  ```json
  {
    "@context": "https://schema.org",
    "@type": "SoftwareApplication",
    "@embed": "@always",
    "offers": {}
  }
  ```
- API Gateway của Mini App Store áp dụng Framing trên server để trả về đúng cấu trúc cây mà UI component cần, loại bỏ logic chuyển đổi phức tạp ở client.

#### 3.4. An toàn Thông tin: Chống SSRF & Độc hại Ngữ cảnh (Context Injection)
- Nguy cơ: Nếu server tự động tải URL khai báo trong `@context` không kiểm soát, kẻ tấn công có thể khai thác SSRF để quét mạng nội bộ hoặc làm tê liệt parser bằng tấn công từ chối dịch vụ (DoS).
- **Quy chuẩn Bảo mật Super App**:
  - Tuyệt đối cấm tải động `@context` từ mạng bên ngoài khi xử lý manifest của Mini App.
  - Tích hợp sẵn bộ nhớ đệm ngoại tuyến (Offline Context Cache) chứa các ontology chuẩn: Schema.org, W3C, DIF.
  - Giới hạn độ sâu phân giải ngữ cảnh $\le 10$ cấp và kiểm tra chặn thuộc tính nguyên mẫu (`__proto__`, `constructor`) nhằm triệt tiêu lỗ hổng Prototype Pollution.

---

## 3. Ma Trận Quy Chuẩn Kỹ Thuật Bắt Buộc (Super-App Store Matrix)

| Lĩnh vực | Tiêu chuẩn | Yêu cầu Kỹ thuật đối với Super App / Mini App | Mức độ Ưu tiên |
| :--- | :--- | :--- | :--- |
| **Bảo mật & Step-Up** | IETF RFC 9470 | Cổng API Gateway trả HTTP 401 `insufficient_user_authentication` kèm `acr_values` và `max_age` cho giao dịch nhạy cảm. | **Bắt buộc (P0)** |
| **Bridge Tương tác** | Native Step-Up Bridge | Native container WebView tự động chặn bắt lỗi 401 RFC 9470, kích hoạt xác thực sinh trắc học bản địa và retry request ngầm. | **Bắt buộc (P0)** |
| **Cô lập Subresource** | COEP Credentialless | Cấu hình header `Cross-Origin-Embedder-Policy: credentialless` cho phép chạy `SharedArrayBuffer` mà không lỗi CDN bên thứ ba. | **Bắt buộc (P1)** |
| **Sandboxing Iframe** | WHATWG Iframe Credentialless | Mọi iframe nhúng widget bên thứ ba trong Mini App bắt buộc phải có thuộc tính `credentialless` để phân vùng dữ liệu. | **Bắt buộc (P1)** |
| **Metadata Danh mục** | W3C JSON-LD 1.1 | Mini App Store Manifest và chia sẻ nội dung sử dụng định dạng JSON-LD 1.1 với Schema.org `SoftwareApplication`. | **Bắt buộc (P1)** |
| **Xử lý Ngữ nghĩa** | JSON-LD Framing | Server-side Framing để định hình danh mục ứng dụng thành cấu trúc cây UI thân thiện với bộ nhớ di động. | **Khuyến nghị (P2)** |
| **Phòng thủ SSRF** | Context Caching | Cấm tải `@context` tùy ý từ Internet; sử dụng bộ cache ontology nội bộ tại Gateway. | **Bắt buộc (P0)** |

---

## 4. Tác Động Thực Tiễn Đến Viettel / Alibaba Cloud Superapp Solution
1. **Bảo vệ Giao dịch Tài chính & Ví Điện tử**: Việc tích hợp chuẩn RFC 9470 giúp các Mini App ngân hàng, ví điện tử, thanh toán hóa đơn trên Super App yêu cầu nâng cấp xác thực vân tay/khuôn mặt ngay tại bước thanh toán mà không làm rớt phiên làm việc.
2. **Nâng cao Hiệu năng Đồ họa Đa luồng**: Hỗ trợ COEP Credentialless mở đường cho các Mini App 3D, WebGPU, xử lý ảnh AI chạy đa luồng bằng WebAssembly và `SharedArrayBuffer` mà không gặp sự cố CORS/CORP từ các kho tài nguyên CDN hiện hữu.
3. **Mở rộng Hệ sinh thái Danh mục Mở**: Kiến trúc danh mục dựa trên JSON-LD 1.1 và Schema.org giúp Super App dễ dàng liên kết, đồng bộ danh mục ứng dụng với các hệ thống phân phối của đối tác doanh nghiệp lớn và công cụ tìm kiếm tổng thể.
