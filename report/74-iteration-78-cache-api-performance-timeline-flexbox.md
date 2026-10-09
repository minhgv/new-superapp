# Chuyên đề 74: W3C Cache API, W3C Performance Timeline Level 2 & CSS Flexible Box Layout Level 1

## 1. Tổng quan nghiên cứu Milestone 78

Milestone 78 tập trung hoàn thiện 3 trụ cột công nghệ nền tảng quyết định khả năng lưu trữ ngoại tuyến tài nguyên gói lập trình được, quan sát hiệu năng phía client bất đồng bộ độ trễ thấp và động cơ dàn trang giao diện co giãn đa màn hình trong kiến trúc siêu ứng dụng (Super-App Mini-App Store Standard):

1. **W3C Cache API & Service Worker Storage Architecture**:
   - Đặc tả chuẩn W3C Cache API cung cấp cơ chế lưu trữ lập trình được (programmable storage) cho các cặp đối tượng Request/Response. Khác với bộ nhớ đệm HTTP Header (RFC 9111) được quản lý tự động bởi tầng mạng trình duyệt, Cache API trao quyền kiểm soát chủ động và tất định cho Service Worker trong việc nạp, khớp và truy xuất tài nguyên gói mini-app.
   - Quản trị không gian tên đa phiên bản thông qua giao diện `CacheStorage` (`window.caches` hoặc `self.caches`). Hỗ trợ cô lập tài nguyên độc lập qua `caches.open(cacheName)` và thực thi quy trình dọn dẹp các gói ứng dụng lỗi thời trong sự kiện Service Worker `activate` thông qua `caches.delete()`, ngăn ngừa ô nhiễm bộ nhớ đệm khi phát hành bản cập nhật OTA.
   - Tối ưu hóa khớp tài nguyên với phương thức `Cache.match(request, options)` và từ điển cấu hình `CacheQueryOptions`. Kích hoạt cờ `ignoreSearch: true` giúp bỏ qua các tham số truy vấn URL động (như mã giới thiệu marketing, UTM tracking), bảo đảm app shell tĩnh lưu cache luôn được khớp thành công mà không gây lỗi 404 ngoại tuyến hoặc gửi request mạng thừa.
   - Bảo đảm tính toàn vẹn cài đặt với phương thức `Cache.addAll(requests)`. Quy trình nạp gói tài nguyên hoạt động theo nguyên lý nguyên tử (atomic): nếu bất kỳ một file script hoặc stylesheet nào gặp sự cố mạng (HTTP 404/500), toàn bộ thao tác bị hủy bỏ và không có tài nguyên nào được lưu vào cache, triệt tiêu hoàn toàn rủi ro khởi chạy mini-app ở trạng thái hỏng hóc hoặc thiếu file.
   - Quản trị hạn mức bộ nhớ đệm (Storage Quota Governance) tích hợp với Storage Standard (`navigator.storage.estimate()`). Áp đặt trần dung lượng lưu trữ (ví dụ: tối đa 50MB cho mỗi mini-app khách) và khuyến nghị cơ chế giải phóng LRU để tránh bị hệ điều hành xóa dữ liệu khi thiết bị gặp áp lực dung lượng đĩa.

2. **W3C Performance Timeline Level 2 & Client Runtime Telemetry**:
   - Chuẩn hóa mô hình đối tượng quan sát hiệu năng hợp nhất W3C Performance Timeline Level 2 (`window.performance`), cung cấp hạ tầng đo lường chính xác với dấu thời gian độ phân giải cao `DOMHighResTimeStamp` tương đối theo `timeOrigin`.
   - Thu thập vi sai bất đồng bộ không gây nghẽn thông qua giao diện `PerformanceObserver`. Thay thế hoàn toàn cơ chế thăm dò đồng bộ `performance.getEntries()`, loại bỏ áp lực dọn rác bộ nhớ (GC churn) và triệt tiêu độ trễ luồng chính, bảo toàn chỉ số phản hồi tương tác Interaction to Next Paint (INP < 200ms).
   - Tùy chọn cấu hình `PerformanceObserver.observe()` với cờ `buffered: true`. Cho phép các SDK giám sát tải chậm (lazy-loaded analytics) thu hồi đầy đủ các bản ghi hiệu năng khởi động cốt lõi (navigation, paint, early resource timings) đã phát sinh trước khi observer được khởi tạo.
   - Trừu tượng hóa đối tượng cơ sở `PerformanceEntry` với các thuộc tính chuẩn hóa: `name`, `entryType`, `startTime`, `duration`, và phương thức tuần tự hóa `toJSON()`, hỗ trợ xuất dữ liệu viễn trắc có cấu trúc về gateway phân tích của chợ ứng dụng.
   - Kiểm soát dung lượng bộ đệm hiệu năng hệ thống (Buffer Governance): thiết lập cảnh báo tràn bộ đệm `onresourcetimingbufferfull` và chủ động giải phóng bộ nhớ bằng `clearMarks()` / `clearMeasures()`, loại bỏ triệt để hiện tượng rò rỉ bộ nhớ heap trong các phiên làm việc kéo dài của siêu ứng dụng.

