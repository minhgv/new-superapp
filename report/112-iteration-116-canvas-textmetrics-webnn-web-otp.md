# Chuyên đề 112: Chuẩn Canvas 2D TextMetrics & Typographic Layout Sandboxing, W3C Web Neural Network API (WebNN) on Hardware Acceleration, và WICG Web OTP API Credential Verification

## 1. Giới thiệu & Tổng quan

Tại Milestone 116 của chuẩn siêu ứng dụng (*Super App Mini App Store Standard*), nghiên cứu tập trung giải quyết 3 thách thức nền tảng trong phát triển mini-app hiện đại:
1. **HTML Canvas 2D TextMetrics & Typographic Layout Sandboxing**: Khả năng tính toán bounding box chính xác đến từng sub-pixel, điều khiển hướng văn bản đa ngôn ngữ (LTR/RTL), căn chỉnh khoảng cách chữ/từ và co giãn phông chữ trong các giao diện Canvas (fintech charts, vé điện tử, digital signature, e-receipt, 2D game UI).
2. **W3C Web Neural Network API (WebNN) & On-Device ML Sandboxing**: Kiến trúc biên dịch đồ thị tính toán ML trực tiếp xuống phần cứng tăng tốc (NPU/GPU/CPU qua DirectML, Core ML, NNAPI), bảo vệ tài nguyên hệ thống, chống tấn công timing cache-leakage và cơ chế fallback WebAssembly SIMD.
3. **WICG Web OTP API & Programmatic SMS Credential Verification**: Cơ chế tự động trích xuất mã xác thực 1-lần (OTP) qua SMS ràng buộc chặt chẽ với domain (`@domain #otp`), thay thế quyền đọc toàn bộ SMS nguy hiểm (READ_SMS), bảo vệ quyền riêng tư người dùng theo Nghị định 13/2023/NĐ-CP và GDPR.

---

## 2. HTML Canvas 2D TextMetrics & Typographic Sandboxing

### 2.1 Bounding Box Typographic và Tọa độ Bề mặt Canvas
Chuẩn WHATWG HTML Canvas 2D mở rộng giao diện `TextMetrics` trả về từ `CanvasRenderingContext2D.measureText()` với các thuộc tính hình học chính xác:
- `actualBoundingBoxLeft` / `actualBoundingBoxRight`: Đo khoảng cách từ điểm căn lề đến điểm cực biên của glyph thực tế.
- `actualBoundingBoxAscent` / `actualBoundingBoxDescent`: Đo chiều cao glyph thực tế vượt trên và dưới đường baseline typographic.
- `fontBoundingBoxAscent` / `fontBoundingBoxDescent`: Đo khung biên font lý thuyết, đảm bảo không cắt ngắn dấu phụ (diacritics tiếng Việt, ký tự dấu tiếng Thái, Arabic).

Trong môi trường siêu ứng dụng, container runtime áp dụng cơ chế xác thực bounds nhằm ngăn ngừa lỗi tràn bộ nhớ framebuffer VRAM do cấp phát kích thước canvas vượt ngưỡng cho phép khi render văn bản động.

### 2.2 Đa Ngôn ngữ Bi-Directional & Micro-Typography
- **`direction`**: Thiết lập luồng văn bản (`ltr`, `rtl`, `inherit`), đồng bộ trực tiếp với ngôn ngữ hiển thị của vỏ Super App Shell.
- **`wordSpacing` & `letterSpacing`**: Cho phép căn lề văn bản quang học (optical justification) chính xác mà không cần tính toán thủ công tọa độ từng glyph trong script JavaScript.
- **`fontStretch` & `fontVariantCaps`**: Cho phép nhúng phông chữ cô đọng (`condensed`) để tối ưu hóa không gian hiển thị trên màn hình di động nhỏ, đồng thời hỗ trợ `small-caps` từ OpenType features mà không cần tải thêm file font phụ trợ.

---

## 3. W3C Web Neural Network API (WebNN) & On-Device ML Sandboxing

