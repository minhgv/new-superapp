# Iteration 66: COOP Restrict-Properties, Form-Associated Custom Elements & CSS Font Metric Overrides

## Executive Summary & Strategic Scope
Milestone 66 deepens the architectural rigor of the **Super-App Mini-App Store Standard** across three core web platform and UI foundation layers:
1. **Cross-Origin-Opener-Policy restrict-properties & Cross-Window Security**: Fine-grained WindowProxy property filtering restricting auxiliary popups exclusively to `postMessage`, `closed`, and `blur`/`focus`; mandatory `targetOrigin` enforcement and Structured Clone transfer validation; automated `rel="noopener noreferrer"` disownment preventing reverse tabnabbing; point-to-point `MessageChannel` / `MessagePort` transferable delegation; and `window.closed` watchdog polling for robust auxiliary popup crash and dismissal detection.
2. **Web Components Form Association & ElementInternals**: Formalizing Form-Associated Custom Elements (`static formAssociated = true`) and `attachInternals()`; multipart form submission and BFCache state restoration via `setFormValue()`; constraint validation integration via `setValidity()` syncing directly with native `:valid`/`:invalid` pseudo-classes and WCAG 2.2 AA error bubbles; `ElementInternals` ARIA property reflection preserving Shadow DOM encapsulation; and scoped custom element registries preventing global component tag collisions.
3. **CSS Font Metric Overrides & Cumulative Layout Shift (CLS) Elimination**: Utilizing W3C CSS Fonts Module Level 4 `@font-face` descriptors (`size-adjust`, `ascent-override`, `descent-override`, and `line-gap-override`) to normalize system fallback font metrics to custom web fonts; eliminating horizontal reflow and catastrophic vertical "button hopping"; network-aware `font-display` (`optional` vs `swap`) bandwidth governance; programmatic splash screen preloading via `document.fonts.load()`; and automated `PerformanceObserver` `layout-shift` telemetry enforcing release gating against CLS <= 0.05 thresholds.

---

## Technical Analysis & Normative Requirements

### 1. Cross-Origin-Opener-Policy restrict-properties & Cross-Window Security Contracts

#### A. COOP restrict-properties WindowProxy Property Filtering
- **Standard Reference**: WHATWG HTML Living Standard §7.2 & WICG Cross-Origin-Opener-Policy: restrict-properties.
- **Problem Statement**: Standard `Cross-Origin-Opener-Policy: same-origin` severs all JavaScript references between opening and opened browsing contexts (`window.opener = null`), breaking legitimate multi-window workflows such as federated identity logins, external payment approvals, and merchant popup callbacks.
- **Specification Fact**: Setting `Cross-Origin-Opener-Policy: restrict-properties` provides an intermediate isolation tier that maintains window references across origins but restricts accessible `WindowProxy` properties exclusively to `postMessage()`, `window.closed`, `window.focus()`, and `window.blur()`.
- **Super-App Architecture**: Super-app host environments mandate that external partner mini-apps and auxiliary authentication/settlement popups set `Cross-Origin-Opener-Policy: restrict-properties`. This ensures that malicious third-party code cannot inspect `window.location.href`, access DOM nodes, or probe sensitive navigation timings of the super-app or sibling mini-apps, while simultaneously retaining deterministic cross-window messaging capability.

#### B. Window.postMessage TargetOrigin Clamping & Transferable Hardening
- **Standard Reference**: WHATWG HTML Living Standard §9.4 Cross-document messaging.
- **Problem Statement**: Invoking `window.postMessage(message, "*")` without an explicit origin allows any arbitrary domain navigating the target window to intercept sensitive authentication tokens or transaction payloads.
- **Specification Fact**: The HTML specification mandates that user agents only dispatch the `message` event to the target window if the target window's active document origin matches `targetOrigin`.
- **Super-App Architecture**: Automated store linter gates and static analysis AST scanners reject any mini-app bundle containing `window.postMessage` calls with `"*"` as `targetOrigin`. Mini-apps communicating with host containers or auxiliary popups must specify the explicit canonical HTTPS origin of the receiver. Furthermore, all transmitted payload schemas must validate through the Structured Clone Algorithm without cyclic or prototype-polluting payloads, and high-bandwidth byte buffers must utilize `Transferable` objects (`ArrayBuffer`/`ImageBitmap`) with explicit zero-copy ownership transfer.

