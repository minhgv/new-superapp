# Chuyên Đề 34: On-Device AI Compute, Advanced Optical Vision & Ergonomic Accessibility Governance (Iteration 38)

## 1. Bối Cảnh & Mục Tiêu Nghiên Cứu
Khi các siêu ứng dụng (Super-Apps) phát triển từ cổng dịch vụ web tĩnh thành nền tảng điện toán toàn diện, nhu cầu tích hợp các năng lực phần cứng tiên tiến ngày càng trở nên cấp bách:
1. **Trí tuệ nhân tạo trên thiết bị (On-Device AI)**: Chạy các mô hình học máy (Machine Learning) cục bộ (NLP nhận diện giọng nói, thị giác máy tính, phân tích dữ liệu tại biên) mà không phụ thuộc độ trễ mạng hay làm lộ dữ liệu người dùng ra máy chủ bên ngoài.
2. **Thị giác quang học & Thích ứng phần cứng gập (Advanced Vision & Foldables)**: Quét mã QR/Barcode phần cứng tăng tốc và bố cục giao diện phản hồi trên các thiết bị màn hình gập (foldable/dual-screen devices).
3. **Xác thực phi ma sát & Tiếp cận kỹ thuật số (Frictionless Verification & Accessibility)**: Tự động trích xuất mã OTP SMS bảo mật cao, mô hình hóa cấu trúc hỗ trợ tiếp cận (AOM) cho các mini-app dựng bằng Canvas/WebGL, và chuẩn hóa phản hồi xúc giác (Haptics).

Chuyên đề này thiết lập khung chuẩn kỹ thuật cho Milestone 38, tích hợp 15 phát hiện chuẩn hóa thực nghiệm (`ai_compute_038_01` đến `05`, `vision_posture_038_01` đến `05`, `input_a11y_038_01` đến `05`), đưa tổng số phát hiện chuẩn mực toàn dự án lên **521 findings**.

---

## 2. On-Device AI Compute & Thermal Pressure Governance (W3C WebNN & Compute Pressure)

### 2.1. W3C Web Neural Network API (WebNN) Sandboxing
- **Cơ chế kiến trúc**: WebNN cung cấp lớp trừu tượng phần cứng (Hardware-Agnostic Abstraction Layer), cho phép JavaScript tương tác trực tiếp với các bộ tăng tốc NPU, GPU và SIMD CPU thông qua `navigator.ml.createContext()`.
- **Phân lập đồ thị tính toán (Graph Sandboxing)**:
  - Khởi tạo thông qua `MLGraphBuilder` để xây dựng đồ thị có hướng không chu trình (DAG).
  - Vùng nhớ Tensor được giới hạn trần cố định (tối đa 256MB cho mỗi phiên mini-app).
  - Giới hạn thời gian thực thi (Execution Timeout Clamping): Không cho phép suy luận liên tục quá 500ms để tránh chiếm dụng hoàn toàn luồng xử lý đồ họa hoặc làm đơ UI.
- **Phòng chống tấn công kênh phụ (Side-Channel Mitigation)**: Làm mịn độ phân giải thời gian (timestamp coarsening) và thêm độ trễ ngẫu nhiên (jitter) vào các Promise trả về từ quá trình suy luận, ngăn chặn việc đo đạc vi kiến trúc bộ nhớ đệm (cache-timing attacks).

### 2.2. Giám Sát Áp Lực Nhiệt Độ Hệ Thống (W3C Compute Pressure API)
- **Chuẩn hóa trạng thái áp lực**: Thông qua `PressureObserver`, hệ thống cung cấp 4 mức áp lực tài nguyên CPU/Nhiệt độ:
  - `nominal`: Tải bình thường, không có suy giảm hiệu năng.
  - `fair`: Áp lực nhẹ, quạt tản nhiệt hoặc tần số CPU bắt đầu điều chỉnh, người dùng chưa cảm nhận giật lag.
  - `serious`: Áp lực nhiệt nghiêm trọng, phần cứng bắt đầu hạ xung nhịp (thermal throttling).
  - `critical`: Nguy cơ tắt nguồn hoặc quá nhiệt phần cứng, cần giảm tải khẩn cấp.
- **Cầu nối hệ điều hành bản địa (Native OS Bridge Mapping)**:
  - **Android**: Lắng nghe `PowerManager.OnThermalStatusChangedListener` (từ `THERMAL_STATUS_LIGHT` đến `THERMAL_STATUS_SHUTDOWN`).
  - **iOS/iPadOS**: Lắng nghe `ProcessInfo.thermalState` (`ProcessInfo.ThermalState.nominal`, `fair`, `serious`, `critical`).
