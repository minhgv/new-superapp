# APPENDIX A — Danh mục Năng lực Nền tảng Web

| Trường | Giá trị |
|---|---|
| ID | `APPENDIX-A` |
| Phiên bản | 1.1.0 |
| Trạng thái | Draft — tài liệu tham khảo |
| Mục đích | Tra cứu năng lực → báo cáo nguồn → SPEC phụ trách |
| Nguồn | 44 báo cáo chuyên đề (`report/48` … `report/91`) |

> Danh mục này chỉ mục hóa các báo cáo `report/48` … `report/91`. Các báo cáo
> mới hơn (nếu có) sẽ được bổ sung vào phiên bản sau của phụ lục này.

---

## 1. Mục đích & cách dùng

Danh mục này là **bảng tra nhanh** toàn bộ năng lực nền tảng web đã nghiên cứu
trong 44 báo cáo chuyên đề (mỗi báo cáo ≈ 15 findings). Mỗi dòng ánh xạ:

```text
Năng lực  →  Báo cáo nguồn  →  SPEC phụ trách  →  Mức hỗ trợ đề xuất
```

> Tài liệu này **không** thay thế các SPEC; nó chỉ giúp tìm đúng SPEC khi cần
> tra cứu một năng lực cụ thể.

**Mức hỗ trợ đề xuất:** `MUST` (bắt buộc host hỗ trợ) · `SHOULD` (khuyến nghị) ·
`MAY` (tùy chọn) · `REF` (chỉ tham khảo, không yêu cầu).

---

## 2. Nhóm năng lực

### A. Giao diện & CSS

| Năng lực | Mô tả | Báo cáo | SPEC | Mức |
|---|---|---|---|---|
| CSS Cascading Level 5 `@layer` | Lớp ưu tiên kiểu, cách ly host/mini-app | `39` | `SPEC-04` | SHOULD |
| CSS Cascading Level 6 `@scope` | Phạm vi kiểu "donut", ranh giới theo độ gần | `39` | `SPEC-04` | MAY |
| CSS Containment Level 2/3 | `content-visibility`, `@container`, cách ly bố cục | `70`, `39` | `SPEC-03` | SHOULD |
| CSS Anchor Positioning | Neo định vị phần tử không cần JS | `60` | `SPEC-03` | MAY |
| HTML Popover / Top-Layer | Lớp đỉnh, bẫy focus, `aria-expanded` tự đồng bộ | `60`, `80` | `SPEC-03` | MAY |
| CSS Grid Layout 1/2 | Bố cục lưới | `75` | `SPEC-03` | REF |
| CSS Flexible Box 1 | Bố cục linh hoạt | `74` | `SPEC-03` | REF |
| CSS Box Alignment / Box Model 3 | Căn chỉnh, mô hình hộp | `89`, `72` | `SPEC-03` | REF |
| CSS Logical Properties 1 | Thuộc tính logic cho BiDi/RTL | `88` | `SPEC-11` | SHOULD |
| CSS Writing Modes / BiDi | Hướng viết, đa ngôn ngữ | `91` | `SPEC-11` | REF |
| CSS Transforms 2 / 3D | Biến đổi 2D/3D | `85` | `SPEC-03` | REF |
| CSS Masking / Filter Effects | Che, hiệu ứng lọc | `85`, `86` | `SPEC-03` | REF |
| CSS Shapes / Positioned Layout / will-change | Hình dạng, bố cục định vị, tối ưu render | `90` | `SPEC-03` | REF |
| CSS Multi-column | Bố cục đa cột | `86` | `SPEC-03` | REF |
| CSS Overflow 3/4 & Scrollbars | Tràn, thanh cuộn ổn định | `84` | `SPEC-03` | REF |
| CSS Scroll Snap / Overscroll | Bám cuộn, chống cuộn tràn | `76` | `SPEC-03` | MAY |
| CSS Text 3 / Text Decoration | Chữ, trang trí văn bản | `79`, `89` | `SPEC-03` | REF |
| CSS Counter Styles 3 | Kiểu đánh số bản địa hóa | `87` | `SPEC-11` | REF |
| CSS Custom Highlight API | Tô sáng văn bản tùy biến | `87` | `SPEC-03` | MAY |
| CSS UI Module 4 | Điều khiển giao diện chuẩn | `91` | `SPEC-03` | REF |
| CSS Transitions 2 / Animations | Chuyển tiếp, hoạt ảnh | `83`, `91` | `SPEC-03` | REF |
| CSS Typed OM | Truy cập thuộc tính CSS có kiểu | `68` | `SPEC-03` | MAY |
| CSS Properties & Values `@property` | Đăng ký thuộc tính CSS tùy biến | `70` | `SPEC-03` | MAY |
| CSS Values & Units 4 (Math) | Hàm toán học CSS | `82` | `SPEC-03` | REF |
| CSS Conditional Rules 4 `@supports` | Phát hiện tính năng, suy thoái tiến bộ | `82`, `39` | `SPEC-03` | SHOULD |
| CSS Color 4 / Color HDR | `display-p3`, `oklch()`, gamut/dynamic-range | `48` | `SPEC-03` | REF |
| CSS Nesting 1 | Lồng CSS | `79` | `SPEC-03` | REF |
| CSS Motion Path | Chuyển động theo đường dẫn | `88` | `SPEC-03` | MAY |
| CSS Font Metric Overrides | Ghi đè chỉ số phông chữ | `62` | `SPEC-03` | MAY |
| Scroll-driven Animations | Hoạt ảnh theo cuộn | `68` | `SPEC-03` | MAY |
| View Transitions 1/2 | Chuyển cảnh mượt | `32`, `47` | `SPEC-03` | MAY |

