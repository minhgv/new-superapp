# Báo Cáo Chuyên Đề: WebRTC Statistics Telemetry, Navigation Preload Boot Concurrency & Page Visibility / Screen Orientation Sandboxing (Milestone 120)

## 1. Giới Thiệu & Bối Cảnh Nghiên Cứu
Trong kiến trúc Super App và Mini App Runtime quy mô lớn, việc kiểm soát hiệu năng khởi động (cold-start latency), khả năng quan sát đo lường chất lượng âm thanh/hình ảnh thời gian thực (real-time telemetry) và quản trị vòng đời hiển thị/xoay màn hình thiết bị (lifecycle & display sandboxing) là ba yếu tố quyết định chất lượng trải nghiệm (QoE) và độ tin cậy của nền tảng.

Milestone 120 mở rộng chuẩn hóa kỹ thuật với 15 phát hiện chuẩn hóa từ W3C và MDN, bao quát 3 lĩnh vực then chốt:
1. **W3C WebRTC Statistics API & MDN Telemetry Introspection (`RTCStatsReport`, `RTCPeerConnection.getStats`, `RTCInboundRtpStreamStats`, `RTCOutboundRtpStreamStats`)**: Chuẩn hóa cơ chế trích xuất dữ liệu đo lường chất lượng luồng truyền thời gian thực (packet loss, jitter buffer delay, frame decoding/encoding latency) một cách bất đồng bộ và bất biến, phục vụ giám sát QoS và cảnh báo suy giảm mạng mà không gây nghẽn main UI thread.
2. **W3C Service Workers & MDN Navigation Preload Manager (`NavigationPreloadManager`, `Service-Worker-Navigation-Preload`, `FetchEvent.preloadResponse`)**: Triệt tiêu độ trễ khởi động lạnh (cold-start delay) thông qua cơ chế song song hóa: trình duyệt kích hoạt tải mạng song song ngay khi Service Worker đang khởi động luồng nền, cho phép ứng dụng mini app đạt tốc độ điều hướng cận tức thì (<100ms) tương đương ứng dụng native.
3. **W3C Page Visibility Level 2 & Screen Orientation API Sandboxing (`Document.visibilityState`, `visibilitychange`, `ScreenOrientation.lock`)**: Thiết lập cơ chế chuẩn hóa điều tiết tiêu thụ điện năng/CPU khi ứng dụng chuyển xuống chạy nền (background throttling), bảo đảm giải phóng phần cứng nhạy cảm (camera, mic) và lưu trữ trạng thái giao dịch kịp thời; đồng thời quản trị quyền khóa hướng xoay màn hình (orientation locking) chống tình trạng mini app phá vỡ layout điều hướng của Super App host.

---

## 2. Danh Mục Các Findings Chuẩn Hóa Được Thu Thập Trong Vòng 120

