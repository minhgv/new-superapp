# Chuyên đề Iteration 55: W3C WebDriver BiDi, IETF RFC 9700 OAuth Security BCP & W3C File API Binary Sandboxing

## 1. Tổng quan điều hành (Executive Summary)

Iteration 55 mở rộng bộ chuẩn kỹ thuật Mini App Store cho Super-App sang 3 lĩnh vực then chốt: kiểm thử tự động hai chiều qua WebDriver BiDi, an toàn định danh OAuth 2.0 theo Best Current Practice mới nhất (RFC 9700 / RFC 8252), và cách ly nhị phân/quản trị bộ nhớ File API (W3C File API):

1. **W3C WebDriver BiDi Architecture & Automated Store Review**:
   - Giao thức chuẩn hóa hai chiều bất đồng bộ qua WebSocket, thay thế mô hình HTTP request-response đơn hướng và cơ chế polling chậm chạp của WebDriver cổ điển.
   - Cơ chế đánh giá mã nguồn trong không gian cách ly (`script.evaluate`, `script.callFunction` với `userContext` / isolated realm), ngăn chặn prototype pollution hoặc các mã độc mini-app dò quét scanner kiểm duyệt.
   - Mô-đun `network` cung cấp các điểm chặn vòng đời mạng (`beforeRequestSent`, `responseStarted`, `authRequired`), cho phép can thiệp lưu lượng, phản hồi giả lập (mock responses), và kiểm tra tuân thủ chính sách CSP egress mà không cần proxy ngoài.
   - Mô-đun `log` phát trực tiếp (`log.entryAdded`) các ngoại lệ không bắt được (unhandled exceptions), cảnh báo deprecation và nhật ký console với stack trace chi tiết, phục vụ điều tra nguyên nhân lỗi tự động.
   - Mô-đun `permissions` mô phỏng trạng thái cấp quyền (`permissions.setPermission`) và kích thước thiết bị, hỗ trợ kiểm thử ma trận âm/dương (positive/negative capability tests) hoàn toàn tự động.

2. **IETF RFC 9700 OAuth 2.0 Security BCP & RFC 8252 Native App Boundaries**:
   - BCP 240 chính thức khai tử luồng Implicit Grant (`response_type=token`) do nguy cơ rò rỉ access token qua lịch sử duyệt web và tiêu đề Referer.
   - Bắt buộc áp dụng Authorization Code Grant đi kèm Proof Key for Code Exchange (PKCE, RFC 7636) với thuật toán mã hóa `code_challenge_method=S256` cho mọi loại client (kể cả public SPA và mini-apps).
   - Ngăn chặn tấn công Mix-up trong môi trường đa nhà cung cấp định danh (Multi-IdP) bằng tham số nhận diện máy chủ ủy quyền `iss` (RFC 9207) và kiểm tra chuỗi khớp chính xác redirect URI.
   - Khuyến nghị bắt buộc ràng buộc người gửi token (Sender-Constrained Tokens) qua DPoP (RFC 9449) hoặc mTLS (RFC 8705); áp dụng xoay vòng Refresh Token một lần (Refresh Token Rotation) và thu hồi toàn bộ dòng họ token (family revocation) khi phát hiện token cũ bị gửi lại.
   - Tuân thủ RFC 8252 (BCP 212): nghiêm cấm sử dụng WebView nhúng nội bộ để người dùng đăng nhập tài khoản bên thứ ba; bắt buộc điều hướng qua trình duyệt hệ thống (Custom Tabs / ASWebAuthenticationSession) nhằm bảo vệ cookie và chống lộ mật khẩu cho host app.

3. **W3C File API: Binary Sandboxing, Ingestion & Memory Governance**:
   - Định nghĩa các giao diện dữ liệu nhị phân bất biến `Blob` và `File`, chuẩn hóa việc làm sạch kiểu MIME (MIME-type sanitization) ngăn chặn tấn công MIME-sniffing.
   - Đọc dữ liệu nhị phân bất đồng bộ qua `FileReader` (`readAsArrayBuffer`, `readAsText`) với các sự kiện tiến trình (`progress`, `loadend`), cảnh báo nguy cơ cạn kiệt bộ nhớ (OOM) khi sử dụng `readAsDataURL` với tệp lớn.
   - Quản trị vòng đời Blob URL (`URL.createObjectURL` và bắt buộc thu hồi bộ nhớ bằng `URL.revokeObjectURL`), loại bỏ rò rỉ bộ nhớ (memory leaks) làm sập tiến trình WebView trên thiết bị di động.
   - Hỗ trợ cắt nhỏ dữ liệu nhị phân không sao chép (`Blob.slice()`) phục vụ tải lên phân mảnh có khả năng phục hồi (resilient resumable chunked uploads) với chi phí bộ nhớ tối thiểu.
   - Bảo mật hệ thống tệp: che giấu đường dẫn tệp thực tế trên hệ điều hành cục bộ (masking path metadata), cách ly Blob URL theo đúng nguồn gốc (origin isolation), ngăn chặn mini-app khác đánh cắp tài nguyên tệp.

