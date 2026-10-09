# Chuyên đề 58: WICG Soft Navigations, Visual Viewport Dynamic Geometry & WHATWG Shared Workers Concurrency Orchestration (Iteration 62)

## 1. Bối cảnh & Tầm quan trọng trong Hệ sinh thái Super App

Trong kiến trúc Super App hiện đại, phần lớn các ứng dụng mini app được triển khai dưới dạng **Single Page Applications (SPA)**. Trong mô hình này, việc chuyển trang hoặc điều hướng màn hình diễn ra thông qua client-side routing (sử dụng History API hoặc Navigation API) kết hợp với các thao tác đột biến DOM (DOM mutations) bất đồng bộ mà không kích hoạt chu trình tải tài liệu HTML đa trang truyền thống. Mô hình này mang lại tốc độ phản hồi vượt trội nhưng đặt ra những thách thức nghiêm trọng về **đo lường hiệu năng (observability)** và **chất lượng trải nghiệm (QoS)**:
1. **Mù mờ về Telemetry & Core Web Vitals:** Trong các SPA truyền thống, Largest Contentful Paint (LCP) chỉ được ghi nhận duy nhất ở lần tải ban đầu; Cumulative Layout Shift (CLS) tích lũy liên tục qua nhiều giờ sử dụng; và Interaction to Next Paint (INP) phản ánh tương tác kém nhất của toàn bộ phiên làm việc, dẫn đến việc đánh giá sai lệch hiệu năng của các màn hình con.
2. **Biến dạng hình học di động & Che khuất giao diện:** Thiết bị di động sở hữu cơ chế bàn phím ảo (virtual software keyboard), cử chỉ thu phóng (pinch-to-zoom), và tai thỏ/dynamic island native. Nếu không phân định rõ ràng giữa **Layout Viewport** và **Visual Viewport**, các form nhập liệu, ô mã OTP, và nút thanh toán (call-to-action) sẽ bị bàn phím che khuất, hoặc các thanh điều hướng native (capsule button) bị xung đột với giao diện mini app.
3. **Lãng phí tài nguyên kết nối & Bất đồng bộ dữ liệu giữa các tab:** Khi mini app mở rộng thành kiến trúc đa màn hình (multi-screen POS, pop-up chat, floating payment window), việc mỗi tab mở một kết nối WebSocket/SSE độc lập gây cạn kiệt socket mạng, hao pin thiết bị và tạo ra các nguy cơ race condition trong giỏ hàng.

Chuyên đề 58 thiết lập bộ chuẩn kỹ thuật toàn diện tích hợp 3 trụ cột tiêu chuẩn web nền tảng:
- **WICG Soft Navigations:** Thuật toán heuristic nhận diện chuyển trang SPA, mở rộng W3C Performance Timeline với `PerformanceSoftNavigationTiming`, cô lập cửa sổ Core Web Vitals (INP/CLS/LCP) theo từng màn hình, và thiết lập ngưỡng kiểm duyệt chất lượng store review.
- **WICG Visual Viewport API:** Kiến trúc viewport kép tách rời Layout Viewport và Visual Viewport, cơ chế xử lý sự kiện `resize` khi bàn phím ảo xuất hiện, bất biến tỷ lệ thu phóng (scale-invariant overlays), đồng bộ compositor scroll, và tương thích an toàn với capsule menu native.
- **WHATWG Shared Workers:** Luồng thực thi ngầm cùng nguồn (same-origin background thread), giao thức bắt tay cổng kết nối `MessagePort`, gom cụm kết nối mạng duy nhất (singleton WebSocket/SSE multiplexing), điều phối đồng bộ bộ nhớ với W3C Web Locks API, và cơ chế giám sát tài nguyên (resource watchdog/memory caps).

---

## 2. WICG Soft Navigations & SPA Route Detection Heuristics

