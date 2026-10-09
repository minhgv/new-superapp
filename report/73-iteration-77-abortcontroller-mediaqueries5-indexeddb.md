# Chuyên đề 73: WHATWG DOM AbortController, W3C Media Queries Level 5 & W3C IndexedDB 3.0

## 1. Tổng quan nghiên cứu Milestone 77

Milestone 77 tập trung hoàn thiện 3 trụ cột công nghệ nền tảng quyết định khả năng hủy bỏ tác vụ bất đồng bộ hợp tác, phân loại sở thích tiếp cận người dùng hệ thống và cơ chế lưu trữ giao dịch phía client có cấu trúc trong kiến trúc siêu ứng dụng (Super-App Mini-App Store Standard):

1. **WHATWG DOM AbortController & Asynchronous Cancellation Architecture**:
   - Chuẩn hóa cơ chế hủy bỏ tác vụ bất đồng bộ hợp tác đa ngữ cảnh thông qua giao diện `AbortController` và tín hiệu `AbortSignal`, cho phép hủy bỏ đồng thời các thao tác I/O mạng (`fetch`), đọc file (`FileReader`), luồng dữ liệu (`ReadableStream`) và lời gọi cầu nối native bridge.
   - Lan truyền nguyên nhân hủy bỏ tất định thông qua thuộc tính `signal.reason`, tự động gán các đối tượng `DOMException` chuẩn mực (`AbortError`, `TimeoutError`) hoặc đối tượng lỗi tùy biến của ứng dụng, hỗ trợ phân loại lỗi chính xác trong hệ thống telemetry và giám sát sự cố của chợ ứng dụng.
   - Khởi tạo hạn chót thời gian thực thi (deadline enforcement) không phụ thuộc bộ định thời thủ công bằng phương thức tĩnh `AbortSignal.timeout(delay)`, bắt buộc áp dụng trần thời gian (timeout ceiling) cho mọi request ra ngoài (5000ms cho REST API, 10000ms cho tải tài nguyên gói) nhằm triệt tiêu request treo (zombie requests) gây hao pin và nghẽn băng thông di động.
   - Điều phối tín hiệu kết hợp (composite signal orchestration) thông qua phương thức tĩnh `AbortSignal.any(signals)`, gộp nhiều nguồn kích hoạt hủy bỏ (nút Cancel của người dùng, tín hiệu đóng container của super-app cha, bộ đếm timeout mạng) thành một tín hiệu điều khiển duy nhất trong quy trình thanh toán và xác thực.
   - Dọn dẹp listener tự động và triệt để bằng tùy chọn `signal` trong `EventTarget.addEventListener(type, listener, { signal })`, bảo đảm mọi sự kiện cấp window/document (`resize`, `pointermove`, `orientationchange`) được hủy đăng ký tự động khi component bị tháo gỡ (unmount), loại bỏ hoàn toàn rò rỉ bộ nhớ DOM.

2. **W3C Media Queries Level 5 & User Preference Accessibility Taxonomy**:
   - Đặc tả hệ thống thuộc tính truyền thông hướng người dùng (user preference media features) của W3C Media Queries Level 5, làm cầu nối phản ánh trực tiếp cấu hình thẩm mỹ, công thái học và khả năng tiếp cận của hệ điều hành vào CSS và JavaScript qua `window.matchMedia()`.
   - Thực thi tiêu chuẩn tiếp cận chuyển động `@media (prefers-reduced-motion: reduce)`, bắt buộc mini-app khách vô hiệu hóa hiệu ứng cuộn thị sai (parallax), tắt tự động chạy carousel quảng cáo và rút ngắn thời lượng hoạt họa nhằm bảo vệ người dùng mắc hội chứng rối loạn tiền đình, đáp ứng chuẩn WCAG 2.2 AA.
   - Đồng bộ hóa giao diện sáng/tối tự động qua `@media (prefers-color-scheme: dark | light)`, loại bỏ triệt để hiện tượng chói mắt đột ngột (white blinding flash) khi khởi chạy mini-app từ môi trường super-app tối trên màn hình OLED và tiết kiệm năng lượng pin di động.
   - Kiểm soát độ tương phản thị giác nghiêm ngặt qua `@media (prefers-contrast: more | less)`, áp đặt tỷ lệ tương phản văn bản tối thiểu 7:1 (chuẩn WCAG AAA) và bổ sung viền rõ nét cho các nút bấm xác nhận thanh toán tài chính đối với người dùng thị lực kém.
   - Bảo toàn khả năng tương thích chế độ màu tương phản cao bắt buộc của hệ điều hành qua `@media (forced-colors: active)`, nghiêm cấm ghi đè tùy tiện bằng thuộc tính `forced-color-adjust: none`, bảo đảm bảng màu tương phản hệ thống Canvas/CanvasText được áp dụng trọn vẹn.

