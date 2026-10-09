# Chuyên Đề 114: Messaging Layer Security (RFC 9420), Service Identity in TLS (RFC 9525) & WHATWG DOM EventTarget Sandboxing Architecture

## 1. Tổng Quan & Bối Cảnh Nghiên Cứu

Trong Milestone 118, nghiên cứu chuẩn hóa Mini App Store trên Super App tiếp tục mở rộng vào 3 trụ cột kỹ thuật trọng yếu của nền tảng web hiện đại và hạ tầng bảo mật cấp ứng dụng:
1. **IETF RFC 9420 Messaging Layer Security (MLS) Protocol**: Chuẩn bảo mật tin nhắn nhóm đầu-cuối (End-to-End Encryption - E2EE) ở tầng ứng dụng, ứng dụng thuật toán thỏa thuận khóa cây bất đồng bộ TreeKEM với độ phức tạp $O(\log N)$, hỗ trợ Forward Secrecy (FS) và Post-Compromise Security (PCS), phân lập dịch vụ phân phối tin cậy zero-trust (Delivery Service) và cơ chế phân phối gói khóa (KeyPackage).
2. **IETF RFC 9525 Service Identity in TLS & PKI Matching**: Chuẩn hóa quy tắc đối sánh định danh dịch vụ trong chứng chỉ số X.509v3 cho TLS (thay thế và phế truất RFC 6125), bắt buộc kiểm tra trường Subject Alternative Name (SAN - dNSName, URI, IP), loại bỏ triệt để việc fallback về Common Name (CN), thắt chặt quy tắc wildcard đa nhãn và xử lý tên miền quốc tế (IDNA), bảo vệ cầu nối mạng container và gateway super-app chống lại tấn công subdomain takeover.
3. **WHATWG DOM EventTarget & Event Flow Sandboxing**: Kiến trúc điều phối sự kiện DOM, chuẩn hóa 3 pha lan truyền sự kiện (Capturing, Target, Bubbling), tối ưu hóa hiệu năng render/cuộn trang qua cờ `passive: true` (AddEventListenerOptions), quản lý rò rỉ bộ nhớ WebView qua `AbortSignal`, ranh giới an ninh `Event.isTrusted` chống clickjacking và gian lận thanh toán tự động, kiểm soát lan truyền sự kiện (`stopPropagation`, `preventDefault`), và bus sự kiện tùy biến `CustomEvent` xuyên ranh giới Shadow DOM.

---

## 2. Chi Tiết Các Chuẩn Kỹ Thuật & Kiến Trúc Kiểm Soát

### 2.1. IETF RFC 9420: Messaging Layer Security (MLS) Protocol

| Thành Phần / Cơ Chế | Nội Dung Chuẩn (IETF RFC 9420 / Datatracker) | Kiểm Soát Kỹ Thuật Trên Super App |
|---|---|---|
| **Thuật toán TreeKEM & Độ Phức Tạp Khóa** | TreeKEM tổ chức các thành viên nhóm thành các nút lá trên cây nhị phân (ratchet tree). Việc thêm/xóa thành viên hoặc cập nhật khóa chỉ yêu cầu cập nhật đường dẫn từ lá lên gốc ($O(\log N)$) thay vì $O(N)$ như các giao thức bắt cặp truyền thống. | Cho phép super-app triển khai các mini-app cộng tác đa bên (hội chẩn y tế từ xa, phê duyệt tín dụng doanh nghiệp, sàn giao dịch B2B) với hàng ngàn người tham gia mà không làm bùng nổ băng thông mạng di động. |
| **Forward Secrecy & Post-Compromise Security** | Mỗi thao tác thay đổi nhóm tạo ra thông điệp Commit đưa nhóm sang một Epoch mới. Bí mật Epoch cũ bị xóa vĩnh viễn (Forward Secrecy). Entropy mới từ thành viên chưa bị xâm phạm phục hồi tính bảo mật của nhóm (Post-Compromise Security). | Ngay cả khi thiết bị di động của một người dùng bị phân tích RAM hoặc trích xuất dữ liệu sau đó, toàn bộ lịch sử tin nhắn trước đó của mini-app vẫn được bảo mật; nếu mã độc bị gỡ bỏ, phiên làm việc sẽ tự động lành ở epoch kế tiếp. |
| **Zero-Trust Delivery Service (DS)** | Kiến trúc MLS tách biệt Delivery Service (DS) chịu trách nhiệm định tuyến thông điệp khỏi Authentication Service (AS) và các điểm cuối mật mã. DS không thể giải mã dữ liệu ứng dụng hay sửa đổi thành viên mà không bị phát hiện. | Hạ tầng cloud của super-app đóng vai trò là DS phi tin cậy (untrusted DS). Dù super-app gateway bị tấn công hay rò rỉ cơ sở dữ liệu, toàn bộ nội dung trao đổi của các mini-app ngân hàng/y tế vẫn duy trì bảo mật đầu-cuối. |
| **KeyPackage Catalog Distribution** | KeyPackage là gói dữ liệu tự chứa có chữ ký số chứa khóa công khai, ciphersuite hỗ trợ và chứng chỉ định danh của client, cho phép thêm thành viên vào nhóm bất đồng bộ (ngay cả khi đối phương offline). | Kho ứng dụng mini-app tích hợp phân phối KeyPackage đã xác thực: container tự động lấy KeyPackage của mini-app đối tác đã ký bởi nhà phát triển đã xác minh danh tính, thiết lập kênh E2EE tức thì mà không cần trao đổi khóa thủ công. |
| **Quản Lý Trạng Thái Container Mobile** | Chuẩn RFC 9420 quy định chu trình xử lý Welcome, Commit và Proposal. Trạng thái cây nhị phân và bí mật epoch cần được duy trì nhất quán. | Container lưu trữ trạng thái MLS trong vùng bộ nhớ bảo mật native (Android Keystore / Apple Keychain và SQLite/OPFS mã hóa), cung cấp bridge API cấp cao cho mini-app JavaScript, ngăn ngừa lộ khóa trong context WebView. |

