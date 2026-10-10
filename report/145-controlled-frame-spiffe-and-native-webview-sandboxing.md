# Chuyên đề 145: WICG Controlled Frame & IWA, CNCF SPIFFE Workload Identity, và Native Platform In-App Web Sandboxing (Android Custom Tabs / TWA & Apple App-Bound Domains)

## 1. Giới thiệu & Bối cảnh Tiêu chuẩn hóa

Trong kiến trúc Super App hiện đại, việc lưu trữ, thực thi và cô lập hàng trăm Mini App từ các nhà phát triển thứ ba (third-party publishers) đặt ra những thách thức bảo mật mang tính sống còn:
1. **Container Webview Containment**: Làm thế nào để nhúng ứng dụng web của bên thứ ba mà không bị ô nhiễm không gian tên DOM (namespace pollution), không bị rò rỉ session storage, và kiểm soát tuyệt đối quyền truy cập phần cứng/API nhạy cảm?
2. **Microservice Workload Identity**: Làm thế nào để các dịch vụ backend phục vụ Mini App xác thực lẫn nhau trong môi trường đa cụm (multi-tenant / multi-cloud) mà không cần lưu trữ static secret hay API token tĩnh trong file cấu hình?
3. **Native OS-Level In-App Web Sandboxing & Deep Link Integrity**: Khi mở các luồng web bên ngoài (in-app browsing) hoặc kích hoạt Mini App thông qua deep link, làm sao để ngăn chặn các ứng dụng độc hại trên thiết bị đánh cắp intent/callback hoặc theo dõi hành vi lướt web của người dùng?

Milestone 145 mở rộng bộ tiêu chuẩn Super App Mini App Store Standard với 15 phát hiện kỹ thuật chuẩn hóa, tập trung vào 3 trụ cột tiêu chuẩn:
- **WICG Controlled Frame API & Isolated Web Apps (IWA)**: Chuẩn hóa thẻ `<controlledframe>` thay thế cho các kiến trúc webview cũ, cung cấp mô hình duyệt top-level cô lập, ủy quyền quyền hạn thông qua sự kiện `permissionrequest`, chèn content script trong isolated world, và phân vùng storage/cookie độc lập.
- **CNCF SPIFFE Standard (Secure Production Identity Framework for Everyone)**: Chuẩn hóa định danh mật mã học `spiffe://` URI, chứng chỉ số X.509-SVID tự động xoay vòng cho mTLS, token JWT-SVID với xác thực audience chặt chẽ, Workload API giao tiếp qua Unix Domain Socket triệt tiêu hoàn toàn static secret dựa trên kernel attestation, và SPIFFE Federation liên kết đa đối tác B2B.
- **Native Platform In-App Web Sandboxing & Deep Linking**: Kiến trúc Android Custom Tabs với cơ chế warm-up `mayLaunchUrl()` và callback telemetry, Android Trusted Web Activities (TWA) với xác thực Digital Asset Links toàn màn hình, Apple WebKit App-Bound Domains (`WKAppBoundDomains`) giới hạn 10 tên miền và cô lập out-of-process, cùng cơ chế xác thực nguồn gốc mật mã học chống cướp link (Android App Links & Apple Universal Links AASA).

---

## 2. Chi tiết 15 Phát hiện Chuẩn hóa (Normative Findings)

### Nhóm 1: WICG Controlled Frame API & Isolated Web Apps (IWA) Sandboxing