3. **W3C IndexedDB 3.0 & Transactional Client Persistence**:
   - Chuẩn hóa hệ quản trị cơ sở dữ liệu đối tượng giao dịch phía client W3C Indexed Database API 3.0, cung cấp không gian lưu trữ bất đồng bộ dung lượng lớn cho danh mục sản phẩm ngoại tuyến, token phiên mã hóa và bản nháp biểu mẫu, vượt trội hoàn toàn so với giới hạn đồng bộ 5MB của localStorage.
   - Kiểm soát vòng đời và độ bền giao dịch thông qua `IDBTransaction` (`readonly`, `readwrite`, `versionchange`), hỗ trợ tùy chọn độ bền `durability: 'strict' | 'relaxed'`. Bắt buộc các giao dịch thanh toán và ghi nhận chứng từ tài chính sử dụng chế độ `strict` để ép ghi đĩa tức thì, chống mất dữ liệu khi ứng dụng bị tắt đột ngột do hệ thống giải phóng RAM.
   - Tối ưu hóa truy vấn phạm vi với `IDBKeyRange` (`only()`, `lowerBound()`, `upperBound()`, `bound()`), cho phép quét dữ liệu theo chỉ mục thời gian và danh mục với độ trễ < 50ms mà không phải nạp toàn bộ bảng ghi vào bộ nhớ heap.
   - Thiết lập chỉ mục đa khóa `IDBIndex` (`unique: true`, `multiEntry: true`) hỗ trợ tìm kiếm phân loại sản phẩm nhanh chóng, ngăn ngừa nghẽn luồng sự kiện chính (event loop) trên thiết bị di động cấu hình yếu.
   - Bảo toàn tính toàn vẹn cấu trúc dữ liệu qua `IDBObjectStore` và cơ chế di chuyển phiên bản tất định `onupgradeneeded`, bảo đảm các bản cập nhật mini-app qua OTA không làm gián đoạn hay phá hỏng cấu trúc dữ liệu người dùng đã lưu offline.

---

## 2. Chi tiết 15 chuẩn kỹ thuật cốt lõi (Normative Findings)

Dưới đây là 15 quy chuẩn kỹ thuật chính thức được trích xuất từ các tài liệu W3C, WHATWG và MDN Web Docs, đã qua kiểm tra hợp lệ, không chứa credential và xác minh HTTP 200:

