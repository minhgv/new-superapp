# Chuyên đề 59: WHATWG FileSystemObserver, WebAuthn Level 3 Cryptographic Extensions & IETF RFC 9457 / RFC 9440 API Resilience Contracts (Iteration 63)

## 1. Bối cảnh & Tầm quan trọng trong Kiến trúc Super App & Mini App Store

Hệ sinh thái Super App hiện đại đòi hỏi sự kết hợp chặt chẽ giữa khả năng lưu trữ ngoại tuyến cục bộ hiệu năng cao, cơ chế xác thực danh tính phân tán không mật khẩu (passwordless passkeys), và các chuẩn giao thức API có khả năng tự phục hồi (resilient API contracts). Khi các mini app phát triển từ các ứng dụng xem nội dung đơn giản thành các giải pháp doanh nghiệp phức tạp (ngân hàng số, chứng khoán, điểm bán lẻ POS đa màn hình, phần mềm văn phòng ngoại tuyến), nền tảng web đối mặt với 3 thách thức kiến trúc trọng yếu:

1. **Tắc nghẽn & Lãng phí Chu kỳ CPU do Cơ chế Polling Thư mục (Filesystem Polling Overhead):**
   Trước đây, để nhận biết các thay đổi trong hệ thống tệp Origin Private File System (OPFS) hoặc thư mục tài liệu do người dùng cấp quyền, mini app buộc phải sử dụng các vòng lặp định kỳ (`setInterval` hoặc `requestAnimationFrame`) để duyệt quét trạng thái cây thư mục (`directory handle iteration`). Điều này gây tiêu tốn pin nghiêm trọng, làm cạn kiệt bộ nhớ và gây xung đột khóa (lock contention) với tiến trình SQLite Write-Ahead Logging (WAL) chạy ngầm trong Web Workers.
2. **Nhu cầu Phân biệt Thiết bị Vật lý & Lưu trữ Khóa Phân tán (Hardware Binding vs. Synced Passkeys):**
   Mặc dù passkey đồng bộ qua đám mây (Apple iCloud Keychain, Google Password Manager) mang lại trải nghiệm tiện lợi vượt trội cho người dùng phổ thông, các nghiệp vụ tài chính giá trị cao và quản trị doanh nghiệp bắt buộc phải có bằng chứng mật mã rằng giao dịch được ký trực tiếp từ phần cứng bảo mật của thiết bị đã đăng ký (Hardware Security Module / Secure Enclave / Android StrongBox), đồng thời cần khả năng ghi dữ liệu mã hóa cục bộ trực tiếp lên thiết bị xác thực (authenticator-bound storage) mà không bị lộ lên hạ tầng máy chủ trung tâm.
3. **Mù mờ Ngữ nghĩa Lỗi HTTP & Ngắt quãng Chuỗi Xác thực mTLS tại Biên Mạng (Edge mTLS Ingress & Problem Details):**
   Trong kiến trúc phân tán microservices của Super App, các cổng ingress proxy (NGINX, Envoy, Cloudflare) thường chấm dứt kết nối Mutual TLS (mTLS) từ đối tác doanh nghiệp tại biên mạng trước khi chuyển tiếp yêu cầu vào các dịch vụ nội bộ. Sự thiếu vắng chuẩn định dạng chuyển tiếp chứng chỉ máy khách dẫn đến các giải pháp độc quyền thiếu an toàn. Đồng thời, việc trả về các phản hồi lỗi HTTP 4xx/5xx dạng văn bản tùy biến hoặc JSON không cấu trúc làm tê liệt khả năng xử lý lỗi tự động, gây khó khăn cho việc truy vết phân tán (distributed tracing) và làm suy giảm độ tin cậy của mini app khi gặp sự cố mạng.

