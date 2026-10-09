# SPEC-07 — Riêng tư & Quản trị dữ liệu

| Trường | Giá trị |
|---|---|
| Spec ID | `SPEC-07` |
| Phiên bản | 1.0.0 |
| Trạng thái | Draft |
| Lớp tuân thủ | **C1 + C2** |
| Nguồn | `report/08`, `report/28`, `report/31`, `report/33`, `report/42`, `report/48`, `report/53`, `report/27`, `report/32` |

---

## 1. Phạm vi & mục tiêu

Đặc tả cách dữ liệu được **thu thập, phân vùng, làm sạch, lưu giữ, đo lường
và xóa** trong hệ sinh thái mini-app.

Mục tiêu: dữ liệu tối thiểu, mục đích rõ ràng, thu hồi được, và không
định danh lại được (re-identification) từ dữ liệu phân tích.

> **Chuẩn tham chiếu:** W3C Storage Partitioning, PrivacyCG Storage Access API,
> IETF CHIPS, WICG Attribution Reporting, Private State Tokens / Privacy Pass,
> IAB TCF v2.2 / GPP, Global Privacy Control, GDPR/OWASP PII guidance.

---

## 2. Thuật ngữ

| Thuật ngữ | Định nghĩa |
|---|---|
| **Data minimization** | Chỉ thu thập dữ liệu thật sự cần cho mục đích đã nêu. |
| **Purpose limitation** | Không dùng dữ liệu cho mục đích ngoài phạm vi đồng ý. |
| **Storage partitioning** | Phân vùng bộ nhớ theo origin/mini-app. |
| **PII scrubbing** | Loại bỏ thông tin định danh cá nhân trước khi xuất. |
| **Differential privacy** | Thêm nhiễu xác suất để chống định danh cá thể. |
| **Contribution budget** | Ngân sách đóng góp (Private Aggregation). |
| **DSAR** | Yêu cầu của chủ thể dữ liệu (access/erasure/portability). |

---

## 3. Yêu cầu chuẩn hóa

### 3.1 Nguyên tắc nền tảng

> **REQ-07-001** (MUST · C1+C2) — Hệ thống **PHẢI** tuân thu thập tối thiểu, giới hạn mục đích, giới hạn lưu trữ và riêng tư theo mặc định.
> *Nguồn:* `→ report/08-...md` · *Kiểm chứng:* review

> **REQ-07-002** (MUST · C2) — Đồng ý **PHẢI** là quyền được thu hồi; thu hồi phải có hiệu lực ngay trên mọi bề mặt đang xử lý dữ liệu đó.
> *Nguồn:* `→ report/08-...md`, `→ report/53-...md` · *Kiểm chứng:* runtime

> **REQ-07-003** (MUST · C1) — Tham số `query`/`referrer` nhạy cảm **KHÔNG ĐƯỢC** lưu ở dạng văn bản thuần trong nhật ký, phân tích hoặc bộ nhớ bền.
> *Nguồn:* `→ report/18-...md`, `→ report/33-...md` · *Kiểm chứng:* static

> **REQ-07-004** (MUST · C2) — Tuyên bố về dữ liệu (data-use summary) trong listing **PHẢI** khớp với hành vi thực tế của mini-app; lệch thì bị đình chỉ.
> *Nguồn:* `→ report/08-...md`, `→ report/12-...md` · *Kiểm chứng:* review

### 3.2 Phân vùng & làm sạch lưu trữ

> **REQ-07-005** (MUST · C1) — Bộ nhớ **PHẢI** được phân vùng theo mini-app/origin; mini-app A **KHÔNG** được đọc dữ liệu của mini-app B.
> *Nguồn:* `→ report/28-...md` · *Kiểm chứng:* runtime

> **REQ-07-006** (SHOULD · C1) — Cookie **NÊN** dùng CHIPS (partitioned double-keying) để không dùng chung giữa các ngữ cảnh.
> *Nguồn:* `→ report/28-...md` · *Kiểm chứng:* runtime

> **REQ-07-007** (MUST · C1) — Việc ủy quyền truy cập lưu trữ chéo origin **PHẢI** qua Storage Access API / `requestStorageAccessFor`, có đồng ý tường minh và có hạn.
> *Nguồn:* `→ report/28-...md` · *Kiểm chứng:* runtime

> **REQ-07-008** (MUST · C1) — Header `Clear-Site-Data` **PHẢI** được hỗ trợ để xóa sạch dữ liệu khi đăng xuất/thu hồi.
> *Nguồn:* `→ report/28-...md` · *Kiểm chứng:* runtime

> **REQ-07-009** (MUST · C1) — Khi gỡ cài/xóa dữ liệu, hệ thống **PHẢI** xóa an toàn dữ liệu còn sót (OWASP MASVS-STORAGE residual) và **zeroize** khóa mật mã.
> *Nguồn:* `→ report/28-...md`, `→ report/20-...md` · *Kiểm chứng:* runtime

