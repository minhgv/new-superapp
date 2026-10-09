# Chuyên đề 113: WebSocketStream Backpressure, HTTP/1.1 RFC 9112 Framing & RFC 8879 TLS Certificate Compression

## 1. Tổng quan nghiên cứu Milestone 117

Milestone 117 tập trung vào 3 trụ cột kỹ thuật nền tảng của tầng mạng, cổng kết nối biên (edge gateway) và tối ưu hóa mật mã học cho super-app mini-app container:

1. **WebSocketStream & WHATWG Streams Sandboxing**: Thay thế giao diện WebSocket hướng sự kiện truyền thống bằng cơ chế stream tích hợp `ReadableStream` và `WritableStream`, kích hoạt cơ chế backpressure tự nhiên của tầng TCP window scaling, giải quyết triệt để rủi ro tràn bộ nhớ (Out-Of-Memory / OOM) và hỗ trợ DedicatedWorker offloading cho các ứng dụng thời gian thực (real-time chat, trading, cloud gaming).
2. **IETF RFC 9112 HTTP/1.1 Framing & Request Smuggling Defense**: Chuẩn hóa cú pháp thông điệp HTTP/1.1, quy tắc ưu tiên tuyệt đối của `Transfer-Encoding` so với `Content-Length`, kiểm tra chunk extensions, chuẩn hóa khoảng trắng tiêu đề (OWASP WSTG-INPV-15, CWE-444) để bảo vệ các bể kết nối dùng chung (keep-alive connection pooling) giữa edge gateway và backend microservices của mini-app.
3. **IETF RFC 8879 TLS Certificate Compression & Encrypted Client Hello (ECH)**: Triển khai TLS 1.3 `compress_certificate` (extension 27) với thuật toán Brotli/Zstandard giảm kích thước chuỗi chứng chỉ X.509 xuống <1KB, đưa quá trình bắt tay TLS về đúng 1-RTT trên mạng di động 4G/5G; kết hợp DNS HTTPS/SVCB (RFC 9460) và ECH mã hóa toàn bộ ClientHello (bao gồm SNI) nhằm bảo vệ quyền riêng tư người dùng trước sự giám sát siêu dữ liệu trên hạ tầng mạng công cộng.

---

## 2. Bảng tổng hợp phát hiện nghiên cứu (Findings Matrix)