Chuyên đề 59 thiết lập bộ chuẩn kỹ thuật toàn diện tích hợp 3 trụ cột tiêu chuẩn web và mạng nền tảng:
- **WHATWG FileSystemObserver:** Cơ chế giám sát biến động hệ thống tệp bất đồng bộ theo sự kiện, cấu trúc bản ghi `FileSystemChangeRecord`, điều phối đồng bộ cơ sở dữ liệu SQLite OPFS WAL giữa các Worker và UI threads, và giải phóng bộ theo dõi kernel OS.
- **WebAuthn Level 3 Extensions & FIDO CTAP 2.1:** Tiện ích mở rộng `largeBlob` cho lưu trữ mật mã phân tán gắn chặt với phần cứng xác thực, tiện ích `devicePubKey` xác thực khóa phần cứng độc lập với passkey đồng bộ đám mây, tích hợp form autofill với `isConditionalMediationAvailable()`, giao thức kết nối chéo thiết bị FIDO CTAP 2.1 Hybrid (caBLE) kèm xác minh khoảng cách Bluetooth BLE, và chuẩn hóa RP ID scoping.
- **IETF RFC 9457 & RFC 9440 API Resilience:** Định dạng biểu diễn lỗi chuẩn hóa `application/problem+json` (thay thế hoàn toàn RFC 7807), trường tiêu đề `Client-Cert` cấu trúc hóa cho việc chuyển tiếp chứng chỉ mTLS qua reverse proxy, mở rộng Problem Details với mã lỗi định danh và mã truy vết W3C Trace Context, tích hợp ràng buộc token OAuth 2.0 RFC 8705, và quy tắc suy thoái giao diện người dùng mượt mà.

---

## 2. WHATWG FileSystemObserver & Quản trị Biến động Lưu trữ Bất đồng bộ

### 2.1. Giao diện FileSystemObserver & Cấu trúc Bản ghi FileSystemChangeRecord
Đặc tả **WHATWG File System Living Standard** chuẩn hóa giao diện `FileSystemObserver`, cho phép ứng dụng web và mini app nhận thông báo bất đồng bộ khi có sự thay đổi trong cấu trúc tệp và thư mục mà không cần thực hiện polling:

```javascript
// Khởi tạo FileSystemObserver với hàm callback nhận mảng bản ghi thay đổi
const observer = new FileSystemObserver((records, observerInstance) => {
  for (const record of records) {
    console.log(`[FS Event] Root: ${record.root.name}, Type: ${record.type}`);
    console.log(`Changed Handle: ${record.changedHandle.name}`);
    console.log(`Relative Path: ${record.relativePathComponents.join('/')}`);
  }
});
```

Cấu trúc bản ghi **`FileSystemChangeRecord`** cung cấp thông tin chi tiết:
1. `root`: Đối tượng `FileSystemHandle` gốc được đăng ký theo dõi ban đầu.
2. `changedHandle`: Đối tượng `FileSystemHandle` (tệp hoặc thư mục) thực tế phát sinh biến động.
3. `type`: Phân loại biến động chuẩn hóa theo máy trạng thái:
   - `'appeared'`: Tệp hoặc thư mục mới được tạo hoặc di chuyển vào cây theo dõi.
   - `'disappeared'`: Tệp hoặc thư mục bị xóa hoặc di chuyển ra khỏi cây theo dõi.
   - `'modified'`: Nội dung tệp bị thay đổi hoặc kích thước bị biến động.
   - `'moved'`: Tệp hoặc thư mục được đổi tên hoặc chuyển vị trí trong cùng cây theo dõi.
   - `'unknown'`: Biến động không xác định do hệ thống tệp kernel bị tràn bộ đệm sự kiện.
4. `relativePathComponents`: Mảng các chuỗi biểu diễn đường dẫn tương đối từ `root` tới phần tử biến động (ví dụ: `['data', 'cache', 'state.json']`).

### 2.2. Phương thức FileSystemObserver.observe() & Giới hạn Theo dõi Đệ quy
Quá trình theo dõi được kích hoạt qua phương thức `observe()`:
```javascript
// Theo dõi toàn bộ cây thư mục đệ quy trong Origin Private File System
const opfsRoot = await navigator.storage.getDirectory();
observer.observe(opfsRoot, { recursive: true });
```
- **Cơ chế Kernel Binding:** Trên môi trường Android và Linux, trình duyệt ánh xạ các lời gọi theo dõi vào `inotify` hoặc `fanotify`; trên macOS/iOS sử dụng `FSEvents`; trên Windows sử dụng `ReadDirectoryChangesW`.
- **Ranh giới Bảo mật Nguồn:** Việc gọi `observe()` trên một handle không thuộc sandbox của mini app hoặc khi quyền đọc bị thu hồi sẽ ném lỗi `SecurityError` hoặc `NotAllowedError`.

