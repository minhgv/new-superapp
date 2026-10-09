# Chuyên đề Iteration 109: Chuẩn Hóa W3C CSS Text Module Level 4, CSS Color Module Level 5 và CSS Scrollbars Styling & Viewport Overscroll Governance

## 1. Bối cảnh & Mục tiêu Kỹ thuật

Trong các kiến trúc container super-app hiện đại (như Alibaba WindVane, WeChat Mini Program, Baidu Smart Program, Viettel SuperApp), giao diện mini app chạy trên môi trường nhúng WebView đa nền tảng (Android System WebView, iOS WKWebView) phải đối mặt với các thách thức lớn về thẩm mỹ hiển thị, khả năng thích ứng động và sự phân lập cử chỉ cuộn:
1. **W3C CSS Text Module Level 4 (Advanced Mobile Typography)**: Cung cấp các cơ chế định dạng văn bản nâng cao như cân bằng dòng tiêu đề (`text-wrap: balance`), loại bỏ từ mồ côi (`text-wrap: pretty`), chuẩn hóa khoảng cách dấu câu CJK (`text-spacing-trim`) cho các thị trường Đông Á, cùng kiểm soát gạch nối mềm (`hyphenate-character`) và nén khoảng trắng (`white-space-collapse`).
2. **W3C CSS Color Module Level 5 (Dynamic Palette Engineering)**: Định nghĩa các không gian màu cảm nhận đồng đều (Oklab/Oklch), hàm trộn màu thuật toán (`color-mix()`) để sinh các trạng thái tương tác từ biến màu thương hiệu của super-app host, cú pháp màu tương đối (Relative Color Syntax) đảm bảo tự động đạt tỷ lệ tương phản WCAG 2.2 AA, cấu hình màu chuẩn (`@color-profile`) cho màn hình dải màu rộng (Display P3/Rec. 2020), và bảo vệ chỉ báo ngữ nghĩa khi hệ điều hành bật chế độ tương phản cao (`forced-color-adjust`).
3. **W3C CSS Scrollbars Styling Module Level 1 & Viewport Overscroll Governance**: Chuẩn hóa việc định kiểu thanh cuộn (`scrollbar-width`, `scrollbar-color`), thiết lập hàng rào kiểm tra tĩnh loại bỏ các selector lỗi thời không chuẩn `::-webkit-scrollbar`, và phân lập hoàn toàn cử chỉ cuộn biên (`overscroll-behavior-inline`, `overscroll-behavior-block`) để ngăn chặn việc xung đột với cử chỉ vuốt cạnh chuyển trang hoặc kéo xuống làm mới (pull-to-refresh) của container super-app gốc.

---

## 2. Chi tiết Các Findings Nghiên cứu (Iteration 109)

### 2.1. Nhóm Chuẩn W3C CSS Text Module Level 4 & Advanced Typography

#### [STANDARDS-W3C-CSS-TEXT-4-CORE-ARCHITECTURE]
- **Tiêu đề**: W3C CSS Text Module Level 4: Advanced Typography Architecture, Line Breaking, and Orthographic Governance
- **Phân loại**: `runtime-environment-and-typography-standards`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://www.w3.org/TR/css-text-4/ (HTTP 200)
- **Nội dung phân tích**:
  Đặc tả W3C CSS Text Module Level 4 thiết lập các thuật toán nền tảng cho việc tạo hình chữ (text shaping), ngắt dòng, căn chỉnh và xử lý khoảng trắng trong môi trường webview di động. Trong hệ sinh thái super-app đa quốc gia, kiểu chữ phải tự động thích ứng với nhiều hệ chữ viết mà không gây giật bố cục (layout shifts) hoặc vỡ cụm ký tự. Chuẩn đưa ra các quy định quy chuẩn cho việc ngắt dòng nhạy cảm với chữ viết, nhận diện ranh giới từ tượng hình CJK và thực thi cấu trúc phân cấp kiểu chữ nghiêm ngặt. Hệ thống kiểm định mini app áp dụng các bài kiểm tra dựng DOM tự động theo chuẩn CSS Text 4 nhằm đảm bảo tính dễ đọc và thẩm mỹ nhất quán trên mọi phiên bản WebView.

