# Chuyên đề Iteration 54: Chuẩn Capability Delegation, WebAuthn Signal API, Passkey Endpoints & WICG Web Install

## 1. Tổng quan điều hành (Executive Summary)

Iteration 54 hoàn thiện nghiên cứu và chuẩn hóa 3 trụ cột kỹ thuật nền tảng cho runtime mini-app, kiến trúc định danh sinh trắc học và cơ chế phân phối/cài đặt ứng dụng trên Super-App:

1. **WICG Capability Delegation & User Activation APIs**:
   - Cơ chế ủy quyền quyền năng động (dynamic delegation) có giới hạn thời gian (time-constrained) thông qua `postMessage()` thay thế cho các thuộc tính iframe tĩnh (`allow` attribute).
   - Kiểm soát nghiêm ngặt kích hoạt người dùng nhất thời (*transient user activation*): việc ủy quyền tiêu thụ activation của bên gửi (*activation consumption*) và cấm tuyệt đối wildcard `targetOrigin: '*'`.
   - Bảng định tuyến nội bộ `DELEGATED_CAPABILITY_TIMESTAMPS` quản lý vòng đời ủy quyền các API nhạy cảm: `payment` (`PaymentRequest.show()`), `fullscreen` (`requestFullscreen()`), và `display-capture` (`mediaDevices.getDisplayMedia()`).
   - Giao diện `navigator.userActivation` kiểm tra trạng thái kích hoạt nhất thời (`isActive`) và kích hoạt vĩnh viễn (`hasBeenActive`) trong Host Bridge, chống clickjacking và lạm dụng iframe lồng nhau.

2. **WebAuthn Signal API & W3C Passkey Endpoints Discovery**:
   - Kênh đồng bộ trạng thái hai chiều giữa máy chủ bên xác thực (Relying Party - RP) và kho lưu trữ khóa xác thực phần cứng/đám mây (Authenticators / Cloud Passkey Vaults).
   - Phương thức tĩnh `PublicKeyCredential.signalUnknownCredential()` giúp xóa bỏ passkey mồ côi (orphaned passkeys) ngay sau khi xác thực thất bại, an toàn khi gọi trong bối cảnh chưa xác thực (unauthenticated).
   - Phương thức tĩnh `PublicKeyCredential.signalAllAcceptedCredentials()` đối soát toàn diện danh sách passkey hợp lệ và áp dụng cơ chế ẩn tạm thời (*soft-hiding*) chống mất khóa ngoài ý muốn.
   - Phương thức tĩnh `PublicKeyCredential.signalCurrentUserDetails()` cập nhật nhãn tài khoản, tên đăng nhập và tên hiển thị đồng bộ trên UI autofill.
   - Chuẩn W3C Passkey Endpoints (RFC 8615 `/.well-known/passkey-endpoints`) công bố siêu dữ liệu JSON cho các luồng `enroll`, `manage`, và `prfUsageDetails`.

3. **WICG Web Install API & Related Apps Verification**:
   - Khung cài đặt ứng dụng web thống nhất cung cấp hai điểm truy cập: giao diện lập trình mệnh lệnh `navigator.install()` và thẻ điều khiển HTML khai báo `<install>` (`HTMLInstallElement`).
   - Ràng buộc kích hoạt người dùng nhất thời, phân giải URL manifest, xác thực chính sách truy cập mạng cục bộ và đối soát kết quả `WebInstallResult` / `DOMException`.
   - Khẳng định định danh Manifest (`manifestId`) và cách ly cài đặt liên nguồn (cross-origin isolation) chống tấn công dò quét ứng dụng (*device app probing*) và thu thập vân tay thiết bị.
   - Chuẩn WICG Get Installed Related Apps API (`navigator.getInstalledRelatedApps()`) với cơ chế xác thực hai chiều (*bidirectional cryptographic association*) bảo vệ quyền riêng tư người dùng.

---

## 2. Bảng đối soát tiêu chuẩn & mức độ bằng chứng (Evidence Matrix)

