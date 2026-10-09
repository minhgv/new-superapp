# Chuyên đề 67 (Iteration 71): W3C Permissions API, WHATWG Streams API & W3C Geometry Interfaces Level 1

## 1. Giới thiệu & Bối cảnh kỹ thuật

Tại Milestone 71, tiêu chuẩn Super-App Mini-App Store Standard tập trung hoàn thiện 3 trụ cột kỹ thuật nền tảng của trình duyệt hiện đại phục vụ môi trường super-app:
1. **W3C Permissions API & Programmatic Permission State Querying**: Chuẩn hóa cơ chế truy vấn trạng thái quyền chủ động thông qua `navigator.permissions.query()`, máy trạng thái 3 giá trị (`granted`, `prompt`, `denied`), đồng bộ sự kiện `change` khi quyền bị thu hồi tại tầng OS/host, và liên kết chặt chẽ với W3C Permissions Policy.
2. **WHATWG Streams API & Zero-Copy Backpressure I/O**: Chuẩn hóa mô hình xử lý luồng dữ liệu tuần tự với `ReadableStream`, `WritableStream`, `TransformStream`, kiểm soát dòng chảy ngược (backpressure signaling via `desiredSize`), đọc dữ liệu zero-copy không cấp phát GC thông qua `ReadableStreamBYOBReader`, và kiểm soát vòng đời khi nhân bản luồng (`tee()`).
3. **W3C Geometry Interfaces Module Level 1 & High-Performance Affine Transforms**: Chuẩn hóa mô hình tính toán hình học 2D/3D affine thông qua `DOMMatrix`, phép chiếu tọa độ và hit-testing với `DOMPoint`, bao đóng hình học và chống che khuất UI với `DOMRect`, mô hình hóa tứ giác biến dạng với `DOMQuad`, và tương thích thuật toán Structured Clone để chuyển giao tính toán nặng sang Web Workers.

Toàn bộ 15 phát hiện kỹ thuật đã được kiểm chứng chuẩn thức (W3C Recommendations / WHATWG Living Standards / MDN References) và tích hợp vào kho bằng chứng canonical tại `state/findings.jsonl`.

---

## 2. Danh mục 15 Phát hiện Kỹ thuật (Findings Detail)

### 2.1 W3C Permissions API & Host Container Permission Mediation

#### [permissions_api_071_01] W3C Permissions API & navigator.permissions.query() Programmatic Status Inspection
- **Tiêu chuẩn**: W3C Permissions §5.1 & §5.2 / MDN Permissions API
- **Evidence Level**: specification
- **URLs**: `https://www.w3.org/TR/permissions/`, `https://w3c.github.io/permissions/`, `https://developer.mozilla.org/en-US/docs/Web/API/Permissions_API`
- **Tóm tắt**: `navigator.permissions.query(permissionDesc)` cho phép ứng dụng bất đồng bộ kiểm tra trạng thái quyền của các tính năng mạnh (camera, geolocation, notifications, screen-wake-lock...) trước khi gọi các API tương tác, tránh hiện tượng bật prompt đột ngột làm gián đoạn trải nghiệm người dùng.
- **Chi tiết kỹ thuật**:
  - *Spec Fact*: `navigator.permissions.query()` nhận vào một `PermissionDescriptor` dictionary có thuộc tính bắt buộc `name` (chuỗi định danh quyền) cùng các tham số mở rộng tùy tính năng (như `userVisibleOnly` cho push, `panTiltZoom` cho camera). Phương thức trả về một Promise giải quyết thành đối tượng `PermissionStatus`. Nếu định danh quyền không hợp lệ, runtime lập tức reject Promise với `TypeError`.
  - *SuperApp Architecture*: Container super-app biến `navigator.permissions.query()` thành chốt chặn bắt buộc (pre-flight gate) trong quy tắc review store. Mini-app không được phép kích hoạt trực tiếp các API nhạy cảm khi chưa kiểm tra trạng thái. Nếu trạng thái là `prompt`, mini-app phải hiển thị giao diện giải thích lý do nghiệp vụ (pre-permission primer modal). Cầu nối native bridge chặn bắt lệnh gọi này và tổng hợp từ cả quyền hệ điều hành (Android ContextCompat / iOS CLLocationManager) lẫn chính sách phân quyền của super-app.

