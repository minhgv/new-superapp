# SPEC-00 — Phạm vi, Thuật ngữ & Mô hình Tuân thủ

| Trường | Giá trị |
|---|---|
| Spec ID | `SPEC-00` |
| Phiên bản | 1.0.0 |
| Trạng thái | Draft |
| Lớp tuân thủ | Tất cả (C1, C2, C3) |
| Nguồn | `report/01-executive-summary.md`, `report/02-reference-architecture-lifecycle.md`, `report/06-gaps-validation-plan.md` |

---

## 1. Phạm vi (Scope)

Bộ đặc tả này định nghĩa **hợp đồng kỹ thuật** giữa ba bên:

1. **Host / Container** — siêu ứng dụng (super app) cung cấp runtime, tài nguyên thiết bị và sandbox.
2. **Store / Control plane** — hệ thống phát hành, kiểm duyệt, phân phối và cập nhật mini-app.
3. **Publisher / Mini App** — nhà phát triển đóng gói, ký số, nộp và vận hành mini-app.

**Trong phạm vi:** định dạng gói, manifest, định danh, vòng đời runtime, bề mặt API,
an ninh sandbox, chuỗi cung ứng, quyền thiết bị, riêng tư dữ liệu, luồng kiểm duyệt,
thương mại điện tử, độ tin cậy/hiệu năng, tuân thủ pháp lý và API control plane.

**Ngoài phạm vi:** thiết kế UI/UX chi tiết của từng mini-app, mô hình kinh doanh cụ thể
của nhà phát triển, hạ tầng mạng lõi của nhà cung cấp viễn thông, và các chuẩn không
liên quan tới việc chạy/kiểm duyệt mini-app.

---

## 2. Thuật ngữ và định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Mini App** | Ứng dụng nhỏ, đóng gói tự chứa (ZIP), chạy trong container của host, dùng Web/JS API được ủy quyền. |
| **Host (Container)** | Phần mềm gốc (native) tải, xác thực, khởi chạy và cách ly mini-app; sở hữu cây giao diện gốc. |
| **View Layer** | Tầng hiển thị (WebView / native component) chỉ phụ trách render. |
| **Logic Layer** | Tầng nghiệp vụ chạy trong JavaScript Worker (JSCore/V8), tách khỏi View. |
| **JSBridge** | Kênh giao tiếp có kiểm soát giữa Logic Layer và Host, cho phép gọi năng lực gốc. |
| **Package** | Tệp ZIP chứa `manifest.json`, `app.js`, `app.css`, `pages/` — đơn vị phát hành. |
| **Release** | Một phiên bản bất biến của package (digest cố định) đã qua kiểm duyệt. |
| **Digest** | Băm mật mã (SHA-256) định danh duy nhất một gói phát hành. |
| **Control plane** | Tầng quyết định "được phép phát hành gì". |
| **Data plane** | Tầng quyết định "được phép tải/chạy gì trên một host cụ thể". |
| **Capability** | Năng lực hệ thống (camera, vị trí, thanh toán…) mà mini-app yêu cầu qua Permissions Policy. |
| **Gate (Cổng)** | Điều kiện bắt buộc phải đạt trước khi chuyển trạng thái phát hành. |
| **SBOM / CBOM** | Bill of Materials phần mềm / mật mã, kê khai thành phần và thuật toán. |
| **Attestation** | Chứng thực mật mã về nguồn gốc, công cụ build và tính toàn vẹn của gói. |

**Nguyên tắc tách bạch bắt buộc:** control plane và data plane **KHÔNG** dùng chung
ranh giới tin cậy ngầm. Quyết định kiểm duyệt ≠ quyết định cho phép chạy.
→ `report/02-reference-architecture-lifecycle.md`

---

## 3. Định danh yêu cầu (Requirement ID)

Mọi yêu cầu chuẩn hóa mang định danh:

```
REQ-<SPEC-SỐ>-<3 CHỮ SỐ>   ví dụ: REQ-01-014
```

Trường đi kèm:

| Trường | Ý nghĩa |
|---|---|
| **Level** | `MUST` / `SHOULD` / `MAY` |
| **Class** | Lớp tuân thủ chịu trách nhiệm: C1 / C2 / C3 |
| **Source** | `→ report/...` hoặc `→ FINDING <id>` hoặc `[PROPOSAL]` |
| **Verify** | Cách kiểm chứng (tự động / review thủ công / runtime assertion) |