| Tham số / Tiêu chí | Quy định Kỹ thuật Web | Yêu cầu Chuẩn hóa Super App Store |
| :--- | :--- | :--- |
| **Phạm vi đệ quy (`recursive`)** | Hỗ trợ `true` / `false` trên DirectoryHandle | Bắt buộc giới hạn độ sâu đệ quy `<= 5` tầng thư mục để ngăn cạn kiệt file descriptor của OS. |
| **Gom cụm sự kiện (Debounce/Batching)** | Coalescing sự kiện tại tầng engine | Micro-task batching: gộp các ghi chép liên tiếp trong vòng 50ms thành một lần dispatch duy nhất. |
| **Phản hồi khi tràn đệm** | Phát sinh record với type `'unknown'` | Mini app bắt buộc phải thực hiện quét toàn diện lại bộ nhớ cache khi nhận type `'unknown'`. |

### 2.3. Vòng đời, Hủy Theo dõi & Thu hồi Tài nguyên Kernel OS
Các bộ theo dõi hệ thống tệp giữ các tham chiếu cấp thấp trong kernel của hệ điều hành. Nếu không được giải phóng kịp thời, chúng sẽ gây rò rỉ file descriptor nghiêm trọng:
```javascript
// Hủy theo dõi một handle cụ thể
observer.unobserve(specificDirHandle);

// Hủy toàn bộ theo dõi và xóa sạch hàng đợi sự kiện
observer.disconnect();
```
**Quy tắc Vòng đời Bắt buộc:**
1. Mini app **bắt buộc** phải gọi `observer.disconnect()` khi component unmount, khi nhận sự kiện Page Lifecycle `'freeze'`, hoặc trước khi đóng cửa sổ (`window.onunload`).
2. Watchdog của Super App Container giám sát số lượng handle inotify/FSEvents đang mở của mỗi tiến trình webview. Nếu một tiến trình mini app duy trì quá 32 kernel file watch descriptors mà không có tương tác người dùng, container có quyền đơn phương chấm dứt webview để bảo vệ tài nguyên thiết bị.

### 2.4. Điều phối Đồng bộ SQLite Write-Ahead Logging (WAL) trong OPFS
Khi mini app sử dụng SQLite WebAssembly trên nền Origin Private File System (OPFS), tệp cơ sở dữ liệu chính (`db.sqlite`) hoạt động cùng hai tệp phụ: tệp nhật ký ghi trước Write-Ahead Log (`db.sqlite-wal`) và tệp bộ nhớ chia sẻ (`db.sqlite-shm`):
- Do worker chuyên trách giữ khóa độc quyền thông qua `FileSystemSyncAccessHandle`, các worker phụ trợ hoặc giao diện người dùng không thể liên tục mở tệp để kiểm tra dữ liệu mới.
- Bằng cách đăng ký `FileSystemObserver` trên thư mục chứa database, các tiến trình đọc (readers) nhận được sự kiện `'modified'` trên tệp `-wal` ngay khi một giao dịch commit thành công.
- Cơ chế này cho phép các màn hình giao diện vô hiệu hóa bộ nhớ đệm (cache invalidation) và kích hoạt truy vấn đọc tức thời mà không gây tranh chấp khóa ghi (write lock contention), duy trì độ trễ phản hồi đồng bộ dưới 100 ms trên các thiết bị bán hàng POS đa màn hình.

---

## 3. WebAuthn Level 3 Cryptographic Extensions & FIDO CTAP 2.1 Passkeys

### 3.1. Tiện ích Mở rộng `largeBlob` cho Lưu trữ Khóa Phân tán
Đặc tả **W3C Web Authentication Level 3 (§ 10.4)** giới thiệu tiện ích mở rộng `largeBlob`, cho phép Relying Party (RP) đọc và ghi các khối dữ liệu nhị phân tùy ý trực tiếp trên thiết bị xác thực FIDO2 / Passkey:

```javascript
// Đăng ký passkey với yêu cầu hỗ trợ largeBlob
const credential = await navigator.credentials.create({
  publicKey: {
    // ... các tham số khởi tạo RP & User tiêu chuẩn
    extensions: {
      largeBlob: { support: "required" }
    }
  }
});
const extResults = credential.getClientExtensionResults();
console.log(`largeBlob supported: ${extResults.largeBlob?.supported}`);
```

Khi thực hiện xác thực (`navigator.credentials.get`), mini app có thể ghi đè hoặc đọc khối dữ liệu:
```javascript
// Ghi dữ liệu nhị phân (ví dụ: khóa gốc ví tiền mã hóa hoặc chứng thư số ngoại tuyến)
const writeResult = await navigator.credentials.get({
  publicKey: {
    // ...
    extensions: {
      largeBlob: { write: rawKeyArrayBuffer }
    }
  }
});

// Đọc dữ liệu nhị phân đã lưu trữ
const readResult = await navigator.credentials.get({
  publicKey: {
    // ...
    extensions: {
      largeBlob: { read: true }
    }
  }
});
const decryptedBlob = readResult.getClientExtensionResults().largeBlob?.blob;
```
- **Mã hóa Cứng (Hardware-Encrypted Storage):** Khối dữ liệu được mã hóa đối xứng trực tiếp trong phần cứng của authenticator bằng khóa mật sinh ra từ quá trình sinh trắc học người dùng. Dữ liệu không thể bị trích xuất nếu không có sự hiện diện vật lý của chủ sở hữu.
- **Tiêu chí Kỹ thuật:** Giới hạn kích thước payload `largeBlob` thường từ 1 KB đến 4 KB. Mini app phải xử lý trường hợp lỗi khi bộ nhớ authenticator bị đầy (`authenticator storage full`).

### 3.2. Tiện ích `devicePubKey` Xác thực Khóa Phần cứng Độc lập
Passkey đa thiết bị (đồng bộ qua iCloud Keychain hoặc Google Password Manager) mang lại sự thuận tiện nhưng không thể chứng minh thiết bị nào trong chuỗi đồng bộ đã thực hiện ký duyệt. Tiện ích **`devicePubKey` (§ 10.5)** giải quyết bài toán này:
- Khi đăng ký hoặc xác thực, RP gửi yêu cầu: `extensions: { devicePubKey: { attestation: "direct" } }`.
- Thiết bị sinh ra một cặp khóa bất đối xứng gắn chặt với phần cứng (`device-bound key`) nằm trong Apple Secure Enclave hoặc Android StrongBox Keystore.
- Chữ ký xác thực trả về chứa hai chữ ký độc lập: chữ ký của Passkey người dùng và chữ ký của Khóa Thiết bị (`devicePubKey signature`).
- **Ứng dụng Tài chính:** Giúp hệ thống Core Banking của Super App phân biệt giữa một thiết bị lạ vừa đồng bộ passkey và thiết bị chính chủ đã đăng ký sinh trắc học trực tiếp tại quầy, cho phép kích hoạt quy trình xác thực nâng cao (step-up authentication) đối với các giao dịch có giá trị trên 50.000.000 VNĐ.

### 3.3. Tích hợp Điền Form Mượt mà với `isConditionalMediationAvailable()`
Loại bỏ hoàn toàn các hộp thoại xác thực modal gây gián đoạn luồng người dùng:
```javascript
// Kiểm tra khả năng hỗ trợ Conditional UI của WebView
const isAvailable = await PublicKeyCredential.isConditionalMediationAvailable();
if (isAvailable) {
  // Kích hoạt lắng nghe thụ động trên trường input username
  navigator.credentials.get({
    mediation: 'conditional',
    publicKey: {
      challenge: serverChallengeUint8Array,
      rpId: 'superapp.vn'
    }
  }).then(handleAuthenticationSuccess);
}
```
Trường HTML input chỉ cần khai báo thuộc tính: `<input type="text" autocomplete="username webauthn">`. Khi người dùng chạm vào ô đăng nhập, danh sách passkey có sẵn sẽ hiển thị trực tiếp trong menu gợi ý của bàn phím ảo. Việc xác thực hoàn tất ngay sau khi chạm vân tay/FaceID mà không cần chuyển trang.

