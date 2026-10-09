# Chuyên đề Iteration 108: Chuẩn Hóa DOM Parsing & Serialization, WHATWG XMLHttpRequest Lifecycle và W3C MathML Core Typography

## 1. Bối cảnh & Mục tiêu Kỹ thuật

Trong các kiến trúc container super-app hiện đại (như Alibaba WindVane, WeChat Mini Program, Baidu Smart Program, Viettel SuperApp), việc xử lý cây tài liệu động, thực thi truyền thông mạng kế thừa và hiển thị ký hiệu toán học khoa học là các thành phần quan trọng đòi hỏi sự phân lập bảo mật chặt chẽ:
1. **W3C DOM Parsing & Serialization**: Cung cấp các giao diện tiêu chuẩn `DOMParser`, `XMLSerializer`, cùng các thuật toán `innerHTML`, `outerHTML`, và `insertAdjacentHTML`. Container cần trung hòa hoàn toàn nguy cơ thực thi script ngoài ý muốn (script execution neutralization) và hỗ trợ tích hợp chính sách Trusted Types.
2. **WHATWG XMLHttpRequest Living Standard**: Định nghĩa máy trạng thái vòng đời kết nối mạng (`readyState` từ `UNSENT` đến `DONE`), truyền tải bộ đệm nhị phân không sao chép (`responseType: arraybuffer/blob`), cách ly thông tin xác thực (`withCredentials` CORS boundaries) và kiểm soát lưu lượng byte tải lên (`xhr.upload`).
3. **W3C MathML Core**: Khuyến nghị chuẩn hóa hiển thị toán học bản địa trên công cụ web hiện đại, loại bỏ các thư viện polyfill nặng nề (như MathJax, KaTeX), hỗ trợ cây ngữ nghĩa trợ năng (screen reader vocalization) và định dạng khoảng cách toán học theo bảng phông OpenType MATH.

---

## 2. Chi tiết Các Findings Nghiên cứu (Iteration 108)

### 2.1. Nhóm Chuẩn W3C DOM Parsing & Serialization

#### [STANDARDS-DOM-PARSING-DOMPARSER-SANDBOXING]
- **Tiêu đề**: W3C DOM Parsing: DOMParser Contextual XML & HTML Parsing and Script Execution Neutralization
- **Phân loại**: `runtime-environment-and-security-sandboxing`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://www.w3.org/TR/DOM-Parsing/ (HTTP 200)
- **Nội dung phân tích**:
  Chuẩn W3C DOM Parsing định nghĩa giao diện `DOMParser` với phương thức `parseFromString(str, type)` hỗ trợ các định dạng `text/html`, `application/xml`, `image/svg+xml`, và `text/xml`. Một bất biến bảo mật then chốt được quy định trong chuẩn là việc phân tích chuỗi HTML/XML qua `DOMParser` TUYỆT ĐỐI KHÔNG được thực thi các khối thẻ `<script>` nhúng hoặc kích hoạt tải tài nguyên mạng ngoài ngay lập tức. Trong container super-app, `DOMParser` hoạt động như một bộ giải tuần tự hóa an toàn trong bộ nhớ cho các gói XML, biểu tượng SVG và đoạn mẫu HTML trước khi đưa qua bộ lọc Sanitizer API, ngăn chặn triệt để lỗ hổng DOM XSS.

#### [STANDARDS-DOM-PARSING-XMLSERIALIZER-CANONICALIZATION]
- **Tiêu đề**: W3C DOM Serialization: XMLSerializer Tree-to-String Canonicalization and Attribute Scoping
- **Phân loại**: `runtime-environment-and-security-sandboxing`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://www.w3.org/TR/DOM-Parsing/ (HTTP 200)
- **Nội dung phân tích**:
  Đặc tả định nghĩa giao diện `XMLSerializer` và phương thức `serializeToString(root)`, cho phép chuyển đổi cây DOM ngược lại thành chuỗi XML/HTML hợp lệ. Thuật toán duyệt cây DOM, giải quyết namespace nhất quán, đóng mở dấu nháy thuộc tính và escape các ký tự đặc biệt (`&`, `<`, `>`, dấu ngoặc kép). Trong super-app, `XMLSerializer` đảm bảo việc chụp snapshot trạng thái cây giao diện, trích xuất đồ họa SVG để lưu trữ hoặc gửi qua native bridge diễn ra nhất quán và an toàn, ngăn chặn việc tiêm nhiễm thuộc tính độc hại qua ranh giới tuần tự hóa.

