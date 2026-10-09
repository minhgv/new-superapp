# SPEC-09 — Thương mại, Thanh toán & Quyết toán

| Trường | Giá trị |
|---|---|
| Spec ID | `SPEC-09` |
| Phiên bản | 1.0.0 |
| Trạng thái | Draft |
| Lớp tuân thủ | **C2** (Store / Commerce) |
| Nguồn | `report/14`, `report/30`, `report/25`, `report/38`, `report/47`, `report/20`, `report/89` |

---

## 1. Phạm vi & mục tiêu

Đặc tả **hợp đồng thương mại**: điều kiện bán hàng, thanh toán, quyền sở hữu số,
thuê bao, hoàn tiền, quyết toán và thanh toán thay thế.

Mục tiêu: giao dịch **không thể gian lận**, **không thể trộn lẫn** tiền của
nhiều bên, và **tuân thủ** chuẩn bảo mật tài chính.

> **Chuẩn tham chiếu:** W3C Payment Request / Payment Handler / Digital Goods,
> OpenID FAPI 2.0, IETF RFC 8705/8693/7516, PCI DSS v4.0.1.

---

## 2. Thuật ngữ

| Thuật ngữ | Định nghĩa |
|---|---|
| **SKU** | Mã sản phẩm số, bất biến theo vòng đời sản phẩm. |
| **Entitlement** | Quyền sở hữu đã mua của người dùng. |
| **Acknowledge** | Xác nhận đã giao hàng số để kích hoạt đối soát. |
| **PSP** | Nhà cung cấp dịch vụ thanh toán (Payment Service Provider). |
| **Split ledger** | Sổ cái phân tách theo từng bên hưởng. |
| **SPC** | Secure Payment Confirmation. |
| **Sender-constrained token** | Token gắn với bên sở hữu, chống tái sử dụng. |

---

## 3. Yêu cầu chuẩn hóa

### 3.1 Điều kiện thương mại & danh tính sản phẩm

> **REQ-09-001** (MUST · C2) — Nhà phát triển muốn bán hàng **PHẢI** đạt điều kiện thương mại (đã xác minh doanh nghiệp, tài khoản thanh toán hợp lệ, thỏa thuận thương mại).
> *Nguồn:* `→ report/14-...md`, `→ report/53-...md` · *Kiểm chứng:* review

> **REQ-09-002** (MUST · C2) — SKU **PHẢI** bất biến trong vòng đời sản phẩm; đổi SKU **PHẢI** là bản sản phẩm mới, không ghi đè.
> *Nguồn:* `→ report/30-...md` · *Kiểm chứng:* static

> **REQ-09-003** (MUST · C2) — Giá hiển thị **PHẢI** dùng định dạng tiền tệ theo locale và khớp với giá thanh toán; không được thay đổi âm thầm sau khi người dùng xác nhận.
> *Nguồn:* `→ report/30-...md`, `→ report/12-...md` · *Kiểm chứng:* runtime

> **REQ-09-004** (MUST · C2) — Danh mục sản phẩm số **PHẢI** truy được qua Digital Goods API (`window.getDigitalGoodsService`, `getDetails`) với giá được bản địa hóa.
> *Nguồn:* `→ report/30-...md` · *Kiểm chứng:* runtime

### 3.2 Luồng thanh toán

> **REQ-09-005** (MUST · C2) — Luồng thanh toán **PHẢI** đi qua W3C Payment Request (`canMakePayment`, `show()`, `abort()`); mini-app **KHÔNG** được tự xử lý số thẻ.
> *Nguồn:* `→ report/20-...md`, `→ report/47-...md` · *Kiểm chứng:* runtime

> **REQ-09-006** (SHOULD · C2) — Công cụ thanh toán **NÊN** là Payment Handler chạy trong Service Worker, với container làm trung gian giao diện thanh toán.
> *Nguồn:* `→ report/38-...md` · *Kiểm chứng:* runtime

> **REQ-09-007** (MUST · C2) — Xác nhận thanh toán nhạy cảm **PHẢI** dùng Secure Payment Confirmation (SPC) và ràng buộc bằng chứng sở hữu (proof-of-possession).
> *Nguồn:* `→ report/20-...md` · *Kiểm chứng:* runtime

> **REQ-09-008** (MUST · C2) — Loại phương thức thanh toán **PHẢI** được khai báo qua Payment Method Manifest; phương thức không được khai báo phải bị từ chối.
> *Nguồn:* `→ report/20-...md` · *Kiểm chứng:* static

> **REQ-09-009** (MUST · C2) — Mọi giao dịch **PHẢI** có khóa idempotency để chống tính tiền nhiều lần khi thử lại hoặc mất kết nối.
> *Nguồn:* `→ report/33-...md`, `→ report/14-...md` · *Kiểm chứng:* runtime

### 3.3 Bảo mật tài chính

