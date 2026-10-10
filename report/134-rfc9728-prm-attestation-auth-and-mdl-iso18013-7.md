# Chuyên Đề 134: RFC 9728 Protected Resource Metadata (PRM), OAuth Attestation-Based Client Authentication & ISO/IEC 18013-7 Remote mDL

## 1. Bối Cảnh & Mục Tiêu Chuẩn Hóa
Trong kiến trúc bảo mật của siêu ứng dụng (Super App) hiện đại, việc tích hợp hàng trăm mini-app của các đối tác thứ ba đặt ra ba thách thức phòng thủ mang tính quyết định:
1. **Khám phá và ràng buộc tài nguyên động**: Làm thế nào để mini-app và gateway phân giải chính xác cấu hình của máy chủ tài nguyên (Resource Server - RS), danh sách máy chủ ủy quyền (Authorization Server - AS) hợp lệ, các phạm vi (scopes), và cơ chế ràng buộc token (DPoP) mà không cần cấu hình tĩnh dễ gây sai sót hay mở ra nguy cơ tấn công mạo danh (impersonation / mix-up attacks).
2. **Xác thực ứng dụng khách dựa trên phần cứng (Hardware Attestation)**: Thay thế hoàn toàn mô hình public client truyền thống (vốn không thể bảo vệ bí mật tĩnh client secret trên thiết bị di động) bằng cơ chế chứng thực tính toàn vẹn của mã nhị phân và môi trường thực thi từ phần cứng (Google Play Integrity, Apple App Attest, StrongBox / Secure Enclave), đảm bảo chỉ các container siêu ứng dụng và gói mini-app nguyên bản mới được cấp quyền truy cập tài nguyên tài chính, viễn thông.
3. **Xác thực định danh từ xa qua mạng (Remote Digital Identity Verification)**: Mở rộng tiêu chuẩn căn cước điện tử và giấy phép lái xe di động (mDL) từ môi trường vật lý cự ly gần (NFC/BLE theo ISO/IEC 18013-5) sang không gian trực tuyến/web (ISO/IEC 18013-7 kết hợp OID4VP và W3C Digital Credentials API), đáp ứng tiêu chuẩn nghiêm ngặt của khung định danh số châu Âu (EUDI Wallet ARF) và bảo vệ tối đa quyền riêng tư bằng các phép kiểm chứng độ tuổi logic (`age_over_18`, `age_over_21`).

Chuyên đề 134 chuẩn hóa 15 phát hiện kỹ thuật (Findings `prm_rfc9728_134_01` đến `mdl_remote_134_15`) thuộc 3 trụ cột tiêu chuẩn quốc tế đã được phê duyệt.

---

## 2. Chi Tiết Các Trụ Cột Kỹ Thuật

### 2.1. IETF RFC 9728: OAuth 2.0 Protected Resource Metadata (PRM)
- **Kiến trúc khám phá cấu hình máy chủ tài nguyên**:
  - RFC 9728 hoàn thiện bộ ba tiêu chuẩn siêu dữ liệu máy-đọc-được của OAuth 2.0, đặt ngang hàng với RFC 7591 (Dynamic Client Registration) và RFC 8414 (Authorization Server Metadata).
  - Điểm truy cập chuẩn hóa: Chèn chuỗi đường dẫn `.well-known` vào định danh tài nguyên để tạo URL khám phá: `/.well-known/oauth-protected-resource`.
  - Tham số bắt buộc: `resource` là BẮT BUỘC (REQUIRED) và phải trùng khớp chính xác 100% với định danh tài nguyên được truy vấn. Mọi sự sai lệch phải dẫn đến việc hủy bỏ kết nối ngay lập tức nhằm ngăn chặn kẻ tấn công chuyển hướng người dùng sang máy chủ tài nguyên giả mạo.
- **Các tham số siêu dữ liệu cốt lõi & Đăng bạ IANA**:
  - `authorization_servers`: Danh sách các URL của Authorization Server đáng tin cậy mà RS này chấp nhận token truy cập. Cho phép mini-app xác định chính xác AS cần liên hệ.
  - `jwks_uri`: Địa chỉ bộ khóa công khai của RS phục vụ việc xác minh chữ ký các phản hồi tài nguyên (JARM) hoặc mã hóa dữ liệu trả về.
  - `scopes_supported`: Danh sách các quyền/phạm vi được hỗ trợ tại tài nguyên này.
  - `bearer_methods_supported`: Các phương thức gửi token mang (`['header', 'body', 'query']`). Các hồ sơ bảo mật nghiêm ngặt bắt buộc chỉ chấp nhận `['header']`, cấm hoàn toàn truyền token qua query string URL.
  - `dpop_bound_access_tokens_required`: Cờ boolean chỉ thị bắt buộc sử dụng token DPoP (RFC 9449) có ràng buộc người gửi.
