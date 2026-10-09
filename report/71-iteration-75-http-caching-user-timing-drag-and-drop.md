# Chuyên đề 71: HTTP Caching (RFC 9111), W3C User Timing Level 3 & WHATWG HTML Drag and Drop API

## 1. Tổng quan nghiên cứu Milestone 75

Milestone 75 tập trung chuẩn hóa 3 trụ cột kỹ thuật nền tảng quyết định hiệu năng phân phối gói ứng dụng, khả năng quan sát profiling thời gian thực và trải nghiệm kéo thả tương tác đa cửa sổ trong kiến trúc siêu ứng dụng (Super-App Mini-App Store Standard):

1. **HTTP Caching (RFC 9111) & Deterministic Package Freshness**:
   - Chuẩn hóa các chỉ thị `Cache-Control` (`public`, `immutable`, `no-cache`, `no-store`) thiết lập hợp đồng lưu đệm phân cấp giữa host client WebView và mạng lưới CDN biên.
   - Ứng dụng định danh thực thể mật mã `ETag` (Entity Tag) và bắt tay điều kiện `If-None-Match` trả về mã `HTTP 304 Not Modified`, triệt tiêu lãng phí băng thông di động và chi phí giải nén CPU.
   - Đo lường độ trễ và độ tươi của bản đệm proxy thông qua trường tiêu đề phản hồi `Age`, phục vụ hệ thống giám sát phân tán phát hiện các điểm hiện diện CDN (PoP) bị trễ đồng bộ khi kích hoạt quy trình rollback bảo mật khẩn cấp.
   - Phân cấp lưu đệm riêng tư (`private`) và chia sẻ (`shared`), bảo vệ thông tin phiên nhạy cảm của người dùng không bao giờ rò rỉ vào bộ đệm công cộng.

2. **W3C User Timing Level 3 & Runtime Performance Profiling**:
   - Giao diện `performance.mark()` cho phép lập trình viên mini-app và container host đánh dấu các mốc thời gian độ phân giải cao (`DOMHighResTimeStamp`) với trường `detail` tùy biến mà không gây nghẽn luồng xử lý chính.
   - Phương thức `performance.measure()` tính toán chính xác khoảng thời gian giữa hai mốc đánh dấu hoặc mốc điều hướng, tích hợp chặt chẽ với các chỉ số Core Web Vitals và SLA khởi động ứng dụng (<800ms).
   - Nội soi và chuẩn hóa các giao diện `PerformanceMark` và `PerformanceMeasure`, hỗ trợ bộ lọc tự động khử nhiễm dữ liệu cá nhân (PII) trước khi đẩy telemetry về trung tâm báo cáo lỗi từ xa.
   - Tích hợp chuẩn User Timing API vào quy trình kiểm thử tự động không đầu (headless synthetic testing) trong cổng kiểm duyệt store, đo đạc hiệu năng CPU và độ mượt khung hình trên thiết bị di động cấu hình thấp.

3. **WHATWG HTML Drag and Drop API & Multi-Window Data Marshaling**:
   - Kiến trúc xử lý sự kiện kéo thả DOM (`HTML Drag and Drop API`) tiêu chuẩn hóa luồng trao đổi dữ liệu trực quan giữa các mini-app trên môi trường đa cửa sổ, chia đôi màn hình (tablet/desktop split-view).
   - Đối tượng điều phối `DataTransfer` đóng gói các cấu trúc dữ liệu theo định dạng MIME (`text/plain`, `application/json`, binary files) và cơ chế phân quyền trực quan `dropEffect`/`effectAllowed`.
   - Vòng đời sự kiện `DragEvent` kiểm soát chặt chẽ cơ chế cấp quyền thả thông qua yêu cầu gọi bắt buộc `event.preventDefault()` trong bộ lắng nghe sự kiện `dragover`.
   - Giao diện `DataTransferItemList` cung cấp khả năng phát hiện kiểu MIME và truyền phát tệp nhị phân theo luồng (streaming) mà không cần tải trước toàn bộ dữ liệu vào bộ nhớ RAM.
   - Mô hình bảo mật 3 trạng thái của WHATWG (`Read/Write`, `Read-Only`, `Protected`), ngăn chặn triệt để các phần tử mục tiêu trỏ chuột nghe lén dữ liệu nhạy cảm trong suốt hành trình kéo thả cho đến khi người dùng chủ động thả.

