# Chuyên Đề 133: OpenID SIOPv2, DIF Peer DID Method (did:peer) & ISO/IEC 18013-5 Mobile Driving Licence (mDL)

## 1. Bối Cảnh & Mục Tiêu Chuẩn Hóa
Trong quá trình tiến hóa từ siêu ứng dụng đơn lẻ sang hệ sinh thái mở đa nền tảng, bài toán định danh số (Digital Identity), quyền riêng tư người dùng (User Privacy) và xác thực không phụ thuộc máy chủ trung tâm (Self-Sovereign Identity) trở thành trục phòng thủ trọng yếu. Mini app store hiện đại không thể tiếp tục dựa vào mô hình ID Token tập trung kiểu cũ (vốn dễ bị nhà cung cấp danh tính theo dõi hành vi xuyên ứng dụng), mà phải chuẩn hóa các cơ chế xác thực tự thân (Self-Issued Provider), định danh ngang hàng không tốn phí sổ cái (Peer DIDs), và các chứng chỉ số đạt chuẩn nhà nước/quốc tế (mDL theo ISO/IEC 18013-5).

Chuyên đề 133 tổng hợp và chuẩn hóa 15 phát hiện kỹ thuật cốt lõi (Findings `siopv2_133_01` đến `mdl_iso_133_15`) thuộc 3 trụ cột kỹ thuật mật mã:
1. **OpenID Connect Self-Issued OpenID Provider v2 (SIOPv2)**: Kiến trúc nơi ví di động hoặc kho khóa của chính Super App đóng vai trò OpenID Provider (OP) cục bộ, hỗ trợ bắt tay xác thực trực tiếp qua `direct_post.jwt`, khóa ký `sub_jwk` tạo định danh giả danh theo cặp (pairwise pseudonymous), và khả năng tích hợp trình bày bằng chứng xác minh (OID4VP) trong một cử chỉ người dùng duy nhất.
2. **DIF Peer DID Method (did:peer) Specification**: Phương pháp định danh phi tập trung hoàn toàn ngoại tuyến, không tốn phí mạng lưới (zero gas/ledger cost), bảo vệ tuyệt đối tính riêng tư giữa hai bên (bilateral privacy) qua Method 0 (inception key), Method 1 (genesis hash), Method 2 (inline multi-key/service encoding), Method 4 (dynamic CRDT delta evolution), kết hợp bao thư bảo mật DIF DIDComm Messaging v2.0 trên IPC bridge.
3. **ISO/IEC 18013-5 Mobile Driving Licence (mDL) & Digital Identity Sandbox**: Cấu trúc dữ liệu nhị phân CBOR/COSE Sign1 với gốc tin cậy MobileSecurityObject (MSO), cơ chế tiết lộ có chọn lọc dựa trên bảng băm (digest-based selective disclosure), bắt tay ECDH xác thực lẫn nhau giữa thiết bị giữ và thiết bị đọc (Device & Reader Mutual Authentication), cầu nối lên Web qua W3C Digital Credentials API (`navigator.identity.get`), và bảo vệ khóa trong Secure Enclave / StrongBox phục vụ eKYC ngân hàng, viễn thông.

---

## 2. Chi Tiết Các Trụ Cột Kỹ Thuật

### 2.1. OpenID Connect Self-Issued OpenID Provider v2 (SIOPv2)
- **Mô hình kiến trúc OP cục bộ**:
  - Không cần máy chủ IdP trung gian. Thiết bị di động của người dùng tự vận hành một OpenID Provider tuân thủ chuẩn OIDC Core 1.0.
  - Định danh issuer chuẩn hóa là `https://self-issued.me/v2` hoặc biểu diễn dưới dạng DID (ví dụ `did:key:...`, `did:peer:...`).
  - Hỗ trợ khám phá siêu dữ liệu (Discovery & Client Metadata) với các thuật toán ký ES256, EdDSA và các kiểu cú pháp chủ thể (`subject_syntax_types_supported`).
- **Thương lượng yêu cầu & Phản hồi an toàn (Direct Post)**:
  - Mini-app (RP) gửi yêu cầu xác thực với `scope=openid`, `response_type=id_token` (hoặc kết hợp `id_token vp_token`).
  - Sử dụng chế độ phản hồi `response_mode=direct_post` hoặc `direct_post.jwt` (theo JARM), đẩy kết quả thẳng tới endpoint của Relying Party mà không làm lộ token lên lịch sử trình duyệt hay URI chuyển hướng front-channel.
  - Sử dụng tham số `nonce` và `state` có độ hỗn loạn cao (entropy >= 128 bit) để triệt tiêu tấn công phát lại (replay attacks).
