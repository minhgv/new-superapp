# SPEC-13 — API của Control Plane

| Trường | Giá trị |
|---|---|
| Spec ID | `SPEC-13` |
| Phiên bản | 1.1.0 |
| Trạng thái | Draft |
| Lớp tuân thủ | **C2** (Store / Control plane) |
| Nguồn | `report/02`, `report/11`, `report/41`, `report/59`, `report/42`, `report/53` |

---

## 1. Phạm vi & mục tiêu

Đặc tả **hồ sơ giao diện lập trình** của control plane: nhóm tài nguyên,
phân trang, xác thực, hợp đồng lỗi, webhook và vòng đời API.

Mục tiêu: API ổn định, có phiên bản, an toàn theo mặc định và đủ để
tự động hóa toàn bộ vòng đời phát hành.

> **Chuẩn tham chiếu:** OpenAPI 3.1.0, IETF RFC 7807, RFC 9421, RFC 8941,
> OAuth 2.0 / OIDC, Standard Webhooks.

---

## 2. Thuật ngữ

| Thuật ngữ | Định nghĩa |
|---|---|
| **Control plane API** | REST API quản lý publisher, app, release, review. |
| **Cursor pagination** | Phân trang bằng con trỏ, ổn định khi dữ liệu thay đổi. |
| **Idempotency key** | Khóa chống lặp cho thao tạo/sửa. |
| **ETag** | Thẻ thực thể để đọc có điều kiện. |
| **Problem Details** | Định dạng lỗi chuẩn RFC 7807. |

---

## 3. Yêu cầu chuẩn hóa

### 3.1 Hồ sơ API chung

> **REQ-13-001** (MUST · C2) — API control plane **PHẢI** được mô tả bằng OpenAPI **3.1.0** và có phiên bản tường minh trong đường dẫn (ví dụ `/v1`).
> *Nguồn:* `→ report/02-...md`, `→ report/41-...md` · *Kiểm chứng:* static

> **REQ-13-002** (MUST · C2) — Xác thực **PHẢI** dùng OAuth 2.0 / OIDC với phân quyền theo vai trò (role-scoped); token phải bị thu hồi được.
> *Nguồn:* `→ report/02-...md`, `→ report/38-...md` · *Kiểm chứng:* runtime

> **REQ-13-003** (MUST · C2) — Mọi endpoint **PHẢI** yêu cầu HTTPS; **KHÔNG** có endpoint nhạy cảm qua HTTP.
> *Nguồn:* `→ report/39-...md` · *Kiểm chứng:* static

> **REQ-13-004** (MUST · C2) — Đọc danh mục **PHẢI** dùng phân trang con trỏ và hỗ trợ `ETag`/`If-None-Match` cho cache hiệu quả.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* runtime

> **REQ-13-005** (MUST · C2) — Thao tác nộp và phát hành **PHẢI** hỗ trợ khóa idempotency (`Idempotency-Key`, UUIDv4).
> *Nguồn:* `→ report/02-...md`, `→ report/33-...md` · *Kiểm chứng:* runtime

> **REQ-13-006** (MUST · C2) — Lỗi **PHẢI** dùng Problem Details (RFC 7807) với `type`, `title`, `status`, `detail`, `instance`; mã lỗi nghiệp vụ phải ổn định giữa các phiên bản.
> *Nguồn:* `→ report/41-...md`, `→ report/59-...md` · *Kiểm chứng:* static

> **REQ-13-007** (MUST · C2) — Phân quyền **PHẢI** theo vai trò tối thiểu; một vai trò **KHÔNG** được có quyền vượt quá chức năng cần thiết.
> *Nguồn:* `→ report/02-...md`, `→ report/38-...md` · *Kiểm chứng:* review

> **REQ-13-008** (MUST · C2) — Mọi thao tác ghi **PHẢI** sinh sự kiện kiểm toán append-only với `correlation_id`.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* static

### 3.2 Nhóm tài nguyên & endpoint

| Nhóm đường dẫn | Phương thức | Mục đích |
|---|---|---|
| `/publishers` | GET, POST, PATCH | Quản lý nhà phát triển |
| `/apps` | GET, POST, PATCH | Quản lý ứng dụng |
| `/apps/{id}/versions` | GET, POST | Quản lý phiên bản |
| `/submissions` | GET, POST | Nộp bản phát hành |
| `/reviews` | GET, POST, PATCH | Quy trình kiểm duyệt |
| `/releases` | GET, POST, PATCH | Điều phối phát hành |
| `/compatibility` | POST | Kiểm tra tương thích |
| `/incidents` | GET, POST, PATCH | Sự cố & thu hồi |
| `/deprecations` | GET, POST | Ngừng hỗ trợ |
| `/audit-events` | GET | Nhật ký kiểm toán (append-only) |