#### [STANDARDS-DOM-PARSING-INNERHTML-OUTERHTML-MUTATION]
- **Tiêu đề**: W3C DOM Parsing: Element.innerHTML & outerHTML Parsing Algorithms and Script Invalidation
- **Phân loại**: `runtime-environment-and-security-sandboxing`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://www.w3.org/TR/DOM-Parsing/ (HTTP 200)
- **Nội dung phân tích**:
  Quy chuẩn định rõ thuật toán phân tích đoạn mã HTML cho `Element.innerHTML`, `Element.outerHTML`, và `Element.insertAdjacentHTML`. Chuẩn HTML quy định các thẻ script tạo bởi innerHTML được đánh dấu là "parser-inserted" và KHÔNG được thực thi. Tuy nhiên, các trình xử lý sự kiện nội dòng (như `onload`, `onerror`) vẫn có thể kích hoạt nếu nội dung không được khử trùng. Container super-app bắt buộc chặn các thao tác gán `innerHTML`/`outerHTML` thông qua Trusted Types policies, yêu cầu mini app định tuyến qua các quy tắc TrustedHTML được phê duyệt.

#### [STANDARDS-DOM-PARSING-INSERTADJACENTHTML-POSITIONING]
- **Tiêu đề**: W3C DOM Parsing: insertAdjacentHTML Contextual Insertion Algorithms & Security Boundaries
- **Phân loại**: `runtime-environment-and-security-sandboxing`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://www.w3.org/TR/DOM-Parsing/ (HTTP 200)
- **Nội dung phân tích**:
  Phương thức `insertAdjacentHTML(position, text)` cho phép chèn đoạn mã đã phân tích vào vị trí chính xác so với phần tử đích thông qua 4 vị trí: `'beforebegin'`, `'afterbegin'`, `'beforeend'`, `'afterend'`. Nếu phần tử ngữ cảnh là `Document` hoặc `DocumentFragment`, việc gọi vị trí ngoài biên sẽ ném ngoại lệ `SyntaxError`. Trong kiến trúc micro-frontend của super-app, `insertAdjacentHTML` cung cấp cơ chế chèn DOM gia tăng với chi phí bộ nhớ thấp, không làm mất event listeners hay layout state của các node anh em, đồng thời tuân thủ kiểm duyệt đầu vào nghiêm ngặt.

#### [STANDARDS-DOM-PARSING-INNERTEXT-OUTERTEXT-AWARENESS]
- **Tiêu đề**: WHATWG/W3C DOM Parsing: Text Content Extraction, Layout-Aware innerText, and Node.textContent Hygiene
- **Phân loại**: `runtime-environment-and-security-sandboxing`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://www.w3.org/TR/DOM-Parsing/ (HTTP 200)
- **Nội dung phân tích**:
  Phân biệt rõ thuật toán trích xuất văn bản thô qua `Node.textContent` và trích xuất nhận biết layout hiển thị qua `Element.innerText`. Trong khi `textContent` đọc trực tiếp text nodes mà không kích hoạt tính toán CSS, `innerText` kích hoạt tính toán reflow đồng bộ để tôn trọng `display: none`, ngắt dòng và chuyển đổi văn bản. Trong hệ thống kiểm duyệt và lập chỉ mục mini app store tự động, công cụ phân tích sử dụng `textContent` để tránh phạt hiệu năng layout thrashing, và chỉ dùng `innerText` cho các bài kiểm tra trợ năng và trình đọc màn hình.

---