### 2.1. Thuật toán Heuristic Nhận diện Chuyển trang SPA
Đặc tả **WICG Soft Navigations** chuẩn hóa thuật toán gồm 3 điều kiện tuần tự để trình duyệt nhận diện một chuyển trang mềm:
1. **Khởi tạo từ tương tác người dùng (User Interaction Task):** Quá trình chuyển trang phải bắt nguồn từ một cử chỉ người dùng hợp lệ (nhấp chuột, chạm màn hình, gõ phím) được theo dõi qua cơ chế Browser Task Attribution.
2. **Đột biến URL (URL Mutation):** Địa chỉ trang web được cập nhật thông qua `history.pushState()`, `history.replaceState()`, hoặc WICG Navigation API.
3. **Đột biến DOM có ý nghĩa (Meaningful DOM Mutation):** Các phần tử nội dung mới được thêm, thay thế, hoặc hiển thị vào vùng nhìn (viewport) dưới dạng kết quả nhân quả trực tiếp từ chuỗi tác vụ bất đồng bộ trên.

Khi cả 3 điều kiện được thỏa mãn, runtime tổng hợp một sự kiện vòng đời điều hướng riêng biệt, đánh dấu điểm kết thúc của màn hình cũ và tái thiết lập các chỉ số hiệu năng cho màn hình mới.

### 2.2. Đo lường Hiệu năng với `PerformanceSoftNavigationTiming`
Đặc tả mở rộng **W3C Performance Timeline** bằng cách giới thiệu kiểu entry `'soft-navigation'`, được biểu diễn bởi giao diện `PerformanceSoftNavigationTiming`:
```javascript
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log(`[SoftNav] Target: ${entry.name}, Start: ${entry.startTime}ms, Duration: ${entry.duration}ms`);
  }
});
observer.observe({ type: 'soft-navigation', buffered: true });
```
- `name`: URL đích đã được chuẩn hóa của màn hình mới.
- `startTime`: Mốc thời gian `DOMHighResTimeStamp` khi cử chỉ người dùng bắt đầu.
- `duration`: Tổng thời gian từ khi tương tác xảy ra đến khi các đột biến DOM ổn định hoàn toàn.

### 2.3. Cô lập Cửa sổ Core Web Vitals (INP, CLS, LCP)
Chuẩn hóa Soft Navigations giải quyết triệt để sự suy thoái đo lường hiệu năng trong các phiên làm việc dài:
- **LCP tái định vị:** Mỗi màn hình mới xác định một ứng viên LCP riêng biệt, đo lường thời gian hiển thị phần tử nội dung lớn nhất của màn hình đó.
- **Phân đoạn CLS:** Layout shift chỉ được tính cho màn hình mà nó phát sinh, không tích lũy dồn dập vào toàn bộ phiên.
- **Cô lập INP:** Phản hồi tương tác được phân vùng độc lập, ngăn chặn việc một thao tác chậm ở màn hình phụ làm hỏng xếp hạng QoS của toàn bộ ứng dụng.

### 2.4. Tiêu chí Kiểm duyệt Store Review cho SPA QoS
- **Bắt buộc hỗ trợ định tuyến chuẩn:** Mini app bắt buộc phải cập nhật URL thông qua History/Navigation API khi chuyển màn hình; nghiêm cấm việc chỉ ẩn hiện CSS display mà không cập nhật trạng thái route.
- **Ngưỡng độ trễ chuyển trang (Latency Budget):** Độ trễ chuyển trang mềm (`duration`) không được vượt quá 1.000 ms trên cấu hình phần cứng di động tầm trung tiêu chuẩn.
- **Ngưỡng INP phân đoạn:** Duy trì `INP <= 200 ms` trên ít nhất 75% các lượt chuyển trang mềm.

---

## 3. WICG Visual Viewport API & Quản trị Hình học Di động Động

