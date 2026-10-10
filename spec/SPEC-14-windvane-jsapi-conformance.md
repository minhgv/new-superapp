# SPEC-14 — Gắn kết WindVane / JSAPI Conformance

| Trường | Giá trị |
|---|---|
| Spec ID | `SPEC-14` |
| Phiên bản | 1.0.0 |
| Trạng thái | Draft |
| Lớp tuân thủ | **C1 + C2 + C3** |
| Nguồn | `report/01`, `report/06`, tài liệu Alibaba Cloud (URL ở mục 7) |

---

## 1. Phạm vi & mục tiêu

Tài liệu này đặc tả **tầng gắn kết tuân thủ (conformance binding)** giữa chuẩn kiến trúc trung lập của bộ SPEC Super App (`SPEC-00` … `SPEC-13`) và nền tảng Alibaba Cloud SuperApp / WindVane POC (EMAS).

Mục tiêu cốt lõi:
- Lấp đầy khoảng trống nghiên cứu đã ghi nhận tại `report/06`: xác định hợp đồng tích hợp native container (Android/iOS SDK), danh mục phân quyền JSAPI, cấu trúc đóng gói và luồng phát hành.
- Chuẩn hóa bảng phân loại quyền (taxonomy) cho 22 lớp WindVane JSBridge API ánh xạ sang mô hình năng lực thiết bị tại `SPEC-06`.
- Thiết lập ranh giới tường minh giữa các đặc tính kỹ thuật đã xác thực từ tài liệu chính thức của hãng và các đề xuất kỹ thuật (`[PROPOSAL]`) cần kiểm chứng thực nghiệm trên thiết bị thật.

---

## 2. Thuật ngữ

| Thuật ngữ | Định nghĩa |
|---|---|
| **EMAS** | Enterprise Mobile Application Studio — nền tảng điện toán di động của Alibaba Cloud cung cấp giải pháp SuperApp. |
| **Application Open Platform** | Control plane của Alibaba Cloud SuperApp quản lý đăng ký ứng dụng, phiên bản và các kênh phát hành. |
| **WindVane Container** | Môi trường runtime nhúng trong ứng dụng native (Android/iOS) chịu trách nhiệm tải, render và cách ly mini-app. |
| **uni-app Container** | Container thay thế do EMAS hỗ trợ cho các mini-app phát triển trên framework uni-app đa nền tảng. |
| **WindVane JSBridge** | Kênh giao tiếp hai chiều giữa Web context trong container và năng lực native của host (`android.taobao.windvane.jsbridge.api.*`). |
| **SuperApp Miniapp Develop Tool** | Tiện ích mở rộng chính thức trên VS Code dùng để khởi tạo dự án, kiểm thử và nén gói phát hành. |
| **Whitelisting Test** | Chế độ thử nghiệm nội bộ giới hạn theo danh sách User ID được cấp phép trước khi mở rộng. |
| **Canary Release** | Phát hành theo tỷ lệ phần trăm người dùng tăng dần nhằm thẩm định độ ổn định trước khi phát hành chính thức. |

---

## 3. Yêu cầu chuẩn hóa

### 3.1 Tích hợp Host SDK & Khởi tạo Container (Android / iOS)

> **REQ-14-001** (MUST · C1) — Host Android **PHẢI** tích hợp core SDK và WindVane SDK (`com.aliyun.emas.suite.foundation:windvane-mini-app:1.4.0`) từ kho Maven private của Apsara Stack được chỉ định.
> *Nguồn:* `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/android-access-1` · *Kiểm chứng:* static

> **REQ-14-002** (MUST · C1) — Host Android **PHẢI** khởi tạo cấu hình qua `MiniAppInitConfig.Builder()`, bật cờ `setUseWindVane(true)` (hoặc `setUseUniApp(true)` khi dùng uni-app) và cung cấp đủ các tham số cấu hình: `accessKey`, `secretKey`, `host`, `appCode` từ Application Open Platform.
> *Nguồn:* `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/android-access-1` · *Kiểm chứng:* runtime

