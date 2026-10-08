# Chuyên đề Iteration 41: Web Packaging Binary Bundles, Remote Attestation (RATS) & Verifiable Credentials / Selective Disclosure

## 1. Bối cảnh & Mục tiêu Nghiên cứu Iteration 41

Hệ sinh thái mini-app trong một siêu ứng dụng (Super-App) cấp độ doanh nghiệp (Enterprise / Financial / Public Services) đối mặt với 3 thách thức kiến trúc then chốt:
1. **Hiệu năng & toàn vẹn đóng gói ứng dụng (Offline Binary Bundling)**: Định dạng nén ZIP truyền thống đòi hỏi bung file ra đĩa flash gây chậm cold-start và rủi ro Zip Slip / Path Traversal. Cần chuẩn hóa cơ chế đóng gói nhị phân ngẫu nhiên (random-access zero-extraction mmap) dựa trên tiêu chuẩn **IETF WPack Bundled HTTP Responses** (`draft-ietf-wpack-bundled-responses`) và **WICG Isolated Web Apps (Signed Web Bundles)**.
2. **Xác thực toàn vẹn thiết bị từ xa (Remote Attestation)**: Các mini-app tài chính (cho thuê tài chính, ngân hàng, thanh toán) bắt buộc phải chứng minh môi trường máy khách (Android Keystore / iOS Secure Enclave) là thiết bị thật, không bị can thiệp root/jailbreak, hooking (Frida/Xposed) hay chạy trên máy ảo/bot farm. Tiêu chuẩn quốc tế **IETF RFC 9334 (RATS Architecture)** kết hợp **Google Play Integrity API** và **Apple App Attest Service (DeviceCheck)** thiết lập khung bằng chứng mật mã (Cryptographic Evidence) toàn diện.
3. **Định danh phi tập trung & Tối thiểu hóa dữ liệu (Verifiable Credentials & Selective Disclosure)**: Người dùng siêu ứng dụng cần cung cấp bằng chứng định danh (chứng chỉ hành nghề, tài liệu ủy quyền cho thuê, hạn mức tín dụng) cho mini-app của đối tác mà không làm lộ các dữ liệu nhạy cảm dư thừa. Khung tiêu chuẩn **W3C Verifiable Credentials Data Model v2.0**, **W3C DID Core v1.0**, **IETF SD-JWT VC** (`draft-ietf-oauth-sd-jwt-vc`), và **OpenID for Verifiable Presentations (OID4VP)** cung cấp cơ chế xác thực trường băm mật mã tối thiểu.

---

## 2. Bảng tổng hợp Findings Iteration 41 (15 Findings Chuẩn hóa)

