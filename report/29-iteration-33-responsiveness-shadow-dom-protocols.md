# Báo Cáo Chuyên Đề: Đo Lường Độ Phản Hồi Runtime, Đóng Gói Giao Diện Streaming & Định Tuyến Giao Thức Tùy Chỉnh (Iteration 33)

## 1. Bối Cảnh và Mục Tiêu Nghiên Cứu
Trong kiến trúc Super-App hiện đại, để đảm bảo hàng trăm ứng dụng nhỏ (mini-app) hoạt động mượt mà, độc lập và có thể phối hợp tương tác mà không phá vỡ tính ổn định của ứng dụng mẹ, ba trụ cột kỹ thuật trọng yếu cần được chuẩn hóa bao gồm:
1. **Đo Lường Độ Phản Hồi Runtime & Phân Tích Cội Nguồn Script (Runtime Responsiveness & Script Attribution):** Làm thế nào để định lượng chính xác độ trễ khung hình, phát hiện các tác vụ dài (long tasks >50ms), cô lập đúng hàm JavaScript và URL gây giật lag (jank), và kiểm soát chỉ số Interaction to Next Paint (INP) dưới 200ms?
2. **Đóng Gói Thành Phần & Thủy Hóa Luồng (Component Encapsulation & Streaming Hydration):** Giải pháp nào cho phép render giao diện web component đóng gói hoàn toàn từ luồng Worker/Server mà không cần nạp JavaScript hydrating cồng kềnh (Declarative Shadow DOM), đồng thời cung cấp giao diện tùy biến giao diện có kiểm soát (CSS Shadow Parts) mà không làm rò rỉ xung đột CSS?
3. **Đăng Ký Xử Lý Giao Thức & Điều Phối Deep-Link Liên Ứng Dụng (Protocol Handlers & Deep-Link Invocation):** Cơ chế nào giúp các mini-app đăng ký URI scheme tùy biến (dạng `web+pay`, `web+map`) một cách khai báo qua manifest hoặc runtime API, xác thực nguồn gốc ứng dụng gọi (caller attestation), phân xử xung đột bộ xử lý (intent disambiguation) và thu hồi quyền từ xa khi xảy ra sự cố?

Iteration 33 bổ sung **15 phát hiện chuẩn hóa mới** (từ `loaf_033_01` đến `proto_033_05`), nâng tổng số phát hiện chuẩn hóa lên **446 phát hiện**, tích hợp toàn diện các chuẩn của W3C, WHATWG, CSSWG và thực tiễn kiến trúc container native.

---

## 2. Đo Lường Độ Phản Hồi Runtime & Phân Tích Cội Nguồn Script (LoAF & Long Tasks)

### 2.1. Tiêu Chuẩn W3C Long Tasks API 1.0 & Ngưỡng 50ms Luồng Giao Diện
Tiêu chuẩn W3C Long Tasks API 1.0 định nghĩa giao diện `PerformanceLongTaskTiming` kế thừa từ `PerformanceEntry`, chuyên trách theo dõi luồng chính (main thread):
- **Ngưỡng 50ms Cốt Lõi:** Mọi tác vụ tính toán đơn lẻ trên luồng giao diện vượt quá 50 mili-giây đều được báo cáo là một `longtask`. Ngưỡng này xuất phát trực tiếp từ mô hình hiệu năng RAIL, đảm bảo rằng trong trường hợp người dùng tương tác ngay giữa một tác vụ 50ms, trình duyệt vẫn hoàn thành phản hồi giao diện trong khoảng thời gian lý tưởng 100ms.
- **Đăng Ký PerformanceObserver:** Ứng dụng và vỏ bọc native lắng nghe sự kiện qua:
  ```javascript
  const observer = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      console.warn(`Long task detected: ${entry.duration}ms at ${entry.startTime}`);
    }
  });
  observer.observe({ entryTypes: ['longtask'] });
  ```
- **TaskAttributionTiming:** Mỗi mục longtask chứa mảng `attribution` chỉ rõ ngữ cảnh chịu trách nhiệm (`containerType`: `iframe`/`embed`, `containerSrc`, `containerId`, `containerName`), cho phép Super-App xác định chính xác jank xuất phát từ WebView chính hay iframe của bên thứ ba.
- **Kiểm Soát Super-App:** Vỏ container tự động tích lũy chỉ số Total Blocking Time (TBT) trong suốt chu kỳ khởi động mini-app. Nếu tổng thời gian nghẽn vượt quá 300ms trước khi đạt First Meaningful Paint, hệ thống cảnh báo telemetry được kích hoạt.

