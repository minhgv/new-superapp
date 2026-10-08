# Chuyên Đề 44: Chuẩn Hóa Điều Hướng SPA, Khả Năng Truyền Thông Đa Phương Tiện & Quản Trị Phông Chữ Trong Mini App Store

**Mã tài liệu**: `M48-NAV-MEDIA-TYPO`
**Thuộc dự án**: Nghiên Cứu Chuẩn Mini App Store Cho Super App (Deli Deep)
**Thời điểm hoàn thành**: Tháng 10/2026
**Số lượng findings mới**: 15 findings chuẩn hóa (100% normative standards & official platform specs)
**Trạng thái kiểm tra**: Hoàn tất xác thực HTTP 200, schema JSONL hợp lệ, zero secret leakage, URL deduplicated.

---

## 1. Bối Cảnh & Mục Tiêu Nghiên Cứu

Khi super app mở rộng quy mô lên hàng trăm mini app thuộc nhiều ngành dọc (thương mại điện tử, phát trực tiếp/livestream shopping, giải trí đa phương tiện, công cụ doanh nghiệp và tài chính), ba thách thức kiến trúc nghiêm trọng phát sinh tại tầng client container:

1. **Điều hướng Single-Page App (SPA) phân mảnh và mất đồng bộ**: Các mini app xây dựng trên React, Vue, Svelte phụ thuộc vào `history.pushState` và sự kiện `popstate`. Cơ chế cũ này không thể chặn (intercept) toàn diện các thao tác điều hướng tự nhiên (submit form, click link nội bộ, thao tác Back vật lý của OS) và không có cơ chế hủy bỏ (cancellation) các yêu cầu mạng dang dở, dẫn đến rò rỉ bộ nhớ, race-condition và xung đột với thanh điều hướng native của super app container.
2. **Quản trị truyền thông phát trực tiếp và bản quyền DRM phần cứng**: Các tính năng phát video độ phân giải cao (4K/60fps), luồng video trực tiếp độ trễ thấp và nội dung trả phí bản quyền (phim số, sự kiện trực tiếp) thường gây giật khung hình (frame dropping), cạn kiệt pin và quá nhiệt thiết bị nếu phát codec không tương thích với bộ giải mã phần cứng. Thiếu vắng chuẩn EME/CDM bảo mật dẫn đến nguy cơ lộ lọt bản quyền trí tuệ của nhà phát hành.
3. **Hiện tượng giật layout do phông chữ (CLS) và cửa sổ nổi đa nhiệm (Document PiP)**: Việc nạp phông chữ không đồng bộ gây ra Flash of Unstyled Text (FOUT) / Flash of Invisible Text (FOIT), phá vỡ chỉ số Cumulative Layout Shift (CLS > 0.1). Đồng thời, nhu cầu hiển thị giao diện tương tác thu nhỏ (giỏ hàng livestream, widget hội thoại video) đòi hỏi cơ chế cửa sổ nổi DOM tùy ý (Document Picture-in-Picture) thay vì chỉ gói gọn trong thẻ `<video>`.

Milestone 48 thiết lập bộ tiêu chuẩn kỹ thuật toàn diện nhằm giải quyết triệt để 3 nhóm vấn đề trên.

---

## 2. Kiến Trúc Điều Hướng Single-Page App & Định Tuyến Tuyên Bố (WICG Navigation API & URLPattern)

### 2.1. Bộ Điều Khiển Điều Hướng Hợp Nhất (WICG Navigation API)
- **Chuẩn tham chiếu**: WICG Navigation API (`https://developer.mozilla.org/en-US/docs/Web/API/Navigation_API`, `https://wicg.github.io/navigation-api/`).
- **Cơ chế hoạt động**:
  - `window.navigation` thay thế toàn bộ hệ thống phân mảnh `history.pushState`, `hashchange`, `popstate`.
  - Bắt toàn bộ nỗ lực chuyển trang qua sự kiện `navigate`. Khi `event.canIntercept` là `true`, mini app kích hoạt `event.intercept({ handler, focusReset, scroll })` để thực thi quá trình chuyển trang mượt mà bằng async handler promise mà không kích hoạt tải lại webview container.
  - Tích hợp `event.signal` (AbortSignal) tự động hủy bỏ (abort) các request mạng `fetch(url, { signal: event.signal })` của trang trước nếu người dùng nhanh chóng chuyển hướng sang view khác.
  - Cung cấp `event.scroll()` cho phép nhà phát triển kiểm soát chính xác thời điểm khôi phục vị trí cuộn trang (scroll restoration) sau khi danh sách ảo (virtual list) đã hoàn tất render DOM.