#### [permissions_api_071_02] PermissionStatus Interface: 'granted', 'prompt', and 'denied' Tri-State Lifecycle
- **Tiêu chuẩn**: W3C Permissions §6 PermissionStatus interface & PermissionState enum
- **Evidence Level**: specification
- **URLs**: `https://developer.mozilla.org/en-US/docs/Web/API/PermissionStatus`, `https://www.w3.org/TR/permissions/`
- **Tóm tắt**: Giao diện `PermissionStatus` công bố thuộc tính `state` thuộc kiểu enum 3 trạng thái (`granted`, `prompt`, `denied`), xác định rõ ràng quyền truy cập và ngăn chặn tình trạng spam prompt.
- **Chi tiết kỹ thuật**:
  - *Spec Fact*: Thuộc tính `state` phản ánh 3 trạng thái nghiêm ngặt: `granted` (được phép thực thi không cần hỏi lại), `denied` (bị từ chối hoàn toàn, gọi API sẽ lập tức văng lỗi `NotAllowedError` mà không hỏi người dùng), và `prompt` (cần sự đồng thuận tương tác của người dùng).
  - *SuperApp Architecture*: Khi `state === 'denied'` do người dùng từ chối tại OS hoặc chính sách bảo mật super-app, mini-app bị nghiêm cấm việc cố tình lặp lại hành động kích hoạt quyền. Thay vào đó, mini-app phải hiển thị nút điều hướng người dùng tới màn hình Cài đặt của Super-App (`superapp.openSettings({ section: "permissions" })`). Kiểm duyệt store tự động kiểm tra xem mã nguồn mini-app có xử lý nhánh `denied` một cách êm thuận (graceful degradation) hay không.

#### [permissions_api_071_03] PermissionStatus 'onchange' Event & Dynamic OS-Level Revocation Synchronization
- **Tiêu chuẩn**: W3C Permissions §6.1 The change event
- **Evidence Level**: specification
- **URLs**: `https://developer.mozilla.org/en-US/docs/Web/API/PermissionStatus/change_event`, `https://www.w3.org/TR/permissions/`
- **Tóm tắt**: Sự kiện `change` trên `PermissionStatus` cho phép mini-app lắng nghe các thay đổi trạng thái quyền phát sinh từ bên ngoài (khi người dùng đổi cài đặt hệ điều hành hoặc admin thu hồi quyền).
- **Chi tiết kỹ thuật**:
  - *Spec Fact*: User agent có trách nhiệm xếp hàng một task để kích hoạt sự kiện `change` tại thực thể `PermissionStatus` tương ứng bất cứ khi nào quyền thay đổi. Sự kiện kế thừa từ `Event`, cho phép gán bộ lắng nghe qua `addEventListener('change', ...)` hoặc `status.onchange = ...`.
  - *SuperApp Architecture*: Khi người dùng tạm ẩn super-app để vào Cài đặt Android/iOS thu hồi quyền máy ảnh hoặc vị trí, super-app lưu trữ webview trong Back/Forward Cache. Khi super-app kích hoạt trở lại (Android `OnRequestPermissionsResultCallback` / iOS `UIApplicationDidBecomeActiveNotification`), container bridge lập tức đồng bộ và bắn sự kiện `change` giả lập vào webview. Mini-app đón nhận sự kiện để giải phóng tài nguyên phần cứng ngay lập tức, ngăn ngừa crash do truy cập tài nguyên đã bị tước quyền.

