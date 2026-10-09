# SPEC-06 — Năng lực & Phân quyền thiết bị

| Trường | Giá trị |
|---|---|
| Spec ID | `SPEC-06` |
| Phiên bản | 1.0.0 |
| Trạng thái | Draft |
| Lớp tuân thủ | **C1** (Host / Container) |
| Nguồn | `report/18`, `report/25`, `report/35`, `report/34`, `report/43`, `report/46`, `report/56`, `report/63`, `report/67`, `report/20` |

---

## 1. Phạm vi & mục tiêu

Đặc tả **mô hình ủy quyền năng lực**: cách mini-app yêu cầu, được cấp,
sử dụng và bị thu hồi quyền truy cập thiết bị.

Mục tiêu: quyền hạn phải **minh bạch, tối thiểu, thu hồi được, ràng buộc
foreground**, và có nhật ký kiểm toán đầy đủ.

> **Chuẩn tham chiếu:** W3C Permissions Policy, W3C Permissions, W3C Geolocation,
> Media Capture and Streams, Generic Sensor, Web Bluetooth, Web NFC,
> Screen Wake Lock, Battery Status.

---

## 2. Thuật ngữ

| Thuật ngữ | Định nghĩa |
|---|---|
| **Declared capability** | Năng lực khai báo trong manifest (`req_permissions`). |
| **Effective capability** | Giao của khai báo ∩ cấp quyền ∩ policy. |
| **Runtime capability** | Năng lực thực sự khả dụng khi chạy. |
| **Capability envelope** | Phong bì quyền: header → iframe allow → consent → liveness → revocation. |
| **Liveness gate** | Cổng ràng buộc đang ở foreground khi dùng năng lực. |
| **Quantization** | Lượng tử hóa dữ liệu cảm biến để giảm entropy. |

---

## 3. Yêu cầu chuẩn hóa

### 3.1 Mô hình phân lớp quyền

> **REQ-06-001** (MUST · C1) — Mỗi mini-app **PHẢI** duy trì ba tập năng lực riêng biệt: **declared**, **effective** và **runtime**; ba tập này phải tra cứu được theo `release_id`.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-06-002** (MUST · C1) — Permissions Policy **PHẢI** là cổng chặn **trước** khi xin đồng ý người dùng: header `Permissions-Policy` của tài liệu và thuộc tính `allow` của iframe phải được sinh từ tập effective.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-06-003** (MUST · C1) — Các tính năng nhạy cảm **MẶC ĐỊNH PHẢI** là `()` (không ai được dùng); **TUYỆT ĐỐI KHÔNG** dùng `*` cho truy cập thiết bị đặc quyền.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-06-004** (MUST · C1) — Chính sách của cha và chính sách container **PHẢI** cùng được đánh giá; iframe **KHÔNG THỂ** cấp tính năng mà header cha đã từ chối.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-06-005** (MUST · C1) — Khi chính sách, origin hoặc định danh gói không khớp, hệ thống **PHẢI** thất bại theo hướng an toàn (fail-closed) và ghi nhận nguyên nhân.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-06-006** (MUST · C1) — Đồng ý của trình duyệt **KHÔNG** được thay thế cho định danh gói host hay ủy quyền bridge; cả hai đều bắt buộc.
> *Nguồn:* `→ report/18-...md`, `→ report/25-...md` · *Kiểm chứng:* review

> **REQ-06-007** (MUST · C1) — Khung ủy quyền **PHẢI** có đủ 5 lớp: `Permissions Policy envelope → host consent UX → foreground liveness gate → throttling → revocation`.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-06-008** (SHOULD · C1) — Chính sách **NÊN** gắn với đúng origin HTTPS/release; khung (frame) được ủy quyền phải bị dỡ khi điều hướng hoặc cách ly.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-06-009** (MUST · C1) — Trạng thái quyền **PHẢI** tra được qua `navigator.permissions.query()` và đồng bộ với host qua `PermissionStatus.onchange`; khi tính năng bị từ chối thì **KHÔNG** được hiển thị lời nhắc người dùng.
> *Nguồn:* `→ report/67-...md` · *Kiểm chứng:* runtime

### 3.2 Vị trí (Geolocation)

