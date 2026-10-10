# APPENDIX C — Bảng đối chiếu POC Alibaba/WindVane ↔ SPEC

| Trường | Giá trị |
|---|---|
| ID | `APPENDIX-C` |
| Phiên bản | 1.1.0 |
| Trạng thái | Draft |
| Mục đích | Đối chiếu từng thành phần POC Alibaba Cloud SuperApp/WindVane với yêu cầu SPEC |
| Nguồn | POC resource sheet (`gid=58086397`, `gid=1488955061`), tài liệu Alibaba Cloud, `report/01`, `report/06` |

---

## 1. Mục đích & cách dùng

Tài liệu này cung cấp bảng đối chiếu chi tiết giữa hiện trạng kỹ thuật của **POC Alibaba Cloud SuperApp / WindVane** (bối cảnh tham chiếu dự án POC Viettel / Alibaba Cloud WindVane Mini App được phân tích trong `report/53`, `report/54`, `report/138`) và hệ thống yêu cầu kỹ thuật trong bộ đặc tả chuẩn hóa `SPEC-00` đến `SPEC-14`.

Bộ đặc tả của Super App được thiết kế theo nguyên tắc **trung lập nhà cung cấp** (vendor-neutral), kế thừa các tiêu chuẩn mở quốc tế từ W3C, WHATWG và IETF. Nhằm đảm bảo tính khả thi thực tế trong triển khai, POC Alibaba Cloud SuperApp / WindVane được sử dụng làm cơ sở đối chuẩn thực nghiệm ban đầu.

### Phân cấp thẩm quyền nguồn tài liệu
Mối quan hệ thẩm quyền giữa các nguồn dữ liệu đối soát được xác định như sau:
1. **POC Resource Sheet (`gid=58086397`, `gid=1488955061`):** Nguồn có thẩm quyền cao nhất về **cấu hình POC cụ thể** (phiên bản SDK thực tế được cung cấp, endpoint môi trường lab POC, kho repository Maven/CocoaPods nội bộ, phiên bản plugin IDE và các demo repository).
2. **Tài liệu công khai Alibaba Cloud (Help Center):** Cung cấp kiến trúc giải pháp tổng thể, hướng dẫn tích hợp chuẩn, tài liệu API và các luồng vận hành chính thức của sản phẩm thương mại.
3. **Bộ đặc tả Super App (`SPEC-00` đến `SPEC-14`):** Đóng vai trò chuẩn hóa trung lập, thiết lập các yêu cầu bắt buộc (MUST/SHOULD) và các ranh giới bảo mật độc lập mà giải pháp POC cần đáp ứng hoặc bao bọc thêm.

Theo định hướng chiến lược tại mục `## Fit to the Alibaba POC` trong `report/01-executive-summary.md`, các thành phần kỹ thuật được phân loại thành ba nhóm hành động chính:
- **KEEP (Giữ nguyên của POC):** Tận dụng tối đa các thành phần nền tảng sẵn có của Alibaba Cloud SuperApp và WindVane container đã chứng minh hiệu quả vận hành về runtime và control plane cơ bản.
- **ADD (Cần bổ sung quanh POC):** Thiết lập các lớp bao bọc độc lập về bảo mật chuỗi cung ứng, chữ ký số, quản trị định danh, kiểm định an ninh tự động và sổ cái kiểm toán để bảo đảm tính mở và an toàn toàn diện.
- **VALIDATE (Cần hãng xác nhận):** Đưa vào danh mục kiểm chứng thực địa các thông số kỹ thuật, lược đồ cấu hình nội bộ và hành vi runtime chưa có tài liệu công khai đầy đủ hoặc chưa được làm rõ trong POC sheet.

Nguyên tắc gán nhãn trạng thái và phân biệt nguồn gốc:
- **Đã xác nhận:** Thành phần đã có nguồn kiểm chứng thực tế rõ ràng. Bắt buộc ghi rõ nguồn: `[POC sheet]` (đối với thông tin từ sheet cấu hình POC) hoặc `[Tài liệu công khai [x]]` (dẫn link tài liệu tại §7).
- **Cần hãng xác nhận:** Thành phần kỹ thuật nội bộ của hãng chưa công bố đầy đủ trong cả tài liệu công khai lẫn POC resource sheet, bắt buộc đo kiểm lab hoặc yêu cầu văn bản xác nhận từ đội ngũ kỹ thuật Alibaba Cloud.
- **[PROPOSAL]:** Yêu cầu kiến trúc bổ sung độc lập từ bộ SPEC, hiện chưa có trong tính năng thương mại gốc của nhà cung cấp.

---

## 2. Bảng đối chiếu chính

