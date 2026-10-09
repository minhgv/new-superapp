# Chuyên đề 80: Chuẩn Hóa WHATWG HTML <dialog>, Top Layer, inert Subtree Sandboxing, ECMA-402 Intl Localization & W3C Service Worker Registration Lifecycle

## 1. Bối cảnh & Mục tiêu Kỹ thuật

Trong kiến trúc container của Super App vận hành hàng nghìn mini app đa bên (third-party tenants), ba thách thức then chốt ảnh hưởng trực tiếp đến bảo mật giao diện người dùng (UI security), hiệu năng bản địa hóa (localization efficiency) và tính tất định của vòng đời cập nhật ứng dụng nền (offline/OTA update lifecycle):

1. **Hiểm họa UI Redressing, Clickjacking & Thoát Stacking Context của Modal/Sheet**: Khi mini app hiển thị các hộp thoại xác thực giao dịch, đồng ý quyền hạn bảo mật hoặc cảnh báo hệ thống, việc sử dụng các thẻ `<div>` thông thường kết hợp `z-index: 999999` thường xuyên dẫn đến xung đột với các thành phần floating khác, bị cắt xén (clipped) bởi `overflow: hidden` của container cha, hoặc bị script độc hại chèn lớp phủ trong suốt (transparent clickjacking overlay). Cần chuẩn hóa việc sử dụng phần tử WHATWG HTML `<dialog>`, cơ chế **Browser Top Layer** và thuộc tính toàn cục `inert` để cô lập hoàn toàn giao diện tương tác.
2. **Gánh nặng Payload của Thư viện Bản địa hóa (i18n Bloat) & Lỗi Ngắt Dòng Ngôn ngữ Châu Á**: Đa số mini app nhúng các thư viện bên thứ ba cồng kềnh (Moment.js, formatjs, hoặc các bảng dữ liệu CLDR thô nặng 150KB - 400KB) chỉ để xử lý hiển thị ngày giờ tương đối, danh sách liệt kê hoặc định dạng tiền tệ. Đồng thời, các ngôn ngữ châu Á không có dấu cách giữa các từ (tiếng Trung, tiếng Nhật, tiếng Thái, tiếng Lào) bị ngắt dòng hoặc cắt chuỗi sai lệch khi dùng JavaScript `split('')`, phá vỡ cụm từ và tổ hợp emoji. Cần chuẩn hóa việc khai thác trực tiếp **ECMA-402 ECMAScript Internationalization API** (`Intl.Segmenter`, `Intl.RelativeTimeFormat`, `Intl.ListFormat`, `Intl.Collator`).
3. **Tính Bất định của Vòng đời Service Worker & Độ trễ Vá Lỗ hổng Khẩn cấp**: Mặc định trình duyệt giữ Service Worker mới ở trạng thái "waiting" cho đến khi người dùng đóng toàn bộ client đang mở. Trong mobile super app, người dùng chuyển đổi qua lại giữa các mini app mà không hề đóng super app, khiến các bản vá bảo mật khẩn cấp bị đình trệ nhiều ngày. Cần chuẩn hóa hợp đồng đăng ký `ServiceWorkerRegistration`, thiết lập `updateViaCache: 'none'`, cơ chế kích hoạt tức thì `self.skipWaiting()` kết hợp `Clients.claim()` và hàm kiểm tra chủ động `registration.update()` khi app resume.

---

## 2. Chi tiết 15 Chuẩn Mực Kỹ Thuật (Normative Standards)

### Nhóm 1: WHATWG HTML <dialog>, Top Layer & inert Subtree Sandboxing