### 2.2. Nhóm Chuẩn WHATWG XMLHttpRequest Living Standard

#### [STANDARDS-WHATWG-XHR-LIFECYCLE-STATE-MACHINE]
- **Tiêu đề**: WHATWG XMLHttpRequest: ReadyState Transition Lifecycle, Event Gating, and Timeout Sandboxing
- **Phân loại**: `runtime-environment-and-network-governance`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://xhr.spec.whatwg.org/ (HTTP 200)
- **Nội dung phân tích**:
  WHATWG XMLHttpRequest chuẩn hóa máy trạng thái kết nối bất đồng bộ qua các bước: `UNSENT` (0), `OPENED` (1), `HEADERS_RECEIVED` (2), `LOADING` (3), và `DONE` (4), phát ra chuỗi sự kiện `ProgressEvent`. Thuộc tính `timeout` quy định giới hạn thời gian tối đa của yêu cầu, tự động hủy và bắn sự kiện `timeout` nếu máy chủ không hoàn tất truyền tải. Container super-app áp dụng trần timeout cưỡng chế (ví dụ tối đa 10 giây) và tự động hủy các kết nối XHR khi mini app chuyển sang trạng thái chạy nền (backgrounded), ngăn ngừa rò rỉ tài nguyên mạng và giữ mở socket vô tận.

#### [STANDARDS-WHATWG-XHR-RESPONSETYPE-BINARY-BUFFERS]
- **Tiêu đề**: WHATWG XMLHttpRequest: responseType Modes, ArrayBuffer Zero-Copy Streaming, and Blob Payloads
- **Phân loại**: `runtime-environment-and-network-governance`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://xhr.spec.whatwg.org/ (HTTP 200)
- **Nội dung phân tích**:
  Thuộc tính `responseType` hỗ trợ các chế độ `''`, `'text'`, `'arraybuffer'`, `'blob'`, `'document'`, và `'json'`. Khi đặt `'arraybuffer'` hoặc `'blob'`, luồng dữ liệu nhị phân từ máy chủ được nạp trực tiếp vào bộ đệm bộ nhớ gốc (zero-copy) mà không cần mã hóa chuỗi trung gian, tối ưu hóa cho WebAssembly, âm thanh hoặc cơ sở dữ liệu nhị phân. Container mini app thiết lập hạn mức bộ đệm tối đa để tránh lỗi Out-Of-Memory (OOM) trên WebView di động, đồng thời cấm triệt để XHR đồng bộ (`open` với `async=false`) trên main thread nhằm loại trừ hiện tượng đơ giao diện.

#### [STANDARDS-WHATWG-XHR-CREDENTIALS-AND-CORS-SECURITY]
- **Tiêu đề**: WHATWG XMLHttpRequest: withCredentials Flag, Cross-Origin Isolation, and Cookie Sandboxing
- **Phân loại**: `runtime-environment-and-network-governance`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://xhr.spec.whatwg.org/ (HTTP 200)
- **Nội dung phân tích**:
  Thuộc tính boolean `withCredentials` điều khiển việc đính kèm cookie, HTTP authentication và chứng chỉ TLS trong các yêu cầu cross-origin. Khi kích hoạt, CORS preflight bắt buộc máy chủ phản hồi với `Access-Control-Allow-Credentials: true` và một origin cụ thể. Container super-app cô lập ranh giới định danh: mini app bên thứ ba không thể bật cờ này để đánh cắp session cookies của super app host hoặc gửi lén thông tin xác thực đến các domain ngoài danh sách cho phép (allowlist) được khai báo trong manifest.

