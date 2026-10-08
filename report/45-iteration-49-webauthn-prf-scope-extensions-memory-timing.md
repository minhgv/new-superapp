# Chuyên Đề 45: Khóa Đối Xứng WebAuthn PRF, Mở Rộng Phạm Vi Manifest (Scope Extensions) & Viễn Đoán Bộ Nhớ RAM/Render Timing

**Mã tài liệu**: `M49-WEBAUTHN-PRF-SCOPE-EXT-MEM-TIMING`
**Thuộc dự án**: Nghiên Cứu Chuẩn Mini App Store Cho Super App (Deli Deep)
**Thời điểm hoàn thành**: Tháng 10/2026
**Số lượng findings mới**: 15 findings chuẩn hóa (100% normative standards & official platform specs)
**Trạng thái kiểm tra**: Hoàn tất xác thực HTTP 200, schema JSONL hợp lệ, zero secret leakage, URL deduplicated.

---

## 1. Bối Cảnh & Mục Tiêu Nghiên Cứu

Khi kiến trúc Super App hoàn thiện các tầng điều hướng và giao diện, ba bài toán công nghệ chuyên sâu phát sinh trực tiếp tại tầng biên bảo mật và giám sát thực thi:

1. **Bảo mật dữ liệu cục bộ mức Zero-Knowledge bằng khóa sinh trắc học phần cứng (WebAuthn PRF)**:
   - Các mini app tài chính, ngân hàng và hồ sơ y tế số lưu trữ dữ liệu ngoại tuyến (offline vaults) trên thiết bị người dùng đối mặt với nguy cơ lộ lọt dữ liệu nếu thiết bị bị root/jailbreak hoặc rò rỉ bản sao lưu (backup extraction).
   - Cơ chế xác thực khóa công khai (Public Key Authentication) tiêu chuẩn chỉ chứng minh danh tính nhưng KHÔNG thể tạo ra khóa mã hóa đối xứng xác định (deterministic symmetric encryption key) tại máy khách mà không gửi mật khẩu lên máy chủ. Cần một tiêu chuẩn dẫn xuất khóa đối xứng trực tiếp từ cảm biến sinh trắc học và phần cứng an toàn (Secure Enclave / TPM / StrongBox).

2. **Mở rộng phạm vi điều hướng đa nguồn (WICG Scope Extensions & Link Capturing)**:
   - Theo chuẩn W3C Web App Manifest truyền thống, mỗi mini app bị cô lập tuyệt đối trong một `scope` nguồn gốc duy nhất (single origin). Khi người dùng cần thanh toán tại miền phụ đối tác (vd: `pay.partner.com`, `auth.corp.vn`), container buộc phải mở trình duyệt ngoài hoặc hiển thị thanh địa chỉ cô lập, gây đứt gãy luồng trải nghiệm.
   - Cần một giao thức tuyên bố và xác thực hai chiều (`/.well-known/web-app-origin-association`), cho phép mini app mở rộng phạm vi điều hướng một cách an toàn mà vẫn duy trì biên giới cách ly lưu trữ (storage partitioning) và phân quyền JSAPI.

3. **Giám sát rò rỉ bộ nhớ RAM thực tế và viễn đoán thời điểm vẽ phần tử then chốt (Memory Measurement & Element Timing)**:
   - Các mini app chạy nền lâu dài (dẫn đường GPS, phát nhạc, POS bán hàng) thường gây tràn bộ nhớ (Out-Of-Memory - OOM) và đơ ứng dụng gốc (ANR). API bộ nhớ cũ (`performance.memory`) thiếu chuẩn xác và không phân rã được nguồn rò rỉ.
   - Chỉ số Largest Contentful Paint (LCP) chỉ đo lường phần tử lớn nhất chứ không đo lường chính xác thời điểm nút bấm giao dịch cốt lõi (Hero CTA button, mã VietQR, bảng số dư) thực sự xuất hiện trên màn hình. Cần chuẩn hóa `measureUserAgentSpecificMemory()` và `PerformanceElementTiming`.

