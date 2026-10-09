# Milestone 83: W3C CSS Nesting Module Level 1, W3C CSS Text Module Level 3 & WHATWG Fetch Priority API Architecture

## 1. Executive Summary

Milestone 83 mở rộng tiêu chuẩn kỹ thuật Super App Mini App Store qua 3 trụ cột web platform, typographic engine và network resource scheduling nền tảng:
1. **W3C CSS Nesting Module Level 1 & Style Encapsulation Architecture**: Chuẩn hóa cú pháp lồng selector bản địa (native CSS nesting), toán tử nesting `&`, cơ chế thừa kế specificity thông qua giả lớp `:is()`, cách ly phong cách nhiều bên thuê (multi-tenant CSS isolation), và trật tự ưu tiên tầng cascade (@layer) nhằm loại bỏ sự phụ thuộc vào preprocessor (SASS/Less), tối ưu hóa CSS OM và loại trừ xung đột giao diện giữa host super-app và mini-app.
2. **W3C CSS Text Module Level 3 & Multilingual Typography Governance**: Chuẩn hóa thuật toán bẻ dòng (line breaking) theo Unicode UAX #14, căn đều văn bản (text-align: justify), kiểm soát ngắt chữ đa ngôn ngữ (`word-break: keep-all` cho CJK/tiếng Hàn vs `break-all` cho bảng số liệu), đóng gói chống tràn layout (`overflow-wrap: anywhere`), và kiểm soát dấu gạch nối từ điển (`hyphens: auto` kết hợp soft hyphen `&shy;`) phục vụ thị trường Đông Nam Á và quốc tế.
3. **WHATWG HTML / Fetch Priority API & Network Scheduling Architecture**: Chuẩn hóa gợi ý ưu tiên mạng (network priority hints) thông qua thuộc tính HTML `fetchpriority` (`high`, `low`, `auto`) trên thẻ `<img>`, `<script>`, `<link>`, thuộc tính DOM `HTMLImageElement.fetchPriority`, `HTMLScriptElement.fetchPriority`, cấu hình `RequestInit.priority` trong `window.fetch()`, tối ưu hóa Largest Contentful Paint (LCP) dưới 1.2s và ánh xạ trực tiếp vào stream priority frames của giao thức HTTP/2 và HTTP/3.

15 phát hiện mới trong milestone này nâng tổng số findings chuẩn hóa của dự án từ **1,181** lên **1,196**.

---

## 2. Danh mục 15 Findings Chi Tiết (Milestone 83)

### Nhóm 1: W3C CSS Nesting Module Level 1 & Component Style Encapsulation

#### 1. STANDARDS-CSS-NESTING-SYNTAX-SCOPING
- **Category**: `architecture-runtime`
- **Title**: W3C CSS Nesting Module Level 1 Syntax Scoping & Runtime Parser Architecture
- **Evidence Level**: W3C Candidate Recommendation Draft / Web Standard
- **URL**: `https://www.w3.org/TR/css-nesting-1/`
- **Spec Anchor**: W3C CSS Nesting Module Level 1 - Section 3 Nesting Style Rules
- **Relevance**: Mini-app yêu cầu styling thành phần theo module. CSS nesting bản địa loại bỏ sai lệch chuyển đổi từ SASS/Less, đơn giản hóa hydration CSS OM trong shadow root và thực thi ranh giới đóng gói thành phần.
- **Normative Requirements**:
  1. Engine webview mini-app phải cài đặt chuẩn parser CSS Nesting Module Level 1, hỗ trợ lồng selector trực tiếp và nesting selector tường minh (`&`).
  2. Quy tắc lồng phải tự động thừa kế độ đặc hiệu (specificity) và ngữ cảnh của selector cha, đánh giá tương đương như được bọc trong giả lớp `:is()`.
  3. Runtime CSS parser phải xử lý chính xác các khai báo lồng theo sau các quy tắc lồng mà không đòi hỏi khối CSS riêng biệt hay phân tích lookahead mơ hồ.
  4. Bộ đóng gói build mini-app phải kiểm tra độ sâu lồng selector không vượt quá giới hạn tối đa (khuyến nghị $\le$ 4 cấp) nhằm tránh bùng nổ specificity và độ trễ tính toán lại phong cách (style recalculation).
  5. Trình tiêm theme của super-app phải hỗ trợ các khối `@media`, `@supports`, `@layer` lồng bên trong phạm vi thành phần.