3. **W3C CSS Flexible Box Layout Module Level 1 & Responsive UI Foundations**:
   - Đặc tả chuẩn W3C CSS Flexible Box Layout Level 1 cung cấp động cơ dàn trang co giãn tối ưu hóa cho giao diện người dùng, tự động phân bổ không gian và căn chỉnh phần tử theo trục chính (main axis) và trục phụ (cross axis) trên nhiều kích thước màn hình từ smartphone nhỏ đến thiết bị gập (foldable) và máy tính bảng đa nhiệm.
   - Thích ứng quốc tế hóa tự động với `flex-direction`: trục dàn trang tự động đảo chiều theo chiều đọc văn bản `writing-mode` và hướng trang (LTR vs RTL), hỗ trợ bản địa hóa thị trường Trung Đông (tiếng Ả Rập) mà không cần viết lại quy tắc CSS riêng.
   - Phân bổ không gian trục chính bằng `justify-content` (`space-between`, `space-around`, `space-evenly`), bảo đảm thanh điều hướng tab dưới cùng và các nút hành động tiêu đề duy trì nhịp điệu thị giác nhất quán, loại bỏ lỗi làm tròn sub-pixel trên màn hình độ phân giải cao.
   - Căn chỉnh trục phụ và kiểm soát công thái học chạm với `align-items` và `align-self`. Bảo đảm các nút bấm, biểu tượng và danh sách tương tác duy trì vùng chạm vật lý tối thiểu 48x48 CSS pixel, tuân thủ tiêu chuẩn tiếp cận WCAG 2.2 Level AA.
   - Loại bỏ giật khung hình và xê dịch bố cục với cú pháp viết tắt `flex` (`flex-grow`, `flex-shrink`, `flex-basis`). Việc khai báo kích thước cơ sở rõ ràng (`flex: 0 0 <dimensions>`) giúp giữ chỗ bố cục trước khi nạp ảnh/video bất đồng bộ, triệt tiêu hoàn toàn xê dịch bố cục tích lũy Cumulative Layout Shift (CLS < 0.1).

---

## 2. Chi tiết 15 chuẩn kỹ thuật cốt lõi (Normative Findings)

Dưới đây là 15 quy chuẩn kỹ thuật chính thức được trích xuất từ các tài liệu W3C, WHATWG và MDN Web Docs, đã qua kiểm tra hợp lệ, không chứa credential và xác minh HTTP 200:

