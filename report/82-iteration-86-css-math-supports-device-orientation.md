# Chuyên đề 82 (Iteration 86): Chuẩn hoá W3C CSS Values and Units Level 4 Math Functions, CSS Conditional Rules @supports & W3C DeviceOrientation Sensor Sandboxing cho Mini App Store trên Super App

## 1. Bối cảnh & Mục tiêu Kỹ thuật

Trong kiến trúc container đa nền tảng của Super App (như Alibaba Cloud Superapp Solution / WindVane Mini App, WeChat Mini Program, Grab, Shopee), mini app phải vận hành ổn định và thích ứng linh hoạt trên hàng loạt biến thể phần cứng thiết bị di động: từ điện thoại giá rẻ với WebView cũ, màn hình gập (foldables), máy tính bảng, cho đến thiết bị POS chuyên dụng.

Để đảm bảo hiệu năng bố cục mượt mà, loại bỏ hiện tượng giật lag khung hình (*layout thrashing* / *forced reflow*), tương thích WebView tiến bộ (progressive enhancement) và kiểm soát an ninh phần cứng chặt chẽ, nền tảng Mini App Store chuẩn hoá 3 hệ thống tiêu chuẩn kỹ thuật cốt lõi:

1. **W3C CSS Values and Units Module Level 4 (Mathematical Expressions)**: Chuẩn hoá các hàm toán học CSS `calc()`, `min()`, `max()`, `clamp()` và các hàm bước nhảy (*stepped-value functions*). Cơ chế này chuyển toàn bộ tác vụ tính toán kích thước động, khoảng cách an toàn tai thỏ/capsule (`env(safe-area-inset-top)`), giới hạn kích thước tối đa trên màn hình gập (`min()`), và ngưỡng tối thiểu công thái học cảm ứng WCAG 2.2 AA (`max(48px, ...)`) về engine layout gốc của trình duyệt, loại bỏ hoàn toàn các script JavaScript lắng nghe `window.onresize` gây nghẽn main-thread.
2. **W3C CSS Conditional Rules Module Level 3 & CSSOM View Module**: Thiết lập cơ chế kiểm thử tính năng CSS cả dạng khai báo (`@supports`) lẫn lập trình (`CSS.supports()`), giúp mini app áp dụng các công nghệ giao diện hiện đại mà không làm đổ vỡ stylesheet trên các WebView phiên bản cũ. Kết hợp giao diện `window.matchMedia()` và vòng đời sự kiện `MediaQueryList` để đồng bộ trạng thái hệ thống (chế độ tối/sáng, độ tương phản cao, chuyển động rút gọn) theo mô hình hướng sự kiện (*event-driven*) không tốn chi phí layout reflow.
3. **W3C DeviceOrientation Event Specification & Hardware Sensor Sandboxing**: Thiết lập kiến trúc kiểm soát và sandbox các luồng cảm biến chuyển động vật lý (`DeviceOrientationEvent` đo góc quay 3 trục alpha/beta/gamma và `DeviceMotionEvent` đo gia tốc động học cô lập trọng lực). Container Super App áp dụng cơ chế ủy quyền quyền hạn chặt chẽ (Permission Policy / Secure Contexts), lượng tử hoá giá trị góc (làm tròn 0.5 độ) chống tấn công kênh bên suy đoán phím bấm (*side-channel keystroke inference*) và nhận dạng thiết bị sinh trắc học (*biometric fingerprinting*), giới hạn tần số lấy mẫu tối đa 60Hz và đình chỉ hoàn toàn cảm biến khi mini app chạy nền.

Toàn bộ 15 chuẩn kỹ thuật dưới đây được xác minh đối chiếu trực tiếp từ các đặc tả chính thức W3C, WHATWG và tài liệu MDN Web Docs (HTTP 200).

---

## 2. Danh mục Chuẩn Kỹ thuật & Bằng chứng Xác minh (15 Findings)

### 2.1. Nhóm 1: W3C CSS Values and Units Level 4 Mathematical Functions & Fluid Sizing