> **REQ-14-003** (MUST · C1) — Host Android **PHẢI** gọi `IMiniAppService.initialize(application, config)` và đăng ký `MiniAppService` vào `ServiceManager` của ứng dụng trước khi xử lý bất kỳ yêu cầu mở mini-app nào.
> *Nguồn:* `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/android-access-1` · *Kiểm chứng:* runtime

> **REQ-14-004** (MUST · C1) — Quy tắc làm mờ mã nguồn (R8/ProGuard) của Host Android **PHẢI** giữ nguyên (keep) toàn bộ namespace `android.taobao.windvane.jsbridge.api.*` để bảo đảm cơ chế phản chiếu (reflection) của JSBridge hoạt động chính xác.
> *Nguồn:* `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/android-access-1` · *Kiểm chứng:* static

> **REQ-14-005** (MUST · C1) — Host iOS **PHẢI** tích hợp `EMASMiniAppAdapter` cùng pod container tương ứng (`EMASWindVaneMiniApp` hoặc `EMASUniappMiniApp`); cờ cấu hình `useWindVane` hoặc `useUniApp` trong mã nguồn **PHẢI** khớp tuyệt đối với pod đã khai báo trong Podfile.
> *Nguồn:* `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/ios-access-11` · *Kiểm chứng:* static

> **REQ-14-006** (MUST · C1) — Host iOS **PHẢI** khởi tạo `EMASMiniAppInitConfig` và đăng ký protocol `EMASMiniAppService` thông qua `EMASServiceManager sharedInstance` ngay tại phương thức `application(_:didFinishLaunchingWithOptions:)` trước mọi chức năng khác.
> *Nguồn:* `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/ios-access-11` · *Kiểm chứng:* runtime

> **REQ-14-007** (MUST · C1) — Tham số cấu hình `host` của container trên cả Android và iOS **PHẢI** trỏ về domain hợp lệ của Application Open Platform và bắt buộc sử dụng giao thức HTTPS.
> *Nguồn:* `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/android-access-1`, `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/ios-access-11` · *Kiểm chứng:* runtime

### 3.2 Giao thức JSBridge & Bảng ánh xạ 22 WindVane API

| Lớp JSBridge (WindVane) | Nhóm năng lực | Mức nhạy cảm | Ràng buộc quyền & Chính sách | Tham chiếu SPEC |
|---|---|---|---|---|
| `WVBase` | Core / JSBridge Runtime | Thấp | Mặc định cấp khi container khởi tạo xong | `SPEC-02`, `SPEC-03` |
| `WVBattery` | Hardware / Năng lượng | Thấp | Kẹp tần số và lượng tử hóa thông số pin | `SPEC-06`, `SPEC-07` |
| `WVBluetooth` | Connectivity / BLE | Cao | Explicit consent, chỉ hoạt động khi foreground | `SPEC-06` |
| `WVCamera` | Media Capture | Rất cao | Explicit consent, foreground only, hiển thị chỉ báo thu nhận | `SPEC-06` |
| `WVContacts` | Sensitive Personal Data | Rất cao | Explicit consent, sử dụng giao diện chọn liên hệ (picker) | `SPEC-06`, `SPEC-07` |
| `WVCookie` | Session / Cache | Trung bình | Cách ly tuyệt đối theo origin phân vùng của mini-app | `SPEC-04`, `SPEC-07` |
| `WVFile` | Sandboxed File I/O | Trung bình | Chỉ truy cập thư mục nội bộ sandbox, áp hạn ngạch quota | `SPEC-04`, `SPEC-10` |
| `WVImage` | Media Processing | Trung bình | Xin quyền tường minh khi ghi tệp vào thư viện hệ thống | `SPEC-06` |
| `WVLocation` | Geolocation | Rất cao | Explicit consent, mặc định coarse, foreground liveness gate | `SPEC-06`, `SPEC-07` |
| `WVMotion` | Motion Sensors | Thấp | Lượng tử hóa góc quay, kẹp tần số đọc, foreground only | `SPEC-06` |
| `WVNativeDetector` | Runtime Capability | Thấp | Giới hạn thuộc tính hệ thống bộc lộ, chống fingerprinting | `SPEC-03`, `SPEC-06` |
| `WVNetwork` | Network Information | Thấp | Chỉ bộc lộ trạng thái loại mạng (wifi/cellular/none) | `SPEC-06`, `SPEC-10` |
| `WVNotification` | User Notification | Cao | Xin phép người dùng, áp hạn ngạch số lượng và giờ yên lặng | `SPEC-06`, `SPEC-08` |
| `WVPrefetch` | Network Prefetching | Thấp | Áp hạn ngạch băng thông và danh sách tên miền cho phép | `SPEC-02`, `SPEC-10` |
| `WVReporter` | Telemetry & Audit | Thấp | Khử định danh (sanitized), cấm thu thập PII | `SPEC-07`, `SPEC-10` |
| `WVScreen` | Screen & Wake Lock | Thấp | Wake lock tự động hủy khi mini-app chuyển background | `SPEC-06` |
| `WVScreenCapture` | Display Protection | Cao | Tuân thủ cờ chống chụp màn hình (FLAG_SECURE) của host | `SPEC-04`, `SPEC-06` |
| `WVSystem` | System Information | Thấp | Giới hạn mã định danh thiết bị duy nhất, che giấu phần cứng | `SPEC-06`, `SPEC-07` |
| `WVUI` | Navigation & Window | Thấp | Bị giới hạn trong phạm vi khung nhìn của container | `SPEC-02`, `SPEC-03` |
| `WVUIDialog` | Native Dialogs | Trung bình | Ràng buộc active frame foreground, chống lừa đảo giao diện | `SPEC-03` |
| `WVUIToast` | Native Toast | Thấp | Giới hạn tần suất gọi (rate-limiting) để chống spam UI | `SPEC-03` |
| `WVVideo` | Media Playback | Thấp | Phát media native, cách ly giao diện toàn màn hình | `SPEC-06` |

