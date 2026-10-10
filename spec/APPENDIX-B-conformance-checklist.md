# APPENDIX B — Checklist Tuân thủ (Conformance Gates)

| Trường | Giá trị |
|---|---|
| ID | `APPENDIX-B` |
| Phiên bản | 1.1.0 |
| Trạng thái | Draft |
| Mục đích | Cổng kiểm tra tuân thủ kiểm chứng được cho mỗi bản phát hành |
| Nguồn | `report/03-security-privacy-control-matrix.md`, `report/06-gaps-validation-plan.md`, `SPEC-00` §4–5 |

---

## 1. Mục đích & cách đánh giá

Checklist này là **cổng bắt buộc** phải qua trước khi một bản phát hành được
chuyển sang `active`. Mỗi mục kiểm tra có:

- **ID** duy nhất `G<cổng>-<số>` để tham chiếu trong báo cáo kiểm duyệt.
- **Phương pháp đánh giá:** `static` (máy) · `review` (người) · `runtime` (thực thi).
- **Điều kiện pass/fail** tường minh.

**Định dạng ghi nhận bằng chứng theo `release_id`:**

```json
{
  "release_id": "rel_01H8...",
  "gate": "G1",
  "check": "G1-03",
  "verdict": "pass | fail | warn",
  "method": "static",
  "evidence": "https://... hoặc artifact digest",
  "checked_at": "2026-10-09T00:00:00Z",
  "policy_version": "2026-10"
}
```

---

## 2. Các cổng bắt buộc

> **Lưu ý:** G1–G4 là bốn cổng **luôn luôn** bắt buộc. G5 (§2 cuối) chỉ áp dụng
> khi mini app có danh tính số / chứng chỉ / chứng thực phần cứng.

### Cổng 1 — Kiểm tra tĩnh (Static gate)

*Mục tiêu: loại bỏ sớm các lỗi cấu trúc và nội dung nguy hiểm.*

| ID | Kiểm tra | Pass khi | Phương pháp | SPEC |
|---|---|---|---|---|
| G1-01 | Manifest hợp lệ schema | Đủ 6 khóa bắt buộc, JSON hợp lệ | static | `SPEC-01` |
| G1-02 | `pages[0]` tồn tại | Tệp trang chủ có trong gói | static | `SPEC-01` |
| G1-03 | Icons có `src` hợp lệ | Mọi biểu tượng tồn tại | static | `SPEC-01` |
| G1-04 | Cấu trúc gói | Có `manifest.json`, `app.js`, `app.css`, `pages/` | static | `SPEC-01` |
| G1-05 | ZIP hợp lệ | UTF-8 tên tệp, CRC32, không mã hóa/chia mảnh | static | `SPEC-01` |
| G1-06 | Trần dung lượng | Gói chính ≤ 4 MB, tổng ≤ 20 MB | static | `SPEC-01` |
| G1-07 | Không tệp thừa | Mọi tài nguyên được khai báo | static | `SPEC-01` |
| G1-08 | Quét secret | Không có API key/token/PEM/private key | static | `SPEC-04` |
| G1-09 | Quét malware | Không mã độc, không tải mã từ xa không SRI | static | `SPEC-04` |
| G1-10 | Trusted Types | Bật `require-trusted-types-for 'script'` | static | `SPEC-04` |
| G1-11 | CSP hợp lệ | Không `*` cho `script-src`/`connect-src` | static | `SPEC-04` |
| G1-12 | Egress allowlist | Danh sách miền tường minh trong manifest | static | `SPEC-04` |
| G1-13 | Không `eval`/`innerHTML` thô | Không có sink XSS không qua Trusted Type | static | `SPEC-04` |
| G1-14 | SRI cho tài nguyên tải | Có digest/chữ ký cho mọi tải từ xa | static | `SPEC-05` |
| G1-15 | SBOM hợp lệ | CycloneDX/SPDX parse được, đủ thành phần | static | `SPEC-05` |
| G1-16 | Giấy phép | Không AGPL/GPL liên kết tĩnh | static | `SPEC-05` |
| G1-17 | Quyền khai báo | `req_permissions` hợp lệ, có `reason` | static | `SPEC-06` |
| G1-18 | Khớp quyền dùng | Quyền khai báo = quyền dùng trong mã | static | `SPEC-12` |
| G1-19 | Ngân sách hiệu năng tĩnh | Gói không chứa asset quá khổ | static | `SPEC-10` |
| G1-20 | i18n đúng BCP 47 | Tệp ngôn ngữ đúng định dạng | static | `SPEC-11` |

