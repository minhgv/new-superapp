# SPEC-03 — Bề mặt API cho Nhà phát triển

| Trường | Giá trị |
|---|---|
| Spec ID | `SPEC-03` |
| Phiên bản | 1.0.0 |
| Trạng thái | Draft |
| Lớp tuân thủ | **C3** (Mini App) — hợp đồng do **C1** thực thi |
| Nguồn | `report/16`, `report/29`, `report/24`, `report/46`, `report/30`, `report/25`, `report/34`, `report/35`, `report/43`, `report/56`, `report/47`, `report/63`, `report/67` |

---

## 1. Phạm vi & mục tiêu

Đây là **hợp đồng API** mà nhà phát triển mini-app được phép dùng trong container.
Tài liệu định nghĩa: nguyên tắc bề mặt API, phân nhóm API, mức hỗ trợ bắt buộc
từ host, và hợp đồng lỗi.

> **Quy tắc vàng:** API là **danh sách cho phép** (allowlist). Không có trong
> danh sách = không tồn tại. Mọi lời gọi đều bất đồng bộ, có kiểm tra quyền,
> và thất bại theo hướng an toàn (fail-closed).

---

## 2. Thuật ngữ

| Thuật ngữ | Định nghĩa |
|---|---|
| **API surface** | Tập API khả dụng cho mini-app trong container. |
| **Host API** | API gốc do container cung cấp qua JSBridge. |
| **Web API** | API chuẩn nền tảng web được container bộc lộ có kiểm soát. |
| **Entitlement** | Quyền vận hành đặc biệt (chạy nền, nghe nền, POS). |
| **Effective capability** | Giao của: đã khai báo ∩ đã cấp quyền ∩ cho phép bởi policy. |

---

## 3. Yêu cầu chuẩn hóa

### 3.1 Nguyên tắc bề mặt API

> **REQ-03-001** (MUST · C1) — Toàn bộ bề mặt API **PHẢI** là allowlist tường minh; API không được liệt kê **KHÔNG** được tồn tại trong ngữ cảnh mini-app.
> *Nguồn:* `→ report/16-...md` · *Kiểm chứng:* static

> **REQ-03-002** (MUST · C1) — Mỗi API nhạy cảm **PHẢI** kiểm tra giao của ba tập: **đã khai báo** trong manifest, **đã cấp quyền** bởi người dùng, và **cho phép** bởi Permissions Policy. Thiếu bất kỳ thành tố nào → từ chối.
> *Nguồn:* `→ report/18-...md`, `→ report/67-...md` · *Kiểm chứng:* runtime

> **REQ-03-003** (MUST · C1) — Mọi API **PHẢI** bất đồng bộ (trả Promise hoặc callback); **KHÔNG ĐƯỢC** cung cấp API đồng bộ chặn luồng chính.
> *Nguồn:* `→ report/29-...md` · *Kiểm chứng:* static

> **REQ-03-004** (MUST · C1) — Lỗi **PHẢI** dùng phân loại chuẩn (taxonomy) với mã ổn định; sự cố bảo mật **KHÔNG ĐƯỢC** tiết lộ nguyên nhân chi tiết cho mini-app.
> *Nguồn:* `→ report/16-...md` · *Kiểm chứng:* runtime

> **REQ-03-005** (SHOULD · C3) — Nhà phát triển **NÊN** luôn xử lý đường nhánh từ chối quyền (denied) và môi trường thiếu tính năng (feature-detect trước khi gọi).
> *Nguồn:* `→ report/67-...md` · *Kiểm chứng:* review

### 3.2 Nhóm giao tiếp & điều hướng

| API | Mô tả | Yêu cầu | Nguồn |
|---|---|---|---|
| `JSBridge.invoke / callHandler` | Gọi năng lực host | MUST-support | `report/16` |
| `MessageChannel / MessagePort` | Kênh điểm-điểm | SHOULD | `report/16` |
| `BroadcastChannel` | Phát-đăng ký liên ngữ cảnh | SHOULD | `report/28` |
| `registerProtocolHandler` | Đăng ký scheme `web+` | MAY | `report/29` |
| `manifest.protocol_handlers` | Ràng buộc khai báo giao thức | SHOULD | `report/29` |
| `window.launchQueue` / `LaunchParams` | Nhận tham số khởi chạy | MUST | `report/30` |
| Navigation API / `URLPattern` | Định tuyến SPA | SHOULD | `report/44` |

