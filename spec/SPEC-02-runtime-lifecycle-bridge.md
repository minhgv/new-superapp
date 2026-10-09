# SPEC-02 — Runtime, Vòng đời & Host Bridge

| Trường | Giá trị |
|---|---|
| Spec ID | `SPEC-02` |
| Phiên bản | 1.0.0 |
| Trạng thái | Draft |
| Lớp tuân thủ | **C1** (Host / Container) |
| Nguồn | `report/18-iteration-22-w3c-miniapp-tuf-device-governance.md`, `report/30-iteration-34-launch-lifecycle-digital-goods.md`, `report/21-iteration-25-concurrency-background-power-governance.md`, `report/16-iteration-19-host-bridge-cross-context.md`, `report/29-iteration-33-responsiveness-shadow-dom-protocols.md`, `report/27-iteration-31-multiprocess-delta-ota-tracing.md` |

---

## 1. Phạm vi & mục tiêu

Đặc tả **môi trường chạy**, **vòng đời** và **kênh giao tiếp host** của mini-app:
kiến trúc đa luồng, các trạng thái ứng dụng/trang, hợp đồng JSBridge, và cơ chế
cô lập giữa các ngữ cảnh.

Mục tiêu: bảo đảm mini-app chạy ổn định, dự đoán được, cách ly an toàn và
không thể leo thang đặc quyền qua kênh giao tiếp.

> **Chuẩn tham chiếu:** W3C MiniApp Lifecycle, W3C MiniApp Standardization
> White Paper v2 (kiến trúc đa luồng), W3C Web App Launch Handler / LaunchQueue,
> WICG Page Lifecycle, WHATWG Web Messaging.

---

## 2. Thuật ngữ

| Thuật ngữ | Định nghĩa |
|---|---|
| **View Layer** | Tầng hiển thị: WebView hoặc thành phần native, chỉ render. |
| **Logic Layer** | Tầng nghiệp vụ chạy trong JavaScript Worker (JSCore/V8). |
| **JSBridge** | Kênh có kiểm soát giữa Logic Layer và Host để gọi năng lực gốc. |
| **GlobalState** | Trạng thái vòng đời toàn ứng dụng. |
| **PageState** | Trạng thái vòng đời cấp trang. |
| **Agent cluster** | Cụm tác nhân cách ly (Origin-Agent-Cluster). |
| **Content world** | Không gian nội dung cách ly trên iOS (WKWebView). |
| **Watchdog** | Cơ chế giám sát nhịp tim chống treo (ANR). |

---

## 3. Yêu cầu chuẩn hóa

### 3.1 Kiến trúc đa luồng (Dual-Thread)

> **REQ-02-001** (MUST · C1) — Runtime **PHẢI** tách bạch **View Layer** (hiển thị) khỏi **Logic Layer** (nghiệp vụ chạy trong JavaScript Worker). Logic **KHÔNG ĐƯỢC** truy cập trực tiếp cây DOM của View.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-02-002** (MUST · C1) — Kênh giao tiếp View ↔ Logic **PHẢI** được thiết lập tường minh trước khi gửi dữ liệu khởi tạo ban đầu từ Logic sang View; trang được đánh dấu `ready` sau khi cập nhật giao diện hoàn tất.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-02-003** (SHOULD · C1) — Render **NÊN** không trạng thái (stateless) với trạng thái nằm ở worker, cho phép render song song và quản lý ngăn xếp trang gốc.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* review

> **REQ-02-004** (MUST · C1) — Mỗi mini-app **PHẢI** chạy trong ngữ cảnh cách ly riêng; hai mini-app trong cùng host **PHẢI** độc lập hoàn toàn (không dùng chung biến toàn cục, bộ nhớ, cửa hàng khóa).
> *Nguồn:* `→ report/18-...md`, `→ report/39-...md` · *Kiểm chứng:* runtime

> **REQ-02-005** (SHOULD · C1) — Host **NÊN** dùng kiến trúc đa tiến trình: tiến trình giao diện host, tiến trình môi giới dịch vụ nền, và tiến trình render mini-app cách ly giao tiếp qua IPC.
> *Nguồn:* `→ report/27-...md` · *Kiểm chứng:* runtime

> **REQ-02-006** (MUST · C1) — Sụp đổ tiến trình render **PHẢI** được bắt bằng callback (ví dụ `onRenderProcessGone` trên Android, `webViewWebContentProcessDidTerminate` trên iOS) và **PHẢI** tái tạo trạng thái phiên bằng ảnh chụp trạng thái giao dịch (transactional snapshot).
> *Nguồn:* `→ report/27-...md`, `→ report/33-...md` · *Kiểm chứng:* runtime