#### Finding 1: CSS-VALUES-MATH-FUNCTIONS-OVERVIEW
- **Tên chuẩn**: W3C CSS Values and Units Module Level 4 Mathematical Functions & Fluid Layout Sandboxing
- **Phân loại**: `layout-and-rendering-sandboxing`
- **Mức độ bằng chứng**: `official-specification` (W3C Working Draft)
- **URL xác thực**: `https://w3c.github.io/csswg-drafts/css-values-4/`
- **Nội dung & Yêu cầu chuẩn hoá**:
  - Đặc tả W3C CSS Values and Units Level 4 (Mục 10) thiết lập hệ thống cú pháp và quy tắc giải quyết kiểu thứ nguyên nghiêm ngặt cho các hàm toán học: `calc()`, `min()`, `max()`, `clamp()`, `round()`, `mod()`, `rem()`.
  - Phân tích và tính toán trực tiếp trong engine layout C++ của WebView, giải phóng hoàn toàn luồng JavaScript khỏi các tác vụ tính toán hình học.
  - Hỗ trợ kết hợp linh hoạt giữa các đơn vị tuyệt đối (`px`), tương đối (`rem`, `em`), đơn vị khung nhìn (`vw`, `vh`, `cqw`, `cqi`) và biến môi trường (`env()`).
  - **Quy tắc kiểm duyệt Mini App Store**: Công cụ kiểm tra tĩnh của Store yêu cầu các mini app sử dụng hàm toán học CSS để tính toán khoảng cách layout động thay vì dùng script JavaScript can thiệp trực tiếp vào `style` của DOM.

#### Finding 2: CSS-CALC-FUNCTION-DIMENSIONAL-COMPUTATION
- **Tên chuẩn**: CSS calc() Function & Mixed-Unit Dynamic Viewport Layouts
- **Phân loại**: `layout-and-rendering-sandboxing`
- **Mức độ bằng chứng**: `official-specification` (W3C CSS Values Level 4 / MDN)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/CSS/calc`
- **Nội dung & Yêu cầu chuẩn hoá**:
  - Cú pháp `calc(expression)` cho phép thực hiện 4 phép tính số học (+, -, *, /) kết hợp nhiều loại đơn vị khác nhau (ví dụ: `width: calc(100% - 32px)`).
  - Quy định cú pháp bắt buộc: phải có khoảng trắng phân tách rõ ràng xung quanh toán tử `+` và `-` để phân biệt với tiền tố dấu âm/dương.
  - Phép nhân yêu cầu ít nhất một toán hạng là `<number>`; phép chia yêu cầu số chia phải là `<number>` khác 0.
  - **Quy tắc kiểm duyệt Mini App Store**: Kiểm duyệt tự động cảnh báo các đoạn mã inline JavaScript tính toán khoảng cách lề và kích thước phần tử khi các giá trị này hoàn toàn có thể biểu diễn qua `calc()`.

#### Finding 3: CSS-MIN-FUNCTION-BOUNDARY-CONTAINMENT
- **Tên chuẩn**: CSS min() Function & Responsive Maximum Bound Containment
- **Phân loại**: `layout-and-rendering-sandboxing`
- **Mức độ bằng chứng**: `official-specification` (W3C CSS Values Level 4 / MDN)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/CSS/min`
- **Nội dung & Yêu cầu chuẩn hoá**:
  - Hàm `min(val1, val2, ...)` nhận danh sách các biểu thức và chọn giá trị nhỏ nhất, hoạt động như một ngưỡng chặn trên (upper-bound ceiling).
  - Ứng dụng phổ biến trong container Super App: thiết lập chiều rộng tối đa cho modal checkout, action-sheet và card sản phẩm (`width: min(100%, 480px)`) giúp giao diện tự động co giãn vừa vặn trên điện thoại nhỏ đồng thời không bị kéo giãn quá mức trên màn hình gập hoặc tablet.
  - Loại bỏ nhu cầu khai báo nhiều quy tắc `max-width` kèm `@media` rườm rà.
  - **Quy tắc kiểm duyệt Mini App Store**: Các component modal và checkout sheet bắt buộc phải khai báo giới hạn độ rộng trên thông qua `min()` hoặc `max-width` để vượt qua bài kiểm tra hiển thị trên thiết bị màn hình rộng.

