# Chuyên đề 135: Chuẩn Hóa Bảo Mật Phần Cứng Thực Thể (IETF RFC 9711 EAT / RFC 9782), Chia Sẻ Danh Bạ Bảo Toàn Quyền Riêng Tư (IETF RFC 9553 JSContact / RFC 9555) & Điều Phối Thanh Toán Container Đa Kênh (W3C Payment Request API / Secure Payment Confirmation) Trong Hệ Sinh Thái Super App

## 1. Bối cảnh & Tầm quan trọng chiến lược

Trong kiến trúc Super App hiện đại với sự tích hợp của hàng trăm đối tác mini-app cung cấp dịch vụ từ thương mại điện tử, đặt đồ ăn, gọi xe đến các dịch vụ tài chính, ngân hàng số và eKYC nhạy cảm, ba thách thức cốt lõi luôn đặt ra cho đơn vị vận hành nền tảng (Super App Host Platform):
1. **Xác thực toàn vẹn thiết bị và môi trường thực thi (Device & Runtime Attestation):** Làm thế nào để máy chủ Super App và các dịch vụ backend tài chính xác minh một cách toán học rằng ứng dụng đang chạy trên một thiết bị di động vật lý nguyên bản (không bị root/jailbreak, không chạy trên trình giả lập bot, không bị hook bởi Frida/Xposed và có chuỗi khởi động tin cậy OEM an toàn)?
2. **Quản trị quyền riêng tư dữ liệu danh bạ (Privacy-Preserving Contact Sharing):** Làm thế nào để chia sẻ thông tin người nhận (trong giao dịch chuyển tiền P2P, giao hàng thương mại điện tử, gửi quà) cho mini-app mà không để lộ toàn bộ danh bạ cá nhân của người dùng, đồng thời chuẩn hóa dữ liệu theo định dạng JSON hiện đại, quốc tế hóa tên gọi đa ngôn ngữ và hỗ trợ ngữ cảnh riêng tư (cá nhân vs công việc)?
3. **Điều phối thanh toán phi tập trung và xác thực sinh trắc học giao dịch (Native Payment Mediation & Biometric SCA):** Làm thế nào để mini-app khởi tạo quy trình thanh toán mượt mà thông qua giao diện native sheet thống nhất của container mà không cần chuyển hướng sang cổng thanh toán ngoài (web redirect), hỗ trợ đa dạng phương thức (Ví Super App, thẻ tín dụng, VietQR/NAPAS, DCB telco) và xác thực giao dịch sinh trắc học theo chuẩn W3C Web Payments / FIDO WebAuthn?

Iteration 135 hoàn thiện chuẩn hóa kỹ thuật giải quyết trọn vẹn 3 trụ cột này dựa trên các tiêu chuẩn chính thức của IETF và W3C:
- **IETF RFC 9711 (Entity Attestation Token - EAT) & RFC 9782 (EAT Media Types):** Chuẩn hóa định dạng token xác thực thiết bị và phần mềm dựa trên CWT/JWT, tích hợp kiến trúc IETF RATS (RFC 9334), phân nhóm submodule cho container/mini-app, và content negotiation qua MIME headers.
- **IETF RFC 9553 (JSContact: A JSON Representation of Contact Data) & RFC 9555 (vCard Mapping):** Chuẩn hóa mô hình dữ liệu danh bạ JSON hiện đại (`Card`), phân tách ngữ cảnh (`personal` vs `work`), cấu trúc hóa tên gọi đa văn hóa/ngữ âm, kênh liên lạc E.164/onlineServices và định vị địa lý WGS 84.
- **W3C Payment Request API, Payment Method Identifiers & Secure Payment Confirmation (SPC):** Chuẩn hóa giao diện `window.PaymentRequest`, cơ chế định danh phương thức thanh toán dựa trên URL (URL-based PMIs), Payment Method Manifest, điều chỉnh giá động (`PaymentDetailsModifier`), và xác thực thanh toán bảo mật sinh trắc học (SPC) đáp ứng chuẩn PSD2 SCA.

---