### 3.1 Biên dịch Đồ thị Tính toán ML trên Phần cứng Đa dạng
WebNN định nghĩa chuẩn chung cho việc thực thi inference mạng nơ-ron sâu trực tiếp trên thiết bị đầu cuối thông qua các backend native:
- **Windows**: DirectML
- **macOS / iOS**: Core ML
- **Android**: NNAPI / Vulkan

Container Super App kiểm soát quá trình tạo context `navigator.ml.createContext()`:
- Quản lý mức tiêu thụ năng lượng (`powerPreference: 'default' | 'high-performance' | 'low-power'`).
- Kiểm tra tính hợp lệ của đồ thị tính toán trước khi biên dịch, chặn nguy cơ tấn công side-channel (cache-timing attacks) rò rỉ dữ liệu qua GPU/NPU dùng chung.
- Quản lý hạn ngạch buffer bộ nhớ tensor (Tensor Memory Quota) tránh làm sập tiến trình host container do Out-Of-Memory (OOM).

### 3.2 Hỗ trợ Lượng tử hóa & Polyfill WebAssembly SIMD
- **Quantization (INT8 / INT4)**: Tối ưu dung lượng mô hình SLM chạy cục bộ trong mini-app mà vẫn giữ độ chính xác nhận diện.
- **WebNN Polyfill**: Cung cấp tầng fallback chuẩn chạy trên WebAssembly Fixed-Width SIMD khi thiết bị chưa hỗ trợ driver WebNN gốc, bảo đảm tính nhất quán chức năng xuyên suốt toàn bộ vòng đời sản phẩm.
- **Benchmark Qualification**: Sử dụng các mẫu MobileNet và SqueezeNet chuẩn từ W3C Web Machine Learning CG để kiểm thử hiệu năng khung hình (SLA >= 60 FPS) trước khi phê duyệt mini-app lên cửa hàng.

---

## 4. WICG Web OTP API & Programmatic SMS Credential Verification

### 4.1 Cú pháp Tin nhắn SMS Xác thực Ràng buộc Tên miền
Để giải quyết bài toán lừa đảo đảo danh (phishing) và đánh cắp mã OTP, WICG chuẩn hóa định dạng SMS:
```text
Ma xac thuc Super App cua ban la 123456.

@miniapp.superapp.vn #123456
```
- Dấu `@` xác định domain mini-app được cấp quyền nhận mã.
- Dấu `#` xác định chuỗi mã OTP.
- Hệ điều hành và runtime Super App đối chiếu domain trong tin nhắn với origin của mini-app đang chạy; nếu không khớp, mã tuyệt đối không được bàn giao.

### 4.2 Tích hợp Credential Management API & Quản lý Vòng đời
- Gọi qua hàm chuẩn:
  ```javascript
  const credential = await navigator.credentials.get({
    otp: { transport: ['sms'] },
    signal: abortController.signal
  });
  ```
- **AbortController**: Thiết lập timeout (60-120 giây) để giải phóng tài nguyên nếu người dùng không nhận được SMS.
- **Iframe Delegation**: Cho phép chia sẻ quyền nhận OTP an toàn tới iframe thanh toán nhúng qua thuộc tính `allow="otp-credentials"`.
- **`CredentialsContainer.preventSilentAccess()`**: Bắt buộc kích hoạt khi người dùng đăng xuất nhằm hủy bỏ trạng thái tự động truy cập credential ngầm, bảo vệ phiên giao dịch của người dùng trên thiết bị chia sẻ.

---

## 5. Bảng Đối chiếu & Tiêu chí Thẩm định Store Review

