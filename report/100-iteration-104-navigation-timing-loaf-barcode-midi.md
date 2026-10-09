# Báo Cáo Chuyên Đề: W3C Navigation Timing 2, LoAF Telemetry, Barcode Detection & Web MIDI Governance

**Mã tài liệu:** `100-iteration-104-navigation-timing-loaf-barcode-midi.md`
**Milestone:** 104
**Trạng thái:** Hoàn thành xác thực và tích hợp vào chuẩn Super App Mini App Store
**Tổng số findings tích lũy:** 1.511 findings (15 findings mới bổ sung trong vòng 104)
**Tác giả:** Hệ thống Tự trị Deli Deep Orchestrator
**Ngày cập nhật:** 2026-10-09

---

## 1. Tổng Quan & Bối Cảnh Kỹ Thuật

Trong vòng nghiên cứu Milestone 104, Deli Deep tập trung giải quyết 3 trụ cột kỹ thuật trọng yếu của nền tảng mini app hiện đại:
1. **Performance Telemetry & Frame Responsiveness (W3C Navigation Timing Level 2, Server Timing & Long Animation Frames API - LoAF):** Đo lường chi tiết sub-millisecond cho toàn bộ vòng đời tải trang mini app, phân lập độ trễ giữa edge proxy/gateway và backend compute, đồng thời cô lập chính xác các script/event-listener gây nghẽn giao diện (INP bottleneck) vượt ngưỡng 50ms.
2. **Computer Vision & Hardware-Accelerated Barcode Detection (WICG Shape Detection API):** Chuẩn hóa việc nhận diện mã vạch 1D/2D (QR code, EAN-13, Code 128) trực tiếp trên phần cứng OS (Apple Vision, Android ML Kit), loại bỏ các thư viện Wasm/JS cồng kềnh, tối ưu hóa camera viewport và ngăn chặn rủi ro XSS/injection từ payload mã vạch.
3. **Audio Hardware & Peripheral Governance (W3C Web MIDI API):** Quản trị kết nối thiết bị ngoại vi âm nhạc, synthesizer phần cứng, midi input/output stream, và kiểm soát nghiêm ngặt đặc quyền System Exclusive (SysEx) để ngăn chặn rủi ro nạp lại firmware phần cứng trái phép.

---

## 2. Chi Tiết Các Findings Bổ Sung (Iteration 104)

### Nhóm 1: W3C Navigation Timing Level 2, Server Timing & LoAF Telemetry