### Cổng 2 — Chuỗi cung ứng & ký số (Provenance gate)

*Mục tiêu: chứng minh nguồn gốc, tính toàn vẹn và độ mới của gói.*

| ID | Kiểm tra | Pass khi | Phương pháp | SPEC |
|---|---|---|---|---|
| G2-01 | Digest khớp | Digest gói khớp attestation | static | `SPEC-05` |
| G2-02 | Provenance SLSA | Có `builder.id`, `buildType`, `subject` | static | `SPEC-05` |
| G2-03 | Chữ ký Sigstore | Chuỗi Fulcio + Rekor inclusion proof hợp lệ | static | `SPEC-05` |
| G2-04 | Identity claims | Danh tính khớp publisher đã đăng ký | static | `SPEC-05` |
| G2-05 | Chuỗi TUF | `Timestamp → Snapshot → Targets` hợp lệ | static | `SPEC-05` |
| G2-06 | Ngưỡng chữ ký | Đủ ngưỡng cho namespace của publisher | static | `SPEC-05` |
| G2-07 | Chống phát lại | Siêu dữ liệu mới hơn đã thấy | runtime | `SPEC-05` |
| G2-08 | Sub-package digest | Mỗi gói phụ có digest riêng | static | `SPEC-01` |
| G2-09 | Attestation neo vào TUF | Subject digest = entry TUF Targets | static | `SPEC-05` |
| G2-10 | Lưu giữ artifact | Artifact + attestation được lưu bất biến | review | `SPEC-05` |

### Cổng 3 — Kiểm duyệt nội dung (Review gate)

*Mục tiêu: bảo đảm nội dung, quyền hạn và trải nghiệm phù hợp.*

| ID | Kiểm tra | Pass khi | Phương pháp | SPEC |
|---|---|---|---|---|
| G3-01 | Listing trung thực | Tên/mô tả/ảnh không gây hiểu lầm | review | `SPEC-12` |
| G3-02 | Nội dung hợp lệ | Không nội dung bị cấm | review | `SPEC-11` |
| G3-03 | Phân loại độ tuổi | Bảng câu hỏi hoàn tất + IARC (nếu có) | review | `SPEC-11` |
| G3-04 | An toàn trẻ em | Mặc định tắt cá nhân hóa khi là trẻ em | review | `SPEC-11` |
| G3-05 | WCAG 2.2 AA | Đạt hoặc có lộ trình khắc phục | static + review | `SPEC-11` |
| G3-06 | Quyền hạn & riêng tư | Khai báo khớp thu thập; có chính sách | review | `SPEC-07` |
| G3-07 | Thương mại minh bạch | Giá khớp; có quy trình hoàn tiền | review | `SPEC-09` |
| G3-08 | Sở hữu trí tuệ | Không vi phạm bản quyền/nhãn hiệu | review | `SPEC-12` |
| G3-09 | Trải nghiệm người dùng | Không dark pattern, không spam | review | `SPEC-12` |
| G3-10 | Bản địa hóa | Hiển thị đúng locale, hỗ trợ RTL khi cần | review | `SPEC-11` |
| G3-11 | Cân nhắc an ninh | Không vector tấn công hiển nhiên | review | `SPEC-04` |
| G3-12 | Reviewer độc lập | Không xung đột lợi ích | review | `SPEC-08` |

### Cổng 4 — Phát hành có kiểm soát (Rollout gate)

*Mục tiêu: giới hạn tác động khi có sự cố và cho phép phục hồi nhanh.*

