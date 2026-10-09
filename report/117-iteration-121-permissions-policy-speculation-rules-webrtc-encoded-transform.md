# Báo Cáo Chuyên Đề: W3C Permissions Policy Level 1, WICG Speculation Rules Prerendering & W3C WebRTC Encoded Transform Sandboxing

**Mã phân loại báo cáo**: `REP-M121-PERMISSIONS-POLICY-SPECULATION-RULES-WEBRTC-ENCODED-TRANSFORM`
**Milestone / Iteration**: Iteration 121 (Tiến trình nghiên cứu chuẩn Mini App Store trên Super App)
**Ngày cập nhật**: 2026-10-09
**Tổng số findings chuẩn hóa lũy kế**: 1766 findings (Tăng trưởng +15 findings từ Milestone 120)
**Trạng thái kiểm tra nguồn**: 100% đạt HTTP 200 OK, định dạng RFC/W3C/WICG/MDN chuẩn mực.

---

## I. TỔNG QUAN ĐIỀU HÀNH & ĐỘNG LỰC NGHIÊN CỨU

Trong kiến trúc Super App hiện đại, việc tích hợp đồng thời hàng trăm mini app từ các nhà phát triển bên thứ ba (third-party developers) đòi hỏi một mô hình quản trị kép: vừa phải đảm bảo an toàn tuyệt đối, phân quyền chặt chẽ các tài nguyên phần cứng (Zero Trust Hardware Capability Sandboxing), vừa phải tối ưu hóa trải nghiệm mượt mà không độ trễ (Zero-Latency Navigation & Sub-50ms Time-to-Interactive), đồng thời cung cấp khả năng bảo mật nội dung truyền thông thời gian thực ở mức cao nhất (End-to-End Encryption - E2EE cho thoại và video).

Milestone 121 tập trung giải quyết 3 trụ cột kỹ thuật nền tảng:
1. **W3C Permissions Policy Level 1 & Container Capability Sandboxing**: Cơ chế chuẩn hóa chính thức của W3C và MDN thay thế Feature-Policy cũ, cung cấp cú pháp HTTP Header có cấu trúc (theo RFC 8941 Structured Field Values) và thuộc tính HTML `allow` trên phần tử `iframe` để phân quyền phân lớp, đóng gói và ủy quyền có điều kiện các năng lực nhạy cảm (camera, microphone, geolocation, payment instruments).
2. **WICG Speculation Rules API & Prerendering Performance Acceleration**: Chuẩn đặc tả WICG và tài liệu MDN về cơ chế nạp trước có tính toán (declarative prefetch & prerender). Thông qua thẻ `<script type="speculationrules">`, super app có thể nạp toàn bộ mini app tiếp theo vào một ngữ cảnh nền ẩn (invisible background context) và kích hoạt tức thì khi người dùng nhấn mở, triệt tiêu hoàn toàn màn hình trắng (blank screen).
3. **W3C WebRTC Encoded Transform & End-to-End Media Sandboxing**: Chuẩn W3C Encoded Transform (trước đây gọi là WebRTC Insertable Streams) và bộ giao diện MDN (`RTCRtpScriptTransform`, `RTCRtpScriptTransformer`, `RTCEncodedAudioFrame`, `RTCEncodedVideoFrame`). Tiêu chuẩn này cho phép chuyển các luồng dữ liệu thoại/video đã nén (compressed chunks) sang một luồng Worker riêng biệt (Dedicated Web Worker) để thực hiện mã hóa đầu cuối E2EE (như SFrame) hoặc kiểm toán nội dung mà không gây giật lag luồng giao diện chính (main thread).

---

## II. DANH MỤC 15 FINDINGS CHI TIẾT ĐÃ KIỂM ĐỊNH (HTTP 200 OK)

### 1. W3C Permissions Policy Level 1 & Container Capability Sandboxing