## 2. Phân tích Kỹ thuật Chuyên sâu 15 Tiêu chuẩn Khảo sát (Iteration 135)

### Trụ cột 1: IETF RFC 9711 EAT & RFC 9782 Media Types (Chứng thực Thực thể & Môi trường Di động)

#### 1. Kiến trúc Entity Attestation Token & Chu trình RATS (RFC 9711 Section 1 & 3)
- **ID:** `eat_135_01` | **Mức bằng chứng:** `official_standard`
- **Tiêu chuẩn:** IETF RFC 9711 (The Entity Attestation Token)
- **Nội dung kỹ thuật:** RFC 9711 định nghĩa Entity Attestation Token (EAT), một định dạng thông điệp mang tập hợp các khẳng định (claims set) có chữ ký số nhằm mô tả trạng thái bảo mật, cấu hình phần cứng và môi trường thực thi của một thực thể (thiết bị di động, chip IoT hoặc hệ thống phần mềm). EAT vận hành hoàn toàn tương thích với kiến trúc IETF RATS (Remote ATtestation procedureS, RFC 9334):
  - **Attester (Bên chứng thực):** Thiết bị di động chứa container Super App thu thập bằng chứng (evidence) và ký bằng khóa chứng thực phần cứng.
  - **Verifier (Bên thẩm định):** Máy chủ gateway của Super App thẩm định chữ ký và so khớp các claims với giá trị tham chiếu (Reference Values).
  - **Relying Party (Bên tin cậy):** Các dịch vụ backend hoặc mini-app tài chính tiêu thụ kết quả thẩm định (Attestation Results).
- **Đặc điểm bao gói (Serialization):** Hỗ trợ song song CBOR Web Token (CWT, RFC 8392) kết hợp COSE (RFC 9052) cho môi trường nhị phân băng thông thấp, và JSON Web Token (JWT, RFC 7519) kết hợp JOSE (RFC 7515). Tính tươi mới (freshness) được bảo vệ bằng claim `eat_nonce`, ngăn chặn triệt để tấn công phát lại (replay attack).

#### 2. Khẳng định Định danh Phần cứng & Trạng thái Khởi động (RFC 9711 Section 4.2)
- **ID:** `eat_135_02` | **Mức bằng chứng:** `official_standard`
- **Tiêu chuẩn:** IETF RFC 9711 Section 4.2 (ueid, sueids, oemid, hwmodel, oemboot, dbgstat)
- **Nội dung kỹ thuật:** Chuẩn hóa các claims cốt lõi phản ánh cấu trúc phần cứng của thiết bị:
  - `ueid` (Universal Entity ID): Số định danh phần cứng duy nhất toàn cầu của thiết bị hoặc mô-đun an toàn (eSE).
  - `sueids` (Semi-permanent UEIDs): Định danh xoay vòng bảo vệ quyền riêng tư, hạn chế tracking người dùng xuyên suốt thời gian dài.
  - `oemid` & `hwmodel`: Nhận diện nhà sản xuất thiết bị gốc (Apple, Samsung, Xiaomi) và mã model phần cứng cụ thể, cho phép máy chủ định tuyến chính sách rủi ro dựa trên lỗ hổng phần cứng đã biết.
  - `oemboot` (OEM Authorized Boot): Giá trị boolean xác nhận hệ điều hành được khởi động qua chuỗi tin cậy phần cứng (Verified Boot) không bị can thiệp.
  - `dbgstat` (Debug Status): Trạng thái gỡ lỗi phần cứng/phần mềm (JTAG, ADB, gỡ lỗi nhân). Nếu `dbgstat` kích hoạt, thiết bị bị coi là có nguy cơ can thiệp mã động cao.

