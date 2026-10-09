# Chuyên đề 103 (Iteration 107): WHATWG Encoding Living Standard & Binary Stream Processing, W3C Performance Timeline Level 2 & PerformanceObserver Telemetry Architecture, and W3C CSS Containment Module Level 1 Strict Layout Sandboxing

## 1. Tổng quan & Bối cảnh kỹ thuật

Iteration 107 tập trung vào ba trụ cột kỹ thuật nền tảng quyết định độ tin cậy dữ liệu, khả năng quan sát hiệu năng và tính cô lập giao diện trong môi trường Super-App đa người thuê (Multi-Tenant Super-App Container):

1. **WHATWG Encoding Living Standard & Binary Stream Processing**: Thiết lập chuẩn mực xử lý chuỗi và luồng nhị phân. Bắt buộc chuẩn hóa UTF-8 tuyệt đối cho mọi dữ liệu văn bản, loại bỏ hoàn toàn các lỗi sai lệch bộ ký tự (encoding corruption) và các lỗ hổng bảo mật tấn công bypass XSS dựa trên mã hóa byte. Tận dụng `TextEncoder`/`TextDecoder` cùng các pipeline stream `TextEncoderStream`/`TextDecoderStream` để truyền tải dữ liệu kích thước lớn theo cơ chế phản áp (backpressure), loại bỏ tình trạng tràn bộ nhớ (Out-Of-Memory) trên thiết bị di động.
2. **W3C Performance Timeline Level 2 & PerformanceObserver Telemetry Architecture**: Chuẩn hóa hệ thống thu thập số liệu hiệu năng client-side thời gian thực với độ phân giải cao (`DOMHighResTimeStamp`). Cung cấp cơ chế tự kiểm tra tính năng tĩnh thông qua `PerformanceObserver.supportedEntryTypes`, giải phóng bộ đệm đồng bộ tức thì bằng `takeRecords()` khi mini-app chuyển trạng thái lifecycle (unmount/background), xử lý hàng loạt qua `PerformanceObserverEntryList`, và chuẩn hóa xuất dữ liệu JSON có cấu trúc bằng `PerformanceEntry.toJSON()` qua Native Bridge về trung tâm quan sát (SOC/APM).
3. **W3C CSS Containment Module Level 1 & Strict Layout Sandboxing**: Thiết lập cơ chế cô lập cây layout con (DOM Subtree Reflow Isolation) bằng thuộc tính `contain` (`strict`, `content`, `size`, `layout`, `paint`, `style`). Ngăn chặn triệt để hiện tượng re-layout lan truyền từ mini-app con làm giật lag giao diện feed chính của Super-App. Kết hợp với `contain-intrinsic-width`/`contain-intrinsic-height` và `content-visibility: auto/hidden` để giải phóng tài nguyên vẽ (paint) và tính toán hình học cho các thành phần ngoài màn hình, duy trì tốc độ khung hình 60fps mượt mà trên phần cứng di động cấu hình thấp.

---

## 2. Chi tiết 15 Chuẩn mực & Findings mới (Iteration 107)

### Nhóm 1: WHATWG Encoding Living Standard & Binary Stream Processing

