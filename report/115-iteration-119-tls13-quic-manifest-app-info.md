# Báo Cáo Chuyên Đề: TLS 1.3 Handshake & Anti-Replay, QUIC Transport Architecture & W3C Manifest App Info Metadata Standard (Milestone 119)

## 1. Giới Thiệu & Bối Cảnh Nghiên Cứu
Trong kiến trúc Super App và Mini App Runtime quy mô lớn, tính nhất quán về bảo mật tầng truyền tải (transport security), khả năng phục hồi mạng di động đa kết nối (network resilience & mobility), và siêu dữ liệu định danh - phân loại chợ ứng dụng (store discovery & taxonomy) là ba trụ cột cốt lõi.

Milestone 119 hoàn thiện nghiên cứu chuẩn hóa kỹ thuật dựa trên 3 khối tiêu chuẩn quốc tế có tính chuẩn mực cao:
1. **IETF RFC 8446 (TLS 1.3 Protocol) & RFC 8470 / HTTP Early-Data**: Chuẩn hóa mật mã học tầng truyền tải hiện đại, triệt tiêu các bộ mã lỗi thời (RSA tĩnh, CBC ciphers, SHA-1), tối ưu thời gian thiết lập kết nối xuống 1-RTT, bảo đảm tính bí mật chuyển tiếp hoàn hảo (Perfect Forward Secrecy - PFS) và thiết lập cơ chế kiểm soát replay 0-RTT nghiêm ngặt đối với các endpoint nhạy cảm thông qua mã phản hồi `HTTP 425 (Too Early)`.
2. **IETF RFC 9000 (QUIC Transport), RFC 9001 (TLS-QUIC Security) & RFC 9002 (Loss Detection & Congestion Control)**: Kiến trúc giao vận UDP đa luồng độc lập, triệt tiêu hoàn toàn hiện tượng Head-of-Line (HoL) blocking giữa các tài nguyên mini-app, hỗ trợ di chuyển kết nối thông minh (Connection Migration) khi thiết bị chuyển đổi Wi-Fi <-> 4G/5G bằng Connection ID (CID), bảo vệ tiêu đề gói tin (Header Protection) và tối ưu thuật toán kiểm soát tắc nghẽn vô tuyến di động (Probe Timeout - PTO).
3. **W3C Web App Manifest - App Information & Marketplace Discovery Metadata (W3C manifest-app-info)**: Chuẩn hóa siêu dữ liệu ứng dụng phục vụ duyệt tìm, phân loại danh mục (categories taxonomy), mô tả tiếp cận (accessible descriptions cho screen readers), kiểm định ảnh chụp màn hình đáp ứng đa kích thước (responsive screenshots) và tích hợp chứng chỉ phân loại độ tuổi toàn cầu của Liên minh IARC (`iarc_rating_id`).

---

## 2. Danh Mục Các Findings Chuẩn Hóa Được Thu Thập Trong Vòng 119

