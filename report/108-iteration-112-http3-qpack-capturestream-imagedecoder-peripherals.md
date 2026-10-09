# 108: Chuẩn Hóa HTTP/3 Wire Transport (RFC 9114), QPACK (RFC 9204), Multimedia CaptureStream, WebCodecs ImageDecoder & Hardware Peripheral Sandboxing (Web Serial, WebHID, WebUSB)

## 1. Tổng Quan Nghiên Cứu Iteration 112
Báo cáo chuyên đề Milestone 112 tiếp tục hoàn thiện khung tiêu chuẩn kỹ thuật cốt lõi cho mini app store trong super app, tập trung vào 4 trụ cột kiến trúc nền tảng:
1. **HTTP/3 & QPACK Wire Transport Architecture (IETF RFC 9114 & RFC 9204)**: Chuẩn hóa tầng mạng truyền dẫn thế hệ mới qua UDP/QUIC, triệt tiêu hiện tượng nghẽn đầu hàng (Head-of-Line Blocking) trên các mạng di động không ổn định, thiết lập cơ chế nén tiêu đề QPACK với stream đơn hướng encoder/decoder và kiểm soát trần bảng động để bảo vệ API Gateway.
2. **Multimedia Display & Canvas Stream Capture Interfaces (captureStream & getDisplayMedia)**: Cơ chế trích xuất luồng MediaStream thời gian thực từ HTMLMediaElement và HTMLCanvasElement phục vụ hội thảo tương tác, trình chiếu màn hình và ghi hình chữ ký số; tích hợp ranh giới bảo mật EME DRM, CORS cross-origin taint và Permissions Policy `display-capture`.
3. **WebCodecs ImageDecoder & Advanced Canvas 2D Graphic Compositing**: Giải mã ảnh động đa khung hình (GIF, animated WebP, AVIF) bất đồng bộ ngoài main thread, kiểm soát vòng lặp animation bằng siêu dữ liệu ImageTrack/ImageTrackList, áp dụng bộ lọc CSS phần cứng `CanvasRenderingContext2D.filter` và 26 chế độ hòa trộn Porter-Duff `globalCompositeOperation`.
4. **Direct Hardware Peripheral Sandboxing (Web Serial, WebHID & WebUSB)**: Giao tiếp phần cứng ngoại vi trực tiếp phục vụ kiosk POS bán lẻ, máy quét mã vạch và máy in nhiệt; áp dụng cơ chế xác thực cử chỉ người dùng bắt buộc (transient user activation), bộ lọc vendorId/productId/usagePage và danh sách chặn tuyệt đối (blocklist) thiết bị nhập liệu hệ thống (keyboard, mouse, security key).

---

## 2. Bảng Tổng Hợp Findings Chi Tiết (Iteration 112)