| Thành phần POC | Mô tả (nguồn Alibaba) | Yêu cầu SPEC tương ứng | Trạng thái | Hành động |
|---|---|---|---|---|
| Application Open Platform (Console) | `[POC sheet]` Control plane quản trị vòng đời miniapp, phiên bản, kênh phân phối tại console `https://poc.superapp-intl.com/superapp#/login` · `[Tài liệu công khai [1], [8], [9]]` | `REQ-08-001`, `REQ-08-011` | Đã xác nhận | KEEP |
| Điểm cuối Khởi tạo Container | `[POC sheet]` Miniapp container initialization service address: `https://poc.superapp-intl.com` | `REQ-02-002`, `SPEC-04` | Đã xác nhận | KEEP |
| Điểm cuối Open API Control Plane | `[POC sheet]` Application Open Platform Open API address: `https://poc.superapp-intl.com` | `REQ-08-001`, `SPEC-13` | Đã xác nhận | KEEP |
| WindVane Container Runtime | `[POC sheet & Tài liệu công khai [2], [4], [6], [7]]` Container WebView tối ưu nhúng vào host app (Android, iOS, Flutter, React Native) để thực thi miniapp | `REQ-02-001`, `REQ-02-004` | Đã xác nhận | KEEP |
| uni-app Container Runtime | `[POC sheet & Tài liệu công khai [1]]` Lựa chọn container thứ hai trên nền tảng Alibaba Cloud EMAS; các dự án demo POC tích hợp song song cả WindVane lẫn uni-app | `REQ-02-001`, `REQ-08-024` | Đã xác nhận | KEEP |
| Nền tảng Host Android (Native SDK) | `[POC sheet]` Maven dependency: `com.aliyun.emas.suite.foundation:mini-app-adapter:1.8.9.2` và `com.aliyun.emas.suite.foundation:windvane-mini-app:1.8.9.2` · `[Tài liệu công khai [2], [3]]` | `REQ-02-026` | Đã xác nhận | KEEP |
| Nền tảng Host iOS (Native Pods) | `[POC sheet]` CocoaPods dependencies: `pod 'EMASServiceManager'`, `pod 'EMASMiniAppAdapter', '1.1.3'`, `pod 'EMASWindVaneMiniApp', '1.2.4'` · `[Tài liệu công khai [4], [5]]` | `REQ-02-027` | Đã xác nhận | KEEP |
| Nền tảng Host Flutter | `[POC sheet & Tài liệu công khai [6]]` Hỗ trợ Flutter host qua `MethodChannel("windvane_miniapp")`, class `WindVaneMiniAppManager`, kế thừa `FlutterActivity` (Android) / `FlutterViewController` + `EMASMainViewController` (iOS) | `REQ-02-001`, `REQ-08-024` | Đã xác nhận | KEEP |
| Nền tảng Host React Native | `[Tài liệu công khai [7]]` Hỗ trợ React Native host nhúng WindVane applet container | `REQ-02-001`, `REQ-08-024` | Đã xác nhận | KEEP |
| Kho lưu trữ Maven Repository (Android) | `[POC sheet]` Maven private repository lưu trữ SDK Android: `https://nexus-console.superapp-intl.com/repository/maven` | `REQ-02-026` | Đã xác nhận | KEEP |
| Kho lưu trữ CocoaPods Specs (iOS) | `[POC sheet]` CocoaPods Specs Gitlab repository lưu trữ SDK iOS: `https://gitlab-console.superapp-intl.com/emas-ios/emas-specs.git` | `REQ-02-027` | Đã xác nhận | KEEP |
| Dự án mẫu Demo (Android, iOS, Flutter) | `[POC sheet]` Kho GitLab dự án demo: Android (`emas-superapp-android`, branch `master`), iOS (`EMAS-Superapp-iOS`, branch `master`), Flutter (`flutter_windvane_demo`, branch `main`) trên `gitlab-console.superapp-intl.com` (thông tin xác thực nằm trong POC sheet — không đưa vào tài liệu) | `REQ-02-026`, `REQ-02-027` | Đã xác nhận | KEEP |
| Cấu hình khởi tạo Host SDK | `[POC sheet & Tài liệu công khai [2], [4]]` Tham số init: `accessKey`, `secretKey`, `host` (`https://poc.superapp-intl.com`), `appCode`, `useWindVane` | `REQ-02-002`, `SPEC-04` | Đã xác nhận | KEEP |
| IDE Plugin (SuperApp Miniapp Develop Tool) | `[POC sheet]` VS Code extension `emas-mini-app-plugin-0.1.1.vsix` tải tại `https://poc.superapp-intl.com/portal/#/resource` hỗ trợ scaffolding, phát triển và đóng gói · `[Tài liệu công khai [1], [17]]` | `REQ-01-001`, `REQ-01-005` | Đã xác nhận | KEEP |
| Quy trình đóng gói bản build | `[Tài liệu công khai [18]]` Lệnh `Publish MiniApp`, chạy `npm run build`, nén thư mục `dist/` tải lên | `REQ-01-005` | Đã xác nhận | KEEP |
| Xem trước bằng mã QR | `[Tài liệu công khai [1], [17]]` Tính năng quét mã QR từ VS Code extension hoặc Console để mở bản preview trên thiết bị thật | `REQ-08-019` | Đã xác nhận | KEEP |
| Phê duyệt yêu cầu phát hành | `[Tài liệu công khai [12], [20]]` Quy trình bắt buộc duyệt Version Release Request trước khi phân phối | `REQ-08-011`, `REQ-12-001` | Đã xác nhận | KEEP |
| Thử nghiệm danh sách Whitelist | `[Tài liệu công khai [14], [15], [20]]` Thử nghiệm nội bộ theo danh sách user ID trước khi phát hành diện rộng | `REQ-08-019` | Đã xác nhận | KEEP |
| Phân phối Canary (Gray Release) | `[Tài liệu công khai [12], [20]]` Cơ chế phát hành theo tỷ lệ phần trăm người dùng (Set Canary Percentage) | `REQ-08-019`, `REQ-08-020` | Đã xác nhận | KEEP |
| Phát hành chính thức (Official) | `[Tài liệu công khai [12], [20]]` Phân phối toàn mạng, tự động lưu trữ (archive) phiên bản cũ cùng kênh | `REQ-08-011`, `REQ-08-028` | Đã xác nhận | KEEP |
| Kiến trúc JSAPI 2 tầng — API bề mặt (`wv.*`) | `[Tài liệu công khai [10]]` JavaScript API bề mặt cho lập trình viên miniapp gọi theo namespace `wv.<method>`, độc lập với cấu trúc class host | `REQ-02-021`, `SPEC-03` | Đã xác nhận | KEEP |
| Kiến trúc JSAPI 2 tầng — Lớp triển khai Android (`WV*`) | `[Tài liệu công khai [2]]` 22 lớp native implementation trên Android thuộc namespace `android.taobao.windvane.jsbridge.api.WV*`, đóng vai trò backend thực thi bridge | `REQ-02-021`, `SPEC-03` | Đã xác nhận | KEEP |
| JSAPI Ủy quyền người dùng (`wv.getAuthCode`) | `[Tài liệu công khai [10]]` API `wv.getAuthCode` xin ủy quyền người dùng, trả về `authCode` để backend SuperApp đổi thông tin người dùng | `REQ-06-001`, `REQ-06-015` | Đã xác nhận | KEEP |
| Open API cấp đổi Token người dùng (`applyToken`) | `[Tài liệu công khai [11]]` Endpoint Open API `/v1/authorizations/applyToken` đổi `authCode` lấy `user_id`, `avatar`, `nickname`, `mobile number`, `region`, `gender`, `date of birth` | `REQ-06-001`, `SPEC-13` | Đã xác nhận | KEEP |
| Phân quyền gọi năng lực gốc | `[Tài liệu công khai [2], [4]]` Cơ chế native capability authorization khi miniapp yêu cầu quyền hệ thống | `REQ-06-001`, `REQ-06-015` | Đã xác nhận | KEEP |
| Xác thực Chuỗi cung ứng & Ký số | `[SPEC proposal]` Xác thực chữ ký TUF, Sigstore Cosign, chứng thực SLSA provenance và in-toto | `REQ-05-001`, `REQ-05-008` | [PROPOSAL] | ADD |
| Danh mục đã ký & Khóa Digest | `[SPEC proposal]` Signed catalog độc lập, khóa digest SHA-256 bất biến cho mọi bản release | `REQ-05-004`, `REQ-08-008` | [PROPOSAL] | ADD |
| Quản lý Publisher, RBAC & KYBC | `[SPEC proposal]` Định danh nhà phát triển, quy trình KYBC theo EU DSA, namespace độc quyền | `REQ-08-003`, `REQ-08-005` | [PROPOSAL] | ADD |
| Hợp đồng Manifest chuẩn W3C | `[SPEC proposal]` Bổ sung `manifest.json` chuẩn W3C MiniApp, Permissions Policy độc lập | `REQ-01-001`, `REQ-01-002` | [PROPOSAL] | ADD |
| Cổng kiểm tra tĩnh tự động | `[SPEC proposal]` Bộ cổng G1–G5: quét secret, quét malware, kiểm tra CSP, Trusted Types | `SPEC-04`, `APPENDIX-B` | [PROPOSAL] | ADD |
| Sổ cái kiểm toán bất biến | `[SPEC proposal]` Append-only audit ledger lưu vết mọi thao tác duyệt, phát hành, đổi quyền | `REQ-08-010`, `REQ-07-009` | [PROPOSAL] | ADD |
| Cơ chế Rollback & Quarantine | `[SPEC proposal]` Rollback một chạm về digest an toàn tốt nhất, cách ly khẩn cấp bản lỗi | `REQ-08-021`, `REQ-08-022` | [PROPOSAL] | ADD |
| Kế hoạch Ngừng hỗ trợ chuẩn hóa | `[SPEC proposal]` Vòng đời Deprecation/Sunset có kế hoạch, thông báo máy đọc và dọn dữ liệu | `REQ-08-028`, `REQ-08-029` | [PROPOSAL] | ADD |
| Bộ giải quyết tương thích đa chiều | `[SPEC proposal]` Compatibility resolver đánh giá ma trận SDK host, bridge API, OS, locale | `REQ-08-024`, `REQ-08-025` | [PROPOSAL] | ADD |
| Giám sát Store & Đo lường SLA | `[SPEC proposal]` Hệ thống thu thập độc lập tỷ lệ crash, ANR, độ trễ và xử lý vi phạm SLA | `REQ-08-020`, `REQ-10-001` | [PROPOSAL] | ADD |
| Hồ sơ Bằng chứng Trợ năng | `[SPEC proposal]` Bộ bằng chứng kiểm định tuân thủ tiêu chuẩn trợ năng WCAG 2.2 AA | `REQ-11-008`, `REQ-12-015` | [PROPOSAL] | ADD |
| Lược đồ Manifest nội bộ | `[Khoảng trống POC]` Cấu trúc trường, khóa định danh và siêu dữ liệu đặc thù của WindVane/uni-app (POC sheet không đề cập) | `REQ-01-001`, `REQ-01-007` | Cần hãng xác nhận | VALIDATE |
| Phân loại quyền hạn JSAPI | `[Khoảng trống POC]` Bảng phân cấp rủi ro (Permission Taxonomy) cho 22 lớp API WindVane và API `wv.*` (POC sheet không đề cập) | `REQ-02-021`, `REQ-06-003` | Cần hãng xác nhận | VALIDATE |
| Ma trận tương thích Host SDK | `[Khoảng trống POC]` Dải tương thích Android API level, iOS target, Flutter/RN runtime và xung đột thư viện native (POC sheet chỉ nêu phiên bản POC cụ thể) | `REQ-08-024`, `REQ-10-008` | Cần hãng xác nhận | VALIDATE |
| Giới hạn dung lượng gói tải lên | `[Khoảng trống POC]` Trần dung lượng gói chính (main) và gói phụ (sub-package) khi nén và giải nén (POC sheet không đề cập) | `REQ-01-006`, `APPENDIX-B` | Cần hãng xác nhận | VALIDATE |
| Ngữ nghĩa API của Control Plane | `[Khoảng trống POC]` Tài liệu đặc tả đầy đủ REST/RPC API phê duyệt, tạm dừng, rollback trên portal hãng ngoài endpoint applyToken | `REQ-08-011`, `SPEC-13` | Cần hãng xác nhận | VALIDATE |
| Đặc tả giao thức Host Bridge | `[Khoảng trống POC]` Cấu trúc request/reply/error, correlation ID, timeout và hủy lệnh gọi; định dạng payload cụ thể chưa có tài liệu xác thực | `REQ-02-022`, `REQ-02-025` | Cần hãng xác nhận | VALIDATE |
| Tầng Cache & Chạy ngoại tuyến | `[Khoảng trống POC]` Cơ chế cache native, invalidation, stale serving và khởi chạy khi offline (POC sheet không giải quyết) | `REQ-02-009`, `REQ-10-005` | Cần hãng xác nhận | VALIDATE |
| Khả năng Trợ năng của Container | `[Khoảng trống POC]` Cây trợ năng native, điều hướng bàn phím, tương thích TalkBack/VoiceOver (POC sheet không giải quyết) | `REQ-02-027`, `REQ-11-008` | Cần hãng xác nhận | VALIDATE |

