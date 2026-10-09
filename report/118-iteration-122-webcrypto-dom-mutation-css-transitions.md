# Chuyên đề 118 (Iteration 122): Web Cryptography Randomness, DOM Mutation & CSS Transitions Sandboxing

## 1. Tổng quan nghiên cứu Milestone 122

Milestone 122 tập trung giải quyết ba trụ cột kiến trúc then chốt trong vận hành runtime của mini-app trong super-app hiện đại:
1. **Web Cryptography API & Cryptographic Randomness Sandboxing**: Chuẩn hóa việc phát sinh số giả ngẫu nhiên an toàn mật mã (`Crypto.getRandomValues`), định danh UUID phiên bản 4 chống va chạm (`Crypto.randomUUID`), và các thuật toán băm/mã hóa/ký số bất đồng bộ ngoài luồng chính (`SubtleCrypto.digest`, `SubtleCrypto.encrypt`, `SubtleCrypto.sign`).
2. **WHATWG DOM Node Mutation & Hierarchy Containment Sandboxing**: Chuẩn hóa cơ chế thay thế node con nguyên tử không kích hoạt HTML parser (`Element.replaceChildren`), nhân bản cây DOM có kiểm soát biên sự kiện (`Node.cloneNode`), kiểm tra tính bao hàm biên (`Node.contains`), định tuyến sự kiện phân tán thu hẹp (`Element.closest`), và kiểm toán trật tự vị trí DOM chống clickjacking (`Node.compareDocumentPosition`).
3. **W3C CSS Transitions Level 1 & State Interpolation Sandboxing**: Chuẩn hóa việc giới hạn thuộc tính chuyển đổi (`transition-property`), trần thời gian ngăn tiêu hao pin (`transition-duration`), đường cong gia tốc vật lý tự nhiên (`transition-timing-function`), cùng cơ chế đồng bộ vòng đời kết thúc (`transitionend`) và hủy bỏ do ngắt quãng cử chỉ (`transitioncancel`).

---

## 2. Chi tiết kỹ thuật & Tiêu chuẩn chuẩn hóa

### 2.1 Web Cryptography API & Cryptographic Randomness Sandboxing

- **Crypto.getRandomValues()**:
  - Chuẩn hóa việc nạp các giá trị ngẫu nhiên an toàn mật mã vào các mảng có kiểu (`TypedArray` như `Uint8Array`, `Int32Array`).
  - Thực thi hạn mức nghiêm ngặt: giới hạn tối đa 65.536 byte (64 KiB) trong mỗi lần gọi; vượt quá sẽ ném ngoại lệ `QuotaExceededError`.
  - Khai thác trực tiếp pool entropy từ hệ điều hành máy trạm (`/dev/urandom` hoặc CSPRNG cấp OS), triệt tiêu hoàn toàn rủi ro suy đoán chuỗi nonce/session token do việc lạm dụng `Math.random()`.
- **Crypto.randomUUID()**:
  - Phát sinh định danh duy nhất toàn cầu RFC 4122 v4 (36 ký tự) nguyên bản mà không cần nạp thư viện ngoài (như gói npm `uuid`).
  - Ràng buộc môi trường: chỉ khả dụng trong Secure Contexts (HTTPS / localhost), ngăn chặn các context không bảo mật sinh định danh giả mạo.
  - Đóng vai trò làm Request Correlation ID và Checkout Idempotency Token cho mini-app.
- **SubtleCrypto.digest()**:
  - Tính toán hàm băm mật mã (`SHA-256`, `SHA-384`, `SHA-512`) bất đồng bộ, trả về `Promise<ArrayBuffer>`.
  - Chạy ngầm ngoài UI thread, bảo đảm việc xác minh tính toàn vẹn của các gói tài nguyên tải về (Subresource Integrity) hoặc kiểm tra checksum cơ sở dữ liệu SQLite offline không gây đứng khung hình giao diện.
- **SubtleCrypto.encrypt() & SubtleCrypto.sign()**:
  - Mã hóa dữ liệu bằng `AES-GCM` với khóa `CryptoKey` không thể trích xuất (`extractable: false`), bảo vệ kho dữ liệu PII và hồ sơ offline trong IndexedDB/OPFS.
  - Ký số số liệu ủy quyền giao dịch tài chính (`ECDSA`, `Ed25519`, `HMAC`) bằng khóa phần cứng, tạo token bằng chứng không thể chối bỏ gửi về super-app host gateway.

---

### 2.2 WHATWG DOM Node Mutation & Hierarchy Containment Sandboxing