> **REQ-14-008** (MUST · C1) — Host Container **PHẢI** kiểm soát lời gọi tới 22 lớp JSBridge theo bảng phân loại quyền ở trên; mọi yêu cầu không được cấp phép **PHẢI** bị từ chối an toàn (fail-closed).
> *Nguồn:* `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/android-access-1`, `→ report/06-...md` · *Kiểm chứng:* runtime

> **REQ-14-009** (MUST · C1) — Các API truy cập dữ liệu nhạy cảm (`WVCamera`, `WVContacts`, `WVLocation`, `WVBluetooth`) **PHẢI** áp dụng cổng foreground liveness gate; khi mini-app ở trạng thái background, mọi lời gọi **PHẢI** bị từ chối ngay lập tức.
> *Nguồn:* `→ spec/SPEC-06-capabilities-permissions.md`, `→ report/18-...md` · *Kiểm chứng:* runtime

> **REQ-14-010** (MUST · C1) — Các API cảm biến (`WVMotion`, `WVBattery`) **PHẢI** áp dụng cơ chế lượng tử hóa dữ liệu và giới hạn tần số đọc tối đa để ngăn ngừa tấn công suy đoán kênh phụ và định danh thiết bị.
> *Nguồn:* `→ spec/SPEC-06-capabilities-permissions.md`, `→ report/46-...md` · *Kiểm chứng:* runtime

> **REQ-14-011** (MUST · C1) — Các API lưu trữ (`WVFile`, `WVCookie`) **PHẢI** bị giam trong phân vùng sandbox riêng biệt của từng mini-app; mini-app **KHÔNG ĐƯỢC** phép truy cập hệ thống tệp ngoài sandbox hoặc đọc cookie của mini-app khác.
> *Nguồn:* `→ spec/SPEC-04-security-sandboxing.md`, `→ report/20-...md` · *Kiểm chứng:* runtime

> **REQ-14-012** (SHOULD · C1) `[PROPOSAL]` — Giao tiếp JSBridge **NÊN** dùng cấu trúc thông điệp chuẩn gồm tên hành động (`action`), tham số (`params`), mã điều phối (`callbackId`), và kết quả phản hồi bất đồng bộ dạng `{ status: 'SUCCESS'|'ERROR', data, message }` — chưa tìm được nguồn xác thực cho ý này; cần hãng xác nhận.
> *Nguồn:* `→ report/06-...md`, `→ report/16-...md` · *Kiểm chứng:* runtime