> **REQ-06-010** (MUST · C1) — Truy cập vị trí **PHẢI** hiển thị cho người dùng, **ràng buộc foreground**, và giới hạn độ chính xác theo nhu cầu đã khai báo.
> *Nguồn:* `→ report/18-...md`, `→ report/43-...md` · *Kiểm chứng:* runtime

> **REQ-06-011** (SHOULD · C1) — Chế độ độ chính xác cao **NÊN** có ngân sách riêng và phải được cấp lại khi hết ngân sách; nền tảng **NÊN** ưu tiên vị trí gần đúng khi mini-app chỉ cần mức thành phố.
> *Nguồn:* `→ report/43-...md` · *Kiểm chứng:* runtime

> **REQ-06-012** (MAY · C3) — Mini-app **NÊN** dùng `getCurrentPosition` cho thao tác một lần và `watchPosition` chỉ khi cần theo dõi liên tục; phải gọi `clearWatch` khi không cần.
> *Nguồn:* `→ report/43-...md` · *Kiểm chứng:* review

### 3.3 Camera & Microphone

> **REQ-06-013** (MUST · C1) — Camera/microphone **PHẢI** được cấp theo từng mini-app, **thu hồi được**, và có chỉ báo riêng tư hệ thống khi đang thu.
> *Nguồn:* `→ report/18-...md`, `→ report/56-...md` · *Kiểm chứng:* runtime

> **REQ-06-014** (MUST · C1) — Thu âm/hình **PHẢI** bị dừng và giải phóng track khi mini-app chuyển `hidden`, trừ khi giữ entitlement tường minh.
> *Nguồn:* `→ report/36-...md` · *Kiểm chứng:* runtime

> **REQ-06-015** (MAY · C3) — Quét mã **NÊN** dùng camera của host (host-mediated) thay vì cho mini-app truy cập luồng camera thô.
> *Nguồn:* `→ report/34-...md` · *Kiểm chứng:* review

### 3.4 Bluetooth & NFC

> **REQ-06-016** (MUST · C1) — Web Bluetooth **PHẢI** bị giới hạn ở thiết bị do **người dùng chọn** và các dịch vụ GATT đã khai báo (`optionalServices`); **KHÔNG** quét tự động nền.
> *Nguồn:* `→ report/46-...md`, `→ report/56-...md` · *Kiểm chứng:* runtime

> **REQ-06-017** (MUST · C1) — Web NFC **PHẢI** chỉ hoạt động khi visible, chỉ NDEF, và **đọc trước khi ghi**; phải giải phóng NFC khi rời trang.
> *Nguồn:* `→ report/46-...md` · *Kiểm chứng:* runtime

> **REQ-06-018** (SHOULD · C1) — Quét BLE nền **NÊN** bị chặn hoặc yêu cầu entitlement riêng; dữ liệu quét không được dùng để định danh người dùng.
> *Nguồn:* `→ report/56-...md` · *Kiểm chứng:* runtime

### 3.5 Cảm biến & dữ liệu chuyển động

> **REQ-06-019** (MUST · C1) — Cảm biến chuyển động **PHẢI** bị ràng buộc foreground và có tần số lấy mẫu bị kẹp (ví dụ 20–30 Hz) để chống nghe lén kênh phụ qua âm thanh.
> *Nguồn:* `→ report/35-...md` · *Kiểm chứng:* runtime

> **REQ-06-020** (MUST · C1) — Dữ liệu cảm biến **PHẢI** được lượng tử hóa và/hoặc thêm nhiễu (gaussian noise) để triệt tiêu suy luận phím bấm, vị trí trong nhà và kênh phản xạ màn hình.
> *Nguồn:* `→ report/35-...md` · *Kiểm chứng:* runtime

> **REQ-06-021** (MUST NOT · C1) — **KHÔNG** được hứa hẹn tần số lấy mẫu phổ quát; mọi ngưỡng phải là giá trị tối đa cam kết, có thể thấp hơn theo tải hệ thống.
> *Nguồn:* `→ report/18-...md`, `→ report/35-...md` · *Kiểm chứng:* runtime