#### 1. WHATWG HTML <dialog> Element & Browser Top Layer Modal Isolation
- **Mã định danh**: `HTML-DIALOG-MODAL-TOP-LAYER-SANDBOXING`
- **Mức bằng chứng**: `official-specification` (WHATWG HTML Living Standard - Section 4.11.1)
- **URL xác thực**: `https://html.spec.whatwg.org/multipage/interactive-elements.html`
- **Chi tiết kỹ thuật**:
  - Hộp thoại modal trong mini app (xác nhận thanh toán, ủy quyền quyền hạn, điều khoản dịch vụ) bắt buộc phải được kích hoạt qua phương thức `HTMLDialogElement.showModal()`.
  - Khi gọi `showModal()`, phần tử `<dialog>` được đưa thẳng vào **Browser Top Layer** - một tầng kết xuất độc lập nằm trên đỉnh mọi stacking context của DOM, hoàn toàn không bị ảnh hưởng bởi thứ bậc `z-index` hay các thuộc tính `overflow: hidden`, `transform`, `filter` của phần tử cha.
  - Loại bỏ hoàn toàn lỗ hổng UI redressing và z-index escalation wars giữa các component đa bên.
  - Tự động bẫy tiêu điểm bàn phím (focus trapping) và xử lý phím ESC gửi sự kiện `cancel` tiêu chuẩn.

#### 2. WHATWG HTML inert Global Attribute & Document Subtree Interaction Sandboxing
- **Mã định danh**: `HTML-INERT-ATTRIBUTE-SUBTREE-ISOLATION`
- **Mức bằng chứng**: `official-specification` (WHATWG HTML Living Standard - Section 6.5.3)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/inert`
- **Chi tiết kỹ thuật**:
  - Thuộc tính boolean toàn cục `inert` được trình duyệt hỗ trợ gốc ở mức engine C++ để vô hiệu hóa toàn bộ cây con DOM (DOM subtree).
  - Khi một nút được gán `inert`: trình duyệt bỏ qua toàn bộ sự kiện nhập liệu từ người dùng (pointer events, clicks, touches, keypresses), loại bỏ hoàn toàn các phần tử con khỏi cây trợ năng (accessibility tree), và ngăn chặn điều hướng tuần tự bằng phím Tab.
  - Container Super App ứng dụng `inert` khi hiển thị Native Bottom Sheet xác thực thanh toán: gắn `inert` vào nút gốc của WebView mini app, ngăn chặn hoàn toàn tương tác ngầm mà không cần duyệt DOM để gán `tabindex="-1"` hay `aria-hidden="true"`.

#### 3. HTMLDialogElement.showModal() Built-In Focus Trapping & Accessible Interaction Architecture
- **Mã định danh**: `HTML-DIALOG-SHOWMODAL-FOCUS-TRAPPING`
- **Mức bằng chứng**: `official-documentation` (MDN Web Docs / WHATWG)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/showModal`
- **Chi tiết kỹ thuật**:
  - `showModal()` tự động thiết lập tiêu điểm vào phần tử đầu tiên có thuộc tính `autofocus` hoặc phần tử tương tác đầu tiên trong dialog.
  - Vòng lặp chuyển tiêu điểm (Tab / Shift+Tab) bị khóa chặt trong phạm vi dialog, ngăn chặn rò rỉ tiêu điểm ra thanh điều hướng capsule của super app.
  - Cơ chế đóng hộp thoại trả về mã trạng thái cấu trúc qua thuộc tính `HTMLDialogElement.returnValue` khi gọi `dialog.close(returnValue)`.
  - Bộ linter kiểm duyệt store tự động kiểm tra: mini app không được phép chặn vĩnh viễn sự kiện `cancel` (phím ESC / cử chỉ back); nếu người dùng bấm back 2 lần liên tiếp, container phải cưỡng chế đóng dialog.