#### [STANDARDS-WHATWG-XHR-UPLOAD-PROGRESS-STREAMING]
- **Tiêu đề**: WHATWG XMLHttpRequest: XMLHttpRequestUpload Event Pipeline and Egress Flow Throttling
- **Phân loại**: `runtime-environment-and-network-governance`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://xhr.spec.whatwg.org/ (HTTP 200)
- **Nội dung phân tích**:
  Giao diện `XMLHttpRequestUpload` thông qua `xhr.upload` cung cấp các sự kiện đo lường luồng byte tải lên (`progress`, `load`, `error`, `abort`). Container mini app giám sát luồng sự kiện này để thực thi chính sách điều tiết dữ liệu ra (egress data governance): phát hiện các hành vi âm thầm upload lượng lớn dữ liệu nhật ký, ảnh chụp màn hình hoặc media trái phép, áp dụng rate limiting và điều tiết băng thông khi người dùng đang kết nối qua mạng di động trả phí.

#### [STANDARDS-WHATWG-XHR-FORMDATA-MULTIPART-SERIALIZATION]
- **Tiêu đề**: WHATWG XMLHttpRequest: FormData Multipart Form-Data Encoding and File Blob Appending
- **Phân loại**: `runtime-environment-and-network-governance`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://xhr.spec.whatwg.org/ (HTTP 200)
- **Nội dung phân tích**:
  Chuẩn FormData tích hợp trong XHR cho phép tạo lập có cấu trúc các cặp key-value cho định dạng `multipart/form-data`, hỗ trợ nạp chuỗi văn bản và các đối tượng `Blob`/`File`. Trình duyệt tự động tạo chuỗi phân cách ranh giới (cryptographic boundary) và gán header Content-Type phù hợp. Container super-app kiểm soát việc gắn tệp vào FormData để ngăn chặn mini app trỏ vào các đường dẫn tệp nhạy cảm trong sandbox cục bộ, đảm bảo tuân thủ tiêu chuẩn an toàn dữ liệu người dùng.

---

### 2.3. Nhóm Chuẩn W3C MathML Core Typography & Accessibility

#### [STANDARDS-W3C-MATHML-CORE-TREE-ARCHITECTURE]
- **Tiêu đề**: W3C MathML Core: MathML DOM Tree Architecture, Native Typography, and Mathematical Layout
- **Phân loại**: `runtime-environment-and-accessibility-standards`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://www.w3.org/TR/mathml-core/ (HTTP 200)
- **Nội dung phân tích**:
  Chuẩn W3C MathML Core tích hợp trực tiếp mô hình toán học vào DOM của công cụ dựng web hiện đại thông qua các thẻ nguyên tử (<mi> ký hiệu, <mn> số, <mo> toán tử) và các thẻ bố cục (<mrow>, <msqrt>, <mfrac>). Bố cục toán học được tính toán trực tiếp bởi bảng OpenType MATH của phông chữ hệ thống, loại bỏ sự phụ thuộc vào các gói JS nặng hàng megabyte của MathJax hay KaTeX. Đối với các mini app giáo dục, tài chính và khoa học trong super app, MathML Core đảm bảo tốc độ khởi động tức thì và khả năng hiển thị toán học siêu sắc nét với chi phí bộ nhớ tối thiểu.

#### [STANDARDS-W3C-MATHML-CORE-ACCESSIBILITY-TREE]
- **Tiêu đề**: W3C MathML Core: Accessibility Tree Semantics, Screen Reader Voicing, and Speech Rule Engines
- **Phân loại**: `runtime-environment-and-accessibility-standards`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://www.w3.org/TR/mathml-core/ (HTTP 200)
- **Nội dung phân tích**:
  MathML Core ánh xạ các phần tử toán học thành cấu trúc cây trợ năng tiêu chuẩn của hệ điều hành (UI Automation, NSAccessibility, AT-SPI). Trình đọc màn hình có thể phát âm chuẩn xác cấu trúc công thức (duyệt biểu thức con, số mũ, phân số) thay vì chỉ đọc các ký tự rời rạc hoặc hình ảnh phẳng với alt text sơ sài. Conformance gate của super app bắt buộc các dịch vụ công, giáo dục và tài chính sử dụng MathML Core cho các biểu thức toán học, đáp ứng yêu cầu tuân thủ chuẩn WCAG 2.2 Level AA.

