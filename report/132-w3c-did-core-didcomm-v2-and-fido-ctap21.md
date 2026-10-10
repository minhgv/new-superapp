# Chuyên Đề 132: W3C DID Core 1.0, DIF DIDComm Messaging v2.0 & FIDO Alliance CTAP 2.1 Hardware Security

## 1. Bối Cảnh & Mục Tiêu Chuẩn Hóa
Trong kiến trúc Super App hiện đại, hạ tầng định danh, bảo mật phần cứng và giao tiếp liên ứng dụng (inter-app communication) đóng vai trò sống còn để chuyển đổi siêu ứng dụng từ mô hình tập trung đóng kín sang hệ sinh thái mở phi tập trung, an toàn tuyệt đối và bảo vệ quyền riêng tư người dùng.

Chuyên đề 132 hoàn thiện 15 phát hiện kỹ thuật cốt lõi (Findings `did_core_132_01` đến `ctap2_132_15`) thuộc 3 lĩnh vực nền tảng:
1. **W3C Decentralized Identifiers (DIDs) v1.0 & did:web Method Architecture**: Chuẩn cú pháp định danh phi tập trung độc lập với DNS/IDP trung ương, quan hệ xác minh tường minh (verification relationships) ngăn ngừa tấn công hoán đổi khóa, mô hình did:web neo vào hạ tầng Web HTTPS và kiến trúc phân giải `resolve()` / `dereference()`.
2. **DIF DIDComm Messaging v2.0 Protocol**: Khung giao vận thông điệp bảo mật đa chặng qua JSON Web Messages (JWM), các chế độ mã hóa xác thực (`authcrypt` vs `anoncrypt`), định tuyến đa bước (multi-hop Forward protocol) với mã hóa củ hành (onion encryption) bảo vệ quyền riêng tư vị trí/IP, và giao thức Out-of-Band (OOB) cho quét mã QR deep-linking tức thì.
3. **FIDO Alliance CTAP 2.1 Hardware Security**: Giao thức nhị phân CBOR giữa client platform và thiết bị bảo mật phần cứng (USB/NFC/BLE / Secure Enclave), cơ chế PIN/UV Auth Protocol v2 chống nghe lén, quản trị khóa thường trú (discoverable credentials / passkeys), phân vùng lưu trữ an toàn `authenticatorLargeBlobs`, và dẫn xuất khóa đối xứng mật mã qua tiện ích PRF (`hmac-secret`).

---

## 2. Chi Tiết Các Trụ Cột Kỹ Thuật

### 2.1. W3C Decentralized Identifiers (DIDs) v1.0 & did:web
- **Cú pháp định danh & Mô hình dữ liệu**: DID là URI chuẩn có dạng `did:<method-name>:<method-specific-id>`. Mỗi DID phân giải thành một DID Document định dạng JSON-LD/JSON chứa các khóa công khai, phương thức xác minh và endpoints dịch vụ.
- **Tách biệt Chủ thể (Subject) và Người kiểm soát (Controller)**: DID Subject là thực thể được đại diện bởi DID; DID Controller là thực thể có thẩm quyền phát hành cập nhật DID Document. Điều này cho phép áp dụng mô hình multisig hoặc ủy quyền quản trị doanh nghiệp cho các tài khoản nhà phát triển mini-app.
- **Quan hệ xác minh tường minh (Verification Relationships)**: Phân tách rõ ràng giữa 5 mục đích mật mã:
  - `authentication`: Chứng minh quyền kiểm soát danh tính trong các phiên đăng nhập.
  - `assertionMethod`: Ký phát hành Verifiable Credentials, ký manifest và gói mã nguồn mini-app.
  - `capabilityInvocation`: Ký ủy quyền thực thi hành động nhạy cảm thay mặt chủ thể.
  - `capabilityDelegation`: Ủy quyền cho bên thứ ba.
  - `keyAgreement`: Thực hiện bắt tay trao đổi khóa Diffie-Hellman (ECDH).
  *Nguyên tắc an ninh*: Khóa chỉ dùng cho `authentication` tuyệt đối không được chấp nhận khi ký manifest phát hành mini-app.
- **Phương pháp `did:web`**: Cho phép doanh nghiệp chuyển đổi tên miền HTTPS truyền thống thành DID mà không tốn phí blockchain (`did:web:developer.example.com` -> `https://developer.example.com/.well-known/did.json`). Đòi hỏi kết nối TLS 1.2+ (khuyến nghị TLS 1.3), kiểm tra DNSSEC và Certificate Transparency logs.
- **Phân giải DID (DID Resolution)**: Chuẩn hóa hàm trừu tượng `resolve(did)` trả về bộ ba `(didDocument, didResolutionMetadata, didDocumentMetadata)` và hàm `dereference(didUrl)` truy xuất trực tiếp các khóa mật mã con (`#key-1`).

