# Báo Cáo Chuyên Đề: Quản Trị Lưu Trữ, Phân Vùng Dữ Liệu và Đồng Bộ Trạng Thái Liên Ngữ Cảnh (Iteration 32)

## 1. Bối Cảnh và Mục Tiêu Nghiên Cứu
Trong kiến trúc Super-App hiện đại với hàng trăm mini-app đa người thuê (multi-tenant) cùng vận hành trên một vỏ native duy nhất, ba bài toán kỹ thuật cốt lõi quyết định tính toàn vẹn bảo mật và trải nghiệm người dùng bao gồm:
1. **Dọn dẹp và thu hồi dữ liệu lưu trữ (Storage Sanitization & Eviction):** Làm sao để xoá sạch cache, cookie, cơ sở dữ liệu cục bộ khi người dùng đăng xuất, chuyển tài khoản hoặc khi mini-app bị gỡ bỏ mà không ảnh hưởng đến dữ liệu của Super-App hay các mini-app khác?
2. **Phân vùng lưu trữ và ủy quyền truy cập (Storage Partitioning & Access Delegation):** Khi mini-app nhúng các widget hoặc dịch vụ của bên thứ ba (bản đồ, thanh toán, cổng xác thực), làm thế nào để ngăn chặn hành vi theo dõi chéo (cross-site tracking) trong khi vẫn cho phép chia sẻ trạng thái có kiểm soát?
3. **Đồng bộ trạng thái liên ngữ cảnh an toàn (Inter-Context Synchronization & Safe Serialization):** Khi một mini-app mở nhiều tab/view song song (multi-instance), cơ chế nào giúp đồng bộ giỏ hàng, làm mới token và phát tín hiệu đăng xuất đồng loạt mà không bị lỗi nhiễm độc nguyên mẫu (prototype pollution) hay cạn kiệt tài nguyên?

Iteration 32 bổ sung **15 phát hiện chuẩn hóa mới** (từ `csd_storage_032_01` đến `bc_clone_032_05`), nâng tổng số phát hiện chuẩn hóa lên **431 phát hiện**, đối chiếu trực tiếp các tiêu chuẩn W3C, PrivacyCG, IETF, WHATWG, Apple WebKit và Android WebView.

---

## 2. Tiêu Chuẩn Quản Trị Lưu Trữ & Thu Hồi Dữ Liệu Vỏ Native

