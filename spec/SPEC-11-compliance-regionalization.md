# SPEC-11 — Tuân thủ & Bản địa hóa

| Trường | Giá trị |
|---|---|
| Spec ID | `SPEC-11` |
| Phiên bản | 1.0.0 |
| Trạng thái | Draft |
| Lớp tuân thủ | **C2** (Store / Compliance) |
| Nguồn | `report/12`, `report/13`, `report/22`, `report/08`, `report/34`, `report/53`, `report/80` |

---

## 1. Phạm vi & mục tiêu

Đặc tả **tuân thủ pháp lý và bản địa hóa**: phân loại độ tuổi, an toàn trẻ em,
tiếp cận, đa ngôn ngữ/đa khu vực và nghĩa vụ quy định.

Mục tiêu: cửa hàng vận hành hợp pháp ở từng khu vực, không gây hại cho
người dùng dễ bị tổn thương, và tiếp cận được với mọi người dùng.

> **Chuẩn tham chiếu:** WCAG 2.2 Level AA, UK ICO Children's Code,
> Apple App Store Review Guidelines, IARC, EU DSA, BCP 47 / CLDR.

---

## 2. Thuật ngữ

| Thuật ngữ | Định nghĩa |
|---|---|
| **Age band** | Dải độ tuổi khai báo của ứng dụng. |
| **IARC** | International Age Rating Coalition — liên minh phân loại độ tuổi. |
| **KYBC** | Know Your Business Customer — xác minh danh tính người bán. |
| **WCAG** | Web Content Accessibility Guidelines. |
| **AOM** | Accessibility Object Model. |
| **CLDR** | Common Locale Data Repository. |
| **Statement of reasons** | Tuyên bố lý do khi gỡ/hạn chế nội dung (DSA). |

---

## 3. Yêu cầu chuẩn hóa

### 3.1 Phân loại độ tuổi

> **REQ-11-001** (MUST · C2) — Mỗi ứng dụng **PHẢI** khai báo dải độ tuổi mục tiêu (`target_audience_age_band`) tường minh trong hồ sơ.
> *Nguồn:* `→ report/13-...md` · *Kiểm chứng:* static

> **REQ-11-002** (MUST · C2) — Bản phát hành **PHẢI** hoàn thành bảng câu hỏi phân loại nội dung; kết quả là cổng chặn phát hành (release gate).
> *Nguồn:* `→ report/13-...md` · *Kiểm chứng:* static

> **REQ-11-003** (SHOULD · C2) — Phân loại **NÊN** dùng chứng chỉ IARC để liên thông toàn cầu và di động giữa các cửa hàng.
> *Nguồn:* `→ report/22-...md` · *Kiểm chứng:* static

> **REQ-11-004** (MUST · C2) — Ứng dụng cho trẻ em hoặc hỗn hợp **PHẢI** chịu chính sách năng lực nghiêm ngặt hơn; thay đổi phân loại **PHẢI** kích hoạt kiểm duyệt lại.
> *Nguồn:* `→ report/13-...md` · *Kiểm chứng:* review

> **REQ-11-005** (MUST · C1) — Tín hiệu độ tuổi **CHỈ** do host cung cấp; mini-app **KHÔNG** được suy đoán độ tuổi từ hành vi.
> *Nguồn:* `→ report/13-...md` · *Kiểm chứng:* runtime

### 3.2 An toàn trẻ em

> **REQ-11-006** (MUST · C2) — Với người dùng là trẻ em: **mặc định** tắt lập hồ sơ, tắt cá nhân hóa quảng cáo, áp hạn chế "nudge" theo UK ICO Children's Code (15 tiêu chuẩn).
> *Nguồn:* `→ report/22-...md` · *Kiểm chứng:* review

> **REQ-11-007** (MUST · C2) — Nền tảng **PHẢI** chịu trách nhiệm host theo Apple App Store Review Guidelines §4.7 và §5.1.4 về riêng tư trẻ em; §1.3 cho danh mục kids.
> *Nguồn:* `→ report/22-...md` · *Kiểm chứng:* review

> **REQ-11-008** (MUST NOT · C2) — **KHÔNG** dùng dark pattern nhắm vào trẻ em; **KHÔNG** thu thập dữ liệu trẻ em khi không có sự đồng ý hợp lệ của người giám hộ.
> *Nguồn:* `→ report/22-...md`, `→ report/08-...md` · *Kiểm chứng:* review

