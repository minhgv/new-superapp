# Chuyên Đề 46: Bộ Chuẩn W3C MiniApp Working Group, Web Bluetooth & Ngoại Vi Web NFC / Generic Sensors

**Mã tài liệu**: `M50-W3C-MINIAPP-BLUETOOTH-NFC-SENSORS`
**Thuộc dự án**: Nghiên Cứu Chuẩn Mini App Store Cho Super App (Deli Deep)
**Thời điểm hoàn thành**: Tháng 10/2026 (Milestone Iteration 50)
**Số lượng findings mới**: 15 findings chuẩn hóa (100% normative standards & official platform specs)
**Trạng thái kiểm tra**: Hoàn tất xác thực HTTP 200, schema JSONL hợp lệ, zero secret leakage, URL deduplicated.
**Tích lũy toàn dự án**: 701 findings chuẩn hóa.

---

## 1. Bối Cảnh & Mục Tiêu Nghiên Cứu

Tròn 50 vòng nghiên cứu chuyên sâu đã xây dựng nền móng toàn diện cho kiến trúc Mini App Store: từ an toàn luồng thực thi, xác thực danh tính FAPI, bảo mật chuỗi cung ứng SLSA/CycloneDX, đến tối ưu hóa giao diện và cách ly đồ họa. Tại cột mốc Milestone 50, nghiên cứu hội tụ vào 3 trụ cột mang tính định hình trực tiếp chuẩn mini app hiện đại và kết nối thế giới vật lý:

1. **Bộ chuẩn W3C MiniApp Working Group Standards Suite (W3C Recommendation & Candidate Recommendation)**:
   - Các hệ sinh thái mini app ban đầu (WeChat, Alipay, Baidu, QuickApp) phát triển với định dạng tệp cấu hình và vòng đời phân mảnh. W3C MiniApps Working Group đã chính thức chuẩn hóa thành bộ tiêu chuẩn mở toàn cầu:
     - **W3C MiniApp Manifest (Candidate Recommendation)**: Cấu trúc JSON chuẩn hóa siêu dữ liệu, phân trang routing, định kiểu cửa sổ window, xoay màn hình và tuyên bố quyền hạn.
     - **W3C MiniApp Packaging (Candidate Recommendation)**: Cấu trúc container đóng gói định dạng zip, phân tách gói phụ (subpackage splitting), tải động theo nhu cầu (on-demand loading) và chữ ký bảo mật.
     - **W3C MiniApp Lifecycle (Working Group Note)**: Hai máy trạng thái vòng đời tách biệt: Vòng đời Ứng dụng (Application Lifecycle: launch, show, hide, error) và Vòng đời Trang (Page Lifecycle: load, show, ready, hide, unload, pullDownRefresh).
     - **W3C MiniApp Addressing (Recommendation)**: Lược đồ định danh chuẩn hóa `miniapp://`, phân giải đường dẫn thứ bậc, truyền tham số query an toàn và gọi liên ứng dụng (inter-app invocation).
     - **W3C MiniApp Widget Requirements (Note)**: Yêu cầu kiến trúc và cơ chế ràng buộc dữ liệu khai báo (declarative data binding) cho các micro-cards nhúng trực tiếp vào trang chủ super app.

2. **Web Bluetooth API & Kết Nối Ngoại Vi Tầm Ngắn (Short-Range Peripherals)**:
   - Nhu cầu kết nối thiết bị IoT y tế (đo huyết áp, nhịp tim), thiết bị bán lẻ POS (máy in hóa đơn, máy quét mã vạch BLE) và khóa thông minh trong mini app.
   - Chuẩn hóa quy trình dò tìm thiết bị có người dùng xác nhận (`navigator.bluetooth.requestDevice`), duyệt cây dịch vụ GATT (`BluetoothRemoteGATTServer`), đọc/ghi thuộc tính (read/write characteristic) và đăng ký luồng thông báo streaming (`startNotifications`).
   - Kiến trúc an toàn Web Bluetooth: Danh sách cấm tuyệt đối **GATT Blocklist** của W3C/Chromium (ngăn chặn chèn phím giả mạo HID, FIDO bypass) và cơ chế phân quyền, tự động ngắt kết nối nền để tiết kiệm pin trên Super App.

