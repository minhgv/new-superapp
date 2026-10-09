# Chuyên đề 54 (Iteration 58): WICG Signature-Based SRI, Network Efficiency Guardrails & Autofill / Managed Configuration

## 1. Tổng quan & Phạm vi nghiên cứu Iteration 58

Trong lộ trình hoàn thiện tiêu chuẩn kỹ thuật toàn diện cho Mini App Store trên nền tảng Super App (tham chiếu khung năng lực POC Viettel / Alibaba Cloud WindVane Mini App), Iteration 58 tập trung nghiên cứu và chuẩn hóa 3 khối công nghệ nền tảng mới:

1. **WICG Signature-Based Subresource Integrity (Signature-Based SRI) & RFC 9421 HTTP Message Signatures**:
   - Khắc phục sự cứng nhắc của cơ chế SRI truyền thống (băm SHA-256/384/512 toàn văn tài nguyên làm vỡ ứng dụng khi CDN nén động, cập nhật bản vá phụ hoặc bản địa hóa).
   - Thiết lập chuẩn xác thực nguồn gốc (cryptographic provenance) bằng chữ ký số bất đối xứng Ed25519 tuân thủ hồ sơ RFC 9421 HTTP Message Signatures.
   - Cơ chế xác minh hai tầng phía máy khách (Two-Stage Verification): kiểm tra hàm băm phần thân giải mã `Integrity-Digest` và tái tạo cơ sở chữ ký `Signature-Input` trước khi giải phóng tài nguyên thực thi.
   - Tích hợp Content Security Policy (CSP Level 3) với cú pháp `script-src 'ed25519-<pubkey>'` và chủ động đàm phán yêu cầu chữ ký thông qua tiêu đề `Accept-Signature`.
   - Kiến trúc Super-App Store: chứng thực chữ ký số cho các thư viện động, micro-frontends và tài nguyên biên của đối tác mà không cần nộp lại gói ứng dụng.

2. **WICG Network Efficiency Guardrails & W3C Document Policy Integration**:
   - Thay thế các đoạn mã JavaScript chắp vá, kém hiệu quả (như can thiệp thô bạo vào `fetch`/`XMLHttpRequest` hay service worker proxy gây hao pin/CPU) bằng chính sách kiểm soát hiệu năng mạng cấp tài liệu (document-scoped policy) do lõi trình duyệt C++ trực tiếp giám sát.
   - Cấu hình qua tiêu đề HTTP `Document-Policy: network-efficiency-guardrails` hoặc thuộc tính `policy` trên iframe, thiết lập các ngưỡng thô (coarse, non-parametric thresholds) chống lại dữ liệu văn bản không nén (thiếu gzip/brotli/zstd), định dạng hình ảnh/phông chữ thô và tải trọng quá khổ.
   - Tích hợp W3C Reporting API truyền báo cáo vi phạm ngoại băng (out-of-band JSON) về máy chủ lưu trữ và cho phép mã JavaScript nội tại lắng nghe vi phạm theo thời gian thực qua `ReportingObserver`.
   - Ranh giới chẩn đoán an toàn quyền riêng tư: giám sát tài nguyên chéo nguồn (cross-origin CDN, media) mà không làm rò rỉ kích thước byte chính xác, triệt tiêu nguy cơ tấn công kênh kề định danh (XS-Leaks).
   - Đánh giá chất lượng mạng trên kho ứng dụng: chấm điểm hiệu năng khi duyệt gói, từ chối phát hành mã lãng phí băng thông và bảo vệ gói cước di động 3G/4G/5G của người dùng.

3. **WICG Autofill Event API & WICG Managed Configuration API**:
   - Kiến trúc sự kiện DOM `autofill` sớm: phát sự kiện trực tiếp tới `Document` trước khi DOM bị đột biến, cung cấp thuộc tính `values` ánh xạ phần tử tới dữ liệu điền sẵn giúp các framework phản ứng (React, Vue) đồng bộ trạng thái mượt mà, triệt tiêu xung đột dữ liệu.
   - Vòng đời điền lại có kiểm soát (Programmatic Refills): giao diện bất đồng bộ `AutofillEvent.refill()` cho phép ứng dụng mở rộng giao diện DOM động (như chọn quốc gia tự thêm tỉnh/thành) rồi kích hoạt trình duyệt điền tiếp dữ liệu, đi kèm cơ chế khóa chống vòng lặp vô tận (`refill === null`).
   - Khai báo phạm vi toàn diện `'full-address'`: nâng cao tính minh bạch, hiển thị modal cấp quyền rõ ràng và ngăn chặn hành vi khai thác điền lén vào các trường ẩn (`display: none`).
   - Tiêu chuẩn cấu hình doanh nghiệp WICG Managed Configuration API: giao diện `navigator.managed.getManagedConfiguration(keys)` bất đồng bộ giúp tiêm tham số, endpoint nội bộ và cờ tính năng cho các mini-app B2B/kiosk mà không cần nhúng cứng cấu hình nhạy cảm.
   - Ranh giới cách ly nguồn gốc (Origin Sandboxing): kiểm soát truy cập zero-trust, phân tách cấu hình độc lập giữa các ứng dụng và ngăn chặn rò rỉ định danh thiết bị chéo tenant.

