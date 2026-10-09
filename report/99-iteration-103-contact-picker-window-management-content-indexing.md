# Chuyên Đề 99 (Iteration 103): Contact Picker API, Window Management API & Content Indexing API Governance

## 1. Bối Cảnh & Mục Tiêu Chuẩn Hóa
Trong kiến trúc Super App hiện đại, sự hội tụ giữa nền tảng web container và các năng lực bản địa của hệ điều hành (native OS integration) đòi hỏi các chuẩn giao tiếp bảo mật cao, tôn trọng quyền riêng tư của người dùng và tối ưu hóa khả năng hoạt động ngoại tuyến.

Chuyên đề 99 thiết lập chuẩn hóa kỹ thuật cho 3 trụ cột giao tiếp bản địa quan trọng:
1. **Kiến Trúc Truy Cập Danh Bạ Tối Giản Quyền Riêng Tư (Contact Picker API & ContactsManager)**: Cho phép mini app yêu cầu thông tin liên hệ từ danh bạ thiết bị theo nhu cầu người dùng (on-demand) mà không cấp quyền đọc toàn bộ danh bạ liên tục, phục vụ chuyển tiền P2P, nạp thẻ điện thoại và chia sẻ hóa đơn.
2. **Quản Trị Đa Màn Hình & Thiết Bị POS/Kiosk Thương Mại (Window Management API & Multi-Screen Topologies)**: Chuẩn hóa cơ chế phát hiện màn hình phụ, quản lý tọa độ hiển thị và điều phối cửa sổ thanh toán đối diện khách hàng (customer-facing display) cho các mini app bán hàng (POS).
3. **Mục Lục Nội Dung Ngoại Tuyến & Đồng Bộ Danh Mục Bản Địa (Content Indexing API & Offline Discovery)**: Cho phép mini app đăng ký các tài nguyên đã lưu trong bộ nhớ đệm (CacheStorage/IndexedDB) vào mục lục tìm kiếm ngoại tuyến tập trung của Super App và hệ điều hành, hỗ trợ trải nghiệm offline tức thì.

---

## 2. Bảng Ma Trận Quy Chuẩn Kỹ Thuật (15 Thành Phần Chuẩn Hóa)

