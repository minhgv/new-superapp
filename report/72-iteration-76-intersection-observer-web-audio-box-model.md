# Chuyên đề 72: W3C Intersection Observer API Level 2, W3C Web Audio API 1.1 & W3C CSS Box Model Module Level 3/4

## 1. Tổng quan nghiên cứu Milestone 76

Milestone 76 tập trung hoàn thiện 3 trụ cột công nghệ nền tảng quyết định khả năng hiển thị bất đồng bộ tối ưu tài nguyên, quản trị âm thanh hộp cát đa luồng và chuẩn hóa bao đóng hình học bố cục trong kiến trúc siêu ứng dụng (Super-App Mini-App Store Standard):

1. **W3C Intersection Observer API Level 2 & Asynchronous Visibility Pipeline**:
   - Chuẩn hóa cơ chế quan sát bất đồng bộ (`IntersectionObserver`) sự giao cắt giữa phần tử DOM mục tiêu với tổ tiên hoặc khung nhìn (viewport), tách rời khỏi luồng sự kiện cuộn chính để triệt tiêu hiện tượng giật khung hình (scroll jank/layout thrashing) và bảo toàn chỉ số phản hồi INP < 200ms.
   - Ứng dụng cấu hình vùng đệm `rootMargin` (ví dụ `200px 0px`) cho phép nạp trước dữ liệu và hình ảnh lười (lazy loading proactive pre-fetch) trước khi phần tử lướt vào tầm nhìn người dùng.
   - Thiết lập mảng ngưỡng giao cắt `threshold` (`[0.0, 0.5, 1.0]`) đo lường độ hiển thị thực tế (viewability telemetry) có thể kiểm toán độc lập cho các thành phần quảng cáo và widget tài trợ.
   - Bóc tách hình học giao cắt thời gian thực (`IntersectionObserverEntry`) bao gồm `boundingClientRect`, `intersectionRect`, `intersectionRatio` và `isIntersecting`, kích hoạt cơ chế tự động tạm dừng hoạt họa Canvas 2D/WebGL và ngắt kết nối WebSocket polling khi phần tử trượt ra khỏi màn hình.
   - Tích hợp phòng thủ chống tấn công Clickjacking và tráo đổi giao diện (UI redressing) thông qua cờ `isVisible` và tùy chọn `trackVisibility` của Intersection Observer v2, bảo vệ nút thanh toán và xác thực giao dịch tài chính.
   - Quản trị vòng đời giải phóng tài nguyên thông qua `unobserve()` và `disconnect()`, loại bỏ triệt để nguy cơ rò rỉ bộ nhớ DOM cô lập (detached DOM nodes) khi chuyển trang.

2. **W3C Web Audio API 1.1 & Sandboxed Audio Graph Governance**:
   - Đặc tả kiến trúc đồ thị định tuyến âm thanh mô-đun (`BaseAudioContext` và `AudioContext`), phân tách luồng âm thanh thành các khối xử lý rời rạc (`AudioNode`) như bộ khuếch đại `GainNode`, bộ lọc tần số `BiquadFilterNode` và bộ làm trễ `DelayNode`.
   - Thực thi nghiêm ngặt máy trạng thái vòng đời âm thanh (`suspended`, `running`, `closed`) và chính sách Autoplay: bắt buộc mọi lệnh `AudioContext.resume()` phải xuất phát từ cử chỉ tương tác chủ động có xác thực (`transient user activation`), triệt tiêu âm thanh tự phát gây khó chịu.
   - Áp đặt hạn mức tài nguyên (quota control) tối đa 64 thực thể `AudioNode` đồng thời cho mỗi mini-app khách, ngăn chặn tình trạng cạn kiệt luồng DSP và nghẽn CPU hệ điều hành.
   - Tích hợp chặt chẽ với W3C Page Visibility Level 2: tự động gọi `AudioContext.suspend()` khi mini-app chuyển sang trạng thái `hidden`, giải phóng khóa chiếm dụng âm thanh (audio ducking/focus) và tiết kiệm pin di động.
   - Cơ chế thu hồi tài nguyên tất định với `AudioContext.close()`, bảo đảm toàn bộ luồng xử lý âm thanh C++ tầng native được đóng hoàn toàn khi đóng hoặc chuyển đổi mini-app.