---

## 3. Nhóm KEEP (giữ nguyên của POC)

Nhóm này bao gồm các năng lực cốt lõi sẵn có của nền tảng Alibaba Cloud SuperApp, đáp ứng tốt yêu cầu thực thi kỹ thuật và được giữ nguyên trong kiến trúc triển khai:

### 3.1 Control Plane, Cổng quản trị & Điểm cuối dịch vụ (Endpoints)
- **Application Open Platform Console:** Giữ nguyên làm cổng quản trị vòng đời ứng dụng tập trung tại địa chỉ `https://poc.superapp-intl.com/superapp#/login`. Nền tảng chịu trách nhiệm tiếp nhận bản build, quản lý phiên bản, cấp phát mã `appCode`, và cấu hình điểm cuối phân phối cho host app.
- **Điểm cuối Dịch vụ Khởi tạo Container:** Địa chỉ `https://poc.superapp-intl.com` dùng để khởi tạo và đồng bộ siêu dữ liệu container trên host app.
- **Điểm cuối Open API:** Địa chỉ `https://poc.superapp-intl.com` cung cấp các API quản trị và tích hợp dịch vụ người dùng.

### 3.2 Container Runtime nhúng trên 4 nền tảng Host (Android, iOS, Flutter, React Native)
Hệ thống POC hỗ trợ 4 nền tảng host di động, thay vì chỉ 2 nền tảng native như trong các khảo sát sơ bộ ban đầu:
- **Nền tảng Android (Native SDK):** Tích hợp qua kho Maven private `https://nexus-console.superapp-intl.com/repository/maven` với 2 artifact bắt buộc:
  - `com.aliyun.emas.suite.foundation:mini-app-adapter:1.8.9.2`
  - `com.aliyun.emas.suite.foundation:windvane-mini-app:1.8.9.2`