- **Implementation Guidelines**: Sử dụng tham chiếu `&` cha để định dạng trạng thái micro-component (`&:hover`, `&:active`, `&.is-selected`) trực tiếp trong định nghĩa thành phần; kết hợp với Declarative Shadow DOM để đóng gói web component.
- **Anti-Patterns**: Lồng selector sâu hơn 5-6 cấp gây sụt giảm hiệu năng tính toán lại style; dùng selector con universal (`& *`) phá vỡ ranh giới đóng gói.

#### 2. STANDARDS-CSS-NESTING-COMPONENT-ENCAPSULATION
- **Category**: `architecture-runtime`
- **Title**: CSS Native Nesting Selector Model & Multi-Tenant Style Isolation
- **Evidence Level**: MDN Web Docs Standard & W3C Technical Architecture
- **URL**: `https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_nesting`
- **Spec Anchor**: MDN Web Docs - CSS Nesting & Selector Specificity Processing Model
- **Relevance**: Khi mini-app bên thứ ba chạy trong webview dùng chung hoặc overlay vỏ super-app, việc cascade không kiểm soát có thể làm ô nhiễm giao diện host hoặc phá hỏng UI mini-app.
- **Normative Requirements**:
  1. Engine CSS webview phải tính toán specificity của các quy tắc lồng theo ngữ nghĩa so khớp `:is()` quy định trong CSS Selectors Level 4 (lấy specificity cao nhất trong danh sách đối số).
  2. Khi gán tiền tố lớp phạm vi của host container (ví dụ `.superapp-scope &`), runtime phải duy trì việc giải quyết selector xác định mà không bị đảo ngược độ đặc hiệu.
  3. Việc tiêm CSS động qua thẻ style hoặc constructable stylesheets phải bảo toàn cấu trúc CSS nesting.
  4. Trình làm sạch (sanitizer) CSS phải từ chối các quy tắc lồng độc hại cố vượt ranh giới thành phần bằng selector gốc (`:root`, `html`, `body`) khi chạy trong sub-context.
  5. Linter kiểm tra kho ứng dụng phải quét bundle CSS để đảm bảo nâng cấp tiền tố vendor cũ sang cú pháp CSS nesting chuẩn.
- **Implementation Guidelines**: Giới hạn toàn bộ CSS mini-app bên thứ ba dưới một wrapper class cấp cao nhất bằng `&` để ngăn ô nhiễm toàn cục; kết hợp với CSS Custom Properties.
- **Anti-Patterns**: Áp dụng quy ước nối chuỗi của preprocessor (như `&__element` trong BEM) vốn không hợp lệ trong native CSS nesting; ghi đè thanh điều hướng host bằng selector gốc un-scoped.

#### 3. STANDARDS-CSS-NESTING-SELECTOR-AMPERSAND-RULES
- **Category**: `architecture-runtime`
- **Title**: W3C Nesting Selector (&) Expansion & Compound Selector Semantics
- **Evidence Level**: MDN Web Docs - CSS Reference
- **URL**: `https://developer.mozilla.org/en-US/docs/Web/CSS/Nesting_selector`
- **Spec Anchor**: MDN Web Docs - Nesting selector (&) Syntax and Usage
- **Relevance**: Đảm bảo lập trình viên viết đúng compound selectors, pseudo-classes và pseudo-elements trong cây DOM mini-app mà không gặp lỗi parse ngầm.
- **Normative Requirements**:
  1. Ký tự nesting selector (`&`) phải đại diện cho các phần tử được khớp bởi danh sách selector của quy tắc cha.
  2. Trong compound selectors, `&` phải nối liền ngay với class, attribute hoặc pseudo-class (ví dụ `&.active`, `&[disabled]`) không có khoảng trắng.
  3. Khoảng trắng giữa `&` và selector kế tiếp phải được diễn giải là tổ hợp con cháu (descendant combinator).
  4. Container CSS runtime phải hỗ trợ nhiều ký tự `&` trong cùng một selector lồng (ví dụ `& + &`, `& > .child + &`) để định nghĩa quan hệ tương đối phức tạp.
  5. Công cụ kiểm tra tĩnh của app store phải quét mã nguồn CSS để gắn cờ các vị trí đặt `&` sai ngữ pháp.
- **Implementation Guidelines**: Sử dụng `& + &` để định dạng khoảng cách lề (margin-top) giữa các phần tử liền kề trong danh sách thành phần.
- **Anti-Patterns**: Nhầm lẫn `&` như một tiền tố chuỗi nối tên class (`&--modifier`), dẫn tới selector không hợp lệ.