### 2.2. DIF DIDComm Messaging v2.0 Protocol
- **Cấu trúc bao thư thông điệp (JWM)**: Đóng gói tin nhắn JSON Web Message với các trường chuẩn: `id`, `type`, `body`, `from`, `to`, `created_time`, `expires_time`.
- **Ba định dạng bao thư**:
  - `Plaintext`: Tin nhắn văn bản thuần (chỉ dùng nội bộ hoặc định tuyến phi bảo mật).
  - `Signed`: Đóng gói chữ ký số JWS bất khả chối bỏ với header `typ: application/didcomm-signed+json`.
  - `Encrypted`: Mã hóa đầu cuối authenticated encryption theo chuẩn JWE.
- **Mã hóa Authcrypt vs Anoncrypt**:
  - `authcrypt`: Sử dụng thuật toán ECDH-1PU (One-Pass Unified Diffie-Hellman) trên đường cong X25519 hoặc NIST P-256 kết hợp mã hóa khối đối xứng `A256GCM` hoặc `A256CBC-HS512`. Đảm bảo tính xác thực người gửi, bảo mật và chống chối bỏ.
  - `anoncrypt`: Khởi tạo khóa tạm (ephemeral key) để che giấu hoàn toàn danh tính người gửi đối với bên nghe lén.
- **Định tuyến đa chặng (Forward Protocol & Onion Encryption)**: Tin nhắn được bọc lồng qua nhiều lớp mã hóa hướng tới các mediator chuyển tiếp. Mỗi trạm trung gian chỉ có thể giải mã lớp ngoài cùng để đọc trường `next`, hoàn toàn không thể xem nội dung bên trong hoặc đích đến cuối cùng, triệt tiêu nguy cơ thu thập IP/vị trí người dùng mobile.
- **Giao thức Out-of-Band (OOB)**: Đóng gói lời mời kết nối thành mã QR hoặc URL deep-link (`https://superapp.example/oob?_oob=...`), cho phép người dùng quét mã tại quầy bán lẻ hoặc đối tác để khởi tạo phiên giao tiếp DIDComm bảo mật ngay lập tức.

### 2.3. FIDO Alliance CTAP 2.1 Hardware Security
- **Kiến trúc giao thức nhị phân CBOR**: Giao thức kết nối giữa Host OS của Super App và Authenticator phần cứng qua USB HID, NFC ISO 7816 hoặc BLE GATT. Gói tin được nén CBOR (RFC 8949) siêu nhẹ, phù hợp chip vi điều khiển bảo mật cao.
- **PIN/UV Auth Protocol v2**: Bắt tay trao đổi khóa tạm ECDH P-256 (`sharedSecret`), bảo vệ mã PIN hoặc dữ liệu sinh trắc học người dùng khỏi nguy cơ bị đọc trộm trên bus vật lý hoặc bởi mã độc trong WebView. Trả về token `pinUvAuthToken` có phạm vi quyền hạn và thời hạn nghiêm ngặt.
- **Credential Management API (`authenticatorCredentialManagement`)**: Cho phép giao diện quản trị Super App liệt kê, kiểm tra và thu hồi các discoverable credentials (passkeys thường trú) gắn với từng mini-app theo mã băm `rpIdHash`.
- **Lưu trữ dữ liệu lớn (`authenticatorLargeBlobs`)**: Phân vùng bộ nhớ an toàn trên khóa bảo mật phần cứng để lưu trữ chứng chỉ, khóa phục hồi DID hoặc phiên làm việc ngoại tuyến, tồn tại bền bỉ ngay cả khi thiết bị di động bị cài lại hệ điều hành.
- **Dẫn xuất khóa đối xứng mật mã (PRF / `hmac-secret`)**: Cho phép trích xuất các khóa bí mật dẫn xuất bằng HMAC-SHA-256 với muối (`salt1`, `salt2`) mà không bao giờ làm lộ khóa chủ (master seed) của phần cứng. Đây là cơ sở cốt lõi để Super App mã hóa cơ sở dữ liệu ngoại tuyến (OPFS/IndexedDB) theo mô hình zero-knowledge client-side encryption.

---

## 3. Khuyến Nghị Áp Dụng Chuẩn Mini App Store

1. **Chuẩn hóa danh tính nhà phát triển bằng W3C DID Core & did:web**:
   - Tích hợp resolver `did:web` vào quy trình ingest và phê duyệt ứng dụng.
   - Bắt buộc khóa ký manifest phải thuộc quan hệ `assertionMethod` trong DID Document của nhà phát triển.
2. **Cầu nối giao tiếp liên mini-app bằng DIF DIDComm v2.0**:
   - Cung cấp native bridge `superapp.didcomm.encrypt` / `decrypt` trong container, giữ khóa bí mật hoàn toàn trong native boundary.
   - Sử dụng mô hình trung gian chuyển tiếp (Mediator Relay) hỗ trợ hàng đợi tin nhắn ngoại tuyến khi thiết bị ngắt kết nối mạng.
3. **Bảo mật giao dịch tài chính bằng FIDO CTAP 2.1 & WebAuthn PRF**:
   - Yêu cầu xác thực phần cứng FIDO2 CTAP 2.1 với PIN/UV Protocol v2 cho các giao dịch tài chính hoặc hành động ký hợp đồng điện tử trong mini-app.
   - Tận dụng WebAuthn PRF extension để cấp phát khóa mã hóa cục bộ cho các mini-app lưu trữ dữ liệu nhạy cảm ngoại tuyến.