#### Finding 1.1: W3C Permissions Policy Level 1 Specification
- **ID**: `STANDARDS-W3C-PERMISSIONS-POLICY-1`
- **Phân loại**: `security-and-trust`
- **Mức độ bằng chứng**: `official_standard`
- **URL xác thực**: https://www.w3.org/TR/permissions-policy-1/
- **Nội dung chuẩn hóa**:
  1. Đặc tả chuẩn W3C Permissions Policy Level 1 quy định cơ chế khai báo có cấu trúc nhằm bật, tắt và ủy quyền có chọn lọc các API phần cứng và năng lực trình duyệt qua các browsing contexts và embedded iframes.
  2. Sử dụng cú pháp header `Permissions-Policy` dựa trên RFC 8941 Structured Field Values (Dictionary/List) để kiểm soát danh sách nguồn gốc (allowlists) như `'self'`, các domain cụ thể, `'*'`, hoặc `'()'`.
  3. Mọi tính năng nhạy cảm không được chỉ định tường minh cho iframe cross-origin sẽ mặc định kế thừa giá trị `'()'` (bị chặn hoàn toàn), bảo vệ thiết bị người dùng khỏi việc lạm dụng phần cứng trái phép.

#### Finding 1.2: MDN Permissions-Policy HTTP Header
- **ID**: `STANDARDS-MDN-PERMISSIONS-POLICY-HEADER`
- **Phân loại**: `security-and-trust`
- **Mức độ bằng chứng**: `official_documentation`
- **URL xác thực**: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Permissions-Policy
- **Nội dung chuẩn hóa**:
  1. Hướng dẫn kỹ thuật chính thức từ MDN về cú pháp và chỉ thị của header `Permissions-Policy`.
  2. Cho phép API Gateway của Super App tiêm trực tiếp header trên mọi gói phản hồi (HTTP response) phân phối mini app, thiết lập posture bảo mật mặc định nghiêm ngặt.
  3. Đảm bảo việc cô lập năng lực ở cấp độ gateway, ngăn chặn rủi ro leo thang đặc quyền (privilege escalation) khi mini app nhúng mã nguồn hoặc thư viện bên thứ ba.

#### Finding 1.3: MDN Permissions-Policy: camera Directive
- **ID**: `STANDARDS-MDN-PERMISSIONS-POLICY-CAMERA`
- **Phân loại**: `security-and-trust`
- **Mức độ bằng chứng**: `official_documentation`
- **URL xác thực**: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Permissions-Policy/camera
- **Nội dung chuẩn hóa**:
  1. Định nghĩa chi tiết chỉ thị `camera`, kiểm soát việc truy cập phần cứng quay phim và chụp ảnh qua `MediaDevices.getUserMedia()` hoặc `ImageCapture`.
  2. Mặc định chỉ cho phép origin `'self'`; khi bị từ chối bởi chính sách, lời gọi API lập tức trả về lỗi `NotAllowedError` mà không hiển thị hộp thoại xin quyền.
  3. Tiêu chuẩn xét duyệt mini app yêu cầu bắt buộc phải khai báo mục đích sử dụng camera trong manifest; runtime super app chỉ kích hoạt camera cho các luồng eKYC hoặc quét mã quang học đã được phê duyệt.

#### Finding 1.4: MDN Permissions-Policy: geolocation Directive
- **ID**: `STANDARDS-MDN-PERMISSIONS-POLICY-GEOLOCATION`
- **Phân loại**: `security-and-trust`
- **Mức độ bằng chứng**: `official_documentation`
- **URL xác thực**: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Permissions-Policy/geolocation
- **Nội dung chuẩn hóa**:
  1. Quản lý chỉ thị `geolocation`, cô lập quyền truy cập tọa độ địa lý vệ tinh (GNSS) và tam giác trạm phát sóng di động/Wi-Fi qua `Geolocation.getCurrentPosition()` và `watchPosition()`.
  2. Khi bị chặn, hàm gọi sẽ trả về mã lỗi `GeolocationPositionError.PERMISSION_DENIED` ngay tức thì.
  3. Các dịch vụ gọi xe, giao vận trong super app phải ràng buộc yêu cầu vị trí với cử chỉ tương tác chủ động của người dùng (user activation); super app sẽ tự động thu hồi quyền khi ứng dụng chuyển xuống chạy ngầm.