#### [STANDARDS-CSS-TEXT-WRAP-BALANCE-PRETTY]
- **Tiêu đề**: CSS Text Wrap: text-wrap balance and pretty for Mobile Viewport Orphan Elimination and Headline Formatting
- **Phân loại**: `runtime-environment-and-typography-standards`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://developer.mozilla.org/en-US/docs/Web/CSS/text-wrap (HTTP 200)
- **Nội dung phân tích**:
  Thuộc tính CSS `text-wrap` (mở rộng với các từ khóa `balance` và `pretty`) chuẩn hóa việc phân phối độ dài dòng trên màn hình di động. Đối với mini app hiển thị tiêu đề sản phẩm, biểu ngữ khuyến mãi và hộp thoại thông báo, `text-wrap: balance` tính toán lặp để cân bằng độ dài các dòng chữ, loại bỏ mép phải nham nhở gây khó chịu thị giác. Ngược lại, `text-wrap: pretty` đánh giá đoạn văn bản nhằm ngăn ngừa từ mồ côi (từ đơn lẻ nằm cô độc ở dòng cuối) mà không tốn chi phí tính toán cân bằng toàn bộ khối văn bản lớn. Hướng dẫn thiết kế giao diện super-app quy định áp dụng `text-wrap: balance` cho tiêu đề và `text-wrap: pretty` cho mô tả sản phẩm.

#### [STANDARDS-CSS-TEXT-SPACING-TRIM-CJK]
- **Tiêu đề**: CSS text-spacing-trim: Standardized CJK Punctuation Trimming and Fullwidth Glyph Kerning in Asian Super-Apps
- **Phatsn loại**: `runtime-environment-and-typography-standards`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://developer.mozilla.org/en-US/docs/Web/CSS/text-spacing-trim (HTTP 200)
- **Nội dung phân tích**:
  Thuộc tính CSS `text-spacing-trim` chuẩn hóa khoảng cách kerning và tự động loại bỏ khoảng trắng dư thừa giữa các dấu câu toàn phần (fullwidth CJK punctuation marks) liền kề như dấu ngoặc, dấu chấm, dấu phẩy và dấu nháy kép. Trong các siêu ứng dụng hàng đầu châu Á (WeChat, Alipay, LINE), việc dấu câu mang khoảng trắng nội tại nửa ký tự thường tạo ra khoảng trống kép rất xấu. Khai báo `text-spacing-trim: trim-start` hoặc `space-first` cắt tỉa khoảng trống thừa trong hộp bao ký tự, đảm bảo văn bản dày dặn, thanh lịch và đạt độ hoàn thiện cao trong các màn hình thanh toán và gian hàng mini app.

#### [STANDARDS-CSS-HYPHENATE-CHARACTER-LAYOUT]
- **Tiêu đề**: CSS hyphenate-character & hyphenate-limit-chars: Responsive Multi-Language Hyphenation Controls
- **Phân loại**: `runtime-environment-and-typography-standards`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://developer.mozilla.org/en-US/docs/Web/CSS/hyphenate-character (HTTP 200)
- **Nội dung phân tích**:
  Thuộc tính CSS `hyphenate-character`, kết hợp cùng `hyphenate-limit-chars`, chuẩn hóa chuỗi ký tự gạch nối khi từ bị ngắt dòng tự động (mặc định là ký tự Unicode hyphen U+2010 hoặc ký tự phù hợp với ngôn ngữ). Trong các thẻ lưới nhỏ, ngăn kéo cạnh hoặc bảng dữ liệu nhiều cột, các chuỗi ký tự dài (từ ghép tiếng Đức, thuật ngữ y tế, mã định danh kỹ thuật) thường gây tràn khung ngang. Khai báo `hyphenate-limit-chars: 5 3 3` giới hạn chỉ những từ có ít nhất 5 ký tự mới được ngắt và giữ lại tối thiểu 3 ký tự ở mỗi phần gạch nối, duy trì bố cục rõ ràng mà không bị xén mép ngang màn hình.