| ID Quy Chuẩn | Giao Diện Chuẩn | Chuẩn Tham Chiếu | Cơ Chế Kỹ Thuật Cốt Lõi | Giá Trị Đối Với Super App Container | Quy Tắc Kiểm Duyệt Store (Store Review) |
|---|---|---|---|---|---|
| `STANDARDS-CONTACT-PICKER-API` | `Contact Picker API` | W3C / WICG Contact Picker API | Hoạt động độc quyền trong Secure Contexts (HTTPS); đòi hỏi kích hoạt tức thời từ người dùng (transient activation) để mở giao diện chọn danh bạ hệ thống; người dùng chủ động chọn mục liên hệ cần chia sẻ. | Loại bỏ rủi ro bảo mật nghiêm trọng khi không phải cấp quyền đọc toàn bộ danh bạ (`READ_CONTACTS`) cho code web bên thứ ba. | Mini app chuyển tiền hoặc nạp tiền điện thoại bắt buộc phải sử dụng Contact Picker API thay cho các container bridge cũ đòi quyền đọc danh bạ toàn diện. |
| `STANDARDS-NAVIGATOR-CONTACTS` | `Navigator.contacts` | W3C / WICG Contact Picker API §2 | Thuộc tính chỉ đọc trả về đối tượng singleton `ContactsManager` khi runtime và hệ điều hành hỗ trợ; trả về `undefined` nếu không hỗ trợ hoặc chạy trong iframe không có delegation. | Giúp mini app nhận biết năng lực hệ thống để tự động chuyển đổi giữa giao diện chọn danh bạ bản địa và biểu mẫu nhập thủ công. | Bắt buộc mini app phải kiểm tra tính khả dụng (`'contacts' in navigator`) trước khi gọi và cung cấp fallback nhập tay nếu môi trường không hỗ trợ. |
| `STANDARDS-CONTACTS-MANAGER` | `ContactsManager` | W3C / WICG Contact Picker API §3 | Đóng gói các phương thức bất đồng bộ `select()` và `getProperties()`; chuẩn hóa dữ liệu trả về theo lược đồ `ContactInfo` thống nhất. | Đồng nhất hợp đồng giao tiếp danh bạ trên cả Android, iOS (thông qua WebKit container polyfill) và desktop client. | Container phải xử lý lỗi chuẩn khi người dùng đóng hộp thoại hủy bỏ mà không làm treo hay crash mini app. |
| `STANDARDS-CONTACTS-MANAGER-SELECT` | `ContactsManager.select()` | W3C / WICG Contact Picker API §3.2 | Nhận mảng các trường dữ liệu cần lấy (name, tel, email) và tùy chọn `multiple`; trả về mảng `ContactInfo`; ném lỗi `SecurityError` nếu không có tương tác người dùng trực tiếp. | Hỗ trợ chọn danh bạ đơn lẻ hoặc hàng loạt (chia tiền nhóm, gửi lì xì) an toàn, ngăn chặn script độc hại tự động mở pop-up rác. | Nghiêm cấm gọi `select()` từ hàm hẹn giờ (`setTimeout`), sự kiện tải trang (`onload`) hoặc luồng ngầm; vi phạm sẽ bị từ chối phát hành. |
| `STANDARDS-CONTACTS-MANAGER-GETPROPERTIES` | `ContactsManager.getProperties()` | W3C / WICG Contact Picker API §3.1 | Phương thức bất đồng bộ trả về danh sách các trường được hệ điều hành máy chủ hỗ trợ (`address`, `email`, `icon`, `name`, `tel`). | Thực thi nguyên tắc tối thiểu hóa dữ liệu (GDPR data minimization); mini app chỉ yêu cầu đúng trường phục vụ tính năng. | Mini app chỉ được yêu cầu đúng trường cần thiết (ví dụ chỉ lấy `tel` khi nạp thẻ điện thoại); kiểm duyệt store sẽ đối chiếu mã nguồn với mục đích nghiệp vụ. |
| `STANDARDS-WINDOW-MANAGEMENT-API` | `Window Management API` | W3C Window Management API §1 | Mở rộng giao diện Screen và Window; bảo vệ bởi quyền `window-management`; cung cấp thông tin hình học các màn hình vật lý mà không làm rò rỉ vân tay thiết bị (anti-fingerprinting). | Nền tảng cho mini app thu ngân / POS hiển thị giao diện thanh toán sang màn hình phụ đối diện khách hàng. | Mini app xin quyền `window-management` phải giải trình rõ nghiệp vụ (như POS đa màn hình hoặc bảng điều khiển kiosk) trong hồ sơ duyệt store. |
| `STANDARDS-WINDOW-GETSCREENDETAILS` | `Window.getScreenDetails()` | W3C Window Management API §4 | Trả về Promise chứa đối tượng `ScreenDetails`; kích hoạt hộp thoại xin quyền hệ thống nếu chưa cấp; từ chối với `NotAllowedError` nếu người dùng từ chối. | Đảm bảo luồng cấp quyền đa màn hình minh bạch, ngăn chặn mã độc âm thầm theo dõi số lượng màn hình phụ của người dùng. | Bắt buộc gọi `getScreenDetails()` từ thao tác cấu hình của người dùng (ví dụ bấm nút 'Kết nối màn hình phụ') thay vì gọi tự động khi khởi động. |
| `STANDARDS-SCREENDETAILS-INTERFACE` | `ScreenDetails` | W3C Window Management API §5 | Đại diện cho cấu hình đa màn hình gồm danh sách tất cả màn hình (`screens`) và con trỏ màn hình hiện tại (`currentScreen`); phát sự kiện `screenschange` và `currentscreenchange`. | Cho phép mini app tính toán tọa độ tuyệt đối để bung cửa sổ thanh toán lên màn hình phụ với độ chính xác từng pixel. | Mini app phải lắng nghe sự kiện `currentscreenchange` để xử lý khi người dùng kéo cửa sổ qua màn hình có độ phân giải hoặc DPI khác nhau. |
| `STANDARDS-SCREENDETAILED-INTERFACE` | `ScreenDetailed` | W3C Window Management API §6 | Kế thừa từ `Screen`; bổ sung `isPrimary` (màn hình chính), `isInternal` (màn hình tích hợp laptop/tablet), `devicePixelRatio` và nhãn màn hình `label`. | Tự động phân luồng hiển thị: mini app nhận diện ngay màn hình phụ bên ngoài (`!isInternal`) mà không cần thu ngân kéo thả thủ công. | Nghiêm cấm thu thập chuỗi nhãn màn hình (`label`) để theo dõi hoặc nhận dạng người dùng xuyên suốt các phiên. |
| `STANDARDS-SCREENDETAILS-SCREENS` | `ScreenDetails.screens` | W3C Window Management API §5.1 | Mảng đóng băng (frozen array) chứa các đối tượng `ScreenDetailed`; cập nhật và phát sự kiện khi có màn hình cắm vào hoặc rút ra. | Đảm bảo ứng dụng POS tự động thu hồi giao diện khách hàng về màn hình chính khi dây cáp màn hình phụ bị ngắt kết nối đột ngột. | Mini app sử dụng màn hình phụ bắt buộc phải triển khai hàm xử lý `screenschange` để thu gọn cửa sổ phụ an toàn khi ngắt kết nối. |
| `STANDARDS-CONTENT-INDEX-API` | `Content Indexing API` | WICG Content Indexing API §1 | Gắn trực tiếp trên `ServiceWorkerRegistration.index`; cho phép mini app đăng ký siêu dữ liệu nội dung đã lưu offline vào bộ mục lục hệ thống. | Tạo trung tâm nội dung ngoại tuyến ('Downloaded Hub') tập trung cho Super App, mở vé xem phim, bài báo hoặc voucher offline tức thì. | Tài nguyên đăng ký vào Content Index bắt buộc phải được nạp đầy đủ trong CacheStorage trước khi gọi hàm đăng ký. |
| `STANDARDS-CONTENTINDEX-INTERFACE` | `ContentIndex` | WICG Content Indexing API §3 | Cung cấp các phương thức `add()`, `delete()`, `getAll()` bất đồng bộ; định phạm vi theo nguồn gốc và Service Worker. | Chuẩn hóa vòng đời lập chỉ mục nội dung cho các đối tác mini app tích hợp vào thanh tìm kiếm tổng của Super App. | Mini app phải duy trì sự đồng bộ giữa bộ nhớ cache thực tế và ContentIndex; không để liên kết chết trong mục lục. |
| `STANDARDS-CONTENTINDEX-ADD` | `ContentIndex.add()` | WICG Content Indexing API §3.1 | Nhận đối tượng `ContentDescription` gồm `id`, `title`, `description`, `category` (homepage, article, video, audio), `url`, `icons`. | Hiển thị các bài viết tin tức, tập podcast hoặc vé máy bay đã tải về trực tiếp trên ngăn kéo ngoại tuyến của Super App. | Nội dung phải được phân loại danh mục (`category`) chính xác; nghiêm cấm gán nhãn sai để lừa người dùng mở app. |
| `STANDARDS-CONTENTINDEX-DELETE` | `ContentIndex.delete()` | WICG Content Indexing API §3.2 | Nhận `id` và gỡ bỏ mục tương ứng khỏi chỉ mục ngoại tuyến; không ném ngoại lệ nếu `id` không tồn tại. | Dọn dẹp vé đã sử dụng hoặc bài báo đã xóa, giữ cho danh mục ngoại tuyến của Super App luôn cập nhật và sạch sẽ. | Mini app có tính năng xóa tải về phải gọi kèm `ContentIndex.delete()` để xóa biểu tượng khỏi giao diện hệ thống. |
| `STANDARDS-SW-REGISTRATION-INDEX` | `ServiceWorkerRegistration.index` | WICG Content Indexing API §2 | Thuộc tính chỉ đọc trên `ServiceWorkerRegistration` trả về đối tượng `ContentIndex`; dùng được cả trong Window và ServiceWorker. | Cho phép các tiến trình nền (Background Sync, Periodic Sync) tự động cập nhật nội dung ngoại tuyến khi thiết bị rảnh rỗi. | Mini app làm mới nội dung định kỳ ngầm phải truy cập thông qua `registration.index` bên trong luồng Service Worker. |