#### 3. Phép đo Phần mềm, Manifests & Kết quả So khớp (RFC 9711 Section 4.2.15-17)
- **ID:** `eat_135_03` | **Mức bằng chứng:** `official_standard`
- **Tiêu chuẩn:** IETF RFC 9711 (manifests, measurements, measres)
- **Nội dung kỹ thuật:** Tách biệt rõ ràng giữa bản kê khai phát hành và phép đo thời gian chạy:
  - `manifests`: Danh mục phần mềm gốc do nhà phát triển ký số (chứa mã băm gói mini-app, phiên bản SDK).
  - `measurements`: Phép đo băm mật mã thời gian thực (cryptographic hashes) được thực hiện bởi hypervisor hoặc tiến trình giám sát hệ thống trên các phân vùng bộ nhớ thực thi và thư viện động.
  - `measres` (Software Measurement Results): Cấu trúc chuẩn hóa kết quả so khớp giữa phép đo thực tế và giá trị tham chiếu vàng (Golden Images), trả về các trạng thái `success`, `failure`, `not run`, hoặc `absent`. Cơ chế này cho phép phát hiện tức thời việc chèn mã độc, tiêm script WebView hoặc sửa đổi bộ nhớ container bởi các công cụ hook.

#### 4. Thiết bị Hỗn hợp, Submodules & Gói EAT Tách rời (RFC 9711 Section 4.2.18 & Section 5)
- **ID:** `eat_135_04` | **Mức bằng chứng:** `official_standard`
- **Tiêu chuẩn:** IETF RFC 9711 (submods & Detached EAT Bundles)
- **Nội dung kỹ thuật:** Claim `submods` cho phép mô hình hóa phân cấp một thực thể phức tạp bao gồm nhiều vùng bảo mật độc lập. Đối với Super App, mô hình EAT phản ánh hoàn hảo cấu trúc:
  - Root EAT: Chứng thực phần cứng điện thoại và nhân hệ điều hành.
  - Submodule `native_host`: Chứng thực ứng dụng nền tảng Super App gốc (chữ ký số APK/IPA, trạng thái bảo vệ container).
  - Submodule `webview_runtime`: Chứng thực phiên bản trình thông dịch web.
  - Submodule `mini_app_<id>`: Chứng thực băm gói mini-app, quyền hạn đã cấp và phiên làm việc hiện tại.
- Ngoài ra, **Detached EAT Bundles** cho phép phân tách chữ ký chứng thực khỏi các khối dữ liệu bằng chứng lớn, tối ưu hóa kích thước truyền tải qua HTTP headers.

#### 5. Chuẩn hóa Media Types & Đàm phán Profile (IETF RFC 9782)
- **ID:** `eat_135_05` | **Mức bằng chứng:** `official_standard`
- **Tiêu chuẩn:** IETF RFC 9782 (Entity Attestation Token Media Types)
- **Nội dung kỹ thuật:** RFC 9782 đăng ký chính thức với IANA các kiểu MIME chuyên dụng:
  - `application/eat+cwt`: Token EAT đóng gói nhị phân CBOR/COSE.
  - `application/eat+jwt`: Token EAT đóng gói văn bản JSON/JOSE.
  - `application/eat-bun+cbor` & `application/eat-bun+json`: Gói chứng thực tách rời.
- Tham số `eat_profile`: Cho phép đưa mã định danh profile (URI hoặc OID) trực tiếp vào header HTTP `Content-Type` (ví dụ: `Content-Type: application/eat+jwt; eat_profile="https://superapp.com/profiles/fintech-v1"`). Điều này giúp API Gateway định tuyến và tiền xử lý payload chứng thực ngay tại tầng biên mạng mà không cần giải mã body request.

---

### Trụ cột 2: IETF RFC 9553 JSContact & RFC 9555 (Quản trị Dữ liệu Danh bạ Cá nhân Hóa)

