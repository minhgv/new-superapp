# SPEC-12 — Quy tắc Kiểm duyệt Cửa hàng

| Trường | Giá trị |
|---|---|
| Spec ID | `SPEC-12` |
| Phiên bản | 1.0.0 |
| Trạng thái | Draft |
| Lớp tuân thủ | **C2 + C3** |
| Nguồn | `report/03`, `report/04`, `report/08`, `report/12`, `report/13`, `report/22`, `report/35`, `report/39` |

---

## 1. Phạm vi & mục tiêu

Đây là **sổ tay vận hành kiểm duyệt**: quy tắc, checklist, cổng từ chối,
thời hạn xử lý và quy trình khiếu nại.

Mục tiêu: kiểm duyệt **nhất quán, minh bạch, tỷ lệ và kháng nghị được**.

---

## 2. Thuật ngữ

| Thuật ngữ | Định nghĩa |
|---|---|
| **Gate** | Cổng bắt buộc phải đạt để qua một bước kiểm duyệt. |
| **Needs changes** | Trạng thái yêu cầu nhà phát triển khắc phục. |
| **SLA** | Thỏa thuận mức dịch về thời gian xử lý. |
| **Self-preferencing** | Tự ưu tiên sản phẩm của nền tảng. |
| **Borderline** | Tình huống biên khó phân loại. |

---

## 3. Yêu cầu chuẩn hóa

### 3.1 Nguyên tắc kiểm duyệt

> **REQ-12-001** (MUST · C2) — Kiểm duyệt **PHẢI** nhất quán: cùng một nội dung, cùng một quyết định, bất kể reviewer nào.
> *Nguồn:* `→ report/03-...md` · *Kiểm chứng:* review

> **REQ-12-002** (MUST · C2) — Quyết định **PHẢI** tỷ lệ với rủi ro; không dùng hình phạt tối đa cho vi phạm nhỏ.
> *Nguồn:* `→ report/03-...md` · *Kiểm chứng:* review

> **REQ-12-003** (MUST · C2) — Mọi quyết định **PHẢI** minh bạch: có lý do, tham chiếu chính sách, và hướng khắc phục cụ thể.
> *Nguồn:* `→ report/08-...md` · *Kiểm chứng:* review

> **REQ-12-004** (MUST · C2) — Quyết định **PHẢI** kháng nghị được qua quy trình phúc thẩm có thời hạn.
> *Nguồn:* `→ report/08-...md`, `→ report/53-...md` · *Kiểm chứng:* review

> **REQ-12-005** (MUST NOT · C2) — **KHÔNG** tự ưu tiên (self-preferencing): sản phẩm của nền tảng bị kiểm duyệt theo đúng quy tắc như mọi nhà phát triển khác.
> *Nguồn:* `→ report/04-...md` · *Kiểm chứng:* review

### 3.2 Checklist kiểm duyệt theo nhóm

> **REQ-12-006** (MUST · C2) — **Siêu dữ liệu & listing:** tên, mô tả, biểu tượng, ảnh chụp phải trung thực, không gây hiểu lầm, không dùng từ khóa spam.
> *Nguồn:* `→ report/04-...md` · *Kiểm chứng:* review

> **REQ-12-007** (MUST · C2) — **Nội dung & đạo đức:** không nội dung bị cấm, không bạo lực/khiêu dâm ngoài phân loại cho phép, không thù ghét, không lừa đảo.
> *Nguồn:* `→ report/08-...md` · *Kiểm chứng:* review

> **REQ-12-008** (MUST · C2) — **Quyền hạn & riêng tư:** quyền khai báo phải khớp quyền sử dụng thực tế trong mã; thừa hoặc thiếu đều bị từ chối.
> *Nguồn:* `→ report/03-...md`, `→ report/18-...md` · *Kiểm chứng:* static + review

> **REQ-12-009** (MUST · C2) — **An ninh kỹ thuật:** qua `SPEC-04` và `SPEC-05` — không secret trong gói, có SBOM/chữ ký, sandbox đúng.
> *Nguồn:* `→ report/03-...md` · *Kiểm chứng:* static

> **REQ-12-010** (MUST · C2) — **Hiệu năng & ổn định:** đạt ngân sách `SPEC-10` (INP ≤ 200 ms, gói ≤ 4 MB / 20 MB).
> *Nguồn:* `→ report/03-...md` · *Kiểm chứng:* static