- **Chiến lược hạ cấp tải động (Graceful Degradation)**:
  - Khi chuyển sang `serious`: Giảm FPS WebView từ 60Hz xuống 30Hz, hạ độ phân giải mô hình ML từ FP32 xuống INT8.
  - Khi chuyển sang `critical`: Đình chỉ toàn bộ tác vụ nền (background tasks) và từ chối cấp phát bộ nhớ GPU/NPU mới.

---

## 3. Advanced Optical Vision & Device Posture Adaptation

### 3.1. Tăng Tốc Thị Giác Máy Tính: WICG Barcode Detection API
- **Phần cứng hóa nhận diện quang học**: Thay thế các thư viện JavaScript nặng nề (như ZXing, jsQR ngốn 200KB-500KB bundle và chiếm dụng CPU) bằng giao diện chuẩn `BarcodeDetector`.
- **Tách biệt luồng video bản địa (Zero Buffer Leakage)**:
  - Mini-app truyền `ImageBitmap`, `HTMLVideoElement` hoặc `OffscreenCanvas` vào `BarcodeDetector.detect()`.
  - Phép phân tích ma trận điểm ảnh diễn ra trên nền tảng thị giác phần cứng (Apple Vision framework hoặc Android MLKit) mà không sao chép mảng byte thô (raw frame buffers) vào JavaScript heap.
- **Chính sách phân quyền**: Ràng buộc chặt chẽ bởi `Permissions-Policy: camera=(self)`. Chỉ tài liệu hoạt động ở khung hình hiển thị (top-level document) mới được phép phân tích hình ảnh, giới hạn tần suất tối đa 10 FPS để tiết kiệm pin.

### 3.2. Thích Ứng Thiết Bị Gập (W3C Device Posture & CSS Viewport Segments)
- **Nhận diện trạng thái gập vật lý**: `navigator.devicePosture.type` phản ánh trạng thái thiết bị:
  - `continuous`: Màn hình phẳng thông thường hoặc mở phẳng 180 độ.
  - `folded`: Thiết bị đang gập lại ở góc bản lề (chế độ lều, chế độ laptop/tabletop).
- **Phân tách giao diện đa cửa sổ (Split Viewports)**:
  - Sử dụng biến môi trường CSS `env(viewport-segment-width)`, `env(viewport-segment-top)` để căn lề giao diện tránh đường nếp gấp vật lý (physical crease).
  - Tự động tách bố cục: bảng điều khiển/danh mục ở nửa dưới hoặc bên trái, khung chi tiết ở nửa trên hoặc bên phải.
  - Khóa xoay màn hình (`screen.orientation.lock()`) bị vô hiệu hóa khi thiết bị ở tư thế gập nhằm ngăn chặn xung đột trải nghiệm bản lề.

### 3.3. Tuân Thủ Chuẩn Mã Thanh Toán Quang Học (EMVCo & VietQR)
- Trình phân tích mã quang học tích hợp bộ giải mã cấu trúc TLV (Tag-Length-Value) theo chuẩn thanh toán quốc tế EMVCo và chuẩn quốc gia VietQR (Quyết định 2345/QĐ-NHNN, NAPAS247).
- Ngăn chặn triệt để mã độc chèn chuỗi URI (`javascript:`, `intent:`) hoặc giả mạo tài khoản nhận tiền bằng cách xác minh chữ ký CRC16 và thông tin Merchant ID tại tầng lõi của Super-App trước khi đẩy vào Mini-App.

---

## 4. Frictionless Input Verification & Ergonomic Accessibility

### 4.1. Tự Động Điền Mã OTP SMS Không Cần Quyền (WICG WebOTP API)
- **Cơ chế an toàn không ma sát**:
  - Mini-app gọi `navigator.credentials.get({otp: {transport: ['sms']}})`.
  - Không yêu cầu quyền nhạy cảm `READ_SMS` hay quyền truy cập danh bạ/hộp thư hệ điều hành.
  - Định dạng tin nhắn chuẩn hóa toàn cầu:
    ```
    Mã xác thực của bạn là: 123456
    @example.superapp.com #123456
    ```