#### [STANDARDS-CSS-WHITE-SPACE-COLLAPSE-HYGIENE]
- **Tiêu đề**: CSS white-space-collapse: Granular Whitespace Normalization and Code Snippet Formatting in Mini-Apps
- **Phân loại**: `runtime-environment-and-typography-standards`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://developer.mozilla.org/en-US/docs/Web/CSS/white-space-collapse (HTTP 200)
- **Nội dung phân tích**:
  Thuộc tính CSS `white-space-collapse` (được chuẩn hóa trong CSS Text Module Level 4 khi tách bạch thuộc tính rút gọn cũ `white-space`) cho phép kiểm soát chi tiết cách thức các chuỗi khoảng trắng được nén lại (sử dụng các từ khóa `collapse`, `discard`, `preserve`, `preserve-breaks`). Đối với mini app hiển thị biên lai số, bảng kê tài chính, mã đơn hàng hoặc nội dung người dùng tải lên, `white-space-collapse` giúp bảo toàn các tab thụt đầu dòng và ngắt dòng có chủ đích mà không tắt khả năng tự động bọc dòng. Cơ chế này ngăn chặn việc hiển thị sai lệch cấu trúc dữ liệu và hỗ trợ lọc an toàn văn bản đầu vào.

---

### 2.2. Nhóm Chuẩn W3C CSS Color Module Level 5 & Dynamic Palette Engineering

#### [STANDARDS-W3C-CSS-COLOR-5-CORE-ARCHITECTURE]
- **Tiêu đề**: W3C CSS Color Module Level 5: Perceptual Color Spaces, Dynamic Gamut Mapping, and Wide-Color Ergonomics
- **Phân loại**: `runtime-environment-and-theming-standards`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://www.w3.org/TR/css-color-5/ (HTTP 200)
- **Nội dung phân tích**:
  Quy chuẩn W3C CSS Color Module Level 5 giới thiệu các phép toán màu động được tính toán trực tiếp trong engine CSS. Không gian màu hex/RGB truyền thống chịu biến dạng phi tuyến tính về độ sáng cảm nhận, dẫn đến các sắc thái trung gian bị đục ngầu và mất tương phản khi đổi chủ đề giao diện. CSS Color 5 đưa vào các không gian màu đồng đều về mặt toán học (Oklab và Oklch) cùng các hàm biến đổi màu sắc có thể lập trình. Super-app host có thể phát sóng các biến token chủ đề gốc, cho phép hàng trăm mini app tự động điều chỉnh độ sáng chế độ tối và tương thích dải màu rộng mà không cần nạp thư viện JavaScript nặng tính toán màu sắc.

#### [STANDARDS-CSS-COLOR-MIX-THEME-DERIVATION]
- **Tiêu đề**: CSS color-mix(): Algorithmic State Variation and Host Theme Tinting in Super-App Mini-Apps
- **Phân loại**: `runtime-environment-and-theming-standards`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/color-mix (HTTP 200)
- **Nội dung phân tích**:
  Cú pháp hàm `color-mix()` cho phép lập trình viên mini app kết hợp hai giá trị màu theo tỷ lệ xác định trong một không gian màu chuyển tiếp (ví dụ: `color-mix(in oklab, var(--primary-color) 80%, black)`). Thay vì khai báo hàng trăm mã màu hex tĩnh cho các trạng thái hover, active, focus và disabled, UI framework của mini app có thể sinh toàn bộ trạng thái tương tác theo thuật toán. Khi super-app host bơm màu thương hiệu đối tác vào môi trường mini app runtime, `color-mix()` đảm bảo các thành phần nút bấm và thẻ thông tin tự động kế thừa độ sâu quang học và đường cong ánh sáng nhất quán giữa hai chế độ sáng/tối.

#### [STANDARDS-CSS-COLOR-PROFILE-WIDE-GAMUT]
- **Tiêu đề**: CSS @color-profile: Declarative ICC Color Space Binding for Wide-Gamut Brand Fidelity in Mini-Apps
- **Phân loại**: `runtime-environment-and-theming-standards`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://developer.mozilla.org/en-US/docs/Web/CSS/@color-profile (HTTP 200)
- **Nội dung phân tích**:
  Quy tắc at-rule CSS `@color-profile` cho phép mini app khai báo các hồ sơ màu được hỗ trợ bởi tệp ICC hoặc định danh nền tảng chuẩn. Các thiết bị di động màn hình OLED hiện đại đa phần hỗ trợ dải màu rộng như Display P3 và Rec. 2020. Định nghĩa màu sRGB tiêu chuẩn thường làm xén mất các sắc độ sống động, làm suy giảm tính chính xác của màu sắc nhận diện thương hiệu xa xỉ và hình ảnh sản phẩm thương mại điện tử. Thông qua `@color-profile`, nền tảng super-app cam kết màu sắc hiển thị trùng khớp với thông số kỹ thuật được hiệu chuẩn bởi doanh nghiệp, xóa bỏ tình trạng lệch màu giữa vỏ super-app gốc và mini app nhúng.

