# APPENDIX C — Bảng đối chiếu POC Alibaba/WindVane ↔ SPEC

| Trường | Giá trị |
|---|---|
| ID | `APPENDIX-C` |
| Phiên bản | 1.0.0 |
| Trạng thái | Draft |
| Mục đích | Đối chiếu từng thành phần POC Alibaba Cloud SuperApp/WindVane với yêu cầu SPEC |
| Nguồn | `report/01`, `report/06`, tài liệu Alibaba Cloud |

---

## 1. Mục đích & cách dùng

Tài liệu này cung cấp bảng đối chiếu chi tiết giữa hiện trạng kỹ thuật của **POC Alibaba Cloud SuperApp / WindVane** (bối cảnh tham chiếu dự án POC Viettel / Alibaba Cloud WindVane Mini App được phân tích trong `report/53`, `report/54`, `report/138`) và hệ thống yêu cầu kỹ thuật trong bộ đặc tả chuẩn hóa `SPEC-00` đến `SPEC-13`.

Bộ đặc tả của Super App được thiết kế theo nguyên tắc **trung lập nhà cung cấp** (vendor-neutral), kế thừa các tiêu chuẩn mở quốc tế từ W3C, WHATWG và IETF. Nhằm đảm bảo tính khả thi thực tế trong triển khai, POC Alibaba Cloud SuperApp / WindVane được sử dụng làm cơ sở đối chuẩn thực nghiệm ban đầu.

Theo định hướng chiến lược tại mục `## Fit to the Alibaba POC` trong `report/01-executive-summary.md`, các thành phần kỹ thuật được phân loại thành ba nhóm hành động chính:
- **KEEP (Giữ nguyên của POC):** Tận dụng tối đa các thành phần nền tảng sẵn có của Alibaba Cloud SuperApp và WindVane container đã chứng minh hiệu quả vận hành về runtime và control plane cơ bản.
- **ADD (Cần bổ sung quanh POC):** Thiết lập các lớp bao bọc độc lập về bảo mật chuỗi cung ứng, chữ ký số, quản trị định danh, kiểm định an ninh tự động và sổ cái kiểm toán để bảo đảm tính mở và an toàn toàn diện.
- **VALIDATE (Cần hãng xác nhận):** Đưa vào danh mục kiểm chứng thực địa các thông số kỹ thuật, lược đồ cấu hình nội bộ và hành vi runtime chưa có tài liệu công khai đầy đủ.

Nguyên tắc gán nhãn trạng thái và bằng chứng:
- **Đã xác nhận:** Thành phần đã được kiểm chứng qua tài liệu chính thức của Alibaba Cloud (kèm URL đối soát tại §7).
- **Cần hãng xác nhận:** Thành phần kỹ thuật nội bộ của hãng chưa công bố đầy đủ, bắt buộc đo kiểm hoặc yêu cầu văn bản xác nhận từ đội ngũ kỹ thuật Alibaba Cloud.
- **[PROPOSAL]:** Yêu cầu kiến trúc bổ sung độc lập từ bộ SPEC, hiện chưa có trong tính năng thương mại gốc của nhà cung cấp.

---

## 2. Bảng đối chiếu chính