- **Chống tấn công lừa đảo (Anti-Phishing Binding)**:
  - Mã xác thực bị ràng buộc định danh duy nhất với tên miền gốc (`@domain`).
  - Nền tảng Super-App đối soát tên miền gửi tin với Publisher ID trong `app.json`. Nếu không khớp, yêu cầu OTP bị từ chối ngay lập tức.
  - Kết hợp kiểm tra thẻ SIM (IMSI binding) tại tầng nhà mạng để phát hiện các cuộc tấn công hoán đổi SIM (SIM swap).

### 4.2. Khung Trợ Năng Phi DOM (WICG Accessibility Object Model - AOM)
- **Vấn đề thực tế**: Các mini-app hiệu năng cao hoặc mini-game dựng trên HTML5 Canvas/WebGL không có cấu trúc thẻ DOM chuẩn, khiến các trình đọc màn hình (TalkBack, VoiceOver) hoàn toàn "mù".
- **Kiến trúc cây trợ năng ảo (Virtual AccessibleNode Tree)**:
  - Mini-app lập trình khai báo các đối tượng `AccessibleNode` tương ứng với các nút ảo trên canvas.
  - Container ánh xạ các node ảo này vào `AccessibilityNodeInfo` (Android) hoặc `UIAccessibilityElement` (iOS) kèm theo tọa độ màn hình chính xác.
  - Kiểm định tự động trong quy trình xét duyệt cửa hàng: Mọi ứng dụng có thành phần Canvas tương tác bắt buộc phải đạt độ phủ trợ năng tối thiểu theo WCAG 2.2 SC 4.1.2.

### 4.3. Quản Trị Rung & Phản Hồi Xúc Giác (W3C Vibration API)
- `navigator.vibrate(pattern)` chỉ được kích hoạt khi có tương tác người dùng chủ động (transient user activation).
- Giới hạn cứng thời gian rung tối đa 200ms cho mỗi xung, tổng chuỗi rung không quá 1000ms.
- Tự động ngắt phản hồi xúc giác khi thiết bị ở chế độ Im Lặng/Không Làm Phiền (DND) hoặc khi mini-app mất tiêu điểm (hidden/blurred).

---

## 5. Ma Trận Tiêu Chuẩn & Mức Độ Tuân Thủ

| Thành Phần | Tiêu Chuẩn Kỹ Thuật | Phân Cấp Bắt Buộc | Cơ Chế Kiểm Soát Trong Store |
|---|---|---|---|
| **WebNN Runtime** | W3C WebNN / W3C WebGPU | Khuyến nghị (Khóa Chặt) | Giới hạn 256MB Tensor RAM, Timeout 500ms, Cấm nạp nhị phân tùy ý |
| **Compute Pressure** | W3C Compute Pressure API | Bắt buộc đối với App nặng | Tự động hạ xung nhịp / FPS khi chuyển trạng thái Serious/Critical |
| **Optical Scanner** | WICG Barcode Detection / EMVCo | Bắt buộc tiêu chuẩn | Ràng buộc Camera Permissions Policy, kiểm tra CRC16 VietQR/EMVCo |
| **Foldable Topology** | W3C Device Posture / CSS Viewport | Tự chọn (UI cấp tiến) | Kiểm tra hiển thị split-screen, vô hiệu hóa orientation lock |
| **WebOTP** | WICG WebOTP API | Bắt buộc cho FinTech/Commerce | Xác thực tên miền SMS @domain, chống quyền READ_SMS |
| **Virtual AOM** | WICG Accessibility Object Model | Bắt buộc cho Canvas UI | Kiểm thử tự động tỷ lệ phủ AccessibleNode với TalkBack/VoiceOver |
| **Haptic Vibration** | W3C Vibration API | Bắt buộc tuân thủ | Kẹp trần 200ms/xung, ngắt khi mất tiêu điểm cửa sổ |

---

## 6. Kết Luận & Tác Động Đến Bản Báo Cáo Chuẩn Hóa
Milestone 38 đã giải quyết triệt để bài toán tích hợp phần cứng thông minh trên thiết bị đầu cuối: biến siêu ứng dụng thành một môi trường tính toán an toàn cho AI tại biên (On-Device AI), tối ưu hóa trải nghiệm thị giác và thiết bị gập thế hệ mới, đồng thời nâng tầm khả năng tiếp cận người khuyết tật và xác thực không ma sát. Toàn bộ 15 tiêu chuẩn đã được kiểm chứng URL chính thức và phân loại mức bằng chứng cụ thể.