### 3.4. Giao thức FIDO CTAP 2.1 Hybrid (caBLE) & Kiểm tra Khoảng cách BLE
Đối với các mini app chạy trên màn hình lớn (Desktop, Smart TV, POS bán lẻ):
1. **Mã QR Động:** Thiết bị hiển thị mã QR chứa URL đường hầm mã hóa và nonce mật mã.
2. **Kênh WebSocket Bảo mật:** Ứng dụng Super App trên điện thoại quét mã QR và thiết lập kênh truyền tin mã hóa đầu-cuối qua máy chủ trung chuyển tín hiệu (Signal Server).
3. **Xác minh Khoảng cách Bluetooth BLE:** Điện thoại phát tín hiệu BLE beacon ngắn hạn. Thiết bị đích bắt buộc phải thu được tín hiệu BLE này để chứng minh hai thiết bị đang ở cùng một vị trí vật lý (bán kính `<= 10m`), triệt tiêu hoàn toàn nguy cơ tấn công trung gian (Man-In-The-Middle) từ xa qua mạng Internet.

---

## 4. IETF RFC 9457 & RFC 9440 API Resilience Contracts

### 4.1. IETF RFC 9457: Problem Details for HTTP APIs (Thay thế RFC 7807)
Đặc tả **RFC 9457** chuẩn hóa định dạng dữ liệu lỗi máy học đọc được (machine-readable) cho toàn bộ các dịch vụ web và API gateway, sử dụng kiểu nội dung `application/problem+json`:

```json
{
  "type": "https://api.superapp.vn/errors/out-of-stock",
  "status": 409,
  "title": "Sản phẩm tạm thời hết hàng",
  "detail": "Mã sản phẩm SKU-9842 không còn đủ số lượng tồn kho theo yêu cầu đặt hàng (yêu cầu: 3, khả dụng: 0).",
  "instance": "/orders/tx-784920/items/sku-9842",
  "code": "INSUFFICIENT_INVENTORY",
  "trace_id": "00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01",
  "invalid_params": [
    {
      "name": "quantity",
      "reason": "Vượt quá tồn kho thực tế",
      "available": 0
    }
  ]
}
```

Các thành viên tiêu chuẩn cốt lõi:
- `type` (chuỗi URI): Định danh loại lỗi; mặc định là `"about:blank"` nếu ánh xạ trực tiếp theo mã HTTP status code.
- `status` (số nguyên): Mã trạng thái HTTP gốc do máy chủ phát sinh.
- `title` (chuỗi ngắn): Tóm tắt ngắn gọn loại lỗi, **không được thay đổi** giữa các lần phát sinh lỗi cùng loại.
- `detail` (chuỗi chi tiết): Giải thích chi tiết nguyên nhân cụ thể của lần phát sinh lỗi này.
- `instance` (chuỗi URI): Định danh duy nhất phiên phát sinh lỗi cụ thể, phục vụ việc tra cứu log hệ thống.

### 4.2. IETF RFC 9440: Client-Cert HTTP Header Field cho Chuyển tiếp mTLS
Trong các cụm dịch vụ đám mây, các proxy biên (Envoy, NGINX, Cloudflare) kết thúc kết nối mTLS của đối tác và chuyển tiếp chứng chỉ vào bên trong qua mạng riêng:
- **Chuẩn hóa RFC 8941 Structured Field:** Tiêu đề `Client-Cert` mang chuỗi nhị phân (Byte Sequence) được mã hóa Base64 theo quy cách `:base64_der_cert:`.
- **Triệt tiêu Giả mạo (Sanitization):** Cổng ingress proxy bắt buộc phải loại bỏ hoặc ghi đè toàn bộ tiêu đề `Client-Cert` do máy khách bên ngoài gửi tới trước khi chuyển tiếp yêu cầu vào mạng nội bộ.
- **Tiêu đề `Client-Cert-Chain`:** Dùng để chuyển tiếp mảng các chứng thư số trung gian (CA chain) nếu máy chủ ứng dụng nội bộ cần xác minh toàn diện chuỗi ủy quyền.