| ID | Tiêu chuẩn / Đặc tả | Phân loại | URL Nguồn | Mức bằng chứng |
| :--- | :--- | :--- | :--- | :--- |
| `WEBSOCKETSTREAM-BACKPRESSURE-MDN` | MDN WebSocketStream API | Runtime & Governance | [developer.mozilla.org](https://developer.mozilla.org/en-US/docs/Web/API/WebSocketStream) | Chuẩn chính thức |
| `WEBSOCKETSTREAM-STREAMING-GUIDE-WEB-DEV` | web.dev WebSocketStream Guide | Runtime & Governance | [web.dev](https://web.dev/websocketstream/) | Chuẩn chính thức |
| `WEBSOCKETSTREAM-CHROME-CAPABILITIES-DEV-CHROME` | Chrome Dev WebSocketStream Capabilities | Runtime & Governance | [developer.chrome.com](https://developer.chrome.com/docs/capabilities/web-apis/websocketstream) | Chuẩn chính thức |
| `WEBSOCKETSTREAM-OPENED-PROMISE-MDN` | MDN WebSocketStream.opened | Runtime & Governance | [developer.mozilla.org](https://developer.mozilla.org/en-US/docs/Web/API/WebSocketStream/opened) | Chuẩn chính thức |
| `WEBSOCKETSTREAM-CLOSED-PROMISE-MDN` | MDN WebSocketStream.closed | Runtime & Governance | [developer.mozilla.org](https://developer.mozilla.org/en-US/docs/Web/API/WebSocketStream/closed) | Chuẩn chính thức |
| `RFC-9112-HTTP11-MESSAGE-FRAMING-RFC-EDITOR` | IETF RFC 9112 HTTP/1.1 | An ninh & Sandbox | [rfc-editor.org](https://www.rfc-editor.org/rfc/rfc9112) | Chuẩn chính thức |
| `RFC-9112-CHUNKED-TRANSFER-SECURITY-IETF-DATATRACKER` | IETF RFC 9112 Chunked Security | An ninh & Sandbox | [datatracker.ietf.org](https://datatracker.ietf.org/doc/html/rfc9112) | Chuẩn chính thức |
| `CWE-444-HTTP-REQUEST-SMUGGLING-MITRE` | MITRE CWE-444 Request Smuggling | An ninh & Sandbox | [cwe.mitre.org](https://cwe.mitre.org/data/definitions/444.html) | Chuẩn chính thức |
| `PORTSWIGGER-HTTP-REQUEST-SMUGGLING-RESEARCH` | PortSwigger HTTP Smuggling Research | An ninh & Sandbox | [portswigger.net](https://portswigger.net/web-security/request-smuggling) | Benchmark ngành |
| `OWASP-HTTP-SPLITTING-SMUGGLING-TESTING-GUIDE` | OWASP WSTG-INPV-15 Testing Guide | An ninh & Sandbox | [owasp.org](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/15-Testing_for_HTTP_Splitting_Smuggling) | Benchmark ngành |
| `RFC-8879-TLS-CERTIFICATE-COMPRESSION-RFC-EDITOR` | IETF RFC 8879 TLS Cert Compression | An ninh & Sandbox | [rfc-editor.org](https://www.rfc-editor.org/rfc/rfc8879) | Chuẩn chính thức |
| `RFC-8879-CERTIFICATE-COMPRESSION-IETF-DATATRACKER` | IETF RFC 8879 Decompression Limits | An ninh & Sandbox | [datatracker.ietf.org](https://datatracker.ietf.org/doc/html/rfc8879) | Chuẩn chính thức |
| `IANA-TLS-EXTENSION-VALUES-COMPRESSION-CERT` | IANA TLS ExtensionType Values Registry | An ninh & Sandbox | [iana.org](https://www.iana.org/assignments/tls-extensiontype-values/tls-extensiontype-values.xhtml) | Chuẩn chính thức |
| `RFC-9460-HTTPS-SVCB-DNS-RR-RFC-EDITOR` | IETF RFC 9460 HTTPS/SVCB DNS RR | An ninh & Sandbox | [rfc-editor.org](https://www.rfc-editor.org/rfc/rfc9460.html) | Chuẩn chính thức |
| `CLOUDFLARE-ENCRYPTED-CLIENT-HELLO-ECH-GUIDE` | Cloudflare Encrypted Client Hello Guide | An ninh & Sandbox | [blog.cloudflare.com](https://blog.cloudflare.com/encrypted-client-hello/) | Benchmark ngành |

---

## 3. Kiến trúc chi tiết & Khuyến nghị kỹ thuật

### 3.1. WebSocketStream & Luồng Điều khiển Backpressure trong Mini-App Container

Trong kiến trúc container WebView truyền thống, các mini-app sử dụng API `WebSocket` chuẩn thường đối mặt với vấn đề tràn đệm (unbounded buffer buildup):
- **Phía nhận (Ingress)**: Khi máy chủ gửi dữ liệu với tốc độ cao hơn tốc độ xử lý của giao diện người dùng, hàng đợi sự kiện `onmessage` tích lũy lượng lớn đối tượng `MessageEvent`, dẫn đến giật khung hình (frame drops) và cuối cùng bị hệ điều hành tiêu diệt vì vượt hạn mức bộ nhớ (OOM killer).
- **Phía phát (Egress)**: Khi ứng dụng gửi dữ liệu nhanh hơn tốc độ đường truyền mạng, thuộc tính `bufferedAmount` tăng không giới hạn mà không có cơ chế chặn luồng phát tự nhiên.

**WebSocketStream API** khắc phục triệt để bằng cách tích hợp với WHATWG Streams:
```javascript
// Khởi tạo WebSocketStream với hỗ trợ signal hủy và backpressure
const abortController = new AbortController();
const wss = new WebSocketStream("wss://gateway.superapp.vn/ws/v1/ticker", {
  protocols: ["superapp-telemetry-v1"],
  signal: abortController.signal
});

try {
  const { readable, writable, protocol, extensions } = await wss.opened;
  console.log(`Kết nối thành công với giao thức: ${protocol}`);

  // Đọc dữ liệu với backpressure tự nhiên
  const reader = readable.getReader();
  while (true) {
    const { value, done } = await reader.read();
    if (done) break;
    await processHeavyMarketData(value); // Xử lý xong mới kéo gói tiếp theo
  }
} catch (err) {
  console.error("Lỗi bắt tay hoặc đứt kết nối stream:", err);
} finally {
  const { code, reason } = await wss.closed;
  console.log(`Đóng stream: code=${code}, reason=${reason}`);
}
```

**Khuyến nghị Container**:
1. Đưa `WebSocketStream` vào danh mục API khuyến nghị cấp 1 (Tier-1) cho các mini-app chứng khoán, ngân hàng số và gaming.
2. Khuyến khích khởi tạo `WebSocketStream` bên trong `DedicatedWorker` để tách toàn bộ tác vụ giải nén, phân tích cú pháp JSON/Protobuf ra khỏi main UI thread, đảm bảo chỉ số Interaction to Next Paint (INP) luôn ≤ 200ms.

---

### 3.2. Chống HTTP Request Smuggling theo IETF RFC 9112 & CWE-444

Mô hình siêu ứng dụng tích hợp hàng trăm mini-app của bên thứ ba truy cập qua cụm API Gateway dùng chung. Rủi ro desynchronization giữa reverse proxy (CDN/WAF) và backend upstream server là mối đe dọa nghiêm trọng.

**Các quy tắc bắt buộc theo RFC 9112**:
1. **Xử lý xung đột độ dài thông điệp (Section 6.1 & 6.3)**: Bất kỳ yêu cầu nào chứa đồng thời cả `Transfer-Encoding` và `Content-Length` đều phải coi `Transfer-Encoding` là chuẩn duy nhất, hoặc gateway phải từ chối ngay lập tức bằng mã lỗi `HTTP 400 Bad Request`.
2. **Khoảng trắng trong tiêu đề (Section 5.1)**: Cấm tuyệt đối khoảng trắng giữa tên trường tiêu đề và dấu hai chấm (`FieldName : Value` -> Lỗi 400).
3. **Chuẩn hóa Chunked Parsing (Section 7.1)**: Kiểm tra giới hạn kích thước chunk-size để tránh tràn số nguyên (integer overflow); loại bỏ các chunk extensions không được định nghĩa trước; từ chối các thông điệp có ký tự xuống dòng dị thường (chỉ chấp nhận `CRLF
`, từ chối `LF` đơn lẻ).
4. **Cô lập Connection Pool**: Mỗi đối tác mini-app (phân định qua `X-MiniApp-ID`) phải có nhóm kết nối upstream riêng biệt hoặc chuyển đổi toàn bộ giao tiếp backend sang HTTP/2 hoặc HTTP/3 Multiplexing để loại bỏ hoàn toàn khả năng lồng ghép yêu cầu (pipelining desync).

---

### 3.3. Tối ưu TLS 1.3 với RFC 8879 Certificate Compression & ECH (RFC 9460)

Trên môi trường di động tại các thị trường Đông Nam Á (Việt Nam, Lào, Campuchia), độ trễ bắt tay TLS ảnh hưởng trực tiếp đến thời gian khởi động (cold-start) của mini-app.

1. **RFC 8879 TLS Certificate Compression**:
   - Khi chuỗi chứng chỉ X.509 vượt quá cửa sổ tắc nghẽn ban đầu (initcwnd ~ 14.6KB hoặc ~10 MSS), gói tin bắt tay bị phân mảnh và đòi hỏi thêm round-trip (RTT).
   - Bằng cách kích hoạt extension `compress_certificate` (type 27) với thuật toán Brotli hoặc Zstandard trên cổng API Gateway, chuỗi chứng chỉ được nén từ 3.5KB xuống còn ~1.1KB, đảm bảo quá trình bắt tay TLS 1.3 hoàn tất chính xác trong **1-RTT**.
   - Thiết lập ngưỡng giải nén tối đa tại client (64KB) để ngăn chặn tấn công từ chối dịch vụ dạng "bom giải nén" (decompression bomb).

2. **Bảo mật siêu dữ liệu với Encrypted Client Hello (ECH) & RFC 9460**:
   - Xuất bản bản ghi `HTTPS` trong DNS theo RFC 9460, thông báo cấu hình ECH (`ech=...`) và ALPN (`alpn="h3,h2"`).
   - Khi client mini-app phân giải tên miền, khóa công khai ECH được nạp đồng thời, cho phép mã hóa toàn bộ trường `Server Name Indication (SNI)`. Điều này ngăn ngừa các mạng Wi-Fi công cộng hoặc ISP theo dõi danh mục dịch vụ nhạy cảm mà người dùng mở trong siêu ứng dụng.

---

## 4. Kế hoạch kiểm thử & Tiêu chuẩn nghiệm thu Store Review

1. **Store Linter Rule SEC-NET-01**: Kiểm tra mã nguồn mini-app, cảnh báo khi sử dụng `new WebSocket()` cho các luồng dữ liệu nhị phân dung lượng cao mà không có cơ chế throttling; hướng dẫn chuyển đổi sang `WebSocketStream`.
2. **Gateway Quality Gate GW-RFC9112**: Bộ kiểm thử tự động của gateway phải giả lập các biến thể tấn công CL.TE, TE.CL, TE.TE (theo tài liệu PortSwigger & OWASP WSTG-INPV-15) và xác nhận 100% bị chặn với mã `400 Bad Request`.
3. **Network Performance Metric NET-TLS-01**: Giám sát tỷ lệ nén chứng chỉ TLS 1.3 đạt tối thiểu 60% mức tiết kiệm byte, đảm bảo độ trễ bắt tay mạng 4G đạt P95 < 120ms.
