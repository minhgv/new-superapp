# SPEC-04 — An ninh & Cách ly (Sandboxing)

| Trường | Giá trị |
|---|---|
| Spec ID | `SPEC-04` |
| Phiên bản | 1.1.0 |
| Trạng thái | Draft |
| Lớp tuân thủ | **C1** (Host / Container) |
| Nguồn | `report/23`, `report/19`, `report/39`, `report/42`, `report/43`, `report/54`, `report/20`, `report/52`, `report/54`, `report/61` |

---

## 1. Phạm vi & mục tiêu

Đặc tả **mô hình an ninh** của container: chống XSS, sandbox mã, cách ly
bộ nhớ/tiến trình, kiểm soát egress mạng, và an toàn mật mã.

Mục tiêu: giả định mini-app là **không tin cậy** (untrusted). Mọi ranh giới
phải fail-closed; một mini-app bị chiếm quyền **không** được leo thang sang
host hoặc mini-app khác.

> **Chuẩn tham chiếu:** W3C Trusted Types, CSP Level 3, Cross-Origin Isolation,
> OWASP MASVS v2.0, NIST SP 800-218 (SSDF).

---

## 2. Thuật ngữ

| Thuật ngữ | Định nghĩa |
|---|---|
| **Sink** | Điểm nguy hiểm ghi dữ liệu vào DOM/exec (innerHTML, eval…). |
| **Trusted Types** | Cơ chế bắt buộc kiểu dữ liệu tin cậy cho sink. |
| **SRI** | Subresource Integrity — xác minh tài nguyên qua digest. |
| **COOP / COEP / CORP** | Chính sách cách ly origin chéo. |
| **Linear memory** | Vùng nhớ tuyến tính của WebAssembly. |
| **Sandbox layer** | Tầng cách ly: JS sandbox → origin sandbox → process sandbox. |
| **Egress allowlist** | Danh sách miền cho phép khi gọi ra ngoài. |

---

## 3. Yêu cầu chuẩn hóa

### 3.1 Chống XSS & Trusted Types

> **REQ-04-001** (MUST · C1) — Container **PHẢI** bật thực thi `require-trusted-types-for 'script'` cho mọi mini-app.
> *Nguồn:* `→ report/23-...md` · *Kiểm chứng:* static + runtime

> **REQ-04-002** (MUST · C1) — Toàn bộ nhóm sink script **PHẢI** bị chặn nếu không có Trusted Type hợp lệ: `innerHTML`, `outerHTML`, `document.write`, `parseFromString`, setter URL script, thực thi kiểu `eval`, điều hướng `javascript:`.
> *Nguồn:* `→ report/23-...md` · *Kiểm chứng:* static

> **REQ-04-003** (MUST · C1) — Chính sách Trusted Types **PHẢI** được tạo cho từng mini-app và thu hồi khi điều hướng hoặc cách ly (quarantine).
> *Nguồn:* `→ report/23-...md` · *Kiểm chứng:* runtime

> **REQ-04-004** (SHOULD · C1) — Host **NÊN** bật `report-sample` trong CSP để ghi mẫu vi phạm phục vụ kiểm duyệt, nhưng **KHÔNG** được để lộ dữ liệu người dùng trong báo cáo.
> *Nguồn:* `→ report/42-...md` · *Kiểm chứng:* runtime

> **REQ-04-005** (MUST · C1) — Dữ liệu chèn vào DOM từ nguồn ngoài **PHẢI** được khử trùng bằng Sanitizer API (hoặc bộ khử trùng tương đương được kiểm chứng) trước khi chèn.
> *Nguồn:* `→ report/52-...md` · *Kiểm chứng:* runtime

### 3.2 CSP & kiểm soát egress

> **REQ-04-006** (MUST · C1) — Mỗi mini-app **PHẢI** có CSP riêng biệt với `script-src`, `connect-src`, `img-src`, `frame-ancestors` tường minh; **KHÔNG ĐƯỢC** dùng `*` cho `connect-src` hoặc `script-src`.
> *Nguồn:* `→ report/19-...md` · *Kiểm chứng:* static

> **REQ-04-007** (MUST · C1) — Egress mạng **PHẢI** bị chặn ở tầng container theo danh sách miền khai báo; CSP là lớp thứ hai, không phải lớp duy nhất.
> *Nguồn:* `→ report/19-...md` · *Kiểm chứng:* runtime

> **REQ-04-008** (MUST · C1) — Mini-app **KHÔNG ĐƯỢC** gọi miền không khai báo trong manifest network endpoints; DNS/HTTP tới miền ngoài phải trả lỗi tường minh.
> *Nguồn:* `→ report/19-...md` · *Kiểm chứng:* runtime