#### 4. STANDARDS-CSS-CASCADE-LAYERS-MULTI-TENANT-GOVERNANCE
- **Category**: `architecture-runtime`
- **Title**: CSS Cascade Layers (@layer) & Multi-Tenant Priority Architecture
- **Evidence Level**: W3C Candidate Recommendation & MDN Web Docs
- **URL**: `https://developer.mozilla.org/en-US/docs/Web/CSS/@layer`
- **Spec Anchor**: MDN Web Docs - @layer & W3C Cascading and Inheritance Level 5
- **Relevance**: Mini-app đồng thời tải design system của host, thư viện component UI và tùy biến giao diện của đối tác. Cascade layers cung cấp cơ chế phân giải thứ tự ưu tiên dứt khoát bất chấp specificity.
- **Normative Requirements**:
  1. Super-app container phải thiết lập trật tự ưu tiên tầng rõ ràng ngay khi khởi tạo: `@layer reset, superapp-base, miniapp-components, miniapp-overrides`.
  2. Phong cách do host tiêm vào phải nằm ở tầng cơ sở ưu tiên thấp hơn, đảm bảo phong cách của nhà phát triển mini-app ghi đè tin cậy mà không cần hack specificity.
  3. Các style không nằm trong layer (unlayered styles) phải được coi là có độ ưu tiên cascade thông thường cao nhất, cho phép host thực hiện hotfix khẩn cấp khi cần thiết.
  4. Toolchain build mini-app phải hỗ trợ đóng gói bundle CSS của bên thứ ba vào named layers (`@layer vendor.library`) để ngăn ghi đè không kiểm soát.
  5. Đánh giá mã tự động của store phải kiểm tra stylesheet không lạm dụng khai báo `!important` trong layer thấp để đảo ngược thứ tự cascade.
- **Implementation Guidelines**: Khai báo thứ tự layer một lần ở phần head tài liệu super-app trước khi nạp CSS mini-app; khuyến khích nhà phát triển tổ chức layer nội bộ (`@layer theme, layout, components, utilities`).
- **Anti-Patterns**: Dùng selector toàn cục không giới hạn kèm `!important` để giải quyết xung đột độ đặc hiệu giữa host framework và mini-app.

#### 5. STANDARDS-CSS-NESTING-ENGINE-OPTIMIZATION-SLA
- **Category**: `performance-optimization`
- **Title**: Native CSS Nesting Engine Performance & Style Recalculation SLAs
- **Evidence Level**: Chrome Developers Architecture Guide & W3C WebPerf
- **URL**: `https://developer.chrome.com/docs/css-ui/css-nesting`
- **Spec Anchor**: Chrome Developers - CSS Nesting Relaxed Parsing & Performance Guidelines
- **Relevance**: CSS lồng quá sâu hoặc không tối ưu làm tăng đáng kể thời gian tính toán lại style trong quá trình điều hướng và cuộn trang, làm suy giảm chỉ số Interaction to Next Paint (INP) và tụt khung hình.
- **Normative Requirements**:
  1. Stylesheet mini-app dùng CSS nesting phải vượt qua linter tự động của store, đảm bảo tổng thời gian tính toán lại style không vượt quá 16ms cho 95% sự kiện tương tác trên phần cứng di động tầm trung.
  2. Runtime webview phải triển khai chuẩn relaxed CSS nesting parsing (loại bỏ yêu cầu cũ bắt buộc selector lồng phải bắt đầu bằng `&` hoặc ký hiệu), đảm bảo tương thích cú pháp hiện đại.
  3. Container phải giám sát bộ nhớ tiêu thụ bởi CSS OM; stylesheet có số lượng quy tắc lồng quá lớn (> 2,000 rules) phải bị cảnh báo khi nộp duyệt kho ứng dụng.
  4. Engine mini-app phải tận dụng constructable stylesheets (`CSSStyleSheet()`) để phân tích và chia sẻ stylesheet lồng qua nhiều web components mà không tốn công phân tích lặp lại.
  5. Công cụ build phải loại bỏ chú thích và các nhánh lồng không dùng đến khi đóng gói mini-app.
- **Implementation Guidelines**: Giữ hệ thống phân cấp lồng nông (tối đa 2-3 cấp: block > element > state); giám sát chỉ số INP qua Chrome DevTools Performance.
- **Anti-Patterns**: Sinh ra hàng nghìn selector lồng phức tạp từ các thư viện CSS-in-JS gây phình bộ nhớ và kích hoạt layout thrashing liên tục.

---

### Nhóm 2: W3C CSS Text Module Level 3 & Multilingual Typography Governance