| Thành phần POC | Mô tả (nguồn Alibaba) | Yêu cầu SPEC tương ứng | Trạng thái | Hành động |
|---|---|---|---|---|
| Application Open Platform | Control plane quản lý vòng đời miniapp, phiên bản, kênh phân phối | `REQ-08-001`, `REQ-08-011` | Đã xác nhận [1] | KEEP |
| WindVane Container Runtime | Container WebView tối ưu nhúng vào native app Android/iOS để chạy miniapp | `REQ-02-001`, `REQ-02-004` | Đã xác nhận [2, 3] | KEEP |
| uni-app Container Runtime | Lựa chọn container thứ hai trên nền tảng Alibaba Cloud EMAS | `REQ-02-001`, `REQ-08-024` | Đã xác nhận [1] | KEEP |
| SDK Android WindVane | Thư viện native `com.aliyun.emas.suite.foundation:windvane-mini-app:1.4.0` | `REQ-02-026` | Đã xác nhận [2] | KEEP |
| SDK iOS WindVane | Các pod CocoaPods: `EMASMiniAppAdapter`, `EMASWindVaneMiniApp`, `EMASUniappMiniApp` | `REQ-02-027` | Đã xác nhận [3] | KEEP |
| Cấu hình khởi tạo Host SDK | Tham số init: `accessKey`, `secretKey`, `host`, `appCode`, `useWindVane` | `REQ-02-002`, `SPEC-04` | Đã xác nhận [2, 3] | KEEP |
| SuperApp Miniapp Develop Tool | VS Code extension hỗ trợ scaffolding, phát triển và đóng gói | `REQ-01-001`, `REQ-01-005` | Đã xác nhận [1, 5] | KEEP |
| Quy trình đóng gói bản build | Lệnh `Publish MiniApp`, chạy `npm run build`, nén thư mục `dist/` tải lên | `REQ-01-005` | Đã xác nhận [4] | KEEP |
| Xem trước bằng mã QR | Tính năng quét mã QR từ VS Code/Console để mở bản preview trên thiết bị | `REQ-08-019` | Đã xác nhận [1] | KEEP |
| Phê duyệt yêu cầu phát hành | Quy trình bắt buộc duyệt Version Release Request trước khi phân phối | `REQ-08-011`, `REQ-12-001` | Đã xác nhận [6] | KEEP |
| Thử nghiệm danh sách Whitelist | Thử nghiệm nội bộ theo danh sách user ID trước khi phát hành diện rộng | `REQ-08-019` | Đã xác nhận [6] | KEEP |
| Phân phối Canary (Gray Release) | Cơ chế phát hành theo tỷ lệ phần trăm người dùng (Set Canary Percentage) | `REQ-08-019`, `REQ-08-020` | Đã xác nhận [6] | KEEP |
| Phát hành chính thức (Official) | Phân phối toàn mạng, tự động lưu trữ (archive) phiên bản cũ cùng kênh | `REQ-08-011`, `REQ-08-028` | Đã xác nhận [6] | KEEP |
| 22 lớp API JSBridge cơ bản | 22 API class thuộc namespace `android.taobao.windvane.jsbridge.api.*` | `REQ-02-021`, `SPEC-03` | Đã xác nhận [2] | KEEP |
| Phân quyền gọi năng lực gốc | Cơ chế native capability authorization khi miniapp yêu cầu quyền hệ thống | `REQ-06-001`, `REQ-06-015` | Đã xác nhận [2, 3] | KEEP |
| Xác thực Chuỗi cung ứng & Ký số | Xác thực chữ ký TUF, Sigstore Cosign, chứng thực SLSA provenance và in-toto | `REQ-05-001`, `REQ-05-008` | [PROPOSAL] | ADD |
| Danh mục đã ký & Khóa Digest | Signed catalog độc lập, khóa digest SHA-256 bất biến cho mọi bản release | `REQ-05-004`, `REQ-08-008` | [PROPOSAL] | ADD |
| Quản lý Publisher, RBAC & KYBC | Định danh nhà phát triển, quy trình KYBC theo EU DSA, namespace độc quyền | `REQ-08-003`, `REQ-08-005` | [PROPOSAL] | ADD |
| Hợp đồng Manifest chuẩn W3C | Bổ sung `manifest.json` chuẩn W3C MiniApp, Permissions Policy độc lập | `REQ-01-001`, `REQ-01-002` | [PROPOSAL] | ADD |
| Cổng kiểm tra tĩnh tự động | Bộ cổng G1–G5: quét secret, quét malware, kiểm tra CSP, Trusted Types | `SPEC-04`, `APPENDIX-B` | [PROPOSAL] | ADD |
| Sổ cái kiểm toán bất biến | Append-only audit ledger lưu vết mọi thao tác duyệt, phát hành, đổi quyền | `REQ-08-010`, `REQ-07-009` | [PROPOSAL] | ADD |
| Cơ chế Rollback & Quarantine | Rollback một chạm về digest an toàn tốt nhất, cách ly khẩn cấp bản lỗi | `REQ-08-021`, `REQ-08-022` | [PROPOSAL] | ADD |
| Kế hoạch Ngừng hỗ trợ chuẩn hóa | Vòng đời Deprecation/Sunset có kế hoạch, thông báo máy đọc và dọn dữ liệu | `REQ-08-028`, `REQ-08-029` | [PROPOSAL] | ADD |
| Bộ giải quyết tương thích đa chiều | Compatibility resolver đánh giá ma trận SDK host, bridge API, OS, locale | `REQ-08-024`, `REQ-08-025` | [PROPOSAL] | ADD |
| Giám sát Store & Đo lường SLA | Hệ thống thu thập độc lập tỷ lệ crash, ANR, độ trễ và xử lý vi phạm SLA | `REQ-08-020`, `REQ-10-001` | [PROPOSAL] | ADD |
| Hồ sơ Bằng chứng Trợ năng | Bộ bằng chứng kiểm định tuân thủ tiêu chuẩn trợ năng WCAG 2.2 AA | `REQ-11-008`, `REQ-12-015` | [PROPOSAL] | ADD |
| Lược đồ Manifest nội bộ | Cấu trúc trường, khóa định danh và siêu dữ liệu đặc thù của WindVane/uni-app | `REQ-01-001`, `REQ-01-007` | Cần hãng xác nhận | VALIDATE |
| Phân loại quyền hạn JSAPI | Bảng phân cấp rủi ro (Permission Taxonomy) cho 22 lớp API WindVane | `REQ-02-021`, `REQ-06-003` | Cần hãng xác nhận | VALIDATE |
| Ma trận tương thích Host SDK | Dải tương thích Android API level, iOS target và xung đột thư viện native | `REQ-08-024`, `REQ-10-008` | Cần hãng xác nhận | VALIDATE |
| Giới hạn dung lượng gói tải lên | Trần dung lượng gói chính (main) và gói phụ (sub-package) khi nén và giải nén | `REQ-01-006`, `APPENDIX-B` | Cần hãng xác nhận | VALIDATE |
| Ngữ nghĩa API của Control Plane | Tài liệu đặc tả REST/RPC API phê duyệt, tạm dừng, rollback trên portal hãng | `REQ-08-011`, `SPEC-13` | Cần hãng xác nhận | VALIDATE |
| Đặc tả giao thức Host Bridge | Cấu trúc request/reply/error, correlation ID, timeout và hủy lệnh gọi | `REQ-02-022`, `REQ-02-025` | Cần hãng xác nhận | VALIDATE |
| Tầng Cache & Chạy ngoại tuyến | Cơ chế cache native, invalidation, stale serving và khởi chạy khi offline | `REQ-02-009`, `REQ-10-005` | Cần hãng xác nhận | VALIDATE |
| Khả năng Trợ năng của Container | Cây trợ năng native, điều hướng bàn phím, tương thích TalkBack/VoiceOver | `REQ-02-027`, `REQ-11-008` | Cần hãng xác nhận | VALIDATE |

