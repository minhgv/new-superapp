# Chuyên Đề 49 (Iteration 53): WICG Sub Apps API, WICG Storage Buckets API, và WICG Local Network Access (LNA) / Private Network Access

## 1. Bối Cảnh & Mục Tiêu Nghiên Cứu
Tại Iteration 53, hệ thống nghiên cứu Deli Deep tiếp tục hoàn thiện khung chuẩn kỹ thuật cho mini app store trong siêu ứng dụng (Super App), tập trung giải quyết 3 thách thức kiến trúc và bảo mật cốt lõi:
1. **WICG Sub Apps API**: Chuẩn hoá cơ chế quản lý vòng đời ứng dụng con phân cấp (`navigator.subApps.add()`, `list()`, `remove()`) bên trong container siêu ứng dụng mẹ. Giải quyết triệt để sự thiếu hụt danh tính OS bản địa của các mini-app nguyên khối bằng cách ánh xạ các sub-app thành các thực thể cửa sổ, biểu tượng thanh tác vụ (taskbar/dock) và bộ xử lý giao thức (protocol handlers) độc lập mà vẫn chia sẻ cùng ranh giới bảo mật và khả năng suy giảm quyền hạn (capability attenuation) từ siêu ứng dụng mẹ.
2. **WICG Storage Buckets API**: Chuẩn hoá cơ chế phân vùng bộ nhớ cục bộ đa bucket (`navigator.storageBuckets.open()`), tách biệt dứt điểm dữ liệu giao dịch cốt lõi (IndexedDB/OPFS) với các kho đệm truyền thông phù du (CacheStorage). Cung cấp các chính sách độ bền ghi đĩa (`strict` vs `relaxed`) nhằm tối ưu hoá tuổi thọ chip nhớ flash và tiêu thụ pin trên di động, đi kèm cơ chế dọn dẹp theo thời gian sống (expiration TTL), thu hồi bộ nhớ từng phần khi chịu áp lực lưu trữ (storage pressure), và xoá dữ liệu nguyên tử (`delete()`) tuân thủ GDPR Điều 17.
3. **WICG Local Network Access (LNA) / Private Network Access (PNA)**: Chuẩn hoá ma trận phân loại không gian địa chỉ IP (`public`, `local` RFC 1918, `loopback`), ngăn chặn các cuộc tấn công CSRF và Blind SSRF từ mã độc mini-app trên đám mây nhắm vào các thiết bị IoT, router, và dịch vụ microservice nội bộ. Bắt buộc thực hiện bắt tay CORS preflight chuyên biệt (`Access-Control-Request-Local-Network`), ép buộc ngữ cảnh bảo mật HTTPS (Secure Contexts), ủy quyền chính sách nhúng (`Permissions-Policy: local-network`), và hộp thoại cấp quyền tường minh cho người dùng.

---

## 2. Chi Tiết Phát Hiện & Bằng Chứng Kỹ Thuật (Findings)

### Nhóm 1: WICG Sub Apps API & Kiến Trúc Vòng Đời Ứng Dụng Phân Cấp