| ID | Kiểm tra | Pass khi | Phương pháp | SPEC |
|---|---|---|---|---|
| G4-01 | Cohort nội bộ | Chạy ổn định trong cohort `internal` | runtime | `SPEC-08` |
| G4-02 | Cohort preview | Không lỗi hồi quy trong `preview` | runtime | `SPEC-08` |
| G4-03 | Ngưỡng sức khỏe | Đạt ngưỡng SLO trong `canary` | runtime | `SPEC-10` |
| G4-04 | Tự động dừng | Cơ chế dừng khi vượt ngưỡng hoạt động | runtime | `SPEC-08` |
| G4-05 | Rollback một chạm | Đã kiểm thử rollback về digest tốt nhất | runtime | `SPEC-05` |
| G4-06 | Nhật ký & giám sát | Có cảnh báo và nhật ký tương quan | runtime | `SPEC-10` |
| G4-07 | Cập nhật delta toàn vẹn | Digest cuối khớp sau delta update | runtime | `SPEC-05` |
| G4-08 | Thông báo sự cố | Quy trình thông báo theo mức nghiêm trọng | review | `SPEC-08` |

---

### Cổng 5 — Danh tính & Chuỗi minh bạch (Identity & provenance gate)

*Bổ sung từ các báo cáo `report/124` … `report/145`. Bắt buộc khi mini app
sử dụng danh tính số, chứng chỉ, hoặc chứng thực phần cứng.*

| ID | Kiểm tra | Pass khi | Phương pháp | SPEC |
|---|---|---|---|---|
| G5-01 | OAuth 2.1 + PKCE S256 | Không implicit/password grant | static | `SPEC-06` |
| G5-02 | JWT tuân thủ RFC 8725 | Không `alg=none`, đủ `iss`/`aud`/`exp` | static | `SPEC-06` |
| G5-03 | Khóa có JWK Thumbprint | Mọi khóa có định danh máy đọc được | static | `SPEC-05` |
| G5-04 | Đối tượng ký chuẩn hóa | CBOR deterministic hoặc JSON canonical | static | `SPEC-05` |
| G5-05 | Thu hồi chứng chỉ hoạt động | Status List trả về trạng thái trong ≤ 60 s | runtime | `SPEC-06` |
| G5-06 | Tiết lộ có chọn lọc | SD-JWT không lộ thuộc tính thừa | review | `SPEC-06` |
| G5-07 | Chuỗi tin cậy hợp lệ | Trust chain về tới trust anchor | runtime | `SPEC-06` |
| G5-08 | EAT/RATS hợp lệ | Attestation result có appraisal | runtime | `SPEC-05` |
| G5-09 | Nguồn gốc nội dung (nếu có) | C2PA manifest ký hợp lệ | static | `SPEC-05` |
| G5-10 | PNA chặn SSRF nội bộ | Không gọi được RFC 1918/loopback | runtime | `SPEC-04` |
| G5-11 | Trợ năng: AccName + APG | Mọi điều khiển có tên + bàn phím đúng | static + review | `SPEC-11` |
| G5-12 | Thời gian tường minh | Không có chuỗi giờ mơ hồ (RFC 9557) | static | `SPEC-11` |

> Cổng 5 chỉ **bổ sung**, không thay thế bốn cổng G1–G4. Một mini app không
> dùng danh tính số vẫn phải qua G1–G4; khi có danh tính số thì thêm G5.

---

## 3. Ma trận mức độ kiểm soát (P0 / P1 / P2)

| Mức | Ý nghĩa | Hậu quả khi fail | Ví dụ kiểm soát |
|---|---|---|---|
| **P0** | Nguy hiểm / bất hợp pháp | **Chặn phát hành** | Mã độc, secret trong gói, nội dung bất hợp pháp, vi phạm PCI |
| **P1** | Nghiêm trọng nhưng không tức thì | **Chặn promote lên `active`** | Không đạt WCAG AA, thiếu chữ ký, quyền khai báo lệch, INP vượt |
| **P2** | Chất lượng / tối ưu | **Cảnh báo + theo dõi** | Thiếu i18n, thiếu VEX, chưa dùng delta update |

> `report/03-security-privacy-control-matrix.md` liệt kê chi tiết từng kiểm soát
> theo mức P0/P1/P2. Ba mức này tương ứng ba hậu quả hành động ở cột hai.

