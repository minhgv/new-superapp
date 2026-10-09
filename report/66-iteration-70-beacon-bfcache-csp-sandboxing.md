# Chuyên đề Iteration 70: W3C Beacon API, Back/Forward Cache (bfcache) & CSP Level 3 Container Sandboxing

## 1. Bối cảnh & Mục tiêu Kỹ thuật

Trong kiến trúc Super App hiện đại, việc vận hành hàng trăm mini app của bên thứ ba trong các WebView nhúng (Android WebView và iOS WKWebView) đặt ra ba thách thức cốt lõi liên quan đến khả năng quan sát (observability), hiệu năng chuyển hướng (navigation performance) và vành đai bảo mật khai báo (declarative security perimeter):

1. **Thu thập dữ liệu Telemetry ngoại băng (Out-of-Band Telemetry Delivery)**: Khi người dùng đóng mini app hoặc chuyển hướng đột ngột, việc cố gắng gửi dữ liệu phân tích (analytics), drop-off funnel hay crash breadcrumbs bằng các lệnh gọi mạng đồng bộ (`XMLHttpRequest` đồng bộ) hoặc các vòng lặp bận (busy-wait loops) trong sự kiện hủy trang (`unload`) gây giật lag nghiêm trọng giao diện Super App, làm treo luồng chính (main thread jank) và kích hoạt bộ đếm watchdog ANR (Application Not Responding) của hệ điều hành. Ngược lại, nếu chỉ dùng `fetch()` bất đồng bộ thông thường, tiến trình render khi bị đóng sẽ lập tức hủy bỏ các socket đang mở, dẫn đến mất mát hoàn toàn dữ liệu telemetry quan trọng. **W3C Beacon API** và cờ `keepalive` của **WHATWG Fetch** cung cấp cơ chế chuyển giao tác vụ mạng ra khỏi luồng xử lý của tài liệu sang tiến trình mạng nền của trình duyệt.
2. **Khôi phục điều hướng tức thì với Back/Forward Cache (bfcache)**: Trong các luồng nghiệp vụ phức tạp của mini app (như duyệt danh mục -> chi tiết sản phẩm -> thanh toán -> trạng thái đơn hàng), người dùng liên tục thao tác nút Back của container. Nếu mỗi lần quay lại trang trước đều phải tải lại từ mạng hoặc phân tích lại DOM/JavaScript từ đầu, thời gian hiển thị (TTFB/FCP) sẽ kéo dài từ hàng trăm mili-giây đến vài giây, tiêu tốn băng thông và pin thiết bị. Chuẩn **WHATWG HTML Session History Traversal** định nghĩa **Back/Forward Cache (bfcache)** đóng băng toàn bộ cây DOM và heap thực thi JavaScript trong bộ nhớ RAM, cho phép khôi phục trang ngay lập tức (0ms load time). Tuy nhiên, mini app cần cơ chế đồng bộ lại trạng thái dữ liệu tài chính (balance, voucher) thông qua sự kiện `pageshow` và cờ `event.persisted`, đồng thời loại bỏ các anti-pattern làm vô hiệu hóa bfcache.
3. **Vành đai bảo mật khai báo CSP Level 3 (Content Security Policy Sandboxing)**: Mini app thường tích hợp nhiều thư viện JavaScript nguồn mở hoặc bên thứ ba. Nếu mini app bị tấn công Cross-Site Scripting (DOM XSS), kẻ tấn công có thể lợi dụng quyền hạn của WebView để gọi các API native bridge đặc quyền (camera, định vị, thanh toán). **W3C Content Security Policy Level 3** thiết lập rào cản phòng thủ theo chiều sâu (defense-in-depth), sử dụng cấu trúc phân tầng chỉ thị (`default-src`, `connect-src`, `base-uri`, `frame-ancestors`, `upgrade-insecure-requests`) để hạn chế điểm cuối mạng được phép kết nối, ngăn chặn đánh cắp đường dẫn tương đối qua `<base>`, triệt tiêu nguy cơ Clickjacking/UI Redressing và tự động nâng cấp lưu lượng HTTP sang HTTPS.

---

## 2. Bảng Tổng hợp 15 Chuẩn & Kiến trúc Triển khai (Iteration 70)