- **Siêu dữ liệu có chữ ký số (Signed Metadata)**:
  - Cung cấp dưới dạng trường `signed_metadata` chứa một JWT ký bằng JWS.
  - Claim `iss` xác định cơ quan chứng thực (chính RS hoặc trung tâm kiểm duyệt Super App).
  - Khi người tiêu dùng siêu dữ liệu hỗ trợ `signed_metadata`, các giá trị bên trong JWT có chữ ký số bắt buộc phải được ưu tiên áp dụng (precedence) so với các trường JSON trần bên ngoài, vô hiệu hóa các cuộc tấn công can thiệp chỉnh sửa trên đường truyền mạng hoặc proxy trung gian.
- **Khám phá động qua phản hồi thử thách HTTP 401 (WWW-Authenticate)**:
  - Khi mini-app thực hiện yêu cầu không kèm token hoặc token không đủ quyền, RS trả về mã lỗi 401 Unauthorized kèm header:
    `WWW-Authenticate: Bearer resource_metadata="https://api.example.com/.well-known/oauth-protected-resource"`
  - Quy tắc kiểm tra bảo mật: Giá trị `resource` trong siêu dữ liệu lấy về bắt buộc phải trùng khớp với URL mà client đã dùng để thực hiện yêu cầu ban đầu.
  - Kết hợp hoàn hảo với OAuth 2.0 Step-Up Authentication (RFC 9470), cho phép mini-app tự động nâng cấp quyền hạn hoặc xác thực lại theo đúng yêu cầu thời gian thực của backend.
- **Kiểm tra chéo hai chiều (Bidirectional Cross-Validation) ngăn chặn mạo danh**:
  - Authorization Server công bố danh sách các tài nguyên hợp lệ qua tham số `protected_resources` (được định nghĩa trong RFC 9728 mở rộng cho RFC 8414).
  - Client / Gateway thực hiện kiểm tra chéo hai chiều: RS phải liệt kê AS trong `authorization_servers`, VÀ AS phải liệt kê RS trong `protected_resources`.
  - Triệt tiêu triệt để cuộc tấn công Mix-Up Attack, trong đó một RS độc hại cố tình chỉ định client gửi token người dùng tới một AS ngoài luồng nhằm đánh cắp token truy cập diện rộng.

---

### 2.2. IETF draft-ietf-oauth-attestation-based-client-auth: Client Attestation Architecture
- **Xác thực thực thể ứng dụng khách di động (Client Instance)**:
  - Giải quyết lỗ hổng cấu trúc cố hữu của ứng dụng di động: Ứng dụng di động là public client, không thể lưu trữ an toàn client secret tĩnh trong mã nguồn hoặc file cấu hình (dễ bị dịch ngược, trích xuất bằng APKTool/Frida).
  - Giới thiệu 4 vai trò kiến trúc: Client Instance (bản cài đặt siêu ứng dụng trên máy người dùng), Client Attester (dịch vụ chứng thực hệ thống từ Apple/Google/Samsung/Enclave), Authorization Server (AS) và Resource Server (RS).
  - Thiết kế bảo vệ quyền riêng tư: Client Instance lấy chứng chỉ chứng thực từ Client Attester mà không để lộ AS đích, ngăn chặn các nhà cung cấp nền tảng theo dõi hành vi tương tác dịch vụ của người dùng.
- **Token chứng thực ràng buộc khóa phần cứng (`cnf.jwk`)**:
  - Client Instance sinh một cặp khóa bất đối xứng cục bộ bên trong phần cứng bảo mật (Secure Enclave / Android StrongBox Keystore).
  - Client Attester ký một token chứng thực (Client Attestation JWT), xác nhận tính toàn vẹn của ứng dụng (mã hash nhị phân, chứng chỉ ký, trạng thái khóa boot) và nhúng khóa công khai của client vào claim `cnf.jwk`.
  - Khóa riêng tương ứng không bao giờ rời khỏi chip bảo mật của thiết bị, biến token chứng thực thành một chứng nhận danh tính bất biến và không thể sao chép sang thiết bị khác.
- **Bằng chứng sở hữu Attestation PoP (Proof-of-Possession)**:
  - Khi gửi yêu cầu lấy token tới AS, client xuất trình đồng thời hai header HTTP:
    - `OAuth-Client-Attestation`: Chứa Client Attestation JWT do attester cấp (thời gian sống 5–15 phút).
    - `OAuth-Client-Attestation-PoP`: Chứa JWT bằng chứng sở hữu do client tự ký bằng khóa riêng trong phần cứng (thời gian sống <= 60 giây).
  - PoP JWT chứa các claim: `aud` (URL token endpoint của AS), `iat`, `exp`, `jti`, và mã ngẫu nhiên `nonce` do AS cung cấp.
  - Tùy chọn ký mã băm của nội dung yêu cầu HTTP (phương thức `htm`, đường dẫn `htu`, hash của body `ath`), đảm bảo ngăn chặn hoàn toàn việc kẻ tấn công chụp trộm gói tin để phát lại (anti-replay).
