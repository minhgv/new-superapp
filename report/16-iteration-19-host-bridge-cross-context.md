# Iteration 19: Host Bridge and Cross-Context Messaging Contracts

## Scope and evidence policy

This section records 12 parent-validated findings from a structurally new direction. The evidence covers WHATWG HTML, RFC 9457, JSON-RPC 2.0, Android WebView/AndroidX WebKit, and Apple WebKit. Official platform behavior is separated from mini-app-store design proposals inside every finding. All 38 unique primary/supporting URLs were rechecked with HTTP 200 on 2026-10-03. Five duplicate primary-URL/platform-wrapper candidates were rejected and are not repeated below. No spreadsheet credentials or raw sheet content were used.

## Findings

### 1. `postmessage_origin_recipient_binding`

- **Source:** WHATWG HTML Standard - Cross-document messaging
- **Evidence level:** `official-normative`
- **Topic:** `host-bridge/postMessage/origin-recipient-binding`
- **Detail:** [SOURCE STATUS] The WHATWG HTML Standard is a Living Standard, not an IETF draft. [NORMATIVE FACT] For Window.postMessage(), the targetOrigin option defaults to '/' (same-origin), a non-wildcard target is parsed to an origin, and the message is discarded when the target window's current Document origin is not the same as that target origin. The resulting MessageEvent carries the sender origin and source WindowProxy; deserialization failure fires messageerror. The security guidance says receivers should check event.origin and expected data format, and authors should not use '*' as targetOrigin for confidential data; it also calls out rate limiting for messages accepted from any origin. [MINI-APP-STORE DESIGN PROPOSAL] Define every native bridge endpoint as a registered capability bound to an exact HTTPS origin, WebView/frame identity, app version, and recipient context. Require an explicit ready handshake, validate both event.origin and event.source against the bound recipient before schema validation/dispatch, prohibit '*' for sensitive capabilities, and correlate each reply to one request_id and capability version. Add host-enforced deadlines, cancellation, duplicate/late-reply rejection, and per-origin quotas. These timeout, cancellation, capability-registry, and native recipient-binding rules are store design proposals; HTML does not define them.
- **Primary URL:** https://html.spec.whatwg.org/multipage/web-messaging.html#security-postmsg
- **Supporting URLs:**
  - https://html.spec.whatwg.org/multipage/web-messaging.html#security-postmsg
- **Verification:** parent recheck passed; candidate JSONL valid; primary/supporting URLs returned HTTP 200.

### 2. `structured_clone_transfer_profile`

- **Source:** WHATWG HTML Standard - Safe passing of structured data
- **Evidence level:** `official-normative`
- **Topic:** `host-bridge/structured-clone/transfer-safety`
- **Detail:** [SOURCE STATUS] The WHATWG HTML Standard is a Living Standard, not an IETF draft. [NORMATIVE FACT] The safe-passing-of-structured-data section defines structured cloning as serialization and deserialization across realm or agent boundaries. Serializable objects are reconstructed independently of a realm; Transferable objects transfer ownership by recreating the object while sharing underlying data and detaching the sender's object. Transfer is irreversible and non-idempotent, not every object or object aspect is transferable, and serialization/deserialization may fail. [MINI-APP-STORE DESIGN PROPOSAL] Use a versioned, schema-validated bridge payload profile instead of treating structured clone as an authorization boundary: default to JSON-compatible data, explicitly allowlist any ArrayBuffer/MessagePort transfer, cap depth/size/transfer count, reject unsupported or unexpected values before invoking native code, and map clone/deserialize failures to a stable protocol error without echoing sensitive data. Treat transferred resources as consumed exactly once and never retry them implicitly. Structured clone specifies data transport semantics; it does not authenticate a sender, bind an origin/recipient, or define cancellation, timeout, or application error policy.
- **Primary URL:** https://html.spec.whatwg.org/multipage/structured-data.html#safe-passing-of-structured-data
- **Supporting URLs:**
  - https://html.spec.whatwg.org/multipage/structured-data.html#safe-passing-of-structured-data
