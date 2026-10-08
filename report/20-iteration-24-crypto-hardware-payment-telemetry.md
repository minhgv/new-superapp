# Iteration 24: Cryptographic Hardware Security, Client-Side Payment Mediation & Runtime Telemetry Reporting

## Overview and Scope

Iteration 24 establishes the normative and platform security boundaries for three critical mini-app runtime tiers:
1. **Cryptographic Key Isolation & Hardware Security Module Binding**: Integrating WebCrypto API, Android Keystore StrongBox/KeyMint, Apple Secure Enclave Keychain services, and OWASP MASVS-CRYPTO standards to protect mini-app signing and encryption keys from exfiltration.
2. **Client-Side Payment Mediation & Biometrically Authenticated Checkout**: Specifying host mediation via the W3C Payment Request API, iframe Permissions Policy `payment` restrictions, W3C Secure Payment Confirmation (SPC) order/payee binding, and W3C Payment Method Manifest origin controls.
3. **Client Runtime Telemetry, Responsiveness & Network Error Reporting**: Instrumenting Core Web Vitals and failure observability via W3C Paint Timing (FP/FCP), Largest Contentful Paint (LCP), Event Timing (INP), the W3C Reporting API, and Network Error Logging (NEL).

---

## Detailed Findings and Normative Controls

### 1. Cryptographic Key Isolation and Hardware Security (MASVS-CRYPTO, WebCrypto, Keystore, Secure Enclave)

#### `crypto-hw-webcrypto-001`: Treat WebCrypto as in-memory cryptography; require host hardware key protection for long-lived credentials
- **Standard Reference**: W3C Web Cryptography API §14 (SubtleCrypto), §14.3.1 (generateKey), §14.3.8-14.3.9 (importKey/exportKey), §18 (Security Considerations)
- **Primary URL**: https://www.w3.org/TR/WebCryptoAPI/
- **Evidence Level**: `normative_standard`
- **[SPEC FACT]**: `SubtleCrypto.generateKey()` and `importKey()` enforce an `extractable` boolean flag. When `extractable: false`, the private key material cannot be exported through `exportKey()` or `wrapKey()`. However, the WebCrypto specification explicitly warns that WebCrypto operates inside the user-agent renderer/JavaScript origin context. Keys marked non-extractable are held in renderer process memory and can be used for arbitrary cryptographic operations by any script executing in that origin. Furthermore, IndexedDB key storage remains vulnerable to renderer memory scraping, local extraction, and origin tampering.
- **[SUPER-APP CONTROL]**: Super-app runtimes must restrict browser-level WebCrypto usage to ephemeral operations (e.g. transient session payload encryption). High-value cryptographic identities (publisher credentials, transaction signing keys, merchant certificates) must NOT rely solely on browser IndexedDB/WebCrypto. The super-app host must expose a hardware-backed JSAPI bridge that delegates key generation, storage, and signing to native platform security hardware (Android Keystore / iOS Secure Enclave).

#### `crypto-hw-android-strongbox-002`: Enforce StrongBox KeyMint isolation and server-side hardware attestation
- **Standard Reference**: Android Keystore Cryptography & KeyGenParameterSpec API, Android Hardware-Backed Keystore & Key Attestation
- **Primary URL**: https://developer.android.com/privacy-and-security/cryptography
- **Supporting URLs**: https://developer.android.com/privacy-and-security/keystore, https://developer.android.com/privacy-and-security/security-key-attestation, https://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec
- **Evidence Level**: `platform_specification`
- **[SPEC FACT]**: Android Keystore provides hardware-backed key protection via Trusted Execution Environments (TEE) and dedicated Secure Elements (`setIsStrongBoxBacked(true)` targeting StrongBox KeyMint). Keys generated with `KeyGenParameterSpec` are non-exportable and bound to device hardware. Android provides cryptographic Key Attestation: the Keystore generates an X.509 certificate chain rooted in the Google hardware attestation root, containing DER-encoded ASN.1 extension sequences certifying that the key is hardware-backed, StrongBox-isolated, and restricted by specific authorization tags (e.g., user authentication required, validity windows).
- **[SUPER-APP CONTROL]**: For regulated mini-apps (fintech, banking, digital signatures), the super-app host container on Android must generate asymmetric key pairs with `setIsStrongBoxBacked(true)` (falling back to TEE only if StrongBox is unavailable) and `setUserAuthenticationRequired(true)`. The host must retrieve the attestation certificate chain and verify it on the super-app server against Google's Root CA before provisioning enterprise identities. Mini-app scripts must never receive raw private keys.