> **REQ-04-009** (SHOULD · C2) — Trước production, **NÊN** chạy CSP ở chế độ `Report-Only` kép để đo lường vi phạm mà không phá trải nghiệm, rồi mới chuyển sang thực thi.
> *Nguồn:* `→ report/36-...md` · *Kiểm chứng:* review

> **REQ-04-010** (MUST · C1) — Chống clickjacking: `frame-ancestors` **PHẢI** giới hạn origin host; mini-app **KHÔNG ĐƯỢC** nhúng vào iframe ngoài phạm vi được ủy quyền.
> *Nguồn:* `→ report/19-...md`, `→ report/66-...md` · *Kiểm chứng:* runtime

> **REQ-04-011** (MUST · C1) — Tăng cường yêu cầu bảo mật kết nối: bật `Upgrade-Insecure-Requests`, dùng `Sec-Fetch-Site/Mode/Dest` ở gateway để chống CSRF/XSSI/XS-Leaks, và `Cross-Origin-Resource-Policy` cho tài nguyên tĩnh.
> *Nguồn:* `→ report/39-...md`, `→ report/36-...md` · *Kiểm chứng:* static

### 3.3 Cách ly JavaScript & module

> **REQ-04-012** (SHOULD · C1) — Logic mini-app **NÊN** chạy trong sandbox JS (ShadowRealm hoặc Secure ECMAScript/Compartments) với khóa nguyên thủy (primordial lockdown) và proxy cầu có thể thu hồi.
> *Nguồn:* `→ report/42-...md` · *Kiểm chứng:* runtime

> **REQ-04-013** (MUST · C1) — Container **PHẢI** chặn sửa đổi prototype (prototype pollution) của các đối tượng chuẩn; đối tượng nguyên thủy phải được khóa.
> *Nguồn:* `→ report/42-...md`, `→ report/16-...md` · *Kiểm chứng:* runtime

> **REQ-04-014** (SHOULD · C1) — Import Maps **NÊN** được dùng để ảo hóa specifier trần (bare specifiers), cho phép đa phiên bản thư viện và chặn module động tùy ý.
> *Nguồn:* `→ report/39-...md` · *Kiểm chứng:* static

> **REQ-04-015** (MUST · C1) — Registry phần tử tùy chỉnh **PHẢI** được ảo hóa theo phạm vi; chống đụng nhãn (tag collision) giữa các mini-app và host.
> *Nguồn:* `→ report/39-...md` · *Kiểm chứng:* static

> **REQ-04-016** (MUST · C1) — Mọi module/tài nguyên tải từ xa **PHẢI** có Subresource Integrity (SRI) hoặc được ký trong gói; **KHÔNG** được tải mã không toàn vẹn.
> *Nguồn:* `→ report/54-...md` · *Kiểm chứng:* static

### 3.4 WebAssembly & tính toán

> **REQ-04-017** (MUST · C1) — WebAssembly **PHẢI** chạy trong vùng nhớ tuyến tính cách ly; lời gọi gián tiếp qua bảng (table indirect calls) phải được kiểm soát.
> *Nguồn:* `→ report/23-...md` · *Kiểm chứng:* runtime

> **REQ-04-018** (MUST · C1) — Việc dùng WASM **PHẢI** được CSP cho phép tường minh qua `wasm-unsafe-eval`; **KHÔNG** dùng `unsafe-eval` chung.
> *Nguồn:* `→ report/23-...md` · *Kiểm chứng:* static

> **REQ-04-019** (SHOULD · C1) — WasmGC và các module tính toán nặng **NÊN** chạy trong worker cách ly với giới hạn bộ nhớ và thời gian thực thi.
> *Nguồn:* `→ report/52-...md`, `→ report/34-...md` · *Kiểm chứng:* runtime

### 3.5 Cách ly origin & tiến trình

> **REQ-04-020** (MUST · C1) — Cross-Origin Isolation **PHẢI** được bật khi dùng `SharedArrayBuffer`/`Atomics`; thiếu COOP/COEP thì tính năng chia sẻ bộ nhớ phải bị tắt.
> *Nguồn:* `→ report/23-...md`, `→ report/36-...md` · *Kiểm chứng:* runtime

> **REQ-04-021** (MUST · C1) — Container **PHẢI** cách ly agent cluster theo origin và vô hiệu hóa `document.domain`.
> *Nguồn:* `→ report/39-...md` · *Kiểm chứng:* runtime

> **REQ-04-022** (SHOULD · C1) — Site/process isolation **NÊN** được bật (OOPIF, Site Isolation) để một tiến trình render bị hỏng không ảnh hưởng host hay mini-app khác.
> *Nguồn:* `→ report/27-...md` · *Kiểm chứng:* runtime

> **REQ-04-023** (MUST · C1) — Bối cảnh nội bộ của host (API khóa, logic thanh toán, mã thông báo) **PHẢI** nằm ngoài JS ngữ cảnh mini-app; mini-app không bao giờ nhìn thấy khóa/tokens gốc.
> *Nguồn:* `→ report/20-...md` · *Kiểm chứng:* static + runtime