3. **W3C CSS Box Model Module Level 3/4 & Multi-Tenant Layout Containment**:
   - Chuẩn hóa mô hình hình học 4 cạnh đồng tâm (`content`, `padding`, `border`, `margin`) của W3C CSS Box Model, bảo đảm tính nhất quán hiển thị trên đa kích thước màn hình và hệ điều hành di động.
   - Áp đặt chuẩn thiết lập toàn cục `box-sizing: border-box` làm baseline bắt buộc cho toàn bộ framework container và mẫu mini-app, ngăn chặn việc thay đổi padding hoặc viền làm biến dạng kích thước ngoài gây ra dịch chuyển bố cục tích lũy (Cumulative Layout Shift - CLS).
   - Cô lập hiện tượng sụp lề dọc (margin collapsing) bằng cách thiết lập ngữ cảnh định dạng khối độc lập (Block Formatting Context - BFC) hoặc Flexbox/Grid, loại bỏ nguy cơ lề của mini-app khách đâm xuyên làm lệch thanh điều hướng header của super-app cha.
   - Căn chỉnh mô hình hộp với CSS Box Model Level 4 và các thuộc tính logic CSS (`inline-size`, `block-size`, `margin-inline`, `padding-block`), hỗ trợ tối ưu giao diện đa ngôn ngữ và các hệ chữ từ phải sang trái (RTL) như tiếng Ả Rập.
   - Cơ chế cắt xén vùng tràn (`overflow-x: clip`, `overflow: auto`) bảo đảm không một phần tử con nào có thể tràn ra ngoài ranh giới cửa sổ được chỉ định hoặc che khuất các biểu tượng bảo mật của ứng dụng cha.

---

## 2. Bảng tổng hợp các tiêu chuẩn & Findings nghiên cứu (15 Quy tắc chuẩn hóa)