#### Finding 4: CSS-MAX-FUNCTION-MINIMUM-LEGIBILITY-THRESHOLDS
- **Tên chuẩn**: CSS max() Function & Defensive Minimum Sizing Guardrails
- **Phân loại**: `layout-and-rendering-sandboxing`
- **Mức độ bằng chứng**: `official-specification` (W3C CSS Values Level 4 / MDN)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/CSS/max`
- **Nội dung & Yêu cầu chuẩn hoá**:
  - Hàm `max(val1, val2, ...)` nhận danh sách biểu thức và chọn giá trị lớn nhất, hoạt động như một ngưỡng chặn dưới (lower-bound floor).
  - Đóng vai trò then chốt trong việc tuân thủ tiêu chuẩn tiếp cận WCAG 2.2 Cấp độ AA: đảm bảo kích thước vùng cảm ứng của nút bấm và trường nhập liệu luôn đạt tối thiểu 48x48px (`min-height: max(48px, 6vh)`).
  - Ngăn ngừa hiện tượng văn bản hoặc nút bấm bị thu nhỏ tới mức không thể thao tác trên các màn hình có độ phân giải siêu cao hoặc tỉ lệ hiển thị hẹp.
  - **Quy tắc kiểm duyệt Mini App Store**: Bộ kiểm tra trợ năng tự động quét các thành phần tương tác chính để xác minh kích thước thực tế luôn duy trì `>= 48px` thông qua `max()`.

#### Finding 5: CSS-CLAMP-FUNCTION-FLUID-TYPOGRAPHY-ERGONOMICS
- **Tên chuẩn**: CSS clamp() Function & Bound-Constrained Fluid Typography Architecture
- **Phân loại**: `layout-and-rendering-sandboxing`
- **Mức độ bằng chứng**: `official-specification` (W3C CSS Values Level 4 / MDN)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/CSS/clamp`
- **Nội dung & Yêu cầu chuẩn hoá**:
  - Cú pháp `clamp(min, preferred, max)` là sự kết hợp toán học của `max(min, min(preferred, max))`.
  - Giúp giá trị thay đổi mượt mà theo kích thước khung nhìn (`font-size: clamp(1rem, 2.5vw, 1.5rem)`) mà không bao giờ vượt qua biên chặn dưới hoặc chặn trên đã định.
  - Triệt tiêu hoàn toàn hiện tượng vỡ giao diện hoặc chữ nhảy giật cục thường gặp khi chuyển đổi qua các điểm ngắt media query rời rạc.
  - **Quy tắc kiểm duyệt Mini App Store**: Bắt buộc các đơn vị trong tham số `min` và `max` của `clamp()` phải dùng đơn vị tương đối (`rem`/`em`) để đảm bảo tuân thủ thiết lập cỡ chữ trợ năng của hệ điều hành.

---

### 2.2. Nhóm 2: CSS Conditional Rules Module Level 3 & CSSOM View Module

#### Finding 6: CSS-CONDITIONAL-RULES-FEATURE-DETECTION-OVERVIEW
- **Tên chuẩn**: CSS Conditional Rules Module Level 3 & Declarative Feature Sandboxing
- **Phân loại**: `layout-and-rendering-sandboxing`
- **Mức độ bằng chứng**: `official-specification` (W3C Candidate Recommendation Draft)
- **URL xác thực**: `https://drafts.csswg.org/css-conditional-3/`
- **Nội dung & Yêu cầu chuẩn hoá**:
  - Đặc tả chuẩn hoá cấu trúc logic điều kiện `@supports` và `@media` trong CSS, hỗ trợ các toán tử logic `and`, `or`, và `not`.
  - Cơ chế kiểm thử an toàn: nếu trình duyệt gặp thuộc tính hoặc giá trị không được hỗ trợ trong khối điều kiện, toàn bộ khối đó sẽ được bỏ qua an toàn mà không làm lỗi cú pháp của toàn bộ file stylesheet.
  - Cho phép mini app áp dụng các tính năng giao diện thế hệ mới (CSS Grid Subgrid, Container Queries, View Transitions) với phương án fallback nguyên vẹn cho các máy Android cũ.
  - **Quy tắc kiểm duyệt Mini App Store**: Mọi thuộc tính CSS mới hoặc mang tính thử nghiệm trong mã nguồn mini app đều phải được bọc trong khối `@supports`.