---

## 2. Phân tích chi tiết & Chuẩn hóa kỹ thuật

### 2.1. WICG Signature-Based Subresource Integrity & RFC 9421
| Tiêu chí | SRI truyền thống (W3C SRI) | Signature-Based SRI (WICG & RFC 9421) | Yêu cầu Super-App Store |
| :--- | :--- | :--- | :--- |
| **Mô hình tin cậy** | Khớp bit tuyệt đối (Bit-exact digest) | Chứng thực nguồn gốc ký số (Cryptographic Provenance) | Khóa công khai Ed25519 gắn với tổ chức phát hành mini-app |
| **Khả năng chịu lỗi CDN** | Rất kém: nén động, đổi header làm hỏng hash gây lỗi nạp | Tuyệt vời: nội dung cập nhật hợp lệ chỉ cần có chữ ký số hợp lệ | Cho phép CDN biên tối ưu hóa tài nguyên mà không vỡ store runtime |
| **Hồ sơ chữ ký** | Không áp dụng | RFC 9421 profile: thuật toán `ed25519`, `keyid`, `@authority`, `@method`, `@path`, `@status` | Kiểm tra tính hợp lệ của timestamp `created`, `expires` và chuỗi `nonce` |
| **Xác minh phía Client** | Băm toàn bộ body khi tải xong | 2 bước: (1) Xác minh chữ ký header; (2) So khớp hash body với `Integrity-Digest` | Hủy ngay việc tiêm DOM và ghi log vi phạm bảo mật nếu chữ ký/digest sai |
| **Tích hợp CSP** | `script-src 'sha256-...'` | `script-src 'ed25519-<base64-key>'` + gửi `Accept-Signature` | Container tự động tiêm chính sách CSP hỗ trợ chữ ký cho mini-app |

### 2.2. WICG Network Efficiency Guardrails & Document Policy
- **Nguyên lý Document Policy**: Hoạt động ở tầng lõi trình duyệt, không đòi hỏi mini-app phải nhúng SDK giám sát mạng nặng nề. Cấu hình bằng tiêu đề `Document-Policy: network-efficiency-guardrails`.
- **Phân loại vi phạm thô (Coarse Thresholds)**:
  - *Uncompressed Text*: Bắt buộc nén HTTP (gzip/brotli/zstd) cho mọi payload văn bản (JSON, JS, CSS, HTML > 10KB).
  - *Uncompressed Media/Fonts*: Ngăn chặn tải hình ảnh định dạng BMP/raw hoặc phông chữ TTF không nén WOFF2.
  - *Oversized Transfers*: Cảnh báo các tài nguyên vượt quá ngưỡng dung lượng đơn lẻ bất thường.
- **Báo cáo W3C Reporting API**:
  - Gửi báo cáo JSON bất đồng bộ tới endpoint được chỉ định bởi tiêu đề `Reporting-Endpoints: net-violation="https://collector.superapp.internal/net"`.
  - Phía client có thể sử dụng `new ReportingObserver((reports) => { ... })` để thu thập dữ liệu chẩn đoán nội bộ mà không làm chậm giao diện người dùng.
- **Bảo vệ quyền riêng tư & kênh kề**:
  - Giữ nguyên URL đầy đủ để chẩn đoán nhưng tuyệt đối không trả về kích thước byte chính xác của tài nguyên chéo nguồn (cross-origin) nếu thiếu tiêu đề `Timing-Allow-Origin` (TAO), loại bỏ nguy cơ rò rỉ dữ liệu tìm kiếm (XS-Leaks).

### 2.3. WICG Autofill Event API & Managed Configuration API
- **Sự kiện DOM `autofill`**:
  - Được kích hoạt trước khi các trường nhập liệu bị thay đổi thực tế.
  - Cung cấp danh sách bất biến `event.values` (ánh xạ từng phần tử HTML tới chuỗi giá trị dự kiến điền), giúp các ứng dụng thương mại điện tử đồng bộ hóa state tức thì.
- **Cơ chế nạp lại `refill()`**:
  - Khi form thay đổi cấu trúc sau khi điền (ví dụ: chọn Quốc gia xong form mới render danh sách Tỉnh/Thành), ứng dụng gọi `await event.refill()`.
  - Trình duyệt quét lại DOM mới và tiếp tục điền dữ liệu tương ứng. Trình duyệt tự khóa (`refill === null`) sau một lần gọi để chặn vòng lặp vô tận gây treo ứng dụng.
