# Chuyên đề Iteration 35: Privacy-Preserving Attribution, Anti-Abuse Private State Tokens, and On-Device Ad Auctions (Protected Audience & Fenced Frames)

## 1. Tổng quan và Bối cảnh Kiến trúc

Trong hệ sinh thái super-app với hàng trăm đối tác thứ ba (mini-apps, nhà quảng cáo, sàn thương mại điện tử), việc đo lường hiệu quả chuyển đổi (conversion attribution), bảo vệ chống gian lận bot/Sybil (anti-abuse trust attestation), và phân phối quảng cáo hướng đối tượng (ad targeting/personalization) thường xung đột trực tiếp với quyền riêng tư của người dùng và các quy định bảo vệ dữ liệu (GDPR, Apple ATT, Google Privacy Sandbox, Luật An ninh mạng).

Iteration 35 thiết lập khung kiến trúc bảo vệ quyền riêng tư toàn diện cho mini-app container dựa trên 3 trụ cột kỹ thuật mở chuẩn hóa quốc tế:
1. **Attribution Reporting & Conversion Measurement (WICG Attribution Reporting API / IETF RFC 8941)**: Chuẩn hóa quy trình ghi nhận nguồn (source registration) và kích hoạt chuyển đổi (trigger registration) cục bộ trên thiết bị, loại bỏ cookies bên thứ ba. Áp dụng cơ chế $\epsilon$-differential privacy với randomized response để giới hạn entropy của báo cáo cấp sự kiện (3 bits cho click, 1 bit cho view), mã hóa HPKE (RFC 9180) cho báo cáo tổng hợp (aggregatable reports) xử lý trong môi trường thực thi tin cậy (TEE), tích hợp đối chuẩn với Apple SKAdNetwork 4.0 / AdAttributionKit và Google Play Install Referrer, cùng cổng Attribution Gateway ký số lượt click chống gian lận quảng cáo.
2. **Anti-Abuse & Anonymous Trust Attestation (WICG Private State Token API / IETF Privacy Pass RFC 9576, RFC 9577, RFC 9578)**: Triển khai giao thức thẻ trạng thái ẩn danh dựa trên hàm giả ngẫu nhiên mù có thể xác minh (Verifiable Oblivious PRF - VOPRF). Cho phép super-app host cấp phát thẻ tín nhiệm (trust tokens) cho người dùng đã hoàn thành KYC/xác thực sinh trắc học mà không thể liên kết (unlinkable) giữa ngữ cảnh cấp phát và ngữ cảnh chuộc thẻ (redemption) trong mini-app. Quy định giới hạn tối đa 2 tổ chức phát hành (issuers) trên mỗi top-level origin, kiểm soát bằng Permissions Policy `private-state-token-redemption`, lưu trữ bộ đệm Redemption Record (RR), tạo lưới tín nhiệm chống bot/click-farm mà không làm lộ số điện thoại hay tài khoản người dùng.
3. **On-Device Ad Auctions & Isolated Rendering (WICG Protected Audience API / Fenced Frames / Shared Storage)**: Chuyển toàn bộ quá trình đấu thầu và khớp lệnh quảng cáo về cục bộ trên máy khách (on-device auction) thông qua `navigator.joinAdInterestGroup()` và `navigator.runAdAuction()`. Mã thực thi của người mua (`generateBid`) và người bán (`scoreAd`) chạy trong các worklet cô lập hoàn toàn không có quyền truy cập mạng. Nội dung quảng cáo trúng thầu được hiển thị bên trong phần tử `<fencedframe>` với cấu hình mờ `FencedFrameConfig` và kích hoạt hàm đóng mạng `disableUntrustedNetwork()`. Dữ liệu đo lường tần suất (reach/frequency) được quản lý qua Shared Storage và Private Aggregation với ngân sách đóng góp L1 (L1 contribution budget) tối đa 65.536 đơn vị.

Tất cả 15 findings đã được kiểm tra tính hợp lệ JSONL, rà soát bí mật/credential, xác thực HTTP 200 trên 7 URL chuẩn chính thức, và ghi nhận append-only vào `state/findings.jsonl`.