- **Cấu trúc Self-Issued ID Token & Khóa ký `sub_jwk`**:
  - ID Token được ký bằng khóa bất đối xứng sinh từ phần cứng thiết bị.
  - Trường `sub` chứa mã băm base64url SHA-256 của khóa công khai (theo RFC 7638 JWK Thumbprint) hoặc DID của người dùng.
  - Khóa công khai tương ứng được đính kèm trong claim hoặc header `sub_jwk`. Mini-app xác minh chữ ký số của token và kiểm tra tính toàn vẹn của mã băm để xác nhận định danh người dùng.
  - Ràng buộc thời gian sống ngắn (`exp` <= 5 phút từ `iat`) và đối tượng nhận `aud` khớp chính xác với client_id của mini-app.
- **Tích hợp OID4VP & Định danh giả danh theo cặp (Pairwise Pseudonymity)**:
  - Hợp nhất quy trình đăng nhập và nộp hồ sơ định danh: Một yêu cầu SIOPv2 có thể kèm truy vấn Verifiable Presentation qua `dcql_query` hoặc `presentation_definition`.
  - Sinh khóa theo cặp cho từng mini-app (Pairwise Keys): Với mỗi mini-app khác nhau, container tự động dẫn xuất một cặp khóa Ed25519/P-256 riêng biệt bằng HKDF, ngăn chặn hoàn toàn việc các mini-app liên kết dữ liệu nhằm vẽ chân dung người dùng.

### 2.2. DIF Peer DID Method (did:peer) Specification
- **Định danh ngang hàng không sổ cái (Off-ledger Zero-cost DIDs)**:
  - Khắc phục nhược điểm chi phí cao và độ trễ của blockchain (did:ion, did:indy) cũng như rủi ro phụ thuộc DNS của did:web.
  - Dành riêng cho quan hệ song phương (bilateral) giữa hai thực thể: giữa mini-app và container Super App, hoặc giữa hai mini-app với nhau.
  - Tuân thủ đầy đủ mô hình dữ liệu trừu tượng W3C DID Core 1.0 với các phương thức xác minh (`authentication`, `assertionMethod`, `keyAgreement`).
- **Các thuật toán khởi tạo Inception**:
  - **Method 0 (`did:peer:0...`)**: Khởi tạo từ dấu vân tay khóa đơn lẻ (tương tự `did:key`). Không cần đồng bộ trạng thái, thích hợp cho các worker thread ngắn hạn.
  - **Method 1 (`did:peer:1...`)**: Khởi tạo từ mã băm SHA-256 của tài liệu DID gốc (Genesis Document). Toàn bộ tài liệu được trao đổi một lần duy nhất qua kênh bắt tay ban đầu; bên nhận kiểm tra tính toán băm để xác lập gốc tin cậy bất biến.
  - **Method 2 (`did:peer:2...`)**: Nhúng trực tiếp nhiều khóa công khai (`.V` cho ký duyệt, `.E` cho mã hóa X25519) và endpoints dịch vụ (`.S`) vào trong chuỗi định danh bằng định dạng phân tách dấu chấm. Cho phép phân giải 100% độc lập (self-contained resolution) mà không cần bước trao đổi tài liệu genesis.
- **Tiến hóa trạng thái động (Method 4 Dynamic Evolution)**:
  - Chuẩn hóa việc xoay vòng khóa (key rotation) và cập nhật endpoint thông qua chuỗi các bản ghi chênh lệch (delta patches) được ký điện tử.
  - Đồng bộ theo nguyên lý CRDT hoặc sổ cái vi mô song phương mà không cần mạng đồng thuận blockchain.
  - Phát hiện và ngắt kết nối lập tức nếu phát hiện chữ ký delta không hợp lệ hoặc có dấu hiệu phân nhánh trạng thái (state bifurcation).
- **Cách ly kênh truyền thông với DIF DIDComm Messaging v2.0**:
  - Toàn bộ gói tin giao tiếp giữa mini-app và host container được mã hóa đầu cuối (AEAD A256GCM/ChaCha20-Poly1305) bằng khóa dẫn xuất từ X25519 keyAgreement của `did:peer`.
  - Đảm bảo tính bí mật hoàn hảo về sau (Perfect Forward Secrecy): Ngay cả khi bộ nhớ WebView hoặc log hệ thống bị lộ, các phiên giao dịch trong quá khứ vẫn không thể bị giải mã.

### 2.3. ISO/IEC 18013-5 Mobile Driving Licence (mDL) & Digital Identity Sandbox
- **Kiến trúc an ninh mDL & MobileSecurityObject (MSO)**:
  - Tiêu chuẩn quốc tế cho giấy phép lái xe và căn cước điện tử trên thiết bị di động, được triển khai rộng rãi tại Bắc Mỹ, Châu Âu và APAC.
  - Dữ liệu được đóng gói nhị phân CBOR (RFC 8949) và bảo vệ bằng chữ ký số COSE_Sign1 (RFC 9052).
  - Trọng tâm an ninh là `MobileSecurityObject` (MSO) do cơ quan có thẩm quyền phát hành ký. MSO không chứa dữ liệu thô mà chứa bảng ánh xạ mã băm (SHA-256 digests) của từng phần tử dữ liệu trong namespace `org.iso.18013.5.1`.