#### Finding 1.5: MDN HTMLIFrameElement.allow Attribute
- **ID**: `STANDARDS-MDN-HTMLIFRAMEELEMENT-ALLOW`
- **Phân loại**: `host-bridge-and-cross-context-messaging`
- **Mức độ bằng chứng**: `official_documentation`
- **URL xác thực**: https://developer.mozilla.org/en-US/docs/Web/API/HTMLIFrameElement/allow
- **Nội dung chuẩn hóa**:
  1. Thuộc tính `allow` trên phần tử `iframe` cho phép tài liệu gốc ủy quyền từng tính năng riêng biệt cho khung nhúng (ví dụ: `allow="payment; camera 'none'"`).
  2. Tuân thủ nguyên tắc giới hạn đặc quyền kế thừa: một iframe không thể nhận năng lực mà chính trang cha đã bị từ chối bởi HTTP header.
  3. Kết hợp với thuộc tính `sandbox` tạo nên mô hình phòng thủ theo chiều sâu (defense-in-depth), ngăn chặn iframe con gọi trái phép các native bridge methods của super app.

---

### 2. WICG Speculation Rules API & Prerendering Performance Acceleration

#### Finding 2.1: WICG Speculation Rules Specification
- **ID**: `STANDARDS-WICG-SPECULATION-RULES-SPEC`
- **Phân loại**: `performance-and-runtime-observability`
- **Mức độ bằng chứng**: `official_standard`
- **URL xác thực**: https://wicg.github.io/nav-speculation/speculation-rules.html
- **Nội dung chuẩn hóa**:
  1. Đặc tả WICG quy định cú pháp JSON khai báo quy tắc nạp trước (prefetch) và kết xuất trước (prerender) qua thẻ `<script type="speculationrules">`.
  2. Hỗ trợ quy tắc dựa trên danh sách URL (`list`) hoặc khớp mẫu tài liệu (`document` selector rules).
  3. Trang được kết xuất trước (prerendered) sẽ thực thi trong một ngữ cảnh nền ẩn (invisible background context), bị giới hạn mạng và không được tương tác với phần cứng nhạy cảm cho đến khi người dùng điều hướng vào.

#### Finding 2.2: MDN <script type="speculationrules">
- **ID**: `STANDARDS-MDN-SCRIPT-TYPE-SPECULATIONRULES`
- **Phân loại**: `performance-and-runtime-observability`
- **Mức độ bằng chứng**: `official_documentation`
- **URL xác thực**: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script/type/speculationrules
- **Nội dung chuẩn hóa**:
  1. Tài liệu MDN hướng dẫn chi tiết cấu trúc JSON của thẻ `<script type="speculationrules">`.
  2. Cung cấp các cấp độ mong muốn (`eagerness`: `'immediate'`, `'eager'`, `'moderate'`, `'conservative'`) tương ứng với hành vi di chuột, chạm giữ hoặc kích hoạt chủ động.
  3. SDK Super App chuẩn hóa giao diện khai báo này để triệt tiêu độ trễ chuyển màn hình mà không cần các native hacks phức tạp.

#### Finding 2.3: MDN Speculation Rules API
- **ID**: `STANDARDS-MDN-SPECULATION-RULES-API`
- **Phân loại**: `performance-and-runtime-observability`
- **Mức độ bằng chứng**: `official_documentation`
- **URL xác thực**: https://developer.mozilla.org/en-US/docs/Web/API/Speculation_Rules_API
- **Nội dung chuẩn hóa**:
  1. Hỗ trợ tiêm quy tắc động qua JavaScript (`document.createElement('script')`) trong các ứng dụng đơn trang (SPA).
  2. Tự động tiết chế (throttle) hoặc tạm dừng kết xuất trước khi phát hiện thiết bị pin yếu, bộ nhớ RAM thấp hoặc người dùng bật chế độ tiết kiệm dữ liệu (Data Saver).
  3. Khắc phục triệt để các lỗ hổng rò rỉ cookie/thông tin cá nhân của thẻ `<link rel="prerender">` cũ nhờ cơ chế phân vùng thông tin xác thực nghiêm ngặt.

#### Finding 2.4: MDN PerformanceNavigationTiming.activationStart
- **ID**: `STANDARDS-MDN-PERF-NAV-ACTIVATIONSTART`
- **Phân loại**: `performance-and-runtime-observability`
- **Mức độ bằng chứng**: `official_documentation`
- **URL xác thực**: https://developer.mozilla.org/en-US/docs/Web/API/PerformanceNavigationTiming/activationStart
- **Nội dung chuẩn hóa**:
  1. Thuộc tính `activationStart` trả về thời điểm (DOMHighResTimeStamp) mà trang prerender được người dùng kích hoạt đưa lên foreground.
  2. Đối với các điều hướng thông thường (không prerender), giá trị này luôn bằng 0.
  3. Giúp hệ thống đo lường hiệu năng của Super App tính toán chính xác các chỉ số Core Web Vitals (FCP, LCP) từ lúc người dùng thực sự nhìn thấy nội dung thay vì từ lúc khởi tạo ngầm.