- **Nền tảng iOS (Native CocoaPods):** Tích hợp qua kho CocoaPods Specs riêng `https://gitlab-console.superapp-intl.com/emas-ios/emas-specs.git` với các pod:
  - `pod 'EMASServiceManager'`
  - `pod 'EMASMiniAppAdapter', '1.1.3'`
  - `pod 'EMASWindVaneMiniApp', '1.2.4'`
- **Nền tảng Flutter:** Hỗ trợ tích hợp WindVane miniapp container vào ứng dụng Flutter thông qua `MethodChannel("windvane_miniapp")` và lớp điều phối `WindVaneMiniAppManager`, kế thừa `FlutterActivity` (Android) hoặc kết hợp `FlutterViewController` và `EMASMainViewController` (iOS).
- **Nền tảng React Native:** Hỗ trợ nhúng WindVane applet container theo đặc tả kỹ thuật và tài liệu tích hợp chính thức của Alibaba Cloud.
- **Cơ chế Dual-Container:** Nền tảng hỗ trợ đồng thời cả WindVane container và uni-app container tùy thuộc cờ cấu hình `useWindVane` hoặc `useUniApp`.
- **Dự án mẫu thực nghiệm (Demo Projects):** Nguồn POC cung cấp 3 repository mẫu tích hợp đầy đủ trên GitLab nội bộ:
  - Android Demo: `https://gitlab-console.superapp-intl.com/SuperAppDemo/emas-superapp-android` (branch `master`)
  - iOS Demo: `https://gitlab-console.superapp-intl.com/SuperAppDemo/EMAS-Superapp-iOS` (branch `master`)
  - Flutter Demo: `https://gitlab-console.superapp-intl.com/SuperAppDemo/flutter_windvane_demo` (branch `main`)
  *(Ghi chú an toàn: Toàn bộ thông tin xác thực truy cập tài nguyên demo nằm trong POC resource sheet — tuyệt đối KHÔNG đưa vào tài liệu).*