| ID Finding | Chủ đề kỹ thuật | Tiêu chuẩn tham chiếu | Mức độ bằng chứng | URLs kiểm tra thực tế (HTTP 200) |
|---|---|---|---|---|
| `cap_del_054_01` | WICG Capability Delegation Architecture | WICG Capability Delegation §1–2 | `official_standard` | [WICG Capability Delegation](https://wicg.github.io/capability-delegation/), [Spec](https://wicg.github.io/capability-delegation/spec.html) |
| `cap_del_054_02` | Capability Delegation Initiation & Consumption | WICG Capability Delegation §3 | `official_standard` | [Capability Delegation Spec §3](https://wicg.github.io/capability-delegation/spec.html), [MDN postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) |
| `cap_del_054_03` | Delegated Capabilities Taxonomy & Timestamps | WICG Capability Delegation §4–5 | `official_standard` | [Capability Delegation Spec §4-5](https://wicg.github.io/capability-delegation/spec.html), [MDN UserActivation](https://developer.mozilla.org/en-US/docs/Web/API/UserActivation) |
| `cap_del_054_04` | UserActivation API (isActive vs hasBeenActive) | HTML Living Standard §7.5.3 / MDN | `official_standard` | [MDN UserActivation](https://developer.mozilla.org/en-US/docs/Web/API/UserActivation), [MDN Navigator.userActivation](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/userActivation) |
| `cap_del_054_05` | Container Capability Delegation Governance | WICG Spec / OWASP MASVS | `official_standard` | [WICG Capability Delegation](https://wicg.github.io/capability-delegation/), [OWASP MASVS](https://mas.owasp.org/MASVS/) |
| `passkey_sync_054_01` | WebAuthn Signal API Architecture | W3C WebAuthn WG / Chrome Identity | `official_standard` | [Chrome WebAuthn Signal API](https://developer.chrome.com/docs/identity/webauthn-signal-api), [W3C WebAuthn](https://w3c.github.io/webauthn/) |
| `passkey_sync_054_02` | PublicKeyCredential.signalUnknownCredential() | MDN Web APIs / W3C WebAuthn | `official_standard` | [MDN signalUnknownCredential](https://developer.mozilla.org/en-US/docs/Web/API/PublicKeyCredential/signalUnknownCredential_static), [Chrome Docs](https://developer.chrome.com/docs/identity/webauthn-signal-api) |
| `passkey_sync_054_03` | PublicKeyCredential.signalAllAcceptedCredentials() | MDN Web APIs / W3C WebAuthn | `official_standard` | [MDN signalAllAcceptedCredentials](https://developer.mozilla.org/en-US/docs/Web/API/PublicKeyCredential/signalAllAcceptedCredentials_static), [MDN Passkeys](https://developer.mozilla.org/en-US/docs/Web/Security/Authentication/Passkeys) |
| `passkey_sync_054_04` | PublicKeyCredential.signalCurrentUserDetails() | MDN Web APIs / W3C WebAuthn | `official_standard` | [MDN signalCurrentUserDetails](https://developer.mozilla.org/en-US/docs/Web/API/PublicKeyCredential/signalCurrentUserDetails_static), [Chrome Docs](https://developer.chrome.com/docs/identity/webauthn-signal-api) |
| `passkey_sync_054_05` | W3C Passkey Endpoints (RFC 8615) | W3C Passkey Endpoints §3 | `official_standard` | [W3C Passkey Endpoints](https://w3c.github.io/webappsec-passkey-endpoints/), [W3C WebAuthn](https://w3c.github.io/webauthn/) |
| `web_install_054_01` | WICG Web Install Architecture | WICG Web Install §1–2 | `official_standard` | [WICG Web Install](https://wicg.github.io/install-element/), [Chrome Origin Trial](https://developer.chrome.com/blog/install-element-ot) |
| `web_install_054_02` | Navigator.install() Imperative API | WICG Web Install §3 | `official_standard` | [WICG Web Install §3](https://wicg.github.io/install-element/), [MDN UserActivation](https://developer.mozilla.org/en-US/docs/Web/API/UserActivation) |
| `web_install_054_03` | HTML `<install>` Element (HTMLInstallElement) | WICG Web Install §4 | `official_standard` | [WICG Web Install §4](https://wicg.github.io/install-element/), [GitHub install-element](https://github.com/WICG/install-element) |
| `web_install_054_04` | Manifest ID Assertions & Anti-Fingerprinting | WICG Web Install §5 | `official_standard` | [WICG Web Install §5](https://wicg.github.io/install-element/), [WICG Manifest Incubations](https://wicg.github.io/manifest-incubations/) |
| `web_install_054_05` | WICG Get Installed Related Apps API | WICG Get Installed Related Apps §4–5 | `official_standard` | [WICG Get Installed Related Apps](https://wicg.github.io/get-installed-related-apps/spec/), [MDN getInstalledRelatedApps](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/getInstalledRelatedApps) |

---

## 3. Kiến trúc chi tiết & Hướng dẫn triển khai cho Super-App

### 3.1. WICG Capability Delegation & User Activation

#### Cơ chế hoạt động của Capability Delegation
Trong mô hình web truyền thống, một frame con (iframe) muốn sử dụng các API đặc quyền (như thanh toán, toàn màn hình, ghi màn hình) phải nhận quyền tĩnh thông qua thuộc tính `allow` (ví dụ: `<iframe allow="payment; fullscreen">`). Tuy nhiên, quyền tĩnh này mở ra lỗ hổng clickjacking hoặc tấn công tự động kích hoạt API khi người dùng không chủ ý tương tác với iframe đó.

WICG Capability Delegation khắc phục bằng cơ chế ủy quyền quyền theo thời gian thực (dynamic, time-constrained delegation):
1. Frame cha (parent frame) nhận được tương tác vật lý thực tế từ người dùng (click, tap).
2. Frame cha gửi `postMessage` sang frame con tin cậy với tùy chọn `delegate: "payment"` (hoặc `"fullscreen"`, `"display-capture"`).
3. Quá trình gửi này **ngay lập tức tiêu thụ kích hoạt của frame cha** (`transient activation consumption`). Cấm gửi tới wildcard `targetOrigin: '*'`.
4. Frame con nhận được quyền trong một khoảng thời gian ngắn (thường từ 3 đến 5 giây, theo `DELEGATED_CAPABILITY_TIMESTAMPS`). Khi frame con gọi `PaymentRequest.show()`, quyền lập tức được sử dụng và xóa bỏ khỏi bảng timestamp.

```javascript
// [Frame Cha - Super App Host / Merchant Mini App]
async function delegatePaymentToCheckoutFrame(checkoutIframe, targetOrigin) {
  if (!navigator.userActivation.isActive) {
    console.error("Không thể ủy quyền: Cần tương tác người dùng nhất thời.");
    return;
  }

  // Tiêu thụ activation và chuyển giao quyền thanh toán cho origin được chỉ định rõ
  checkoutIframe.contentWindow.postMessage({
    action: "INITIATE_CHECKOUT",
    orderId: "ORDER_987654",
    amount: "150000"
  }, {
    targetOrigin: targetOrigin, // Bắt buộc: KHÔNG được dùng '*'
    delegate: "payment"        // Feature identifier được chuẩn hóa
  });
}

// [Frame Con - Banking Sub-App / Checkout Widget]
window.addEventListener("message", async (event) => {
  if (event.origin !== "https://trusted-superapp.com") return;

  if (event.data.action === "INITIATE_CHECKOUT") {
    const paymentDetails = {
      total: { label: "Tổng tiền", amount: { currency: "VND", value: event.data.amount } }
    };
    const request = new PaymentRequest([{ supportedMethods: "https://trusted-superapp.com/pay" }], paymentDetails);

    try {
      // Nhờ capability delegation, phương thức show() thành công dù frame con chưa nhận click trực tiếp
      const response = await request.show();
      await response.complete("success");
    } catch (err) {
      console.error("Lỗi thực thi PaymentRequest:", err);
    }
  }
});
```

#### Bảng trạng thái UserActivation
- `navigator.userActivation.isActive`: Kích hoạt nhất thời (transient activation). Trả về `true` ngay sau cử chỉ bấm/chạm và trở về `false` sau vài giây hoặc sau khi bị một API tiêu thụ (như `postMessage` có `delegate` hoặc mở popup).
- `navigator.userActivation.hasBeenActive`: Kích hoạt vĩnh viễn (sticky activation). Trả về `true` nếu người dùng từng tương tác với tài liệu ít nhất một lần kể từ khi mở trang.

---

### 3.2. WebAuthn Signal API & Passkey Endpoints

#### Đồng bộ trạng thái Passkey hai chiều
Một vấn đề cố hữu trong việc triển khai Passkey là sự lệch pha giữa máy chủ Relying Party và Authenticator của người dùng. Khi người dùng xóa passkey trên trang web từ thiết bị A, Authenticator trên thiết bị B vẫn hiển thị passkey đó trong menu gợi ý đăng nhập tự động (Conditional Mediation Autofill). Khi người dùng nhấn vào passkey mồ côi đó, xác thực thất bại gây trải nghiệm rất xấu.

WebAuthn Signal API giải quyết triệt để vấn đề này với 3 API tĩnh:

```javascript
// 1. Dọn dẹp passkey không tồn tại sau khi xác thực thất bại (An toàn khi chưa đăng nhập)
async function handleAssertionFailure(rejectedCredId, rpDomain) {
  if (window.PublicKeyCredential && PublicKeyCredential.signalUnknownCredential) {
    await PublicKeyCredential.signalUnknownCredential({
      rpId: rpDomain,
      credentialId: rejectedCredId // Chuỗi base64url của credential không hợp lệ
    });
    console.log("Đã phát tín hiệu xóa passkey mồ côi khỏi Authenticator");
  }
}

// 2. Đối soát toàn diện danh sách passkey sau khi người dùng đăng nhập thành công
async function reconcileUserPasskeys(rpDomain, userAccount) {
  if (window.PublicKeyCredential && PublicKeyCredential.signalAllAcceptedCredentials) {
    await PublicKeyCredential.signalAllAcceptedCredentials({
      rpId: rpDomain,
      userId: userAccount.base64UserId,
      allAcceptedCredentialIds: userAccount.validCredentialIdList // Danh sách các ID hợp lệ
    });
    // Các passkey không nằm trong danh sách sẽ được authenticator chuyển sang trạng thái "hidden"
  }
}

// 3. Đồng bộ thông tin tên hiển thị khi người dùng đổi profile
async function syncUserProfile(rpDomain, userAccount) {
  if (window.PublicKeyCredential && PublicKeyCredential.signalCurrentUserDetails) {
    await PublicKeyCredential.signalCurrentUserDetails({
      rpId: rpDomain,
      userId: userAccount.base64UserId,
      name: userAccount.email,
      displayName: userAccount.fullName
    });
  }
}
```

#### Cấu hình W3C Passkey Endpoints (`/.well-known/passkey-endpoints`)
Tất cả mini-app hỗ trợ Passkey phải triển khai file cấu hình tĩnh tại đường dẫn gốc:

```json
{
  "enroll": "https://auth.miniapp.com/passkeys/new",
  "manage": "https://auth.miniapp.com/settings/security/passkeys",
  "prfUsageDetails": "https://miniapp.com/docs/security/local-vault-encryption"
}
```

---

### 3.3. WICG Web Install API & Related Apps Verification

#### Luồng cài đặt Declarative vs Imperative
WICG Web Install thay thế cơ chế đón bắt sự kiện `beforeinstallprompt` không đồng nhất giữa các trình duyệt bằng cơ chế cài đặt chuẩn mực:

1. **Khởi tạo Mệnh lệnh (`navigator.install()`)**:
```javascript
async function installMiniApp(manifestUrl, manifestIdAssertion) {
  if (!navigator.userActivation.isActive) {
    throw new Error("Yêu cầu tương tác người dùng để cài đặt mini app.");
  }

  try {
    const result = await navigator.install({
      manifest: manifestUrl,
      manifestId: manifestIdAssertion
    });
    console.log("Cài đặt thành công mini app:", result);
  } catch (error) {
    if (error.name === "AbortError") {
      console.log("Người dùng hủy hộp thoại cài đặt.");
    } else if (error.name === "DataError") {
      console.error("Dữ liệu manifest không hợp lệ.");
    }
  }
}
```

2. **Khởi tạo Khai báo (`<install>` HTML Element)**:
Thẻ `<install>` do chính User Agent / Super-App Container vẽ (*User-Agent rendered control*). Nó đảm bảo chống giả mạo giao diện, ngăn chặn tấn công phủ mờ (`opacity: 0`) và đảm bảo người dùng chủ ý cài đặt:

```html
<!-- Nút cài đặt an toàn do Host Container kiểm soát hiển thị -->
<install manifest="/mini/fintech/manifest.webmanifest"
         manifestid="https://fintech.example.com/app"
         oninstallresult="handleInstallResult(event)">
</install>
```

3. **Bảo vệ quyền riêng tư qua WICG Get Installed Related Apps**:
```javascript
const relatedApps = await navigator.getInstalledRelatedApps();
relatedApps.forEach((app) => {
  console.log("Ứng dụng liên kết đã cài:", app.platform, app.id, app.version);
});
```
Yêu cầu bắt buộc: Phải có liên kết xác thực hai chiều (*bidirectional association*) giữa Web App Manifest (`related_applications`) và Native App Asset Links / Apple App Site Association. Nếu thiếu liên kết hai chiều, trình duyệt trả về mảng rỗng để chống thu thập vân tay danh mục phần mềm của người dùng.

---

## 4. Danh mục Tiêu chuẩn Kiểm định (Verification Checklist)

1. [x] **Capability Delegation**: Giao tiếp giữa frame cha và các widget thanh toán/fullscreen tuân thủ tiêu chuẩn WICG Capability Delegation, tiêu thụ transient user activation và chỉ định cụ thể `targetOrigin`.
2. [x] **Activation Gating**: Mọi thao tác gọi API bảo mật cao (Payment, Biometrics, Web Install) trong Host Bridge đều kiểm tra `navigator.userActivation.isActive === true`.
3. [x] **Passkey Cleanup**: Luồng đăng nhập bằng Passkey trên mini-app tích hợp `signalUnknownCredential()` để tự động dọn dẹp các khóa hỏng/đã bị xóa trên server.
4. [x] **Passkey Reconciliation**: Quản lý tài khoản người dùng tích hợp `signalAllAcceptedCredentials()` và `signalCurrentUserDetails()`, tránh rò rỉ credential IDs khi chưa xác thực.
5. [x] **Passkey Endpoints**: Tệp `/.well-known/passkey-endpoints` được triển khai đúng định dạng JSON không chuyển hướng (HTTP 200) cho các Relying Party mini-app.
6. [x] **Web Install**: Giao diện kho ứng dụng hỗ trợ cả `navigator.install()` có kiểm tra `manifestId` và thẻ khai báo `<install>`, loại bỏ hành vi dò quét ứng dụng cross-origin.
7. [x] **Related Apps Isolation**: Kiểm tra liên kết hai chiều trước khi cho phép mini-app kiểm tra sự hiện diện của ứng dụng native trên máy.

---

## 5. Kết luận & Kế hoạch Iteration kế tiếp

Iteration 54 đã mở rộng và chuẩn hóa toàn diện 15 findings mới, nâng tổng số findings chuẩn mực trong kho dữ liệu `state/findings.jsonl` lên **761 findings**.

Kế hoạch cho **Iteration 55**:
- Đây là cột mốc **bội số của 5 (Iteration 55 Milestone)** theo quy định của Deli Deep Protocol.
- Nhiệm vụ trọng tâm: Triển khai kiểm tra toàn diện, chạy rà soát secret scan trên kho mã nguồn sanitized `/home/minhgv/code/github/new-superapp`, đồng bộ hóa các tệp báo cáo chuyên đề và bằng chứng (`evidence/findings.jsonl`, `evidence/validation-summary.json`), chạy `git diff --cached --check`, tạo commit conventional và thực hiện push an toàn lên remote `origin main`.