| # | Khía cạnh / Sự kiện | Nguồn chuẩn | Mức độ chứng cứ | Tóm tắt quy chuẩn và yêu cầu kiến trúc Super-App | URL nguồn đã kiểm tra |
|---|---|---|---|---|---|
| 1 | `cache_api_078_01` | w3c-service-workers | official_standard | W3C Cache API cung cấp cơ chế lưu trữ lập trình được cho các cặp Request/Response trong không gian tên cô lập theo origin; Service Worker sử dụng Cache API để nạp sẵn mã nguồn và app shell, bảo đảm khởi chạy offline tức thì không cần mạng. | `https://www.w3.org/TR/service-workers/` |
| 2 | `cache_api_078_02` | w3c-service-workers | specification | Giao diện CacheStorage quản lý nhiều bộ nhớ đệm có tên (`caches.open`); mini-app bắt buộc đặt tên cache theo phiên bản ngữ nghĩa (v1.4.2) và dọn dẹp các phiên bản cũ trong sự kiện `activate` qua `caches.delete()` để chống tràn bộ nhớ. | `https://developer.mozilla.org/en-US/docs/Web/API/CacheStorage` |
| 3 | `cache_api_078_03` | w3c-service-workers | specification | Phương thức `Cache.match()` hỗ trợ cấu hình `ignoreSearch: true`, ngăn chặn các tham số URL marketing hoặc UTM tracking làm mất khả năng khớp cache của app shell tĩnh, duy trì khả năng mở trang offline mượt mà. | `https://developer.mozilla.org/en-US/docs/Web/API/Cache/match` |
| 4 | `cache_api_078_04` | w3c-service-workers | specification | Phương thức `Cache.addAll()` nạp và lưu nhiều tài nguyên theo nguyên lý nguyên tử; nếu một tài nguyên bất kỳ gặp lỗi mạng thì toàn bộ promise bị reject, ngăn chặn tuyệt đối tình trạng cài đặt mini-app dở dang hoặc thiếu file. | `https://developer.mozilla.org/en-US/docs/Web/API/Cache/addAll` |
| 5 | `cache_api_078_05` | w3c-service-workers | specification | Dữ liệu Cache API chia sẻ chung hạn mức lưu trữ với IndexedDB và OPFS; chợ ứng dụng áp đặt giới hạn tối đa 50MB cache cho mỗi mini-app khách và yêu cầu cơ chế giải phóng LRU để tránh bị OS xóa ngoài ý muốn. | `https://developer.mozilla.org/en-US/docs/Web/API/Cache` |
| 6 | `perf_timeline_078_01` | w3c-performance-timeline-2 | official_standard | W3C Performance Timeline Level 2 thiết lập mô hình đối tượng hợp nhất ghi nhận số liệu hiệu năng với dấu thời gian độ phân giải cao DOMHighResTimeStamp, làm nền tảng cho hệ thống Real User Monitoring (RUM) của super-app. | `https://www.w3.org/TR/performance-timeline-2/` |
| 7 | `perf_timeline_078_02` | w3c-performance-timeline-2 | specification | Giao diện PerformanceObserver phân phối bản ghi hiệu năng bất đồng bộ qua event loop, loại bỏ việc thăm dò định kỳ gây tốn CPU và áp lực dọn rác bộ nhớ, bảo toàn chỉ số phản hồi tương tác INP dưới 200ms. | `https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver` |
| 8 | `perf_timeline_078_03` | w3c-performance-timeline-2 | specification | Cấu hình `PerformanceObserver.observe()` với cờ `buffered: true` cho phép thu hồi các chỉ số hiệu năng khởi động ban đầu (navigation, paint, resource timings) ngay cả khi SDK viễn trắc được nạp chậm (lazy load). | `https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver/observe` |
| 9 | `perf_timeline_078_04` | w3c-performance-timeline-2 | specification | Trừu tượng hóa PerformanceEntry chuẩn hóa các thuộc tính `name`, `entryType`, `startTime`, `duration` và phương thức `toJSON()`, hỗ trợ đóng gói và truyền tải dữ liệu viễn trắc cấu trúc về backend giám sát. | `https://developer.mozilla.org/en-US/docs/Web/API/PerformanceEntry` |
| 10 | `perf_timeline_078_05` | w3c-performance-timeline-2 | specification | Bộ đệm hiệu năng của trình duyệt có giới hạn hữu hạn (250 mục); mini-app bắt buộc dọn dẹp các mốc mark/measure hoặc sử dụng PerformanceObserver để tiêu thụ dữ liệu liên tục, triệt tiêu rò rỉ RAM trong phiên chạy dài. | `https://developer.mozilla.org/en-US/docs/Web/API/Performance/getEntries` |
| 11 | `flexbox_078_01` | w3c-css-flexbox-1 | official_standard | W3C CSS Flexible Box Layout Level 1 cung cấp động cơ bố cục co giãn thích ứng với đa kích thước màn hình và thiết bị gập, loại bỏ các phép tính toán tọa độ thủ công bằng JavaScript trên luồng chính. | `https://www.w3.org/TR/css-flexbox-1/` |
| 12 | `flexbox_078_02` | w3c-css-flexbox-1 | specification | Thuộc tính `flex-direction` tự động đảo chiều trục bố cục theo `writing-mode` và hướng đọc LTR/RTL, cho phép mini-app bản địa hóa giao diện đa ngôn ngữ (tiếng Ả Rập) mà không cần tạo nhánh CSS riêng biệt. | `https://developer.mozilla.org/en-US/docs/Web/CSS/flex-direction` |
| 13 | `flexbox_078_03` | w3c-css-flexbox-1 | specification | Thuộc tính `justify-content` phân bổ không gian trục chính (space-between, space-evenly) cho thanh điều hướng tab và lưới sản phẩm, ngăn chặn lỗi làm tròn sub-pixel và rung giật giao diện khi xoay màn hình. | `https://developer.mozilla.org/en-US/docs/Web/CSS/justify-content` |
| 14 | `flexbox_078_04` | w3c-css-flexbox-1 | specification | Thuộc tính `align-items` và `align-self` bảo đảm căn chỉnh trục phụ chính xác; hỗ trợ định hình vùng tương tác đạt chuẩn tiếp cận WCAG 2.2 Level AA với kích thước tối thiểu 48x48 CSS pixel. | `https://developer.mozilla.org/en-US/docs/Web/CSS/align-items` |
| 15 | `flexbox_078_05` | w3c-css-flexbox-1 | specification | Cú pháp viết tắt `flex` (`flex: 0 0 <size>`) cố định kích thước cơ sở trước khi tải dữ liệu đa phương tiện bất đồng bộ, triệt tiêu hoàn toàn xê dịch bố cục tích lũy Cumulative Layout Shift (CLS). | `https://developer.mozilla.org/en-US/docs/Web/CSS/flex` |