| ID | Chủ đề Chuẩn | Đặc tả / Tiêu chuẩn | Hạng mục Super App | Trạng thái Nguồn |
|---|---|---|---|---|
| `beacon_api_070_01` | W3C Beacon API & Asynchronous Out-of-Band Delivery | W3C Beacon API CR §4 | Observability & SRE | HTTP 200 (Verified) |
| `beacon_api_070_02` | navigator.sendBeacon Interface Semantics & Quotas | MDN / W3C Beacon API §4.1 | Observability & SRE | HTTP 200 (Verified) |
| `beacon_api_070_03` | Beacon API & Modern Page Lifecycle Integration | MDN / W3C Beacon API §2 | Observability & SRE | HTTP 200 (Verified) |
| `beacon_api_070_04` | WHATWG Fetch keepalive Flag & Transport Convergence | WHATWG Fetch Standard §5.4 | Observability & SRE | HTTP 200 (Verified) |
| `beacon_api_070_05` | Beacon API CORS Enforcement & Egress Boundaries | W3C Beacon API §5 | Security & Boundaries | HTTP 200 (Verified) |
| `bfcache_070_01` | Back/Forward Cache (bfcache) Architecture | WHATWG HTML Standard §7.4 | Architecture & Offline | HTTP 200 (Verified) |
| `bfcache_070_02` | PageTransitionEvent & event.persisted Reconciliation | MDN / WHATWG HTML §7.4.5 | Architecture & Offline | HTTP 200 (Verified) |
| `bfcache_070_03` | Window pagehide Event & Clean Resource Suspension | MDN / WHATWG HTML §7.4.4 | Architecture & Offline | HTTP 200 (Verified) |
| `bfcache_070_04` | PerformanceNavigationTiming & Navigation Telemetry | W3C Navigation Timing L2 §4.2 | Observability & SRE | HTTP 200 (Verified) |
| `bfcache_070_05` | bfcache Blocking Factors & Store Audit Governance | WHATWG HTML / Engine Spec | Governance & Policies | HTTP 200 (Verified) |
| `csp_governance_070_01` | W3C CSP Level 3 & Container Directive Architecture | W3C CSP Level 3 §3 | Security & Boundaries | HTTP 200 (Verified) |
| `csp_governance_070_02` | CSP default-src Fallbacks & connect-src Egress | MDN / W3C CSP L3 §4.2, §4.4 | Security & Boundaries | HTTP 200 (Verified) |
| `csp_governance_070_03` | CSP base-uri Directive & DOM Hijacking Mitigation | MDN / W3C CSP L3 §5.1 | Security & Boundaries | HTTP 200 (Verified) |
| `csp_governance_070_04` | CSP frame-ancestors & Clickjacking Defense | MDN / W3C CSP L3 §5.3 | Security & Boundaries | HTTP 200 (Verified) |
| `csp_governance_070_05` | CSP upgrade-insecure-requests & Mixed-Content Defense | MDN / W3C Upgrade Insecure Req | Security & Boundaries | HTTP 200 (Verified) |

---

## 3. Phân tích Chuyên sâu Từng Trụ cột Công nghệ

### 3.1. W3C Beacon API & Thu thập Telemetry Ngoại băng

#### 3.1.1. Cơ chế Hoạt động & Giới hạn Bộ nhớ đệm (Quota Clamping)
Phương thức `navigator.sendBeacon(url, data)` được thiết kế nhằm giải quyết bài toán gửi dữ liệu telemetry khi trang web sắp bị hủy. Khi được gọi:
- Trình duyệt sao chép dữ liệu (`ArrayBuffer`, `Blob`, `DOMString`, hoặc `FormData`) vào bộ đệm mạng nội bộ của tiến trình trình duyệt (browser process).
- Phương thức trả về giá trị kiểu boolean ngay lập tức (`true` nếu tác vụ được xếp hàng thành công, `false` nếu kích thước payload vượt quá giới hạn đệm còn lại của trình duyệt, thông thường là 64KB cho tất cả các yêu cầu đang chờ xử lý trên một tiến trình client).
- Quá trình gửi HTTP POST diễn ra hoàn toàn bất đồng bộ và độc lập với vòng đời của tài liệu. Ngay cả khi WebView hoặc tab bị hủy ngay lập tức sau đó, tiến trình mạng của hệ thống vẫn hoàn tất việc truyền gói tin tới máy chủ.
- `sendBeacon` không trả về Promise và mã JavaScript không thể đọc phản hồi HTTP từ máy chủ, giúp ngăn chặn việc treo tài nguyên giải phóng bộ nhớ.