3. **Web NFC (NDEFReader) & Khung Cảm Biến W3C Generic Sensor / Geolocation Sensor**:
   - Giao tiếp trường gần NFC cho thanh toán chạm, thẻ xe buýt, thẻ thành viên số và thẻ thông minh POS bán lẻ thông qua giao diện `NDEFReader`, bắt sự kiện `reading`, bóc tách cấu trúc bản ghi `NDEFRecord`.
   - Kiểm soát ghi dữ liệu NFC (`ndef.write`) và bảo vệ chống ghi đè dữ liệu thẻ vật lý.
   - Khung cảm biến hướng đối tượng **W3C Generic Sensor API**: Máy trạng thái 4 bước (idle, activating, activated, errored), thuật toán kẹp tần số lấy mẫu (sampling rate clamping <=60Hz) chống rò rỉ âm học gõ phím, và tự động đình chỉ khi ẩn ứng dụng.
   - **W3C Geolocation Sensor API**: Chuẩn hóa cảm biến định vị GNSS thế hệ mới, thay thế API callback cũ bằng Promise (`GeolocationSensor.read()`) và phân tầng độ chính xác (high vs low) để bảo vệ quyền riêng tư người dùng.

---

## 2. Chuẩn Hóa W3C MiniApp Working Group Standards Suite

```
+-----------------------------------------------------------------------------------+
|                        W3C MiniApp Standards Suite Architecture                   |
+-----------------------------------------------------------------------------------+
|                                                                                   |
|  +-----------------------------------------------------------------------------+  |
|  | [W3C MiniApp Manifest] (CR)                                                 |  |
|  | - app_id, name, version, pages: ["pages/index/index", "pages/pay/pay"]       |  |
|  | - window: { navigationBarTitleText, backgroundColor, orientation }          |  |
|  | - req_permissions: ["bluetooth", "nfc", "location"]                         |  |
|  +-----------------------------------------------------------------------------+  |
|                                       |                                           |
|  +------------------------------------+----------------------------------------+  |
|  |                                    |                                        |  |
|  v                                    v                                        v  |
|  +------------------------+  +--------------------------+  +-------------------+  |
|  | [W3C Packaging] (CR)   |  | [W3C Lifecycle] (Note)   |  | [W3C Addressing]  |  |
|  | - Zip container        |  | Dual-thread state machine|  | (Recommendation)  |  |
|  | - manifest.json at root|  | - App: launch/show/hide  |  | - miniapp:// scheme| |
|  | - Main package (<=4MB) |  | - Page: load/ready/unload|  | - Path resolution |  |
|  | - Subpackages (<=20MB) |  | - Background freeze      |  | - Inter-app calls |  |
|  +------------------------+  +--------------------------+  +-------------------+  |
|                                       |                                           |
|  +------------------------------------+----------------------------------------+  |
|  |                                                                                |
|  v                                                                                |
|  +-----------------------------------------------------------------------------+  |
|  | [W3C MiniApp Widget Requirements] (Note)                                    |  |
|  | - Instant render (<=50ms) via declarative JSON state                        |  |
|  | - Zero unconstrained JS / Sandbox isolation                                 |  |
|  | - Embedded glanceable cards on Super App dashboard                          |  |
|  +-----------------------------------------------------------------------------+  |
+-----------------------------------------------------------------------------------+
```

### 2.1. W3C MiniApp Manifest (Candidate Recommendation)
- **Chuẩn tham chiếu**: `https://www.w3.org/TR/miniapp-manifest/`
- **Cấu trúc dữ liệu chuẩn hóa**:
  - `app_id`: Định danh duy nhất toàn cầu cho mini app trong store catalog.
  - `name`: Tên ứng dụng hiển thị với người dùng cuối và công cụ tìm kiếm.
  - `version`: Đối tượng phiên bản `{ versionName: "1.2.0", versionCode: 120 }`.
  - `pages`: Mảng danh sách đường dẫn các trang, phần tử đầu tiên mặc định là trang chủ khởi động (`entry page`).
  - `window`: Cấu hình thanh tiêu đề, màu nền và chế độ hiển thị hệ thống (`navigationBarTitleText`, `navigationBarBackgroundColor`, `navigationBarTextStyle: "black" | "white"`).
  - `req_permissions`: Khai báo trước danh sách quyền truy cập phần cứng cần cấp quyền lúc cài đặt hoặc lúc gọi hàm.