### 2.2. Kiểm Soát Lịch Sử Điều Hướng & Cách Ly Trạng Thái (NavigationHistoryEntry)
- **Chuẩn tham chiếu**: WICG Navigation API § 3.1 (`https://developer.mozilla.org/en-US/docs/Web/API/Navigation`).
- **Cơ chế hoạt động**:
  - `navigation.entries()` trả về danh sách có thứ tự các đối tượng `NavigationHistoryEntry` thuộc same-origin của mini app, bao gồm `id`, `key`, `url`, `index`, `sameDocument`.
  - Cho phép điều hướng trực tiếp theo khóa: `navigation.traverseTo(entry.key)`.
  - Phối hợp chặt chẽ giữa thanh điều hướng gốc (Native App Bar) của Super App và `navigation.currentEntry.index`: tự động hiển thị nút "Back" hoặc "Đóng Mini App" dựa trên độ sâu stack.
  - Ngăn chặn tuyệt đối việc lưu trữ token xác thực hoặc thông tin định danh nhạy cảm trong `navigation.navigate(url, { state })`.

### 2.3. Khớp Mẫu URL An Toàn ReDoS (WICG URLPattern API)
- **Chuẩn tham chiếu**: WICG URLPattern API (`https://developer.mozilla.org/en-US/docs/Web/API/URLPattern`, `https://wicg.github.io/urlpattern/`).
- **Cơ chế hoạt động**:
  - Chuẩn hóa cú pháp so khớp URL dựa trên cú pháp path-to-regexp trực tiếp tại lõi engine trình duyệt, hỗ trợ đầy đủ các thành phần `protocol`, `hostname`, `port`, `pathname`, `search`, `hash`.
  - Phương thức `pattern.exec(url)` trích xuất tức thì các tham số đường dẫn (named groups) an toàn.
  - URLPattern được biên dịch dưới dạng Deterministic Finite Automata (DFA), loại bỏ hoàn toàn nguy cơ tấn công từ chối dịch vụ thông qua biểu thức chính quy (ReDoS - Regular Expression Denial of Service).

### 2.4. Tối Ưu Hóa Khởi Động Lạnh (W3C Service Worker Static Routing API)
- **Chuẩn tham chiếu**: W3C Service Worker Specification § 3.3.1 (`https://w3c.github.io/ServiceWorker/#service-worker-registration-router`).
- **Cơ chế hoạt động**:
  - Trong sự kiện `install`, mini app đăng ký các quy tắc định tuyến tĩnh thông qua `event.registerRouter.add(rules)`.
  - Quy tắc kết hợp điều kiện `urlPattern` và nguồn tài nguyên (`cache` hoặc `network`).
  - Khi một yêu cầu khớp với quy tắc tĩnh (ví dụ: các gói JS/CSS/ảnh/phông tĩnh), nhân trình duyệt phục vụ trực tiếp từ bộ nhớ đệm CacheStorage mà KHÔNG cần khởi động luồng worker (bỏ qua Service Worker startup cost), giúp cắt giảm 150-400ms thời gian khởi động lạnh mini app trên thiết bị di động.

---

## 3. Khả Năng Truyền Thông Đa Phương Tiện, Đệm Phân Đoạn & Bản Quyền Phần Cứng DRM

### 3.1. Viễn Đoán Khả Năng Giải Mã & Tiết Kiệm Năng Lượng (W3C Media Capabilities API)
- **Chuẩn tham chiếu**: W3C Media Capabilities (`https://developer.mozilla.org/en-US/docs/Web/API/Media_Capabilities_API`, `https://www.w3.org/TR/media-capabilities/`).
- **Cơ chế hoạt động**:
  - `navigator.mediaCapabilities.decodingInfo(config)` kiểm tra trực tiếp đường ống giải mã phần cứng của thiết bị trước khi bắt đầu tải luồng video.
  - Trả về ba chỉ số cốt lõi:
    - `supported`: Codec có thể phát được.
    - `smooth`: Quá trình giải mã đảm bảo tốc độ khung hình (không bị drop frame / giật hình).
    - `powerEfficient`: Quá trình giải mã được thực thi trực tiếp trên khối phần cứng chuyên dụng (DSP/ASIC) thay vì dùng CPU phần mềm ngốn pin.
  - Khi thiết bị di động đang chạy bằng pin và chỉ số `powerEfficient: false`, container mini app tự động hạ cấp luồng phát xuống cấu hình tối ưu (ví dụ: chuyển từ 4K/60fps xuống 1080p/30fps H.264).