#### [permissions_api_071_04] W3C Permissions Policy Cohesion & Cross-Origin Iframe Permission Attenuation
- **Tiêu chuẩn**: W3C Permissions §4 & W3C Permissions Policy §4
- **Evidence Level**: specification
- **URLs**: `https://w3c.github.io/permissions/`, `https://developer.mozilla.org/en-US/docs/Web/HTTP/Permissions_Policy`
- **Tóm tắt**: Truy vấn trạng thái quyền tuân thủ triệt để tiêu chuẩn W3C Permissions Policy; bất kỳ tính năng nào bị chặn bởi header `Permissions-Policy` hoặc thuộc tính iframe `allow` sẽ ngay lập tức trả về `denied`.
- **Chi tiết kỹ thuật**:
  - *Spec Fact*: Thuật toán truy vấn của Permissions API kiểm tra chính sách Permissions Policy trước khi xét đến sự đồng ý của người dùng. Nếu chính sách vô hiệu hóa tính năng (ví dụ `camera=()`), phương thức `query()` trả về `state: "denied"` ngay cả khi người dùng từng cấp quyền trước đó.
  - *SuperApp Architecture*: Gateway super-app tự động tổng hợp header `Permissions-Policy` dựa trên manifest mà mini-app đã đăng ký và được phê duyệt khi submit lên store. Nếu mini-app không khai báo quyền `camera` trong `manifest.json`, header sẽ áp dụng `camera=()`. Khi bất kỳ mã độc của bên thứ 3 nào trong webview gọi `navigator.permissions.query({ name: 'camera' })`, kết quả lập tức là `denied`, chặn đứng hoàn toàn nguy cơ leo thang đặc quyền qua iframe nhúng.

#### [permissions_api_071_05] Super-App Container Multi-Tenant Permission Mediation & Store Compliance Gating
- **Tiêu chuẩn**: W3C Permissions §8 Security and Privacy Considerations & Container Native Bridge Architecture
- **Evidence Level**: specification
- **URLs**: `https://developer.mozilla.org/en-US/docs/Web/API/Permissions_API`, `https://www.w3.org/TR/permissions/`
- **Tóm tắt**: Kiến trúc container super-app hòa giải các truy vấn quyền thông qua ma trận đánh giá 4 tầng: Khai báo Manifest, Chính sách Doanh nghiệp Tenant, Bảo vệ Ngữ cảnh Host, và Phân quyền Native OS.
- **Chi tiết kỹ thuật**:
  - *Spec Fact*: Tiêu chuẩn cho phép user agent tùy biến thuật toán cấp quyền, áp dụng cơ chế cấp quyền tạm thời một lần (ephemeral/one-time grants) tự động hủy khi context kết thúc để chống theo dõi chéo nguồn.
  - *SuperApp Architecture*: Trong các hệ sinh thái lớn (Viettel, WeChat, Alipay), container áp dụng quy trình 4 tầng: (1) Manifest Scope Verification (quyền phải nằm trong danh mục đã duyệt trên Store); (2) Enterprise Tenant Policy (chính sách MDM doanh nghiệp có thể cấm toàn bộ Bluetooth/Ghi màn hình); (3) Container Trust Guard (kiểm tra `UserActivation.isActive` để chống bot tự động query); và (4) Native OS Permissions. Quy trình duyệt tự động trên store dùng static analysis để bảo đảm mọi lệnh gọi API nhạy cảm đều có pre-flight query tương ứng.

---

### 2.2 WHATWG Streams API & Zero-Copy Backpressure I/O