#### 1. WHATWG Encoding Living Standard: UTF-8 Exclusivity & Deterministic Character Decoding
- **ID**: `STANDARDS-WHATWG-ENCODING-LIVING-STANDARD`
- **Category**: `data-encoding-and-stream-processing`
- **Evidence Level**: `living-standard`
- **URL**: [https://encoding.spec.whatwg.org/](https://encoding.spec.whatwg.org/)
- **Quy định kỹ thuật (Spec Fact)**: WHATWG Encoding Living Standard quy định thuật toán chuyển đổi chuẩn giữa luồng byte và code points. Chuẩn hóa UTF-8 là định dạng duy nhất hợp lệ cho mọi API web hiện đại. Cấm ghi/mã hóa sang các định dạng cũ (như Shift_JIS, GBK, Windows-1252), đồng thời áp dụng thuật toán thay thế ký tự lỗi an toàn (U+FFFD) cho các chuỗi byte không hợp lệ.
- **Kiểm soát Super-App (Container Control)**: Ép buộc chuẩn hóa mã hóa UTF-8 trên toàn bộ Native Bridge, kênh IPC giữa WebWorker và UI Thread, và tầng lưu trữ client của mini-app. Loại bỏ hoàn toàn nguy cơ tấn công tiêm mã độc XSS thông qua kỹ thuật khai thác sai lệch bảng mã (character set confusion).
- **Quy tắc kiểm duyệt Store (Store Review Rule)**: Gói mã nguồn mini-app và các payload trao đổi dữ liệu qua bridge phải khai báo rõ ràng và tuân thủ UTF-8. Các gói chứa tệp tin mã hóa legacy không được hỗ trợ sẽ bị từ chối tự động tại cổng kiểm duyệt.

#### 2. TextEncoder API: Zero-Copy UTF-8 Byte Array Serialization & Memory Buffer Management
- **ID**: `STANDARDS-WHATWG-TEXTENCODER-API`
- **Category**: `data-encoding-and-stream-processing`
- **Evidence Level**: `official-specification`
- **URL**: [https://developer.mozilla.org/en-US/docs/Web/API/TextEncoder](https://developer.mozilla.org/en-US/docs/Web/API/TextEncoder)
- **Quy định kỹ thuật (Spec Fact)**: `TextEncoder.encode()` chuyển đổi chuỗi JavaScript thành một mảng `Uint8Array` chứa các byte UTF-8 hợp lệ. Phương thức tối ưu hóa cao `encodeInto(string, uint8Array)` cho phép mã hóa trực tiếp vào mảng đệm đích được cấp phát trước, trả về số lượng code points đã đọc và số bytes đã ghi, ngăn chặn cấp phát bộ nhớ rác (GC churn).
- **Kiểm soát Super-App (Container Control)**: Cần thiết cho các pipeline mã hóa JSON RPC tốc độ cao, ký số mật mã `WebCrypto`, và truyền tải thông điệp nhị phân giữa Web Workers mà không làm nghẽn luồng xử lý giao diện chính (Main UI Thread).
- **Quy tắc kiểm duyệt Store (Store Review Rule)**: Các mini-app xử lý khối lượng lớn giao dịch nhị phân hoặc ký mã xác thực phải sử dụng `TextEncoder` native thay vì các thư viện polyfill JavaScript cồng kềnh để tránh phân mảnh bộ nhớ RAM.

#### 3. TextDecoder API: Multi-Charset Legacy Fallback & Streaming Multi-Chunk Decoding
- **ID**: `STANDARDS-WHATWG-TEXTDECODER-API`
- **Category**: `data-encoding-and-stream-processing`
- **Evidence Level**: `official-specification`
- **URL**: [https://developer.mozilla.org/en-US/docs/Web/API/TextDecoder](https://developer.mozilla.org/en-US/docs/Web/API/TextDecoder)
- **Quy định kỹ thuật (Spec Fact)**: `TextDecoder.decode()` chuyển đổi `ArrayBuffer` hoặc buffer view thành chuỗi ký tự JavaScript, hỗ trợ tùy chọn `{ stream: true }` để giải mã lũy tiến qua các ranh giới gói tin mà không làm đứt đoạn các ký tự multibyte (ví dụ các ký tự Unicode nhiều byte). Cung cấp tùy chọn `fatal: true` (bắn ngoại lệ khi gặp byte lỗi) và `ignoreBOM`.
- **Kiểm soát Super-App (Container Control)**: Đảm bảo giải mã mượt mà và an toàn các luồng dữ liệu mạng, Server-Sent Events (SSE), và phản hồi từ các hệ thống máy chủ đối tác legacy mà không làm gián đoạn ứng dụng khi gặp ký tự phân mảnh.
- **Quy tắc kiểm duyệt Store (Store Review Rule)**: Mini-app xử lý dữ liệu nhị phân từ mạng phải cấu hình cơ chế bắt lỗi giải mã rõ ràng hoặc dùng fallback thay thế ký tự an toàn để ngăn chặn sự cố sập ứng dụng (runtime crashes).

#### 4. TextEncoderStream API: TransformStream Asynchronous UTF-8 Byte Pipelining
- **ID**: `STANDARDS-WHATWG-TEXTENCODERSTREAM-API`
- **Category**: `data-encoding-and-stream-processing`
- **Evidence Level**: `official-specification`
- **URL**: [https://developer.mozilla.org/en-US/docs/Web/API/TextEncoderStream](https://developer.mozilla.org/en-US/docs/Web/API/TextEncoderStream)
- **Quy định kỹ thuật (Spec Fact)**: Hiện thực giao diện `TransformStream`, chuyển đổi luồng chuỗi ký tự đầu vào thành luồng các khối `Uint8Array` UTF-8 theo cơ chế bất đồng bộ, tích hợp trực tiếp với các đường ống `pipeThrough()` và cơ chế truyền phản áp (backpressure).
- **Kiểm soát Super-App (Container Control)**: Cho phép các mini-app tài chính và thương mại điện tử xuất báo cáo CSV/JSON dung lượng lớn hoặc ghi dữ liệu ngoại tuyến vào OPFS mà không cần giữ toàn bộ chuỗi khổng lồ trong bộ nhớ RAM của thiết bị di động.
- **Quy tắc kiểm duyệt Store (Store Review Rule)**: Các chức năng tải tệp xuất dữ liệu lớn hơn 5MB bắt buộc phải sử dụng `TextEncoderStream` để đảm bảo lượng RAM tiêu thụ luôn được giới hạn dưới 64KB trong suốt quá trình truyền tải.

#### 5. TextDecoderStream API: Chunked Stream Processing Without Main-Thread Memory Spikes
- **ID**: `STANDARDS-WHATWG-TEXTDECODERSTREAM-API`
- **Category**: `data-encoding-and-stream-processing`
- **Evidence Level**: `official-specification`
- **URL**: [https://developer.mozilla.org/en-US/docs/Web/API/TextDecoderStream](https://developer.mozilla.org/en-US/docs/Web/API/TextDecoderStream)
- **Quy định kỹ thuật (Spec Fact)**: Cung cấp `TransformStream` nhận các khối byte nhị phân và phát ra các chuỗi ký tự đã được giải mã chính xác, tự động quản lý bộ đệm trung gian khi các ký tự multibyte bị cắt đôi giữa hai gói tin mạng.
- **Kiểm soát Super-App (Container Control)**: Cho phép các mini-app trợ lý AI hội thoại (Generative AI) và truyền phát trực tiếp (live streaming) hiển thị văn bản theo thời gian thực (token-by-token streaming) mà không gây giật lag giao diện và không tạo đỉnh tiêu thụ bộ nhớ (RAM spikes).
- **Quy tắc kiểm duyệt Store (Store Review Rule)**: Ứng dụng AI hội thoại hiển thị câu trả lời dạng stream bắt buộc dùng `TextDecoderStream` kết hợp DOM patching tối ưu để duy trì độ trễ tương tác dưới 100ms.

---

### Nhóm 2: W3C Performance Timeline Level 2 & PerformanceObserver Telemetry Architecture

#### 6. W3C Performance Timeline Level 2: Unified Client Observability & High-Resolution Metrics
- **ID**: `STANDARDS-W3C-PERF-TIMELINE-LEVEL-2`
- **Category**: `performance-and-runtime-observability`
- **Evidence Level**: `official-specification`
- **URL**: [https://w3c.github.io/performance-timeline/](https://w3c.github.io/performance-timeline/)
- **Quy định kỹ thuật (Spec Fact)**: Chuẩn hóa framework đo lường hiệu năng client-side thông qua giao diện `Performance`, đồng hồ đơn điệu độ phân giải micro-giây `DOMHighResTimeStamp` (`performance.now()`), và kiến trúc đệm số liệu độc lập cho toàn bộ các metric (Navigation, Resource, Paint, Long Task, Event Timing).
- **Kiểm soát Super-App (Container Control)**: Cung cấp nền tảng chuẩn mực cho SDK giám sát hiệu năng của Super-App, đo đạc chính xác thời gian khởi động mini-app, các mốc hiển thị hình ảnh đầu tiên, và độ trễ phản hồi của Native Bridge.
- **Quy tắc kiểm duyệt Store (Store Review Rule)**: Mini-app không được phép ghi đè các hàm gốc của `performance` hoặc can thiệp làm sai lệch dữ liệu đo lường; tính toàn vẹn của dữ liệu hiệu năng là tiêu chí bắt buộc trong xếp hạng ứng dụng.

#### 7. PerformanceObserver.supportedEntryTypes: Static Feature Introspection & Graceful Telemetry Gating
- **ID**: `STANDARDS-W3C-PERFOBSERVER-SUPPORTED-ENTRY-TYPES`
- **Category**: `performance-and-runtime-observability`
- **Evidence Level**: `official-specification`
- **URL**: [https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver/supportedEntryTypes_static](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver/supportedEntryTypes_static)
- **Quy định kỹ thuật (Spec Fact)**: Thuộc tính tĩnh `PerformanceObserver.supportedEntryTypes` trả về một mảng đóng băng (frozen array) chứa danh sách các chuỗi định danh loại metric được WebView hỗ trợ (ví dụ: 'navigation', 'paint', 'largest-contentful-paint', 'layout-shift', 'long-animation-frame').
- **Kiểm soát Super-App (Container Control)**: Giúp SDK Super-App kiểm tra tính năng động trước khi đăng ký lắng nghe số liệu, đảm bảo mã nguồn chạy ổn định không phát sinh lỗi trên các phiên bản WebView cũ của Android và iOS WKWebView.
- **Quy tắc kiểm duyệt Store (Store Review Rule)**: Mã giám sát hiệu năng trong mini-app phải kiểm tra `supportedEntryTypes` trước khi gọi `observe({ type })` để tránh gây ra ngoại lệ `TypeError` chưa được xử lý.

#### 8. PerformanceObserver.takeRecords(): Synchronous Buffer Drainage on Lifecycle Teardown
- **ID**: `STANDARDS-W3C-PERFOBSERVER-TAKERECORDS`
- **Category**: `performance-and-runtime-observability`
- **Evidence Level**: `official-specification`
- **URL**: [https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver/takeRecords](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver/takeRecords)
- **Quy định kỹ thuật (Spec Fact)**: Phương thức `takeRecords()` lập tức rút sạch và trả về danh sách các bản ghi hiệu năng đang chờ xử lý trong hàng đợi của observer, đồng thời đặt lại hàng đợi về rỗng. Đảm bảo dữ liệu không bị thất thoát khi context bị đóng đột ngột.
- **Kiểm soát Super-App (Container Control)**: Khi người dùng đóng mini-app hoặc điều hướng sang trang khác (`pagehide`/`freeze`), Super-App container kích hoạt `takeRecords()` để đóng gói toàn bộ dữ liệu LCP, INP và Resource Timing vào beacon gửi về server trước khi tiến trình WebView bị hủy.
- **Quy tắc kiểm duyệt Store (Store Review Rule)**: SDK thu thập telemetry phải gọi `takeRecords()` trong sự kiện kết thúc vòng đời trang để đảm bảo tỷ lệ thu thập số liệu đạt 100% cho các phiên giao dịch ngắn.

#### 9. PerformanceObserverEntryList: Batch Telemetry Querying & Type-Specific Filtering
- **ID**: `STANDARDS-W3C-PERFOBSERVER-ENTRY-LIST`
- **Category**: `performance-and-runtime-observability`
- **Evidence Level**: `official-specification`
- **URL**: [https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserverEntryList](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserverEntryList)
- **Quy định kỹ thuật (Spec Fact)**: Cung cấp đối tượng tập hợp các bản ghi hiệu năng truyền vào callback của `PerformanceObserver`, hỗ trợ các phương thức lọc tối ưu sẵn trong lõi C++: `getEntries()`, `getEntriesByType(type)`, và `getEntriesByName(name, type)`.
- **Kiểm soát Super-App (Container Control)**: Cho phép lọc chính xác loại metric cần phân tích (như chỉ trích xuất các bản ghi 'paint' hoặc 'resource') mà không cần cấp phát thêm các mảng mảng phụ hoặc lặp qua các bản ghi không liên quan trong JavaScript.
- **Quy tắc kiểm duyệt Store (Store Review Rule)**: Bộ thu thập số liệu phải ưu tiên dùng `getEntriesByType()` thay vì duyệt lọc thủ công mảng `getEntries()` để tiết kiệm chu kỳ xử lý của CPU.

#### 10. PerformanceEntry.toJSON(): Serialized Metric Export & Native Bridge Telemetry Marshaling
- **ID**: `STANDARDS-W3C-PERFENTRY-TOJSON`
- **Category**: `performance-and-runtime-observability`
- **Evidence Level**: `official-specification`
- **URL**: [https://developer.mozilla.org/en-US/docs/Web/API/PerformanceEntry/toJSON](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceEntry/toJSON)
- **Quy định kỹ thuật (Spec Fact)**: Phương thức `toJSON()` chuẩn hóa việc chuyển đổi đối tượng DOM PerformanceEntry thành một JavaScript object thuần (plain object) có thể tuần tự hóa, bao gồm đầy đủ các thuộc tính định danh, timestamp và thời lượng.
- **Kiểm soát Super-App (Container Control)**: Cung cấp định dạng tuần tự hóa an toàn để chuyển tiếp dữ liệu đo đạc qua JavaScript-Native Bridge hoặc gửi đi qua `navigator.sendBeacon()` mà không lo ngại lỗi circular reference hay prototype pollution.
- **Quy tắc kiểm duyệt Store (Store Review Rule)**: Các thư viện ghi log chẩn đoán trong mini-app phải sử dụng `entry.toJSON()` khi đóng gói báo cáo hiệu năng để đảm bảo tính toàn vẹn của payload.

---

### Nhóm 3: W3C CSS Containment Module Level 1 & Strict Layout Sandboxing

#### 11. W3C CSS Containment Module Level 1: Subtree Reflow Isolation & Layout Decoupling
- **ID**: `STANDARDS-W3C-CSS-CONTAIN-SPECIFICATION`
- **Category**: `layout-containment-and-rendering-isolation`
- **Evidence Level**: `official-specification`
- **URL**: [https://drafts.csswg.org/css-contain-1/](https://drafts.csswg.org/css-contain-1/)
- **Quy định kỹ thuật (Spec Fact)**: W3C CSS Containment Level 1 quy định cơ chế cô lập các nhánh DOM con khỏi cây layout tổng thể. Định nghĩa thuộc tính `contain` với các loại cô lập cốt lõi: Layout (thay đổi kích thước bên trong không lan truyền ra ngoài), Paint (nội dung không vẽ tràn ra ngoài biên và tạo stacking context), và Size (kích thước phần tử được tính độc lập với con cái).
- **Kiểm soát Super-App (Container Control)**: Là công cụ kiến trúc sống còn khi nhúng các widget, thẻ thông tin mini-app của bên thứ ba vào màn hình chính của Super-App, ngăn ngừa hoàn toàn hiện tượng layout shift làm giật lag giao diện feed của Super-App.
- **Quy tắc kiểm duyệt Store (Store Review Rule)**: Thẻ widget mini-app nhúng trên trang chủ Super-App bắt buộc phải áp dụng layout và paint containment để đảm bảo hiệu năng cuộn trang đạt 60fps.

#### 12. CSS contain Property: Granular Containment Values & Micro-Frontend Sandboxing
- **ID**: `STANDARDS-W3C-CSS-CONTAIN-PROPERTY`
- **Category**: `layout-containment-and-rendering-isolation`
- **Evidence Level**: `official-specification`
- **URL**: [https://developer.mozilla.org/en-US/docs/Web/CSS/contain](https://developer.mozilla.org/en-US/docs/Web/CSS/contain)
- **Quy định kỹ thuật (Spec Fact)**: Quy định cú pháp và các giá trị của thuộc tính `contain`: `strict` (áp dụng đồng thời size, layout, paint, style), `content` (áp dụng layout, paint, style), `layout`, `paint`, `size`.
- **Kiểm soát Super-App (Container Control)**: Thiết lập hàng rào phòng thủ trực quan declarative bao quanh component của mini-app, ngăn chặn việc vẽ đè lên các thành phần điều hướng hệ thống (system navigation bar, capsule button) hoặc chiếm quyền hiển thị giao diện cha.
- **Quy tắc kiểm duyệt Store (Store Review Rule)**: Các thành phần dialog, popup, hoặc floating widget của mini-app phải khai báo `contain: layout paint` để đảm bảo không rò rỉ pixel ra ngoài vùng chứa được cấp quyền.

#### 13. CSS contain-intrinsic-width Property: Declarative Inline Sizing & Layout Stability
- **ID**: `STANDARDS-W3C-CSS-CONTAIN-INTRINSIC-WIDTH`
- **Category**: `layout-containment-and-rendering-isolation`
- **Evidence Level**: `official-specification`
- **URL**: [https://developer.mozilla.org/en-US/docs/Web/CSS/contain-intrinsic-width](https://developer.mozilla.org/en-US/docs/Web/CSS/contain-intrinsic-width)
- **Quy định kỹ thuật (Spec Fact)**: Quy định chiều rộng dự phòng rõ ràng khi phần tử chịu tác động của size containment hoặc `content-visibility: auto`. Hỗ trợ cú pháp `auto <length>`, cho phép trình duyệt ghi nhớ kích thước đã render thực tế để sử dụng lại khi phần tử cuộn ra ngoài màn hình.
- **Kiểm soát Super-App (Container Control)**: Giữ ổn định kích thước chiều ngang cho các danh sách cuộn ngang (horizontal carousel), thẻ sản phẩm và banner trong mini-app, loại bỏ hoàn toàn hiện tượng nhảy giao diện khi người dùng vuốt nhanh.
- **Quy tắc kiểm duyệt Store (Store Review Rule)**: Các khối giao diện sử dụng size containment bắt buộc phải khai báo `contain-intrinsic-width` để vượt qua bài kiểm tra chỉ số CLS (< 0.05).

#### 14. CSS contain-intrinsic-height Property: Block-Axis Geometry Preservation for Infinite Feeds
- **ID**: `STANDARDS-W3C-CSS-CONTAIN-INTRINSIC-HEIGHT`
- **Category**: `layout-containment-and-rendering-isolation`
- **Evidence Level**: `official-specification`
- **URL**: [https://developer.mozilla.org/en-US/docs/Web/CSS/contain-intrinsic-height](https://developer.mozilla.org/en-US/docs/Web/CSS/contain-intrinsic-height)
- **Quy định kỹ thuật (Spec Fact)**: Xác định chiều cao dự phòng theo trục dọc (block axis) khi phần tử được un-render bởi `content-visibility: auto`, giúp trình duyệt tính toán chính xác tổng chiều cao thanh cuộn của trang mà không cần render DOM thực tế.
- **Kiểm soát Super-App (Container Control)**: Cực kỳ quan trọng đối với các luồng danh sách sản phẩm thương mại điện tử, bảng tin tin tức và lịch sử giao dịch dài hàng trăm mục trong mini-app: giảm tải tới 80% bộ nhớ DOM mà vẫn duy trì thanh cuộn chính xác.
- **Quy tắc kiểm duyệt Store (Store Review Rule)**: Các trang danh sách render hơn 50 phần tử động phải kết hợp `content-visibility: auto` và `contain-intrinsic-height` để tránh lỗi tràn bộ nhớ trên các dòng điện thoại phổ thông.

#### 15. CSS content-visibility: hidden vs auto: Off-Screen Subtree Memory De-allocation & Lazy Layout
- **ID**: `STANDARDS-W3C-CSS-CONTENT-VISIBILITY-OFFSCREEN-DEALLOCATION`
- **Category**: `layout-containment-and-rendering-isolation`
- **Evidence Level**: `official-specification`
- **URL**: [https://developer.mozilla.org/en-US/docs/Web/CSS/content-visibility](https://developer.mozilla.org/en-US/docs/Web/CSS/content-visibility)
- **Quy định kỹ thuật (Spec Fact)**: Quy định thuộc tính `content-visibility`: `auto` tự động bỏ qua layout và paint cho các nhánh DOM ngoài màn hình; `hidden` bỏ qua render hoàn toàn bất kể vị trí (như `display: none` nhưng vẫn giữ nguyên trạng thái DOM cache và cấu trúc cây).
- **Kiểm soát Super-App (Container Control)**: Các tab bar và màn hình con trong mini-app có thể áp dụng `content-visibility: hidden` cho các tab đang ẩn thay vì hủy bỏ DOM hoàn toàn, giúp chuyển tab tức thì dưới 10ms mà không tốn GPU render ngầm.
- **Quy tắc kiểm duyệt Store (Store Review Rule)**: Mini-app có cấu trúc nhiều tab hoặc menu trượt phức tạp được khuyến nghị dùng `content-visibility: hidden/auto` để tối ưu hóa thời gian phản hồi giao diện.

---

## 3. Ma trận Tổng hợp Tiêu chuẩn Store Vòng 107

| Lĩnh vực | Tiêu chuẩn cốt lõi | API / Cơ chế chủ đạo | Giá trị kiểm soát Super-App | Cấp độ kiểm duyệt Store |
|---|---|---|---|---|
| **Dữ liệu & Encoding** | WHATWG Encoding Living Standard | `TextEncoder`, `TextDecoder`, `TextEncoderStream`, `TextDecoderStream` | Chuẩn hóa UTF-8 tuyệt đối; xử lý luồng nhị phân và tài liệu lớn qua TransformStream với phản áp, RAM < 64KB | Bắt buộc (Mandatory) cho gói mã nguồn & xuất/nhập tệp lớn |
| **Quan sát & Telemetry** | W3C Performance Timeline Level 2 | `PerformanceObserver`, `supportedEntryTypes`, `takeRecords()`, `toJSON()` | Đo lường hiệu năng client độ phân giải cao; giải phóng đệm đồng bộ khi đóng app; đóng gói JSON chuẩn hóa qua bridge | Bắt buộc (Mandatory) cho SDK giám sát hiệu năng |
| **Bảo vệ Giao diện & Layout** | W3C CSS Containment Module Level 1 | `contain: strict/layout/paint`, `contain-intrinsic-size`, `content-visibility` | Cô lập nhánh layout con; loại bỏ reflow lan truyền ra feed Super-App; giảm 80% RAM DOM cho danh sách dài | Bắt buộc (Mandatory) cho Widget nhúng & Danh sách động |

---

## 4. Tác động tới Kiến trúc & Roadmap Kế tiếp

1. **Chuẩn hóa Bộ chuyển tiếp Dữ liệu (Streaming Data Bridge)**: Super-App container tích hợp sẵn các pipeline `TextDecoderStream` vào các kênh WebSocket và AI Chat streaming để đảm bảo hiệu năng tối đa.
2. **Nâng cấp Hệ thống Đo đạc APM Container**: Tận dụng `takeRecords()` và `toJSON()` của `PerformanceObserver` để đảm bảo 100% các phiên tương tác ngắn trong mini-app được ghi nhận đầy đủ chỉ số Core Web Vitals (FCP, LCP, INP, CLS).
3. **Chính sách Kiểm soát Thẻ nhúng (Embedded Widget Governance)**: Bổ sung quy định kiểm duyệt tự động đối với các mini-app cung cấp widget nhúng trang chủ: bắt buộc khai báo `contain: layout paint` và `contain-intrinsic-height`.