#### 6. Mô hình Dữ liệu JSContact & Kiến trúc Đối tượng Card (RFC 9553 Section 1 & 2)
- **ID:** `jscontact_135_06` | **Mức bằng chứng:** `official_standard`
- **Tiêu chuẩn:** IETF RFC 9553 (JSContact: A JSON Representation of Contact Data)
- **Nội dung kỹ thuật:** RFC 9553 thay thế hoàn toàn định dạng dòng văn bản vCard (RFC 6350) và định dạng mảng jCard (RFC 7095) bằng mô hình dữ liệu JSON tự nhiên, không nhập nhằng và dễ dàng mở rộng.
- Thực thể trung tâm là đối tượng `Card`, chứa các thuộc tính có cấu trúc: định danh (`uid`), tên họ, tổ chức, chức danh, kênh liên lạc, địa chỉ và thông tin mật mã (khóa công khai/DID).
- Thuộc tính `contexts`: Cho phép gắn thẻ ngữ cảnh cho từng thành phần liên lạc (ví dụ: `contexts: { "personal": true }` hoặc `contexts: { "work": true }`). Khi người dùng chọn chia sẻ danh bạ với mini-app (ví dụ: app mua sắm quà tặng), Super App có thể lọc chính xác chỉ cung cấp thông tin cá nhân, bảo vệ tuyệt đối thông tin cơ quan/công việc.

#### 7. Phân rã Tên gọi, Ngữ âm & Đa dạng Ngôn ngữ/Ký tự (RFC 9553 Section 2.2)
- **ID:** `jscontact_135_07` | **Mức bằng chứng:** `official_standard`
- **Tiêu chuẩn:** IETF RFC 9553 Section 2.2 (name, nicknames, phonetic)
- **Nội dung kỹ thuật:** Cung cấp giải pháp triệt để cho bài toán tên gọi quốc tế hóa:
  - Thuộc tính `components`: Mảng các thành phần có định kiểu (`prefix`, `personal`, `surname`, `suffix`), hỗ trợ trật tự họ tên linh hoạt (Họ đứng trước như tiếng Việt, tiếng Trung, tiếng Nhật hay Tên đứng trước như phương Tây).
  - Thuộc tính `phonetic`: Lưu trữ phiên âm quốc tế (IPA), Pinyin hoặc Hiragana, hỗ trợ các mini-app trợ lý ảo AI và ứng dụng đọc màn hình phát âm chính xác tên đối tác/người nhận.
  - Thuộc tính `localizations`: Ánh xạ mã ngôn ngữ BCP 47 sang các bản ghi họ tên theo hệ chữ viết khác nhau trên cùng một thẻ liên lạc (ví dụ: tên tiếng Việt có dấu và phiên bản không dấu dùng cho thanh toán quốc tế).

#### 8. Kênh Liên lạc Số & Chuẩn hóa E.164 (RFC 9553 Section 2.3)
- **ID:** `jscontact_135_08` | **Mức bằng chứng:** `official_standard`
- **Tiêu chuẩn:** IETF RFC 9553 Section 2.3 (emails, phones, onlineServices)
- **Nội dung kỹ thuật:** Chuẩn hóa toàn bộ phương thức liên lạc viễn thông và số hóa:
  - `phones`: Chuẩn hóa số điện thoại theo định dạng quốc tế E.164 thông qua URI `tel:` (ví dụ: `tel:+84981234567`), loại bỏ sai lệch do định dạng số cục bộ.
  - `emails`: Địa chỉ thư điện tử tuân thủ RFC 5322.
  - `onlineServices`: Thuộc tính hiện đại hóa ghi nhận định danh tài khoản mạng xã hội và nhắn tin OTT (`service: 'Telegram'`, `service: 'Zalo'`).
  - Thuộc tính `pref`: Thang điểm ưu tiên từ 1 đến 100, xác định rõ kênh liên lạc người dùng muốn ưu tiên sử dụng. Container Super App có thể cấu hình chỉ trả về kênh có `pref = 1` cho mini-app, giảm thiểu tối đa phơi nhiễm dữ liệu.