- **Verification:** parent recheck passed; candidate JSONL valid; primary/supporting URLs returned HTTP 200.

### 3. `bridge_problem_details_error_profile`

- **Source:** IETF RFC 9457 - Problem Details for HTTP APIs
- **Evidence level:** `official-proposed-standard`
- **Topic:** `host-bridge/problem-details/structured-errors`
- **Detail:** [SOURCE STATUS] RFC 9457 is a published IETF RFC with status Proposed Standard (July 2023); it obsoletes RFC 7807 and was developed from draft-ietf-httpapi-rfc7807bis, so it is not a current draft. [NORMATIVE RFC FACT] The JSON representation uses application/problem+json. The type member is a URI reference identifying the problem type; status is advisory but a generator MUST use the same status code in the actual HTTP response; title, detail, and instance have defined summary, occurrence-explanation, and occurrence-identifier roles. Problem types MAY add extension members, and clients MUST ignore extensions they do not recognize; consumers SHOULD NOT parse detail for machine semantics. The RFC is an HTTP API error representation, not a native WebView transport. [MINI-APP-STORE DESIGN PROPOSAL] For an HTTP-backed bridge adapter, use stable absolute type URIs for malformed_request, capability_denied, origin_mismatch, recipient_mismatch, unsupported_version, deadline_exceeded, cancelled, and policy_rejected. For a JSON-RPC or native reply, embed the same problem object under the protocol's error data rather than pretending an HTTP status exists; carry an opaque instance/request correlation handle, keep detail free of credentials and sensitive user data, and define extensions such as retryable and retry_after only in the host profile. Apply the RFC's actual-status requirement only where an HTTP response exists. Cancellation, timeout behavior, and bridge-specific type names remain store design choices.
- **Primary URL:** https://datatracker.ietf.org/doc/html/rfc9457
- **Supporting URLs:**
  - https://datatracker.ietf.org/doc/html/rfc9457
- **Verification:** parent recheck passed; candidate JSONL valid; primary/supporting URLs returned HTTP 200.

### 4. `jsonrpc_request_response_error_correlation`

- **Source:** JSON-RPC Working Group - JSON-RPC 2.0 Specification
- **Evidence level:** `neutral-protocol-specification`
- **Topic:** `host-bridge/json-rpc/request-response-errors-cancellation`
- **Detail:** [SOURCE STATUS] The official JSON-RPC 2.0 page records an origin date of 2010-03-26 and an update on 2013-01-04; it is a versioned protocol specification, not an IETF RFC or current draft. [NORMATIVE SPEC FACT] A request uses jsonrpc exactly '2.0', a method, optional structured params, and an optional String/Number/Null id; when present, the server MUST echo the same id. Notifications omit id and MUST NOT receive a response, so they are not confirmable. A response MUST contain either result or error, never both, and its id is required; an Error object has an integer code, short message, and optional structured data, with reserved codes for parse error, invalid request, method not found, invalid params, and internal error. Batch responses may be processed concurrently and returned in any order, matched by id. [MINI-APP-STORE DESIGN PROPOSAL] Carry JSON-RPC inside the bridge data channel with non-null opaque string ids, an allowlisted method-to-capability table, and a server-side transaction/idempotency key. Never use notifications for privileged or state-changing calls. Put origin/recipient binding, protocol version, deadline, cancellation token, and capability grant in a separately validated bridge envelope or method parameters; reject duplicate ids and late replies, and return one terminal response per accepted request. Define a cancel method or transport abort plus deadline-exceeded behavior in the host profile. JSON-RPC itself is transport agnostic and does not define origin authentication, native capability policy, cancellation, timeouts, or replay protection.
- **Primary URL:** https://www.jsonrpc.org/specification
- **Supporting URLs:**
  - https://www.jsonrpc.org/specification
