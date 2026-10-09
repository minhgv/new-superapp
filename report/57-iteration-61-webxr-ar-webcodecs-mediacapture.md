# Chuyên đề 57: WebXR AR Session Governance, WebCodecs VideoFrame Hardware Pipelines & W3C Media Capture Transform (Breakout Box)

## 1. Bối cảnh & Mục tiêu Kỹ thuật

Trong các siêu ứng dụng (Super-Apps) hiện đại, nhu cầu tích hợp các tính năng đa phương tiện thế hệ mới ngày càng trở nên sống còn:
1. **Thương mại điện tử & Tương tác không gian (AR E-Commerce & Retail)**: Trải nghiệm ướm thử trang phục, mỹ phẩm ảo, mô phỏng đồ nội thất theo kích thước thực tế trong không gian người dùng đòi hỏi chuẩn kết nối AR hiệu năng cao và an toàn.
2. **Xử lý luồng Video phần cứng độ trễ thấp (Hardware-Accelerated Zero-Copy Video Processing)**: Live-commerce, phát sóng trực tiếp, biên tập video ngắn đòi hỏi quyền truy cập giải mã/mã hóa phần cứng trực tiếp từ SoC mà không gây nghẽn tiến trình UI hoặc làm nổ bộ nhớ RAM điện thoại.
3. **Xử lý luồng camera/microphone theo thời gian thực (Real-Time Streams Breakout Box & Computer Vision)**: eKYC nhận diện khuôn mặt chống giả mạo liveness, quét mã vạch tốc độ cao, xóa phông nền AI trong cuộc gọi video đòi hỏi cơ chế tách luồng stream trực tiếp sang Web Worker và mã hóa đầu-cuối (E2EE) trước khi gửi qua mạng.

Chuyên đề này chuẩn hóa 3 trụ cột kỹ thuật nền tảng:
- **W3C WebXR Device API & AR Module**: Quản trị vòng đời phiên AR hòa trộn (immersive-ar), raycasting hit-testing lên bề mặt vật lý, neo tọa độ không gian (anchors), và kiểm soát ngân sách nhiệt (thermal)/tốc độ khung hình (framerate).
- **W3C WebCodecs API**: Kiến trúc VideoFrame zero-copy ánh xạ trực tiếp phần cứng GPU, cơ chế giải phóng bộ nhớ bắt buộc `videoFrame.close()` ngăn rò rỉ RAM/VRAM gây crash OOM, giải mã bất đồng bộ qua VideoDecoder/AudioDecoder, và cô lập xử lý sang Dedicated Web Worker nhằm bảo đảm chỉ số INP < 200 ms.
- **W3C Media Capture Transform (Breakout Box) & WebRTC Streams**: Chuyển đổi luồng camera trực tiếp thành WHATWG ReadableStream qua MediaStreamTrackProcessor, tái tạo luồng tổng hợp qua MediaStreamTrackGenerator, ghép nối pipeline thị giác máy tính trong Web Worker qua TransformStream, chèn mã hóa đầu-cuối WebRTC E2EE qua RTCRtpScriptTransform, và cơ chế thu hồi quyền phần cứng tức thì khi mini-app vào background.

---

## 2. Chuẩn W3C WebXR Device API & Quản trị Phiên Augmented Reality (AR)

### 2.1. Vòng đời Phiên Immersive-AR & Bắt tay Kích hoạt Tương tác (User Gesture)
- **Truy vấn năng lực phần cứng**: Mini-app gọi `navigator.xr.isSessionSupported('immersive-ar')` để kiểm tra khả năng hỗ trợ AR hòa trộn quang học/camera passthrough của thiết bị.
- **Ràng buộc kích hoạt người dùng**: Lệnh `navigator.xr.requestSession('immersive-ar', ...)` bắt buộc phải được kích hoạt trực tiếp từ cử chỉ người dùng (transient user activation / click event). Nếu không có cử chỉ, trình duyệt native ném ngoại lệ `SecurityError`.
- **Chính sách cấp quyền (Permissions Policy)**: Container siêu ứng dụng bắt buộc kiểm soát tính năng qua chỉ thị `xr-spatial-tracking`. Thuộc tính iframe nhúng mini-app phải khai báo tường minh:
  ```html
  <iframe src="https://store.superapp.vn/miniapp/ar-fitting/"
          allow="xr-spatial-tracking; camera"></iframe>
  ```