| ID | Tiêu Chuẩn / API | Phân Loại | Mức Minh Chứng | URL Xác Thực (HTTP 200) | Tóm Tắt Quy Chuẩn Cho Mini App Store |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `STANDARDS-IETF-RFC9114-HTTP3-WIRE-TRANSPORT-SANDBOXING` | IETF RFC 9114 (HTTP/3) | Network & Transport Sandboxing | Official Standard | `https://www.rfc-editor.org/rfc/rfc9114.html` | Ánh xạ HTTP trên nền QUIC, ghép kênh luồng độc lập loại bỏ Head-of-Line blocking, duy trì độ trễ tải mini app cực thấp trên mạng di động. |
| `STANDARDS-IETF-RFC9204-QPACK-FIELD-COMPRESSION-GOVERNANCE` | IETF RFC 9204 (QPACK) | Network & Transport Sandboxing | Official Standard | `https://www.rfc-editor.org/rfc/rfc9204.html` | Chuẩn hóa nén trường tiêu đề cho HTTP/3, giới hạn dung lượng bảng động (4096B) và số luồng nghẽn để phòng chống tấn công CRIME/BREACH tại gateway. |
| `STANDARDS-WHATWG-HTMLMEDIAELEMENT-CAPTURESTREAM-PIPELINES` | HTMLMediaElement.captureStream | Multimedia & Hardware Sandboxing | Official Standard | `https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/captureStream` | Trích xuất luồng MediaStream từ thẻ video/audio đang phát, tự động tắt hình/tiếng nếu gặp luồng EME DRM hoặc cross-origin không có CORS. |
| `STANDARDS-W3C-HTMLCANVASELEMENT-CAPTURESTREAM-GRAPHICS` | HTMLCanvasElement.captureStream | Multimedia & Hardware Sandboxing | Official Standard | `https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/captureStream` | Trích xuất video stream từ canvas 2D/WebGL với trần tốc độ khung hình (30fps), tự động ngắt khi mini app mất tiêu điểm để chống cạn kiệt pin GPU. |
| `STANDARDS-W3C-MEDIADEVICES-GETDISPLAYMEDIA-SCREEN-SANDBOXING` | MediaDevices.getDisplayMedia | Multimedia & Hardware Sandboxing | Official Standard | `https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getDisplayMedia` | Bật hộp thoại native chọn bề mặt chia sẻ màn hình/cửa sổ ứng dụng, kiểm soát bằng Permissions Policy display-capture và hiển thị thanh cảnh báo bảo mật. |
| `STANDARDS-W3C-WEBCODECS-IMAGEDECODER-OFFTHREAD-SANDBOXING` | WebCodecs ImageDecoder | Multimedia & Hardware Sandboxing | Official Standard | `https://developer.mozilla.org/en-US/docs/Web/API/ImageDecoder` | Giải mã ảnh động đa khung hình (GIF/WebP/AVIF) ngoài main thread, trả về VideoFrame GPU, loại bỏ nghẽn giao diện (INP) trên danh mục e-commerce. |
| `STANDARDS-W3C-WEBCODECS-IMAGETRACKLIST-CONTAINER-INTROSPECTION` | WebCodecs ImageTrackList | Multimedia & Hardware Sandboxing | Official Standard | `https://developer.mozilla.org/en-US/docs/Web/API/ImageTrackList` | Kiểm tra danh sách track hình ảnh trong container đa khung hình, quản lý chọn track hiển thị và bắt buộc xử lý ngoại lệ khi container rỗng. |
| `STANDARDS-W3C-WEBCODECS-IMAGETRACK-ANIMATION-TELEMETRY` | WebCodecs ImageTrack | Multimedia & Hardware Sandboxing | Official Standard | `https://developer.mozilla.org/en-US/docs/Web/API/ImageTrack` | Đo lường số khung hình (frameCount) và chu kỳ lặp (repetitionCount); tự động giới hạn tối đa 3 vòng lặp cho animation nền để tiết kiệm năng lượng. |
| `STANDARDS-W3C-CANVAS2D-FILTER-HARDWARE-SHADERS` | CanvasRenderingContext2D.filter | Styling & Layout Sandboxing | Official Standard | `https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/filter` | Áp dụng hiệu ứng bộ lọc CSS trực tiếp lên canvas bằng GPU shader, giới hạn bán kính blur <= 20px để tránh suy giảm fill-rate trên màn hình Retina. |
| `STANDARDS-W3C-CANVAS2D-GLOBALCOMPOSITEOPERATION-PORTER-DUFF` | Canvas 2D globalCompositeOperation | Styling & Layout Sandboxing | Official Standard | `https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/globalCompositeOperation` | Kiểm soát 26 chế độ hòa trộn và che phủ Porter-Duff, bắt buộc cô lập trạng thái vẽ bằng ctx.save() / ctx.restore() tránh tràn hiệu ứng ra card cha. |
| `STANDARDS-WICG-SERIALPORT-POS-HARDWARE-SANDBOXING` | Web Serial SerialPort | Multimedia & Hardware Sandboxing | Official Standard | `https://developer.mozilla.org/en-US/docs/Web/API/SerialPort` | Truyền nhận dữ liệu nhị phân hai chiều qua cổng nối tiếp (RS-232/USB-CDC) cho máy in hóa đơn và cân điện tử POS; quản lý đóng cổng khi unmount. |
| `STANDARDS-WICG-SERIAL-REQUESTPORT-USER-MEDIATION` | navigator.serial.requestPort | Multimedia & Hardware Sandboxing | Official Standard | `https://developer.mozilla.org/en-US/docs/Web/API/Serial/requestPort` | Hộp thoại chọn thiết bị serial yêu cầu cử chỉ người dùng, lọc chính xác theo USB vendorId/productId đã khai báo trong hồ sơ doanh nghiệp. |
| `STANDARDS-WICG-HIDDEVICE-REPORT-SANDBOXING` | WebHID HIDDevice | Multimedia & Hardware Sandboxing | Official Standard | `https://developer.mozilla.org/en-US/docs/Web/API/HIDDevice` | Trao đổi input/output report với thiết bị ngoại vi chuyên dụng; áp dụng danh sách chặn bàn phím/chuột hệ thống và khóa bảo mật FIDO. |
| `STANDARDS-WICG-HID-REQUESTDEVICE-FILTER-SANDBOXING` | navigator.hid.requestDevice | Multimedia & Hardware Sandboxing | Official Standard | `https://developer.mozilla.org/en-US/docs/Web/API/HID/requestDevice` | Yêu cầu quyền truy cập thiết bị HID với bộ lọc usagePage nghiêm ngặt (vd: 0x8C máy quét mã vạch), cấm hoàn toàn quét wildcard thiết bị. |
| `STANDARDS-WICG-USBDEVICE-INTERFACE-CLAIMING-GOVERNANCE` | WebUSB USBDevice | Multimedia & Hardware Sandboxing | Official Standard | `https://developer.mozilla.org/en-US/docs/Web/API/USBDevice` | Quản lý kết nối USB mức thấp (bulk/interrupt transfer) cho thiết bị công nghiệp; cấm chiếm dụng các interface bảo vệ chuẩn (audio, mass storage). |