#### Finding 2.5: MDN Document.prerendering Property
- **ID**: `STANDARDS-MDN-DOCUMENT-PRERENDERING`
- **Phân loại**: `Lifecycle, Power & Process Governance`
- **Mức độ bằng chứng**: `official_documentation`
- **URL xác thực**: https://developer.mozilla.org/en-US/docs/Web/API/Document/prerendering
- **Nội dung chuẩn hóa**:
  1. Thuộc tính `document.prerendering` trả về `true` khi trang đang được chuẩn bị ngầm và chuyển sang `false` khi trang được kích hoạt; phát sự kiện `prerenderingchange`.
  2. Quy chuẩn bắt buộc: Mọi tác vụ gây xáo trộn (phát âm thanh, ghi nhận log quảng cáo, mở popup thanh toán, quét sinh trắc học) PHẢI bị hoãn khi `document.prerendering` đang là `true`.
  3. Tiêu chí kiểm định mini app store yêu cầu hoãn các tác vụ tính toán nặng cho đến khi nhận được sự kiện `prerenderingchange`.

---

### 3. W3C WebRTC Encoded Transform & End-to-End Media Sandboxing

#### Finding 3.1: W3C WebRTC Encoded Transform Specification
- **ID**: `STANDARDS-W3C-WEBRTC-ENCODED-TRANSFORM`
- **Phân loại**: `Real-Time Networking & Transport`
- **Mức độ bằng chứng**: `official_standard`
- **URL xác thực**: https://www.w3.org/TR/webrtc-encoded-transform/
- **Nội dung chuẩn hóa**:
  1. Chuẩn W3C Encoded Transform quy định việc chuyển trực tiếp các gói khung media đã nén vào Dedicated Web Worker để mã hóa/giải mã hoặc xử lý bằng TransformStream.
  2. Hỗ trợ thiết lập mã hóa đầu cuối (E2EE) chuẩn SFrame trên trình duyệt, không phụ thuộc vào hạ tầng máy chủ trung gian.
  3. Đảm bảo luồng giao diện chính (main thread) hoàn toàn không bị đứng hình khi thực thi các phép biến đổi mã hóa phức tạp.

#### Finding 3.2: MDN RTCRtpScriptTransform Interface
- **ID**: `STANDARDS-MDN-RTCRTPSCRIPTTRANSFORM`
- **Phân loại**: `Real-Time Networking & Transport`
- **Mức độ bằng chứng**: `official_documentation`
- **URL xác thực**: https://developer.mozilla.org/en-US/docs/Web/API/RTCRtpScriptTransform
- **Nội dung chuẩn hóa**:
  1. Cung cấp hàm khởi tạo `new RTCRtpScriptTransform(worker, options, transferList)` để liên kết luồng Worker với `RTCRtpSender.transform` hoặc `RTCRtpReceiver.transform`.
  2. Đóng gói việc xử lý khung hình bên trong phạm vi Worker, cô lập mã độc bên thứ ba không thể trích xuất buffer video/audio thô.
  3. Áp dụng cho các mini app y tế từ xa (telehealth), ví điện tử và hội thoại bảo mật cao trong Super App.

#### Finding 3.3: MDN RTCRtpScriptTransformer Interface
- **ID**: `STANDARDS-MDN-RTCRTPSCRIPTTRANSFORMER`
- **Phân loại**: `Real-Time Networking & Transport`
- **Mức độ bằng chứng**: `official_documentation`
- **URL xác thực**: https://developer.mozilla.org/en-US/docs/Web/API/RTCRtpScriptTransformer
- **Nội dung chuẩn hóa**:
  1. Giao diện được tiếp cận trong DedicatedWorkerGlobalScope qua sự kiện `rtctransform`.
  2. Cung cấp các luồng `transformer.readable` và `transformer.writable`, hỗ trợ đường ống `pipeThrough` với cơ chế kiểm soát áp lực ngược (backpressure) tự động.
  3. Cách ly lỗi (fault isolation): Ngoại lệ không bắt được trong worker chỉ gây rớt khung hình cục bộ, không làm sập container mini app hay ứng dụng Super App mẹ.