#### Finding 1: WICG Controlled Frame Element Model & Isolated Web App (IWA) Top-Level Browsing Context
- **Mã định danh**: `controlled_frame_145_1`
- **Tiêu chuẩn**: WICG Controlled Frame API Specification (Draft Community Group Report)
- **Nguồn xác thực**: https://wicg.github.io/controlled-frame/ (HTTP 200)
- **Mức độ bằng chứng**: `authoritative_specification`
- **Chi tiết kỹ thuật**: Đặc tả `<controlledframe>` định nghĩa một phần tử HTML chuyên dụng để nhúng nội dung web tùy ý của bên thứ ba bên trong môi trường ứng dụng web cô lập (Isolated Web Application - IWA) hoặc native container an toàn. Không giống như thẻ `<iframe>` thông thường vốn có thể gây rò rỉ ngữ cảnh duyệt hoặc bị tấn công prototype tampering, nội dung chạy bên trong `<controlledframe>` hoạt động trong một browsing context cấp cao nhất (top-level browsing context) hoàn toàn độc lập do chính frame cha kiểm soát. Thẻ hỗ trợ các thuộc tính khai báo bao gồm `src` và `partition` (chỉ định vùng lưu trữ tách biệt). Host container thực thi giám sát vòng đời chặt chẽ thông qua các sự kiện xác định: `loadstart`, `loadcommit`, `loadstop`, `loadabort`, và `loadredirect`. Điều này cho phép Super App theo dõi chính xác từng giai đoạn điều hướng mà không cho phép frame khách truy cập hoặc thăm dò cây DOM của host.
- **Tác động Super App**: Super App có thể thay thế các bridge wrapper tự chế tiềm ẩn lỗ hổng bảo mật bằng thẻ chuẩn hóa `<controlledframe>`, đảm bảo cách ly tiến trình vật lý và triệt tiêu nguy cơ frame con thao túng host application.

#### Finding 2: Granular Permission Mediation & permissionrequest Lifecycle Interception
- **Mã định danh**: `controlled_frame_145_2`
- **Tiêu chuẩn**: WICG Controlled Frame API Specification Section 3.7.4 (permissionrequest Event)
- **Nguồn xác thực**: https://wicg.github.io/controlled-frame/ (HTTP 200)
- **Mức độ bằng chứng**: `authoritative_specification`
- **Chi tiết kỹ thuật**: Trong mô hình Controlled Frame, bất kỳ yêu cầu truy cập API thiết bị nhạy cảm nào (định vị GPS, camera, microphone, thông báo push, pointer lock) phát sinh từ Mini App đều kích hoạt sự kiện đồng bộ `permissionrequest` trên phần tử `<controlledframe>` của host. Đối tượng event cung cấp đầy đủ siêu dữ liệu của yêu cầu và phơi bày trực tiếp hai phương thức `request.allow()` và `request.deny()`. Cơ chế này ngăn chặn tuyệt đối việc frame con tự ý kích hoạt các hộp thoại xin quyền trình duyệt tới người dùng; thay vào đó, Super App chặn bắt toàn bộ yêu cầu, đối chiếu với ma trận phân quyền đã cấp duyệt trên Store, kiểm tra hạn mức sử dụng (rate limit / quota), và quyết định cho phép hoặc từ chối theo chính sách bảo mật Zero Trust.
- **Tác động Super App**: Bắt buộc mọi yêu cầu truy cập phần cứng và cảm biến của Mini App phải đi qua lớp trung gian kiểm soát của Super App, ngăn chặn hành vi lén lút kích hoạt micro hoặc theo dõi vị trí nền.

#### Finding 3: Isolated Script Injection & Content Script Sandboxing via executeScript and addContentScripts
- **Mã định danh**: `controlled_frame_145_3`
- **Tiêu chuẩn**: WICG Controlled Frame API Specification Section 3.4 (Scripting methods)
- **Nguồn xác thực**: https://wicg.github.io/controlled-frame/ (HTTP 200)
- **Mức độ bằng chứng**: `authoritative_specification`
- **Chi tiết kỹ thuật**: Controlled Frame cung cấp giao diện API có cấu trúc chặt chẽ để tiêm mã JavaScript và CSS vào ngữ cảnh frame con: `addContentScripts(sequence<ContentScriptDetails>)`, `removeContentScripts()`, `executeScript(optional InjectDetails)`, và `insertCSS(optional InjectDetails)`. Thay vì thực thi chuỗi eval nguy hiểm hoặc lắng nghe sự kiện `postMessage` không xác thực, `ContentScriptDetails` cho phép khai báo URL pattern đối soát, thời điểm thực thi chính xác (`document_start`, `document_end`, `document_idle`), và đặc biệt là cơ chế thế giới thực thi cô lập (isolated execution worlds). Nhờ đó, các script SDK và API bridge do Super App tiêm vào chạy trong một ngữ cảnh JS biệt lập, hoàn toàn miễn nhiễm trước việc bị Mini App ghi đè nguyên mẫu (prototype pollution), can thiệp Object prototype, hoặc đánh chặn cuộc gọi.
- **Tác động Super App**: Chuẩn hóa việc phân phối JS-SDK của Super App vào môi trường Mini App thông qua isolated worlds, đảm bảo tính toàn vẹn của runtime bridge mà không lo bị mã độc sửa đổi runtime methods.