- **Element.replaceChildren()**:
  - Thay thế toàn bộ node con của một Element bằng danh sách node hoặc chuỗi mới trong một thao tác nguyên tử.
  - Loại bỏ hoàn toàn mô hình `innerHTML = ''` vốn dễ kích hoạt trình phân tích cú pháp HTML và tiềm ẩn rủi ro parser-mutation XSS.
  - Phát sinh một đợt cập nhật MutationObserver thống nhất, giảm thiểu tối đa hiện tượng layout thrashing.
- **Node.cloneNode()**:
  - Nhân bản sâu (`deep: true`) toàn bộ cây con mà KHÔNG sao chép các listener sự kiện gắn qua `addEventListener` hay thuộc tính `onclick`, tạo ra ranh giới sự kiện hoàn toàn sạch và cô lập.
  - Nhân bản nội dung thẻ `<template>` một cách trơ (`inert`), không thực thi script cho đến khi gắn vào cây DOM hoạt động.
- **Node.contains() & Element.closest()**:
  - `Node.contains`: Kiểm tra phần tử mục tiêu có nằm trong ranh giới của container hay không; là nền tảng cốt lõi cho các thuật toán đóng sheet/dropdown khi người dùng click bên ngoài và ngăn chặn widget lấn chiếm viewport.
  - `Element.closest`: Duyệt ngược cây DOM tìm ancestor khớp với CSS selector; tối ưu hóa mô hình ủy quyền sự kiện (event delegation) từ root container, giảm tải số lượng listener dư thừa.
- **Node.compareDocumentPosition()**:
  - Trả về bitmask vị trí tương đối giữa hai node (`DOCUMENT_POSITION_DISCONNECTED`, `PRECEDING`, `FOLLOWING`, `CONTAINS`, `CONTAINED_BY`).
  - Hỗ trợ công cụ kiểm toán bảo mật kiểm tra thứ tự render DOM (DOM tree order), bảo đảm văn bản cảnh báo/xác nhận thanh toán luôn xuất hiện trước nút bấm xác nhận, ngăn chặn tấn công đánh tráo thứ tự hoặc che mờ giao diện (clickjacking).

---

### 2.3 W3C CSS Transitions Level 1 & State Interpolation Sandboxing

- **transition-property**:
  - Ràng buộc các thuộc tính được phép áp dụng hiệu ứng chuyển đổi; cấm sử dụng `all` trong các stylesheet sản xuất của mini-app nhằm tránh hiện tượng kích hoạt reflow/repaint không mong muốn trên các thuộc tính đắt đỏ (`width`, `height`, `margin`).
  - Giới hạn chuyển đổi vào các thuộc tính tối ưu hóa cho GPU compositor (`transform`, `opacity`, `filter`).
- **transition-duration**:
  - Thiết lập trần thời gian thực thi (khuyến nghị <= 400ms cho modal sheet và <= 250ms cho micro-interactions), bảo đảm giao diện mượt mà và ngắt đánh thức GPU kịp thời để tiết kiệm pin.
  - Tự động đặt về `0s` khi kích hoạt media query `@media (prefers-reduced-motion: reduce)` theo tiêu chuẩn trợ năng WCAG 2.2.
- **transition-timing-function**:
  - Chuẩn hóa hàm nội suy gia tốc (`cubic-bezier`, `steps`) tạo cảm giác vật lý tự nhiên, đồng bộ với các motion token của iOS UIKit và Android Material Design.
- **transitionend & transitioncancel**:
  - `transitionend`: Bắt sự kiện hoàn thành để thực hiện các tác vụ dọn dẹp DOM bất đồng bộ (ví dụ gỡ bỏ modal DOM sau khi fade-out).
  - `transitioncancel`: Bắt sự kiện chuyển đổi bị hủy bỏ đột ngột do tương tác chạm nhanh của người dùng hoặc thay đổi thuộc tính giữa chừng, lập tức giải phóng tài nguyên GPU và reset cờ khóa trạng thái mà không bị treo promise.

---

## 3. Bảng tổng hợp Findings Milestone 122

