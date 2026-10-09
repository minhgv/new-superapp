# Chuyên đề 101: Web Locks Concurrency, Document Picture-in-Picture & OS Badging / Web Share Target Architecture (Milestone 105)

## 1. Tổng quan & Bối cảnh Tiêu chuẩn hóa

Trong kiến trúc Super App hiện đại, mini app không chỉ hoạt động như một trang web đơn lẻ mà vận hành như một thực thể ứng dụng đa ngữ cảnh (multi-context application). Mini app đồng thời sở hữu giao diện người dùng chính (foreground window/iframe), các luồng tính toán song song (DedicatedWorkers), các tiến trình đồng bộ dữ liệu chạy nền (ServiceWorkers/SharedWorkers), và thậm chí cả các cửa sổ phụ trợ luôn nổi trên cùng (Always-on-top Document Picture-in-Picture).

Để đảm bảo mini app vận hành ổn định, không gây xung đột dữ liệu cục bộ, không làm đơ giao diện người dùng và tích hợp liền mạch với hệ điều hành máy chủ, Milestone 105 thiết lập 3 chuẩn kỹ thuật cốt lõi:
1. **W3C Web Locks API & LockManager Concurrency Governance**: Cơ chế khóa bất đồng bộ cấp nguồn gốc (origin-scoped asynchronous locking) theo mô hình độc quyền (exclusive) hoặc chia sẻ (shared), tích hợp kiểm tra snapshot khóa, xử lý hủy qua AbortSignal và cơ chế phòng chống deadlock.
2. **WICG Document Picture-in-Picture API & Always-On-Top Window Sandboxing**: Kiến trúc cửa sổ nổi tự do chứa toàn bộ cây DOM tương tác tùy ý, cho phép duy trì trải nghiệm đa nhiệm (video call, ride-hailing tracking, stock tickers) độc lập với vòng đời khung nhìn chính.
3. **W3C Badging API & Web App Manifest 'share_target' Declarative Ingestion Architecture**: Tích hợp sâu vào hệ điều hành thông qua huy hiệu biểu tượng (icon badges) trên launcher/dock và tiếp nhận dữ liệu chia sẻ (text, URLs, files) từ các ứng dụng native thông qua khai báo manifest và Service Worker.

---

## 2. W3C Web Locks API & LockManager Concurrency Governance

### 2.1. Bản chất Kỹ thuật & Không gian Tên Cấp Nguồn Gốc (Origin Scope)
Web Locks API giải quyết triệt để vấn đề xung đột tài nguyên giữa nhiều tab hoặc giữa UI thread và background worker trong cùng một mini app:
- **Phạm vi bảo vệ**: Khóa được đặt tên bằng chuỗi tùy ý (DOMString) và được giới hạn nghiêm ngặt theo nguồn gốc (`origin`). Mini app của nhà phát triển A tuyệt đối không thể can thiệp hay đọc khóa của mini app B.
- **Vòng đời khóa (Lock Lifecycle)**: Khóa được cấp phát khi callback bất đồng bộ bắt đầu thực thi và tự động giải phóng khi Promise do callback trả về chuyển sang trạng thái settled (resolved hoặc rejected).
- **Thu hồi tự động (Automatic Garbage Collection)**: Nếu ngữ cảnh thực thi (tab, iframe hoặc worker) bị crash, bị đóng hoặc bị đóng băng bởi hệ điều hành, User Agent cam kết tự động giải phóng toàn bộ khóa mà ngữ cảnh đó đang nắm giữ, ngăn chặn vĩnh viễn nguy cơ tắc nghẽn tài nguyên (resource starvation).

### 2.2. Phân loại Khóa & Các Tùy chọn Điều phối Nâng cao
Phương thức `navigator.locks.request(name, [options], callback)` hỗ trợ các cấu hình then chốt:
1. **Chế độ Khóa (`mode`)**:
   - `'exclusive'` (mặc định): Chỉ một ngữ cảnh duy nhất được giữ khóa tại một thời điểm. Dùng cho các thao tác ghi dữ liệu (IndexedDB transactions, ghi file OPFS, ghi cấu hình token).
   - `'shared'`: Nhiều ngữ cảnh có thể đồng thời giữ khóa miễn là không có yêu cầu exclusive nào đang chờ trước đó. Cho phép hàng loạt luồng đọc đồng thời mà không bị khóa chặn.
2. **Thử khóa không chặn (`ifAvailable: true`)**:
   - Nếu khóa đang bận, callback sẽ được gọi ngay lập tức với giá trị `null` thay vì bị đẩy vào hàng đợi. Rất hữu ích cho các tác vụ dọn dẹp bộ nhớ đệm chạy nền (background cache cleanup) hoặc đồng bộ dữ liệu cơ hội.