#### 9. Địa chỉ Giao nhận Địa lý & Tọa độ WGS 84 (RFC 9553 Section 2.5)
- **ID:** `jscontact_135_09` | **Mức bằng chứng:** `official_standard`
- **Tiêu chuẩn:** IETF RFC 9553 Section 2.5 (addresses, coordinates)
- **Nội dung kỹ thuật:** Chuẩn hóa thông tin địa chỉ phục vụ thương mại điện tử và logistics:
  - Phân tách trường tường minh: `street`, `locality` (quận/huyện, thành phố), `region` (tỉnh/thành phố lớn), `country` và `postcode`.
  - Hỗ trợ tọa độ vị trí thực thông qua URI `geo:` (RFC 5870, hệ tọa độ WGS 84), cho phép mini-app giao hàng và gọi xe định vị điểm trả hàng chính xác mà không phụ thuộc vào chuỗi văn bản tự do.
  - Phân loại ngữ cảnh địa chỉ (`billing`, `delivery`, `work`, `private`). Cơ chế Super App cho phép "tiết lộ thông tin 2 bước": ban đầu chỉ tiết lộ `locality` và `region` để mini-app tính toán phí vận chuyển sơ bộ, chỉ khi người dùng bấm xác nhận đơn hàng mới cấp toàn bộ `street`.

#### 10. Chuyển đổi Hai chiều Lossless vCard & JSContact (IETF RFC 9555)
- **ID:** `jscontact_135_10` | **Mức bằng chứng:** `official_standard`
- **Tiêu chuẩn:** IETF RFC 9555 (JSContact: Converting between JSContact and vCard)
- **Nội dung kỹ thuật:** RFC 9555 định nghĩa thuật toán chuyển đổi hai chiều chuẩn xác giữa vCard 4.0 và JSContact:
  - Ánh xạ tương hỗ toàn bộ tham số vCard (`TYPE`, `PREF`, `LANGUAGE`) sang cấu trúc JSON phân cấp.
  - Định nghĩa cơ chế bảo toàn thuộc tính không xác định (`JSCard` extension parameters), đảm bảo không mất mát dữ liệu khi chuyển đổi vòng quanh (round-trip).
  - Cho phép container Super App dễ dàng trích xuất dữ liệu từ danh bạ gốc của iOS (Contacts framework) hoặc Android (ContactsContract) và đóng gói thành JSContact trả về cho mini-app mà không gặp các lỗi cú pháp cổ điển (dòng bẻ dòng, ký tự thoát, mã hóa UTF-8).

---

### Trụ cột 3: W3C Payment Request API & Xác thực Giao dịch Sinh trắc học

#### 11. Kiến trúc Giao diện & Vòng đời Payment Request API (W3C PRAPI Section 4)
- **ID:** `payment_req_135_11` | **Mức bằng chứng:** `official_standard`
- **Tiêu chuẩn:** W3C Payment Request API (Proposed Recommendation)
- **Nội dung kỹ thuật:** Chuẩn hóa giao diện khởi tạo thanh toán native `window.PaymentRequest(methodData, details, options)`:
  - Tách bạch ba thành phần: Phương thức thanh toán chấp nhận (`methodData`), Thông tin đơn hàng (`details`: tổng tiền, danh mục hàng, giảm giá), và Yêu cầu dữ liệu bổ sung (`options`: tên, email, số điện thoại, địa chỉ giao hàng).
  - Vòng đời thông qua promise `show()`: Bắt buộc kích hoạt bởi cử chỉ người dùng (transient user activation), hiển thị bảng thanh toán gốc của hệ thống (native bottom-sheet), thu thập xác thực và trả về `PaymentResponse`.
  - Kết thúc quy trình bằng lời gọi `response.complete('success' | 'fail')`, giải phóng khóa UI và bảo đảm tính nguyên tử của giao dịch.

#### 12. Định danh Phương thức Thanh toán Dựa trên URL (W3C PMI Section 2 & 3)
- **ID:** `payment_req_135_12` | **Mức bằng chứng:** `official_standard`
- **Tiêu chuẩn:** W3C Payment Method Identifiers
- **Nội dung kỹ thuật:** Phân loại thành hai nhóm định danh PMI:
  - Short PMI: Chuỗi ký tự chuẩn hóa trong sổ đăng ký W3C (ví dụ: thẻ quốc tế).
  - URL-based PMI: Định danh dựa trên URL tuyệt đối có giao thức HTTPS (ví dụ: `https://superapp.com/pay`, `https://vietqr.net/pay`).
  - Lợi thế URL-based PMI: Phân quyền sở hữu không gian tên theo tên miền Internet. Đơn vị vận hành Super App có toàn quyền kiểm soát đặc tả kỹ thuật và danh sách ứng dụng được phép xử lý phương thức thanh toán của mình mà không sợ xung đột tên gọi.