> **REQ-06-022** (SHOULD · C1) — Ánh sáng môi trường **NÊN** được lượng tử hóa theo bậc (ví dụ 50 lux) để chống kênh phụ phản xạ màn hình.
> *Nguồn:* `→ report/35-...md` · *Kiểm chứng:* runtime

### 3.6 Pin, tải & giữ màn hình

> **REQ-06-023** (MUST · C1) — `getBattery()` **PHẢI** được ảo hóa qua host bridge: làm tròn mức pin theo bước 5–10%, **bỏ** thời gian sạc ước tính.
> *Nguồn:* `→ report/21-...md` · *Kiểm chứng:* runtime

> **REQ-06-024** (MUST · C1) — Khi pin dưới 15%, hệ thống **PHẢI** tự giảm tải (throttle): giảm tốc khung hình, giảm tần suất mạng, ngừng hoạt động nền không thiết yếu.
> *Nguồn:* `→ report/21-...md` · *Kiểm chứng:* runtime

> **REQ-06-025** (MUST · C1) — Screen Wake Lock **PHẢI** hiển thị, thu hồi được, có ngân sách, và tự giải phóng khi `visibilitychange`.
> *Nguồn:* `→ report/43-...md`, `→ report/21-...md` · *Kiểm chứng:* runtime

> **REQ-06-026** (SHOULD · C1) — `PressureObserver` **NÊN** được dùng để điều tiết tải theo 4 trạng thái (`nominal`, `fair`, `serious`, `critical`) với `sampleInterval` ≥ 1000 ms.
> *Nguồn:* `→ report/63-...md` · *Kiểm chứng:* runtime

> **REQ-06-027** (MUST · C1) — `takeRecords()` **PHẢI** được gọi trước `disconnect()` để giải phóng handle giám sát nhiệt của hệ điều hành.
> *Nguồn:* `→ report/63-...md`, `→ report/77-...md` · *Kiểm chứng:* runtime

### 3.7 Đầu vào & bảng tạm

> **REQ-06-028** (MUST · C1) — Clipboard **PHẢI** yêu cầu kích hoạt người dùng nhất thời (transient activation), hiển thị thông báo khi dán, và **chống** quét nền trộm.
> *Nguồn:* `→ report/35-...md`, `→ report/63-...md` · *Kiểm chứng:* runtime

> **REQ-06-029** (MUST · C1) — Định dạng tùy biến clipboard **PHẢI** dùng tiền tố `web ` trong `ClipboardItem`, kiểm tra `ClipboardItem.supports()` trước khi dùng.
> *Nguồn:* `→ report/63-...md` · *Kiểm chứng:* runtime

> **REQ-06-030** (MUST · C1) — Pointer Lock **PHẢI** đảm bảo cử chỉ thoát không thể chặn (un-interceptable host escape gesture) để người dùng không bị giam chuột.
> *Nguồn:* `→ report/35-...md` · *Kiểm chứng:* runtime

> **REQ-06-031** (MUST · C1) — Bàn phím ảo **PHẢI** được trung gian bởi host bảo mật; **KHÔNG** cho phép mini-app ghi lại phím IME của hệ thống.
> *Nguồn:* `→ report/35-...md`, `→ report/77-...md` · *Kiểm chứng:* runtime

> **REQ-06-032** (SHOULD · C1) — Gamepad **NÊN** có polling cách ly, che GUID tay cầm và trung gian phản hồi rung.
> *Nguồn:* `→ report/35-...md` · *Kiểm chứng:* runtime

### 3.8 Danh tính & ngoại vi cấp cao

> **REQ-06-033** (MUST · C1) — Tính năng `identity-credentials-get` **MẶC ĐỊNH** là `'self'`; iframe chéo origin **KHÔNG** được gọi `navigator.credentials.get({identity})` nếu không có ủy quyền tường minh.
> *Nguồn:* `→ report/25-...md` · *Kiểm chứng:* runtime

> **REQ-06-034** (SHOULD · C1) — Danh tính khách (guest PPID) **NÊN** được thu hẹp phạm vi theo mini-app, không dùng chung định danh giữa các mini-app.
> *Nguồn:* `→ report/25-...md` · *Kiểm chứng:* runtime