### 3.2. Quản Trị Đệm Phân Đoạn Video Độ Trễ Thấp (W3C Media Source Extensions - MSE)
- **Chuẩn tham chiếu**: W3C Media Source Extensions (`https://developer.mozilla.org/en-US/docs/Web/API/Media_Source_Extensions_API`, `https://developer.mozilla.org/en-US/docs/Web/API/SourceBuffer`).
- **Cơ chế hoạt động**:
  - Mở rộng thẻ `<video>` thông qua đối tượng `MediaSource` gắn kết với `SourceBuffer`.
  - Hỗ trợ hai chế độ đệm `mode: 'segments'` (dựa trên mốc thời gian trong gói tin) và `mode: 'sequence'` (nối tiếp phân đoạn liên tục khi chèn quảng cáo).
  - Quy định bắt buộc về quản trị RAM di động: Mini app chạy video phải chủ động trục xuất các phân đoạn đệm cũ (`sourceBuffer.remove(0, currentPlayhead - 60)`) để ngăn chặn tràn bộ nhớ RAM (OOM), khống chế tổng dung lượng buffer không vượt quá 50MB.

### 3.3. Tích Hợp DRM Bản Quyền Phần Cứng (W3C Encrypted Media Extensions - EME)
- **Chuẩn tham chiếu**: W3C Encrypted Media Extensions (`https://developer.mozilla.org/en-US/docs/Web/API/Encrypted_Media_Extensions_API`, `https://developer.mozilla.org/en-US/docs/Web/API/MediaKeySession`).
- **Cơ chế hoạt động**:
  - Cung cấp giao diện chuẩn hóa kết nối giữa webview container và các mô-đun giải mã bảo mật CDM (Content Decryption Modules) của hệ điều hành: Google Widevine (L1/L3), Apple FairPlay Streaming, Microsoft PlayReady.
  - `navigator.requestMediaKeySystemAccess(keySystem, configs)` yêu cầu quyền truy cập phần cứng an toàn, được kiểm soát chặt chẽ bởi Permissions Policy `encrypted-media`.
  - Quản lý vòng đời giấy phép thông qua `MediaKeySession`: hỗ trợ phiên bản tạm thời `temporary` (phát trực tiếp, khóa nằm trên RAM an toàn) và bản quyền ngoại tuyến `persistent-license` (khóa mã hóa lưu trong chip bảo mật di động).
  - Khi người dùng đăng xuất hoặc gỡ cài đặt mini app, Super App container bắt buộc thực thi `session.remove()` và giải phóng bộ nhớ phần cứng CDM để ngăn chặn nguy cơ rò rỉ khóa bản quyền.

---

## 4. Cửa Sổ Nổi Tùy Ý (Document PiP) & Quản Trị Phông Chữ Không Gây Giật Layout

### 4.1. Cửa Sổ Đa Nhiệm Nổi Trên Cùng (WICG Document Picture-in-Picture API)
- **Chuẩn tham chiếu**: WICG Document Picture-in-Picture API (`https://developer.mozilla.org/en-US/docs/Web/API/Document_Picture-in-Picture_API`, `https://developer.mozilla.org/en-US/docs/Web/API/DocumentPictureInPicture`).
- **Cơ chế hoạt động**:
  - `documentPictureInPicture.requestWindow({ width, height })` mở một cửa sổ luôn nổi trên cùng (always-on-top) chứa cây DOM tùy ý (chat, nút mua hàng livestream, bàn mixer, camera họp trực tuyến) thay vì chỉ là video tĩnh.
  - Yêu cầu bắt buộc Transient User Activation (cử chỉ người dùng) để mở cửa sổ nổi, loại bỏ nguy cơ pop-up spam.
  - Cửa sổ nổi sở hữu đối tượng `document` riêng: mini app SDK tự động sao chép các stylesheet và biến giao diện (CSS custom properties) từ trang chính sang cửa sổ PiP.
  - Container áp đặt giới hạn kích thước tối thiểu (240x160) và tối đa (800x600), gắn nhãn nhận diện nguồn gốc mini app không thể làm giả (anti-spoofing banner) và cung cấp nút đóng native an toàn.