- **Verification:** parent recheck passed; candidate JSONL valid; primary/supporting URLs returned HTTP 200.

### 5. `javascript_interface_no_origin_and_minimal_facade`

- **Source:** Android Developers — WebView – Native bridges / JavascriptInterface API reference / Access native APIs with JavaScript bridge
- **Evidence level:** `primary-platform+store-design-proposal`
- **Topic:** `android-webview/javascript-interface/native-method-exposure/origin-binding`
- **Detail:** [PLATFORM FACT — ANDROID-SPECIFIC] Android Developers documents that addJavascriptInterface injects the supplied Java object into every WebView frame, including iframes, and gives the app no mechanism to verify the calling frame's origin. The bridge guide labels it legacy, says WebView.getUrl() cannot safely identify the calling frame, and warns that API level 16 and earlier is particularly risky; the JavascriptInterface reference says that from API 17/JELLY_BEAN_MR1 only public methods explicitly marked @JavascriptInterface are exposed. [INSECURE/LEGACY API FLAG] Annotation gating is not origin authentication, frame isolation, or package identity binding; Android guidance does not make this a safe arbitrary-method bridge. [STORE DESIGN PROPOSAL — not an Android universal standard] Treat addJavascriptInterface as a compatibility escape hatch only: expose a tiny typed facade with no reflection/eval, bearer tokens, session handles, filesystem access, or generic method dispatcher; remove it before untrusted navigation. Bind each accepted call in the trusted host/gateway to the registered package/app ID, signed release/version, current WebView instance/session, and approved origin, rejecting any mismatch before native work.
- **Primary URL:** https://developer.android.com/privacy-and-security/risks/insecure-webview-native-bridges
- **Supporting URLs:**
  - https://developer.android.com/privacy-and-security/risks/insecure-webview-native-bridges
  - https://developer.android.com/develop/ui/views/layout/webapps/native-api-access-jsbridge
  - https://developer.android.com/reference/android/webkit/JavascriptInterface
- **Verification:** parent recheck passed; candidate JSONL valid; primary/supporting URLs returned HTTP 200.

### 6. `safe_browsing_enabled_and_fail_closed_hit_handling`

- **Source:** Android Developers — WebSettings and WebViewClient API references
- **Evidence level:** `primary-platform+store-design-proposal`
- **Topic:** `android-webview/safe-browsing/onSafeBrowsingHit/fail-closed`
- **Detail:** [PLATFORM FACT — ANDROID-SPECIFIC] WebSettings.setSafeBrowsingEnabled says Safe Browsing verifies links to protect against malware and phishing and is enabled by default on devices that support it. WebViewClient.onSafeBrowsingHit requires the app to invoke the response callback; the documented default is a user interstitial, while custom UI may choose backToSafety or proceed according to the user's response. [STORE DESIGN PROPOSAL — not an Android universal standard] Keep Safe Browsing enabled for mini-app WebViews and treat a hit as a hard capability boundary: cancel or fail closed for the pending bridge call, invalidate result handles and ports, show host-owned warning UI, and require an explicit host policy/user decision before any retry. Never silently call proceed for privileged operations. Safe Browsing is an additional signal; it does not replace the package-specific HTTPS navigation/origin allowlist or package/app identity binding, and a clean Safe Browsing result is not proof that a mini-app is trusted.
- **Primary URL:** https://developer.android.com/reference/android/webkit/WebSettings
- **Supporting URLs:**
  - https://developer.android.com/reference/android/webkit/WebSettings
  - https://developer.android.com/reference/android/webkit/WebViewClient
- **Verification:** parent recheck passed; candidate JSONL valid; primary/supporting URLs returned HTTP 200.

### 7. `webview_error_renderer_lifecycle_and_bridge_teardown`