#### Finding 4: Storage Partitioning & Ephemeral Session Governance via Partition Attribute
- **Mã định danh**: `controlled_frame_145_4`
- **Tiêu chuẩn**: WICG Controlled Frame API Specification Section 3.2 (Attributes: partition)
- **Nguồn xác thực**: https://wicg.github.io/controlled-frame/ (HTTP 200)
- **Mức độ bằng chứng**: `authoritative_specification`
- **Chi tiết kỹ thuật**: Thuộc tính `partition` trên thẻ `<controlledframe>` phân định ranh giới lưu trữ dữ liệu hoàn toàn độc lập cho phiên làm việc của Mini App, bao gồm Cookies, IndexedDB, CacheStorage, và LocalStorage. Vùng phân vùng này có thể được cấu hình lưu trữ lâu dài theo tên bucket (ví dụ: `persist:miniapp_vendor_a`) hoặc cấu hình ở chế độ tạm thời (ephemeral - chỉ lưu trong RAM và tự động hủy ngay khi đóng thẻ). Kiến trúc này ngăn chặn hoàn toàn việc theo dõi người dùng chéo giữa các Mini App (cross-miniapp tracking), tránh rò rỉ session token giữa các phiên đăng nhập, và loại bỏ nguy cơ một Mini App độc hại ghi tràn bộ nhớ làm cạn kiệt dung lượng lưu trữ của Super App chính.
- **Tác động Super App**: Thiết lập cơ chế cô lập kho lưu trữ theo từng Mini App, cho phép xóa sạch toàn bộ dữ liệu tạm thời ngay khi gỡ cài đặt hoặc đăng xuất Mini App mà không làm ảnh hưởng đến dữ liệu của các dịch vụ khác.

#### Finding 5: Navigation Control, New-Window Policy & Chromium Architectural Lineage
- **Mã định danh**: `controlled_frame_145_5`
- **Tiêu chuẩn**: WICG Controlled Frame API Specification & Chrome Platform Status #5137024755105792
- **Nguồn xác thực**: https://chromestatus.com/feature/5137024755105792 (HTTP 200)
- **Mức độ bằng chứng**: `authoritative_specification`
- **Chi tiết kỹ thuật**: Kế thừa những ưu điểm của thẻ `<webview>` trong Chrome Apps và kết hợp triết lý Fenced Frame hiện đại, Controlled Frame thay thế các API webview lỗi thời vốn không an toàn bằng một cấu trúc web chuẩn hóa cho các ứng dụng web cô lập cấp doanh nghiệp. Nó cung cấp các phương thức điều hướng tất định (`back()`, `forward()`, `reload()`, `stop()`) và bẫy chặn mọi hành vi mở cửa sổ mới thông qua sự kiện `newwindow`. Super App container có thể kiểm tra URL đích đối chiếu với danh sách cho phép (allowlist), ngăn chặn tuyệt đối hành vi chuyển hướng người dùng sang các trang web lừa đảo hoặc mã độc bên ngoài.
- **Tác động Super App**: Cung cấp lộ trình chuẩn hóa web-platform cho các runtime container của Super App trên máy tính để bàn (Desktop PWA) và các thiết bị nhúng, loại bỏ rủi ro bảo mật của webview truyền thống trong khi vẫn giữ vững quyền kiểm soát kho ứng dụng.

---

### Nhóm 2: CNCF SPIFFE Standard & Cryptographic Workload Identity