### 3.3 Chu trình Đóng gói & Phát triển Developer
- **IDE Plugin chuyên dụng:** Tiện ích mở rộng chính thức `emas-mini-app-plugin-0.1.1.vsix` được phân phối trực tiếp tại cổng tài nguyên `https://poc.superapp-intl.com/portal/#/resource` để cài đặt vào VS Code.
- **Quy trình đóng gói & Preview:** Hỗ trợ scaffolding khởi tạo dự án mẫu (React, Vue, HTML5), biên dịch (`npm run build`), nén tài nguyên thư mục `dist/`, tải lên qua lệnh `Publish MiniApp` và quét mã QR để mở bản preview trực tiếp trên thiết bị di động thật.

### 3.4 Quy trình Phân phối Staged Rollout
- **Phê duyệt & Điều phối phát hành:** Tận dụng quy trình bắt buộc duyệt Version Release Request trên portal.
- **Phân phối theo tầng:** Áp dụng luồng phân phối tuần tự: kiểm thử Whitelist theo danh sách user ID → phát hành thử nghiệm Canary theo tỷ lệ phần trăm (Set Canary Percentage) → phát hành chính thức Official Release và tự động lưu trữ phiên bản cũ trên cùng channel.

### 3.5 Kiến trúc JSAPI 2 tầng & Năng lực Định danh Người dùng
Kiến trúc gọi API cầu nối được làm rõ theo mô hình 2 tầng tường minh:
1. **Tầng API bề mặt JavaScript (`wv.*`):** Đây là API thực tế mà nhà phát triển miniapp tiếp xúc và gọi trong mã nguồn web. Ví dụ thực tế: `wv.getAuthCode` được dùng để hiển thị hộp thoại xin ủy quyền của người dùng và trả về mã `authCode`.
2. **Tầng Lớp triển khai Native Android (`WV*`):** 22 lớp native implementation thuộc namespace `android.taobao.windvane.jsbridge.api.WV*` (`WVBase`, `WVBattery`, `WVBluetooth`, `WVCamera`, `WVContacts`, `WVCookie`, `WVFile`, `WVImage`, `WVLocation`, `WVMotion`, `WVNativeDetector`, `WVNetwork`, `WVNotification`, `WVPrefetch`, `WVReporter`, `WVScreen`, `WVScreenCapture`, `WVSystem`, `WVUI`, `WVUIDialog`, `WVUIToast`, `WVVideo`) là mã nguồn xử lý phía dưới (backend implementation) của Android host WebView bridge, không phải là API trực diện cho developer gọi.
3. **Luồng Cấp đổi Token & Dữ liệu người dùng (Open API):** Kết hợp giữa JSAPI `wv.getAuthCode` trên client và Open API `/v1/authorizations/applyToken` tại `https://poc.superapp-intl.com` phía server backend để đổi `authCode` lấy thông tin cá nhân cơ bản: `user_id`, `avatar`, `nickname`, `mobile number`, `region`, `gender`, `date of birth`.

---

## 4. Nhóm ADD (cần bổ sung quanh POC)

Nhóm này bao gồm các lớp kiểm soát bảo mật, chính sách quản trị và tính mở trung lập cần xây dựng bổ sung bao quanh giải pháp của nhà cung cấp để đáp ứng chuẩn SPEC:

### 4.1 Bảo mật chuỗi cung ứng & Ký số (`SPEC-05`)
- Xây dựng hệ thống ký số độc lập dựa trên mô hình TUF (The Update Framework) với 4 vai trò tách biệt: Root, Targets, Snapshot, Timestamp.
- Tích hợp Sigstore Cosign, chứng thực nguồn gốc SLSA v1.0 provenance (tối thiểu SLSA Level 2/3), sinh SBOM CycloneDX v1.6 và CBOM mật mã trước khi artifact được đẩy vào portal của hãng.