---

## 4. Bảng quyết định theo lớp tuân thủ

| Lớp | Chịu trách nhiệm | Cổng bắt buộc | Điều kiện pass |
|---|---|---|---|
| **C1 — Host/Container** | Runtime, sandbox, quyền hạn | G1 (từ phía host), G2, G4 | Sandbox fail-closed; SLO đạt; rollback hoạt động |
| **C2 — Store/Control plane** | Kiểm duyệt, phát hành, thương mại | G1, G2, G3, G4 | Tất cả 4 cổng pass; có audit trail đầy đủ |
| **C3 — Mini App** | Mã nguồn, manifest, listing | G1, G3 | Manifest hợp lệ; không secret; WCAG AA; listing trung thực |

> **Nguyên tắc:** một cổng fail **không** được bù bằng cổng khác pass.
> Cả bốn cổng phải pass đồng thời trước `active`.

---

## 5. Hồ sơ bằng chứng bắt buộc theo `release_id`

- [ ] `manifest.json` đã validate + báo cáo schema validation.
- [ ] Digest gói (SHA-256) + kết quả xác minh.
- [ ] SBOM (CycloneDX/SPDX) + CBOM (nếu có).
- [ ] Provenance SLSA (attestation) + chứng chỉ Sigstore + Rekor inclusion proof.
- [ ] Kết quả chuỗi TUF (`Timestamp → Snapshot → Targets`).
- [ ] Kết quả quét secret + malware + SRI.
- [ ] Kết quả kiểm tra tĩnh (Trusted Types, CSP, egress, quyền khai báo).
- [ ] Kết quả kiểm thử trên ma trận SDK host/thiết bị.
- [ ] Quyết định reviewer + `policy_version` + lý do.
- [ ] Kế hoạch rollout + đích rollback + ngưỡng sức khỏe.
- [ ] Kết quả kiểm toán WCAG 2.2 AA.
- [ ] Bảng phân loại nội dung / chứng chỉ IARC (nếu có).
- [ ] Nhật ký kiểm toán (append-only) với `correlation_id`.

---

## 6. Các mục chưa quyết định — **KHÔNG** được đánh dấu pass

Các mục sau còn thiếu bằng chứng/quyết định và **phải** được đánh dấu
`pending` trong hồ sơ tuân thủ, tuyệt đối không được pass:

| ID | Hạng mục | Vì sao chưa quyết định |
|---|---|---|
| G2-08a | Hợp đồng ký cho **sub-package** | W3C MiniApp Packaging chưa kết thúc phần này |
| G2-03a | Mức **SLSA L2 hay L3** cho gói nhạy cảm | Cần quyết định chính sách theo rủi ro |
| G3-07a | **Entitlement vận hành** (chạy nền, nghe nền) | Chưa có danh mục quyền ưu tiên chuẩn |
| G2-05a | **Phân giải nhà cung cấp gói** (provider registry) | Chưa có hợp đồng ánh xạ chuẩn |
| G3-03a | Tín hiệu độ tuổi cho **thị trường mới** | Chưa có yêu cầu pháp lý cụ thể |
| G4-06a | Ngưỡng **burn-rate** cảnh báo cụ thể | Cần dữ liệu vận hành ban đầu |

→ Chi tiết: `report/06-gaps-validation-plan.md`

---

## 7. Ghi chú vận hành

- **Không bỏ qua cổng:** kể cả bản vá khẩn cấp cũng đi qua đủ 4 cổng, chỉ được ưu tiên thời gian xử lý.
- **Nhất quán `policy_version`:** mọi kết quả phải gắn phiên bản chính sách để tái lập được.
- **Tự động hóa tối đa:** G1 và G2 nên chạy hoàn toàn tự động; G3 kết hợp người + máy; G4 tự động có ngưỡng.
- **Kiểm thử rollback:** rollback phải được kiểm thử định kỳ, không chỉ khi có sự cố.
- **Cập nhật checklist:** khi có tiền lệ mới, bổ sung vào bảng quyết định của `SPEC-12` và nâng phiên bản tài liệu này.
