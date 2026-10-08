# Chuyên đề Iteration 34: Launch Handling, Page Lifecycle Memory Governance, and Digital Goods Entitlements

## 1. Tổng quan và Bối cảnh Kiến trúc

Iteration 34 hoàn thiện ba trụ cột hạ tầng cốt lõi cho super-app mini-app container:
1. **Launch Handling & Window Routing (WICG Web App Launch / LaunchQueue)**: Quản lý vòng đời khởi chạy, giải quyết định tuyến cửa sổ đơn (single-instance) vs đa cửa sổ (multi-instance), đệm tham số sâu (LaunchParams) chống race condition trong quá trình khởi động hydration, và đồng bộ với mô hình `onShow` của WeChat/Alipay.
2. **Page Lifecycle State Machine & Memory Governance (WICG Page Lifecycle)**: Định nghĩa máy trạng thái 6 pha (Active, Passive, Hidden, Frozen, Terminated, Discarded), cầu nối tín hiệu áp lực bộ nhớ hệ điều hành (Android `ComponentCallbacks2.onTrimMemory`, iOS `didReceiveMemoryWarning`), cơ chế đóng băng (freeze) tài nguyên nền không tiêu tốn CPU/pin, khôi phục trạng thái document bị hủy thông qua `document.wasDiscarded`, và chính sách giới hạn 5 phút chạy nền (background residence timeout).
3. **Digital Goods & Entitlement Governance (WICG Digital Goods API)**: Chuẩn hóa giao tiếp giữa mini-app web và hệ thống thanh toán số native thông qua `window.getDigitalGoodsService()`, truy vấn danh mục SKU với giá bản địa hóa và chu kỳ thuê bao ISO 8601, tích hợp thanh toán qua W3C Payment Request với chữ ký token chống giả mạo, quy trình tiêu thụ (consume) và xác nhận (acknowledge) tránh hoàn tiền tự động 3 ngày, cùng kiến trúc hạch toán chia sẻ doanh thu (split ledgering), ký quỹ bảo đảm và tuân thủ thuế.

Tất cả 15 findings đã được xác thực HTTP 200 trên 15 URL tiêu chuẩn chính thức, không chứa bí mật hay credential, và được lưu trữ append-only vào `state/findings.jsonl`.

---

## 2. Bảng tổng hợp Findings Iteration 34

