# SPEC-08 — Control Plane của Cửa hàng

| Trường | Giá trị |
|---|---|
| Spec ID | `SPEC-08` |
| Phiên bản | 1.0.0 |
| Trạng thái | Draft |
| Lớp tuân thủ | **C2** (Store / Control plane) |
| Nguồn | `report/01`, `report/02`, `report/04`, `report/09`, `report/10`, `report/13`, `report/22`, `report/53` |

---

## 1. Phạm vi & mục tiêu

Đặc tả **hệ thống phát hành**: danh tính nhà phát triển, nộp bản phát hành,
kiểm duyệt, điều phối phát hành có kiểm soát, danh mục và thu hồi.

Mục tiêu: mọi quyết định "được phát hành hay không" phải **truy được,
nhất quán, kháng khiếu nại được** và tách bạch khỏi quyết định chạy.

---

## 2. Thuật ngữ

| Thuật ngữ | Định nghĩa |
|---|---|
| **Publisher** | Nhà phát triển/tổ chức có danh tính đã xác minh. |
| **Submission** | Lần nộp bản phát hành để kiểm duyệt. |
| **Release** | Phiên bản bất biến đã được duyệt. |
| **Rollout** | Quá trình phát hành có kiểm soát theo cohort. |
| **Quarantine** | Cách ly gói nghi ngờ, chặn cài/chạy mới. |
| **Sunset** | Ngừng hỗ trợ có kế hoạch. |

---

## 3. Yêu cầu chuẩn hóa

### 3.1 Ranh giới tin cậy

> **REQ-08-001** (MUST · C2) — Control plane (quyết định phát hành) và data plane (quyết định tải/chạy) **PHẢI** tách bạch; **KHÔNG** dùng chung ranh giới tin cậy ngầm.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* review

> **REQ-08-002** (MUST · C2) — Mọi quyết định kiểm duyệt **PHẢI** tham chiếu `policy_version` để có thể tái lập khi chính sách thay đổi.
> *Nguồn:* `→ report/02-...md`, `→ report/09-...md` · *Kiểm chứng:* static

### 3.2 Nhà phát triển (Publisher)

> **REQ-08-003** (MUST · C2) — Publisher **PHẢI** có danh tính/tổ chức đã xác minh, namespace riêng, trạng thái MFA, vai trò (roles), liên hệ an ninh và liên hệ lạm dụng.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* review

> **REQ-08-004** (MUST · C2) — Hồ sơ publisher **PHẢI** ghi trạng thái niềm tin (trust status), trạng thái đình chỉ/cách ly, điều khoản và chấp nhận chính sách riêng tư.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* static

> **REQ-08-005** (MUST · C2) — Với người bán (trader), nền tảng **PHẢI** xác minh doanh nghiệp (KYBC) theo EU DSA trước khi cho phép thương mại.
> *Nguồn:* `→ report/53-...md` · *Kiểm chứng:* review

> **REQ-08-006** (MUST · C2) — Namespace của publisher **PHẢI** độc quyền; publisher khác **KHÔNG** được phát hành trong namespace đó.
> *Nguồn:* `→ report/18-...md`, `→ report/02-...md` · *Kiểm chứng:* static

### 3.3 Nguồn lực cốt lõi (bắt buộc tra cứu được)

> **REQ-08-007** (MUST · C2) — Bản ghi App **PHẢI** chứa: `app_id`, publisher, tên hiển thị, siêu dữ liệu đa ngôn ngữ, danh mục, URL hỗ trợ, URL riêng tư, tóm tắt sử dụng dữ liệu, đối tượng mục tiêu, ảnh chụp, locale hỗ trợ, họ host/runtime, năng lực yêu cầu, mô hình doanh thu, trạng thái vòng đời.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* static

> **REQ-08-008** (MUST · C2) — Bản ghi Release **PHẢI** chứa: `release_id`, `app_id`, SemVer, digest gói bất biến, kích thước, media type, entry point, dải SDK host, host API yêu cầu, khai báo năng lực, endpoint mạng, tham chiếu SBOM/provenance/chữ ký, ghi chú phát hành, quyết định reviewer, chính sách rollout, đích rollback, và các mốc thời gian.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* static

> **REQ-08-009** (MUST · C2) — Bản ghi Review decision **PHẢI** chứa: `submission_id`, kiểm tra tự động, danh tính/vai trò reviewer, `policy_version`, phát hiện, khắc phục, chấp nhận/từ chối, trạng thái khiếu nại, liên kết bằng chứng.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* static

> **REQ-08-010** (MUST · C2) — Sự kiện kiểm toán **PHẢI** là append-only, gồm chủ thể/dịch vụ, hành động, tài nguyên/phiên bản, thời điểm, lý do, `policy_version`, tham chiếu before/after và `correlation_id`.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* static