#### Finding 6: SPIFFE ID Standard URI Scheme & Multi-Tenant Super-App Trust Domains
- **Mã định danh**: `spiffe_identity_145_6`
- **Tiêu chuẩn**: CNCF SPIFFE Identity and Verifiable Identity Document (SPIFFE-ID Standard)
- **Nguồn xác thực**: https://github.com/spiffe/spiffe/blob/main/standards/SPIFFE-ID.md (HTTP 200)
- **Mức độ bằng chứng**: `authoritative_specification`
- **Chi tiết kỹ thuật**: Tiêu chuẩn SPIFFE định nghĩa lược đồ định danh mật mã học đồng nhất dưới dạng URI: `spiffe://<trust-domain>/<workload-path>`. Trong đó, `trust-domain` đại diện cho một cơ quan quản trị có thẩm quyền (ví dụ: `superapp.internal` hoặc `partner.fintech`), còn các đường dẫn phân cấp xác định chính xác dịch vụ thực thi (ví dụ: `/miniapp/banking/core-payment` hoặc `/store/catalog-indexer`). SPIFFE quy định nghiêm ngặt rằng tên trust domain phải là chữ thường theo chuẩn DNS, và đường dẫn workload tuân thủ cú pháp URI không chứa chuỗi truy vấn (query string) hay fragment. Ranh giới tin cậy được thực thi nghiêm ngặt: một tiến trình chỉ có thể nhận thông tin xác thực cho định danh thuộc trust domain cục bộ của nó, trừ khi có cấu hình liên kết federation rõ ràng.
- **Tác động Super App**: Thiết lập hệ thống phân loại định danh chuẩn hóa, không phụ thuộc nhà cung cấp cho toàn bộ vi dịch vụ nội bộ Super App, cổng API gateway, và backend của các đối tác Mini App, xóa bỏ hoàn toàn việc sử dụng API key tĩnh hoặc tên dịch vụ mơ hồ.

#### Finding 7: X.509-SVID Standard: SAN URI Extension & Automated Ephemeral mTLS Rotation
- **Mã định danh**: `spiffe_identity_145_7`
- **Tiêu chuẩn**: CNCF SPIFFE X.509-SVID Standard Specification
- **Nguồn xác thực**: https://github.com/spiffe/spiffe/blob/main/standards/X509-SVID.md (HTTP 200)
- **Mức độ bằng chứng**: `authoritative_specification`
- **Chi tiết kỹ thuật**: Chứng chỉ X.509-SVID (SPIFFE Verifiable Identity Document) là chứng chỉ số tuân thủ chuẩn ITU-T X.509, đóng gói định danh SPIFFE ID bên trong phần mở rộng Subject Alternative Name (SAN) kiểu `uniformResourceIdentifier` (URI). Để giảm thiểu rủi ro lộ khóa riêng trong môi trường container biến động liên tục, SPIFFE quy định X.509-SVID phải có thời gian sống rất ngắn (short-lived, thông thường từ 1 giờ đến 24 giờ) và được SPIFFE agent cục bộ tự động gia hạn trước khi hết hạn. Các vi dịch vụ sử dụng trực tiếp các chứng chỉ này để thiết lập kết nối mutual TLS (mTLS), đảm bảo mã hóa đường truyền, xác thực hai chiều chống mạo danh, và tự động thu hồi mà không cần dựa vào hạ tầng CRL/OCSP chậm chạp và dễ gãy vỡ.
- **Tác động Super App**: Cung cấp khả năng mã hóa Zero Trust trên toàn bộ lớp microservice mesh của Super App, ngăn chặn nghe lén và tấn công Man-in-the-Middle giữa backend của Mini App và sổ cái thanh toán tài chính.