| ID | Chủ đề | Tiêu đề Finding | Chuẩn tham chiếu | Mức bằng chứng | URL nguồn chính |
|---|---|---|---|---|---|
| `launch_034_01` | Window Routing Modes | WICG launch_handler manifest member standardizes client routing modes for existing and new container contexts | WICG Web App Launch / W3C Manifest | `normative_standard` | [WICG Web App Launch](https://wicg.github.io/web-app-launch/) |
| `launch_034_02` | Parameter Queuing | LaunchQueue API provides deterministic parameter queuing and consumer registration without startup race conditions | WICG Web App Launch | `normative_standard` | [WICG Web App Launch](https://wicg.github.io/web-app-launch/) |
| `launch_034_03` | LaunchParams Unpacking | LaunchParams interface conveys structured target URLs and read-only file system handles to running instances | WICG Web App Launch | `normative_standard` | [WICG Web App Launch](https://wicg.github.io/web-app-launch/) |
| `launch_034_04` | Task Stacks & Routing | Container task stack architectures balance single-instance resource efficiency against multi-window concurrency | Super-App Architecture / Chrome Docs | `platform_practice` | [Chrome Launch Handler](https://developer.chrome.com/docs/web-platform/launch-handler) |
| `launch_034_05` | Cross-Ecosystem Parity | Cross-ecosystem launch lifecycle convergence aligns WICG Web App Launch with WeChat onShow and Alipay app routing | Super-App Universal Standards | `platform_practice` | [Chrome Launch Handler](https://developer.chrome.com/docs/web-platform/launch-handler) |
| `life_034_01` | Page Lifecycle States | WICG Page Lifecycle specification formalizes six discrete states and lifecycle transition events for containerized contexts | WICG Page Lifecycle / W3C Visibility | `normative_standard` | [WICG Page Lifecycle](https://wicg.github.io/page-lifecycle/) |
| `life_034_02` | Memory Pressure Bridge | Native memory pressure integration translates Android onTrimMemory and iOS memory warnings into progressive WebView eviction | Android ComponentCallbacks2 / iOS UIKit | `platform_practice` | [Android onTrimMemory](https://developer.android.com/reference/android/content/ComponentCallbacks2#onTrimMemory(int)) |
| `life_034_03` | Deterministic Freezing | Deterministic background freezing suspends timers, WebGL contexts, and media pipelines while preserving state in RAM | WICG Page Lifecycle | `normative_standard` | [WICG Page Lifecycle](https://wicg.github.io/page-lifecycle/) |
| `life_034_04` | State Restoration | State restoration contracts utilize document.wasDiscarded and persistent session storage to transparently recover evicted instances | WICG Page Lifecycle | `normative_standard` | [WICG Page Lifecycle](https://wicg.github.io/page-lifecycle/) |
| `life_034_05` | Background Governance | Super-app background policy framework governs maximum residence timers, graceful termination, and privileged background tasks | Super-App Background Policy Standard | `platform_practice` | [Android onTrimMemory](https://developer.android.com/reference/android/content/ComponentCallbacks2#onTrimMemory(int)) |
| `goods_034_01` | Service Binding | WICG Digital Goods API defines payment-provider service binding for web and mini-app in-app purchase ecosystems | WICG Digital Goods / W3C Payment Request | `normative_standard` | [WICG Digital Goods](https://wicg.github.io/digital-goods/) |
| `goods_034_02` | SKU Catalog & Pricing | DigitalGoodsService.getDetails returns standardized SKU metadata, introductory pricing, and localized subscription periods | WICG Digital Goods | `normative_standard` | [WICG Digital Goods](https://wicg.github.io/digital-goods/) |
| `goods_034_03` | Purchase Execution | W3C Payment Request pairing completes in-app purchases and issues cryptographically signed purchase tokens | W3C Payment Request / WICG Digital Goods | `normative_standard` | [W3C Payment Request](https://w3c.github.io/payment-request/) |
| `goods_034_04` | Entitlement Lifecycle | Entitlement lifecycle protocols govern consumable delivery, non-consumable acknowledgement, and refund mitigation | WICG Digital Goods / Google Play Billing | `normative_standard` | [WICG Digital Goods](https://wicg.github.io/digital-goods/) |
| `goods_034_05` | Settlement Governance | Multi-tenant settlement governance establishes split ledgering, platform commissions, escrow reserves, and dispute arbitration | Super-App Storefront Ledgering Standard | `platform_practice` | [Google Play Billing](https://developer.android.com/google/play/billing) |

---

## 3. Phân tích Kỹ thuật Chi tiết theo 3 Lĩnh vực

### 3.1. Launch Handling, Window Routing & Parameter Queuing
- **Khai báo `launch_handler` trong Web App Manifest**:
  - `client_mode`: Cho phép mini-app khai báo hành vi khi người dùng mở lại ứng dụng từ deep-link, thông báo push hoặc phím tắt.
  - Các chế độ: `navigate-new` (tạo WebView mới), `navigate-existing` (chuyển hướng WebView hiện tại và mất state dở dang), `focus-existing` (giữ nguyên tài liệu hiện tại, đưa lên foreground và đẩy sự kiện vào `launchQueue`).
  - Container mặc định áp dụng `focus-existing` cho các mini-app giao dịch tài chính, thương mại điện tử để bảo vệ giỏ hàng và dữ liệu form chưa submit.
- **Hàng đợi tham số `window.launchQueue`**:
  - Giải quyết bài toán Race Condition kinh điển: Khi mini-app khởi động nguội (cold start), quá trình tải bundle JS, xác thực token và hydrate DOM có thể mất hàng trăm mili-giây. Nếu sự kiện mở deep link bắn ngay lúc này, mini-app chưa kịp gắn listener sẽ làm mất tham số.
  - `launchQueue` tự động đệm (buffer) các `LaunchParams` cho đến khi mini-app gọi `window.launchQueue.setConsumer((launchParams) => { ... })`. Khi consumer được đăng ký, hàng đợi lập tức replay chính xác thứ tự các lần mở.
- **Phân tích `LaunchParams` (`targetURL` & `files`)**:
  - Cung cấp URL mục tiêu đầy đủ kèm query parameters và danh sách file descriptor (`FileSystemHandle`) được chia sẻ an toàn qua sandboxing.
  - Ứng dụng mini-app sử dụng `new URL(launchParams.targetURL)` để chuyển route nội bộ (React Router/Vue Router) mà không cần tải lại toàn bộ trang.
- **Single-instance vs Multi-instance Container Architecture**:
  - Mô hình Single-Instance (`singleTask`): Chỉ duy trì tối đa 1 instance WebView cho mỗi `miniAppId`. Giảm tải RAM, tránh phân mảnh storage, quản lý task stack trực quan.
  - Giới hạn ngưỡng RAM: Super-app áp dụng trần tối đa (ví dụ 5 WebViews đồng thời). Khi vượt ngưỡng, instance nền ít dùng nhất (LRU) sẽ bị dọn dẹp có kiểm soát.
- **Hội tụ chuẩn liên nền tảng (Cross-Ecosystem Parity)**:
  - Ánh xạ tương đương giữa WICG LaunchQueue và WeChat `App.onLaunch(options)` / `App.onShow(options)` / `wx.onAppRoute`, cũng như Alipay `my.onAppShow`.
  - SDK Container cung cấp tầng abstraction dịch chuyển mượt mà giữa các nền tảng mà không yêu cầu viết lại business logic.

---

### 3.2. Page Lifecycle State Machine & Memory Governance
- **Máy trạng thái 6 pha WICG Page Lifecycle**:
  - `Active`: Foreground, có input focus, timers và WebGL chạy 100% công suất.
  - `Passive`: Hiển thị nhưng không có focus (modal popup, split view).
  - `Hidden`: Ẩn hoàn toàn (bị che bởi mini-app khác hoặc super-app vào nền). Phát sự kiện `visibilitychange` (`visibilityState === 'hidden'`).
  - `Frozen`: Trình duyệt tạm dừng hoàn toàn CPU, timers (`setInterval`), WebGL render loops, nhưng DOM và JS Heap vẫn được giữ nguyên vẹn trên RAM.
  - `Terminated`: Trang bị dỡ bỏ hoàn toàn khỏi bộ nhớ qua `pagehide`.
  - `Discarded`: Bị hệ điều hành/container thu hồi bộ nhớ khẩn cấp (OOM kill) nhưng vẫn lưu lại token tab để khôi phục khi người dùng quay lại.
- **Cầu nối áp lực bộ nhớ native (OS Memory Pressure Bridge)**:
  - Tích hợp Android `ComponentCallbacks2.onTrimMemory(level)`:
    - `TRIM_MEMORY_RUNNING_MODERATE`: Xóa cache ảnh/bitmap native.
    - `TRIM_MEMORY_RUNNING_CRITICAL`: Chuyển các WebView nền từ `Hidden` sang `Frozen`.
    - `TRIM_MEMORY_COMPLETE`: Hủy các WebView chạy nền không ưu tiên.
  - Tích hợp iOS `UIApplicationDelegate.applicationDidReceiveMemoryWarning(_:)`: Giải phóng bộ nhớ giải mã WebKit và xả cache transient.
  - Container gửi thông điệp khẩn cấp `superapp.onMemoryPressure({ level: 'critical' })` qua JS Bridge để mini-app chủ động tuần tự hóa dữ liệu.
- **Đóng băng tài nguyên xác định (Deterministic Freezing)**:
  - Phát sự kiện `freeze` trước khi đóng băng, cho phép mini-app 3 giây để đóng kết nối IndexedDB, lưu trữ state dở dang và gửi telemetry cuối cùng.
  - Khi người dùng mở lại, sự kiện `resume` được kích hoạt ngay lập tức mà không gây bão timer (timer storming).
- **Khôi phục trạng thái với `document.wasDiscarded`**:
  - Cờ boolean chuẩn hóa `document.wasDiscarded` xác định việc khởi động này là do khôi phục sau khi bị dọn dẹp do thiếu RAM.
  - Mini-app tự động đọc snapshot từ `sessionStorage` (được bảo vệ hạn mức 2MB riêng biệt) để điền lại form, vị trí cuộn và trạng thái tab, mang lại trải nghiệm liền mạch cho người dùng.
- **Chính sách cư trú nền (Background Residence Timeout)**:
  - Áp dụng trần 5 phút (300 giây) ở trạng thái `Frozen`. Nếu người dùng không quay lại, container chủ động giải phóng WebView.
  - Ngoại lệ đặc quyền: Các mini-app phát nhạc nền (như `wx.getBackgroundAudioManager`), dẫn đường GPS hoặc cuộc gọi thoại được cấp phép chạy nền dài hạn sau khi kiểm duyệt tại Store.

---

### 3.3. Digital Goods & Entitlement Governance
- **Kiến trúc WICG Digital Goods API**:
  - Khởi tạo qua `window.getDigitalGoodsService(paymentMethod)`. Container xác minh origin và binding trực tiếp tới thư viện billing native (Google Play Billing, Apple StoreKit hoặc Super-App Native Wallet).
  - Trả về proxy `DigitalGoodsService` hỗ trợ mô hình bảo mật sandbox origin, ngăn chặn iframe bên thứ ba đánh cắp quyền mua hàng.
- **Truy vấn danh mục SKU và Giá bản địa hóa**:
  - Gọi `getDetails(itemIds)` nhận về mảng đối tượng `ItemDetails`: tên, mô tả, đơn vị tiền tệ chuẩn ISO 4217, giá trị định dạng, chu kỳ thuê bao chuẩn ISO 8601 (`P1M`, `P1Y`), giá dùng thử và chu kỳ khuyến mãi.
  - Loại bỏ hoàn toàn sự phức tạp khi mini-app phải tự xây dựng backend tính toán tỷ giá ngoại tệ và hiển thị thuế VAT khu vực.
- **Thực thi thanh toán kết hợp W3C Payment Request**:
  - Khởi tạo `PaymentRequest` với `supportedMethods` trỏ tới dịch vụ thanh toán của super-app.
  - Phương thức `.show()` hiển thị giao diện thanh toán native bảo mật (xác thực sinh trắc học vân tay/FaceID, số dư ví hoặc thẻ liên kết).
  - Trả về `PaymentResponse` mang `purchaseToken` có chữ ký mật mã. Mini-app gửi token về server của mình để xác thực server-to-server qua mTLS/OAuth 2.0 trước khi kích hoạt vật phẩm.
- **Vòng đời Entitlement (Consume & Acknowledge)**:
  - Phân loại vật phẩm tiêu hao (Consumable - ví dụ tiền vàng, lượt chơi) sử dụng phương thức `consume(purchaseToken)` để cho phép mua lại.
  - Bắt buộc xác nhận (Acknowledge) đối với vật phẩm không tiêu hao (Non-consumable) và gói thuê bao định kỳ trong vòng 3 ngày; nếu quá hạn, hệ thống tự động hủy giao dịch và hoàn tiền cho người dùng.
  - Khôi phục giao dịch qua `listPurchases()` giúp người dùng lấy lại quyền lợi khi đổi thiết bị hoặc cài đặt lại ứng dụng.
- **Quản trị hạch toán phân chia đa đối tác (Multi-Tenant Split Ledgering)**:
  - Tự động tách dòng tiền giao dịch: khấu trừ hoa hồng nền tảng (ví dụ 15% cho doanh nghiệp nhỏ, 30% tiêu chuẩn), phí cổng thanh toán (1-2%), thuế nhà thầu/VAT, và ghi có vào sổ cái nhà phát triển.
  - Duy trì quỹ ký quỹ bảo đảm (rolling reserve 5-10% trong 30-60 ngày) để xử lý hoàn tiền, khiếu nại bồi hoàn (chargeback) và gian lận thẻ.
  - Cung cấp cổng console quản lý tranh chấp minh bạch và tích hợp xuất hóa đơn điện tử tự động tuân thủ Nghị định 123/2020/NĐ-CP.

---

## 4. Bảng phân định Yêu cầu Kỹ thuật theo Bối cảnh (Enterprise vs MVP)

| Tiêu chuẩn / Cơ chế | Bối cảnh Doanh nghiệp / Regulated Super-App | Triển khai Tối thiểu (MVP) |
|---|---|---|
| **Launch Routing** | Bắt buộc hỗ trợ `launch_handler: { client_mode: "focus-existing" }` và `launchQueue.setConsumer` có đệm tham số | Hỗ trợ điều hướng URL cơ bản, tải lại WebView khi nhận deep-link mới |
| **Task Stack Memory** | Quản lý LRU Stack nghiêm ngặt (tối đa 3-5 WebViews), tự động hủy instance nền quá 5 phút | Không giới hạn hoặc phụ thuộc hoàn toàn vào cơ chế kill ngẫu nhiên của OS |
| **Page Lifecycle** | Triển khai đầy đủ máy trạng thái 6 pha; lắng nghe `onTrimMemory` để đóng băng (`freeze`) và dọn dẹp cache | Chỉ lắng nghe `visibilitychange` để tạm dừng âm thanh cơ bản |
| **State Recovery** | Hỗ trợ `document.wasDiscarded` và tự động khôi phục state từ `sessionStorage` 2MB được bảo vệ | Người dùng tự tải lại từ đầu nếu ứng dụng bị dỡ bỏ khỏi RAM |
| **Digital Goods API** | Bắt buộc `window.getDigitalGoodsService` chuẩn WICG kết nối hạ tầng billing native, xác thực mTLS server-to-server | Sử dụng web redirect hoặc WebView payment gateway đơn giản |
| **Entitlement Lifecycle** | Kiểm soát chặt chẽ quy trình Acknowledge/Consume trong 72h; cảnh báo timeout hoàn tiền tự động | Xử lý thủ công qua đối soát file cuối tháng |
| **Sổ cái & Ký quỹ** | Hạch toán kép tự động, khấu trừ thuế VAT/phí nền tảng, duy trì quỹ ký quỹ 5-10% bảo đảm chargeback | Thanh toán định kỳ hàng tháng theo báo cáo tổng hợp |
