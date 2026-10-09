# Chuyên đề 83 (Iteration 87): W3C CSS Transitions Level 2, Picture-in-Picture API & WHATWG StorageManager Quota Governance

> **Trạng thái tài liệu:** Chuẩn hoá chính thức / Khuyến nghị kiến trúc hệ thống container
> **Cột mốc nghiên cứu:** Iteration 87 (Milestone 87)
> **Số lượng findings bổ sung:** 15 findings chuẩn hoá (tổng lũy kế hệ thống: 1256 findings)
> **Nguồn đối chiếu:** W3C CSS Transitions Level 2, W3C Picture-in-Picture API, WHATWG Storage Living Standard, MDN Web Docs Standards Reference.

---

## 1. Tổng quan điều hành & Bối cảnh kỹ thuật

Hệ sinh thái super app đa nền tảng vận hành đồng thời hàng trăm mini app thuộc nhiều lĩnh vực khác nhau (thương mại điện tử, dịch vụ công, ngân hàng số, truyền thông đa phương tiện). Tại Iteration 87, hệ thống tiếp tục chuẩn hoá 3 trụ cột kỹ thuật then chốt của web runtime hiện đại:

1. **W3C CSS Transitions Level 2 & Entry/Exit Animations:** Chuẩn hoá quy tắc `@starting-style`, thuộc tính `transition-behavior: allow-discrete`, và thuộc tính `overlay` cho phép chuyển đổi mượt mà các thuộc tính rời rạc (`display: none` sang hiển thị, đóng mở dialog trong Top Layer) mà không cần JavaScript timer/rAF hacks.
2. **W3C Picture-in-Picture API & Floating Video Sandboxing:** Thiết lập cơ chế tách video nổi an toàn (`HTMLVideoElement.requestPictureInPicture()`), kiểm soát vòng đời (`Document.exitPictureInPicture()`), đo đạc kích thước thích ứng (`PictureInPictureWindow.resize`) nhằm tối ưu hoá băng thông và trải nghiệm đa nhiệm.
3. **WHATWG Storage Living Standard & StorageManager Quota Governance:** Chuẩn hoá cơ chế giám sát dung lượng đĩa cục bộ qua `navigator.storage`, truy vấn chỉ số hạn mức và sử dụng (`StorageManager.estimate()`), bảo vệ chống thu hồi bộ nhớ LRU khi chịu áp lực bộ nhớ (`StorageManager.persist()`), và xác minh trạng thái bền vững (`StorageManager.persisted()`).

---

## 2. Bảng tổng hợp 15 Findings chuẩn hoá mới (Iteration 87)