#### `crypto-hw-apple-secureenclave-003`: Protect mini-app signing identities via Apple Secure Enclave and Keychain Access Controls
- **Standard Reference**: Apple Developer Documentation: Storing Keys in the Secure Enclave, Protecting Keys with the Secure Enclave, SecAccessControlCreateFlags
- **Primary URL**: https://developer.apple.com/documentation/security/certificate_key_and_trust_services/keys/storing_keys_in_the_secure_enclave
- **Supporting URLs**: https://developer.apple.com/documentation/security/protecting-keys-with-the-secure-enclave.md, https://developer.apple.com/documentation/security/SecAccessControlCreateFlags
- **Evidence Level**: `platform_specification`
- **[SPEC FACT]**: Apple Secure Enclave is a hardware-isolated coprocessor. Elliptic curve private keys (NIST P-256) generated with `kSecAttrTokenIDSecureEnclave` are created inside the Secure Enclave, never leave its memory boundaries, and are inaccessible to the iOS kernel or user-space processes. Access controls (`SecAccessControlCreateFlags`) enforce hardware-level gates such as `.biometryAny`, `.biometryCurrentSet` (invalidating keys when biometrics change), or `.userPresence`. Signing operations occur exclusively within the coprocessor.
- **[SUPER-APP CONTROL]**: On iOS, the super-app host must store high-assurance mini-app credentials in the Keychain backed by the Secure Enclave with `kSecAccessControlBiometryCurrentSet | kSecAccessControlPrivateKeyUsage`. If a device's biometric configuration changes (e.g. new fingerprint/FaceID enrolled), the key is invalidated by hardware, protecting the host against unauthorized access. Mini-apps trigger signing via host-mediated bridge requests.

#### `crypto-hw-masvs-crypto-1-004`: Enforce up-to-date and proven cryptographic algorithms across host and mini-app boundaries
- **Standard Reference**: OWASP Mobile Application Security Verification Standard (MASVS) v2.0.0, Control MASVS-CRYPTO-1
- **Primary URL**: https://mas.owasp.org/MASVS/controls/MASVS-CRYPTO-1/
- **Supporting URLs**: https://mas.owasp.org/MASVS/, https://mas.owasp.org/MASTG/
- **Evidence Level**: `security_standard`
- **[SPEC FACT]**: OWASP MASVS-CRYPTO-1 mandates that applications use only industry-standard, proven cryptographic algorithms, modes, and parameters that are not deprecated. Custom cryptographic implementations, weak symmetric algorithms (DES, 3DES, RC4), broken hashing functions (MD5, SHA-1 for signatures), insecure block cipher modes (ECB, static IVs in CBC), and weak key sizes (<2048-bit RSA, <256-bit ECC) are strictly prohibited.
- **[SUPER-APP CONTROL]**: The mini-app review engine and host SDK must ban non-standard or obsolete cryptographic primitives. Automated static analysis during package submission must flag deprecated cipher suites. The host bridge must expose only hardened primitives: AES-256-GCM, ChaCha20-Poly1305, Ed25519/ECDSA P-256, and SHA-256/SHA-512.

#### `crypto-hw-masvs-crypto-2-005`: Enforce secure cryptographic key lifecycle management and hardware storage
- **Standard Reference**: OWASP Mobile Application Security Verification Standard (MASVS) v2.0.0, Control MASVS-CRYPTO-2
- **Primary URL**: https://mas.owasp.org/MASVS/controls/MASVS-CRYPTO-2/
- **Supporting URLs**: https://mas.owasp.org/MASVS/, https://mas.owasp.org/MASTG/
- **Evidence Level**: `security_standard`
- **[SPEC FACT]**: OWASP MASVS-CRYPTO-2 requires that cryptographic keys are managed securely throughout their entire lifecycle (generation, storage, usage, derivation, destruction). Hardcoding keys in application code, storing plaintext keys in shared preferences/local files, or transmitting unprotected keys over IPC constitutes critical security defects. Keys must be stored in hardware-backed storage facilities provided by the platform.
- **[SUPER-APP CONTROL]**: Super-app package validation must perform AST and byte-level scanning for hardcoded symmetric keys, private key PEMs, and static tokens. The host container must isolate mini-app key stores per mini-app ID (`appId`), ensuring that mini-app A cannot read or invoke keys owned by mini-app B. Keys must be zeroized upon mini-app uninstallation or container data wiping.

---

### 2. Client-Side Payment Mediation and Authenticated Checkout (W3C Payment Request, SPC, Payment Method Manifest)