### B. Cấu trúc & DOM

| Năng lực | Mô tả | Báo cáo | SPEC | Mức |
|---|---|---|---|---|
| HTML `<dialog>` / `inert` | Hộp thoại, đóng băng cây con | `80` | `SPEC-03` | SHOULD |
| HTML Template & Shadow DOM Slot | Thành phần, ghép nội dung | `75` | `SPEC-03` | REF |
| Declarative Shadow DOM | Thủy hóa không-JS | `29` | `SPEC-03` | SHOULD |
| Custom Elements / ElementInternals | Thành phần tùy biến, form-associated | `39`, `62` | `SPEC-04` | SHOULD |
| Shadow DOM Scoping / `::part()` | Cách ly kiểu có kiểm soát | `39`, `29` | `SPEC-04` | SHOULD |
| FormData / URLSearchParams | Tuần tự hóa tải trọng | `75` | `SPEC-03` | REF |
| HTML Constraint Validation | Kiểm tra biểu mẫu, bảo mật form | `86` | `SPEC-03` | REF |
| DOM TreeWalker / NodeFilter / NodeIterator | Duyệt cây DOM | `88` | `SPEC-03` | REF |
| DOM AbortController | Hủy thao tác bất đồng bộ | `73` | `SPEC-03` | SHOULD |
| MutationObserver | Kiểm toán biến động DOM | `43`, `81` | `SPEC-03` | SHOULD |
| ResizeObserver | Theo dõi kích thước | `81` | `SPEC-03` | SHOULD |
| IntersectionObserver 1/2 | Theo dõi hiển thị | `72` | `SPEC-03` | MUST |
| CSS Container Queries | Truy vấn vùng chứa | `81` | `SPEC-03` | SHOULD |
| Selection API | Chọn văn bản | `69` | `SPEC-03` | REF |
| Touch Events | Cảm ứng, cử chỉ di động | `87` | `SPEC-03` | REF |
| Drag and Drop | Kéo thả | `71` | `SPEC-03` | REF |
| `Invoker Commands` | Hành vi khai báo trên nút | `64` | `SPEC-03` | MAY |
| CloseWatcher | Đóng giao diện có kiểm soát | `63` | `SPEC-03` | MAY |
| `dialog` Top-Layer / Popover | Phối hợp lớp đỉnh | `60`, `80` | `SPEC-03` | MAY |

### C. Hiệu năng & đo lường

| Năng lực | Mô tả | Báo cáo | SPEC | Mức |
|---|---|---|---|---|
| Performance Timeline 2 | Dòng hiệu năng | `74` | `SPEC-10` | SHOULD |
| User Timing 3 | Đo hiệu năng tùy biến | `71` | `SPEC-10` | REF |
| Resource Timing 2 | Thời gian tải tài nguyên, `workerStart` | `65` | `SPEC-10` | SHOULD |
| Paint Timing (FP/FCP), LCP, Event Timing (INP) | Chỉ số Core Web Vitals | `20` | `SPEC-10` | MUST |
| PerformanceObserver | Theo dõi hiệu năng | `74` | `SPEC-10` | SHOULD |
| ReportingObserver / Reporting API | Báo cáo lỗi bất đồng bộ | `77`, `36` | `SPEC-10` | SHOULD |
| Long Tasks 1.0 / LoAF | Phát hiện tác vụ dài, phân tích khung | `29` | `SPEC-10` | MUST |
| Compute Pressure | Tải CPU/nhiệt, shed load | `63`, `34` | `SPEC-10` | SHOULD |
| Idle Detection | Phát hiện nhàn rỗi | `55` | `SPEC-07` | MAY |
| Prioritized Task Scheduling | Hàng đợi ưu tiên, `scheduler.yield()` | `36` | `SPEC-03` | MAY |
| Network Error Logging (NEL) | Nhật ký lỗi kết nối | `20`, `33` | `SPEC-10` | MAY |
| Network Information API | `effectiveType`, `downlink`, `saveData` | `33` | `SPEC-10` | SHOULD |
| Priority Hints / Fetch Priority | Ưu tiên tải tài nguyên | `33`, `79` | `SPEC-10` | SHOULD |
| HTTP 103 Early Hints | Tải trước phụ thuộc | `33` | `SPEC-10` | MAY |
| Server-Timing / Trace Context | Phân rã truy vết dịch vụ | `65`, `27` | `SPEC-10` | SHOULD |
| Device Memory / Device Memory API | Lượng tử hóa RAM | `38` | `SPEC-07` | MAY |

### D. Worker & đồng thời

| Năng lực | Mô tả | Báo cáo | SPEC | Mức |
|---|---|---|---|---|
| Web Workers (Dedicated) | Chạy nền ngoài luồng chính | `85` | `SPEC-02` | MUST |
| Shared Workers | Chia sẻ ngữ cảnh đa trang | `58` | `SPEC-02` | SHOULD |
| Service Workers (lifecycle, Navigation Preload) | Bắt sự kiện, tải trước | `80`, `61`, `41` | `SPEC-02` | SHOULD |
| AudioWorklet | Xử lý âm thanh chuyên dụng | `77` | `SPEC-03` | MAY |
| Streams API / Transform Streams | Luồng dữ liệu | `67`, `85` | `SPEC-03` | SHOULD |
| Atomics / SharedArrayBuffer | Đồng bộ đa luồng | `36`, `23` | `SPEC-04` | MAY |
| BroadcastChannel | Phát-đăng ký liên ngữ cảnh | `28` | `SPEC-03` | SHOULD |
| Web Locks | Khóa độc quyền/chia sẻ | `21` | `SPEC-10` | MUST |
| Compression Streams | Nén/giải nén stream | `32` | `SPEC-03` | SHOULD |
| Task Scheduling / `scheduler.postTask` | Ưu tiên tác vụ | `36` | `SPEC-03` | MAY |