> **REQ-14-013** (SHOULD · C1) `[PROPOSAL]` — Mọi lời gọi qua JSBridge **NÊN** áp đặt thời hạn chờ mặc định (timeout) không quá 10 giây; khi vượt quá thời hạn hoặc khi trang bị hủy (unload), toàn bộ callback đang chờ **PHẢI** bị giải phóng ngay — chưa tìm được nguồn xác thực cho ý này; cần hãng xác nhận.
> *Nguồn:* `→ report/06-...md`, `→ spec/SPEC-02-runtime-lifecycle-bridge.md` · *Kiểm chứng:* runtime

> **REQ-14-014** (MUST · C1) — API `WVScreenCapture` **PHẢI** kích hoạt cơ chế bảo vệ hiển thị (`FLAG_SECURE` trên Android hoặc ẩn snapshot trên iOS) khi mini-app đang xử lý giao dịch thanh toán hoặc dữ liệu định danh người dùng.
> *Nguồn:* `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/android-access-1`, `→ spec/SPEC-04-security-sandboxing.md` · *Kiểm chứng:* runtime

> **REQ-14-015** (MUST · C1) — Dữ liệu telemetry thu thập qua `WVReporter` **PHẢI** được host loại bỏ hoàn toàn các thông tin định danh cá nhân (PII) trước khi truyền về trung tâm giám sát.
> *Nguồn:* `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/android-access-1`, `→ spec/SPEC-07-privacy-data-governance.md` · *Kiểm chứng:* review

### 3.3 Đóng gói, Cấu trúc dự án & Scaffolding Tooling

> **REQ-14-016** (MUST · C3) — Dự án mini-app WindVane **PHẢI** được khởi tạo dựa trên một trong 7 template chính thức của SuperApp Miniapp Develop Tool (React, Vue, Angular với 2 biến thể mỗi loại, hoặc template framework-agnostic).
> *Nguồn:* `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/use-of-scaffolding` · *Kiểm chứng:* static

> **REQ-14-017** (MUST · C3) — Cấu trúc mã nguồn mini-app **PHẢI** tuân thủ bố cục tiêu chuẩn: `public/` (tài nguyên tĩnh), `scripts/` (kịch bản build), `src/` (chứa `assets/`, `components/`, và điểm vào `src/index.js`), cùng tệp cấu hình `package.json`.
> *Nguồn:* `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/use-of-scaffolding` · *Kiểm chứng:* static

> **REQ-14-018** (MUST · C3) — Quy trình đóng gói phát hành **PHẢI** thực thi lệnh `npm run build` để xuất gói tài nguyên hoàn chỉnh vào thư mục `dist/` (hoặc thư mục cấu hình trong Settings của plugin) trước khi nén và tải lên máy chủ.
> *Nguồn:* `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/package-build-1` · *Kiểm chứng:* static

> **REQ-14-019** (SHOULD · C3) — Nhà phát triển **NÊN** kích hoạt và sử dụng gợi ý mã (IntelliSense) của extension VS Code (nhận diện tiền tố `WV`) để bảo đảm gọi đúng tên API và đúng cấu trúc tham số của WindVane.
> *Nguồn:* `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/use-of-scaffolding` · *Kiểm chứng:* review

> **REQ-14-020** (MUST · C3) — Phiên bản phát hành khi đóng gói **PHẢI** sử dụng phiên bản khuyến nghị do công cụ tự động tính toán từ máy chủ hoặc phiên bản tùy chỉnh tuân theo quy tắc số hiệu tăng dần.
> *Nguồn:* `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/package-build-1` · *Kiểm chứng:* static

> **REQ-14-021** (SHOULD · C3) `[PROPOSAL]` — Dung lượng tệp nén của thư mục `dist/` khi upload lên Application Open Platform **NÊN** không vượt quá 8MB đối với gói tải trực tiếp — chưa tìm được nguồn xác thực cho ý này; cần hãng xác nhận.
> *Nguồn:* `→ report/06-...md` · *Kiểm chứng:* static