### 2.2. Chuẩn PerformanceLongAnimationFrameTiming (LoAF) & Bóc Tách Khung Hình
Giao diện Long Animation Frames (LoAF) thay thế các hạn chế của Long Tasks bằng cách gắn chặt độ trễ với chu kỳ khung hình (frame lifecycle) của trình duyệt:
- **Đăng Ký LoAF:** `observer.observe({ type: 'long-animation-frame', buffered: true });`
- **Bóc Tách Các Pha Render:**
  - `duration`: Tổng thời gian thực thi khung hình từ khi bắt đầu tác vụ đầu tiên đến khi hoàn tất hiển thị.
  - `blockingDuration`: Tổng thời gian các tác vụ trên main thread chặn các tác vụ khác trong cùng frame.
  - `renderDuration`: Thời gian dành riêng cho pha hiển thị giao diện của engine trình duyệt.
  - `styleAndLayoutDuration`: Thời gian tính toán lại style (Recalculate Style), bố cục (Layout) và tiền hiển thị (Pre-paint).
  - `desiredExecutionStart`: Mốc thời gian dự kiến chạy của callback (như `requestAnimationFrame`).
- **Phân Định Trách Nhiệm Kỹ Thuật:** LoAF cho phép kỹ sư vận hành phân biệt rành mạch giữa tắc nghẽn do JavaScript tính toán nặng (`blockingDuration` cao) và tắc nghẽn do cấu trúc DOM/CSS quá phức tạp gây giật khung hình (`styleAndLayoutDuration` cao).

### 2.3. PerformanceScriptTiming & Xác Định Chính Xác Mã Gây Lỗi
Thuộc tính `scripts` trong LoAF trả về danh sách các đối tượng `PerformanceScriptTiming`, cung cấp thông tin truy vết dòng mã chi tiết:
- **Hàm & Bộ Khởi Tạo (`invoker` / `invokerType`):** Phân loại nguyên nhân gọi mã thành `user-callback` (click, touch), `event-listener`, `promise-resolve`, `render-request`, hoặc `classic-script`.
- **Định Vị Tệp & Ký Tự (`sourceURL` / `sourceFunctionName` / `sourceCharPosition`):** Xác định URL của tệp JS, tên hàm thực thi và vị trí ký tự chính xác trong mã nguồn.
- **Thời Gian Chạy vs Thời Gian Dừng (`executionDuration` / `pauseDuration`):** Tách biệt thời gian CPU chủ động chạy lệnh với thời gian bị đóng băng đồng bộ do gọi `alert()`, `confirm()` hoặc XHR đồng bộ.
- **Hệ Thống Phân Tích Super-App:** Cổng APM của Super-App tự động map vị trí lỗi với file Source Map đã tải lên trong console nhà phát triển, đồng thời lọc bỏ cảnh báo sai từ các SDK đo lường ngoài.

### 2.4. Phân Tích Cội Nguồn Chỉ Số Interaction to Next Paint (INP)
INP đánh giá toàn diện độ nhạy tương tác của mini-app trong suốt phiên trải nghiệm:
- **Ba Pha Trễ:**
  1. *Độ Trễ Đầu Vào (Input Delay):* Thời gian chờ luồng chính rảnh để bắt đầu xử lý sự kiện.
  2. *Thời Gian Xử Lý (Processing Duration):* Thời gian thực thi mã trong trình xử lý sự kiện.
  3. *Độ Trễ Hiển Thị (Presentation Delay):* Thời gian compositor trình duyệt tính toán và vẽ lại khung hình mới.
- **Tiêu Chuẩn Đạt Chuẩn:** Ngưỡng P75 của INP phải duy trì dưới **200 mili-giây**. Container tích hợp watchdog tự động ghi log LoAF snapshot mỗi khi phát hiện tương tác người dùng vượt ngưỡng 200ms.

### 2.5. Cơ Chế Giám Sát Watchdog & Giảm Tải Ứng Dụng Chạy Nền
Để bảo vệ độ ổn định của ứng dụng mẹ và thời lượng pin thiết bị:
- **Watchdog Thread:** Luồng kiểm tra native gửi tín hiệu ping tới WebView mỗi 500ms. Nếu luồng chính không phản hồi sau 3000ms, hệ thống lưu lại stack trace. Nếu đóng băng quá 5000ms, container kích hoạt cơ chế khôi phục khẩn cấp để ngăn chặn lỗi hệ thống Application Not Responding (ANR).
- **Điều Tiết Mini-App Chạy Ẩn:** Khi mini-app chuyển xuống chế độ chạy nền, container tự động kẹp tần số timer (`setTimeout`/`setInterval`) tối thiểu về >=1000ms và tạm dừng `requestAnimationFrame`, trả lại tài nguyên CPU cho Super-App.