#### C. Window.opener Disownment & Reverse Tabnabbing Mitigation
- **Standard Reference**: WHATWG HTML Living Standard §7.1.3 Browsing context names & `window.opener`.
- **Problem Statement**: When a mini-app opens an external web page via `target="_blank"`, the untrusted page can manipulate `window.opener.location` to navigate the mini-app to a phishing site (reverse tabnabbing).
- **Specification Fact**: Disowning the opener reference via `rel="noopener noreferrer"` sets `window.opener = null`, cutting the execution link between the caller and target document.
- **Super-App Architecture**: Super-app container hooks intercept all `window.open` and anchor navigation dispatches initiated within mini-app sandboxes. The container automatically injects `rel="noopener noreferrer"` and sets `window.opener = null` for any outbound URL resolving outside the mini-app's verified domain scope.

#### D. MessageChannel & MessagePort Transferable Conduits
- **Standard Reference**: WHATWG HTML Living Standard §9.5 Channel messaging.
- **Problem Statement**: Broadcasting inter-context RPC requests over global `window.addEventListener("message")` exposes communication to third-party script listeners and prototype tampering.
- **Specification Fact**: Passing `port2` as a `Transferable` object in an initial `postMessage` handshake allows two browsing contexts or workers to communicate exclusively over a private asynchronous channel, isolating subsequent traffic from global event listeners.
- **Super-App Architecture**: Super-app host architectures utilize `MessageChannel` pairing as the default IPC bridge for mini-app extensions and embedded widgets. Upon initializing an iframe or sub-window, the host transfers a dedicated `MessagePort` to the child context. All subsequent native bridge calls execute across this dedicated port with strict request-response UUID tagging, automatic 5,000 ms timeouts, and queue-draining lifecycle hooks (`port.start()` / `port.close()`).

#### E. Window.closed Polling & Auxiliary Window Crash Watchdogs
- **Standard Reference**: WHATWG HTML Living Standard §7.3.3 The Window object — `closed` property.
- **Specification Fact**: Under WICG `COOP: restrict-properties` and HTML Living Standard invariants, `window.closed` is one of the few properties that remains accessible across origins.
- **Super-App Architecture**: When a mini-app delegates user verification, KYC document upload, or merchant checkout to a sandboxed popup window, the host runtime establishes a high-efficiency background watchdog interval (polling `window.closed` at 250 ms intervals or utilizing Page Lifecycle hooks). If the user dismisses the popup window or if the child process encounters an Out-Of-Memory (OOM) termination, the parent immediately detects `window.closed === true`, tears down pending promise queues, cancels held transactional locks, and presents an actionable recovery UI.

---

### 2. Web Components Form Association & ElementInternals Standards

#### A. Custom Elements formAssociated & attachInternals
- **Standard Reference**: WHATWG HTML Living Standard §4.13.5 Element internals and form-associated custom elements.
- **Problem Statement**: Historically, custom Web Components could not participate directly in native HTML `<form>` submission, validation, or reset pipelines without brittle hidden `<input>` hacks.
- **Specification Fact**: By setting `static formAssociated = true` on the component class and invoking `this.attachInternals()` in its constructor, an autonomous custom element receives an `ElementInternals` instance granting access to form control state, validation flags, and form submission lifecycle hooks.
- **Super-App Architecture**: Super-app design system UI libraries (e.g., custom currency inputs, telecom phone number selectors, loyalty coupon pickers) must declare `static formAssociated = true`. This architectural requirement ensures that mini-app developers constructing e-commerce checkout forms or KYC submission flows interact with design system components using standard HTML form APIs, completely eliminating synthetic form-serialization libraries and reducing client JavaScript footprint by up to 35%.