### 2.2. IETF RFC 9525: Service Identity in TLS & Public Key Infrastructure

| Quy Tắc / Kiểm Soát | Chuẩn Quy Định (IETF RFC 9525 / CAB Forum) | Ứng Dụng Trên Super App Mini App Store |
|---|---|---|
| **Bắt Buộc Kiểm Tra SAN & Phế Truất CN** | Khách hàng TLS bắt buộc phải đối sánh định danh tham chiếu với Subject Alternative Name (SAN: dNSName, URI, iPAddress). Nghiêm cấm fallback về trường Common Name (CN) nếu SAN vắng mặt. | Gateway super-app và container engine từ chối kết nối tới các backend mini-app sử dụng chứng chỉ cũ chỉ có trường CN. Công cụ kiểm duyệt tự động quét và cảnh báo nhà phát triển cập nhật chứng chỉ SAN hợp lệ. |
| **Giới Hạn Ký Tự Wildcard (*)** | Dấu sao (*) chỉ được phép xuất hiện là toàn bộ nhãn đầu tiên bên trái (ví dụ: `*.example.com`). Cấm wildcard chứa ký tự khác (`*foo.example.com`) hoặc nằm ở giữa. Wildcard KHÔNG BAO GIỜ được khớp qua nhiều nhãn phân cách (`*.example.com` không khớp `a.b.example.com`). | Pipeline kiểm duyệt manifest ngăn chặn việc khai báo whitelist tên miền dạng wildcard lỏng lẻo (`*.mycompany.com`), đảm bảo mini-app không vô tình cho phép kết nối tới các subdomain chưa được kiểm định an ninh. |
| **Xử Lý Tên Miền Quốc Tế (IDNA)** | Khi đối sánh các tên miền quốc tế chứa ký tự phi Latin (IDN), việc so sánh bắt buộc phải thực hiện trên định dạng A-label (punycode, tiền tố `xn--`). | Hệ thống container chuẩn hóa URL trước khi kiểm tra whitelist, ngăn chặn tấn công giả mạo thị giác (homograph/punycode spoofing) trong các mini-app thương mại điện tử đa quốc gia. |
| **Tuân Thủ CA/Browser Forum Baseline Requirements** | Các chứng chỉ TLS công khai phải tuân thủ Baseline Requirements về phương thức xác minh quyền kiểm soát tên miền (DCV) và duy trì dịch vụ kiểm tra thu hồi chứng chỉ (OCSP/CRL). | Nền tảng super-app tích hợp kiểm tra OCSP stapling tự động đối với các endpoint backend của đối tác, lập tức thu hồi quyền truy cập mạng nếu chứng chỉ của đối tác bị xâm phạm hoặc bị CA thu hồi. |
| **Chống Tấn Công Subdomain Takeover** | Đối sánh định danh tham chiếu nghiêm ngặt theo RFC 9525 ngăn ngừa kịch bản máy chủ đích phản hồi chứng chỉ của một tên miền khác khi bản ghi DNS CNAME bị trỏ nhầm về tài nguyên đám mây hết hạn. | Container bridge proxy kiểm tra tính hợp lệ của TLS handshake đối với mọi request từ mini-app, chặn đứng việc gửi token nhạy cảm hoặc dữ liệu định danh người dùng tới các endpoint bị chiếm quyền kiểm soát. |