---

## 3. Kiến Trúc Tương Tác Native Của Super App Web Container

```
+-----------------------------------------------------------------------------------------+
|                               SUPER APP NATIVE RUNTIME SHELL                            |
|                                                                                         |
|  +-------------------------+  +----------------------------+  +----------------------+  |
|  | Native Address Book     |  | Multi-Display Window Server|  | Unified Offline Hub  |  |
|  | (OS Contacts Provider)  |  | (Screen Manager / Wayland) |  | (System Search Index)|  |
|  +------------+------------+  +-------------+--------------+  +----------+-----------+  |
|               |                             |                            |              |
|   Modal Sheet | User Disclosed              | Dual Coordinates           | Index Sync   |
|   (Permission)| Contact Records             | (Merchant / Customer)      | Metadata     |
|               v                             v                            v              |
|  +-------------------------+  +----------------------------+  +----------------------+  |
|  | ContactsManager Bridge  |  | WindowManagement Bridge    |  | ContentIndex Service |  |
|  | - select() filter       |  | - getScreenDetails()       |  | - add() / delete()   |  |
|  | - getProperties() check |  | - screenschange hot-plug   |  | - getAll() audit     |  |
|  +------------+------------+  +-------------+--------------+  +----------+-----------+  |
|               |                             |                            |              |
+---------------|-----------------------------|----------------------------|--------------+
|               v                             v                            v              |
|  +-----------------------------------------------------------------------------------+  |
|  |                        MINI APP WEB CONTAINER RUNTIME                             |  |
|  |                                                                                   |  |
|  |  +-----------------------------------------------------------------------------+  |  |
|  |  | Window Execution Context / Secure Context (HTTPS)                           |  |  |
|  |  |  - navigator.contacts.select(['name', 'tel'], {multiple: false})             |  |  |
|  |  |  - window.getScreenDetails() -> ScreenDetailed (isPrimary, !isInternal)     |  |  |
|  |  +-----------------------------------------------------------------------------+  |  |
|  |                                                                                   |  |
|  |  +-----------------------------------------------------------------------------+  |  |
|  |  | ServiceWorker Context (sw.js)                                                |  |  |
|  |  |  - self.registration.index.add({id, title, category: 'article', url, icons})|  |  |
|  |  |  - self.registration.index.delete(staleId)                                  |  |  |
|  |  +-----------------------------------------------------------------------------+  |  |
+-----------------------------------------------------------------------------------------+
```