- **Quy chuẩn kiểm duyệt Store**: Cổng ingest của store chạy bộ kiểm tra JSON schema đối chiếu với đặc tả W3C. Nếu tệp manifest thiếu mảng `pages` hoặc các tệp trang không tồn tại trong gói nén zip, hệ thống tự động từ chối xuất bản.

### 2.2. W3C MiniApp Packaging & Phân Tách Gói Phụ (Subpackage Splitting)
- **Chuẩn tham chiếu**: `https://www.w3.org/TR/miniapp-packaging/`
- **Quy cách đóng gói container**:
  - Mini app được đóng gói dưới định dạng chuẩn nén ZIP, chứa tệp `manifest.json` tại thư mục gốc, cùng mã nguồn logic (`app.js`), giao diện (`app.css`) và các thư mục trang.
  - **Cơ chế phân tách gói phụ**: Nhằm đảm bảo thời gian khởi động lạnh (cold start) < 300ms qua mạng di động, kiến trúc chia làm:
    1. *Gói chính (Main Package)*: Chứa mã nguồn khung lõi, thanh tab bar và trang khởi động ban đầu. Giới hạn dung lượng nén tối đa 4MB.
    2. *Các gói phụ (Subpackages)*: Khai báo qua trường `subpackages: [ { root: "sub_pay", pages: [...] } ]`. Gói phụ chỉ được tải nạp khi người dùng thực hiện điều hướng sang phân hệ đó. Tổng dung lượng toàn bộ gói không vượt quá 20MB.
- **Quy chuẩn Store**: Tự động xác thực chữ ký số SHA-256 đối chiếu với cặp khóa công khai của nhà phát triển đăng ký trên cổng đối tác.

### 2.3. W3C MiniApp Lifecycle: Hai Tầng Máy Trạng Thái Ứng Dụng & Trang
- **Chuẩn tham chiếu**: `https://www.w3.org/TR/miniapp-lifecycle/`
- **Mô hình luồng kép (Dual-Thread Model)**:
  - *Luồng logic (Logical Thread)*: Chạy worker JavaScript thực thi nghiệp vụ, gọi API cầu nối native.
  - *Luồng kết xuất (Render Thread)*: Quản lý WebView hoặc giao diện native hiển thị cây DOM.
- **Máy trạng thái vòng đời**:
  - **Vòng đời Ứng dụng (App Level)**:
    - `onLaunch`: Kích hoạt khi super app nạp gói ứng dụng vào bộ nhớ.
    - `onShow`: Mini app xuất hiện trên màn hình chính (foreground).
    - `onHide`: Người dùng chuyển sang ứng dụng khác hoặc thu nhỏ super app (background).
    - `onError`: Bắt ngoại lệ JavaScript chưa được xử lý trong luồng logic.
  - **Vòng đời Trang (Page Level)**:
    - `onLoad`: Khởi tạo trang với các tham số query từ luồng điều hướng.
    - `onShow`: Giao diện trang bắt đầu hiển thị.
    - `onReady`: Cây kết xuất DOM đã hoàn tất vẽ đầu tiên và sẵn sàng nhận tương tác.
    - `onHide`: Trang bị che khuất bởi trang mới đẩy vào ngăn xếp (navigation stack).
    - `onUnload`: Trang bị loại bỏ khỏi ngăn xếp (bấm quay lại hoặc đóng mini app).
    - `onPullDownRefresh`: Người dùng thực hiện cử chỉ vuốt kéo xuống để làm mới dữ liệu.
- **Quy chuẩn Store**: Khi mini app nhận sự kiện `onHide`, container Super App tự động đóng băng (freeze) luồng kết xuất và hạ mức ưu tiên luồng logic. Các mini app tiêu thụ CPU > 5% trong trạng thái ẩn quá 30 giây mà không có quyền chạy nền hợp lệ sẽ bị giám sát watchdog thu hồi bộ nhớ ngay lập tức.

### 2.4. W3C MiniApp Addressing & Định Danh `miniapp://`
- **Chuẩn tham chiếu**: `https://www.w3.org/TR/miniapp-addressing/`
- **Cú pháp định danh chuẩn**:
  - `miniapp://{miniapp-id}/{page-path}?{query}#{fragment}`
  - Trong đó `{miniapp-id}` đại diện cho mã định danh ứng dụng đã phê duyệt trên store catalog; `{page-path}` trỏ tới tệp trang đã đăng ký trong manifest.