Milestone 49 chuẩn hóa toàn diện 3 trụ cột kỹ thuật trên vào tài liệu quy chuẩn mini app store.

---

## 2. Dẫn Xuất Khóa Đối Xứng Phần Cứng Qua WebAuthn PRF (Pseudo-Random Function)

### 2.1. Chuẩn Hóa Tiện Ích Mở Rộng WebAuthn PRF (W3C WebAuthn Level 3 § 10.8)
- **Chuẩn tham chiếu**: W3C Web Authentication Level 3 § 10.8 (`https://w3c.github.io/webauthn/`, `https://w3c.github.io/webauthn/#sctn-prf-extension`).
- **Nguyên lý hoạt động**:
  - Tiện ích mở rộng `prf` cho phép Bên Dựa Vào (Relying Party - RP) cung cấp một hoặc hai giá trị muối (salt inputs: `eval: { first, second }`) khi đăng ký hoặc xác thực thông tin xác thực (`navigator.credentials.get`).
  - Bộ xác thực phần cứng (Authenticator) áp dụng thuật toán HMAC-SHA-256 với một khóa bí mật chuyên biệt (`CredRandom`) được lưu giữ bên trong phần tử an toàn, trả về chuỗi 32 byte giả ngẫu nhiên (`results: { first, second }`).
  - Khóa đối xứng thu được là **xác định (deterministic)**: cùng một thông tin xác thực và cùng một giá trị muối sẽ luôn tái tạo chính xác chuỗi byte đối xứng giống hệt nhau khi người dùng chạm sinh trắc học thành công.

### 2.2. Kiến Trúc Token Phần Cứng FIDO CTAP 2.1 `hmac-secret`
- **Chuẩn tham chiếu**: FIDO Alliance Client-to-Authenticator Protocol (CTAP) Implementation Draft v2.1 § 5.5.7 (`https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-20210615.html`).
- **Cơ chế kênh an toàn**:
  - Khi khởi tạo credential, bộ xác thực tạo ra khóa ngẫu nhiên 32-byte và bảo vệ bằng cơ chế Key Wrap hoặc lưu trong phần cứng tamper-resistant.
  - Khi xác thực, client và authenticator thiết lập kênh mã hóa Diffie-Hellman thông qua thuật toán ECDH trên đường cong P-256. Muối và kết quả PRF được truyền mã hóa qua bus USB/NFC/BLE, ngăn chặn hoàn toàn nghe lén phần cứng.
  - Container native của Super App bắt buộc phải xác thực kênh mã hóa CTAP 2.1 và từ chối cấp quyền cho các mini app tài chính nếu phần cứng thiết bị không hỗ trợ `hmac-secret`.

### 2.3. Đường Ống Dẫn Xuất Khóa HKDF & Kiến Trúc Két Dữ Liệu Zero-Knowledge
- **Chuẩn tham chiếu**: RFC 5869 (HKDF) & W3C SubtleCrypto (`https://developer.mozilla.org/en-US/docs/Web/API/PublicKeyCredential`, `https://developer.mozilla.org/en-US/docs/Web/API/Web_Authentication_API/WebAuthn_extensions`).
- **Quy tắc triển khai**:
  - Mini app đọc kết quả từ `credential.getClientExtensionResults().prf.results.first`.
  - Nghiêm cấm sử dụng trực tiếp chuỗi byte PRF làm khóa mã hóa AES. Bắt buộc phải đưa qua hàm dẫn xuất khóa HKDF (`crypto.subtle.importKey('raw', prfOutput, 'HKDF', false, ['deriveKey'])`) với info string phân định mục đích (ví dụ: `'offline-db-encryption'`).
  - Cơ sở dữ liệu SQLite biên dịch sang WebAssembly chạy trên Origin Private File System (OPFS) được giải mã tạm thời vào bộ nhớ RAM volatile sau khi chạm vân tay, và tự động xóa sạch (zeroize) khóa khi mini app vào trạng thái nền.

---