### 3.1. Kiến trúc Viewport Kép (Dual Viewport Architecture)
Trên thiết bị di động, không gian hiển thị được phân chia thành:
- **Layout Viewport:** Không gian tọa độ CSS cố định dùng để tính toán các phần tử `position: fixed` và đánh giá media queries.
- **Visual Viewport:** Vùng thực tế mà người dùng đang nhìn thấy trên màn hình, thay đổi liên tục khi người dùng phóng to (zoom) hoặc khi bàn phím ảo xuất hiện.

Đối tượng `window.visualViewport` cung cấp các thuộc tính đo lường thời gian thực:
- `width` và `height`: Kích thước vùng nhìn thấy thực tế (pixel CSS).
- `pageLeft` và `pageTop`: Tọa độ góc trên bên trái của Visual Viewport so với Layout Viewport.
- `scale`: Tỷ lệ phóng đại hiện tại của vùng nhìn.

### 3.2. Xử lý Thích ứng Bàn phím Ảo (Virtual Keyboard Adaptation)
Khi bàn phím ảo mở ra, `visualViewport.height` co lại đáng kể trong khi Layout Viewport giữ nguyên. Lắng nghe sự kiện `resize` trên `visualViewport` cho phép ứng dụng tự động điều chỉnh cuộn trang mượt mà:
```javascript
window.visualViewport.addEventListener('resize', () => {
  const keyboardHeight = window.innerHeight - window.visualViewport.height;
  if (keyboardHeight > 100) {
    // Cuộn form input hoặc nút submit vào vùng nhìn thấy
    activeElement.scrollIntoView({ behavior: 'smooth', block: 'center' });
  }
});
```
**Quy tắc kiểm duyệt:** Nghiêm cấm ghim các nút thanh toán hoặc submit bằng `position: fixed; bottom: 0` mà không bù trừ độ co rút của `visualViewport.height`, tránh tình trạng bàn phím che lấp nút bấm cốt lõi.

### 3.3. Bất biến Tỷ lệ Thu phóng (Scale-Invariant Overlays)
Khi người dùng phóng to trang web (`visualViewport.scale > 1.0`), các phần tử `position: fixed` thông thường sẽ bị phóng to và trôi ra khỏi màn hình. Áp dụng ma trận biến đổi CSS động giúp cố định các menu bảo mật và hộp thoại cảnh báo:
```css
/* Đảm bảo capsule hoặc modal luôn vừa vặn trong tầm nhìn người dùng */
transform: translate(calc(var(--vv-left) * 1px), calc(var(--vv-top) * 1px)) scale(calc(1 / var(--vv-scale)));
```
**Quy chuẩn Super App:** Thanh menu điều hướng an toàn (security capsule) của Super App luôn hiển thị ở tỷ lệ chuẩn 1:1, không bị biến dạng khi người dùng thực hiện thao tác zoom để đọc văn bản nhỏ.

### 3.4. Điều phối Inset Giao diện Native & An toàn Vùng nhìn (Safe Area Integration)
Tích hợp hình học Visual Viewport với các biến môi trường CSS Safe Area (`env(safe-area-inset-top)`, `env(safe-area-inset-bottom)`) và bounding rect của nút capsule native (`wx.getMenuButtonBoundingClientRect()` hoặc tương đương):
- Ngăn ngừa tình trạng các nút bấm của mini app va chạm hoặc nằm đè lên thanh điều hướng native.
- Tự động mở rộng hoặc co hẹp vùng render khi Super App mở rộng thanh thông báo hệ thống.

---

## 4. WHATWG Shared Workers & Điều phối Đồng thời Đa Ngữ cảnh

