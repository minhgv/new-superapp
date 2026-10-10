# Chuyên đề 144: W3C WAI-ARIA 1.2 / 1.3, W3C AccName 1.2 / Core-AAM 1.2, và W3C ARIA Authoring Practices Guide (APG) Standards

## 1. Tóm tắt điều hành & Bối cảnh kỹ thuật

Vòng nghiên cứu 144 của tiêu chuẩn Mini App Store trên Super App tập trung chuẩn hóa toàn diện ba trụ cột cốt lõi về khả năng tiếp cận web (Web Accessibility), kiến trúc cây tiếp cận (Accessibility Tree - AXTree) và mô hình thành phần tương tác chuẩn:

1. **W3C WAI-ARIA 1.2 / 1.3 (Accessible Rich Internet Applications):** Thiết lập bản đồ phân loại vai trò phân cấp (Abstract, Widget, Document Structure, Landmark), phân biệt chặt chẽ giữa trạng thái động (States) và thuộc tính (Properties), cơ chế Live Regions thông báo tiếng nói theo mức ưu tiên (`aria-live="polite"` vs `assertive`) kèm thuật toán chống nghẽn bão thông báo (announcement storm mitigation) cho các mini app cập nhật thời gian thực, tuân thủ nguyên tắc vàng 'First Rule of ARIA' ưu tiên ngữ nghĩa HTML5 bản địa, và đón đầu các cải tiến của ARIA 1.3 (thuộc tính `aria-braillelabel` cho màn hình chữ nổi Braille, vai trò chuyên biệt `suggestion`, `comment`, `mark`).

2. **W3C AccName 1.2 & Core-AAM 1.2 (Accessible Name Computation & Core Accessibility API Mappings):** Chuẩn hóa thang đo thứ tự ưu tiên tính toán tên tiếp cận 6 bước (từ kiểm tra ẩn 2A, duyệt `aria-labelledby` 2B, nhãn trực tiếp `aria-label` 2C, nhãn bản địa HTML 2D, duyệt chuỗi con 2E, đến fallback tooltip 2F), thuật toán chống đệ quy lặp vô hạn và chuẩn hóa chuỗi phẳng (whitespace collapsing). Đồng thời, Core-AAM 1.2 chuẩn hóa tầng ánh xạ từ DOM sang các API trợ năng hệ điều hành gốc (Microsoft UIA, Apple NSAccessibility, Linux AT-SPI, Android AccessibilityNodeInfo), vòng đời sự kiện đột biến AXTree, tỉa cành DOM ẩn tối ưu hiệu năng, và quy trình kiểm thử tự động headless (Playwright/Puppeteer AX snapshot) loại bỏ triệt để nút bấm vô danh.

3. **W3C ARIA Authoring Practices Guide (APG) Patterns & Chuẩn mực WCAG 2.2:** Chuẩn hóa các mẫu tương tác bàn phím mẫu: Hộp thoại Modal Dialog (khóa tiêu điểm focus trapping, phím Escape đóng hộp thoại và khôi phục tiêu điểm ban đầu), Tabs & Accordion (điều hướng bàn phím roving tabindex, chuyển đổi tabpanel), Combobox & Menu (quản lý tiêu điểm ảo hiệu năng cao qua `aria-activedescendant`), tiêu chí vòng viền tiêu điểm rõ ràng (:focus-visible, tương phản tối thiểu 3:1) loại bỏ bẫy bàn phím theo WCAG 2.2 SC 2.1.2, và chuẩn hóa kích thước mục tiêu cảm ứng di động (tối thiểu 24x24px, khuyến nghị 44x44px) bảo toàn thao tác trợ năng TalkBack / VoiceOver trên thiết bị di động.

---

## 2. Danh mục 15 Findings chi tiết (Milestone 144)