3. **Cướp quyền khẩn cấp (`steal: true`)**:
   - Buộc hủy tất cả các khóa hiện tại mang tên đó và cấp phát ngay cho yêu cầu này. Chỉ dành cho các kịch bản phục hồi lỗi nghiêm trọng (failover recovery).
4. **Tích hợp Hủy bỏ (`signal: AbortSignal`)**:
   - Cho phép đặt thời gian chờ tối đa bằng `AbortSignal.timeout(ms)`. Nếu hết thời gian mà khóa chưa được cấp, Promise yêu cầu sẽ reject với `AbortError` mà không làm treo hàng đợi.

### 2.3. Giám sát Bế tắc & Phân tích Đột xuất qua `LockManager.query()`
Phương thức `navigator.locks.query()` trả về một `LockManagerSnapshot` gồm hai danh sách: `held` (các khóa đang được giữ) và `pending` (các yêu cầu đang chờ trong hàng đợi). Mỗi bản ghi cung cấp:
- `name`: Tên định danh của tài nguyên bị khóa.
- `mode`: Chế độ `'exclusive'` hoặc `'shared'`.
- `clientId`: Mã định danh Client duy nhất của cửa sổ hoặc worker đang sở hữu hoặc chờ khóa.

Super App SDK tích hợp watchdog tự động gọi `query()` mỗi 30 giây: nếu phát hiện một khóa exclusive bị giữ quá 15 giây bởi một worker chạy nền, SDK sẽ tự động kích hoạt hủy worker đó và tái tạo luồng mới, bảo vệ độ mượt mà của giao diện mini app.

---

## 3. WICG Document Picture-in-Picture API & Always-On-Top Window Sandboxing

### 3.1. Sự Khác biệt Kiến trúc so với Video Picture-in-Picture
Khác với chuẩn `HTMLVideoElement.requestPictureInPicture()` vốn chỉ chiếu một luồng thẻ `<video>`, Document Picture-in-Picture cung cấp một đối tượng `Window` hoàn chỉnh luôn nổi trên cùng (always-on-top):
- Hỗ trợ toàn bộ phần tử HTML tùy ý: form nhập liệu, bảng chat trực tiếp, nút bấm điều khiển, canvas 2D/WebGL, bảng giá chứng khoán, bản đồ hành trình tài xế.
- Chia sẻ chung môi trường thực thi, cookie, bộ nhớ cache và storage jar với cửa sổ gốc (opener window).
- Không yêu cầu iframe trung gian, cho phép truyền trực tiếp đối tượng DOM hoặc gọi hàm JavaScript chéo cửa sổ mà không gặp rào cản Cross-Origin.

### 3.2. Vòng đời Khởi tạo & Kích hoạt Người dùng
- **Bắt buộc Kích hoạt Tạm thời (Transient User Activation)**: Lệnh gọi `documentPictureInPicture.requestWindow(options)` chỉ được phép thực thi ngay sau một tương tác vật lý của người dùng (click, tap). Nghiêm cấm tự động bật cửa sổ nổi mà không có sự đồng ý.
- **Quy tắc Cửa sổ Đơn lẻ (Single Instance Invariant)**: Tại mỗi thời điểm, một User Agent chỉ cho phép tối đa một cửa sổ Document PiP duy nhất tồn tại. Việc mở một cửa sổ mới sẽ tự động đóng cửa sổ PiP hiện tại.
- **Kế thừa và Sao chép Kiểu dáng (Style Copying)**: Do cửa sổ PiP khởi tạo với một Document trắng rỗng, mini app phải chủ động sao chép các stylesheet (`document.styleSheets`) hoặc phần tử `<link rel="stylesheet">` sang cửa sổ phụ trợ để đảm bảo giao diện đồng nhất.
- **Dọn dẹp và Thu hồi**: Khi cửa sổ PiP đóng (do người dùng click nút đóng hệ thống hoặc gọi `pipWindow.close()`), sự kiện `pagehide` trên PiP window sẽ phát ra, cho phép mini app di chuyển các node DOM trở lại cây DOM chính một cách an toàn mà không làm mất trạng thái (state).

---

## 4. W3C Badging API & Web App Manifest 'share_target' Declarative Ingestion