### 3.2 Vòng đời ứng dụng (Global Lifecycle)

> **REQ-02-007** (MUST · C1) — Runtime **PHẢI** triển khai đủ 5 trạng thái ứng dụng toàn cục: `launched`, `shown`, `hidden`, `error`, `unloaded`, bộc lộ qua `GlobalState`.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-02-008** (MUST · C1) — Runtime **PHẢI** gọi đủ các handler: `ongloballaunched`, `onglobalshown`, `onglobalhidden`, `onglobalerror`, `onglobalunloaded`.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-02-009** (MUST · C1) — Đóng hoặc đưa xuống nền **KHÔNG ĐƯỢC** hủy mini-app ngay lập tức; user agent **CÓ THỂ** unload sau đó theo tiêu chí tài nguyên/thời gian.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

### 3.3 Vòng đời trang (Page Lifecycle)

> **REQ-02-010** (MUST · C1) — Runtime **PHẢI** triển khai đủ 5 trạng thái trang: `loaded`, `ready`, `shown`, `hidden`, `unloaded`, bộc lộ qua `PageState`.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-02-011** (MUST · C1) — Runtime **PHẢI** gọi đủ các handler: `onpageloaded`, `onpageready`, `onpageshown`, `onpagehidden`, `onpageunloaded`.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-02-012** (MUST · C1) — Trình tự render đầu tiên **PHẢI** là: khởi tạo View + Logic → thiết lập kênh giao tiếp → gửi dữ liệu ban đầu từ Logic sang View → đánh dấu trang `ready` sau cập nhật giao diện.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-02-013** (MUST · C1) — Hai tầng trạng thái (toàn ứng dụng và trang) **PHẢI** được giữ tách bạch; trạng thái **PHẢI** được bảo toàn khi `hidden` nếu chính sách cho phép, và unload dưới áp lực tài nguyên.
> *Nguồn:* `→ report/18-...md`, `→ report/30-...md` · *Kiểm chứng:* runtime

> **REQ-02-014** (MAY · C1) — `PageInputObject.pageInputQuery` và `global input` (đường dẫn trang/referrer) **CÓ THỂ** được cung cấp cho trang; host **PHẢI** cảnh báo không lưu các giá trị này cục bộ và **KHÔNG ĐƯỢC** để tham số nhạy cảm ở dạng văn bản thuần.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-02-015** (SHOULD · C1) — Host **NÊN** dùng callback vòng đời cho treo/giải phóng tài nguyên có tính quyết định, phân tích cấp trang, báo cáo lỗi/sụp đổ và dọn cache.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* review

### 3.4 Khởi chạy & định tuyến

> **REQ-02-016** (MUST · C1) — Host **PHẢI** hỗ trợ `launch_handler` với chính sách `client_mode` và `window.launchQueue` để đệm tham số khởi chạy.
> *Nguồn:* `→ report/30-...md` · *Kiểm chứng:* runtime

> **REQ-02-017** (MUST · C1) — `LaunchParams` **PHẢI** bộc lộ `targetURL` và handle tệp khi khởi chạy bằng tệp; việc giải nén tệp phải nằm trong sandbox đường dẫn.
> *Nguồn:* `→ report/30-...md`, `→ report/32-...md` · *Kiểm chứng:* runtime

> **REQ-02-018** (MUST · C1) — Container **PHẢI** định tuyến tường minh giữa chế độ một phiên bản và đa phiên bản (single vs multi-instance) của cùng một mini-app.
> *Nguồn:* `→ report/30-...md` · *Kiểm chứng:* runtime

> **REQ-02-019** (MUST · C1) — Bộ phân giải protocol handler **PHẢI** ràng buộc origin và có cơ chế chống chiếm luồng điều hướng liên ứng dụng; `web+` scheme và token `%s` phải được xử lý theo quy tắc đăng ký chuẩn.
> *Nguồn:* `→ report/29-...md` · *Kiểm chứng:* runtime

> **REQ-02-020** (SHOULD · C1) — Khi một mini-app gọi liên ứng dụng, host **NÊN** cấp token chứng thực người gọi (caller attestation) để mini-app đích kiểm chứng nguồn gọi.
> *Nguồn:* `→ report/29-...md` · *Kiểm chứng:* runtime

### 3.5 JSBridge & giao tiếp liên ngữ cảnh

> **REQ-02-021** (MUST · C1) — Bề mặt JSBridge **PHẢI** là **danh sách cho phép tường minh** (allowlist). Chỉ các API được khai báo trong manifest **và** được cấp quyền mới được gọi; mặc định là từ chối (fail-closed).
> *Nguồn:* `→ report/16-...md`, `→ report/18-...md` · *Kiểm chứng:* static + runtime