- **Phòng chống thiết bị giả lập, bot farm và mã nhị phân can thiệp**:
  - Attester xác thực trạng thái khởi động an toàn (Verified Boot), tính toàn vẹn của nhân hệ điều hành (kernel integrity), và trạng thái không bị can thiệp bởi các framework hook (Frida, Xposed, Magisk).
  - Chặn đứng các mạng lưới bot gian lận, farm tài khoản và công cụ quét tự động masquerading như ứng dụng di động chính thức.
  - Cung cấp nhật ký kiểm toán không thể chối bỏ về mức độ an toàn phần cứng của thiết bị tại thời điểm thực hiện giao dịch tài chính nhạy cảm.
- **Cơ chế cầu nối chứng thực hai tầng (Two-Tier Attestation Bridge)**:
  - Trong siêu ứng dụng, container máy chủ đóng vai trò attester cục bộ và cầu nối ủy quyền cho các mini-app nhúng.
  - Khi mini-app cần gọi ra AS doanh nghiệp của đối tác, host container đính kèm chứng thực phần cứng của container đồng thời nhúng mã băm gói phần mềm của mini-app (`mini_app_package_hash`) và DID của nhà phát triển vào ngữ cảnh chứng thực.
  - Cho phép các mini-app độc lập đạt được mức độ xác thực doanh nghiệp cao nhất mà không cần quản lý khóa tĩnh hay lộ thông tin đăng nhập nhạy cảm.

---

### 2.3. ISO/IEC 18013-7 & OpenID4VP Remote mDL Architecture
- **Chuẩn hóa xác thực trực tuyến mDL & Hồ sơ OpenID4VP**:
  - ISO/IEC 18013-7 mở rộng tiêu chuẩn giấy phép lái xe và căn cước số trên thiết bị di động (mDL/mdoc) từ môi trường tiếp xúc gần (NFC/BLE theo ISO/IEC 18013-5) lên môi trường mạng trực tuyến / web.
  - Sử dụng giao thức OpenID for Verifiable Presentations (OID4VP) làm lớp vận chuyển chuẩn mực chạy trên HTTPS/REST.
  - Hỗ trợ ba mô hình tương tác chính: Yêu cầu trực tiếp trong mini-app WebView, chuyển hướng ứng dụng-qua-ứng dụng (app-to-app wallet flow), và quét mã QR tương tác xuyên thiết bị (cross-device flow) giữa máy tính/kiosk và điện thoại.
- **Ràng buộc ngữ cảnh phiên làm việc (Session Transcript Binding) chống MITM**:
  - Để ngăn chặn kẻ trung gian nghe lén (Man-in-the-Middle) và tấn công chuyển tiếp (relay attacks), chữ ký xác thực thiết bị phải được ràng buộc mật mã với phiên làm việc từ xa.
  - Cấu trúc `SessionTranscript` được xây dựng bằng cách kết hợp mã băm SHA-256 của `client_id` của bên xác minh, giá trị ngẫu nhiên `nonce` của phiên, và các tham số kênh truyền TLS.
  - Phần cứng bảo mật của điện thoại ký trực tiếp lên mã băm của `SessionTranscript` này trong cấu trúc `DeviceSignature` của CBOR mdoc. Bất kỳ sự thay đổi nào về redirect URI, danh tính người yêu cầu hay nonce đều làm chữ ký phần cứng vô hiệu lập tức.
- **Bảo vệ quyền riêng tư & Tiết lộ có chọn lọc (Selective Disclosure)**:
  - Phân vùng dữ liệu danh tính vào các namespace chuẩn hóa (`org.iso.18013.5.1` cho dữ liệu lái xe toàn cầu, `org.iso.18013.5.1.aamva` cho Bắc Mỹ, hoặc namespace quốc gia).
  - Hỗ trợ các thuộc tính vị từ logic (boolean predicates) như `age_over_18`, `age_over_21` thay vì phải tiết lộ toàn bộ ngày tháng năm sinh, họ tên hay địa chỉ thường trú.
  - Cơ quan phát hành ký bảng ánh xạ mã băm (digests) trong `MobileSecurityObject` (MSO). Ví chỉ giải mã và gửi duy nhất thuộc tính được yêu cầu; mini-app đối chiếu mã băm với MSO mà không thể thu thập thêm bất kỳ dữ liệu cá nhân nào khác.
  - Đáp ứng trọn vẹn nguyên tắc giảm thiểu dữ liệu (Data Minimization) của GDPR, NĐ 13/2023/NĐ-CP và Luật Bảo vệ Dữ liệu Cá nhân Việt Nam.