#### 4. Browser Top Layer Stacking Architecture & UI Redressing Elimination
- **Mã định danh**: `BROWSER-TOP-LAYER-ZINDEX-ELIMINATION`
- **Mức bằng chứng**: `official-documentation` (MDN Web Docs / W3C Top Layer)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Glossary/Top_layer`
- **Chi tiết kỹ thuật**:
  - Browser Top Layer quản lý các phần tử được kích hoạt qua `<dialog showModal>`, Popover API (`popover="auto"`), và Fullscreen API (`requestFullscreen()`).
  - Thứ tự hiển thị trong Top Layer hoàn toàn tuân theo nguyên tắc LIFO (Last In, First Out) dựa trên thời điểm phần tử được đưa vào top layer, độc lập với cây DOM.
  - Native Shell của Super App duy trì một lớp Top Layer cấp hệ điều hành (Native Overlay Layer) nằm trên cả Top Layer của WebView, đảm bảo capsule điều hướng (nút đóng, nút menu) và các cảnh báo lừa đảo cấp hệ thống luôn luôn hiển thị trên cùng.

#### 5. CSS ::backdrop Pseudo-Element Stacking & Modal Dimming Governance
- **Mã định danh**: `CSS-BACKDROP-PSEUDO-ELEMENT-DIMMING`
- **Mức bằng chứng**: `official-documentation` (MDN Web Docs / W3C CSS Scoping)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/CSS/::backdrop`
- **Chi tiết kỹ thuật**:
  - Phần tử giả `::backdrop` được sinh ra tự động ngay phía sau phần tử trong Top Layer và che phủ toàn bộ viewport.
  - Hỗ trợ làm mờ nền tăng tốc phần cứng (`backdrop-filter: blur(4px)`) và hiệu ứng chuyển cảnh mượt mà khi mở/đóng modal.
  - Ngăn chặn click xuyên thấu (click-through) xuống nội dung nền của mini app.
  - Design system của Store quy định token chuẩn cho `::backdrop` (ví dụ: `rgba(0, 0, 0, 0.45)`), nghiêm cấm mini app thiết lập backdrop hoàn toàn mờ đục nhằm giả mạo màn hình khóa của hệ điều hành.

---

### Nhóm 2: ECMA-402 Intl API & Runtime Localization Architecture

#### 6. ECMA-402 ECMAScript Internationalization API Standard & Engine Runtime Architecture
- **Mã định danh**: `ECMA402-INTERNATIONALIZATION-ARCHITECTURE`
- **Mức bằng chứng**: `official-specification` (Ecma International Standard ECMA-402)
- **URL xác thực**: `https://tc39.es/ecma402/`
- **Chi tiết kỹ thuật**:
  - Chuẩn hóa việc sử dụng không gian tên toàn cục `Intl` được tích hợp sẵn trong engine V8 (Android) và JavaScriptCore (iOS).
  - Tận dụng dữ liệu ICU cấp hệ điều hành, giúp mini app tiết kiệm 150KB - 400KB dung lượng gói bundle do không phải nhúng các thư viện i18n bên ngoài.
  - Thuật toán đàm phán ngôn ngữ xác định (lookup vs best fit) dựa trên thẻ ngôn ngữ BCP 47 kèm các khóa mở rộng Unicode (Unicode locale extension sequences).
  - Quy tắc kiểm duyệt store: Quét bundle mini app; cảnh báo và yêu cầu gỡ bỏ nếu phát hiện các bản dựng Moment.js hoặc date-fns trùng lặp chức năng của `Intl`.

#### 7. Intl.Segmenter Locale-Sensitive Text Boundary Analysis & Word Segmentation
- **Mã định danh**: `INTL-SEGMENTER-LOCALE-TEXT-BOUNDARY`
- **Mức bằng chứng**: `official-documentation` (MDN Web Docs / ECMA-402)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/Segmenter`
- **Chi tiết kỹ thuật**:
  - Cung cấp giải thuật ngắt văn bản theo ranh giới grapheme, word, hoặc sentence nhạy cảm với ngữ tộc: `new Intl.Segmenter(locale, { granularity: 'word' })`.
  - Giải quyết dứt điểm bài toán ngắt từ trong các ngôn ngữ không có khoảng trắng như tiếng Trung, tiếng Nhật, tiếng Thái, tiếng Lào.
  - Bảo toàn trọn vẹn các cụm emoji phức tạp có ký tự nối Zero-Width Joiner (ZWJ) và bộ biến đổi tông màu da (skin tone modifiers), ngăn chặn tình trạng cắt cụt chuỗi làm xuất hiện ký tự rác ``.
  - Ứng dụng trong mini app: Tokenizer tìm kiếm tức thì tại client, bộ soạn thảo rich-text, và căn chỉnh bố cục typography chuẩn xác.

#### 8. Intl.RelativeTimeFormat Localized Time Elapsed & Humanized Chronology Governance
- **Mã định danh**: `INTL-RELATIVETIMEFORMAT-DYNAMIC-UPDATES`
- **Mức bằng chứng**: `official-documentation` (MDN Web Docs / ECMA-402)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/RelativeTimeFormat`
- **Chi tiết kỹ thuật**:
  - Định dạng thời gian tương đối theo ngôn ngữ bản địa: `new Intl.RelativeTimeFormat(locale, { numeric: 'auto' })` (ví dụ: 'hôm qua', '3 phút trước', 'in 2 hours').
  - Hỗ trợ phương thức `formatToParts()` trả về mảng các token cho phép áp dụng CSS highlight lên con số thời gian mà không phá vỡ ngữ pháp câu.
  - Triệt tiêu hoàn toàn độ trễ mạng khi cập nhật feed giao dịch, vị trí tài xế giao hàng hoặc dòng thời gian đơn hàng trong mini app.