| ID | Danh Mục | Tiêu Đề Chuẩn Hóa | Mức Bằng Chứng | URL Nguồn Chuẩn |
|---|---|---|---|---|
| `TLS-13-RFC8446-SECURITY-CORE` | Security & Trust | IETF RFC 8446 TLS 1.3 Cryptographic Handshake & Forward Secrecy Sandboxing | `official_standard` | [RFC 8446](https://www.rfc-editor.org/rfc/rfc8446) |
| `TLS-13-MDN-SANDBOX-PRACTICAL-GUIDE` | Security & Trust | MDN Transport Layer Security (TLS) Web Security & Container Configuration Standard | `official_documentation` | [MDN TLS Guide](https://developer.mozilla.org/en-US/docs/Web/Security/Transport_Layer_Security) |
| `TLS-13-EARLY-DATA-ANTI-REPLAY` | Security & Trust | HTTP Early-Data Header & 0-RTT Anti-Replay Mitigation Standard | `official_documentation` | [MDN Early-Data](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Early-Data) |
| `TLS-13-DATATRACKER-SPEC-LIFECYCLE` | Security & Trust | IETF Datatracker RFC 8446 Lifecycle & Cryptographic Extension Governance | `official_standard` | [IETF RFC 8446](https://datatracker.ietf.org/doc/html/rfc8446) |
| `TLS-13-RFC8470-EARLY-DATA-GOVERNANCE` | Security & Trust | IETF RFC 8470 Using Early Data in HTTP & 425 Too Early Response Code | `official_standard` | [RFC 8470](https://www.rfc-editor.org/rfc/rfc8470) |
| `QUIC-RFC9000-TRANSPORT-SANDBOXING` | Performance & Cacheability | IETF RFC 9000 QUIC A UDP-Based Multiplexed and Secure Transport Standard | `official_standard` | [RFC 9000](https://www.rfc-editor.org/rfc/rfc9000) |
| `QUIC-DATATRACKER-SPEC-LIFECYCLE` | Performance & Cacheability | IETF Datatracker RFC 9000 QUIC Specification & Protocol Governance | `official_standard` | [IETF RFC 9000](https://datatracker.ietf.org/doc/html/rfc9000) |
| `QUIC-RFC9001-SECURITY-TLS-MAPPING` | Security & Trust | IETF RFC 9001 Using TLS to Secure QUIC & Crypto Stream Key Phase Rotation | `official_standard` | [RFC 9001](https://www.rfc-editor.org/rfc/rfc9001) |
| `QUIC-RFC9002-LOSS-DETECTION-CONGESTION` | Performance & Cacheability | IETF RFC 9002 QUIC Loss Detection and Congestion Control Architecture | `official_standard` | [RFC 9002](https://www.rfc-editor.org/rfc/rfc9002) |
| `QUIC-MDN-PRACTICAL-TLS-DEPLOYMENT` | Security & Trust | MDN TLS Practical Implementation Guide & Modern Super-App Cipher Governance | `official_documentation` | [MDN TLS Implementation](https://developer.mozilla.org/en-US/docs/Web/Security/Practical_implementation_guides/TLS) |
| `MANIFEST-APP-INFO-W3C-SPEC` | Discovery & Metadata | W3C Web App Manifest - App Information & Marketplace Discovery Metadata Standard | `official_standard` | [W3C Manifest App Info](https://www.w3.org/TR/manifest-app-info/) |
| `MANIFEST-CATEGORIES-TAXONOMY-MDN` | Discovery & Metadata | MDN Web App Manifest categories Member & Marketplace Taxonomy Governance | `official_documentation` | [MDN Manifest categories](https://developer.mozilla.org/en-US/docs/Web/Manifest/categories) |
| `MANIFEST-DESCRIPTION-ACCESSIBILITY-MDN` | Discovery & Metadata | MDN Web App Manifest description Member & Localized Storefront Information | `official_documentation` | [MDN Manifest description](https://developer.mozilla.org/en-US/docs/Web/Manifest/description) |
| `MANIFEST-SCREENSHOTS-MEDIA-MDN` | Discovery & Metadata | MDN Web App Manifest screenshots Member & Responsive Visual Asset Verification | `official_documentation` | [MDN Manifest screenshots](https://developer.mozilla.org/en-US/docs/Web/Manifest/screenshots) |
| `MANIFEST-IARC-RATING-MDN` | Store Governance & Review | MDN Web App Manifest iarc_rating_id & Global Content Age Rating Federation | `official_documentation` | [MDN Manifest iarc_rating_id](https://developer.mozilla.org/en-US/docs/Web/Manifest/iarc_rating_id) |

---

## 3. Phân Tích Kỹ Thuật Chi Tiết

### 3.1. IETF RFC 8446 & RFC 8470: TLS 1.3 Handshake & Kiểm Soát Chống Replay 0-RTT
1. **Loại bỏ toàn bộ mật mã yếu**: TLS 1.3 loại bỏ tĩnh RSA (static RSA), CBC ciphers (dễ tổn thương trước Padding Oracle / POODLE), SHA-1 và RC4. Bắt buộc sử dụng Authenticated Encryption with Associated Data (AEAD) như AES-128-GCM, AES-256-GCM, hoặc ChaCha20-Poly1305.
2. **Bảo mật chuyển tiếp (Perfect Forward Secrecy - PFS)**: Mọi phiên kết nối buộc phải dùng trao đổi khóa Diffie-Hellman tạm thời (Ephemeral DHE/ECDHE), bảo đảm kẻ tấn công ghi nhận traffic không thể giải mã lại dữ liệu trong tương lai khi khóa dài hạn bị lộ.
3. **Hiểm họa 0-RTT Replay & Mã phản hồi HTTP 425**: Dữ liệu gửi trước trong gói 0-RTT (Early Data) có thể bị nghe lén và gửi lại bởi kẻ tấn công mạng. Do đó, API Gateway của Super App phải kiểm tra:
   - Các phương thức HTTP không an toàn (`POST`, `PUT`, `DELETE`, `PATCH`) hoặc các giao dịch tài chính nếu đến dưới dạng 0-RTT phải bị Gateway từ chối ngay lập tức bằng mã phản hồi `425 (Too Early)`.
   - Trình duyệt/WebView SDK tự động nhận mã 425 và phát lại yêu cầu một cách an toàn trên kết nối 1-RTT đã hoàn tất bắt tay mà không gây lỗi cho ứng dụng mini app.

### 3.2. IETF RFC 9000, 9001 & 9002: QUIC Transport & Di Chuyển Kết Nối Di Động
1. **Triệt tiêu Head-of-Line Blocking**: Trong TCP truyền thống qua HTTP/2, khi một gói tin bị mất, toàn bộ các luồng (streams) đều bị nghẽn lại cho đến khi gói tin đó được truyền lại. Với QUIC (RFC 9000) trên nền UDP, mỗi luồng truyền dữ liệu độc lập với bộ đệm luồng riêng biệt, việc mất gói tin ở một luồng tài nguyên ảnh không ảnh hưởng đến luồng RPC/API giao diện.
2. **Di chuyển kết nối (Connection Migration)**: Thiết bị di động thường xuyên chuyển mạng giữa Wi-Fi gia đình/văn phòng và mạng 4G/5G khi di chuyển. QUIC định danh kết nối qua Connection ID (CID) thay vì IP 4-tuple. Khi IP thay đổi, kết nối duy trì thông suốt không cần bắt tay lại TLS, duy trì phiên người dùng hoàn hảo.
3. **Mã hóa tiêu đề & Phát hiện mất gói**: RFC 9001 mã hóa cả số thứ tự gói tin trong tiêu đề QUIC (Header Protection), ngăn chặn các nhà mạng can thiệp hoặc lập hồ sơ người dùng. Thuật toán Probe Timeout (PTO) trong RFC 9002 giúp phát hiện sự cố rơi gói tức thì trên mạng vô tuyến với độ trễ tối thiểu.

### 3.3. W3C Manifest App Info: Chuẩn Hóa Khám Phá & Phân Loại Ứng Dụng
1. **categories taxonomy chuẩn hóa**: Tiêu chuẩn phân loại theo mảng chuỗi chữ thường chuẩn hóa (business, finance, utilities, lifestyle, shopping), chặn việc nhồi nhét từ khóa rác và tối ưu thuật toán gợi ý của Mini App Store.
2. **description phục vụ trợ năng (Accessibility)**: Trường mô tả chuẩn định dạng văn bản thuần túy được tích hợp trực tiếp với bộ đọc màn hình (Screen Readers) cho người khiếm thị, đồng thời được quét bảo mật tự động chống tấn công Stored XSS trong giao diện Storefront.
3. **screenshots đa kích thước**: Cấu trúc mảng ảnh có trường `form_factor` (`wide` cho tablet/desktop và `narrow` cho điện thoại) và `label` trợ năng, cho phép ứng dụng Store render xem trước mượt mà trước khi mở container.
4. **Chứng chỉ độ tuổi quốc tế IARC (`iarc_rating_id`)**: Tích hợp với hệ thống phân loại độ tuổi quốc tế IARC, tự động đối chiếu API để áp dụng chính sách kiểm soát phụ huynh (Parental Controls) và điều kiện phân phối theo luật pháp của từng quốc gia (ESRB, PEGI, USK, ACB).

---

## 4. Khuyến Nghị Áp Dụng Cho Super App Mini App Store

1. **Cấu hình Gateway mạng chuẩn hóa**:
   - Ép buộc giao thức TLS 1.3 và TLS 1.2 AEAD cho toàn bộ domain mini app egress.
   - Bật cờ `Early-Data` và xử lý mã `HTTP 425 (Too Early)` tự động tại WebView Bridge SDK cho các yêu cầu có nguy cơ rủi ro giao dịch.
   - Triển khai QUIC/HTTP-3 cho các endpoint tải tài nguyên tĩnh (static bundle zip, hình ảnh, âm thanh) nhằm tối ưu trải nghiệm mạng yếu và giảm thiểu gián đoạn khi chuyển mạng.
2. **Quy chuẩn Linting Manifest lúc Submission**:
   - Kiểm tra trường `categories` chỉ chứa các giá trị hợp lệ trong danh mục W3C đã công bố (tối đa 5 categories).
   - Kiểm tra định dạng `screenshots` bao gồm cả hình ảnh dạng dọc (`narrow`) với kích thước và tỷ lệ khung hình đạt chuẩn.
   - Bắt buộc khai báo `iarc_rating_id` đối với mọi mini app phục vụ thị trường tiêu dùng để kích hoạt tính năng kiểm soát phụ huynh tự động.