> **REQ-03-006** (MUST · C1) — Host **PHẢI** hỗ trợ hợp đồng `JSBridge.invoke` bất đồng bộ kèm correlation ID và phân giải Promise/điều khiển lỗi.
> *Nguồn:* `→ report/16-...md` · *Kiểm chứng:* runtime

> **REQ-03-007** (MUST · C1) — Mọi lời gọi liên mini-app **PHẢI** đi qua bộ phân giải deep-link duy nhất và mang token chứng thực người gọi.
> *Nguồn:* `→ report/29-...md`, `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-03-008** (MAY · C3) — Đăng ký scheme `web+` **CÓ THỂ** được dùng nhưng **PHẢI** qua quy tắc token `%s` chuẩn; host **PHẢI** chống chiếm luồng điều hướng.
> *Nguồn:* `→ report/29-...md` · *Kiểm chứng:* static

### 3.3 Nhóm mạng & dữ liệu

| API | Mô tả | Yêu cầu | Nguồn |
|---|---|---|---|
| `fetch` (có allowlist domain) | Gọi mạng | MUST | `report/19` |
| `WebSocket` (RFC 6455) | Kênh hai chiều | SHOULD | `report/69` |
| `EventSource` / SSE | Luồng sự kiện | SHOULD | `report/42` |
| `WebTransport` | HTTP/3 datagram | MAY | `report/26` |
| `CompressionStream` | Nén/giải nén stream | SHOULD | `report/32` |
| `Cache API` | Bộ đệm | SHOULD | `report/74` |
| `IndexedDB` (v3) | Cửa hàng bất đối xứng | MUST | `report/73` |
| OPFS / File System Access | Hệ tệp sandbox | SHOULD | `report/52`, `report/32` |
| Storage Buckets | Nhóm lưu trữ + TTL | MAY | `report/49` |
| `navigator.storage.persist()` | Xin lưu trữ bền | MAY | `report/33` |

> **REQ-03-009** (MUST · C1) — Gọi mạng **PHẢI** qua danh sách miền cho phép; miền ngoài allowlist phải bị chặn ở tầng container, không phụ thuộc CSP đơn thuần.
> *Nguồn:* `→ report/19-...md` · *Kiểm chứng:* runtime

> **REQ-03-010** (MUST · C1) — Nén/giải nén payload lớn (>5 MB) **PHẢI** chạy trong Dedicated Worker/Service Worker; **KHÔNG** được làm chặn luồng giao diện.
> *Nguồn:* `→ report/32-...md` · *Kiểm chứng:* runtime

> **REQ-03-011** (MUST · C1) — Truy cập hệ tệp **PHẢI** bị giới hạn bằng sandbox đường dẫn (virtual chroot), chặn `../`, và có danh sách từ chối của hệ điều hành.
> *Nguồn:* `→ report/32-...md`, `→ report/47-...md` · *Kiểm chứng:* runtime

> **REQ-03-012** (SHOULD · C3) — Ghi tệp **NÊN** dùng `FileSystemWritableFileStream` (ghi nguyên tử) để tránh tệp hỏng khi ngắt giữa chừng.
> *Nguồn:* `→ report/32-...md` · *Kiểm chứng:* review

### 3.4 Nhóm giao diện & hiệu năng

| API | Mô tả | Yêu cầu | Nguồn |
|---|---|---|---|
| `VirtualKeyboard` | Bàn phím ảo & `geometrychange` | SHOULD | `report/77` |
| Visual Viewport | Hình học viewport động | SHOULD | `report/58` |
| `IntersectionObserver` | Quan sát hiển thị | MUST | `report/72` |
| `ResizeObserver` / Container Queries | Thích ứng bố cục | SHOULD | `report/81` |
| `MutationObserver` | Kiểm toán DOM | SHOULD | `report/43` |
| CSS Anchor / Popover / Top-Layer | Neo định vị & lớp đỉnh | MAY | `report/60` |
| CSS Containment / `@layer` | Cách ly kiểu & bố cục | SHOULD | `report/39`, `report/70` |
| View Transitions | Chuyển cảnh mượt | MAY | `report/32` |
| `OffscreenCanvas` / `ImageBitmap` | Đồ họa ngoài luồng | SHOULD | `report/43`, `report/76` |
| `scheduler.postTask` | Lập lịch ưu tiên | MAY | `report/36` |

> **REQ-03-013** (MUST · C1) — Host **PHẢI** bộc lộ sự kiện `geometrychange` của bàn phím ảo và biến môi trường CSS viewport để mini-app tránh bị che khuất.
> *Nguồn:* `→ report/77-...md` · *Kiểm chứng:* runtime

> **REQ-03-014** (SHOULD · C1) — Container **NÊN** dùng Declarative Shadow DOM cho thủy hóa không-JS và cho phép `::part()`/`exportparts` có kiểm soát để theme an toàn.
> *Nguồn:* `→ report/29-...md` · *Kiểm chứng:* static

> **REQ-03-015** (MUST · C1) — Mọi vòng lặp layout bất tận từ observer **PHẢI** có ngưỡng giới hạn và bị cắt với cảnh báo.
> *Nguồn:* `→ report/81-...md` · *Kiểm chứng:* runtime

### 3.5 Nhóm media

| API | Mô tả | Yêu cầu | Nguồn |
|---|---|---|---|
| `getUserMedia` / MediaStream | Camera & mic | MUST-support | `report/32`, `report/36` |
| `MediaRecorder` | Ghi âm/hình | SHOULD | `report/35` |
| Web Audio / `AudioWorklet` | Xử lý âm thanh | SHOULD | `report/72`, `report/77` |
| Media Session API | Điều khiển phát | SHOULD | `report/21` |
| WebCodecs / `VideoFrame` | Xử lý video | MAY | `report/26`, `report/57` |
| `ManagedMediaSource` | Streaming thích ứng | MAY | `report/64` |
| WebXR Device API | AR/VR | MAY | `report/26`, `report/57` |
| Region / Screen Capture | Chia sẻ màn hình | MAY | `report/60`, `report/36` |
| Picture-in-Picture | Cửa sổ nổi | MAY | `report/83` |

> **REQ-03-016** (MUST · C1) — `VideoFrame`, `ImageBitmap` và `AudioContext` **PHẢI** được giải phóng tường minh (`.close()`, `suspend()`, `disconnect()`); container **PHẢI** có watchdog phát hiện rò rỉ khung hình.
> *Nguồn:* `→ report/26-...md` · *Kiểm chứng:* runtime

> **REQ-03-017** (MUST · C1) — Khi mini-app ở trạng thái `hidden`, `AudioContext` **PHẢI** tự `suspend()` trong 500 ms, trừ khi giữ entitlement `background-audio`.
> *Nguồn:* `→ report/36-...md` · *Kiểm chứng:* runtime

> **REQ-03-018** (SHOULD · C1) — Streaming video **NÊN** dùng `ManagedMediaSource` để cho phép user agent điều tiết bộ đệm và thực thi Coded Frame Eviction khi thiếu RAM.
> *Nguồn:* `→ report/64-...md` · *Kiểm chứng:* runtime

> **REQ-03-019** (MUST · C1) — Quay/chụp màn hình **PHẢI** chặn che khuất nội dung nhạy cảm bằng `CropTarget`/Region Capture và cờ `FLAG_SECURE` của cửa sổ host.
> *Nguồn:* `→ report/60-...md`, `→ report/36-...md` · *Kiểm chứng:* runtime

### 3.6 Nhóm thiết bị & cảm biến

| API | Mô tả | Yêu cầu | Nguồn |
|---|---|---|---|
| Barcode Detection | Quét mã | SHOULD | `report/34` |
| Geolocation | Vị trí | MUST-support | `report/43`, `report/18` |
| Accelerometer / Gyroscope / Magnetometer | Chuyển động | MAY | `report/35`, `report/46` |
| Device Posture | Trạng thái gập | MAY | `report/34` |
| Battery / Compute Pressure | Pin & tải | MAY | `report/41`, `report/63` |
| Screen Wake Lock | Giữ màn hình | MAY | `report/43`, `report/21` |
| Vibration | Rung | MAY | `report/34` |
| Gamepad | Tay cầm | MAY | `report/35` |
| Web Bluetooth / BLE | Bluetooth | MAY | `report/46`, `report/56` |
| Web NFC | NFC | MAY | `report/46` |
| Clipboard | Bảng tạm | SHOULD | `report/35`, `report/63` |
| File System Access | Tệp | SHOULD | `report/32` |

> **REQ-03-020** (MUST · C1) — Mọi truy cập cảm biến/phần cứng **PHẢI** tuân `SPEC-06`: ràng buộc foreground, lượng tử hóa tần số lấy mẫu, có chỉ báo riêng tư, và thu hồi được.
> *Nguồn:* `→ report/35-...md`, `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-03-021** (MAY · C3) — Quét mã vạch **NÊN** dùng `BarcodeDetector` phần cứng trên `ImageBitmap`/`VideoFrame`, **KHÔNG** để bộ đệm camera thô chạm JavaScript.
> *Nguồn:* `→ report/34-...md` · *Kiểm chứng:* review