---

## 2. Bảng tổng hợp các tiêu chuẩn & Findings nghiên cứu (15 Quy tắc chuẩn hóa)

| Mã ID | Trụ cột công nghệ | Tiêu chuẩn tham chiếu | Phân loại | Tóm tắt quy định kiến trúc Super-App | URL nguồn xác thực |
|---|---|---|---|---|---|
| **FINDING-1062** | HTTP Caching | RFC 9111 / MDN Cache-Control | Normative | Các gói tĩnh mini-app có hash nội dung bắt buộc khai báo `Cache-Control: public, max-age=31536000, immutable`. Các tệp manifest động và quyền hạn bắt buộc dùng `no-cache` để đảm bảo store có thể thu hồi hoặc chuyển phiên bản tức thì. | [MDN Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control) |
| **FINDING-1063** | HTTP Caching | RFC 9111 / MDN ETag | Normative | Cổng phân phối mini-app bắt buộc tạo mã ETag mạnh (SHA-256) cho toàn bộ asset và gói zip bytecode nhằm loại bỏ tính không nhất quán giữa các node CDN biên. | [MDN ETag](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag) |
| **FINDING-1064** | HTTP Caching | RFC 9111 / MDN If-None-Match | Normative | Client container kích hoạt bắt tay điều kiện `If-None-Match` trong lúc rảnh rỗi để nhận diện gói 304 Not Modified, tiết kiệm tối đa dữ liệu mạng di động 4G/5G. | [MDN If-None-Match](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-None-Match) |
| **FINDING-1065** | HTTP Caching | RFC 9111 / MDN Age | Normative | Hệ thống chẩn đoán container bóc tách tiêu đề `Age` để phát hiện các node CDN bị trễ cache hoặc phân phối asset cũ khi đang tiến hành chiến dịch khôi phục sự cố bảo mật. | [MDN Age](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Age) |
| **FINDING-1066** | HTTP Caching | RFC 9111 / MDN HTTP Caching | Normative | Phân tách nghiêm ngặt giữa bộ đệm riêng tư (`private` trên WebView/OPFS thiết bị) cho dữ liệu người dùng/giỏ hàng và bộ đệm chia sẻ (`shared` trên CDN) cho framework SDK và icon kho ứng dụng. | [MDN HTTP Caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching) |
| **FINDING-1067** | User Timing L3 | W3C User Timing / MDN Performance.mark | Normative | Đo đạc chính xác thời điểm khởi tạo hydration framework, khởi động cầu nối native bridge và hiển thị trang thanh toán với mốc timestamp độ phân giải cao và trường `detail`. | [MDN Performance.mark](https://developer.mozilla.org/en-US/docs/Web/API/Performance/mark) |
| **FINDING-1068** | User Timing L3 | W3C User Timing / MDN Performance.measure | Normative | Đo lường thời lượng giữa các mốc sự kiện với tùy chọn `measureOptions`, cung cấp dữ liệu định lượng cho cổng kiểm duyệt tự động và thuật toán xếp hạng store. | [MDN Performance.measure](https://developer.mozilla.org/en-US/docs/Web/API/Performance/measure) |
| **FINDING-1069** | User Timing L3 | W3C User Timing / MDN PerformanceMark | Normative | Khử nhiễm cấu trúc `PerformanceMark.detail` loại bỏ hoàn toàn các thông tin PII trước khi gửi về hệ thống giám sát từ xa của siêu ứng dụng. | [MDN PerformanceMark](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceMark) |
| **FINDING-1070** | User Timing L3 | W3C User Timing / MDN PerformanceMeasure | Normative | Theo dõi sự suy giảm hiệu năng qua từng phiên bản mini-app (regression testing) bằng cách thu thập bản ghi `PerformanceMeasure` trong môi trường sandbox kiểm duyệt. | [MDN PerformanceMeasure](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceMeasure) |
| **FINDING-1071** | User Timing L3 | W3C User Timing / MDN User Timing API | Normative | Hợp nhất profiling cấp ứng dụng vào timeline hiệu năng chung của trình duyệt, phục vụ bài kiểm thử hiệu năng tổng thể trên phần cứng di động bình dân. | [MDN User Timing API](https://developer.mozilla.org/en-US/docs/Web/API/User_Timing_API) |
| **FINDING-1072** | Drag & Drop | WHATWG HTML / MDN Drag & Drop API | Normative | Chuẩn hóa tương tác kéo thả dữ liệu giữa các mini-app trong chế độ chia đôi màn hình tablet/desktop, kèm cơ chế lọc sự kiện bảo mật ngăn rò rỉ ngữ cảnh chéo. | [MDN Drag and Drop API](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API) |
| **FINDING-1073** | Drag & Drop | WHATWG HTML / MDN DataTransfer | Normative | Lớp bảo vệ host container chặn bắt khởi tạo `DataTransfer` để ngăn chặn việc lộ lọt token xác thực phiên hoặc thông tin bí mật từ container cha sang guest mini-app. | [MDN DataTransfer](https://developer.mozilla.org/en-US/docs/Web/API/DataTransfer) |
| **FINDING-1074** | Drag & Drop | WHATWG HTML / MDN DragEvent | Normative | Bắt buộc phần tử nhận dữ liệu phải gọi `event.preventDefault()` trong sự kiện `dragover` để xác nhận khả năng tiếp nhận, ngăn chặn việc vô tình nuốt tệp hoặc tấn công UI redressing. | [MDN DragEvent](https://developer.mozilla.org/en-US/docs/Web/API/DragEvent) |
| **FINDING-1075** | Drag & Drop | WHATWG HTML / MDN DataTransferItemList | Normative | Kiểm tra metadata tệp (tên, dung lượng, MIME type) trước khi tải vào bộ nhớ RAM, bảo vệ thiết bị di động khỏi các cuộc tấn công gây cạn kiệt bộ nhớ OOM. | [MDN DataTransferItemList](https://developer.mozilla.org/en-US/docs/Web/API/DataTransferItemList) |
| **FINDING-1076** | Drag & Drop | WHATWG HTML Multipage Drag & Drop | Normative | Áp dụng mô hình bảo vệ 3 trạng thái của WHATWG (`Read/Write`, `Read-Only`, `Protected`) để bảo vệ dữ liệu kéo thả không bị các phần tử lướt qua đọc trộm. | [WHATWG HTML DND](https://html.spec.whatwg.org/multipage/dnd.html) |

---

## 3. Kiến trúc Triển khai & Đặc tả Kỹ thuật chi tiết

### 3.1. Hợp đồng Lưu đệm Phân cấp (Hierarchical Caching Contract)

```
+-----------------------------------------------------------------------------------+
|                            SUPER-APP CDN EDGE NETWORK                            |
|                                                                                   |
|   +---------------------------------------+   +-------------------------------+   |
|   | Content-Hashed Mini-App Bundles       |   | App Manifest & Permissions    |   |
|   | Cache-Control: public, max-age=31536000|   | Cache-Control: no-cache       |   |
|   | immutable, ETag: "sha256-abc123xyz"   |   | ETag: "sha256-meta-ver75"     |   |
|   +---------------------------------------+   +-------------------------------+   |
|                       |                                       |                   |
+-----------------------|---------------------------------------|-------------------+
                        |                                       |
           HTTP/3 Fetch | (304 Not Modified)       Conditional  | If-None-Match
                        v                                       v
+-----------------------------------------------------------------------------------+
|                        MOBILE CLIENT RUNTIME CONTAINER                            |
|                                                                                   |
|   +---------------------------------------+   +-------------------------------+   |
|   | Local Disk Cache / OPFS Storage       |   | Manifest Memory State         |   |
|   | Bundle v1.4.2 [Cached & Validated]    |   | Fresh manifest applied        |   |
|   +---------------------------------------+   +-------------------------------+   |
+-----------------------------------------------------------------------------------+
```

### 3.2. Vòng đời Giám sát Hiệu năng User Timing Level 3

Quy trình chuẩn hóa các mốc đánh dấu hiệu năng trong vòng đời nạp mini-app:
1. `performance.mark('miniapp-init-start', { detail: { appId, version } })`: Khi khung WebView bắt đầu nhận lệnh khởi chạy.
2. `performance.mark('framework-hydrated', { detail: { bundleSizeKB } })`: Khi engine hoàn tất khởi tạo DOM ảo và các thành phần cốt lõi.
3. `performance.mark('native-bridge-ready')`: Khi cầu nối giao tiếp Native-JS hoàn tất bắt tay bảo mật.
4. `performance.measure('cold-start-duration', { start: 'miniapp-init-start', end: 'native-bridge-ready' })`: Tính toán thời gian khởi động lạnh. Nếu thời gian này vượt ngưỡng 800ms trên thiết bị tham chiếu kiểm duyệt, mini-app sẽ bị đánh dấu cảnh báo hiệu năng trong cổng kiểm duyệt.

### 3.3. Mô hình An toàn Kéo thả Đa Ngữ cảnh (WHATWG Drag Data Store Modes)

Trong kiến trúc siêu ứng dụng hỗ trợ tablet và desktop, việc bảo vệ dữ liệu kéo thả giữa các ngữ cảnh được thực thi nghiêm ngặt:
- **Chế độ Read/Write**: Chỉ khả dụng duy nhất trong sự kiện `dragstart` tại phần tử nguồn. Tại đây, ứng dụng nguồn đóng gói dữ liệu vào `DataTransfer`.
- **Chế độ Protected**: Kích hoạt trong suốt các sự kiện `dragenter`, `dragover`, `dragleave`. Tại các sự kiện này, các phần tử DOM mục tiêu chỉ có thể đọc danh sách các kiểu dữ liệu (`types`) để quyết định xem có hỗ trợ định dạng này hay không, nhưng tuyệt đối không thể đọc nội dung dữ liệu bên trong. Điều này ngăn chặn việc ứng dụng độc hại lén lút thu thập dữ liệu nhạy cảm khi người dùng vô tình kéo ngang qua cửa sổ của nó.
- **Chế độ Read-Only**: Chỉ kích hoạt khi sự kiện `drop` diễn ra trên phần tử đích hợp lệ đã gọi `event.preventDefault()` trong sự kiện `dragover`. Lúc này, phần tử đích mới được cấp quyền giải mã dữ liệu an toàn.

---

## 4. Khuyến nghị Thực thi cho Siêu ứng dụng Viettel / Alibaba Cloud Superapp

1. **CDN Edge Caching Configuration**:
   - Thiết lập cấu hình NGINX/Envoy tại edge gateway để tự động chèn chỉ thị `Cache-Control: public, max-age=31536000, immutable` cho mọi tệp asset có chứa hash trong đường dẫn `/assets/[name].[hash].[ext]`.
   - Bắt buộc kiểm tra `ETag` và hỗ trợ trả về `304 Not Modified` đối với tất cả các API endpoint cung cấp tài nguyên mini-app.

2. **Automated User Timing Benchmark in CI/CD**:
   - Tích hợp một suite kiểm thử tự động sử dụng `PerformanceObserver` bắt các bản ghi `entryType: 'measure'` do mini-app phát ra.
   - Thiết lập ngưỡng kiểm duyệt chất lượng (Quality Gate): nếu `cold-start-duration` trung bình qua 10 lượt chạy thử vượt quá 1000ms trên máy ảo cấu hình tương đương điện thoại phổ thông (2GB RAM, 4 CPU cores), hệ thống tự động từ chối bản build và gửi thông báo tối ưu hóa cho nhà phát triển.

3. **Multi-Window Drag & Drop Security Policy**:
   - Cung cấp thành phần giao diện chuẩn (SDK UI Component) cho hành động kéo thả tệp và hóa đơn thanh toán.
   - Container WebView host phải tự động khử trùng đối tượng `DataTransfer` trước khi kích hoạt sự kiện `dragstart`, chặn hoàn toàn việc rò rỉ các cookie phiên đăng nhập hoặc khóa API của ứng dụng cha.