#### [STANDARDS-CSS-RELATIVE-COLORS-PALETTE-SYNTHESIS]
- **Tiêu đề**: CSS Relative Color Syntax: Deconstructive Color Channel Derivation for Monochromatic and Accessible UI Palettes
- **Phân loại**: `runtime-environment-and-theming-standards`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_colors/Relative_colors (HTTP 200)
- **Nội dung phân tích**:
  Cú pháp CSS Relative Color Syntax cho phép bóc tách một màu hiện có thành các kênh thành phần (thông qua từ khóa `from`, như `oklch(from var(--brand-accent) calc(l - 0.2) c h / 0.85)`) và tổng hợp các biến thể màu hài hòa tức thì. Trong tiêu chuẩn super-app mini app, hướng dẫn tiếp cận bắt buộc đảm bảo tỷ lệ tương phản tối thiểu 4.5:1 cho chữ và nút bấm điều khiển. Cú pháp màu tương đối cho phép thư viện thành phần tự động tạo ra cặp màu nền-chữ dễ đọc trực tiếp trong CSS: khi đối tác nhập màu thương hiệu sáng, công thức chữ tự động đảo kênh sáng thành tối, đảm bảo tuân thủ WCAG 2.2 Level AA mà không cần quan sát DOM bằng JavaScript.

#### [STANDARDS-CSS-FORCED-COLOR-ADJUST-ACCESSIBILITY]
- **Tiêu đề**: CSS forced-color-adjust: Preservation of Critical Semantic Badges and Charts in High-Contrast Modes
- **Phân loại**: `runtime-environment-and-theming-standards`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://developer.mozilla.org/en-US/docs/Web/CSS/forced-color-adjust (HTTP 200)
- **Nội dung phân tích**:
  Thuộc tính CSS `forced-color-adjust` xác định xem màu sắc của một phần tử mini app có bị hệ điều hành ghi đè cưỡng bức bởi bảng màu trợ năng tương phản cao (`forced-colors`) hay không (giá trị: `auto` hoặc `none`). Mặc dù mini app bắt buộc phải tuân thủ chế độ tương phản cao của người dùng đối với phông chữ và nền chung, việc ghi đè mù quáng các nhãn trạng thái (như nhãn xanh 'Thành công' và đỏ 'Thất bại') hoặc các biểu đồ xu hướng sẽ phá hủy ý nghĩa ngữ nghĩa đối với người dùng khiếm thị một phần. Bằng cách áp dụng có chọn lọc `forced-color-adjust: none` kết hợp bảng màu hệ thống tương thích, mini app bảo toàn được tính toàn vẹn dữ liệu và vượt qua cổng kiểm duyệt trợ năng.

---

### 2.3. Nhóm Chuẩn W3C CSS Scrollbars Styling Module Level 1 & Viewport Overscroll Governance

#### [STANDARDS-W3C-CSS-SCROLLBARS-1-ARCHITECTURE]
- **Tiêu đề**: W3C CSS Scrollbars Styling Module Level 1: Standardized Scrollbar Sizing, Theming, and WebView Sandboxing
- **Phân loại**: `runtime-environment-and-viewport-standards`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://www.w3.org/TR/css-scrollbars-1/ (HTTP 200)
- **Nội dung phân tích**:
  Đặc tả W3C CSS Scrollbars Styling Module Level 1 thiết lập cơ chế chuẩn hóa mức cao để định dạng thanh cuộn mà không làm tổn hại đến khả năng tiếp cận hoặc hiệu năng dựng hình của nền tảng. Lịch sử webview di động bị phân mảnh bởi các pseudo-element `::-webkit-scrollbar` gây lỗi phân phối sự kiện chạm (touch events), giật bố cục và không tôn trọng chế độ tối của hệ điều hành. CSS Scrollbars 1 chuẩn hóa `scrollbar-width` (`auto`, `thin`, `none`) và `scrollbar-color` (màu tay nắm thumb và màu rãnh track). Đối với super-app chạy trên cả Android WebView và iOS WKWebView, chuẩn này mang lại thanh cuộn gọn nhẹ, đồng bộ giao diện với vỏ super-app và triệt tiêu lỗi cuộn đặc thù trình duyệt.