> **REQ-02-022** (MUST · C1) — Mọi lời gọi bridge **PHẢI** có **ID tương quan** (correlation ID) để ghép yêu cầu/phản hồi bất đồng bộ; giao thức nên là JSON-RPC có kiểm soát.
> *Nguồn:* `→ report/16-...md` · *Kiểm chứng:* runtime

> **REQ-02-023** (MUST · C1) — Thông điệp liên ngữ cảnh **PHẢI** được ràng buộc **origin và người nhận** (recipient binding); chủ thể gửi **PHẢI** được xác thực trước khi chuyển tiếp.
> *Nguồn:* `→ report/16-...md` · *Kiểm chứng:* runtime

> **REQ-02-024** (MUST · C1) — Ngữ cảnh nhận **PHẢI** giới hạn Structured Clone: chặn Prototype Pollution, giới hạn độ sâu/kích thước, và **KHÔNG ĐƯỢC** cho phép truyền hàm hoặc prototype tùy ý.
> *Nguồn:* `→ report/16-...md` · *Kiểm chứng:* runtime

> **REQ-02-025** (MUST · C1) — Lỗi trả về qua bridge **PHẢI** là lỗi có cấu trúc (structured errors) với mã lỗi ổn định, không rò rỉ đường dẫn nội bộ, stack nội bộ hoặc thông tin host.
> *Nguồn:* `→ report/16-...md`, `→ report/33-...md` · *Kiểm chứng:* runtime

> **REQ-02-026** (MUST · C1) — Trên Android, `addJavascriptInterface` **PHẢI** được siết chặt: chỉ bộc lộ các phương thức được đánh dấu tường minh, chặn phản chiếu Java, chặn truy cập lớp hệ thống.
> *Nguồn:* `→ report/16-...md` · *Kiểm chứng:* static + runtime

> **REQ-02-027** (MUST · C1) — Trên iOS, các nội dung bridge **PHẢI** chạy trong **content world** cách ly của WKWebView; điều hướng và phản hồi (reply) phải được kiểm soát, không cho phép tràn sang chuỗi thông điệp ngoài.
> *Nguồn:* `→ report/16-...md` · *Kiểm chứng:* runtime

> **REQ-02-028** (SHOULD · C1) — Giao tiếp điểm-điểm **NÊN** dùng `MessageChannel`/`MessagePort` với cơ chế chuyển quyền (Transferable) sao chép-không, tránh nhân bản dữ liệu lớn.
> *Nguồn:* `→ report/16-...md`, `→ report/28-...md` · *Kiểm chứng:* runtime

> **REQ-02-029** (MUST · C1) — Việc ủy quyền năng lực qua bridge **PHẢI** đi qua `SPEC-06` (Permissions Policy + consent), **KHÔNG** được cấp trực tiếp từ lệnh gọi API.
> *Nguồn:* `→ report/16-...md`, `→ report/18-...md` · *Kiểm chứng:* review

### 3.6 Đa ngữ cảnh & đồng thời

> **REQ-02-030** (SHOULD · C1) — Các ngữ cảnh chia sẻ (Shared Worker) **NÊN** được điều phối theo hàng đợi có giới hạn, có watchdog chống deadlock và giới hạn thời gian giữ khóa (`AbortSignal` sau ngưỡng hết hạn).
> *Nguồn:* `→ report/58-...md`, `→ report/21-...md` · *Kiểm chứng:* runtime

> **REQ-02-031** (MUST · C1) — Khóa độc quyền giữ quá thời gian thực thi (ví dụ 5.000 ms) **PHẢI** bị watchdog thu hồi bằng `AbortSignal`; tác vụ bảo trì host **CÓ THỂ** dùng `steal: true` để tránh mini-app không hợp tác chặn dỡ trạng thái container.
> *Nguồn:* `→ report/21-...md` · *Kiểm chứng:* runtime

> **REQ-02-032** (SHOULD · C1) — Host **NÊN** triển khai watchdog nhịp tim: worker độc lập ping luồng chính mỗi 500 ms; 5 nhịp liên tiếp trễ (≈2.500 ms) thì ghi nhận ANR và cho phép người dùng tải lại thay vì đóng băng hoàn toàn.
> *Nguồn:* `→ report/29-...md`, `→ report/33-...md` · *Kiểm chứng:* runtime

### 3.7 Nền, đóng băng và bộ nhớ