#### Finding 7: CSS-AT-SUPPORTS-DECLARATIVE-FEATURE-QUERIES
- **Tên chuẩn**: CSS @supports At-Rule & Progressive Enhancement Sandboxing
- **Phân loại**: `layout-and-rendering-sandboxing`
- **Mức độ bằng chứng**: `official-specification` (W3C CSS Conditional Rules 3 / MDN)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/CSS/@supports`
- **Nội dung & Yêu cầu chuẩn hoá**:
  - Quy tắc `@supports (property: value)` kiểm tra trực tiếp khả năng hỗ trợ tính năng của engine CSS native.
  - Hỗ trợ kiểm tra selector cú pháp mới thông qua cú pháp `@supports selector(:has(a))`.
  - Ngăn ngừa hiện tượng phân mảnh phiên bản WebView trên hệ sinh thái Android (từ Android 8 đến Android 15), đảm bảo trải nghiệm người dùng nhất quán.
  - **Quy tắc kiểm duyệt Mini App Store**: Kiểm duyệt tự động xác minh các file CSS sử dụng hiệu ứng phức tạp (`backdrop-filter`) phải cung cấp màu nền tĩnh thay thế bên ngoài khối `@supports`.

#### Finding 8: CSS-SUPPORTS-PROGRAMMATIC-API-GATING
- **Tên chuẩn**: CSS.supports() Programmatic Interface & Dynamic Style Gating
- **Phân loại**: `layout-and-rendering-sandboxing`
- **Mức độ bằng chứng**: `official-specification` (W3C CSS Conditional Rules 3 / MDN)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/API/CSS/supports_static`
- **Nội dung & Yêu cầu chuẩn hoá**:
  - Phương thức tĩnh `CSS.supports(property, value)` hoặc `CSS.supports(conditionText)` cho phép JavaScript kiểm tra trực tiếp khả năng render của WebView trước khi khởi tạo component.
  - Giúp các thư viện UI mini app quyết định nên render bằng tính năng native hay nạp thêm polyfill mà không cần phân tích chuỗi `navigator.userAgent` vốn dễ sai lệch và thiếu tin cậy.
  - Thực thi tức thì, trả về boolean mà không gây ép buộc tính toán lại layout (*forced reflow*).
  - **Quy tắc kiểm duyệt Mini App Store**: Nghiêm cấm hành vi kiểm tra khả năng render CSS bằng cách phân tích chuỗi User-Agent; yêu cầu chuẩn hoá 100% qua `CSS.supports()`.