#### [STANDARDS-W3C-MATHML-CORE-OPERATOR-DICTIONARY-SPACING]
- **Tiêu đề**: W3C MathML Core: Operator Dictionary Spacing, Form Invariants, and Stretchable Glyphs
- **Phân loại**: `runtime-environment-and-accessibility-standards`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://www.w3.org/TR/mathml-core/ (HTTP 200)
- **Nội dung phân tích**:
  Thành phần cốt lõi của MathML Core là từ điển toán tử chuẩn hóa quy tắc giãn cách mặc định (`lspace`, `rspace`) và xác định dạng toán tử (prefix, infix, postfix) dựa trên vị trí xung quanh. Chuẩn hóa thuật toán co giãn ký tự (stretchable glyphs như dấu ngoặc đơn, căn bậc hai) tự động bao bọc nội dung con thông qua bảng OpenType MathVariants. Nhờ đó, giao diện mini app duy trì độ ổn định bố cục tuyệt đối, loại bỏ hiện tượng giật layout (CLS) và duy trì cuộn mượt mà 60fps.

#### [STANDARDS-W3C-MATHML-CORE-CSS-STYLING-INTEGRATION]
- **Tiêu đề**: W3C MathML Core: CSS Math Styling Integration, math-style, and math-depth Properties
- **Phân loại**: `runtime-environment-and-accessibility-standards`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://www.w3.org/TR/mathml-core/ (HTTP 200)
- **Nội dung phân tích**:
  Quy chuẩn định rõ sự tích hợp giữa MathML và CSS thông qua các thuộc tính như `math-style` (`normal` vs `compact`) và `math-depth`. Đặt `math-style: compact` giúp tự động co nhỏ cỡ chữ và khoảng trống dọc trong các chỉ số dưới, chỉ số trên và tử số phân số. Thuộc tính `math-depth` tự động tăng trong các cấu trúc lồng nhau, điều khiển tỷ lệ phông chữ qua kế thừa CSS `font-size`. Nhờ đó, các widget toán học có thể đóng gói trong Shadow DOM và biến đổi theo chế độ dark mode mà không làm hỏng căn chỉnh không gian toán học.

#### [STANDARDS-W3C-MATHML-CORE-SECURITY-XML-SANDBOXING]
- **Tiêu đề**: W3C MathML Core: XML Namespace Scoping, Disallowed Elements, and XSS Sanitization Policies
- **Phân loại**: `runtime-environment-and-security-sandboxing`
- **Mức độ bằng chứng**: `official_standard`
- **Nguồn xác thực**: https://www.w3.org/TR/mathml-core/ (HTTP 200)
- **Nội dung phân tích**:
  MathML Core giới hạn nghiêm ngặt tập hợp phần tử hợp lệ trong namespace MathML (`http://www.w3.org/1998/Math/MathML`), chủ động loại bỏ các thẻ cũ tiềm ẩn lỗ hổng bảo mật trong lịch sử (như `<maction>`, `<semantics>`). Các thẻ MathML Core không cho phép nhúng script thực thi hoặc gán event handler trực tiếp. Các bộ lọc DOM Sanitizer của super app dễ dàng kiểm tra tính hợp lệ của MathML, loại bỏ hoàn toàn các vector tấn công XSS trong khi vẫn hiển thị hoàn hảo các công thức khoa học.

---

## 3. Tổng kết & Tích hợp Tiêu chuẩn

Qua Iteration 108, hệ thống tiêu chuẩn kỹ thuật của Super App Mini App Store đã được bổ sung 15 phát hiện then chốt:
- **Tổng số phát hiện lũy kế**: 1.571 findings.
- **Tính toàn vẹn**: 100% URL được xác thực HTTP 200 từ các tổ chức tiêu chuẩn W3C và WHATWG.
- **Đóng góp kiến trúc**: Cung cấp khung chuẩn hóa toàn diện cho việc phân tích DOM trong bộ nhớ, quản trị mạng kế thừa qua XHR và hiển thị toán học bản địa trợ năng qua MathML Core.