#### 9. Intl.ListFormat Localized Item Enumeration & Grammatical Joining Standards
- **Mã định danh**: `INTL-LISTFORMAT-CONJUNCTION-DISJUNCTION`
- **Mức bằng chứng**: `official-documentation` (MDN Web Docs / ECMA-402)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/ListFormat`
- **Chi tiết kỹ thuật**:
  - Định dạng danh sách liệt kê danh từ theo chuẩn ngữ pháp từng quốc gia qua `Intl.ListFormat`: kiểu nối `conjunction` ('A, B và C'), kiểu lựa chọn `disjunction` ('A, B hoặc C'), và kiểu đơn vị `unit`.
  - Thay thế hoàn toàn logic ghép chuỗi thô bằng dấu phẩy gây sai lệch văn phong (ví dụ: dấu phẩy liệt kê tiếng Trung `、` thay vì `,`, quy tắc Oxford comma trong tiếng Anh).
  - Bắt buộc áp dụng trong màn hình xác nhận đơn hàng đa sản phẩm, bảng liệt kê danh sách quyền truy cập thiết bị (permission scopes), và điều khoản pháp lý mini app.

#### 10. Intl.Collator High-Performance Locale-Sensitive Sorting & Diacritic Normalization
- **Mã định danh**: `INTL-COLLATOR-SEARCH-SORT-LOCALIZATION`
- **Mức bằng chứng**: `official-documentation` (MDN Web Docs / ECMA-402)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/Collator`
- **Chi tiết kỹ thuật**:
  - Cung cấp hàm so sánh chuỗi theo ngữ tộc: `new Intl.Collator(locale, { sensitivity: 'base', numeric: true })`.
  - Tùy chọn `sensitivity: 'base'` bỏ qua dấu phụ và chữ hoa/thường, cho phép tìm kiếm mờ (fuzzy search) tiếng Việt không dấu siêu tốc ngay tại client.
  - Tùy chọn `numeric: true` sắp xếp chính xác các chuỗi có chứa số (ví dụ: 'Gói 2', 'Gói 9', 'Gói 10' thay vì '10' đứng trước '2').
  - Thực nghiệm đo lường: Khởi tạo instance `Intl.Collator` và tái sử dụng cho `Array.prototype.sort` cho tốc độ xử lý nhanh hơn 20x - 50x so với gọi `String.prototype.localeCompare` lặp lại trên mảng lớn (>500 phần tử).

---

### Nhóm 3: W3C Service Worker Registration & Lifecycle Governance