### E. Media & đồ họa

| Năng lực | Mô tả | Báo cáo | SPEC | Mức |
|---|---|---|---|---|
| Media Capture and Streams | `getUserMedia`, track lifecycle | `32`, `84` | `SPEC-06` | MUST |
| MediaRecorder | Ghi âm/hình, timeslice | `35` | `SPEC-03` | SHOULD |
| Web Audio API 1.1 | Xử lý âm thanh, `suspend()` | `72`, `36` | `SPEC-03` | SHOULD |
| Media Session API | Điều khiển phát | `21` | `SPEC-03` | SHOULD |
| WebCodecs / VideoFrame | Xử lý video, `close()` | `26`, `57` | `SPEC-03` | MAY |
| MediaSource / ManagedMediaSource | Streaming thích ứng, eviction | `64`, `78` | `SPEC-03` | MAY |
| Media Capture Transform (Breakout Box) | Xử lý luồng trong worker | `57` | `SPEC-03` | MAY |
| WebXR Device API | AR/VR, `xr-spatial-tracking` | `26`, `57` | `SPEC-06` | MAY |
| Screen / Region Capture | Chia sẻ màn hình, `CropTarget` | `36`, `60` | `SPEC-06` | MAY |
| Picture-in-Picture | Cửa sổ nổi | `83` | `SPEC-03` | MAY |
| Canvas 2D / Path2D | Đồ họa 2D | `76`, `84` | `SPEC-03` | REF |
| ImageBitmap / OffscreenCanvas | Đồ họa ngoài luồng, transferable | `76`, `43` | `SPEC-03` | SHOULD |
| WebGL 2.0 (context loss) | Đồ họa 3D, phục hồi ngữ cảnh | `61` | `SPEC-03` | MAY |
| WebGPU / WGSL | Đồ họa & tính toán GPU | `26` | `SPEC-03` | MAY |
| SVG 2 | Đồ họa vector | `78` | `SPEC-03` | REF |
| ImageCapture | Chụp ảnh, `grabFrame()` | `56` | `SPEC-03` | MAY |
| Audio Output Devices | Điều hướng thiết bị phát | `60` | `SPEC-06` | MAY |
| MediaStream Recording | Xem `MediaRecorder` | `35` | `SPEC-03` | SHOULD |
| WebRTC 1.0 (PeerConnection, DataChannel) | Truyền thông thời gian thực | `32` | `SPEC-03` | MAY |
| SFrame (RFC 9605) E2EE | Mã hóa đầu-cuối media | `64` | `SPEC-04` | MAY |

### F. Mạng & HTTP

| Năng lực | Mô tả | Báo cáo | SPEC | Mức |
|---|---|---|---|---|
| HTTP Semantics / HTTP/2 (RFC 9110/9113) | Ngữ nghĩa HTTP | `78` | `SPEC-04` | REF |
| HTTP Caching (RFC 9111) | Bộ đệm HTTP | `71` | `SPEC-10` | REF |
| Fetch Metadata (`Sec-Fetch-*`) | Chống CSRF/XSSI/XS-Leaks | `36` | `SPEC-04` | MUST |
| CORP / COOP / COEP | Cách ly origin chéo | `36`, `23`, `62` | `SPEC-04` | MUST |
| CSP Level 3 | Chính sách nội dung | `23`, `66` | `SPEC-04` | MUST |
| WebSocket (RFC 6455) | Kênh hai chiều | `69` | `SPEC-03` | SHOULD |
| WebTransport (HTTP/3 datagram) | Truyền tải thế hệ mới | `26` | `SPEC-03` | MAY |
| Server-Sent Events / EventSource | Luồng sự kiện | `42` | `SPEC-03` | SHOULD |
| Compression Dictionary (RFC 9842) | Nén từ điển chia sẻ, delta | `47` | `SPEC-10` | MAY |
| Speculation Rules / Speculative Loading | Tải trước/tải trước kết xuất | `32` | `SPEC-10` | MAY |
| Beacon API | Gửi khi rời trang | `66` | `SPEC-03` | SHOULD |
| Back/Forward Cache (bfcache) | Bộ đệm điều hướng | `66` | `SPEC-10` | MAY |
| Soft Navigations | Phát hiện điều hướng SPA | `58` | `SPEC-03` | MAY |
| Navigation API / URLPattern | Định tuyến SPA | `44` | `SPEC-03` | SHOULD |
| Cache API | Bộ đệm ứng dụng | `74` | `SPEC-03` | SHOULD |
| Local Network Access (LNA) | Truy cập mạng cục bộ | `49` | `SPEC-04` | MAY |
| MIME Sniffing / `nosniff` | An toàn kiểu dữ liệu | `65` | `SPEC-04` | MUST |
| HTTP Signatures (RFC 9421) / Digest (RFC 9530) | Chữ ký & toàn vẹn HTTP | `41` | `SPEC-05` | SHOULD |