#### [streams_api_071_01] WHATWG Streams API: ReadableStream Architecture, Controller States, and Backpressure Flow Control
- **Tiêu chuẩn**: WHATWG Streams Standard §3.2 & §3.3
- **Evidence Level**: specification
- **URLs**: `https://streams.spec.whatwg.org/`, `https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream`
- **Tóm tắt**: `ReadableStream` cung cấp cơ chế tiêu thụ dữ liệu tuần tự theo từng chunk bất đồng bộ, tích hợp sẵn tín hiệu dòng chảy ngược (backpressure signaling) qua thuộc tính `desiredSize` để chống tràn bộ nhớ ứng dụng tiêu thụ.
- **Chi tiết kỹ thuật**:
  - *Spec Fact*: `ReadableStream` đóng gói nguồn dữ liệu bên dưới và được điều phối bởi `ReadableStreamDefaultController`. Khi người tiêu thụ gọi `getReader()`, stream bị khóa (`locked = true`). Controller duy trì hàng đợi và công bố thuộc tính `desiredSize`. Nếu phía đọc xử lý chậm hơn phía ghi, `desiredSize` giảm về 0 hoặc số âm, phát tín hiệu yêu cầu phía phát tạm dừng nạp dữ liệu.
  - *SuperApp Architecture*: Thiết bị di động có dung lượng RAM hữu hạn. Nếu tải các file dữ liệu lớn (bản đồ offline, danh mục sản phẩm, báo cáo tài chính) nguyên khối dạng ArrayBuffer, hiện tượng cấp phát bộ nhớ đột biến sẽ kích hoạt cơ chế Low Memory Killer (LMK) của Android hoặc Jetsam của iOS làm văng app. Super-app chuẩn hóa việc truyền tải dữ liệu qua `ReadableStream`. Container bridge theo dõi `desiredSize` để tạm dừng đọc socket mạng khi webview bị nghẽn.

#### [streams_api_071_02] WritableStream & UnderlyingSink Lifecycle: Asynchronous Write Pacing and Error Propagation
- **Tiêu chuẩn**: WHATWG Streams Standard §4.2 & §4.3
- **Evidence Level**: specification
- **URLs**: `https://developer.mozilla.org/en-US/docs/Web/API/WritableStream`, `https://streams.spec.whatwg.org/`
- **Tóm tắt**: Giao diện `WritableStream` trừu tượng hóa điểm đích ghi dữ liệu (sink), điều phối các promise ghi, hàng đợi backpressure và lan truyền tín hiệu hủy/lỗi.
- **Chi tiết kỹ thuật**:
  - *Spec Fact*: `WritableStream` bọc một `underlyingSink` với các hook vòng đời: `start`, `write(chunk, controller)`, `close`, và `abort(reason)`. Lệnh gọi `writer.write(chunk)` trả về Promise chỉ hoàn thành khi sink đã tiếp nhận dữ liệu. Thuộc tính `writer.ready` báo hiệu khi hàng đợi có khoảng trống để tiếp tục ghi. Nếu xảy ra lỗi hoặc gọi `writer.abort()`, lỗi lan truyền toàn diện qua pipeline và giải phóng tài nguyên.
  - *SuperApp Architecture*: Mini-app thường xuyên đẩy các gói log vi phạm, telemetry và giao dịch offline lên server. Đóng gói các yêu cầu tải lên qua `WritableStream` cho phép container điều phối tốc độ ghi, tránh làm nghẽn kênh IPC native-to-web. Nếu mạng bị đứt giữa chừng, container kích hoạt abort để giải phóng ngay các buffer native đang giữ.

#### [streams_api_071_03] TransformStream & pipeThrough/pipeTo Composition: Composable In-Flight Processing
- **Tiêu chuẩn**: WHATWG Streams Standard §5.2 & §3.2.5.2
- **Evidence Level**: specification
- **URLs**: `https://developer.mozilla.org/en-US/docs/Web/API/TransformStream`, `https://streams.spec.whatwg.org/`
- **Tóm tắt**: `TransformStream` kết hợp một đầu đọc và một đầu ghi thành chuỗi biến đổi dữ liệu song công (duplex pipeline), hỗ trợ giải nén, giải mã mật mã và phân tích cú pháp trực tiếp trong quá trình truyền qua `pipeThrough()`.
- **Chi tiết kỹ thuật**:
  - *Spec Fact*: `TransformStream` chứa một cặp stream: writable side nhận chunk đầu vào thô và readable side phát ra chunk đã biến đổi thông qua `transform(chunk, controller)`. Phương thức `pipeThrough()` và `pipeTo()` tự động kết nối tín hiệu backpressure ngược từ sink cuối cùng về tận producer ban đầu, đồng thời bảo đảm lan truyền lỗi và hủy bỏ toàn vẹn.
  - *SuperApp Architecture*: Các gói tài nguyên mini-app được nén và mã hóa tại biên. Client super-app thiết lập đường ống: `response.body.pipeThrough(new DecompressionStream('gzip')).pipeThrough(new DecryptionTransformStream(key)).pipeThrough(new JsonLinesParserStream())`. Dữ liệu cấu hình được hiển thị ngay khi các packet đầu tiên về máy, rút ngắn chỉ số Time to Interactive (TTI) mà không tạo ra các file tạm chiếm dụng RAM.

