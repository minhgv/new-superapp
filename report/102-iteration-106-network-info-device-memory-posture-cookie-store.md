# Chuyên đề Iteration 106: Network Information API & Device Memory Hardware Optimization, W3C Device Posture API, and Cookie Store API Asynchronous Session Governance

## 1. Bối cảnh & Mục tiêu Kỹ thuật
Trong kiến trúc siêu ứng dụng (Super App Mini App Ecosystem), việc tối ưu hoá trải nghiệm người dùng trên các cấu hình phần cứng đa dạng—từ thiết bị di động phổ thông dung lượng RAM hạn chế (<1GB RAM), kết nối mạng chập chờn 2G/3G, cho đến các thiết bị cao cấp màn hình gập (Foldables) hai màn hình—đòi hỏi nền tảng phải cung cấp các chuẩn giao tiếp phần cứng tiêu chuẩn và phi chặn.

Tại Milestone 106, Deli Deep tập trung chuẩn hoá 3 khía cạnh nền tảng cốt lõi:
1. **Network Information API & W3C Device Memory 1**: Đo lường băng thông, độ trễ và lượng RAM định lượng để tự động phân cấp tải tài nguyên (resource degradation), tiết kiệm dữ liệu (`saveData`), và quản lý bộ nhớ tiến trình webview.
2. **W3C Device Posture API**: Điều phối layout giao diện phản ứng theo trạng thái cơ học (gập / phẳng) của thiết bị màn hình gập mà không để lộ cảm biến góc gập chi tiết gây fingerprinting.
3. **Cookie Store API & CookieChangeEvent**: Quản trị phiên và xác thực phi đồng bộ (asynchronous cookie access) ngoài luồng chính (off-main-thread) và trong Service Worker, phản ứng tức thì với việc đăng xuất/thu hồi token qua pipeline sự kiện thời gian thực.

---

## 2. Chi tiết Nghiên cứu & Chuẩn hoá Kiến trúc

### A. Network Information API & Device Memory Hardware Optimization
* **NetworkInformation (`navigator.connection`)**: Cung cấp telemetry thời gian thực về giao diện mạng đang hoạt động trong ngữ cảnh an toàn HTTPS. Bao gồm `downlink` (băng thông danh định ước lượng Mbps), `rtt` (thời gian khứ hồi ms), và lắng nghe sự kiện `change`.
* **Phân lớp chất lượng mạng (`effectiveType`)**: Chuẩn hoá trạng thái mạng thành các nhóm định danh: `'slow-2g'`, `'2g'`, `'3g'`, `'4g'`. Mini App container sử dụng tín hiệu này để hạ cấp đường truyền: tắt tính năng tự động phát video/animation, giảm độ phân giải hình ảnh, và trì hoãn các tác vụ đồng bộ nền không khẩn cấp.
* **Tiết kiệm dữ liệu người dùng (`saveData`)**: Cờ boolean phản ánh cài đặt Data Saver từ hệ điều hành. Khi bật, mini app bắt buộc phải cắt giảm tải trọng subresource, không tải trước tài nguyên suy đoán (speculative prefetch), và tôn trọng hạn mức băng thông người dùng. Siêu ứng dụng tự động inject header HTTP `Save-Data: on` vào các network proxy subresource.
* **W3C Device Memory 1**: Định lượng dung lượng RAM thiết bị thành các luỹ thừa cơ số 2 được chuẩn hoá: 0.25, 0.5, 1, 2, 4, 8 GiB. Trên các thiết bị <= 1GB RAM, container tự động giới hạn kích thước WebAssembly heap, giảm kích thước buffer WebGL, và thực hiện thu hồi tài nguyên tiến trình nền quyết liệt để tránh hiện tượng Out-Of-Memory (OOM) crash ở tầng OS.