> **REQ-03-022** (MAY · C3) — Nhận dạng tư thế/động tác **NÊN** dùng cảm biến đã lượng tử hóa; **KHÔNG** hứa hẹn tần số lấy mẫu phổ quát.
> *Nguồn:* `→ report/35-...md` · *Kiểm chứng:* review

### 3.7 Nhóm danh tính & bảo mật

| API | Mô tả | Yêu cầu | Nguồn |
|---|---|---|---|
| WebAuthn / Passkeys | Xác thực | SHOULD | `report/45`, `report/56` |
| WebAuthn PRF | Dẫn xuất khóa đối xứng | MAY | `report/45` |
| Digital Credentials API | Trình chiếu giấy tờ | MAY | `report/43` |
| `navigator.credentials` + FedCM | Liên kết danh tính | MAY | `report/25` |
| Cookie Store API | Cookie bất đồng bộ | SHOULD | `report/43` |
| SubtleCrypto / `CryptoKey` | Mã hóa | MUST-support | `report/43` |
| WebOTP | OTP gắn origin | MAY | `report/34` |
| Permissions API | Trạng thái quyền | MUST | `report/67` |

> **REQ-03-023** (MUST · C1) — Host **PHẢI** bộc lộ WebCrypto với `CryptoKey` **không xuất được** (non-extractable) cho khóa nhạy cảm; khóa phải nằm trong kho khóa cách ly theo `app_id` và bị xóa khi gỡ cài.
> *Nguồn:* `→ report/20-...md`, `→ report/43-...md` · *Kiểm chứng:* runtime