#### [streams_api_071_04] ReadableStreamBYOBReader & Byte Streams: Zero-Copy ArrayBuffer Transfer & Memory Optimization
- **Tiêu chuẩn**: WHATWG Streams Standard §3.7 & §3.5
- **Evidence Level**: specification
- **URLs**: `https://developer.mozilla.org/en-US/docs/Web/API/ReadableStreamBYOBReader`, `https://streams.spec.whatwg.org/`
- **Tóm tắt**: `ReadableStreamBYOBReader` ("Bring Your Own Buffer") cho phép đọc byte stream trực tiếp vào bộ đệm ArrayBuffer do ứng dụng tự cấp phát trước, triệt tiêu hoàn toàn gánh nặng rác bộ nhớ (GC churn).
- **Chi tiết kỹ thuật**:
  - *Spec Fact*: Khởi tạo bằng `stream.getReader({ mode: "byob" })` trên byte stream. Phương thức `reader.read(view)` nhận vào một `ArrayBufferView` (như Uint8Array) từ phía gọi. Engine trình duyệt chuyển quyền sở hữu vùng nhớ trực tiếp cho buffer I/O native và trả về view đã chứa dữ liệu kèm `bytesRead`. Việc tái sử dụng cùng một vùng nhớ qua các vòng đọc giúp số lượng đối tượng rác sinh ra trên heap của JavaScript bằng 0, loại bỏ hoàn toàn hiện tượng GC pause.
  - *SuperApp Architecture*: Khi mini-app xử lý các tác vụ streaming tần suất cao (đồ thị chứng khoán thời gian thực, đồng bộ game state, nạp texture WebGPU, quét barcode từ camera), việc GC ngắt quãng luồng xử lý sẽ gây giật lag tụt khung hình dưới 60fps. Container super-app chuẩn hóa cơ chế BYOB trên các cầu nối truyền dữ liệu nhị phân tốc độ cao, dùng vòng đệm tĩnh 64KB để giữ vững độ mượt UI.

#### [streams_api_071_05] Stream Teeing (ReadableStream.tee()), Locking Invariants & Memory Leak Prevention
- **Tiêu chuẩn**: WHATWG Streams Standard §3.2.5.4 & §3.2.2
- **Evidence Level**: specification
- **URLs**: `https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream/tee`, `https://streams.spec.whatwg.org/`
- **Tóm tắt**: Phương thức `tee()` tách một `ReadableStream` thành 2 nhánh độc lập; đòi hỏi quản trị vòng đời nghiêm ngặt để tránh rò rỉ bộ nhớ khi hai nhánh tiêu thụ dữ liệu với tốc độ lệch pha.
- **Chi tiết kỹ thuật**:
  - *Spec Fact*: `readableStream.tee()` khóa stream gốc và trả về 2 thực thể stream mới `[branch1, branch2]`. Nếu một nhánh đọc dữ liệu còn nhánh kia bị bỏ quên hoặc tạm dừng, engine stream buộc phải lưu tạm (buffer) toàn bộ các chunk trong RAM để chờ nhánh còn lại. Nếu nhánh không dùng không được gọi `cancel()`, bộ nhớ đệm sẽ phình to không giới hạn gây tràn RAM.
  - *SuperApp Architecture*: Trong kiến trúc super-app, luồng dữ liệu mạng thường được nhân bản: nhánh 1 cấp cho UI hiển thị, nhánh 2 ghi vào cache cục bộ hoặc audit log. Container SDK cung cấp cơ chế bọc an toàn: nếu nhánh audit không tiêu thụ dữ liệu trong vòng 5 giây, container tự động kích hoạt `branch2.cancel('analytics_timeout')` để xả hàng đợi, bảo vệ nhánh UI chính và triệt tiêu nguy cơ rò rỉ bộ nhớ.