#### Finding 8: JWT-SVID Standard: Compact Cryptographic Claims for REST/gRPC Gateway Ingress
- **Mã định danh**: `spiffe_identity_145_8`
- **Tiêu chuẩn**: CNCF SPIFFE JWT-SVID Standard Specification
- **Nguồn xác thực**: https://github.com/spiffe/spiffe/blob/main/standards/JWT-SVID.md (HTTP 200)
- **Mức độ bằng chứng**: `authoritative_specification`
- **Chi tiết kỹ thuật**: Tiêu chuẩn JWT-SVID định nghĩa profile JSON Web Token để chứng thực workload qua các proxy trung gian tầng ứng dụng (Layer 7) và hàng đợi thông điệp bất đồng bộ - nơi kết nối mTLS bị chấm dứt (terminated). JWT-SVID mã hóa SPIFFE ID trong claim `sub` (subject) và bắt buộc phải có claim `aud` (audience) chỉ định chính xác dịch vụ nhận, ngăn chặn việc tái sử dụng token để gọi các dịch vụ khác (replay attacks). Tiêu đề JOSE phải sử dụng chữ ký số bất đối xứng (ES256, RS256, Ed25519) và nghiêm cấm thuật toán `none` hoặc khóa đối xứng HMAC. Thời gian hết hạn (`exp`) bị giới hạn trong khoảng thời gian rất ngắn nhằm giảm thiểu nguy cơ khi token bị rò rỉ tạm thời.
- **Tác động Super App**: Cung cấp định danh mật mã học nhỏ gọn, an toàn truyền tải trong tiêu đề HTTP `Authorization: Bearer` từ cổng reverse proxy tới các endpoint backend của Mini App, đảm bảo xác thực người gọi không thể chối bỏ.

#### Finding 9: SPIFFE Workload API & Kernel-Level Process Attestation without Static Secrets
- **Mã định danh**: `spiffe_identity_145_9`
- **Tiêu chuẩn**: CNCF SPIFFE Workload API Standard Specification
- **Nguồn xác thực**: https://github.com/spiffe/spiffe/blob/main/standards/SPIFFE_Workload_API.md (HTTP 200)
- **Mức độ bằng chứng**: `authoritative_specification`
- **Chi tiết kỹ thuật**: SPIFFE Workload API cung cấp giao diện IPC/gRPC cục bộ thông qua Unix Domain Socket (UDS) (hoặc Windows Named Pipe), loại bỏ hoàn toàn việc lưu trữ mật khẩu, khóa bí mật tĩnh và API token trong mã nguồn, file cấu hình, hay biến môi trường container. Khi một dịch vụ yêu cầu cấp phát SVID, SPIFFE agent cục bộ sẽ truy vấn nhân hệ điều hành (kernel) để chứng thực các thuộc tính của tiến trình gọi (Linux PID, UID, GID, cgroup, container runtime ID, và Kubernetes service account/namespace). Chứng chỉ SVID và bundle khóa công khai gốc được trả về trực tiếp trong bộ nhớ RAM và được truyền phát liên tục qua kênh gRPC có độ trễ cực thấp.
- **Tác động Super App**: Loại bỏ nguy cơ rò rỉ thông tin đăng nhập trong các hàm serverless và container xử lý tác vụ của Mini App bằng cách chuyển đổi việc cấp phát định danh sang cơ chế chứng thực môi trường do kernel bảo vệ.

#### Finding 10: SPIFFE Federation: Cryptographic Multi-Cloud & Cross-Partner Trust Bundles
- **Mã định danh**: `spiffe_identity_145_10`
- **Tiêu chuẩn**: CNCF SPIFFE Federation Standard Specification
- **Nguồn xác thực**: https://github.com/spiffe/spiffe/blob/main/standards/SPIFFE_Federation.md (HTTP 200)
- **Mức độ bằng chứng**: `authoritative_specification`
- **Chi tiết kỹ thuật**: SPIFFE Federation cho phép các workload thuộc các trust domain độc lập, do các tổ chức khác nhau quản lý có thể xác thực và thiết lập kênh kết nối an toàn mTLS mà không cần dùng chung một Certificate Authority (CA) gốc. Theo đặc tả, mỗi trust domain xuất bản một điểm cuối SPIFFE Bundle an toàn qua HTTPS, phơi bày các khóa công khai và chứng chỉ CA gốc hiện tại (định dạng JWKS hoặc RFC 8410). SPIFFE agent cục bộ định kỳ đồng bộ các trust bundle liên kết này, cho phép các dịch vụ thuộc lõi Super App (`spiffe://superapp.com`) kiểm tra tính hợp lệ của X.509-SVID hoặc JWT-SVID gửi đến từ hệ thống backend của đối tác ngân hàng/viễn thông (`spiffe://partnerbank.vn`) một cách tự động.
- **Tác động Super App**: Giải quyết bài toán liên kết B2B đa đối tác cho các Mini App của bên thứ ba, cho phép tích hợp Zero Trust an toàn giữa nền tảng Super App viễn thông và các nhà cung cấp dịch vụ doanh nghiệp mà không cần quản lý chéo chứng chỉ PKI phức tạp.