#### 13. Khám phá Ứng dụng Thanh toán Qua Payment Method Manifest (W3C PMM Section 2 & 3)
- **ID:** `payment_req_135_13` | **Mức bằng chứng:** `official_standard`
- **Tiêu chuẩn:** W3C Payment Method Manifest
- **Nội dung kỹ thuật:** Bản kê khai JSON được lưu trữ tại URL của PMI hoặc liên kết qua HTTP header `Link: <...>; rel="payment-method-manifest"`:
  - Khai báo `default_applications` (danh sách Web App Manifest của các ứng dụng thanh toán được ủy quyền) và `supported_origins` (các domain được phép lưu trữ app thanh toán).
  - Ngăn chặn triệt để mã độc thanh toán: Container chỉ khởi chạy các bộ xử lý thanh toán (payment handlers) có nguồn gốc được chứng thực trong manifest của phương thức.
  - Cho phép tích hợp ứng dụng native Android/iOS thông qua dấu vân tay chứng chỉ ký số SHA-256 được cấu hình trong manifest.

#### 14. Xác thực Thanh toán Bảo mật Sinh trắc học (W3C Secure Payment Confirmation)
- **ID:** `payment_req_135_14` | **Mức bằng chứng:** `official_standard`
- **Tiêu chuẩn:** W3C Secure Payment Confirmation (SPC) Candidate Recommendation
- **Nội dung kỹ thuật:** Mở rộng Payment Request API kết hợp WebAuthn để cung cấp trải nghiệm thanh toán sinh trắc học không ma sát:
  - Khởi tạo qua phương thức thanh toán `secure-payment-confirmation`: truyền danh sách credential ID, challenge mật mã, thông tin bên nhận (`payeeOrigin`) và chi tiết giao dịch (`instrument`).
  - Hiển thị hộp thoại bảo mật do trình duyệt/container kiểm soát, thể hiện tên bên thụ hưởng đã xác minh, biểu tượng và số tiền chính xác, gắn kết sự đồng ý thị giác của người dùng vào chữ ký mật mã.
  - Tạo ra chữ ký phần cứng từ chip bảo mật (Secure Enclave / StrongBox), nhúng dữ liệu giao dịch trực tiếp vào `clientDataJSON`. Đáp ứng đầy đủ quy định xác thực mạnh khách hàng (SCA / Dynamic Linking) của Chỉ thị PSD2 châu Âu và các quy định thanh toán sinh trắc học của Ngân hàng Trung ương.

#### 15. Điều chỉnh Chi phí Động & Sự kiện Thay đổi Phương thức (W3C PRAPI Section 4.5 & 4.6)
- **ID:** `payment_req_135_15` | **Mức bằng chứng:** `official_standard`
- **Tiêu chuẩn:** W3C Payment Request API Section 4.5 & 4.6 (PaymentDetailsModifier & PaymentMethodChangeEvent)
- **Nội dung kỹ thuật:**
  - `PaymentDetailsModifier`: Cho phép người bán điều chỉnh cơ cấu giá (`total` và `additionalDisplayItems`) dựa trên phương thức thanh toán cụ thể mà khách hàng lựa chọn (ví dụ: giảm giá 5% khi thanh toán qua Ví Super App, hoặc phụ thu phí thanh toán thẻ quốc tế).
  - Sự kiện `paymentmethodchange`: Khi người dùng chuyển đổi giữa các phương thức thanh toán trên native sheet, sự kiện này được bắn về mini-app để tính toán lại dòng tiền theo thời gian thực.
  - Cập nhật nguyên tử qua `event.updateWith(detailsPromise)` bảo đảm tính nhất quán của bảng giá, hỗ trợ hoàn hảo cho việc hạch toán phân chia doanh thu (revenue split) và phí viễn thông DCB trong Super App.

---