- **Source:** Android Developers — WebViewClient API reference
- **Evidence level:** `primary-platform+store-design-proposal`
- **Topic:** `android-webview/lifecycle/error/renderer-crash/ssl-error/bridge-teardown`
- **Detail:** [PLATFORM FACT — ANDROID-SPECIFIC] WebViewClient.onReceivedError(WebView, WebResourceRequest, WebResourceError) is called for any resource, including iframes and images, so Android recommends minimum work and the host should distinguish request.isForMainFrame. For recoverable certificate errors, onReceivedSslError's documented default is to cancel; Android warns users are unlikely to make an informed decision and that a proceed/cancel choice may be retained. onRenderProcessGone says the affected WebView cannot be used, must be removed from the view hierarchy, and all references cleaned; the callback is per affected WebView and returning true indicates the host handled it, otherwise a renderer crash can crash the app or a system kill can kill it. [DEPRECATED API FLAG] The legacy onReceivedError(WebView, int, String, String) overload is deprecated in API 23; use the WebResourceRequest overload. [STORE DESIGN PROPOSAL — not an Android universal standard] Model bridge state as pending → ready only after an approved main-frame navigation and origin-bound handshake. On main-frame error, SSL error, Safe Browsing hit, disallowed navigation, or renderer exit, transition to failed-closed, reject in-flight calls, close ports, erase per-view capability context, and return a deterministic host error. Never reuse a renderer-gone WebView; create a fresh instance only after package/app, release, origin, and policy revalidation. Do not tear down the whole mini-app session for a subresource error unless policy requires it.
- **Primary URL:** https://developer.android.com/reference/android/webkit/WebViewClient
- **Supporting URLs:**
  - https://developer.android.com/reference/android/webkit/WebViewClient
- **Verification:** parent recheck passed; candidate JSONL valid; primary/supporting URLs returned HTTP 200.

### 8. `wkwebview_reply_contract_and_error_boundary`

- **Source:** Apple Developer Documentation — WebKit WKScriptMessageHandler / WKScriptMessageHandlerWithReply
- **Evidence level:** `official-platform-documentation+design-proposal`
- **Topic:** `wkwebview-bridge/reply/error-contract`
- **Detail:** [APPLE PLATFORM BEHAVIOR] WKScriptMessageHandler receives a targeted JavaScript message through userContentController(_:didReceive:); Apple directs implementations that need a response to WKScriptMessageHandlerWithReply, whose userContentController(_:didReceive:replyHandler:) receives the message and a reply callback. The JavaScript entry point is window.webkit.messageHandlers.<name>.postMessage(<messageBody>). Apple documents reply values as limited to NSNumber, NSString, NSDate, NSArray, NSDictionary, and NSNull; the reply must be nil when an error occurred, and errorMessage is nil on success or a string describing the error. [STORE DESIGN PROPOSAL] Define a versioned, bounded bridge envelope and validate operation, request identifier, and payload schema before native work; return only the documented Foundation-compatible types and use an app-defined stable error code/message inside that envelope. Treat missing, unsupported, or malformed bodies as rejected requests. [DOCUMENTED LIMIT] These pages do not define authentication, origin trust, replay protection, timeouts, a JavaScript Promise mapping, or a required error vocabulary; those are host policy choices, not Apple guarantees.
- **Primary URL:** https://developer.apple.com/documentation/webkit/wkscriptmessagehandlerwithreply/usercontentcontroller(_:didreceive:replyhandler:)
- **Supporting URLs:**
  - https://developer.apple.com/documentation/webkit/wkscriptmessagehandlerwithreply/usercontentcontroller(_:didreceive:replyhandler:)
  - https://developer.apple.com/documentation/webkit/wkscriptmessagehandlerwithreply
  - https://developer.apple.com/documentation/webkit/wkscriptmessagehandler
  - https://developer.apple.com/documentation/webkit/wkscriptmessage/body
- **Verification:** parent recheck passed; candidate JSONL valid; primary/supporting URLs returned HTTP 200.

