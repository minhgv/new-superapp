# Chuyên đề 52: WICG Sanitizer API, WasmGC Compute Sandboxing & WHATWG Origin Private File System (Iteration 56)

## 1. Giới thiệu & Bối cảnh Tiêu chuẩn hóa

Trong kiến trúc Super-App hiện đại (đặc biệt khi mở rộng quy mô từ mô hình PoC của Alibaba Cloud Superapp Solution / WindVane Container lên nền tảng vận hành cấp doanh nghiệp), 3 rào cản kỹ thuật nghiêm trọng nhất tác động trực tiếp đến an ninh, hiệu năng và độ ổn định bao gồm:
1. **DOM Cross-Site Scripting (DOM XSS) & Parser Mutation (mXSS):** Việc các mini-app sử dụng các thư viện lọc HTML tầng người dùng (userland regex/DOMPurify) để parse nội dung từ API hoặc CMS thường xuyên đối mặt với các kỹ thuật tấn công mXSS do sự khác biệt giữa parser của thư viện và parser native của trình duyệt. Tiêu chuẩn **WICG Sanitizer API** và phương thức `Element.setHTML()` chuyển toàn bộ cơ chế làm sạch HTML vào thẳng pipeline C++ của engine trình duyệt.
2. **Kích thước gói & Chi phí Runtime của Managed Languages (WasmGC):** Khi phát triển mini-app bằng các framework hiện đại như Flutter (Dart) hoặc Kotlin Multiplatform, việc biên dịch sang WebAssembly 1.0 trước đây buộc gói ứng dụng phải kèm theo toàn bộ bộ thu gom rác (userland GC runtime), làm phình to gói thêm 1MB - 3MB và gây ra hiện tượng giật khung hình (stop-the-world GC pauses). Chuẩn **WasmGC (WebAssembly Garbage Collection)** đưa các kiểu dữ liệu struct/array có quản lý trực tiếp vào bytecode Wasm, chia sẻ trực tiếp Garbage Collector của trình duyệt (V8 / JavaScriptCore).
3. **Hiệu năng lưu trữ & Tính toàn vẹn CSDL Client-Side (WHATWG OPFS):** Lưu trữ truyền thống qua IndexedDB có độ trễ lớn do phụ thuộc vào cơ chế bất đồng bộ microtask của Event Loop, không đáp ứng được yêu cầu của các CSDL quan hệ phức tạp trong mini-app tài chính/ERP. Tiêu chuẩn **WHATWG Origin Private File System (OPFS)** kết hợp cùng `FileSystemSyncAccessHandle` trong Web Workers mang lại tốc độ I/O đồng bộ ngang bằng với hệ thống tệp cục bộ (POSIX direct I/O) cho **SQLite Wasm**, bảo đảm tính toàn vẹn giao dịch ACID tuyệt đối với Write-Ahead Logging (WAL).

---

## 2. Bảng Tổng hợp Findings Chuẩn hóa (Iteration 56)