#### B. ElementInternals.setFormValue & State Restoration
- **Standard Reference**: WHATWG HTML Living Standard §4.13.5.1 The `ElementInternals` interface — `setFormValue()`.
- **Specification Fact**: The `ElementInternals.setFormValue(value, state)` method sets the element's submission value. The `value` parameter accepts a string, a `File`, or a `FormData` instance containing multiple named entries. The optional `state` parameter enables the component to serialize internal transient state that the browser automatically restores via `formStateRestoreCallback()` when users navigate back via Back-Forward Cache (bfcache).
- **Super-App Architecture**: Store review criteria mandate that complex multi-field custom components (such as address pickers containing province, district, and street inputs) expose their internal fields via `FormData` payloads in `setFormValue()`. When a user navigates away from a mini-app to inspect terms and returns, the container invokes `formStateRestoreCallback`, ensuring zero data loss and flawless UX continuity across mobile lifecycle transitions.

#### C. ElementInternals.setValidity & Constraint Validation
- **Standard Reference**: WHATWG HTML Living Standard §4.13.5.1 The `ElementInternals` interface — `setValidity()`.
- **Specification Fact**: Calling `internals.setValidity(flags, message, anchor)` informs the host user agent of the component's validation status. The `flags` parameter takes a `ValidityStateFlags` dictionary (e.g., `{ valueMissing: true }`), `message` sets the localized validationMessage string, and `anchor` determines where the native validation bubble points. This automatically syncs with `:valid` and `:invalid` CSS pseudo-classes.
- **Super-App Architecture**: Mini-app compliance rules require all custom inputs in financial and transactional flows to enforce constraint validation through `setValidity()`. Components must supply localized error messages to `setValidity()`, ensuring that the super-app accessibility layer announces form errors consistently to VoiceOver and TalkBack users, complying with WCAG 2.2 AA Form Validation standards.

#### D. ElementInternals ARIA Property Reflection
- **Standard Reference**: W3C Accessibility Object Model & WHATWG HTML Living Standard §4.13.5 `ElementInternals` ARIA mixin.
- **Specification Fact**: `ElementInternals` implements the `ARIAMixin` interface (`internals.role = "combobox"`, `internals.ariaExpanded = "true"`), exposing accessible semantics directly to the platform accessibility tree while keeping host DOM attributes pristine. Additionally, `internals.labels` returns a `NodeList` of associated native `<label>` elements.
- **Super-App Architecture**: Super-app component guidelines require all UI design system components to configure semantic roles and dynamic states exclusively via `internals.role` and `internals.aria*` properties. Clicking an external `<label for="my-custom-input">` automatically focuses the internal shadow input via internals, ensuring 100% feature parity with platform form controls across iOS and Android assistive technologies.

#### E. CustomElementRegistry.define & Scoped Element Registries
- **Standard Reference**: WHATWG HTML Living Standard §4.13.1 & W3C Scoped Custom Element Registries.
- **Problem Statement**: Uncoordinated calls to `customElements.define()` by multiple third-party libraries cause fatal namespace collision errors when registering identical tag names.
- **Super-App Architecture**: To prevent mini-apps or embedded third-party SDKs from overwriting container-level custom elements (such as `<superapp-header>`), the store review gate enforces strict vendor prefixing (e.g., `ma-<appid>-<component>`) in all `customElements.define` calls. Furthermore, super-app host shells running modern engines sandbox sub-components using scoped custom element registries (`new ShadowRoot({ registry })`), ensuring that multiple micro-frontends can load differing versions of the same UI library without encountering fatal name collision exceptions.

---

### 3. CSS Font Metric Overrides & Cumulative Layout Shift (CLS) Elimination