### B. W3C Device Posture API & Foldable/Dual-Screen Mechanical Layout Orchestration
* **Chuẩn W3C Device Posture API**: Tiêu chuẩn hoá cách thức web container nhận diện tư thế cơ học của màn hình dẻo hoặc bản lề kép thông qua giao diện `DevicePosture` trên `navigator.devicePosture`.
* **Bảo vệ quyền riêng tư (Anti-Fingerprinting)**: Tiêu chuẩn cố tình không để lộ số đo góc bản lề chi tiết (hinge angle in degrees) nhằm ngăn chặn việc nhận diện định danh phần cứng độc nhất hoặc tấn công acoustic side-channel qua con quay hồi chuyển; API chỉ trả về các trạng thái phân loại định danh.
* **Thuộc tính `DevicePosture.type`**: Cung cấp hai trạng thái nền tảng: `'continuous'` (màn hình phẳng liên tục) và `'folded'` (thiết bị đang gập tạo góc giữa các bề mặt hiển thị). Tích hợp với CSS Media Queries thông qua `@media (device-posture: folded)` cho phép nhà phát triển bố trí lại giao diện chia đôi (ví dụ: hiển thị camera/bản đồ ở nửa trên và bàn phím/bảng điều khiển ở nửa dưới).
* **Đồng bộ chu kỳ vẽ (`change` event)**: Sự kiện `change` kích hoạt đồng bộ với reflow của viewport. Host container tự động áp dụng cơ chế throttling nhằm ngăn chặn rung lắc bố cục (layout thrashing) hoặc gia tăng Cumulative Layout Shift (CLS) khi người dùng di chuyển hoặc mở/gập thiết bị liên tục.

### C. Cookie Store API & Asynchronous Session Governance
* **Asynchronous Cookie Architecture**: Khắc phục nhược điểm nghiêm trọng của `document.cookie` (vốn là API đồng bộ gây block main-thread I/O khi đọc/ghi file lưu trữ trên đĩa). `cookieStore.get()`, `cookieStore.set()`, `cookieStore.delete()` trả về Promise và có thể vận hành trong cả Worker / ServiceWorker.
* **Xác thực phi đồng bộ trong Service Worker**: Cho phép background service worker kiểm tra cookie xác thực trước khi định tuyến request hoặc cập nhật bộ nhớ đệm CacheStorage, hỗ trợ kiến trúc Offline-First an toàn.
* **Giám sát phiên thời gian thực (`change` event & `CookieChangeEvent`)**: Cung cấp danh sách các cookie được cập nhật (`changed`) hoặc xoá bỏ (`deleted`). Giúp giao diện mini app và worker nhận biết ngay lập tức sự kiện đăng xuất SSO, hết hạn token hoặc bị thu hồi quyền từ siêu ứng dụng cha mà không cần polling định kỳ.
* **Bảo mật Sandbox Cookie**: Mọi cookie được khởi tạo qua `cookieStore.set()` mặc định tuân thủ `SameSite=Lax` và `Secure=true`. Siêu ứng dụng cô lập cookie jar theo từng partition origin của mini app, triệt tiêu nguy cơ rò rỉ session cross-app.

---

## 3. Danh mục 15 Findings Chuẩn hoá Mới (Milestone 106)

