# SPEC-01 — Đóng gói, Manifest & Định danh Mini App

| Trường | Giá trị |
|---|---|
| Spec ID | `SPEC-01` |
| Phiên bản | 1.0.0 |
| Trạng thái | Draft |
| Lớp tuân thủ | **C3** (Mini App / Publisher) — kiểm tra bởi C2 |
| Nguồn | `report/18-iteration-22-w3c-miniapp-tuf-device-governance.md`, `report/46-iteration-50-w3c-miniapp-suite-bluetooth-nfc-sensors.md`, `report/60-iteration-64-manifest-shortcuts-popover-anchor-subcapture.md`, `report/10-iteration-5-portable-catalog-and-evidence-federation.md`, `report/02-reference-architecture-lifecycle.md` |

---

## 1. Phạm vi & mục tiêu

Đặc tả hợp đồng **đóng gói**, **kê khai** và **định danh** của một mini-app:
cách một ứng dụng được cấu trúc thành tệp ZIP, khai báo siêu dữ liệu trong
`manifest.json`, và được tìm thấy / khởi chạy bằng định danh chuẩn.

Mục tiêu: bảo đảm mọi mini-app trong cửa hàng **không thể mơ hồ** về
định danh, phiên bản, dung lượng, đường dẫn và nguồn gốc — từ đó cho phép
kiểm duyệt tự động và xác thực toàn vẹn ở phía host.

> **Chuẩn tham chiếu cốt lõi:** W3C MiniApp Manifest, W3C MiniApp Packaging,
> W3C MiniApp Addressing, W3C MiniApp Standardization White Paper v2.

---

## 2. Thuật ngữ

| Thuật ngữ | Định nghĩa |
|---|---|
| **Package** | Tệp ZIP chứa toàn bộ mã nguồn và tài nguyên của mini-app. |
| **Main package** | Gói chính, luôn được tải đầu tiên. |
| **Sub-package** | Gói phụ, tải động theo nhu cầu (on-demand). |
| **Manifest** | Tệp `manifest.json` ở thư mục gốc — hợp đồng siêu dữ liệu. |
| **Entry point** | Trang khởi chạy, là `pages[0]` trong manifest. |
| **Digest** | Băm mật mã SHA-256 của gói phát hành. |
| **Provider / Host namespace** | Không gian định danh logic — **không** phải vị trí tải xuống. |
| **identify** | Thành phần URI chứa `id` bắt buộc và `;version=` tùy chọn. |

---

## 3. Yêu cầu chuẩn hóa

### 3.1 Hợp đồng Manifest

> **REQ-01-001** (MUST · C3) — Tệp `manifest.json` **PHẢI** nằm ở thư mục gốc của gói và là tài liệu JSON hợp lệ theo lược đồ W3C MiniApp Manifest.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-01-002** (MUST · C3) — Gốc manifest **PHẢI** chứa đủ các khóa bắt buộc: `app_id`, `icons`, `name`, `pages`, `platform_version`, `version`.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-01-003** (MUST · C3) — Mỗi mục trong `icons` **PHẢI** có thuộc tính `src` trỏ tới tài nguyên biểu tượng tồn tại trong gói.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-01-004** (MUST · C3) — Danh sách `pages` **PHẢI** liệt kê các đường dẫn trang dạng tương đối; mục đầu tiên (`pages[0]`) là trang chủ / điểm vào mặc định.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-01-005** (SHOULD · C2) — Kiểm duyệt **NÊN** xuất trang khởi chạy mặc định từ `pages[0]` và không cho phép manifest trỏ vào tệp không tồn tại trong gói.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-01-006** (MUST · C3) — `app_id` và `version` **PHẢI** ổn định và tuân theo SemVer cho các lần phát hành của cùng một ứng dụng.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* static

> **REQ-01-007** (MUST · C3) — `color_scheme` **CHỈ ĐƯỢC** nhận giá trị `auto`, `light` hoặc `dark`; giá trị này ghi đè các màu cửa sổ liên quan.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-01-008** (MAY · C3) — `req_permissions` **CÓ THỂ** khai báo các tính năng hệ thống cụ thể (`location`, `contacts`, `sensors`, `camera`…) kèm trường `reason` giải thích; user agent xin phép đồng ý và cửa hàng **CÓ THỂ** lọc mini-app dựa trên dữ liệu này, chính sách riêng tư, tùy chọn người dùng hoặc khả năng thiết bị.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* review