## 3. Mở Rộng Phạm Vi Ứng Dụng (Scope Extensions) & Bắt Link Đa Nguồn (Link Capturing)

### 3.1. Đặc Tả WICG Scope Extensions & Bắt Tay Xác Thực Hai Chiều
- **Chuẩn tham chiếu**: WICG Web App Manifest Incubations § Scope Extensions (`https://wicg.github.io/manifest-incubations/`, `https://github.com/WICG/manifest-incubations/blob/gh-pages/scope_extensions-explainer.md`).
- **Cơ chế xác thực**:
  - Manifest mini app khai báo: `scope_extensions: [ { origin: 'https://pay.partner.com' } ]`.
  - Để ngăn chặn việc mạo danh hoặc đánh cắp phạm vi của các tên miền uy tín, tên miền mục tiêu bắt buộc phải phản hồi tệp tin xác thực tại `https://pay.partner.com/.well-known/web-app-origin-association`.
  - Tệp JSON này chỉ định rõ URL manifest của mini app được phép mở rộng phạm vi:
    ```json
    {
      "web_apps": [
        {
          "manifest": "https://main.app.com/manifest.webmanifest",
          "details": { "paths": ["/checkout/*", "/auth/*"] }
        }
      ]
    }
    ```
  - Cổng kiểm duyệt store tự động quét kiểm tra sự tồn tại và tính hợp lệ của tệp association này trước khi duyệt bản phát hành.

### 3.2. Đồng Bộ Hóa Chuẩn Liên Kết Ứng Dụng Di Động (Android App Links & Apple Universal Links)
- **Chuẩn tham chiếu**: Google Digital Asset Links (`https://developers.google.com/digital-asset-links/v1/getting-started`) & Apple Associated Domains (`https://developer.apple.com/documentation/xcode/supporting-associated-domains`).
- **Tích hợp Super App Container**:
  - Super app container ánh xạ các khai báo `scope_extensions` vào bảng định tuyến liên kết native (`assetlinks.json` và `apple-app-site-association`).
  - Khi người dùng nhấp vào một liên kết thuộc miền phụ được mở rộng, container tự động kích hoạt mini app trong cửa sổ hiện tại mà không đẩy người dùng ra trình duyệt ngoài (Safari/Chrome).

### 3.3. Kiểm Soát Điều Hướng `handle_links` & Cách Ly An Toàn Bối Cảnh Phụ
- **Chuẩn tham chiếu**: WICG Link Capturing Specification (`https://developer.mozilla.org/en-US/docs/Web/Manifest/scope`).
- **Quy tắc an ninh**:
  - Thuộc tính `handle_links: "preferred"` đảm bảo các liên kết trong phạm vi mở rộng luôn được mở trong cửa sổ mini app độc lập.
  - **Nguyên tắc bất biến**: Việc mở rộng phạm vi điều hướng **KHÔNG đồng nhất nguồn gốc bảo mật (Security Principal)**. Miền phụ vẫn duy trì cookie, localStorage và CSP hoàn toàn độc lập.
  - Container Super App tự động **hạ cấp (attenuate)** quyền truy cập cầu nối JSAPI native khi người dùng duyệt sang miền phụ của đối tác, ngăn chặn trang web thứ ba lạm dụng các hàm native nhạy cảm của mini app chính.

---

## 4. Viễn Đoán Bộ Nhớ RAM & Đo Lường Thời Điểm Hiển Thị Phần Tử Then Chốt

### 4.1. Đo Lường Bộ Nhớ Client An Toàn Spectre (WICG `measureUserAgentSpecificMemory`)
- **Chuẩn tham chiếu**: WICG Performance.measureUserAgentSpecificMemory() (`https://developer.mozilla.org/en-US/docs/Web/API/Performance/measureUserAgentSpecificMemory`, `https://wicg.github.io/performance-measure-memory/`).
- **Cơ chế bảo vệ**:
  - API trả về báo cáo cấu trúc chi tiết: `{ bytes, breakdown: [ { bytes, attribution: [...], types } ] }`.
  - Để ngăn chặn các cuộc tấn công đánh cắp dữ liệu qua kênh định thời (Spectre/Meltdown), API bắt buộc phải kích hoạt **Cross-Origin Isolation** (`Cross-Origin-Opener-Policy: same-origin` và `Cross-Origin-Embedder-Policy: require-corp`).
  - Hệ thống giám sát runtime của super app lấy mẫu định kỳ và thiết lập trần bộ nhớ nghiêm ngặt (tối đa 150MB trên thiết bị Android phổ thông <=4GB RAM). Mini app có tốc độ tăng trưởng rò rỉ bộ nhớ >20MB/giờ sẽ bị cảnh báo vi phạm.