### 4.1. Quản trị Huy hiệu Thông báo Passive qua Badging API
Badging API cung cấp giải pháp thông báo thụ động (passive notification) thanh lịch:
- **`navigator.setAppBadge(count)`**: Thiết lập một con số số nguyên dương trên icon ứng dụng ở màn hình chính, taskbar hoặc dock. Nếu gọi không tham số, hệ thống sẽ hiển thị một chấm báo (unread dot). Khi truyền `0`, hệ thống tự động xóa huy hiệu.
- **`navigator.clearAppBadge()`**: Xóa hoàn toàn huy hiệu unread.
- **Hỗ trợ Service Worker (`WorkerNavigator.setAppBadge`)**: Cho phép cập nhật số lượng tin nhắn chưa đọc hoặc trạng thái đơn hàng trực tiếp từ background push notification ngay cả khi mini app đang đóng hoàn toàn.
- **Hạn chế Lạm dụng**: Super App Store giới hạn số lần cập nhật badge từ background worker tối đa 20 lần/giờ để tiết kiệm pin và chống làm phiền người dùng.

### 4.2. Khai báo Điểm Tiếp nhận Chia sẻ qua Manifest `share_target`
Thành viên `share_target` trong Web App Manifest biến mini app thành một ứng dụng đích tiếp nhận dữ liệu chia sẻ hệ thống:
- **Phương thức GET**: Dành cho việc chia sẻ văn bản, đường dẫn (URL), truy vấn tìm kiếm ngắn.
- **Phương thức POST với `multipart/form-data`**: Bắt buộc khi tiếp nhận tập tin nhị phân (hình ảnh, video, PDF).
- **Đánh chặn qua Service Worker**: Yêu cầu POST được gửi tới Service Worker của mini app. Service Worker sử dụng `event.request.formData()` để trích xuất mảng `File`, lưu tạm vào CacheStorage hoặc IndexedDB, sau đó mở cửa sổ mini app với URL xem trước tập tin.
- **Kiểm soát Bảo mật**: Container Super App thực hiện quét virus, xác minh MIME type và giới hạn kích thước tệp (tối đa 50MB/lần chia sẻ) trước khi chuyển giao tập tin cho mini app.

---

## 5. Bảng Đối chiếu Chi tiết 15 Tiêu chuẩn Kỹ thuật Bổ sung (Milestone 105)