- **Bảo mật điều hướng liên mini app**: Cổng điều hướng (Intent Router) của Super App kiểm tra quyền gọi ứng dụng (inter-app capability allowlist) trước khi chuyển hướng luồng dữ liệu, ngăn chặn tấn công giả mạo liên kết sâu (deep link hijacking).

### 2.5. W3C MiniApp Widget: Micro-Card Khai Báo Cho Trang Chủ Super App
- **Chuẩn tham chiếu**: `https://www.w3.org/TR/miniapp-widget-req/`
- **Đặc tính kỹ thuật micro-card**:
  - Hiển thị thông tin tức thì (glanceable) như: trạng thái cuốc xe đang di chuyển, số dư tài khoản ngân hàng, thông báo chuyển phát bưu kiện, thẻ tích điểm.
  - Kết xuất siêu tốc: Thời gian vẽ lần đầu <= 50ms nhờ cơ chế liên kết dữ liệu khai báo JSON thuần túy (declarative data binding), không khởi tạo WebView hoàn chỉnh.
  - Cách ly tuyệt đối: Widget không có quyền truy cập trực tiếp cảm biến phần cứng hay bộ nhớ máy khách. Tương tác chạm được đóng gói thành sự kiện intent chuyển tiếp lên vỏ máy Super App xử lý.

---

## 3. Web Bluetooth API & Quản Trị Ngoại Vi Tầm Ngắn

### 3.1. Dò Tìm Thiết Bị Có Người Dùng Xác Nhận (`requestDevice`)
- **Chuẩn tham chiếu**: `https://webbluetoothcg.github.io/web-bluetooth/`, `https://developer.mozilla.org/en-US/docs/Web/API/Web_Bluetooth_API`, `https://developer.chrome.com/docs/capabilities/bluetooth`
- **Nguyên lý thực thi**:
  - Kích hoạt thông qua cử chỉ người dùng trực tiếp (Transient User Activation: nhấp chuột/chạm tay).
  - Khai báo bộ lọc tường minh:
    ```javascript
    const device = await navigator.bluetooth.requestDevice({
      filters: [{ services: ['heart_rate'] }],
      optionalServices: ['battery_service']
    });
    ```
  - Trình duyệt/Super App hiển thị bảng chọn thiết bị native an toàn, ngăn chặn việc ứng dụng web tự ý quét lén không gian vật lý xung quanh người dùng.

### 3.2. Cây Dịch Vụ GATT Server & Khôi Phục Kết Nối Ngoại Vi
- **Chuẩn tham chiếu**: `https://developer.mozilla.org/en-US/docs/Web/API/BluetoothRemoteGATTServer`, `https://developer.mozilla.org/en-US/docs/Web/API/BluetoothDevice`
- **Quản lý phiên GATT**:
  - Khởi tạo kết nối qua `const server = await device.gatt.connect()`.
  - Bắt sự kiện ngắt kết nối vật lý qua `device.addEventListener('gattserverdisconnected', onDisconnect)` để thực hiện tái kết nối lặp lại với thuật toán trễ mũ (exponential backoff).
  - Tự động ngắt kết nối: Nếu mini app không phát sinh trao đổi dữ liệu qua BLE trong 3 phút, container Super App tự động ngắt phiên kết nối GATT để bảo vệ dung lượng pin thiết bị và ngoại vi.

### 3.3. Đọc, Ghi & Nhận Thông Báo Thuộc Tính GATT
- **Chuẩn tham chiếu**: `https://developer.mozilla.org/en-US/docs/Web/API/Web_Bluetooth_API`
- **Phân tách thao tác**:
  - Đọc dữ liệu: `const value = await characteristic.readValue()` trả về đối tượng `DataView`.
  - Ghi có phản hồi: `await characteristic.writeValueWithResponse(data)` bắt buộc áp dụng cho các lệnh giao dịch tài chính, khóa cửa, điều khiển y tế để đảm bảo thiết bị đã nhận trọn vẹn gói tin.
  - Ghi không phản hồi: `characteristic.writeValueWithoutResponse(data)` cho các tác vụ streaming tốc độ cao độ trễ thấp.
  - Luồng thông báo: Kích hoạt `await characteristic.startNotifications()` và lắng nghe sự kiện `characteristicvaluechanged`. Mini app bắt buộc phải gọi `stopNotifications()` khi hủy trang.