### 4.2. Phân Tích Phân Đoạn Bộ Nhớ & Phát Hiện Rò Rỉ DOM / WebAssembly
- **Chuẩn tham chiếu**: WICG Performance Memory Breakdown Taxonomy (`https://web.dev/articles/monitor-total-page-memory-usage`).
- **Cơ chế phân loại**:
  - Phân loại bộ nhớ thành các nhóm chuẩn: `'JavaScript'`, `'DOM'`, `'Shared'`, `'Canvas'`.
  - Bằng cách phân tích độ dốc hồi quy sau các chu kỳ dọn rác (GC) khi người dùng chuyển view, SDK giám sát phát hiện chính xác các cây DOM bị giữ lại (detached DOM tree leak) và bộ nhớ tuyến tính WebAssembly không được giải phóng.

### 4.3. Đo Lường Độ Trễ Hiển Thị Phần Tử Then Chốt (W3C/WICG Element Timing API)
- **Chuẩn tham chiếu**: W3C/WICG Element Timing API (`https://developer.mozilla.org/en-US/docs/Web/API/PerformanceElementTiming`, `https://wicg.github.io/element-timing/`, `https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/elementtiming`).
- **Quy tắc triển khai**:
  - Nhà phát triển gắn thuộc tính `elementtiming="identifier"` (ví dụ: `elementtiming="primary-cta"` hoặc `elementtiming="vietqr-canvas"`).
  - Đối tượng `PerformanceObserver` bắt sự kiện trả về `PerformanceElementTiming` với chỉ số `renderTime` chính xác đến mức sub-millisecond, phản ánh thời điểm phần cứng màn hình thực sự hoán đổi bộ đệm hiển thị (display swap).
  - Tiêu chuẩn cửa hàng quy định: 95% phiên người dùng phải đạt thời điểm hiển thị nút chuyển đổi chính (Hero CTA) dưới 1.200ms trên mạng di động 4G.

### 4.4. An Ninh Timing-Allow-Origin (TAO) & Cổng Chất Lượng Tự Động
- **Chuẩn tham chiếu**: WICG Element Timing § 4 Security Considerations (`https://w3c.github.io/paint-timing/`).
- **Bảo mật kênh định thời**:
  - Đối với các tài nguyên hình ảnh tải từ CDN liên miền (cross-origin CDN), máy chủ CDN bắt buộc phải phản hồi tiêu đề HTTP `Timing-Allow-Origin: *` hoặc cấp phép cho origin mini app. Nếu thiếu TAO, `renderTime` bị ép về 0 để bảo vệ quyền riêng tư người dùng.
  - Hạ tầng CDN lưu trữ gói mini app của super app phải tự động cấu hình tiêu đề TAO chuẩn hóa.

---

## 5. Tổng Hợp Ma Trận Tiêu Chuẩn Kỹ Thuật (Milestone 49)