### 4.2 Quản trị Publisher, Phân quyền & Định danh (`SPEC-08`)
- Thiết lập cổng xác minh danh tính tổ chức (KYBC) tuân thủ quy định EU Digital Services Act (DSA).
- Ràng buộc không gian tên định danh (`namespace`) độc quyền cho từng nhà phát triển, áp dụng cơ chế xác thực đa yếu tố (MFA) và phân quyền kiểm duyệt RBAC.

### 4.3 Chuẩn hóa Manifest & Permissions Policy (`SPEC-01`, `SPEC-06`)
- Áp dụng cấu trúc `manifest.json` chuẩn W3C MiniApp độc lập với cấu hình riêng của hãng.
- Thiết lập ranh giới Permissions Policy fail-closed: phân tách 3 tập quyền rõ ràng (declared, effective, runtime). Mặc định cấm truy cập phần cứng nhạy cảm trừ khi được người dùng cấp quyền rõ ràng tại runtime.

### 4.4 Cổng an ninh độc lập G1–G5 (`SPEC-04`, `APPENDIX-B`)
- Bổ sung hệ thống kiểm tra tự động trước phê duyệt: quét mã độc (malware), quét lộ lọt API key/secret, kiểm tra chính sách bảo mật nội dung CSP nghiêm ngặt (không dùng wildcard `*`), và cưỡng chế Trusted Types chống XSS.

### 4.5 Sổ cái kiểm toán & Cơ chế Rollback chủ động (`SPEC-07`, `SPEC-08`)
- Thiết lập sổ cái kiểm toán dạng append-only độc lập ghi nhận mọi thao tác phát hành, phê duyệt, cấp quyền và sự kiện an ninh.
- Xây dựng cơ chế rollback một chạm về digest an toàn tốt nhất đã biết và lệnh cách ly khẩn cấp (`quarantine`) vô hiệu hóa tức thì các bản phát hành có lỗ hổng.

### 4.6 Bộ giải quyết tương thích & Giám sát SLA (`SPEC-08`, `SPEC-10`, `SPEC-11`)
- Xây dựng bộ giải quyết tương thích (Compatibility Resolver) độc lập để đánh giá ma trận thiết bị, phiên bản SDK host (Android, iOS, Flutter, React Native) và dải API khả dụng trước khi cho phép tải ứng dụng.
- Thiết lập hệ thống đo kiểm độc lập tỷ lệ sập (crash rate), hiện tượng treo giao diện (ANR), độ trễ phản hồi bridge, và thu thập bằng chứng trợ năng theo tiêu chuẩn WCAG 2.2 AA.

---

## 5. Nhóm VALIDATE (cần hãng xác nhận)

Các nội dung kỹ thuật bắt buộc phải đo kiểm thực tế hoặc yêu cầu văn bản xác nhận chính thức từ Alibaba Cloud trong quá trình triển khai, do cả tài liệu công khai lẫn POC resource sheet đều chưa công bố hoặc chưa giải quyết:

### 5.1 Cấu trúc Manifest nội bộ của WindVane/uni-app
- Cần làm rõ lược đồ schema chi tiết, các trường cấu hình bắt buộc và tùy chọn mà container đọc từ gói `dist/`, cách khai báo trang khởi chạy và các khóa cấu hình mạng. POC resource sheet không đề cập đến lược đồ schema này.

### 5.2 Bảng phân loại quyền hạn JSAPI (Taxonomy)
- Cần xác nhận ma trận phân cấp rủi ro cho 22 lớp API WindVane và các API `wv.*`: quyền nào cần người dùng đồng ý (runtime prompt), quyền nào được cấp tĩnh theo manifest, và cách máy chủ quản trị cấu hình danh sách cho phép (allowlist).

### 5.3 Ma trận tương thích Host SDK & Xung đột Native
- Cần xác định dải phiên bản hệ điều hành hỗ trợ (Android SDK min/target API level, iOS deployment target, Flutter SDK version matrix, React Native versions), danh mục thư viện phụ thuộc và kiểm tra xung đột với kiến trúc native hiện có của Super App host. POC sheet chỉ cung cấp phiên bản cố định cho lab POC.

### 5.4 Giới hạn dung lượng gói phát hành
- Cần xác nhận trần dung lượng tệp ZIP khi upload lên Application Open Platform, giới hạn kích thước gói chính (main package) và các gói phụ (sub-packages) khi nén và sau khi giải nén trên bộ nhớ máy khách.

### 5.5 Ngữ nghĩa API của Control Plane
- Ngoài endpoint `/v1/authorizations/applyToken` đã được ghi nhận, cần văn bản đặc tả chi tiết giao diện lập trình REST/RPC của Application Open Platform đối với các thao tác tự động hóa: phê duyệt bản phát hành, điều chỉnh tỷ lệ canary, tạm dừng phân phối hoặc gỡ bỏ khẩn cấp.

### 5.6 Giao thức Host Bridge & Xử lý lỗi
- Cần xác nhận cấu trúc thông điệp trao đổi giữa WebView và Native host: định dạng payload, cơ chế gắn ID tương quan, quy tắc đặt timeout và phương thức hủy lời gọi bất đồng bộ.
- **Chưa tìm được nguồn xác thực cho định dạng payload cụ thể:** Không có tài liệu nào của hãng nêu JSON-RPC hay bất kỳ định dạng nào khác; POC resource sheet cũng không định nghĩa hợp đồng này. Cần hãng xác nhận trước khi đưa vào hợp đồng chính thức.