### G. Bộ nhớ & lưu trữ

| Năng lực | Mô tả | Báo cáo | SPEC | Mức |
|---|---|---|---|---|
| IndexedDB 3.0 | Cửa hàng bất đối xứng, ACID | `73`, `33` | `SPEC-03` | MUST |
| OPFS (Origin Private File System) | Hệ tệp riêng theo origin | `52` | `SPEC-04` | SHOULD |
| File System Access API | Chọn/ghi tệp, atomic write | `32` | `SPEC-06` | SHOULD |
| File API (binary sandboxing) | Xử lý tệp an toàn | `51` | `SPEC-04` | SHOULD |
| Storage Buckets | Nhóm lưu trữ + TTL | `49` | `SPEC-07` | MAY |
| StorageManager Quota | Hạn ngạch lưu trữ | `83` | `SPEC-10` | SHOULD |
| Storage Partitioning / CHIPS | Phân vùng cookie & bộ nhớ | `28` | `SPEC-07` | MUST |
| Storage Access API | Truy cập chéo origin | `28` | `SPEC-07` | SHOULD |
| Clear-Site-Data | Xóa sạch dữ liệu | `28` | `SPEC-07` | MUST |
| Cookie Store API | Cookie bất đồng bộ | `43` | `SPEC-03` | SHOULD |
| FileSystemObserver | Theo dõi biến động tệp | `59` | `SPEC-03` | MAY |
| File Handling / `file_handlers` | Liên kết tệp với ứng dụng | `43` | `SPEC-03` | MAY |
| `navigator.storage.persist()` | Lưu trữ bền | `33` | `SPEC-07` | MAY |
| Content Indexing API | Khám phá ngoại tuyến | `38` | `SPEC-03` | MAY |

### H. Thiết bị & cảm biến

| Năng lực | Mô tả | Báo cáo | SPEC | Mức |
|---|---|---|---|---|
| Geolocation | Vị trí, độ chính xác | `43`, `18` | `SPEC-06` | MUST |
| Accelerometer / Gyroscope / Magnetometer | Chuyển động, lượng tử hóa | `35`, `46` | `SPEC-06` | MAY |
| Orientation / RelativeOrientation | Hướng thiết bị | `35`, `82` | `SPEC-06` | MAY |
| Ambient Light Sensor | Ánh sáng, lượng tử hóa | `35` | `SPEC-06` | MAY |
| Generic Sensor Framework | Khung cảm biến chung | `46` | `SPEC-06` | MAY |
| Device Posture | Trạng thái gập | `34` | `SPEC-06` | MAY |
| Battery Status | Pin, lượng tử hóa | `21`, `41` | `SPEC-06` | MAY |
| Screen Wake Lock | Giữ màn hình | `43`, `21` | `SPEC-06` | MAY |
| Vibration | Rung, transient activation | `34` | `SPEC-06` | MAY |
| Gamepad API & Extensions | Tay cầm, phản hồi rung | `35` | `SPEC-06` | MAY |
| Web Bluetooth / BLE Scanning | Bluetooth, GATT filtering | `46`, `56` | `SPEC-06` | MAY |
| Web NFC | NFC, NDEF, read-before-write | `46` | `SPEC-06` | MAY |
| WebUSB / WebHID / Web Serial | Ngoại vi, sandbox exploit defense | `25` | `SPEC-06` | MAY |
| VirtualKeyboard | Bàn phím ảo, `geometrychange` | `77` | `SPEC-03` | SHOULD |
| Visual Viewport | Hình học viewport động | `58` | `SPEC-03` | SHOULD |
| Pointer Events 3 / Pointer Lock | Đầu vào độ trễ thấp, khóa con trỏ | `65`, `35` | `SPEC-06` | SHOULD |
| Input Events 2 | `beforeinput`, IME mediation | `35` | `SPEC-06` | SHOULD |
| Keyboard Map | Bố cục phím độc lập | `35` | `SPEC-06` | MAY |
| Clipboard API / Custom Formats | Bảng tạm, pickling | `35`, `63` | `SPEC-06` | SHOULD |
| Barcode Detection | Quét mã phần cứng | `34` | `SPEC-03` | SHOULD |
| ImageCapture | Chụp ảnh từ camera | `56` | `SPEC-03` | MAY |
| Geometry Interfaces 1 | DOMRect, kích thước | `67` | `SPEC-03` | REF |

### I. Danh tính & bảo mật