#### 11. ServiceWorkerRegistration Interface & Container Registration Scope Security
- **Mã định danh**: `SERVICEWORKER-REGISTRATION-LIFECYCLE-GOVERNANCE`
- **Mức bằng chứng**: `official-documentation` (MDN Web Docs / W3C Service Workers)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerRegistration`
- **Chi tiết kỹ thuật**:
  - Đối tượng `ServiceWorkerRegistration` đóng vai trò quản trị vòng đời Service Worker và phân định ranh giới can thiệp mạng.
  - Quy tắc an ninh container Super App: Phạm vi đăng ký (`registration.scope`) của mini app bắt buộc phải bị giới hạn nghiêm ngặt trong thư mục con định danh của ứng dụng đó (ví dụ: `/miniapps/{app_id}/`), tuyệt đối ngăn chặn Service Worker đăng ký tại scope gốc `/` gây rò rỉ hoặc đánh cắp lưu lượng mạng của mini app khác.
  - Quản lý các cổng subsystem nền: `registration.pushManager`, `registration.sync`, và `registration.periodicSync`.

#### 12. navigator.serviceWorker.register() Options & Atomic Container Bootstrap Contracts
- **Mã định danh**: `SERVICEWORKERCONTAINER-REGISTER-ATOMIC-BOOTSTRAP`
- **Mức bằng chứng**: `official-documentation` (MDN Web Docs / W3C)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerContainer/register`
- **Chi tiết kỹ thuật**:
  - Đăng ký Service Worker với các tham số tối ưu hóa: `navigator.serviceWorker.register(scriptURL, { scope: '/miniapps/xyz/', type: 'module', updateViaCache: 'none' })`.
  - Thiết lập `updateViaCache: 'none'` ép buộc trình duyệt luôn kiểm tra nội dung byte của script Service Worker trực tiếp từ server phân phối của Store, bỏ qua cache HTTP trung gian, loại bỏ hoàn toàn lỗi "zombie worker" lưu trữ mã nguồn lỗi thời.
  - Tham số `type: 'module'` kích hoạt cú pháp ES Modules bên trong worker, chuẩn hóa việc tái sử dụng code giữa worker và luồng chính.

#### 13. self.skipWaiting() & Immediate Service Worker Activation Pipeline
- **Mã định danh**: `SERVICEWORKER-SKIPWAITING-SEAMLESS-ACTIVATION`
- **Mức bằng chứng**: `official-documentation` (MDN Web Docs / W3C)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/skipWaiting`
- **Chi tiết kỹ thuật**:
  - Khi triển khai các bản vá bảo mật nóng (hotfix OTA), Service Worker mới được cài đặt sẽ gọi `self.skipWaiting()` ngay trong sự kiện `install`.
  - Lệnh này cưỡng chế Service Worker chuyển ngay từ trạng thái "installed/waiting" sang trạng thái "activating" mà không cần đợi người dùng đóng các tab/màn hình hiện tại.
  - Rút ngắn thời gian phát tán bản vá lỗ hổng nghiêm trọng từ hàng ngày xuống chỉ còn vài mili-giây trên toàn bộ thiết bị người dùng.
  - Phải kết hợp dọn dẹp các cache phiên bản cũ trong sự kiện `activate` qua `caches.delete(oldCacheName)`.

#### 14. Clients.claim() First-Load Network Interception & Controller Synchronization
- **Mã định danh**: `SERVICEWORKER-CLIENTS-CLAIM-CONTROLLER-HANDSHAKE`
- **Mức bằng chứng**: `official-documentation` (MDN Web Docs / W3C)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/API/Clients/claim`
- **Chi tiết kỹ thuật**:
  - Khi người dùng truy cập mini app lần đầu tiên, trang HTML tải qua mạng trước khi Service Worker hoàn tất đăng ký, khiến các tài nguyên phụ của lần tải đầu không được Service Worker kiểm soát.
  - Gọi `self.clients.claim()` trong sự kiện `activate` sẽ lập tức chiếm quyền điều khiển toàn bộ các client đang mở trong scope hợp lệ.
  - Kích hoạt sự kiện `controllerchange` trên `navigator.serviceWorker` tại trang chính để đồng bộ trạng thái runtime.
  - Đảm bảo ngay từ lần truy cập đầu tiên, các asset tĩnh đã được lưu vào CacheStorage, sẵn sàng chuyển sang chế độ ngoại tuyến (offline-ready) ngay lập tức.

