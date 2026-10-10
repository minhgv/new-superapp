# Chuyên đề 137: C2PA v2.1 Content Provenance, EUDI Wallet ARF (eIDAS 2.0) & IETF SCITT Transparency Ledgers

## 1. Bối cảnh & Tầm quan trọng Kiến trúc Super-App

Khi hệ sinh thái Super-App mở rộng thành trung tâm kinh tế số tích hợp đa tiện ích (thanh toán, eKYC, truyền thông đa phương tiện, phát hành mini-app phi tập trung), ba thách thức an ninh và pháp lý cốt lõi nổi lên:
1. **Gian lận danh tính & Thao túng nội dung số (Deepfakes / Synthetic AIGC)**: Sự bùng nổ của AI tạo sinh dẫn đến các cuộc tấn công giả mạo tài liệu eKYC, video call giả mạo để vượt qua sinh trắc học ngân hàng, và nội dung xuyên tạc trên mạng xã hội. Cần một tiêu chuẩn mở, bất biến để xác thực nguồn gốc và lịch sử chỉnh sửa của tệp đa phương tiện từ cảm biến phần cứng đến giao diện người dùng.
2. **Tuân thủ khung định danh xuyên biên giới & eIDAS 2.0 (Regulation EU 2024/1183)**: Quy định châu Âu yêu cầu các nền tảng trực tuyến rất lớn (VLOPs) và các ứng dụng cung cấp dịch vụ điều tiết bắt buộc phải chấp nhận Ví định danh kỹ thuật số châu Âu (EUDI Wallet). Tiêu chuẩn này thiết lập quyền riêng tư toán học (Unobservability, Issuer Blindness, Selective Disclosure) nhằm ngăn chặn việc theo dõi hành vi công dân qua các phiên đăng nhập.
3. **Toàn vẹn chuỗi cung ứng phần mềm & Sổ cái minh bạch (IETF SCITT)**: Các cuộc tấn công chuỗi cung ứng (chèn mã độc vào mini-app bundle, chiếm quyền tài khoản nhà phát triển, tấn công chèn phụ thuộc) đòi hỏi một sổ cái minh bạch không thể sửa đổi (Transparency Ledger) sử dụng chữ ký COSE và bằng chứng Merkle để chứng minh tính hợp pháp của từng gói phát hành trước khi container client cho phép nạp mã.

Milestone 137 bổ sung 15 chuẩn kỹ thuật chuẩn hóa quốc tế, nâng tổng số findings chuẩn hóa của dự án lên **2,006 findings**.

---

## 2. Chi tiết 15 Chuẩn Kỹ thuật & Bằng chứng Chuẩn hóa

### Nhóm 1: C2PA Technical Specification v2.1 & Digital Media Provenance (c2pa_137_01 -> c2pa_137_05)

1. **c2pa_137_01 - C2PA Technical Specification v2.1: JUMBF Manifest Store Encapsulation & Provenance Tree Architecture**
   - **Tiêu chuẩn**: C2PA Technical Specification v2.1 Section 6, 7 & 11; ISO/IEC 19566-5 (JUMBF).
   - **Cơ chế kỹ thuật**: Đóng gói siêu dữ liệu xuất xứ vào các hộp JUMBF (`c2pa` box) lồng trong định dạng tệp chuẩn (JPEG APP11, MP4 moov/c2pa, PNG eXIf). Thiết lập cấu trúc cây xuất xứ phân cấp: Active Manifest đại diện cho trạng thái hiện tại, Ingredient Manifests liên kết các tệp nguồn upstream. Hỗ trợ manifest nhúng trực tiếp hoặc manifest từ xa tham chiếu qua HTTP URI.
   - **Áp dụng Super-App**: Tích hợp bộ phân tích JUMBF vào đường ống xử lý ảnh/video trên client container. Bắt buộc mini-app tạo sinh AI phải nhúng manifest C2PA v2.1 khai báo nguồn gốc mô hình và tham số prompt.