---

## 3. Nhóm KEEP (giữ nguyên của POC)

Nhóm này bao gồm các năng lực cốt lõi sẵn có của nền tảng Alibaba Cloud SuperApp, đáp ứng tốt yêu cầu thực thi kỹ thuật và được giữ nguyên trong kiến trúc triển khai:

### 3.1 Control Plane & Quản lý vòng đời
- **Application Open Platform:** Giữ nguyên làm cổng quản trị vòng đời ứng dụng tập trung. Nền tảng chịu trách nhiệm tiếp nhận bản build, quản lý phiên bản, cấp phát mã `appCode`, và cấu hình điểm cuối phân phối cho host app.

### 3.2 Container Runtime nhúng đa nền tảng
- **WindVane & uni-app Runtime:** Duy trì container nhúng gốc trên Android và iOS. Mini app được thực thi trong môi trường WebView tối ưu hóa cao của Alibaba Cloud EMAS, hỗ trợ cả hai loại hình container tùy thuộc vào cấu hình `useWindVane` hoặc `useUniApp`.
- **SDK tích hợp Native:** Giữ nguyên SDK Android (`com.aliyun.emas.suite.foundation:windvane-mini-app:1.4.0`) và các CocoaPods iOS (`EMASMiniAppAdapter`, `EMASWindVaneMiniApp`, `EMASUniappMiniApp`).