| Mã Finding | Danh mục | Tiêu chuẩn / Đặc tả | Khía cạnh Kỹ thuật & Kiểm soát Vận hành | Mức bằng chứng |
|---|---|---|---|---|
| `sanitizer_056_01` | An ninh & Quyền riêng tư | WICG Sanitizer API Spec | **WICG HTML Sanitizer API**: Triệt tiêu triệt để DOM XSS bằng parser native C++, thay thế các hàm inject chuỗi thô nguy hiểm (`innerHTML`, `outerHTML`) bằng `Element.setHTML()`. | `official_standard` |
| `sanitizer_056_02` | An ninh & Quyền riêng tư | WICG Sanitizer API § 2 | **SanitizerConfig Controls**: Thiết lập cấu hình allowlist/blocklist chi tiết cho thẻ, thuộc tính và namespace; xây dựng bộ hồ sơ `StoreStrictProfile` và `StoreRichTextProfile`. | `official_standard` |
| `sanitizer_056_03` | An ninh & Quyền riêng tư | WICG Sanitizer API § 3 | **Element.setHTML() Lifecycle**: Cơ chế parse và chèn DOM nguyên tử một bước (single-pass), ngăn chặn hoàn toàn tấn công Parser-Mutation XSS (mXSS). | `official_standard` |
| `sanitizer_056_04` | An ninh & Quyền riêng tư | WICG Sanitizer API § 4 | **Document.parseHTML()**: Cơ chế parse phân đoạn HTML an toàn thành `DocumentFragment` tách biệt khỏi DOM sống, phục vụ tiền kiểm duyệt và render danh sách ảo. | `official_standard` |
| `sanitizer_056_05` | An ninh & Quyền riêng tư | WICG Sanitizer API § 6 | **Tương thích W3C Trusted Types & Strict CSP**: Kết hợp Sanitizer API với chính sách `require-trusted-types-for script`, bảo đảm luồng dữ liệu an toàn bất khả xâm phạm. | `official_standard` |
| `wasmgc_056_01` | Nền tảng & Đóng gói | W3C WasmGC Community Spec | **WasmGC Managed Objects**: Tích hợp kiểu dữ liệu struct, array và anyref có quản lý trực tiếp vào WebAssembly, chia sẻ chung engine GC native của trình duyệt. | `official_standard` |
| `wasmgc_056_02` | Nền tảng & Đóng gói | Chrome Devs / WasmGC | **Tối ưu hóa Kích thước Gói & Khởi động**: Cắt giảm 60-70% dung lượng bundle Wasm và tăng tốc khởi động nguội gấp 2 lần cho mini-app phát triển bằng Dart/Flutter và Kotlin. | `vendor_documentation` |
| `wasmgc_056_03` | Nền tảng & Đóng gói | V8 Wasm-GC Porting Architecture | **Zero-Copy JS Interop**: Cầu nối giao tiếp hai chiều tốc độ cao giữa WasmGC struct và đối tượng JavaScript thông qua `externref`, triệt tiêu chi phí serialize JSON. | `official_standard` |
| `wasmgc_056_04` | An ninh & Quyền riêng tư | WasmGC Formal Type Safety | **Loại bỏ Lỗ hổng Trỏ bộ nhớ & Tràn bộ đệm**: Hệ thống kiểu tĩnh bảo đảm an toàn bộ nhớ tuyệt đối, ngăn chặn tấn công chiếm quyền điều khiển bên trong sandbox. | `official_standard` |
| `wasmgc_056_05` | Độ tin cậy & Khả năng phục hồi | V8 Multi-Tenant Isolation | **Giám sát Heap Đa người thuê & Phòng ngừa OOM**: Hạn ngạch bộ nhớ heap 128MB cho WasmGC, kích hoạt GC tự động khi mini-app vào nền tránh crash LMK. | `vendor_documentation` |
| `opfs_056_01` | Lưu trữ & Cô lập Dữ liệu | WHATWG File System Standard § 5 | **Origin Private File System (OPFS)**: Hệ thống tệp phân vùng theo origin qua `navigator.storage.getDirectory()`, hoàn toàn ẩn và bảo vệ trước hệ điều hành host và app khác. | `official_standard` |
| `opfs_056_02` | Lưu trữ & Cô lập Dữ liệu | WHATWG File System Standard § 7 | **FileSystemSyncAccessHandle**: Cơ chế I/O đồng bộ trực tiếp trong Web Worker, cung cấp tốc độ đọc/ghi nhị phân thô không làm nghẽn UI thread. | `official_standard` |
| `opfs_056_03` | Lưu trữ & Cô lập Dữ liệu | SQLite Wasm OPFS Documentation | **SQLite Wasm Enterprise Persistence**: Kiến trúc VFS trên OPFS với chế độ WAL bảo đảm toàn vẹn giao dịch ACID, chịu lỗi crash process đột ngột. | `vendor_documentation` |
| `opfs_056_04` | Lưu trữ & Cô lập Dữ liệu | Web.dev OPFS Storage Quotas | **Quản trị Hạn ngạch & Cơ chế Phân loại Giải phóng**: Giám sát dung lượng qua `estimate()`, phân quyền nâng cấp `persist()` miễn trừ tự động dọn dẹp khi áp lực bộ nhớ cao. | `official_standard` |
| `opfs_056_05` | Lưu trữ & Cô lập Dữ liệu | WHATWG File System Standard § 7.1 | **Khóa Tệp Độc quyền & Phòng ngừa Deadlock**: Quản lý vòng đời khóa đồng thời trên FileSystemSyncAccessHandle, tự động giải phóng khóa khi worker chấm dứt. | `official_standard` |