> **REQ-14-022** (SHOULD · C3) `[PROPOSAL]` — Gói build **NÊN** duy trì bản kê khai quyền tương thích với W3C MiniApp Manifest trong thư mục gốc để phục vụ kiểm tra tĩnh tự động trước khi triển khai — chưa tìm được nguồn xác thực cho ý này; cần hãng xác nhận.
> *Nguồn:* `→ report/06-...md`, `→ spec/SPEC-01-package-manifest-addressing.md` · *Kiểm chứng:* static

### 3.4 Vòng đời phát hành, Kiểm duyệt & Quản lý Kênh

> **REQ-14-023** (MUST · C2) — Mọi phiên bản mini-app trước khi phát hành **PHẢI** hoàn thành yêu cầu phát hành phiên bản (version release request) và **PHẢI** được phê duyệt trên Application Open Platform.
> *Nguồn:* `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/user-guide/release-version` · *Kiểm chứng:* review

> **REQ-14-024** (MUST · C2) — Phiên bản phát hành lần đầu tiên của một mini-app **CHỈ ĐƯỢC** phát hành theo hình thức Official Release; **KHÔNG ĐƯỢC** chọn hình thức Whitelisting test hay Canary release cho lần phát hành đầu.
> *Nguồn:* `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/user-guide/release-version` · *Kiểm chứng:* static

> **REQ-14-025** (MAY · C2) — Đối với các phiên bản tiếp theo, nhà vận hành **CÓ THỂ** kích hoạt Whitelisting test thông qua thao tác `Set Whitelist` (bổ sung User ID) và `Start Test` để kiểm thử nội bộ trước khi mở rộng.
> *Nguồn:* `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/user-guide/release-version` · *Kiểm chứng:* runtime

> **REQ-14-026** (MAY · C2) — Nhà vận hành **CÓ THỂ** thực hiện phát hành Canary bằng cách đặt tỷ lệ phần trăm (`Set Canary Percentage`) và kích hoạt `Start Canary Release` để kiểm soát rủi ro trước khi chuyển sang phát hành chính thức.
> *Nguồn:* `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/user-guide/release-version` · *Kiểm chứng:* runtime

> **REQ-14-027** (MUST · C2) — Khi một phiên bản mới được phát hành chính thức (Official Release) trên một kênh phân phối, phiên bản cũ đang chạy trên cùng kênh phân phối đó **PHẢI** được hệ thống tự động lưu trữ (archive).
> *Nguồn:* `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/user-guide/release-version` · *Kiểm chứng:* runtime

> **REQ-14-028** (MUST · C1) — Host Container **PHẢI** hỗ trợ cơ chế quét mã QR tạo ra từ công cụ phát triển để tải và xem trước (preview) bản build tức thì trên thiết bị mục tiêu trước khi nộp duyệt.
> *Nguồn:* `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/package-build-1` · *Kiểm chứng:* runtime

---

## 4. Ghi chú triển khai (informative)

### 4.1 Khởi tạo Container trên Android và iOS

```java
// Android: Khởi tạo IMiniAppService trong Application class
MiniAppInitConfig config = new MiniAppInitConfig.Builder()
    .setUseWindVane(true)
    .setAccessKey(CONFIG_ACCESS_KEY)
    .setSecretKey(CONFIG_SECRET_KEY)
    .setHost("emas-publish-intl.emas-poc.com")
    .setAppCode(CONFIG_APP_CODE)
    .build();

IMiniAppService miniAppService = new MiniAppService();
miniAppService.initialize(this, config);
ServiceManager.getInstance().registerService(IMiniAppService.class.getName(), miniAppService);
```

```objc
// iOS: Khởi tạo trong AppDelegate
EMASMiniAppInitConfig *config = [[EMASMiniAppInitConfig alloc] init];
config.useWindVane = YES;
config.useUniApp = NO;
config.accessKey = CONFIG_ACCESS_KEY;
config.secretKey = CONFIG_SECRET_KEY;
config.host = @"emas-publish-intl.emas-poc.com";
config.appCode = CONFIG_APP_CODE;

[[EMASServiceManager sharedInstance] registerServiceProtocol:@"EMASMiniAppService"];
```