> **REQ-13-009** (MUST · C2) — API **PHẢI** cung cấp đủ 10 nhóm tài nguyên ở bảng trên với ngữ nghĩa CRUD tương ứng.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* static

> **REQ-13-010** (MUST · C2) — `/compatibility` **PHẢI** trả về hai kết quả riêng: `installable` và `launchable`, kèm lý do khi không khả dụng.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* runtime

> **REQ-13-011** (MUST · C2) — `/audit-events` **PHẢI** là chỉ đọc, phân trang con trỏ, và **KHÔNG** cho phép ghi/xóa/sửa.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* static

> **REQ-13-012** (SHOULD · C2) — API **NÊN** hỗ trợ bộ lọc theo `app_id`, `publisher_id`, `release_id`, khoảng thời gian và trạng thái.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* runtime

### 3.3 Webhook & sự kiện

> **REQ-13-013** (MUST · C2) — Webhook **PHẢI** được ký bằng HTTP Message Signatures (RFC 9421) hoặc HMAC theo Standard Webhooks (`webhook-signature`, `webhook-timestamp`, `webhook-id`).
> *Nguồn:* `→ report/41-...md` · *Kiểm chứng:* runtime

> **REQ-13-014** (MUST · C2) — Chống phát lại **PHẢI** dùng `webhook-timestamp` với cửa sổ thời gian hợp lệ; chống trùng lặp dùng `webhook-id`.
> *Nguồn:* `→ report/41-...md` · *Kiểm chứng:* runtime

> **REQ-13-015** (SHOULD · C2) — Sự kiện webhook **NÊN** dùng định dạng CloudEvents (binary hoặc structured HTTP framing) để nhất quán siêu dữ liệu.
> *Nguồn:* `→ report/42-...md` · *Kiểm chứng:* static

> **REQ-13-016** (MUST · C2) — Webhook **PHẢI** có cơ chế thử lại có giới hạn và cảnh báo khi điểm cuối không phản hồi.
> *Nguồn:* `→ report/41-...md` · *Kiểm chứng:* runtime

> **REQ-13-017** (MUST · C2) — Toàn vẹn tải trọng **PHẢI** được bảo đảm bằng Digest Fields (RFC 9530) khi truyền dữ liệu nhạy cảm.
> *Nguồn:* `→ report/41-...md` · *Kiểm chứng:* runtime

### 3.4 Hợp đồng dự phòng & lỗi

> **REQ-13-018** (MUST · C2) — Hợp đồng lỗi nghiệp vụ **PHẢI** phân biệt: `validation_error`, `conflict`, `not_found`, `forbidden`, `rate_limited`, `quota_exceeded`, `dependency_unavailable`, `internal`.
> *Nguồn:* `→ report/11-...md`, `→ report/59-...md` · *Kiểm chứng:* static

> **REQ-13-019** (MUST · C2) — Phản hồi bị giới hạn tốc độ **PHẢI** kèm header `Retry-After` và ngữ nghĩa lùi bước rõ ràng.
> *Nguồn:* `→ report/11-...md`, `→ report/17-...md` · *Kiểm chứng:* runtime

> **REQ-13-020** (SHOULD · C2) — Lỗi phụ thuộc **NÊN** dùng RFC 9457/9440 (API resilience contracts) để phân biệt lỗi của hệ thống với lỗi của bên thứ ba.
> *Nguồn:* `→ report/59-...md` · *Kiểm chứng:* static

### 3.5 Hạn ngạch & giới hạn

> **REQ-13-021** (MUST · C2) — API **PHẢI** công bố hạn ngạch cho từng tenant và trả thông báo khi gần hết hạn ngạch.
> *Nguồn:* `→ report/11-...md` · *Kiểm chứng:* runtime

> **REQ-13-022** (MUST · C2) — Giới hạn tải **PHẢI** công bằng giữa các tenant; một tenant **KHÔNG** được chiếm toàn bộ dung lượng.
> *Nguồn:* `→ report/11-...md`, `→ report/17-...md` · *Kiểm chứng:* runtime

### 3.6 Vòng đời API

> **REQ-13-023** (MUST · C2) — API **PHẢI** có chính sách phiên bản và ngừng hỗ trợ: biến cũ bị đánh dấu `deprecated` trước khi bị gỡ, kèm hướng dẫn chuyển đổi.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* review