| ID Tiêu chuẩn | Phân loại | Tên Chuẩn / Giao diện | Mức độ Chứng cứ | URL Nguồn Chính thức | Điểm Neo Kỹ thuật Cốt lõi |
|---|---|---|---|---|---|
| `STANDARDS-WEB-LOCKS-SPECIFICATION` | concurrency-and-state-sync | W3C Web Locks API Standard | standards_specification | https://w3c.github.io/web-locks/ | Cấp phát khóa origin-bound, chế độ exclusive/shared, FIFO queue, giải phóng tự động khi crash |
| `STANDARDS-WEB-LOCKS-API-MDN` | concurrency-and-state-sync | Web Locks API Architecture | official_documentation | https://developer.mozilla.org/en-US/docs/Web/API/Web_Locks_API | Vòng đời khóa theo Promise settlement, triệt tiêu polling localStorage, an toàn đa luồng |
| `STANDARDS-LOCK-MANAGER-INTERFACE` | concurrency-and-state-sync | LockManager Interface | official_documentation | https://developer.mozilla.org/en-US/docs/Web/API/LockManager | navigator.locks namespace, phương thức request/query, phân tách queue theo tên khóa |
| `STANDARDS-LOCK-MANAGER-REQUEST-METHOD` | concurrency-and-state-sync | LockManager.request() Options | official_documentation | https://developer.mozilla.org/en-US/docs/Web/API/LockManager/request | Tham số mode, ifAvailable trylock, steal preemption, AbortSignal timeout integration |
| `STANDARDS-LOCK-MANAGER-QUERY-METHOD` | concurrency-and-state-sync | LockManager.query() Telemetry | official_documentation | https://developer.mozilla.org/en-US/docs/Web/API/LockManager/query | Snapshot cấu trúc held & pending, clientId correlation, phát hiện deadlock thời gian thực |
| `STANDARDS-DOCUMENT-PIP-SPECIFICATION` | windowing-and-display-governance | WICG Document Picture-in-Picture | standards_specification | https://wicg.github.io/document-picture-in-picture/ | Cửa sổ nổi always-on-top chứa arbitrary DOM, cùng nguồn gốc với opener, bắt buộc user activation |
| `STANDARDS-DOCUMENT-PIP-REQUEST-WINDOW` | windowing-and-display-governance | DocumentPictureInPicture.requestWindow() | official_documentation | https://developer.mozilla.org/en-US/docs/Web/API/DocumentPictureInPicture/requestWindow | Cấu hình kích thước width/height, disallowReturnToOpener, đảm bảo quy tắc single-instance |
| `STANDARDS-DOCUMENT-PIP-WINDOW-PROPERTY` | windowing-and-display-governance | DocumentPictureInPicture.window Property | official_documentation | https://developer.mozilla.org/en-US/docs/Web/API/DocumentPictureInPicture/window | Tham chiếu trực tiếp cửa sổ PiP đang mở, tự động gán null khi đóng, đồng bộ trạng thái UI |
| `STANDARDS-DOCUMENT-PIP-EVENT-INTERFACE` | windowing-and-display-governance | DocumentPictureInPictureEvent Lifecycle | official_documentation | https://developer.mozilla.org/en-US/docs/Web/API/DocumentPictureInPictureEvent | Sự kiện 'enter' khởi tạo tài nguyên phụ trợ, sự kiện 'pagehide' dọn dẹp và thu hồi DOM |
| `STANDARDS-DOCUMENT-PIP-DEVELOPER-GUIDE` | windowing-and-display-governance | Chrome Document PiP Guide | official_documentation | https://developer.chrome.com/docs/web-platform/document-picture-in-picture | Sao chép styleSheets chủ động, hỗ trợ canvas/WebGL, responsive resizing và an toàn hiển thị |
| `STANDARDS-BADGING-API-SPECIFICATION` | os-integration-and-application-lifecycle | W3C Badging API Standard | official_documentation | https://developer.mozilla.org/en-US/docs/Web/API/Badging_API | Thông báo thụ động qua icon badge, điều phối launcher hệ điều hành, chống spam notification |
| `STANDARDS-NAVIGATOR-SET-APP-BADGE` | os-integration-and-application-lifecycle | Navigator.setAppBadge() Semantics | official_documentation | https://developer.mozilla.org/en-US/docs/Web/API/Navigator/setAppBadge | Đặt giá trị số nguyên hoặc unread dot, hỗ trợ WorkerNavigator trong Service Worker push |
| `STANDARDS-NAVIGATOR-CLEAR-APP-BADGE` | os-integration-and-application-lifecycle | Navigator.clearAppBadge() Semantics | official_documentation | https://developer.mozilla.org/en-US/docs/Web/API/Navigator/clearAppBadge | Xóa huy hiệu lập tức khi đọc tin, tương đương setAppBadge(0), đồng bộ đa thiết bị |
| `STANDARDS-MANIFEST-SHARE-TARGET-MEMBER` | os-integration-and-application-lifecycle | Manifest 'share_target' Member | official_documentation | https://developer.mozilla.org/en-US/docs/Web/Manifest/share_target | Khai báo action, method GET/POST, enctype multipart/form-data, tích hợp share sheet hệ thống |
| `STANDARDS-WEB-SHARE-TARGET-CHROME-GUIDE` | os-integration-and-application-lifecycle | Web Share Target File Ingestion | official_documentation | https://developer.chrome.com/docs/capabilities/web-apis/web-share-target | Service Worker fetch interception, trích xuất FormData Blobs, lưu cache và sandbox file |

---

## 6. Tiêu chí Đánh giá Super App Store (Review & Certification Gates)

1. **Gate Concurrency & Deadlock (Bảo vệ Xung đột Dữ liệu)**:
   - Mini app sử dụng IndexedDB hoặc OPFS trên nhiều luồng bắt buộc phải đồng bộ qua `navigator.locks.request`.
   - Mọi yêu cầu khóa đều phải gắn kèm `signal: AbortSignal.timeout(10000)` để ngăn chặn tình trạng treo vô hạn hàng đợi khóa.
2. **Gate Floating Window (Kiểm soát Cửa sổ Nổi)**:
   - Lệnh gọi `documentPictureInPicture.requestWindow()` phải có transient user gesture rõ ràng.
   - Khi đóng cửa sổ PiP, mini app phải bảo toàn nguyên vẹn dữ liệu form và trạng thái media mà không gây lỗi tham chiếu DOM mồ côi (orphaned nodes).
3. **Gate OS Integration (Huy hiệu & Điểm Tiếp nhận Chia sẻ)**:
   - Huy hiệu `setAppBadge` phải phản ánh chính xác số lượng thông báo thực tế; cấm giữ badge vĩnh viễn khi không có việc cần xử lý.
   - Khai báo `share_target` tiếp nhận tệp tin phải đi kèm cơ chế kiểm tra định dạng và khử độc dữ liệu trước khi xử lý nghiệp vụ.
