# SPEC-10 — Độ tin cậy, Hiệu năng & Quản trị tài nguyên

| Trường | Giá trị |
|---|---|
| Spec ID | `SPEC-10` |
| Phiên bản | 1.0.0 |
| Trạng thái | Draft |
| Lớp tuân thủ | **C1** (Host / Container) |
| Nguồn | `report/09`, `report/11`, `report/17`, `report/21`, `report/27`, `report/29`, `report/30`, `→ report/33`, `report/63` |

---

## 1. Phạm vi & mục tiêu

Đặc tả **SLO, hạn ngạch, công bằng tài nguyên, chống lạm dụng, ngân sách
hiệu năng, phục hồi sự cố và vận hành nền**.

Mục tiêu: mini-app không được làm suy giảm trải nghiệm chung, không được
chiếm tài nguyên vô hạn, và mọi sự cố phải được phát hiện, phục hồi và học hỏi.

> **Chuẩn tham chiếu:** W3C Performance Timeline, Long Tasks API, Web Locks,
> Page Lifecycle, Compute Pressure, Background Sync/Fetch, OWASP guidance.

---

## 2. Thuật ngữ

| Thuật ngữ | Định nghĩa |
|---|---|
| **SLO** | Mục tiêu mức dịch (Service Level Objective). |
| **Error budget** | Ngân sách lỗi cho phép trong kỳ. |
| **INP** | Interaction to Next Paint — chỉ số phản hồi tương tác. |
| **LoAF** | Long Animation Frame — khung hình animation dài. |
| **ANR** | Application Not Responding — ứng dụng không phản hồi. |
| **RTO / RPO** | Mục tiêu thời gian phục hồi / mức mất dữ liệu chấp nhận. |
| **Load shedding** | Chủ động giảm tải khi quá tải. |
| **Entitlement** | Quyền vận hành đặc biệt (chạy nền, nghe nền). |

---

## 3. Yêu cầu chuẩn hóa

### 3.1 SLO & ngân sách lỗi

> **REQ-10-001** (MUST · C1) — Nền tảng **PHẢI** định nghĩa SLO cho các bề mặt: khởi chạy, tải trang, lời gọi bridge, thanh toán — gồm tính khả dụng, độ trễ và tỷ lệ không sụp đổ.
> *Nguồn:* `→ report/09-...md` · *Kiểm chứng:* review

> **REQ-10-002** (MUST · C1) — Ngân sách lỗi **PHẢI** được tính theo kỳ và hành vi vượt ngân sách phải ảnh hưởng quyết định phát hành (dừng phát hành tiếp).
> *Nguồn:* `→ report/09-...md` · *Kiểm chứng:* runtime

> **REQ-10-003** (SHOULD · C1) — Cảnh báo **NÊN** dùng tốc độ đốt ngân sách (burn-rate) thay vì ngưỡng tuyệt đối.
> *Nguồn:* `→ report/09-...md` · *Kiểm chứng:* runtime

> **REQ-10-004** (MUST · C1) — Lược đồ telemetry **PHẢI** được quản trị: có phiên bản, có kiểm tra tương thích ngược, không đổi tên trường âm thầm.
> *Nguồn:* `→ report/09-...md`, `→ report/27-...md` · *Kiểm chứng:* static

> **REQ-10-005** (SHOULD · C1) — Mục tiêu phục hồi **NÊN** có RTO/RPO tường minh cho từng lớp dịch vụ.
> *Nguồn:* `→ report/09-...md` · *Kiểm chứng:* review

### 3.2 Hạn ngạch & cách ly tài nguyên

> **REQ-10-006** (MUST · C1) — Mỗi mini-app **PHẢI** có hạn ngạch CPU, bộ nhớ, lưu trữ và mạng riêng; vượt hạn ngạch thì bị giảm tải hoặc dừng.
> *Nguồn:* `→ report/11-...md`, `→ report/17-...md` · *Kiểm chứng:* runtime

> **REQ-10-007** (MUST · C1) — Cách ly đa thuê **PHẢI** bảo đảm một mini-app **KHÔNG** làm suy giảm hiệu năng của mini-app khác.
> *Nguồn:* `→ report/11-...md` · *Kiểm chứng:* runtime