#### 15. ServiceWorkerRegistration.update() Programmatic OTA Refresh & Delta Verification
- **Mã định danh**: `SERVICEWORKER-REGISTRATION-UPDATE-OTA-ORCHESTRATION`
- **Mức bằng chứng**: `official-documentation` (MDN Web Docs / W3C)
- **URL xác thực**: `https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerRegistration/update`
- **Chi tiết kỹ thuật**:
  - Trình duyệt tự động kiểm tra cập nhật Service Worker khi chuyển trang sau mỗi 24 giờ. Tuy nhiên, tiêu chuẩn Store yêu cầu điều phối cập nhật chủ động theo thời gian thực.
  - Mini app container kích hoạt `registration.update()` khi:
    1. Ứng dụng khôi phục từ nền (`document.visibilityState === 'visible'`).
    2. Nhận được tín hiệu cập nhật qua WebSocket / Push Message từ Super App Server.
    3. Điều hướng đến các màn hình giao dịch tài chính nhạy cảm.
  - Đáp ứng SLA cập nhật bản vá lỗ hổng khẩn cấp của Store trong vòng dưới 15 phút.

---

## 3. Ma Trận Đối Soát & Kiểm Duyệt Store (Store Review & Conformance Matrix)

| Mã Chuẩn Mực | Phân Loại Kiểm Tra | Mức Độ | Công Cụ Kiểm Duyệt (Automated / Dynamic) | Tiêu Chí Đạt (Acceptance Criteria) |
|---|---|---|---|---|
| `HTML-DIALOG-MODAL-TOP-LAYER-SANDBOXING` | Security & UI Sandboxing | Bắt buộc (Mandatory) | AST Linter / DOM Hierarchy Inspector | Toàn bộ modal phải gọi `showModal()` hoặc Sheet Native; cấm thẻ div `z-index: 999999`. |
| `HTML-INERT-ATTRIBUTE-SUBTREE-ISOLATION` | Accessibility & Interaction | Bắt buộc (Mandatory) | Axe Accessibility Engine / Playwright | Gắn `inert` vào panel/view ẩn; không rò rỉ sự kiện bấm và tiêu điểm bàn phím. |
| `HTML-DIALOG-SHOWMODAL-FOCUS-TRAPPING` | Accessibility & UX Integrity | Bắt buộc (Mandatory) | Playwright Keyboard Traversal Test | Tiêu điểm bàn phím bị khóa trong dialog; phím ESC đóng dialog trong tối đa 2 lần bấm. |
| `BROWSER-TOP-LAYER-ZINDEX-ELIMINATION` | Visual Security & Layout | Bắt buộc (Mandatory) | CSS Computed Style Analyzer | Không có phần tử che khuất native capsule; Top Layer được sử dụng cho modal/popover. |
| `CSS-BACKDROP-PSEUDO-ELEMENT-DIMMING` | Design System Conformance | Khuyến nghị (Recommended) | Visual Regression / Token Checker | Màu và độ mờ của `::backdrop` tuân thủ design token (`rgba(0,0,0,0.45)`); cấm backdrop mờ đục 100%. |
| `ECMA402-INTERNATIONALIZATION-ARCHITECTURE` | Performance & Bundle Budget | Khuyến nghị (Recommended) | Bundle Analyzer / Static AST Scan | Cảnh báo khi bundle chứa Moment.js / date-fns; ưu tiên sử dụng `Intl` native engine. |
| `INTL-SEGMENTER-LOCALE-TEXT-BOUNDARY` | Localization & Typography | Bắt buộc cho CJK/SEA | Unit Test với chuỗi ký tự CJK / Thái / Lào | Sử dụng `Intl.Segmenter` cho chức năng tìm kiếm, đếm từ và ngắt dòng văn bản phức tạp. |
| `INTL-RELATIVETIMEFORMAT-DYNAMIC-UPDATES` | UX & Localization Fidelity | Khuyến nghị (Recommended) | Feed Rendering Benchmark | Sử dụng `Intl.RelativeTimeFormat` cho các nhãn thời gian tương đối; không ghép chuỗi cứng. |
| `INTL-LISTFORMAT-CONJUNCTION-DISJUNCTION` | UX & Multilingual Grammar | Bắt buộc cho E-commerce | i18n Linter | Danh sách liệt kê sản phẩm/quyền hạn phải qua `Intl.ListFormat`. |
| `INTL-COLLATOR-SEARCH-SORT-LOCALIZATION` | Performance & Search Quality | Bắt buộc cho Catalog > 500 mục | Performance Profiler (60 FPS test) | Tái sử dụng instance `Intl.Collator` cho mảng lớn; tìm kiếm tiếng Việt không phân biệt dấu. |
| `SERVICEWORKER-REGISTRATION-LIFECYCLE-GOVERNANCE` | Security Isolation | Bắt buộc (Mandatory) | Package Manifest Security Scanner | Scope đăng ký phải nằm trong `/miniapps/{app_id}/`; script URL cùng origin an toàn (HTTPS). |
| `SERVICEWORKERCONTAINER-REGISTER-ATOMIC-BOOTSTRAP` | Reliability & Cache Freshness | Bắt buộc (Mandatory) | Service Worker Registration Linter | Thiết lập `updateViaCache: 'none'` và `type: 'module'` trong tham số đăng ký. |
| `SERVICEWORKER-SKIPWAITING-SEAMLESS-ACTIVATION` | OTA Deployment SLA | Bắt buộc cho Hotfix | OTA Pipeline Emulator | Service Worker hotfix gọi `self.skipWaiting()` trong sự kiện `install`. |
| `SERVICEWORKER-CLIENTS-CLAIM-CONTROLLER-HANDSHAKE` | Offline Performance | Bắt buộc (Mandatory) | First-Load Service Worker Test | Service Worker gọi `self.clients.claim()` trong sự kiện `activate`. |
| `SERVICEWORKER-REGISTRATION-UPDATE-OTA-ORCHESTRATION` | Operation & Security SLA | Bắt buộc (Mandatory) | App Lifecycle Telemetry Check | Container gọi `registration.update()` khi app resume và khi nhận push thông báo phiên bản mới. |