### 4.3. Ràng buộc Token mTLS (RFC 8705 & FAPI 2.0 Integration)
Tích hợp giữa tiêu đề `Client-Cert` của RFC 9440 và chuẩn bảo mật ngân hàng mở Financial-Grade API (FAPI 2.0):
1. Khi máy khách gọi API kèm Access Token có ràng buộc chứng chỉ (`cnf.x5t#S256`), API Gateway trích xuất chứng chỉ DER từ tiêu đề RFC 9440 `Client-Cert`.
2. Gateway tính toán mã băm SHA-256 trên toàn bộ chứng chỉ và so khớp chính xác với giá trị `x5t#S256` trong JWT claim.
3. Nếu phát hiện sai lệch (tấn công token replay hoặc thay thế chứng chỉ), Gateway lập tức từ chối yêu cầu và trả về lỗi HTTP 401 với cấu trúc RFC 9457:
   ```json
   {
     "type": "https://api.superapp.vn/errors/token-binding-mismatch",
     "status": 401,
     "title": "Ràng buộc chứng chỉ mTLS không hợp lệ",
     "detail": "Mã băm chứng chỉ máy khách không khớp với xác nhận cnf trong Access Token.",
     "code": "CERTIFICATE_HASH_MISMATCH",
     "trace_id": "00-6a8b7c9d-01"
   }
   ```

### 4.4. Xử lý Lỗi Phía Máy Khách & Suy thoái Giao diện Mượt mà (Graceful Degradation)
Thư viện HTTP Client của Mini App SDK bắt buộc tích hợp bộ chặn phản hồi (Response Interceptor):
- Tự động nhận diện `Content-Type: application/problem+json`.
- Ánh xạ mã định danh `code` và mảng `invalid_params` trực tiếp vào các component giao diện tương ứng (ví dụ: làm đỏ viền ô nhập liệu và hiển thị lỗi cụ thể dưới trường form).
- Đọc tiêu đề `Retry-After` hoặc thuộc tính mở rộng `retry_after` để hiển thị đồng hồ đếm ngược khi bị giới hạn tốc độ (Rate Limit 429), ngăn chặn người dùng bấm liên tục gây nghẽn hệ thống.

---

## 5. Ma trận Kiểm soát Tuân thủ & Tiêu chí Kiểm duyệt Mini App Store

| Mã Kiểm soát | Lĩnh vực Tiêu chuẩn | Mô tả Quy tắc Kỹ thuật | Mức độ Tuân thủ | Phương thức Đánh giá Store Review |
| :--- | :--- | :--- | :--- | :--- |
| **CTL-FS-01** | WHATWG FileSystemObserver | Bắt buộc sử dụng FileSystemObserver thay thế vòng lặp polling khi theo dõi thư mục. | **Bắt buộc (Normative)** | Static Code Analysis: phát hiện và từ chối các lệnh `setInterval` chứa hàm duyệt handle thư mục. |
| **CTL-FS-02** | Quản trị Kernel Handle | Bắt buộc gọi `disconnect()` khi component unmount hoặc khi trang chuyển sang trạng thái frozen. | **Bắt buộc (Normative)** | Runtime Sandbox Watchdog: đo lường số lượng OS inotify watch descriptors, giới hạn tối đa `<= 32`. |
| **CTL-WA-01** | WebAuthn Conditional UI | Hỗ trợ đăng nhập tự động qua form autofill khi `isConditionalMediationAvailable()` trả về true. | **Khuyến nghị Cao** | Kiểm duyệt UX: xác minh luồng đăng nhập không hiển thị modal dialog chắn màn hình khi khởi chạy. |
| **CTL-WA-02** | Khóa Phần cứng Phân biệt | Nghiệp vụ giao dịch tài chính > 50 triệu VNĐ bắt buộc xác minh tiện ích mở rộng `devicePubKey`. | **Bắt buộc (Phân hệ Tài chính)** | Gateway Conformance Suite: kiểm tra chữ ký enclave phần cứng trong payload xác thực giao dịch. |
| **CTL-WA-03** | largeBlob Cryptography | Sử dụng `largeBlob` để lưu trữ khóa gốc ví; xử lý an toàn lỗi bộ nhớ authenticator đầy. | **Bắt buộc (Ví & Danh tính)** | Kiểm duyệt An ninh: kiểm tra khả năng mã hóa dữ liệu cục bộ và xử lý ngoại lệ tràn bộ nhớ authenticator. |
| **CTL-API-01** | RFC 9457 Problem Details | Toàn bộ phản hồi API lỗi 4xx/5xx phải trả về đúng định dạng `application/problem+json`. | **Bắt buộc (Normative)** | Automated API Linter: kiểm tra trường `type`, `title`, `status`, `detail`, `code`, và `trace_id`. |
| **CTL-API-02** | RFC 9440 Edge Ingress | Reverse Proxy biên mạng bắt buộc khử tiêu đề `Client-Cert` bên ngoài và đóng gói chứng chỉ DER chuẩn RFC 8941. | **Bắt buộc (Hạ tầng Nền tảng)** | Pentest & Security Audit: tiêm tiêu đề `Client-Cert` giả lập từ mạng Internet công cộng để kiểm tra khả năng loại bỏ. |
| **CTL-API-03** | Ràng buộc FAPI Token | Đối chiếu SHA-256 fingerprint giữa tiêu đề RFC 9440 và claim `cnf.x5t#S256` trong Access Token. | **Bắt buộc (B2B & Banking)** | Automated Security Gate: gửi token với chứng chỉ giả lập để kiểm tra phản hồi từ chối 401 chuẩn hóa. |