#### [STANDARDS-CSS-SCROLLBARS-STYLING-MDN-PRACTICE]
- **Tiêu đề**: CSS Scrollbars Styling: Modern Cross-Platform Ergonomics and Accessible Touch Container Scrollbars
- **Phân loại**: `runtime-environment-and-viewport-standards`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_scrollbars_styling (HTTP 200)
- **Nội dung phân tích**:
  Thực hành định kiểu thanh cuộn CSS hiện đại yêu cầu các thanh cuộn tùy biến không được làm suy giảm độ nhạy chạm hoặc vi phạm tương phản theo tiêu chuẩn WCAG. Trên hệ điều hành di động nơi thanh cuộn dạng lớp phủ tự động hiện ra khi vuốt chạm và biến mất khi nghỉ, thanh cuộn tùy chỉnh phải đảm bảo độ tương phản rõ ràng mà không làm dịch chuyển lề nội dung (content margins). Cổng kiểm duyệt mini app xác minh rằng các phần tử cuộn bên trong (như lịch sử giao dịch, biên lai dài hoặc danh mục cài đặt) sử dụng thuộc tính thanh cuộn chuẩn kết hợp mã màu tương phản cao, mang lại phản hồi định vị không gian chính xác cho người dùng.

#### [STANDARDS-CSS-WEBKIT-SCROLLBAR-LEGACY-MIGRATION]
- **Tiêu đề**: CSS ::-webkit-scrollbar: Legacy Vendor Pseudo-Element Containment and Static Analysis Deprecation Gate
- **Phân loại**: `runtime-environment-and-viewport-standards`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://developer.mozilla.org/en-US/docs/Web/CSS/::-webkit-scrollbar (HTTP 200)
- **Nội dung phân tích**:
  Họ pseudo-element phi chuẩn `::-webkit-scrollbar` (bao gồm `::-webkit-scrollbar-thumb`, `::-webkit-scrollbar-track`) vốn được thiết kế cho WebKit máy tính và không thuộc bất kỳ chuẩn chính thức nào. Trong kiến trúc container super-app đa công cụ render, các quy tắc này thường khiến thanh cuộn biến mất bất thường hoặc gây tràn bố cục trên các bản cập nhật WebView mới. Bộ linter của kho mini app kích hoạt cổng kiểm tra tĩnh tự động cảnh báo các quy tắc `::-webkit-scrollbar` và yêu cầu lập trình viên bổ sung dự phòng `scrollbar-width` và `scrollbar-color`, đảm bảo tính ổn định và tương thích lâu dài cho gói ứng dụng.

#### [STANDARDS-CSS-OVERSCROLL-BEHAVIOR-INLINE-CONTAINMENT]
- **Tiêu đề**: CSS overscroll-behavior-inline: Horizontal Scroll Containment and Native Edge-Swipe Gesture Isolation
- **Phân loại**: `runtime-environment-and-viewport-standards`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://developer.mozilla.org/en-US/docs/Web/CSS/overscroll-behavior-inline (HTTP 200)
- **Nội dung phân tích**:
  Thuộc tính CSS `overscroll-behavior-inline` cô lập hành vi cuộn dây chuyền (scroll chaining) dọc theo trục ngang (trong chế độ viết ngang-trên-xuống), hỗ trợ các giá trị `auto`, `contain`, và `none`. Trong các mini app sở hữu băng chuyền sản phẩm ngang (carousels), dải thẻ lọc danh mục hoặc màn hình nhiều tab, thao tác vuốt ngang của người dùng khi chạm đến điểm giới hạn biên cuộn thường kích hoạt nhầm cử chỉ vuốt cạnh của super-app (như trở về trang chủ hoặc đóng mini app). Khai báo `overscroll-behavior-inline: contain` ngăn chặn hiện tượng cuộn dây chuyền thoát ra vùng WebView và lớp nhận diện cử chỉ gốc, bảo vệ trải nghiệm tương tác liền mạch.

