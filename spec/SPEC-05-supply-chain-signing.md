# SPEC-05 — Chuỗi cung ứng & Ký số

| Trường | Giá trị |
|---|---|
| Spec ID | `SPEC-05` |
| Phiên bản | 1.1.0 |
| Trạng thái | Draft |
| Lớp tuân thủ | **C1 + C2** |
| Nguồn | `report/18`, `report/37`, `report/41`, `report/39`, `report/54`, `report/45`, `report/47`, `report/20` |

---

## 1. Phạm vi & mục tiêu

Đặc tả **đảm bảo nguồn gốc và tính toàn vẹn** của gói mini-app suốt vòng đời:
từ build, ký số, phân phối, cài đặt đến cập nhật và thu hồi.

Mục tiêu: một gói **không** được chấp nhận nếu không chứng minh được
(1) ai xây dựng, (2) từ đâu, (3) có bị sửa đổi không, (4) có còn mới không.

> **Chuẩn tham chiếu:** TUF 1.x, SLSA v1.0, Sigstore, OWASP CycloneDX v1.6,
> LF SPDX 3.0, in-toto, IETF RFC 9421/9530/9052.

---

## 2. Thuật ngữ

| Thuật ngữ | Định nghĩa |
|---|---|
| **Provenance** | Chứng thực nguồn gốc build của một artifact. |
| **Attestation** | Tuyên bố đã ký về đặc tính của artifact. |
| **Threshold signature** | Chữ ký ngưỡng — cần N/M khóa đồng ý. |
| **Transparency log** | Nhật ký bất biến (Rekor) chống chối bỏ. |
| **SBOM / CBOM** | Danh mục thành phần phần mềm / mật mã. |
| **VEX** | Vulnerability Exploitability eXchange. |
| **Keyless signing** | Ký bằng danh tính OIDC + chứng chỉ ngắn hạn. |
| **Anti-rollback** | Chống ép về phiên bản cũ có lỗ hổng. |

---

## 3. Yêu cầu chuẩn hóa

### 3.1 Mô hình tin cậy TUF

> **REQ-05-001** (MUST · C2) — Kho gói **PHẢI** dùng mô hình TUF với bốn vai trò tách bạch: `Root`, `Targets`, `Snapshot`, `Timestamp`.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-05-002** (MUST · C2) — Mỗi vai trò **PHẢI** cho phép nhiều khóa và **ngưỡng chữ ký** (threshold); khóa Root **PHẢI** được lưu ngoại tuyến và nằm ngoài JavaScript của mini-app.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* review

> **REQ-05-003** (MUST · C2) — Vai trò `Targets` **PHẢI** được ủy quyền (delegation) theo publisher/tenant/namespace với ngưỡng tường minh; một publisher **KHÔNG ĐƯỢC** ký namespace của publisher khác.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-05-004** (MUST · C1) — Trước khi cài/cập nhật, client **PHẢI** xác minh đầy đủ chuỗi `Timestamp → Snapshot → Targets`, sau đó xác minh digest gói và độ dài.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-05-005** (MUST · C1) — `Snapshot` **PHẢI** chặn phối trộn (mix-and-match) siêu dữ liệu; `Timestamp` **PHẢI** chặn phát lại siêu dữ liệu cũ (anti-freeze / anti-replay).
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-05-006** (MUST · C1) — Quyết định cài/cập/rollback/cách ly **CHỈ** được đưa ra sau khi xác minh ngưỡng chữ ký và digest phía host. CDN có thể phục vụ byte nhưng **KHÔNG** được ủy quyền gói.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

### 3.2 Nguồn gốc build (SLSA)

> **REQ-05-007** (MUST · C2) — Mỗi artifact nộp lên **PHẢI** kèm provenance gắn với digest mật mã, mô tả `builder.id`, `buildType`, `externalParameters`, `resolvedDependencies` và `subject`.
> *Nguồn:* `→ report/18-...md`, `→ report/41-...md` · *Kiểm chứng:* static

> **REQ-05-008** (MUST · C2) — Build **PHẢI** chạy trên nền tảng hosted cách ly, có signer của control plane; **KHÔNG ĐƯỢC** để khóa ký nằm trong bước build do người dùng kiểm soát.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* review