> **REQ-02-033** (MUST · C1) — Khi mini-app chuyển sang `hidden`/`frozen`, container **PHẢI** đóng băng có tính quyết định: tạm dừng timer, WebGL và audio trong vòng 500 ms, trừ khi mini-app giữ quyền `background-audio` được khai báo và xác minh.
> *Nguồn:* `→ report/21-...md`, `→ report/32-...md` · *Kiểm chứng:* runtime

> **REQ-02-034** (MUST · C1) — Giới hạn cư trú nền **PHẢI** được áp tường minh (ví dụ 5 phút); vượt ngưỡng thì unload và ghi `document.wasDiscarded` để hydrate trạng thái khi mở lại.
> *Nguồn:* `→ report/30-...md` · *Kiểm chứng:* runtime

> **REQ-02-035** (MUST · C1) — Áp lực bộ nhớ hệ điều hành **PHẢI** được nối vào runtime (`onTrimMemory` trên Android, `didReceiveMemoryWarning` trên iOS) để chủ động giải phóng cache và unload.
> *Nguồn:* `→ report/30-...md` · *Kiểm chứng:* runtime

> **REQ-02-036** (MUST · C1) — Tách cụm tác nhân theo origin (`Origin-Agent-Cluster`) **PHẢI** được bật; khóa `document.domain` phải bị vô hiệu hóa để tránh làm lỏng ranh giới cách ly.
> *Nguồn:* `→ report/39-...md` · *Kiểm chứng:* runtime

---

## 4. Ghi chú triển khai (informative)

**Sơ đồ trạng thái:**

```text
Ứng dụng: launched ──▶ shown ⇄ hidden ──▶ unloaded
                          │
                          └──▶ error ──▶ unloaded

Trang:    loaded ──▶ ready ──▶ shown ⇄ hidden ──▶ unloaded
```

**Nguyên tắc thiết kế bridge:** mỗi API bridge là một "cửa đặc quyền" —
không có cửa nào mở sẵn. Mọi lời gọi đi qua bốn bước: (1) kiểm tra allowlist,
(2) kiểm tra quyền đã cấp, (3) kiểm tra trạng thái foreground, (4) ghi nhật ký
tương quan. Bỏ qua bất kỳ bước nào là lỗi tuân thủ.

**Lưu ý tương thích:** các host khác nhau (WeChat, Alipay, WindVane) đặt tên
API bridge khác nhau (`wx.*`, `my.*`, `WVJBridge`). Đặc tả này chuẩn hóa
**hợp đồng hành vi**, không chuẩn hóa tên namespace — việc ánh xạ tên là
trách nhiệm của lớp SDK.

---

## 5. Tiêu chí tuân thủ

- [ ] View/Logic tách luồng; Logic không chạm DOM trực tiếp.
- [ ] Đủ 5 trạng thái ứng dụng + 5 trạng thái trang, đủ 10 handler.
- [ ] Trình tự render đầu tiên đúng thứ tự, `ready` sau cập nhật giao diện.
- [ ] JSBridge là allowlist, fail-closed, có correlation ID.
- [ ] Mọi thông điệp ràng buộc origin + người nhận; giới hạn Structured Clone.
- [ ] Android/iOS bridge hardening theo §3.5.
- [ ] Đa tiến trình + phục hồi trạng thái khi render crash.
- [ ] Đóng băng nền ≤ 500 ms; giới hạn cư trú nền tường minh.
- [ ] Watchdog ANR hoạt động; nhật ký tương quan đầy đủ.
- [ ] Cách ly agent cluster theo origin; khóa `document.domain` bị chặn.

---

## 6. Cân nhắc an ninh & riêng tư

- **Leo thang đặc quyền:** bề mặt bridge là vector tấn công số một — luôn fail-closed.
- **Rò rỉ referrer/query:** tham số điều hướng có thể chứa mã định danh người dùng; tham chiếu `SPEC-07`.
- **Chặn luồng điều hướng:** protocol handler và deep-link cần chứng thực người gọi.
- **Sụp đổ có kiểm soát:** phục hồi trạng thái phải không tải lại dữ liệu nhạy cảm đã bị đóng băng.
- **Đồng thời:** deadlock worker có thể trở thành từ chối dịch vụ cục bộ; watchdog là bắt buộc.

---

## 7. Tham chiếu

- W3C MiniApp Lifecycle — https://w3c.github.io/miniapp-lifecycle/
- W3C MiniApp White Paper v2 — https://w3c.github.io/miniapp-white-paper/
- WICG Web App Launch Handler — `report/30`
- WICG Page Lifecycle — `report/30`
- WHATWG Web Messaging — `report/16`
- OWASP MASVS-PLATFORM / MASVS-RESILIENCE
- Báo cáo nguồn: `report/18`, `report/30`, `report/21`, `report/16`, `report/29`, `report/27`, `report/33`
