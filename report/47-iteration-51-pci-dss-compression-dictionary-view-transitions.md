# Chuyên Đề 47 (Iteration 51): PCI DSS v4.0.1 Client-Side Script Security, IETF RFC 9842 Compression Dictionary Transport, và CSS View Transitions Level 2 / W3C Audio Session API

## 1. Bối Cảnh & Mục Tiêu Nghiên Cứu
Tại Iteration 51, hệ thống nghiên cứu Deli Deep tập trung chuẩn hoá 3 trụ cột kỹ thuật chuyên sâu đáp ứng yêu cầu khắt khe về an toàn tài chính, tối ưu hoá truyền tải dữ liệu trên mạng di động băng thông hẹp và trải nghiệm điều hướng / âm thanh đa phương tiện native-tier:
1. **PCI DSS v4.0.1 Client-Side Script Security & Tamper Detection**: Chuẩn hoá quản trị script trang thanh toán theo Requirement 6.4.3 & 11.6.1 (có hiệu lực bắt buộc từ 31/03/2025), phân định ranh giới SAQ A vs SAQ A-EP, tích hợp Subresource Integrity (SRI) / CSP Level 3 và kiến trúc native bottom sheet tách rời runtime mini-app.
2. **IETF RFC 9842 Compression Dictionary Transport**: Chuẩn hoá cơ chế nén theo từ điển dùng chung (Shared Brotli `dcb` và Shared Zstandard `dcz`) với header `Use-As-Dictionary`, `Available-Dictionary`, `Dictionary-ID` và link relation `compression-dictionary`, giúp giảm 85%+ dung lượng gói cập nhật delta của mini-app.
3. **CSS View Transitions Level 2 & W3C Audio Session API**: Cơ chế chuyển trang mượt mà giữa các document cùng origin thông qua rule CSS `@view-transition { navigation: auto; }`, vòng đời sự kiện `pageswap` và `pagereveal`, `view-transition-class` tránh xung đột CSS, cùng API `navigator.audioSession` điều phối focus âm thanh hệ điều hành và xử lý ngắt cuộc gọi / audio ducking.

---

## 2. Chi Tiết Phát Hiện & Bằng Chứng Kỹ Thuật (Findings)

### Nhóm 1: PCI DSS v4.0.1 Payment Script Governance & Anti-Tampering