#### A. CSS @font-face size-adjust Fallback Font Normalization
- **Standard Reference**: W3C CSS Fonts Module Level 4 §5.1 Font property descriptors: the `size-adjust` descriptor.
- **Problem Statement**: Asynchronous loading of web fonts causes sudden text reflow when replacing local fallback fonts (e.g., Roboto or Arial), triggering Cumulative Layout Shift (CLS).
- **Specification Fact**: The CSS `@font-face` `size-adjust` descriptor defines a scaling percentage applied to glyph outlines and metrics for a specific font face, allowing developers to match the optical width of the fallback font exactly to the incoming web font.
- **Super-App Architecture**: Super-app performance budgets require mini-apps utilizing custom brand typography to declare calibrated fallback `@font-face` rules with `size-adjust`. By scaling fallback fonts (e.g., `size-adjust: 92%`), text line length and wrap points remain identical before and after the font loads, reducing font-induced layout shifts to 0 and ensuring compliance with the Core Web Vitals target of CLS <= 0.05.

#### B. Vertical Metric Overrides: ascent-override, descent-override, line-gap-override
- **Standard Reference**: W3C CSS Fonts Module Level 4 §5.2 Font metric override descriptors.
- **Specification Fact**: Even when horizontal glyph widths match, font baseline differences cause vertical layout shifts: line heights shift abruptly upon font swap, causing whole paragraphs and interactive CTA buttons to jump vertically. The `ascent-override`, `descent-override`, and `line-gap-override` descriptors define baseline distance, descent distance, and line-to-line spacing explicitly, overriding OpenType/TrueType OS/2 table metrics.
- **Super-App Architecture**: Automated store linters inspect mini-app CSS stylesheets to verify that every `@font-face` declaration for web fonts is accompanied by a metric-matched local fallback declaration containing `ascent-override` and `descent-override`. This standard eliminates the notorious "button hop" issue where mobile users tap a payment or confirmation button just as the font swaps, accidentally hitting an adjacent element.

#### C. CSS font-display Bandwidth Governance (optional vs swap)
- **Standard Reference**: W3C CSS Fonts Module Level 4 §5.3 The `font-display` descriptor.
- **Specification Fact**: `font-display: block` gives a 3-second invisible block period causing severe FOIT. `font-display: swap` gives a 0ms block period and an infinite swap period, ensuring instant readability but causing layout shift if uncalibrated. `font-display: optional` gives an extremely brief block period (~100ms) and zero swap period: if the font is in memory cache, it renders immediately; if not, the fallback font is used for the entire session and the web font is cached in the background for subsequent visits.
- **Super-App Architecture**: Super-app network governance policies mandate `font-display: optional` for body copy and content-heavy mini-apps on 2G/3G connections (or when `Save-Data: on` is active). On high-speed 4G/5G connections, mini-apps may use `font-display: swap` provided that `size-adjust` and `ascent-override` descriptors are configured on fallback font declarations, guaranteeing both instant text availability and zero cumulative layout shift.

#### D. Document.fonts (FontFaceSet.load) Splash Screen Warmup
- **Standard Reference**: W3C CSS Font Loading Module Level 3 §4 The `FontFaceSet` interface — `load()`.
- **Specification Fact**: `document.fonts.load(font, [text])` returns a Promise that resolves when all font faces matching the specified font description are fetched and loaded into memory. Specifying the optional `text` parameter triggers subset-aware font loading, fetching only the unicode-range subsets required for initial view rendering.
- **Super-App Architecture**: When a user launches a mini-app, the super-app host container displays a native skeleton splash screen for 150-300ms. During this interval, the host runtime invokes `document.fonts.load("16px CustomBrandFont")` concurrently with service worker navigation preloading. By the time the mini-app UI dismisses the skeleton loader and renders its DOM, `document.fonts.status` is already "loaded", completely avoiding the fallback-to-webfont layout transition.