---

### 2.3 W3C Geometry Interfaces Module Level 1 & High-Performance Affine Transforms

#### [geometry_interfaces_071_01] W3C DOMMatrix Interface: 2D & 3D Affine Transformations, Matrix Math, and Hardware Composition
- **Tiêu chuẩn**: W3C Geometry Interfaces Module Level 1 §3 / MDN DOMMatrix
- **Evidence Level**: specification
- **URLs**: `https://www.w3.org/TR/geometry-1/`, `https://drafts.fxtf.org/geometry/`, `https://developer.mozilla.org/en-US/docs/Web/API/DOMMatrix`
- **Tóm tắt**: Giao diện `DOMMatrix` cung cấp ma trận toán học 4x4 chuẩn hóa, tăng tốc phần cứng cho các phép biến đổi hệ tọa độ affine 2D và 3D trong engine kết xuất web.
- **Chi tiết kỹ thuật**:
  - *Spec Fact*: `DOMMatrix` hỗ trợ đầy đủ biến đổi 2D (các hệ số a, b, c, d, e, f) và 3D (từ m11 đến m44). Cung cấp các phương thức toán học chuyên sâu: `translate`, `scale`, `rotate`, `multiply`, `inverse`, `transformPoint`. Thuộc tính `is2D` cho phép engine tối ưu hóa bằng cách bỏ qua các phép tính phối cảnh 3D không cần thiết. Phương thức `fromFloat32Array()` hỗ trợ nạp trực tiếp ma trận từ WebGL/WebGPU vertex buffer.
  - *SuperApp Architecture*: Mini-app thường tích hợp bản đồ số, biểu đồ động và canvas tương tác. Tính toán ma trận thủ công bằng JavaScript tiêu tốn CPU và tạo rác bộ nhớ. Bằng cách dùng `DOMMatrix` nguyên bản, các phép nghịch đảo ma trận và nhân ma trận phức tạp được giao thẳng cho tầng C++ của trình duyệt và phần cứng GPU, đảm bảo các thao tác zoom/pan cử chỉ đạt tốc độ 60fps mượt mà.

#### [geometry_interfaces_071_02] DOMPoint & Coordinate Projection: 2D/3D Point Arithmetic and Matrix Multiplication
- **Tiêu chuẩn**: W3C Geometry Interfaces Module Level 1 §1 / MDN DOMPoint
- **Evidence Level**: specification
- **URLs**: `https://developer.mozilla.org/en-US/docs/Web/API/DOMPoint`, `https://www.w3.org/TR/geometry-1/`
- **Tóm tắt**: Giao diện `DOMPoint` đại diện cho tọa độ không gian 2D/3D (x, y, z, w), hỗ trợ phép biến đổi ma trận thông qua `matrixTransform()` để thực hiện phép chiếu tọa độ và hit-testing chính xác.
- **Chi tiết kỹ thuật**:
  - *Spec Fact*: `DOMPointReadOnly` và `DOMPoint` lưu trữ các thành phần x, y, z và hệ số phối cảnh w (mặc định 1.0). Phương thức `matrixTransform(matrix)` nhân vector tọa độ thuần nhất 4 chiều với ma trận 4x4 của `DOMMatrix`, trả về thực thể `DOMPoint` mới với tọa độ đã biến đổi trong thời gian hằng số O(1).
  - *SuperApp Architecture*: Trong các ứng dụng e-commerce hay kiosk bán vé (chọn ghế rạp chiếu phim, sơ đồ mặt bằng), các phần tử đồ họa thường bị xoay hoặc thu phóng. Khi người dùng chạm màn hình (`PointerEvent.clientX/Y`), việc quy đổi tọa độ màn hình về tọa độ nội tại của phần tử được thực hiện tức thì qua `DOMPoint.fromPoint({x, y}).matrixTransform(elementMatrix.inverse())`, loại bỏ các đoạn mã lượng giác phức tạp và dễ lỗi.