### 5.7 Tầng Cache & Hợp đồng Ngoại tuyến (Offline)
- Cần kiểm chứng cơ chế lưu đệm native của WindVane: chính sách vô hiệu hóa bộ nhớ đệm (cache invalidation) khi phát hành bản mới, quy tắc phục vụ nội dung cũ khi mất mạng (stale serving) và khả năng khởi chạy độc lập khi không có kết nối. POC sheet không giải quyết các cơ chế này.

### 5.8 Khả năng tiếp cận & Trợ năng Native
- Cần đo kiểm mức độ hỗ trợ của container WindVane đối với các công cụ trợ năng hệ thống: chuyển đổi sang cây trợ năng native (accessibility tree), điều hướng tiêu điểm bàn phím và phản hồi qua TalkBack (Android) và VoiceOver (iOS). POC sheet không đề cập đến trợ năng.

---

## 6. Khoảng trống chưa lấp

Bốn vấn đề kiến trúc cốt lõi trích từ `report/06-gaps-validation-plan.md` vẫn **chưa được giải quyết** bởi POC resource sheet và cần tập trung xử lý trong kế hoạch đo kiểm thực địa:

1. **Manifest & Danh mục quyền JSAPI đặc thù:** Các trường manifest đặc thù của WindVane/uni-app, bảng phân loại quyền hạn JSAPI, ma trận tương thích SDK host mở rộng và trần dung lượng gói chưa có tài liệu đối chiếu công khai đầy đủ; bắt buộc phải thực hiện đo kiểm thực tế trong lab POC.
2. **Hợp đồng Host Bridge chi tiết:** Cầu nối giao tiếp host hiện chưa được ánh xạ hoàn toàn sang hợp đồng JSAPI thực tế của Alibaba/WindVane, bao gồm: siêu dữ liệu origin/frame, vòng đời handler/port, định dạng request/reply/error có cấu trúc, và ngữ nghĩa hủy gọi/hết hạn (cancellation & timeout).
3. **Bộ nhớ đệm & Hợp đồng chạy ngoại tuyến (Offline):** Các Web API tiêu chuẩn mới chỉ cung cấp nguyên thủy cơ bản mà chưa tạo thành hợp đồng ngoại tuyến hoàn chỉnh xuyên suốt runtime; tầng cache native của WindVane/uni-app, cơ chế vô hiệu hóa cache, phục vụ dữ liệu cũ (stale serving) và khởi chạy không mạng vẫn là các yếu tố chưa xác định.
4. **Bằng chứng Trợ năng trên thiết bị thật:** Hồ sơ bằng chứng trợ năng hiện mới dừng lại ở mức hồ sơ kiểm soát lý thuyết, chưa chứng minh được cầu nối host Alibaba/WindVane và các mini app đối tác đáp ứng trọn vẹn chuẩn WCAG; hành vi trên cây trợ năng native, điều hướng bàn phím và trình đọc màn hình cần kiểm chứng thực tế trên thiết bị.

---

## 7. Tham chiếu

### 7.1 Dữ liệu cấu hình từ POC Resource Sheet (Tab `gid=58086397`)
- **Console Application Open Platform:** `https://poc.superapp-intl.com/superapp#/login`
- **Container Init Service Address:** `https://poc.superapp-intl.com`
- **Application Open Platform Open API Address:** `https://poc.superapp-intl.com`
- **IDE Extension Resource:** `emas-mini-app-plugin-0.1.1.vsix` tại `https://poc.superapp-intl.com/portal/#/resource`
- **Maven Dependency (Android):** `com.aliyun.emas.suite.foundation:mini-app-adapter:1.8.9.2`, `com.aliyun.emas.suite.foundation:windvane-mini-app:1.8.9.2` tại `https://nexus-console.superapp-intl.com/repository/maven`
- **CocoaPods Dependency (iOS):** `pod 'EMASServiceManager'`, `pod 'EMASMiniAppAdapter', '1.1.3'`, `pod 'EMASWindVaneMiniApp', '1.2.4'` tại repo Gitlab `https://gitlab-console.superapp-intl.com/emas-ios/emas-specs.git`
- **Demo Projects:**
  - Android: `https://gitlab-console.superapp-intl.com/SuperAppDemo/emas-superapp-android` (branch `master`)
  - iOS: `https://gitlab-console.superapp-intl.com/SuperAppDemo/EMAS-Superapp-iOS` (branch `master`)
  - Flutter: `https://gitlab-console.superapp-intl.com/SuperAppDemo/flutter_windvane_demo` (branch `main`)
  *(Ghi chú: Thông tin xác thực truy cập tài nguyên demo được lưu giữ nội bộ trong POC resource sheet — không xuất hiện trong tài liệu).*