> **REQ-05-009** (SHOULD · C2) — Mức tối thiểu thực tế là **SLSA Build L2** (provenance hosted có xác thực); **L3** là đích cho các gói nhạy cảm hoặc tính năng nguy cơ cao.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* review

> **REQ-05-010** (MUST · C2) — Store **PHẢI** đối chiếu provenance với publisher đã đăng ký và chính sách phát hành; từ chối khi attestation thiếu hoặc digest không khớp.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-05-011** (MUST · C2) — Subject digest của attestation **PHẢI** được neo vào bản ghi TUF Targets để một attestation hợp lệ cho gói này **không** thể tái sử dụng cho gói khác.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* static

### 3.3 Ký số không khóa (Sigstore)

> **REQ-05-012** (SHOULD · C2) — Ký số **NÊN** dùng Sigstore keyless: Fulcio cấp chứng chỉ ngắn hạn gắn khóa tạm với danh tính OIDC; sự kiện ký được ghi vào Rekor.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-05-013** (MUST · C2) — Bộ xác minh **PHẢI** kiểm tra: chữ ký, chuỗi chứng chỉ Fulcio và issuer, các claim danh tính publisher/repository/workflow, digest artifact, và bằng chứng bao gồm (inclusion proof) của Rekor.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-05-014** (SHOULD · C2) — Nền tảng **NÊN** giám sát nhật ký minh bạch để phát hiện danh tính bất thường hoặc attestation trùng/xung đột.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-05-015** (MUST NOT · C2) — **KHÔNG** được coi keyless signing là chứng minh tính đúng đắn của build, độ mới của TUF hay độ an toàn của gói; phải kết hợp cả ba.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* review

### 3.4 SBOM, CBOM & VEX

> **REQ-05-016** (MUST · C2) — Mỗi gói **PHẢI** kèm SBOM máy đọc được (OWASP CycloneDX v1.6 hoặc LF SPDX 3.0) liệt kê toàn bộ thành phần và phiên bản.
> *Nguồn:* `→ report/41-...md` · *Kiểm chứng:* static

> **REQ-05-017** (SHOULD · C2) — SBOM **NÊN** tích hợp CBOM (Cryptography Bill of Materials) để kiểm toán mật mã và lộ trình hậu lượng tử.
> *Nguồn:* `→ report/41-...md`, `→ report/40-...md` · *Kiểm chứng:* static

> **REQ-05-018** (MUST · C2) — Giấy phép **PHẢI** được kiểm tra: không cho phép thư viện AGPL/GPL liên kết tĩnh với SDK host độc quyền; vi phạm thì yêu cầu giải trình pháp lý hoặc thay thế.
> *Nguồn:* `→ report/41-...md` · *Kiểm chứng:* static

> **REQ-05-019** (SHOULD · C2) — Khi phát hiện lỗ hổng, **NÊN** phát hành VEX (OpenVEX) nêu rõ trạng thái khai thác được để tránh cảnh báo sai.
> *Nguồn:* `→ report/41-...md` · *Kiểm chứng:* static

> **REQ-05-020** (MUST · C2) — Toàn bộ chuỗi build **PHẢI** tuân khung in-toto: mỗi bước build có attestation riêng, liên kết được về artifact cuối.
> *Nguồn:* `→ report/41-...md` · *Kiểm chứng:* static

### 3.5 Ký giao thức & toàn vẹn truyền tải

> **REQ-05-021** (SHOULD · C1) — Chữ ký gói **NÊN** dùng COSE (RFC 9052/9053) dạng CBOR nhịp gọn cho bundle.
> *Nguồn:* `→ report/39-...md` · *Kiểm chứng:* static

> **REQ-05-022** (SHOULD · C2) — Webhook và API nhạy cảm **NÊN** dùng HTTP Message Signatures (RFC 9421) kèm Digest Fields (RFC 9530) để chống giả mạo và chống sửa.
> *Nguồn:* `→ report/41-...md` · *Kiểm chứng:* runtime