> **REQ-11-009** (MUST · C2) — Nội dung không phù hợp trẻ em **PHẢI** bị chặn hiển thị theo dải độ tuổi đã khai báo và theo cài đặt của phụ huynh.
> *Nguồn:* `→ report/13-...md` · *Kiểm chứng:* runtime

> **REQ-11-010** (SHOULD · C2) — Nền tảng **NÊN** có quy trình báo cáo nội dung nguy hại cho trẻ em với thời hạn xử lý nghiêm ngặt.
> *Nguồn:* `→ report/22-...md` · *Kiểm chứng:* review

### 3.3 Tiếp cận (Accessibility)

> **REQ-11-011** (MUST · C2) — Mini-app **PHẢI** đạt **WCAG 2.2 Level AA**; đây là cổng bắt buộc khi review.
> *Nguồn:* `→ report/08-...md`, `→ report/34-...md` · *Kiểm chứng:* static + review

> **REQ-11-012** (MUST · C2) — Cửa hàng **PHẢI** có kiểm toán tuân thủ WCAG tự động cho từng bản phát hành và ghi nhận kết quả.
> *Nguồn:* `→ report/34-...md` · *Kiểm chứng:* static

> **REQ-11-013** (MUST · C1) — Ứng dụng canvas/WebGL **PHẢI** chiếu cây trợ năng qua Accessibility Object Model (AOM) sang `AccessibilityNodeInfo` (Android) và `UIAccessibilityElement` (iOS).
> *Nguồn:* `→ report/34-...md` · *Kiểm chứng:* runtime

> **REQ-11-014** (MUST · C1) — Giao diện **PHẢI** hỗ trợ điều hướng bàn phím/focus, tương phản màu, cỡ chữ động, trình đọc màn hình và chế độ giảm chuyển động.
> *Nguồn:* `→ report/08-...md` · *Kiểm chứng:* review

> **REQ-11-015** (SHOULD · C3) — Nhà phát triển **NÊN** dùng HTML ngữ nghĩa, nhãn `aria` đúng, và không dùng màu làm kênh thông tin duy nhất.
> *Nguồn:* `→ report/08-...md` · *Kiểm chứng:* review

### 3.4 Bản địa hóa & khu vực

> **REQ-11-016** (MUST · C2) — Hợp đồng locale **PHẢI** theo BCP 47 / CLDR; đàm phán ngôn ngữ **PHẢI** dùng `Accept-Language`.
> *Nguồn:* `→ report/12-...md` · *Kiểm chứng:* static

> **REQ-11-017** (MUST · C3) — Tệp i18n trong gói **PHẢI** dùng tên theo thẻ BCP 47; hiển thị ngày/số/tiền tệ phải theo quy ước khu vực.
> *Nguồn:* `→ report/12-...md`, `→ report/80-...md` · *Kiểm chứng:* static

> **REQ-11-018** (SHOULD · C1) — Giao diện **NÊN** hỗ trợ RTL và thuộc tính logic CSS (logical properties) cho ngôn ngữ viết từ phải sang trái.
> *Nguồn:* `→ report/88-...md` · *Kiểm chứng:* review

> **REQ-11-019** (MUST · C2) — Dự phòng locale **PHẢI** tôn trọng riêng tư: không suy đoán vị trí/ngôn ngữ từ dữ liệu không được cấp.
> *Nguồn:* `→ report/12-...md` · *Kiểm chứng:* review

> **REQ-11-020** (MUST · C2) — Lưu trú dữ liệu **PHẢI** khớp cam kết khu vực; chuyển xuyên biên giới phải có cơ sở pháp lý.
> *Nguồn:* `→ report/53-...md`, `→ report/08-...md` · *Kiểm chứng:* review

### 3.5 Pháp lý khu vực

> **REQ-11-021** (MUST · C2) — Nền tảng **PHẢI** xác minh doanh nghiệp người bán (KYBC) theo EU DSA trước khi cho phép thương mại.
> *Nguồn:* `→ report/53-...md` · *Kiểm chứng:* review

> **REQ-11-022** (MUST · C2) — Khi gỡ/hạn chế nội dung, nền tảng **PHẢI** phát hành "statement of reasons" và cho phép kháng nghị.
> *Nguồn:* `→ report/53-...md`, `→ report/08-...md` · *Kiểm chứng:* review