---

## 2. Bảng tổng hợp Findings Iteration 35

| ID | Chủ đề | Tiêu đề Finding | Chuẩn tham chiếu | Mức bằng chứng | URL nguồn chính |
|---|---|---|---|---|---|
| `attr_035_01` | Attribution Architecture | WICG Attribution Reporting API Architecture: Source Registration, Trigger Registration, and Event-Level vs Aggregatable Conversion Delivery | WICG Attribution Reporting API / RFC 8941 | `normative_standard` | [WICG Attribution Reporting](https://wicg.github.io/attribution-reporting-api/) |
| `attr_035_02` | Differential Privacy | Differential Privacy & Randomized Response: Event-Level Noise, Coarse Value Bucketing, and Delay-Randomized Postback Scheduling | WICG Attribution Reporting API / Diff Privacy | `normative_standard` | [WICG Attribution Reporting](https://wicg.github.io/attribution-reporting-api/) |
| `attr_035_03` | Aggregatable Reports & TEE | Aggregatable Reports & TEE Aggregation Service: 128-Bit Histogram Buckets, HPKE Payload Encryption, and Centralized Differential Privacy | WICG Attribution Reporting / RFC 9180 HPKE | `normative_standard` | [WICG Attribution Reporting](https://wicg.github.io/attribution-reporting-api/) |
| `attr_035_04` | Native Attribution Parity | Native Host Attribution Parity: Apple SKAdNetwork 4.0 / AdAttributionKit and Google Play Install Referrer API Integration | Apple SKAdNetwork 4.0 / Google Play Referrer | `platform_practice` | [Apple SKAdNetwork](https://developer.apple.com/documentation/storekit/skadnetwork) |
| `attr_035_05` | Attribution Gateway | Super-App Attribution Gateway Architecture: Cryptographic Click Attestation, Host-Mediated Redirects, and Ad Fraud Mitigations | WICG Attribution Reporting / OWASP Mobile | `platform_practice` | [WICG Attribution Reporting](https://wicg.github.io/attribution-reporting-api/) |
| `pst_035_01` | Private State Tokens | WICG Private State Token API: Privacy Pass Verifiable Oblivious PRF (VOPRF) Primitives and Cross-Site Anti-Abuse Tokens | WICG PST / RFC 9576 Privacy Pass | `normative_standard` | [WICG Trust Token API](https://wicg.github.io/trust-token-api/) |
| `pst_035_02` | PST Protocol Lifecycle | Private State Token Protocol Lifecycle: Key Commitments, HTTP Sec-Headers, and Redemption Record Caching | WICG PST / WHATWG Fetch / RFC 8446 | `normative_standard` | [WICG Trust Token API](https://wicg.github.io/trust-token-api/) |
| `pst_035_03` | Privacy Pass Suite | IETF Privacy Pass Standards Suite: RFC 9576 Architecture, RFC 9577 Two-Party Protocols, and RFC 9578 HTTP Authentication | IETF RFC 9576 / RFC 9577 / RFC 9578 | `normative_standard` | [RFC 9576 Privacy Pass](https://www.rfc-editor.org/rfc/rfc9576.html) |
| `pst_035_04` | PST Security Boundaries | PST Security Limits: 2-Issuer Top-Level Origin Clamping, Permissions Policy Gating, and Identity Leakage Mitigations | WICG PST / W3C Permissions Policy | `normative_standard` | [WICG Trust Token API](https://wicg.github.io/trust-token-api/) |
| `pst_035_05` | Anti-Abuse Trust Mesh | Super-App Anti-Abuse Trust Mesh: Host Account Verification, Mini-App Bot Mitigation, and CAPTCHA-Free Frictionless Flow | WICG PST / RFC 9576 / OWASP OAT | `platform_practice` | [WICG Trust Token API](https://wicg.github.io/trust-token-api/) |
| `pa_035_01` | On-Device Ad Auctions | WICG Protected Audience API: On-Device Ad Auctions, Interest Groups (joinAdInterestGroup), and Isolated Buyer/Seller Bidding Logic | WICG Protected Audience / WHATWG Worklets | `normative_standard` | [WICG Protected Audience](https://wicg.github.io/turtledove/) |
| `pa_035_02` | Fenced Frames Sandboxing | WICG Fenced Frames Specification: Opaque URL Rendering, FencedFrameConfig, and Cryptographic Network Disabling | WICG Fenced Frame / W3C Permissions Policy | `normative_standard` | [WICG Fenced Frame](https://wicg.github.io/fenced-frame/) |
| `pa_035_03` | Shared Storage Worklets | WICG Shared Storage API: Unpartitioned Cross-Site Storage, Worklet Operations, and Private Aggregation Output Gates | WICG Shared Storage / WHATWG Web IDL | `normative_standard` | [WICG Shared Storage](https://wicg.github.io/shared-storage/) |
| `pa_035_04` | Private Aggregation | Cross-Site Private Aggregation: contributeToHistogram(), L1 Contribution Budgets, and Filtering IDs | WICG Shared Storage / RFC 9180 HPKE | `normative_standard` | [WICG Shared Storage](https://wicg.github.io/shared-storage/) |
| `pa_035_05` | Zero-PII Ad Exchange | Super-App In-Container Ad Exchange: On-Device Personalization, Fenced Mini-App Banners, and Zero-PII Egress Governance | WICG Protected Audience / Fenced Frame | `platform_practice` | [WICG Protected Audience](https://wicg.github.io/turtledove/) |

---

## 3. Phân tích Kỹ thuật Chi tiết theo 3 Phân vùng Nghiên cứu

### 3.1. Attribution Reporting & Conversion Measurement

#### 3.1.1. Kiến trúc Đăng ký Nguồn (Source) và Kích hoạt Chuyển đổi (Trigger)
- **Đăng ký Nguồn (Source Registration)**:
  - Khi người dùng nhấp vào biểu tượng mini-app trên trang chủ super-app hoặc tương tác với banner liên kết chéo, trình duyệt gửi yêu cầu mạng mang header `Attribution-Reporting-Eligible: navigation-source` (cho điều hướng) hoặc `event-source` (cho cuộc gọi fetch nền).
  - Máy chủ super-app phản hồi với header `Attribution-Reporting-Register-Source`:
    ```http
    Attribution-Reporting-Register-Source: {
      "source_event_id": "64321987512",
      "destination": "https://merchant-miniapp.superapp.vn",
      "expiry": "2592000",
      "priority": "100",
      "filter_data": { "conversion_subdomain": ["checkout", "cart"] }
    }
    ```
  - Trình duyệt lưu trữ nguồn đăng ký cục bộ trong cơ sở dữ liệu Attribution Storage, gắn nhãn với trang đích `destination` mà không truyền thông tin này sang ngữ cảnh của bên thứ ba.
- **Kích hoạt Chuyển đổi (Trigger Registration)**:
  - Khi người dùng hoàn thành giao dịch mua sắm bên trong mini-app đích, mini-app gửi yêu cầu POST đến endpoint ghi nhận chuyển đổi với header `Attribution-Reporting-Eligible: trigger`.
  - Phản hồi trả về header `Attribution-Reporting-Register-Trigger`:
    ```http
    Attribution-Reporting-Register-Trigger: {
      "event_trigger_data": [{ "trigger_data": "3", "priority": "10" }],
      "aggregatable_trigger_data": [{ "key_piece": "0x400", "source_keys": ["campaign_id"] }],
      "aggregatable_values": { "campaign_id": 1250 }
    }
    ```
  - Trình duyệt tự động khớp top-level site của nguồn và đích. Nếu khớp, báo cáo chuyển đổi được tạo ra hoàn toàn offline trên thiết bị.

#### 3.1.2. Cơ chế $\epsilon$-Differential Privacy và Randomized Response
- Để ngăn chặn việc bên phát hành liên kết báo cáo chuyển đổi với hành vi cá nhân của người dùng, báo cáo cấp sự kiện (event-level reports) bị giới hạn dung lượng thông tin:
  - **Lượt click (Navigation sources)**: Dữ liệu chuyển đổi bị gán trần ở mức 3 bits (8 trạng thái phân loại từ 0 đến 7).
  - **Lượt xem (View sources)**: Dữ liệu chuyển đổi chỉ cho phép đúng 1 bit (0 hoặc 1).
- **Randomized Response Noise**:
  - Với xác suất $p$, trình duyệt sẽ không gửi giá trị chuyển đổi thực mà chọn ngẫu nhiên một giá trị trong không gian $2^b$ trạng thái khả dĩ.
  - Cơ chế này đảm bảo tính khả phủ nhận hợp lý (plausible deniability) đạt chuẩn toán học $\epsilon$-differential privacy, triệt tiêu khả năng kẻ tấn công dùng các chuỗi chuyển đổi độc nhất để tạo vân tay người dùng (fingerprinting).
- **Lập lịch gửi trễ ngẫu nhiên (Delay-Randomized Scheduling)**:
  - Báo cáo không được gửi ngay lập tức sau khi chuyển đổi xảy ra để tránh tương quan thời gian mạng (traffic timing correlation).
  - Thay vào đó, thời gian gửi được chia thành các cửa sổ báo cáo cố định (2 ngày, 7 ngày, 30 ngày) cộng thêm một khoảng nhiễu ngẫu nhiên (random jitter vài giờ).

#### 3.1.3. Báo cáo Tổng hợp (Aggregatable Reports) và Dịch vụ TEE
- Đối với nhu cầu phân tích chi tiết đa chiều (giá trị đơn hàng, mã SKU, vùng địa lý), chuẩn WICG đưa ra mô hình Báo cáo Tổng hợp:
  - Đóng góp chuyển đổi được biểu diễn dưới dạng histogram bucket 128-bit kết hợp với giá trị số nguyên có giới hạn ngân sách L1.
  - Payload của báo cáo được mã hóa bất đối xứng ngay trên thiết bị bằng chuẩn **HPKE (RFC 9180)** sử dụng khóa công khai của Dịch vụ Tổng hợp (Aggregation Service). Cả super-app host lẫn ad-tech collector đều không thể đọc được nội dung bên trong gói tin mã hóa.
  - Báo cáo được gom cụm (batched) và gửi đến Dịch vụ Tổng hợp chạy trong môi trường điện toán bảo mật phần cứng (Trusted Execution Environment - AWS Nitro Enclaves hoặc GCP Confidential Space).
  - TEE thực hiện giải mã hàng loạt, tính tổng các bucket, cộng nhiễu Laplace theo tham số $\epsilon$, và phát hành Báo cáo Tóm tắt (Summary Report) duy nhất. Mọi nỗ lực gửi trùng lặp báo cáo đều bị từ chối thông qua bảng theo dõi ID báo cáo đơn nhất (anti-replay cache).

#### 3.1.4. Đối chuẩn Hạ tầng Native: SKAdNetwork và Install Referrer
- **Apple iOS Parity**:
  - Mini-app chạy trên iOS tuân thủ chính sách App Tracking Transparency (ATT). Super-app container không được phép thu thập IDFA nếu người dùng từ chối.
  - Tích hợp với **SKAdNetwork 4.0 / AdAttributionKit**: Hỗ trợ định danh nguồn phân cấp (2 đến 4 chữ số), giá trị chuyển đổi thô (coarse conversion values: fine, low, medium, high), và 3 cửa sổ gửi báo cáo (0-2 ngày, 3-7 ngày, 8-35 ngày).
- **Google Android Parity**:
  - Sử dụng **Google Play Install Referrer API**: Đọc an toàn các tham số chiến dịch tiếp thị cài đặt thông qua dịch vụ AIDL/IPC được ký số, có dấu thời gian nhấp chuột và cài đặt chính xác.
  - Tuyệt đối cấm mini-app truy cập trực tiếp Google Advertising ID (AAID) hoặc IMEI/MAC phần cứng.

---

### 3.2. Anti-Abuse & Anonymous Trust Attestation (Private State Tokens)

#### 3.2.1. Nền tảng Mật mã học Privacy Pass và VOPRF
- **Vấn đề cốt lõi**: Các hệ thống phòng chống gian lận (anti-bot, click fraud, voucher abuse) thường sử dụng cookie theo dõi, IP fingerprinting, hoặc CAPTCHA gây phiền hà cho người dùng.
- **Giải pháp WICG Private State Tokens (PST)**:
  - Ứng dụng bộ chuẩn **IETF Privacy Pass (RFC 9576, RFC 9577, RFC 9578)**.
  - Sử dụng hàm Verifiable Oblivious Pseudorandom Function (VOPRF - RFC 9497) trên đường cong elliptic prime-order (P-256 hoặc Curve25519).
  - **Quy trình Mù hóa (Blinding)**:
    1. Thiết bị người dùng (Client) tạo ra một chuỗi ngẫu nhiên (token) và làm mù toán học chuỗi này trước khi gửi cho Super-App Trust Issuer.
    2. Super-App Trust Issuer (đã xác thực người dùng là tài khoản thật qua sinh trắc học/SMS KYC) ký số lên token đã bị mù bằng khóa bí mật của mình và trả về cho Client.
    3. Client bỏ lớp làm mù toán học để nhận được một chữ ký số hợp lệ trên token ban đầu.
    4. Khi người dùng truy cập một mini-app yêu cầu độ tin cậy cao, Client gửi token chưa bị mù này cho Issuer để chuộc (Redemption). Issuer xác minh chữ ký của chính mình nhưng hoàn toàn không biết token này đã được cấp phát cho ai, vào thời điểm nào, hay trên thiết bị nào.

#### 3.2.2. Vòng đời Giao thức và Tích hợp Fetch API
- **Cam kết Khóa (Key Commitments)**:
  - Issuer duy trì endpoint công khai tại `/.well-known/pst-key-commitment` cung cấp khóa công khai và siêu dữ liệu thời hạn hiệu lực. Trình duyệt tải định kỳ và lưu vào `pstKeyCommitments`.
- **Cấp phát Thẻ (Issuance)**:
  ```javascript
  // Mini-app container kích hoạt yêu cầu cấp thẻ tin cậy từ Super-App Identity Engine
  fetch("https://trust.superapp.vn/issue-token", {
    privateToken: {
      version: 1,
      operation: "token-request"
    }
  });
  ```
  - Trình duyệt tự động gắn header `Sec-Private-State-Token` chứa danh sách token đã mù hóa.
- **Chuộc Thẻ (Redemption) & Hồ sơ Chuộc (Redemption Record)**:
  ```javascript
  // Mini-app thanh toán/săn mã giảm giá thực hiện chuộc thẻ
  fetch("https://trust.superapp.vn/redeem-token", {
    privateToken: {
      version: 1,
      operation: "token-redemption",
      refreshPolicy: "none"
    }
  });
  ```
  - Issuer phản hồi với header `Sec-Private-State-Token` chứa bản ghi chuộc thẻ (Redemption Record - RR) và `Sec-Private-State-Token-Lifetime: 86400` (hiệu lực 24 giờ).
  - Trong các yêu cầu HTTP tiếp theo đến máy chủ mini-app, trình duyệt đính kèm header `Sec-Redemption-Record`, chứng minh thiết bị đã được tin cậy mà không lộ thông tin cá nhân.

#### 3.2.3. Ranh giới Bảo mật và Giới hạn Lưu trữ
- **Giới hạn 2 Tổ chức Phát hành (2-Issuer Origin Clamping)**:
  - Để ngăn chặn các trang web độc hại câu kết với nhau tạo ra kênh truyền định danh bằng cách tích lũy nhiều thẻ từ các bên khác nhau, trình duyệt chỉ cho phép liên kết tối đa 2 tổ chức phát hành trên mỗi top-level origin.
- **Kiểm soát Bằng Permissions Policy**:
  - Hoạt động chuộc thẻ và gửi bản ghi chuộc bị kiểm soát bởi tính năng `private-state-token-redemption`. Giá trị mặc định là `self`, chặn đứng các iframe nhúng lậu quyền truy cập thẻ.
- **Dọn dẹp Đồng bộ**:
  - Khi người dùng xóa dữ liệu duyệt web (`Clear-Site-Data: "storage", "cookies"`), toàn bộ token và bản ghi chuộc trong `tokenStore` bị xóa tức thì.

---

### 3.3. On-Device Ad Auctions & Isolated Rendering (Protected Audience & Fenced Frames)

#### 3.3.1. Kiến trúc Đấu giá Quảng cáo Trên Thiết bị (Protected Audience API)
- **Nhóm Quan tâm (Interest Groups)**:
  - Khi người dùng tương tác với danh mục sản phẩm (ví dụ: mini-app bán vé máy bay), mini-app có thể thêm người dùng vào nhóm quan tâm:
    ```javascript
    await navigator.joinAdInterestGroup({
      owner: "https://travel-ads.superapp.vn",
      name: "frequent-flyer-danang",
      biddingLogicURL: "https://travel-ads.superapp.vn/bidding.js",
      dailyUpdateURL: "https://travel-ads.superapp.vn/daily-update.json",
      userBiddingSignals: { tier: "gold" }
    }, 2592000); // 30 ngày
    ```
- **Đấu giá Cục bộ (On-Device Auction Execution)**:
  - Khi một mini-app tin tức hiển thị banner quảng cáo, mini-app gọi `navigator.runAdAuction()`:
    ```javascript
    const auctionConfig = {
      seller: "https://ads-exchange.superapp.vn",
      decisionLogicURL: "https://ads-exchange.superapp.vn/decision.js",
      interestGroupBuyers: ["https://travel-ads.superapp.vn", "https://retail-ads.superapp.vn"],
      auctionSignals: { slot: "top-banner", context: "travel-news" }
    };
    const fencedFrameConfig = await navigator.runAdAuction(auctionConfig);
    ```
  - Trình duyệt tự động tải script `generateBid()` của bên mua và `scoreAd()` của bên bán vào các môi trường Worklet cô lập. Các worklet này không có quyền truy cập DOM, không có quyền gọi mạng ngoài các truy vấn tín hiệu tin cậy đã được kiểm duyệt (trusted bidding signals), triệt tiêu hoàn toàn khả năng rò rỉ dữ liệu qua mạng.

#### 3.3.2. Hiển thị Cô lập với Fenced Frames và FencedFrameConfig
- **Hạn chế của iframe truyền thống**: iframe cho phép liên lạc hai chiều qua `postMessage`, đọc kích thước, hoặc dò tìm lịch sử duyệt web thông qua timing attack.
- **Phần tử `<fencedframe>`**:
  - Không nhận trực tiếp URL quảng cáo mà nhận một đối tượng mờ `FencedFrameConfig` (hoặc định danh URN UUID nội bộ). Mã JavaScript của mini-app chứa không thể đọc được URL thực tế hay giá thầu của quảng cáo chiến thắng:
    ```html
    <fencedframe config="fencedFrameConfig" mode="opaque-ads"></fencedframe>
    ```
  - **Khóa Mạng Mật mã (`disableUntrustedNetwork`)**:
    - Sau khi nạp xong tài nguyên tĩnh ban đầu, script bên trong fenced frame có thể gọi `window.fence.disableUntrustedNetwork()`.
    - Lệnh này ngắt toàn bộ quyền truy cập mạng của frame, đảm bảo dữ liệu hiển thị không bao giờ bị tuồn ra các máy chủ bên ngoài.

#### 3.3.3. Đo lường Đoạn Đóng góp với Shared Storage và Private Aggregation
- **Shared Storage**:
  - Cung cấp kho lưu trữ khóa-giá trị liên miền (cross-site) không bị phân vùng, nhưng **không cho phép đọc trực tiếp** trong ngữ cảnh tài liệu thông thường (`window.sharedStorage.get()` bị cấm hoàn toàn).
  - Dữ liệu chỉ có thể được đọc bên trong `SharedStorageWorklet`.
- **Cổng Ra Private Aggregation**:
  - Bên trong worklet, lập trình viên đo lường số lần tiếp cận (reach) bằng cách ghi nhận đóng góp vào biểu đồ:
    ```javascript
    class ReachOperation {
      async run(data) {
        const hasReported = (await sharedStorage.get("reported-user")) === "true";
        if (!hasReported) {
          privateAggregation.contributeToHistogram({
            bucket: convertCampaignToBucket(data.campaignId),
            value: 65536 // Đóng góp toàn bộ ngân sách L1
          });
          await sharedStorage.set("reported-user", "true");
        }
      }
    }
    ```
  - Ngân sách đóng góp L1 được giới hạn nghiêm ngặt ở mức 65.536 đơn vị cho mỗi cửa sổ tính toán (10 phút hoặc 24 giờ). Nếu vượt quá ngân sách, đóng góp sẽ bị từ chối hoặc làm tràn để bảo đảm nhiễu vi phân được tính toán chính xác.

---

## 4. Khung Ma trận Kiểm soát và Khuyến nghị Triển khai Super-App

| Hạng mục | Mức độ Bắt buộc | Cơ chế Kỹ thuật | Rủi ro nếu không áp dụng |
|---|---|---|---|
| **Attribution Gateway Signing** | Bắt buộc (Regulated & Standard) | Ký số lượt nhấp bằng khóa bất đối xứng host, kiểm tra domain allowlist | Gian lận lượt nhấp (click injection), tranh chấp doanh thu affiliate |
| **Privacy-Preserving Conversion** | Bắt buộc (Production) | WICG Attribution Reporting API với $\epsilon$-DP randomized response | Vi phạm luật bảo vệ dữ liệu cá nhân, bị App Store/Google Play gỡ app do vi phạm tracking |
| **TEE Summary Aggregation** | Khuyến nghị cho Enterprise Scale | Triển khai Aggregation Service trên AWS Nitro Enclaves / GCP Confidential Space | Lộ dữ liệu giỏ hàng và thói quen tiêu dùng chi tiết của người dùng |
| **Anti-Abuse Trust Mesh (PST)** | Bắt buộc cho giao dịch nhạy cảm | Cấp phát thẻ Privacy Pass RFC 9576-9578 sau khi xác thực KYC | Bị tấn công bot càn quét voucher, tạo tài khoản giả mạo (Sybil attack) |
| **2-Issuer Clamping & Permissions Policy** | Bắt buộc (Core Sandbox) | Trình duyệt/Container giới hạn 2 issuers trên mỗi top-level origin | Kẻ xấu liên kết thẻ giữa nhiều bên để giải mã danh tính người dùng |
| **On-Device Ad Auction Worklets** | Bắt buộc khi có sàn quảng cáo | Protected Audience `generateBid`/`scoreAd` chạy trong worklet không mạng | Rò rỉ lịch sử duyệt hàng của người dùng sang các mạng quảng cáo bên ngoài |
| **Fenced Frame Opaque Rendering** | Bắt buộc cho banner bên thứ ba | Render qua `<fencedframe>` với `FencedFrameConfig` và chặn liên lạc chéo | Tấn công clickjacking, đánh cắp token phiên, UI redressing |
| **Shared Storage L1 Budgeting** | Bắt buộc cho đo lường tần suất | Áp giới hạn 65.536 units và mã hóa báo cáo bằng HPKE | Rò rỉ dữ liệu người dùng qua các truy vấn vi mô lặp lại |

---

## 5. Kết luận và Kế hoạch Tiếp theo

Iteration 35 đã hoàn thành nghiên cứu toàn diện về hạ tầng quyền riêng tư, đo lường chuyển đổi an toàn và chống gian lận phi tập trung cho super-app. Hệ thống hiện đã tích lũy **476 findings** được chuẩn hóa và kiểm chứng nghiêm ngặt.

Do Iteration 35 là mốc bội số của 5 ($35 = 5 \times 7$), toàn bộ báo cáo, tệp bằng chứng `findings.jsonl` và bản tóm tắt kiểm định `validation-summary.json` sẽ được đồng bộ hóa và phát hành lên public repository `new-superapp` theo đúng quy trình sanitized commit & push.