> **REQ-01-009** (MAY · C3) — `widgets` **CÓ THỂ** khai báo các tiện ích bằng cặp `name`/`path`; dữ liệu widget **PHẢI** được cô lập với host và giữa các widget với nhau.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-01-010** (MAY · C3) — Đối tượng `window` **CỌ THỂ** cấu hình điều hướng/tiêu đề/nền, toàn màn hình, kéo để làm mới và hướng màn hình.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-01-011** (MUST · C2) — Manifest **KHÔNG ĐƯỢC** coi `theme_color`, `scope` và `shortcuts` của Web App Manifest là được hỗ trợ: các khóa này **KHÔNG** được các user agent MiniApp hỗ trợ hiện tại. Chúng **PHẢI** bị coi là không tương thích (non-portable) thay vì âm thầm giả định hành vi PWA.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-01-012** (SHOULD · C2) — Các trường `name`, `description` và mô tả siêu dữ liệu hiển thị **NÊN** được bản địa hóa và kiểm tra độ dài/ngôn ngữ phù hợp danh mục.
> *Nguồn:* `→ report/10-...md` · *Kiểm chứng:* review

### 3.2 Đóng gói (Packaging)

> **REQ-01-013** (MUST · C3) — Gói tuân thủ **PHẢI** dùng một thư mục gốc kèm thư mục `pages`, và **PHẢI** chứa `manifest.json`, `app.js`, `app.css` ở gốc; toàn bộ gói được bọc trong tệp ZIP.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-01-014** (MUST · C3) — Tài nguyên trang được khớp theo tên tệp gốc (cấu trúc HTML, CSS và JS cùng basename) cho từng tuyến đã liệt kê trong manifest.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-01-015** (MAY · C3) — Thư mục `common/` và `i18n/` là tùy chọn; tên tệp i18n **PHẢI** dùng thẻ ngôn ngữ BCP 47.
> *Nguồn:* `→ report/12-...md`, `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-01-016** (MUST · C3) — Thùng ZIP **PHẢI** dùng tên tệp UTF-8 và CRC32 cho từng tệp; **KHÔNG ĐƯỢC** dùng mã hóa ZIP, tệp chia mảnh/spanned, hoặc tính năng có bản quyền.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-01-017** (MUST · C2) — Loại MIME khi phân phối **PHẢI** là `application/miniapp-pkg+zip`; trình tự xử lý là: tải → xác minh → giải nén → phân tích `manifest.json` → chuẩn bị runtime → chọn trang khởi chạy.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-01-018** (MUST · C2) — Sandbox khi xử lý gói **PHẢI** chặn theo mặc định các tính năng mạnh, áp dụng hạn chế CSP/tài nguyên, và **loại bỏ** mọi tài nguyên trong gói không được khai báo.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* static + runtime

> **REQ-01-019** (MUST NOT · C2) — **KHÔNG ĐƯỢC** khuyến nghị mã hóa bảo mật nội dung gói trong đặc tả này; tính toàn vẹn và chữ ký là cơ chế bảo vệ, không phải mã hóa gói.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* review

> **REQ-01-020** (SHOULD · C2) — Khối ký tùy chọn **NÊN** đặt ngay trước bảng mục lục trung tâm (central directory) của ZIP, theo định dạng khối ký RPK (`RPK Sig Block 42` với trường chiều dài uint64/uint32, cặp ID-giá trị, phần digest/chứng chỉ/chữ ký).
> *Nguồn:* `→ report/18-...md`, `→ report/37-...md` · *Kiểm chứng:* static

### 3.3 Sub-package và giới hạn dung lượng

> **REQ-01-021** (MUST · C3) — Gói chính **KHÔNG ĐƯỢC** vượt quá **4 MB**; tổng toàn bộ gói (chính + phụ) **KHÔNG ĐƯỢC** vượt quá **20 MB**.
> *Nguồn:* `→ report/46-...md` · *Kiểm chứng:* static

> **REQ-01-022** (SHOULD · C2) — Nền tảng **NÊN** hỗ trợ tách gói phụ, tải động theo nhu cầu, dọn cache theo chính sách và tải lũy tiến (progressive loading).
> *Nguồn:* `→ report/18-...md`, `→ report/46-...md` · *Kiểm chứng:* runtime

> **REQ-01-023** (MUST · C2) — Vì chuẩn W3C MiniApp Packaging **chưa kết thúc** hợp đồng chữ ký cho từng sub-package, mọi digest/chữ ký riêng cho gói phụ **PHẢI** được định nghĩa là **mở rộng nền tảng tường minh** (`[PROPOSAL]`), kèm tài liệu hợp đồng riêng.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* review

> **REQ-01-024** (MUST · C2) — Mỗi sub-package **PHẢI** có digest riêng được ghi vào siêu dữ liệu mục tiêu (TUF Targets) để xác minh độc lập khi tải động.
> *Nguồn:* `→ report/18-...md`, `→ report/41-...md` · *Kiểm chứng:* static

### 3.4 Định danh & điều hướng (Addressing)

> **REQ-01-025** (MUST · C2) — URI mini-app **PHẢI** tuân ngữ pháp: `miniappuri = uri-prefix uri-infix "/" identify path-abempty ["?" query] ["#" fragment]`, trong đó `uri-prefix` là custom-scheme kèm `://` hoặc đường dẫn host HTTPS, `uri-infix` cố định là `miniapp`, và `identify` gồm `id` bắt buộc kèm `;version=` tùy chọn.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-01-026** (MUST NOT · C2) — Viết tắt `miniapp://` **KHÔNG PHẢI** ngữ pháp tường minh; dạng chuẩn là `platform://miniapp/<id>/pages/<path>` hoặc `https://platform.org/miniapp/<id>/pages/<path>`. Bộ phân tích **KHÔNG ĐƯỢC** gộp custom-scheme và dạng HTTPS thành một bộ phân tích duy nhất không xác thực.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-01-027** (MUST · C2) — Thành phần `host` là **không gian định danh logic** và **KHÔNG** được coi là vị trí tải xuống. Nền tảng **PHẢI** duy trì ánh xạ tường minh provider/host → registry gói.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* review