#### `payment-request-core-001`: Treat PaymentRequest as user-agent mediation, not authorization
- **Standard Reference**: W3C Payment Request API §§3.3-3.5, 3.4, 14.1, 14.10, 19.1-19.3
- **Primary URL**: https://www.w3.org/TR/payment-request/
- **Evidence Level**: `normative_standard`
- **[SPEC FACT]**: W3C Payment Request API places the user agent between the payee, payer, and payment method. `show()` initiates user interaction, returns a promise resolved when the user accepts, selects registered payment handlers, and rejects with `NotSupportedError` when no handler is available. The API is strictly restricted to secure contexts (`https://`). `canMakePayment()` queries support for a method, but a `true` return does not guarantee an active provisioned instrument. After user acceptance, a `PaymentResponse` is returned: `complete()` signals the outcome, and `retry()` allows user correction while requiring merchant re-validation.
- **[SUPER-APP CONTROL]**: The super-app checkout broker must never treat `PaymentRequest.show()` resolution as proof of payment authorization. The mini-app must submit an order intent to the host backend; the host broker creates the payment request with a server-signed order snapshot (amount, currency, merchant ID, nonce, expiry). Backend authorization with the payment gateway/PSP must succeed before `response.complete("success")` is called.

#### `payment-request-iframe-002`: Gate cross-origin mini-app payment frames with the payment Permissions Policy
- **Standard Reference**: W3C Payment Request API §§2.10, 16, 19.2-19.3
- **Primary URL**: https://w3c.github.io/payment-request/
- **Supporting URLs**: https://www.w3.org/TR/payment-request/
- **Evidence Level**: `normative_standard`
- **[SPEC FACT]**: Payment Request defines `payment` as a policy-controlled feature with a default allowlist of `self`. Cross-origin iframes require an explicit feature delegation: `allow="payment"`. Documents lacking this policy cannot construct a `PaymentRequest`. Furthermore, payment handlers inspect both the top-level origin and the specific initiator iframe origin.
- **[SUPER-APP CONTROL]**: The super-app container shell must enforce origin-scoped Permissions Policies. Payment capabilities must be denied by default to mini-app iframes, marketing banners, and third-party webviews. Only verified merchant checkout frames matching declared store origins may receive the `payment` feature delegation.

#### `payment-spc-auth-003`: Use Secure Payment Confirmation (SPC) to cryptographically bind user verification to order and payee
- **Standard Reference**: W3C Secure Payment Confirmation §§4.2-4.11, 7.1, 8-9, 11.1, 12.1
- **Primary URL**: https://www.w3.org/TR/secure-payment-confirmation/
- **Supporting URLs**: https://www.w3.org/TR/payment-request/
- **Evidence Level**: `normative_standard`
- **[SPEC FACT]**: W3C Secure Payment Confirmation (SPC) defines the payment method identifier `secure-payment-confirmation`. It binds WebAuthn credentials to transaction details (`PaymentCredentialInstrument`), displaying payee name/origin, currency, total amount, and transaction icons within a trusted browser UI. The resulting assertion is cryptographically signed over the transaction challenge and payee origin.
- **[SUPER-APP CONTROL]**: Super-app checkout flows should integrate SPC for high-value transactions and 3DS compliance. The super-app backend generates the cryptographic challenge bound to the specific order ID and merchant origin. The mini-app host container presents the trusted SPC confirmation sheet. The backend verifies the signature, challenge, and payee origin against the order ledger before committing funds.

#### `payment-method-manifest-004`: Enforce Payment Method Manifest discovery and origin allowlisting for wallet apps
- **Standard Reference**: W3C Payment Method Manifest §§1.2-1.3, 2, 3.2-3.5, 4
- **Primary URL**: https://www.w3.org/TR/payment-method-manifest/
- **Supporting URLs**: https://www.w3.org/TR/payment-request/
- **Evidence Level**: `normative_standard`
- **[SPEC FACT]**: URL-based payment method identifiers point to a machine-readable manifest via an HTTP `Link: <manifest>; rel="payment-method-manifest"` header. The manifest defines `default_applications` and `supported_origins`. The user agent fetches the manifest without credentials, validates HTTPS origins, and verifies that calling payment applications match allowed origins. Wildcard origins (`"*"`) open payment handling to arbitrary sites.
- **[SUPER-APP CONTROL]**: Super-app payment gateways must publish signed payment method manifests from trusted HTTPS endpoints. Wildcards in `supported_origins` must be forbidden in mini-app store governance. The super-app host verifies manifest signatures and enforces strict origin matching before delegating payment transactions to third-party digital wallets or payment mini-apps.

---

### 3. Client Runtime Telemetry, Responsiveness & Network Error Reporting (Paint Timing, LCP, INP, Reporting API, NEL)