#### Finding 9: WINDOW-MATCHMEDIA-RESPONSIVE-QUERYING
- **Tên chuẩn**: Window.matchMedia() API & Programmatic Media Query Synchronization
- **Phân loại**: `layout-and-rendering-sandboxing`
- **Mức độ bằng chứng**: `official-specification` (CSSOM View Module / MDN)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/API/Window/matchMedia`
- **Nội dung & Yêu cầu chuẩn hoá**:
  - Phương thức `window.matchMedia(mediaQueryString)` đánh giá media query đồng bộ và trả về đối tượng `MediaQueryList`.
  - Cho phép Super App cầu nối và mini app đọc trạng thái thiết bị: chế độ sáng/tối (`(prefers-color-scheme: dark)`), kích thước màn hình, hoặc trạng thái xoay ngang/dọc mà không cần chèn node DOM hay đọc hình học layout.
  - Triệt tiêu hoàn toàn chi phí layout thrashing so với việc đọc liên tục thuộc tính `element.offsetWidth`.
  - **Quy tắc kiểm duyệt Mini App Store**: Các mini app phản hồi theo kích thước khung nhìn bắt buộc sử dụng `matchMedia()` thay vì lắng nghe sự kiện `resize` để đọc kích thước cửa sổ.

#### Finding 10: MEDIAQUERYLIST-EVENT-LIFECYCLE-MANAGEMENT
- **Tên chuẩn**: MediaQueryList Interface & Asynchronous Environmental Change Handling
- **Phân loại**: `layout-and-rendering-sandboxing`
- **Mức độ bằng chứng**: `official-specification` (CSSOM View Module / MDN)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/API/MediaQueryList`
- **Nội dung & Yêu cầu chuẩn hoá**:
  - Đăng ký lắng nghe sự kiện bất đồng bộ `mql.addEventListener('change', callback)` khi môi trường hiển thị thay đổi.
  - Cực kỳ quan trọng đối với các thiết bị màn hình gập (khi gập/mở máy), máy tính bảng xoay chiều hoặc chế độ chia màn hình (Split View / Samsung DeX).
  - Yêu cầu giải phóng tài nguyên: bắt buộc gọi `removeEventListener('change', handler)` khi component unmount để tránh rò rỉ bộ nhớ closure trong các ứng dụng đơn trang (SPA).
  - **Quy tắc kiểm duyệt Mini App Store**: Công cụ kiểm tra rò rỉ bộ nhớ sẽ đánh cờ cảnh báo đối với các instance `MediaQueryList` không được gỡ bỏ listener khi đóng mini app.

---

### 2.3. Nhóm 3: W3C DeviceOrientation Event Specification & Motion Governance

#### Finding 11: DEVICE-ORIENTATION-SPEC-FRAMEWORK
- **Tên chuẩn**: W3C DeviceOrientation Event Specification & Motion Sensor Sandboxing
- **Phân loại**: `hardware-and-sensor-governance`
- **Mức độ bằng chứng**: `official-specification` (W3C Working Draft)
- **URL xác thực**: `https://w3c.github.io/deviceorientation/`
- **Nội dung & Yêu cầu chuẩn hoá**:
  - Đặc tả W3C định nghĩa hai sự kiện cốt lõi: `DeviceOrientationEvent` (đo hướng xoay góc vật lý) và `DeviceMotionEvent` (đo gia tốc động học).
  - Nguy cơ an ninh: luồng dữ liệu cảm biến tần số cao có thể bị phần mềm độc hại khai thác để tái tạo thao tác gõ phím (mã PIN/mật khẩu) hoặc nhận dạng thói quen người dùng (*keystroke inference attack*).
  - Yêu cầu bắt buộc: chỉ hoạt động trong môi trường bảo mật (HTTPS/Secure Contexts) và phải có sự đồng ý tường minh từ người dùng (User Gesture Permission trên iOS 13+ và Android hiện đại).
  - **Quy tắc kiểm duyệt Mini App Store**: Mini app yêu cầu quyền truy cập chuyển động hoặc hướng thiết bị phải khai báo lý do nghiệp vụ rõ ràng trong file cấu hình `app.json`.