### Finding 1: WAI-ARIA Role Taxonomy: Abstract, Widget, Document Structure & Landmark Boundaries
- **ID:** `wai_aria_144_01`
- **Chủ đề (Topic):** `wai_aria_role_taxonomy_and_structure`
- **Phân loại (Category):** Web Accessibility, Semantics & Screen Reader Integration
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** W3C WAI-ARIA 1.2 Section 5 (Role Taxonomy) & Section 5.3 (Categorization of Roles)
- **URL chính thức:** [https://www.w3.org/TR/wai-aria-1.2/](https://www.w3.org/TR/wai-aria-1.2/)
- **URL phụ trợ:** [https://www.w3.org/TR/wai-aria-1.3/](https://www.w3.org/TR/wai-aria-1.3/)

#### Các điểm cốt lõi (Key Takeaways):
- Hierarchical Role Classification: Defines roles into four distinct classes: Abstract roles (base types used by specification, e.g. `roletype`, `widget`, `structure`), Widget roles (interactive standalone or composite UI elements, e.g. `button`, `checkbox`, `dialog`, `tab`, `grid`), Document Structure roles (non-interactive content organization, e.g. `article`, `heading`, `list`, `row`), and Landmark roles (navigational landmarks, e.g. `banner`, `main`, `navigation`, `complementary`).
- Abstract Role Guardrail: Abstract roles exist strictly to define the ontology and property inheritance; authors and mini-app developers MUST NOT use abstract roles in HTML content, as accessibility APIs do not recognize them as concrete accessible objects.
- Subclass Role Inheritance: Widget roles inherit required, supported, and inherited attributes from parent abstract roles (e.g. `button` inherits from `command` -> `widget` -> `roletype`), ensuring consistent property exposure across assistive technologies.
- Composite Widget Hierarchy: Composite widgets (`tablist`, `combobox`, `tree`, `grid`) require explicit child container relationships (`tab`, `treeitem`, `gridcell`), providing screen readers with row/column counts and item indices (`aria-posinset`, `aria-setsize`).
- Super-App WebView Landmark Architecture: Enforces mandatory landmark semantics (`role="main"`) around primary mini-app content viewports to allow TalkBack and VoiceOver users to immediately bypass host shell headers and jump into mini-app interfaces.

#### Khuyến nghị triển khai trên Super App:
- Mandate that all interactive custom components in mini-apps declare a valid, concrete WAI-ARIA widget role if native HTML5 interactive elements are not utilized.
- Incorporate automated static analysis in the super-app app store CI review pipeline that rejects any mini-app bundle containing disallowed abstract ARIA roles or improperly nested composite widget children.

---

### Finding 2: ARIA States & Properties Architecture: Global Attributes, Widget States & Relationship Graphs
- **ID:** `wai_aria_144_02`
- **Chủ đề (Topic):** `wai_aria_states_and_properties_taxonomy`
- **Phân loại (Category):** Web Accessibility, Semantics & Screen Reader Integration
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** W3C WAI-ARIA 1.2 Section 6 (Supported States and Properties)
- **URL chính thức:** [https://www.w3.org/TR/wai-aria-1.2/](https://www.w3.org/TR/wai-aria-1.2/)
- **URL phụ trợ:** [https://www.w3.org/TR/wai-aria-1.3/](https://www.w3.org/TR/wai-aria-1.3/)

#### Các điểm cốt lõi (Key Takeaways):
- States vs Properties Distinction: In WAI-ARIA, 'states' represent dynamic characteristics that change frequently in response to user interaction or execution state (e.g. `aria-busy`, `aria-checked`, `aria-disabled`, `aria-expanded`, `aria-hidden`), whereas 'properties' represent static or less frequently changing features of an element (e.g. `aria-haspopup`, `aria-label`, `aria-labelledby`, `aria-controls`).
- Global Attribute Foundation: Global states and properties are valid on any element regardless of role; this includes core accessibility hooks (`aria-label`, `aria-labelledby`, `aria-describedby`), dynamic status flags (`aria-hidden`, `aria-busy`), and live region indicators (`aria-live`, `aria-atomic`, `aria-relevant`).
- Relationship Attribute Pointers: Relationship attributes (`aria-controls`, `aria-owns`, `aria-flowto`, `aria-details`) take one or more whitespace-separated IDREFs to establish semantic links across non-adjacent DOM subtrees that screen readers cannot infer from standard hierarchical nesting.
- `aria-hidden` Subtree Gating: Setting `aria-hidden="true"` completely removes an element and all of its descendants from the Accessibility Tree (AXTree) while keeping it visually rendered in the DOM, preventing background mini-app elements from cluttering assistive technology focus when an overlay is open.
- Dynamic State Mutation Handshake: Mini-app frameworks must synchronize DOM visual state changes with immediate ARIA state updates (e.g. toggling `aria-expanded="true"` upon menu expansion), ensuring platform accessibility APIs broadcast property-change events without delay.

#### Khuyến nghị triển khai trên Super App:
- Require mini-apps to implement explicit `aria-expanded` and `aria-controls` relationships on all collapsibles, drawers, and disclosure controls to guarantee assistive technology awareness.
- Enforce that background mini-app viewports set `aria-hidden="true"` whenever a host modal dialog, payment sheet, or biometric authorization sheet is active.

---

### Finding 3: WAI-ARIA Live Regions: Dynamic Speech Scheduling & Announcement Storm Mitigation
- **ID:** `wai_aria_144_03`
- **Chủ đề (Topic):** `wai_aria_live_regions_and_telemetry`
- **Phân loại (Category):** Web Accessibility, Real-Time Announcements & Assistive Speech Scheduling
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** W3C WAI-ARIA 1.2 Section 5.2.8 (Live Region Roles) & Section 6.6.1 (Live Region Attributes)
- **URL chính thức:** [https://www.w3.org/TR/wai-aria-1.2/](https://www.w3.org/TR/wai-aria-1.2/)
- **URL phụ trợ:** [https://www.w3.org/TR/wai-aria-1.3/](https://www.w3.org/TR/wai-aria-1.3/)

#### Các điểm cốt lõi (Key Takeaways):
- Live Region Core Mechanics: `aria-live` designates a DOM container whose dynamic text mutations must be announced by assistive technologies without requiring the user to navigate focus to that container.
- Urgency Politeness Taxonomy: Supports three levels: `aria-live="off"` (suppresses announcements; default), `aria-live="polite"` (waits for the user to complete their current action or speech sequence before speaking, recommended for standard updates), and `aria-live="assertive"` (immediately interrupts the assistive technology's speech queue, reserved for critical errors, security warnings, or time-sensitive alerts).
- Atomic Granularity Control: `aria-atomic="true"` instructs assistive technologies to present the entire contents of the live region container as a coherent whole when any sub-node changes, preventing fragmented word announcements in multi-token counters or price calculations.
- Change Filtering with `aria-relevant`: Allows fine-tuning which mutations trigger speech (`additions`, `removals`, `text`, or `all`), ensuring deletion of messages or list items does not produce unwanted chatter unless explicitly configured.
- Announcement Storm Mitigation in Fast-Updating Apps: High-frequency data streams (e.g., real-time ride-hailing tracking, stock price tickers, crypto rates, live flash-sale inventory) can overwhelm screen readers if bound to unthrottled live regions; `aria-busy="true"` must be set while sub-nodes are rapidly updating and cleared only when settled, combined with minimum 500ms debounce timers.

#### Khuyến nghị triển khai trên Super App:
- Provide a native Super-App Accessibility Announcement SDK method (`superapp.accessibility.announce(text, { priority: 'polite' | 'assertive' })`) that handles speech debouncing and queue management across host and mini-app boundaries.
- Prohibit unthrottled `aria-live="assertive"` regions in mini-apps during store review, limiting assertive announcements to emergency alerts, security step-up prompts, and transaction timeouts.

---

### Finding 4: First Rule of ARIA: Implicit Native Semantics, Role Presentation & Anti-Pattern Elimination
- **ID:** `wai_aria_144_04`
- **Chủ đề (Topic):** `wai_aria_native_html_parity_and_first_rule`
- **Phân loại (Category):** Web Accessibility, Semantics & DOM Hygiene
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** W3C WAI-ARIA 1.2 Section 7 (WAI-ARIA Roles and Attributes in HTML) & W3C HTML-AAM 1.0
- **URL chính thức:** [https://www.w3.org/TR/wai-aria-1.2/](https://www.w3.org/TR/wai-aria-1.2/)
- **URL phụ trợ:** [https://www.w3.org/TR/html-aam-1.0/](https://www.w3.org/TR/html-aam-1.0/)

#### Các điểm cốt lõi (Key Takeaways):
- The First Rule of ARIA Mandate: 'If you can use a native HTML element or attribute with the semantics and behavior you require already built-in, then do so instead of re-purposing an element and adding an ARIA role, state or property.' Native elements (`<button>`, `<input>`, `<select>`, `<dialog>`) provide built-in keyboard navigation, focus management, and accessibility API mappings that custom `div[role=button]` elements frequently lack.
- Implicit Semantic Collisions: Adding redundant ARIA roles that match the native element (e.g., `<button role="button">` or `<nav role="navigation">`) causes unnecessary DOM verbosity, while adding conflicting roles (e.g., `<a href="..." role="button">`) strips native link behaviors and confuses assistive technology users.
- Semantic De-escalation with `role="presentation"` / `role="none"`: Removes an element's implicit role from the accessibility tree without affecting its child elements or CSS visual styling, essential for stripping table semantics from layout tables or presentational SVG wrapper elements.
- Interactive Child Protection: If an element with `role="presentation"` or `role="none"` contains interactive child elements or elements with explicit `tabindex`, user agents MUST ignore the presentation role and expose the element to prevent keyboard/assistive lockouts.
- Store Gating on ARIA Anti-Patterns: Review criteria must reject pseudo-buttons (`<div onclick="...">`) that lack both keyboard event listeners (`keydown` Enter/Space) and explicit `role="button"` + `tabindex="0"`, ensuring equal access for switch devices and screen readers.

#### Khuyến nghị triển khai trên Super App:
- Enforce a mandatory store review linting rule checking that mini-apps prioritize semantic HTML5 elements before resorting to custom ARIA polyfills.
- Flag and reject mini-apps containing non-interactive DOM elements (`div`, `span`, `i`) with click listeners that lack valid keyboard handlers, focusability, and accessible role declarations.

---

### Finding 5: WAI-ARIA 1.3 Advancements: Braille Attributes, Specialized Roles & Automated Accessibility Gating
- **ID:** `wai_aria_144_05`
- **Chủ đề (Topic):** `wai_aria_1_3_advancements_and_store_gating`
- **Phân loại (Category):** Web Accessibility Standards, Emerging ARIA 1.3 & CI/CD Review Gating
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** W3C WAI-ARIA 1.3 (Working Draft) & W3C ARIA Store Compliance Gating
- **URL chính thức:** [https://www.w3.org/TR/wai-aria-1.3/](https://www.w3.org/TR/wai-aria-1.3/)
- **URL phụ trợ:** [https://www.w3.org/TR/wai-aria-1.2/](https://www.w3.org/TR/wai-aria-1.2/)

#### Các điểm cốt lõi (Key Takeaways):
- WAI-ARIA 1.3 New Attribute Innovations: Introduces `aria-braillelabel` and `aria-brailleroledescription`, providing explicit, tailored strings for refreshable Braille displays where compact contract-grade Braille abbreviations are preferred over verbose spoken text.
- Expanded Role Taxonomy in 1.3: Formalizes specialized roles including `suggestion`, `comment`, `mark`, `meter`, `time`, and `code`, allowing rich collaborative document mini-apps and IDE tools to expose fine-grained semantic markers to accessibility APIs without custom heuristics.
- Contextual Role Descriptions with `aria-roledescription`: Allows authors to define a human-readable, localized category descriptor for custom widgets (e.g. `aria-roledescription="Slide"` on a carousel panel), but strictly prohibits overriding foundational widget roles without preserving core keyboard behaviors.
- Automated AXTree Review Architecture: Modern super-app store review pipelines can execute automated headless browser passes (using axe-core or Playwright accessibility snapshots) against submitted mini-app packages, verifying ARIA attribute validity, IDREF integrity, and role conformance prior to human review.
- Standardized Store Gating Checklist: Establishes a zero-tolerance policy for critical accessibility defects: (1) orphan IDREFs in `aria-labelledby`/`aria-controls`, (2) invalid role strings, (3) missing required ARIA states for declared roles, and (4) interactive elements lacking accessible names.

#### Khuyến nghị triển khai trên Super App:
- Integrate automated WAI-ARIA 1.2 / 1.3 conformance validation into the super-app CLI devtool (`superapp-cli lint --a11y`), alerting developers to invalid roles and missing states during local development.
- Incorporate automated accessibility gating into the public store submission workflow, rejecting mini-apps with Level A/AA ARIA syntax violations before approval.

---

### Finding 6: W3C AccName 1.2 Algorithmic Precedence: The Recursive Accessible Name Computation Ladder
- **ID:** `accname_core_aam_144_06`
- **Chủ đề (Topic):** `accname_algorithmic_precedence_and_calculation`
- **Phân loại (Category):** Accessibility API Mapping, Name Computation & Screen Reader Labeling
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** W3C Accessible Name and Description Computation 1.2 Section 4.3 (Computation Steps)
- **URL chính thức:** [https://www.w3.org/TR/accname-1.2/](https://www.w3.org/TR/accname-1.2/)
- **URL phụ trợ:** [https://www.w3.org/TR/core-aam-1.2/](https://www.w3.org/TR/core-aam-1.2/)

#### Các điểm cốt lõi (Key Takeaways):
- Precise Algorithmic Precedence Order: The AccName 1.2 specification defines an authoritative, deterministic order of precedence for calculating an accessible object's name: Step 2A (Hidden subtrees skipped unless referenced by IDREF), Step 2B (`aria-labelledby` / `aria-labeledby` traversal), Step 2C (Direct `aria-label`), Step 2D (Native host language labeling, e.g. `<label>`, `alt`, `placeholder`), Step 2E (Subtree text traversal for roles allowing name from content), Step 2F (Tooltip / `title` attribute fallback).
- `aria-labelledby` Primacy: If an element has an `aria-labelledby` attribute pointing to valid existing element IDs, its value takes absolute precedence over `aria-label`, native label text, inner subtree text, and title attributes, overriding all other naming sources.
- Direct `aria-label` Override: When `aria-labelledby` is absent or unresolvable, an explicit `aria-label` attribute on the element takes precedence over all native labels and child content, allowing developers to provide concise, screen-reader-specific labels for icon buttons without altering DOM layout.
- Native Attribute Semantics: If no ARIA labeling attributes exist, AccName defers to HTML-AAM rules: for `<img>`, the `alt` attribute is consumed; for `<input>`, associated `<label for="...">` or wrapping `<label>` is consumed, followed by `placeholder` as a last-resort fallback.
- Name from Content Role Boundaries: AccName strictly limits text subtree traversal (Step 2E) to specific roles permitted by WAI-ARIA to receive their name from contents (e.g. `button`, `link`, `tab`, `heading`, `checkbox`); roles like `textbox`, `combobox`, `dialog`, or `slider` DO NOT derive accessible names from inner content, requiring explicit labels.

#### Khuyến nghị triển khai trên Super App:
- Mandate in the mini-app development guide that all icon-only interactive controls (e.g., shopping cart, back buttons, search triggers) declare an explicit `aria-label` or `aria-labelledby` compliant with AccName 1.2.
- Implement automated AccName verification in the store review toolchain, ensuring that every interactive control resolves to a non-empty, meaningful accessible name string.

---

### Finding 7: AccName Recursion Control, Loop Prevention & Flat Text String Serialization
- **ID:** `accname_core_aam_144_07`
- **Chủ đề (Topic):** `accname_loop_prevention_and_text_serialization`
- **Phân loại (Category):** Accessibility Algorithms, String Serialization & Infinite Loop Defense
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** W3C Accessible Name and Description Computation 1.2 Section 4.3 (Recursion and Loop Prevention)
- **URL chính thức:** [https://www.w3.org/TR/accname-1.2/](https://www.w3.org/TR/accname-1.2/)
- **URL phụ trợ:** [https://www.w3.org/TR/core-aam-1.2/](https://www.w3.org/TR/core-aam-1.2/)

#### Các điểm cốt lõi (Key Takeaways):
- Circular Reference and Loop Defense: To eliminate infinite recursion, the AccName algorithm maintains a traversal tracking set of visited DOM nodes during `aria-labelledby` and `aria-describedby` resolution; if a node is encountered that is already in the current traversal stack, the algorithm immediately returns an empty string for that branch.
- Single-Hop `aria-labelledby` Chain Limit: When calculating the name of a referenced node during `aria-labelledby` resolution, subsequent `aria-labelledby` attributes on child or referenced nodes are strictly ignored, preventing multi-hop indirection loops.
- Hidden Referenced Element Traversal: While hidden elements (`display: none`, `visibility: hidden`, `aria-hidden="true"`) are normally excluded from the accessibility tree, AccName Step 2B explicitly traverses into hidden subtrees IF they are specifically referenced by an `aria-labelledby` IDREF, allowing offscreen accessible labels.
- Whitespace Normalization & Separator Collapsing: Text collected from recursive subtree node traversals must be serialized into a flat string where consecutive whitespace characters (spaces, tabs, newlines) are collapsed into a single space, and leading/trailing whitespace is trimmed.
- Embedded Control Text Extraction: When an accessible name traverses across an embedded form control (such as an `<input>` or `<select>` inside a label), the algorithm extracts the current value or selection of that control rather than its placeholder, ensuring live screen reader accuracy.

#### Khuyến nghị triển khai trên Super App:
- Require mini-app developers to test complex compound labels against the AccName loop-prevention rules to avoid silent empty-string accessible name failures.
- Standardize headless DOM tree serialization checks in super-app review pipelines to verify that dynamic form labels containing embedded controls produce expected string outputs.

---

### Finding 8: W3C Core-AAM 1.2: DOM-to-Platform Accessibility API Translation Architecture
- **ID:** `accname_core_aam_144_08`
- **Chủ đề (Topic):** `core_aam_platform_accessibility_api_mapping`
- **Phân loại (Category):** Accessibility Engine Architecture, Platform Bridge & OS Accessibility APIs
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** W3C Core Accessibility API Mappings 1.2 Section 5 (Exposing WAI-ARIA in Accessibility APIs)
- **URL chính thức:** [https://www.w3.org/TR/core-aam-1.2/](https://www.w3.org/TR/core-aam-1.2/)
- **URL phụ trợ:** [https://www.w3.org/TR/html-aam-1.0/](https://www.w3.org/TR/html-aam-1.0/)

#### Các điểm cốt lõi (Key Takeaways):
- Platform API Abstraction Layer: Core-AAM 1.2 standardizes how user agent engines translate web DOM nodes, ARIA roles, states, and properties into native operating system accessibility frameworks: Microsoft UI Automation (UIA) on Windows, Apple NSAccessibility on macOS/iOS, AT-SPI / ATK on Linux, and Android AccessibilityNodeInfo on Android.
- Unified Role Mapping Table: Core-AAM maps each ARIA widget and document role to corresponding platform constants (e.g., `role="button"` maps to UIA `ButtonControlTypeId`, NSAccessibility `NSAccessibilityButtonRole`, and Android `android.widget.Button` class name).
- State & Property Translation: Dynamic ARIA attributes map directly to platform accessibility properties (e.g., `aria-expanded="true"` maps to UIA `ExpandCollapsePattern.ExpandCollapseState.Expanded` and NSAccessibility `NSAccessibilityExpandedAttribute`).
- Virtual Node Hierarchy Projection: The accessibility tree (AXTree) is constructed as a decoupled, parallel hierarchy reflecting only semantically meaningful nodes, stripping layout-only DOM elements while preserving interactive and document landmarks.
- Mobile WebView Bridging Parity: On mobile operating systems (iOS and Android), the super-app host container renders mini-apps within native WebViews; Core-AAM compliance in Chromium (Android WebView) and WebKit (WKWebView) ensures that custom mini-app controls are correctly projected into native OS accessibility caches.

#### Khuyến nghị triển khai trên Super App:
- Ensure the native super-app mobile shell preserves accessibility API bridge propagation between the native platform layer and embedded WebViews without disabling assistive node caching.
- Establish automated multi-platform accessibility testing across Android (TalkBack AccessibilityNodeInfo) and iOS (VoiceOver UIAccessibility) during mini-app store validation.

---

### Finding 9: AXTree Lifecycle, Mutation Firing & Subtree Culling in Core-AAM 1.2
- **ID:** `accname_core_aam_144_09`
- **Chủ đề (Topic):** `core_aam_axtree_lifecycle_and_events`
- **Phân loại (Category):** Accessibility Tree Architecture, Mutation Events & Performance Optimization
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** W3C Core Accessibility API Mappings 1.2 Section 6 (Events)
- **URL chính thức:** [https://www.w3.org/TR/core-aam-1.2/](https://www.w3.org/TR/core-aam-1.2/)
- **URL phụ trợ:** [https://www.w3.org/TR/wai-aria-1.2/](https://www.w3.org/TR/wai-aria-1.2/)

#### Các điểm cốt lõi (Key Takeaways):
- Accessibility Object Lifecycle: Core-AAM specifies the exact conditions under which an accessible object is created, updated, or destroyed in memory as DOM nodes are inserted, mutated, or removed by JavaScript frameworks.
- Standardized Platform Event Dispatch: When ARIA attributes change, user agents must fire corresponding platform events: focus changes dispatch `focus-event`, selection changes dispatch `selection-changed`, value updates dispatch `value-changed`, and dynamic content additions trigger `children-changed` or live-region announcements.
- Subtree Culling for Performance: DOM subtrees flagged with `display: none`, `visibility: hidden`, or `aria-hidden="true"` are culled from the accessibility tree, dramatically reducing memory overhead and tree traversal latency on complex mini-app pages.
- Focus Event Synchronization: User interaction leading to keyboard focus must dispatch accessibility focus events strictly after the DOM has updated and layout calculations have settled, preventing screen readers from reading stale text.
- High-Throughput Mutation Throttling: Rapid DOM updates in mini-apps (e.g. infinite scrolling lists, virtualized tables) can trigger AXTree thrashing; Core-AAM implementations batch accessibility event dispatching to prevent main-thread UI stutter on low-tier mobile devices.

#### Khuyến nghị triển khai trên Super App:
- Mandate that virtualized lists in mini-apps maintain proper accessibility lifecycle events and focus synchronization as items are dynamically recycled.
- Benchmark mini-app accessibility tree mutation performance in super-app review pipelines, rejecting mini-apps that trigger AXTree thrashing or excessive main-thread blocking.

---

### Finding 10: Super-App Headless AXTree Validation & TalkBack/VoiceOver Quality Gating
- **ID:** `accname_core_aam_144_10`
- **Chủ đề (Topic):** `accname_core_aam_quality_gating_and_headless_testing`
- **Phân loại (Category):** Quality Assurance, Headless Automation & Store Review Enforcement
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** W3C AccName 1.2 & Core-AAM 1.2 Verification Architecture / WCAG 2.2 Success Criterion 4.1.2
- **URL chính thức:** [https://www.w3.org/TR/accname-1.2/](https://www.w3.org/TR/accname-1.2/)
- **URL phụ trợ:** [https://www.w3.org/TR/core-aam-1.2/](https://www.w3.org/TR/core-aam-1.2/), [https://www.w3.org/TR/WCAG22/](https://www.w3.org/TR/WCAG22/)

#### Các điểm cốt lõi (Key Takeaways):
- Automated Headless AXTree Inspection: Store review pipelines can inspect the browser's rendered accessibility tree via Chrome DevTools Protocol (`Accessibility.getFullAXTree`) or Playwright accessibility snapshots, evaluating computed names, roles, and states without manual screen reader operation.
- Zero Unlabeled Interactive Elements Policy: Automated review gating verifies that every focusable or interactive element (buttons, links, inputs, custom widgets) has a non-empty, descriptive computed accessible name string, directly failing packages with unlabeled icons.
- Label-in-Name Verification (WCAG 2.2 SC 2.5.3): AccName computation allows testing that for controls with visible text labels, the accessible name includes the exact visual text string, ensuring speech-input users can activate controls by speaking their visible name.
- Cross-WebView Bridge State Integrity: Verifies that custom gestures in mini-apps (swipes, pinches, long presses) have programmatic ARIA state equivalents that TalkBack and VoiceOver can trigger via platform action interfaces (`performAction`).
- Store Rejection Metric Thresholds: Establishes clear store review gating metrics: 0 missing accessible names on primary checkout and login paths, 100% role-attribute validity across custom components, and zero unlabeled media elements.

#### Khuyến nghị triển khai trên Super App:
- Integrate automated Headless AXTree checking into the super-app app store CI review harness to catch unlabeled buttons and broken `aria-labelledby` references automatically.
- Publish an official Super-App Accessibility Testing Matrix providing developers with concrete test fixtures for TalkBack (Android) and VoiceOver (iOS) verification.

---

### Finding 11: APG Modal Dialog Pattern: Focus Trapping, Escape Dismissal & Trigger Focus Restoration
- **ID:** `aria_apg_144_11`
- **Chủ đề (Topic):** `apg_modal_dialog_pattern`
- **Phân loại (Category):** Interactive Patterns, Dialog Sandboxing & Keyboard Accessibility
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** W3C ARIA Authoring Practices Guide (APG) - Dialog (Modal) Pattern
- **URL chính thức:** [https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/)
- **URL phụ trợ:** [https://www.w3.org/WAI/ARIA/apg/](https://www.w3.org/WAI/ARIA/apg/), [https://www.w3.org/TR/WCAG22/](https://www.w3.org/TR/WCAG22/)

#### Các điểm cốt lõi (Key Takeaways):
- Semantic Role & Modal Tagging: A modal dialog requires `role="dialog"` (or `role="alertdialog"` for urgent confirmations) and `aria-modal="true"`, signaling to assistive technologies that content outside the dialog container is inert and non-interactive.
- Accessible Label Association: The dialog container MUST be labeled with `aria-labelledby` pointing to the dialog's header title, and optionally described with `aria-describedby` pointing to dialog explanation text.
- Initial Focus Management: Upon dialog presentation, focus MUST be explicitly moved to the first focusable element inside the dialog, or to the dialog container itself (or a cancel button if the primary action is destructive).
- Keyboard Focus Trapping: When the Tab or Shift+Tab keys are pressed, focus must cycle strictly within focusable elements inside the dialog, preventing keyboard focus from escaping into background content (eliminating keyboard traps outside the dialog).
- Escape Dismissal & Focus Restoration: Pressing the Escape key MUST close the dialog, and upon closure, keyboard focus MUST be restored to the exact element that originally triggered the dialog, preserving keyboard context.

#### Khuyến nghị triển khai trên Super App:
- Incorporate a pre-built, APG-compliant Modal Dialog component in the super-app UI library that automatically handles `aria-modal="true"`, focus trapping, Escape dismissal, and focus restoration.
- Require in store review that all custom mini-app popups, bottom sheets, and confirmation dialogs adhere strictly to the APG Modal Dialog keyboard and focus pattern.

---

### Finding 12: APG Tabs & Accordion Patterns: Keyboard Interaction Models & Dynamic State Binding
- **ID:** `aria_apg_144_12`
- **Chủ đề (Topic):** `apg_tabs_and_accordion_patterns`
- **Phân loại (Category):** Interactive Patterns, Navigation Widgets & Composite Keyboard Interaction
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** W3C ARIA Authoring Practices Guide (APG) - Tabs Pattern & Accordion Pattern
- **URL chính thức:** [https://www.w3.org/WAI/ARIA/apg/patterns/tabs/](https://www.w3.org/WAI/ARIA/apg/patterns/tabs/)
- **URL phụ trợ:** [https://www.w3.org/WAI/ARIA/apg/patterns/accordion/](https://www.w3.org/WAI/ARIA/apg/patterns/accordion/), [https://www.w3.org/WAI/ARIA/apg/](https://www.w3.org/WAI/ARIA/apg/)

#### Các điểm cốt lõi (Key Takeaways):
- Tabs Composite Role Architecture: A tabbed interface requires a container with `role="tablist"`, individual tab controls with `role="tab"`, and associated content panels with `role="tabpanel"`.
- Bidirectional Linkage Attributes: Each `role="tab"` must have `aria-controls="panel_id"` pointing to its corresponding panel, and each `role="tabpanel"` must have `aria-labelledby="tab_id"` pointing back to its header tab.
- Roving Tabindex Keyboard Navigation: In a tablist, only the currently active tab has `tabindex="0"`, while all inactive tabs have `tabindex="-1"`; pressing Left/Right Arrow keys moves focus between tabs and dynamically updates tabindex and `aria-selected`.
- Selection Follows Focus vs Manual Activation: In fast interfaces, moving focus with arrow keys immediately selects the tab ('selection follows focus'); however, if tab switching requires asynchronous network calls or heavy rendering, focus moves without selecting until Enter or Space is pressed ('manual activation').
- Accordion Disclosure Semantics: For vertical accordion stacks, each section header contains a button with `aria-expanded="true|false"` and `aria-controls="section_id"`, allowing independent expansion without the mutual-exclusion constraints of a tablist.

#### Khuyến nghị triển khai trên Super App:
- Require mini-apps using tabbed navigation bars to implement the APG roving tabindex pattern, ensuring Arrow-key cycling and clean Tab-key egress into the active tabpanel.
- Validate in store review that all multi-section accordion views synchronize visual collapse states with programmatic `aria-expanded` attributes.

---

### Finding 13: APG Combobox & Menu Patterns: Virtual Focus Architecture via `aria-activedescendant`
- **ID:** `aria_apg_144_13`
- **Chủ đề (Topic):** `apg_combobox_and_menu_virtual_focus`
- **Phân loại (Category):** Interactive Patterns, Composite Widgets & Virtual Focus Governance
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** W3C ARIA Authoring Practices Guide (APG) - Combobox Pattern & Menu Pattern
- **URL chính thức:** [https://www.w3.org/WAI/ARIA/apg/patterns/combobox/](https://www.w3.org/WAI/ARIA/apg/patterns/combobox/)
- **URL phụ trợ:** [https://www.w3.org/WAI/ARIA/apg/patterns/menu/](https://www.w3.org/WAI/ARIA/apg/patterns/menu/), [https://www.w3.org/WAI/ARIA/apg/](https://www.w3.org/WAI/ARIA/apg/)

#### Các điểm cốt lõi (Key Takeaways):
- Combobox Composite Structure: A combobox input combines an editable input field with an associated popup listbox, grid, or tree, designated by `role="combobox"`, `aria-expanded="true|false"`, `aria-haspopup="listbox"`, and `aria-controls="listbox_id"`.
- Virtual Focus with `aria-activedescendant`: To avoid costly DOM focus shifting across hundreds of candidate items, the text input retains physical DOM focus (`document.activeElement`) while `aria-activedescendant` is dynamically updated with the ID of the currently highlighted option, causing screen readers to announce it as focused.
- Autocomplete Behavior Modes: The `aria-autocomplete` attribute communicates filtering mechanics: `none` (no dynamic suggestions), `list` (presents matching suggestions in a dropdown), `inline` (autocompletes text inside input), or `both` (simultaneous dropdown and inline completion).
- Menu vs Navigation Distinction: APG strictly distinguishes between application menus (`role="menu"` / `role="menuitem"`) designed for application actions (e.g. cut, copy, paste, delete) and website navigation lists (`<nav>` with links), warning against using menu roles for standard navigation links.
- Keyboard Event Dispatch Table: Mandates standard keyboard behavior: Down/Up Arrows navigate suggestion options, Enter selects the active option and closes the popup, Escape closes the popup without selecting, and Home/End navigate to list boundaries.

#### Khuyến nghị triển khai trên Super App:
- Provide a standardized, high-performance Combobox component in the super-app SDK utilizing `aria-activedescendant` for autocomplete search and location pickers.
- Enforce that search and suggestion widgets in mini-apps adhere to the APG Combobox keyboard interaction standard during automated store review passes.

---

### Finding 14: APG Keyboard Navigation Governance, Roving Tabindex & WCAG 2.2 Focus Sandboxing
- **ID:** `aria_apg_144_14`
- **Chủ đề (Topic):** `apg_keyboard_governance_and_focus_rings`
- **Phân loại (Category):** Keyboard Navigation, Focus Indicator Governance & WCAG 2.2 Compliance
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** W3C APG Keyboard Interface Guidelines & WCAG 2.2 Success Criteria 2.1.1, 2.1.2, 2.4.7, 2.4.11, 2.4.13
- **URL chính thức:** [https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)
- **URL phụ trợ:** [https://www.w3.org/TR/WCAG22/](https://www.w3.org/TR/WCAG22/)

#### Các điểm cốt lõi (Key Takeaways):
- Fundamental Keyboard Operability (WCAG 2.1.1): Every interactive control and workflow in a mini-app MUST be fully operable using only a physical keyboard without requiring specific timings for individual keystrokes.
- Elimination of Keyboard Traps (WCAG 2.1.2): If keyboard focus can be moved to any component, focus must be able to be moved away from that component using only standard keyboard keys (Tab, Shift+Tab, Escape); if non-standard keys are required, the user must be explicitly advised of the exit method.
- Roving Tabindex vs Virtual Focus Mechanics: APG outlines two standard patterns for composite widgets: Roving Tabindex (dynamically shifting `tabindex="0"` to the active item and `-1` to siblings) and Virtual Focus (`aria-activedescendant`); roving tabindex is preferred for small-to-medium widgets while virtual focus is preferred for large datasets.
- Focus Appearance & Contrast (WCAG 2.2 SC 2.4.11 & 2.4.13): Mini-app CSS styling MUST NOT suppress default focus outlines (`outline: none` or `outline: 0`) without replacing them with high-contrast custom focus indicators (`:focus-visible`) meeting minimum 3:1 contrast ratios against adjacent background colors.
- Sequential Navigation Order: The logical Tab navigation order MUST mirror the visual layout order; arbitrary positive `tabindex` values (`tabindex="1"` or higher) are strictly prohibited as an anti-pattern that disrupts natural reading flow.

#### Khuyến nghị triển khai trên Super App:
- Prohibit `outline: none` without an accompanying `:focus-visible` replacement rule in the super-app mini-app style guide and automated CSS linting gates.
- Reject mini-apps that employ positive `tabindex` values or introduce keyboard traps during automated headless review testing.

---

### Finding 15: Super-App Custom Component Review Matrix & Assistive Mobile Gesture Harmonization
- **ID:** `aria_apg_144_15`
- **Chủ đề (Topic):** `apg_store_review_matrix_and_mobile_gestures`
- **Phân loại (Category):** Store Review Policy, Mobile Accessibility & Assistive Touch Harmonization
- **Cấp độ bằng chứng (Evidence Level):** `official_standard`
- **Tiêu chuẩn tham chiếu:** W3C APG Design Patterns & WCAG 2.2 Success Criterion 2.5.8 (Target Size - Minimum)
- **URL chính thức:** [https://www.w3.org/WAI/ARIA/apg/](https://www.w3.org/WAI/ARIA/apg/)
- **URL phụ trợ:** [https://www.w3.org/TR/WCAG22/](https://www.w3.org/TR/WCAG22/)

#### Các điểm cốt lõi (Key Takeaways):
- Custom Component Certification Matrix: Defines an objective store review scoring rubric mapping custom mini-app widgets against APG patterns: verifying correct role assignments, keyboard focusability, state synchronization, and accessible name computation.
- Touch Target Sizing Standard (WCAG 2.2 SC 2.5.8): All interactive touch targets in mobile mini-apps MUST meet minimum dimensions of 24x24 CSS pixels, or provide sufficient spacing such that a 24px diameter circle centered on the target does not intersect another target, with 44x44px strongly recommended for primary controls.
- Mobile Assistive Gesture Interoperability: On mobile devices with screen readers active (Android TalkBack, iOS VoiceOver), standard touch gestures are intercepted by the OS for exploration; mini-apps MUST NOT override single-finger swipe or double-tap gestures with custom web touch handlers without providing accessible alternatives.
- Pointer Cancellation Safety (WCAG 2.1 SC 2.5.2): For touch and mouse clicks, the activation of the function must occur on the `up` event (e.g. `touchend`, `mouseup`) rather than the `down` event, allowing users to abort accidental taps by sliding off the target.
- Automated Keyboard & Screen Reader Emulation Gating: Store submission verification executes automated headless Puppeteer/Playwright scripts executing Tab, Shift+Tab, Arrow, and Enter sequences to guarantee zero accessibility dead-ends across all primary user journeys.

#### Khuyến nghị triển khai trên Super App:
- Require all submitted mini-apps to pass an automated APG component conformance suite verifying keyboard operability and touch target minimum dimensions prior to publication.
- Publish an official Super-App Accessibility Playbook illustrating compliant APG implementations of dialogs, tabs, comboboxes, and drawers for mini-app developers.

---