> **REQ-07-010** (SHOULD · C1) — Hạn ngạch lưu trữ **NÊN** có chính sách thu hồi (LRU/TTL); `navigator.storage.persist()` chỉ cấp cho nhu cầu có cơ sở.
> *Nguồn:* `→ report/33-...md`, `→ report/49-...md` · *Kiểm chứng:* runtime

> **REQ-07-011** (MUST · C1) — Nội dung tải về **KHÔNG ĐƯỢC** giữ lại ngoài mục đích đã nêu; tệp tạm phải được dọn theo vòng đời.
> *Nguồn:* `→ report/32-...md` · *Kiểm chứng:* runtime

### 3.3 Telemetry & quan sát

> **REQ-07-012** (MUST · C1+C2) — Mọi dữ liệu phân tích **PHẢI** được scrub PII trước khi rời thiết bị/máy chủ xử lý.
> *Nguồn:* `→ report/27-...md`, `→ report/33-...md` · *Kiểm chứng:* static

> **REQ-07-013** (MUST · C1) — Breadcrumb chẩn đoán **PHẢI** là bộ đệm vòng (ring buffer) cố định dung lượng và **không** chứa PII.
> *Nguồn:* `→ report/33-...md` · *Kiểm chứng:* static

> **REQ-07-014** (SHOULD · C1) — Nhịp quan sát **NÊN** dùng Performance Timeline / Paint Timing / LCP / INP / Event Timing / Resource Timing; đồng hồ **NÊN** được làm thô (jitter/coarsening) để chống fingerprinting.
> *Nguồn:* `→ report/33-...md`, `→ report/23-...md` · *Kiểm chứng:* runtime

> **REQ-07-015** (MUST · C1) — Truy xuất thời gian tải tài nguyên **PHẢI** được kiểm soát bằng `Timing-Allow-Origin` để chống rò rỉ kênh phụ XS-Leaks.
> *Nguồn:* `→ report/65-...md` · *Kiểm chứng:* runtime

> **REQ-07-016** (SHOULD · C1) — Truyền ngữ cảnh OpenTelemetry qua JSBridge **NÊN** được redaction; **KHÔNG** truyền định danh thô vào header trace.
> *Nguồn:* `→ report/27-...md` · *Kiểm chứng:* runtime

> **REQ-07-017** (MUST · C2) — Chính sách lưu giữ (retention) **PHẢI** được định nghĩa theo loại dữ liệu và **PHẢI** được thực thi tự động bằng lịch xóa.
> *Nguồn:* `→ report/08-...md` · *Kiểm chứng:* review

> **REQ-07-018** (MUST · C2) — Người dùng **PHẢI** có khả năng xuất/xóa dữ liệu (DSAR) theo quy định; quy trình phải có thời hạn xử lý.
> *Nguồn:* `→ report/08-...md` · *Kiểm chứng:* review

### 3.4 Đo lường & quảng cáo tôn trọng riêng tư

> **REQ-07-019** (SHOULD · C2) — Đo lường chuyển đổi **NÊN** dùng Attribution Reporting API với phản hồi ngẫu nhiên (differential privacy) và báo cáo tổng hợp qua TEE/HPKE.
> *Nguồn:* `→ report/31-...md` · *Kiểm chứng:* runtime

> **REQ-07-020** (MUST · C2) — **KHÔNG** dùng định danh chéo trang (third-party cookie, fingerprint) để đo lường; mọi đo lường phải đi qua cơ chế bảo toàn riêng tư.
> *Nguồn:* `→ report/31-...md`, `→ report/48-...md` · *Kiểm chứng:* review

> **REQ-07-021** (SHOULD · C1) — Chống lạm dụng **NÊN** dùng Private State Tokens / Privacy Pass (RFC 9576/9577/9578) thay cho thách thức xâm phạm riêng tư.
> *Nguồn:* `→ report/31-...md` · *Kiểm chứng:* runtime

> **REQ-07-022** (MAY · C2) — Đấu giá quảng cáo **NÊN** chạy trên thiết bị (Protected Audience) trong worklet cách ly; kết quả ra vào qua Fenced Frames với `disableUntrustedNetwork`.
> *Nguồn:* `→ report/31-...md` · *Kiểm chứng:* runtime

> **REQ-07-023** (MUST · C2) — Private Aggregation **PHẢI** tuân ngân sách đóng góp (contribution budget) để chống suy luận cá thể từ histogram.
> *Nguồn:* `→ report/31-...md` · *Kiểm chứng:* runtime

> **REQ-07-024** (SHOULD · C2) — Topics API **NÊN** dùng phân loại v2 và phải tôn trọng tín hiệu `Sec-Browsing-Topics`; không suy luận chủ đề từ hành vi hẹp.
> *Nguồn:* `→ report/48-...md` · *Kiểm chứng:* runtime

> **REQ-07-025** (MUST · C2) — Nền tảng **PHẢI** hỗ trợ Global Privacy Control (`Sec-GPC: 1`, `navigator.globalPrivacyControl`, `/.well-known/gpc.json`) và tuân thủ TCF v2.2 / GPP khi phục vụ quảng cáo.
> *Nguồn:* `→ report/42-...md` · *Kiểm chứng:* runtime