---

## 3. Kiến Trúc Kỹ Thuật & Khuyến Nghị Vận Hành

### 3.1. Tầng Mạng HTTP/3 & QPACK Cho Super-App Gateway
- **Loại bỏ Head-of-Line Blocking**: Khi thiết bị di động chuyển vùng giữa 4G/5G và Wi-Fi, việc mất gói tin ngẫu nhiên trên giao thức TCP truyền thống sẽ làm gián đoạn toàn bộ các yêu cầu HTTP/2 song song. HTTP/3 giải quyết triệt để vấn đề này nhờ cơ chế quản lý luồng độc lập của QUIC.
- **Bảo Vệ Bộ Nhớ Đệm QPACK**: Gateway phân phối mini app phải thiết lập cấu hình `SETTINGS_QPACK_MAX_TABLE_CAPACITY = 4096` bytes và `SETTINGS_QPACK_BLOCKED_STREAMS = 16` để ngăn chặn tấn công từ chối dịch vụ (DoS) do các mini app độc hại cố tình gửi các chuỗi tiêu đề phân mảnh.

### 3.2. Quản Trị Trích Xuất Luồng Truyền Thông (CaptureStream) & Màn Hình
- **DRM & Bản Quyền Nội Dung**: `HTMLMediaElement.captureStream()` tự động bị ngắt (ném ngoại lệ `SecurityError` hoặc hiển thị khung đen) nếu nội dung được bảo vệ bởi Encrypted Media Extensions (EME). Mini app không được phép dùng API này để sao chép nội dung bản quyền.
- **Tiết Kiệm Năng Lượng**: `HTMLCanvasElement.captureStream()` phải được cấu hình giới hạn 30fps đối với đồ họa động thông thường. Khi mini app chuyển sang trạng thái ẩn (`visibilityState === 'hidden'`), hệ thống container sẽ tự động tạm ngưng (`track.enabled = false`) để bảo toàn pin thiết bị.

### 3.3. Giải Mã Ảnh Đồ Họa Ngoài Luồng Chính (ImageDecoder)
- **Cắt Giảm Triệt Để Độ Trễ Giao Diện (INP)**: Việc giải mã ảnh GIF dung lượng lớn bằng thẻ `<img>` tiêu chuẩn gây khóa main thread nhiều trăm mili-giây. Với `ImageDecoder`, mini app phân tích luồng nhị phân trong Web Worker và nhận về các đối tượng `VideoFrame` trực tiếp trên VRAM GPU.
- **Chống Tràn Bộ Nhớ Đệm**: Bắt buộc gọi `ImageDecoder.close()` khi component bị hủy (unmount), thu hồi toàn bộ frame buffers.

### 3.4. Quản Lý Thiết Bị Ngoại Vi Trực Tiếp (Serial / HID / USB) Cho Môi Trường Doanh Nghiệp & POS
- **Nguyên Tắc Đặc Quyền Tối Thiểu (PoLP)**: Chỉ cho phép các mini app thuộc phân nhóm doanh nghiệp (Enterprise / B2B) đã qua kiểm duyệt xác minh danh tính (KYBC) truy cập các API Web Serial, WebHID và WebUSB.
- **Chặn Nghe Lén Phím Bấm (Keystroke Injection Defense)**: Bộ lọc WebHID của super app áp dụng danh sách đen phần cứng nghiêm ngặt, từ chối mọi yêu cầu kết nối tới các thiết bị thuộc lớp bàn phím (`Usage Page 0x01, Usage 0x06`) hoặc khóa bảo mật FIDO (`Usage Page 0xF1D0`).