### 3.6 Bộ nhớ, tệp & sandbox đường dẫn

> **REQ-04-024** (MUST · C1) — Truy cập hệ tệp **PHẢI** dùng sandbox đường dẫn (virtual chroot), chặn thư mục cha (`../`), và có danh sách từ chối của OS.
> *Nguồn:* `→ report/32-...md`, `→ report/47-...md` · *Kiểm chứng:* runtime

> **REQ-04-025** (MUST · C1) — Giải nén tệp tải về **PHẢI** có trần kích thước và trần tỷ lệ nén để chống bom nén (decompression bomb); tệp nghi ngờ phải bị cách ly (quarantine).
> *Nguồn:* `→ report/32-...md` · *Kiểm chứng:* runtime

> **REQ-04-026** (MUST · C1) — Lưu trữ **PHẢI** phân vùng theo mini-app; mini-app A **KHÔNG ĐƯỢC** đọc/ghi vùng của mini-app B.
> *Nguồn:* `→ report/19-...md`, `→ report/28-...md` · *Kiểm chứng:* runtime

### 3.7 Mật mã & kho khóa

> **REQ-04-027** (MUST · C1) — Khóa bí mật **PHẢI** dùng `CryptoKey` không xuất được, được giữ trong phần cứng (Android Keystore StrongBox/KeyMint, Apple Secure Enclave).
> *Nguồn:* `→ report/20-...md`, `→ report/43-...md` · *Kiểm chứng:* runtime

> **REQ-04-028** (MUST · C1) — Kho khóa **PHẢI** cách ly theo `app_id`; khóa phải được xóa (zeroize) khi gỡ cài hoặc xóa dữ liệu container.
> *Nguồn:* `→ report/20-...md` · *Kiểm chứng:* runtime

> **REQ-04-029** (MUST · C1) — Kiểm duyệt **PHẢI** quét ở mức AST/byte để tìm khóa đối xứng hardcode, PEM khóa riêng, token tĩnh trong gói; phát hiện thì từ chối.
> *Nguồn:* `→ report/20-...md` · *Kiểm chứng:* static

> **REQ-04-030** (SHOULD · C2) — Nền tảng **NÊN** lên kế hoạch mật mã hậu lượng tử (post-quantum) cho các khóa dài hạn và chuỗi cung ứng.
> *Nguồn:* `→ report/40-...md` · *Kiểm chứng:* review

### 3.8 Chống giả mạo & đàn hồi (Resilience)

> **REQ-04-031** (SHOULD · C1) — Host **NÊN** có RASP/chống giả mạo: phát hiện root/jailbreak, hook, gỡ gỡ, giả lập — và phản hồi theo chính sách (chặn hoặc giảm chức năng).
> *Nguồn:* `→ report/37-...md`, `→ report/39-...md` · *Kiểm chứng:* runtime

> **REQ-04-032** (MUST · C1) — Chứng thực thiết bị **PHẢI** được kiểm tra khi có giao dịch nhạy cảm (thanh toán, danh tính) qua Play Integrity / App Attest, **KHÔNG** được tin thiết bị không qua attestation.
> *Nguồn:* `→ report/37-...md` · *Kiểm chứng:* runtime

> **REQ-04-033** (MUST · C1) — Tín hiệu bảo mật bất thường **PHẢI** được phát đi qua Shared Signals (CAEP/RISC) để thu hồi phiên và quyền theo thời gian thực.
> *Nguồn:* `→ report/53-...md` · *Kiểm chứng:* runtime

> **REQ-04-034** (SHOULD · C2) — Cửa hàng **NÊN** công bố `security.txt` (RFC 9116) để nhận báo cáo lỗ hổng có trách nhiệm.
> *Nguồn:* `→ report/39-...md` · *Kiểm chứng:* static

---

### 3.6 Cách ly mạng, bảo toàn riêng tư & chống khai thác

*Bổ sung từ `report/136`, `report/138`, `report/141`, `report/142`, `report/145`.*

> **REQ-04-035** (MUST · C1) — Truy cập mạng cục bộ từ mini app **PHẢI** đi qua **Private Network Access (PNA)** preflight (`Access-Control-Request-Private-Network`); **KHÔNG** cho phép gọi thẳng vào mạng nội bộ.
> *Nguồn:* `→ report/142-...md` · *Kiểm chứng:* static

> **REQ-04-036** (MUST · C1) — Không gian địa chỉ IP **PHẢI** được phân loại theo **RFC 1918** (public / private / loopback / link-local); bộ chặn **PHẢI** chống SSRF nội bộ và dò loopback.
> *Nguồn:* `→ report/142-...md` · *Kiểm chứng:* runtime