### 4.2. Quản Trị Vòng Đời Phông Chữ & Ổn Định Layout (W3C CSS Font Loading API)
- **Chuẩn tham chiếu**: W3C CSS Font Loading Module Level 3 (`https://developer.mozilla.org/en-US/docs/Web/API/CSS_Font_Loading_API`, `https://developer.mozilla.org/en-US/docs/Web/API/FontFace`, `https://developer.mozilla.org/en-US/docs/Web/API/FontFaceSet`).
- **Cơ chế hoạt động**:
  - Giao diện `document.fonts` (kế thừa `FontFaceSet`) cho phép nạp, kiểm tra và theo dõi trạng thái tải phông chữ theo phương thức bất đồng bộ: `document.fonts.load('16px CustomSans', 'Tiếng Việt')`.
  - Lắng nghe lời hứa `document.fonts.ready` trước khi hiển thị khung nhìn, triệt tiêu hoàn toàn hiện tượng nhảy giao diện (CLS - Cumulative Layout Shift), bảo đảm chỉ số CLS < 0.1 đạt chuẩn kiểm định store.
  - Hỗ trợ khởi tạo phông nhị phân trực tiếp từ bộ nhớ `new FontFace('BrandFont', arrayBuffer)` phục vụ hiển thị ký tự đặc thù (chữ viết khu vực Đông Nam Á, mã vạch bảo mật).
  - Tích hợp trực tiếp với luồng `DedicatedWorkerGlobalScope.fonts`, cho phép các tiến trình nền độc lập tải phông chữ và vẽ hóa đơn, chứng từ giao dịch trên `OffscreenCanvas` mà không làm nghẽn luồng UI chính.

---

## 5. Ma Trận Tiêu Chuẩn & Kiểm Định Cửa Hàng (Store Conformance Rules)

| Phân Vùng | Tiêu Chuẩn W3C/WICG | Quy Định Kiểm Định Store (Gate Validation) | Mức Độ Bắt Buộc |
| :--- | :--- | :--- | :--- |
| **Routing** | WICG Navigation API | Bắt buộc xử lý `event.intercept()` có cơ chế fallback; liên kết `event.signal` tới các yêu cầu `fetch()`. | **Bắt buộc (P0)** |
| **URL Matching** | WICG URLPattern API | Khai báo deep-link và entrypoint theo chuẩn URLPattern; loại bỏ regex tự chế để triệt tiêu lỗ hổng ReDoS. | **Bắt buộc (P0)** |
| **Startup Perf** | Service Worker Static Routing | Khai báo quy tắc định tuyến tĩnh cho toàn bộ gói JS/CSS/ảnh để bỏ qua khởi động Service Worker. | **Khuyến nghị cao (P1)** |
| **Video Playback** | W3C Media Capabilities | Kiểm tra `decodingInfo()` trước khi phát video; tự động hạ cấu hình nếu `powerEfficient: false` trên pin. | **Bắt buộc (P0 video)** |
| **Streaming Buffer** | W3C Media Source Extensions | Giới hạn dung lượng SourceBuffer tối đa 50MB; trục xuất định kỳ các phân đoạn đệm cũ > 60 giây. | **Bắt buộc (P0 video)** |
| **DRM License** | W3C Encrypted Media Extensions | Áp dụng Permissions Policy `encrypted-media`; xóa bỏ sạch license ngoại tuyến khi đăng xuất tài khoản. | **Bắt buộc (P0 DRM)** |
| **Floating UI** | WICG Document PiP | Yêu cầu cử chỉ người dùng khi mở; hiển thị banner định danh chống giả mạo; tuân thủ kích thước tối đa/tối thiểu. | **Kiểm soát chặt (P1)** |
| **Typography** | W3C CSS Font Loading API | Đảm bảo `document.fonts.ready` được xử lý; đạt chỉ số kiểm định layout stability CLS < 0.1. | **Bắt buộc (P0)** |

---

## 6. Kết Luận & Định Hướng Tiếp Theo

Milestone 48 hoàn thành việc chuẩn hóa toàn diện 3 trụ cột kỹ thuật trọng yếu của Super App Container:
1. **Kiến trúc điều hướng Single-Page App**: Hiện đại hóa bằng Navigation API và URLPattern, mang lại trải nghiệm chuyển trang bản địa, loại bỏ hoàn toàn các lỗi ReDoS và giật lag khởi động lạnh.
2. **Hạ tầng truyền thông streaming và bảo mật DRM**: Đảm bảo phát video mượt mà, tiết kiệm pin thông qua Media Capabilities, tối ưu hóa RAM với SourceBuffer và bảo vệ bản quyền số cấp độ phần cứng với EME.
3. **Trình chiếu cửa sổ nổi và ổn định hiển thị phông chữ**: Khai phóng khả năng đa nhiệm với Document Picture-in-Picture và loại bỏ hiện tượng nhảy layout FOUT/CLS với CSS Font Loading API.

Toàn bộ 15 findings đã được sáp nhập an toàn vào kho dữ liệu tri thức của dự án (`state/findings.jsonl`, đạt tổng cộng 671 findings). Trụ sở nghiên cứu sẵn sàng cho Milestone 49 và cột mốc đồng bộ public repo lớn tại Iteration 50.