2. **c2pa_137_02 - C2PA Claims, Assertions Taxonomy & Provenance Graph: Actions, Hash-Data, Exif & AIGC Metadata**
   - **Tiêu chuẩn**: C2PA Technical Specification v2.1 Section 10 & 6.
   - **Cơ chế kỹ thuật**: Chuẩn hóa từ điển hành động provenance (`c2pa.actions`: `c2pa.created`, `c2pa.edited`, `c2pa.cropped`, `c2pa.ai_generated`). Đặc tả thông tin tác nhân AI (tên mô hình, phiên bản, hash prompt) để minh bạch hóa nội dung do máy tạo. Biểu diễn đồ thị có hướng không chu trình (DAG) liên kết các nguyên liệu đầu vào (`c2pa.ingredient`).
   - **Áp dụng Super-App**: Hiển thị huy hiệu xác thực trực quan (C2PA content credentials badge) trên bảng tin super-app. Cho phép mini-app đọc thông tin xuất xứ qua API cầu nối an toàn `superapp.provenance.getManifest(mediaUri)`.

3. **c2pa_137_03 - C2PA Cryptographic Signature Standards: COSE_Sign1, X.509 Certificate Chains & RFC 3161 Timestamping**
   - **Tiêu chuẩn**: C2PA Technical Specification v2.1 Section 13, 14 & 15; IETF RFC 9052; IETF RFC 3161; IETF RFC 6960.
   - **Cơ chế kỹ thuật**: Ký số bằng cấu trúc `COSE_Sign1` sử dụng Ed25519 hoặc ECDSA (P-256/P-384). Neo giữ niềm tin qua chuỗi chứng chỉ X.509 (`x5chain`) liên kết đến danh sách tin cậy C2PA Trust List. Bắt buộc gắn thẻ thời gian mật mã RFC 3161 (Time-Stamp Token - TST) từ TSA độc lập để chứng minh chữ ký hợp lệ trước khi chứng thư hết hạn. Kiểm tra thu hồi chứng chỉ qua OCSP hoặc CRL.
   - **Áp dụng Super-App**: Cổng kiểm soát (Gateway) super-app tự động cập nhật C2PA Trust List và xác thực chữ ký COSE cùng tem thời gian RFC 3161, từ chối các tệp sử dụng thuật toán băm yếu hoặc chứng chỉ bị thu hồi.

4. **c2pa_137_04 - C2PA Content Binding Mechanics: Hard Byte-Range Hashing, Soft Watermarking & Assertion Redaction Governance**
   - **Tiêu chuẩn**: C2PA Technical Specification v2.1 Section 9 & Section 6.8.
   - **Cơ chế kỹ thuật**: Hard Binding tính toán băm SHA-256/SHA-384 trên các dải byte chính xác của nội dung media (loại trừ hộp manifest). Soft Binding hỗ trợ các kênh phân phối nén làm mất dữ liệu thông qua băm tri giác (perceptual hashing) hoặc thủy vân số. Cơ chế Redaction cho phép lược bỏ thông tin nhạy cảm (tọa độ GPS, dữ liệu khuôn mặt) bằng cách thay thế assertion bằng hộp `c2pa.redacted` có muối digest mà không làm mất tính toàn vẹn của chữ ký gốc.
   - **Áp dụng Super-App**: Tự động che chắn (redact) tọa độ GPS của người dùng trước khi hiển thị công khai trên dòng thời gian, bảo vệ quyền riêng tư cá nhân theo Luật Bảo vệ Dữ liệu Cá nhân.

5. **c2pa_137_05 - Super-App C2PA Runtime Integration: Financial eKYC Anti-Spoofing, Camera Binding & Store Integrity Gating**
   - **Tiêu chuẩn**: C2PA Technical Specification v2.1 Section 16 & 17.
   - **Cơ chế kỹ thuật**: Neo giữ luồng chụp camera phần cứng trực tiếp vào vùng đệm an toàn của OS (Secure Enclave / StrongBox), tạo manifest C2PA được ký bởi khóa phần cứng của thiết bị trước khi chuyển sang WebView của mini-app. Ngăn chặn triệt để tấn công chèn hình ảnh giả lập (virtual camera injection), giả mạo giấy tờ tùy thân trong luồng onboarding tài chính.
   - **Áp dụng Super-App**: Bắt buộc các mini-app tài chính/ngân hàng/cho thuê tài chính chỉ chấp nhận ảnh chụp mang chữ ký C2PA phần cứng từ API `superapp.camera.takeVerifiedPhoto()`, cấm tải ảnh từ thư viện mở cho các bước sinh trắc học nhạy cảm.

---

### Nhóm 2: European Digital Identity (EUDI) Wallet ARF & eIDAS 2.0 (eudi_137_06 -> eudi_137_10)

