# Chuyên Đề 35: Quản Trị Cảm Biến Môi Trường, Input Độ Trễ Thấp, Điều Phối Viewport Và Clipboard Cho Mini-App Store (Iteration 39)

> **Căn cứ kỹ thuật & Tiêu chuẩn Quốc tế:**
> - **W3C Accelerometer & Gyroscope:** [https://w3c.github.io/accelerometer/](https://w3c.github.io/accelerometer/), [https://w3c.github.io/gyroscope/](https://w3c.github.io/gyroscope/)
> - **W3C Magnetometer & Orientation Sensor:** [https://w3c.github.io/magnetometer/](https://w3c.github.io/magnetometer/), [https://w3c.github.io/orientation-sensor/](https://w3c.github.io/orientation-sensor/)
> - **W3C Ambient Light Sensor:** [https://w3c.github.io/ambient-light/](https://w3c.github.io/ambient-light/)
> - **W3C Pointer Events Level 3 & Pointer Lock API:** [https://w3c.github.io/pointerevents/](https://w3c.github.io/pointerevents/), [https://w3c.github.io/pointerlock/](https://w3c.github.io/pointerlock/)
> - **W3C Input Events Level 2:** [https://w3c.github.io/input-events/](https://w3c.github.io/input-events/)
> - **W3C Gamepad API & Extensions:** [https://w3c.github.io/gamepad/](https://w3c.github.io/gamepad/)
> - **WICG Keyboard Map API:** [https://wicg.github.io/keyboard-map/](https://wicg.github.io/keyboard-map/)
> - **W3C IntersectionObserver & CSSWG Resize Observer:** [https://w3c.github.io/IntersectionObserver/](https://w3c.github.io/IntersectionObserver/), [https://drafts.csswg.org/resize-observer/](https://drafts.csswg.org/resize-observer/)
> - **W3C Clipboard API:** [https://w3c.github.io/clipboard-apis/](https://w3c.github.io/clipboard-apis/)
> - **W3C MediaStream Recording API:** [https://w3c.github.io/mediacapture-record/](https://w3c.github.io/mediacapture-record/)
> - **W3C Resource Timing Level 2:** [https://w3c.github.io/resource-timing/](https://w3c.github.io/resource-timing/)

---

## 1. Bối Cảnh Và Yêu Cầu Cấp Thiết

Trong hệ sinh thái Super-App hiện đại (Alibaba Cloud Superapp, WeChat Mini Program, Alipay, TikTok), mini-app không còn chỉ là các trang web biểu mẫu tĩnh mà mở rộng thành các ứng dụng tương tác cao:
1. **Trải nghiệm tương tác vật lý (Physical Motion & Environmental Sensing):** Ứng dụng AR, xem sản phẩm 3D, đo đạc mặt bằng, tự động chuyển giao diện theo ánh sáng phòng, và các tiện ích thể thao/sức khỏe. Tuy nhiên, việc cung cấp cảm biến tần số cao không kiểm soát tạo ra nguy cơ nghe lén bằng âm thanh (acoustic side-channel attacks), suy luận mã PIN qua rung động bàn phím và định vị trong nhà trái phép qua từ trường.
2. **Input độ trễ thấp và phần cứng điều khiển (Low-Latency Input & Controllers):** Ký số hợp đồng mượt mà, công cụ vẽ kỹ thuật số, giả lập 3D góc nhìn thứ nhất và trò chơi điều khiển bằng tay cầm Bluetooth. Điều này đòi hỏi container xử lý lấy mẫu toạ độ chính xác (coalesced events), ngoại suy nét vẽ (predicted events), khóa con trỏ (pointer lock), cách ly IME gõ tiếng Việt/châu Á và điều phối rung kép (dual-rumble haptic) an toàn.
3. **Theo dõi Viewport, Clipboard Và Hiệu Năng Subresource (Viewport Observation & Data Flow):** Đo lường lượt xem quảng cáo không gây nghẽn main thread (IntersectionObserver), điều chỉnh layout linh hoạt khi gập/mở màn hình (ResizeObserver), bảo vệ dữ liệu bộ nhớ tạm clipboard chống quét ngầm, ghi âm nén phân đoạn (MediaRecorder timeslice) tránh tràn RAM OOM, và kiểm toán tài nguyên mạng độc lập (Resource Timing).

---

## 2. Kiến Trúc Chi Tiết 3 Phân Hệ Kỹ Thuật (15 Kiểm Soát Chuẩn)

### Phân Hệ 1: Quản Trị Cảm Biến Vật Lý & Môi Trường (Physical Sensors Governance)

| Mã Kiểm Soát | Tiêu Chuẩn & API | Cơ Chế Kỹ Thuật Chi Tiết | Giải Pháp Bảo Mật & Phòng Thủ Rò Rỉ |
|---|---|---|---|
| `CTRL-SEN-01` | **W3C Accelerometer** | Cung cấp gia tốc 3 trục `LinearAccelerationSensor` (bỏ trọng lực) và `Accelerometer`. Container giới hạn tần số lấy mẫu (frequency cap) ở mức 20-30Hz. | Triệt tiêu nguy cơ keylogger gián tiếp và nghe lén âm học tần số cao (>100Hz); tự động ngắt kết nối cảm biến khi mất focus hoặc visibility chuyển `hidden`. |
| `CTRL-SEN-02` | **W3C Gyroscope** | Đo vận tốc góc 3 trục (rad/s). Container lượng tử hóa (rounding 3 chữ số thập phân) giá trị vận tốc góc. | Ngăn chặn việc kết hợp dữ liệu gia tốc và con quay hồi chuyển để tái tạo thao tác gõ bàn phím ảo hoặc mã PIN giao dịch. |
| `CTRL-SEN-03` | **W3C Magnetometer** | Đo cảm ứng từ trường địa phương ($\mu T$). Bơm nhiễu ngẫu nhiên Gaussian vào vector từ trường đối với các mini-app thông thường. | Vô hiệu hóa kỹ thuật lập bản đồ vân tay từ trường kết cấu thép tòa nhà để định vị người dùng trong nhà mà không xin quyền Location. |
| `CTRL-SEN-04` | **W3C Orientation Sensor** | Hợp nhất cảm biến (sensor fusion AHRS) xuất Quaternion 4 chiều `[x, y, z, w]` và ma trận xoay. Ưu tiên `RelativeOrientationSensor` không dùng từ trường. | Đảm bảo tính toán chuyển động xoay 3D mượt mà 30Hz đồng bộ `requestAnimationFrame` mà không làm lộ hướng la bàn tuyệt đối hay vị trí. |
| `CTRL-SEN-05` | **W3C Ambient Light Sensor** | Đo cường độ ánh sáng môi trường (LUX). Container phân đoạn giá trị LUX thành các bậc cố định (50-lux buckets) hoặc nhãn (`dim`, `bright`). | Chống tấn công đo ánh sáng phản xạ màn hình lên áo/tường để đọc lén nội dung nhạy cảm của các tab hoặc ứng dụng khác. |

### Phân Hệ 2: Input Độ Trễ Thấp, Điều Khiển Phần Cứng & IME (Low-Latency Input & Controllers)

| Mã Kiểm Soát | Tiêu Chuẩn & API | Cơ Chế Kỹ Thuật Chi Tiết | Giải Pháp Bảo Mật & Trải Nghiệm Chuẩn |
|---|---|---|---|
| `CTRL-INP-01` | **W3C Pointer Events Level 3** | Hợp nhất chuột, cảm ứng và bút stylus. `getCoalescedEvents()` trích xuất toạ độ phần cứng giữa các frame; `getPredictedEvents()` ngoại suy nét vẽ. | Đem lại trải nghiệm ký hợp đồng điện tử và vẽ kỹ thuật số không độ trễ; container tự động nhả pointer capture khi chạm vào vùng viền điều hướng hệ thống. |
| `CTRL-INP-02` | **W3C Pointer Lock API** | Khóa con trỏ vào canvas, cung cấp luồng toạ độ tương đối `movementX`/`movementY` không giới hạn màn hình. | Bắt buộc transient user activation; container giữ quyền ưu tiên phím thoát khẩn cấp (ESC hoặc cử chỉ back) không cho mini-app chặn hoặc bẫy người dùng. |
| `CTRL-INP-03` | **W3C Input Events Level 2** | Sự kiện `beforeinput` có thể hủy trước khi DOM biến đổi, chuẩn hóa hơn 30 `inputType` và phương thức `getTargetRanges()`. | Hỗ trợ bộ gõ tiếng Việt Telex/VNI và IME chữ tượng hình mượt mà; tự động chuyển sang bàn phím số bảo mật của super-app khi nhập PIN/mật mã. |
| `CTRL-INP-04` | **W3C Gamepad & Extensions** | Truy vấn `navigator.getGamepads()`, chuẩn hóa 16 nút và 4 trục analog; kích hoạt rung kép `GamepadHapticActuator.playEffect('dual-rumble')`. | Ẩn chuỗi nhận dạng phần cứng (USB vendor/product ID) để chống fingerprinting; chỉ cho phép đọc trạng thái gamepad khi tab đang focus và có tương tác người dùng. |
| `CTRL-INP-05` | **WICG Keyboard Map API** | `navigator.keyboard.getLayoutMap()` chuyển đổi mã phím vật lý (`KeyW`, `KeyA`) sang ký tự tương ứng trên bàn phím AZERTY/QWERTY. | Cho phép điều hướng trò chơi và phím tắt chuẩn mà không phụ thuộc layout bàn phím; khóa cứng danh sách tổ hợp phím hệ thống (Alt+F4, Back, Đóng app). |

### Phân Hệ 3: Theo Dõi Viewport, Clipboard Và Ghi Đo Hiệu Năng (Viewport, Clipboard & Metrics)

| Mã Kiểm Soát | Tiêu Chuẩn & API | Cơ Chế Kỹ Thuật Chi Tiết | Giải Pháp Bảo Mật & Tối Ưu Tài Nguyên |
|---|---|---|---|
| `CTRL-VCP-01` | **W3C Intersection Observer** | Tính toán tỷ lệ giao cắt của DOM phần tử với viewport bất đồng bộ tại luồng compositor, tránh cuộn giật (scroll jank). | Đánh giá chính xác chuẩn viewability quảng cáo (hiển thị $\ge 50\%$ trong 1 giây liên tục); cô lập root observer trong phạm vi WebView của mini-app. |
| `CTRL-VCP-02` | **CSSWG Resize Observer** | Giám sát kích thước hộp phần tử (`borderBoxSize`, `contentBoxSize`, `devicePixelContentBoxSize`) với độ chính xác sub-pixel. | Ngăn chặn mờ canvas trên màn hình Retina/High-DPI; thuật toán ngắt đệ quy vòng lặp layout bảo vệ container khỏi treo đơ giao diện khi gập/mở thiết bị. |
| `CTRL-VCP-03` | **W3C Clipboard API** | Đọc/ghi bất đồng bộ bộ nhớ tạm qua `navigator.clipboard`. Yêu cầu sự kiện click trực tiếp (transient user activation) và cấp quyền người dùng. | Hiển thị thông báo (toast) minh bạch khi mini-app đọc clipboard; khử nhiễm mã HTML độc hại; cấm tuyệt đối hành vi quét lén clipboard khi mở app. |
| `CTRL-VCP-04` | **W3C MediaStream Recording** | Nén âm thanh/video thời gian thực thành định dạng WebM/MP4 có phần cứng hỗ trợ. Sử dụng tham số `timeslice` (ví dụ 1000ms) để nhận dữ liệu chia nhỏ. | Khống chế trần bộ nhớ tạm trong RAM (tối đa 50MB) trước khi đẩy vào sandbox file stream, ngăn ngừa lỗi Out-Of-Memory (OOM); ngắt ghi âm ngay khi app blur. |
| `CTRL-VCP-05` | **W3C Resource Timing** | Bóc tách chi tiết từng giai đoạn nạp mạng của tài nguyên phụ qua `PerformanceResourceTiming` (DNS, TCP, TLS, TTFB, transferSize). | Yêu cầu tiêu đề `Timing-Allow-Origin` từ server; làm mờ đồng hồ `performance.now()` ở mức 5 micro-giây để vô hiệu hóa tấn công định thời bộ nhớ đệm (cache-timing attacks). |

---

## 3. Ma Trận Tích Hợp Vào Quy Trình Kiểm Duyệt Store (Store Review Pipeline)

Mọi mini-app gửi lên Store đều phải trải qua kiểm tra tự động tại các cổng kỹ thuật (Quality Gates):

1. **Cổng Kiểm Tra Cảm Biến (Sensors Gate):**
   - Mini-app khai báo quyền cảm biến trong `app.json` phải kèm mục đích sử dụng rõ ràng.
   - Các mini-app không thuộc danh mục Thể thao/AR/Bản đồ mà yêu cầu gia tốc hoặc con quay hồi chuyển sẽ bị gắn cờ kiểm toán thủ công.
   - Quét mã nguồn tĩnh (AST scan) cấm triệt để việc lặp vòng `setInterval` đọc cảm biến với chu kỳ $<30ms$.

2. **Cổng Kiểm Duyệt Input & Clipboard (Input & Data Leak Gate):**
   - Cấm các lệnh gọi `navigator.clipboard.readText()` trong hàm khởi tạo hoặc sự kiện `onShow`. Mọi lệnh đọc clipboard bắt buộc phải gắn với hành vi bấm nút của người dùng.
   - Quét mã nguồn ngăn chặn việc giả lập phím ảo hoặc chặn sự kiện bàn phím thoát hiểm của hệ thống.

3. **Cổng Đo Lường Viewport & Bộ Nhớ (Viewport & Memory Gate):**
   - Kiểm tra việc giải phóng listener: mọi đối tượng `IntersectionObserver`, `ResizeObserver`, `MediaRecorder` phải có mã lệnh `disconnect()` hoặc `stop()` khi page unmount.
   - Thử nghiệm tự động đo tải CPU khi chạy vòng lặp canvas với ResizeObserver đảm bảo không phát sinh layout loop cảnh báo đỏ.

---

## 4. Kết Luận & Lộ Trình Triển Khai

Việc chuẩn hóa 15 cơ chế điều phối phần cứng và viewport trong Iteration 39 hoàn thiện lớp giao tiếp phần cứng ngoại vi và hiển thị của Super-App. Toàn bộ 15 finding đã được kiểm tra tính xác thực nguồn (HTTP 200), không rò rỉ mã bí mật, và được tích lũy vào hệ thống tri thức chuẩn mực của dự án.