---

## 4. Quy trình đánh giá tuân thủ

Mỗi release đi qua bốn lớp kiểm tra, tất cả phải ghi nhận bằng chứng theo `release_id`:

1. **Kiểm tra tĩnh (static gate)** — schema manifest, cấu trúc gói, kích thước, đường dẫn,
   thuật ngữ khai báo, quét secret/malware, SBOM hợp lệ.
2. **Ký số & chuỗi cung ứng (provenance gate)** — xác minh chữ ký, chuỗi TUF
   `Timestamp → Snapshot → Targets`, attestation SLSA/Sigstore khớp digest.
3. **Kiểm duyệt nội dung (review gate)** — người + máy đánh giá nội dung, quyền riêng tư,
   quyền hạn, UX, an ninh, chính sách, độ tuổi.
4. **Phát hành có kiểm soát (rollout gate)** — internal → preview → canary → staged → active,
   kèm ngưỡng sức khỏe và cơ chế dừng/rollback tự động.

> **Nguyên tắc bất biến:** một release **KHÔNG** được chuyển sang `active` trừ khi
> digest gói, danh tính publisher, kết quả tương thích, chính sách quyền hạn,
> kết quả kiểm tra tự động, quyết định reviewer, đích phát hành và đích rollback
> đều tra cứu được theo `release_id`.
> → `report/02-reference-architecture-lifecycle.md`

---

## 5. Máy trạng thái phát hành

```text
draft → submitted → automated_checks_passed → human_review → approved
  → internal → preview → canary → staged → active

Trạng thái phụ: rejected, needs_changes, quarantined, paused, aborted,
                withdrawn, deprecated, sunset_scheduled, blocked
```

Mọi chuyển trạng thái **PHẢI** được ủy quyền và ghi vào sổ cái append-only
(actor, action, resource, timestamp, reason, policy_version, before/after, correlation_id).

---

## 6. Chính sách bằng chứng (Evidence policy)

- Chuẩn quy phạm (normative) phải tách biệt khỏi thực hành nhà cung cấp (vendor practice).
- Khuyến nghị phải ghi nhãn đề xuất, không trình bày như yêu cầu phổ quát.
- Khoảng trống bằng chứng được giữ nguyên là khoảng trống, không lấp bằng giả định.
- Mọi bằng chứng phải truy được về URL nguồn và mức bằng chứng.
- Thông tin xác thực (credential) của nguồn tuyệt đối không được đưa vào báo cáo hay repo.

→ `../methodology/README.md`, `report/06-gaps-validation-plan.md`

---

## 7. Các quyết định còn mở

Các mục sau chưa có bằng chứng đầy đủ và **PHẢI** được quyết định trước khi đưa ra production:

1. Hợp đồng ký số cho **sub-package** (chuẩn W3C MiniApp Packaging chưa kết thúc phần này).
2. Ngưỡng chính sách **cross-tenant build isolation** (SLSA L2 hay L3) cho các gói nhạy cảm.
3. Mô hình **quyền ưu tiên vận hành** (host entitlement) cho năng lực chạy nền/nghe nhạc nền.
4. Chi tiết **phân giải nhà cung cấp gói** (provider → package-registry mapping) cho deep-link.

→ `report/06-gaps-validation-plan.md`

---

## 8. Tài liệu tham chiếu chuẩn

| Chuẩn | Phạm vi áp dụng |
|---|---|
| W3C MiniApp Manifest / Packaging / Addressing / Lifecycle | SPEC-01, SPEC-02 |
| W3C MiniApp Standardization White Paper v2 | SPEC-02 |
| W3C Permissions Policy, W3C Permissions | SPEC-06 |
| TUF Specification 1.x | SPEC-05 |
| SLSA v1.0 (Build track) | SPEC-05 |
| Sigstore (Fulcio / Rekor / Cosign) | SPEC-05 |
| OWASP CycloneDX v1.6, LF SPDX 3.0, OpenVEX, in-toto | SPEC-05 |
| OWASP MASVS v2.0, OWASP ASVS | SPEC-04 |
| NIST SP 800-218 (SSDF), NIST SP 800-161 (C-SCRM) | SPEC-05, SPEC-08 |
| IETF RFC 9421 (HTTP Signatures), RFC 7807 (Problem Details) | SPEC-13 |
| PCI DSS v4.0.1 | SPEC-09 |
| WCAG 2.2 Level AA | SPEC-11 |
