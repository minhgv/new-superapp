# Chuyên Đề 48 (Iteration 52): Privacy Sandbox Topics API, W3C CSS Color 4 / CSS Color HDR, và W3C Web of Things (WoT) IoT Orchestration

## 1. Bối Cảnh & Mục Tiêu Nghiên Cứu
Tại Iteration 52, hệ thống nghiên cứu Deli Deep tiếp tục hoàn thiện khung chuẩn kỹ thuật cho mini app store trong siêu ứng dụng (Super App), tập trung vào 3 trụ cột công nghệ nền tảng:
1. **Privacy Sandbox Topics API**: Chuẩn hoá cơ chế cá nhân hoá quảng cáo và đề xuất nội dung mini-app trên thiết bị (on-device interest personalization) theo chu kỳ epoch hàng tuần, bổ sung nhiễu vi phân (differential privacy) và loại trừ tuyệt đối các danh mục nhạy cảm (GDPR Article 9). Thay thế triệt để các kỹ thuật fingerprinting, IMEI hay UUID tracking ngầm bằng API `document.browsingTopics()` và header mạng `Sec-Browsing-Topics`.
2. **W3C CSS Color Module Level 4 & CSS Color HDR**: Chuẩn hoá không gian màu dải rộng (Wide Gamut) `color(display-p3)` mở rộng hơn 50% dải màu so với sRGB trên màn hình OLED/Retina di động; hệ màu cảm nhận đồng nhất OKLCH `oklch(L C H)` phục vụ thiết kế giao diện động và tự động tính toán tỷ lệ tương phản WCAG 2.2 AA; media query `@media (color-gamut)` và `@media (dynamic-range: high)` kiểm soát ánh xạ sắc độ (tone mapping) và kẹp dải sáng SDR cho thanh điều hướng/nút bấm container.
3. **W3C Web of Things (WoT) Standards Suite**: Chuẩn hoá kiến trúc điều phối thiết bị phần cứng IoT thông minh thông qua bộ chuẩn W3C Recommendation (Thing Description 1.1 JSON-LD, WoT Architecture 1.1 Servient, WoT Discovery mDNS/DNS-SD, WoT Profile HTTP/Async và Binding Templates cho HTTP/MQTT/CoAP). Loại bỏ hoàn toàn sự phụ thuộc vào các SDK phần cứng đóng kín độc quyền, cho phép mini-app giao tiếp trực tiếp với thiết bị gia dụng và cảm biến thông qua giao diện chuẩn hóa an toàn.

---

## 2. Chi Tiết Phát Hiện & Bằng Chứng Kỹ Thuật (Findings)

### Nhóm 1: Privacy Sandbox Topics API & On-Device Personalization