> **REQ-10-008** (MUST · C1) — Hạn ngạch **PHẢI** được quảng bá (advertised) qua API để mini-app tự điều chỉnh, kèm thông báo sắp hết hạn ngạch.
> *Nguồn:* `→ report/11-...md` · *Kiểm chứng:* runtime

> **REQ-10-009** (MUST · C1) — Khi hết tài nguyên (bộ nhớ, inode, handle), hệ thống **PHẢI** trả lỗi `resource_exhausted` tường minh, không sụp đổ im lặng.
> *Nguồn:* `→ report/11-...md` · *Kiểm chứng:* runtime

> **REQ-10-010** (SHOULD · C2) — Chi phí **NÊN** được gán cho từng tenant/mini-app, gồm phần chi phí dùng chung và nhàn rỗi để minh bạch.
> *Nguồn:* `→ report/11-...md` · *Kiểm chứng:* review

> **REQ-10-011** (MUST · C2) — Giới hạn chi tiêu cứng **PHẢI** được áp để tránh truy cập trái phép gây tốn kém vô hạn.
> *Nguồn:* `→ report/17-...md` · *Kiểm chứng:* runtime

### 3.3 Công bằng & chống lạm dụng

> **REQ-10-012** (MUST · C1) — Ứng xử quá tải **PHẢI** theo cấp độ (graded): giảm tải dần, ưu tiên làn quan trọng, tránh sụp đổ hàng loạt.
> *Nguồn:* `→ report/17-...md` · *Kiểm chứng:* runtime

> **REQ-10-013** (MUST · C1) — Giới hạn tốc độ **PHẢI** có hợp đồng phản hồi rõ ràng: mã lỗi, `Retry-After`, và ngữ nghĩa lùi bước (backoff) có thể đoán được.
> *Nguồn:* `→ report/11-...md` · *Kiểm chứng:* runtime

> **REQ-10-014** (MUST · C1) — Điều khiển chống lạm dụng **PHẢI** thích ứng (adaptive) theo tín hiệu rủi ro, không phải ngưỡng cứng duy nhất.
> *Nguồn:* `→ report/17-...md` · *Kiểm chứng:* runtime

> **REQ-10-015** (MUST · C1) — Lời gọi có tác dụng phụ **PHẢI** hỗ trợ `Idempotency-Key` (UUIDv4) để chống lặp khi thử lại.
> *Nguồn:* `→ report/33-...md`, `→ report/17-...md` · *Kiểm chứng:* runtime

> **REQ-10-016** (SHOULD · C2) — Tín hiệu gian lận **NÊN** được tổng hợp từ nhiều nguồn và dùng Privacy Pass/Private State Token thay cho thách thức xâm phạm riêng tư.
> *Nguồn:* `→ report/31-...md`, `→ report/17-...md` · *Kiểm chứng:* runtime

### 3.4 Ngân sách hiệu năng

> **REQ-10-017** (MUST · C1) — Chỉ số INP **PHẢI** được giữ ≤ 200 ms trên thiết bị tham chiếu; vượt thì mini-app bị cảnh báo khi review.
> *Nguồn:* `→ report/29-...md` · *Kiểm chứng:* runtime

> **REQ-10-018** (MUST · C1) — Tác vụ dài trên luồng chính **PHẢI** được phát hiện theo ngưỡng 50 ms (Long Tasks) và phân tích bằng LoAF / `PerformanceScriptTiming` để truy nguồn.
> *Nguồn:* `→ report/29-...md` · *Kiểm chứng:* runtime