---

## 4. Nguyên Tắc Kiểm Duyệt & Vận Hành Trên Mini App Store
1. **Kiểm Soát Quyền Riêng Tư Danh Bạ**:
   - Mọi mini app yêu cầu thông tin danh bạ phải sử dụng phương thức phân giải tối thiểu dữ liệu (`ContactsManager.getProperties()`), chỉ truy vấn các thuộc tính trực tiếp phục vụ giao dịch.
   - Cấm lưu trữ hàng loạt thông tin liên hệ của bên thứ ba khi chưa có sự đồng ý rõ ràng.
2. **Quản Lý Cửa Sổ Đa Màn Hình Bán Hàng**:
   - Mini app triển khai giao diện màn hình phụ phải đăng ký phạm vi sử dụng trong file cấu hình manifest.
   - Bắt buộc xử lý biến cố ngắt kết nối màn hình (`screenschange`) bằng cách tự động kéo cửa sổ hoặc đưa giao diện phụ về trạng thái chờ, tránh gây treo giao diện thanh toán.
3. **Đồng Bộ Dữ Liệu Ngoại Tuyến**:
   - Khi mini app bị gỡ cài đặt hoặc người dùng xóa dữ liệu cục bộ, container Super App sẽ tự động quét và thu hồi toàn bộ các bản ghi `ContentIndex` liên quan để tránh hiện tượng liên kết ma (ghost links) trong thanh tìm kiếm hệ điều hành.