| # | Khía cạnh / Sự kiện | Nguồn chuẩn | Mức độ chứng cứ | Tóm tắt quy chuẩn và yêu cầu kiến trúc Super-App | URL nguồn đã kiểm tra |
|---|---|---|---|---|---|
| 1 | `abortcontroller-cancellation-architecture` | whatwg-dom-abortcontroller | normative | Giao diện AbortController cung cấp cơ chế hủy bỏ hợp tác chuẩn mực cho các tác vụ bất đồng bộ; mini-app và cầu nối native bridge bắt buộc tích hợp AbortController để hủy bỏ các request treo khi chuyển trang hoặc đóng container. | `https://developer.mozilla.org/en-US/docs/Web/API/AbortController` |
| 2 | `abortsignal-lifecycle-and-reason-propagation` | whatwg-dom-abortcontroller | normative | Giao diện AbortSignal thông báo trạng thái hủy bỏ và lan truyền lý do hủy (`signal.reason`) dưới dạng DOMException (AbortError/TimeoutError), phân tách lỗi mạng hệ thống với thao tác người dùng hủy bỏ phục vụ giám sát store review. | `https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal` |
| 3 | `abortsignal-timeout-deterministic-deadlines` | whatwg-dom-abortcontroller | normative | Phương thức tĩnh `AbortSignal.timeout(delay)` tạo tín hiệu hủy tự động sau một khoảng thời gian định trước; bắt buộc áp dụng trần thời gian (5000ms cho REST, 10000ms cho tài nguyên) triệt tiêu zombie fetch gây cạn kiệt tài nguyên. | `https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal/timeout_static` |
| 4 | `abortsignal-any-composite-signal-orchestration` | whatwg-dom-abortcontroller | normative | Phương thức tĩnh `AbortSignal.any(signals)` gộp nhiều tín hiệu hủy thành một luồng hợp nhất; kích hoạt hủy ngay khi bất kỳ tín hiệu thành phần nào phát sinh (người dùng bấm hủy, super-app đóng capsule, hết hạn timeout). | `https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal/any_static` |
| 5 | `whatwg-dom-event-listener-abort-cleanup` | whatwg-dom-abortcontroller | normative | Đặc tả WHATWG DOM cho phép truyền `{ signal }` vào `addEventListener()`, tự động gỡ bỏ listener khi tín hiệu hủy phát sinh; ngăn chặn triệt để rò rỉ bộ nhớ listener trên window/document khi component unmount. | `https://dom.spec.whatwg.org/` |
| 6 | `media-queries-5-user-preference-taxonomy` | w3c-mediaqueries-5 | normative | Chuẩn W3C Media Queries Level 5 thiết lập hệ thống thuộc tính truyền thông sở thích người dùng, cầu nối các thiết lập tiếp cận và công thái học của hệ điều hành vào CSS/JS của mini-app nhằm bảo đảm trải nghiệm bao hàm. | `https://www.w3.org/TR/mediaqueries-5/` |
| 7 | `prefers-reduced-motion-accessibility-contract` | w3c-mediaqueries-5 | normative | Thuộc tính `@media (prefers-reduced-motion)` phát hiện yêu cầu giảm chuyển động; bắt buộc mini-app tắt hiệu ứng cuộn thị sai, dừng carousel tự động và rút ngắn thời lượng hoạt họa cho người dùng rối loạn tiền đình (WCAG 2.2 AA). | `https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion` |
| 8 | `prefers-color-scheme-theme-synchronization` | w3c-mediaqueries-5 | normative | Thuộc tính `@media (prefers-color-scheme)` phát hiện theme sáng/tối của hệ điều hành; bắt buộc mini-app đồng bộ bảng màu giao diện để tránh chói mắt khi khởi chạy từ super-app tối và tối ưu tiêu thụ điện trên màn hình OLED. | `https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme` |
| 9 | `prefers-contrast-visual-legibility-governance` | w3c-mediaqueries-5 | normative | Thuộc tính `@media (prefers-contrast)` phát hiện yêu cầu tăng độ tương phản; bắt buộc giao diện thanh toán và form nhập liệu áp dụng tỷ lệ tương phản văn bản >= 7:1 (WCAG AAA) và hiển thị viền rõ ràng cho người thị lực kém. | `https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-contrast` |
| 10 | `forced-colors-high-contrast-mode-preservation` | w3c-mediaqueries-5 | normative | Thuộc tính `@media (forced-colors)` nhận diện chế độ màu tương phản cao bắt buộc của hệ thống; nghiêm cấm sử dụng `forced-color-adjust: none` tùy tiện, bảo toàn bảng màu hệ thống cho công nghệ trợ năng. | `https://developer.mozilla.org/en-US/docs/Web/CSS/@media/forced-colors` |
| 11 | `indexeddb-3-transactional-storage-architecture` | w3c-indexeddb-3 | normative | W3C IndexedDB 3.0 cung cấp hệ thống lưu trữ đối tượng giao dịch bất đồng bộ an toàn theo nguồn gốc; đóng vai trò động cơ lưu trữ chính cho danh mục sản phẩm offline và token mã hóa vượt trên hạn mức của localStorage. | `https://www.w3.org/TR/IndexedDB-3/` |
| 12 | `idbtransaction-lifecycle-and-durability-governance` | w3c-indexeddb-3 | normative | Giao diện IDBTransaction bảo đảm tính toàn vẹn ACID với các chế độ readonly/readwrite; tùy chọn độ bền `durability: 'strict'` bắt buộc áp dụng cho thao tác ghi sổ tài chính để ép ghi đĩa ngay, chống mất dữ liệu khi bị kill ứng dụng. | `https://developer.mozilla.org/en-US/docs/Web/API/IDBTransaction` |
| 13 | `idbkeyrange-query-boundary-isolation` | w3c-indexeddb-3 | normative | Giao diện IDBKeyRange định nghĩa các khoảng khóa liên tục (only, lowerBound, upperBound, bound); hỗ trợ tìm kiếm phân trang và truy vấn danh mục nhanh chóng dưới 50ms mà không gây nghẽn RAM heap di động. | `https://developer.mozilla.org/en-US/docs/Web/API/IDBKeyRange` |
| 14 | `idbindex-multi-key-query-optimization` | w3c-indexeddb-3 | normative | Giao diện IDBIndex cho phép truy vấn bản ghi bất đồng bộ theo khóa phụ và mảng đa mục (multiEntry); loại bỏ việc duyệt lọc thủ công gây nghẽn luồng xử lý giao diện chính (main event loop) của mini-app. | `https://developer.mozilla.org/en-US/docs/Web/API/IDBIndex` |
| 15 | `idbobjectstore-key-generator-and-schema-integrity` | w3c-indexeddb-3 | normative | Giao diện IDBObjectStore quản lý lược đồ và bản ghi dữ liệu; bắt buộc quản trị phiên bản nâng cấp cấu trúc tất định trong sự kiện `onupgradeneeded` nhằm bảo vệ dữ liệu người dùng khi cập nhật gói OTA. | `https://developer.mozilla.org/en-US/docs/Web/API/IDBObjectStore` |