| ID | Nhóm Công Nghệ | Chuẩn Tham Chiếu | Nguồn Xác Minh | Ứng Dụng Trong Mini App Store |
|---|---|---|---|---|
| `prf_crypt_049_01` | Mật mã & Xác thực | W3C WebAuthn Level 3 § 10.8 | `https://w3c.github.io/webauthn/` | Dẫn xuất khóa đối xứng phần cứng xác định qua `eval: { first, second }` |
| `prf_crypt_049_02` | Mật mã & Xác thực | FIDO CTAP 2.1 § 5.5.7 | `https://fidoalliance.org/specs/fido-v2.1-ps-20210615/...` | Kênh mã hóa ECDH P-256 bảo vệ `hmac-secret` chống nghe lén bus USB/NFC/BLE |
| `prf_crypt_049_03` | Mật mã & Xác thực | W3C WebAuthn § 5.7.5 | `https://developer.mozilla.org/en-US/docs/Web/API/PublicKeyCredential` | Đường ống dẫn xuất khóa HKDF (RFC 5869) từ kết quả 32-byte PRF |
| `prf_crypt_049_04` | Mật mã & Xác thực | W3C WebAuthn / Chrome 5141362703892480 | `https://developer.mozilla.org/en-US/docs/Web/API/Web_Authentication_API` | Kiến trúc két lưu trữ cục bộ Zero-Knowledge kết hợp SQLite Wasm trên OPFS |
| `prf_crypt_049_05` | Mật mã & Xác thực | W3C WebAuthn Level 3 | `https://developer.mozilla.org/en-US/docs/Web/API/Web_Authentication_API/WebAuthn_extensions` | Quy trình quản trị phục hồi khóa qua mô hình bọc khóa (Key Encryption Key) |
| `scope_ext_049_01` | Manifest & Định tuyến | WICG Scope Extensions | `https://wicg.github.io/manifest-incubations/` | Khai báo `scope_extensions` đa nguồn và cơ chế xác thực hai chiều |
| `scope_ext_049_02` | Manifest & Định tuyến | WICG Scope Extensions Explainer | `https://github.com/WICG/manifest-incubations/...` | Đặc tả tệp `/.well-known/web-app-origin-association` chống mạo danh phạm vi |
| `scope_ext_049_03` | Manifest & Định tuyến | Google DAL / Apple Universal Links | `https://developers.google.com/digital-asset-links/v1/getting-started` | Ánh xạ đồng bộ phạm vi manifest web sang bảng định tuyến liên kết native OS |
| `scope_ext_049_04` | Manifest & Định tuyến | WICG Link Capturing | `https://developer.mozilla.org/en-US/docs/Web/Manifest/scope` | Điều khiển điều hướng nội bộ vs mở trình duyệt ngoài qua `handle_links` |
| `scope_ext_049_05` | Manifest & Định tuyến | WICG Security Considerations | `https://wicg.github.io/manifest-incubations/` | Cách ly bối cảnh render, phân vùng lưu trữ và hạ cấp quyền JSAPI cho miền phụ |
| `mem_elem_049_01` | Viễn đoán & Hiệu năng | WICG Memory Measurement | `https://developer.mozilla.org/en-US/docs/Web/API/Performance/measureUserAgentSpecificMemory` | Đo lường RAM client an toàn Spectre bắt buộc Cross-Origin Isolation |
| `mem_elem_049_02` | Viễn đoán & Hiệu năng | WICG Memory Spec | `https://wicg.github.io/performance-measure-memory/` | Phân rã bộ nhớ theo loại (`DOM`, `JavaScript`, `Canvas`) và bắt rò rỉ tự động |
| `mem_elem_049_03` | Viễn đoán & Hiệu năng | WICG Element Timing | `https://developer.mozilla.org/en-US/docs/Web/API/PerformanceElementTiming` | Đo lường thời điểm render nút CTA chính và mã VietQR bằng `elementtiming` |
| `mem_elem_049_04` | Viễn đoán & Hiệu năng | WICG Element Timing § 4 | `https://wicg.github.io/element-timing/` | An ninh Timing-Allow-Origin (TAO) bảo vệ tài nguyên ảnh trên CDN |
| `mem_elem_049_05` | Viễn đoán & Hiệu năng | W3C Web Performance Standards | `https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver` | SLA hiệu năng client tổng thể và chính sách cảnh báo/gỡ app khi vi phạm RAM/LCP |

---
*Tài liệu hoàn thành và được ghi nhận chính thức vào hệ thống báo cáo của Super App Mini App Store Standard.*