### 3.3 Chu trình Đóng gói & Phát triển Developer
- **VS Code Extension & Scaffolding:** Tận dụng tiện ích mở rộng SuperApp Miniapp Develop Tool để tạo dự án mẫu, biên dịch (`npm run build`), nén thư mục `dist/`, tải lên qua lệnh `Publish MiniApp` và quét mã QR để kiểm tra trước trên thiết bị thật.

### 3.4 Quy trình Phân phối Staged Rollout
- **Phê duyệt & Điều phối phát hành:** Tận dụng quy trình bắt buộc duyệt Version Release Request trên portal. Áp dụng luồng phân phối tuần tự: kiểm thử Whitelist theo danh sách user ID → phát hành thử nghiệm Canary theo tỷ lệ phần trăm → phát hành chính thức Official Release và tự động lưu trữ phiên bản cũ trên cùng channel.

### 3.5 Bộ 22 lớp API JSBridge sẵn có
- **Thư viện API phần cứng:** Tận dụng 22 lớp JSBridge trong namespace `android.taobao.windvane.jsbridge.api.*` bao gồm: `WVBase`, `WVBattery`, `WVBluetooth`, `WVCamera`, `WVContacts`, `WVCookie`, `WVFile`, `WVImage`, `WVLocation`, `WVMotion`, `WVNativeDetector`, `WVNetwork`, `WVNotification`, `WVPrefetch`, `WVReporter`, `WVScreen`, `WVScreenCapture`, `WVSystem`, `WVUI`, `WVUIDialog`, `WVUIToast`, `WVVideo`.

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
- Xây dựng bộ giải quyết tương thích (Compatibility Resolver) độc lập để đánh giá ma trận thiết bị, phiên bản SDK host và dải API khả dụng trước khi cho phép tải ứng dụng.
- Thiết lập hệ thống đo kiểm độc lập tỷ lệ sập (crash rate), hiện tượng treo giao diện (ANR), độ trễ phản hồi bridge, và thu thập bằng chứng trợ năng theo tiêu chuẩn WCAG 2.2 AA.

---

## 5. Nhóm VALIDATE (cần hãng xác nhận)

Các nội dung kỹ thuật bắt buộc phải đo kiểm thực tế hoặc yêu cầu văn bản xác nhận chính thức từ Alibaba Cloud trong quá trình triển khai:

### 5.1 Cấu trúc Manifest nội bộ của WindVane/uni-app
- Cần làm rõ lược đồ schema chi tiết, các trường cấu hình bắt buộc và tùy chọn mà container đọc từ gói `dist/`, cách khai báo trang khởi chạy và các khóa cấu hình mạng.

### 5.2 Bảng phân loại quyền hạn JSAPI (Taxonomy)
- Cần xác nhận ma trận phân cấp rủi ro cho 22 lớp API WindVane: quyền nào cần người dùng đồng ý (runtime prompt), quyền nào được cấp tĩnh theo manifest, và cách máy chủ quản trị cấu hình danh sách cho phép (allowlist).

### 5.3 Ma trận tương thích Host SDK & Xung đột Native
- Cần xác định dải phiên bản hệ điều hành hỗ trợ (Android SDK min/target API level, iOS deployment target), danh mục thư viện phụ thuộc và kiểm tra xung đột với kiến trúc native hiện có của Super App host.

### 5.4 Giới hạn dung lượng gói phát hành
- Cần xác nhận trần dung lượng tệp ZIP khi upload lên Application Open Platform, giới hạn kích thước gói chính (main package) và các gói phụ (sub-packages) khi nén và sau khi giải nén trên bộ nhớ máy khách.

### 5.5 Ngữ nghĩa API của Control Plane
- Cần văn bản đặc tả chi tiết giao diện lập trình REST/RPC của Application Open Platform đối với các thao tác tự động hóa: phê duyệt bản phát hành, điều chỉnh tỷ lệ canary, tạm dừng phân phối hoặc gỡ bỏ khẩn cấp.

### 5.6 Giao thức Host Bridge & Xử lý lỗi
- Cần xác nhận cấu trúc thông điệp trao đổi giữa WebView và Native host: định dạng payload, cơ chế gắn ID tương quan, quy tắc đặt timeout và phương thức hủy lời gọi bất đồng bộ.
- **Chưa tìm được nguồn xác thực cho định dạng payload cụ thể** — không có tài liệu nào của hãng nêu JSON-RPC hay bất kỳ định dạng nào khác. Cần hãng xác nhận trước khi đưa vào hợp đồng.