- **Cô lập phiên đơn (Single Active Session)**: Khung chứa siêu ứng dụng thực thi chính sách chỉ cho phép 1 phiên AR hoạt động duy nhất trên toàn hệ thống. Ngay khi ứng dụng siêu app bị chuyển sang background hoặc màn hình bị khóa, container tự động kích hoạt `session.end()` để ngắt cảm biến và camera.

### 2.2. WebXR Hit-Testing & Neo Tọa độ Không gian (Spatial Anchors)
- **Bắn tia nhận diện bề mặt (Raycasting Plane Detection)**: Mini-app thương lượng tính năng `hit-test` trong `optionalFeatures` khi khởi tạo phiên, tạo `XRHitTestSource` từ không gian camera (`viewerSpace`). Mỗi khung hình render, lệnh `frame.getHitTestResults(hitTestSource)` trả về danh sách các điểm va chạm với bề mặt vật lý thực tế.
- **Bảo mật dữ liệu topo phòng (Topological Mesh Privacy)**: WebXR Hit Test API cô lập tuyệt đối dữ liệu đám mây điểm (point cloud) thô và lưới 3D chi tiết của căn phòng người dùng. Mini-app chỉ nhận được tọa độ giao điểm vector 3D và góc xoay bề mặt (surface normal quaternion), ngăn chặn phần mềm gián điệp quét dựng lại cấu trúc tư gia người dùng.
- **Neo tọa độ ổn định (XRAnchor)**: Để tránh mô hình 3D bị trôi lệch vị trí khi người dùng di chuyển camera, mini-app gọi `hitResult.createAnchor()`. Hệ thống native (ARKit trên iOS hoặc ARCore trên Android) liên tục tính toán bù trừ SLAM cho `XRAnchor.anchorSpace`. Mọi anchor bắt buộc bị hủy giải phóng khỏi RAM khi phiên AR kết thúc.

### 2.3. Đầu vào Không gian (Spatial Input), Phản hồi Haptic & Cân bằng Khung hình
- **Phân loại nguồn tương tác (XRInputSource)**: Hệ thống ánh xạ cử chỉ người dùng qua `targetRayMode`: `screen` (chạm cảm ứng trên màn hình điện thoại), `tracked-pointer` (tay cầm 6DoF phát tia laser), hoặc `gaze` (hướng nhìn kết hợp nút bấm).
- **Kiểm soát rung phản hồi (Haptic Pulse Governance)**: Tương tác rung xúc giác qua `inputSource.gamepad.hapticActuators.pulse(value, duration)` bị giới hạn thời gian rung tối đa 1.000 ms và duty-cycle tối đa 50% để tránh quá nhiệt và cạn pin.
- **Ước lượng ánh sáng phòng (Lighting Estimation)**: Cảm biến `XRLightProbe` tính toán hệ số cầu điều hòa (Spherical Harmonics) để đổ bóng đồ vật ảo ăn khớp ánh sáng thực tế. Để bảo mật, bản đồ phản chiếu khối (cubemap) bị lượng tử hóa độ phân giải thấp, nghiêm cấm phản chiếu khuôn mặt hoặc màn hình máy tính của người dùng vào môi trường 3D.
- **Ngân sách nhiệt & Tần số quét động (Dynamic Viewport Scaling)**: Phiên AR được đồng bộ với W3C Compute Pressure API. Khi trạng thái nhiệt độ thiết bị chạm mức `serious` hoặc `critical`, container siêu ứng dụng tự động yêu cầu hạ tần số quét xuống 30 fps và kích hoạt `XRWebGLLayer.requestViewportScale(0.75)` để giảm phân giải đồ họa, duy trì tỷ lệ rớt khung hình (frame drop) không vượt quá 15%.