---

## 6. Lộ trình Triển khai Kỹ thuật & Khuyến nghị Vận hành

### 6.1. Giai đoạn 1: Chuẩn hóa Gateway Biên & Lớp Xử lý Lỗi RFC 9457 (Tháng 1 - Tháng 2)
1. Cấu hình Envoy / NGINX Ingress Controller tại các cụm Kubernetes của Super App để triển khai tiêu chuẩn RFC 9440, chuyển tiếp chứng chỉ máy khách mTLS dưới dạng byte sequence chuẩn RFC 8941.
2. Nâng cấp toàn bộ các API Gateway công khai và cổng kết nối đối tác mini app sang định dạng lỗi RFC 9457 `application/problem+json`, tích hợp sẵn trường `trace_id` đồng bộ với W3C Trace Context.
3. Phát hành bản cập nhật Mini App Core SDK (JavaScript/TypeScript) tích hợp sẵn bộ giải mã lỗi Problem Details tự động.

### 6.2. Giai đoạn 2: Tối ưu hóa Lưu trữ OPFS với FileSystemObserver (Tháng 3 - Tháng 4)
1. Cập nhật nhân WebView của Super App (Chromium WebView trên Android và WKWebView trên iOS) để kích hoạt cờ thử nghiệm và hỗ trợ đầy đủ giao diện `FileSystemObserver`.
2. Hướng dẫn các nhà phát triển mini app phân hệ thương mại và POS chuyển đổi hệ thống lưu trữ SQLite Wasm sang mô hình lắng nghe biến động tệp `-wal` bất đồng bộ.
3. Thiết lập công cụ kiểm tra tĩnh (linter rules) trong quy trình nộp duyệt ứng dụng của Mini App Store để phát hiện các vòng lặp polling tệp dư thừa.

### 6.3. Giai đoạn 3: Tích hợp Toàn diện Passkeys Phần cứng & Cross-Device Auth (Tháng 5 - Tháng 6)
1. Triển khai Relying Party Gateway hỗ trợ tiện ích mở rộng `devicePubKey` và `largeBlob` cho các mini app thuộc phân hệ tài chính, chứng khoán và ví điện tử.
2. Tích hợp giao thức FIDO CTAP 2.1 Hybrid trên các ứng dụng Super App phiên bản Desktop và Web Portal, cho phép người dùng đăng nhập bằng cách quét mã QR và xác thực qua Bluetooth BLE với điện thoại.
3. Ban hành tiêu chuẩn kiểm duyệt Store bắt buộc áp dụng Conditional UI Autofill cho toàn bộ các màn hình đăng nhập tài khoản.