---

### Nhóm 3: Native Platform In-App Web Sandboxing & Deep Link Integrity

#### Finding 11: Android Custom Tabs Architecture, Session Pre-Warming & Bi-Directional Telemetry
- **Mã định danh**: `native_webview_145_11`
- **Tiêu chuẩn**: AndroidX Browser Custom Tabs API Specification (CustomTabsClient & CustomTabsIntent)
- **Nguồn xác thực**: https://developer.android.com/reference/androidx/browser/customtabs/CustomTabsIntent (HTTP 200)
- **Mức độ bằng chứng**: `authoritative_specification`
- **Chi tiết kỹ thuật**: Android Custom Tabs cung cấp trải nghiệm duyệt web được hệ điều hành làm trung gian, trong đó Super App hiển thị nội dung web do chính trình duyệt mặc định của hệ thống dựng hình thay vì sử dụng WebView nhúng nội bộ. Lớp `CustomTabsClient` hỗ trợ làm ấm trước tiến trình trình duyệt ở chế độ nền (`warmup()`) và tải trước URL của Mini App (`mayLaunchUrl()`), giúp triệt tiêu độ trễ khởi động xuống gần như bằng 0. Giao diện `CustomTabsCallback` gửi về Super App các thông tin viễn trắc vòng đời định hướng (`NAVIGATION_STARTED`, `NAVIGATION_FINISHED`, `NAVIGATION_FAILED`, `NAVIGATION_ABORTED`), nhưng vẫn giữ nguyên sự cách ly tiến trình tuyệt đối: Super App hoàn toàn không thể đọc trộm hay sửa đổi cây DOM, cookies, hoặc phím bấm của người dùng trên trang web khách.
- **Tác động Super App**: Cho phép Super App đạt hiệu năng khởi chạy tức thì tương đương ứng dụng native đối với các cổng thông tin và Mini App dạng web bên ngoài, đồng thời tuân thủ hoàn hảo các chính sách bảo vệ quyền riêng tư của Google Play Store.

#### Finding 12: Android Trusted Web Activities (TWA) & Digital Asset Links Cryptographic Origin Binding
- **Mã định danh**: `native_webview_145_12`
- **Tiêu chuẩn**: AndroidX Browser Trusted Web Activity Specification (TrustedWebActivityIntentBuilder)
- **Nguồn xác thực**: https://developer.android.com/reference/androidx/browser/trusted/TrustedWebActivityIntentBuilder (HTTP 200)
- **Mức độ bằng chứng**: `authoritative_specification`
- **Chi tiết kỹ thuật**: Trusted Web Activities (TWA) mở rộng Custom Tabs bằng cách thiết lập liên kết xác thực bằng mật mã giữa gói ứng dụng Android (APK) và tên miền web thông qua chuẩn Digital Asset Links (`assetlinks.json`). Khi chữ ký vân tay SHA-256 của chứng chỉ ký APK khớp với khai báo tại `https://domain/.well-known/assetlinks.json`, trình duyệt hệ điều hành sẽ ẩn toàn bộ thanh địa chỉ và các nút điều hướng của trình duyệt, hiển thị Mini App dưới dạng ứng dụng native toàn màn hình chân thực. TWA chia sẻ kho cookie, passkey WebAuthn, và dữ liệu IndexedDB trực tiếp với trình duyệt chính của người dùng mà không để lộ dữ liệu nhạy cảm này cho mã nguồn APK của host.
- **Tác động Super App**: Giúp Super App và các đối tác Mini App phát hành ứng dụng web với giao diện native mượt mà, không có thanh URL gây vướng víu, đồng thời bảo đảm chứng thực quyền sở hữu tên miền bằng mật mã học.

