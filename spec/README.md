# Mini App Store Specification Suite (SPEC)

Bộ tài liệu đặc tả (specification) chuẩn hóa cho **Mini App Store trên Super App**,
được tổng hợp ở thời điểm *iteration 95* từ 92 báo cáo nghiên cứu và 1.376 bằng chứng
đã kiểm chứng (`report/`, `evidence/findings.jsonl`).

> **Lưu ý về số liệu:** vòng nghiên cứu Deli Deep vẫn chạy liên tục và tiếp tục
> bổ sung báo cáo + findings vào `report/` và `evidence/findings.jsonl`. Các con số
> trong tài liệu này là số liệu **tại thời điểm tổng hợp**, không phải tổng hiện tại.
> Số liệu mới nhất nằm trực tiếp ở hai file nguồn đó.

> Bộ tài liệu này **tổng hợp** nghiên cứu thành các yêu cầu chuẩn hóa (normative requirements).
> Nhật ký nghiên cứu gốc (theo iteration) vẫn được giữ nguyên tại [`../report/`](../report/) để tra cứu nguồn.

---

## 1. Mô hình tuân thủ (Conformance model)

Tài liệu dùng từ khóa RFC 2119 / RFC 8174:

| Từ khóa | Ý nghĩa |
|---|---|
| **MUST / MUST NOT / REQUIRED / SHALL** | Bắt buộc. Vi phạm = không đạt tuân thủ. |
| **SHOULD / SHOULD NOT / RECOMMENDED** | Khuyến nghị mạnh. Phải có lý do ghi nhận nếu lệch. |
| **MAY / OPTIONAL** | Tùy chọn, không ảnh hưởng tuân thủ. |

**Ba lớp tuân thủ (conformance classes):**

| Lớp | Đối tượng | Bắt buộc tuân thủ |
|---|---|---|
| **C1 — Host / Container** | Super app, native container, runtime mini-app | SPEC-02, 04, 05, 06, 10 |
| **C2 — Store / Control plane** | Hệ thống phát hành, review, catalog | SPEC-08, 13, 05, 11 |
| **C3 — Mini App / Developer** | Nhà phát triển mini-app | SPEC-01, 03, 12 |

Một phiên bản mini-app chỉ được **publish** khi cả C1, C2, C3 đều đạt các cổng (gate)
tại [APPENDIX-B](APPENDIX-B-conformance-checklist.md).

---

## 2. Chỉ mục tài liệu

| ID | Tài liệu | Nội dung | Lớp |
|---|---|---|---|
| **SPEC-00** | [Scope & Conformance](SPEC-00-scope-conformance.md) | Phạm vi, thuật ngữ, định danh yêu cầu, quy trình đánh giá | Tất cả |
| **SPEC-01** | [Package, Manifest & Addressing](SPEC-01-package-manifest-addressing.md) | Đóng gói `.zip`, `manifest.json`, URI deep-link, phiên bản, subpackage | C3 |
| **SPEC-02** | [Runtime, Lifecycle & Host Bridge](SPEC-02-runtime-lifecycle-bridge.md) | Kiến trúc đa luồng, vòng đời app/trang, JSBridge, cross-context | C1 |
| **SPEC-03** | [Developer API Surface](SPEC-03-developer-api-surface.md) | API tham chiếu cho lập trình viên: JSBridge + Web API | C3 |
| **SPEC-04** | [Security & Sandboxing](SPEC-04-security-sandboxing.md) | XSS, CSP, sandbox, cô lập tiến trình, WASM, Trusted Types | C1 |
| **SPEC-05** | [Supply Chain & Signing](SPEC-05-supply-chain-signing.md) | TUF, SLSA, Sigstore, SBOM/CBOM, attestation, anti-rollback | C1+C2 |
| **SPEC-06** | [Capabilities & Permissions](SPEC-06-capabilities-permissions.md) | Permissions Policy, quyền thiết bị, cảm biến, delegation | C1 |
| **SPEC-07** | [Privacy & Data Governance](SPEC-07-privacy-data-governance.md) | Privacy, telemetry, storage, attribution, consent | C1+C2 |
| **SPEC-08** | [Store Control Plane](SPEC-08-store-control-plane.md) | Publisher, submission, review, release lifecycle, catalog | C2 |
| **SPEC-09** | [Commerce & Billing](SPEC-09-commerce-billing.md) | Thanh toán, digital goods, subscription, settlement, PCI DSS | C2 |
| **SPEC-10** | [Reliability & Performance](SPEC-10-reliability-performance.md) | SLO, quota, resource governance, crash, ANR, power | C1 |
| **SPEC-11** | [Compliance & Regionalization](SPEC-11-compliance-regionalization.md) | Age rating, child safety, localization, accessibility, luật khu vực | C2 |
| **SPEC-12** | [Review Guidelines](SPEC-12-review-guidelines.md) | Quy tắc review cửa hàng, checklist nhà phát triển | C2+C3 |
| **SPEC-13** | [Control-Plane API](SPEC-13-control-plane-api.md) | Hồ sơ REST API của control plane (OpenAPI) | C2 |
| **A** | [Capability Catalog](APPENDIX-A-capability-catalog.md) | Danh mục năng lực Web Platform theo nhóm | Tham khảo |
| **B** | [Conformance Checklist](APPENDIX-B-conformance-checklist.md) | Cổng kiểm tra tuân thủ có thể kiểm chứng tự động | Tất cả |

---

## 3. Nguồn bằng chứng

- **Báo cáo nghiên cứu gốc**: [`../report/`](../report/) — các tài liệu lõi (01–06) và các chuyên đề theo iteration.
- **Bằng chứng thô**: [`../evidence/findings.jsonl`](../evidence/findings.jsonl) — các bản ghi append-only, mỗi bản ghi có URL nguồn và mức bằng chứng (`normative_standard` / `industry_standard` / `platform_practice`).
- **Phương pháp**: [`../methodology/README.md`](../methodology/README.md) — quy trình kiểm chứng URL, phân loại bằng chứng, tách chuẩn chính thức khỏi thực hành nhà cung cấp.

### Thỏa thuận dẫn nguồn
Mọi yêu cầu trong bộ SPEC đánh dấu nguồn bằng một trong hai dạng:
- `→ report/<file>`: dẫn tới báo cáo nghiên cứu gốc.
- `→ FINDING <id>`: dẫn tới bản ghi trong `evidence/findings.jsonl`.

Yêu cầu **không có nguồn** là đề xuất kiến trúc (proposal) của nhóm thiết kế, được
đánh dấu rõ `[PROPOSAL]` — không phải yêu cầu chuẩn quốc tế.

---

## 4. Phiên bản

| Phiên bản | Ngày | Thay đổi |
|---|---|---|
| 1.0.0 | 2026-10 | Tổng hợp lần đầu từ các báo cáo nghiên cứu (tới iteration 95) |

Trạng thái hiện tại: **Draft**. Bộ tài liệu sẽ được nâng phiên bản khi có thêm bằng chứng
hoặc quyết định kiến trúc mới. Các phát hiện lịch sử không bị ghi đè; chỉ bổ sung.