### 4.2 Luồng phát triển và phân phối phiên bản

```text
[Scaffolding Template] -> [Code w/ WV IntelliSense] -> [npm run build] -> [Output dist/]
                                                                               |
                                                                               v
[Official Release (V1)] <--- [Approved] <--- [Release Request] <--- [VS Code Publish Tool]
          |
          v (Từ V2 trở đi)
[Set Whitelist (User IDs)] -> [Set Canary %] -> [Official Release] -> [Auto-archive bản cũ]
```

---

## 5. Tiêu chí tuân thủ

- [ ] Host Android tích hợp đúng SDK WindVane 1.4.0 và giữ cấu hình ProGuard cho `android.taobao.windvane.jsbridge.api.*`.
- [ ] Host iOS hoàn tất khởi tạo `EMASMiniAppInitConfig` tại `didFinishLaunchingWithOptions`.
- [ ] 22 lớp WindVane JSBridge API được kiểm soát đúng ranh giới phân quyền nhạy cảm và ràng buộc foreground.
- [ ] API cảm biến và thông số pin được áp dụng lượng tử hóa dữ liệu và kẹp tần số đọc.
- [ ] Dữ liệu lưu trữ `WVFile` và `WVCookie` bị phân vùng cô lập tuyệt đối theo từng mini-app.
- [ ] Bản build được tạo từ cấu trúc scaffolding chuẩn và xuất gói qua thư mục `dist/`.
- [ ] Quy trình phát hành tuân thủ phê duyệt phiên bản, quy tắc Official Release cho lần đầu và tự động archive phiên bản cũ trên cùng kênh phân phối.

---

## 6. Cân nhắc an ninh & riêng tư

- **Bảo vệ khóa xác thực host:** Các tham số `accessKey`, `secretKey`, `appCode` là bí mật hạ tầng cấp host, **TUYỆT ĐỐI KHÔNG** được đóng gói vào mã nguồn của mini-app hay bộc lộ qua JSBridge.
- **Ngăn ngừa bridge injection:** JSBridge của WindVane chỉ được nạp trong khung nhìn của mini-app đã được ký số và kiểm duyệt; cấm nạp vào webview mở tải URL tùy ý từ bên ngoài.
- **Bảo mật dữ liệu viễn trắc (telemetry):** Toàn bộ dữ liệu ghi nhận qua `WVReporter` phải được duyệt lọc để ngăn rò rỉ token phiên, mật khẩu hoặc số thẻ thanh toán.
- **Ràng buộc tương thích:** Khi chuyển đổi giữa WindVane và uni-app container, host phải bảo đảm cách ly dữ liệu độc lập giữa hai loại runtime.

---

## 7. Tham chiếu

1. Alibaba Cloud — Rapid Development of WindVane Applets or H5 Applications: `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/getting-started/rapid-development-of-windvane-applets-or-h5-applications` (Cập nhật: 19/09/2025)
2. Alibaba Cloud — Android Access: `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/android-access-1`
3. Alibaba Cloud — iOS Access: `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/ios-access-11`
4. Alibaba Cloud — Package and Build: `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/package-build-1` (Cập nhật: 02/06/2026)
5. Alibaba Cloud — Use of Scaffolding: `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/use-of-scaffolding` (Cập nhật: 19/09/2025)
6. Alibaba Cloud — Release Version: `https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/user-guide/release-version` (Cập nhật: 18/04/2026)
7. Báo cáo nguồn trong repository: `report/01-executive-summary.md`, `report/06-gaps-validation-plan.md`, `report/16-iteration-19-host-bridge-cross-context.md`, `report/18-iteration-22-w3c-miniapp-tuf-device-governance.md`, `report/20-iteration-24-app-identity-secure-storage-network.md`, `report/46-iteration-50-w3c-miniapp-suite-bluetooth-nfc-sensors.md`.
8. Đặc tả kỹ thuật liên quan: `SPEC-01`, `SPEC-02`, `SPEC-03`, `SPEC-04`, `SPEC-06`, `SPEC-07`, `SPEC-08`, `SPEC-10`, `SPEC-13`.