---

## 3. Đóng Gói Thành Phần & Thủy Hóa Luồng (Declarative Shadow DOM & CSS Parts)

### 3.1. WHATWG Declarative Shadow DOM (DSD) & <template shadowrootmode>
Chuẩn WHATWG HTML Living Standard chuẩn hóa cú pháp Declarative Shadow DOM cho phép tạo cây shadow root trực tiếp bằng HTML không cần JavaScript:
- **Cú Pháp Khai Báo:**
  ```html
  <host-element>
    <template shadowrootmode="open" shadowrootclonable shadowrootdelegatesfocus>
      <style>:host { display: block; border: 1px solid #ccc; }</style>
      <slot></slot>
    </template>
  </host-element>
  ```
- **Thủy Hóa Không Trễ (Zero-JS SSR Hydration):** Bộ phân tích cú pháp HTML của WebView tự động gắn ShadowRoot và chuyển nội dung template vào bên trong ngay trong pha parse HTML đầu tiên, loại bỏ hoàn toàn hiện tượng chớp giao diện (Flash of Unstyled Content - FOUC).
- **Nhân Bản Cây Shadow (`shadowrootclonable`):** Cho phép gọi `cloneNode(true)` trên phần tử cha mà không làm mất cây shadow root.
- **Ủy Quyền Tiêu Điểm (`shadowrootdelegatesfocus`):** Khi click vào vùng trống của custom element, tiêu điểm tự động chuyển vào phần tử nhập liệu đầu tiên bên trong shadow tree.

### 3.2. Vòng Đời WHATWG Custom Elements
Các thành phần giao diện SDK Super-App được xây dựng trên chuẩn Custom Elements:
- `connectedCallback()`: Kích hoạt khi phần tử được đưa vào DOM.
- `disconnectedCallback()`: Bắt buộc giải phóng tài nguyên, hủy lắng nghe sự kiện để tránh rò rỉ bộ nhớ khi chuyển trang.
- `attributeChangedCallback(name, oldValue, newValue)`: Phản ứng nhanh với các thuộc tính được liệt kê trong mảng `observedAttributes`.

### 3.3. CSS Shadow Parts (::part() & exportparts)
Giải quyết bài toán xung đột phong cách giao diện giữa Super-App và Mini-App:
- **Giao Diện Kiểu Dáng Có Kiểm Soát:**
  ```html
  <!-- Bên trong shadow tree -->
  <button part="submit-btn icon">Xác nhận</button>
  ```
  ```css
  /* Mã CSS bên ngoài */
  my-checkout::part(submit-btn) {
    background-color: var(--superapp-primary-color);
  }
  ```
- **Xuất Thuộc Tính Lồng Nhau (`exportparts`):** Cho phép các widget phức hợp chuyển tiếp thuộc tính part của component con ra bên ngoài (`exportparts="icon: button-icon"`).
- **Bảo Vệ Tính Bao Gói:** Tất cả các phần tử DOM bên trong không được gắn thẻ `part` đều được bảo vệ tuyệt đối trước CSS bên ngoài, ngăn chặn hoàn toàn việc xung đột CSS cascade.

### 3.4. Kiến Trúc Hai Luồng (Dual-Thread) Với DSD Streaming
Các nền tảng mini-app như WindVane và WeChat phân chia logic thành luồng Dịch vụ (Worker) và luồng Hiển thị (WebView):
- Luồng Worker biên dịch cấu trúc trang thành các chuỗi HTML chứa Declarative Shadow DOM.
- Vỏ container truyền thẳng luồng dữ liệu byte vào WebView parser qua stream native hoặc `element.setHTMLUnsafe()`.
- Kết quả: Rút ngắn thời gian First Contentful Paint (FCP) từ ~400ms xuống dưới 100ms trên các thiết bị di động tầm trung.