> **REQ-11-023** (SHOULD · C2) — Nền tảng **NÊN** công bố báo cáo minh bạch định kỳ theo DSA.
> *Nguồn:* `→ report/53-...md` · *Kiểm chứng:* review

> **REQ-11-024** (MUST · C2) — Nền tảng **PHẢI** có cơ chế thông báo và xử lý nội dung bất hợp pháp (notice-and-action) có thời hạn.
> *Nguồn:* `→ report/53-...md` · *Kiểm chứng:* review

> **REQ-11-025** (MUST · C2) — Quyền của chủ thể dữ liệu (truy cập, xóa, di chuyển) **PHẢI** được thực thi với thời hạn xử lý.
> *Nguồn:* `→ report/08-...md` · *Kiểm chứng:* review

> **REQ-11-026** (SHOULD · C2) — Quyền rút lui đối với nội dung số **NÊN** được thông báo rõ trước khi người dùng kích hoạt sử dụng.
> *Nguồn:* `→ report/22-...md`, `→ report/14-...md` · *Kiểm chứng:* review

### 3.6 Nội dung bị cấm & bị hạn chế

> **REQ-11-027** (MUST · C2) — Danh mục nội dung bị cấm **PHẢI** được công bố và thực thi nhất quán giữa các reviewer.
> *Nguồn:* `→ report/08-...md`, `→ report/12-...md` · *Kiểm chứng:* review

> **REQ-11-028** (MUST · C2) — Quy trình khiếu nại/phúc thẩm **PHẢI** có thời hạn, lý do bằng văn bản và người xử lý độc lập.
> *Nguồn:* `→ report/08-...md` · *Kiểm chứng:* review

---

## 4. Ghi chú triển khai (informative)

**Ma trận rủi ro theo đối tượng:**

| Đối tượng | Rủi ro chính | Kiểm soát bắt buộc |
|---|---|---|
| Trẻ em | Thu thập dữ liệu, dark pattern | Mặc định tắt cá nhân hóa, đồng ý của người giám hộ |
| Người khuyết tật | Không dùng được sản phẩm | WCAG 2.2 AA + AOM cho canvas |
| Người dùng EU | Quyền dữ liệu, minh bạch | DSAR, statement of reasons, KYBC |
| Người dùng đa ngôn ngữ | Hiểu sai, hiển thị sai | BCP 47 / CLDR, RTL, định dạng khu vực |

---

## 5. Tiêu chí tuân thủ

- [ ] Khai báo dải độ tuổi + bảng phân loại nội dung hoàn tất.
- [ ] Chứng chỉ IARC (khuyến nghị) + kiểm duyệt lại khi đổi phân loại.
- [ ] Trẻ em: mặc định tắt cá nhân hóa; không dark pattern.
- [ ] WCAG 2.2 AA bắt buộc; kiểm toán tự động mỗi bản phát hành.
- [ ] AOM cho canvas/WebGL; hỗ trợ bàn phím, trình đọc màn hình.
- [ ] BCP 47 / CLDR; hỗ trợ RTL; định dạng khu vực.
- [ ] KYBC + statement of reasons + kháng nghị (DSA).
- [ ] DSAR + lưu trú dữ liệu khớp cam kết.

---

## 6. Cân nhắc an ninh & riêng tư

- **Trẻ em là nhóm rủi ro cao nhất:** mọi lỗ hở riêng tư đều nghiêm trọng hơn.
- **Nhất quán giữa các reviewer:** không được tùy tiện trong phân loại nội dung.
- **Kháng nghị là quyền:** thiếu cơ chế kháng nghị là vi phạm.
- **Không suy đoán:** độ tuổi, vị trí, ngôn ngữ chỉ dùng khi được cấp.

---

## 7. Tham chiếu

- WCAG 2.2 Level AA — https://www.w3.org/TR/WCAG22/
- UK ICO Children's Code (Age Appropriate Design Code)
- Apple App Store Review Guidelines §1.3, §4.7, §5.1.4
- IARC — https://www.globalratings.com/
- EU Digital Services Act (DSA)
- BCP 47 / CLDR — https://cldr.unicode.org/
- Báo cáo nguồn: `report/08, 12, 13, 22, 34, 53, 80, 88`