---

## 3. Chuẩn W3C WebCodecs: Kiến trúc VideoFrame Phần cứng & Chống Rò rỉ RAM

### 3.1. Đối tượng VideoFrame Zero-Copy & Ánh xạ Bộ nhớ GPU
- **Truy cập phần cứng trực tiếp**: Thay thế các giải pháp giải mã phần mềm ngốn CPU bằng JavaScript Wasm (như ffmpeg.js), W3C WebCodecs cung cấp quyền truy cập trực tiếp bộ giải mã/mã hóa phần cứng tích hợp trên SoC điện thoại (Snapdragon, Apple Silicon, MediaTek).
- **Cấu trúc VideoFrame**: Đối tượng `VideoFrame` bao bọc trực tiếp bộ đệm đồ họa gốc của hệ điều hành (Android `GraphicBuffer`/`HardwareBuffer`, Apple iOS `CVPixelBuffer`/`IOSurface`). Đối tượng mang thông tin định dạng điểm ảnh (`NV12`, `I420`, `RGBA`), kích thước pixel, tỷ lệ hiển thị và timestamp microsecond.
- **Zero-Copy GPU Upload**: VideoFrame có thể được tải trực tiếp lên WebGL hoặc WebGPU texture thông qua `gl.texImage2D(..., videoFrame)` hoặc `device.queue.copyExternalImageToTexture({ source: videoFrame }, ...)` hoàn toàn không thông qua sao chép bộ nhớ trung gian trên CPU, đạt hiệu năng đồ họa tối đa.

### 3.2. Cơ chế Giải phóng Bộ nhớ Bắt buộc (`videoFrame.close()`) & Phòng ngừa Crash OOM
- **Mù bộ nhớ ở tầng JavaScript Garbage Collection**: JavaScript GC chỉ nhận biết dung lượng con trỏ đối tượng trên JS heap (vài chục byte), hoàn toàn không nhận biết được bộ nhớ VRAM / shared memory native khổng lồ mà VideoFrame đang chiếm giữ (mỗi frame 4K NV12 chiếm ~12 MB VRAM). Nếu phó mặc cho GC dọn dẹp, bộ nhớ đồ họa của điện thoại sẽ cạn kiệt trong vòng vài giây xử lý video, dẫn tới việc hệ điều hành buộc dừng tiến trình (Low Memory Killer / OOM crash).
- **Nghiêm ngặt `videoFrame.close()`**: Đặc tả WebCodecs quy định mọi đối tượng VideoFrame bắt buộc phải được đóng tường minh qua `videoFrame.close()`. Ngay sau khi gọi `.close()`, bộ đệm phần cứng native được hoàn trả ngay lập tức về bộ nhớ hệ thống.
- **Cổng kiểm duyệt tĩnh & Giám sát Runtime**:
  - Quy trình CI/CD Store quét mã nguồn mini-app, bắt buộc mọi vòng lặp xử lý VideoFrame phải đặt trong cấu trúc `try { ... } finally { frame.close(); }` hoặc pipeline stream có hook giải phóng.
  - Watchdog runtime của container đếm số lượng VideoFrame mở: nếu mini-app tích tụ quá 30 frames chưa giải phóng (~150 MB VRAM), container tự động kích hoạt lệnh dọn dẹp cưỡng chế và ghi nhận lỗi độ tin cậy.