> **REQ-03-024** (SHOULD · C3) — Khóa bí mật dùng PRF của WebAuthn **NÊN** để dẫn xuất khóa phiên thay vì lưu khóa thô.
> *Nguồn:* `→ report/45-...md` · *Kiểm chứng:* review

> **REQ-03-025** (MUST · C1) — Trạng thái quyền **PHẢI** tra được qua `navigator.permissions.query()` và đồng bộ với trạng thái host qua `PermissionStatus.onchange`.
> *Nguồn:* `→ report/67-...md` · *Kiểm chứng:* runtime

### 3.8 Nhóm thương mại

| API | Mô tả | Yêu cầu | Nguồn |
|---|---|---|---|
| Digital Goods API | Danh mục SKU | SHOULD | `report/30` |
| Payment Request | Thanh toán | MUST-support | `report/20`, `report/89` |
| Payment Handler | Công cụ thanh toán SW | MAY | `report/38` |
| Window Management | POS đa màn hình | MAY | `report/38` |
| Presentation API | Trình chiếu không dây | MAY | `report/38` |

> **REQ-03-026** (MUST · C2) — Luồng mua hàng **PHẢI** qua `Payment Request` / `Payment Handler`; mini-app **KHÔNG ĐƯỢC** tự xử lý số thẻ.
> *Nguồn:* `→ report/20-...md`, `→ report/47-...md` · *Kiểm chứng:* static

> **REQ-03-027** (MUST · C2) — Quyền sở hữu số (entitlement) **PHẢI** có vòng đời `consume`/`acknowledge` tường minh để tránh bồi hoàn gian lận trong 3 ngày.
> *Nguồn:* `→ report/30-...md` · *Kiểm chứng:* runtime

### 3.9 Nhóm thông báo & vòng đời nền

| API | Mô tả | Yêu cầu | Nguồn |
|---|---|---|---|
| Notifications API | Thông báo | SHOULD | `report/24` |
| Badging API | Huy hiệu biểu tượng | MAY | `report/24`, `report/41` |
| Push (RFC 8030/8291/8292) | Đẩy | MAY | `report/24` |
| Background Sync / Periodic Sync / Fetch | Đồng bộ nền | MAY | `report/21`, `report/41` |
| Page Lifecycle | 6 trạng thái | MUST | `report/30` |
| `navigator.sendBeacon` | Gửi khi rời trang | SHOULD | `report/66` |
| ReportingObserver / `Reporting-Endpoints` | Báo cáo lỗi | SHOULD | `report/77`, `report/36` |

> **REQ-03-028** (MUST · C1) — Thông báo **PHẢI** được đăng ký dưới `NotificationChannel` riêng của từng mini-app trong nhóm "Mini Apps" để một mini-app spam không tắt toàn bộ thông báo host.
> *Nguồn:* `→ report/24-...md` · *Kiểm chứng:* runtime