### 2.3. WHATWG DOM EventTarget, Dispatching & Event Flow Sandboxing

| Giao Diện / Thuộc Tính | Đặc Tả Chuẩn (WHATWG DOM Living Standard) | Thiết Kế Kiểm Soát Container & Store Review |
|---|---|---|
| **EventTarget & Chu Trình Dispatch 3 Pha** | Quá trình điều phối sự kiện trải qua 3 pha: Capturing (từ Window xuống target), Target (tại phần tử đích), và Bubbling (từ target ngược lên Window). | Vỏ bọc container đăng ký capturing listener ở cấp độ root để đón bắt các cử chỉ điều hướng hoặc vi phạm an ninh, đảm bảo mini-app con không thể vô hiệu hóa nút back phần cứng hoặc thanh điều khiển host. |
| **AddEventListenerOptions: Passive & Signal** | Cờ `passive: true` cam kết listener không gọi `preventDefault()`, giải phóng luồng compositor cuộn trang mượt mà. Tham số `signal` liên kết listener với `AbortSignal`. | Tiêu chuẩn store review đánh giá hiệu năng bắt buộc các sự kiện cuộn/chạm (`touchmove`, `wheel`) phải dùng `passive: true` (đạt INP $\le 200$ms). Khi unmount trang mini-app, truyền `signal` dọn dẹp sạch sẽ listener, triệt tiêu memory leak. |
| **Event.isTrusted - Ranh Giới An Ninh Phần Cứng** | `Event.isTrusted` trả về `true` khi sự kiện xuất phát từ thao tác vật lý thực của người dùng (chạm màn hình, bấm phím) và `false` khi sinh ra qua script (`dispatchEvent()`, `click()`). | Bắt buộc đối với các luồng nhạy cảm: nút bấm Xác nhận Thanh toán, Cấp quyền riêng tư, và Ký xác thực sinh trắc học BẮT BUỘC kiểm tra `event.isTrusted === true`. Chặn đứng 100% tấn công clickjacking hoặc mã độc tự động kích hoạt giao dịch. |
| **Kiểm Soát Lan Truyền Sự Kiện** | `stopPropagation()` ngăn sự kiện lan tiếp; `stopImmediatePropagation()` ngăn các listener khác trên cùng phần tử; `preventDefault()` hủy bỏ hành vi mặc định của nền tảng. | Bộ quy tắc store review cấm mini-app lạm dụng `stopPropagation()` trên các phần tử gốc làm hỏng cơ chế pull-to-refresh của super-app hoặc ngắt luồng telemetry thu thập lỗi của hệ thống. |
| **CustomEvent & Bus Sự Kiện Đóng Gói** | `CustomEvent` cho phép truyền tải payload tùy biến trong thuộc tính `detail`, có thể xuyên qua Shadow DOM khi `composed: true`. | Container sử dụng `CustomEvent` để phát thông điệp vòng đời chuẩn hóa (`miniAppResume`, `networkChange`) tới mini-app. Dữ liệu trong `detail` được structured-clone để chống ô nhiễm prototype (prototype pollution). |

---

## 3. Bảng Kiểm Kê 15 Bằng Chứng Nghiên Cứu Mới (Iteration 118)