### 3.5. Cô Lập An Toàn Cho Widget Bên Thứ Ba
- **Closed Shadow Root:** Các thẻ widget nhúng của đối tác trên màn hình chính (feed cards, checkout drawer) được bao bọc trong `shadowrootmode="closed"`, ngăn ngừa mã JavaScript của ứng dụng khác can thiệp hoặc đánh cắp dữ liệu trường nhập liệu.
- **CSS Layout Containment:** Áp dụng `contain: content;` trên khung chứa widget để cô lập hoàn toàn việc tính toán lại layout và paint, không làm gián đoạn luồng hiển thị của trang chính.

---

## 4. Đăng Ký Xử Lý Giao Thức & Điều Phối Deep-Link Liên Ứng Dụng

### 4.1. WHATWG Navigator.registerProtocolHandler
Chuẩn WHATWG HTML Living Standard chuẩn hóa API:
```javascript
navigator.registerProtocolHandler("web+pay", "https://superapp.host/miniapp/checkout?uri=%s");
```
- **Quy Tắc Tiền Tố:** Bắt buộc phải có tiền tố `web+` hoặc `ext+` (trừ các scheme chuẩn như `mailto`, `tel`, `sms`).
- **Mã Thay Thế `%s`:** Trình duyệt/container thay thế toàn bộ URI được kích hoạt vào vị trí `%s` sau khi đã mã hóa an toàn.
- **Ràng Buộc Same-Origin:** URL đích xử lý bắt buộc phải cùng origin với tài liệu thực hiện đăng ký.

### 4.2. Khai Báo Trong W3C Web App Manifest
Chuẩn W3C Web App Manifest cho phép đăng ký giao thức ngay trong tệp cấu hình triển khai:
```json
{
  "name": "Thanh Toán Nhanh Mini-App",
  "protocol_handlers": [
    {
      "protocol": "web+pay",
      "url": "/pay?order=%s"
    }
  ]
}
```
Việc khai báo trong manifest giúp hệ điều hành và Super-App gắn kết giao thức ngay khi cài đặt mini-app mà không cần bật popup xin quyền gây phiền hà trong lúc đang chạy ứng dụng.

### 4.3. Bảo Mật Định Tuyến & Phòng Chống Tấn Công Scheme Hijacking
- **Chống Chiếm Đoạt Giao Thức (Scheme Squatting):** Cửa hàng mini-app duy trì sổ bộ định danh giao thức có ký số. Các scheme nhạy cảm liên quan đến tài chính, thanh toán hoặc viễn thông chỉ được cấp cho các nhà xuất bản doanh nghiệp đã xác thực danh tính.
- **Vệ Sinh Tham Số & Chống Điều Hướng Mở (Open Redirect):** Bộ định tuyến native lọc bỏ các ký tự điều khiển CRLF, ngăn chặn giao thức giả mạo (`javascript:`) và bắt buộc URL chuyển tiếp phải nằm trong phạm vi domain hợp lệ của mini-app.

### 4.4. Hợp Đồng Gọi Liên Mini-App & Xác Thực Nguồn Gốc (Caller Attestation)
Khi một mini-app gọi sang một mini-app khác (ví dụ: ứng dụng Đặt xe gọi sang ứng dụng Bản đồ hoặc Ví điện tử):
- Vỏ container tạo một token JWT ký số chứa thông tin định danh ứng dụng gọi (`callerAppId`), domain thuê bao và quyền hạn của người dùng.
- Mini-app đích nhận token, xác minh tính hợp lệ và xử lý giao dịch.
- Sau khi hoàn thành, ứng dụng gọi lệnh `superapp.returnResult({ transactionId: "...", status: "SUCCESS" })`. Container tự động phục hồi ngăn xếp giao diện và gửi kết quả về ứng dụng ban đầu mà không cần tải lại trang.

### 4.5. Phân Xử Xung Đột Giao Thức & Thu Hồi Từ Xa (Revocation)
- **Menu Phân Xử Ý Định (Intent Disambiguation):** Khi nhiều mini-app cùng đăng ký một scheme chung (ví dụ cả Grab và Gojek cùng hỗ trợ `web+delivery`), container hiển thị bảng chọn gốc: "Mở bằng..." kèm tùy chọn "Chỉ một lần" hoặc "Luôn luôn".
- **Thu Hồi Từ Xa Khẩn Cấp:** Nếu phát hiện mini-app có hành vi lừa đảo hoặc vi phạm bảo mật, bảng điều khiển vận hành có thể phát tín hiệu thu hồi giao thức tức thì tới toàn bộ máy khách trên mạng mà không cần phát hành bản cập nhật ứng dụng Super-App mới lên Apple App Store / Google Play Store.

---