> **REQ-10-019** (MUST · C1) — Gói chính **PHẢI** ≤ 4 MB và tổng gói ≤ 20 MB; thời gian hiển thị lần đầu phải nằm trong ngân sách đã định.
> *Nguồn:* `→ report/46-...md`, `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-10-020** (MUST · C1) — Dưới áp lực tính toán (`Compute Pressure` trạng thái `serious`/`critical`), hệ thống **PHẢI** giảm tải: giảm tốc khung hình, giảm độ phân giải, tạm dừng tác vụ nền.
> *Nguồn:* `→ report/63-...md`, `→ report/34-...md` · *Kiểm chứng:* runtime

> **REQ-10-021** (MUST · C1) — VRAM/bộ nhớ GPU **PHẢI** được thu hồi khi mini-app chuyển nền (`device.destroy()`), và `VideoFrame`/`ImageBitmap` phải được `.close()` tường minh.
> *Nguồn:* `→ report/26-...md` · *Kiểm chứng:* runtime

> **REQ-10-022** (SHOULD · C1) — Tải tài nguyên **NÊN** dùng Priority Hints (`fetchpriority`) và HTTP 103 Early Hints (RFC 8297) để giảm FCP.
> *Nguồn:* `→ report/33-...md`, `→ report/79-...md` · *Kiểm chứng:* runtime

### 3.5 Sụp đổ, ANR & chẩn đoán

> **REQ-10-023** (MUST · C1) — Container **PHẢI** bắt lỗi toàn cục (`window.onerror`, `unhandledrejection`) và ghi nhận mọi ngoại lệ chưa xử lý.
> *Nguồn:* `→ report/33-...md` · *Kiểm chứng:* runtime

> **REQ-10-024** (MUST · C1) — Watchdog **PHẢI** chạy trong worker/luồng riêng: ping luồng chính mỗi 500 ms; 5 nhịp liên tiếp trễ (~2.500 ms) thì ghi nhận ANR và cho phép tải lại.
> *Nguồn:* `→ report/29-...md`, `→ report/33-...md` · *Kiểm chứng:* runtime

> **REQ-10-025** (SHOULD · C1) — Symbolication **NÊN** dùng Source Map Rev 3 (Base64-VLQ) phía máy chủ để không lộ bản đồ mã nguồn cho client.
> *Nguồn:* `→ report/33-...md` · *Kiểm chứng:* review

> **REQ-10-026** (MUST · C1) — Breadcrumb chẩn đoán **PHẢI** là ring buffer cố định dung lượng, không PII.
> *Nguồn:* `→ report/33-...md` · *Kiểm chứng:* static

> **REQ-10-027** (SHOULD · C2) — Cổng chẩn đoán cho nhà phát triển **NÊN** có cơ chế đóng băng phát hành tự động khi tỷ lệ sụp đổ vượt ngưỡng.
> *Nguồn:* `→ report/33-...md` · *Kiểm chứng:* runtime

> **REQ-10-028** (MUST · C1) — Sụp đổ tiến trình render **PHẢI** được phục hồi bằng ảnh chụp trạng thái giao dịch, không mất dữ liệu người dùng đang làm.
> *Nguồn:* `→ report/27-...md`, `→ report/33-...md` · *Kiểm chứng:* runtime

### 3.6 Vận hành nền & năng lượng

> **REQ-10-029** (MUST · C1) — Khi ở nền, mini-app **PHẢI** bị đóng băng có tính quyết định (tạm dừng timer, WebGL, audio) trong 500 ms, trừ khi giữ entitlement tương ứng.
> *Nguồn:* `→ report/21-...md`, `→ report/30-...md` · *Kiểm chứng:* runtime

> **REQ-10-030** (MUST · C1) — Giới hạn cư trú nền **PHẢI** tường minh (ví dụ 5 phút); vượt thì unload và đánh dấu `document.wasDiscarded` để hydrate lại.
> *Nguồn:* `→ report/30-...md` · *Kiểm chứng:* runtime

> **REQ-10-031** (MUST · C1) — Áp lực bộ nhớ OS **PHẢI** được nối vào runtime (`onTrimMemory`, `didReceiveMemoryWarning`) để chủ động giải phóng.
> *Nguồn:* `→ report/30-...md` · *Kiểm chứng:* runtime

> **REQ-10-032** (MUST · C1) — Tải nền (Background Fetch) **PHẢI** khai báo `downloadTotal` (trần 150 MB) và hiển thị UI native có nút hủy.
> *Nguồn:* `→ report/41-...md` · *Kiểm chứng:* runtime

> **REQ-10-033** (SHOULD · C1) — Đồng bộ nền **NÊN** dùng Background Sync / Periodic Sync với hàng đợi thử lại có jitter để tránh bùng nổ đồng thời.
> *Nguồn:* `→ report/21-...md`, `→ report/41-...md` · *Kiểm chứng:* runtime

> **REQ-10-034** (MUST · C1) — Wake Lock **PHẢI** có ngân sách và tự giải phóng khi `visibilitychange`; pin dưới 15% thì tự động thu hồi.
> *Nguồn:* `→ report/21-...md`, `→ report/43-...md` · *Kiểm chứng:* runtime

### 3.7 Ngoại tuyến, cập nhật & phục hồi

> **REQ-10-035** (MUST · C1) — Chức năng ngoại tuyến **PHẢI** dùng nhật ký hai chiều (two-way journaling) với đồng hồ vector để giải quyết xung đột khi đồng bộ lại.
> *Nguồn:* `→ report/33-...md` · *Kiểm chứng:* runtime

> **REQ-10-036** (SHOULD · C1) — Đồng bộ trạng thái **NÊN** dùng JSON Patch (RFC 6902) / JSON Merge Patch (RFC 7396) để giảm 80–95% băng thông.
> *Nguồn:* `→ report/33-...md` · *Kiểm chứng:* runtime

> **REQ-10-037** (SHOULD · C1) — Thích ứng mạng **NÊN** dùng Network Information API (`effectiveType`, `downlink`, `saveData`) với lượng tử hóa để giảm entropy.
> *Nguồn:* `→ report/33-...md` · *Kiểm chứng:* runtime

> **REQ-10-038** (MUST · C1) — Cập nhật **PHẢI** được phát hành có kiểm soát với ngưỡng sức khỏe, tự động dừng, và rollback một chạm về digest tốt nhất đã biết.
> *Nguồn:* `→ report/09-...md`, `→ report/27-...md` · *Kiểm chứng:* review

> **REQ-10-039** (SHOULD · C1) — Cập nhật **NÊN** dùng delta (VCDIFF RFC 3284) + nén từ điển (Zstd RFC 8878) nhưng vẫn xác minh digest cuối.
> *Nguồn:* `→ report/27-...md` · *Kiểm chứng:* runtime

> **REQ-10-040** (MUST · C1) — Đồng thời **PHẢI** được điều phối bằng Web Locks với giới hạn thời gian giữ khóa và `AbortSignal`; watchdog thu hồi khóa quá hạn.
> *Nguồn:* `→ report/21-...md` · *Kiểm chứng:* runtime

---

## 4. Ghi chú triển khai (informative)

**Ngân sách hiệu năng đề xuất:**

| Chỉ số | Ngân sách | Hậu quả vượt |
|---|---|---|
| INP | ≤ 200 ms | Cảnh báo khi review |
| Long Task | ≤ 50 ms | Ghi nhận + truy nguồn |
| Gói chính | ≤ 4 MB | Từ chối phát hành |
| Tổng gói | ≤ 20 MB | Từ chối phát hành |
| Đóng băng nền | ≤ 500 ms | Vi phạm C1 |
| ANR | 5 nhịp trễ | Ghi nhận sự cố |

---

## 5. Tiêu chí tuân thủ

- [ ] SLO + ngân sách lỗi được định nghĩa và ảnh hưởng quyết định phát hành.
- [ ] Hạn ngạch CPU/RAM/storage/network per mini-app, có quảng bá hạn ngạch.
- [ ] Quá tải theo cấp độ; có hợp đồng lỗi + `Retry-After`.
- [ ] INP ≤ 200 ms; phát hiện Long Task/LoAF; gói ≤ 4 MB / 20 MB.
- [ ] Watchdog ANR hoạt động; breadcrumb không PII.
- [ ] Đóng băng nền ≤ 500 ms; giới hạn cư trú nền tường minh.
- [ ] Nhật ký ngoại tuyến hai chiều + delta sync + rollback một chạm.

---

## 6. Cân nhắc an ninh & riêng tư

- **Từ chối dịch vụ cục bộ:** một mini-app vòng lặp vô hạn có thể làm chậm cả host — watchdog + hạn ngạch là bắt buộc.
- **Rò rỉ qua chẩn đoán:** stack trace và breadcrumb không được chứa PII hay khóa.
- **Công bằng:** hạn ngạch không được phân biệt đối xử tùy tiện giữa các nhà phát triển.
- **Năng lượng:** tiêu hao pin quá mức là lỗi tuân thủ, không chỉ là vấn đề trải nghiệm.

---

## 7. Tham chiếu

- W3C Performance Timeline, Long Tasks API 1.0, LoAF
- W3C Web Locks, Page Lifecycle, Compute Pressure, Screen Wake Lock
- WICG Background Sync / Periodic Sync / Background Fetch
- IETF RFC 3284, RFC 8878, RFC 6902, RFC 7396, RFC 8297
- Báo cáo nguồn: `report/09, 11, 17, 21, 26, 27, 29, 30, 33, 41, 43, 63`