- **Phạm vi `full-address` & Ngăn chặn thu hoạch dữ liệu lén**:
  - Đòi hỏi thuộc tính `autocomplete="full-address"` trên thẻ `<form>`.
  - Trình duyệt/Super-App hiển thị bảng xác nhận quyền (consent sheet) minh bạch toàn bộ các trường sẽ chia sẻ.
  - Nghiêm cấm và vô hiệu hóa việc tự động điền vào các phần tử ẩn (`display: none`, `visibility: hidden`), tự động gắn cờ vi phạm gian lận trải nghiệm (dark pattern).
- **Cấu hình doanh nghiệp `navigator.managed`**:
  - Gọi bất đồng bộ `navigator.managed.getManagedConfiguration(['apiEndpoint', 'tenantId', 'debugMode'])`.
  - Kết nối trực tiếp với chính sách MDM/EMM của thiết bị (Android Enterprise / Apple Managed App Configuration), phục vụ triển khai mini-app nội bộ doanh nghiệp mà không để lộ cấu hình nhạy cảm.

---

## 3. Ma trận kiểm soát & Tiêu chuẩn kỹ thuật áp dụng (Store Control Matrix)

| Mã kiểm soát | Hạng mục tiêu chuẩn | Cấp độ chứng cứ | Tác nhân thực thi | Hành vi bắt buộc |
| :--- | :--- | :--- | :--- | :--- |
| **SEC-SRI-01** | WICG Signature-Based SRI | `specification` | Developer Tools / Pipeline | Ký số tài nguyên tĩnh động bằng Ed25519 và đăng ký public key trong manifest. |
| **SEC-SRI-02** | RFC 9421 Compliance | `specification` | Gateway / CDN | Sinh các trường `@authority`, `@method`, `@path`, `@status`, `created`, `expires`, `nonce`. |
| **SEC-SRI-03** | Client Two-Stage Verification | `specification` | WebView Container Runtime | Kiểm tra chữ ký header trước, sau đó đối chiếu hash body với `Integrity-Digest`. |
| **NET-GRD-01** | Document-Policy Header | `specification` | Reverse Proxy / Web Server | Tiêm `Document-Policy: network-efficiency-guardrails` cho toàn bộ tài liệu mini-app. |
| **NET-GRD-02** | Compression Mandatory | `industry_standard` | Store Ingestion CI/CD | Chặn phát hành mini-app có trên 5% tài nguyên văn bản không được nén truyền tải. |
| **NET-GRD-03** | W3C Reporting API Collector | `specification` | Telemetry Infrastructure | Thu nạp báo cáo vi phạm hiệu năng mạng qua endpoint ngoại băng và phân tích xu hướng. |
| **UI-AUTO-01** | Autofill Event Integration | `specification` | Framework / Mini-App SDK | Lắng nghe sự kiện `autofill` để cập nhật state phản ứng, tránh mất đồng bộ dữ liệu nhập. |
| **UI-AUTO-02** | Safe Programmatic Refill | `specification` | Container Runtime | Giới hạn thời gian chờ `refill()` tối đa 2 giây và triệt tiêu các lệnh gọi lặp lại. |
| **UI-AUTO-03** | Hidden Field Harvester Block | `industry_standard` | Security Scanner | Từ chối tự động điền vào các thẻ input bị ẩn CSS nhằm bảo vệ dữ liệu cá nhân người dùng. |
| **ENT-CFG-01** | Managed Configuration API | `specification` | Enterprise Super-App Portal | Hỗ trợ phân phối JSON policy qua `navigator.managed` cho các mini-app B2B chuyên biệt. |

---

## 4. Kế hoạch xác minh & Khuyến nghị triển khai

1. **Khuyến nghị cho bộ phận Nền tảng (Platform Engineering)**:
   - Tích hợp mô-đun xác minh RFC 9421 vào thư viện mạng của Container SDK (Android / iOS / HarmonyOS), chuẩn bị sẵn sàng cho việc nạp tài nguyên từ các micro-frontend phân tán.
   - Bật Document Policy trên các môi trường thử nghiệm (canary/staging) nhằm đo lường tỷ lệ tiết kiệm băng thông và phát hiện tài nguyên chưa tối ưu nén trước khi đưa lên store.
2. **Khuyến nghị cho bộ phận Đánh giá Store (Review & Governance)**:
   - Xây dựng bài kiểm tra tự động bằng WebDriver BiDi quét các tài nguyên tải về trong quá trình chạy thử mini-app, cảnh báo các tệp ảnh lớn chưa nén hoặc phông chữ thô.
   - Thêm quy chuẩn kiểm duyệt biểu mẫu thanh toán/giao hàng: mini-app bắt buộc sử dụng chuẩn `autocomplete` thay vì các trường nhập liệu tùy biến khó đoán định.