| Mã Tiêu Chí | Lĩnh Vực | Chuẩn Kỹ Thuật | Yêu Cầu Thẩm Định Store Review | Mức Bằng Chứng |
|---|---|---|---|---|
| `CANVAS-TEXTMETRICS-TYPOGRAPHIC-BOUNDS-HTML-CANVAS` | Canvas Graphics | WHATWG Canvas TextMetrics | Bắt buộc sử dụng bounding box đa trục khi render text tùy biến để tránh cắt diacritics. | Official Standard |
| `CANVAS-DIRECTION-BIDI-TEXT-MDN` | Canvas Graphics | MDN Canvas 2D direction | Tự động thích ứng hướng văn bản LTR/RTL theo shell locale của siêu ứng dụng. | Official Standard |
| `CANVAS-WORDSPACING-JUSTIFICATION-MDN` | Canvas Graphics | MDN Canvas 2D wordSpacing | Đồng bộ khoảng cách từ với TextMetrics width để tránh tràn lề màn hình di động. | Official Standard |
| `CANVAS-FONTSTRETCH-VARIABLE-TYPOGRAPHY-MDN` | Canvas Graphics | MDN Canvas 2D fontStretch | Sử dụng fontStretch chuẩn để tối ưu hóa hiển thị dữ liệu tài chính mật độ cao. | Official Standard |
| `CANVAS-FONTVARIANTCAPS-SMALL-CAPS-MDN` | Canvas Graphics | MDN Canvas 2D fontVariantCaps | Áp dụng typography small-caps chuẩn OpenType, không tải thêm tệp font trùng lặp. | Official Standard |
| `WEBNN-COMPUTE-GRAPH-ONDEVICE-W3C-TR` | On-Device ML | W3C WebNN Recommendation | Kiểm duyệt đồ thị tính toán, giới hạn quota tensor buffer bộ nhớ GPU/NPU. | Official Standard |
| `WEBNN-EXPLAINER-ARCHITECTURE-PRIVACY-SANDBOX` | On-Device ML | WebNN Explainer Specification | Sandbox origin, yêu cầu ngữ cảnh bảo mật HTTPS và quản lý powerPreference. | Official Standard |
| `WEBNN-COMMUNITY-GROUP-BENCHMARKS-STANDARDS` | On-Device ML | W3C Web Machine Learning CG | Vượt qua bộ kiểm thử Web Platform Tests (WPT) cho các toán tử đồ thị nơ-ron. | Official Standard |
| `WEBNN-POLYFILL-CPU-WASM-SANDBOXING` | On-Device ML | WebNN Wasm SIMD Polyfill | Tích hợp engine fallback Wasm SIMD bảo đảm hoạt động trên phần cứng không có NPU. | Official Standard |
| `WEBNN-SAMPLES-VISION-NLP-STORE-BENCHMARKS` | On-Device ML | WebNN Samples & Benchmarks | Đạt ngưỡng hiệu năng 60 FPS khi chạy inference model thị giác/ngôn ngữ. | Official Standard |
| `WEB-OTP-CREDENTIAL-INTERFACE-MDN` | Authentication | MDN OTPCredential API | Xác thực 1-tap OTP qua navigator.credentials.get(), hiển thị consent sheet của OS. | Official Standard |
| `WEB-OTP-CODE-PROPERTY-ABORT-CONTROLLER-MDN` | Authentication | MDN OTPCredential.code | Thiết lập AbortSignal timeout từ 60s đến 120s cho mọi luồng nhận mã OTP. | Official Standard |
| `WEB-OTP-IMPLEMENTATION-GUIDE-WEB-DEV` | Authentication | web.dev Web OTP Guide | Khai báo allow="otp-credentials" khi ủy quyền nhận OTP trong iframe thanh toán. | Official Standard |
| `WEB-OTP-EXPLAINER-SECURITY-BOUNDARY-WICG-GH` | Authentication | WICG Web OTP Security Explainer | Cấm tuyệt đối yêu cầu quyền READ_SMS nguyên bản; bắt buộc dùng Web OTP API. | Official Standard |
| `WEB-CREDENTIALS-PREVENT-SILENT-ACCESS-MDN` | Authentication | MDN CredentialsContainer | Gọi preventSilentAccess() ngay khi logout hoặc hết hạn phiên giao dịch. | Official Standard |