| Mã ID | Trụ cột công nghệ | Tiêu chuẩn tham chiếu | Phân loại | Tóm tắt quy định kiến trúc Super-App | URL nguồn xác thực |
|---|---|---|---|---|---|
| **FINDING-1077** | Intersection Observer | W3C TR intersection-observer | Normative | Thay thế toàn bộ trình lắng nghe cuộn `onscroll` đồng bộ bằng `IntersectionObserver` bất đồng bộ để bảo đảm danh sách ứng dụng và bảng tin cuộn mượt mà 60/120fps với INP < 200ms. | [W3C Intersection Observer](https://www.w3.org/TR/intersection-observer/) |
| **FINDING-1078** | Intersection Observer | MDN Intersection Observer API | Normative | Thiết lập vùng đệm `rootMargin: '200px 0px'` để nạp lười tài nguyên trước khi chạm màn hình, kết hợp mảng ngưỡng `threshold: [0.0, 0.5, 1.0]` đo lường viewability quảng cáo độc lập. | [MDN Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API) |
| **FINDING-1079** | Intersection Observer | MDN IntersectionObserverEntry | Normative | Container bóc tách `isIntersecting` và `intersectionRatio` để tự động đóng băng hoạt họa Canvas/WebGL và ngắt polling mạng khi thành phần mini-app cuộn ra ngoài màn hình. | [MDN IntersectionObserverEntry](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry) |
| **FINDING-1080** | Intersection Observer | MDN IntersectionObserver | Normative | Thực thi cơ chế phát hiện che khuất `isVisible` của Intersection Observer v2 để ngăn chặn tấn công Clickjacking và UI redressing trên các nút thanh toán tài chính. | [MDN IntersectionObserver](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserver) |
| **FINDING-1081** | Intersection Observer | MDN IntersectionObserver | Normative | Vòng đời mini-app bắt buộc gọi `unobserve()` và `disconnect()` khi hủy phần tử DOM để loại bỏ triệt để tình trạng rò rỉ bộ nhớ detached DOM nodes. | [MDN IntersectionObserver](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserver) |
| **FINDING-1082** | Web Audio API | W3C TR webaudio-1.1 | Normative | Khởi tạo âm thanh qua đồ thị `AudioContext` kết nối với `GainNode` tổng của host container, cho phép ứng dụng cha điều phối âm lượng tổng thể và quyền ưu tiên âm thanh hệ thống. | [W3C Web Audio API 1.1](https://www.w3.org/TR/webaudio-1.1/) |
| **FINDING-1083** | Web Audio API | MDN AudioContext | Normative | Bắt buộc `AudioContext.resume()` phải diễn ra trong ngữ cảnh cử chỉ người dùng chủ động (`transient user activation`), cấm hoàn toàn âm thanh tự động phát khi chưa tương tác. | [MDN AudioContext](https://developer.mozilla.org/en-US/docs/Web/API/AudioContext) |
| **FINDING-1084** | Web Audio API | MDN AudioNode | Normative | Giới hạn tối đa 64 nút `AudioNode` đồng thời cho mỗi ngữ cảnh mini-app khách để bảo vệ hạn mức CPU và luồng xử lý DSP âm thanh của thiết bị di động. | [MDN AudioNode](https://developer.mozilla.org/en-US/docs/Web/API/AudioNode) |
| **FINDING-1085** | Web Audio API | MDN Web Audio API | Normative | Tự động kích hoạt `AudioContext.suspend()` khi mini-app chuyển sang nền ẩn (`visibilityState: hidden`), giải phóng kênh phần cứng và tiết kiệm năng lượng pin. | [MDN Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API) |
| **FINDING-1086** | Web Audio API | MDN AudioContext | Normative | Bắt buộc gọi `audioContext.close()` trong hook dọn dẹp khi đóng mini-app để giải phóng dứt điểm các luồng C++ xử lý âm thanh tầng native. | [MDN AudioContext](https://developer.mozilla.org/en-US/docs/Web/API/AudioContext) |
| **FINDING-1087** | CSS Box Model | W3C TR css-box-3 | Normative | Chuẩn hóa mô hình hình học 4 cạnh (`content`, `padding`, `border`, `margin`) làm ranh giới cấu trúc giao diện, chống tràn bố cục và bảo đảm độ chính xác khi chạm màn hình. | [W3C CSS Box Model L3](https://www.w3.org/TR/css-box-3/) |
| **FINDING-1088** | CSS Box Model | MDN box-sizing | Normative | Thiết lập baseline `box-sizing: border-box` toàn cục cho container, bảo đảm padding và border không làm thay đổi kích thước tổng thể gây hiện tượng CLS. | [MDN box-sizing](https://developer.mozilla.org/en-US/docs/Web/CSS/box-sizing) |
| **FINDING-1089** | CSS Box Model | MDN CSS Box Model Basics | Normative | Ngăn chặn hiện tượng sụp lề dọc (`margin collapsing`) giữa mini-app và header siêu ứng dụng bằng cách thiết lập ranh giới BFC hoặc Flexbox/Grid độc lập. | [MDN CSS Box Model Basics](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Styling_basics/Box_model) |
| **FINDING-1090** | CSS Box Model | W3C TR css-box-4 | Normative | Căn chỉnh bố cục theo các thuộc tính logic CSS (`inline-size`, `block-size`, `margin-inline`) hỗ trợ bản địa hóa giao diện tự động cho các ngôn ngữ RTL. | [W3C CSS Box Model L4](https://www.w3.org/TR/css-box-4/) |
| **FINDING-1091** | CSS Box Model | MDN Introduction to Box Model | Normative | Áp đặt bao đóng cắt xén tràn (`overflow-x: clip`, `overflow: auto`) tại phần tử gốc mini-app, ngăn chặn việc che khuất các biểu tượng huy hiệu bảo mật của ứng dụng cha. | [MDN Introduction to Box Model](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_box_model/Introduction_to_the_CSS_box_model) |

---

## 3. Kiến trúc Triển khai & Đặc tả Kỹ thuật chi tiết

### 3.1. Kiến trúc Quan sát Bất đồng bộ và Bảo vệ Chống Clickjacking

```
+-----------------------------------------------------------------------------------+
|                        SUPER-APP VIEWPORT CONTAINER                               |
|                                                                                   |
|   +---------------------------------------------------------------------------+   |
|   | Host Header Capsule (BFC Boundary / z-index: 1000)                        |   |
|   +---------------------------------------------------------------------------+   |
|                                                                                   |
|   [Active Scrollport]                                                             |
|   +---------------------------------------------------------------------------+   |
|   | Mini-App Catalog Banner                                                   |   |
|   | isIntersecting: true, intersectionRatio: 1.0                              |   |
|   | v2 isVisible: true --> Verified safe payment / action button              |   |
|   +---------------------------------------------------------------------------+   |
|                                                                                   |
|   + - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +   |
|   | Lazy Loaded Mini-App Card (In rootMargin buffer: +200px)                  |   |
|   | isIntersecting: true, intersectionRatio: 0.05                             |   |
|   | Trigger background fetch & pre-decode image resources                      |   |
|   + - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +   |
|                                                                                   |
|   [Off-screen Region]                                                             |
|   +---------------------------------------------------------------------------+   |
|   | Heavy Mini-App Game Canvas / WebGL Banner                                 |   |
|   | isIntersecting: false, intersectionRatio: 0.0                             |   |
|   | --> Action: Cancel requestAnimationFrame, pause render loops & polling    |   |
|   +---------------------------------------------------------------------------+   |
+-----------------------------------------------------------------------------------+
```

### 3.2. Đồ thị Quản trị Âm thanh Hộp cát (Sandboxed Audio Graph)

```
+-----------------------------------------------------------------------------------+
|                        GUEST MINI-APP AUDIOCONTEXT                                |
|                                                                                   |
|   +-----------------------+     +-----------------------+                         |
|   | Oscillator / Buffer   | --> | BiquadFilterNode      | \                       |
|   | Source Node           |     | (Frequency shaping)   |  \                      |
|   +-----------------------+     +-----------------------+   \                     |
|                                                              +--> +-----------+   |
|   +-----------------------+                                       | App Gain  |   |
|   | MediaElement Source   | ------------------------------------> | Node      |   |
|   +-----------------------+                                       +-----+-----+   |
+-------------------------------------------------------------------------|---------+
                                                                          | connect()
                                                                          v
+-----------------------------------------------------------------------------------+
|                        SUPER-APP HOST AUDIO CONTROLLER                            |
|                                                                                   |
|   +-----------------------+     +-----------------------+     +---------------+   |
|   | Master Container Gain | --> | Audio Focus Mediation | --> | Hardware      |   |
|   | (Mute / Ducking)      |     | (Call / Push override)|     | Speakers / DAC|   |
|   +-----------------------+     +-----------------------+     +---------------+   |
+-----------------------------------------------------------------------------------+
```

---

## 4. Khuyến nghị Thực thi cho Siêu ứng dụng Viettel / Alibaba Cloud Superapp

1. **Mandatory IntersectionObserver Adoption in Developer Guidelines**:
   - Yêu cầu mọi mini-app trong tài liệu SDK phải thay thế toàn bộ logic cuộn vô tận (infinite list) và lazy loading bằng `IntersectionObserver`.
   - Cổng kiểm thử tĩnh tự động quét mã nguồn (static analysis scan) gắn cờ cảnh báo nếu phát hiện việc đăng ký sự kiện `window.addEventListener('scroll', ...)` mà không áp dụng debounce/throttle thích hợp.

2. **Host-Enforced Audio Sandboxing**:
   - Container WebView tự động inject lớp bọc (wrapper) cho `window.AudioContext`.
   - Tự động gắn hook lắng nghe `visibilitychange`: nếu trang chuyển sang `hidden`, tự động đình chỉ mọi audio context đang chạy để tránh việc mini-app phát âm thanh ngầm tiêu hao pin hoặc gây phiền toái cho người dùng.

3. **Global CSS Reset & Layout Boundary Standard**:
   - Bộ khung phát triển mini-app (Starter Kit / CLI) bắt buộc đính kèm tệp cấu hình CSS reset chuẩn với `*, *::before, *::after { box-sizing: border-box; }`.
   - Vỏ bọc WebView của container cha áp dụng `display: flex; flex-direction: column; overflow-x: clip;` để cô lập hoàn toàn lề dọc và ngăn chặn tràn ngang giao diện.