### 3.3. VideoDecoder, AudioDecoder & Cô lập Tuyệt đối sang Dedicated Web Worker
- **Bắt tay cấu hình Codec**: Mini-app thăm dò `VideoDecoder.isConfigSupported(config)` để kiểm tra năng lực giải mã phần cứng cho các định dạng hiện đại (H.264, VP9, AV1, HEVC). Tham số chuỗi codec tuân thủ cú pháp RFC 6381 (ví dụ `'avc1.42001E'`).
- **Xử lý gói nén EncodedVideoChunk**: Các gói nén được nạp qua `decoder.decode(chunk)`. Luồng giải mã giám sát áp lực hàng đợi thông qua `decoder.decodeQueueSize`: nếu hàng đợi tăng cao, ứng dụng chủ động bỏ qua các delta frame không quan trọng để duy trì độ trễ thời gian thực.
- **AudioDecoder & Đồng bộ Trục Thời gian A/V**: `AudioDecoder` giải mã các gói nén (Opus, AAC) thành các đối tượng `AudioData` chứa mẫu PCM âm thanh tuyến tính. Khung chứa áp dụng bộ lọc đồng bộ timestamp giữa AudioData và VideoFrame với độ lệch tối đa cho phép trong ngưỡng ±40 ms.
- **Triệt tiêu nghẽn luồng UI (0 ms INP Degradation)**: Việc chạy giải mã và vẽ video trên luồng chính (Main UI Thread) làm tê liệt trình xử lý sự kiện, dẫn tới chỉ số Interaction to Next Paint (INP) tăng vọt vượt ngưỡng 500 ms. Do đó, toàn bộ kiến trúc giải mã WebCodecs và render OffscreenCanvas bắt buộc phải được đẩy hoàn toàn sang Dedicated Web Worker, bảo đảm luồng UI hoàn toàn mượt mà.

---

## 4. W3C Media Capture Transform (Breakout Box) & Luồng Xử lý Thời gian thực

### 4.1. MediaStreamTrackProcessor: Hóa giải Luồng Camera thành ReadableStream
- **Khái niệm Breakout Box**: Cho phép tách nhỏ luồng media trực tiếp từ camera hoặc microphone (MediaStreamTrack) thành các gói dữ liệu thô nối tiếp nhau theo chuẩn WHATWG Streams API.
- **Khởi tạo Processor**: `new MediaStreamTrackProcessor({ track: videoTrack, maxBufferSize: 2 })`. Thuộc tính `processor.readable` là một `ReadableStream` cung cấp trực tiếp các đối tượng `VideoFrame` (hoặc `AudioData`).
- **Kiểm soát hàng đợi chống phình bộ nhớ**: Thuộc tính `maxBufferSize` (chuẩn hóa ở mức 1 đến 3 frames) bảo đảm khi tác vụ xử lý thị giác máy tính phía sau bị chậm, các khung hình cũ nhất sẽ tự động bị loại bỏ (drop frame), duy trì luồng hình ảnh luôn sát với thời gian thực và không gây đầy RAM.

### 4.2. MediaStreamTrackGenerator: Tổng hợp Luồng MediaStreamTrack từ WritableStream
- **Bộ tạo luồng tổng hợp**: `new MediaStreamTrackGenerator({ kind: 'video' })`. Kế thừa trực tiếp từ `MediaStreamTrack`, đối tượng này cung cấp thuộc tính `generator.writable` là một WHATWG `WritableStream`.
- **Ghi luồng video đã xử lý**: Sau khi xử lý (ví dụ: làm mờ hậu cảnh, chèn watermark bảo mật, lọc nhiễu âm thanh), mini-app ghi trực tiếp `VideoFrame` hoặc `AudioData` vào writer của generator.
- **Hiệu năng vượt trội so với HTMLCanvasElement**: Thay vì phải vẽ lên thẻ canvas rồi gọi `canvas.captureStream()` gây hao tổn tài nguyên rasterization và tụt pin, luồng pipeline sử dụng `MediaStreamTrackGenerator` đạt hiệu suất cao hơn tới 4 lần, tiết kiệm tối đa điện năng trên thiết bị di động.

### 4.3. Pipeline TransformStream trong Web Worker & Thị giác Máy tính (AI / CV)
- **Chuỗi nối dòng Stream**: Pipeline hoàn chỉnh được ghép nối liền mạch:
  ```javascript
  processor.readable
    .pipeThrough(transformStream)
    .pipeTo(generator.writable);
  ```