---

## 3. Kiến trúc Chi tiết & Hướng dẫn Thực thi Kỹ thuật

### 3.1. Native DOM XSS Defense với WICG Sanitizer API & Trusted Types

Truyền thống phát triển mini-app dựa nhiều vào việc render mã HTML nhận từ CMS hoặc API bên thứ ba. Việc này tạo ra bề mặt tấn công cực lớn cho DOM XSS.

#### Nguyên lý Triệt tiêu mXSS với `Element.setHTML()`
Phương thức `element.setHTML(untrustedMarkup, { sanitizer: customSanitizer })` thực hiện tuần tự việc phân tích chuỗi, lọc các token nguy hiểm và gắn trực tiếp các node an toàn vào DOM đích trong một chu trình xử lý duy nhất của engine:
```javascript
// Chuẩn hóa bộ lọc Sanitizer cho nội dung phong phú của Mini-App
const miniAppRichTextSanitizer = new Sanitizer({
  elements: ["p", "b", "i", "em", "strong", "ul", "ol", "li", "span", "div", "img", "a"],
  attributes: {
    "a": ["href", "title", "target"],
    "img": ["src", "alt", "width", "height", "loading"],
    "*": ["class", "id"]
  },
  removeElements: ["script", "iframe", "object", "embed", "frame"],
  removeAttributes: ["onerror", "onload", "onclick", "onmouseover"]
});

// Chèn an toàn trực tiếp - Engine tự động loại bỏ javascript: hoặc event handlers
contentContainer.setHTML(untrustedExternalHtml, { sanitizer: miniAppRichTextSanitizer });
```

#### Phối hợp với W3C Trusted Types
Để ngăn chặn hoàn toàn việc các lập trình viên sử dụng các hàm gán chuỗi nguy hiểm cũ (`innerHTML`, `outerHTML`, `document.write`), container WebView cấu hình Content Security Policy bắt buộc:
```http
Content-Security-Policy: require-trusted-types-for 'script'; trusted-types default miniapp-sanitizer;
```
Bất kỳ nỗ lực gán trực tiếp chuỗi thô nào mà không thông qua chính sách Trusted Types đã đăng ký sẽ bị engine chặn lại ngay lập tức và phát sinh lỗi runtime `TypeError`.

---

### 3.2. Tối ưu Hóa Runtime với WebAssembly Garbage Collection (WasmGC)

Sự ra đời của WasmGC là bước ngoặt quyết định cho các Mini-App hiệu năng cao được xây dựng bằng Dart/Flutter hoặc Kotlin Multiplatform:

```
+-----------------------------------------------------------------------+
|                    Super-App Host Container (V8 Engine)               |
|                                                                       |
|  +---------------------------+       +-----------------------------+  |
|  |    JavaScript Runtime     |       |    WasmGC Mini-App Core     |  |
|  | - DOM Event Dispatching   |       | - Flutter / Dart UI logic   |  |
|  | - Super-App Host Bridge   |       | - Managed Structs & Arrays  |  |
|  | - externref handles       |       | - anyref / eqref hierarchy  |  |
|  +-------------+-------------+       +--------------+--------------+  |
|                |                                    |                 |
|                +------------------+-----------------+                 |
|                                   |                                   |
|                                   v                                   |
|              +-----------------------------------------+              |
|              |      Shared Native Garbage Collector    |              |
|              |  - Oilpan / Orinoco Generational Sweep  |              |
|              |  - Zero-Copy Heap Compaction & Tracing  |              |
|              +-----------------------------------------+              |
+-----------------------------------------------------------------------+
```

#### Các lợi ích đo lường thực tế:
1. **Dung lượng tải về giảm 60-70%:** Do không phải tải kèm mã nguồn bộ thu gom rác (Dart runtime GC).
2. **Khởi động nguội (Cold Start) < 150ms:** Bytecode WasmGC được V8 biên dịch phân tầng trực tiếp (Liftoff -> TurboFan) mà không cần bước phân tích cú pháp JavaScript phức tạp.
3. **Cầu nối Zero-Copy:** Dữ liệu truyền giữa JavaScript và WasmGC thông qua con trỏ `externref` được bảo vệ kiểu tĩnh, loại bỏ hoàn toàn các chu kỳ mã hóa/giải mã chuỗi JSON.