### 3.4 Máy trạng thái phát hành

> **REQ-08-011** (MUST · C2) — Vòng đời **PHẢI** là: `draft → submitted → automated_checks_passed → human_review → approved → internal → preview → canary → staged → active`, kèm các trạng thái phụ `rejected`, `needs_changes`, `quarantined`, `paused`, `aborted`, `withdrawn`, `deprecated`, `sunset_scheduled`, `blocked`.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* static

> **REQ-08-012** (MUST · C2) — Mọi chuyển trạng thái **PHẢI** được ủy quyền và ghi nhật ký; **KHÔNG** có chuyển trạng thái ngầm.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* static

> **REQ-08-013** (MUST · C2) — Release **KHÔNG ĐƯỢC** chuyển sang `active` trừ khi digest, danh tính, kết quả tương thích, chính sách năng lực, kết quả kiểm tra tự động, quyết định reviewer, đích rollout và đích rollback đều tra cứu được theo `release_id`.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* static

### 3.5 Kiểm tra tự động & kiểm duyệt

> **REQ-08-014** (MUST · C2) — Kiểm tra tự động **PHẢI** gồm: xác thực manifest/schema, kích thước/đường dẫn/entry point, năng lực khai báo, phân tích tĩnh, quét malware, quét secret, SBOM/provenance/chữ ký, và bài kiểm thử trên ma trận SDK host/thiết bị.
> *Nguồn:* `→ report/02-...md`, `→ report/03-...md` · *Kiểm chứng:* static

> **REQ-08-015** (MUST · C2) — Kiểm duyệt thủ công **PHẢI** đánh giá đủ các chiều: nội dung, riêng tư, quyền hạn, UX, an ninh và chính sách.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* review

> **REQ-08-016** (MUST · C2) — Reviewer **PHẢI** độc lập với publisher được duyệt; xung đột lợi ích phải được khai báo và loại trừ.
> *Nguồn:* `→ report/08-...md` · *Kiểm chứng:* review

> **REQ-08-017** (MUST · C2) — Quy trình **PHẢI** có `needs_changes` với yêu cầu khắc phục cụ thể, thời hạn, và hỗ trợ khiếu nại/phúc thẩm.
> *Nguồn:* `→ report/08-...md`, `→ report/22-...md` · *Kiểm chứng:* review

> **REQ-08-018** (SHOULD · C2) — Kiểm duyệt **NÊN** dùng phân tích tự động về quyền khai báo vs quyền sử dụng thực tế trong mã.
> *Nguồn:* `→ report/03-...md` · *Kiểm chứng:* static

### 3.6 Điều phối phát hành

> **REQ-08-019** (MUST · C2) — Phát hành **PHẢI** đi qua các cohort: `internal → preview → canary → staged → active`.
> *Nguồn:* `→ report/02-...md`, `→ report/09-...md` · *Kiểm chứng:* review

> **REQ-08-020** (MUST · C2) — Ngưỡng sức khỏe **PHẢI** được định nghĩa trước cho từng cohort; vượt ngưỡng thì **tự động dừng** phát hành.
> *Nguồn:* `→ report/09-...md` · *Kiểm chứng:* runtime

> **REQ-08-021** (MUST · C2) — Nền tảng **PHẢI** giữ digest tốt nhất đã biết và hỗ trợ rollback một chạm về bản đó.
> *Nguồn:* `→ report/09-...md`, `→ report/02-...md` · *Kiểm chứng:* review

> **REQ-08-022** (MUST · C2) — Khi có sự cố: **PHẢI** dừng phát hành, cách ly digest, chặn cài mới, thu hồi/hạn chế niềm tin, vô hiệu cache không an toàn, và thông báo theo mức nghiêm trọng.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* review

> **REQ-08-023** (SHOULD · C2) — Sự cố vật chất **NÊN** chuyển thành vòng học không đổ lỗi (blameless post-mortem) và cập nhật chính sách.
> *Nguồn:* `→ report/09-...md` · *Kiểm chứng:* review

### 3.7 Tương thích & danh mục

> **REQ-08-024** (MUST · C2) — Bộ giải quyết tương thích **PHẢI** đánh giá: nền tảng, dải SDK host, dải bridge/API, giao chính sách năng lực, locale, định dạng gói, phiên bản chính sách.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* static

> **REQ-08-025** (MUST · C2) — "Cài được" và "chạy được" **PHẢI** là hai kết quả riêng; bản phát hành **CÓ THỂ** hiển thị nhưng không khả dụng kèm lý do.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* runtime