#### Finding 13: Apple WebKit App-Bound Domains Architecture & In-App Browsing Privacy Boundaries
- **Mã định danh**: `native_webview_145_13`
- **Tiêu chuẩn**: WebKit App-Bound Domains Specification & WKWebViewConfiguration.limitsNavigationsToAppBoundDomains
- **Nguồn xác thực**: https://webkit.org/blog/10882/app-bound-domains/ (HTTP 200)
- **Mức độ bằng chứng**: `authoritative_specification`
- **Chi tiết kỹ thuật**: Được giới thiệu trong iOS 14 / macOS Big Sur, kiến trúc WebKit App-Bound Domains giới hạn các phiên bản `WKWebView` chỉ được phép tương tác sâu với danh sách tối đa 10 tên miền cốt lõi được khai báo trước trong tệp `Info.plist` dưới khóa `WKAppBoundDomains`. Khi cờ `limitsNavigationsToAppBoundDomains = YES` được kích hoạt trên `WKWebViewConfiguration`, WebKit sẽ tự động vô hiệu hóa các phương thức tiêm script nhạy cảm (`evaluateJavaScript`), các bộ xử lý message handler, và quyền truy cập cookie đối với bất kỳ điều hướng nào ra ngoài 10 tên miền đã đăng ký. Khi người dùng điều hướng ra liên kết bên ngoài, webview tự động chuyển sang chế độ duyệt ngoài tiến trình (out-of-process isolation), ngăn chặn host app theo dõi lịch sử duyệt web, chụp phím bấm, hoặc đánh chặn lưu lượng mạng.
- **Tác động Super App**: Đảm bảo tuân thủ nghiêm ngặt Hướng dẫn Đánh giá App Store của Apple (Mục 2.5.6), khẳng định Super App không thực hiện hành vi theo dõi lén lút người dùng hoặc đánh cắp session token của bên thứ ba khi duyệt web in-app.

#### Finding 14: Android App Links Cryptographic Site Association & Deep Link Anti-Hijacking
- **Mã định danh**: `native_webview_145_14`
- **Tiêu chuẩn**: Android Developers App Links Architecture & Asset Links Verification Protocol
- **Nguồn xác thực**: https://developer.android.com/training/app-links/verify-site-associations (HTTP 200)
- **Mức độ bằng chứng**: `authoritative_specification`
- **Chi tiết kỹ thuật**: Android App Links nâng cấp các deep link sử dụng lược đồ tùy biến (custom URI scheme như `myapp://`) lên các liên kết HTTPS chuẩn mực (như `https://superapp.com/miniapp/flight`). Điểm yếu chết người của custom scheme là bất kỳ ứng dụng độc hại nào trên máy cũng có thể đăng ký cùng scheme để đánh chặn dữ liệu. Android App Links bắt buộc phải trải qua quy trình xác thực máy chủ tự động bằng Digital Asset Links (`assetlinks.json`). Trong quá trình cài đặt hoặc cập nhật ứng dụng, hệ điều hành Android sẽ tải tệp `assetlinks.json` qua HTTPS, đối chiếu vân tay chứng chỉ ký của ứng dụng với khai báo trên máy chủ, và tự động mở ứng dụng đã xác thực mà không hiển thị hộp thoại hỏi người dùng (disambiguation dialog).
- **Tác động Super App**: Ngăn chặn tuyệt đối các ứng dụng giả mạo trên thiết bị đánh chặn Mini App deep links, callback thanh toán ngân hàng, và mã xác thực OTP dùng một lần, đảm bảo định tuyến an toàn tuyệt đối trên Android.