6. **eudi_137_06 - Regulation (EU) 2024/1183 (eIDAS 2.0): Legal Framework, European Digital Identity Wallet & Mandatory VLOP Acceptance**
   - **Tiêu chuẩn**: Quy định (EU) 2024/1183 sửa đổi Quy định (EU) 910/2014; Chiến lược Kỹ thuật số Ủy ban Châu Âu.
   - **Cơ chế kỹ thuật**: Thiết lập khuôn khổ pháp lý ràng buộc yêu cầu tất cả 27 quốc gia thành viên cung cấp Ví định danh kỹ thuật số (EUDI Wallet) cho công dân trước năm 2026. Bắt buộc các nền tảng trực tuyến rất lớn (VLOPs) và các đơn vị cung cấp dịch vụ điều tiết bắt buộc phải chấp nhận Ví EUDI. Chữ ký điện tử đủ điều kiện (QES) tạo từ ví có giá trị pháp lý tương đương chữ ký tay. Nghiêm cấm nhà phát hành và vận hành ví theo dõi giao dịch của người dùng.
   - **Áp dụng Super-App**: Định hình kiến trúc Super-App sẵn sàng đóng vai trò Bên phụ thuộc (Relying Party), hỗ trợ đăng nhập và ký kết hợp đồng pháp lý số thông qua EUDI Wallet trên thị trường quốc tế.

7. **eudi_137_07 - EUDI Wallet Architecture & Reference Framework (ARF): Four-Party Trust Model & Common Secure Cryptographic Applications**
   - **Tiêu chuẩn**: EUDI Wallet ARF Section 2 & 3.
   - **Cơ chế kỹ thuật**: Mô hình tin cậy 4 bên phi tập trung: User/Holder, Issuer, Relying Party, Wallet Provider. Đặc tả ứng dụng mật mã an toàn chung (CSCA) và thiết bị tạo chữ ký đủ điều kiện (QSCD) yêu cầu khóa riêng phải được bảo vệ trong phần cứng đạt chứng chỉ Common Criteria EAL4+ (hoặc giải pháp HSM đám mây được phê duyệt). Chuẩn hóa luồng tương tác gần (NFC/BLE/QR) và luồng tương tác từ xa (HTTPS REST).
   - **Áp dụng Super-App**: Đồng bộ hóa kiến trúc ví của super-app với mô hình EUDI ARF, bảo vệ khóa định danh người dùng bằng Android StrongBox và iOS Secure Enclave.

8. **eudi_137_08 - EUDI ARF Person Identification Data (PID) & QEAA: Data Models, OpenID Protocols & Status Lifecycle**
   - **Tiêu chuẩn**: EUDI Wallet ARF Section 4 & 5; OID4VCI; OID4VP; ISO/IEC 18013-5 mDL; IETF SD-JWT VC.
   - **Cơ chế kỹ thuật**: Quản lý hai loại chứng chỉ cốt lõi: Dữ liệu nhận dạng cá nhân (PID) và Chứng nhận thuộc tính điện tử đủ điều kiện (QEAA - bằng cấp, giấy phép lái xe, chứng chỉ nghề nghiệp). Hỗ trợ song song cấu trúc nhị phân ISO 18013-5 mDL (CBOR/COSE) và định dạng web IETF SD-JWT VC. Phát hành qua OID4VCI với Proof of Possession (c_nonce); xuất trình qua OID4VP với ràng buộc phiên mật mã `SessionTranscript`. Kiểm tra thu hồi qua W3C Bitstring Status List hoặc IETF Status List.
   - **Áp dụng Super-App**: Tích hợp module OID4VP vào container super-app, cho phép mini-app yêu cầu các thuộc tính xác thực cụ thể (ví dụ: bằng lái xe hợp lệ khi thuê xe) một cách an toàn và tức thì.