---

## 3. Kiến trúc thực thi và Kiểm soát chất lượng Store Review (Gate Validation)

1. **Bộ kiểm tra vòng đời bộ nhớ đệm Cache API và tính toàn vẹn gói**:
   - Kiểm tra mã Service Worker: thẩm định việc triển khai xóa các khóa cache cũ trong sự kiện `activate` bằng cách đối chiếu phiên bản `package.json` với danh sách `caches.keys()`.
   - Phân tích tĩnh lệnh gọi `Cache.addAll()` trong quá trình cài đặt: bảo đảm mọi URL được chỉ định đều hợp lệ và trả về mã HTTP 200, ngăn chặn lỗi từ chối cài đặt nguyên tử trên thiết bị người dùng.
   - Rà soát dung lượng lưu trữ bộ nhớ đệm: quét tổng dung lượng tài nguyên tĩnh của gói mini-app nhằm bảo đảm không vượt quá ngưỡng trần 50MB do chợ ứng dụng quy định.

2. **Bộ kiểm tra giám sát hiệu năng Performance Timeline và chống rò rỉ bộ nhớ**:
   - Thẩm định cơ chế thu thập viễn trắc: bắt buộc SDK phân tích sử dụng `PerformanceObserver` thay vì vòng lặp thăm dò `performance.getEntries()`.
   - Kiểm tra tính đầy đủ của dữ liệu khởi động: xác minh việc cấu hình cờ `buffered: true` khi đăng ký observer để không bỏ sót các mốc `navigation` và `paint` trong giai đoạn bootstrap.
   - Giám sát mức tiêu thụ bộ đệm: kiểm tra việc dọn dẹp các mốc đo lường `performance.clearMarks()` trong các chu kỳ chuyển đổi màn hình mini-app nhằm tránh giữ tham chiếu bộ nhớ heap kéo dài.

3. **Bộ kiểm toán công thái học bố cục CSS Flexbox và khả năng tiếp cận (Accessibility Gating)**:
   - Quét tự động vùng chạm (Touch Target Size): kiểm tra các nút tương tác và trường nhập liệu trong vùng chứa Flexbox, bảo đảm kích thước hiển thị thực tế không nhỏ hơn 48x48px (chuẩn WCAG 2.2 Level AA).
   - Kiểm tra ngăn ngừa CLS: thẩm định việc khai báo `flex-basis` hoặc kích thước tỷ lệ cố định (`aspect-ratio`) cho các khung chứa hình ảnh và banner quảng cáo trước khi dữ liệu được tải về.
   - Thử nghiệm giao diện đảo chiều (Bi-directional Testing): tự động hiển thị giao diện với thuộc tính `dir="rtl"` để xác thực việc sắp xếp các phần tử `flex-direction: row` không bị chồng lấn hoặc lệch khỏi khung bảo vệ của siêu ứng dụng.