| Năng lực | Mô tả | Báo cáo | SPEC | Mức |
|---|---|---|---|---|
| WebAuthn 2/3, Passkeys | Xác thực không mật khẩu | `45`, `56`, `59` | `SPEC-03` | SHOULD |
| WebAuthn PRF | Dẫn xuất khóa đối xứng | `45` | `SPEC-04` | MAY |
| WebAuthn Signal API / Passkey Endpoints | Đồng bộ, thu hồi passkey | `50` | `SPEC-06` | MAY |
| Digital Credentials API | Giấy tờ số, tiết lộ chọn lọc | `43` | `SPEC-06` | MAY |
| Credential Management / FedCM | Liên kết danh tính | `25` | `SPEC-06` | MAY |
| WebOTP | OTP gắn origin | `34` | `SPEC-03` | MAY |
| WebCrypto / SubtleCrypto / CryptoKey | Mã hóa, khóa không xuất | `43`, `20` | `SPEC-04` | MUST |
| Permissions API / Permissions Policy | Trạng thái & ủy quyền quyền | `67`, `18` | `SPEC-06` | MUST |
| Capability Delegation | Ủy quyền năng lực | `50` | `SPEC-06` | SHOULD |
| Private State Tokens / Privacy Pass | Chống lạm dụng riêng tư | `31` | `SPEC-07` | SHOULD |
| Trusted Types / Sanitizer API | Chống XSS | `23`, `52` | `SPEC-04` | MUST |
| Subresource Integrity / Signature-based SRI | Toàn vẹn tài nguyên | `54` | `SPEC-05` | MUST |
| Web Install | Cài đặt ứng dụng | `50` | `SPEC-03` | MAY |
| Scope Extensions / Link Capturing | Mở rộng phạm vi, bắt link | `45` | `SPEC-04` | MAY |
| Remote Attestation (RATS, Play Integrity, App Attest) | Chứng thực thiết bị | `37` | `SPEC-05` | SHOULD |
| Verifiable Credentials / DID / SD-JWT | Danh tính phi tập trung | `37` | `SPEC-05` | MAY |
| SCIM 2.0 / CIBA | Quản trị danh tính doanh nghiệp | `55` | `SPEC-08` | MAY |
| WebDriver BiDi | Kiểm thử tự động | `51` | `SPEC-12` | MAY |
| Certificate Transparency v2 (RFC 9162) | Minh bạch chứng chỉ | `36` | `SPEC-05` | SHOULD |

### J. Thương mại & thanh toán

| Năng lực | Mô tả | Báo cáo | SPEC | Mức |
|---|---|---|---|---|
| Payment Request API | Chuẩn thanh toán | `20`, `89` | `SPEC-09` | MUST |
| Payment Handler API | Công cụ thanh toán SW | `38` | `SPEC-09` | MAY |
| Payment Method Manifest | Khai báo phương thức | `20` | `SPEC-09` | MUST |
| Secure Payment Confirmation | Xác nhận bảo mật | `20` | `SPEC-09` | MUST |
| Digital Goods API | Danh mục SKU, quyền sở hữu | `30` | `SPEC-09` | SHOULD |
| Window Management API | POS đa màn hình | `38` | `SPEC-09` | MAY |
| Presentation API | Trình chiếu không dây | `38` | `SPEC-09` | MAY |
| OpenID FAPI 2.0 / mTLS / Token Exchange | Bảo mật API tài chính | `38` | `SPEC-09` | SHOULD |

### K. Thông báo, văn bản & quốc tế hóa

| Năng lực | Mô tả | Báo cáo | SPEC | Mức |
|---|---|---|---|---|
| Notifications API | Thông báo | `24` | `SPEC-03` | SHOULD |
| Web Push (RFC 8030/8291/8292) | Đẩy, mã hóa, VAPID | `24` | `SPEC-03` | MAY |
| Badging API | Huy hiệu biểu tượng | `24`, `41` | `SPEC-03` | MAY |
| Notification Channels (Android) | Cách ly kênh thông báo | `24` | `SPEC-03` | MUST |
| display_override / Window Controls Overlay | Kiểm soát cửa sổ | `24` | `SPEC-03` | MAY |
| ECMA-402 Intl | Quốc tế hóa | `80` | `SPEC-11` | REF |
| Local Font Access API | Truy cập phông chữ | `38` | `SPEC-07` | MAY |
| CSS Counter Styles / Logical Properties | Bản địa hóa hiển thị | `87`, `88` | `SPEC-11` | REF |
| Web App Manifest Shortcuts | Khởi chạy nhanh | `60` | `SPEC-01` | MAY |

### L. IoT, AI & năng lực mở rộng

| Năng lực | Mô tả | Báo cáo | SPEC | Mức |
|---|---|---|---|---|
| Web of Things (TD 1.1, Profiles, Bindings) | Điều phối IoT | `48` | `SPEC-03` | MAY |
| Web Neural Network API (WebNN) | Suy luận AI trên thiết bị | `34` | `SPEC-03` | MAY |
| Protected Audience / Fenced Frames / Shared Storage | Quảng cáo trên thiết bị | `31` | `SPEC-07` | MAY |
| Attribution Reporting | Đo lường riêng tư | `31` | `SPEC-07` | SHOULD |
| Topics API | Chủ đề duyệt web | `48` | `SPEC-07` | MAY |
| Global Privacy Control / TCF / GPP | Tín hiệu riêng tư | `42` | `SPEC-07` | MUST |
| OpenFeature / Shared Signals (CAEP/RISC) | Feature gating, tín hiệu bảo mật | `53` | `SPEC-04` | SHOULD |
| Sub Apps API | Ứng dụng con, OS windowing | `49` | `SPEC-02` | MAY |
| WASI Component Model | Module hệ thống | `61` | `SPEC-04` | MAY |
| Telco Network APIs (CAMARA/GSMA) | API mạng viễn thông | `40` | `SPEC-03` | MAY |
| Post-Quantum Cryptography | Mật mã hậu lượng tử | `40` | `SPEC-05` | SHOULD |

---

## 3. Bảng tra nhanh A–Z