### 7.2 Danh mục 20 tài liệu chính thức từ Tab `gid=1488955061` (Alibaba Cloud Help Center)
1. [Introduction to SuperApp](https://www.alibabacloud.com/help/en/superapp/latest/introduction-to-superapp-1) — Giới thiệu tổng quan kiến trúc và giải pháp SuperApp.
2. [Android Access Guide](https://www.alibabacloud.com/help/en/superapp/latest/android-access) — Hướng dẫn tích hợp SDK native Android và 22 lớp JSBridge `WV*`.
3. [Android SDK Release Notes](https://www.alibabacloud.com/help/en/superapp/latest/android-sdk-release-notes-1) — Lịch sử các phiên bản phát hành của Android SDK.
4. [iOS Access Guide](https://www.alibabacloud.com/help/en/superapp/latest/ios-access) — Hướng dẫn tích hợp CocoaPods trên iOS và cấu hình khởi tạo.
5. [iOS SDK Release Notes](https://www.alibabacloud.com/help/en/superapp/latest/ios-sdk-release-notes-1) — Lịch sử các phiên bản phát hành của iOS SDK.
6. [Flutter Access Guide](https://www.alibabacloud.com/help/en/superapp/latest/flutter-access-1) — Tích hợp WindVane container trên Flutter qua `MethodChannel`.
7. [React Native App Access WindVane Applet Container](https://www.alibabacloud.com/help/en/superapp/latest/reactnative-app-access-windvane-applet-container-1) — Tích hợp container vào React Native host app.
8. [Developer Guide (for Developers) Overview](https://www.alibabacloud.com/help/en/superapp/latest/overview) — Tài liệu hướng dẫn tổng quan dành cho nhà phát triển miniapp.
9. [Developer Guide (for Platform Operators) Overview](https://www.alibabacloud.com/help/en/superapp/latest/overview-2) — Tài liệu hướng dẫn dành cho đơn vị vận hành nền tảng SuperApp.
10. [JSAPI wv.getAuthCode User Authorization](https://www.alibabacloud.com/help/en/superapp/latest/wv-getauthcode-1) — Đặc tả API bề mặt JavaScript `wv.getAuthCode` xin ủy quyền người dùng.
11. [Open API /v1/authorizations/applyToken](https://www.alibabacloud.com/help/en/superapp/latest/v1-authorizations-applytoken-1) — Đặc tả Open API server cấp đổi `authCode` lấy token và hồ sơ người dùng.
12. [SuperApp Technical Standards Implementation Best Practices](https://www.alibabacloud.com/help/en/superapp/latest/superapp-technical-standards-implementation-best-practices-1) — Thực tiễn triển khai chuẩn kỹ thuật SuperApp.
13. [H5 Application Released as SuperApp Applet](https://www.alibabacloud.com/help/en/superapp/latest/use-cases/h5-application-released-as-superapp-applet) — Trường hợp phát hành ứng dụng HTML5 dưới dạng miniapp.
14. [Invite Third-Party Space User Guide](https://www.alibabacloud.com/help/en/superapp/latest/user-guide/invite-thirdparty-space) — Hướng dẫn mời không gian phát triển của đối tác bên thứ ba.
15. [Invite Members to Space User Guide](https://www.alibabacloud.com/help/en/superapp/latest/user-guide/invite-members-to-space) — Hướng dẫn phân quyền thành viên vào không gian làm việc.
16. [Associated EMAS](https://www.alibabacloud.com/help/en/superapp/latest/associated-emas-1) — Hướng dẫn liên kết giải pháp SuperApp với hạ tầng Alibaba Cloud EMAS.
17. [Debugging MiniApp](https://www.alibabacloud.com/help/en/superapp/latest/debugging) — Quy trình kiểm thử và gỡ lỗi miniapp trên thiết bị thật và IDE.
18. [Package Build](https://www.alibabacloud.com/help/en/superapp/latest/package-build-1) — Quy trình biên dịch mã nguồn và đóng gói tài nguyên `dist/`.
19. [Plug-in Development for MiniApp Data Reporting](https://www.alibabacloud.com/help/en/superapp/latest/use-cases/plug-in-development-plug-in-development-for-small-program-data-reporting) — Phát triển plugin ghi nhận và báo cáo dữ liệu viễn trắc (telemetry).
20. [Release Version User Guide](https://www.alibabacloud.com/help/en/superapp/latest/user-guide/release-version) — Hướng dẫn quy trình phê duyệt phát hành, Whitelist, Canary và Official release.

### 7.3 Báo cáo & Đặc tả nội bộ
- `report/01-executive-summary.md` § Fit to the Alibaba POC
- `report/06-gaps-validation-plan.md` § POC Validation Checklist & Architecture Gaps
- `spec/SPEC-01-package-manifest-addressing.md`
- `spec/SPEC-02-runtime-lifecycle-bridge.md`
- `spec/SPEC-04-security-sandboxing.md`
- `spec/SPEC-05-supply-chain-signing.md`
- `spec/SPEC-06-capabilities-permissions.md`
- `spec/SPEC-08-store-control-plane.md`
- `spec/SPEC-10-reliability-performance.md`
- `spec/SPEC-11-compliance-regionalization.md`
- `spec/SPEC-12-review-guidelines.md`
- `spec/SPEC-13-control-plane-api.md`
- `spec/SPEC-14-windvane-jsapi-conformance.md`
- `spec/APPENDIX-A-capability-catalog.md`
- `spec/APPENDIX-B-conformance-checklist.md`