#### Finding 1: W3C Navigation Timing Level 2: Sub-Millisecond Page Lifecycle Telemetry
- **ID:** `STANDARDS-NAVIGATION-TIMING-2`
- **Phân loại:** `performance-telemetry-and-lifecycle-governance`
- **Mức độ bằng chứng:** `official_standard` (W3C Recommendation)
- **Nguồn xác thực:** [https://www.w3.org/TR/navigation-timing-2/](https://www.w3.org/TR/navigation-timing-2/)
- **Yêu cầu kỹ thuật cốt lõi:**
  1. Container mini app bắt buộc phải cung cấp đối tượng `PerformanceNavigationTiming` thông qua Performance Timeline API (`window.performance.getEntriesByType('navigation')`).
  2. SDK container phải thu thập các mốc thời gian trọng yếu: `responseStart`, `domInteractive`, `domContentLoadedEventEnd`, và `loadEventEnd` nhằm phân tích chính xác thời gian cold-start.
  3. Giá trị thời gian phải tuân thủ chuẩn `DOMHighResTimeStamp` với cơ chế làm thô xung nhịp (clock coarsening tối thiểu 5 microsecond) trong ngữ cảnh untrusted để chống side-channel fingerprinting.
  4. Container phải che giấu hoặc xóa mốc `redirectStart` và `redirectEnd` đối với các chuyển hướng cross-origin nhằm tránh rò rỉ URL nội bộ.
  5. Store review pipeline phải đo kiểm median `loadEventEnd` theo baseline phần cứng (ví dụ: <1.200ms trên thiết bị Android tầm trung).

#### Finding 2: W3C Server Timing API: End-to-End Gateway & Backend Latency Traceability
- **ID:** `STANDARDS-SERVER-TIMING-API`
- **Phân loại:** `performance-telemetry-and-lifecycle-governance`
- **Mức độ bằng chứng:** `official_standard` (W3C Recommendation)
- **Nguồn xác thực:** [https://www.w3.org/TR/server-timing/](https://www.w3.org/TR/server-timing/)
- **Yêu cầu kỹ thuật cốt lõi:**
  1. API Gateway của Super App và backend mini app nên phát sinh header `Server-Timing` (ví dụ: `Server-Timing: db;dur=53, edge;dur=12`) trên các HTTP response.
  2. Client runtime phải trích xuất các mục `PerformanceServerTiming` thông qua thuộc tính `.serverTiming` trên đối tượng `PerformanceResourceTiming` và `PerformanceNavigationTiming`.
  3. Không được tiết lộ thông tin hạ tầng nội bộ nhạy cảm (như hostname cụ thể của microservice nội bộ) vào header `Server-Timing` gửi xuống guest mini app.
  4. Nền tảng quan sát phải tương quan giữa thời gian mạng client và thời gian xử lý server để phân định rõ độ trễ mạng biên và độ trễ backend.
  5. Tài nguyên cross-origin bắt buộc phải khai báo header `Timing-Allow-Origin` thì client mới được phép truy cập chỉ số Server Timing chi tiết.

#### Finding 3: W3C Long Animation Frames API (LoAF): Main-Thread UI Responsiveness Diagnostics
- **ID:** `STANDARDS-LONG-ANIMATION-FRAMES-API`
- **Phân loại:** `performance-telemetry-and-lifecycle-governance`
- **Mức độ bằng chứng:** `official_standard` (W3C Working Draft)
- **Nguồn xác thực:** [https://www.w3.org/TR/long-animation-frames/](https://www.w3.org/TR/long-animation-frames/)
- **Yêu cầu kỹ thuật cốt lõi:**
  1. Container nên kích hoạt `PerformanceLongAnimationFrameTiming` để ghi nhận các frame animation vượt quá 50ms, phục vụ tối ưu hóa chỉ số Interaction to Next Paint (INP).
  2. SDK giám sát phải kiểm tra danh sách `PerformanceScriptTiming` lồng bên trong bản ghi LoAF để định vị chính xác URL script, thời gian thực thi, và loại invoker của các event handler gây nghẽn.
  3. Áp dụng quy tắc bảo mật làm mờ source script URL và vị trí ký tự đối với các script cross-origin không có CORS headers.
  4. Store quality gate phải gắn cờ cảnh báo các mini app phát sinh quá nhiều bản ghi LoAF trong các luồng thanh toán hoặc đặt hàng (checkout flow).
  5. Dữ liệu LoAF phải được tổng hợp cục bộ thông qua `PerformanceObserver` với cờ `buffered: true` và flush về server định kỳ trong thời gian rảnh (`requestIdleCallback`).

#### Finding 4: PerformanceNavigationTiming: Navigation Type & Cache Validation Telemetry
- **ID:** `STANDARDS-PERFORMANCE-NAVIGATION-TIMING-INTERFACE`
- **Phân loại:** `performance-telemetry-and-lifecycle-governance`
- **Mức độ bằng chứng:** `developer_documentation` (MDN Web Docs)
- **Nguồn xác thực:** [https://developer.mozilla.org/en-US/docs/Web/API/PerformanceNavigationTiming](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceNavigationTiming)
- **Yêu cầu kỹ thuật cốt lõi:**
  1. Container phải điền chính xác giá trị `PerformanceNavigationTiming.type` (`navigate`, `reload`, `back_forward`, `prerender`) để chuẩn hóa ngữ cảnh telemetry.
  2. SDK quan sát phải so sánh `transferSize` với `encodedBodySize` để tính toán chính xác tỷ lệ cache-hit của disk cache và Service Worker.
  3. Mini app phải sử dụng mốc `domInteractive` để đánh giá chi phí parse DOM độc lập với thời gian nạp các tài nguyên phụ tải trễ.
  4. Thực thi cô lập origin tuyệt đối: iframe nhúng không thể truy vấn timing của cửa sổ host cha.
  5. Báo cáo đo kiểm phải ghi chú rõ ràng các lần điều hướng được khôi phục từ BFCache để không làm méo mó chỉ số cold-start KPI.

#### Finding 5: PerformanceLongAnimationFrameTiming: Script Invocation & Frame Attribution
- **ID:** `STANDARDS-PERFORMANCE-LONG-ANIMATION-FRAME-TIMING`
- **Phân loại:** `performance-telemetry-and-lifecycle-governance`
- **Mức độ bằng chứng:** `developer_documentation` (MDN Web Docs)
- **Nguồn xác thực:** [https://developer.mozilla.org/en-US/docs/Web/API/PerformanceLongAnimationFrameTiming](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceLongAnimationFrameTiming)
- **Yêu cầu kỹ thuật cốt lõi:**
  1. WebView container (Chromium 123+) phải hỗ trợ interface `PerformanceLongAnimationFrameTiming` qua `PerformanceObserver`.
  2. Developer phải phân tích mảng `scripts` để phân biệt re-render của UI framework với độ trễ từ các SDK tracking bên thứ ba.
  3. Hệ thống kiểm thử tự động tính tổng thời gian chặn (blocking time) bằng cách lấy thời lượng LoAF trừ đi chu kỳ quét màn hình chuẩn (16,6ms hoặc 8,3ms).
  4. Pipeline kiểm thử pre-submission phải mô phỏng thao tác vuốt chạm phức tạp và gắn cờ nếu giai đoạn từ `renderStart` đến `styleAndLayoutEnd` vượt quá 30ms.
  5. Cầu nối telemetry phải áp dụng cơ chế debounce và rate-limit để tránh vòng lặp tự giám sát làm đầy bộ nhớ thiết bị.

---

### Nhóm 2: Computer Vision & Barcode Detection Hardware Acceleration

#### Finding 6: WICG Shape Detection API: Hardware-Accelerated Barcode & Vision Extraction
- **ID:** `STANDARDS-SHAPE-DETECTION-API`
- **Phân loại:** `computer-vision-and-peripheral-governance`
- **Mức độ bằng chứng:** `specification_draft` (WICG Community Draft)
- **Nguồn xác thực:** [https://wicg.github.io/shape-detection-api/](https://wicg.github.io/shape-detection-api/)
- **Yêu cầu kỹ thuật cốt lõi:**
  1. Container hỗ trợ quét vé và thanh toán thương mại nên cung cấp `BarcodeDetector` tận dụng API thị giác máy tính phần cứng native của hệ điều hành (Apple Vision framework, Google ML Kit).
  2. Mini app phải thăm dò định dạng mã vạch hỗ trợ thông qua `BarcodeDetector.getSupportedFormats()` trước khi khởi tạo luồng quét.
  3. Quyền truy cập camera và user activation tức thời phải được xác minh trước khi cấp dữ liệu video stream vào pipeline xử lý.
  4. Dữ liệu thô trích xuất từ mã vạch phải được làm sạch và xác thực nghiêm ngặt trước khi chuyển vào bộ định tuyến thanh toán hoặc deep link.
  5. Khả năng quét mã vạch và nhận diện khuôn mặt phải được quản lý chặt chẽ bằng chính sách `Permissions-Policy` của container để ngăn chặn theo dõi ngầm.

#### Finding 7: Barcode Detection API: Low-Latency QR & Barcode Recognition Architecture
- **ID:** `STANDARDS-BARCODE-DETECTOR-API`
- **Phân loại:** `computer-vision-and-peripheral-governance`
- **Mức độ bằng chứng:** `developer_documentation` (MDN Web Docs)
- **Nguồn xác thực:** [https://developer.mozilla.org/en-US/docs/Web/API/BarcodeDetector](https://developer.mozilla.org/en-US/docs/Web/API/BarcodeDetector)
- **Yêu cầu kỹ thuật cốt lõi:**
  1. Mini app phải khởi tạo `BarcodeDetector` với tùy chọn định dạng rõ ràng (ví dụ: `['qr_code', 'ean_13']`) để tối ưu hóa bộ tăng tốc phần cứng và tiết kiệm điện năng.
  2. Phương thức `detect()` nhận đối tượng nguồn `ImageBitmapSource` và trả về danh sách `DetectedBarcode` chứa `rawValue`, `format`, `boundingBox`, và `cornerPoints`.
  3. Container phải giới hạn tần số quét liên tục (tối đa 15-30 fps) để chống quá nhiệt và hao pin trên điện thoại di động.
  4. Payload chuỗi giải mã phải được xem là dữ liệu không tin cậy, yêu cầu khử khuẩn chống XSS, SQL injection và giả mạo URL scheme.
  5. Cung cấp fallback module Wasm/JS trên các nền tảng cũ thiếu hỗ trợ BarcodeDetector native từ OS.

#### Finding 8: BarcodeDetector.getSupportedFormats(): Dynamic Scanner Capability Negotiation
- **ID:** `STANDARDS-BARCODE-DETECTOR-GET-SUPPORTED-FORMATS`
- **Phân loại:** `computer-vision-and-peripheral-governance`
- **Mức độ bằng chứng:** `developer_documentation` (MDN Web Docs)
- **Nguồn xác thực:** [https://developer.mozilla.org/en-US/docs/Web/API/BarcodeDetector/getSupportedFormats_static](https://developer.mozilla.org/en-US/docs/Web/API/BarcodeDetector/getSupportedFormats_static)
- **Yêu cầu kỹ thuật cốt lõi:**
  1. Component quét mã phải gọi phương thức tĩnh `BarcodeDetector.getSupportedFormats()` khi khởi động để điều chỉnh giao diện quét tương ứng.
  2. Host container phải trả về danh sách chuỗi định dạng bất biến theo đúng đặc tả WICG.
  3. Nếu định dạng thương mại chuyên dụng (như PDF417 trên vé máy bay hoặc Code 39 trong kho vận) không được phần cứng hỗ trợ, app phải chuyển đổi sang giải pháp WebAssembly hoặc giải mã phía server.
  4. Quy trình duyệt app phải kiểm tra tính linh hoạt khi fallback định dạng để tránh crash runtime trên thiết bị giá rẻ.
  5. Truy vấn tính năng phần cứng không được chứa mã định danh thiết bị vĩnh viễn nhằm chống rủi ro fingerprinting người dùng.

#### Finding 9: BarcodeDetector.detect(): Asynchronous Frame Analysis & Coordinate Geometry
- **ID:** `STANDARDS-BARCODE-DETECTOR-DETECT-METHOD`
- **Phân loại:** `computer-vision-and-peripheral-governance`
- **Mức độ bằng chứng:** `developer_documentation` (MDN Web Docs)
- **Nguồn xác thực:** [https://developer.mozilla.org/en-US/docs/Web/API/BarcodeDetector/detect](https://developer.mozilla.org/en-US/docs/Web/API/BarcodeDetector/detect)
- **Yêu cầu kỹ thuật cốt lõi:**
  1. Mini app phải truyền các element hợp lệ (`canvas`, `img`, `video`) vào `detect()` và xử lý promise rejection khi buffer ảnh bị tách rời hoặc dữ liệu không hợp lệ.
  2. Mảng tọa độ `DetectedBarcode.cornerPoints` phải được sử dụng để vẽ khung ngắm tương tác thực tế ảo (AR) chính xác quanh vật thể quét.
  3. Luồng quét camera phải tạm dừng hoặc giảm tần suất ngay khi phát hiện mã thành công để ngăn chặn sự kiện kích hoạt kép ngoài ý muốn.
  4. Container phải đảm bảo frame video truyền vào xuất phát từ `MediaStreamTrack` đang hoạt động với sự đồng thuận của người dùng.
  5. Luồng video độ phân giải cao nên được crop hoặc thu nhỏ về vùng khung ngắm để tăng tốc độ phân tích và giảm dung lượng RAM.

#### Finding 10: PerformanceScriptTiming: JavaScript Execution Attribution in LoAF Runtimes
- **ID:** `STANDARDS-PERFORMANCE-SCRIPT-TIMING-INTERFACE`
- **Phân loại:** `performance-telemetry-and-lifecycle-governance`
- **Mức độ bằng chứng:** `developer_documentation` (MDN Web Docs)
- **Nguồn xác thực:** [https://developer.mozilla.org/en-US/docs/Web/API/PerformanceScriptTiming](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceScriptTiming)
- **Yêu cầu kỹ thuật cốt lõi:**
  1. Container hỗ trợ LoAF phải điền chi tiết loại invoker (`user-callback`, `event-listener`, `promise-resolve`, `classic-script`).
  2. Công cụ giám sát hiệu năng phải tính toán tỷ lệ giữa thời gian thực thi (execution duration) và thời gian biên dịch (compile duration) để phát hiện độ trễ JIT compilation khi mở app.
  3. Các script cross-origin thiếu cờ `Timing-Allow-Origin` phải được làm sạch các trường `sourceURL`, `sourceFunctionName` và `sourceCharPosition` để bảo vệ mã nguồn.
  4. SDK chẩn đoán của mini app phải dùng dữ liệu này để phát hiện chuỗi promise đệ quy không giới hạn hoặc các DOM event handler nặng nề.
  5. Bộ tổng hợp telemetry của kho ứng dụng phải nhóm các lỗi chậm trễ theo nhà cung cấp SDK bên thứ ba để thiết lập hạn ngạch hiệu năng cho toàn hệ sinh thái.

---

### Nhóm 3: Audio Hardware & Peripheral Governance (W3C Web MIDI API)

#### Finding 11: W3C Web MIDI API: Musical Instrument & Peripheral Hardware Governance
- **ID:** `STANDARDS-WEBMIDI-API`
- **Phân loại:** `audio-hardware-and-peripheral-governance`
- **Mức độ bằng chứng:** `official_standard` (W3C Recommendation)
- **Nguồn xác thực:** [https://www.w3.org/TR/webmidi/](https://www.w3.org/TR/webmidi/)
- **Yêu cầu kỹ thuật cốt lõi:**
  1. Lệnh gọi `navigator.requestMIDIAccess()` phải được kiểm soát qua hộp thoại xin quyền rõ ràng và chỉ thị `Permissions-Policy ('midi')` của container.
  2. Yêu cầu có tùy chọn `sysex: true` (System Exclusive) bắt buộc phải có bước xác nhận bảo mật cao từ người dùng do nguy cơ can thiệp nạp lại firmware phần cứng.
  3. Container phải cô lập cổng MIDI giữa các mini app khác nhau, ngăn chặn các tab chạy nền nghe lén luồng thông điệp MIDI.
  4. Mini app tương tác với nhạc cụ phải xử lý mềm dẻo các sự kiện ngắt kết nối và kết nối lại thông qua sự kiện `statechange`.
  5. Chính sách duyệt app phải thẩm tra chặt chẽ các app yêu cầu quyền SysEx, từ chối các app thông thường không thuộc danh mục sáng tạo âm nhạc hoặc trò chơi.

#### Finding 12: MIDIAccess Interface: Device Enumeration & Connection State Machine
- **ID:** `STANDARDS-MIDI-ACCESS-INTERFACE`
- **Phân loại:** `audio-hardware-and-peripheral-governance`
- **Mức độ bằng chứng:** `developer_documentation` (MDN Web Docs)
- **Nguồn xác thực:** [https://developer.mozilla.org/en-US/docs/Web/API/MIDIAccess](https://developer.mozilla.org/en-US/docs/Web/API/MIDIAccess)
- **Yêu cầu kỹ thuật cốt lõi:**
  1. Thuộc tính `inputs` và `outputs` trên `MIDIAccess` phải cung cấp iterator bất biến đối với danh sách thiết bị phần cứng đang kết nối.
  2. Container phải phát sự kiện `statechange` bất cứ khi nào bộ điều khiển phần cứng được cắm vào hoặc rút ra khỏi máy.
  3. Mini app phải kiểm tra cờ boolean `sysexEnabled` trước khi gửi các gói tin cấu hình tham số phần cứng đặc thù của hãng.
  4. Container phải áp dụng ranh giới origin, đảm bảo đối tượng MIDIAccess không bị rò rỉ qua các iframe hoặc cửa sổ đa tenant.
  5. Cơ chế giám sát container phải theo dõi tần suất gọi MIDIAccess để phát hiện hành vi quét thiết bị trái phép nhằm định danh thiết bị (device fingerprinting).

#### Finding 13: MIDIPort Interface: Peripheral Port Identity & Connection Mechanics
- **ID:** `STANDARDS-MIDI-PORT-INTERFACE`
- **Phân loại:** `audio-hardware-and-peripheral-governance`
- **Mức độ bằng chứng:** `developer_documentation` (MDN Web Docs)
- **Nguồn xác thực:** [https://developer.mozilla.org/en-US/docs/Web/API/MIDIPort](https://developer.mozilla.org/en-US/docs/Web/API/MIDIPort)
- **Yêu cầu kỹ thuật cốt lõi:**
  1. Các phương thức bất đồng bộ `MIDIPort.open()` và `MIDIPort.close()` phải được sử dụng để quản lý tài nguyên bus và driver phần cứng một cách tất định.
  2. Container phải chuẩn hóa chuỗi `name` và `manufacturer` trước khi chuyển giao cho code guest để ngăn chặn thu thập dấu vân tay phần cứng.
  3. Trạng thái kết nối của port (`open`, `closed`, `pending`) phải phản ánh chính xác trạng thái của hệ thống âm thanh OS (CoreMIDI trên iOS/macOS, ALSA trên Linux, Windows MIDI).
  4. Mini app phải chủ động đóng tất cả các cổng MIDI khi chuyển trang hoặc bước vào trạng thái suspended/background.
  5. Host runtime phải lập tức thu hồi quyền truy cập port nếu người dùng tắt quyền phần cứng trong cài đặt của Super App.

#### Finding 14: MIDIInput Interface: Low-Latency Message Stream Ingestion & Timing
- **ID:** `STANDARDS-MIDI-INPUT-INTERFACE`
- **Phân loại:** `audio-hardware-and-peripheral-governance`
- **Mức độ bằng chứng:** `developer_documentation` (MDN Web Docs)
- **Nguồn xác thực:** [https://developer.mozilla.org/en-US/docs/Web/API/MIDIInput](https://developer.mozilla.org/en-US/docs/Web/API/MIDIInput)
- **Yêu cầu kỹ thuật cốt lõi:**
  1. Listener `onmidimessage` nhận đối tượng `MIDIMessageEvent` chứa payload mảng byte `Uint8Array` và nhãn thời gian `DOMHighResTimeStamp`.
  2. Container phải kiểm tra độ dài thông điệp nghiêm ngặt (1-3 bytes đối với channel voice message) để chống tấn công tràn bộ đệm tại tầng bridge native.
  3. Mini app nên xử lý các thông điệp control change tần số cao trong Web Worker hoặc thông qua Web Audio AudioWorklet để tránh giật lag giao diện chính.
  4. Container phải tạm ngưng gửi thông điệp MIDIInput khi cửa sổ mini app mất focus hoặc bị che khuất.
  5. Audit duyệt kho phải kiểm tra app xử lý giải phóng nốt (note-off) đúng quy cách để tránh hiện tượng treo tiếng âm thanh (stuck note).

#### Finding 15: MIDIOutput Interface: Hardware Sequencing & Timestamped Message Dispatch
- **ID:** `STANDARDS-MIDI-OUTPUT-INTERFACE`
- **Phân loại:** `audio-hardware-and-peripheral-governance`
- **Mức độ bằng chứng:** `developer_documentation` (MDN Web Docs)
- **Nguồn xác thực:** [https://developer.mozilla.org/en-US/docs/Web/API/MIDIOutput](https://developer.mozilla.org/en-US/docs/Web/API/MIDIOutput)
- **Yêu cầu kỹ thuật cốt lõi:**
  1. Phương thức `MIDIOutput.send(data, timestamp)` nhận chuỗi byte chuẩn MIDI 1.0 và lập lịch truyền phát theo dòng thời gian `DOMHighResTimeStamp`.
  2. Container phải áp dụng giới hạn thông lượng (chuẩn băng thông 31.25 kbaud) để chống tấn công làm nghẽn bus (DoS) trên thiết bị gắn ngoài.
  3. Lệnh gửi SysEx phải được kiểm duyệt theo danh mục mã nhà sản xuất hợp lệ nếu chính sách bảo mật container cấm can thiệp firmware tự do.
  4. Mini app nên hỗ trợ phương thức `clear()` để hủy hàng đợi lệnh khi người dùng bấm dừng hoặc thoát màn hình.
  5. Cầu nối container native phải bắt và cô lập triệt để các lỗi truyền driver phần cứng để tránh làm sập ứng dụng Super App mẹ.

---

## 3. Ma Trận Đánh Giá Kiểm Duyệt & Giám Sát Runtime

| Thành Phần Chuẩn | Mức Độ Rủi Ro | Kiểm Duyệt Pre-submission (Store Review) | Cơ Chế Giám Sát Runtime (Container Defense) |
|---|---|---|---|
| **W3C Navigation Timing 2** | Thấp (Fingerprinting) | Kiểm tra baseline `loadEventEnd` theo phân tầng thiết bị (<1.200ms). | Làm thô timestamp (clock coarsening 5us); xóa mốc redirectStart/End cross-origin. |
| **W3C Server Timing** | Trung bình (Rò rỉ hạ tầng) | Quét kiểm tra định dạng header `Server-Timing` không chứa IP/host nội bộ. | Yêu cầu `Timing-Allow-Origin` cho tài nguyên cross-origin trước khi cho phép đọc. |
| **W3C Long Animation Frames (LoAF)** | Trung bình (Tắc nghẽn UI) | Chạy automated test mô phỏng vuốt chạm, gắn cờ nếu `styleAndLayoutEnd` > 30ms. | Che giấu source URL nếu script thiếu CORS; debounce lưu telemetry qua requestIdleCallback. |
| **WICG Barcode Detection** | Cao (XSS / Injection / Quyền Camera) | Thẩm định xử lý sanitize chuỗi rawValue; kiểm tra fallback Wasm khi thiếu native scanner. | Kiểm soát qua `Permissions-Policy`; giới hạn tần số quét tối đa 15-30 fps; pause quét khi match. |
| **W3C Web MIDI API** | Nghiêm trọng (Nạp Firmware / Bus DoS) | Từ chối cấp quyền SysEx cho app thông thường; chỉ phê duyệt app thuộc danh mục nhạc/sáng tạo. | Bắt buộc prompt xác nhận quyền cấp cao; áp giới hạn tốc độ 31.25 kbaud; cô lập origin đa tenant. |

---

## 4. Kết Luận & Hướng Nghiên Cứu Tiếp Theo

Milestone 104 đã hoàn thành bổ sung 15 tiêu chuẩn và yêu cầu kỹ thuật nền tảng, nâng tổng số findings chuẩn hóa của Super App Mini App Store lên **1.511 findings**. Hệ thống đã mở rộng từ các tiêu chuẩn web cốt lõi sang các khả năng phần cứng chuyên sâu (thị giác máy tính tăng tốc phần cứng và điều khiển ngoại vi âm nhạc), đồng thời hoàn thiện bộ đo kiểm quan sát hiệu năng sub-millisecond cho container.