| Năng lực | Nhóm | Nguồn | SPEC |
|---|---|---|---|
| AbortController | DOM | `73` | `SPEC-03` |
| Accelerometer | Thiết bị | `35` | `SPEC-06` |
| Ambient Light Sensor | Thiết bị | `35` | `SPEC-06` |
| Atomics | Worker | `36` | `SPEC-04` |
| AudioWorklet | Media | `77` | `SPEC-03` |
| Badging | Thông báo | `24` | `SPEC-03` |
| Barcode Detection | Thiết bị | `34` | `SPEC-03` |
| Battery Status | Thiết bị | `21` | `SPEC-06` |
| Beacon | Mạng | `66` | `SPEC-03` |
| bfcache | Mạng | `66` | `SPEC-10` |
| BroadcastChannel | Worker | `28` | `SPEC-03` |
| Cache API | Lưu trữ | `74` | `SPEC-03` |
| Canvas 2D | Media | `76` | `SPEC-03` |
| CHIPS | Lưu trữ | `28` | `SPEC-07` |
| Clear-Site-Data | Lưu trữ | `28` | `SPEC-07` |
| Clipboard | Thiết bị | `35` | `SPEC-06` |
| CloseWatcher | DOM | `63` | `SPEC-03` |
| CloudEvents | Mạng | `42` | `SPEC-13` |
| Compute Pressure | Hiệu năng | `63` | `SPEC-10` |
| Container Queries | DOM | `81` | `SPEC-03` |
| Cookie Store | Lưu trữ | `43` | `SPEC-03` |
| COOP/COEP/CORP | Mạng | `36` | `SPEC-04` |
| Credential Management | Danh tính | `25` | `SPEC-06` |
| CSP | Mạng | `23` | `SPEC-04` |
| Custom Elements | DOM | `39` | `SPEC-04` |
| Declarative Shadow DOM | DOM | `29` | `SPEC-03` |
| Device Posture | Thiết bị | `34` | `SPEC-06` |
| Digital Credentials | Danh tính | `43` | `SPEC-06` |
| Digital Goods | Thương mại | `30` | `SPEC-09` |
| Drag and Drop | DOM | `71` | `SPEC-03` |
| FedCM | Danh tính | `25` | `SPEC-06` |
| Fetch Metadata | Mạng | `36` | `SPEC-04` |
| File Handling | Lưu trữ | `43` | `SPEC-03` |
| File System Access | Lưu trữ | `32` | `SPEC-06` |
| FileSystemObserver | Lưu trữ | `59` | `SPEC-03` |
| Filter Effects | CSS | `86` | `SPEC-03` |
| FormData | DOM | `75` | `SPEC-03` |
| Gamepad | Thiết bị | `35` | `SPEC-06` |
| Geolocation | Thiết bị | `43` | `SPEC-06` |
| Global Privacy Control | Riêng tư | `42` | `SPEC-07` |
| HTTP Caching | Mạng | `71` | `SPEC-10` |
| HTTP Signatures | Mạng | `41` | `SPEC-05` |
| Idle Detection | Hiệu năng | `55` | `SPEC-07` |
| ImageBitmap | Media | `76` | `SPEC-03` |
| ImageCapture | Thiết bị | `56` | `SPEC-03` |
| IndexedDB | Lưu trữ | `73` | `SPEC-03` |
| IntersectionObserver | DOM | `72` | `SPEC-03` |
| Invoker Commands | DOM | `64` | `SPEC-03` |
| Keyboard Map | Thiết bị | `35` | `SPEC-06` |
| Long Tasks / LoAF | Hiệu năng | `29` | `SPEC-10` |
| ManagedMediaSource | Media | `64` | `SPEC-03` |
| Media Session | Media | `21` | `SPEC-03` |
| MediaRecorder | Media | `35` | `SPEC-03` |
| MIME Sniffing | Mạng | `65` | `SPEC-04` |
| MutationObserver | DOM | `43` | `SPEC-03` |
| Navigation API | Mạng | `44` | `SPEC-03` |
| Notifications | Thông báo | `24` | `SPEC-03` |
| OPFS | Lưu trữ | `52` | `SPEC-04` |
| Orientation Sensor | Thiết bị | `82` | `SPEC-06` |
| Page Visibility | Hiệu năng | `69` | `SPEC-10` |
| Passkeys | Danh tính | `56` | `SPEC-03` |
| Payment Handler | Thương mại | `38` | `SPEC-09` |
| Payment Request | Thương mại | `20` | `SPEC-09` |
| Permissions API | Danh tính | `67` | `SPEC-06` |
| Picture-in-Picture | Media | `83` | `SPEC-03` |
| Pointer Events | Thiết bị | `65` | `SPEC-06` |
| Popover / Top-Layer | CSS | `60` | `SPEC-03` |
| Priority Hints | Hiệu năng | `33` | `SPEC-10` |
| Protected Audience | Riêng tư | `31` | `SPEC-07` |
| Push | Thông báo | `24` | `SPEC-03` |
| ResizeObserver | DOM | `81` | `SPEC-03` |
| Resource Timing | Hiệu năng | `65` | `SPEC-10` |
| Sanitizer API | Bảo mật | `52` | `SPEC-04` |
| Screen Wake Lock | Thiết bị | `43` | `SPEC-06` |
| Scroll Snap | CSS | `76` | `SPEC-03` |
| Selection API | DOM | `69` | `SPEC-03` |
| Service Worker | Worker | `41` | `SPEC-02` |
| Shared Storage | Riêng tư | `31` | `SPEC-07` |
| SharedArrayBuffer | Worker | `36` | `SPEC-04` |
| Shared Workers | Worker | `58` | `SPEC-02` |
| Soft Navigations | Mạng | `58` | `SPEC-03` |
| Speculation Rules | Mạng | `32` | `SPEC-10` |
| Storage Buckets | Lưu trữ | `49` | `SPEC-07` |
| Streams API | Worker | `67` | `SPEC-03` |
| Sub Apps | Runtime | `49` | `SPEC-02` |
| SRI | Bảo mật | `54` | `SPEC-05` |
| Touch Events | DOM | `87` | `SPEC-03` |
| Topics API | Riêng tư | `48` | `SPEC-07` |
| TreeWalker | DOM | `88` | `SPEC-03` |
| Trusted Types | Bảo mật | `23` | `SPEC-04` |
| User Timing | Hiệu năng | `71` | `SPEC-10` |
| Verifiable Credentials | Danh tính | `37` | `SPEC-05` |
| Vibration | Thiết bị | `34` | `SPEC-06` |
| VirtualKeyboard | Thiết bị | `77` | `SPEC-03` |
| Visual Viewport | Thiết bị | `58` | `SPEC-03` |
| WAAPI / Transitions | CSS | `68` | `SPEC-03` |
| WasmGC | Bảo mật | `52` | `SPEC-04` |
| Web Bluetooth | Thiết bị | `46` | `SPEC-06` |
| Web Crypto | Danh tính | `43` | `SPEC-04` |
| Web NFC | Thiết bị | `46` | `SPEC-06` |
| Web Neural Network | AI | `34` | `SPEC-03` |
| Web of Things | IoT | `48` | `SPEC-03` |
| Web Push | Thông báo | `24` | `SPEC-03` |
| Web Serial / USB / HID | Thiết bị | `25` | `SPEC-06` |
| Web Workers | Worker | `85` | `SPEC-02` |
| WebAuthn | Danh tính | `45` | `SPEC-03` |
| WebCodecs | Media | `26` | `SPEC-03` |
| WebGL 2.0 | Media | `61` | `SPEC-03` |
| WebGPU | Media | `26` | `SPEC-03` |
| WebRTC | Media | `32` | `SPEC-03` |
| WebTransport | Mạng | `26` | `SPEC-03` |
| WebXR | Media | `57` | `SPEC-06` |
| WebSocket | Mạng | `69` | `SPEC-03` |
| Window Controls Overlay | Thông báo | `24` | `SPEC-03` |
| Window Management | Thương mại | `38` | `SPEC-09` |