### 5.7 Tầng Cache & Hợp đồng Ngoại tuyến (Offline)
- Cần kiểm chứng cơ chế lưu đệm native của WindVane: chính sách vô hiệu hóa bộ nhớ đệm (cache invalidation) khi phát hành bản mới, quy tắc phục vụ nội dung cũ khi mất mạng (stale serving) và khả năng khởi chạy độc lập khi không có kết nối.

### 5.8 Khả năng tiếp cận & Trợ năng Native
- Cần đo kiểm mức độ hỗ trợ của container WindVane đối với các công cụ trợ năng hệ thống: chuyển đổi sang cây trợ năng native (accessibility tree), điều hướng tiêu điểm bàn phím và phản hồi qua TalkBack (Android) và VoiceOver (iOS).

---

## 6. Khoảng trống chưa lấp

Các vấn đề kiến trúc cốt lõi trích từ `report/06-gaps-validation-plan.md` cần tập trung xử lý trong kế hoạch đo kiểm:

1. **Manifest & Danh mục quyền JSAPI đặc thù:** Các trường manifest đặc thù của WindVane/uni-app, bảng phân loại quyền hạn JSAPI, ma trận tương thích SDK host và trần dung lượng gói chưa có tài liệu đối chiếu công khai đầy đủ; bắt buộc phải thực hiện đo kiểm thực tế trong lab POC.
2. **Hợp đồng Host Bridge chi tiết:** Cầu nối giao tiếp host hiện chưa được ánh xạ hoàn toàn sang hợp đồng JSAPI thực tế của Alibaba/WindVane, bao gồm: siêu dữ liệu origin/frame, vòng đời handler/port, định dạng request/reply/error có cấu trúc, và ngữ nghĩa hủy gọi/hết hạn (cancellation & timeout).
3. **Bộ nhớ đệm & Hợp đồng chạy ngoại tuyến (Offline):** Các Web API tiêu chuẩn mới chỉ cung cấp nguyên thủy cơ bản mà chưa tạo thành hợp đồng ngoại tuyến hoàn chỉnh xuyên suốt runtime; tầng cache native của WindVane/uni-app, cơ chế vô hiệu hóa cache, phục vụ dữ liệu cũ (stale serving) và khởi chạy không mạng vẫn là các yếu tố chưa xác định.
4. **Bằng chứng Trợ năng trên thiết bị thật:** Hồ sơ bằng chứng trợ năng hiện mới dừng lại ở mức hồ sơ kiểm soát lý thuyết, chưa chứng minh được cầu nối host Alibaba/WindVane và các mini app đối tác đáp ứng trọn vẹn chuẩn WCAG; hành vi trên cây trợ năng native, điều hướng bàn phím và trình đọc màn hình cần kiểm chứng thực tế trên thiết bị.

---

## 7. Tham chiếu

### Tài liệu chính thức Alibaba Cloud SuperApp
1. [Rapid Development of WindVane Applets or H5 Applications](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/getting-started/rapid-development-of-windvane-applets-or-h5-applications) — Khởi tạo dự án, công cụ phát triển, đóng gói và phát hành.
2. [Android Access Guide](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/android-access-1) — Tích hợp SDK Android `windvane-mini-app:1.4.0`, cấu hình host, 22 lớp JSBridge API.
3. [iOS Access Guide](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/ios-access-11) — Tích hợp CocoaPods `EMASMiniAppAdapter`, `EMASWindVaneMiniApp`, cấu hình khởi tạo.
4. [Package Build](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/package-build-1) — Quy trình biên dịch và nén tài nguyên ứng dụng.
5. [Use of Scaffolding](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/use-of-scaffolding) — Sử dụng khung mẫu phát triển và tiện ích mở rộng VS Code.
6. [Release Version User Guide](https://www.alibabacloud.com/help/en/superapp/superapp-bap-public-intl/user-guide/release-version) — Quản lý phê duyệt phát hành, thử nghiệm Whitelist, Canary Release và Official Release.

### Báo cáo & Đặc tả nội bộ
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
- `spec/APPENDIX-A-capability-catalog.md`
- `spec/APPENDIX-B-conformance-checklist.md`