- **Tiết lộ có chọn lọc dựa trên bảng băm (Digest-Based Selective Disclosure)**:
  - Mỗi thuộc tính danh tính nằm trong một cấu trúc `IssuerSignedItem` gồm mã định danh, muối ngẫu nhiên (salt) và giá trị thuộc tính.
  - Khi mini-app chỉ yêu cầu chứng minh độ tuổi (ví dụ: `age_over_18`), ví Super App chỉ trích xuất duy nhất phần tử này kèm muối ngẫu nhiên.
  - Trình xác minh mini-app tính toán lại mã băm của phần tử được cung cấp và đối chiếu với digest tương ứng trong MSO đã ký. Các thông tin nhạy cảm khác (họ tên, địa chỉ, số CMND/CCCD, ngày sinh chính xác) được bảo vệ tuyệt đối.
- **Bắt tay thiết bị & Xác thực hai chiều (Device & Reader Mutual Authentication)**:
  - Trao đổi khóa tạm ECDH (P-256 hoặc Brainpool) tạo cặp khóa phiên `SKDevice` và `SKReader`.
  - Thiết bị người dùng ký chứng minh sở hữu phần cứng qua `DeviceSignature` hoặc `DeviceMac`.
  - Thiết bị đọc (Reader/Mini-app) phải xuất trình chứng chỉ X.509 hợp lệ và ký tham số phiên. Super App từ chối cung cấp dữ liệu mDL nếu bên yêu cầu không có chứng chỉ Reader được cấp phép.
- **Cầu nối lên Web & Mô đun phần cứng bảo mật**:
  - Mini-app gọi API chuẩn của trình duyệt `navigator.identity.get()` (W3C Digital Credentials API) với bộ chọn mdoc hoặc thông qua OID4VP `format: mso_mdoc`.
  - Khóa ký thiết bị (`DeviceKey`) bắt buộc lưu trữ trong phần cứng chống can thiệp: Android StrongBox Keystore hoặc Apple Secure Enclave (đạt chuẩn FIPS 140-3 Level 2/3 hoặc Common Criteria EAL4+).
  - Bắt buộc xác thực sinh trắc học người dùng (vân tay, khuôn mặt) tại thời điểm xuất trình, tạo hồ sơ bằng chứng eKYC có giá trị pháp lý cao cho các mini-app tài chính ngân hàng.

---

## 3. Khuyến Nghị Áp Dụng Chuẩn Mini App Store

1. **Chuẩn Hóa Đăng Nhập Tự Thân (SIOPv2) Trong Mini App Store**:
   - Tích hợp sẵn OP runtime cục bộ trong Super App Client SDK, cung cấp API JavaScript `superapp.auth.requestSIOPAssertion(options)` trả về Self-Issued ID Token trong < 50ms.
   - Bắt buộc mini-app tiếp nhận xác thực qua chế độ `direct_post.jwt` với khóa ký mã hóa JARM, ngăn chặn hoàn toàn rò rỉ token trên IPC logs.
   - Triển khai giao diện Consent Sheet thống nhất của container hiển thị rõ ràng danh tính đã kiểm duyệt của mini-app trước khi phát hành ID Token.

2. **Áp Dụng did:peer & DIDComm Cho Giao Tiếp Cố Trình Giữa Mini App & Native Host**:
   - Sử dụng định dạng `did:peer:2...` làm định danh kênh truyền cho từng phiên làm việc của mini-app, mã hóa cả khóa Ed25519 (ký) và X25519 (mã hóa).
   - Tự động sinh khóa theo cặp (pairwise) khi khởi chạy mini-app; tuyệt đối không tái sử dụng DID qua lại giữa các mini-app độc lập.
   - Kích hoạt cơ chế Method 4 để hỗ trợ xoay vòng khóa không gián đoạn đối với các phiên làm việc doanh nghiệp kéo dài.

3. **Thiết Lập Khung Tuân Thủ eKYC Với mDL ISO/IEC 18013-5**:
   - Phân hạng quyền truy cập danh tính trong App Store Console: Phân loại quyền truy cập mDL vào nhóm quyền hạn tối cao (`Privileged / Regulated Scope`).
   - Kiểm duyệt chặt chẽ chính sách giảm thiểu dữ liệu (Data Minimization): Từ chối các mini-app yêu cầu toàn bộ hồ sơ mDL khi chỉ cần các cờ xác nhận logic (như `age_over_18` hoặc `driving_privileges`).
   - Bắt buộc xác thực sinh trắc học trực tiếp trên phần cứng (Hardware Biometric Gating) đối với mọi yêu cầu xuất trình mDL, ghi nhật ký kiểm toán bất biến (tamper-evident audit logs) tại tầng máy chủ bảo mật của Super App.