> **REQ-09-010** (MUST · C2) — Tùy chọn thanh toán **PHẢI** tuân PCI DSS v4.0.1: Req 6.4.3 (ủy quyền + toàn vẹn script trên trang thanh toán) và Req 11.6.1 (phát hiện thay đổi/trộm trang thanh toán).
> *Nguồn:* `→ report/47-...md` · *Kiểm chứng:* static + runtime

> **REQ-09-011** (SHOULD · C2) — Phạm vi PCI **NÊN** được thu hẹp bằng SAQ A khi dùng hosted fields/PSP; mini-app không bao giờ chạm dữ liệu thẻ (PAN/CVC).
> *Nguồn:* `→ report/47-...md` · *Kiểm chứng:* review

> **REQ-09-012** (MUST · C2) — API tài chính **PHẢI** tuân OpenID FAPI 2.0 Security Profile: token gắn người gửi (sender-constrained), client assertion, toàn vẹn ủy quyền.
> *Nguồn:* `→ report/38-...md` · *Kiểm chứng:* runtime

> **REQ-09-013** (SHOULD · C2) — Xác thực lẫn nhau **NÊN** dùng mTLS (RFC 8705) với token gắn chứng chỉ (`x5t#S256`) làm proof-of-possession.
> *Nguồn:* `→ report/38-...md` · *Kiểm chứng:* runtime

> **REQ-09-014** (SHOULD · C2) — Ủy quyền hẹp **NÊN** dùng Token Exchange (RFC 8693) để thu hẹp phạm vi và delegate theo claim `act`.
> *Nguồn:* `→ report/38-...md` · *Kiểm chứng:* runtime

> **REQ-09-015** (MUST · C2) — Dữ liệu tải trọng nhạy cảm **PHẢI** được mã hóa xác thực (JWE RFC 7516) khi truyền qua kênh không tin cậy.
> *Nguồn:* `→ report/38-...md` · *Kiểm chứng:* runtime

> **REQ-09-016** (MUST · C1) — Khóa mật mã cho giao dịch **PHẢI** nằm trong phần cứng (Keystore/Secure Enclave), không xuất được, cách ly theo `app_id`.
> *Nguồn:* `→ report/20-...md` · *Kiểm chứng:* runtime

> **REQ-09-017** (MUST · C2) — Hệ thống **PHẢI** phát hiện gian lận: bất thường tốc độ giao dịch, trùng lặp, mismatch thiết bị/attestation.
> *Nguồn:* `→ report/17-...md`, `→ report/37-...md` · *Kiểm chứng:* runtime

### 3.4 Quyền sở hữu số & thuê bao

> **REQ-09-018** (MUST · C2) — Entitlement **PHẢI** có vòng đời `purchase → consume/acknowledge` tường minh; thiếu bước acknowledge thì giao dịch **PHẢI** bị treo đối soát để tránh bồi hoàn gian lận trong 3 ngày.
> *Nguồn:* `→ report/30-...md` · *Kiểm chứng:* runtime

> **REQ-09-019** (MUST · C2) — Trạng thái entitlement **PHẢI** nhất quán giữa thiết bị, server nhà phát triển và hệ thống quyết toán; xung đột phải được giải quyết bằng bất biến giao dịch, không bằng "lần ghi cuối".
> *Nguồn:* `→ report/14-...md` · *Kiểm chứng:* runtime

> **REQ-09-020** (MUST · C2) — Thuê bao **PHẢI** có trạng thái rõ ràng: đang hoạt động, ân hạn, hết hạn, hủy, hoàn tiền; mọi chuyển trạng thái phải được ghi log.
> *Nguồn:* `→ report/14-...md` · *Kiểm chứng:* static

> **REQ-09-021** (MUST · C2) — Hoàn tiền **PHẢI** đi qua quy trình có phê duyệt, đối soát với giao dịch gốc, và ghi vào sổ cái phân tách.
> *Nguồn:* `→ report/14-...md` · *Kiểm chứng:* review

> **REQ-09-022** (MUST · C2) — Với nội dung số, người dùng **PHẢI** được thông báo rõ về quyền rút lui và hậu quả khi kích hoạt ngay.
> *Nguồn:* `→ report/08-...md`, `→ report/22-...md` · *Kiểm chứng:* review

### 3.5 Quyết toán & sổ cái

> **REQ-09-023** (MUST · C2) — Sổ cái **PHẢI** phân tách theo từng bên hưởng (split ledger), ghi rõ hoa hồng và phần của publisher; **KHÔNG** trộn tiền nhiều bên.
> *Nguồn:* `→ report/14-...md`, `→ report/30-...md` · *Kiểm chứng:* static

> **REQ-09-024** (MUST · C2) — Đối soát **PHẢI** chạy định kỳ giữa PSP, ngân hàng và sổ cái nền tảng; sai lệch phải được cảnh báo và điều tra.
> *Nguồn:* `→ report/14-...md` · *Kiểm chứng:* review