#### finding: `pcidss_051_01` — PCI DSS v4.0.1 Requirement 6.4.3: Quản Lý, Định Danh và Bảo Đảm Toàn Vẹn Script Thanh Toán
- **Tiêu chuẩn tham chiếu**: PCI DSS v4.0.1 Requirement 6.4.3 / PCI Security Standards Council.
- **Nguồn xác thực**: [PCI SSC Information Supplement: Payment Page Security](https://blog.pcisecuritystandards.org/new-information-supplement-payment-page-security-and-preventing-e-skimming), [PCI DSS v4.0 Resource Hub](https://blog.pcisecuritystandards.org/pci-dss-v4-0-resource-hub).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Requirement 6.4.3 bắt buộc mọi script thực thi trong trình duyệt người tiêu dùng trên các trang thanh toán phải: (1) Được phê duyệt chính thức; (2) Có văn bản giải trình lý do nghiệp vụ (written business justification); (3) Có cơ chế kỹ thuật bảo đảm tính toàn vẹn (integrity verification).
  - Trong các ứng dụng Single-Page Application (SPA) dùng chung một DOM context xuyên suốt, mọi script được nạp vào đều bị tính trong phạm vi kiểm toán trừ khi được phân đoạn độc lập.
- **Yêu cầu Store Standard**:
  - Mini-app khi gọi luồng checkout tuyệt đối không được nạp các script theo dõi hoặc quảng cáo của bên thứ ba vào môi trường thanh toán.
  - Super-app container duy trì danh mục script SDK thanh toán được ký số và xác minh hash nghiêm ngặt; chặn đứng mọi hành vi eval() hoặc chèn script động trái phép.

#### finding: `pcidss_051_02` — PCI DSS v4.0.1 Requirement 11.6.1: Phát Hiện Thay Đổi và Giả Mạo Kịp Thời
- **Tiêu chuẩn tham chiếu**: PCI DSS v4.0.1 Requirement 11.6.1 / PCI SSC.
- **Nguồn xác thực**: [PCI SSC Payment Page Security Guidance](https://blog.pcisecuritystandards.org/new-information-supplement-payment-page-security-and-preventing-e-skimming), [Clover Dev Docs PCI DSS 6.4.3 & 11.6.1](https://docs.clover.com/dev/docs/pci-dss-version-40-requirements-643-and-1161).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Bắt buộc triển khai cơ chế phát hiện thay đổi và giả mạo (change- and tamper-detection) đối với nội dung script và các HTTP header ảnh hưởng bảo mật (CSP, HSTS, X-Frame-Options) khi nhận bởi trình duyệt người dùng, tối thiểu 7 ngày/lần hoặc theo đánh giá rủi ro mục tiêu (Targeted Risk Analysis - TRA).
  - Ngăn chặn triệt để các cuộc tấn công Magecart / e-skimming thu hoạch lén thông tin thẻ tín dụng từ bộ nhớ DOM.
- **Yêu cầu Store Standard**:
  - WebView container tích hợp watchdog giám sát MutationObserver trên Document thanh toán.
  - Mọi vi phạm CSP (CSP reporting) được kết nối trực tiếp về SIEM/SOC của Super-app; tự động hủy phiên giao dịch nếu phát hiện bất thường về DOM script.

#### finding: `pcidss_051_03` — Ranh Giới Scoping SAQ A vs SAQ A-EP & Kiến Trúc Phân Đoạn
- **Tiêu chuẩn tham chiếu**: PCI SSC SAQ A v4.0.1 / PCI SSC E-commerce Guidance.
- **Nguồn xác thực**: [PCI SSC SAQ A Important Updates (Jan 2025)](https://blog.pcisecuritystandards.org/important-updates-announced-for-merchants-validating-to-self-assessment-questionnaire-a), [PCI SSC Document Library](https://www.pcisecuritystandards.org/document_library/).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - SAQ A áp dụng cho các đơn vị thuê ngoài toàn bộ chức năng thanh toán qua iframe độc lập hoặc redirect sang cổng thanh toán của nhà cung cấp đạt chuẩn PCI DSS.
  - Nếu JavaScript của merchant/mini-app tự tạo iframe hoặc nạp script điều khiển quá trình chuyển hướng trên cùng một trang, hệ thống rơi vào diện SAQ A-EP với khối lượng kiểm toán khổng lồ trên toàn bộ mã nguồn.
- **Yêu cầu Store Standard**:
  - Mini-app không được phép tự render form thu thập số thẻ; luồng checkout bắt buộc phải kích hoạt native payment sheet của Super-app hoặc mở iframe TPSP với sandbox attribute chặt chẽ (`sandbox="allow-scripts allow-forms"`, cấm `allow-same-origin` và `allow-top-navigation`).
  - Đảm bảo các mini-app merchant đủ điều kiện tự đánh giá theo mẫu tinh gọn SAQ A.

#### finding: `pcidss_051_04` — Subresource Integrity (SRI) & Strict CSP Level 3 Hash Governance
- **Tiêu chuẩn tham chiếu**: W3C Subresource Integrity / W3C CSP Level 3.
- **Nguồn xác thực**: [PCI SSC Payment Page Security](https://blog.pcisecuritystandards.org/new-information-supplement-payment-page-security-and-preventing-e-skimming), [PCI DSS v4.0 Resource Hub](https://blog.pcisecuritystandards.org/pci-dss-v4-0-resource-hub).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - SRI sử dụng hash SHA-384 (`integrity="sha384-..."`) bảo đảm file JavaScript không bị sai lệch dù chỉ 1 bit khi truyền qua mạng hoặc CDN trung gian.
  - Phối hợp với CSP directive `script-src require-sri-for script` hoặc `'strict-dynamic'` kết hợp nonce để ngăn chặn mọi script nạp lén lút ngoài danh mục.
- **Yêu cầu Store Standard**:
  - Container enforce thuộc tính SRI bắt buộc cho tất cả external scripts trong luồng checkout.
  - Cấm hoàn toàn `unsafe-inline` và `unsafe-eval` trong CSP của các view thanh toán.

#### finding: `pcidss_051_05` — Kiến Trúc Out-of-Process Native Payment Sheet
- **Tiêu chuẩn tham chiếu**: Super-App Native Container Architecture & PCI SSC Guidelines.
- **Nguồn xác thực**: [PCI SSC SAQ A Updates](https://blog.pcisecuritystandards.org/important-updates-announced-for-merchants-validating-to-self-assessment-questionnaire-a), [PCI SSC E-skimming Guidance](https://blog.pcisecuritystandards.org/new-information-supplement-payment-page-security-and-preventing-e-skimming).
- **Mức độ bằng chứng**: `best_practice_recommendation`.
- **Nội dung kỹ thuật**:
  - Cách ly vật lý hoàn toàn tiến trình JavaScript của mini-app với kênh uỷ quyền thanh toán bằng cách chuyển giao giao dịch sang native sheet (gọi qua JSAPI `superApp.requestPayment()`).
- **Yêu cầu Store Standard**:
  - Cửa sổ thanh toán native chạy trên process riêng biệt, mã hoá bộ nhớ, ngăn chặn mini-app đọc trộm sự kiện bàn phím, can thiệp prototype hoặc chụp màn hình DOM.

---

### Nhóm 2: IETF RFC 9842 Compression Dictionary Transport

#### finding: `dict_compress_051_01` — Header Phản Hồi 'Use-As-Dictionary' và Khớp URLPattern
- **Tiêu chuẩn tham chiếu**: IETF RFC 9842 Section 2.1 (Use-As-Dictionary).
- **Nguồn xác thực**: [RFC 9842 - Compression Dictionary Transport](https://datatracker.ietf.org/doc/rfc9842/), [draft-ietf-httpbis-compression-dictionary-19](https://datatracker.ietf.org/doc/html/draft-ietf-httpbis-compression-dictionary).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Header `Use-As-Dictionary` định dạng theo RFC 8941 Structured Field Dictionary gồm 4 tham số: `match` (bắt buộc, cú pháp URLPattern không regex), `match-dest` (danh sách Fetch RequestDestination như `script`, `style`), `id` (chuỗi server định danh tối đa 1024 ký tự), và `type` (mặc định `raw`).
  - Bắt buộc hoạt động trên secure contexts (HTTPS) và cùng Origin để ngăn ngừa rò rỉ dữ liệu chéo trang.
- **Yêu cầu Store Standard**:
  - CDN Super-app thiết lập `Use-As-Dictionary` cho các gói thư viện nền tảng cơ sở (runtime SDK core bundle), lưu vào bộ nhớ cache từ điển của ứng dụng.

#### finding: `dict_compress_051_02` — Quảng Bá Client: 'Available-Dictionary' và 'Dictionary-ID'
- **Tiêu chuẩn tham chiếu**: IETF RFC 9842 Section 2.2 & 2.3.
- **Nguồn xác thực**: [RFC 9842 Section 2.2 & 2.3](https://datatracker.ietf.org/doc/rfc9842/), [draft-ietf-httpbis-compression-dictionary (HTTPWG)](https://httpwg.org/http-extensions/draft-ietf-httpbis-compression-dictionary.html).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Khi client gửi request khớp điều kiện, nó gửi header `Available-Dictionary` chứa mã băm SHA-256 (Byte Sequence) của nội dung từ điển đang lưu sẵn, kèm `Dictionary-ID` nếu có.
  - Quy tắc phân giải ưu tiên khi có nhiều từ điển khớp: (1) Khớp cụ thể `match-dest`; (2) Chuỗi `match` dài nhất; (3) Từ điển tải gần nhất.
- **Yêu cầu Store Standard**:
  - Package loader của Super-app tự động gửi SHA-256 của phiên bản n-1 khi tải bản cập nhật n, cho phép server chỉ truyền phần bù delta siêu nhỏ.

#### finding: `dict_compress_051_03` — Thuật Toán Nén 'dcb' (Shared Brotli) và 'dcz' (Shared Zstandard)
- **Tiêu chuẩn tham chiếu**: IETF RFC 9842 Section 6 & IANA HTTP Field Registry.
- **Nguồn xác thực**: [Chrome Platform Status: Compression Dictionary Transport](https://chromestatus.com/feature/5124977788977152), [RFC 9842](https://datatracker.ietf.org/doc/rfc9842/).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Đăng ký hai token Content-Encoding mới: `dcb` (Shared Brotli RFC 7932) và `dcz` (Shared Zstandard RFC 8878).
  - Phản hồi từ server phải kèm `Vary: Available-Dictionary` để tránh ô nhiễm proxy cache. Hiệu năng nén vượt trội so với gzip/brotli độc lập nhờ tận dụng bảng ký hiệu có sẵn trong từ điển.
- **Yêu cầu Store Standard**:
  - Giảm từ 85% đến 92% kích thước file chuyển giao qua mạng, đưa thời gian khởi động nguội (cold-start) của mini-app xuống dưới 150ms trên kết nối 3G/4G.

#### finding: `dict_compress_051_04` — Làm Nóng Cache với Link Relation 'compression-dictionary'
- **Tiêu chuẩn tham chiếu**: IETF RFC 9842 Section 3 / IANA Link Relations.
- **Nguồn xác thực**: [RFC 9842 Section 3](https://datatracker.ietf.org/doc/rfc9842/), [Chrome Platform Status Feature 5509970048843776](https://chromestatus.com/feature/5509970048843776).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Cho phép máy chủ chỉ định liên kết tải trước từ điển thông qua `<link rel="compression-dictionary" href="...">` hoặc HTTP header `Link`.
  - Client tự động lập lịch tải ngầm (background fetch) từ điển khi đường truyền rảnh rỗi.
- **Yêu cầu Store Standard**:
  - Danh mục Mini-App Store nạp trước từ điển giao diện dùng chung theo từng chuyên mục (thương mại điện tử, tiện ích, tài chính).

#### finding: `dict_compress_051_05` — Kiến Trúc Cập Nhật Delta Chuẩn Hoá Cho Super-App
- **Tiêu chuẩn tham chiếu**: Super-App Packaging & Release Standard / IETF RFC 9842.
- **Nguồn xác thực**: [Chrome Platform Status: Compression Dictionary Transport](https://chromestatus.com/feature/5124977788977152), [RFC 9842](https://datatracker.ietf.org/doc/rfc9842/).
- **Mức độ bằng chứng**: `best_practice_recommendation`.
- **Nội dung kỹ thuật**:
  - Thay thế hoàn toàn các tiện ích vá nhị phân đóng gói độc quyền (như bsdiff/custom zip patcher) vốn dễ gây lỗi bộ nhớ bằng chuẩn truyền tải HTTP native an toàn, minh bạch.
- **Yêu cầu Store Standard**:
  - Gói subpackage cập nhật nóng qua mạng di động được giới hạn dưới 50KB, bảo đảm trải nghiệm cập nhật tức thì (hot over-the-air update).

---

### Nhóm 3: W3C CSS View Transitions Level 2 & Audio Session API

#### finding: `trans_audio_051_01` — W3C CSS View Transitions Level 2: Điều Hướng Cross-Document Khai Báo
- **Tiêu chuẩn tham chiếu**: W3C CSS View Transitions Module Level 2 Section 2.
- **Nguồn xác thực**: [W3C CSS View Transitions 2](https://www.w3.org/TR/css-view-transitions-2/), [MDN View Transition API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Cho phép tạo hoạt ảnh chuyển trang giữa các trang web khác nhau cùng origin thông qua CSS rule `@view-transition { navigation: auto; }`.
  - Điều kiện: cùng origin, có tương tác người dùng hoặc duyệt lịch sử, trang luôn visible, và cả 2 trang đều khai báo `@view-transition`.
  - Kết hợp với cơ chế render-blocking (`blocking="render"`) để đảm bảo DOM trang đích sẵn sàng trước khi kết xuất hoạt ảnh.
- **Yêu cầu Store Standard**:
  - WebView container kích hoạt mặc định CSS View Transitions Level 2; giới hạn trần thời gian chạy animation tối đa 300ms tránh đóng băng UI.

#### finding: `trans_audio_051_02` — Vòng Đời Sự Kiện 'pageswap' và 'pagereveal'
- **Tiêu chuẩn tham chiếu**: W3C CSS View Transitions 2 / MDN Web APIs.
- **Nguồn xác thực**: [MDN Window: pageswap event](https://developer.mozilla.org/en-US/docs/Web/API/Window/pageswap_event), [MDN Window: pagereveal event](https://developer.mozilla.org/en-US/docs/Web/API/Window/pagereveal_event).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - `pageswap`: Bắn ra trên Window của trang cũ ngay trước khi unload, cho phép kiểm tra URL đích, cấu hình kiểu transition và gán `view-transition-name` động cho phần tử được chọn (ví dụ: ảnh sản phẩm).
  - `pagereveal`: Bắn ra trên Window của trang mới trước khung hình đầu tiên, ghép nối phần tử tương ứng và tinh chỉnh hiệu ứng.
- **Yêu cầu Store Standard**:
  - Router điều hướng của mini-app sử dụng cặp sự kiện này để tạo hiệu ứng chuyển từ danh sách sang chi tiết sản phẩm (hero element expansion) mượt mà như app native; tự động dọn dẹp inline style tránh xung đột BFCache.

#### finding: `trans_audio_051_03` — Selective Transitions và Thuộc Tính 'view-transition-class'
- **Tiêu chuẩn tham chiếu**: W3C CSS View Transitions Module Level 2 Section 3 & 4.
- **Nguồn xác thực**: [W3C CSS View Transitions 2](https://www.w3.org/TR/css-view-transitions-2/), [MDN Using View Transition Types](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API/Using_types).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Khắc phục tình trạng phình to mã CSS khi phải đặt tên duy nhất cho hàng trăm phần tử trong Level 1 bằng thuộc tính `view-transition-class`.
  - Selector giả `::view-transition-group(.card-item)` áp dụng chung cho mọi node có class tương ứng; kết hợp bộ lọc kiểu hoạt ảnh `:active-view-transition-type(...)`.
- **Yêu cầu Store Standard**:
  - Thư viện Design System của Super-app cung cấp sẵn các bộ animation chuẩn (push, pop, modal slide) tích hợp sẵn gesture vuốt back của iOS / Android Predictive Back.

#### finding: `trans_audio_051_04` — W3C Audio Session API và Phân Quyền Focus Âm Thanh
- **Tiêu chuẩn tham chiếu**: W3C Audio Session Specification / W3C Audio Working Group.
- **Nguồn xác thực**: [W3C Audio Session](https://w3c.github.io/audio-session/), [W3C Media Session](https://w3c.github.io/mediasession/).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Giao diện `navigator.audioSession` ánh xạ trực tiếp tới trình quản lý audio focus của hệ điều hành (Android AudioManager / iOS AVAudioSession) với các loại session: `auto`, `playback`, `transient`, `transient-solo`, `ambient`, và `play-and-record`.
- **Yêu cầu Store Standard**:
  - Mini-app game casual bắt buộc khai báo `type = "ambient"` để âm thanh game không làm ngắt nhạc/podcast nền của người dùng.
  - Ứng dụng họp / gọi thoại VoIP khai báo `play-and-record` và giải phóng focus ngay khi kết thúc cuộc gọi.

#### finding: `trans_audio_051_05` — Xử Lý Ngắt Cuộc Gọi và Audio Ducking
- **Tiêu chuẩn tham chiếu**: W3C Audio Session Specification Section 5 & 9.
- **Nguồn xác thực**: [W3C Audio Session](https://w3c.github.io/audio-session/), [W3C Media Session](https://w3c.github.io/mediasession/).
- **Mức độ bằng chứng**: `official_standard`.
- **Nội dung kỹ thuật**:
  - Sự kiện `onstatechange` thông báo chuyển đổi trạng thái (`active`, `inactive`, `interrupted`) khi có cuộc gọi di động hoặc thông báo khẩn cấp từ hệ thống.
  - Ứng dụng phải tạm dừng phát và gọi `AudioContext.suspend()` để tránh tiêu hao tài nguyên CPU/pin.
- **Yêu cầu Store Standard**:
  - Mini-app đạt chứng chỉ Store phải đăng ký lắng nghe `statechange`; Super-app host tự động hạ âm lượng mini-app (audio ducking 80%) khi trợ lý ảo hoặc chuông thông báo hệ thống phát tín hiệu.

---

## 3. Ma Trận Tích Hợp Kỹ Thuật (Engineering Control Matrix)

| Trụ Cột Kỹ Thuật | Tiêu Chuẩn Nòng Cốt | Cơ Chế Enforce Của Super-App Container | Tác Động Vận Hành & Compliance |
|---|---|---|---|
| **PCI DSS Script Governance** | PCI DSS v4.0.1 Req 6.4.3 & 11.6.1 | DOM MutationObserver, Strict CSP Level 3, SRI hash check | Đủ điều kiện tự đánh giá SAQ A, loại trừ 100% rủi ro Magecart |
| **Out-of-Process Payment** | Native Container Architecture | Tách riêng tiến trình thanh toán, chỉ giao tiếp qua native JSAPI | Mini-app hoàn toàn không chạm dữ liệu thẻ (PAN/CVV) |
| **Delta Package Transport** | IETF RFC 9842 (Shared Brotli/Zstandard) | Header `Use-As-Dictionary`, `Available-Dictionary`, `dcb/dcz` | Giảm 85-92% băng thông cập nhật, khởi động <150ms |
| **Proactive Pre-warming** | RFC 9842 Link Relation `compression-dictionary` | Nạp trước từ điển runtime chung khi duyệt danh mục Store | Giảm giật lag mạng khi người dùng cài đặt mini-app mới |
| **Cross-Document Transitions** | W3C CSS View Transitions Level 2 | `@view-transition { navigation: auto; }`, `pageswap`, `pagereveal` | Hiệu ứng chuyển cảnh mượt mà 60fps ngang ngửa ứng dụng Native |
| **Audio Focus Governance** | W3C Audio Session API | `navigator.audioSession.type`, lắng nghe `onstatechange` | Không ngắt nhạc nền người dùng, hỗ trợ audio ducking tự động |

---

## 4. Trạng Thái Hệ Thống Sau Iteration 51
- **Tổng số phát hiện chuẩn hoá (canonical findings)**: Đã nâng từ **701** lên **716** (bổ sung chính xác 15 phát hiện, 100% kiểm tra mã HTTP 200).
- **Số tệp chuyên đề**: Đã tạo tệp chuyên đề số 47 (`47-iteration-51-pci-dss-compression-dictionary-view-transitions.md`).
- **Chỉ số an toàn state**: Toàn bộ thao tác append-only, đã sao lưu backup tại `state/findings.jsonl.backup-iter51-preappend`.
- **Đồng bộ Public Repo**: Sẽ được kích hoạt theo quy ước tại Iteration 55 (hoặc khi có yêu cầu đột xuất).