- **Cô lập sang Web Worker**: Cả `processor.readable` và `generator.writable` đều là các đối tượng chuyển nhượng được (Transferable Objects). Toàn bộ pipeline được gửi sang Dedicated Web Worker qua `worker.postMessage(...)`.
- **Ứng dụng AI/ML & eKYC**: Bên trong phương thức `transform(frame, controller)` của TransformStream, mini-app nhúng thư viện Wasm OpenCV hoặc W3C WebNN để nhận diện khuôn mặt, đo độ sắc nét giấy tờ tùy thân eKYC. Sau khi phân tích, frame đầu vào bắt buộc được giải phóng ngay lập tức bằng `frame.close()`.

### 4.4. WebRTC Encoded Transform (RTCRtpScriptTransform) & Mã hóa Đầu-Cuối (E2EE)
- **Can thiệp tầng gói nén trước khi đóng gói mạng**: `RTCRtpScriptTransform` cho phép can thiệp trực tiếp vào các gói khung hình đã nén (`RTCEncodedVideoFrame`, `RTCEncodedAudioFrame`) ngay trước khi bộ phát RTP đóng gói gửi đi.
- **Mã hóa E2EE không cần tin tưởng Server trung gian**: Mini-app doanh nghiệp mã hóa tải trọng video bằng thuật toán WebCrypto AES-GCM với khóa mã hóa bí mật được chia sẻ riêng giữa các bên tham gia (sử dụng WebAuthn PRF hoặc Messaging Layer Security). Các máy chủ trung gian định tuyến luồng (SFU) chỉ đọc phần header RTP để phân phối gói tin mà hoàn toàn không thể giải mã nội dung hình ảnh/tiếng nói của người dùng.

### 4.5. Bảo mật Cảm biến Camera & Thu hồi Quyền Tức thì
- **Đèn báo quyền riêng tư phần cứng**: Khi luồng camera được xử lý qua MediaStreamTrackProcessor, hệ điều hành native bắt buộc phải duy trì đèn chỉ báo trạng thái (chấm xanh lá trên thanh trạng thái iOS/Android) xuyên suốt thời gian luồng hoạt động, kể cả khi quá trình xử lý diễn ra ngầm trong Web Worker.
- **Thu hồi tức thì khi chuyển Background**: Khi mini-app mất tiêu điểm (mất focus, người dùng chuyển app, hoặc màn hình khóa), container siêu ứng dụng tự động kích hoạt `track.stop()`. Ngay lập tức, `processor.readable` phát tín hiệu kết thúc stream và toàn bộ pipeline media bị giải phóng ngay lập tức, ngăn ngừa mọi hành vi quay lén trong nền.

---

## 5. Ma trận Kiểm soát Kỹ thuật Chuẩn Siêu ứng dụng (Audit Matrix)