### 4.1. Vòng đời Thực thi Ngầm & Cô lập Same-Origin
`SharedWorker` đại diện cho một ngữ cảnh thực thi ngầm (`SharedWorkerGlobalScope`) chạy trên một thread OS độc lập, có khả năng kết nối đồng thời với nhiều browsing contexts (tab, window, iframe) có **cùng nguồn gốc (same-origin)**:
```javascript
const worker = new SharedWorker('/shared-core.js', 'superapp-mini-core');
worker.port.start();
```
- **Cô lập bảo mật:** Trình duyệt ngăn chặn hoàn toàn việc các mini app khác nguồn gốc kết nối vào chung một SharedWorker (vi phạm Same-Origin Policy trả về `SecurityError`).
- **Tái chế bộ nhớ:** Khi tất cả các tab kết nối bị đóng, runtime duy trì worker trong tối đa 30-60 giây trước khi thu hồi tiến trình và giải phóng RAM.

### 4.2. Bắt tay Cổng Kết nối `MessagePort` & Structured Cloning
Mọi giao tiếp giữa các tab và SharedWorker đều thông qua giao diện `MessagePort`:
```javascript
// Bên trong shared-core.js
self.onconnect = (e) => {
  const port = e.ports[0];
  port.start();
  port.onmessage = (event) => {
    // Xử lý message nhận được qua thuật toán Structured Clone
    port.postMessage({ status: 'ACK', payload: event.data });
  };
};
```
Thuật toán Structured Clone cho phép truyền dữ liệu phức tạp (ArrayBuffer, Blob, ImageBitmap) một cách an toàn mà không gây xung đột bí danh bộ nhớ (memory aliasing).

### 4.3. Gom cụm Kết nối Mạng Đơn nhất (Singleton WebSocket/SSE Multiplexing)
Thay vì mỗi tab mở một kết nối socket riêng gây hao pin và lãng phí cổng máy chủ, SharedWorker đóng vai trò là một hub mạng tập trung:
- Mở duy nhất một kết nối WebSocket hoặc Server-Sent Events (SSE) tới backend.
- Nhận và phân phối (demultiplex) các luồng dữ liệu thời gian thực tới tất cả các tab đang mở.
- Gom cụm các hành động từ các tab khác nhau để truyền qua một socket duy nhất.
- Giới hạn tối đa 2 kết nối WebSocket đồng thời trên mỗi SharedWorker nguồn mini app.

### 4.4. Điều phối Trạng thái Bộ nhớ & Chống Race Condition với W3C Web Locks
SharedWorker lưu trữ bộ nhớ trạng thái tập trung (giỏ hàng, danh sách yêu thích, thông tin phiên), loại bỏ nhu cầu đọc/ghi liên tục xuống đĩa (IndexedDB/localStorage).
Kết hợp với **W3C Web Locks API** (`navigator.locks.request()`), SharedWorker đảm bảo các thao tác cập nhật giỏ hàng hoặc xác nhận thanh toán giữa nhiều tab diễn ra tuần tự và an toàn, loại bỏ hoàn toàn nguy cơ trùng lặp đơn hàng hoặc số dư âm.

### 4.5. Quản trị Tài nguyên & Giám sát Watchdog
Để ngăn chặn tình trạng script chạy ngầm vô tận gây cạn kiệt pin và RAM:
- **Giới hạn RAM Heap:** Áp dụng trần bộ nhớ heap tối đa 64 MB cho SharedWorker trên phần cứng di động.
- **Watchdog Eviction:** Hủy bỏ tiến trình worker nếu sau 60 giây không còn cổng kết nối UI nào hoạt động.
- **Đóng băng ngầm (Background Freezing):** Khi Super App bị ẩn xuống nền (minimized), mức tiêu thụ CPU của SharedWorker bị ép về 0% thông qua chính sách cgroups/QoS của hệ điều hành.

---

## 5. Bảng Đối chiếu & Tổng hợp Yêu cầu Kiểm duyệt Marketplace (Store Gate Matrix)