| ID Bằng Chứng | Tiêu Đề Bằng Chứng & Phân Loại | Mức Bằng Chứng | URL Xác Thực (HTTP 200) |
|---|---|---|---|
| `RFC-9420-MLS-PROTOCOL-CORE-SPEC` | IETF RFC 9420: Messaging Layer Security (MLS) Protocol Core Architecture & TreeKEM | `official_standard` | `https://www.rfc-editor.org/rfc/rfc9420.html` |
| `RFC-9420-MLS-FORWARD-SECRECY-POST-COMPROMISE` | IETF RFC 9420 (Datatracker): Forward Secrecy, Post-Compromise Security & Epoch Key Evolution | `official_standard` | `https://datatracker.ietf.org/doc/html/rfc9420` |
| `MLS-ROCKS-PROTOCOL-ARCHITECTURE-OVERVIEW` | MLS Architecture & Threat Model: Zero-Trust Delivery Service Isolation & Metadata Minimization | `industry_benchmark` | `https://messaginglayersecurity.rocks/` |
| `IETF-MLS-WORKING-GROUP-STANDARDIZATION-CHARTER` | IETF MLS Working Group: Cross-Platform Interoperability, Key Packages & Extension Governance | `official_standard` | `https://datatracker.ietf.org/wg/mls/about/` |
| `RFC-9420-WELCOME-COMMIT-LIFECYCLE-CONTAINER` | IETF RFC 9420: Group Lifecycle State Machine (Welcome, Commit, Proposal) & Mobile Container Memory Governance | `official_standard` | `https://www.rfc-editor.org/rfc/rfc9420` |
| `RFC-9525-TLS-SERVICE-IDENTITY-SPEC` | IETF RFC 9525: Service Identity in TLS & Public Key Infrastructure / Modern X.509 Certificate Matching | `official_standard` | `https://www.rfc-editor.org/rfc/rfc9525.html` |
| `RFC-9525-WILDCARD-AND-IDNA-MATCHING-RULES` | IETF RFC 9525 (Datatracker): Wildcard Domain Restrictions, IDNA Internationalization & Subdomain Boundary Auditing | `official_standard` | `https://datatracker.ietf.org/doc/html/rfc9525` |
| `RFC-6125-HISTORICAL-COMPARISON-DEPRECATION` | IETF RFC 6125 (Historical Evolution): Representation & Verification of Domain-Based Application Service Identity | `official_standard` | `https://www.rfc-editor.org/rfc/rfc6125.html` |
| `CAB-FORUM-BASELINE-REQUIREMENTS-TLS-IDENTITY` | CA/Browser Forum Baseline Requirements: SAN Mandatory Inclusion, Certificate Profile & Revocation Standards | `industry_benchmark` | `https://cabforum.org/baseline-requirements-documents/` |
| `RFC-9525-MULTI-TENANT-CONTAINER-ALLOWLIST-SECURITY` | IETF RFC 9525: Multi-Tenant Host Bridge Network Egress & Subdomain Takeover Defense | `official_standard` | `https://www.rfc-editor.org/rfc/rfc9525` |
| `DOM-EVENTTARGET-INTERFACE-DISPATCH-MODEL` | WHATWG DOM Standard: EventTarget Interface Architecture, Event Phases & Dispatch Algorithm | `official_standard` | `https://dom.spec.whatwg.org/#interface-eventtarget` |
| `DOM-ADDEVENTLISTENER-OPTIONS-PASSIVE-CLEANUP` | WHATWG EventTarget.addEventListener: Passive Listeners, Once Execution & Signal Lifecycle Binding | `official_standard` | `https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener` |
| `DOM-EVENT-ISTRUSTED-SECURITY-BOUNDARY` | WHATWG Event.isTrusted: Synthetic Event Discrimination & Anti-Clickjacking Financial Gatekeeping | `official_standard` | `https://developer.mozilla.org/en-US/docs/Web/API/Event/isTrusted` |
| `DOM-EVENT-STOPPROPAGATION-CONTAINMENT` | WHATWG DOM Event Propagation Controls: stopPropagation, stopImmediatePropagation & preventDefault Hygiene | `official_standard` | `https://developer.mozilla.org/en-US/docs/Web/API/Event/stopPropagation` |
| `DOM-CUSTOMEVENT-SANDBOXED-MESSAGE-BUS` | WHATWG CustomEvent Interface: Structured Detail Payload Serialization & Composed Boundary Traversal | `official_standard` | `https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent/CustomEvent` |

---

## 4. Tác Động Vào Tiêu Chuẩn Mini App Store & Kiến Trúc Vận Hành

1. **Khuyến Nghị Cấp Phép Mini App B2B / Fintech E2EE (RFC 9420)**:
   - Các mini-app xử lý giao dịch tài chính nhóm hoặc hội thoại nhạy cảm được yêu cầu tích hợp MLS KeyPackage thông qua Store Registry.
   - Host bridge cung cấp native runtime cho TreeKEM, bảo vệ khóa mật mã trong phần cứng chuyên dụng (Secure Enclave / StrongBox Keystore).
2. **Cổng Kiểm Duyệt Tự Động Định Danh TLS (RFC 9525)**:
   - Trình quét tự động của Store kiểm tra toàn bộ URL backend khai báo trong manifest: yêu cầu chứng chỉ số hỗ trợ SAN dNSName hợp lệ theo RFC 9525, loại bỏ chứng chỉ chỉ có CN hoặc wildcard sai quy cách.
3. **Bộ Quy Tắc Store Review Về Sự Kiện Giao Diện & Bảo Mật (WHATWG DOM)**:
   - Nghiêm cấm mô phỏng sự kiện nhân tạo (`isTrusted === false`) để tự động vượt qua các màn hình xác thực hoặc thanh toán.
   - Yêu cầu kỹ thuật gắn `passive: true` cho các touch event và liên kết `AbortSignal` khi đăng ký listener để ngăn ngừa rò rỉ RAM và giật lag khung hình trên thiết bị di động.