> **REQ-08-026** (SHOULD · C2) — Danh mục **NÊN** xuất chiếu siêu dữ liệu di động (Schema.org) và hỗ trợ trao đổi bằng chứng liên thông (SPDX/CycloneDX/OpenVEX/in-toto).
> *Nguồn:* `→ report/10-...md` · *Kiểm chứng:* static

> **REQ-08-027** (MUST · C2) — Đọc danh mục **PHẢI** dùng phân trang con trỏ (cursor) và ETag; thao tác nộp/điều phối **PHẢI** dùng khóa idempotency.
> *Nguồn:* `→ report/02-...md`, `→ report/11-...md` · *Kiểm chứng:* static

### 3.8 Ngừng hỗ trợ & thu hồi

> **REQ-08-028** (MUST · C2) — Chuẩn bị ngừng hỗ trợ **PHẢI** đi qua `active → deprecated → sunset_scheduled → blocked → withdrawn/deleted`, chỉ khi pháp lý, an ninh và chính sách lưu giữ cho phép.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* review

> **REQ-08-029** (MUST · C2) — Thông báo ngừng hỗ trợ **PHẢI** có: bản kế nhiệm/hướng dẫn chuyển đổi, chủ sở hữu, ngày thông báo, ngày ngừng, phạm vi, và hành vi xuất/lưu dữ liệu.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* review

> **REQ-08-030** (SHOULD · C2) — API liên quan **NÊN** dùng tín hiệu HTTP `Deprecation`/`Sunset` máy đọc được, nhưng **KHÔNG** chỉ dựa vào header cho UX cửa hàng.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* static

### 3.9 Đa thuê & chống lạm dụng

> **REQ-08-031** (MUST · C2) — Cách ly đa thuê **PHẢI** bảo đảm một tenant **KHÔNG** ảnh hưởng dữ liệu/uy tín/danh mục của tenant khác.
> *Nguồn:* `→ report/11-...md`, `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-08-032** (MUST · C2) — Cơ chế phản kháng khiếu nại và ra quyết định **PHẢI** minh bạch, có lý do, và kháng nghị được (DSA statement of reasons).
> *Nguồn:* `→ report/53-...md`, `→ report/22-...md` · *Kiểm chứng:* review

> **REQ-08-033** (SHOULD · C2) — Nền tảng **NÊN** công bố báo cáo minh bạch định kỳ theo DSA.
> *Nguồn:* `→ report/53-...md` · *Kiểm chứng:* review

---

## 4. Ghi chú triển khai (informative)

**Bốn cổng bắt buộc trước `active`:**

| Cổng | Chịu trách nhiệm | Kết quả |
|---|---|---|
| Static | Máy | Schema, kích thước, secret, SBOM |
| Provenance | Máy | TUF + SLSA + Sigstore |
| Review | Người + máy | Nội dung, quyền hạn, riêng tư |
| Rollout | Máy có ngưỡng | Cohort + tự động dừng |

**Nguyên tắc:** không có cổng nào thay thế cổng nào. Bỏ qua một cổng là
phá vỡ toàn bộ mô hình tin cậy.

---

## 5. Tiêu chí tuân thủ

- [ ] Publisher có danh tính xác minh + namespace độc quyền + MFA.
- [ ] Đủ trường bắt buộc cho App, Release, Review decision, Audit event.
- [ ] Máy trạng thái đầy đủ; mọi chuyển trạng thái được ghi log.
- [ ] 4 cổng hoàn tất trước `active`.
- [ ] Reviewer độc lập; có quy trình phúc thẩm.
- [ ] Ngưỡng sức khỏe + tự động dừng + rollback một chạm.
- [ ] Bộ giải quyết tương thích tách "cài được" / "chạy được".
- [ ] Quy trình deprecation có bản kế nhiệm + ngày ngừng.

---

## 6. Cân nhắc an ninh & riêng tư

- **Không tự ưu tiên:** nền tảng không được ưu tiên sản phẩm của mình trong review.
- **Kháng nghị được:** mọi từ chối phải có lý do và đường phúc thẩm.
- **Nhất quán chính sách:** `policy_version` phải gắn mọi quyết định.
- **Dữ liệu kiểm toán:** append-only, không ghi đè, không xóa ngầm.

---

## 7. Tham chiếu

- `report/02` Reference Architecture & Lifecycle
- `report/01` Executive Summary
- `report/03` Security, Privacy & Control Matrix
- `report/04` Store UX, Benchmark & Operating Model
- EU Digital Services Act (DSA) — trader traceability, statement of reasons
- Apple App Store Review Guidelines §4.7, UK ICO Children's Code
- Báo cáo nguồn: `report/01, 02, 03, 04, 09, 10, 13, 22, 53`