#### 3.1.2. Tích hợp Vòng đời Trang Hiện đại & Hội tụ với Fetch keepalive
- Thay vì sử dụng sự kiện `unload` vốn không còn được hỗ trợ ổn định trên các nền tảng di động hiện đại, khuyến nghị kỹ thuật của W3C và MDN yêu cầu kích hoạt `sendBeacon` bên trong trình lắng nghe sự kiện `visibilitychange` khi `document.visibilityState === 'hidden'`.
- Đối với các mini app tài chính hoặc thanh toán cần gửi dữ liệu telemetry kèm theo chữ ký xác thực hoặc mã thông báo Bearer token, cờ `keepalive: true` trong **WHATWG Fetch Standard** (`fetch(url, { method: 'POST', body, keepalive: true, headers: { 'Authorization': 'Bearer ...' } })`) cung cấp khả năng tồn tại tương đương sendBeacon nhưng hỗ trợ đầy đủ các tiêu đề HTTP tùy chỉnh và phương thức kiểm soát lỗi.

#### 3.1.3. Chính sách CORS & Kiểm soát Egress trên Super App
- Chuẩn Beacon bắt buộc tuân thủ CORS: nếu payload sử dụng Content-Type không đơn giản (ví dụ `application/json`), trình duyệt phải gửi yêu cầu preflight OPTIONS. Do đó, để tránh việc gói tin bị hủy do preflight không kịp hoàn tất khi trang đóng, chuẩn Super App khuyến nghị đóng gói payload dưới dạng `text/plain` hoặc sử dụng cổng proxy native bridge của host container để chuyển tiếp gói tin an toàn.
- Host container áp dụng bộ lọc tên miền nghiêm ngặt: mọi URL truyền vào `sendBeacon` phải thuộc danh sách tên miền được cấp phép (`network_egress`) đã khai báo trong tệp `manifest.json` của mini app.

---

### 3.2. Back/Forward Cache (bfcache) & Tối ưu hóa Điều hướng Tức thì

#### 3.2.1. Đóng băng Trạng thái & Khôi phục 0ms TTFB
- Khác với cơ chế cache HTTP thông thường (chỉ lưu trữ tài nguyên tĩnh HTML/JS/CSS trên đĩa), **Back/Forward Cache (bfcache)** lưu giữ toàn bộ snapshot của tài liệu trong bộ nhớ RAM, bao gồm:
  - Cây DOM hoàn chỉnh và trạng thái giao diện hiện tại.
  - Heap thực thi JavaScript với toàn bộ đối tượng, biến và trạng thái đóng gói.
  - Các worker thread và vị trí cuộn trang (scroll position).
- Khi người dùng điều hướng quay lại, trình duyệt phục hồi trang ngay tức thì (0ms TTFB, 0ms FCP), loại bỏ hoàn toàn màn hình trắng và không cần phân tích lại cú pháp JavaScript. Trong thời gian lưu trú trong bfcache, toàn bộ các bộ định thời (`setTimeout`, `setInterval`) và luồng mạng đều được tạm dừng để tiết kiệm pin.

#### 3.2.2. Nhận biết Phục hồi qua PageTransitionEvent.persisted
- Khi tài liệu được hiển thị hoặc khôi phục, trình duyệt phát sự kiện `pageshow` trên đối tượng `Window`. Thuộc tính `event.persisted` trả về `true` báo hiệu rằng trang được khôi phục từ bfcache chứ không phải tải mới.
- Do các hàm khởi tạo ban đầu (`DOMContentLoaded`, `load`) không chạy lại, mini app phải cài đặt bộ lắng nghe `pageshow`:
  ```javascript
  window.addEventListener('pageshow', (event) => {
    if (event.persisted) {
      // Tái xác thực phiên người dùng và làm mới dữ liệu động nhẹ nhàng
      revalidateUserSession();
      refreshDynamicBalances();
    }
  });
  ```
- Quy định này đặc biệt quan trọng trong các mini app tài chính để tránh việc hiển thị số dư tài khoản hoặc mã khuyến mãi đã hết hạn.