### 9. `wkwebview_content_world_namespace_boundary`

- **Source:** Apple Developer Documentation — WebKit WKContentWorld
- **Evidence level:** `official-platform-documentation+design-proposal`
- **Topic:** `wkwebview-content-worlds/isolation-boundary`
- **Detail:** [APPLE PLATFORM BEHAVIOR] WKContentWorld defines a JavaScript execution scope/namespace. Apple describes separate copies of JavaScript environment variables for an app’s web environment and individual webpages or scripts, and recommends separate worlds for app-specific bridge logic versus content-specific scripts. The page world is the current webpage’s content; defaultClient is the default world for clients; custom worlds can be created by name. A world does not persist data outside the current web view or webpage, variables from a previous page are gone after navigation, and variables in the same-named world do not appear across different WKWebView instances. [IMPORTANT LIMIT] The content world does not apply to the document or DOM: DOM changes are visible to all script code regardless of world, and Apple’s description is namespace/environment separation rather than an origin, authentication, or complete DOM-isolation guarantee. [STORE DESIGN PROPOSAL] Put host-injected bridge adapters and host-owned handlers in a named app world or defaultClient as appropriate, expose a deliberately narrow page-facing entry point only when required, and never treat the world identifier alone as the mini-app identity or authorization proof. Bind the bridge to the registered web view, package/version, expected page, and host policy separately.
- **Primary URL:** https://developer.apple.com/documentation/webkit/wkcontentworld
- **Supporting URLs:**
  - https://developer.apple.com/documentation/webkit/wkcontentworld
  - https://developer.apple.com/documentation/webkit/wkcontentworld/page
  - https://developer.apple.com/documentation/webkit/wkcontentworld/defaultclient
- **Verification:** parent recheck passed; candidate JSONL valid; primary/supporting URLs returned HTTP 200.

### 10. `wkwebview_handler_registration_and_message_provenance`

- **Source:** Apple Developer Documentation — WebKit WKUserContentController / WKScriptMessage
- **Evidence level:** `official-platform-documentation+design-proposal`
- **Topic:** `wkwebview-bridge/registration/message-validation/origin-metadata`
- **Detail:** [APPLE PLATFORM BEHAVIOR] add(_:contentWorld:name:) requires a WKScriptMessageHandler, scopes it to the selected WKContentWorld, requires the name to be unique within the user content controller and non-empty, and defines window.webkit.messageHandlers.<name>.postMessage(<messageBody>) in all frames in that specified world. The reply-capable addScriptMessageHandler(_:contentWorld:name:) follows the same naming/world model for WKScriptMessageHandlerWithReply. WebKit packages the posted parameter into an appropriate type and delivers a WKScriptMessage containing body, name, webView, world, and frameInfo. Apple documents the body as Any with allowed Foundation types NSNumber, NSString, NSDate, NSArray, NSDictionary, and NSNull. frameInfo exposes isMainFrame, the frame request, web view, and securityOrigin; the security origin consists of host, protocol, and port. WKFrameInfo and WKSecurityOrigin are transient data-only objects and do not uniquely identify a frame/origin across multiple delegate calls. [STORE DESIGN PROPOSAL] On every bridge call, check the exact handler name, expected content world, bound web view, frame/main-frame policy, current request/origin tuple, and a strict schema/type allowlist; default-deny subframes unless a manifest explicitly permits them. [DOCUMENTED LIMIT] Registration and these metadata fields do not themselves establish sender authentication, schema validation, replay resistance, or trust in the page; the host must implement those controls.
- **Primary URL:** https://developer.apple.com/documentation/webkit/wkusercontentcontroller/add(_:contentworld:name:)
- **Supporting URLs:**
  - https://developer.apple.com/documentation/webkit/wkusercontentcontroller/add(_:contentworld:name:)
  - https://developer.apple.com/documentation/webkit/wkusercontentcontroller/addscriptmessagehandler(_:contentworld:name:)
  - https://developer.apple.com/documentation/webkit/wkscriptmessage
  - https://developer.apple.com/documentation/webkit/wkscriptmessage/body
  - https://developer.apple.com/documentation/webkit/wkscriptmessage/frameinfo
  - https://developer.apple.com/documentation/webkit/wkscriptmessage/world
  - https://developer.apple.com/documentation/webkit/wkframeinfo
  - https://developer.apple.com/documentation/webkit/wkframeinfo/securityorigin
  - https://developer.apple.com/documentation/webkit/wksecurityorigin