| ID | Danh Mục | Tiêu Đề Chuẩn Hóa | Mức Bằng Chứng | URL Nguồn Chuẩn |
|---|---|---|---|---|
| `STANDARDS-W3C-WEBRTC-STATS-SPEC` | Real-Time Networking & Transport | W3C Identifiers for WebRTC's Statistics API: Real-Time Telemetry & Quality Architecture | `official_standard` | [W3C WebRTC Stats](https://www.w3.org/TR/webrtc-stats/) |
| `STANDARDS-MDN-RTCSTATSREPORT-MAP` | Performance & Observability | MDN RTCStatsReport Interface: Read-Only Map-Like Telemetry Introspection | `official_documentation` | [MDN RTCStatsReport](https://developer.mozilla.org/en-US/docs/Web/API/RTCStatsReport) |
| `STANDARDS-MDN-RTCPEERCONNECTION-GETSTATS` | Real-Time Networking & Transport | MDN RTCPeerConnection.getStats(): Asynchronous Telemetry Extraction & Track Filtering | `official_documentation` | [MDN RTCPeerConnection.getStats](https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection/getStats) |
| `STANDARDS-MDN-RTCINBOUNDRTPSTREAMSTATS` | Performance & Observability | MDN RTCInboundRtpStreamStats: Inbound Media Quality & Packet Degradation Telemetry | `official_documentation` | [MDN RTCInboundRtpStreamStats](https://developer.mozilla.org/en-US/docs/Web/API/RTCInboundRtpStreamStats) |
| `STANDARDS-MDN-RTCOUTBOUNDRTPSTREAMSTATS` | Performance & Observability | MDN RTCOutboundRtpStreamStats: Outbound Transmission & Encoder Performance Sandboxing | `official_documentation` | [MDN RTCOutboundRtpStreamStats](https://developer.mozilla.org/en-US/docs/Web/API/RTCOutboundRtpStreamStats) |
| `STANDARDS-W3C-SERVICE-WORKERS-NAVIGATION-PRELOAD` | Offline Resilience & Caching | W3C Service Workers: Navigation Preload & Parallel Boot Network Pipeline | `official_standard` | [W3C Service Workers](https://www.w3.org/TR/service-workers/) |
| `STANDARDS-MDN-NAVIGATION-PRELOAD-MANAGER` | Platform Architecture & Mini-App Runtime | MDN NavigationPreloadManager Interface: Cold-Start Preload State & Configuration | `official_documentation` | [MDN NavigationPreloadManager](https://developer.mozilla.org/en-US/docs/Web/API/NavigationPreloadManager) |
| `STANDARDS-MDN-NAV-PRELOAD-ENABLE` | Offline Resilience & Caching | MDN NavigationPreloadManager.enable(): Declarative Boot-Phase Network Concurrency | `official_documentation` | [MDN NavigationPreloadManager.enable](https://developer.mozilla.org/en-US/docs/Web/API/NavigationPreloadManager/enable) |
| `STANDARDS-MDN-NAV-PRELOAD-SET-HEADER-VALUE` | Platform Architecture & Mini-App Runtime | MDN NavigationPreloadManager.setHeaderValue(): Upstream Server Differentiation Telemetry | `official_documentation` | [MDN NavigationPreloadManager.setHeaderValue](https://developer.mozilla.org/en-US/docs/Web/API/NavigationPreloadManager/setHeaderValue) |
| `STANDARDS-MDN-FETCH-EVENT-PRELOAD-RESPONSE` | Offline Resilience & Caching | MDN FetchEvent.preloadResponse: Promise-Based Early Response Arbitration | `official_documentation` | [MDN FetchEvent.preloadResponse](https://developer.mozilla.org/en-US/docs/Web/API/FetchEvent/preloadResponse) |
| `STANDARDS-W3C-PAGE-VISIBILITY-SPEC` | Lifecycle, Power & Process Governance | W3C Page Visibility Level 2: Execution State & Power Governance Standard | `official_standard` | [W3C Page Visibility](https://www.w3.org/TR/page-visibility-2/) |
| `STANDARDS-MDN-DOCUMENT-VISIBILITYSTATE` | Lifecycle, Power & Process Governance | MDN Document.visibilityState: Viewport Inspection & Resource Throttling | `official_documentation` | [MDN Document.visibilityState](https://developer.mozilla.org/en-US/docs/Web/API/Document/visibilityState) |
| `STANDARDS-MDN-DOCUMENT-VISIBILITYCHANGE-EVENT` | Lifecycle, Power & Process Governance | MDN Document visibilitychange Event: Reactive Lifecycle Transition Arbitration | `official_documentation` | [MDN visibilitychange event](https://developer.mozilla.org/en-US/docs/Web/API/Document/visibilitychange_event) |
| `STANDARDS-W3C-SCREEN-ORIENTATION-SPEC` | Windowing & Display Governance | W3C Screen Orientation API: Display Geometry & Orientation Lock Standard | `official_standard` | [W3C Screen Orientation](https://www.w3.org/TR/screen-orientation/) |
| `STANDARDS-MDN-SCREEN-ORIENTATION-LOCK` | Windowing & Display Governance | MDN ScreenOrientation.lock(): Programmatic Viewport Locking & Permission Sandboxing | `official_documentation` | [MDN ScreenOrientation.lock](https://developer.mozilla.org/en-US/docs/Web/API/ScreenOrientation/lock) |

---

## 3. Phân Tích Kỹ Thuật Chi Tiết

### 3.1. WebRTC Statistics API: Đo Lường Chất Lượng & Telemetry Luồng Truyền Thực
1. **Kiến trúc bản đồ chỉ số bất biến (`RTCStatsReport`)**: `getStats()` trả về `RTCStatsReport` triển khai giao diện read-only Map, cô lập hoàn toàn trạng thái đo đạc với luồng thực thi media. Điều này ngăn chặn mã độc hại hoặc kịch bản bên thứ ba sửa đổi số liệu đo lường băng thông.
2. **Theo dõi chất lượng chiều nhận (`RTCInboundRtpStreamStats`)**:
   - `jitter` và `jitterBufferDelay`: Đo lường độ biến thiên độ trễ gói tin và thời gian đệm, phục vụ tự động điều chỉnh bộ đệm thích ứng (adaptive jitter buffer).
   - `packetsLost` và `packetsReceived`: Tính toán tỷ lệ rớt gói (Packet Loss Rate) theo thời gian thực để kích hoạt cơ chế giảm độ phân giải video hoặc chuyển codec âm thanh nhẹ hơn (Opus narrowband).
   - `framesDropped` và `framesDecoded`: Cung cấp thước đo chính xác về tình trạng nghẽn giải mã phần cứng trên thiết bị client yếu.
3. **Theo dõi chất lượng chiều gửi & giới hạn phần cứng (`RTCOutboundRtpStreamStats`)**:
   - Trường `qualityLimitationReason` (`cpu`, `bandwidth`, `none`, `other`) giúp hệ thống Super App phát hiện chính xác nguyên nhân mini app bị mờ/giật là do thiết bị quá nóng/nghẽn CPU hay do nghẽn băng thông vô tuyến, hỗ trợ cảnh báo người dùng trực quan.

### 3.2. Navigation Preload Manager: Xóa Bỏ Điểm Nghẽn Khởi Động Lạnh Service Worker
1. **Hiện tượng nghẽn luồng Service Worker Boot**: Theo mô hình truyền thống, khi người dùng mở mini app, trình duyệt phải khởi động tiến trình/luồng Service Worker trước khi bắt đầu tải trang HTML từ mạng hoặc cache. Quá trình boot này thường mất từ 50ms - 250ms trên điện thoại di động tầm trung.
2. **Kích hoạt song song (`NavigationPreloadManager.enable()`)**: Với Navigation Preload, trình duyệt phát ngay yêu cầu HTTP tải trang điều hướng song song tại thời điểm Service Worker vừa bắt đầu thức dậy.
3. **Phân biệt yêu cầu từ máy chủ qua header tùy biến**:
   - `navigationPreload.setHeaderValue(val)` cho phép gửi tiêu đề HTTP `Service-Worker-Navigation-Preload: <custom-value>` lên Gateway.
   - API Gateway có thể nhận biết đây là yêu cầu preload để trả về payload tối ưu (chỉ trả JSON state thay vì full HTML server-side rendering), giúp giảm tiêu thụ băng thông di động.
4. **Phân xử phản hồi với `event.preloadResponse`**: Trong hàm xử lý `fetch`, Service Worker đón nhận `event.preloadResponse` dạng Promise. Nếu cache hit sẵn có, Service Worker trả về từ cache; nếu không, response từ mạng đã sẵn sàng ngay mà không phải chờ đợi Service Worker boot xong.

### 3.3. Page Visibility Level 2 & Screen Orientation Sandboxing
1. **Quản trị vòng đời & năng lượng (`Document.visibilityState` & `visibilitychange`)**:
   - `document.visibilityState` phản ánh chính xác trạng thái hiển thị (`visible`, `hidden`). Khi mini app chuyển sang `hidden`, Super App WebView runtime tự động kích hoạt điều tiết giới hạn xung nhịp timers (`setInterval`/`setTimeout` bị làm trễ đến 1000ms+), đóng băng vòng lặp WebGL (`requestAnimationFrame`), và giải phóng camera/microphone.
   - Sự kiện `visibilitychange` là điểm neo tin cậy duy nhất trên nền tảng di động để lưu trữ trạng thái giao dịch chưa hoàn tất vào `IndexedDB` trước khi tiến trình WebView bị hệ điều hành (Android LMK / iOS Jetsam) thu hồi bộ nhớ.
2. **Chuẩn hóa xoay màn hình (`ScreenOrientation.lock`)**:
   - Cho phép các mini app đặc thù (trò chơi, phát video toàn màn hình, biểu đồ tài chính) khóa hướng xoay ngang (`landscape`) hoặc dọc (`portrait`).
   - Super App container sandbox buộc phải kiểm soát API này: chỉ cho phép gọi `screen.orientation.lock()` khi mini app đang ở chế độ Fullscreen hoặc Container độc lập, đồng thời tự động mở khóa (`screen.orientation.unlock()`) khi người dùng vuốt thoát ra ngoài trang chủ Super App để tránh làm lệch giao diện máy chủ.

---

## 4. Khuyến Nghị Thực Tiễn Cho Kiến Trúc Super App Mini App Store

1. **Khuyến nghị kiến trúc Runtime**:
   - **Tích hợp Navigation Preload mặc định**: Khuyến nghị mọi template Service Worker của Mini App Store kích hoạt `registration.navigationPreload.enable()` và sử dụng `event.preloadResponse` trong bộ điều hướng chính.
   - **Cơ chế Watchdog giám sát WebRTC**: Mini App Store SDK tích hợp bộ quan sát đo đạc `RTCPeerConnection.getStats()` với chu kỳ 2000ms, tự động ghi nhận tỷ lệ rớt gói và độ trễ vào hệ thống APM tập trung của Super App.
2. **Quy tắc Kiểm duyệt Ứng dụng (Store Review Guidelines)**:
   - **Bắt buộc xử lý `visibilitychange`**: Mọi mini app có tính năng nhập liệu form thanh toán hoặc trò chơi trực tuyến phải cài đặt trình lắng nghe `visibilitychange` để lưu trữ dữ liệu cục bộ an toàn.
   - **Kiểm soát quyền Orientation**: Mini app phải khai báo rõ trong manifest cấu hình `orientation` mặc định; mọi lời gọi `ScreenOrientation.lock()` tự do mà không qua tương tác người dùng (User Activation) hoặc ngoài chế độ fullscreen sẽ bị hệ thống sandbox từ chối với ngoại lệ `NotSupportedError`.