#### 3.2.3. Quy chế Kiểm duyệt Store Loại bỏ Anti-Patterns Làm Hỏng bfcache
Hệ thống kiểm duyệt tự động của Super App Store quét mã nguồn mini app để loại bỏ các nguyên nhân phổ biến ngăn cản trang vào bfcache:
1. **Cấm tuyệt đối lắng nghe sự kiện `unload`**: Các engine render (Chromium, WebKit) sẽ tự động vô hiệu hóa bfcache nếu phát hiện có listener `unload`. Mini app phải chuyển hoàn toàn sang sự kiện `pagehide`.
2. **Quản lý kết nối mở**: Mini app phải chủ động đóng hoặc tạm dừng các kết nối WebSocket, WebRTC peer connections, hoặc giải phóng các khóa độc quyền `W3C Web Locks` trong sự kiện `pagehide`.
3. **Tiêu đề Cache-Control**: Khuyến nghị không sử dụng `Cache-Control: no-store` trên trang chính trừ khi bắt buộc tuyệt đối về mặt bảo mật, vì một số trình duyệt coi đây là chỉ thị không lưu trạng thái vào bộ nhớ tạm.

---

### 3.3. W3C Content Security Policy Level 3 & Vành đai Bảo mật Container

#### 3.3.1. Cấu trúc Phân tầng Chỉ thị & Phòng thủ Chiều sâu
W3C CSP Level 3 cung cấp cơ chế khai báo chính sách bảo mật cho WebView của mini app:
- Chỉ thị cơ sở `default-src 'none'` đóng vai trò là chính sách mặc định an toàn nhất, buộc nhà phát triển phải khai báo rõ ràng từng loại tài nguyên được phép tải.
- Chỉ thị `connect-src` giới hạn các điểm cuối mà mini app có thể kết nối thông qua các giao diện mạng (`fetch`, `XHR`, `WebSocket`, `EventSource`, `sendBeacon`). Super App tự động đồng bộ danh sách này từ manifest permissions của mini app, triệt tiêu nguy cơ rò rỉ dữ liệu qua các script gián điệp.

#### 3.3.2. Chống Tấn công Thay đổi Đường dẫn Tương đối (base-uri)
- Kẻ tấn công có thể chèn thẻ `<base href="https://attacker.com/">` thông qua các lỗ hổng chèn mã HTML. Khi đó, tất cả các tài nguyên tải qua đường dẫn tương đối (như `./app.js`) sẽ bị trình duyệt phân giải sang máy chủ của kẻ tấn công.
- Chỉ thị `base-uri 'none'` hoặc `base-uri 'self'` ngăn chặn hoàn toàn việc chèn hoặc sửa đổi thẻ `<base>`, bảo vệ toàn vẹn không gian nạp module nội bộ của mini app package.

#### 3.3.3. Phòng chống Clickjacking với frame-ancestors & Nâng cấp HTTPS
- Chỉ thị `frame-ancestors 'none'` thay thế hoàn toàn tiêu đề lỗi thời `X-Frame-Options: DENY`, ngăn chặn các trang web bên ngoài nhúng mini app vào trong `<iframe>`. Điều này bảo vệ các màn hình xác thực giao dịch, chuyển tiền hoặc nhập mật mã khỏi các cuộc tấn công UI Redressing và Clickjacking.
- Chỉ thị `upgrade-insecure-requests` tự động chuyển đổi tất cả các yêu cầu tài nguyên con từ HTTP sang HTTPS trước khi gửi gói tin qua mạng, loại bỏ cảnh báo Mixed Content và bảo vệ người dùng trên các mạng Wi-Fi công cộng không an toàn.

---

## 4. Tác động Kiến trúc Đối với Super App Store Standard

1. **Khung Đo lường RUM QoS**: Đưa chỉ số tỷ lệ khôi phục thành công qua bfcache (`bfcache hit ratio`) và số lượng gói tin telemetry bị hủy (`beacon buffer drops`) vào bảng điều khiển chất lượng dịch vụ vận hành thời gian thực.
2. **Cơ chế Sandbox Tự động (Container Proxy CSP)**: Host container chịu trách nhiệm tự động tạo và tiêm tiêu đề `Content-Security-Policy` nghiêm ngặt vào các phản hồi HTML cục bộ của mini app, đảm bảo nhà phát triển không thể vô hiệu hóa vành đai bảo mật.
3. **Tiêu chí Kiểm duyệt Tự động (Store Review Gating)**: Hệ thống CI/CD kiểm duyệt gói bundle mini app sẽ tự động quét mã nguồn AST để phát hiện và từ chối các gói có chứa lệnh gọi `window.addEventListener('unload', ...)` hoặc các lệnh gọi mạng đồng bộ trong các handler đóng trang.