- **Verification:** parent recheck passed; candidate JSONL valid; primary/supporting URLs returned HTTP 200.

### 11. `wkwebview_handler_unregistration_scope_and_lifecycle`

- **Source:** Apple Developer Documentation — WebKit WKUserContentController handler removal
- **Evidence level:** `official-platform-documentation+design-proposal`
- **Topic:** `wkwebview-bridge/handler-removal/lifecycle`
- **Detail:** [APPLE PLATFORM BEHAVIOR] removeScriptMessageHandler(forName:contentWorld:) uninstalls a custom handler from the specified content world; if no handler with that name exists, it does nothing. The older removeScriptMessageHandler(forName:) overload removes the handler installed in the page content world and does not remove one installed in a different content world. WKUserContentController also exposes removal of all handlers from a specified world and removal of all handlers associated with the controller. [STORE DESIGN PROPOSAL] Treat every add operation as a scoped registration record keyed by web view, content world, handler name, mini-app/package version, and capability set; pair it with the matching world-specific remove operation on mini-app unmount, web-view retirement/reuse, capability revocation, quarantine, or session teardown, and use remove-all only when the controller is being retired or deliberately reset. [DOCUMENTED LIMIT] Apple documents the removal operations and their world scope, but does not state that handlers automatically unregister on navigation, page change, deallocation, or any other mini-app lifecycle event. WKContentWorld’s loss of JavaScript variables after navigation must not be treated as proof that native handler registrations were removed.
- **Primary URL:** https://developer.apple.com/documentation/webkit/wkusercontentcontroller/removescriptmessagehandler(forname:contentworld:)
- **Supporting URLs:**
  - https://developer.apple.com/documentation/webkit/wkusercontentcontroller/removescriptmessagehandler(forname:contentworld:)
  - https://developer.apple.com/documentation/webkit/wkusercontentcontroller/removescriptmessagehandler(forname:)
  - https://developer.apple.com/documentation/webkit/wkusercontentcontroller
- **Verification:** parent recheck passed; candidate JSONL valid; primary/supporting URLs returned HTTP 200.

### 12. `wkwebview_navigation_action_policy_host_gate`

- **Source:** Apple Developer Documentation — WebKit WKNavigationDelegate / WKNavigationAction
- **Evidence level:** `official-platform-documentation+design-proposal`
- **Topic:** `wkwebview-navigation/action-policy/restrictions`
- **Detail:** [APPLE PLATFORM BEHAVIOR] WKNavigationDelegate provides policy callbacks to allow or reject navigation changes; Apple says the action callback runs after the interaction but before the web view attempts to load content, and the delegate must execute the decisionHandler at some point, synchronously or asynchronously. If the preferences variant is implemented, WebKit does not call the simpler action-policy method. WKNavigationAction exposes the URL request, sourceFrame, optional targetFrame, and navigationType; targetFrame is nil when the target is a new window. WKNavigationActionPolicy provides allow, cancel, and download outcomes. [STORE DESIGN PROPOSAL] Make navigation policy host-owned and inspect the exact request scheme/host/path, source-frame security origin, main-frame versus subframe, target-frame presence, navigation type, and registered mini-app route before allowing it. Cancel or explicitly route external URLs, new-window actions, downloads, and host deep links rather than letting page JavaScript choose the result; log only non-sensitive decision metadata. [DOCUMENTED LIMIT] Apple documents the callback and policy outcomes, not an allowlist, origin-validation rule, external-routing policy, or bridge-specific security guarantee; those controls must be defined by the store.
- **Primary URL:** https://developer.apple.com/documentation/webkit/wknavigationdelegate/webview(_:decidepolicyfor:decisionhandler:)-2ni62
- **Supporting URLs:**
  - https://developer.apple.com/documentation/webkit/wknavigationdelegate/webview(_:decidepolicyfor:decisionhandler:)-2ni62
  - https://developer.apple.com/documentation/webkit/wknavigationdelegate
  - https://developer.apple.com/documentation/webkit/wknavigationaction
  - https://developer.apple.com/documentation/webkit/wknavigationaction/targetframe
  - https://developer.apple.com/documentation/webkit/wknavigationactionpolicy