> **REQ-05-023** (MUST · C1) — Nội dung tải từ CDN **PHẢI** có SRI hoặc digest xác minh trước khi thực thi; TLS là bắt buộc nhưng **không đủ**.
> *Nguồn:* `→ report/54-...md`, `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-05-024** (SHOULD · C1) — Chuẩn Web Bundles (bundled HTTP responses) **NÊN** được dùng để đóng gói tài nguyên có chỉ mục truy cập ngẫu nhiên O(1) và chạy qua mmap; CSP ranh giới bundle phải được duy trì.
> *Nguồn:* `→ report/37-...md` · *Kiểm chứng:* runtime

### 3.6 Chứng thực thiết bị & danh tính phi tập trung

> **REQ-05-025** (SHOULD · C1) — Với giao dịch nhạy cảm, host **NÊN** dùng kiến trúc RATS (RFC 9334) kết hợp Play Integrity / App Attest để chứng thực thiết bị.
> *Nguồn:* `→ report/37-...md` · *Kiểm chứng:* runtime

> **REQ-05-026** (MAY · C2) — Danh tính publisher **CÓ THỂ** dùng Verifiable Credentials (W3C VC v2.0), DID và SD-JWT để xác minh chọn lọc mà không lộ dữ liệu thừa.
> *Nguồn:* `→ report/37-...md` · *Kiểm chứng:* static

### 3.7 Cập nhật, rollback & thu hồi

> **REQ-05-027** (MUST · C2) — Mỗi release **PHẢI** giữ digest tốt nhất đã biết (last-known-good) và hỗ trợ rollback một chạm.
> *Nguồn:* `→ report/02-...md`, `→ report/27-...md` · *Kiểm chứng:* review

> **REQ-05-028** (MUST · C1) — Cập nhật **PHẢI** chống rollback về phiên bản có lỗ hổng; client **PHẢI** từ chối siêu dữ liệu cũ hơn đã thấy.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-05-029** (SHOULD · C1) — Cập nhật **NÊN** dùng delta (VCDIFF RFC 3284) kèm nén từ điển (Zstd RFC 8878) để giảm băng thông, nhưng vẫn phải xác minh digest cuối.
> *Nguồn:* `→ report/27-...md`, `→ report/47-...md` · *Kiểm chứng:* runtime

> **REQ-05-030** (MUST · C2) — Khi có sự cố: **PHẢI** dừng phát hành, cách ly digest, chặn cài mới, thu hồi/hạn chế niềm tin, vô hiệu cache không an toàn, và thông báo theo mức độ nghiêm trọng.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* review

> **REQ-05-031** (MUST · C2) — Artifact đã phát hành **PHẢI** được lưu giữ bất biến kèm attestation trong suốt vòng đời và thời gian lưu trữ theo quy định.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* review

---

### 3.6 Chứng thực nguồn gốc & minh bạch chuỗi cung ứng

*Được bổ sung từ `report/135`, `report/137`, `report/140`, `report/143`.*

> **REQ-05-032** (SHOULD · C1) — Chứng thực thiết bị **NÊN** dùng **RFC 9711** (Entity Attestation Token, EAT) theo chu trình **RATS**: evidence → appraisal → attestation result.
> *Nguồn:* `→ report/135-...md` · *Kiểm chứng:* runtime

> **REQ-05-033** (SHOULD · C1) — EAT **NÊN** mang khẳng định định danh phần cứng, trạng thái khởi động (verified/unlocked) và phép đo phần mềm; media type theo **RFC 9782**.
> *Nguồn:* `→ report/135-...md` · *Kiểm chứng:* static

> **REQ-05-034** (SHOULD · C3) — Nội dung số do mini app tạo ra **NÊN** gắn **C2PA Technical Specification v2.1** (manifest nguồn gốc, chữ ký COSE, watermark).
> *Nguồn:* `→ report/137-...md` · *Kiểm chứng:* static

> **REQ-05-035** (SHOULD · C2) — Khai báo nguồn gốc quan trọng **NÊN** được ghi vào **IETF SCITT** (transparency ledger) để truy xuất và chống chối bỏ.
> *Nguồn:* `→ report/137-...md` · *Kiểm chứng:* runtime

> **REQ-05-036** (SHOULD · C2) — Bằng chứng minh bạch **NÊN** dùng Merkle tree proof với mốc thời gian **RFC 3161**; trạng thái chứng chỉ kiểm tra qua **RFC 6960** (OCSP) hoặc CRL.
> *Nguồn:* `→ report/137-...md` · *Kiểm chứng:* runtime

> **REQ-05-037** (SHOULD · C1) — Khóa phần cứng xác thực **NÊN** tra cứu **FIDO Alliance Metadata Service 3.0** (MDS 3.0) để biết trạng thái chứng nhận của thiết bị.
> *Nguồn:* `→ report/140-...md` · *Kiểm chứng:* runtime

> **REQ-05-038** (MUST · C1) — Khóa trong mọi khai báo **PHẢI** có định danh máy đọc được bằng **JWK Thumbprint** (RFC 7638); **KHÔNG** định danh khóa bằng tên tệp hay chuỗi tùy ý.
> *Nguồn:* `→ report/135-...md` · *Kiểm chứng:* static

> **REQ-05-039** (MUST · C1) — Đối tượng được ký (attestation, manifest, quyết định) **PHẢI** được mã hóa **CBOR xác định (deterministic, RFC 8949)** hoặc JSON canonical (RFC 8785) trước khi ký — bảo đảm cùng nội dung cho cùng digest.
> *Nguồn:* `→ report/143-...md`, `→ report/131-...md` · *Kiểm chứng:* static

> **REQ-05-040** (SHOULD · C1) — Chữ ký đối tượng nhị phân **NÊN** dùng **COSE** (RFC 9052/9053); **KHÔNG** trộn lẫn định dạng chữ ký khác nhau cho cùng loại đối tượng.
> *Nguồn:* `→ report/137-...md`, `→ report/143-...md` · *Kiểm chứng:* static

## 4. Ghi chú triển khai (informative)

**Phân công ba tầng (không được trộn lẫn):**

| Tầng | Công nghệ | Trả lời câu hỏi |
|---|---|---|
| Xây dựng | **SLSA** | Artifact này do ai, build thế nào? |
| Phân phối & cập nhật | **TUF** | Artifact này còn mới không? Có bị thay thế không? |
| Danh tính nhà phát triển | **Sigstore** | Ai chịu trách nhiệm ký artifact này? |
| Thành phần | **SBOM/CBOM + VEX** | Bên trong có gì? Có lỗ hổng khai thác được không? |

Một kiểm tra đơn lẻ **không** thay thế được kiểm tra khác. Ví dụ TLS/CDN
chứng minh kênh truyền an toàn — **không** chứng minh gói an toàn.

---

## 5. Tiêu chí tuân thủ

- [ ] TUF 4 vai trò + ngưỡng chữ ký; Root ngoại tuyến.
- [ ] Xác minh chuỗi Timestamp→Snapshot→Targets trước khi cài.
- [ ] Provenance SLSA gắn digest; không khóa ký trong build container.
- [ ] Sigstore verification đủ claim + Rekor inclusion proof.
- [ ] SBOM (CycloneDX/SPDX) đầy đủ; kiểm tra giấy phép.
- [ ] SRI/digest xác minh mọi nội dung tải.
- [ ] Chống rollback ở client; có last-known-good + rollback một chạm.
- [ ] Quy trình sự cố: dừng → cách ly → thu hồi → thông báo.

---

## 6. Cân nhắc an ninh & riêng tư

- **Neo tin cậy là khóa Root,** không phải TLS/CDN.
- **Chống phối trộn:** Snapshot ngăn ghép siêu metadata của hai phiên bản khác nhau.
- **Chống đóng băng:** Timestamp ngăn server giam client ở phiên bản cũ.
- **Danh tính ≠ an toàn:** keyless signing chỉ chứng minh "ai ký", không chứng minh "ký cái gì là tốt".
- **Khoảng trống còn mở:** hợp đồng ký cho từng sub-package — xem `report/06-gaps-validation-plan.md`.

---

## 7. Tham chiếu

- TUF Specification 1.x — https://theupdateframework.io/specification/latest/
- SLSA v1.0 — https://slsa.dev/spec/v1.0/
- Sigstore — https://docs.sigstore.dev/
- OWASP CycloneDX v1.6, LF SPDX 3.0, OpenVEX, in-toto
- IETF RFC 9052/9053 COSE, RFC 9421, RFC 9530, RFC 3284, RFC 8878
- Báo cáo nguồn: `report/18, 20, 27, 37, 39, 40, 41, 45, 47, 54`