#### Finding 3.4: MDN RTCEncodedAudioFrame Interface
- **ID**: `STANDARDS-MDN-RTCENCODEDAUDIOFRAME`
- **Phân loại**: `Real-Time Networking & Transport`
- **Mức độ bằng chứng**: `official_documentation`
- **URL xác thực**: https://developer.mozilla.org/en-US/docs/Web/API/RTCEncodedAudioFrame
- **Nội dung chuẩn hóa**:
  1. Đại diện cho khối âm thanh nén (như Opus), cung cấp buffer `data` (ArrayBuffer) và metadata (RTP timestamp, SSRC, payload type).
  2. Cho phép gắn watermark âm thanh hoặc thẻ xác thực mật mã trực tiếp vào từng gói âm thanh.
  3. Container Super App yêu cầu kiểm tra kích thước và cấu trúc buffer để ngăn chặn khai thác lỗi tràn bộ đệm (buffer overflow) trên bộ giải mã phía người nhận.

#### Finding 3.5: MDN RTCEncodedVideoFrame Interface
- **ID**: `STANDARDS-MDN-RTCENCODEDVIDEOFRAME`
- **Phân loại**: `Real-Time Networking & Transport`
- **Mức độ bằng chứng**: `official_documentation`
- **URL xác thực**: https://developer.mozilla.org/en-US/docs/Web/API/RTCEncodedVideoFrame
- **Nội dung chuẩn hóa**:
  1. Đại diện cho khung hình video nén (H.264, VP8/VP9, AV1), phân biệt rõ khung hình khóa (`type: 'key'`) và khung hình vi phân (`type: 'delta'`).
  2. Cho phép bảo vệ phần đầu gói tin hoặc mã hóa toàn phần payload dữ liệu video.
  3. Yêu cầu bảo toàn thứ tự frame sequence và cấu trúc NAL unit header để giữ tính ổn định đồng bộ hình ảnh khi mạng truyền thông gặp chập chờn hoặc mất gói.

---

## III. BẢNG MA TRẬN ĐỐI CHIẾU TIÊU CHUẨN & ÁP DỤNG TRONG SUPER APP

| Lĩnh vực | Tiêu chuẩn cốt lõi | Cơ chế kỹ thuật | Trách nhiệm Super App Host | Yêu cầu đối với Mini App |
|---|---|---|---|---|
| **Phân quyền Phần cứng** | W3C Permissions Policy 1 | HTTP Header `Permissions-Policy` & `allow` attribute | Tiêm header giới hạn; chỉ mở quyền khi manifest hợp lệ | Khai báo quyền rõ ràng trong manifest; xử lý lỗi `NotAllowedError` |
| **Tăng tốc Điều hướng** | WICG Speculation Rules | `<script type="speculationrules">`, `activationStart` | Tự động kích hoạt nạp trước dựa trên viewport/scroll | Hoãn các tác vụ gây hiệu ứng phụ khi `document.prerendering == true` |
| **Bảo mật Truyền thông** | W3C WebRTC Encoded Transform | `RTCRtpScriptTransform`, Worker TransformStream | Cung cấp luồng Worker an toàn; giám sát bộ nhớ worker | Triển khai E2EE chuẩn mực; không can thiệp sâu làm hỏng cấu trúc frame |

---

## IV. KẾT LUẬN & ĐỊNH HƯỚNG BƯỚC TIẾP THEO

Việc chuẩn hóa Milestone 121 đã hoàn thiện các mắt xích trọng yếu:
- Kiểm soát phần cứng ngoại vi và cảm biến nhạy cảm bằng **W3C Permissions Policy Level 1**.
- Tối ưu hóa chuyển trang tức thì đạt cấp độ gốc (native-like) bằng **WICG Speculation Rules & Prerendering**.
- Đảm bảo an toàn truyền thông thời gian thực và y tế số với **W3C WebRTC Encoded Transform & DedicatedWorker Sandboxing**.

Toàn bộ 15 findings đã được tích hợp append-only vào kho dữ liệu chuẩn `state/findings.jsonl`, nâng tổng số chuẩn mực xác thực lên **1766 findings**.