### 3.4. Danh Sách Đen GATT Blocklist & Chống Tấn Công Chèn Mã
- **Chuẩn tham chiếu**: Web Bluetooth Specification § 8 / Chromium GATT Blocklist (`https://developer.chrome.com/docs/capabilities/bluetooth`)
- **Cơ chế phòng thủ**:
  - Cấm tuyệt đối quyền truy cập vào các UUID dịch vụ nhạy cảm: Dịch vụ bàn phím/chuột Human Interface Device (HID - `0x1812`), Khóa bảo mật FIDO/U2F (`0xFFFD`), và điểm kiểm soát nhịp tim (`0x2A39`).
  - Nếu mini app cố tình yêu cầu các UUID này, runtime lập tức ném ngoại lệ `SecurityError` DOMException.
  - Cổng duyệt store quét mã nguồn tĩnh (AST) để phát hiện và đình chỉ ngay lập tức các mini app cố tình sử dụng ID tùy biến để lách luật hoặc tấn công thiết bị ngoại vi.

### 3.5. Trọng Tài Tài Nguyên Bluetooth Trong Super App
- **Chuẩn tham chiếu**: Super-App Peripheral Governance Framework
- **Quy tắc điều phối**:
  - Khi người dùng chuyển đổi giữa các mini app, container tự động đình chỉ quyền nhận thông báo nền của mini app trước đó.
  - Xử lý xung đột tài nguyên: Đối với các thiết bị ngoại vi dùng chung tại quầy bán lẻ (máy in POS, máy quét mã vạch), container native xây dựng hàng đợi tuần tự hóa các lệnh gửi nhận dữ liệu, loại bỏ hoàn toàn các lỗi xung đột phần cứng `GATT_BUSY` hoặc `GATT_ERROR`.

---

## 4. Web NFC & Khung Cảm Biến W3C Generic Sensor / Geolocation Sensor

### 4.1. Vòng Đời Quét NDEFReader & Quyền Hạn Web NFC
- **Chuẩn tham chiếu**: `https://w3c.github.io/web-nfc/`, `https://developer.mozilla.org/en-US/docs/Web/API/NDEFReader`, `https://developer.chrome.com/docs/capabilities/nfc`
- **Quy trình quét thẻ**:
  - Bắt buộc chạy trong Ngữ Cảnh An Toàn (Secure Context - HTTPS) và tài liệu đang có tiêu điểm (active document focus).
  - Khởi tạo trình đọc:
    ```javascript
    const ndef = new NDEFReader();
    const abortController = new AbortController();
    await ndef.scan({ signal: abortController.signal });
    ndef.addEventListener("reading", ({ message, serialNumber }) => {
      for (const record of message.records) {
        console.log(`Record type: ${record.recordType}`);
      }
    });
    ```
  - Cơ chế Permissions Policy: Gated bởi chỉ thị `Permissions-Policy: nfc=(self)`. Iframe nhúng không thể sử dụng NFC nếu không có sự ủy quyền rõ ràng từ tài liệu cha.

### 4.2. Giải Mã Bản Ghi NDEFRecord & Lọc An Toàn
- **Chuẩn tham chiếu**: `https://developer.mozilla.org/en-US/docs/Web/API/NDEFReader`
- **Cấu trúc trường chuẩn**:
  - `recordType`: Các loại bản ghi chuẩn hóa gồm `'text'`, `'url'`, `'mime'`, `'empty'`.
  - `data`: Trả về `DataView` chứa mảng byte thô. Việc giải mã chuỗi text phải tuân thủ thuộc tính `record.encoding` ('utf-8' hoặc 'utf-16') và nhãn ngôn ngữ `record.lang`.
  - **Nguyên tắc an ninh**: Dữ liệu URL trích xuất từ thẻ NFC phải được xác thực theo chính sách bảo mật nội dung (CSP) của mini app. Nghiêm cấm thực thi trực tiếp các đoạn mã script động hoặc gọi hàm `eval()` từ dữ liệu nhúng trong thẻ NFC.