> **REQ-06-035** (MUST · C1) — Trình chiếu thông tin xác thực số **PHẢI** dùng tiết lộ chọn lọc và ràng buộc `SessionTranscript` nonce để chống phát lại.
> *Nguồn:* `→ report/43-...md` · *Kiểm chứng:* runtime

> **REQ-06-036** (MUST · C1) — WebUSB / WebHID / Web Serial **PHẢI** theo ma trận phần cứng 3 tầng, cách ly sandbox và áp các biện pháp chống khai thác sandbox đã biết.
> *Nguồn:* `→ report/25-...md` · *Kiểm chứng:* runtime

### 3.9 Nhật ký kiểm toán quyền

> **REQ-06-037** (MUST · C1) — Mọi biến cố cấp/từ chối/thu hồi quyền **PHẢI** được ghi nhật ký với ID tương quan, thời điểm, `app_id`, `release_id`, lý do và chủ thể ra quyết định.
> *Nguồn:* `→ report/18-...md`, `→ report/09-...md` · *Kiểm chứng:* static

> **REQ-06-038** (MUST · C1) — Nhật ký quyền **KHÔNG ĐƯỢC** chứa dữ liệu nhạy cảm ở dạng văn bản thuần (tọa độ chính xác, nội dung clipboard, ảnh).
> *Nguồn:* `→ report/18-...md`, `→ report/33-...md` · *Kiểm chứng:* static

> **REQ-06-039** (MUST · C1) — Thu hồi quyền **PHẢI** có hiệu lực ngay lập tức trên mọi phiên đang mở và **PHẢI** phát tín hiệu qua Shared Signals để thu hồi liên hoàn.
> *Nguồn:* `→ report/53-...md` · *Kiểm chứng:* runtime

---

## 4. Ghi chú triển khai (informative)

**Ba tập năng lực — ví dụ:**

| Năng lực | Declared | Effective | Runtime | Ghi chú |
|---|---|---|---|---|
| `camera` | ✔ | ✔ | ✔ | Chỉ khi foreground, có chỉ báo |
| `geolocation` | ✔ | ✔ (near) | ✔ | Không xin độ chính xác cao |
| `bluetooth` | ✘ | ✘ | ✘ | Không khai báo → không hiện |
| `background-audio` | ✔ | ✘ | ✘ | Bị từ chối khi review |

**Nguyên tắc tối thiểu quyền:** khai báo thừa cũng bị từ chối khi review.
Khai báo thiếu nhưng code gọi → lỗi runtime tường minh, không âm thầm cấp.

---

## 5. Tiêu chí tuân thủ

- [ ] 3 tập năng lực (declared/effective/runtime) được duy trì và tra cứu được.
- [ ] Permissions Policy là cổng trước consent; mặc định `()`, không `*`.
- [ ] Ràng buộc foreground cho mọi năng lực nhạy cảm.
- [ ] Lượng tử hóa + kẹp tần số cho cảm chuyển động, pin, ánh sáng.
- [ ] Chỉ báo riêng tư cho camera/mic/vị trí/Bluetooth/NFC.
- [ ] Thu hồi có hiệu lực ngay + phát tín hiệu Shared Signals.
- [ ] Nhật ký kiểm toán quyền đầy đủ, không chứa PII.

---

## 6. Cân nhắc an ninh & riêng tư

- **Kênh phụ (side-channel):** cảm biến có thể suy luận phím bấm, vị trí, nội dung màn hình — lượng tử hóa là bắt buộc.
- **Ủy quyền chồng chéo:** ủy quyền bridge **không** được vượt qua Permissions Policy.
- **Đồng ý phải cụ thể:** không dùng đồng ý chung chung cho nhiều năng lực.
- **Trẻ em:** ràng buộc năng lực nghiêm ngặt hơn — tham chiếu `SPEC-11`.

---

## 7. Tham chiếu

- W3C Permissions Policy — https://w3c.github.io/webappsec-permissions-policy/
- W3C Permissions — https://w3c.github.io/permissions/
- W3C Geolocation, Media Capture, Generic Sensors, Screen Wake Lock
- Web Bluetooth / Web NFC (W3C CG)
- Báo cáo nguồn: `report/18, 20, 25, 34, 35, 43, 46, 56, 63, 67`