#### finding: `topics_052_01` — WICG Topics API: Cá Nhân Hoá Sở Thích Trên Thiết Bị Không Dùng Cookie Hay Fingerprinting
- **Tiêu chuẩn tham chiếu**: WICG Topics API / MDN Web APIs Topics API.
- **Nguồn xác thực**: [MDN Topics API](https://developer.mozilla.org/en-US/docs/Web/API/Topics_API), [Chrome Privacy Sandbox Topics](https://developer.chrome.com/docs/privacy-sandbox/topics), [Privacy Sandbox Technologies](https://privacysandbox.com/open-web/#the-privacy-sandbox-technologies).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Topics API cung cấp cơ chế bảo vệ quyền riêng tư cho quảng cáo theo sở thích mà không cần theo dõi lịch sử duyệt web xuyên ứng dụng/trang web.
  - Runtime trình duyệt hoặc host container phân loại hostname của mini-app theo một phân loại học (taxonomy) thô được tuyển chọn công khai (ví dụ: 'Fitness', 'Travel', 'Consumer Electronics').
  - Định kỳ mỗi tuần (epoch), thiết bị tính toán 5 chủ đề hàng đầu của người dùng dựa trên tần suất tương tác, cộng thêm 1 chủ đề ngẫu nhiên (xác suất 5%) để bảo đảm tính ẩn danh vi phân (differential privacy) và khả năng phủ nhận hợp lý (plausible deniability). Chủ đề được lưu trữ tối đa trong 3 epoch (3 tuần) trước khi tự động xoá.
  - Một bên gọi (caller) chỉ có thể nhận được một chủ đề nếu bên đó đã từng hiện diện và ghi nhận quan sát trên mini-app thuộc chủ đề đó trong cùng epoch.
- **Yêu cầu Store Standard**:
  - Mạng lưới quảng cáo nội bộ và bộ máy gợi ý mini-app phải chuyển dịch từ fingerprinting/IMEI/UUID sang Topics API.
  - Container WebView kiểm soát quyền đọc chủ đề thông qua Permissions Policy `browsing-topics`.
  - Cổng phân tích của Super-app kiểm toán tần suất truy cập taxonomy nhằm ngăn chặn việc tổng hợp biểu đồ định danh xuyên epoch.

#### finding: `topics_052_02` — JavaScript Interface 'document.browsingTopics()': Điều Phối Quan Sát và Bộ Lọc Epoch
- **Tiêu chuẩn tham chiếu**: WICG Topics API Section 3 / MDN Document.browsingTopics.
- **Nguồn xác thực**: [MDN Document: browsingTopics() method](https://developer.mozilla.org/en-US/docs/Web/API/Document/browsingTopics), [Chrome Privacy Sandbox Topics Overview](https://developer.chrome.com/docs/privacy-sandbox/topics/overview).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Phương thức `document.browsingTopics({skipObservation: boolean})` trả về một Promise phân giải thành danh sách các đối tượng Topic đại diện cho sở thích của người dùng trong 3 epoch gần nhất.
  - Mỗi đối tượng Topic chứa: `configVersion` (phiên bản cấu hình bộ phân loại), `modelVersion` (định danh mô hình ML), `taxonomyVersion` (số phiên bản taxonomy chuẩn), `topic` (ID số nguyên ánh xạ vào taxonomy công khai), và `version` (chuỗi thực thi kết hợp).
  - Tùy chọn `{skipObservation: true}` cho phép các hệ thống đấu thầu quảng cáo kiểm tra chủ đề hiện tại mà không làm ghi nhận lượt truy cập của trang hiện tại vào lịch sử quan sát, tránh làm loãng chủ đề khi prefetch ngầm.
- **Yêu cầu Store Standard**:
  - Mini-app ở chế độ khách (guest/unauthenticated) nhận mảng rỗng khi gọi `document.browsingTopics()`.
  - Quy trình kiểm duyệt store bắt buộc mini-app phải khai báo quyền đọc topics trong `manifest.json` (`privacy_sandbox.topics: true`).
  - Khi người dùng bật chế độ ẩn danh (Incognito/Private Mode) trong Super-app, API lập tức trả về mảng rỗng, ngắt hoàn toàn liên kết với hồ sơ người dùng.

#### finding: `topics_052_03` — Taxonomy Phân Cấp & Chính Sách Loại Trừ Danh Mục Nhạy Cảm (Sensitive Exclusion)
- **Tiêu chuẩn tham chiếu**: Chrome Privacy Sandbox Topics Classification Architecture / WICG Topics Taxonomy.
- **Nguồn xác thực**: [Chrome Topics Topic Classification](https://developer.chrome.com/docs/privacy-sandbox/topics/topic-classification), [Chrome Privacy Sandbox Topics Overview](https://developer.chrome.com/docs/privacy-sandbox/topics/overview).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Phân loại Topics sử dụng một taxonomy dạng cây có phiên bản công khai (Taxonomy v2 chứa khoảng 469 chủ đề). Quá trình phân loại thực thi thông qua mô hình TensorFlow Lite gọn nhẹ trên thiết bị hoặc bảng ánh xạ tĩnh tên miền.
  - Taxonomy áp dụng nguyên tắc loại trừ tuyệt đối đối với các danh mục nhạy cảm được quy định bởi GDPR Điều 9 và pháp luật bảo vệ dữ liệu cá nhân: nghiêm cấm sự tồn tại của các chủ đề liên quan đến tình trạng y tế/sức khoẻ, nguồn gốc chủng tộc/dân tộc, quan điểm chính trị, niềm tin tôn giáo, tư cách công đoàn và khuynh hướng tình dục.
- **Yêu cầu Store Standard**:
  - Danh mục ngành hàng trên Super-app Store ánh xạ trực tiếp sang mã Topic ID chuẩn hoá.
  - Các mini-app thuộc lĩnh vực nhạy cảm (khám bệnh từ xa, tôn giáo, trợ cấp tài chính/nợ xấu) được gắn nhãn `no-topics`, được miễn trừ hoàn toàn khỏi quá trình phân loại và quan sát.
  - Host runtime ngăn chặn mọi hành vi của SDK bên thứ ba cố tình suy diễn trạng thái sức khỏe/tài chính từ các cụm chủ đề thông thường.

#### finding: `topics_052_04` — Đàm Phán Chủ Đề Qua Header HTTP: 'Sec-Browsing-Topics' và 'Observe-Browsing-Topics'
- **Tiêu chuẩn tham chiếu**: WICG Topics API HTTP Fetch Integration / Fetch Standard Sec-Browsing-Topics.
- **Nguồn xác thực**: [Chrome Privacy Sandbox Topics](https://developer.chrome.com/docs/privacy-sandbox/topics), [MDN Topics API](https://developer.mozilla.org/en-US/docs/Web/API/Topics_API).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Tích hợp đàm phán chủ đề trực tiếp vào tầng mạng HTTP mà không cần thực thi JavaScript. Khi gọi `fetch(url, {browsingTopics: true})` hoặc nhúng `<iframe src="..." browsingtopics>`, trình duyệt tính toán các topic đủ điều kiện và gửi kèm request header `Sec-Browsing-Topics`.
  - Phía server nếu muốn ghi nhận quan sát trên thiết bị sẽ phản hồi bằng response header `Observe-Browsing-Topics: ?1`.
  - Cơ chế này giúp tối ưu hóa hiệu năng, giảm thiểu rác CPU trên main-thread do không phải khởi tạo và chạy các đoạn script DOM phức tạp.
- **Yêu cầu Store Standard**:
  - Proxy mạng và bộ chặn tài nguyên của Super-app kiểm soát việc lan truyền header `Sec-Browsing-Topics`: tự động loại bỏ header này đối với các domain cross-origin không nằm trong danh sách cho phép (allowlist) của manifest.
  - Giới hạn tốc độ (rate limit) đối với header `Observe-Browsing-Topics: ?1` để ngăn chặn các đối tác quảng cáo spam lịch sử quan sát.
  - Nghiêm cấm tuyệt đối chèn header này trong các luồng giao dịch thanh toán hoặc xác thực KYC.

#### finding: `topics_052_05` — Quản Trị Phân Quyền 'browsing-topics' & Bảng Điều Khiển Minh Bạch Người Dùng
- **Tiêu chuẩn tham chiếu**: WICG Topics API Security & Privacy / W3C Permissions Policy 'browsing-topics'.
- **Nguồn xác thực**: [Chrome Privacy Sandbox Topics Overview](https://developer.chrome.com/docs/privacy-sandbox/topics/overview), [MDN Topics API](https://developer.mozilla.org/en-US/docs/Web/API/Topics_API).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - API được bảo vệ nghiêm ngặt bởi tính năng Permissions Policy `browsing-topics`. Document cấp cao nhất kiểm soát quyền ủy quyền cho iframe thông qua HTTP header `Permissions-Policy: browsing-topics=()` (vô hiệu hóa hoàn toàn) hoặc `browsing-topics=(self)` (chỉ cho phép mã nguồn first-party).
  - Chuẩn quy định tác nhân người dùng phải cung cấp giao diện trực quan cho phép người dùng xem danh sách các chủ đề đang được lưu trữ trên thiết bị, cho phép xoá từng chủ đề cụ thể hoặc tắt hoàn toàn tính năng.
- **Yêu cầu Store Standard**:
  - Super App cung cấp trang 'Cài đặt Quảng cáo & Sở thích' hiển thị minh bạch các chủ đề đang hoạt động.
  - Người dùng có thể xóa từng chủ đề hoặc tắt đồng bộ hóa quảng cáo cá nhân hoá toàn hệ thống bất cứ lúc nào.
  - Hệ thống quét mã nguồn tự động gắn cờ cảnh báo nếu mini-app tìm cách vượt qua cơ chế Permissions Policy qua các kênh lưu trữ phụ.

---

### Nhóm 2: W3C CSS Color 4 & High Dynamic Range (HDR) Graphics Governance

#### finding: `color_hdr_052_01` — W3C CSS Color 4: Không Gian Màu Dải Rộng 'color(display-p3)' Cho Thiết Bị Di Động
- **Tiêu chuẩn tham chiếu**: W3C CSS Color Module Level 4 Section 10 (Specifying Colors in Predefined Color Spaces).
- **Nguồn xác thực**: [W3C CSS Color Module Level 4](https://www.w3.org/TR/css-color-4/), [MDN CSS color() function](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/color).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - CSS Color 4 giới thiệu hàm `color()` cho phép định nghĩa màu sắc trong các không gian màu mở rộng vượt ra ngoài sRGB truyền thống, tiêu biểu là `display-p3`, `a98-rgb`, `prophoto-rgb` và `rec2020`.
  - Không gian màu Display P3 (chuẩn trên màn hình Apple Retina và Android OLED cao cấp) bao phủ thể tích màu lớn hơn khoảng 50% so với sRGB, đặc biệt ở các dải màu xanh lục và đỏ rực rỡ. Cú pháp định nghĩa giá trị float từ 0 đến 1: `color(display-p3 1 0.1 0.2)`.
- **Yêu cầu Store Standard**:
  - Design system của Super-app xuất bản bộ token giao diện hỗ trợ Display P3 kèm fallback sRGB tự động.
  - Pipeline tải tài nguyên mini-app chấp nhận ảnh SVG/PNG dải màu rộng mà không tự ý nén ép dải màu về sRGB gây xỉn màu thương hiệu.
  - WebView container kích hoạt wide-color rendering buffer trên phần cứng hỗ trợ mà không gây hiện tượng phân dải màu (banding).

#### finding: `color_hdr_052_02` — W3C CSS Color 4 'oklch()': Bảng Màu Cảm Nhận Đồng Nhất & Tự Động Hoá Tương Phản Truy Cập
- **Tiêu chuẩn tham chiếu**: W3C CSS Color Module Level 4 Section 8 (Perceptually Uniform Color Spaces: OKLab and OKLCH).
- **Nguồn xác thực**: [MDN CSS oklch() function](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/oklch), [W3C CSS Color Module Level 4](https://www.w3.org/TR/css-color-4/).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Các hệ màu tọa độ trụ truyền thống như HSL có nhược điểm chí mạng là phi đồng nhất về cảm nhận: màu vàng ở độ sáng 50% trông chói mắt trong khi màu xanh lam ở 50% lại rất tối.
  - OKLCH (`oklch(L C H [/ A])`) tách biệt toán học hoàn toàn giữa độ sáng cảm nhận (Lightness L: 0–1 hoặc 0–100%), độ rực màu (Chroma C: 0–0.4+) và góc sắc độ (Hue H: 0–360deg). Khi xoay góc Hue 360 độ, độ sáng cảm nhận của mắt người hoàn toàn không đổi. Điều này cho phép tạo các trạng thái giao diện (hover, active, disabled) và bảo đảm độ tương phản theo thuật toán một cách tuyệt đối.
- **Yêu cầu Store Standard**:
  - Động cơ chuyển đổi Dark/Light Mode của container sử dụng số học OKLCH để sinh các token tương phản đồng nhất.
  - Công cụ quét tiếp cận (accessibility audit) của Store tính toán tỷ lệ tương phản WCAG 2.2 AA dựa trên độ sáng cảm nhận OKLCH.
  - Giới hạn trần Chroma đối với các theme động để ngăn ngừa hiện tượng cắt gọt dải màu (gamut clipping) ngoài ý muốn.

#### finding: `color_hdr_052_03` — Media Query '@media (color-gamut)': Tối Ưu Tiến Hóa Giao Diện Đồ Họa Theo Phần Cứng
- **Tiêu chuẩn tham chiếu**: W3C Media Queries Level 4 / CSS Color Module Level 4 / MDN @media/color-gamut.
- **Nguồn xác thực**: [MDN @media: color-gamut](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/color-gamut), [W3C CSS Color Module Level 4](https://www.w3.org/TR/css-color-4/).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - `@media (color-gamut: ...)` cho phép truy vấn dải màu phần cứng của màn hình hiển thị với các giá trị: `srgb` (thiết bị phổ thông), `p3` (smartphone flagship, máy tính bảng hiện đại), và `rec2020` (màn hình chuyên nghiệp).
  - Cho phép lập trình viên áp dụng kỹ thuật progressive enhancement: khai báo CSS sRGB làm nền tảng và bọc các rule màu rực rỡ bên trong `@media (color-gamut: p3) { ... }`, ngăn ngừa tình trạng méo màu hoặc xỉn màu trên màn hình đời cũ.
- **Yêu cầu Store Standard**:
  - Banner quảng cáo và hoạt họa trong mini-app bắt buộc khai báo baseline sRGB và mở rộng P3 tiến tiến qua media query.
  - CDN proxy tối ưu hoá hình ảnh tự động phân phối định dạng AVIF/WebP hệ màu P3 cho thiết bị hỗ trợ, tiết kiệm băng thông mạng đối với thiết bị chỉ có màn hình sRGB.

#### finding: `color_hdr_052_04` — W3C CSS Color HDR & Media Query '@media (dynamic-range)': Ánh Xạ Sắc Độ & Điều Tiết Độ Sáng
- **Tiêu chuẩn tham chiếu**: W3C CSS Color HDR Specification / W3C Media Queries Level 5.
- **Nguồn xác thực**: [W3C CSS Color HDR](https://www.w3.org/TR/css-color-hdr/), [MDN @media: dynamic-range](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/dynamic-range).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Màn hình di động hiện đại đạt đỉnh độ sáng (peak brightness) từ 1500 đến hơn 2500 nits khi phát nội dung HDR. CSS Color HDR định nghĩa cơ chế kết hợp giữa SDR và HDR trên cùng một lớp đồ hoạ compositing layer.
  - `@media (dynamic-range: high)` xác định màn hình hỗ trợ dải sáng cao và độ tương phản cao đồng thời. Các thuộc tính kiểm soát khoảng sáng động (`dynamic-range-limit: standard | high | constrained-high`) ngăn chặn các điểm sáng HDR quá mức làm mờ hoặc chói mắt các vùng văn bản giao diện lân cận, đồng thời bảo vệ pin thiết bị.
- **Yêu cầu Store Standard**:
  - Mini-app streaming video hoặc thương mại điện tử có video sản phẩm HDR phải tuân thủ chính sách tone-mapping.
  - Container áp đặt giới hạn SDR trên tất cả các thanh công cụ điều hướng hệ thống (nút capsule, menu cài đặt, hộp thoại thông báo) để giao diện Super-app không bị lóa trắng.
  - Khi thiết bị kích hoạt chế độ tiết kiệm pin (Battery Saver Mode), container tự động ép `dynamic-range` về `standard`.

#### finding: `color_hdr_052_05` — Cú Pháp Màu Tương Đối (Relative Color Syntax) & Thuật Toán Gamut Mapping Delta-E OKLCH
- **Tiêu chuẩn tham chiếu**: W3C CSS Color Module Level 4 Section 13 (Gamut Mapping) & Section 14 (Relative Colors).
- **Nguồn xác thực**: [W3C CSS Color Module Level 4](https://www.w3.org/TR/css-color-4/), [MDN CSS oklch() function](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/oklch).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Relative Color Syntax cho phép bóc tách một biến màu sẵn có thành các kênh thành phần (L, C, H, alpha) và biến đổi toán học trực tiếp: `background: oklch(from var(--brand) calc(l * 0.9) c h)`.
  - Khi một giá trị màu vượt quá không gian màu vật lý của màn hình, CSS Color 4 quy định thuật toán Gamut Mapping Delta-E OKLCH: tự động giảm dần Chroma trong khi giữ nguyên Lightness và Hue cho đến khi màu vừa khít vào thể tích đích, giữ trọn vẹn sắc thái thị giác thay vì cắt xén RGB méo mó.
- **Yêu cầu Store Standard**:
  - Framework UI của Super-app cung cấp 1 biến màu thương hiệu duy nhất; các sắc thái (tint/shade/alpha) được tính toán tự động bằng Relative Color Syntax.
  - Giảm tới 70% dung lượng CSS theme do không phải lưu trữ các bảng mã màu tĩnh trùng lặp.
  - Bảo đảm hiển thị mượt mà trên mọi thiết bị Android giá rẻ mà không xuất hiện lỗi đồ họa.

---

### Nhóm 3: W3C Web of Things (WoT) IoT & Smart Hardware Orchestration

#### finding: `wot_iot_052_01` — W3C WoT Thing Description 1.1: Mô Hình Ngữ Nghĩa JSON-LD & Khả Năng Tương Tác Phần Cứng
- **Tiêu chuẩn tham chiếu**: W3C Recommendation 05 December 2023 (Web of Things Thing Description 1.1).
- **Nguồn xác thực**: [W3C WoT Thing Description 1.1](https://www.w3.org/TR/wot-thing-description11/), [W3C WoT Architecture 1.1](https://www.w3.org/TR/wot-architecture11/).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - WoT Thing Description (TD) 1.1 định nghĩa lược đồ JSON-LD chuẩn hóa mô tả năng lực phần cứng của thiết bị IoT.
  - Mô hình hoá tương tác thiết bị thành 3 loại khả năng cốt lõi (Affordances):
    1. `Properties`: trạng thái đọc/ghi của thiết bị (nhiệt độ hiện tại, mức pin, trạng thái bật/tắt).
    2. `Actions`: các thao tác có thể kích hoạt kèm schema đầu vào/đầu ra (khởi động lại, mở khóa thông minh, hẹn giờ).
    3. `Events`: các thông báo đẩy và sự kiện bất đồng bộ phát ra từ thiết bị (phát hiện chuyển động, cảnh báo rò rỉ nước).
  - Tích hợp sẵn định nghĩa bảo mật (NoSecurity, Basic, Digest, Bearer, OAuth2, APIKey, PSK) và schema dữ liệu dựa trên JSON Schema.
- **Yêu cầu Store Standard**:
  - Nhà cung cấp phần cứng thông minh (khóa cửa, điều hòa, sạc xe điện) muốn tích hợp vào siêu ứng dụng phải cung cấp file TD JSON-LD hợp chuẩn.
  - Super-app container biên dịch TD thành các đối tượng proxy JavaScript trong sandbox (`thing.readProperty()`, `thing.invokeAction()`), giải phóng mini-app khỏi việc phải nhúng các SDK IoT độc quyền nặng nề.

#### finding: `wot_iot_052_02` — W3C WoT Architecture 1.1: Mô Hình Servient Runtime & Phân Lập An Toàn Giao Thức
- **Tiêu chuẩn tham chiếu**: W3C Recommendation 05 December 2023 (Web of Things Architecture 1.1).
- **Nguồn xác thực**: [W3C WoT Architecture 1.1](https://www.w3.org/TR/wot-architecture11/), [W3C WoT Thing Description 1.1](https://www.w3.org/TR/wot-thing-description11/).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - WoT Architecture 1.1 xác lập kiến trúc kết nối giữa Things (vật lý), Consumers (ứng dụng điều khiển) và Intermediaries (cổng gateway trung gian).
  - Khối phần mềm cốt lõi là 'WoT Servient', chứa ngăn xếp ràng buộc giao thức (Protocol Binding Stack), runtime Scripting API và bộ điều phối bảo mật. Consumer Servient đọc Thing Description, xác thực bảo mật, tự động đàm phán giao thức (HTTP/HTTPS, CoAP/CoAPS, MQTT, WebSocket) và mở ra giao diện lập trình cấp cao cho script ứng dụng.
- **Yêu cầu Store Standard**:
  - Host container của Super-app đóng vai trò là một Consumer Servient được chứng nhận.
  - Mini-app chỉ thực thi ở tầng ứng dụng cấp cao, tuyệt đối không được cấp quyền mở raw TCP/UDP socket trực tiếp tới mạng nội bộ.
  - Container trung gian bắt buộc người dùng xác thực sinh trắc học (vân tay/khuôn mặt) trước khi cho phép thực thi các hành động nhạy cảm (mở khóa cửa, tắt camera an ninh).

#### finding: `wot_iot_052_03` — W3C WoT Discovery: Khám Phá Thiết Bị mDNS/DNS-SD & Ghép Đôi Không Cần Cấu Hình
- **Tiêu chuẩn tham chiếu**: W3C Recommendation 05 December 2023 (Web of Things Discovery).
- **Nguồn xác thực**: [W3C WoT Discovery](https://www.w3.org/TR/wot-discovery/), [W3C WoT Architecture 1.1](https://www.w3.org/TR/wot-architecture11/).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - WoT Discovery định nghĩa 3 cấp độ khám phá thiết bị: (1) Khám phá cục bộ thông qua Multicast DNS (mDNS) và DNS-SD với định danh dịch vụ `_wot._tcp` và `_wot._sub._http`; (2) Thăm dò trực tiếp cổng CoRE Link Format (`/.well-known/wot-thing-description`); (3) Truy vấn danh bạ thiết bị tập trung (Thing Directory) qua REST API hỗ trợ SPARQL/JSONPath.
  - Dữ liệu khám phá trả về URL trỏ tới Thing Description kèm mã băm toàn vẹn (metadata hash) ngăn chặn tấn công giả mạo man-in-the-middle.
- **Yêu cầu Store Standard**:
  - Quá trình ghép đôi thiết bị trong mini-app được thực hiện thông qua module discovery ngầm của Super App trên Wi-Fi được cấp phép.
  - Super App hiển thị danh sách thiết bị tìm thấy qua native UI picker; cấm mini-app tự ý quét ngầm toàn bộ dải mạng LAN của người dùng.
  - File TD tải về được lưu trong bộ nhớ đệm container và xác minh chữ ký chứng chỉ trước khi mini-app được phép gửi lệnh.

#### finding: `wot_iot_052_04` — W3C WoT Profile: Bộ Ràng Buộc Chuẩn Bảo Đảm Khả Năng Tương Thích Thiết Bị Giới Hạn
- **Tiêu chuẩn tham chiếu**: W3C Recommendation 05 December 2023 (Web of Things Profile).
- **Nguồn xác thực**: [W3C WoT Profile](https://www.w3.org/TR/wot-profile/), [W3C WoT Thing Description 1.1](https://www.w3.org/TR/wot-thing-description11/).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Do Thing Description 1.1 có độ linh hoạt rất lớn, W3C WoT Profile định nghĩa các bộ ràng buộc thực thi chuẩn hoá nhằm bảo đảm tính tương thích tuyệt đối giữa các thiết bị độc lập:
    1. `HTTP Basic Profile`: quy định bắt buộc giao thức HTTP/1.1 hoặc HTTP/2, cấu trúc JSON tuần tự hoá nghiêm ngặt và các phương thức bảo mật cơ bản (NoSecurity, APIKey, Bearer).
    2. `Async Profile`: chuẩn hoá các mẫu truyền tin bất đồng bộ qua SSE, WebSocket hoặc MQTT với các cấp độ QoS và nhịp tim (heartbeat) đồng nhất.
- **Yêu cầu Store Standard**:
  - Hồ sơ đăng ký thiết bị IoT trên Store bắt buộc phải chứng nhận tương thích với WoT HTTP Basic Profile hoặc Async Profile.
  - Bộ kiểm thử tự động của Store từ chối các thiết bị không vượt qua bộ test suite WoT Profile Conformance.
  - Các thiết bị cũ sử dụng giao thức độc quyền phải đi qua một bộ chuyển đổi gateway đạt chuẩn trước khi được kết nối vào hệ thống.

#### finding: `wot_iot_052_05` — W3C WoT Binding Templates: Ánh Xạ Giao Thức Tuần Tự Hoá HTTP, MQTT, CoAP & Modbus
- **Tiêu chuẩn tham chiếu**: W3C Note / Working Group Note (Web of Things Binding Templates).
- **Nguồn xác thực**: [W3C WoT Binding Templates](https://www.w3.org/TR/wot-binding-templates/), [W3C WoT Thing Description 1.1](https://www.w3.org/TR/wot-thing-description11/).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Binding Templates định nghĩa từ vựng và quy tắc ánh xạ các hành vi tương tác cấp cao (`readProperty`, `writeProperty`, `observeProperty`, `invokeAction`) thành các gói tin giao thức cụ thể.
  - Với HTTP: `readProperty` ánh xạ thành HTTP GET, `writeProperty` thành PUT/POST, `invokeAction` thành POST.
  - Với MQTT: `observeProperty` ánh xạ thành lệnh SUBSCRIBE vào topic đo lường viễn thấu, trong khi `writeProperty` publish vào command topic với cờ retain phù hợp.
  - Với CoAP: ánh xạ thành CoAP GET kèm tùy chọn Observe (RFC 7641) và mã hóa nhị phân CBOR gọn nhẹ.
- **Yêu cầu Store Standard**:
  - SDK phát triển mini-app tích hợp sẵn module Binding Templates trong thư viện trừu tượng hóa phần cứng.
  - Lập trình viên chỉ cần gọi các hàm JavaScript bất đồng bộ tiêu chuẩn mà không cần tự viết mã quản lý kết nối socket theo từng giao thức.
  - Tường lửa sandbox của container ngăn chặn việc gửi các payload nhị phân bất thường gây tràn bộ đệm phần cứng.

---

## 3. Ma Trận Đối Soát Tiêu Chuẩn & Khuyến Nghị Kiến Trúc Siêu Ứng Dụng

| Hạng Mục | Tiêu Chuẩn Kỹ Thuật | Phạm Vi Áp Dụng Trên Super App | Mức Độ Tuân Thủ Store |
| :--- | :--- | :--- | :--- |
| **On-Device Personalization** | WICG Topics API / Privacy Sandbox Taxonomy v2 | Đề xuất mini-app & cá nhân hoá banner không dùng cookie/fingerprinting | Bắt buộc (Mandatory) cho mạng ad network nội bộ |
| **Quyền Riêng Tư & Kiểm Soát** | W3C Permissions Policy `browsing-topics` | Bảng điều khiển minh bạch sở thích người dùng, chế độ ẩn danh | Bắt buộc (Mandatory) |
| **Hiển Thị Dải Màu Rộng** | W3C CSS Color Module Level 4 `color(display-p3)` | Tối ưu hiển thị đồ họa rực rỡ trên màn hình OLED/Retina cao cấp | Khuyến nghị (Recommended) |
| **Bảng Màu Đồng Nhất & Contrast** | W3C CSS Color 4 `oklch()` | Thiết kế theme động Light/Dark và kiểm toán tương phản WCAG 2.2 AA | Bắt buộc (Mandatory) cho Core UI Framework |
| **Thích Ứng Dải Sáng Cao (HDR)** | W3C CSS Color HDR & `@media (dynamic-range)` | Hiển thị video sản phẩm HDR, kẹp độ sáng an toàn cho nút bấm container | Bắt buộc (Mandatory) cho mini-app Media |
| **Mô Tả Thiết Bị IoT Chuẩn Hoá** | W3C WoT Thing Description 1.1 JSON-LD | Định nghĩa thuộc tính, hành động, sự kiện cho thiết bị phần cứng thông minh | Bắt buộc (Mandatory) cho thiết bị Smart Home |
| **Phân Lập Giao Thức IoT** | W3C WoT Architecture 1.1 & Binding Templates | Container đóng vai trò Consumer Servient điều phối HTTP/MQTT/CoAP | Bắt buộc (Mandatory) tầng Host Bridge |
| **Khám Phá & Ghép Đôi Không Cấu Hình** | W3C WoT Discovery (mDNS/DNS-SD) | Native UI picker ghép đôi thiết bị, bảo vệ an toàn mạng LAN người dùng | Bắt buộc (Mandatory) cho module kết nối thiết bị |

---

## 4. Kế Hoạch Tiếp Theo (Milestone 53+)
- Nghiên cứu cơ chế W3C Push API / Web Push Protocol nâng cao kết hợp VAPID và mã hóa RFC 8291 với phân tầng thông báo ưu tiên hệ điều hành.
- Chuẩn bị tổng kết đợt nghiên cứu chu kỳ 5 vòng (Iterations 51–55) cho đợt đồng bộ hoá Public Repository tiếp theo tại Milestone 55.