#### Finding 15: Apple Universal Links & apple-app-site-association (AASA) Native Routing Integrity
- **Mã định danh**: `native_webview_145_15`
- **Tiêu chuẩn**: Apple Developer Universal Links Architecture & Shared Web Credentials Protocol
- **Nguồn xác thực**: https://developer.apple.com/documentation/xcode/supporting-universal-links-in-your-app (HTTP 200)
- **Mức độ bằng chứng**: `authoritative_specification`
- **Chi tiết kỹ thuật**: Apple Universal Links thiết lập các liên kết HTTPS chuẩn mực có khả năng mở trực tiếp bên trong ứng dụng native trên iOS, và tự động chuyển hướng mượt mà về trang web Safari nếu ứng dụng chưa được cài đặt. Để chống cướp liên kết, iOS yêu cầu tệp `apple-app-site-association` (AASA) định dạng JSON phải được đặt tại `https://domain/.well-known/apple-app-site-association` qua kết nối TLS hợp lệ. Tệp AASA định nghĩa Application Identifier (`TeamID.BundleID`) và mẫu đường dẫn URL được phép định tuyến. Universal Links được hệ điều hành xử lý ở tầng native không thông qua chuyển hướng HTTP (redirects), đảm bảo token xác thực, callback giao dịch, và tham số khởi động Mini App không thể bị các tiện ích mở rộng Safari hoặc URL handler giả mạo đọc lén.
- **Tác động Super App**: Mang lại trải nghiệm kích hoạt Mini App không gián đoạn trên iOS, bảo vệ các luồng thanh toán và tiếp thị liên kết chéo giữa các Mini App với tính toàn vẹn mật mã học cao nhất.

---

## 3. Ma trận Đối soát & Hướng dẫn Thực thi Kiến trúc

| Chiều kiểm soát | WICG Controlled Frame (IWA) | CNCF SPIFFE Workload Identity | Native In-App Web (TWA / App-Bound) |
|---|---|---|---|
| **Môi trường thực thi** | Container webview cô lập top-level, hỗ trợ Desktop PWA | Microservice backend mesh, container Kubernetes, serverless | Webview native trên Android (TWA) và iOS (WKWebView) |
| **Cơ chế xác thực / Phân quyền** | Sự kiện `permissionrequest` cho phép host duyệt/từ chối theo chính sách | X.509-SVID (mTLS) và JWT-SVID (`aud` validation), chứng thực qua kernel | Digital Asset Links (`assetlinks.json`) và Apple AASA manifest |
| **Cô lập dữ liệu / Lưu trữ** | Thuộc tính `partition` tạo session ephemeral hoặc phân vùng tách biệt | Không lưu static secret, chứng chỉ lưu trong bộ nhớ và xoay vòng định kỳ | Trình duyệt quản lý riêng biệt; WebKit App-Bound ngăn truy cập cookie ngoài phạm vi |
| **Bảo vệ chống thao túng / Can thiệp** | Script được tiêm qua isolated execution worlds (chống prototype pollution) | Xác thực danh tính caller qua Linux PID/UID/cgroups qua Workload API | Chặn cướp link (anti-hijacking) ở tầng OS, ngăn host app đọc lén DOM bên ngoài |
| **Quy chuẩn Store Compliance** | Đạt chuẩn Isolated Web App, kiểm soát điều hướng qua `newwindow` | Đáp ứng kiểm toán Zero Trust đa đám mây, B2B federated trust | Tuân thủ Google Play Privacy Policy & Apple Store Review Guideline 2.5.6 |

---

## 4. Kết luận

Với việc tích hợp Milestone 145, tiêu chuẩn Mini App Store cho Super App đã hoàn thiện thêm một tầng bảo mật then chốt: liên kết chặt chẽ giữa **cô lập container tại client** (Controlled Frame & Native Sandboxing) và **chứng thực danh tính mật mã học tại backend** (SPIFFE/SPIRE). Tổng số phát hiện kỹ thuật đã xác thực đạt **2.126 phát hiện**, sẵn sàng cho việc đồng bộ và xuất bản lên repository công khai.