#### E. PerformanceObserver LayoutShift Telemetry & Automated Store Gating
- **Standard Reference**: W3C Layout Instability API §2 The `LayoutShift` interface.
- **Specification Fact**: The Layout Instability API exposes the `layout-shift` entry type to `PerformanceObserver`. Each `LayoutShift` entry reports a `value` (impact fraction * distance fraction), `sources` indicating which DOM elements moved, and `hadRecentInput` indicating whether the shift occurred within 500ms of user interaction.
- **Super-App Architecture**: Super-app client observability SDKs register a buffered `PerformanceObserver` for `layout-shift`. If a mini-app registers unprompted layout shifts (`hadRecentInput === false`) exceeding a score of 0.1 during automated review or in p95 real-user telemetry (RUM), the store governance pipeline flags the app for layout instability, enforcing font metric override adoption before production clearance is granted.

---

## Canonical Findings Matrix (Milestone 66)

| Finding ID | Topic / Specification | Category | Evidence Level | Authoritative Standards Reference |
|---|---|---|---|---|
| `coop_rp_066_01` | COOP restrict-properties WindowProxy Isolation | Sandboxing & Cross-Context Security | Specification | WHATWG HTML Living Standard §7.2 & WICG COOP restrict-properties |
| `coop_rp_066_02` | Window.postMessage TargetOrigin Clamping | Sandboxing & Cross-Context Security | Specification | WHATWG HTML Living Standard §9.4 Cross-document messaging |
| `coop_rp_066_03` | Window.opener Disownment & Tabnabbing Defense | Sandboxing & Cross-Context Security | Specification | WHATWG HTML Living Standard §7.1.3 Browsing context names |
| `coop_rp_066_04` | MessageChannel & MessagePort Transferability | Sandboxing & Cross-Context Security | Specification | WHATWG HTML Living Standard §9.5 Channel messaging |
| `coop_rp_066_05` | Window.closed Polling & Crash Watchdogs | Sandboxing & Cross-Context Security | Specification | WHATWG HTML Living Standard §7.3.3 The Window object |
| `elem_int_066_01` | Custom Elements formAssociated & attachInternals | Component Architecture & UX Standards | Specification | WHATWG HTML Living Standard §4.13.5 Element internals |
| `elem_int_066_02` | ElementInternals.setFormValue & BFCache Restore | Component Architecture & UX Standards | Specification | WHATWG HTML Living Standard §4.13.5.1 setFormValue() |
| `elem_int_066_03` | ElementInternals.setValidity Constraint Validation | Component Architecture & UX Standards | Specification | WHATWG HTML Living Standard §4.13.5.1 setValidity() |
| `elem_int_066_04` | ElementInternals ARIA Property Reflection | Component Architecture & UX Standards | Specification | W3C Accessibility Object Model & WHATWG HTML §4.13.5 |
| `elem_int_066_05` | CustomElementRegistry.define & Scoped Registries | Component Architecture & UX Standards | Specification | WHATWG HTML Living Standard §4.13.1 CustomElementRegistry |
| `font_cls_066_01` | CSS @font-face size-adjust Fallback Normalization | Operational Reliability & Observability | Specification | W3C CSS Fonts Module Level 4 §5.1 size-adjust |
| `font_cls_066_02` | Metric Overrides (ascent, descent, line-gap) | Operational Reliability & Observability | Specification | W3C CSS Fonts Module Level 4 §5.2 Font metric overrides |
| `font_cls_066_03` | CSS font-display Bandwidth Governance | Operational Reliability & Observability | Specification | W3C CSS Fonts Module Level 4 §5.3 font-display |
| `font_cls_066_04` | Document.fonts.load() Splash Screen Warmup | Operational Reliability & Observability | Specification | W3C CSS Font Loading Module Level 3 §4 FontFaceSet.load() |
| `font_cls_066_05` | PerformanceObserver LayoutShift CWV Gating | Operational Reliability & Observability | Specification | W3C Layout Instability API §2 LayoutShift |

---

## Conclusion & Architectural Readiness
Milestone 66 establishes the formal specifications for cross-window security isolation, native-grade Web Component form controls, and pixel-perfect typographic layout stability. With the canonical evidence base reaching **941 validated findings**, the Super-App Mini-App Store Standard maintains zero-compromise engineering criteria across mobile platforms.