- **Đồng bộ với Khung Kiến Trúc Ví Định Danh Số Châu Âu (EUDI Wallet ARF)**:
  - EUDI Wallet ARF định vị ISO/IEC 18013-7 và OID4VP là tiêu chuẩn cốt lõi cho các giao dịch định danh số xuyên biên giới đạt Mức độ Đảm bảo Cao (Level of Assurance High).
  - Bắt buộc khóa ký thiết bị phải được lưu trữ trong Thiết bị Tạo Chữ ký Đạt chuẩn (QSCD) hoặc Secure Element (eSE/SIM/eUICC) đạt chuẩn FIPS 140-3 Level 3 / CC EAL4+.
  - Kết nối với danh sách tin cậy quốc gia (Trust Registries) và nhà cung cấp dịch vụ tin cậy đủ điều kiện (QTSP) để kiểm tra tính hợp lệ của chứng chỉ cơ quan cấp phát trong thời gian thực.
- **Cầu nối API JavaScript chuẩn hóa trong Mini App Container**:
  - Siêu ứng dụng cung cấp API JavaScript chuẩn hóa:
    `superapp.identity.requestMdocPresentation({ docType: 'org.iso.18013.5.1.mDL', elements: ['age_over_18'] })`
  - Container xử lý toàn bộ các thao tác nhị phân CBOR, ký số COSE và ràng buộc SessionTranscript hoàn toàn bên trong tầng mã nguồn native độc lập với WebView.
  - Hiển thị bảng hỏi ý kiến (Consent Sheet) minh bạch cho người dùng và lưu trữ nhật ký kiểm toán bất biến (audit trail) phục vụ việc tuân thủ pháp lý tài chính, ngân hàng và chống rửa tiền (AML/eKYC).

---

## 3. Khuyến Nghị Thực Thi Tiêu Chuẩn Cho Mini App Store

1. **Chuẩn Hóa Khám Phá Tài Nguyên Với RFC 9728 PRM**:
   - Bắt buộc tất cả các máy chủ API nội bộ và đối tác tích hợp trong siêu ứng dụng xuất bản tài liệu PRM tại `/.well-known/oauth-protected-resource`.
   - Cấu hình Super App API Gateway tự động truy vấn và kiểm tra chéo hai chiều (bidirectional validation) giữa PRM của đối tác và danh mục `protected_resources` của Authorization Server trung tâm.
   - Kích hoạt bắt buộc cờ `dpop_bound_access_tokens_required: true` cho mọi endpoint tài chính và nhạy cảm, loại bỏ hoàn toàn việc sử dụng token Bearer không ràng buộc người gửi.

2. **Áp Dụng Client Attestation Cho Mọi Kết Nối OAuth Từ Thiết Bị Di Động**:
   - Loại bỏ hoàn toàn các chuỗi client secret tĩnh nhúng trong mã nguồn mini-app hoặc SDK di động.
   - Tích hợp mô-đun sinh khóa phần cứng và Client Attestation PoP vào Super App Native Shell, tự động gắn kèm header `OAuth-Client-Attestation` và `OAuth-Client-Attestation-PoP` khi thực hiện yêu cầu cấp token.
   - Thiết lập chính sách kiểm duyệt: Chặn đứng các thiết bị không đạt chuẩn tính toàn vẹn phần cứng (`MEETS_STRONG_INTEGRITY`) khi truy cập các mini-app ngân hàng, chứng khoán hoặc bảo hiểm.

3. **Thiết Lập Cổng Định Danh Trực Tuyến ISO/IEC 18013-7 & OID4VP**:
   - Xây dựng phân hệ Ví Danh Tính Số (Digital Identity Wallet) trong siêu ứng dụng hỗ trợ lưu trữ chứng chỉ mDL/mdoc và xuất trình trực tuyến qua giao thức OID4VP.
   - Ban hành quy chế kiểm duyệt mini-app: Bắt buộc mini-app chỉ được yêu cầu các vị từ logic (`age_over_18`) khi xác thực độ tuổi; cấm tuyệt đối việc yêu cầu toàn bộ thông tin cá nhân của người dùng nếu không có giấy phép kinh doanh ngành nghề có điều kiện.
   - Lưu trữ nhật ký kiểm toán cục bộ có chữ ký số xác nhận cho từng lần xuất trình mdoc, cho phép người dùng tra cứu lịch sử chia sẻ dữ liệu trực tiếp trong mục Cài đặt Bảo mật của Super App.