#### Finding 12: DEVICE-ORIENTATION-EVENT-ANGULAR-COORDINATES
- **Tên chuẩn**: DeviceOrientationEvent Interface & 3-Axis Angular Orientation Governance
- **Phân loại**: `hardware-and-sensor-governance`
- **Mức độ bằng chứng**: `official-specification` (W3C DeviceOrientation / MDN)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/API/DeviceOrientationEvent`
- **Nội dung & Yêu cầu chuẩn hoá**:
  - Thuộc tính sự kiện cung cấp góc xoay 3 trục theo hệ toạ độ Descartes bàn tay phải: `alpha` (0 đến 360 độ quanh trục Z), `beta` (-180 đến 180 độ quanh trục X), `gamma` (-90 đến 90 độ quanh trục Y), và cờ `absolute`.
  - Cơ chế phòng vệ kênh bên của Super App: runtime tự động lượng tử hoá (quantize) các giá trị góc quay (làm tròn về 0.5 độ gần nhất) để vô hiệu hoá các thuật toán nhận dạng sinh trắc học vi mô.
  - Ngay lập tức ngắt phân phối sự kiện khi `document.visibilityState` chuyển sang `'hidden'` nhằm bảo vệ quyền riêng tư và tiết kiệm pin.
  - **Quy tắc kiểm duyệt Mini App Store**: Mini app không được duy trì listener `deviceorientation` khi đang chạy ngầm hoặc trong các iframe không nhìn thấy.

#### Finding 13: DEVICE-MOTION-EVENT-KINEMATIC-MEASUREMENT
- **Tên chuẩn**: DeviceMotionEvent Interface & Real-Time Kinematic Acceleration Telemetry
- **Phân loại**: `hardware-and-sensor-governance`
- **Mức độ bằng chứng**: `official-specification` (W3C DeviceOrientation / MDN)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/API/DeviceMotionEvent`
- **Nội dung & Yêu cầu chuẩn hoá**:
  - Cung cấp dữ liệu gia tốc chuyển động thời gian thực (`acceleration`, `accelerationIncludingGravity`), tốc độ xoay góc (`rotationRate`) và chu kỳ lấy mẫu (`interval`).
  - Hỗ trợ các tính năng tương tác như lắc điện thoại làm mới/nhận quà (*shake gesture*), đếm bước chân và điều khiển game nghiêng máy.
  - Khung bảo mật Super App giới hạn tần số lấy mẫu tối đa ở mức 60Hz nhằm ngăn ngừa tình trạng CPU wake-up liên tục gây nóng máy và chai pin.
  - **Quy tắc kiểm duyệt Mini App Store**: Hệ thống kiểm duyệt kiểm tra khả năng xử lý an toàn của mini app khi người dùng từ chối cấp quyền cảm biến chuyển động (không được văng lỗi fatal).

#### Finding 14: DEVICE-MOTION-ACCELERATION-VECTORS
- **Tên chuẩn**: DeviceMotionEvent.acceleration & Linear Force Vector Isolation
- **Phân loại**: `hardware-and-sensor-governance`
- **Mức độ bằng chứng**: `official-specification` (W3C DeviceOrientation / MDN)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/API/DeviceMotionEvent/acceleration`
- **Nội dung & Yêu cầu chuẩn hoá**:
  - Thuộc tính `acceleration` (gồm 3 trục x, y, z tính bằng m/s²) tách biệt hoàn toàn lực tác động của người dùng khỏi hằng số trọng lực 9.81 m/s² của Trái Đất nhờ thuật toán sensor fusion phần cứng.
  - Trả về `null` nếu phần cứng thiết bị không có cảm biến con quay hồi chuyển (gyroscope) để bù trừ trọng lực.
  - Ngưỡng nhận diện lắc máy chuẩn hoá của Super App: biên độ vector tổng hợp $\sqrt{x^2 + y^2 + z^2} > 15 \text{ m/s}^2$ duy trì qua 3 chu kỳ liên tiếp.
  - **Quy tắc kiểm duyệt Mini App Store**: Bắt buộc mọi mini app có tính năng lắc máy phải cung cấp nút bấm cảm ứng thay thế trên màn hình để phục vụ người dùng hạn chế vận động (WCAG 2.2 2.5.4).

#### Finding 15: DEVICE-MOTION-ROTATION-RATE-ANGULAR-VELOCITY
- **Tên chuẩn**: DeviceMotionEvent.rotationRate & Angular Velocity Telemetry Sandboxing
- **Phân loại**: `hardware-and-sensor-governance`
- **Mức độ bằng chứng**: `official-specification` (W3C DeviceOrientation / MDN)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/API/DeviceMotionEvent/rotationRate`
- **Nội dung & Yêu cầu chuẩn hoá**:
  - Đo tốc độ xoay góc quanh 3 trục: `alpha` (quanh trục vuông góc màn hình), `beta` (quanh trục ngang), `gamma` (quanh trục dọc) tính bằng độ/giây (deg/s).
  - Cầu nối container Super App áp dụng bộ lọc thông thấp (low-pass filter) khử nhiễu phần cứng và giới hạn tốc độ tối đa có thể báo cáo ở mức 720 deg/s.
  - Khung sandbox cô lập origin: nghiêm cấm truyền luồng `rotationRate` sang các SDK quảng cáo bên thứ ba nhằm ngăn chặn theo dõi hành vi cầm nắm thiết bị của người dùng.
  - **Quy tắc kiểm duyệt Mini App Store**: Các ứng dụng game mini sử dụng con quay hồi chuyển điều khiển góc nhìn bắt buộc phải cung cấp thanh điều khiển ảo dự phòng.