### 4.3. Ghi Dữ Liệu Thẻ NFC & Chống Ghi Đè (Overwrite Safeguards)
- **Chuẩn tham chiếu**: `https://developer.chrome.com/docs/capabilities/nfc`
- **Thực thi ghi thẻ**:
  - Ghi dữ liệu được thực thi thông qua phương thức `ndef.write(message, { overwrite: false, signal })`.
  - Cờ `overwrite: false` đảm bảo rằng nếu thẻ vật lý đã có dữ liệu trước đó, thao tác ghi sẽ bị từ chối ngay lập tức để tránh làm mất mát thông tin quan trọng của người dùng.
  - Cổng kiểm duyệt store phân loại thao tác ghi thẻ NFC vào nhóm quyền hạn doanh nghiệp đặc quyền (Enterprise Privilege), yêu cầu hồ sơ đăng ký mục đích sử dụng rõ ràng (quản lý kho, cấp phát thẻ thành viên).

### 4.4. Khung Cảm Biến Đối Tượng W3C Generic Sensor API
- **Chuẩn tham chiếu**: `https://www.w3.org/TR/generic-sensor/`, `https://developer.mozilla.org/en-US/docs/Web/API/Sensor_APIs`
- **Máy trạng thái cảm biến (Sensor Lifecycle)**:
  - 4 trạng thái định danh: `idle` (chờ) -> `activating` (kết nối phần cứng) -> `activated` (đang phát dữ liệu `reading`) -> `errored` (lỗi kết nối hoặc mất quyền).
  - Thuật toán bảo vệ người dùng:
    - **Kẹp tần số (Sampling Clamping)**: Giới hạn tần số lấy mẫu tối đa <= 60Hz. Điều này ngăn chặn kẻ tấn công phân tích độ rung siêu nhỏ của cảm biến gia tốc (accelerometer) và con quay hồi chuyển (gyroscope) để tái tạo phím gõ mật khẩu của người dùng.
    - **Tự động ngưng khi ẩn**: Khi `document.visibilityState === "hidden"`, container lập tức ngắt toàn bộ luồng phát sự kiện cảm biến.

### 4.5. W3C Geolocation Sensor API: Định Vị GNSS Thế Hệ Mới
- **Chuẩn tham chiếu**: `https://w3c.github.io/geolocation-sensor/`
- **Cải tiến vượt bậc so với Geolocation cũ**:
  - Tích hợp hoàn toàn vào kiến trúc `Sensor`.
  - Cung cấp phương thức bất đồng bộ Promise tĩnh lấy tọa độ đơn lẻ:
    ```javascript
    const position = await GeolocationSensor.read({ accuracy: "high" });
    console.log(`Lat: ${position.latitude}, Lon: ${position.longitude}`);
    ```
  - Đo lường liên tục bằng cách khởi động cảm biến `const geo = new GeolocationSensor(); geo.start();`.
  - **Phân tầng độ chính xác tại Store**:
    - Mini app đặt xe và giao hàng: Được phép dùng tọa độ chính xác cao (`accuracy: "high"` - GPS vệ tinh) trong thời gian chuyến đi đang diễn ra.
    - Mini app thời tiết, mua sắm bán lẻ: Bị giới hạn ở tọa độ mờ (coarsened location - làm tròn bán kính 1km) để bảo vệ sự riêng tư về vị trí sinh hoạt của người dùng.

---

## 5. Bảng Đối Chiếu Ma Trận Kiểm Soát Ngoại Vi & MiniApp Store