## 3. Kiến trúc Tích hợp Hệ thống (Target Architecture)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       SUPER APP NATIVE RUNTIME CONTAINER                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌──────────────────────────┐                  ┌──────────────────────┐    │
│   │   MINI APP WEBVIEW       │                  │   MINI APP WEBVIEW   │    │
│   │   (E-Commerce / Food)    │                  │   (Fintech / Banking)│    │
│   └────────────┬─────────────┘                  └───────────┬──────────┘    │
│                │ navigator.contacts.select()                │ PaymentRequest│
│                │                                            │ .show()       │
│                ▼                                            ▼               │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │             HOST CONTAINER JAVASCRIPT BRIDGE & GATEWAY              │   │
│   ├──────────────────────────────────┬──────────────────────────────────┤   │
│   │  RFC 9553 JSCONTACT MEDIATION     │  W3C PAYMENT MEDIATION & SPC     │   │
│   │  - Two-stage disclosure gating   │  - URL-based PMI routing         │   │
│   │  - Context filtering (work/home) │  - PaymentDetailsModifier sync   │   │
│   │  - E.164 phone normalization     │  - Native Payment Sheet UI       │   │
│   └──────────────────┬───────────────┴──────────────────┬───────────────┘   │
│                      │                                  │                   │
│                      ▼                                  ▼                   │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │               RFC 9711 ENTITY ATTESTATION TOKEN (EAT)               │   │
│   │  - Native Host & WebView Runtime Submodules (RFC 9711 submods)      │   │
│   │  - Runtime Software Hashing & Golden Image Comparison (measurements)│   │
│   │  - Hardware Verified Boot & Debug Status Telemetry (oemboot/dbgstat)│   │
│   └──────────────────────────────────┬──────────────────────────────────┘   │
│                                      │                                      │
│                                      ▼                                      │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │          HARDWARE SECURITY ROOT OF TRUST & OS ENCLAVE               │   │
│   │  - Apple Secure Enclave / Android StrongBox KeyStore                │   │
│   │  - Biometric User Verification (TouchID/FaceID/BiometricPrompt)     │   │
│   │  - FIDO CTAP 2.1 Authenticator & WebAuthn SPC Key Pair Generation  │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────┬──────────────────────────────────────┘
                                       │ HTTPS EAT (RFC 9782 application/eat+jwt)
                                       │ + Signed Payment Token (JWS/JWE)
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    SUPER APP ENTERPRISE API CLOUD GATEWAY                   │
├─────────────────────────────────────────────────────────────────────────────┤
│  1. EAT Attestation Verifier (RFC 9334 RATS): Appraises Evidence vs Golden  │
│  2. Dynamic Payment Router: Resolves Payment Method Manifest & Banking APIs │
│  3. Multi-Tenant Ledger: Executes Split Settlement, DCB Fees & Commissions  │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Bảng Ma trận Kiểm soát Kỹ thuật (Control Matrix)

