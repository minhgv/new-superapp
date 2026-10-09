# Chuyên đề Iteration 98: W3C CSS Display Module Level 3, W3C CSS Generated Content & Lists Module Level 3, WHATWG URL Living Standard, W3C Media Fragments URI 1.0 & WHATWG DOM StaticRange / AbstractRange

## 1. Giới thiệu & Bối cảnh kỹ thuật

Trong kiến trúc Super App hiện đại lưu trữ hàng trăm mini app từ các đối tác thương mại, giải trí và thanh toán, container runtime phải giải quyết đồng thời ba thách thức cốt lõi về giao diện và điều hướng dữ liệu:
1. **Kiểm soát phân rã Box Model & Chống tràn BFC (Block Formatting Context)**: Các thành phần UI của mini app (như widget giỏ hàng, thẻ ưu đãi, huy hiệu đối tác) khi được nhúng trực tiếp vào container shell thường gây ra lỗi sụp đổ lề (margin collapse) hoặc phá vỡ cấu trúc CSS Flex/Grid của ứng dụng mẹ nếu không có cơ chế phân tách tường minh giữa cách hiển thị bên ngoài (`display-outside`) và mô hình định dạng bên trong (`display-inside`). Chuẩn **W3C CSS Display Module Level 3** cung cấp cú pháp đa từ khóa, `display: contents` và `display: flow-root` để cô lập BFC triệt để.
2. **Nội dung sinh động khai báo & Tiếp cận thông tin (Accessibility)**: Việc render các chỉ báo trạng thái, danh mục điều khoản, ký hiệu tiền tệ và trích dẫn bản địa hóa thường bị lạm dụng thông qua việc chèn hàng loạt node DOM rác bằng JavaScript, gây áp lực lên bộ nhớ webview. Chuẩn **W3C CSS Generated Content Level 3** (với cú pháp thay thế văn bản tiếp cận `content: url(...) / "Alt text"` và `quotes`) cùng **W3C CSS Lists and Counters Level 3** (pseudo-element `::marker` và `list-style-type`) cho phép sinh giao diện trực tiếp trên cây render tree với chi phí tài nguyên tối thiểu.
3. **Phân tích URL Xác định & Quản lý Phân vùng Lựa chọn Bất biến (StaticRange)**: Sự sai lệch giữa bộ phân tích URL của hệ điều hành bản địa (Android Intent/iOS URL) và engine webview tiềm ẩn nguy cơ bảo mật nghiêm trọng (SSRF, Open Redirect, Bypass Whitelist). Chuẩn **WHATWG URL Living Standard** (`URL.canParse()`, `URL.parse()`) loại bỏ hoàn toàn các lỗ hổng parser differential. Kết hợp với **W3C Media Fragments URI 1.0** (cắt video/audio trực tiếp qua URI hash mà không cần transcode server) và **WHATWG DOM StaticRange / AbstractRange** (quản lý vùng chọn văn bản bất biến không kích hoạt mutation observer), hệ thống đảm bảo luồng điều hướng và tương tác nội dung đạt độ mượt mà tuyệt đối.

---

## 2. Bảng tổng hợp các tiêu chuẩn & Findings nghiên cứu (Iteration 98)