### 2.1. W3C Clear-Site-Data Header & Chỉ Thị Xóa Dữ Liệu
Chuẩn W3C `Clear-Site-Data` (Working Draft & Editors' Draft) định nghĩa một HTTP response header mang tính mệnh lệnh từ máy chủ tới trình duyệt/WebView để dọn dẹp các loại dữ liệu cục bộ thuộc về origin phản hồi:
- `cache`: Xóa sạch HTTP cache, prerendered pages, script caches, và WebGL shader caches.
- `cookies`: Xóa toàn bộ cookie của origin và các domain con, thông tin xác thực cơ bản (HTTP Basic/Digest) và origin-bound tokens.
- `storage`: Xóa toàn bộ cơ chế lưu trữ DOM bao gồm `localStorage`, `sessionStorage`, `IndexedDB`, WebSQL và đăng ký Service Worker.
- `executionContexts`: Vô hiệu hóa (neuter) và tải lại toàn bộ browsing contexts đang hiển thị origin đó.
- `*`: Ký tự đại diện thực thi đồng thời tất cả các nhóm trên.

*Ràng buộc bảo mật*: `Clear-Site-Data` **bắt buộc** phải được truyền qua kết nối HTTPS đã xác thực (`a priori authenticated URL`) và **tuyệt đối không được xử lý** nếu phản hồi được tạo ra bởi một Service Worker tổng hợp nội bộ.

### 2.2. Vòng Đời Mini-App & Kích Hoạt Xóa Dữ Liệu
Vỏ bọc Super-App container kích hoạt quy trình dọn dẹp theo ma trận 4 kịch bản:
1. **Đăng xuất người dùng (User Logout):** Ngắt kết nối phiên, xóa sạch token xác thực và gọi Clear-Site-Data để dọn dẹp DOM storage cục bộ, ngăn ngừa tấn công cố định phiên (session fixation).
2. **Chuyển đổi tài khoản (Account Switching):** Khi người dùng đổi profile trên cùng thiết bị, container cô lập hoặc xóa phân vùng storage cũ, loại bỏ rủi ro rò rỉ dữ liệu giữa các cá nhân.
3. **Gỡ cài đặt Mini-App (Uninstallation):** Thu hồi toàn bộ thư mục tệp tin riêng, xóa IndexedDB và giải phóng dung lượng ổ cứng.
4. **Xóa sổ bảo mật khẩn cấp (Security Incident Remote Wipe):** Bảng điều khiển vận hành phát lệnh xóa dữ liệu từ xa tới toàn bộ máy khách đang cài đặt mini-app vi phạm.

### 2.3. API Xóa Dữ Liệu WebView Cấp Native (WebKit & Android)
Super-App trừu tượng hóa các API native thành module `MiniAppStorageManager`:
- **Apple iOS WebKit:** Sử dụng `WKWebsiteDataStore.defaultDataStore()` hoặc non-persistent data store. Truy vấn các bản ghi dữ liệu bằng `fetchDataRecordsOfTypes:completionHandler:` với các loại `WKWebsiteDataTypeCookies`, `WKWebsiteDataTypeDiskCache`, `WKWebsiteDataTypeLocalStorage`, `WKWebsiteDataTypeIndexedDBDatabases`, sau đó gọi `removeDataOfTypes:forDataRecords:completionHandler:`.
- **Android WebView:** Sử dụng `WebStorage.getInstance().deleteOrigin(origin)` để xóa LocalStorage/IndexedDB, `CookieManager.getInstance().removeAllCookies()` để xóa cookie, và `WebView.clearCache(true)`.

### 2.4. Tiêu Chuẩn OWASP MASVS-STORAGE & Giới Hạn Quota
- **OWASP MASVS-STORAGE & HTML5 Security:** Cấm tuyệt đối việc lưu trữ bearer token, khóa bí mật hoặc PII chưa mã hóa trong `localStorage`/`sessionStorage` do nguy cơ bị đánh cắp qua DOM XSS. Container cung cấp API lưu trữ an toàn mã hóa phần cứng (KeyStore/Secure Enclave).
- **Hạn Mức Bộ Nhớ & Thu Hồi LRU:** Container áp đặt hạn mức lưu trữ (WeChat: 10MB/mini-app, tối đa 50MB tổng; Super-App chuẩn hóa: 10MB cho ứng dụng tiêu chuẩn, 50MB cho media/game). Khi bộ nhớ thiết bị chạm ngưỡng cảnh báo, thuật toán Least-Recently-Used (LRU) sẽ tự động dọn dẹp bộ nhớ đệm của các mini-app không hoạt động.

---

## 3. Phân Vùng Lưu Trữ & Ủy Quyền Truy Cập (PrivacyCG SAA & IETF CHIPS)

### 3.1. PrivacyCG Storage Access API (SAA)
Trước sự thoái lui của cookie bên thứ ba chưa phân vùng, PrivacyCG chuẩn hóa Storage Access API cho phép các iframe nhúng truy vấn và xin quyền truy cập lưu trữ gốc:
- `document.hasStorageAccess()`: Trả về Promise kiểu boolean kiểm tra trạng thái truy cập hiện tại.
- `document.requestStorageAccess()`: Yêu cầu kích hoạt quyền truy cập storage từ người dùng.
- *Điều kiện tiên quyết*: Bắt buộc phải có cử chỉ kích hoạt của người dùng (`UserActivation.isActive` qua click/tap) và thuộc tính iframe phải khai báo `allow="storage-access"`.

### 3.2. Mở Rộng SAA Cho Bộ Nhớ Không Phải Cookie (Non-Cookie Storage)
Chuẩn mở rộng PrivacyCG cho phép iframe nhúng yêu cầu mở khóa unpartitioned `localStorage`, `sessionStorage`, `indexedDB` và `caches`. Quyền truy cập được gắn chặt với ngữ cảnh gọi và tự động hết hiệu lực khi document bị dỡ bỏ.

### 3.3. API requestStorageAccessFor() Cho Trang Top-Level
Để tối ưu trải nghiệm và tránh popup liên tục làm gián đoạn người dùng, `document.requestStorageAccessFor(requestedOrigin)` cho phép trang Super-App chủ động xin quyền truy cập thay cho origin của mini-app đối tác đã xác minh trước khi view được hiển thị.

### 3.4. IETF CHIPS (Cookies Having Independent Partitioned State)
Bản thảo IETF `draft-cutler-httpbis-partitioned-cookies` đưa vào chỉ thị `Partitioned`:
```http
Set-Cookie: __Host-id=session123; Secure; Path=/; SameSite=None; Partitioned;
```
Cookie được khóa kép theo bộ đôi `{top-level site, partition key}`. Nếu hai mini-app khác nhau cùng nhúng một widget thanh toán, widget đó nhận hai hũ cookie hoàn toàn độc lập, ngăn chặn hoàn toàn việc theo dõi hành vi người dùng xuyên ứng dụng.

---

## 4. Đồng Bộ Trạng Thái Liên Ngữ Cảnh & Tuần Tự Hóa Dữ Liệu An Toàn

### 4.1. WHATWG HTML BroadcastChannel API
Cung cấp kênh truyền thông xuất bản - đăng ký (pub-sub) nhẹ giữa các browsing context cùng origin (cửa sổ, iframe, web worker):
```javascript
const channel = new BroadcastChannel('miniapp_cart_sync');
channel.postMessage({ type: 'ITEM_ADDED', itemId: 'SKU123' });
channel.onmessage = (event) => { updateCartBadge(event.data); };
```
Origin isolation đảm bảo các kênh có cùng tên nhưng ở origin khác nhau hoàn toàn không thể lắng nghe hoặc can thiệp lẫn nhau.

### 4.2. WHATWG Structured Clone Algorithm & Chuyển Giao Đối Tượng (Transferable Objects)
Thuật toán nhân bản có cấu trúc (`structuredClone`) đảm bảo an toàn tuyệt đối khi trao đổi thông điệp qua ranh giới context hoặc cầu nối JSAPI native:
- Hỗ trợ các đồ thị đối tượng phức tạp, có chu trình, Map, Set, Blob, ArrayBuffer.
- **Tước bỏ nguyên mẫu (Prototype Stripping):** Loại bỏ chuỗi nguyên mẫu và các phương thức thực thi, ngăn chặn triệt để lỗ hổng Prototype Pollution.
- **Chuyển giao quyền sở hữu bộ nhớ:** Hỗ trợ `ArrayBuffer` và `MessagePort` với chi phí sao chép bằng 0 (`zero-copy memory transfer`), tối ưu hóa hiệu năng đồ họa và truyền tải dữ liệu lớn.

### 4.3. WHATWG StorageEvent & Kiến Trúc Đồng Bộ Đa Phiên Bản (Multi-Instance)
- Lắng nghe sự kiện `StorageEvent` trên `window` khi có biến động `localStorage` từ các cửa sổ khác cùng origin.
- Phối hợp đồng bộ giỏ hàng thời gian thực, truyền thông làm mới access token giữa các view đang chạy ngầm, và phát tín hiệu đăng xuất đồng loạt (`Global Logout Broadcast`).

### 4.4. Quản Trị Bus Sự Kiện Trên Super-App
Container thiết lập các giới hạn bảo vệ:
- Khống chế kích thước gói tin tối đa **64KB** (các tệp lớn bắt buộc lưu vào IndexedDB/OPFS và chỉ truyền URI tham chiếu).
- Tự động gắn tiền tố không gian tên (`miniapp_${appId}_${channelName}`) để ngăn va chạm kênh.
- Giới hạn tốc độ phát tin (tối đa 100 thông điệp/giây, burst 200) để chống tấn công từ chối dịch vụ (DoS) treo luồng giao diện.

---

## 5. Bảng Đối Chiếu Ma Trận Kiểm Soát Kỹ Thuật (Control Matrix)

| ID Phát Hiện | Lĩnh Vực Kiểm Soát | Tiêu Chuẩn Nòng Cốt | Cơ Chế Triển Khai Trên Super-App | Cấp Bằng Chứng |
|---|---|---|---|---|
| `csd_storage_032_01` | Xóa dữ liệu máy khách | W3C Clear-Site-Data | Gửi header `Clear-Site-Data: "*"` khi đăng xuất, native container dọn dẹp cache | `working_draft` |
| `csd_storage_032_02` | Kích hoạt vòng đời | Lifecycle Governance | Purge storage khi user logout, switch account, app uninstall, incident wipe | `platform_practice` |
| `csd_storage_032_03` | API Native WebView | WebKit / Android WebStorage | Trừu tượng hóa `WKWebsiteDataStore` & `WebStorage.deleteOrigin()` | `platform_practice` |
| `csd_storage_032_04` | Bảo vệ dữ liệu tồn dư | OWASP MASVS-STORAGE | Cấm lưu token trong Web Storage; cung cấp API lưu trữ mã hóa phần cứng | `open-specification` |
| `csd_storage_032_05` | Quota & Thu hồi LRU | Quota Governance | Hạn mức 10MB-50MB; giải phóng cache tự động theo giải thuật LRU | `platform_practice` |
| `saa_part_032_01` | Ủy quyền truy cập storage | PrivacyCG Storage Access API | `requestStorageAccess()` có cử chỉ người dùng cho widget nhúng | `open-specification` |
| `saa_part_032_02` | Non-cookie storage SAA | PrivacyCG Non-Cookie Storage | Cho phép truy cập unpartitioned IndexedDB/localStorage có giới hạn | `open-specification` |
| `saa_part_032_03` | Ủy quyền từ Top-Level | PrivacyCG requestStorageAccessFor | Super-App chủ động cấp quyền cho mini-app đối tác đã xác minh | `open-specification` |
| `saa_part_032_04` | Phân vùng cookie kép | IETF CHIPS | Khóa kép `{top-level site, partition key}` với cờ `Partitioned` | `standards-track-draft` |
| `saa_part_032_05` | Cô lập đối tác đa thuê | Container Partitioning | Phân vùng mặc định toàn bộ widget bên thứ ba; liên kết bằng bridge token | `platform_practice` |
| `bc_clone_032_01` | Giao tiếp Broadcast | WHATWG BroadcastChannel | Kênh pub-sub cùng origin hỗ trợ đa trang mini-app | `living_standard` |
| `bc_clone_032_02` | Nhân bản có cấu trúc | WHATWG Structured Clone | Tước bỏ prototype, chuyển giao ArrayBuffer zero-copy qua bridge | `living_standard` |
| `bc_clone_032_03` | Đồng bộ StorageEvent | WHATWG Web Storage | Lắng nghe biến động `localStorage` để đồng bộ view tức thời | `living_standard` |
| `bc_clone_032_04` | Đồng bộ đa instance | Multi-Instance Architecture | Đồng bộ giỏ hàng, refresh token ngầm, đăng xuất đồng loạt | `platform_practice` |
| `bc_clone_032_05` | Quản trị bus sự kiện | Event Bus Governance | Giới hạn 64KB/tin, tiền tố định danh namespace, rate limit 100 msg/s | `platform_practice` |

---

## 6. Lộ Trình Áp Dụng Cho Super-App MVP và Phiên Bản Doanh Nghiệp

1. **Giai đoạn MVP:**
   - Triển khai `MiniAppStorageManager` tích hợp gọi `WKWebsiteDataStore` (iOS) và `WebStorage` (Android) khi người dùng đóng/xóa mini-app.
   - Bắt buộc trả về header `Clear-Site-Data: "*"` tại endpoint đăng xuất người dùng.
   - Kích hoạt phân vùng mặc định cho cookie và Web Storage của mini-app.
   - Hạn mức lưu trữ mặc định 10MB mỗi mini-app.
2. **Giai đoạn Hoàn Thiện & Mở Rộng Doanh Nghiệp:**
   - Hỗ trợ đầy đủ PrivacyCG `Storage Access API` và `requestStorageAccessFor` cho mạng lưới đối tác hệ sinh thái.
   - Áp dụng thuật toán LRU tự động quản trị cache máy khách khi bộ nhớ đầy.
   - Chuẩn hóa bus sự kiện liên ngữ cảnh với `BroadcastChannel` và thuật toán kiểm soát tốc độ (rate-limiting) chống tấn công DoS.