| ID Finding | Danh mục | Tiêu chuẩn / Giao diện | Nguồn xác thực (HTTP 200) |
|---|---|---|---|
| `STANDARDS-MDN-CRYPTO-GETRANDOMVALUES` | Security Architecture | `Crypto.getRandomValues()` CSPRNG TypedArray Quotas | `https://developer.mozilla.org/en-US/docs/Web/API/Crypto/getRandomValues` |
| `STANDARDS-MDN-CRYPTO-RANDOMUUID` | Security Architecture | `Crypto.randomUUID()` RFC 4122 v4 Collision Resistance | `https://developer.mozilla.org/en-US/docs/Web/API/Crypto/randomUUID` |
| `STANDARDS-MDN-SUBTLECRYPTO-DIGEST` | Security Architecture | `SubtleCrypto.digest()` Asynchronous Cryptographic Hash | `https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/digest` |
| `STANDARDS-MDN-SUBTLECRYPTO-ENCRYPT` | Security Architecture | `SubtleCrypto.encrypt()` Hardware-Accelerated AES-GCM | `https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/encrypt` |
| `STANDARDS-MDN-SUBTLECRYPTO-SIGN` | Security Architecture | `SubtleCrypto.sign()` Digital Signatures & Non-Repudiation | `https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/sign` |
| `STANDARDS-MDN-ELEMENT-REPLACECHILDREN` | Interoperability & Runtime | `Element.replaceChildren()` Atomic Node Replacement | `https://developer.mozilla.org/en-US/docs/Web/API/Element/replaceChildren` |
| `STANDARDS-MDN-NODE-CLONENODE` | Interoperability & Runtime | `Node.cloneNode()` Subtree Replication & Event Isolation | `https://developer.mozilla.org/en-US/docs/Web/API/Node/cloneNode` |
| `STANDARDS-MDN-NODE-CONTAINS` | Security Architecture | `Node.contains()` Subtree Boundary Checking | `https://developer.mozilla.org/en-US/docs/Web/API/Node/contains` |
| `STANDARDS-MDN-ELEMENT-CLOSEST` | Interoperability & Runtime | `Element.closest()` Upward Traversal & Event Delegation | `https://developer.mozilla.org/en-US/docs/Web/API/Element/closest` |
| `STANDARDS-MDN-NODE-COMPAREDOCUMENTPOSITION` | Security Architecture | `Node.compareDocumentPosition()` Topological Bitmask Audit | `https://developer.mozilla.org/en-US/docs/Web/API/Node/compareDocumentPosition` |
| `STANDARDS-MDN-CSS-TRANSITION-PROPERTY` | Interoperability & Runtime | CSS `transition-property` Compositor Scoping | `https://developer.mozilla.org/en-US/docs/Web/CSS/transition-property` |
| `STANDARDS-MDN-CSS-TRANSITION-DURATION` | Interoperability & Runtime | CSS `transition-duration` Battery & Motion Limits | `https://developer.mozilla.org/en-US/docs/Web/CSS/transition-duration` |
| `STANDARDS-MDN-CSS-TRANSITION-TIMING-FUNCTION` | Interoperability & Runtime | CSS `transition-timing-function` Physics Curves | `https://developer.mozilla.org/en-US/docs/Web/CSS/transition-timing-function` |
| `STANDARDS-MDN-ELEMENT-TRANSITIONEND-EVENT` | Interoperability & Runtime | `Element transitionend` Lifecycle Synchronization | `https://developer.mozilla.org/en-US/docs/Web/API/Element/transitionend_event` |
| `STANDARDS-MDN-ELEMENT-TRANSITIONCANCEL-EVENT` | Interoperability & Runtime | `Element transitioncancel` Interruption Handling | `https://developer.mozilla.org/en-US/docs/Web/API/Element/transitioncancel_event` |

---

## 4. Khuyến nghị thực thi trong Tiêu chuẩn Super-App Store

1. **Chính sách Mật mã & Tính ngẫu nhiên**:
   - Cấm hoàn toàn việc sử dụng `Math.random()` trong việc sinh mã xác thực, mã CSRF token, khóa phiên, hoặc transaction ID. Bắt buộc sử dụng `crypto.getRandomValues()` (tối đa 64 KiB/lần) hoặc `crypto.randomUUID()` trong Secure Contexts.
   - Các gói mini-app xử lý giao dịch thanh toán hoặc dữ liệu sinh trắc học bắt buộc sử dụng `SubtleCrypto.encrypt` (AES-GCM) và `SubtleCrypto.sign` với khóa không trích xuất được để chống đánh cắp dữ liệu lưu trữ cục bộ.
2. **Quy tắc Thao tác DOM An toàn**:
   - Sử dụng `replaceChildren()` để làm trống hoặc hoán đổi giao diện động thay vì `innerHTML = ''` hoặc gán HTML trực tiếp.
   - Thao tác ủy quyền sự kiện trên các danh sách phần tử lớn phải thông qua `element.closest()` để triệt tiêu việc đăng ký hàng nghìn listener gây cạn kiệt bộ nhớ.
3. **Quy chuẩn Hiệu ứng Chuyển động (Transitions)**:
   - Cấm khai báo `transition: all` trong file CSS nộp duyệt lên kho ứng dụng; chỉ cho phép transition trên các thuộc tính hỗ trợ tăng tốc GPU (`transform`, `opacity`, `filter`).
   - Mọi animation/transition phải tích hợp bộ xử lý `transitioncancel` để xử lý mượt mà khi người dùng tương tác cử chỉ liên tục, tránh rò rỉ bộ nhớ hoặc bẫy promise.