9. **eudi_137_09 - EUDI ARF Privacy Engineering: Unobservability, Selective Disclosure & Anti-Correlation Protections**
   - **Tiêu chuẩn**: EUDI Wallet ARF Section 6 & Annex C.
   - **Cơ chế kỹ thuật**: Tính chất không thể quan sát (Unobservability): Nhà phát hành chứng chỉ không nhận được thông báo khi người dùng xuất trình chứng chỉ cho bên thứ ba (Issuer Blindness). Tiết lộ chọn lọc và kiểm tra vị từ (Predicate Verification): người dùng chỉ chia sẻ kết quả logic (ví dụ: `age_over_18 = true`) mà không lộ ngày sinh hay họ tên. Chống tương quan (Anti-correlation): sử dụng mã định danh giả danh theo cặp (PPID) và khóa thiết bị riêng biệt cho từng bên xác thực, ngăn chặn các super-app hoặc mini-app liên kết dữ liệu hành vi người dùng.
   - **Áp dụng Super-App**: Bắt buộc mini-app trong các lĩnh vực giải trí/thương mại chỉ được yêu cầu vị từ tuổi thay vì toàn bộ hồ sơ cá nhân, tuân thủ nghiêm ngặt nguyên tắc giảm thiểu dữ liệu (Data Minimization).

10. **eudi_137_10 - Super-App EUDI Integration: Cross-Border Onboarding, e-Commerce Age Gating & Regulated API Delegation**
    - **Tiêu chuẩn**: EUDI Wallet ARF Section 7 & 8; W3C Digital Credentials API.
    - **Cơ chế kỹ thuật**: Tích hợp EUDI Wallet thông qua chuẩn W3C Digital Credentials API (`navigator.identity.get`), cung cấp trải nghiệm native sheet liền mạch. Ràng buộc giao dịch thanh toán giá trị cao với chữ ký xác nhận số theo quy định PSD2/SCA. Kiểm soát quyền truy cập của mini-app thông qua chứng chỉ Relying Party được chứng thực bởi cơ quan giám sát quốc gia.
    - **Áp dụng Super-App**: Xóa bỏ hoàn toàn quy trình KYC thủ công bằng giấy tờ đối với công dân quốc tế; mini-app chỉ cần gọi API container để nhận về kết quả xác minh danh tính cấp độ bảo đảm cao (High LoA).

---

### Nhóm 3: IETF SCITT Transparency Ledgers & COSE Merkle Tree Proofs (scitt_137_11 -> scitt_137_15)

11. **scitt_137_11 - IETF SCITT Architecture: Supply Chain Integrity, Transparency and Trust Model & Core Roles**
    - **Tiêu chuẩn**: IETF draft-ietf-scitt-architecture Section 1, 2 & 3; draft-ietf-scitt-software-use-cases.
    - **Cơ chế kỹ thuật**: Đặc tả kiến trúc sổ cái minh bạch chuỗi cung ứng mở với 4 vai trò: Issuer (bên phát hành và ký tuyên bố), Transparency Service / Notary (dịch vụ công chứng kiểm tra tuyên bố và ghi vào sổ cái append-only), Ledger (sổ cái Merkle bất biến), Verifier (bên xác thực tuyên bố và biên nhận). Tách biệt hoàn toàn việc tạo phát biểu của nhà phát triển và việc công chứng toàn cầu.
    - **Áp dụng Super-App**: Vận hành dịch vụ công chứng SCITT nội bộ trong hạ tầng quản lý kho ứng dụng super-app, ghi lại toàn bộ nhật ký phê duyệt phát hành, quét bảo mật và thu hồi mini-app.

12. **scitt_137_12 - SCITT Signed Statements: COSE_Sign1 Envelopes, Feeds & Canonical Payload Binding**
    - **Tiêu chuẩn**: IETF draft-ietf-scitt-architecture Section 4 & 5; IETF RFC 9052 / RFC 9053; IETF RFC 8949.
    - **Cơ chế kỹ thuật**: Đóng gói tuyên bố chuỗi cung ứng trong cấu trúc `COSE_Sign1` nhị phân chuẩn hóa. Sử dụng trường `feed` trong protected header để nhóm chuỗi sự kiện vòng đời của một mã ứng dụng (ví dụ: `miniapp.food.delivery/releases`). Tải trọng (payload) chứa SBOM (CycloneDX/SPDX), báo cáo OpenVEX hoặc chứng chỉ xuất xứ SLSA. Mã hóa nhị phân chuẩn Canonical CBOR đảm bảo tính đơn nhất của hàm băm.
    - **Áp dụng Super-App**: Yêu cầu đường ống CI/CD của nhà phát triển mini-app xuất ra tệp phát biểu SCITT ký số, gắn kèm mã băm SHA-256 của gói mini-app và commit hash Git trước khi nộp lên kho ứng dụng.