> **REQ-04-037** (SHOULD · C1) — Cách ly tải chéo origin **NÊN** dùng **COEP `credentialless`** và `iframe credentialless` để tải tài nguyên chéo origin **KHÔNG** kèm thông tin xác thực.
> *Nguồn:* `→ report/138-...md` · *Kiểm chứng:* static

> **REQ-04-038** (SHOULD · C1) — Yêu cầu cần ẩn danh **NÊN** dùng **RFC 9458** (Oblivious HTTP) với gateway-Relay không thông đồng, khung Binary HTTP và HPKE.
> *Nguồn:* `→ report/136-...md` · *Kiểm chứng:* runtime

> **REQ-04-039** (SHOULD · C2) — Đo lường thống kê đa bên **NÊN** dùng **IETF DAP** (Distributed Aggregation Protocol) với VDAF `Prio3` để tổng hợp mà **KHÔNG** bên nào thấy dữ liệu thô.
> *Nguồn:* `→ report/136-...md` · *Kiểm chứng:* runtime

> **REQ-04-040** (SHOULD · C1) — Chống lạm dụng giữ riêng tư **NÊN** dùng **RFC 9497** (Oblivious PRF) và **Private State Tokens** (phát hành/chuộc, chống theo dõi).
> *Nguồn:* `→ report/136-...md` · *Kiểm chứng:* runtime

> **REQ-04-041** (MUST · C1) — Biểu thức chính quy do người dùng hoặc nhà phát triển cung cấp **PHẢI** thuộc tập **I-Regexp** (RFC 9485) hoặc được kiểm soát theo thời gian chạy — **KHÔNG** chấp nhận vector ReDoS.
> *Nguồn:* `→ report/141-...md` · *Kiểm chứng:* static

> **REQ-04-042** (SHOULD · C1) — Mô-đun WebAssembly **NÊN** chạy trong sandbox cách ly, tách khỏi luồng chính; tích hợp bất đồng bộ qua **Wasm JSPI** thay vì chặn luồng chính.
> *Nguồn:* `→ report/142-...md`, `→ report/145-...md` · *Kiểm chứng:* runtime

## 4. Ghi chú triển khai (informative)

**Mô hình phòng thủ theo lớp (defense in depth):**

```text
Lớp 1  Trusted Types + Sanitizer   → chống chèn mã
Lớp 2  CSP + egress allowlist      → chống gọi ra ngoài tùy ý
Lớp 3  JS sandbox / Import Maps    → chống leo thang trong JS
Lớp 4  Origin & agent cluster      → chống lây lan chéo mini-app
Lớp 5  Process isolation           → chống sụp đổ lan
Lớp 6  Hardware keys + attestation → chống giả mạo thiết bị
```

Một lỗ hổng vượt qua một lớp **không** được phép xuyên qua các lớp còn lại.
Thiết kế phải giả định lớp nào đó thất bại.

---

## 5. Tiêu chí tuân thủ

- [ ] Trusted Types được thực thi; mọi sink bị chặn nếu thiếu Trusted Type.
- [ ] CSP riêng từng mini-app, không `*` cho `script-src`/`connect-src`.
- [ ] Egress bị chặn ở container, không chỉ ở CSP.
- [ ] SRI/kiểm tra toàn vẹn cho mọi tài nguyên tải.
- [ ] WASM cách ly tuyến tính + `wasm-unsafe-eval`.
- [ ] COOP/COEP khi dùng SharedArrayBuffer; agent cluster cách ly.
- [ ] Sandbox đường dẫn + chống bom nén cho tệp.
- [ ] Kho khóa phần cứng, cách ly theo app_id, zeroize khi gỡ.
- [ ] Quét secret ở mức AST/byte khi kiểm duyệt.

---

## 6. Cân nhắc an ninh & riêng tư

- **Giả định mini-app độc hại:** không tin dữ liệu đầu vào, không tin mã nguồn.
- **Không lộ nguyên nhân:** lỗi bảo mật phải mơ hồ với mini-app, chi tiết chỉ ở phía host.
- **Báo cáo CSP không được chứa PII:** lọc trước khi đưa vào hệ thống quan sát.
- **Thu hồi phải lan:** một sự cố bảo mật phải kéo theo thu hồi quyền + phiên ngay lập tức.

---

## 7. Tham chiếu

- W3C Trusted Types — https://w3c.github.io/trusted-types/dist/spec/
- W3C CSP Level 3 — `report/23`, `report/42`
- WHATWG Cross-Origin Isolation — `report/23`
- OWASP MASVS v2.0 (PLATFORM, CODE, RESILIENCE)
- NIST SP 800-218 SSDF v1.1
- Báo cáo nguồn: `report/19, 20, 23, 27, 36, 39, 40, 42, 43, 47, 52, 53, 54`