> **REQ-13-024** (SHOULD · C2) — API **NÊN** dùng tín hiệu HTTP `Deprecation`/`Sunset` máy đọc được cho từng endpoint.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* static

> **REQ-13-025** (MUST · C2) — Thay đổi không tương thích **PHẢI** đi kèm phiên bản API mới; **KHÔNG** đổi âm thầm ngữ nghĩa hiện có.
> *Nguồn:* `→ report/02-...md`, `→ report/09-...md` · *Kiểm chứng:* review

---

### 3.7 Định danh, truy vấn & minh bạch cache

*Bổ sung từ `report/140`, `report/141`, `report/143`.*

> **REQ-13-026** (MUST · C2) — Định danh tài nguyên **PHẢI** dùng **UUIDv7** (RFC 9562) để đơn điệu theo thời gian, tránh bội chi và truy vấn theo thứ tự được; **KHÔNG** dùng UUIDv4 cho đối tượng có thứ tự nghiệp vụ.
> *Nguồn:* `→ report/141-...md` · *Kiểm chứng:* static

> **REQ-13-027** (SHOULD · C2) — Truy vấn có cấu trúc **NÊN** dùng **RFC 9535** (JSONPath) cho bộ lọc máy đọc được; biểu thức **PHẢI** thuộc tập an toàn (không ReDoS).
> *Nguồn:* `→ report/140-...md` · *Kiểm chứng:* static

> **REQ-13-028** (SHOULD · C2) — Phản hồi **NÊN** tách bạch cache theo **RFC 9209** (`CDN-Cache-Control`, `Surrogate-Control`) và **RFC 9211** (`Cache-Status`) để chẩn đoán được cache đang dùng hay bỏ qua.
> *Nguồn:* `→ report/141-...md` · *Kiểm chứng:* runtime

> **REQ-13-029** (MAY · C2) — Liên kết giữa tài nguyên **CÓ THỂ** dùng **RFC 8288** (Web Linking) và kênh sự kiện **CÓ THỂ** dùng WebSub; **PHẢI** xác minh chủ đề qua chữ ký.
> *Nguồn:* `→ report/141-...md`, `→ report/109-...md` · *Kiểm chứng:* static

## 4. Ghi chú triển khai (informative)

**Ví dụ phản hồi lỗi (RFC 7807):**

```json
{
  "type": "https://api.store.example/problems/validation-error",
  "title": "Manifest validation failed",
  "status": 422,
  "detail": "pages[0] points to a missing file",
  "instance": "/v1/submissions/sub_01H8...",
  "correlation_id": "corr_9f2a..."
}
```

**Nguyên tắc thiết kế:**
- Idempotency cho mọi thao tác có tác dụng phụ.
- Phân trang con trỏ — không dùng offset vì dữ liệu thay đổi liên tục.
- Mọi thao tác ghi đều sinh sự kiện kiểm toán.
- Không bao giờ lộ stack trace nội bộ trong phản hồi.

---

## 5. Tiêu chí tuân thủ

- [ ] OpenAPI 3.1.0 đầy đủ, có phiên bản trong đường dẫn.
- [ ] OAuth 2.0/OIDC + phân quyền theo vai trò.
- [ ] Phân trang con trỏ + ETag cho đọc danh mục.
- [ ] Idempotency-Key cho thao tạo/sửa.
- [ ] Problem Details (RFC 7807) cho mọi lỗi.
- [ ] Đủ 10 nhóm tài nguyên.
- [ ] Webhook có chữ ký + chống phát lại + chống trùng.
- [ ] Hạn ngạch công bằng + `Retry-After`.
- [ ] Chính sách phiên bản và ngừng hỗ trợ tường minh.

---

## 6. Cân nhắc an ninh & riêng tư

- **Không lộ nội bộ:** lỗi là mã + thông điệp chung, không có đường dẫn tệp.
- **Chống giả mạo webhook:** thiếu chữ ký = từ chối xử lý.
- **Phân quyền tối thiểu:** một token bị lộ không được phép phá toàn bộ hệ thống.
- **Nhật ký kiểm toán bất biến:** append-only, không có endpoint xóa.

---

## 7. Tham chiếu

- OpenAPI 3.1.0 — https://spec.openapis.org/oas/v3.1.0
- IETF RFC 7807 Problem Details, RFC 9421 HTTP Signatures, RFC 8941 Structured Fields, RFC 9530 Digest, RFC 9457/9440
- Standard Webhooks — https://standardwebhooks.com/
- CNCF CloudEvents v1.0.2
- Báo cáo nguồn: `report/02, 11, 17, 33, 41, 42, 59`