---

## 2. Bảng đối soát tiêu chuẩn & mức độ bằng chứng (Evidence Matrix)

| ID Finding | Chủ đề kỹ thuật | Tiêu chuẩn tham chiếu | Mức độ bằng chứng | URLs kiểm tra thực tế (HTTP 200) |
|---|---|---|---|---|
| `bidi_055_01` | WebDriver BiDi Protocol Architecture | W3C WebDriver BiDi §1–2 | `official_standard` | [W3C WebDriver BiDi](https://w3c.github.io/webdriver-bidi/), [MDN Commands](https://developer.mozilla.org/en-US/docs/Web/WebDriver/Commands) |
| `bidi_055_02` | Isolated Realm Script Evaluation | W3C WebDriver BiDi §5–6 | `official_standard` | [W3C WebDriver BiDi](https://w3c.github.io/webdriver-bidi/), [MDN Commands](https://developer.mozilla.org/en-US/docs/Web/WebDriver/Commands) |
| `bidi_055_03` | BiDi Network Interception & Mocking | W3C WebDriver BiDi §7 | `official_standard` | [W3C WebDriver BiDi](https://w3c.github.io/webdriver-bidi/), [MDN Commands](https://developer.mozilla.org/en-US/docs/Web/WebDriver/Commands) |
| `bidi_055_04` | BiDi Log Module & Diagnostic Streaming | W3C WebDriver BiDi §8 | `official_standard` | [W3C WebDriver BiDi](https://w3c.github.io/webdriver-bidi/), [MDN Commands](https://developer.mozilla.org/en-US/docs/Web/WebDriver/Commands) |
| `bidi_055_05` | BiDi Permissions & Device Simulation | W3C WebDriver BiDi §9 | `official_standard` | [W3C WebDriver BiDi](https://w3c.github.io/webdriver-bidi/), [MDN Capabilities](https://developer.mozilla.org/en-US/docs/Web/WebDriver/Capabilities) |
| `oauth_055_01` | RFC 9700 OAuth BCP & PKCE Everywhere | IETF RFC 9700 (BCP 240) §2.1 | `official_standard` | [IETF RFC 9700](https://datatracker.ietf.org/doc/html/rfc9700), [RFC 8252](https://datatracker.ietf.org/doc/html/rfc8252) |
| `oauth_055_02` | RFC 9700 Mix-Up & Exact URI Matching | IETF RFC 9700 §4.4 / RFC 9207 | `official_standard` | [IETF RFC 9700](https://datatracker.ietf.org/doc/html/rfc9700), [RFC 8252](https://datatracker.ietf.org/doc/html/rfc8252) |
| `oauth_055_03` | Sender-Constrained Tokens & Refresh Rotation | IETF RFC 9700 §2.2, §2.4 / RFC 9449 | `official_standard` | [IETF RFC 9700](https://datatracker.ietf.org/doc/html/rfc9700), [RFC 9449](https://datatracker.ietf.org/doc/html/rfc9449) |
| `oauth_055_04` | CSRF Mitigation & Cryptographic State | IETF RFC 9700 §4.7 | `official_standard` | [IETF RFC 9700](https://datatracker.ietf.org/doc/html/rfc9700), [RFC 8252](https://datatracker.ietf.org/doc/html/rfc8252) |
| `oauth_055_05` | RFC 8252 External User-Agent Delegation | IETF RFC 8252 (BCP 212) §4.1, §8.12 | `official_standard` | [IETF RFC 8252](https://datatracker.ietf.org/doc/html/rfc8252), [RFC 9700](https://datatracker.ietf.org/doc/html/rfc9700) |
| `fileapi_055_01` | W3C Blob & File Interface Sandboxing | W3C File API §3–4 | `official_standard` | [W3C File API](https://w3c.github.io/FileAPI/), [MDN File](https://developer.mozilla.org/en-US/docs/Web/API/File) |
| `fileapi_055_02` | FileReader Asynchronous Ingestion & OOM | W3C File API §6 | `official_standard` | [W3C File API](https://w3c.github.io/FileAPI/), [MDN FileReader](https://developer.mozilla.org/en-US/docs/Web/API/FileReader) |
| `fileapi_055_03` | URL.createObjectURL & Deterministic Revocation | W3C File API §8 | `official_standard` | [W3C File API](https://w3c.github.io/FileAPI/), [MDN createObjectURL](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL_static) |
| `fileapi_055_04` | Blob.slice() Zero-Copy Resilient Chunking | W3C File API §3.2 | `official_standard` | [W3C File API](https://w3c.github.io/FileAPI/), [MDN Blob.slice](https://developer.mozilla.org/en-US/docs/Web/API/Blob/slice) |
| `fileapi_055_05` | File API Path Masking & Origin Isolation | W3C File API §10 | `official_standard` | [W3C File API](https://w3c.github.io/FileAPI/), [MDN File](https://developer.mozilla.org/en-US/docs/Web/API/File) |

---

## 3. Hướng dẫn thiết kế kiến trúc & Triển khai cho Super-App

### 3.1. Hệ thống kiểm duyệt tự động dựa trên WebDriver BiDi

Để đảm bảo quy trình phê duyệt mini-app trên Store diễn ra tự động, độc lập và bảo mật, hệ thống CI/CD cần tích hợp kiến trúc kiểm thử WebDriver BiDi:

```
+-----------------------------------------------------------------------+
|                Store Automated Review Orchestrator                     |
+-----------------------------------------------------------------------+
        |                                                 ^
  WebSocket JSON-RPC                               Real-time Events
  (Commands: script, network, log)            (log.entryAdded, network.*)
        v                                                 |
+-----------------------------------------------------------------------+
|                    Headless WebView Runtime Container                 |
|  +-------------------------+      +--------------------------------+  |
|  | Isolated Audit Realm    |      | Mini-App Context (Untrusted)   |  |
|  | - Security Probes       | <--> | - DOM Tree / Layout            |  |
|  | - DOM Inspection Hooks  | (DOM)| - Application Logic            |  |
|  +-------------------------+      +--------------------------------+  |
|  +-----------------------------------------------------------------+  |
|  | Network Interception (beforeRequestSent, provideResponse)       |  |
|  +-----------------------------------------------------------------+  |
|  | Permissions Simulation (permissions.setPermission: denied/allow)|  |
+-----------------------------------------------------------------------+
```

```javascript
// Ví dụ: Kịch bản kiểm thử bảo mật tự động của Store qua WebSocket BiDi Client
async function runMiniAppSecurityAudit(bidiSession, miniAppUrl) {
  // 1. Đăng ký lắng nghe log và ngoại lệ thời gian thực
  await bidiSession.send("session.subscribe", {
    events: ["log.entryAdded", "network.beforeRequestSent", "network.responseCompleted"]
  });

  // 2. Chặn lưu lượng mạng để kiểm tra vi phạm egress CSP
  await bidiSession.send("network.addIntercept", {
    phases: ["beforeRequestSent"],
    urlPatterns: [{ type: "pattern", protocol: "https" }]
  });

  bidiSession.on("network.beforeRequestSent", (event) => {
    const destinationUrl = new URL(event.request.url);
    if (!isApprovedDomain(destinationUrl.hostname)) {
      console.error(`[STORE SECURITY GATE] Mini-app gửi yêu cầu ra ngoài domain cho phép: ${destinationUrl.href}`);
      // Từ chối request ngay lập tức
      bidiSession.send("network.failRequest", { request: event.request.request });
    }
  });

  // 3. Thực thi kiểm tra cấu trúc DOM trong Isolated Execution Realm
  const auditResult = await bidiSession.send("script.evaluate", {
    expression: "document.querySelectorAll('iframe').length",
    target: { context: bidiSession.currentContext, sandbox: "store_audit_sandbox" },
    awaitPromise: true
  });

  console.log("Số lượng iframe nhúng phát hiện trong mini-app:", auditResult.result.value);
}
```

### 3.2. Định danh OAuth 2.0 & Biên giới WebView (RFC 9700 & RFC 8252)

Khi mini-app tích hợp đăng nhập tài khoản nội bộ Super-App hoặc tài khoản doanh nghiệp (Enterprise SSO), hệ thống phải tuân thủ nghiêm ngặt BCP 240 và BCP 212:

```
[Mini-App Client]                  [Super-App Native Host]               [OAuth Authorization Server]
       |                                      |                                        |
       |-- 1. Yêu cầu đăng nhập (PKCE S256) ->|                                        |
       |      (code_challenge, state)         |-- 2. Mở Custom Tab / ASWebAuthSession ->|
       |                                      |      (Cấm nhúng WebView trực tiếp)     |
       |                                      |                                        |-- 3. Người dùng nhập mật khẩu/MFA
       |                                      |<-- 4. Trả Authorization Code (iss) ----|
       |                                      |      (Khớp chính xác Redirect URI)     |
       |<-- 5. Trả kết quả code & iss --------|                                        |
       |                                      |                                        |
       |-- 6. Đổi Token kèm DPoP Proof ----------------------------------------------->|
       |      (code_verifier, DPoP key)                                                |
       |<-- 7. Trả Access Token (DPoP-bound) & Refresh Token (Rotated) ----------------|
```

- **Cấm Implicit Grant**: Tuyệt đối không chấp nhận cấu hình `response_type=token` trong Developer Console.
- **Bắt buộc PKCE S256**: Mọi yêu cầu cấp mã xác thực phải có `code_challenge_method=S256`.
- **Bảo vệ Mix-up**: Máy chủ ủy quyền phải gửi tham số `iss` trong phản hồi theo RFC 9207.
- **Cách ly WebView**: Không cho phép mini-app tự nhúng form đăng nhập của bên thứ ba trong `iframe` hoặc WebView con. Bắt buộc chuyển giao cho trình duyệt hệ thống độc lập.

### 3.3. Xử lý tệp nhị phân & Quản trị bộ nhớ an toàn (W3C File API)

```javascript
// Triển khai module tải lên tệp an toàn, chống rò rỉ bộ nhớ trong Mini-App SDK
class ResilientFileUploader {
  constructor(file, chunkSize = 2 * 1024 * 1024) { // Cắt mảnh 2MB
    this.file = file;
    this.chunkSize = chunkSize;
    this.totalChunks = Math.ceil(file.size / chunkSize);
  }

  // Cắt tệp nhị phân không sao chép dữ liệu bộ nhớ (zero-copy slice)
  async uploadInChunks(uploadEndpoint, onProgress) {
    for (let index = 0; index < this.totalChunks; index++) {
      const start = index * this.chunkSize;
      const end = Math.min(start + this.chunkSize, this.file.size);

      // Blob.slice() chỉ tạo con trỏ vùng nhớ, không sao chép mảng byte
      const chunkBlob = this.file.slice(start, end, this.file.type);

      // Đọc phân mảnh dưới dạng ArrayBuffer bất đồng bộ
      const buffer = await this.readChunkAsBuffer(chunkBlob);

      // Gửi phân mảnh kèm byte-range headers
      await fetch(uploadEndpoint, {
        method: "PUT",
        headers: {
          "Content-Range": `bytes ${start}-${end - 1}/${this.file.size}`,
          "Content-Type": "application/octet-stream"
        },
        body: buffer
      });

      if (onProgress) {
        onProgress({ uploadedBytes: end, totalBytes: this.file.size, percent: Math.round((end / this.file.size) * 100) });
      }
    }
  }

  readChunkAsBuffer(blob) {
    return new Promise((resolve, reject) => {
      const reader = new FileReader();
      reader.onload = () => resolve(reader.result);
      reader.onerror = () => reject(reader.error);
      reader.readAsArrayBuffer(blob);
    });
  }

  // Quản lý Blob URL an toàn: Tự động thu hồi bộ nhớ sau khi hiển thị xem trước
  static createTemporaryPreview(imageElement, fileBlob) {
    const objectUrl = URL.createObjectURL(fileBlob);
    imageElement.src = objectUrl;

    // Giải phóng bộ nhớ ngay khi hình ảnh đã render xong
    imageElement.onload = () => {
      URL.revokeObjectURL(objectUrl);
      console.log(`Đã giải phóng Blob URL: ${objectUrl}`);
    };
  }
}
```

---

## 4. Danh mục Tiêu chí Đánh giá & Cổng Kiểm duyệt Store (Store Gating Rules)

1. **Gate 55.1: WebDriver BiDi Automated Negative Test**:
   - Mọi bản nộp mini-app phải vượt qua kịch bản kiểm thử BiDi trong chế độ từ chối toàn bộ quyền (`permissions.setPermission: denied`). Nếu ứng dụng bị crash hoặc màn hình trắng không có thông báo cho người dùng, hệ thống lập tức chấm điểm Fail.
2. **Gate 55.2: OAuth 2.0 RFC 9700 & RFC 8252 Compliance**:
   - Kiểm tra mã nguồn tĩnh: Cấm tuyệt đối cấu hình `response_type=token` và cấm sử dụng form đăng nhập nội bộ WebView đối với các tên miền định danh bên ngoài.
   - Bắt buộc kiểm tra tham số `code_challenge_method=S256` tại API Gateway.
3. **Gate 55.3: Memory Leak & Blob URL Deallocation**:
   - Kiểm tra động: Scanner theo dõi số lượng Blob URL được tạo (`URL.createObjectURL`) và đối soát số lượng URL được thu hồi (`URL.revokeObjectURL`) khi chuyển đổi màn hình. Bất kỳ mini-app nào giữ lại Blob URL vượt quá 30 giây sau khi unmount component sẽ bị cảnh báo vi phạm rò rỉ tài nguyên.