---

## 4. Kiến Trúc Tham Chiếu Triển Khai (Reference Implementation)

```typescript
// ============================================================================
// 1. Module Quản Lý Modal An Toàn Qua HTML <dialog> & Top Layer
// ============================================================================
export class SecureModalController {
  private dialogElement: HTMLDialogElement;
  private backgroundRoot: HTMLElement;

  constructor(dialogId: string, backgroundRootId: string) {
    this.dialogElement = document.getElementById(dialogId) as HTMLDialogElement;
    this.backgroundRoot = document.getElementById(backgroundRootId) as HTMLElement;
  }

  public openModal(): Promise<string> {
    return new Promise((resolve, reject) => {
      if (!this.dialogElement) {
        return reject(new Error('Dialog element not found'));
      }

      // Kích hoạt thuộc tính inert lên cây con nền để cách ly tương tác
      this.backgroundRoot.toggleAttribute('inert', true);

      // Đưa dialog vào Browser Top Layer với bẫy tiêu điểm tự động
      this.dialogElement.showModal();

      const handleClose = () => {
        this.backgroundRoot.toggleAttribute('inert', false);
        this.dialogElement.removeEventListener('close', handleClose);
        resolve(this.dialogElement.returnValue || 'dismissed');
      };

      this.dialogElement.addEventListener('close', handleClose);
    });
  }

  public closeModal(resultCode: string) {
    if (this.dialogElement && this.dialogElement.open) {
      this.dialogElement.close(resultCode);
    }
  }
}

// ============================================================================
// 2. Module Bản Địa Hóa Chuẩn ECMA-402 Tối Ưu Bộ Nhớ
// ============================================================================
export class SuperAppI18nEngine {
  private locale: string;
  private collatorInstance: Intl.Collator;
  private segmenterInstance: Intl.Segmenter;
  private relativeTimeFormatter: Intl.RelativeTimeFormat;
  private listFormatter: Intl.ListFormat;

  constructor(locale: string = 'vi-VN') {
    this.locale = locale;
    // Khởi tạo và lưu cache instance để tái sử dụng, tăng tốc độ xử lý 50x
    this.collatorInstance = new Intl.Collator(this.locale, {
      usage: 'search',
      sensitivity: 'base',
      numeric: true
    });
    this.segmenterInstance = new Intl.Segmenter(this.locale, {
      granularity: 'word'
    });
    this.relativeTimeFormatter = new Intl.RelativeTimeFormat(this.locale, {
      numeric: 'auto',
      style: 'short'
    });
    this.listFormatter = new Intl.ListFormat(this.locale, {
      type: 'conjunction',
      style: 'long'
    });
  }

  // Sắp xếp danh mục sản phẩm lớn chuẩn ngữ tộc
  public sortCatalog(items: Array<{ name: string }>): Array<{ name: string }> {
    return items.sort((a, b) => this.collatorInstance.compare(a.name, b.name));
  }

  // Tách từ khóa tìm kiếm tiếng Á không bị gãy cụm
  public tokenizeSearchQuery(query: string): string[] {
    const segments = this.segmenterInstance.segment(query);
    return Array.from(segments)
      .filter(seg => seg.isWordLike)
      .map(seg => seg.segment);
  }

  // Hiển thị thời gian tương đối
  public formatElapsed(value: number, unit: Intl.RelativeTimeFormatUnit): string {
    return this.relativeTimeFormatter.format(value, unit);
  }

  // Ghép danh sách điều khoản hoặc danh sách mặt hàng
  public formatItemList(items: string[]): string {
    return this.listFormatter.format(items);
  }
}

// ============================================================================
// 3. Module Đăng Ký & Điều Phối Vòng Đời Service Worker Cập Nhật Tức Thì
// ============================================================================
export class MiniAppServiceWorkerOrchestrator {
  private registration: ServiceWorkerRegistration | null = null;
  private appId: string;

  constructor(appId: string) {
    this.appId = appId;
  }

  public async bootstrap(): Promise<void> {
    if (!('serviceWorker' in navigator)) {
      console.warn('Service Worker is not supported in this runtime environment');
      return;
    }

    try {
      const scopeUrl = `/miniapps/${this.appId}/`;
      this.registration = await navigator.serviceWorker.register(
        `${scopeUrl}sw.js`,
        {
          scope: scopeUrl,
          type: 'module',
          updateViaCache: 'none' // Luôn kiểm tra byte freshness từ server
        }
      );

      // Lắng nghe sự kiện controllerchange khi worker mới chiếm quyền điều khiển
      navigator.serviceWorker.addEventListener('controllerchange', () => {
        console.log('[SW] New active controller detected. Synchronizing runtime state...');
      });

      // Lắng nghe trạng thái visibilitychange để cập nhật tức thì khi app resume
      document.addEventListener('visibilitychange', () => {
        if (document.visibilityState === 'visible') {
          this.triggerUpdateCheck();
        }
      });

      console.log('[SW] Service Worker registered with scope:', this.registration.scope);
    } catch (err) {
      console.error('[SW] Registration failed:', err);
    }
  }

  public async triggerUpdateCheck(): Promise<void> {
    if (this.registration) {
      try {
        await this.registration.update();
        console.log('[SW] Checked for service worker updates');
      } catch (err) {
        console.warn('[SW] Update check failed gracefully:', err);
      }
    }
  }
}

// ============================================================================
// Mã nguồn bên trong sw.js của Mini App
// ============================================================================
/*
self.addEventListener('install', (event: ExtendableEvent) => {
  // Cưỡng chế kích hoạt ngay lập tức, không chờ tab đóng
  self.skipWaiting();
});

self.addEventListener('activate', (event: ExtendableEvent) => {
  event.waitUntil(
    Promise.all([
      // Chiếm quyền kiểm soát ngay lập tức đối với client đang mở
      self.clients.claim(),
      // Dọn dẹp cache phiên bản cũ
      caches.keys().then((keys) =>
        Promise.all(
          keys.filter((key) => key !== 'miniapp-cache-v2').map((key) => caches.delete(key))
        )
      )
    ])
  );
});
*/
```