> **REQ-01-028** (MUST · C2) — Trình tự phân giải: hệ điều hành điều phối theo tiền tố URI → user agent phân tích `id`/`version`/`path`/`query`/`fragment` → **ưu tiên gói cục bộ** trước máy chủ gói từ xa → phân giải tài nguyên theo quy tắc manifest → **bắt buộc** xử lý lỗi phân giải (dereferencing failure).
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-01-029** (MUST · C2) — Khi `version` bị bỏ qua, user agent/nhà cung cấp **PHẢI** chọn phiên bản hiện tại/mặc định/được ánh xạ một cách tường minh và ghi nhận vào nhật ký.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-01-030** (MUST · C2) — Dạng HTTPS **PHẢI** yêu cầu HTTPS, trả lỗi cho các trường hợp: xác thực thất bại, phương thức không hỗ trợ, không tìm thấy; và **NÊN** kèm định danh toàn vẹn (integrity identifier) cùng định dạng gói.
> *Nguồn:* `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-01-031** (MUST · C2) — Bảo mật điều hướng **PHẢI** gồm kiểm tra chống phát lại (anti-replay) và kiểm tra toàn vẹn gói trước khi khởi chạy.
> *Nguồn:* `→ report/18-...md`, `→ report/54-...md` · *Kiểm chứng:* runtime

> **REQ-01-032** (MUST · C2) — Một bộ phân giải deep-link chuẩn **duy nhất** **PHẢI** được dùng chung cho mã QR, kết quả tìm kiếm, thẻ danh mục, thông báo đẩy và điều hướng liên mini-app.
> *Nguồn:* `→ report/29-...md`, `→ report/18-...md` · *Kiểm chứng:* review

> **REQ-01-033** (MUST · C2) — Lỗi trả về cho đường dẫn thiếu/bị từ chối/phiên bản không tương thích **PHẢI** là lỗi có thể hành động (actionable) và tuân hợp đồng lỗi RFC 7807.
> *Nguồn:* `→ report/33-...md`, `→ report/41-...md` · *Kiểm chứng:* review

### 3.5 Bất biến và phiên bản

> **REQ-01-034** (MUST · C2) — Mỗi release **PHẢI** mang digest bất biến; các phiên bản đã phát hành **KHÔNG ĐƯỢC** bị ghi đè — chỉ được phát hành bản mới.
> *Nguồn:* `→ report/02-...md`, `→ report/18-...md` · *Kiểm chứng:* static

> **REQ-01-035** (MUST · C2) — Bản phát hành **PHẢI** khai báo dải SDK host tương thích (`min_host_version` / dải đã kiểm thử) và danh sách host API bắt buộc.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* static

> **REQ-01-036** (MUST · C2) — Kết quả "cài được" (installable) và "chạy được" (launchable) **PHẢI** là hai kết quả riêng; bản phát hành **CÓ THỂ** hiển thị nhưng không khả dụng trên host không tương thích kèm lý do giải thích.
> *Nguồn:* `→ report/02-...md` · *Kiểm chứng:* runtime

> **REQ-01-037** (SHOULD · C2) — Danh mục **NÊN** xuất chiếu siêu dữ liệu di động theo Schema.org / W3C để hỗ trợ tìm kiếm và liên thông.
> *Nguồn:* `→ report/10-...md` · *Kiểm chứng:* static

---

## 4. Ghi chú triển khai (informative)

**Cấu trúc gói tham chiếu:**

```text
my-miniapp/
├── manifest.json          # siêu dữ liệu (bắt buộc)
├── app.js                 # logic toàn cục (bắt buộc)
├── app.css                # kiểu toàn cục (bắt buộc)
├── pages/
│   ├── index/             # pages[0] = trang chủ
│   │   ├── index.html
│   │   ├── index.js
│   │   └── index.css
│   └── detail/
│       ├── detail.html
│       ├── detail.js
│       └── detail.css
├── common/                # (tùy chọn) dùng chung
└── i18n/                  # (tùy chọn) vi-VN.json, en-US.json
```

- `pages[0]` quyết định trang khởi chạy — không cần cấu hình riêng.
- Chuỗi `;version=` trong URI dùng để ghim một phiên bản cụ thể khi cần tái lập sự cố.
- Việc tách sub-package nên theo **tuyến người dùng**, không theo loại tệp, để giảm tải lần đầu.

**Những gì W3C MiniApp *không* giải quyết (cần mở rộng nền tảng):**
chữ ký từng sub-package, hợp đồng tải lũy tiến, chính sách thu hồi cache, và
phân giải nhà cung cấp gói (provider registry). Xem `SPEC-05` và `report/06-gaps-validation-plan.md`.

---

## 5. Tiêu chí tuân thủ

- [ ] `manifest.json` hợp lệ, đủ 6 khóa bắt buộc, mọi `icons[].src` tồn tại.
- [ ] `pages` liệt kê đúng các tuyến; `pages[0]` tồn tại và là trang chủ.
- [ ] Gói ZIP dùng UTF-8 tên tệp, CRC32, không mã hóa/chia mảnh.
- [ ] Có `manifest.json`, `app.js`, `app.css` ở gốc; thư mục `pages/` tồn tại.
- [ ] Gói chính ≤ 4 MB, tổng ≤ 20 MB.
- [ ] Không dùng `theme_color` / `scope` / `shortcuts` như tính năng được hỗ trợ.
- [ ] URI deep-link dùng đúng ngữ pháp `miniappuri`; có cơ chế chống phát lại.
- [ ] Digest gói bất biến; có attestation/chữ ký theo `SPEC-05`.
- [ ] Mọi tài nguyên trong gói đều được khai báo (không có tệp thừa).

---

## 6. Cân nhắc an ninh & riêng tư

- **Chống giả mạo gói:** digest + chữ ký + chuỗi TUF; TLS/CDN chỉ là bằng chứng vận chuyển, **không** phải neo tin cậy của gói.
- **Chống vượt thư mục:** kiểm tra `path-abempty` chống `../../` khi phân giải tài nguyên nội bộ.
- **Chống đánh cắp điều hướng:** chống phát lại và ràng buộc origin cho mọi deep-link.
- **Riêng tư tham số:** tham số `query` nhạy cảm **KHÔNG ĐƯỢC** lưu ở dạng văn bản thuần trong nhật ký/phân tích; tham chiếu `SPEC-07`.
- **Tệp thừa:** loại bỏ tài nguyên không khai báo để giảm bề mặt tấn công và tránh rò rỉ tài nguyên nội bộ.

---

## 7. Tham chiếu

- W3C MiniApp Manifest — https://w3c.github.io/miniapp-manifest/
- W3C MiniApp Packaging — https://w3c.github.io/miniapp-packaging/
- W3C MiniApp Addressing — https://w3c.github.io/miniapp-addressing/
- W3C MiniApp Standardization White Paper v2 — https://w3c.github.io/miniapp-white-paper/
- TUF Specification 1.x — https://theupdateframework.io/specification/latest/
- IETF RFC 7807 Problem Details — `SPEC-13`
- Báo cáo nguồn: `report/18`, `report/46`, `report/60`, `report/10`, `report/02`