- **Verification:** parent recheck passed; candidate JSONL valid; primary/supporting URLs returned HTTP 200.

## Consolidated standard implications

1. **Define a host-owned bridge profile, not an arbitrary native API surface.** Register each bridge method as a capability bound to mini-app ID, immutable release/digest, exact origin tuple, WebView instance, frame policy, protocol version, and host policy version. The package manifest and review decision should expose the capability/data declaration before activation.
2. **Treat origin and recipient binding as separate checks.** `targetOrigin`/allowed-origin rules filter delivery, but they do not prove package identity, sender authenticity, or absence of XSS. Validate source origin, source/frame identity, current navigation epoch, host session, handler/world name, and registered release before native dispatch.
3. **Use a versioned, bounded envelope.** Require non-null request IDs, capability/method, protocol version, deadline, payload schema, and transaction/idempotency key. Enforce size/depth/transfer limits, reject unsupported structured-clone values, return exactly one terminal response, and never use fire-and-forget notifications for privileged state changes.
4. **Make failure deterministic and fail closed.** Map parse, origin, capability, version, timeout, cancellation, navigation, renderer, Safe Browsing, and policy failures to stable machine-readable errors. Close ports, cancel pending work, invalidate handles, and remove bridge registrations on disallowed navigation, logout, quarantine, takedown, WebView retirement, or renderer failure.
5. **Keep platform adapters explicit.** Android `addJavascriptInterface` is a legacy compatibility path with no calling-frame origin verification; AndroidX WebMessageListener provides allowed-origin and source metadata but still needs host policy. Apple `WKContentWorld` is a namespace boundary, not authentication or DOM isolation; message handlers run in all frames in the selected world unless the host applies frame/origin policy. These facts should become conformance tests for each supported host SDK.
6. **Recommended P0 conformance fixtures:** wildcard target-origin rejection for sensitive calls; wrong-origin/source/frame/handler/world rejection; navigation-epoch race; malformed/oversized/unsupported payload; structured-clone failure; duplicate/late response; timeout/cancel race; transferred-port reuse; Android legacy bridge exposure; Android renderer/SSL/Safe Browsing failure; Apple handler removal and content-world mismatch; and exactly-once side-effect behavior under retry.

## Remaining validation gaps

- The cited standards and platform APIs do not select a universal bridge wire format, capability registry, timeout/size values, cancellation transport, package identity proof, or native host isolation boundary. These are store policy decisions requiring Security and platform-owner sign-off.
- The Alibaba/WindVane POC must be exercised on supported Android, iOS, and Flutter host versions to map its actual JSAPI/bridge semantics, origin metadata, frame handling, port lifecycle, navigation hooks, renderer failure behavior, and teardown guarantees.
- A clean origin match, Safe Browsing result, content world, or WebView message channel must not be treated as proof of publisher trust or release integrity; package signature, provenance, review, runtime policy, and host identity checks remain separate controls.
- Performance and availability budgets for bridge calls, cancellation propagation, retries, and offline behavior remain deployment-specific and should be measured using the iteration-19 conformance fixtures.