#### finding: `sub_apps_053_01` — WICG Sub Apps API: Kiến Trúc Vòng Đời Phân Cấp, Cài Đặt và Gỡ Bỏ Trong Container Mẹ
- **Tiêu chuẩn tham chiếu**: WICG Sub Apps Specification Section 3 (API Description) & Section 4 (Installation Lifecycle).
- **Nguồn xác thực**: [WICG Sub Apps Specification](https://wicg.github.io/sub-apps/), [WICG Sub Apps Repository](https://github.com/WICG/sub-apps), [WICG Sub Apps Index](https://wicg.github.io/sub-apps/index.html).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Chuẩn hoá giao diện lập trình `navigator.subApps` dành cho các ứng dụng web/container được cài đặt trên thiết bị người dùng.
  - Cung cấp 3 phương thức bất đồng bộ cốt lõi:
    1. `navigator.subApps.add(install_options)`: Tiếp nhận từ điển định danh sub-app và URL manifest tương ứng, xác thực rằng mỗi sub-app phải thuộc cùng hệ thống phân cấp origin hoặc một phạm vi liên kết được xác thực bằng mật mã.
    2. `navigator.subApps.list()`: Trả về bản đồ tất cả các sub-app đã cài đặt kèm trạng thái vòng đời và tham số khởi chạy.
    3. `navigator.subApps.remove(sub_app_id)`: Gỡ bỏ sạch sẽ một sub-app và tự động thu hồi các tài nguyên cửa sổ/bộ nhớ liên kết.
  - Thiết lập mô hình phân cấp chính thức: Siêu ứng dụng đóng vai trò bộ điều phối gốc (root orchestrator) quản lý nhiều mô-đun trải nghiệm con.
- **Yêu cầu Store Standard**:
  - Runtime container của siêu ứng dụng phải cài đặt các hàm gốc tương đương Sub Apps API.
  - Khi người dùng chọn cài đặt mini-app từ store catalog, container gọi `navigator.subApps.add()` để đăng ký mini-app với hệ điều hành.
  - Khi gỡ bỏ siêu ứng dụng mẹ, hệ điều hành hoặc container phải tự động gỡ bỏ dây chuyền (cascade removal) toàn bộ các sub-app, ngăn chặn dữ liệu rác hoặc tiến trình ngầm bị mồ côi (orphaned storage/zombie processes).

#### finding: `sub_apps_053_02` — Mô Hình Bảo Mật Sub Apps: Khóa Phạm Vi Origin, Cách Ly Hộp Cát và Ranh Giới Thông Tin Xác Thực
- **Tiêu chuẩn tham chiếu**: WICG Sub Apps Explainer Section 4 (Security & Privacy Considerations).
- **Nguồn xác thực**: [WICG Sub Apps Repository](https://github.com/WICG/sub-apps), [WICG Sub Apps Specification](https://wicg.github.io/sub-apps/), [WICG Sub Apps Index](https://wicg.github.io/sub-apps/index.html).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Sub-app không thể tồn tại độc lập mà luôn gắn chặt với ngữ cảnh của ứng dụng mẹ (parent context).
  - API áp dụng cơ chế khóa phạm vi origin nghiêm ngặt (origin clamping): Ứng dụng mẹ không thể cài đặt một sub-app từ một origin bên ngoài phạm vi phân cấp của nó trừ khi các bắt tay liên kết origin mật mã hai chiều được xác minh hợp lệ.
  - Sub-app chia sẻ cùng ranh giới bảo mật cơ bản với ứng dụng mẹ nhưng bị cấm tuyệt đối việc tự gọi `navigator.subApps.add()` để cài đặt các ứng dụng con lồng nhau vô hạn (chống recursive nested application hijacking).
- **Yêu cầu Store Standard**:
  - Quy trình kiểm duyệt store quét tĩnh cấu trúc `manifest.json` của sub-app để đảm bảo các đường dẫn định tuyến nằm trong namespace được cấp phép.
  - Runtime của siêu ứng dụng chặn bắt lệnh gọi `subApps.add()` để xác thực chữ ký số của gói bundle sub-app trước khi giải nén và nạp mã.
  - Giao tiếp giữa các sub-app đồng cấp (sibling sub-apps) bắt buộc phải đi qua cầu nối thông điệp (host message bridge) đã được kiểm toán, ngăn chặn can thiệp DOM hoặc vùng nhớ trực tiếp.

#### finding: `sub_apps_053_03` — Tích Hợp OS Bản Địa Cho Sub Apps: Danh Tính Thanh Tác Vụ, Cửa Sổ Độc Lập & Điều Phối Giao Thức
- **Tiêu chuẩn tham chiếu**: WICG Sub Apps Specification Section 5 (OS Integration and Windowing).
- **Nguồn xác thực**: [WICG Sub Apps Specification](https://wicg.github.io/sub-apps/), [WICG Sub Apps Repository](https://github.com/WICG/sub-apps).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Giải quyết nhược điểm mất danh tính hệ điều hành trong các siêu ứng dụng đơn khối: Cho phép chỉ thị user agent đăng ký từng sub-app với OS như một thực thể cửa sổ riêng biệt.
  - Khi khởi chạy, trình quản lý cửa sổ OS hiển thị biểu tượng riêng của sub-app, màu sắc chủ đề tuỳ biến, và thanh tiêu đề độc lập trên thanh tác vụ (taskbar/dock) và bộ chuyển đổi tác vụ (Alt-Tab / App Switcher).
  - Sub-app có thể đăng ký các bộ xử lý giao thức (protocol handlers) và xử lý tệp (file handlers) trong phạm vi của mình, cho phép OS định tuyến deep link trực tiếp đến cửa sổ sub-app mà không cần mở qua màn hình chính của siêu ứng dụng mẹ.
- **Yêu cầu Store Standard**:
  - Container siêu ứng dụng trên desktop và mobile phải nối các hook cửa sổ của Sub Apps API vào shell của nền tảng native.
  - Các liên kết sâu (ví dụ: `telco-bank://pay?id=123`) kích hoạt mở trực tiếp cửa sổ sub-app tài chính tương ứng.
  - Container mẹ duy trì quyền giám sát vòng đời cấp cao: Tự động đóng/khoá cửa sổ sub-app nếu phiên đăng nhập của siêu ứng dụng hết hạn hoặc bước vào trạng thái khoá sinh trắc học.

#### finding: `sub_apps_053_04` — Liên Kết Khai Báo Trong Manifest Của Sub Apps: Khai Báo Khối, Khối Phân Tích & Xác Thực Tính Toàn Vẹn
- **Tiêu chuẩn tham chiếu**: WICG Sub Apps Specification Section 6 (Web App Manifest Extensions).
- **Nguồn xác thực**: [WICG Sub Apps Repository](https://github.com/WICG/sub-apps), [WICG Sub Apps Specification](https://wicg.github.io/sub-apps/).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Bổ sung thành viên khai báo mở rộng `sub_apps` trong Web App Manifest của ứng dụng mẹ.
  - Cho phép khai báo mảng các bộ mô tả sub-app được chứng thực (ví dụ: `{ "id": "/mini/wallet", "manifest_url": "/mini/wallet/manifest.webmanifest", "install_mode": "on-demand" }`).
  - Cho phép user agent nạp trước (pre-cache) vỏ mã nguồn (app shell) của các sub-app thiết yếu hoặc hiển thị danh sách các phân hệ tính năng trực tiếp trong hộp thoại cài đặt ứng dụng mẹ của trình duyệt.
- **Yêu cầu Store Standard**:
  - Bộ biên dịch danh mục store tự động sinh các ánh xạ manifest sub-app được ký số.
  - Bảng băm (cryptographic hashes) trong manifest mẹ ngăn chặn hành vi giả mạo URL hoặc chuyển hướng MITM sang máy chủ bên thứ ba.
  - Dịch vụ cập nhật phiên bản của store đánh giá độ lệch phiên bản giữa manifest mẹ và các sub-app trong một giao dịch xác thực nguyên tử duy nhất.

#### finding: `sub_apps_053_05` — Suy Giảm Năng Lực Sub Apps: Thừa Kế Quyền Hạn, Thu Hồi Theo Phạm Vi & Hồ Sơ Năng Lực
- **Tiêu chuẩn tham chiếu**: WICG Sub Apps Explainer Section 5 (Permissions and Device Access).
- **Nguồn xác thực**: [WICG Sub Apps Specification](https://wicg.github.io/sub-apps/), [WICG Sub Apps Repository](https://github.com/WICG/sub-apps), [WICG Sub Apps Index](https://wicg.github.io/sub-apps/index.html).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Thiết lập nguyên tắc suy giảm năng lực (capability attenuation): Trần quyền hạn tối đa của một sub-app luôn bị giới hạn bởi quyền hạn đã được cấp cho siêu ứng dụng mẹ.
  - Nếu người dùng thu hồi quyền Camera của siêu ứng dụng mẹ, tất cả các sub-app bên dưới lập tức mất quyền truy cập camera mà không có ngoại lệ.
  - Ngược lại, một sub-app có thể bị giới hạn trong một hồ sơ năng lực hẹp (chỉ được lưu trữ, không được định vị GPS hoặc camera) ngay cả khi siêu ứng dụng mẹ có đầy đủ các quyền này.
  - Hộp thoại quyền hạn người dùng hiển thị thuộc tính kép minh bạch (ví dụ: "Mini App X bên trong Siêu Ứng Dụng Y yêu cầu quyền truy cập Camera").
- **Yêu cầu Store Standard**:
  - Bảng điều khiển nhà phát triển của store gán hồ sơ năng lực dựa trên vai trò cho từng sub-app.
  - Cầu nối native của container chặn bắt và kiểm tra trạng thái uỷ quyền của cả siêu ứng dụng mẹ và sub-app đối với mọi yêu cầu truy cập phần cứng.
  - Giao diện cài đặt quyền riêng tư của siêu ứng dụng cung cấp các công tắc bật/tắt quyền riêng biệt cho từng sub-app.

---

### Nhóm 2: WICG Storage Buckets API & Phân Vùng Lưu Trữ Cục Bộ

#### finding: `storage_buckets_053_01` — WICG Storage Buckets API: Tổ Chức Đa Bucket, Ranh Giới Thu Hồi Độc Lập & Phân Đoạn API
- **Tiêu chuẩn tham chiếu**: WICG Storage Buckets Specification Section 3 (Storage Buckets Infrastructure) & Section 4 (API).
- **Nguồn xác thực**: [WICG Storage Buckets Specification](https://wicg.github.io/storage-buckets/), [WICG Storage Buckets Explainer](https://wicg.github.io/storage-buckets/explainer.html), [WICG Storage Buckets Repository](https://github.com/WICG/storage-buckets).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Khắc phục hạn chế của chuẩn Storage Standard cũ (vốn coi lưu trữ của một origin là một khối đơn nhất, khi xoá sẽ xoá sạch toàn bộ dữ liệu người dùng lẫn cache phù du).
  - Phương thức `navigator.storageBuckets.open(name, { durability, quota, expires })` khởi tạo các hộp cát lưu trữ có tên độc lập.
  - Mỗi bucket chứa các thể hiện riêng biệt của IndexedDB (`bucket.indexedDB`), Cache API (`bucket.caches`), và Origin Private File System (`bucket.getDirectory()`).
  - Cho phép tách biệt rạch ròi dữ liệu giao dịch cốt lõi với bộ nhớ đệm hình ảnh tạm thời.
- **Yêu cầu Store Standard**:
  - Runtime của mini-app bắt buộc phân vùng theo bucket: bucket `critical-state` (biểu mẫu, bản nháp, hạt giống xác thực) và bucket `ephemeral-media` (banner, ảnh thu nhỏ sản phẩm).
  - Container siêu ứng dụng cô lập các mini-app đồng cấp vào các không gian tên bucket riêng biệt.
  - Công cụ giám sát của store sử dụng `navigator.storageBuckets.keys()` để đo lường footprint dung lượng của từng bucket.

#### finding: `storage_buckets_053_02` — Chính Sách Độ Bền Của Storage Buckets: Điều Khiển Ghi Đĩa 'strict' vs 'relaxed' & Tối Ưu Hóa Hao Mòn Flash/Pin
- **Tiêu chuẩn tham chiếu**: WICG Storage Buckets Explainer Section 2 (Durability Options).
- **Nguồn xác thực**: [WICG Storage Buckets Explainer](https://wicg.github.io/storage-buckets/explainer.html), [WICG Storage Buckets Specification](https://wicg.github.io/storage-buckets/), [Chrome Platform Status Feature 5739224579964928](https://chromestatus.com/feature/5739224579964928).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Chuẩn hoá tuỳ chọn độ bền `durability` khi tạo bucket:
    1. `strict`: Đảm bảo các thao tác ghi dữ liệu được cam kết vật lý xuống chip nhớ flash (fsync) trước khi Promise hoàn thành. Bắt buộc cho các giao dịch tài chính và sổ cái ngoại tuyến nhằm ngăn chặn hư hỏng dữ liệu khi mất nguồn đột ngột.
    2. `relaxed`: Cho phép user agent đệm các thao tác ghi vào bộ nhớ RAM của OS và xả xuống đĩa theo từng đợt trễ. Tối ưu hoá thông lượng cho dữ liệu viễn trắc tần suất cao, bộ đệm bản đồ và tài nguyên game, đồng thời bảo vệ tuổi thọ chip nhớ NAND và tiết kiệm pin di động.
- **Yêu cầu Store Standard**:
  - Quy chuẩn dữ liệu của store yêu cầu mini-app tài chính/thanh toán phải khai báo `durability: strict` cho sổ cái đơn hàng.
  - Mini-app truyền thông, streaming, và game bị giới hạn ở mức `durability: relaxed` đối với các tầng cache nhằm tránh gây hao mòn bộ nhớ thiết bị.
  - Nhật ký kiểm toán container ghi lại các khai báo độ bền để đối chiếu với hồ sơ phân loại mini-app.

#### finding: `storage_buckets_053_03` — Thu Hồi Bộ Nhớ Từng Phần & Thời Gian Hết Hạn: Quản Lý TTL Tự Động và Ưu Tiên Áp Lực Bộ Nhớ
- **Tiêu chuẩn tham chiếu**: WICG Storage Buckets Specification Section 5 (Bucket Eviction and Expiration Algorithms).
- **Nguồn xác thực**: [WICG Storage Buckets Specification](https://wicg.github.io/storage-buckets/), [WICG Storage Buckets Explainer](https://wicg.github.io/storage-buckets/explainer.html), [MDN Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Cho phép tạo bucket kèm thời hạn hết hạn tường minh: `navigator.storageBuckets.open('promo_cache', { expires: Date.now() + 86400000 })`.
  - Bộ máy thu hồi bộ nhớ của trình duyệt tự động quét và xoá sạch các bucket đã hết hạn trong các chu kỳ nghỉ (idle cycles) mà không gây mất dữ liệu toàn bộ origin.
  - Khi thiết bị cạn kiệt dung lượng đĩa (storage pressure), thuật toán LRU sẽ đánh giá và thu hồi từng bucket đơn lẻ: Xoá các bucket cache tạm thời trước, giữ nguyên vẹn các bucket chứa văn bản và trạng thái người dùng.
- **Yêu cầu Store Standard**:
  - Các chiến dịch marketing ngắn hạn và mini-app khuyến mãi theo mùa bắt buộc phải cấu hình `expires` phù hợp với ngày kết thúc chiến dịch.
  - Trình quản lý bộ nhớ của container đón bắt tín hiệu cảnh báo bộ nhớ thấp từ OS để chủ động dọn dẹp các bucket phù du của mini-app.
  - Ngăn ngừa hoàn toàn nguy cơ ứng dụng bị crash do đầy bộ nhớ thiết bị mà không làm gián đoạn phiên đăng nhập của người dùng.

#### finding: `storage_buckets_053_04` — Viễn Trắc Hạn Mức Và Mức Độ Sử Dụng: Hàm 'estimate()' Cho Từng Bucket và Ngân Sách Lưu Trữ
- **Tiêu chuẩn tham chiếu**: WICG Storage Buckets Specification Section 6 (Bucket Storage Estimate).
- **Nguồn xác thực**: [WICG Storage Buckets Specification](https://wicg.github.io/storage-buckets/), [MDN Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API), [WICG Storage Buckets Repository](https://github.com/WICG/storage-buckets).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Mỗi đối tượng bucket cung cấp phương thức `bucket.estimate()`, trả về Promise phân giải thành `{ usage: number, quota: number }`.
  - Cung cấp khả năng hiển thị chi tiết số byte đang sử dụng của từng cơ sở dữ liệu IndexedDB hoặc phân vùng cache cụ thể.
  - Hỗ trợ yêu cầu trần hạn mức khi mở bucket: `navigator.storageBuckets.open('temp', { quota: 50 * 1024 * 1024 })`, ngăn chặn các tài nguyên rác phình to mất kiểm soát.
- **Yêu cầu Store Standard**:
  - Cầu nối lưu trữ của container áp đặt giới hạn dung lượng cứng (ví dụ: 50MB cho mini-app tiện ích, 200MB cho game offline).
  - Hệ thống viễn trắc giám sát số liệu từ `bucket.estimate()`, phát cảnh báo sớm cho nhà phát triển trước khi chạm ngưỡng lỗi `QuotaExceededError`.
  - Giao diện cài đặt bộ nhớ của siêu ứng dụng hiển thị biểu đồ phân rã dung lượng theo từng bucket, cho phép người dùng chủ động xoá cache hình ảnh mà vẫn giữ dữ liệu tài khoản.

#### finding: `storage_buckets_053_05` — Hợp Đồng Vòng Đời & Xoá Dữ Liệu: Hàm 'navigator.storageBuckets.delete()' và Thu Dọn Nguyên Tử
- **Tiêu chuẩn tham chiếu**: WICG Storage Buckets Specification Section 7 (Bucket Deletion Operations).
- **Nguồn xác thực**: [WICG Storage Buckets Repository](https://github.com/WICG/storage-buckets), [WICG Storage Buckets Specification](https://wicg.github.io/storage-buckets/), [WICG Storage Buckets Explainer](https://wicg.github.io/storage-buckets/explainer.html).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Phương thức bất đồng bộ `navigator.storageBuckets.delete(name)` thực hiện xoá nguyên tử một bucket cùng toàn bộ các tài nguyên bên trong (IndexedDB, CacheStorage, thư mục OPFS).
  - Khi bắt đầu xoá, các kết nối đang mở đến DB trong bucket nhận sự kiện `close`, các giao dịch đang chờ xử lý bị huỷ bỏ (abort), và thư mục vật lý trên đĩa bị huỷ liên kết (unlink).
  - Trả về Promise phân giải thành `true` khi hoàn tất hoặc `false` nếu bucket không tồn tại, loại trừ nguy cơ race condition trong quá trình dọn dẹp.
- **Yêu cầu Store Standard**:
  - Khi người dùng đăng xuất khỏi mini-app hoặc phiên làm việc của tenant, các bucket lưu trữ theo tenant lập tức bị xoá sạch qua lệnh `delete()`.
  - Quy trình cập nhật phiên bản mini-app tự động xoá các bucket của phiên bản cũ sau khi hoàn tất di chuyển dữ liệu (migration).
  - Đáp ứng nghiêm ngặt quyền được lãng quên (Right to Erasure) theo GDPR Điều 17 và Nghị định 13/2023/NĐ-CP của Việt Nam.

---

### Nhóm 3: WICG Local Network Access (LNA) / Private Network Access

#### finding: `lna_sec_053_01` — Phân Loại Không Gian Địa Chỉ IP Của WICG LNA: 'public', 'local', 'loopback' và Ranh Giới Lưu Lượng Egress
- **Tiêu chuẩn tham chiếu**: WICG Local Network Access Section 3 (IP Address Spaces).
- **Nguồn xác thực**: [WICG Local Network Access Specification](https://wicg.github.io/local-network-access/), [WICG Private Network Access](https://wicg.github.io/private-network-access/), [MDN Local Network Access](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Local_network_access).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Phân loại toàn bộ các endpoint mạng thành 3 không gian địa chỉ riêng biệt:
    1. `public`: Các địa chỉ IP có thể định tuyến công khai trên Internet toàn cầu.
    2. `local`: Các dải địa chỉ mạng riêng IPv4 theo RFC 1918 (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`), Carrier-Grade NAT RFC 6598 (`100.64.0.0/10`), IPv6 Unique Local RFC 4193, và link-local RFC 3927.
    3. `loopback`: `127.0.0.0/8`, `::1`, và các socket domain nội bộ.
  - Quy tắc phân cấp: Một trang web hoặc mini-app thực thi trong không gian ít riêng tư hơn (ví dụ: CDN công khai) bị nghiêm cấm gửi yêu cầu đến không gian riêng tư hơn (mạng LAN nội bộ hoặc loopback) trừ khi có sự đồng thuận mật mã tường minh.
- **Yêu cầu Store Standard**:
  - WebView container của siêu ứng dụng phải cài đặt các chốt chặn không gian địa chỉ LNA.
  - Mini-app tải từ máy chủ đám mây bị chặn mặc định khi cố gắng quét cổng hoặc kết nối đến thiết bị IoT trong mạng Wi-Fi gia đình hoặc mạng nội bộ doanh nghiệp.
  - Ngăn chặn hoàn toàn các cuộc tấn công CSRF / SSRF nhắm vào router văn phòng và cơ sở dữ liệu nội bộ.

#### finding: `lna_sec_053_02` — Giao Thức Bắt Tay CORS Preflight Của LNA: Header 'Access-Control-Request-Local-Network' và Sự Đồng Thuận Của Máy Chủ Đích
- **Tiêu chuẩn tham chiếu**: WICG Local Network Access Section 4 (Preflight Requests & Headers).
- **Nguồn xác thực**: [MDN Local Network Access](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Local_network_access), [WICG Local Network Access Specification](https://wicg.github.io/local-network-access/), [WICG Private Network Access](https://wicg.github.io/private-network-access/).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Để ngăn chặn việc gửi các payload mù (Blind CSRF) đến các thiết bị mạng cục bộ có lỗ hổng (như router gia đình có API không cần xác thực), LNA yêu cầu user agent gửi yêu cầu OPTIONS preflight trước khi thực hiện bất kỳ lệnh fetch hoặc WebSocket nào qua ranh giới không gian địa chỉ.
  - Yêu cầu preflight bắt buộc đính kèm header: `Access-Control-Request-Local-Network: true`.
  - Thiết bị hoặc máy chủ cục bộ phải chủ động cho phép bằng cách phản hồi với header: `Access-Control-Allow-Local-Network: true` cùng các header CORS thông thường.
  - Nếu thiết bị không gửi header này, user agent lập tức huỷ bỏ yêu cầu với lỗi mạng, không bao giờ phát tán payload thực tế đến thiết bị.
- **Yêu cầu Store Standard**:
  - Mini-app kết nối đến các thiết bị biên tại cơ sở (on-premise edge gateway, POS thanh toán) phải đáp ứng đầy đủ bắt tay CORS LNA.
  - Hệ thống kiểm duyệt tự động của store quét mã nguồn gói bundle nhằm phát hiện các hành vi thăm dò IP mạng cục bộ không khai báo.
  - Tài liệu hướng dẫn nhà phát triển cung cấp mẫu cấu hình máy chủ nội bộ với header `Access-Control-Allow-Local-Network: true`.

#### finding: `lna_sec_053_03` — Bắt Buộc Ngữ Cảnh Bảo Mật (Secure Context) Cho LNA: Ép Buộc HTTPS và Triệt Tiêu Nội Dung Hỗn Hợp
- **Tiêu chuẩn tham chiếu**: WICG Local Network Access Section 5 (Secure Contexts Enforcement).
- **Nguồn xác thực**: [WICG Local Network Access Specification](https://wicg.github.io/local-network-access/), [MDN Local Network Access](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Local_network_access), [WICG Private Network Access](https://wicg.github.io/private-network-access/).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Quy tắc bất biến: Chỉ các Ngữ cảnh Bảo mật (Secure Contexts - các trang phục vụ qua HTTPS hoặc từ loopback đã được xác thực) mới được phép kích hoạt yêu cầu đến các không gian mạng riêng tư hơn.
  - Một origin HTTP không an toàn nếu cố tình gọi một IP mạng cục bộ sẽ bị trình duyệt chặn ngay lập tức, kể cả khi thiết bị đích có trả về header cho phép.
  - Triệt tiêu hoàn toàn vector tấn công man-in-the-middle trên mạng Wi-Fi công cộng (kẻ tấn công chèn mã độc vào luồng HTTP cleartext để tấn công leo thang vào mạng LAN của nạn nhân).
- **Yêu cầu Store Standard**:
  - Cổng kiểm duyệt store từ chối ngay lập tức mọi mini-app sử dụng giao thức HTTP không mã hoá.
  - Container siêu ứng dụng tắt hoàn toàn cờ cho phép nội dung hỗn hợp (mixed content).
  - Đảm bảo các cơ chế phòng vệ của LNA không bao giờ bị vô hiệu hoá do suy thoái giao thức truyền tải.

#### finding: `lna_sec_053_04` — Hộp Thoại Cấp Quyền Người Dùng LNA: Công Bố Ranh Giới Thiết Bị và Ngăn Chặn Thăm Dò Cổng Ngầm
- **Tiêu chuẩn tham chiếu**: WICG Local Network Access Section 6 (Permission Integration & Prompts).
- **Nguồn xác thực**: [WICG Local Network Access Specification](https://wicg.github.io/local-network-access/), [MDN Local Network Access](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Local_network_access).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Tích hợp với khung Web Permissions API: Khi một trang web công khai cố gắng tương tác lần đầu với một địa chỉ IP mạng nội bộ, user agent tạm dừng thực thi và hiển thị hộp thoại cảnh báo: "Ứng dụng X muốn kết nối với các thiết bị trên mạng cục bộ của bạn (192.168.1.50)".
  - Người dùng có thể chọn cho phép một lần, cho phép vĩnh viễn, hoặc từ chối chặn đứng yêu cầu.
  - Ngăn chặn hoàn toàn hành vi quét cổng ngầm (port scanning) dựa trên việc đo thời gian phản hồi gói tin TCP SYN/RST hoặc lỗi mạng fetch để vẽ bản đồ topology mạng nội bộ.
- **Yêu cầu Store Standard**:
  - Container siêu ứng dụng hiển thị hộp thoại cấp quyền native mô tả rõ địa chỉ IP đích và danh tính mini-app yêu cầu.
  - Trung tâm cài đặt quyền riêng tư của siêu ứng dụng duy trì danh sách cấp phép mạng cục bộ theo từng mini-app, cho phép thu hồi bất kỳ lúc nào.
  - Phát hiện các hành vi quét cổng hàng loạt để tự động kích hoạt cảnh báo vi phạm bảo mật, đình chỉ mini-app vi phạm ngay lập tức.

#### finding: `lna_sec_053_05` — Điều Khiển Bằng Permissions Policy: Chỉ Thị 'local-network' và Quản Lý Phân Quyền Khung Nhúng (Iframe)
- **Tiêu chuẩn tham chiếu**: WICG Local Network Access Section 7 (Permissions Policy Integration).
- **Nguồn xác thực**: [WICG Local Network Access Specification](https://wicg.github.io/local-network-access/), [MDN Local Network Access](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Local_network_access), [WICG Private Network Access](https://wicg.github.io/private-network-access/).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Giới thiệu định danh chỉ thị Permissions Policy chuyên biệt: `local-network`.
  - Ứng dụng mẹ cấp cao kiểm soát quyền truy cập mạng cục bộ của các iframe nhúng, sub-app, hoặc tiện ích mở rộng thông qua header HTTP phản hồi hoặc thuộc tính iframe: `Permissions-Policy: local-network=(self)`.
  - Mọi script bên thứ ba hoặc iframe nhúng nếu không được uỷ quyền tường minh qua `allow="local-network"` sẽ bị chặn ngay lập tức với lỗi `SecurityError` khi cố gắng phát yêu cầu đến IP nội bộ.
- **Yêu cầu Store Standard**:
  - Container siêu ứng dụng thiết lập cấu hình mặc định `Permissions-Policy: local-network=()` trên tất cả các webview chung, khoá hoàn toàn truy cập LAN.
  - Chỉ các mini-app nhà thông minh hoặc giải pháp doanh nghiệp được cấp chứng nhận đặc biệt mới được uỷ quyền `local-network=(self)`.
  - Ngăn chặn các SDK quảng cáo hoặc phân tích nhúng bên thứ ba lạm dụng quyền hạn của ứng dụng máy chủ để xâm nhập mạng nội bộ.

---

## 3. Ma Trận Điều Khiển & Phân Vùng Kiến Trúc (Architecture & Control Matrix)

| Trụ Cột Kỹ Thuật | Thành Phần / API Cốt Lõi | Cơ Chế Bảo Mật & Ranh Giới | Ràng Buộc Vận Hành Store Standard |
|---|---|---|---|
| **WICG Sub Apps API** | `navigator.subApps.add()`, `list()`, `remove()` | Origin Clamping; Capability Attenuation; cấm đệ quy lồng nhau | Đăng ký cửa sổ/taskbar OS độc lập; xoá cascade khi gỡ app mẹ; kiểm soát manifest đa tầng |
| **WICG Storage Buckets** | `navigator.storageBuckets.open(name, opts)` | Bucket Isolation; Phân tách IndexedDB/Cache/OPFS; Quota Budgeting | Phân định `critical-state` vs `ephemeral-media`; thu hồi bộ nhớ từng phần theo TTL và LRU |
| **Storage Durability** | `durability: 'strict' \| 'relaxed'` | Fsync đĩa tức thì vs Xả bộ nhớ trễ | Bắt buộc `strict` cho mini-app tài chính; giới hạn `relaxed` cho game/media bảo vệ chip flash |
| **WICG Local Network Access** | Taxonomy: `public`, `local`, `loopback` | Gradient chặn ranh giới; Chống Blind CSRF & SSRF nội bộ | Chặn mặc định các yêu cầu từ mini-app đám mây vào mạng LAN/Wi-Fi gia đình |
| **LNA CORS Preflight** | `Access-Control-Request-Local-Network` | Server đích phải trả lời `Access-Control-Allow-Local-Network: true` | Quét tĩnh mã nguồn bundle; bắt buộc cấu hình endpoint edge gateway/POS chuẩn hoá |
| **LNA Context & Policy** | Secure Contexts; `Permissions-Policy: local-network` | Ép buộc HTTPS; Chặn nội dung hỗn hợp; Khoá iframe bên thứ ba | Cấm HTTP cleartext; cấp quyền qua hộp thoại người dùng; phát hiện & chặn port scan |

---

## 4. Kế Hoạch Kiểm Thử & Tiêu Chí Nghiệm Thu Tự Động (Automated Conformance Testing)

Hệ thống CI/CD kiểm duyệt gói mini-app của Store tích hợp bộ test tự động đánh giá sự tuân thủ:
1. **Kiểm Tra Vòng Đời Sub-Apps**:
   - Khởi tạo sub-app ảo bằng `navigator.subApps.add()`, xác nhận định danh cửa sổ OS và biểu tượng taskbar được cấp phát chính xác.
   - Thử nghiệm gọi đệ quy `subApps.add()` từ bên trong sub-app -> Phải bị từ chối với ngoại lệ `SecurityError`.
   - Thực hiện gỡ bỏ siêu ứng dụng mẹ -> Xác nhận toàn bộ sub-app và dữ liệu đệm liên kết được thu dọn sạch sẽ.
2. **Kiểm Tra Phân Vùng Storage Buckets**:
   - Mở đồng thời hai bucket `critical-state` (`durability: 'strict'`) và `cache-media` (`durability: 'relaxed'`).
   - Ghi dữ liệu vào cả hai bucket và kích hoạt tình huống giả lập cạn kiệt dung lượng (storage pressure).
   - Xác nhận bucket `cache-media` bị thu hồi giải phóng bộ nhớ, trong khi bucket `critical-state` được bảo toàn nguyên vẹn.
   - Gọi `navigator.storageBuckets.delete('cache-media')` -> Xác nhận trả về `true` và giải phóng dung lượng đĩa tức thì.
3. **Kiểm Tra Ranh Giới Local Network Access**:
   - Chạy kịch bản fetch từ mini-app xuất phát từ domain công khai đến địa chỉ IP riêng (`192.168.1.1` và `127.0.0.1`).
   - Đảm bảo trình duyệt tự động gửi OPTIONS preflight với header `Access-Control-Request-Local-Network: true`.
   - Giả lập thiết bị không phản hồi header `Access-Control-Allow-Local-Network` -> Yêu cầu fetch phải thất bại với lỗi mạng mà không phát tán payload.
   - Thử nghiệm quét dải cổng IP cục bộ -> Hệ thống phát hiện hành vi bất thường, kích hoạt khoá tạm thời mini-app.

---

## 5. Lộ Trình Triển Khai Cho Doanh Nghiệp (Enterprise Adoption Roadmap)

1. **Giai Đoạn 1 (Tháng 1-2) — Chuẩn Hoá Bộ Nhớ Đa Phân Vùng (Storage Buckets)**:
   - Triển khai SDK cho phép mini-app khai báo các bucket lưu trữ chuyên biệt thay cho việc dùng chung monolithic storage.
   - Kích hoạt chính sách độ bền `strict` cho luồng xác thực và thanh toán; đưa `relaxed` vào luồng cache tĩnh.
2. **Giai Đoạn 2 (Tháng 3-4) — Tăng Cường Bảo Vệ Mạng Nội Bộ (Local Network Access)**:
   - Thiết lập cấu hình WebView container bật kiểm soát LNA và `Permissions-Policy: local-network=()`.
   - Phát hành bộ công cụ và tài liệu hướng dẫn cho các đối tác IoT và thiết bị POS tích hợp header phản hồi CORS LNA.
3. **Giai Đoạn 3 (Tháng 5-6) — Tích Hợp Hoàn Thiện Vòng Đời Sub Apps**:
   - Nâng cấp shell siêu ứng dụng trên Desktop và Mobile để ánh xạ các mini-app cài đặt thành các cửa sổ sub-app bản địa.
   - Hoàn thiện bảng điều khiển quản lý quyền riêng tư phân cấp, cho phép người dùng kiểm soát độc lập quyền hạn của từng sub-app.