---

## 5. Bổ sung từ nhóm báo cáo mới (report/92 … report/145)

Các năng lực sau **chưa** có trong bảng ở mục 2–3, được bổ sung từ 50 báo cáo mới.

| Năng lực | Mô tả | Báo cáo | SPEC | Mức |
|---|---|---|---|---|
| OAuth 2.1 + PKCE | Đường cơ sở ủy quyền, S256 bắt buộc | `125`, `127` | `SPEC-06` | MUST |
| DPoP (RFC 9449) | Token gắn người gửi | `125` | `SPEC-06` | SHOULD |
| PAR (RFC 9126) | Đẩy tham số ủy quyền lên server | `127` | `SPEC-06` | SHOULD |
| JAR (RFC 9101) | Yêu cầu ủy quyền được ký | `124` | `SPEC-06` | SHOULD |
| JWT BCP (RFC 8725) | Chống giả mạo JWT | `124` | `SPEC-06` | MUST |
| Step-Up Auth (RFC 9470) | Nâng cấp xác thực theo ngữ cảnh | `138` | `SPEC-06` | SHOULD |
| Token introspection/revocation | RFC 7662 / RFC 7009 / RFC 8707 | `126` | `SPEC-06` | MUST |
| OpenID Connect Core | Liên kết danh tính, PPID | `126` | `SPEC-06` | SHOULD |
| OIDC Logout (RP/Back/Front) | Đăng xuất đầy đủ, không phiên mồ côi | `126` | `SPEC-06` | SHOULD |
| Device Authorization (RFC 8628) | TV, kiosk, POS | `126` | `SPEC-06` | MAY |
| OpenID Federation 1.0 | Trust chain đa phương | `127` | `SPEC-06` | SHOULD |
| GNAP | Cấp phép nhiều token | `128` | `SPEC-06` | MAY |
| FAPI 2.0 + RISC/SET (RFC 8417) | Sự kiện bảo mật liên bên | `128` | `SPEC-06` | SHOULD |
| OID4VCI 1.0 | Phát hành chứng chỉ số | `129` | `SPEC-06` | SHOULD |
| OID4VP 1.0 | Trình bày chứng chỉ, QR liên thiết bị | `130` | `SPEC-06` | SHOULD |
| SD-JWT / SD-JWT VC | Tiết lộ có chọn lọc | `129`, `130` | `SPEC-06` | SHOULD |
| OAuth Token Status List | Thu hồi mật mã | `129` | `SPEC-06` | MUST |
| Bitstring Status List 1.0 | Danh sách trạng thái nén | `131` | `SPEC-06` | MUST |
| VC Data Integrity 1.0 | Ed25519 / ECDSA / BBS+ | `131` | `SPEC-06` | SHOULD |
| DIF Presentation Exchange 2.1 | Mô tả yêu cầu dữ liệu | `131` | `SPEC-06` | SHOULD |
| DID Core 1.0 / did:web / did:peer | Danh tính phi tập trung | `132`, `133` | `SPEC-06` | MAY |
| DIDComm Messaging v2.0 | Truyền thông ví danh tính | `132` | `SPEC-06` | MAY |
| ISO/IEC 18013-5 / 18013-7 | Giấy phép lái xe số (mDL) | `133`, `134` | `SPEC-06` | MAY |
| CHAPI | Credential Handler API | `142` | `SPEC-06` | MAY |
| SIOPv2 | Self-Issued OpenID Provider | `133` | `SPEC-06` | MAY |
| Protected Resource Metadata (RFC 9728) | Metadata tài nguyên bảo vệ | `134` | `SPEC-06` | SHOULD |
| Entity Attestation Token (RFC 9711) | Chứng thực phần cứng, RATS | `135` | `SPEC-05` | SHOULD |
| C2PA v2.1 | Nguồn gốc nội dung số | `137` | `SPEC-05` | SHOULD |
| IETF SCITT | Sổ cái minh bạch | `137` | `SPEC-05` | SHOULD |
| RFC 3161 / RFC 6960 | Mốc thời gian, trạng thái chứng chỉ | `137` | `SPEC-05` | SHOULD |
| FIDO MDS 3.0 | Metadata thiết bị xác thực | `140` | `SPEC-05` | SHOULD |
| JWK Thumbprint (RFC 7638) | Định danh khóa máy đọc được | `135` | `SPEC-05` | MUST |
| Deterministic CBOR (RFC 8949) / COSE | Đối tượng ký chuẩn hóa | `143` | `SPEC-05` | MUST |
| Private Network Access (PNA) | Chống SSRF nội bộ | `142` | `SPEC-04` | MUST |
| COEP credentialless | Cách ly tải chéo origin | `138` | `SPEC-04` | SHOULD |
| Oblivious HTTP (RFC 9458) | Yêu cầu ẩn danh | `136` | `SPEC-04` | SHOULD |
| IETF DAP / Prio3 VDAF | Đo lường đa bên | `136` | `SPEC-04` | SHOULD |
| Oblivious PRF (RFC 9497) / PST | Chống lạm dụng riêng tư | `136` | `SPEC-04` | SHOULD |
| I-Regexp (RFC 9485) | Chống ReDoS | `141` | `SPEC-04` | MUST |
| Wasm JSPI | Tích hợp bất đồng bộ WebAssembly | `142` | `SPEC-04` | SHOULD |
| Controlled Frame / IWA | Nhúng web cách ly, sandbox | `145` | `SPEC-03` | SHOULD |
| CNCF SPIFFE | Danh tính workload | `145` | `SPEC-03` | SHOULD |
| Houdini Worklets | CSS Layout / Animation Worklet | `139` | `SPEC-03` | SHOULD |
| WebSocketStream | Kênh hai chiều có backpressure | `113` | `SPEC-03` | SHOULD |
| WebNN | Suy luận AI trên thiết bị | `112` | `SPEC-03` | MAY |
| Contact Picker API | Danh bạ theo phiên | `99` | `SPEC-03` | MAY |
| Web MIDI | Thiết bị MIDI, lọc sys-ex | `100` | `SPEC-03` | MAY |
| WebVTT | Phụ đề đồng bộ Media Session | `110` | `SPEC-03` | MAY |
| HTML Microdata / JSON-LD 1.1 | Ngữ nghĩa trang máy đọc được | `93`, `138` | `SPEC-03` | SHOULD |
| WAI-ARIA 1.2/1.3 | Phân loại role chuẩn | `144` | `SPEC-11` | MUST |
| AccName 1.2 | Tên truy cập tính được | `144` | `SPEC-11` | MUST |
| Core-AAM 1.2 / HTML-AAM 1.0 | Ánh xạ API trợ năng | `144` | `SPEC-11` | MUST |
| ARIA APG | Mẫu bàn phím & widget | `144` | `SPEC-11` | MUST |
| ARIA Live Regions | Thông báo động có kiểm soát | `144` | `SPEC-11` | SHOULD |
| RFC 9557 (IXDTF) | Thời gian có múi giờ tường minh | `143` | `SPEC-11` | MUST |
| TC39 Temporal / đa lịch | Lịch Gregory + lịch khác | `143` | `SPEC-11` | SHOULD |
| UUIDv7 (RFC 9562) | Định danh đơn điệu theo thời gian | `141` | `SPEC-13` | MUST |
| JSONPath (RFC 9535) | Truy vấn có cấu trúc an toàn | `140` | `SPEC-13` | SHOULD |
| RFC 9209 / RFC 9211 | Header proxy/cache minh bạch | `141` | `SPEC-13` | SHOULD |
| Web Linking (RFC 8288) / WebSub | Liên kết & kênh sự kiện | `109`, `141` | `SPEC-13` | MAY |
| Solid Protocol / WAC / ACP | Pod dữ liệu, kiểm soát truy cập | `140` | `SPEC-07` | MAY |
| VISS v2 | Telematics ô tô | `140` | `SPEC-03` | MAY |
| JSContact (RFC 9553/9555) | Danh bạ chuẩn hóa | `135` | `SPEC-07` | MAY |

---

## 6. Ghi chú

- 44 báo cáo nguồn (`report/48` … `report/91`) đều có ~15 findings với URL chuẩn
  và mức bằng chứng; bảng này chỉ là **chỉ mục**.
- Mức hỗ trợ `MUST` trong bảng là **đề xuất kiến trúc** của bộ SPEC này, không
  phải yêu cầu của chính chuẩn nền tảng tương ứng.
- Năng lực chưa có ở bảng nhưng xuất hiện trong báo cáo: xem trực tiếp
  `report/README.md` để biết vị trí.