| Mã Finding | Danh mục | Tiêu chuẩn / Công nghệ | URL Tài liệu Tham chiếu | Mức độ Bằng chứng | Ứng dụng Super App Container |
|---|---|---|---|---|---|
| `STANDARDS-CSS-DISPLAY-3-SPEC` | rendering-and-box-layout | W3C CSS Display Module Level 3 Specification | [W3C Recommendation](https://www.w3.org/TR/css-display-3/) | official-specification | Phân rã mô hình tạo box thành outer và inner formatting context, kiểm soát luồng hiển thị |
| `STANDARDS-CSS-DISPLAY-BOX-CONTENTS` | rendering-and-box-layout | CSS display: contents Box Deconstruction | [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/CSS/display-box) | official-specification | Bỏ qua box container trung gian, chiếu thẳng child elements vào CSS Grid/Flex của host |
| `STANDARDS-CSS-DISPLAY-INSIDE-FLOW-ROOT` | rendering-and-box-layout | CSS display-inside: flow-root BFC Containment | [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/CSS/display-inside) | official-specification | Tạo BFC độc lập tuyệt đối, ngăn chặn margin collapse và float bleeding sang container shell |
| `STANDARDS-CSS-DISPLAY-OUTSIDE-FLOW` | rendering-and-box-layout | CSS display-outside: Inline & Block Mediation | [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/CSS/display-outside) | official-specification | Quy định cách thức box tham gia vào luồng inline hoặc block bên ngoài host |
| `STANDARDS-CSS-CONTENT-3-SPEC` | generated-content-and-typography | W3C CSS Generated Content Module Level 3 | [W3C Candidate Recommendation](https://www.w3.org/TR/css-content-3/) | official-specification | Sinh nội dung trang trí khai báo không cần can thiệp hoặc làm phình to DOM tree |
| `STANDARDS-CSS-CONTENT-PROPERTY-SYNTAX` | generated-content-and-typography | CSS content Property with Accessible Alt Text | [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/CSS/content) | official-specification | Hỗ trợ cú pháp gạch chéo `/ "alt text"` cung cấp nhãn tiếp cận cho screen readers |
| `STANDARDS-CSS-QUOTES-PROPERTY` | generated-content-and-typography | CSS quotes Multilingual Typographic Localization | [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/CSS/quotes) | official-specification | Tự động áp dụng cặp dấu ngoặc kép phù hợp với chuẩn ngôn ngữ và quốc gia |
| `STANDARDS-CSS-LISTS-3-SPEC` | generated-content-and-typography | W3C CSS Lists and Counters Module Level 3 | [W3C Candidate Recommendation](https://www.w3.org/TR/css-lists-3/) | official-specification | Chuẩn hóa cấu trúc list-item formatting và marker box positioning |
| `STANDARDS-CSS-PSEUDO-MARKER` | generated-content-and-typography | CSS ::marker Pseudo-Element Sandboxing | [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/CSS/::marker) | official-specification | Tùy biến ký hiệu đầu dòng an toàn không gây reflow kích thước phần tử cha |
| `STANDARDS-CSS-LIST-STYLE-TYPE` | generated-content-and-typography | CSS list-style-type Typographic Governance | [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/CSS/list-style-type) | official-specification | Định dạng ký hiệu danh sách theo hệ số đếm bản địa (CJK, Lao, Thai, Latin) |
| `STANDARDS-URL-CANPARSE-STATIC` | url-parsing-and-navigation | URL.canParse() Static Fast Validation | [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/API/URL/canParse_static) | official-specification | Kiểm tra tính hợp lệ của URL đồng bộ không phát sinh ngoại lệ DOMException |
| `STANDARDS-URL-PARSE-STATIC` | url-parsing-and-navigation | URL.parse() Static Safe Construction | [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/API/URL/parse_static) | official-specification | Khởi tạo đối tượng URL an toàn trả về null khi lỗi thay vì văng TypeError |
| `STANDARDS-W3C-MEDIA-FRAGMENTS-SPEC` | media-and-navigation-sandboxing | W3C Media Fragments URI 1.0 Specification | [W3C Recommendation](https://www.w3.org/TR/media-frags/) | official-specification | Định vị và cắt video/audio trực tiếp theo thời gian/không gian qua URI hash |
| `STANDARDS-DOM-STATICRANGE-INTERFACE` | dom-traversal-and-range-sandboxing | WHATWG DOM StaticRange Interface | [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/API/StaticRange) | official-specification | Vùng chọn DOM bất biến nhẹ không đăng ký listener theo dõi biến động cây |
| `STANDARDS-DOM-ABSTRACTRANGE-INTERFACE` | dom-traversal-and-range-sandboxing | WHATWG DOM AbstractRange Base Interface | [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/API/AbstractRange) | official-specification | Giao diện cơ sở đa hình cho các bộ quét bảo mật và công cụ accessibility |

---

## 3. Phân tích Kỹ thuật Chuyên sâu

### 3.1. W3C CSS Display Module Level 3 & Cơ chế Cách ly Khung BFC

Trước chuẩn CSS Display Level 3, thuộc tính `display` bị gán ghép giữa cách phần tử hiển thị đối với cha của nó và cách các phần tử con sắp xếp bên trong. Chuẩn Level 3 tách bạch rõ ràng hai khía cạnh này:
```css
/* Cú pháp hiển thị đa từ khóa chuẩn hóa */
.mini-app-feed-card {
  display: block flow-root; /* Outer: block trong feed của Super App; Inner: BFC độc lập */
}

.mini-app-action-chip {
  display: inline flex;      /* Outer: inline nằm cạnh text; Inner: flex layout bên trong */
  align-items: center;
}
```
* **Lợi ích đối với Container Super App**:
  - `display: flow-root`: Loại bỏ hoàn toàn sự cố tràn float và margin collapsing. Các thẻ widget mini app không thể đẩy lề làm biến dạng header hoặc footer của ứng dụng mẹ.
  - `display: contents`: Cho phép loại bỏ các thẻ `div` bọc trung gian của các framework (React Fragment, Vue template wrapper) trên cây layout mà vẫn giữ nguyên ngữ cảnh ngữ nghĩa, giúp các thành phần con gắn kết liền mạch vào hệ thống CSS Grid 12 cột của Super App.

### 3.2. W3C CSS Generated Content & Danh sách Định dạng Bản địa hóa

Việc nhúng các chỉ báo thị giác (badge "Đã xác minh", icon "Giao hàng hỏa tốc", ký hiệu giảm giá) thường làm tăng số lượng DOM node nếu viết bằng HTML thẻ `<span>` hoặc `<img>`. W3C CSS Generated Content Level 3 cho phép đưa nội dung trực tiếp vào render tree thông qua CSS, kết hợp tính năng hỗ trợ công nghệ trợ năng:
```css
/* Chỉ báo trạng thái có kèm văn bản đọc cho người khiếm thị */
.verified-merchant::after {
  content: url("/assets/icons/check-badge.svg") / "Đối tác đã xác thực bởi Super App";
}

/* Bản địa hóa dấu trích dẫn theo chuẩn văn hóa từng quốc gia */
:lang(vi) { quotes: "“" "”" "‘" "’"; }
:lang(fr) { quotes: "« " " »" "‹ " " ›"; }
:lang(ja) { quotes: "「" "」" "『" "』"; }
```
Pseudo-element `::marker` của CSS Lists Level 3 cũng giới hạn nghiêm ngặt các thuộc tính được phép can thiệp (chỉ cho phép đổi `color`, `font`, `content`), ngăn chặn triệt để hành vi cố tình chèn script hoặc làm giật khung hình (layout shift) từ các mini app bên thứ ba.

### 3.3. WHATWG URL Standard & Tối ưu hóa Đoạn Media Fragments

#### 3.3.1. Chống Lỗ hổng Parser Differential qua `URL.canParse()` & `URL.parse()`
Sự khác biệt trong việc chuẩn hóa URL giữa tầng Native Java/Kotlin (Android), Swift (iOS) và JavaScript V8 Webview là nguyên nhân phổ biến dẫn đến các cuộc tấn công SSRF hoặc vượt qua danh sách trắng tên miền (Domain Whitelist Bypass). Bằng cách áp dụng WHATWG URL Living Standard:
```javascript
// Kiểm tra trước khi thực thi lệnh điều hướng Native Bridge
if (URL.canParse(targetUrl, baseOrigin)) {
  const parsed = URL.parse(targetUrl, baseOrigin);
  if (allowedDomains.has(parsed.hostname)) {
    superAppBridge.dispatchNavigation(parsed.href);
  } else {
    superAppBridge.showSecurityWarning("Tên miền ngoài danh sách cho phép");
  }
} else {
  console.error("URL không hợp lệ:", targetUrl);
}
```
Phương thức `URL.canParse()` thực thi kiểm tra đồng bộ trong vòng micro-giây mà không ném lỗi `DOMException`, giúp vòng lặp xử lý deep link đạt hiệu năng tối đa.

#### 3.3.2. W3C Media Fragments URI 1.0 & WHATWG DOM StaticRange
* **Media Fragments**: Cú pháp `https://cdn.superapp.vn/intro.mp4#t=5,15` kích hoạt cơ chế HTTP Byte-Range (RFC 9110), chỉ tải đúng đoạn dữ liệu từ giây thứ 5 đến giây thứ 15, tiết kiệm hơn 80% băng thông tải video giới thiệu cho người dùng kết nối mạng 3G/4G chập chờn.
* **StaticRange**: Thay thế đối tượng `Range` truyền thống vốn luôn duy trì kết nối lắng nghe biến động cây DOM (DOM Mutation Listeners). `StaticRange` tạo ra một ảnh chụp tọa độ ký tự bất biến, phục vụ tính năng sao chép mã khuyến mãi hoặc tô sáng từ khóa tìm kiếm mà không gây giật lag trình duyệt.

---

## 4. Bộ Tiêu chí Kiểm duyệt Mini App Store (Store Review Rules)

1. **Quy tắc Cách ly Khung BFC (Review Code: `REV-CSS-BFC-01`)**:
   - Tất cả các widget thẻ nhúng hoặc banner quảng cáo mini app phải khai báo `display: flow-root` hoặc `display: block flow-root` tại container gốc để bảo vệ giao diện ứng dụng mẹ khỏi lỗi sụp lề.
2. **Quy tắc Tiếp cận Nội dung CSS Generated (Review Code: `REV-A11Y-GEN-01`)**:
   - Khi mini app sử dụng thuộc tính `content` để chèn icon hoặc biểu tượng thị giác mang thông tin quan trọng, bắt buộc phải khai báo nhãn văn bản thay thế bằng cú pháp `/ "Accessible description"` hoặc cung cấp văn bản dự phòng tương đương trên cây DOM.
3. **Quy tắc Chuẩn hóa URL Đầu vào (Review Code: `REV-NAV-URL-01`)**:
   - Mọi URL điều hướng deep link, endpoint gọi API hoặc liên kết chuyển tiếp xác thực OAuth đều phải vượt qua bộ kiểm tra `URL.canParse()` theo chuẩn WHATWG URL Standard. Các URL chứa ký tự điều khiển bất thường hoặc port bị cấm sẽ bị từ chối phê duyệt.
4. **Quy tắc Sử dụng StaticRange trong Xử lý Văn bản (Review Code: `REV-DOM-RANGE-01`)**:
   - Các tính năng tìm kiếm trong trang, highlight từ khóa và trích xuất nội dung văn bản tần suất cao phải sử dụng `StaticRange` thay vì `Range` trực tiếp để tránh chiếm dụng chu kỳ CPU của main thread.
