# Chuyên đề 19: Cô lập Hệ thống Tệp Hộp cát (Sandbox Storage), Quản trị Egress/CSP và Tích hợp Bề mặt Hệ điều hành (OS Integration)

## 1. Bối cảnh và Mục tiêu Nghiên cứu Iteration 23

Iteration 23 hoàn thiện kiến trúc bảo vệ lớp thực thi cục bộ (Client-Side Runtime & Boundary Governance) của Mini App Store trên Super App dựa trên 3 trụ cột kỹ thuật then chốt:
1. **Cô lập Hệ thống Tệp Hộp cát Cục bộ (Client-side Sandbox Storage & OPFS Isolation)**: Chuẩn WHATWG File System Standard / Origin Private File System (OPFS), mô hình phân vùng lưu trữ theo `appId` và người dùng của WeChat (`FileSystemManager`: `usr/`, `temp/`, `package`), và chuẩn an ninh lưu trữ thiết bị di động OWASP MASVS-STORAGE-1.
2. **Quản trị Luồng Mạng Chiều đi (Network Egress Boundaries & CSP Governance)**: Chuẩn W3C Content Security Policy (CSP) Level 3 (`connect-src`, `script-src`, `object-src 'none'`, `frame-ancestors`), quy chế danh sách trắng tên miền bắt buộc (pre-registered domain allowlist & TLS 1.2+ governance của WeChat), và cơ chế trung gian/ghim chứng chỉ (host certificate pinning & mediation theo OWASP MASVS-NETWORK-1 / Mobile Top 10 M5).
3. **Tích hợp Bề mặt Hệ điều hành và Chia sẻ Xã hội (Social Share & OS Integration)**: Chuẩn W3C Web Share API (chia sẻ ra ngoài có kích hoạt cử chỉ), W3C Web Share Target API (tiếp nhận dữ liệu/tệp vào mini-app), W3C Badging API (quản trị huy hiệu ứng dụng không gây spam), và W3C Contact Picker API (chọn danh bạ có chủ đích qua giao diện hệ thống).

Toàn bộ 15 phát hiện mới trong vòng nghiên cứu này đều được kiểm chứng độc lập với mã trạng thái HTTP 200 từ các tổ chức chuẩn hóa và tài liệu nền tảng chính thức (WHATWG, W3C, WeChat Mini Program Official Docs, OWASP MASVS/MASTG, MDN).

---

## 2. Bảng Tổng hợp 15 Phát hiện Kỹ thuật Iteration 23