---

## 3. Kiến trúc thực thi và Kiểm soát chất lượng Store Review (Gate Validation)

1. **Bộ kiểm tra hủy bỏ bất đồng bộ và rò rỉ listener tự động**:
   - Môi trường kiểm thử headless giả lập chuyển trang và ngắt kết nối đột ngột: kiểm tra xem các lời gọi bridge kéo dài có phản hồi sự kiện abort của `AbortController` hay không.
   - Phân tích cú pháp AST kiểm tra các lệnh gọi `addEventListener` trên `window` và `document`: khuyến nghị hoặc bắt buộc truyền `{ signal }` để bảo đảm tự động dọn dẹp bộ nhớ.

2. **Bộ kiểm tra khả năng tiếp cận và công thái học hiển thị (User Preference Testing)**:
   - Giả lập chế độ `@media (prefers-reduced-motion: reduce)`: đo lường xem tần suất và thời lượng hoạt họa có giảm xuống ngưỡng an toàn hay không (loại bỏ hoàn toàn các chuyển động rung lắc/xoay vòng lớn).
   - Kiểm tra tương thích giao diện tối qua `@media (prefers-color-scheme: dark)`: xác minh tính hợp lệ của thẻ meta `theme-color` và tỷ lệ tương phản màu sắc tránh gây lóa mắt.
   - Xác thực tỷ lệ tương phản tối thiểu 7:1 khi kích hoạt `@media (prefers-contrast: more)` trên các thành phần nút bấm và văn bản quan trọng.

3. **Kiểm toán cơ sở dữ liệu IndexedDB 3.0 và an toàn giao dịch**:
   - Kiểm tra mã nguồn khai báo giao dịch tài chính: bắt buộc cấu hình `{ durability: 'strict' }` cho các đối tượng lưu trữ số dư, hóa đơn hoặc chứng từ giao dịch ngoại tuyến.
   - Thẩm định logic xử lý trong `onupgradeneeded`: cấm các thao tác xóa trắng kho dữ liệu (`clear()`) khi nâng cấp phiên bản lược đồ trừ khi có xác nhận chủ ý của nhà phát triển.