| Danh mục | Chuẩn Kỹ thuật | Yêu cầu Bắt buộc đối với Mini App | Tiêu chí Kiểm duyệt Store Review (Gate) |
| :--- | :--- | :--- | :--- |
| **SPA Route Detection** | WICG Soft Navigations | Cập nhật URL qua History/Navigation API khi chuyển view; duy trì liên kết nhân quả giữa cử chỉ và DOM mutation | Từ chối mini app chỉ ẩn hiện CSS display mà không cập nhật route; cảnh báo nếu không kích hoạt được entry soft-nav |
| **SPA Performance QoS** | WICG Soft Navigations & W3C Timeline | Tích hợp PerformanceObserver theo dõi `PerformanceSoftNavigationTiming`; duy trì INP <= 200 ms | Độ trễ chuyển trang <= 1.000 ms; INP phân đoạn <= 200 ms trên >= 75% lượt chuyển trang; hạ xếp hạng nếu vượt trần |
| **Mobile Form UX** | WICG Visual Viewport API | Lắng nghe `visualViewport.onresize` để cuộn các ô nhập liệu và nút CTA vào vùng nhìn thấy khi bàn phím mở | Kiểm thử tự động với chiều cao bàn phím 300px; cấm ghim nút submit `bottom: 0` gây che khuất |
| **Zoom & Capsule Safety** | WICG Visual Viewport API | Sử dụng ma trận biến đổi tọa độ visual viewport cho các menu capsule, modal disclosures | Cấm tắt tính năng thu phóng (`user-scalable=no`); đảm bảo capsule native không bị che lấp ở mọi mức zoom |
| **Cross-Tab Concurrency** | WHATWG Shared Workers | Cô lập same-origin tuyệt đối; quản lý vòng đời `MessagePort` chặt chẽ và đóng kết nối khi tab tắt | Linter kiểm tra bắt tay `connect` và gọi `port.start()`; cấm rò rỉ kết nối sau khi đóng view |
| **Network Efficiency** | WHATWG Shared Workers & WebSockets | Multiplexing kết nối WebSocket/SSE qua SharedWorker đối với mini app đa tab | Giới hạn tối đa 2 kết nối WebSocket đồng thời/worker; kiểm tra cơ chế ngắt kết nối và đóng băng ngầm |
| **Data Integrity** | WHATWG Shared Workers & W3C Web Locks | Đồng bộ giỏ hàng và dữ liệu tài chính qua SharedWorker kết hợp khóa Web Locks | Kiểm thử tự động click đồng thời trên 2 tab; xác nhận không xảy ra tình trạng double-spend hoặc race condition |
| **Worker Governance** | Super App Container Watchdog | Tuân thủ giới hạn heap 64 MB; không thực thi vòng lặp vô tận chạy ngầm | Tự động hủy worker nếu không có tab kết nối sau 60s; đóng băng CPU 0% khi host app chuyển sang background |

---

## 6. Kết luận & Lộ trình Triển khai Kế tiếp

Milestone 62 đã mở rộng hệ thống chuẩn mực kỹ thuật Super App Mini App Store lên **881 findings**, giải quyết trọn vẹn 3 mắt xích quan trọng trong vận hành runtime hiện đại:
1. Chuẩn hóa đo lường hiệu năng SPA với **WICG Soft Navigations**, mang lại khả năng phân tích QoS chính xác đến từng view.
2. Bảo đảm trải nghiệm thị giác di động mượt mà và an toàn với **WICG Visual Viewport API**, loại bỏ hoàn toàn các lỗi che khuất bàn phím và xung đột capsule.
3. Tối ưu hóa tài nguyên mạng và đảm bảo toàn vẹn dữ liệu đa tab với **WHATWG Shared Workers**, tạo nền tảng vững chắc cho các mini app thương mại điện tử và tài chính phức tạp.

Theo quy định, toàn bộ tài liệu báo cáo và kho evidence đã được đồng bộ chuẩn xác. Đợt đồng bộ mã nguồn công khai (Public Repo Sync) sẽ được thực hiện tại Milestone 65 (bội số của 5).