> **REQ-12-011** (MUST · C2) — **Thương mại & thanh toán:** qua `SPEC-09`; giá minh bạch, không dark pattern mua hàng, có quy trình hoàn tiền.
> *Nguồn:* `→ report/14-...md`, `→ report/22-...md` · *Kiểm chứng:* review

> **REQ-12-012** (MUST · C2) — **Trợ năng:** đạt WCAG 2.2 Level AA (`SPEC-11` §3.3).
> *Nguồn:* `→ report/08-...md` · *Kiểm chứng:* static + review

> **REQ-12-013** (SHOULD · C2) — **Bản địa hóa:** hiển thị đúng theo locale, hỗ trợ RTL khi cần, không dùng văn bản cứng không dịch được.
> *Nguồn:* `→ report/12-...md` · *Kiểm chứng:* review

> **REQ-12-014** (MUST · C2) — **Độ tuổi & trẻ em:** qua `SPEC-11` §3.1–3.2.
> *Nguồn:* `→ report/13-...md`, `→ report/22-...md` · *Kiểm chứng:* review

> **REQ-12-015** (MUST · C2) — **Sở hữu trí tuệ & thương hiệu:** không vi phạm bản quyền, nhãn hiệu, không giả mạo thương hiệu khác.
> *Nguồn:* `→ report/08-...md` · *Kiểm chứng:* review

### 3.3 Cổng từ chối (Rejection gates)

> **REQ-12-016** (MUST · C2) — Từ chối **tức thì** khi phát hiện: mã độc, đánh cắp dữ liệu, gian lận thanh toán, nội dung bất hợp pháp, giả mạo danh tính, secret trong gói.
> *Nguồn:* `→ report/03-...md`, `→ report/20-...md` · *Kiểm chứng:* static

> **REQ-12-017** (MUST · C2) — Trạng thái `needs_changes` **PHẢI** kèm danh sách việc cần sửa, thời hạn, và số lần nộp lại tối đa trước khi chuyển thành từ chối.
> *Nguồn:* `→ report/08-...md` · *Kiểm chứng:* review

> **REQ-12-018** (SHOULD · C2) — SLA xử lý **NÊN** được công bố theo loại bản phát hành (mới, cập nhật, khẩn cấp) và phải được đo lường.
> *Nguồn:* `→ report/04-...md` · *Kiểm chứng:* review

> **REQ-12-019** (MUST · C2) — Bản phát hành khẩn cấp (sửa lỗi bảo mật) **PHẢI** được xử lý ưu tiên nhưng vẫn qua đủ bốn cổng.
> *Nguồn:* `→ report/02-...md`, `→ report/09-...md` · *Kiểm chứng:* review

### 3.4 Khiếu nại & xử lý vi phạm

> **REQ-12-020** (MUST · C2) — Quy trình khiếu nại **PHẢI** có: nộp khiếu nại, xem xét bởi người khác reviewer ban đầu, quyết định bằng văn bản, thời hạn cụ thể.
> *Nguồn:* `→ report/08-...md`, `→ report/53-...md` · *Kiểm chứng:* review

> **REQ-12-021** (MUST · C2) — Hình phạt **PHẢI** theo cấp độ: cảnh báo → yêu cầu khắc phục → đình chỉ tạm → thu hồi → cấm vĩnh viễn; mỗi cấp phải có lý do.
> *Nguồn:* `→ report/08-...md` · *Kiểm chứng:* review

> **REQ-12-022** (MUST · C2) — Vi phạm an ninh nghiêm trọng **PHẢI** kích hoạt quy trình sự cố của `SPEC-08` §3.6: cách ly digest, chặn cài mới, thu hồi.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* review

> **REQ-12-023** (SHOULD · C2) — Nền tảng **NÊN** công bố báo cáo minh bạch về số lượng quyết định, khiếu nại và kết quả phúc thẩm.
> *Nguồn:* `→ report/53-...md` · *Kiểm chứng:* review

### 3.5 Bảng quyết định nhanh (tình huống biên)