| Mã Finding | Tiêu đề Chuẩn & Phạm vi | Nguồn / Tiêu chuẩn Quy chiếu | Cấp độ Bằng chứng | URL Xác thực |
|---|---|---|---|---|
| `webbundle_041_01` | IETF WPack Bundled HTTP Responses: Đóng gói CBOR nhị phân, gom cụm subresources và nạp ngẫu nhiên không giải nén | IETF draft-ietf-wpack-bundled-responses, RFC 8949 (CBOR) | `normative_standard` | [IETF Datatracker](https://datatracker.ietf.org/doc/draft-ietf-wpack-bundled-responses/) |
| `webbundle_041_02` | Bảng chỉ mục nhị phân Web Packaging (Section Index): Nạp subresource ngẫu nhiên O(1) và POSIX mmap zero-extraction | IETF draft-ietf-wpack-bundled-responses, IEEE Std 1003.1 (POSIX mmap) | `normative_standard` | [IETF Draft HTML](https://datatracker.ietf.org/doc/html/draft-ietf-wpack-bundled-responses-01) |
| `webbundle_041_03` | WICG Isolated Web Apps & Signed Web Bundles: Nguồn gốc mật mã suy dẫn từ khóa Ed25519 (`isolated-app://`) | WICG Isolated Web Apps, RFC 8032 (Ed25519) | `consortium_specification` | [IETF Datatracker](https://datatracker.ietf.org/doc/draft-ietf-wpack-bundled-responses/) |
| `webbundle_041_04` | HTTP 206 Partial Content Range Requests & Phân phối cập nhật vi sai khối nhị phân cho Web Bundles | RFC 9110 (HTTP Semantics §14), draft-ietf-wpack-bundled-responses | `normative_standard` | [IETF Draft HTML](https://datatracker.ietf.org/doc/html/draft-ietf-wpack-bundled-responses-01) |
| `webbundle_041_05` | Ranh giới thực thi & Chính sách Bảo mật Nội dung (CSP) nghiêm ngặt cho Bundled HTTP Exchanges trong Container WebView | W3C CSP Level 3, draft-ietf-wpack-bundled-responses, OWASP MASVS | `normative_standard` | [IETF Datatracker](https://datatracker.ietf.org/doc/draft-ietf-wpack-bundled-responses/) |
| `rats_attest_041_01` | IETF RFC 9334: Kiến trúc Remote Attestation Procedures (RATS) và Bằng chứng Mật mã Thiết bị trong Super-App | IETF RFC 9334, TCG TPM 2.0, ISO/IEC 11889 | `normative_standard` | [RFC 9334 RFC-Editor](https://www.rfc-editor.org/info/rfc9334) |
| `rats_attest_041_02` | Google Play Integrity API: Phán quyết Giải mã Token, MEETS_STRONG_INTEGRITY TEE và Kiểm tra Giấy phép App | Google Play Integrity API, Android Keystore, RFC 9334 | `platform_standard` | [Android Developers](https://developer.android.com/google/play/integrity) |
| `rats_attest_041_03` | Apple App Attest Service (DeviceCheck): Chứng thực Khóa Phần cứng Secure Enclave & Chữ ký Khẳng định Chống Replay | Apple DeviceCheck App Attest, Secure Enclave, RFC 9334 | `platform_standard` | [Apple Developer Docs](https://developer.apple.com/documentation/devicecheck/validating-apps-that-connect-to-your-server) |
| `rats_attest_041_04` | Unified Remote Attestation Bridge: Cầu nối Trừu tượng hóa Đánh giá Rủi ro và JWT Assertion cho Mini-App Backend | IETF RFC 9334, RFC 7519 (JWT), RFC 7517 (JWK) | `industry_practice` | [IETF RFC 9334 Datatracker](https://datatracker.ietf.org/doc/rfc9334/) |
| `rats_attest_041_05` | RASP & Chống Can thiệp Runtime: Phòng thủ Trình giả lập (Emulator), Hooking (Frida) và Giám sát Toàn vẹn Bytecode | OWASP MASVS-RESILIENCE, Apple App Attest Guidelines | `industry_practice` | [Apple Developer Docs](https://developer.apple.com/documentation/devicecheck/preparing-to-use-the-app-attest-service) |
| `vc_did_041_01` | W3C Verifiable Credentials Data Model v2.0: Mô hình Khẳng định Mật mã Tam giác Issuer-Holder-Verifier & Ví Super-App | W3C VC Data Model v2.0, ISO/IEC 18013-5 (mDL), W3C DID Core | `normative_standard` | [W3C VC v2.0 TR](https://www.w3.org/TR/vc-data-model-2.0/) |
| `vc_did_041_02` | W3C Decentralized Identifiers (DIDs) v1.0: Cấu trúc URI, DID Documents và Ngăn chặn Liên kết Chéo (Pairwise DIDs) | W3C DID Core v1.0, RFC 3986, W3C VC Data Model | `normative_standard` | [W3C DID Core TR](https://www.w3.org/TR/did-core/) |
| `vc_did_041_03` | IETF SD-JWT VC: Tiết lộ Chọn lọc Trường Thông tin với Muối Mật mã (Salt Digests) & Giảm thiểu Dữ liệu Cá nhân PII | IETF draft-ietf-oauth-sd-jwt-vc, RFC 7519, RFC 7515 | `normative_standard` | [IETF Datatracker](https://datatracker.ietf.org/doc/draft-ietf-oauth-sd-jwt-vc/) |
| `vc_did_041_04` | OpenID for Verifiable Presentations (OID4VP): Trao đổi Bằng chứng Xác thực Số và Kiểm soát Sinh trắc Học Host | OpenID OID4VP, DIF Presentation Exchange 2.0, W3C VC | `consortium_specification` | [W3C VC v2.0 TR](https://www.w3.org/TR/vc-data-model-2.0/) |
| `vc_did_041_05` | Quản trị Danh tính Nhà phát triển Phi tập trung: Chữ ký Gói Mini-App qua DIDs và Sổ cái Niềm tin (Trust Registry) | W3C DID Core v1.0, W3C VC Data Model, SLSA Level 3 | `industry_practice` | [W3C DID Core TR](https://www.w3.org/TR/did-core/) |

---

## 3. Phân tích Kỹ thuật Chuyên sâu

### 3.1. IETF WPack Bundled HTTP Responses & WICG Isolated Web Apps

1. **Cơ chế CBOR Encapsulation & Random Access**:
   - Web Bundles đóng gói các cặp HTTP Request/Response vào một file nhị phân duy nhất bằng chuẩn mã hóa CBOR (RFC 8949).
   - Điểm khác biệt mấu chốt so với tệp ZIP là bảng `section index`: ánh xạ trực tiếp URI tài nguyên (`/index.html`, `/app.js`, `/assets/logo.png`) tới byte offset và độ dài trong khối dữ liệu. Container siêu ứng dụng sử dụng lệnh `mmap` ở tầng OS để ánh xạ file gói vào bộ nhớ, cho phép đọc tài nguyên với độ phức tạp $O(1)$ mà không cần tốn dung lượng giải nén ra đĩa Flash (loại bỏ hoàn toàn rủi ro hao mòn flash storage và lỗ hổng bảo mật Zip Slip / Directory Traversal).
2. **Nguồn gốc Mật mã (Cryptographic Origins)**:
   - Trong WICG Isolated Web Apps, ứng dụng mini-app không gắn với một tên miền DNS thông thường mà có nguồn gốc dạng `isolated-app://<base32-encoded-ed25519-public-key>`.
   - Toàn bộ gói ứng dụng được ký số bởi khóa riêng Ed25519 của nhà phát triển hoặc cửa hàng. Mọi tài nguyên cục bộ (IndexedDB, OPFS, Cache Storage) được cô lập theo khóa công khai này. Nếu có bất kỳ byte nào bị thay đổi trong quá trình truyền tải, chữ ký sẽ không khớp và container từ chối khởi chạy.
3. **Cập nhật Vi sai Khối Byte (HTTP 206 Byte-Range Updates)**:
   - Nhờ cấu trúc chỉ mục cố định, khi có bản cập nhật nhỏ (chỉ thay đổi 1 file JS hoặc hình ảnh), client chỉ cần gửi các truy vấn `Range: bytes=start-end` (RFC 9110 §14) để kéo các khối byte thay đổi mà không phải tải lại toàn bộ gói, giảm tới 75% băng thông mạng di động.

---

### 3.2. Remote Attestation (RFC 9334 RATS), Google Play Integrity & Apple App Attest

1. **Kiến trúc RATS trong Môi trường Doanh nghiệp**:
   - RFC 9334 chuẩn hóa 4 vai trò:
     - **Attester (Máy khách)**: Thiết bị di động chứa phần cứng bảo mật (TEE, StrongBox KeyMint trên Android; Secure Enclave trên iOS).
     - **Relying Party (Máy chủ Siêu ứng dụng & Backend Mini-App)**: Nơi nhận các yêu cầu giao dịch nhạy cảm.
     - **Verifier**: Máy chủ phân tích và thẩm định bằng chứng mật mã (Evidence) dựa trên các chính sách (Appraisal Policy).
     - **Endorsement Authority**: Nhà sản xuất phần cứng/hệ điều hành chứng nhận tính hợp pháp của khóa bảo mật.
2. **Google Play Integrity API Integration**:
   - Máy khách gửi một `nonce` ngẫu nhiên do backend cấp vào cuộc gọi SDK `requestIntegrityToken()`.
   - Backend giải mã token và kiểm tra chặt chẽ:
     - `deviceIntegrity`: Đối với các giao dịch tài chính/cho thuê, bắt buộc đạt `MEETS_STRONG_INTEGRITY` (xác thực bootloader bị khóa, chạy trên phần cứng đạt chứng nhận Android TEE/StrongBox).
     - `appIntegrity`: Giá trị `appRecognitionVerdict` phải là `PLAY_RECOGNIZED` với mã SHA-256 certificate khớp với chứng chỉ phát hành chính thức của Super-App.
3. **Apple DeviceCheck DCAppAttestService**:
   - Trên iOS, container tạo cặp khóa bất đối xứng trong Secure Enclave thông qua `generateKey()`.
   - Gọi `attestKey()` để Apple ký xác nhận khóa này thuộc về thiết bị thật và App ID chỉ định.
   - Khi thực hiện giao dịch quan trọng, client gọi `signData()`, đóng gói payload cùng một bộ đếm số nguyên tăng đơn điệu (monotonic counter) vào khối CBOR. Backend xác minh chữ ký và kiểm tra bộ đếm để ngăn chặn hoàn toàn tấn công phát lại (replay attacks).
4. **Super-App Unified Attestation Bridge**:
   - Cung cấp API nội bộ `superapp.security.getAttestationAssertion()` nhằm giải phóng lập trình viên mini-app khỏi việc phải trực tiếp tích hợp SDK native phức tạp.
   - Super-App xác thực phần cứng, tổng hợp điểm rủi ro (`integrityRiskScore: 0-100`), và cấp một JWT ngắn hạn ký bằng khóa riêng của Super-App (RS256/ES256). Mini-app backend chỉ cần xác minh JWT này qua endpoint JWKS công khai (`/.well-known/jwks.json`).

---

### 3.3. W3C Verifiable Credentials v2.0, DIDs & IETF SD-JWT VC

1. **Mô hình Dữ liệu Bằng chứng Mật mã (VC v2.0)**:
   - Siêu ứng dụng đóng vai trò là Ví định danh (Digital Wallet / Holder).
   - Cơ quan có thẩm quyền hoặc doanh nghiệp (Issuer) cấp chứng chỉ số (VC) cho người dùng (Subject).
   - Khi mini-app của bên thứ ba (Verifier) yêu cầu kiểm tra điều kiện (ví dụ: tư cách người đại diện doanh nghiệp thuê tài sản), người dùng có thể xuất trình Bằng chứng Xác thực (Verifiable Presentation) kèm chữ ký chống giả mạo mà không để lộ các thông tin dư thừa.
2. **Định danh Cặp Giả danh (Pairwise Pseudonymous DIDs - W3C DID Core)**:
   - Nhằm ngăn chặn các mini-app liên kết ngầm để theo dõi hành vi người dùng, siêu ứng dụng tự động sinh các định danh phi tập trung duy nhất (`did:key:...`) riêng biệt cho từng mini-app.
   - Hai mini-app khác nhau dù thuộc cùng một nhà phát triển cũng không thể so khớp định danh của cùng một người dùng nếu không có sự đồng ý rõ ràng.
3. **Tiết lộ Chọn lọc (Selective Disclosure - SD-JWT VC)**:
   - Theo dự thảo IETF `draft-ietf-oauth-sd-jwt-vc`, các trường thông tin trong credential được băm kèm muối (salt) và đưa vào mảng `_sd` trong JWT.
   - Khi mini-app chỉ yêu cầu chứng minh "người dùng đã trên 18 tuổi" hoặc "có quốc tịch Việt Nam", ví Super-App chỉ tiết lộ chuỗi muối của đúng trường đó. Mini-app tính lại mã băm SHA-256 và so khớp với mảng `_sd` đã được ký bởi Issuer, xác thực thành công mà không biết ngày tháng năm sinh cụ thể hay số CMND/CCCD.
4. **Giao thức OpenID for Verifiable Presentations (OID4VP)**:
   - Mini-app gửi yêu cầu `Presentation Definition` qua bridge JSAPI.
   - Container siêu ứng dụng hiển thị màn hình cấp quyền chuẩn cấp hệ thống, yêu cầu người dùng xác thực sinh trắc học (FaceID / Vân tay) trước khi ký phát hành Verifiable Presentation gửi lại cho mini-app.

---

## 4. Kế hoạch Triển khai & Kiểm thử cho Chuẩn Mini App Store

1. **Giai đoạn Ingestion & Store Validation**:
   - Thêm bộ kiểm tra định dạng gói: hỗ trợ song song chuẩn ZIP truyền thống và gói nhị phân Web Bundle (`.wbn`).
   - Kiểm tra chữ ký số nhà phát triển dựa trên DID và danh mục tin cậy (Trust Registry) trước khi duyệt đưa lên Store.
2. **Giai đoạn Runtime Security & Attestation**:
   - Kích hoạt RASP engine và kiểm tra Play Integrity / App Attest tại thời điểm khởi động container.
   - Cung cấp module JSAPI bridge `superapp.security.getAttestationAssertion()` và `superapp.id.presentCredential()`.
3. **Chỉ số Đánh giá (Metrics & SLI/SLO)**:
   - Thời gian khởi động lạnh mini-app từ Web Bundle giảm $\ge 40\%$ so với giải nén ZIP.
   - Tỷ lệ xác thực thành công MEETS_STRONG_INTEGRITY / App Attest $\ge 99.5\%$ trên các thiết bị đủ điều kiện.
   - Độ trễ phản hồi bridge xác thực định danh OID4VP $\le 800\text{ ms}$ (bao gồm cả bước xác thực sinh trắc học người dùng).