> **REQ-07-026** (MUST NOT · C2) — **KHÔNG** dùng cơ sở "legitimate interest" cho lập hồ sơ quảng cáo theo TCF v2.2.
> *Nguồn:* `→ report/42-...md` · *Kiểm chứng:* review

### 3.5 Chống fingerprinting

> **REQ-07-027** (MUST · C1) — Mọi API tạo entropy (pin, cảm biến, đồng hồ, GPU, phông chữ, độ phân giải) **PHẢI** được lượng tử hóa hoặc làm thô.
> *Nguồn:* `→ report/23-...md`, `→ report/35-...md`, `→ report/42-...md` · *Kiểm chứng:* runtime

> **REQ-07-028** (SHOULD · C1) — Thông tin client (User-Agent Client Hints) **NÊN** được quản trị theo độ entropy thấp/trung bình; chỉ cấp entropy cao khi có cơ sở.
> *Nguồn:* `→ report/38-...md` · *Kiểm chứng:* runtime

> **REQ-07-029** (MUST · C1) — Giờ hệ thống dùng cho mục đích bảo mật **PHẢI** được làm thô để chống kênh phụ thời gian.
> *Nguồn:* `→ report/23-...md` · *Kiểm chứng:* runtime

### 3.6 Người dùng dễ bị tổn thương & trẻ em

> **REQ-07-030** (MUST · C2) — Với người dùng là trẻ em, **MẶC ĐỊNH** tắt lập hồ sơ, tắt cá nhân hóa quảng cáo, và áp "nudge restrictions" theo ICO Children's Code.
> *Nguồn:* `→ report/22-...md`, `→ report/13-...md` · *Kiểm chứng:* review

> **REQ-07-031** (MUST · C2) — Không suy đoán độ tuổi từ hành vi; chỉ dùng tín hiệu độ tuổi do host cung cấp.
> *Nguồn:* `→ report/13-...md` · *Kiểm chứng:* review

### 3.7 Chuyển dữ liệu & pháp lý

> **REQ-07-032** (MUST · C2) — Chuyển dữ liệu xuyên biên giới **PHẢI** có cơ sở pháp lý và được ghi nhận; lưu trú dữ liệu phải khớp cam kết.
> *Nguồn:* `→ report/08-...md`, `→ report/53-...md` · *Kiểm chứng:* review

> **REQ-07-033** (SHOULD · C2) — Chuẩn sự cố dữ liệu **NÊN** có quy trình thông báo rò rỉ trong thời hạn quy định.
> *Nguồn:* `→ report/08-...md` · *Kiểm chứng:* review

---

## 4. Ghi chú triển khai (informative)

**Ma trận dữ liệu đề xuất:**

| Loại dữ liệu | Mục đích | Lưu giữ | Định danh? | Ghi chú |
|---|---|---|---|---|
| Nhật ký hiệu năng | Đo SLO | 30 ngày | Không | Đã scrub PII |
| Breadcrumb sự cố | Chẩn đoán | 7 ngày | Không | Ring buffer cố định |
| Tín hiệu phân bổ | Đo chuyển đổi | Tổng hợp | Không | Differential privacy |
| Dữ liệu tài khoản | Vận hành dịch vụ | Theo hợp đồng | Có | Đủ điều kiện DSAR |
| Dữ liệu trẻ em | Cung cấp dịch vụ | Tối thiểu | Có | Không quảng cáo |

---

## 5. Tiêu chí tuân thủ

- [ ] Phân vùng bộ nhớ theo mini-app/origin; CHIPS cho cookie.
- [ ] Xóa an toàn dữ liệu còn sót + zeroize khóa khi gỡ cài.
- [ ] Scrub PII trước khi xuất telemetry; ring buffer không PII.
- [ ] Không dùng định danh chéo trang cho đo lường.
- [ ] Hỗ trợ GPC + TCF v2.2; không dùng legitimate interest cho quảng cáo.
- [ ] Lượng tử hóa mọi API tạo entropy.
- [ ] Có chính sách lưu giữ, DSAR, thông báo rò rỉ.
- [ ] Trẻ em: mặc định tắt cá nhân hóa.

---

## 6. Cân nhắc an ninh & riêng tư

- **Định danh lại:** dữ liệu tổng hợp vẫn có thể định danh lại nếu ngân sách đóng góp không được kiểm soát.
- **Nhất quán lời hứa:** tuyên bố trong listing phải khớp hành vi — lệch là vi phạm.
- **Thu hồi phải lan:** thu hồi đồng ý phải chạm mọi nơi dữ liệu đang tồn tại.
- **Fingerprinting là rủi ro ngầm:** ngay cả API "vô hại" cũng tạo entropy khi kết hợp.

---

## 7. Tham chiếu

- PrivacyCG Storage Access API, IETF CHIPS
- WICG Attribution Reporting, Protected Audience, Fenced Frames, Shared Storage
- IETF Privacy Pass RFC 9576/9577/9578
- IAB TCF v2.2, GPP; Global Privacy Control
- UK ICO Children's Code
- Báo cáo nguồn: `report/08, 22, 27, 28, 31, 32, 33, 38, 42, 48, 53`