#### `telemetry-paint-timing-001`: Capture First Paint (FP) and First Contentful Paint (FCP) via PerformanceObserver
- **Standard Reference**: W3C Paint Timing §3 (PerformancePaintTiming), §4 (Processing Model)
- **Primary URL**: https://www.w3.org/TR/paint-timing/
- **Evidence Level**: `normative_standard`
- **[SPEC FACT]**: W3C Paint Timing standardizes `PerformancePaintTiming` records for `first-paint` and `first-contentful-paint`. These entries record the exact monotonically increasing DOMHighResTimeStamp when the browser first renders any pixels or first renders DOM content (text, image, non-white canvas). Observers register via `PerformanceObserver` with `buffered: true` to reliably capture metrics even if registered after initial rendering.
- **[SUPER-APP CONTROL]**: The super-app webview container must inject an early performance observer to collect FP and FCP metrics for every mini-app launch. These metrics provide objective startup latency telemetry. The store review pipeline and runtime guardian must enforce SLO thresholds (e.g. FCP < 1.2s on 4G) to prevent unresponsive mini-apps from degrading super-app UX.

#### `telemetry-lcp-metric-002`: Monitor Largest Contentful Paint (LCP) candidates and dispatch finalized render timestamps
- **Standard Reference**: W3C Largest Contentful Paint §3 (LargestContentfulPaint), §4 (Processing Model)
- **Primary URL**: https://www.w3.org/TR/largest-contentful-paint/
- **Evidence Level**: `normative_standard`
- **[SPEC FACT]**: W3C Largest Contentful Paint tracks the largest visual element (image or text block) rendered in the viewport. The user agent emits multiple candidate `LargestContentfulPaint` entries as larger elements render, stopping candidate emission upon the first user interaction (key down, click, scroll). The metric records `renderTime` (or `loadTime` if cross-origin timing is withheld without `Timing-Allow-Origin`).
- **[SUPER-APP CONTROL]**: The host container monitors LCP candidate streams and records the final candidate prior to user interaction. Cross-origin assets loaded by mini-apps must include `Timing-Allow-Origin` to avoid inaccurate render timing. LCP metrics must be fed into the store quality score, influencing search ranking and alerting operators to slow mini-app asset bundles.

#### `telemetry-event-timing-inp-003`: Measure Interaction to Next Paint (INP) and input processing delays
- **Standard Reference**: W3C Event Timing §3 (PerformanceEventTiming), §4 (Processing Model)
- **Primary URL**: https://www.w3.org/TR/event-timing/
- **Evidence Level**: `normative_standard`
- **[SPEC FACT]**: W3C Event Timing measures user interaction responsiveness by recording `PerformanceEventTiming` entries for keyboard, pointer, and touch events. It records `startTime`, `processingStart`, `processingEnd`, and `duration`. INP (Interaction to Next Paint) measures the total latency from user input until the next frame is presented. Unresponsive event handlers blocking the main JavaScript thread are directly surfaced.
- **[SUPER-APP CONTROL]**: Mini-app containers must track input delay and total event duration via Event Timing. If a mini-app's p95 interaction latency exceeds 200ms, the host container logs thread-blocking warnings. Persistent responsiveness failures trigger store automated warnings and downgrade the mini-app's health rating.

#### `telemetry-reporting-api-004`: Deploy out-of-band crash, deprecation, and intervention reporting
- **Standard Reference**: W3C Reporting API §3 (Interface Report), §4 (Report Delivery and Endpoints)
- **Primary URL**: https://www.w3.org/TR/reporting-1/
- **Evidence Level**: `normative_standard`
- **[SPEC FACT]**: The W3C Reporting API decouples error generation from page execution. When configured via HTTP `Reporting-Endpoints`, the user agent buffers crash reports, CSP violations, deprecation notices, and browser interventions, asynchronously delivering them via POST requests to specified backend endpoints even if the page crashes or navigates away.
- **[SUPER-APP CONTROL]**: Super-app webviews must inject a standard `Reporting-Endpoints: superapp-telemetry="https://telemetry.superapp.internal/reports"` header on all mini-app document requests. This ensures that unhandled renderer crashes, deprecated JSAPI calls, and CSP egress violations are automatically captured by the super-app observability pipeline without relying on in-page JavaScript error handlers.

#### `telemetry-network-error-logging-005`: Instrument Network Error Logging (NEL) for client connection and DNS failure visibility
- **Standard Reference**: W3C Network Error Logging §3 (NEL Policy), §4 (Error Types)
- **Primary URL**: https://www.w3.org/TR/network-error-logging/
- **Evidence Level**: `normative_standard`
- **[SPEC FACT]**: Network Error Logging (NEL) allows origin servers to declare error logging policies via the `NEL` HTTP response header. When enabled, user agents generate structured reports for low-level network failures—including DNS resolution timeouts, TCP connection resets, TLS handshake errors, and HTTP protocol errors—and queue them for delivery via the Reporting API.
- **[SUPER-APP CONTROL]**: Super-app edge gateways and package CDN distribution nodes must emit `NEL` headers configured with report sampling. When mini-apps experience packet loss, CDN partition, or SSL validation failures on mobile carrier networks, the super-app operations center receives authoritative client-side connection telemetry, accelerating MTTR and isolating CDN issues from application code defects.