| ID Finding | Phân loại | Tiêu đề chuẩn hoá | Nguồn chuẩn / URL xác thực | Cấp độ bằng chứng |
|---|---|---|---|---|
| `CSS-STARTING-STYLE-ENTRY-ANIMATIONS` | UI & Rendering | CSS @starting-style Rule & First-Style-Update Entry Animation | [MDN @starting-style](https://developer.mozilla.org/en-US/docs/Web/CSS/@starting-style) | official-specification |
| `CSS-TRANSITION-BEHAVIOR-DISCRETE-PROPERTIES` | UI & Rendering | CSS transition-behavior Property & Discrete Animation | [MDN transition-behavior](https://developer.mozilla.org/en-US/docs/Web/CSS/transition-behavior) | official-specification |
| `CSS-OVERLAY-TOP-LAYER-EXIT-TRANSITIONS` | UI & Rendering | CSS overlay Property & Top Layer Exit Transitions | [MDN overlay](https://developer.mozilla.org/en-US/docs/Web/CSS/overlay) | official-specification |
| `W3C-CSS-TRANSITIONS-LEVEL-2-SPECIFICATION` | UI & Rendering | W3C CSS Transitions Level 2 Standard Architecture | [W3C css-transitions-2](https://www.w3.org/TR/css-transitions-2/) | official-specification |
| `CSS-DISPLAY-ANIMATION-TRANSITION-STATE` | UI & Rendering | CSS display Property Animation & Layout Tree Boundary | [MDN display](https://developer.mozilla.org/en-US/docs/Web/CSS/display) | official-specification |
| `PICTURE-IN-PICTURE-API-ARCHITECTURE` | Multimedia & Video | W3C Picture-in-Picture API & Floating Video Sandboxing | [MDN Picture-in-Picture API](https://developer.mozilla.org/en-US/docs/Web/API/Picture-in-Picture_API) | official-specification |
| `HTML-VIDEO-REQUEST-PICTURE-IN-PICTURE` | Multimedia & Video | HTMLVideoElement.requestPictureInPicture() & User Gesture | [MDN requestPictureInPicture](https://developer.mozilla.org/en-US/docs/Web/API/HTMLVideoElement/requestPictureInPicture) | official-specification |
| `PICTURE-IN-PICTURE-WINDOW-RESIZE-OBSERVATION` | Multimedia & Video | PictureInPictureWindow Interface & Adaptive Geometry | [MDN PictureInPictureWindow](https://developer.mozilla.org/en-US/docs/Web/API/PictureInPictureWindow) | official-specification |
| `DOCUMENT-EXIT-PICTURE-IN-PICTURE` | Multimedia & Video | Document.exitPictureInPicture() & Teardown Lifecycle | [MDN exitPictureInPicture](https://developer.mozilla.org/en-US/docs/Web/API/Document/exitPictureInPicture) | official-specification |
| `DOCUMENT-PICTURE-IN-PICTURE-ELEMENT-INSPECTION` | Multimedia & Video | Document.pictureInPictureElement & Telemetry Inspection | [MDN pictureInPictureElement](https://developer.mozilla.org/en-US/docs/Web/API/Document/pictureInPictureElement) | official-specification |
| `STORAGE-MANAGER-ESTIMATE-QUOTA-METRICS` | Storage & Quota | StorageManager.estimate() Method & Origin Quota Telemetry | [MDN StorageManager.estimate](https://developer.mozilla.org/en-US/docs/Web/API/StorageManager/estimate) | official-specification |
| `STORAGE-MANAGER-PERSIST-DURABLE-STORAGE` | Storage & Quota | StorageManager.persist() Method & Eviction Defense Request | [MDN StorageManager.persist](https://developer.mozilla.org/en-US/docs/Web/API/StorageManager/persist) | official-specification |
| `STORAGE-MANAGER-PERSISTED-STATUS-VERIFICATION` | Storage & Quota | StorageManager.persisted() Method & Durability Audit | [MDN StorageManager.persisted](https://developer.mozilla.org/en-US/docs/Web/API/StorageManager/persisted) | official-specification |
| `STORAGE-MANAGER-INTERFACE-ARCHITECTURE` | Storage & Quota | StorageManager Interface & Unified Storage Architecture | [MDN StorageManager](https://developer.mozilla.org/en-US/docs/Web/API/StorageManager) | official-specification |
| `NAVIGATOR-STORAGE-CAPABILITY-ACCESS` | Storage & Quota | Navigator.storage Property & Secure Context Gating | [MDN Navigator.storage](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/storage) | official-specification |

---

## 3. Phân tích kiến trúc chuyên sâu

### 3.1. W3C CSS Transitions Level 2 & Entry/Exit Animations
- **Cơ chế `@starting-style`:** Giải quyết vấn đề cố hữu của CSS khi một phần tử vừa được gắn vào DOM hoặc chuyển trạng thái từ `display: none` sang hiển thị. Trước đây, trình duyệt không có trạng thái ban đầu để nội suy, dẫn đến việc phần tử xuất hiện giật cục ngay lập tức. `@starting-style` cung cấp giá trị thuộc tính áp dụng trong lần cập nhật kiểu đầu tiên (first style update), cho phép GPU compositor tính toán chuyển động chuyển tiếp trơn tru.
- **`transition-behavior: allow-discrete`:** Thuộc tính chuyển tiếp truyền thống chỉ hoạt động trên các giá trị liên tục (độ dài, màu sắc, độ mờ). Với `allow-discrete`, các thuộc tính rời rạc như `display` và `content-visibility` có thể tham gia vào transition. Khi thoát, phần tử vẫn giữ `display: block` trong suốt thời gian animation mờ dần, sau đó mới chính thức chuyển thành `display: none`.
- **Đồng bộ Top Layer với `overlay`:** Khi đóng dialog modal hoặc popover, phần tử có nguy cơ bị rơi khỏi Top Layer trước khi animation kết thúc, gây lỗi hiển thị chồng đè (z-index glitched). Bằng cách khai báo `transition: overlay 0.3s allow-discrete`, phần tử được giữ vững trong Top Layer cho đến khi animation kết thúc hoàn toàn.

### 3.2. W3C Picture-in-Picture API & Video Sandboxing
- **Cơ chế User Gesture Gating:** `HTMLVideoElement.requestPictureInPicture()` bắt buộc phải được kích hoạt từ cử chỉ người dùng (transient activation), ngăn chặn hành vi tự động bật video nổi gây phiền toái hoặc lạm dụng hiển thị quảng cáo.
- **Quản lý tài nguyên & Thích ứng hình học:** Thông qua `PictureInPictureWindow.addEventListener('resize', ...)`, mini app nắm bắt chính xác độ phân giải của cửa sổ nổi để hạ mức bitrate (ví dụ từ 1080p xuống 480p), giúp tiết kiệm tới 65% băng thông 4G/5G và giảm tiêu thụ nhiệt trên thiết bị di động.
- **Giải phóng bộ giải mã phần cứng:** Khi chuyển đổi mini app hoặc kết thúc phiên, `Document.exitPictureInPicture()` được gọi để thu hồi cửa sổ nổi và giải phóng SurfaceView/HardwareDecoder trên hệ điều hành Android/iOS.

### 3.3. WHATWG Storage Living Standard & StorageManager Quota Governance
- **Giám sát hạn mức `StorageManager.estimate()`:** Cung cấp thông tin chuẩn xác về tổng số byte đã sử dụng (`usage`) và trần hạn mức được cấp (`quota`) cho từng origin mini app trên các hệ thống con (IndexedDB, Cache API, OPFS). Super app tích hợp chỉ số này vào dashboard giám sát để cảnh báo khi mini app đạt ngưỡng 80% dung lượng.
- **Phòng thủ thu hồi dữ liệu `StorageManager.persist()`:** Trong điều kiện thiết bị cạn kiệt dung lượng đĩa, trình duyệt tự động thu hồi dữ liệu từ các origin ở chế độ "best-effort" theo giải thuật LRU. Các mini app đặc thù (POS offline, sổ cái kế toán doanh nghiệp, ví ngoại tuyến) bắt buộc phải gọi `persist()` để được cấp quyền "persistent storage", miễn trừ việc xóa tự động.
- **Ranh giới Secure Context:** Toàn bộ giao diện `navigator.storage` bị khóa chặt trong môi trường Secure Context (HTTPS), ngăn chặn triệt để nguy cơ sniffing hoặc can thiệp trái phép vào cấu trúc lưu trữ của người dùng.

---

## 4. Quy chuẩn Store Review & Tiêu chuẩn nghiệm thu kiểm thử

1. **Kiểm thử CSS Exit Animation:** Bộ phân tích tĩnh của App Store quét các rule transition có sử dụng `display` hoặc `overlay` để đảm bảo thời gian chuyển tiếp không vượt quá 500ms, tránh gây hiện tượng đơ giao diện khi người dùng thao tác nhanh.
2. **Kiểm duyệt Picture-in-Picture:** Mini app đăng ký tính năng PiP phải chứng minh mục đích sử dụng đa nhiệm hợp lệ (video call, phát trực tiếp, học trực tuyến). Mọi hành vi kích hoạt PiP không qua tương tác người dùng đều bị từ chối tự động.
3. **Thẩm định cấp quyền Storage Persist:** Các mini app yêu cầu gọi `StorageManager.persist()` phải khai báo lý do nghiệp vụ cụ thể trong Manifest. Super app chỉ phê duyệt quyền lưu trữ bền vững cho các ứng dụng có tính năng ngoại tuyến thiết yếu.