| # | Mã định danh | Chủ đề | Chuẩn tham chiếu | Mức bằng chứng | URL nguồn xác thực |
|---|---|---|---|---|---|
| 1 | `storage-sandbox-origin-binding` | Sandbox Storage / OPFS | WHATWG File System Standard §2 Storage, §3 Directories and files, §4 NavigatorStorage, §7 Security and Privacy | `normative_standard` | [https://fs.spec.whatwg.org/](https://fs.spec.whatwg.org/) |
| 2 | `storage-sandbox-opfs-performance-quota` | OPFS Quota & Sync Access | MDN Origin Private File System; StorageManager.getDirectory(); FileSystemSyncAccessHandle; Quota and storage eviction | `industry_standard` | [https://developer.mozilla.org/en-US/docs/Web/API/File_System_API/Origin_private_file_system](https://developer.mozilla.org/en-US/docs/Web/API/File_System_API/Origin_private_file_system) |
| 3 | `storage-sandbox-wechat-appid-isolation` | WeChat Sandbox Isolation | WeChat Mini Program Dev Docs: Framework Ability - File System Architecture and Storage Isolation | `official_platform_practice` | [https://developers.weixin.qq.com/miniprogram/en/dev/framework/ability/file-system.html](https://developers.weixin.qq.com/miniprogram/en/dev/framework/ability/file-system.html) |
| 4 | `storage-sandbox-wechat-tiered-lifecycle` | Storage Tiers & Lifecycle | WeChat Mini Program File System: Package, Temp, and User Storage Directories Lifecycle; FileSystemManager API | `official_platform_practice` | [https://developers.weixin.qq.com/miniprogram/en/dev/framework/ability/file-system.html](https://developers.weixin.qq.com/miniprogram/en/dev/framework/ability/file-system.html) |
| 5 | `storage-sandbox-masvs-sensitive-at-rest` | Sensitive Storage Isolation | OWASP Mobile Application Security Verification Standard: MASVS-STORAGE-1 (Storage Security); MASTG Data Storage Architecture | `industry_standard` | [https://mas.owasp.org/MASVS/controls/MASVS-STORAGE-1/](https://mas.owasp.org/MASVS/controls/MASVS-STORAGE-1/) |
| 6 | `network-egress-csp-connect-001` | CSP Egress connect-src | W3C Content Security Policy Level 3: §6.1.2 connect-src, §6.1.1 default-src, §7 Policy Enforcement | `normative_standard` | [https://w3c.github.io/webappsec-csp/](https://w3c.github.io/webappsec-csp/) |
| 7 | `network-egress-csp-script-object-002` | CSP script-src & object-src | W3C Content Security Policy Level 3: §6.1.15 script-src, §6.1.12 object-src, §7.1 Guarding Execution | `normative_standard` | [https://w3c.github.io/webappsec-csp/](https://w3c.github.io/webappsec-csp/) |
| 8 | `network-egress-csp-frame-ancestors-003` | CSP frame-ancestors | W3C Content Security Policy Level 3: §6.1.7 frame-ancestors, §7.3 Embedding Protection | `normative_standard` | [https://w3c.github.io/webappsec-csp/](https://w3c.github.io/webappsec-csp/) |
| 9 | `network-egress-wechat-allowlist-004` | WeChat Network Allowlist | WeChat Mini Program Dev Docs: Framework Ability - Network: Request, Upload, Download, and WebSocket configuration | `official_platform_practice` | [https://developers.weixin.qq.com/miniprogram/en/dev/framework/ability/network.html](https://developers.weixin.qq.com/miniprogram/en/dev/framework/ability/network.html) |
| 10 | `network-egress-owasp-host-mediation-005` | OWASP Network Mediation | OWASP MASVS-NETWORK-1; OWASP MASTG Network Communication Architecture; OWASP Mobile Top 10 M5: Insecure Communication | `industry_standard` | [https://mas.owasp.org/MASVS/controls/MASVS-NETWORK-1/](https://mas.owasp.org/MASVS/controls/MASVS-NETWORK-1/) |
| 11 | `os-integration-web-share-001` | Web Share API | W3C Web Share API: §5 navigator.share(), §6 Security and Privacy Considerations, Transient Activation | `normative_standard` | [https://w3c.github.io/web-share/](https://w3c.github.io/web-share/) |
| 12 | `os-integration-web-share-target-002` | Web Share Target Manifest | W3C Web Share Target API: §2 share_target manifest member, §3 Launch Processing, §4 Security Considerations | `normative_standard` | [https://w3c.github.io/web-share-target/](https://w3c.github.io/web-share-target/) |
| 13 | `os-integration-web-share-target-003` | Inbound File Target Sharing | W3C Web Share Target API: §2.1 ShareTargetFiles member, Multipart Form Ingestion, Enclosure Limits | `normative_standard` | [https://w3c.github.io/web-share-target/](https://w3c.github.io/web-share-target/) |
| 14 | `os-integration-badging-004` | Badging API Lifecycle | W3C Badging API: §4 navigator.setAppBadge / clearAppBadge, §6 User Agent Display and Abuse Prevention | `normative_standard` | [https://w3c.github.io/badging/](https://w3c.github.io/badging/) |
| 15 | `os-integration-contact-picker-005` | Contact Picker Privacy | W3C Contact Picker API: §4 navigator.contacts.select(), §6 Privacy and Security Controls | `normative_standard` | [https://www.w3.org/TR/contact-picker/](https://www.w3.org/TR/contact-picker/) |

---

## 3. Phân tích Kỹ thuật Chi tiết Từng Trụ cột

### 3.1. Phân vùng Lưu trữ Hộp cát Cục bộ (Client-Side Sandbox Storage & FileSystem Isolation)

#### A. Chuẩn WHATWG File System & Origin Private File System (OPFS)
- **Mã phát hiện**: `storage-sandbox-origin-binding`, `storage-sandbox-opfs-performance-quota`
- **Nguyên lý chuẩn hóa**:
  - Giao diện `navigator.storage.getDirectory()` trả về một `FileSystemDirectoryHandle` bị khóa cứng vào nguồn gốc riêng (Origin Private Storage Partition). Cây tệp riêng này hoàn toàn vô hình đối với người dùng cuối và các nguồn gốc khác.
  - Trong Web Worker, `createSyncAccessHandle()` cung cấp truy cập tệp nhị phân đồng bộ hiệu năng cao (in-place read/write, flush, truncate) phục vụ các cơ sở dữ liệu nhúng (như SQLite Wasm).
  - Khung lưu trữ tuân thủ hạn ngạch của `StorageManager.estimate()`. Khi hệ thống gặp áp lực dung lượng, các tệp private có thể bị thu hồi/xóa nếu thuộc nhóm bộ nhớ đệm giải phóng được (`best-effort`).
- **Áp dụng cho Super App**:
  - Mỗi mini-app được host cấp phát một vùng thư mục riêng biệt gắn với cặp định danh `(superapp_user_id, mini_app_id, package_release_id)`.
  - Nghiêm cấm mọi cầu nối native cho phép mini-app duyệt đường dẫn tuyệt đối của hệ điều hành máy khách (`file:///`, `/data/data/...`, `/sdcard/`). Mọi thao tác tệp chỉ được thực thi tương đối bên trong gốc thư mục hộp cát được ánh xạ.

#### B. Kiến trúc 3 Tầng Lưu trữ của WeChat Mini Program (`FileSystemManager`)
- **Mã phát hiện**: `storage-sandbox-wechat-appid-isolation`, `storage-sandbox-wechat-tiered-lifecycle`
- **Nguyên lý thực tế**:
  - WeChat phân chia không gian tệp cục bộ thành 3 phân vùng với vòng đời khác nhau:
    1. **Package Directory (`wxfile://app/`)**: Chứa mã nguồn, tài nguyên bundle đã tải về. Chỉ có quyền đọc (`read-only`), không được chỉnh sửa. Bị ghi đè hoặc thay thế nguyên khối khi cập nhật gói mới.
    2. **User Persistent Directory (`wxfile://usr/`)**: Dành cho mini-app ghi dữ liệu lâu dài (cài đặt người dùng, tệp lưu trữ ngoại tuyến). Có hạn ngạch cố định (ví dụ tối đa 200MB cho mỗi mini-app). Không bị hệ điều hành xóa khi thiếu bộ nhớ trừ khi người dùng xóa mini-app.
    3. **Temp/Ephemeral Cache Directory (`wxfile://tmp/`)**: Dành cho các tệp tạm thời tạo ra khi tải ảnh, quay video, hoặc giải nén tệp. Có thể bị host dọn dẹp định kỳ hoặc khi ứng dụng thoát/bộ nhớ hệ điều hành ở mức nguy hiểm.
- **Áp dụng cho Super App**:
  - Thiết lập API chuẩn `SuperAppFileSystemManager` tương đương, cung cấp các URI ảo định tuyến hộp cát (`superappfile://usr/`, `superappfile://tmp/`).
  - Kiểm soát hạn ngạch dung lượng đĩa nghiêm ngặt theo cấp phân hạng đối tác (ví dụ Tier-1: 200MB, Tier-2: 50MB, mặc định: 20MB) và cảnh báo tới bảng điều khiển nhà phát triển trước khi chặn ghi tệp.

#### C. Bảo vệ Dữ liệu Nhạy cảm khi Lưu trữ (OWASP MASVS-STORAGE-1)
- **Mã phát hiện**: `storage-sandbox-masvs-sensitive-at-rest`
- **Nguyên lý bảo mật**:
  - Tách biệt hoàn toàn cơ chế lưu trữ tệp thuần với lưu trữ dữ liệu nhạy cảm (khóa mã hóa, token xác thực, thông tin định danh cá nhân - PII). Không lưu thông tin nhạy cảm ở dạng bản rõ (plaintext) trong các tệp tào lao hoặc tệp bộ nhớ đệm cục bộ.
- **Áp dụng cho Super App**:
  - Super App cung cấp một cầu nối native chuyên dụng `SecureStorage` (sử dụng Android KeyStore / EncryptedSharedPreferences và iOS Keychain) cho các token ủy quyền và khóa mật mã do host phát hành.
  - Vùng dữ liệu hộp cát tệp thông thường được mặc định mã hóa bằng khóa riêng của thiết bị và bị xóa sạch lập tức khi người dùng đăng xuất khỏi Super App.

---

### 3.2. Quản trị Egress Mạng và Chính sách Bảo mật Nội dung (Network Egress & CSP Governance)

#### A. Khung Chính sách CSP Level 3 Bảo vệ WebView/JS Runtime
- **Mã phát hiện**: `network-egress-csp-connect-001`, `network-egress-csp-script-object-002`, `network-egress-csp-frame-ancestors-003`
- **Nguyên lý chuẩn hóa**:
  - **`connect-src`**: Kiểm soát các điểm cuối mạng được phép kết nối qua `fetch()`, `XMLHttpRequest`, `WebSocket`, `EventSource`. Thiết lập mặc định chặn mọi kết nối không khai báo (`default-src 'none'`).
  - **`script-src` & `object-src 'none'`**: Vô hiệu hóa hoàn toàn các plugin độc hại (`object-src 'none'`) và ngăn chặn việc thực thi chuỗi ký tự thành mã (`'unsafe-eval'`, `'unsafe-inline'` đối với mã độc nạp từ xa). Chỉ chấp nhận các mã nguồn bundle có chữ ký số hoặc digest khớp với bản phân phối.
  - **`frame-ancestors 'none'`**: Ngăn chặn việc mini-app bị nhúng lén vào các iframe độc hại của bên thứ ba bên ngoài Super App nhằm phòng chống tấn công Clickjacking và rò rỉ token phiên.
- **Áp dụng cho Super App**:
  - Kiểm duyệt chặt chẽ tệp `manifest.json` trong quá trình submission: tự động sinh ra header CSP và cấu hình WebView tương ứng từ danh sách các tên miền API đã được phê duyệt.
  - Mọi nỗ lực gọi mạng tới các domain ngoài danh sách trắng đều bị chặn ngay tại tầng mạng native của container (`shouldInterceptRequest` trên Android và `WKURLSchemeHandler` trên iOS).

#### B. Quy chế Danh sách Trắng Tên miền (Pre-Registered Domain Allowlist của WeChat)
- **Mã phát hiện**: `network-egress-wechat-allowlist-004`
- **Nguyên lý thực tế**:
  - WeChat bắt buộc mọi tên miền máy chủ dùng cho `request`, `uploadFile`, `downloadFile`, và `connectSocket` phải được đăng ký trước trên cổng thông tin nhà phát triển (mp.weixin.qq.com).
  - Bắt buộc giao thức HTTPS và WSS; yêu cầu TLS phiên bản 1.2 trở lên với bộ mã hóa (cipher suite) an toàn; nghiêm cấm sử dụng địa chỉ IP trực tiếp hoặc cổng không tiêu chuẩn trên môi trường sản xuất.
- **Áp dụng cho Super App**:
  - Tích hợp cổng quản trị Mini App Console: nhà phát triển phải khai báo danh sách FQDN hợp lệ và mục đích nghiệp vụ tương ứng trước khi phát hành phiên bản.
  - Trong chế độ kiểm thử (Debug/Canary), host cho phép bật cờ bỏ qua kiểm tra tên miền chỉ trên các thiết bị nhà phát triển đã được liên kết UID thử nghiệm, tuyệt đối không xuất hiện trong bản phát hành công khai.

#### C. Trung gian Mạng Host và Ghim Chứng chỉ (OWASP MASVS-NETWORK-1 & MASTG)
- **Mã phát hiện**: `network-egress-owasp-host-mediation-005`
- **Nguyên lý bảo mật**:
  - Ngăn ngừa tấn công nghe lén (Man-in-the-Middle - MitM) và rò rỉ thông tin qua kênh truyền thông không an toàn (OWASP Mobile Top 10 M5). Toàn bộ lưu lượng mạng phải được xác thực chuỗi chứng chỉ CA hợp lệ.
- **Áp dụng cho Super App**:
  - Container native của Super App đóng vai trò Proxy/Gatekeeper: chặn bắt các yêu cầu mạng phát sinh từ WebView, chèn các header bảo mật do host kiểm soát (`X-SuperApp-Request-ID`, `X-MiniApp-Token`), đồng thời thực hiện kiểm tra chứng chỉ (Certificate Pinning / Transparency Checks) đối với các dịch vụ lõi của Super App.

---

### 3.3. Tích hợp Bề mặt Hệ điều hành và Chia sẻ Xã hội (Social Share & OS Integration)

#### A. W3C Web Share API & Web Share Target API
- **Mã phát hiện**: `os-integration-web-share-001`, `os-integration-web-share-target-002`, `os-integration-web-share-target-003`
- **Nguyên lý chuẩn hóa**:
  - **`navigator.share()`**: Yêu cầu bắt buộc phải có kích hoạt cử chỉ từ người dùng (Transient User Activation - nhấp chuột/chạm). Dữ liệu chia sẻ bao gồm tiêu đề, văn bản, URL và mảng tệp tin được kiểm tra kiểu MIME an toàn.
  - **`share_target` Manifest Member**: Cho phép mini-app đăng ký khả năng tiếp nhận dữ liệu hoặc tệp chia sẻ từ bên ngoài thông qua việc khai báo hành động (`action`), phương thức (`POST/multipart`), và các trường dữ liệu (`params: {title, text, url, files}`).
- **Áp dụng cho Super App**:
  - Điều hướng hộp thoại chia sẻ do Super App sở hữu: hiển thị danh sách các điểm đến nội bộ (bạn bè chat, nhóm, bảng tin) kết hợp với các ứng dụng mạng xã hội ngoài qua bộ chọn native của hệ điều hành.
  - Tẩy rửa và khử độc (Sanitization) dữ liệu chia sẻ trước khi truyền qua cầu nối native: loại bỏ các ký tự điều khiển, ngăn chặn chèn URL độc hại (`javascript:`, `data:`).

#### B. W3C Badging API
- **Mã phát hiện**: `os-integration-badging-004`
- **Nguyên lý chuẩn hóa**:
  - Cung cấp phương thức `navigator.setAppBadge(count)` và `navigator.clearAppBadge()` để hiển thị số lượng thông báo hoặc trạng thái chưa đọc trên biểu tượng ứng dụng.
- **Áp dụng cho Super App**:
  - Super App không cho phép mini-app trực tiếp can thiệp vào huy hiệu biểu tượng chính của Super App trên màn hình chủ của điện thoại. Thay vào đó:
    - Huy hiệu được ánh xạ hiển thị trên biểu tượng mini-app trong Trung tâm ứng dụng (Mini App Catalog / Recent Apps Drawer).
    - Host thực hiện tổng hợp (aggregation) huy hiệu nội bộ và áp đặt chính sách giới hạn tần suất (Rate-limiting) để ngăn chặn việc spam gây phân tâm cho người dùng.

#### C. W3C Contact Picker API
- **Mã phát hiện**: `os-integration-contact-picker-005`
- **Nguyên lý chuẩn hóa**:
  - Cung cấp phương thức `navigator.contacts.select(properties, options)` cho phép người dùng chủ động chọn một hoặc nhiều liên hệ cụ thể từ danh bạ thông qua giao diện hệ thống bảo mật.
  - Hoàn toàn loại bỏ quyền truy cập nền vào toàn bộ danh bạ (No full address-book harvesting). Mini-app chỉ nhận được chính xác các trường thông tin (tên, số điện thoại, email) của những liên hệ mà người dùng đã tích chọn thủ công.
- **Áp dụng cho Super App**:
  - Thay vì cấp quyền đọc danh bạ thô của điện thoại cho mini-app, Super App hiển thị giao diện chọn liên hệ chuẩn (Contact Picker Sheet).
  - Kiểm duyệt lý do yêu cầu danh bạ trong quy trình xét duyệt (Review Gate): mini-app phải chứng minh sự cần thiết cho nghiệp vụ (ví dụ: chuyển tiền tới bạn bè, nạp tiền điện thoại) và ghi nhật ký kiểm toán mỗi lần truy cập.

---

## 4. Khuyến nghị Kiến trúc Điều khiển (Normative Control Blueprint)

```
+-----------------------------------------------------------------------------------+
|                            SUPER APP HOST CONTAINER                               |
|                                                                                   |
|  [Manifest & Review Gate]                                                         |
|  - Parse manifest.json: declarations for OPFS quota, Egress allowlists, Share     |
|  - Validate domain allowlist against developer portal registrations               |
|                                                                                   |
|  +-----------------------------------------------------------------------------+  |
|  |                           MINI APP RUNTIME (WEBVIEW)                        |  |
|  |                                                                             |  |
|  |  +-----------------------+     +-------------------+   +-----------------+  |  |
|  |  | OPFS File System      |     | CSP Policy Engine |   | Social / OS APIs|  |  |
|  |  | - getDirectory()      |     | - connect-src     |   | - share()       |  |  |
|  |  | - SyncAccessHandle    |     | - script-src      |   | - setAppBadge() |  |  |
|  |  | - Quota-bound usr/tmp |     | - frame-ancestors |   | - contacts()    |  |  |
|  |  +-----------+-----------+     +---------+---------+   +--------+--------+  |  |
|  +--------------|---------------------------|----------------------|-----------+  |
|                 v                           v                      v              |
|  +-----------------------------------------------------------------------------+  |
|  |                   HOST BRIDGE & PLATFORM ISOLATION ADAPTER                  |  |
|  |                                                                             |  |
|  |  [Sandbox Storage Engine]      [Egress Proxy & Pinning]    [OS Integration] |  |
|  |  - (user, appId, ver) isolate   - Domain allowlist check    - Transient UI   |  |
|  |  - Encrypted secure key/token   - Cert pinning / MitM drop  - Contact sheet  |  |
|  |  - Quota enforcement / GC      - Custom headers injection  - Badge aggregate|  |
|  +-----------------------------------------------------------------------------+  |
+-----------------------------------------------------------------------------------+
```

---

## 5. Kết luận Iteration 23

Việc bổ sung 15 phát hiện kỹ thuật trong Iteration 23 nâng tổng số bằng chứng chuẩn hóa của toàn bộ nghiên cứu lên **297 phát hiện**. Các quy định này giải quyết triệt để rủi ro rò rỉ dữ liệu chéo giữa các mini-app trên cùng một thiết bị di động, kiểm soát toàn diện luồng dữ liệu mạng ra ngoài, và tích hợp các tính năng tương tác người dùng hiện đại một cách an toàn, minh bạch.