| Nhóm Chuẩn | Tiêu Chuẩn Tham Chiếu | Phạm Vi Kiểm Soát Store & Container | Rủi Ro Triệt Tiêu |
|---|---|---|---|
| **W3C MiniApp Manifest** | W3C CR MiniApp Manifest | Linter kiểm tra schema JSON, xác thực định tuyến `pages`, kiểm soát `req_permissions`. | Lỗi điều hướng trang, crash giao diện, yêu cầu quyền hạn không khai báo. |
| **W3C MiniApp Packaging** | W3C CR MiniApp Packaging | Giới hạn dung lượng: gói chính <= 4MB, tổng gói <= 20MB. Kiểm tra chữ ký số SHA-256. | Thời gian tải ứng dụng quá lâu, giả mạo gói cài đặt trên đường truyền CDN. |
| **W3C MiniApp Lifecycle** | W3C Note MiniApp Lifecycle | Điều phối sự kiện luồng kép (logic/render), đóng băng ứng dụng nền sau 30 giây. | Rò rỉ tài nguyên, tiêu hao pin máy khách, xung đột luồng UI native. |
| **W3C MiniApp Addressing** | W3C REC MiniApp Addressing | Chuẩn hóa URI `miniapp://`, phân giải đường dẫn tài nguyên nội bộ, xác thực quyền gọi liên app. | Tấn công vượt thư mục (`../../`), đánh cắp luồng điều hướng deep-link. |
| **W3C MiniApp Widget** | W3C Note Widget Requirements | Thẻ micro-card kết xuất tức thì (<=50ms), ràng buộc dữ liệu khai báo, cô lập không chạy JS tự do. | Lag giật màn hình chính Super App, tiêu tốn RAM feed tổng thể. |
| **Web Bluetooth Discovery** | Web Bluetooth Specification | Yêu cầu cử chỉ người dùng, hiển thị bảng chọn native, kiểm duyệt UUID dịch vụ khai báo. | Quét trộm không gian phần cứng xung quanh người dùng, kết nối trái phép. |
| **Web Bluetooth GATT Blocklist** | Chromium GATT Blocklist / W3C | Chặn tuyệt đối UUID dịch vụ bàn phím HID (0x1812), khóa FIDO (0xFFFD), đo nhịp tim nhạy cảm. | Giả lập phím bấm độc hại từ xa, vượt qua xác thực hai yếu tố phần cứng. |
| **Web Bluetooth Governance** | Super-App Peripheral Framework | Tự động ngắt kết nối idle sau 3 phút, điều phối hàng đợi lệnh in POS/quét mã tránh xung đột. | Cạn kiệt pin ngoại vi, nghẽn bus truyền thông phần cứng của hệ thống. |
| **Web NFC Scan Lifecycle** | W3C Web NFC Working Draft | Bắt buộc Secure Context, cờ Permissions-Policy `nfc`, ngắt quét khi chuyển tab. | Đọc trộm thẻ không tiếp xúc khi người dùng không chú ý, rò rỉ bối cảnh iframe. |
| **Web NFC Payload Security** | W3C Web NFC § 6 NDEF Record | Lọc bản ghi URL theo CSP, giải mã text an toàn, cấm hoàn toàn thực thi mã script từ thẻ. | Chèn mã độc qua thẻ NFC (NFC XSS/Exploit), lừa đảo điều hướng trang web giả mạo. |
| **Web NFC Tag Writing** | Chrome Capabilities NFC | Bắt buộc cờ `overwrite: false`, giới hạn quyền doanh nghiệp, giám sát timeout ghi. | Xóa nhầm dữ liệu thẻ khách hàng, hỏng thẻ do rút thiết bị quá sớm. |
| **W3C Generic Sensor State** | W3C Generic Sensor API REC | Quản lý vòng đời 4 trạng thái, tự động đình chỉ khi ẩn, kẹp tần số lấy mẫu <= 60Hz. | Tấn công phân tích rung động đoán phím bấm (keystroke eavesdropping), hao pin. |
| **W3C Geolocation Sensor** | W3C Geolocation Sensor | Promise `read()`, phân tầng cấp quyền GPS độ chính xác cao vs tọa độ làm tròn 1km. | Theo dõi hành trình người dùng trái phép, lạm dụng dữ liệu vị trí nhạy cảm. |

---

## 6. Kết Luận & Hướng Nghiên Cứu Tiếp Theo

Milestone 50 đánh dấu bước tiến mang tính bước ngoặc của nghiên cứu Deli Deep:
- Chuẩn hóa toàn bộ hệ thống tiêu chuẩn quốc tế mở của **W3C MiniApps Working Group**, giúp kiến trúc Mini App Store không còn phụ thuộc vào các giải pháp đóng của từng nhà cung cấp riêng lẻ, mà hoàn toàn tương thích với các tiêu chuẩn Web toàn cầu.
- Thiết lập khung quản trị kết nối phần cứng ngoại vi và cảm biến vật lý (**Web Bluetooth, Web NFC, Generic Sensors, Geolocation Sensors**), mở rộng năng lực phục vụ của Super App sang các lĩnh vực IoT thông minh, y tế số, bán lẻ hiện đại và logistics.
- Đạt mốc **701 findings chuẩn hóa**, duy trì tính toàn vẹn 100% append-only của tệp bằng chứng và kích hoạt quy trình đồng bộ hóa định kỳ sang kho lưu trữ công khai `new-superapp`.