| Tình huống | Quyết định | Tham chiếu |
|---|---|---|
| Khai báo `camera` nhưng chỉ dùng để quét QR | Chấp nhận nếu review xác nhận; gợi ý dùng camera host | `SPEC-06` |
| Thư viện AGPL liên kết tĩnh với SDK host | Từ chối, yêu cầu thay thế hoặc giải trình pháp lý | `SPEC-05` |
| Gói 5 MB, chạy mượt trên máy mạnh | Từ chối — vượt trần 4 MB gói chính | `SPEC-01` |
| Ứng dụng trẻ em có quảng cáo cá nhân hóa | Từ chối — vi phạm mặc định tắt cá nhân hóa | `SPEC-11` |
| Có SBOM nhưng thiếu chữ ký | `needs_changes` — thiếu attestation | `SPEC-05` |
| Ủng hộ chính trị, không vi phạm pháp luật | Chấp nhận nếu phân loại đúng, không kích động | `SPEC-11` |
| Dùng `eval` cho template đơn giản | Từ chối — vi phạm Trusted Types | `SPEC-04` |
| Giá hiển thị khác giá thanh toán | Từ chối tức thì | `SPEC-09` |
| Không hỗ trợ trình đọc màn hình | Từ chối — không đạt WCAG 2.2 AA | `SPEC-11` |
| Chỉ có bản cập nhật sửa lỗi bảo mật | Ưu tiên xử lý, vẫn qua đủ 4 cổng | `SPEC-08` |

> **REQ-12-024** (MUST · C2) — Tình huống biên **PHẢI** được quyết định theo bảng trên hoặc được leo thang (escalate) về hội đồng chính sách, và quyết định phải được ghi vào tiền lệ.
> *Nguồn:* `→ report/03-...md` · *Kiểm chứng:* review

> **REQ-12-025** (MUST · C2) — Bảng quyết định **PHẢI** được cập nhật khi có tiền lệ mới, kèm `policy_version`.
> *Nguồn:* `→ report/09-...md` · *Kiểm chứng:* review

---

## 4. Checklist cho nhà phát triển (trước khi nộp)

- [ ] Manifest hợp lệ, đủ khóa bắt buộc, `pages[0]` là trang chủ.
- [ ] Gói ≤ 4 MB (chính) / ≤ 20 MB (tổng); không tệp thừa.
- [ ] Không secret/token/khóa trong mã nguồn hoặc bundle.
- [ ] SBOM + chữ ký + provenance đầy đủ.
- [ ] Quyền khai báo khớp quyền dùng trong mã.
- [ ] CSP + egress allowlist đã cấu hình.
- [ ] INP ≤ 200 ms; không có Long Task > 50 ms kéo dài.
- [ ] WCAG 2.2 AA: bàn phím, tương phản, trình đọc màn hình, giảm chuyển động.
- [ ] Giá hiển thị khớp giá thanh toán; có quy trình hoàn tiền.
- [ ] Phân loại độ tuổi khai báo đúng; nội dung phù hợp.
- [ ] Đa ngôn ngữ: dùng BCP 47, hỗ trợ RTL khi cần.
- [ ] Chính sách riêng tư khớp hành vi thu thập dữ liệu.

---

## 5. Tiêu chí tuân thủ

- [ ] Kiểm duyệt nhất quán, tỷ lệ, minh bạch, kháng nghị được.
- [ ] Không self-preferencing.
- [ ] Đủ 10 nhóm checklist §3.2.
- [ ] Cổng từ chối tức thì được áp đúng.
- [ ] `needs_changes` có danh sách khắc phục + thời hạn.
- [ ] Quy trình khiếu nại/phúc thẩm có người xử lý độc lập.
- [ ] Bảng quyết định biên được duy trì và cập nhật.

---

## 6. Cân nhắc an ninh & riêng tư

- **Reviewer là con người:** cần đào tạo, xoay vòng và chống thiên vị.
- **Không lộ quy tắc chống lạm dụng:** một số cổng không được công bố chi tiết.
- **Nhất quán > tốc độ:** quyết định vội vàng tạo tiền lệ xấu.
- **Ghi lại mọi thứ:** mọi quyết định đều phải tái lập được.

---

## 7. Tham chiếu

- `report/03` Security, Privacy & Control Matrix (P0/P1/P2)
- `report/04` Store UX, Benchmark & Operating Model
- EU DSA — statement of reasons, appeals
- Apple App Store Review Guidelines, UK ICO Children's Code
- Báo cáo nguồn: `report/03, 04, 08, 12, 13, 22, 35, 39`