13. **scitt_137_13 - SCITT Transparency Receipts: Merkle Tree Inclusion Proofs, Tree Head Signing & draft-ietf-cose-merkle-tree-proofs**
    - **Tiêu chuẩn**: IETF draft-ietf-cose-merkle-tree-proofs Section 3; draft-ietf-scitt-architecture Section 6; draft-birkholz-scitt-receipts.
    - **Cơ chế kỹ thuật**: Dịch vụ công chứng tạo Biên nhận SCITT (SCITT Receipt) gồm: Bằng chứng bao hàm cây Merkle (Inclusion Proof) chứng minh vị trí lá của tuyên bố và Gốc cây đã ký (Signed Tree Head - STH). Cấu trúc mã hóa theo COSE Merkle Tree Proofs cho phép xác thực ngoại tuyến (offline verification) độc lập mà không cần kết nối trực tiếp đến máy chủ sổ cái. Ngăn chặn triệt để tấn công phân tách thế giới (split-world attacks).
    - **Áp dụng Super-App**: Đóng gói biên nhận SCITT trực tiếp vào gói `.zip` của mini-app; container client xác thực bằng chứng bao hàm Merkle trước khi giải nén và thực thi mã.

14. **scitt_137_14 - SCITT Supply Chain Attestation Gating: SBOMs, OpenVEX Exploitation Status & SLSA Provenance Binding**
    - **Tiêu chuẩn**: IETF draft-ietf-scitt-software-use-cases Section 3 & 4; OpenVEX Specification; OpenSSF SLSA v1.0.
    - **Cơ chế kỹ thuật**: SCITT đóng vai trò lớp niềm tin thống nhất liên kết SBOM (Software Bill of Materials), đánh giá lỗ hổng động OpenVEX và xuất xứ bản dựng SLSA. Khi phát hiện CVE mới trong thư viện bên thứ ba của mini-app, nhà phát triển hoặc hệ thống quét tự động ghi tuyên bố OpenVEX (`affected`, `fixed`, `not_affected`) lên feed SCITT. Cho phép kích hoạt quy trình thu hồi hoặc vá lỗi tự động dựa trên bằng chứng mật mã không thể chối bỏ.
    - **Áp dụng Super-App**: Hệ thống duyệt kho ứng dụng tự động kiểm tra trạng thái OpenVEX trên feed SCITT, tự động cảnh báo hoặc tạm dừng phân phối các mini-app chứa lỗ hổng nghiêm trọng chưa được vá.

15. **scitt_137_15 - Super-App SCITT Release Pipeline: Transparent Store Notarization, Instant Revocation & Zero-Trust Client Execution**
    - **Tiêu chuẩn**: IETF draft-ietf-scitt-architecture Section 7 & 8; RFC 9053.
    - **Cơ chế kỹ thuật**: Thiết lập mô hình công chứng hai giai đoạn (Two-Phase Notarization): (1) Nhà phát triển ký phát biểu bản dựng trên feed riêng; (2) Kho ứng dụng kiểm toán và ký phát biểu phê duyệt trên feed công khai của nền tảng. Container người dùng chỉ thực thi mini-app nếu gói bundle có đầy đủ bằng chứng Merkle hợp lệ từ cả hai bên. Hỗ trợ cơ chế ngắt khẩn cấp (Emergency Kill-Switch): đăng ký tuyên bố thu hồi trên sổ cái SCITT và phát sóng tức thì qua WebPush/WebSocket để container ngừng chạy mã độc hại ngay lập tức.
    - **Áp dụng Super-App**: Loại trừ hoàn toàn rủi ro nội bộ (nhân viên quản trị tự ý chèn backdoor vào mini-app), biến kho ứng dụng thành môi trường thực thi Zero-Trust có thể kiểm toán công khai.

---

## 3. Tổng kết & Tình trạng Đồng bộ Kho Lưu trữ

- **Tổng số Findings toàn diện**: **2,006 findings** (vượt mốc 2,000 chuẩn mực kỹ thuật).
- **Tình trạng Dữ liệu**:
  - `state/findings.jsonl`: 2,006 dòng hợp lệ, append-only tuyệt đối.
  - `state/progress.json`: iteration 137, total_findings 2006, stale_count 0.
  - `state/directions_tried.json`: 129 hướng nghiên cứu độc lập.
- **Báo cáo chuyên đề**: Lưu trữ tại `/home/minhgv/mData/hermes_vps/super-app-mini-app-store-standard-report/137-c2pa-provenance-eudi-wallet-arf-and-ietf-scitt.md`.