> **REQ-09-025** (MUST · C2) — Bằng chứng chi trả cho publisher **PHẢI** truy được theo chu kỳ, gồm giao dịch nguồn, khấu trừ, dự phòng và tranh chấp.
> *Nguồn:* `→ report/14-...md` · *Kiểm chứng:* review

> **REQ-09-026** (SHOULD · C2) — Quỹ dự phòng và cơ chế xử lý tranh chấp **NÊN** được định nghĩa tường minh trong hợp đồng thương mại.
> *Nguồn:* `→ report/14-...md` · *Kiểm chứng:* review

> **REQ-09-027** (MUST · C2) — Ghi sổ **PHẢI** theo nguyên tắc bút toán kép (double-entry); tổng nợ phải bằng tổng có theo từng kỳ.
> *Nguồn:* `→ report/14-...md` · *Kiểm chứng:* static

### 3.6 Thanh toán thay thế & đa nhà cung cấp

> **REQ-09-028** (MUST · C2) — Nền tảng **PHẢI** hỗ trợ thanh toán thay thế khi pháp lý yêu cầu, với báo cáo doanh số theo quy định (ví dụ Google Play User Choice Billing 24h).
> *Nguồn:* `→ report/25-...md` · *Kiểm chứng:* review

> **REQ-09-029** (MUST · C2) — Chính sách phí **PHẢI** minh bạch theo tầng (fee tiers) và không phân biệt đối xử với nhà phát triển dùng thanh toán thay thế.
> *Nguồn:* `→ report/25-...md` · *Kiểm chứng:* review

> **REQ-09-030** (SHOULD · C2) — Lớp trừu tượng thanh toán đa nhà cung cấp **NÊN** cung cấp một giao diện duy nhất cho nhiều PSP để nhà phát triển không bị khóa nhà cung cấp.
> *Nguồn:* `→ report/25-...md` · *Kiểm chứng:* review

> **REQ-09-031** (MAY · C2) — Với thị trường EU, nền tảng **CÓ THỂ** cấp entitlement mua ngoài (External Purchase) theo điều khoản StoreKit, kèm giảm phí theo quy định.
> *Nguồn:* `→ report/25-...md` · *Kiểm chứng:* review

### 3.7 POS & đa màn hình

> **REQ-09-032** (MAY · C2) — Ứng dụng POS **CÓ THỂ** dùng Window Management API để điều phối màn hình người bán/màn hình khách; trình chiếu không dây qua Presentation API.
> *Nguồn:* `→ report/38-...md` · *Kiểm chứng:* runtime

> **REQ-09-033** (MUST · C2) — Khi ngoại tuyến, POS **PHẢI** xếp hàng giao dịch có khóa idempotency và đồng bộ lại an toàn khi có mạng; **KHÔNG** tính tiền trùng.
> *Nguồn:* `→ report/33-...md` · *Kiểm chứng:* runtime

---

## 4. Ghi chú triển khai (informative)

**Trạng thái entitlement:**

```text
pending → paid → acknowledged → active
                 ↘ expired / refunded / revoked
```

**Ba ràng buộc bất biến của thương mại:**
1. Không chạm dữ liệu thẻ (PCI scope tối thiểu).
2. Không trộn tiền nhiều bên (split ledger).
3. Không tính tiền trùng (idempotency + acknowledge).

---

## 5. Tiêu chí tuân thủ

- [ ] Publisher đạt điều kiện thương mại + KYBC.
- [ ] SKU bất biến; giá hiển thị khớp giá thanh toán.
- [ ] Thanh toán qua Payment Request/Handler; không chạm PAN/CVC.
- [ ] PCI DSS v4.0.1 Req 6.4.3 & 11.6.1 được đáp ứng.
- [ ] FAPI 2.0 + mTLS + JWE cho API tài chính.
- [ ] Entitlement có consume/acknowledge; chống bồi hoàn gian lận.
- [ ] Split ledger + đối soát định kỳ + bút toán kép.
- [ ] Hỗ trợ thanh toán thay thế khi pháp lý yêu cầu.

---

## 6. Cân nhắc an ninh & riêng tư

- **Gian lận hoàn tiền:** bước acknowledge là rào chắn bắt buộc.
- **Rò rỉ dữ liệu thẻ:** mọi thu hẹp PCI scope đều giảm rủi ro.
- **Đạo đức bán hàng:** không dark pattern, không ép mua, không giá ẩn.
- **Trẻ em:** chặn mua hàng khi không có sự đồng ý của người giám hộ (xem `SPEC-11`).

---

## 7. Tham chiếu

- W3C Payment Request / Payment Handler / Digital Goods API
- OpenID FAPI 2.0, IETF RFC 8705 / 8693 / 7516
- PCI DSS v4.0.1 (Req 6.4.3, 11.6.1, SAQ A)
- Google Play Billing Choice, Apple StoreKit External Purchase
- Báo cáo nguồn: `report/14, 20, 25, 30, 38, 47, 89`