#### [STANDARDS-CSS-OVERSCROLL-BEHAVIOR-BLOCK-PULL-TO-REFRESH]
- **Tiêu đề**: CSS overscroll-behavior-block: Vertical Scroll Chaining Isolation and Super-App Pull-to-Refresh Collision Prevention
- **Phân loại**: `runtime-environment-and-viewport-standards`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://developer.mozilla.org/en-US/docs/Web/CSS/overscroll-behavior-block (HTTP 200)
- **Nội dung phân tích**:
  Thuộc tính CSS `overscroll-behavior-block` quản lý hành vi biên giới hạn cuộn dọc theo trục khối (thẳng đứng). Container super-app gốc thường tích hợp sẵn hành vi kéo xuống để làm mới (pull-to-refresh) và vuốt xuống để đóng hộp thoại mini app dạng bảng nổi. Khi một thùng chứa phụ bên trong (như hộp thoại cuộn điều khoản dịch vụ, giỏ hàng hoặc danh sách bình luận) chạm đỉnh hoặc đáy, việc tiếp tục vuốt sẽ kích hoạt làm mới toàn bộ trang, dẫn đến việc mất mát dữ liệu biểu mẫu đang nhập của người dùng. Áp dụng bắt buộc `overscroll-behavior-block: contain` trên các vùng cuộn nhúng chặn hoàn toàn việc lan truyền cử chỉ ra ngoài, tạo sự phân lập tương tác an toàn tuyệt đối.

---

## 3. Ma Trận Đối Soát & Kiểm Soát Quản Trị Store (Store Governance Control Matrix)

| Mã Kiểm Soát | Lĩnh Vực | Chuẩn Quy Chiếu | Yêu Cầu Tuân Thủ Store & Container SDK | Mức Độ Bắt Buộc | Cổng Kiểm Tra (Gate) |
|---|---|---|---|---|---|
| **CTL-TYPO-01** | Typography | W3C CSS Text 4 | Áp dụng `text-wrap: balance` cho các tiêu đề `<h1-h3>` và `text-wrap: pretty` cho đoạn văn bản giới thiệu sản phẩm. | Khuyến nghị (Best Practice) | Static CSS Audit & Linter |
| **CTL-TYPO-02** | CJK Kerning | W3C CSS Text 4 | Bắt buộc khai báo `text-spacing-trim` cho các gói mini app nhắm tới thị trường ngôn ngữ Đông Á (Trung, Nhật, Hàn) nhằm xóa khoảng trắng dư thừa dấu câu. | Bắt buộc (Mandatory) cho CJK | UI Localization Conformance Gate |
| **CTL-COLOR-01** | Dynamic Palette | W3C CSS Color 5 | Sử dụng `color-mix()` hoặc Relative Color Syntax để sinh các biến thể trạng thái nút từ biến gốc `--superapp-theme-color`. | Tiêu chuẩn kiến trúc (Architectural) | Container Theme Injection Test |
| **CTL-COLOR-02** | Accessibility | W3C CSS Color 5 & WCAG 2.2 | Áp dụng `forced-color-adjust: none` kèm định kiểu hệ thống cho các nhãn trạng thái giao dịch trọng yếu để tránh mất thông tin khi bật High Contrast. | Bắt buộc (Mandatory) | Automated Accessibility Gate |
| **CTL-SCROLL-01** | Scrollbars | W3C CSS Scrollbars 1 | Sử dụng chuẩn `scrollbar-width` và `scrollbar-color`; cấm dùng đơn độc các pseudo-element lỗi thời `::-webkit-scrollbar` mà không có fallback chuẩn. | Bắt buộc (Mandatory) | Static Code Analysis |
| **CTL-SCROLL-02** | Gesture Containment | W3C CSS Overscroll | Khai báo `overscroll-behavior-block: contain` trên các modal/sheet nội bộ và `overscroll-behavior-inline: contain` trên horizontal carousel để triệt tiêu xung đột pull-to-refresh và edge-swipe. | Bắt buộc (Mandatory) | UX & Touch Gesture Sandbox Gate |

---

## 4. Lộ Trình Triển Khai & Khuyến Nghị Tích Hợp Container

1. **Giai đoạn Tích hợp SDK WebView**:
   - Cập nhật engine Chromium WebView lên phiên bản hỗ trợ đầy đủ CSS Text 4 (`text-wrap`), CSS Color 5 (`color-mix`, Oklab) và CSS Scrollbars 1.
   - Bơm mặc định các biến theme quang học đồng đều vào root `:root` của container mini app.
2. **Giai đoạn Kiểm duyệt Gói (Package Pipeline Submission)**:
   - Thêm bộ kiểm tra CSS linter quét các selector `::-webkit-scrollbar`, nhắc nhở lập trình viên khai báo thuộc tính chuẩn W3C.
   - Kiểm tra các phần tử cuộn tràn (`overflow: auto/scroll`) xem đã khai báo đầy đủ `overscroll-behavior` hay chưa nhằm tránh xung đột cử chỉ với thanh điều hướng của super-app.