---

## 3. Ma trận Ánh xạ Khung Tiêu chuẩn & Tác động Vận hành

| Nhóm Tiêu Chuẩn | Cấp Độ Áp Dụng | Rủi Ro Kỹ Thuật Khi Vi Phạm | Kiểm Duyệt Store & Rào Cản Tự Động |
|---|---|---|---|
| **W3C CSS Math Functions (calc, min, max, clamp)** | **BẮT BUỘC (MANDATORY)** | Bùng nổ layout reflow do dùng JS resize listener; vỡ giao diện trên màn hình gập; vi phạm diện tích cảm ứng tối thiểu WCAG 2.2 | Linter tĩnh quét mã nguồn, cảnh báo các vòng lặp tính layout trong `window.onresize`, kiểm tra ngưỡng touch target `>= 48px`. |
| **CSS Conditional Rules (@supports, matchMedia)** | **BẮT BUỘC (MANDATORY)** | Lỗi sập stylesheet trên Android WebView cũ; layout bị phá vỡ khi chuyển đổi đa cửa sổ/màn hình gập; rò rỉ bộ nhớ listener | Kiểm tra tính hiện diện của quy tắc fallback ngoài `@supports`, quét bộ nhớ phát hiện orphaned MediaQueryList listeners. |
| **W3C DeviceOrientation & Motion Sandboxing** | **QUY ĐỊNH CHẶT CHẼ (RESTRICTED)** | Tấn công nghe trộm mã PIN qua rung chấn (keystroke eavesdropping); cạn kiệt pin và quá nhiệt thiết bị do polling liên tục | Yêu cầu khai báo quyền trong `app.json`, tự động ngắt cảm biến khi chạy nền, ép buộc cung cấp nút bấm thay thế thao tác lắc (WCAG 2.5.4). |

---

## 4. Khuyến nghị Tích hợp Kiến trúc Super App (Container & SDK)

1. **Khung Sizing Động & Biến Môi Trường (Safe Area Tokenization)**:
   - Super App Container SDK cần cung cấp sẵn bộ biến CSS Custom Properties toàn cục được định dạng sẵn qua các hàm toán học:
     ```css
     :root {
       --superapp-header-height: calc(44px + env(safe-area-inset-top, 0px));
       --superapp-sheet-max-width: min(100vw - 32px, 540px);
       --superapp-touch-target: max(48px, 6vh);
     }
     ```
   - Hướng dẫn mini app kế thừa các biến này nhằm giảm thiểu 95% mã CSS định vị phức tạp.

2. **Cơ chế Fallback Tính Năng Khai Báo (Declarative Feature Degradation)**:
   - Khuyến khích lập trình viên áp dụng triết lý *Progressive Enhancement*:
     ```css
     .mini-app-card {
       /* Fallback cho WebView cũ */
       padding: 16px;
       border-radius: 8px;
     }
     @supports (backdrop-filter: blur(10px)) {
       .mini-app-card {
         background: rgba(255, 255, 255, 0.7);
         backdrop-filter: blur(10px);
       }
     }
     ```

3. **Cổng Kiểm Soát Cảm Biến Phần Cứng (Host Sensor Gateway & Quota)**:
   - Container WebView phải đóng gói API cảm biến qua cầu nối bảo mật:
     - Lắng nghe sự kiện `visibilitychange`, khi sang `hidden` lập tức tự động gỡ bỏ hook phần cứng native.
     - Lượng tử hoá toạ độ góc và gia tốc, thêm nhiễu vi mô (*micro-jitter*) để bảo vệ dấu vết riêng tư của người dùng.
     - Cưỡng chế tần số lấy mẫu cảm biến không vượt quá 60Hz.