#### 6. STANDARDS-CSS-TEXT-BREAKING-ALIGNMENT-STANDARDS
- **Category**: `architecture-runtime`
- **Title**: W3C CSS Text Module Level 3 Processing Model & International Text Typography
- **Evidence Level**: W3C Candidate Recommendation Draft - CSS Text Module Level 3
- **URL**: `https://www.w3.org/TR/css-text-3/`
- **Spec Anchor**: W3C CSS Text Module Level 3 - Section 5 Line Breaking and Word Boundaries
- **Relevance**: Super-app phục vụ thị trường đa ngôn ngữ (tiếng Việt, tiếng Lào, tiếng Thái, tiếng Trung, tiếng Anh). Quy tắc ngắt dòng chính xác ngăn chặn vỡ khung giao diện và cắt cụt văn bản trên màn hình di động hẹp.
- **Normative Requirements**:
  1. Engine typography webview của mini-app phải tuân thủ thuật toán ngắt dòng của W3C CSS Text Module Level 3 trên các chữ viết phức tạp và phi Latin.
  2. Runtimes phải hỗ trợ `text-align: justify` với `text-justify: inter-character / inter-word` theo ngữ nghĩa ngôn ngữ tài liệu được khai báo trong thuộc tính `lang`.
  3. Các thuộc tính biến đổi văn bản (`text-transform: uppercase, lowercase, capitalize`) phải thực thi ánh xạ chữ hoa/thường nhạy cảm theo locale (ví dụ chữ i có dấu/không dấu tiếng Thổ Nhĩ Kỳ, dấu phụ tiếng Hy Lạp).
  4. Cơ hội ngắt dòng phải tôn trọng thuộc tính ngắt dòng Unicode (UAX #14), ngăn tràn không gạch nối ra ngoài ranh giới vùng chứa cha.
  5. Trình quét tuân thủ UI của store phải xác minh các khối văn bản chỉ định hành vi wrap phù hợp để tránh lỗi cuộn ngang trên màn hình di động nhỏ ($\le$ 360px).
- **Implementation Guidelines**: Luôn khai báo rõ thuộc tính `lang` trên phần tử gốc và các khối văn bản đa ngữ; sử dụng `white-space: normal` hoặc `white-space: pre-wrap` cho thẻ nội dung người dùng tạo.
- **Anti-Patterns**: Áp dụng `white-space: nowrap` trên các khối văn bản dài không giới hạn chiều rộng, làm giao diện tràn khỏi màn hình.

#### 7. STANDARDS-CSS-TEXT-TYPOGRAPHY-SYSTEM-STANDARDS
- **Category**: `architecture-runtime`
- **Title**: CSS Text Typography Processing System & Font Metrics Alignment
- **Evidence Level**: MDN Web Docs Standard & W3C CSS Text 3
- **URL**: `https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_text`
- **Spec Anchor**: MDN Web Docs - CSS text Overview & Text Styling Primitives
- **Relevance**: Đảm bảo giao diện mini-app tuân thủ design tokens kiểu chữ của super-app chủ quản, tránh phân mảnh thị giác và thứ bậc chữ khó đọc.
- **Normative Requirements**:
  1. Token thiết kế kiểu chữ của container phải khai báo tỷ lệ `letter-spacing` và `line-height` chuẩn hóa tương ứng với từng cấp `font-size`.
  2. Node văn bản mini-app phải tính toán căn chỉnh đường cơ sở (baseline alignment) xác định qua cả webfont tải về và font chữ hệ thống dự phòng.
  3. Container runtime phải hỗ trợ các thuộc tính `text-decoration` (`text-decoration-line`, `text-decoration-color`, `text-decoration-thickness`) tuân thủ chuẩn CSS Text 3.
  4. Hướng dẫn UI của kho ứng dụng phải bắt buộc tỷ lệ tương phản tối thiểu (WCAG 2.2 AA 4.5:1 cho văn bản thường, 3:1 cho văn bản lớn).
  5. Cỡ chữ động thông qua hàm CSS `clamp()` (ví dụ `clamp(14px, 2.5vw, 18px)`) phải được hỗ trợ trên toàn bộ webview.
- **Implementation Guidelines**: Công khai tokens typography của host dưới dạng CSS variables (`--superapp-text-sm`, `--superapp-font-regular`); áp dụng `letter-spacing: -0.01em` cho tiêu đề lớn.
- **Anti-Patterns**: Cố định `line-height` theo pixel nhỏ hơn `font-size` tính toán, làm các dòng chữ đè lên nhau khi người dùng bật tính năng phóng to chữ hệ thống.

#### 8. STANDARDS-CSS-OVERFLOW-WRAP-CONTAINMENT
- **Category**: `architecture-runtime`
- **Title**: CSS overflow-wrap (word-wrap) Layout Containment & Text Overflow Defense
- **Evidence Level**: MDN Web Docs & W3C CSS Text Module Level 3
- **URL**: `https://developer.mozilla.org/en-US/docs/Web/CSS/overflow-wrap`
- **Spec Anchor**: MDN Web Docs - overflow-wrap & W3C CSS Text 3 Section 5.5
- **Relevance**: Mini-app thường xuyên hiển thị chuỗi ký tự liền không dấu cách như URL, mã định danh tài khoản, mã hash giao dịch. Thiếu `overflow-wrap`, các chuỗi này gây vỡ khung và cuộn ngang.
- **Normative Requirements**:
  1. Tất cả vùng chứa văn bản trong mini-app render chuỗi động từ ngoài (username, mã giao dịch, địa chỉ ví) phải chỉ định `overflow-wrap: break-word` hoặc `overflow-wrap: anywhere`.
  2. Khi áp dụng `overflow-wrap: anywhere`, layout engine phải xem xét các cơ hội ngắt ký tự tùy ý, ngăn min-content intrinsic sizing đẩy chiều rộng vùng chứa vượt màn hình.
  3. Reset CSS của webview container phải mặc định gán `body`, `p`, `span`, `card` sang `overflow-wrap: break-word`.
  4. Các phần tử con flex và grid chứa văn bản phải thiết lập `min-width: 0` để thuật toán ngắt của `overflow-wrap` hoạt động chính xác trong các track auto-sizing.
  5. Kiểm tra tự động của store phải chụp ảnh màn hình ở độ rộng 320px để xác nhận chuỗi test dài không gây tràn cuộn ngang.
- **Implementation Guidelines**: Áp dụng `overflow-wrap: anywhere` trên các chuỗi mã hash giao dịch hoặc URL dài; kết hợp `min-width: 0` cho phần tử con flex.
- **Anti-Patterns**: Chỉ dựa vào `overflow: hidden` mà không có `overflow-wrap`, khiến văn bản quan trọng bị cắt cụt vô hình.

#### 9. STANDARDS-CSS-WORD-BREAK-CJK-MULTILINGUAL-GOVERNANCE
- **Category**: `architecture-runtime`
- **Title**: CSS word-break (break-all, keep-all) Multilingual Word Boundary Governance
- **Evidence Level**: MDN Web Docs & W3C CSS Text 3
- **URL**: `https://developer.mozilla.org/en-US/docs/Web/CSS/word-break`
- **Spec Anchor**: MDN Web Docs - word-break Property & Multilingual Typography Processing
- **Relevance**: Mini-app phục vụ cộng đồng đa ngôn ngữ cần ngắt chữ nhận biết hệ chữ viết. Tiếng CJK cần `keep-all` để giữ trọn cụm từ, trong khi bảng biểu tài chính cần `break-all` cho mật độ dữ liệu nhỏ.
- **Normative Requirements**:
  1. Engine bố cục mini-app phải hỗ trợ đầy đủ các giá trị của `word-break`: `normal`, `break-all`, `keep-all` tuân thủ W3C CSS Text 3.
  2. Đối với văn bản CJK/Hàn Quốc, mini-app phải hỗ trợ `word-break: keep-all` để ngăn ngắt giữa các âm tiết Hangul hoặc cụm từ tiếng Trung trừ khi bắt buộc.
  3. Đối với bảng tài chính và cột số liệu hẹp, `word-break: break-all` phải được phép sử dụng để ngắt bất cứ đâu khi độ rộng ô bị giới hạn.
  4. Container phải đảm bảo hành vi `word-break` tương tác nhất quán với `overflow-wrap` mà không triệt tiêu các điểm ngắt nối mềm.
  5. Danh mục rà soát bản địa hóa của store phải kiểm tra ứng dụng theo đúng ngôn ngữ công bố để đảm bảo ngắt chữ đúng quy ước địa phương.
- **Implementation Guidelines**: Dùng `word-break: normal; overflow-wrap: break-word` làm chuẩn cho văn bản hỗn hợp Latin và đa ngữ; dùng `keep-all` cho giao diện tiếng Hàn/Trung.
- **Anti-Patterns**: Gán `word-break: break-all` toàn cục cho đoạn văn bản tiếng Latin/Anh, tạo ra các từ bị ngắt đôi khó coi (ví dụ `su- / perapp`) làm suy giảm trải nghiệm đọc.

#### 10. STANDARDS-CSS-HYPHENS-AUTOMATED-LOCALIZATION
- **Category**: `architecture-runtime`
- **Title**: CSS hyphens (none, manual, auto) & Dictionary-Driven Text Layout
- **Evidence Level**: MDN Web Docs & W3C CSS Text Module Level 3
- **URL**: `https://developer.mozilla.org/en-US/docs/Web/CSS/hyphens`
- **Spec Anchor**: MDN Web Docs - hyphens Property & W3C CSS Text 3 Section 5.4
- **Relevance**: Mini-app tin tức, thương mại và bài viết đòi hỏi căn đều thẩm mỹ cao. Dấu nối tự động giúp triệt tiêu các khoảng trắng thưa thớt khó coi trong văn bản căn đều (justified).
- **Normative Requirements**:
  1. Webview mini-app phải hỗ trợ thuộc tính `hyphens` với các giá trị `none`, `manual`, và `auto`.
  2. Khi khai báo `hyphens: auto`, engine trình duyệt phải chọn mẫu từ điển gạch nối khớp với ngôn ngữ chỉ định trong thuộc tính `lang`.
  3. Dấu gạch nối mềm (soft hyphen, `U+00AD` hoặc `&shy;`) phải luôn được nhận diện và hiển thị như cơ hội ngắt dòng có điều kiện kể cả khi không bật auto.
  4. Runtime webview phải tích hợp sẵn gói từ điển gạch nối cho các ngôn ngữ chính được nền tảng hỗ trợ (tiếng Anh, tiếng Việt, tiếng Pháp,...).
  5. Đánh giá kiểm duyệt store phải xác minh mini-app bật `hyphens: auto` có khai báo mã ngôn ngữ BCP 47 hợp lệ trên phần tử cha để tránh lỗi im lặng.
- **Implementation Guidelines**: Kết hợp `text-align: justify` với `hyphens: auto` trên các thẻ đọc tin tức; chèn `&shy;` thủ công vào các danh từ kỹ thuật dài hoặc tên thương hiệu.
- **Anti-Patterns**: Thiết lập `hyphens: auto` mà không khai báo thuộc tính `lang`; bật gạch nối trên các nút bấm (button) hoặc tab điều hướng.

---

### Nhóm 3: WHATWG HTML / Fetch Priority API & Network Scheduling Architecture

#### 11. STANDARDS-FETCH-PRIORITY-ATTRIBUTE-SPEC
- **Category**: `performance-optimization`
- **Title**: HTML fetchpriority Attribute & Network Loading Prioritization Architecture
- **Evidence Level**: WHATWG HTML / Fetch Living Standard & MDN Web Docs
- **URL**: `https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes/fetchpriority`
- **Spec Anchor**: MDN Web Docs - fetchpriority HTML attribute & WICG Priority Hints
- **Relevance**: Mini-app phải khởi động dưới 500ms trên thiết bị di động. `fetchpriority` cho phép lập trình viên chủ động đẩy nhanh ảnh hero và script khởi động quan trọng trước các beacon phân tích hay tải nền.
- **Normative Requirements**:
  1. Container webview mini-app phải hỗ trợ thuộc tính HTML `fetchpriority` với các giá trị: `high`, `low`, và `auto`.
  2. Bộ điều phối mạng (network dispatcher) của container phải ánh xạ trực tiếp giá trị `fetchpriority` sang các frame ưu tiên luồng HTTP/2 và HTTP/3 (cây phụ thuộc/trọng số) trên kết nối multiplexed.
  3. Banner hero chính hoặc ảnh ứng viên LCP của mini-app phải khai báo `fetchpriority="high"` để đẩy nhanh phát hiện và giải mã tài nguyên trước các media dưới màn hình.
  4. Các script phân tích của bên thứ ba, tracking pixels và tài nguyên preload ngoài màn hình phải khai báo `fetchpriority="low"` để tránh chiếm dụng băng thông mạng di động.
  5. Bộ quét hiệu năng tự động của store phải xác minh không có quá 2 ảnh đồng thời đánh dấu `fetchpriority="high"` để tránh pha loãng độ ưu tiên mạng.
- **Implementation Guidelines**: Kết hợp `<link rel="preload" as="image" fetchpriority="high">` với phần tử ảnh LCP trong luồng HTML ban đầu; đánh giá ưu tiên qua cột Priority trong Network panel của DevTools.
- **Anti-Patterns**: Gán bừa bãi `fetchpriority="high"` cho toàn bộ ảnh và script trên trang, làm mất tác dụng điều phối mạng.

#### 12. STANDARDS-IMAGE-FETCH-PRIORITY-LCP-OPTIMIZATION
- **Category**: `performance-optimization`
- **Title**: HTMLImageElement fetchPriority DOM Property & Dynamic Media Scheduling
- **Evidence Level**: WHATWG HTML Living Standard & MDN Web Docs
- **URL**: `https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement/fetchPriority`
- **Spec Anchor**: MDN Web Docs - HTMLImageElement: fetchPriority property
- **Relevance**: Cho phép các framework client động (React, Vue, Svelte, Web Components) điều khiển ưu tiên tải ảnh bằng code dựa trên việc giao cắt viewport trước khi gắn vào DOM.
- **Normative Requirements**:
  1. Giao diện `HTMLImageElement` trong webview mini-app phải phơi bày thuộc tính IDL `fetchPriority` phản ánh giá trị thuộc tính nội dung.
  2. Thiết lập `img.fetchPriority = 'high'` bằng code trước khi gán `img.src` phải cập nhật ngay lập tức mức ưu tiên của yêu cầu mạng bên dưới.
  3. Trình tải ảnh động và carousel phải hạ `fetchPriority` xuống `'low'` đối với các ảnh slide nằm ngoài ranh giới hiển thị trước mắt.
  4. Khi kết hợp `loading="lazy"` với `fetchPriority="high"`, container phải đánh giá quy tắc giao cắt trước, áp dụng ưu tiên cao khi ngưỡng tải lazy được chạm tới.
  5. Tiêu chuẩn đánh giá store phải đo lường sự cải thiện LCP từ việc tối ưu `fetchPriority`, gắn cờ các ứng dụng có LCP vượt quá 2.5s trên mạng 4G giả lập.
- **Implementation Guidelines**: Đặt `img.fetchPriority = 'high'` cho slide hero hiện tại, và `'low'` cho các slide đệm lân cận; bọc trong component chuẩn như `<MiniImage hero={true} />`.
- **Anti-Patterns**: Gán `img.fetchPriority = 'high'` sau khi đã thiết lập `img.src`, làm request bị điều phối ở mức ưu tiên mặc định từ trước.

#### 13. STANDARDS-SCRIPT-FETCH-PRIORITY-STARTUP-SLA
- **Category**: `performance-optimization`
- **Title**: HTMLScriptElement fetchPriority & Main Thread Execution Governance
- **Evidence Level**: WHATWG HTML Living Standard & MDN Web Docs
- **URL**: `https://developer.mozilla.org/en-US/docs/Web/API/HTMLScriptElement/fetchPriority`
- **Spec Anchor**: MDN Web Docs - HTMLScriptElement: fetchPriority property
- **Relevance**: Mini-app phụ thuộc lớn vào các bundle JavaScript module. Khai báo `fetchPriority` phù hợp trên entrypoints rút ngắn đáng kể Time to Interactive (TTI) và First Contentful Paint (FCP).
- **Normative Requirements**:
  1. Pipeline tải script của webview phải tôn trọng `fetchPriority` trên cả script cổ điển và script module (`<script type="module">`).
  2. Bundle bootstrap khởi động cốt lõi của mini-app phải được tải với `fetchpriority="high"` (hoặc preload) để thúc đẩy DOM hydration.
  3. Các script phụ trợ (chat SDK, cổng thanh toán tải có điều kiện, bundle tính năng tải chậm) phải chỉ định `fetchpriority="low"`.
  4. Script bất đồng bộ (async) không khai báo ưu tiên mặc định là thấp; lập trình viên được phép chủ động nâng lên `high` khi cần thực thi sớm.
  5. Trình xác thực đóng gói của store phải kiểm tra các bundle script của đối tác không tự ý tiêm script ưu tiên cao vào tài liệu chính mà không qua phê duyệt của platform.
- **Implementation Guidelines**: Nạp component UI thiết yếu bằng `<script type="module" src="app.js" fetchpriority="high">`; gán `low` cho các module tính năng nạp động qua import.
- **Anti-Patterns**: Đánh dấu nhiều bundle script kích thước lớn là `high`, gây nghẽn hàng đợi mạng và bỏ đói việc tải stylesheet CSS quan trọng.

#### 14. STANDARDS-FETCH-API-REQUEST-PRIORITY-SPEC
- **Category**: `performance-optimization`
- **Title**: Fetch API RequestInit priority Property & API Call Prioritization
- **Evidence Level**: WHATWG Fetch Living Standard & MDN Web Docs
- **URL**: `https://developer.mozilla.org/en-US/docs/Web/API/RequestInit`
- **Spec Anchor**: MDN Web Docs - RequestInit: priority property & WHATWG Fetch Request Priority
- **Relevance**: Mini-app đồng thời thực hiện nhiều cuộc gọi API (dữ liệu người dùng, danh mục hàng hóa, nhật ký phân tích, prefetch). Thuộc tính `priority` giúp ngăn lưu lượng nền làm chậm giao dịch của người dùng.
- **Normative Requirements**:
  1. Runtime JavaScript của mini-app phải hỗ trợ `RequestInit.priority` với các giá trị được phép: `'high'`, `'low'`, và `'auto'`.
  2. Yêu cầu API giao dịch trọng yếu (thanh toán, xác thực, gửi đơn hàng) phải được gửi đi với `priority: 'high'`.
  3. Nhật ký đo lường, giám sát phân tích và prefetch phỏng đoán phải được gửi với `priority: 'low'`.
  4. Khi bỏ qua `priority` hoặc để `'auto'`, container phải gán ưu tiên mặc định theo suy nghiệm chuẩn của đặc tả Fetch.
  5. API proxy gateway và Service Worker của super-app phải kiểm tra metadata ưu tiên và bảo toàn mức ưu tiên luồng qua các chặng microservice phía sau.
- **Implementation Guidelines**: Đóng gói hàm gọi API tập trung tự động gắn `priority` dựa trên phân loại endpoint (mutation vs telemetry); dùng `low` cho các thao tác prefetch khi hover link.
- **Anti-Patterns**: Gửi các bản tin telemetry hoặc cảm biến tần suất cao với `priority: 'high'`, làm chậm các phản hồi API nghiệp vụ trên mạng di động có độ trễ lớn.

#### 15. STANDARDS-FETCH-RESOURCE-PRIORITIZATION-PRACTICE
- **Category**: `performance-optimization`
- **Title**: End-to-End Resource Prioritization & Network Scheduling SLA Governance
- **Evidence Level**: MDN Web Docs & Web Performance Working Group Guidelines
- **URL**: `https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch`
- **Spec Anchor**: MDN Web Docs - Using Fetch: Prioritizing Requests and Network Governance
- **Relevance**: Cung cấp kiến trúc chuẩn toàn diện cho lập trình viên và đơn vị vận hành kho ứng dụng nhằm đạt hiệu năng tải dưới 1 giây trong môi trường mạng di động 3G/4G/5G chập chờn.
- **Normative Requirements**:
  1. Package mini-app phải thực thi ngân sách ưu tiên mạng nghiêm ngặt: tối đa 1 ảnh ưu tiên cao, 1 stylesheet ưu tiên cao và 1 script bootstrap ưu tiên cao trong lần tải đầu tiên.
  2. Thẻ `<link rel="preload">` phải khai báo thuộc tính `fetchpriority` nhất quán với thẻ phần tử thực tế trong DOM để tránh lỗi tải trùng lặp (double-download).
  3. Khi điều kiện mạng suy giảm xuống 2G/3G chậm (phát hiện qua NetworkInformation API), runtime phải tự động hạ các request `'auto'` xuống `'low'`, ngoại trừ giao dịch `'high'` tường minh.
  4. Tầng mạng của container phải hỗ trợ chuẩn RFC 9218 Extensible Prioritization Schemes (`urgency=u`, `incremental=i`) ánh xạ trực tiếp mức ưu tiên sang header mạng.
  5. Quy trình nộp duyệt kho ứng dụng phải chạy kiểm thử bóp băng thông tự động (4G 1.6Mbps / 150ms RTT) và xác nhận quá trình hiển thị trực quan ban đầu hoàn tất trong 1,200ms.
- **Implementation Guidelines**: Tài liệu hóa các mẫu nạp tài nguyên chuẩn trong SDK starter kit của super-app; tích hợp rule lint trong CLI để phát hiện xung đột cấu hình preload và fetchpriority.
- **Anti-Patterns**: Preload font chữ hoặc ảnh phụ trợ với `fetchpriority="high"`, chặn đứng việc tải stylesheet bố cục cốt lõi; bỏ qua sự thay đổi trạng thái mạng di động và tiếp tục đồng bộ dữ liệu nền nặng.

---

## 3. Kiến Trúc & Quy Chuẩn Kiểm Duyệt Store Bổ Sung

1. **Static Linting**:
   - Quét độ sâu CSS Nesting $\le 4$ cấp; từ chối các selector lồng không hợp lệ.
   - Kiểm tra thuộc tính `lang` trên tài liệu khi sử dụng `hyphens: auto` hoặc quy tắc ngắt dòng CJK.
   - Giới hạn số lượng tài nguyên gán `fetchpriority="high"` trong bundle ban đầu ($\le 2$ ảnh, $\le 1$ script).
2. **Dynamic Profiling**:
   - Đo thời gian Style Recalculation $\le 16$ms cho 95% tương tác.
   - Thử nghiệm bóp băng thông 4G (1.6 Mbps / 150ms RTT), đảm bảo LCP $\le 1.2$s.
   - Kiểm tra chống vỡ layout trên viewport 320px với chuỗi ký tự dài thông qua `overflow-wrap: anywhere`.