#### [geometry_interfaces_071_03] DOMRect & Bounding Geometry: Axis-Aligned Box Mathematics and Viewport Collision Sandboxing
- **Tiêu chuẩn**: W3C Geometry Interfaces Module Level 1 §2 / MDN DOMRect
- **Evidence Level**: specification
- **URLs**: `https://developer.mozilla.org/en-US/docs/Web/API/DOMRect`, `https://www.w3.org/TR/geometry-1/`
- **Tóm tắt**: `DOMRect` định nghĩa hình chữ nhật chuẩn hóa qua tọa độ gốc (x, y) và kích thước (width, height), đóng vai trò nền tảng cho việc tính toán biên và phát hiện va chạm hộp trục tọa độ (AABB).
- **Chi tiết kỹ thuật**:
  - *Spec Fact*: `DOMRect` công bố các thuộc tính x, y, width, height, cùng các cạnh biên dẫn xuất: top, right, bottom, left. Tiêu chuẩn hỗ trợ cả trường hợp kích thước âm: nếu width âm thì left là `x + width` và right là `x`. `DOMRect` là đối tượng trả về của các API quan sát layout chủ chốt như `Element.getBoundingClientRect()`.
  - *SuperApp Architecture*: Container super-app bắt buộc bảo vệ các nút điều hướng native (như nút capsule đóng/mở mini-app, header an toàn) không bị giao diện mini-app đè lên hoặc che khuất (chống tấn công Clickjacking/UI Redressing). Hệ thống giám sát layout định kỳ kiểm tra `DOMRect` của các lớp phủ modal trong mini-app; nếu phát hiện va chạm biên với vùng tọa độ capsule (`wx.getMenuButtonBoundingClientRect`), container sẽ tự động đẩy lùi hoặc cắt tỉa phần tử vi phạm.

#### [geometry_interfaces_071_04] DOMQuad Interface: Non-Axis-Aligned Quadrilateral Sandboxing and Bounds Calculation
- **Tiêu chuẩn**: W3C Geometry Interfaces Module Level 1 §4 / MDN DOMQuad
- **Evidence Level**: specification
- **URLs**: `https://developer.mozilla.org/en-US/docs/Web/API/DOMQuad`, `https://www.w3.org/TR/geometry-1/`
- **Tóm tắt**: Giao diện `DOMQuad` đại diện cho một hình tứ giác tùy ý được xác định bởi 4 đỉnh `DOMPoint` (p1, p2, p3, p4), cho phép mô hình hóa chính xác các phần tử bị biến dạng xoay/nghiêng 3D.
- **Chi tiết kỹ thuật**:
  - *Spec Fact*: Khác với `DOMRect` chỉ dành cho hình chữ nhật song song với trục tọa độ, `DOMQuad` mô tả các tứ giác lồi bất kỳ sau biến đổi CSS transform. Phương thức then chốt `getBounds()` tự động tính toán và trả về một `DOMRectReadOnly` bao quanh nhỏ nhất ôm trọn cả 4 góc của tứ giác sau khi đã tính đến độ nghiêng và xoay 3D.
  - *SuperApp Architecture*: Trong các tác vụ quét giấy tờ tùy thân eKYC, nhận diện hóa đơn OCR và quét mã QR từ luồng video camera, tài liệu thực tế luôn bị chụp ở góc nghiêng. SDK thị giác máy tính của super-app trả về kết quả định vị dưới dạng `DOMQuad`. Mini-app sử dụng trực tiếp đối tượng này để vẽ khung viền căn chỉnh và cắt phối cảnh mà không cần chuyển đổi trung gian tốn kém.