| # | Finding ID | Phân loại | Tiêu đề Chuẩn & API | Mức độ Bằng chứng | URL Xác thực (HTTP 200) |
|---|---|---|---|---|---|
| 1 | `STANDARDS-NETWORK-INFORMATION-MDN` | runtime-environment-and-hardware-adaptation | Network Information API: Dynamic Bandwidth & Latency Telemetry for Adaptive Mini-App Runtimes | official_documentation | [MDN NetworkInformation](https://developer.mozilla.org/en-US/docs/Web/API/NetworkInformation) |
| 2 | `STANDARDS-NETWORK-INFORMATION-EFFECTIVETYPE` | runtime-environment-and-hardware-adaptation | NetworkInformation.effectiveType: Cellular Quality Bucketing & Asset Degradation Architecture | official_documentation | [MDN effectiveType](https://developer.mozilla.org/en-US/docs/Web/API/NetworkInformation/effectiveType) |
| 3 | `STANDARDS-NETWORK-INFORMATION-SAVEDATA` | runtime-environment-and-hardware-adaptation | NetworkInformation.saveData: End-User Data-Saving Preference Ingestion & Payload Budgeting | official_documentation | [MDN saveData](https://developer.mozilla.org/en-US/docs/Web/API/NetworkInformation/saveData) |
| 4 | `STANDARDS-NAVIGATOR-CONNECTION-INTERFACE` | runtime-environment-and-hardware-adaptation | Navigator.connection: Centralized Gateway for Mobile Connectivity State Machines | official_documentation | [MDN Navigator.connection](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/connection) |
| 5 | `STANDARDS-W3C-DEVICE-MEMORY-SPECIFICATION` | runtime-environment-and-hardware-adaptation | W3C Device Memory 1: Hardware RAM Quantization & Client-Hints Resource Allocation | standards_specification | [W3C Device Memory 1](https://www.w3.org/TR/device-memory-1/) |
| 6 | `STANDARDS-W3C-DEVICE-POSTURE-SPECIFICATION` | runtime-environment-and-hardware-adaptation | W3C Device Posture API: Physical Form Factor & Hinge Sensor Orchestration | standards_specification | [W3C Device Posture](https://www.w3.org/TR/device-posture/) |
| 7 | `STANDARDS-DEVICE-POSTURE-API-MDN` | runtime-environment-and-hardware-adaptation | Device Posture API Architecture: Responsive Foldable UI Adapters & Posture Events | official_documentation | [MDN Device Posture API](https://developer.mozilla.org/en-US/docs/Web/API/Device_Posture_API) |
| 8 | `STANDARDS-DEVICEPOSTURE-INTERFACE` | runtime-environment-and-hardware-adaptation | DevicePosture Interface: Event-Driven Lifecycle & DOM Integration | official_documentation | [MDN DevicePosture](https://developer.mozilla.org/en-US/docs/Web/API/DevicePosture) |
| 9 | `STANDARDS-DEVICEPOSTURE-TYPE-ATTRIBUTE` | runtime-environment-and-hardware-adaptation | DevicePosture.type: Categorical Posture Enum & Layout State Mapping | official_documentation | [MDN DevicePosture.type](https://developer.mozilla.org/en-US/docs/Web/API/DevicePosture/type) |
| 10 | `STANDARDS-DEVICEPOSTURE-CHANGE-EVENT` | runtime-environment-and-hardware-adaptation | DevicePosture change Event: Dynamic Hardware Transition Orchestration & Viewport Reflow | official_documentation | [MDN DevicePosture change_event](https://developer.mozilla.org/en-US/docs/Web/API/DevicePosture/change_event) |
| 11 | `STANDARDS-COOKIESTORE-CHANGE-EVENT` | security-and-data-protection | CookieStore change Event: Reactive Session Invalidation & Real-Time Auth Synchronization | official_documentation | [MDN CookieStore change_event](https://developer.mozilla.org/en-US/docs/Web/API/CookieStore/change_event) |
| 12 | `STANDARDS-COOKIESTORE-GET-METHOD` | security-and-data-protection | CookieStore.get(): Non-Blocking Asynchronous Cookie Access & Attribute Filtering | official_documentation | [MDN CookieStore.get](https://developer.mozilla.org/en-US/docs/Web/API/CookieStore/get) |
| 13 | `STANDARDS-COOKIESTORE-SET-METHOD` | security-and-data-protection | CookieStore.set(): Structured Asynchronous Cookie Authoring & Security Parameter Enforcement | official_documentation | [MDN CookieStore.set](https://developer.mozilla.org/en-US/docs/Web/API/CookieStore/set) |
| 14 | `STANDARDS-COOKIESTORE-DELETE-METHOD` | security-and-data-protection | CookieStore.delete(): Deterministic Client-Side Cookie Revocation & Sandbox Cleansing | official_documentation | [MDN CookieStore.delete](https://developer.mozilla.org/en-US/docs/Web/API/CookieStore/delete) |
| 15 | `STANDARDS-COOKIECHANGEEVENT-INTERFACE` | security-and-data-protection | CookieChangeEvent: Structured Payload Schema for Audit Logging & Security Forensics | official_documentation | [MDN CookieChangeEvent](https://developer.mozilla.org/en-US/docs/Web/API/CookieChangeEvent) |