> **REQ-03-029** (MUST · C1) — Tải nền (Background Fetch) **PHẢI** khai báo `downloadTotal`, có trần (ví dụ 150 MB), và hiển thị UI native có nút hủy.
> *Nguồn:* `→ report/41-...md` · *Kiểm chứng:* runtime

> **REQ-03-030** (MUST · C3) — Trước khi rời trang, mini-app **PHẢI** gọi `takeRecords()` của observer (nếu có) rồi `navigator.sendBeacon()` để không mất dữ liệu; đồng thời `disconnect()` để không rò rỉ.
> *Nguồn:* `→ report/77-...md` · *Kiểm chứng:* review

### 3.10 Hợp đồng lỗi chuẩn

> **REQ-03-031** (MUST · C1) — Lỗi API **PHẢI** tuân hợp đồng RFC 7807 (Problem Details) ở tầng HTTP và bản ánh xạ tương ứng ở tầng JSBridge, gồm `type`, `title`, `status`, `detail`, `instance`.
> *Nguồn:* `→ report/41-...md`, `→ report/59-...md` · *Kiểm chứng:* static

> **REQ-03-032** (MUST · C1) — Phân loại lỗi **PHẢI** phân biệt rõ: `permission_denied`, `not_supported`, `rate_limited`, `invalid_argument`, `resource_exhausted`, `internal`, `network_error`.
> *Nguồn:* `→ report/11-...md`, `→ report/16-...md` · *Kiểm chứng:* static

> **REQ-03-033** (MUST · C1) — Thao tác tạo/sửa có tác dụng phụ **PHẢI** hỗ trợ `Idempotency-Key` (UUIDv4) để chống lặp khi thử lại.
> *Nguồn:* `→ report/33-...md` · *Kiểm chứng:* runtime

---

## 4. Bảng mức hỗ trợ bắt buộc từ Host

| Nhóm | MUST-support | SHOULD | MAY |
|---|---|---|---|
| Giao tiếp & điều hướng | JSBridge.invoke, launchQueue, BroadcastChannel | MessageChannel, Navigation API | protocol_handlers |
| Mạng & dữ liệu | fetch (allowlist), IndexedDB | WebSocket, Cache API, OPFS, Compression | WebTransport, Storage Buckets |
| Giao diện & hiệu năng | IntersectionObserver | VirtualKeyboard, ResizeObserver, OffscreenCanvas | Anchor/Popover, scheduler.postTask |
| Media | getUserMedia, MediaRecorder | Web Audio, Media Session, ManagedMediaSource | WebXR, WebCodecs, PiP |
| Thiết bị | Geolocation | Clipboard, File System Access | Cảm biến, Bluetooth, NFC, Gamepad |
| Danh tính & bảo mật | WebCrypto, Permissions API | WebAuthn, Cookie Store | FedCM, Digital Credentials, PRF |
| Thương mại | Payment Request | Digital Goods | Payment Handler, Window Mgmt |
| Thông báo & nền | Page Lifecycle | Notifications, sendBeacon, ReportingObserver | Push, Background Sync |

---

## 5. Tiêu chí tuân thủ

- [ ] API surface là allowlist tường minh, fail-closed.
- [ ] Mọi API nhạy cảm kiểm tra 3 tập: khai báo ∩ cấp quyền ∩ policy.
- [ ] Toàn bộ API bất đồng bộ; không API đồng bộ chặn luồng.
- [ ] Lỗi có taxonomy ổn định, theo RFC 7807, không rò rỉ nội bộ.
- [ ] Giải phóng tài nguyên tường minh cho media/observer/worker.
- [ ] Host hỗ trợ đúng cột MUST-support ở bảng §4.

---

## 6. Cân nhắc an ninh & riêng tư

- **Không có API "ẩn":** mọi bề mặt phải được liệt kê và kiểm duyệt.
- **Không lộ stack nội bộ:** lỗi là mã + thông điệp chung, không có đường dẫn tệp host.
- **Giải phóng đúng hạn:** rò rỉ `VideoFrame`/`AudioContext`/`observer` là nguyên nhân sụp đổ bộ nhớ hàng đầu.
- **Giới hạn dữ liệu:** tham số nhạy cảm trong query/referrer không được ghi vào nhật ký (xem `SPEC-07`).

---

## 7. Tham chiếu

- W3C Permissions API — `report/67`
- WHATWG Web Workers / Streams — `report/85`, `report/67`
- IETF RFC 7807 Problem Details — `SPEC-13`
- OWASP MASVS v2.0 — `SPEC-04`
- Báo cáo nguồn: `report/16, 24, 25, 29, 30, 34, 35, 41, 43, 46, 47, 56, 63, 67, 77`