---

### 3.3. Enterprise Persistence với Origin Private File System & SQLite Wasm

Đối với các ứng dụng mini-app chuyên ngành tài chính, bảo hiểm, logistic cần xử lý lượng dữ liệu lớn offline, IndexedDB bộc lộ nhược điểm về hiệu năng ghi và độ phức tạp khi truy vấn quan hệ. Giải pháp tối ưu tiêu chuẩn là kết hợp **OPFS** và **SQLite Wasm**:

```javascript
// Thực thi trong Dedicated Web Worker
import { sqlite3Worker1Promiser } from '@sqlite.org/sqlite-wasm';

const promiser = await new Promise(resolve => {
  const p = sqlite3Worker1Promiser({
    onready: () => resolve(p)
  });
});

// Khởi tạo CSDL SQLite bền vững trên OPFS
const openResponse = await promiser('open', {
  filename: 'miniapp_transactions.sqlite3',
  vfs: 'opfs' // Kích hoạt Virtual File System trên FileSystemSyncAccessHandle
});

// Thiết lập chế độ WAL (Write-Ahead Logging) cho hiệu năng ghi cực đại
await promiser('exec', {
  dbId: openResponse.dbId,
  sql: 'PRAGMA journal_mode=WAL; PRAGMA synchronous=NORMAL;'
});
```

#### Quy tắc An toàn và Kiểm soát Tài nguyên:
- **Lập lịch Worker Chuyên dụng:** `FileSystemSyncAccessHandle` chỉ được phép khởi tạo trong Dedicated Worker. Tuyệt đối cấm can thiệp storage đồng bộ trên UI Thread.
- **Hạn ngạch Phân tầng:** Mỗi mini-app mặc định được cấp phát hạn ngạch OPFS tối đa 50MB. Các ứng dụng thuộc danh mục Doanh nghiệp / Ngân hàng có thể yêu cầu hạn ngạch mở rộng lên đến 250MB sau khi vượt qua quy trình thẩm định.
- **Giải phóng Khóa Tệp khi Gặp Sự cố:** Hệ thống Worker Wrapper bắt buộc phải cài đặt khối lệnh `try...finally` để luôn đảm bảo gọi `accessHandle.close()`, phòng tránh hiện tượng kẹt tài nguyên tệp khi worker bị crash đột ngột.

---

## 4. Danh mục Tiêu chí Kiểm định Cửa hàng (Store Review Checklist)

1. **Rà soát Mã Nguồn DOM XSS (Static Code Analysis):**
   - [ ] Mini-app không sử dụng các lệnh gán chuỗi trực tiếp (`element.innerHTML = ...`, `outerHTML = ...`).
   - [ ] Mọi thao tác chèn HTML động từ nguồn ngoài bắt buộc phải sử dụng `element.setHTML()` kèm đối tượng cấu hình `Sanitizer`.
   - [ ] Đối với các ứng dụng giao dịch tài chính, bắt buộc cấu hình CSP kích hoạt `require-trusted-types-for 'script'`.
2. **Quy chuẩn Đóng gói WebAssembly (Wasm Verification):**
   - [ ] Ứng dụng Wasm biên dịch từ managed languages (Dart/Kotlin) phải kích hoạt cờ WasmGC tiêu chuẩn.
   - [ ] Dung lượng nhị phân Wasm nén gzipped không vượt quá 4MB cho gói khởi động ban đầu.
   - [ ] Kiểm tra tính toàn vẹn kiểu tĩnh, không export các hàm can thiệp bộ nhớ thô bất hợp pháp.
3. **Tuân thủ Lưu trữ Dữ liệu Bền vững (OPFS Storage Audit):**
   - [ ] CSDL quan hệ cục bộ bắt buộc sử dụng SQLite Wasm cấu hình OPFS VFS.
   - [ ] Mọi hoạt động đọc/ghi đồng bộ qua `FileSystemSyncAccessHandle` phải được cô lập 100% trong Dedicated Web Worker.
   - [ ] Ứng dụng đăng ký bộ lắng nghe sự kiện dọn dẹp khi dung lượng đạt ngưỡng cảnh báo 80% hạn ngạch.