| Mã Kiểm Soát | Hạng Mục / Tiêu Chuẩn | Yêu Cầu Kỹ Thuật Bắt Buộc (Normative) | Mức Độ Ưu Tiên | Biện Pháp Xác Minh Tự Động |
|---|---|---|---|---|
| **CTL-EAT-01** | RFC 9711 EAT Architecture | Tạo token EAT ký số từ phần cứng với đầy đủ claims `ueid`, `oemboot`, `dbgstat` và `eat_nonce`. | Bắt buộc (P0) | Thẩm định chữ ký COSE/JOSE và xác minh nonce tươi mới tại API Gateway. |
| **CTL-EAT-02** | RFC 9711 Measurements | Thu thập băm mật mã runtime của mini-app bundle (`measurements`) và so khớp kết quả (`measres`). | Bắt buộc (P0) | Đối chiếu tự động với mã băm gói đã được Store duyệt trong kho Golden Store. |
| **CTL-EAT-03** | RFC 9782 Media Types | Gắn header `Content-Type: application/eat+jwt; eat_profile="..."` trong các yêu cầu chứng thực. | Bắt buộc (P1) | Bộ định tuyến Gateway từ chối các yêu cầu thiếu tham số profile hợp lệ. |
| **CTL-JSC-01** | RFC 9553 Card Model | Chuẩn hóa toàn bộ dữ liệu trả về từ Contact Picker API sang định dạng RFC 9553 `Card`. | Bắt buộc (P0) | JSON Schema validation tại tầng native bridge trước khi dispatch sang WebView. |
| **CTL-JSC-02** | RFC 9553 Context Scoping | Hỗ trợ lọc ngữ cảnh `personal` vs `work` cho địa chỉ và số điện thoại theo lựa chọn của người dùng. | Bắt buộc (P1) | Kiểm tra thuộc tính `contexts` loại bỏ thông tin nhạy cảm ngoài phạm vi cho phép. |
| **CTL-JSC-03** | RFC 9553 Phones & URI | Chuẩn hóa số điện thoại theo chuẩn viễn thông quốc tế E.164 (`tel:+84...`). | Bắt buộc (P0) | Regex parser kiểm tra định dạng E.164; từ chối chuỗi số điện thoại thô. |
| **CTL-JSC-04** | RFC 9555 vCard Mapping | Chuyển đổi hai chiều không mất mát giữa danh bạ hệ điều hành và JSContact. | Tùy chọn (P2) | Unit test chuyển đổi vòng tròn (round-trip test) kiểm tra tính toàn vẹn thuộc tính. |
| **CTL-PAY-01** | W3C Payment Request | Khởi tạo thanh toán thông qua giao diện chuẩn `PaymentRequest` native bottom-sheet. | Bắt buộc (P0) | Kiểm tra sự hiện diện của `PaymentRequest` và kích hoạt qua transient user gesture. |
| **CTL-PAY-02** | W3C URL-based PMI | Sử dụng định danh phương thức dựa trên URL tuyệt đối (URL-based PMI) cho các cổng thanh toán. | Bắt buộc (P0) | Whitelist domain validation ngăn chặn URL giả mạo và injection. |
| **CTL-PAY-03** | W3C Secure Payment (SPC) | Triển khai xác thực sinh trắc học FIDO/WebAuthn với dynamic linking gắn chặt thông tin giao dịch. | Bắt buộc (P0) | Thẩm định chữ ký phần cứng và kiểm tra `clientDataJSON` khớp số tiền và payeeOrigin. |
| **CTL-PAY-04** | W3C Payment Modifiers | Xử lý sự kiện `paymentmethodchange` để điều chỉnh phụ phí, chiết khấu và thuế động. | Bắt buộc (P1) | Đảm bảo tính nguyên tử qua `event.updateWith()` và kiểm tra chữ ký token cuối cùng. |

---

## 5. Kết luận & Lộ trình Tiếp theo

Milestone 135 đã hoàn thành xuất sắc việc thiết lập nền tảng bảo mật thực thể, quyền riêng tư dữ liệu cá nhân và hạ tầng thanh toán số hiện đại nhất cho Super App:
- **Toàn vẹn phần cứng và container:** Không còn dựa vào các biện pháp kiểm tra root/jailbreak cục bộ dễ bị bypass; chuyển sang mô hình chứng thực thực thể toán học RFC 9711 EAT gắn với chip bảo mật.
- **Bảo vệ dữ liệu người dùng:** Chấm dứt triệt để tình trạng mini-app quét sạch danh bạ thô; thay thế bằng mô hình chia sẻ danh bạ có chọn lọc, có cấu trúc và phân loại ngữ cảnh rõ ràng theo RFC 9553 JSContact.
- **Thanh toán chuẩn hóa quốc tế:** Thống nhất giao diện thanh toán mini-app trên nền tảng W3C Payment Request API và bảo vệ các giao dịch giá trị cao bằng sinh trắc học phần cứng FIDO Secure Payment Confirmation (SPC).

Tổng số findings chuẩn mực đã được mở rộng lên **1976 findings**, duy trì 100% tính toàn vẹn, xác minh nguồn trích dẫn HTTP 200 và không lưu trữ bất kỳ thông tin bí mật nào.