#### [geometry_interfaces_071_05] Geometry Interfaces Serialization, Structured Clone Compatibility, and Worker Offloading
- **Tiêu chuẩn**: W3C Geometry Interfaces Module Level 1 §5 & WHATWG HTML Structured Clone
- **Evidence Level**: specification
- **URLs**: `https://drafts.fxtf.org/geometry/`, `https://www.w3.org/TR/geometry-1/`
- **Tóm tắt**: Toàn bộ các giao diện hình học W3C đều tương thích hoàn toàn với thuật toán Structured Clone và hỗ trợ `toJSON()`, cho phép chuyển giao các phép toán hình học nặng sang Web Workers.
- **Chi tiết kỹ thuật**:
  - *Spec Fact*: Các đối tượng `DOMPoint`, `DOMRect`, `DOMQuad`, `DOMMatrix` đều là đối tượng có thể tuần tự hóa (serializable). Chúng có thể được gửi trực tiếp qua `postMessage()` giữa Window, Dedicated Web Workers và Shared Workers mà không cần ép kiểu thủ công. Mỗi giao diện đều có phương thức `toJSON()` trả về object thuần chứa các tọa độ số.
  - *SuperApp Architecture*: Các mini-app hiệu năng cao (xem bản vẽ CAD kỹ thuật, thiết kế 3D, game engine) phải thực hiện hàng ngàn phép nhân ma trận và tính toán va chạm mỗi giây. Chạy các phép toán này trên main thread sẽ phá vỡ chỉ số Interaction to Next Paint (INP) và gây giật khung hình. Kiến trúc chuẩn super-app yêu cầu chuyển toàn bộ việc tính cây va chạm và biến đổi hình học sang Web Worker bằng cách truyền `DOMMatrix` và `DOMQuad` qua `postMessage()`, giữ vững tốc độ khung hình 120fps cho giao diện người dùng.

---

## 3. Ma trận Kiến trúc Tiêu chuẩn Super-App (Milestone 71 Integration)

| Thành phần | Tiêu chuẩn Cốt lõi | Trách nhiệm Mini-App | Trách nhiệm Super-App Container / Gateway | Chỉ số Kiểm duyệt Store (Gating Criteria) |
|---|---|---|---|---|
| **Permission Introspection** | W3C Permissions API (`query`) | Gọi `navigator.permissions.query()` kiểm tra trước khi gọi API nhạy cảm; hiển thị pre-permission primer khi `state === 'prompt'`. | Proxy lệnh gọi query sang ma trận quyền 4 tầng (Manifest, Enterprise MDM, UserActivation, Native OS TCC); bắn sự kiện `change` khi OS thay đổi. | Cấm tuyệt đối spam prompt; từ chối app nếu gọi camera/location mà không có pre-flight check hoặc thiếu rationale. |
| **Streaming Data Flow** | WHATWG Streams API (`ReadableStream`, `WritableStream`) | Sử dụng stream pipelines với `pipeThrough()` để xử lý dữ liệu lớn; áp dụng `ReadableStreamBYOBReader` cho dữ liệu nhị phân tốc độ cao. | Giám sát `desiredSize` để điều phối dòng chảy ngược (backpressure); tự động cancel các nhánh `tee()` analytics bị timeout sau 5s. | Kiểm toán rò rỉ RAM; cấm tải nguyên khối buffer >10MB trên mạng di động; kiểm tra xử lý lỗi pipeline `abort()`. |
| **Affine Geometry Math** | W3C Geometry Interfaces (`DOMMatrix`, `DOMPoint`, `DOMQuad`) | Sử dụng các đối tượng hình học native thay vì thư viện ma trận JS ngoài; offload tính toán va chạm sang Dedicated Web Worker. | Sử dụng `DOMRect` để bảo vệ vùng nút capsule điều hướng; cung cấp `DOMQuad` từ engine OCR/vision native sang webview không qua serialize trung gian. | Đo lường chỉ số INP < 200ms; phát hiện và chặn các phần tử UI vi phạm vùng an toàn của host container capsule. |

---

## 4. Kết luận & Định hướng Kế tiếp

Milestone 71 đã nâng tổng số phát hiện chuẩn mực trong kho tri thức canonical lên **1016 findings** (vượt mốc 1000 findings). Ba mảng công nghệ vừa hoàn thiện đóng vai trò xương sống cho việc tối ưu hóa hiệu năng, trải nghiệm cấp quyền minh bạch và đồ họa tính toán chuẩn xác cho các siêu ứng dụng thế hệ mới.