| ID Kiểm soát | Hạng mục Chuẩn hóa | Yêu cầu Bắt buộc Siêu ứng dụng | Mức độ Tuân thủ |
| :--- | :--- | :--- | :--- |
| **CTL-XR-01** | Bắt tay Phiên WebXR AR | Yêu cầu cử chỉ người dùng (user activation) khi gọi `requestSession('immersive-ar')`; chỉ thị `xr-spatial-tracking` trong Permissions Policy. | **Bắt buộc (P0)** |
| **CTL-XR-02** | Bảo mật Topo Phòng Hit-Test | Chỉ cho phép truy xuất vector bề mặt va chạm và quaternion; cô lập đám mây điểm 3D thô; hủy toàn bộ XRAnchor khi đóng app. | **Bắt buộc (P0)** |
| **CTL-XR-03** | Ngân sách Nhiệt & Framerate | Giám sát trạng thái W3C Compute Pressure; tự động giảm độ phân giải `requestViewportScale` và hạ fps khi quá nhiệt; tỷ lệ rớt khung < 15%. | **Bắt buộc (P1)** |
| **CTL-CDC-01** | Quản trị Bộ đệm VideoFrame | Yêu cầu gọi `videoFrame.close()` tường minh trong khối `finally` cho 100% frame; watchdog container cưỡng chế đóng nếu tồn đọng > 30 frames. | **Bắt buộc (P0)** |
| **CTL-CDC-02** | Thăm dò Codec Phần cứng | Bắt buộc kiểm tra `VideoDecoder.isConfigSupported()` trước khi giải mã; hỗ trợ fallback codec tiêu chuẩn; xác thực cú pháp RFC 6381. | **Bắt buộc (P0)** |
| **CTL-CDC-03** | Đẩy Xử lý sang Web Worker | Toàn bộ luồng WebCodecs và canvas render chuyển sang Dedicated Web Worker; luồng chính duy trì chỉ số tương tác INP < 200 ms. | **Bắt buộc (P1)** |
| **CTL-STR-01** | Luồng MediaStreamTrackProcessor | Kẹp kích thước đệm `maxBufferSize` từ 1 đến 3 frames nhằm chống phình RAM và giảm độ trễ thực; tự động bỏ frame cũ khi tắc nghẽn. | **Bắt buộc (P0)** |
| **CTL-STR-02** | Luồng Tổng hợp Generator | Cấm sử dụng `canvas.captureStream()` cho video thời gian thực; bắt buộc sử dụng `MediaStreamTrackGenerator` để tiết kiệm điện năng và GPU. | **Bắt buộc (P1)** |
| **CTL-STR-03** | Ngắt Cảm biến Camera Background | Tự động kích hoạt `track.stop()` và đóng toàn bộ ReadableStream khi mini-app vào background; duy trì đèn báo privacy LED gốc của HĐH. | **Bắt buộc (P0)** |
| **CTL-STR-04** | Mã hóa E2EE WebRTC Transform | Hỗ trợ `RTCRtpScriptTransform` với chuẩn mã hóa AES-GCM cho mini-app bảo mật; cô lập khóa mã hóa trong WebCrypto/Secure Enclave. | **Khuyến nghị (P2)** |

---

## 6. Lộ trình Thực thi & Đóng gói Triển khai (Implementation Checklist)

1. **Giai đoạn Đánh giá Hồ sơ Nộp Store (Submission & Static Analysis Gate)**:
   - Quét AST mã nguồn mini-app tìm kiếm việc sử dụng các API camera, WebXR và WebCodecs.
   - Kiểm tra khai báo quyền trong `app.json`: yêu cầu giải trình mục đích sử dụng camera cho tính năng AR hoặc xử lý video.
   - Quét kiểm tra cú pháp việc giải phóng tài nguyên: kiểm tra sự hiện diện của lệnh `.close()` đối với mọi biến đại diện cho `VideoFrame` và `AudioData`.
2. **Giai đoạn Khởi chạy & Kiểm thử Tự động (Automated Container Test Run)**:
   - Chạy mini-app trong môi trường giả lập headless (WebDriver BiDi) kích hoạt luồng camera ảo.
   - Theo dõi biểu đồ tiêu thụ bộ nhớ VRAM và RAM gốc của hệ điều hành trong 3 phút xử lý liên tục. Bác bỏ mini-app nếu phát hiện rò rỉ bộ nhớ tuyến tính dốc đứng.
   - Thử nghiệm chuyển ứng dụng vào trạng thái nền (background) và xác minh luồng camera bị cắt đứt hoàn toàn trong vòng < 500 ms.
3. **Giai đoạn Vận hành Runtime Siêu ứng dụng (Host Runtime Enforcement)**:
   - Tiêm lớp vỏ bọc bảo vệ (Bridge Shim) xung quanh các đối tượng `VideoFrame` và `MediaStreamTrackProcessor` để thu thập telemetry vi phạm.
   - Hiển thị menu điều khiển capsule trên góc màn hình cho phép người dùng xem danh sách quyền cảm biến đang mở và chạm để thu hồi quyền camera/AR tức thì.