## 5. Danh Mục 15 Phát Hiện Chuẩn Hóa Mới (Iteration 33)

| Mã ID | Chủ đề | Cấp độ chứng cứ | Tóm tắt kỹ thuật chuẩn hóa |
|---|---|---|---|
| `loaf_033_01` | W3C Long Tasks API 1.0 | `w3c_recommendation` | Ngưỡng 50ms theo dõi UI thread, TaskAttributionTiming cô lập container/iframe và giám sát TBT khởi động. |
| `loaf_033_02` | LoAF Timing Specification | `living_standard` | Bóc tách chi tiết khung hình >50ms thành `renderDuration`, `styleAndLayoutDuration` và `blockingDuration`. |
| `loaf_033_03` | PerformanceScriptTiming | `living_standard` | Cô lập chính xác hàm, tệp mã nguồn và vị trí ký tự gây jank bằng `sourceURL`, `sourceCharPosition` và `invoker`. |
| `loaf_033_04` | INP Root Cause Attribution | `platform_practice` | Phân tích 3 pha tương tác (Input Delay, Processing, Presentation Delay) đảm bảo ngưỡng INP P75 <200ms. |
| `loaf_033_05` | Watchdog Governance | `platform_practice` | Luồng kiểm soát native phát hiện treo 3s/5s, kẹp tần số timer nền >=1000ms và phòng chống lỗi ANR. |
| `dsd_033_01` | Declarative Shadow DOM | `living_standard` | WHATWG DSD sử dụng `<template shadowrootmode="open|closed">` thủy hóa giao diện SSR không cần JS. |
| `dsd_033_02` | Custom Elements Lifecycle | `living_standard` | Chuẩn hóa `connectedCallback`, `disconnectedCallback` dọn dẹp bộ nhớ và quan sát thuộc tính phản ứng. |
| `dsd_033_03` | CSS Shadow Parts | `w3c_recommendation` | Kiểm soát tùy biến giao diện qua `::part()` và `exportparts` mà không làm vỡ tính bao gói CSS. |
| `dsd_033_04` | Dual-Thread DSD Streaming | `platform_practice` | Worker tạo template DSD truyền thẳng vào luồng parser của WebView, đạt FCP <100ms. |
| `dsd_033_05` | Third-Party Widget Isolation | `platform_practice` | Đóng gói widget nhúng trong Closed Shadow DOM và áp dụng `contain: content;` chống clickjacking và CSS spoofing. |
| `proto_033_01` | registerProtocolHandler | `living_standard` | WHATWG đăng ký scheme tùy biến tiền tố `web+`, thay thế tham số `%s` và ràng buộc domain same-origin. |
| `proto_033_02` | Manifest protocol_handlers | `w3c_recommendation` | Khai báo giao thức trực tiếp trong tệp manifest, gắn kết tự động khi cài đặt không cần popup xin quyền. |
| `proto_033_03` | Protocol Dispatch Security | `platform_practice` | Chống chiếm đoạt scheme, lọc bỏ ký tự CRLF và ngăn chặn lỗ hổng điều hướng mở (Open Redirect). |
| `proto_033_04` | Inter-App Caller Attestation | `platform_practice` | Truyền token xác thực nguồn gốc ứng dụng gọi có ký số và xử lý kết quả hoàn trả qua handle an toàn. |
| `proto_033_05` | Protocol Intent Registry | `platform_practice` | Bảng điều phối intent phân xử chọn ứng dụng xử lý, lưu tùy chọn mặc định và hỗ trợ thu hồi từ xa. |

---

## 6. Khuyến Nghị Vận Hành Cho Super-App Core Team
1. **Tích Hợp LoAF & Watchdog Vào Container SDK:** Triển khai module `PerformanceObserver` tự động bắt các khung hình rơi vào diện LoAF trong WebView và tự động gửi snapshot lỗi đã symbolicate về hệ thống APM.
2. **Chuẩn Hóa Khung Giao Diện DSD & Custom Elements:** Khuyến khích các đội phát triển mini-app sử dụng Declarative Shadow DOM trong pipeline build để giảm tải thời gian khởi tạo VDOM, đặc biệt trên các máy Android cấu hình yếu.
3. **Thiết Lập Cổng Quản Trị Scheme Tập Trung:** Ban hành quy chế đăng ký tiền tố `web+`, xác thực chữ ký số của nhà phát triển và thiết lập giao diện điều phối "Open with..." trong ứng dụng mẹ để đảm bảo quyền lựa chọn minh bạch cho người dùng.
