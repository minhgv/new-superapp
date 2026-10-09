# Chuyên đề 68: W3C CSS Typed OM, Scroll-driven Animations & Web Animations API

## 1. Tóm tắt điều hành & Bối cảnh kỹ thuật

Trong kiến trúc Super App hiện đại, mini-app chạy trong môi trường WebView nhúng (Android WebView, iOS WKWebView) hoặc Web Container đa tiến trình trên thiết bị di động với tài nguyên CPU và bộ nhớ bị giới hạn nghiêm ngặt. Trải nghiệm người dùng mượt mà (fluid UI, 60fps/120fps scrolling, instant touch feedback) và chỉ số hiệu năng tương tác **Interaction to Next Paint (INP < 200ms)** đóng vai trò quyết định đến tỷ lệ giữ chân người dùng trong các mini-app thương mại điện tử, đặt vé, giao dịch tài chính và dịch vụ tiện ích.

Trước đây, việc thao tác giao diện động và chuyển động trong mini-app gặp phải ba điểm nghẽn nghiêm trọng:
1. **Thao tác chuỗi CSSOM truyền thống (Legacy CSSOM String Manipulation)**: Thao tác `element.style.transform = "translate3d(" + x + "px, " + y + "px, 0)"` buộc C++ parser của trình duyệt phải tokenize, parse cú pháp, tạo áp lực Garbage Collection (GC) liên tục trên main-thread, gây ra hiện tượng micro-stutter và lỗi phân tích ngầm (silent parse failures).
2. **Hiện tượng giật cuộn (Scroll Jank) do Main-Thread Scroll Events**: Việc lắng nghe sự kiện `scroll` trên main-thread và tính toán lại vị trí qua `requestAnimationFrame()` khiến animation không thể đồng bộ kịp tốc độ cuộn cảm ứng phần cứng, gây lag và nghẽn luồng JavaScript chính.
3. **Quản lý vòng đời chuyển động tùy tiện và Rò rỉ Bộ nhớ CSSOM**: Sử dụng CSS animation class hoặc `fill: "forwards"` mà không dọn dẹp khiến các animation object tích tụ vô hạn trong bộ nhớ webview, dẫn đến tràn RAM và bị OS buộc dừng (Android onTrimMemory / iOS jetsam).

Milestone 72 chuẩn hóa bộ ba tiêu chuẩn W3C chuyên sâu về Engine hiển thị và Chuyển động hiệu năng cao:
- **W3C CSS Typed Object Model (Typed OM) API Level 1**: Biểu diễn giá trị CSS dưới dạng JavaScript Object có kiểu dữ liệu tường minh (`CSSStyleValue`, `StylePropertyMap`, `CSSNumericValue`, `CSSTransformValue`), loại bỏ 100% chi phí parse chuỗi và cấp phát chuỗi rác trên main-thread, tăng tốc độ thao tác style từ 3x đến 5x.
- **W3C Scroll-driven Animations Level 1**: Tách chuyển động cuộn khỏi thời gian thực (wall-clock time) và liên kết trực tiếp với tiến trình cuộn (`ScrollTimeline`, `ViewTimeline`), thực thi hoàn toàn trên **Compositor Thread / GPU Rasterizer**, duy trì 60fps/120fps tuyệt đối ngay cả khi JavaScript main-thread bị nghẽn.
- **W3C Web Animations API (WAAPI) Level 1/2**: Hợp nhất mô hình chuyển động giữa CSS và JavaScript qua `Element.animate()`, máy trạng thái phát chuyển động chặt chẽ (`playState`), các phép kết hợp nâng cao (`composite: "add"`), Promise bất đồng bộ (`ready`, `finished`), và phương thức thu gom bộ nhớ `Animation.commitStyles()`.

---

## 2. Bảng đối chiếu các tiêu chuẩn & Cơ chế kỹ thuật

| Tiêu chuẩn / Đặc tả | Interface / Thuộc tính cốt lõi | Cơ chế vận hành Engine | Lợi ích trong Super App Mini-App Container |
|---|---|---|---|
| **W3C CSS Typed OM API Level 1** | `element.attributeStyleMap`<br>`element.computedStyleMap()`<br>`CSSNumericValue`<br>`CSSTransformValue` | Thao tác trực tiếp trên biểu diễn C++ typed nội bộ của trình duyệt; không qua bộ phân tích chuỗi (parser bypass); kiểm tra kiểu dữ liệu tại thời điểm ghi (fail-fast type check). | Loại bỏ GC churn khi xử lý cảm ứng kéo thả; đảm bảo an toàn đơn vị tính toán; chuyển đổi trực tiếp sang `DOMMatrix` cho GPU rendering; chống CSS injection khi truyền theme token. |
| **W3C Scroll-driven Animations Level 1** | `animation-timeline`<br>`scroll-timeline-name / axis`<br>`view-timeline-name / inset`<br>`animation-range` | Liên kết keyframe animation với độ dời cuộn thay vì đồng hồ hệ thống; tính toán ma trận biến đổi per-vsync trực tiếp trong GPU shader trên Compositor thread. | Triệt tiêu hoàn toàn scroll jank; header co giãn / mờ dần mượt mà theo thanh điều hướng host capsule; thanh tiến trình đọc chạy không tốn một chu kỳ CPU main-thread. |
| **W3C Web Animations API (WAAPI)** | `element.animate()`<br>`Animation`<br>`KeyframeEffect`<br>`animation.ready / finished`<br>`animation.commitStyles()` | Cung cấp runtime engine chuyển động thống nhất cho JS; quản lý máy trạng thái phát (`running`, `paused`, `finished`); hỗ trợ cộng dồn chuyển động (`composite: "add"`). | Điều phối chuyển động đa bước (checkout, eKYC) qua async/await; đồng bộ tạm dừng animation khi app vào background; dọn dẹp triệt để rò rỉ bộ nhớ CSSOM. |

---

## 3. Danh mục 15 Phát hiện Chuẩn hóa Chi tiết (Evidence Records)

### [css_typed_om_072_01] W3C CSS Typed Object Model API: Typed Representation & Elimination of CSSOM Parsing Overhead
- **Chủ đề**: `css_typed_om_architecture_vs_legacy_cssom`
- **Phân loại**: High-Performance Rendering & Animation | **Mức độ chứng cứ**: specification
- **Tiêu chuẩn tham chiếu**: W3C CSS Typed OM Level 1 §1 Overview & §2 Architecture
- **URL chính thức**: [https://www.w3.org/TR/css-typed-om-1/](https://www.w3.org/TR/css-typed-om-1/)
- **URL hỗ trợ**: [https://drafts.css-houdini.org/css-typed-om-1/](https://drafts.css-houdini.org/css-typed-om-1/), [https://developer.mozilla.org/en-US/docs/Web/API/CSS_Typed_OM_API](https://developer.mozilla.org/en-US/docs/Web/API/CSS_Typed_OM_API)
- **Tóm tắt cốt lõi**: The CSS Typed OM API exposes CSS values as typed JavaScript objects rather than raw strings, eliminating repetitive string serialization and CSS parser roundtrips in rendering engines.
- **Phân tích kỹ thuật & Kiến trúc Super App**:
  [SPEC FACT] Under W3C CSS Typed OM Level 1 §1-§2, the traditional CSS Object Model (CSSOM) requires developers to manipulate styles as concatenated strings (e.g., element.style.opacity = '0.5' or style.transform = 'translate3d(' + x + 'px, ' + y + 'px, 0)'). Every string assignment forces the browser engine's C++ style engine to tokenize, parse, and validate the string into internal CSS representations, creating significant CPU overhead, string garbage collection (GC) pressure, and silent parse failures on typo errors. The CSS Typed OM introduces typed JavaScript objects implementing 'CSSStyleValue' and 'StylePropertyMap', allowing direct manipulation of the engine's internal typed representation. Numerical values are stored as 'CSSNumericValue' subclasses with explicit unit types, bypassing the CSS parser completely during programmatic mutations. Performance benchmarks demonstrate up to a 3x to 5x reduction in JavaScript-to-CSS execution time for high-frequency style manipulations. [SUPERAPP ARCHITECTURE] Super-app mini-apps executing rich dynamic user interfaces, interactive gesture tracking, custom charting, and touch drag-and-drop frequently update element positions at 60Hz or 120Hz. In resource-constrained mobile webviews, allocating and parsing hundreds of style strings per frame induces micro-stutters and triggers Garbage Collection pauses, directly degrading Interaction to Next Paint (INP). The super-app standard specifies CSS Typed OM as the required styling interface for high-frequency runtime UI mutations, enabling mini-apps to interact directly with the browser compositing subsystem while eliminating string allocation bottlenecks.

---
### [css_typed_om_072_02] StylePropertyMap & computedStyleMap(): Map-like Operations and Computed Style Introspection
- **Chủ đề**: `stylepropertymap_and_computedstylemap_interfaces`
- **Phân loại**: High-Performance Rendering & Animation | **Mức độ chứng cứ**: specification
- **Tiêu chuẩn tham chiếu**: W3C CSS Typed OM Level 1 §3 Style Propertymap Interfaces
- **URL chính thức**: [https://developer.mozilla.org/en-US/docs/Web/API/Element/computedStyleMap](https://developer.mozilla.org/en-US/docs/Web/API/Element/computedStyleMap)
- **URL hỗ trợ**: [https://www.w3.org/TR/css-typed-om-1/](https://www.w3.org/TR/css-typed-om-1/), [https://drafts.css-houdini.org/css-typed-om-1/](https://drafts.css-houdini.org/css-typed-om-1/)
- **Tóm tắt cốt lõi**: StylePropertyMap provides an idiomatic Map-like interface (get, set, append, delete, has, clear) on element.attributeStyleMap, while element.computedStyleMap() returns resolved typed values.
- **Phân tích kỹ thuật & Kiến trúc Super App**:
  [SPEC FACT] W3C CSS Typed OM §3 defines the 'StylePropertyMapReadOnly' and its mutable derivative 'StylePropertyMap'. Elements expose 'element.attributeStyleMap' (an instance of StylePropertyMap representing inline styles) and 'element.computedStyleMap()' (an instance of StylePropertyMapReadOnly returning computed styles). In contrast to legacy 'window.getComputedStyle(element)', which returns an un-typed CSSStyleDeclaration where every property must be accessed as a string and parsed via parseFloat(), computedStyleMap() returns resolved 'CSSStyleValue' objects directly matching the computed value specification. The StylePropertyMap interface implements standard Map semantics: 'get(property)', 'getAll(property)', 'has(property)', 'set(property, ...values)', 'append(property, ...values)', 'delete(property)', and 'clear()'. Passing invalid typed arguments throws a synchronous JavaScript TypeError or SyntaxError at write time, turning silent CSS parsing errors into fail-fast runtime exceptions. [SUPERAPP ARCHITECTURE] In super-app containers, host shell scripts and mini-app frameworks inspect computed dimensions, safe-area insets, and root theme variables to calculate dynamic layout bounds. Using legacy getComputedStyle() triggers synchronous layout recalculations and string parsing across the DOM tree. The super-app SDK mandates computedStyleMap() for host container UI layout queries, allowing container runtime guards to inspect mini-app styles with typed accuracy, zero string allocations, and precise CSS variable resolution.

---
### [css_typed_om_072_03] CSSNumericValue & CSSUnitValue: Mathematical Operations and Type-Safe Dimensional Units
- **Chủ đề**: `cssnumericvalue_arithmetic_and_unit_conversions`
- **Phân loại**: High-Performance Rendering & Animation | **Mức độ chứng cứ**: specification
- **Tiêu chuẩn tham chiếu**: W3C CSS Typed OM Level 1 §5 CSSNumericValue & §6 Numeric Factory Functions
- **URL chính thức**: [https://developer.mozilla.org/en-US/docs/Web/API/CSSNumericValue](https://developer.mozilla.org/en-US/docs/Web/API/CSSNumericValue)
- **URL hỗ trợ**: [https://www.w3.org/TR/css-typed-om-1/](https://www.w3.org/TR/css-typed-om-1/), [https://drafts.css-houdini.org/css-typed-om-1/](https://drafts.css-houdini.org/css-typed-om-1/)
- **Tóm tắt cốt lõi**: CSSNumericValue models mathematical dimensional values and exposes arithmetic methods (add, sub, mul, div, min, max) and unit conversion methods (to, toSum) with compile-time dimensional safety.
- **Phân tích kỹ thuật & Kiến trúc Super App**:
  [SPEC FACT] W3C CSS Typed OM §5 specifies 'CSSNumericValue' as the abstract base class for numeric CSS representations, including 'CSSUnitValue' (single value + unit pair, e.g., CSS.px(10), CSS.em(2), CSS.percent(50)) and complex expressions like 'CSSMathSum', 'CSSMathProduct', 'CSSMathMin', 'CSSMathMax', and 'CSSMathNegate' representing CSS calc() trees. CSSNumericValue provides native mathematical methods: 'add(...values)', 'sub(...values)', 'mul(...values)', 'div(...values)', 'min(...values)', and 'max(...values)'. It also provides 'to(unit)' for converting compatible units (e.g., converting 'CSS.deg(180).to("rad")' or converting between 'in', 'cm', 'mm', 'pt', 'pc', 'px') and 'equals(...values)' for structural equality checks. Attempting to add dimensionally incompatible units without a calc representation (such as adding pixels to seconds) throws a TypeError, preventing dimensional corruption at runtime. [SUPERAPP ARCHITECTURE] Mini-app UI components frequently compute complex offsets involving fixed pixel sizes and relative viewport units (e.g., 'calc(100vw - 32px)'). In legacy code, developers concatenated string expressions, which required recurring calc() re-evaluation. With CSSNumericValue, mini-app component libraries construct typed math trees like 'CSS.vw(100).sub(CSS.px(32))' and assign them directly to attributeStyleMap. This ensures mathematically verified dimensional operations, eliminates regex/string sanitization vulnerabilities, and allows the native browser rendering engine to optimize unit conversions directly.

---
### [css_typed_om_072_04] CSSTransformValue & DOMMatrix Interoperability: Zero-Allocation 2D/3D Transform Sandboxing
- **Chủ đề**: `css_typed_om_transform_and_matrix_interoperability`
- **Phân loại**: High-Performance Rendering & Animation | **Mức độ chứng cứ**: specification
- **Tiêu chuẩn tham chiếu**: W3C CSS Typed OM Level 1 §8 Transform Component Values
- **URL chính thức**: [https://www.w3.org/TR/css-typed-om-1/](https://www.w3.org/TR/css-typed-om-1/)
- **URL hỗ trợ**: [https://drafts.css-houdini.org/css-typed-om-1/](https://drafts.css-houdini.org/css-typed-om-1/), [https://developer.mozilla.org/en-US/docs/Web/API/CSS_Typed_OM_API](https://developer.mozilla.org/en-US/docs/Web/API/CSS_Typed_OM_API)
- **Tóm tắt cốt lõi**: CSSTransformValue represents multi-component 2D and 3D transforms (CSSTranslate, CSSRotate, CSSScale, CSSMatrixComponent), providing direct conversion to DOMMatrix via toMatrix().
- **Phân tích kỹ thuật & Kiến trúc Super App**:
  [SPEC FACT] W3C CSS Typed OM §8 defines 'CSSTransformValue' as an iterable list of 'CSSTransformComponent' subclasses, including 'CSSTranslate', 'CSSRotate', 'CSSScale', 'CSSSkew', 'CSSSkewX', 'CSSSkewY', 'CSSPerspective', and 'CSSMatrixComponent'. Each component encapsulates discrete typed parameters (e.g., CSSTranslate(CSS.px(10), CSS.px(20), CSS.px(0))). Critically, CSSTransformValue and all CSSTransformComponent subclasses implement the 'toMatrix()' method, which instantaneously computes and returns a native 'DOMMatrix' instance representing the resolved 4x4 affine transformation. The 'is2D' boolean attribute allows the rendering pipeline to bypass 3D perspective math when transforms are purely planar. [SUPERAPP ARCHITECTURE] Super-app mini-apps incorporating interactive gesture recognizers (e.g., pinch-to-zoom product carousels, swipeable cards, drag-to-dismiss sheets) require continuous transform updates. In legacy architectures, serializing matrices into 'matrix3d(...) strings and passing them across webview boundaries incurred massive serialization overhead. By composing CSSTransformValue instances or directly assigning CSSMatrixComponent(domMatrix) to 'element.attributeStyleMap.set("transform", ...)', mini-apps achieve zero-allocation transform pipelines that feed directly into GPU compositing layers.

---
### [css_typed_om_072_05] Super-App Container Theme Token Propagation, CSS Custom Property Sandboxing & Security Gates
- **Chủ đề**: `superapp_host_theme_token_propagation_via_typed_om`
- **Phân loại**: High-Performance Rendering & Animation | **Mức độ chứng cứ**: specification
- **Tiêu chuẩn tham chiếu**: W3C CSS Typed OM Level 1 §4 CSSStyleValue Subclasses & Custom Properties
- **URL chính thức**: [https://drafts.css-houdini.org/css-typed-om-1/](https://drafts.css-houdini.org/css-typed-om-1/)
- **URL hỗ trợ**: [https://www.w3.org/TR/css-typed-om-1/](https://www.w3.org/TR/css-typed-om-1/), [https://developer.mozilla.org/en-US/docs/Web/API/CSS_Typed_OM_API](https://developer.mozilla.org/en-US/docs/Web/API/CSS_Typed_OM_API)
- **Tóm tắt cốt lõi**: CSS Typed OM handles CSS Custom Properties via CSSUnparsedValue and CSSKeywordValue, enabling secure host theme token injection and preventing CSS injection attacks.
- **Phân tích kỹ thuật & Kiến trúc Super App**:
  [SPEC FACT] Under W3C CSS Typed OM §4, CSS Custom Properties (variables prefixed with '--') are accessed and mutated as typed 'CSSUnparsedValue' or specific registered types when registered via CSS.registerProperty(). CSSUnparsedValue preserves string segments and variable references ('CSSVariableReferenceValue') as structured tokens rather than raw, unvalidated text. This structural representation prevents CSS injection attacks where an attacker inserts semicolons, curly braces, or malformed expressions (e.g., '--color: red; } input { display: none; }') to hijack stylesheet execution. Furthermore, typed manipulation of custom properties through StylePropertyMap ensures that injected theme tokens adhere to expected token boundaries and cannot break out of their variable declaration context. [SUPERAPP ARCHITECTURE] The super-app host container enforces platform-wide design system consistency (dark/light mode, brand accent colors, font scale factors, corner radii, and safe area padding) by broadcasting theme tokens to embedded mini-apps. Rather than allowing mini-apps or third-party plugins to execute arbitrary style injection, the super-app bridge injects validated design tokens via the root 'document.documentElement.attributeStyleMap.set("--superapp-brand-primary", CSS.hex("#0052CC"))'. The mini-app store review gate audits mini-app stylesheets to ensure they consume host custom properties via typed tokens rather than overriding global styling rules.

---
### [scroll_animations_072_01] W3C Scroll-driven Animations Level 1: Compositor Thread Execution & Elimination of Scroll Jank
- **Chủ đề**: `scroll_driven_animations_architecture_and_compositor_offloading`
- **Phân loại**: High-Performance Rendering & Animation | **Mức độ chứng cứ**: specification
- **Tiêu chuẩn tham chiếu**: W3C Scroll-driven Animations Level 1 §1 Overview & Compositor Model
- **URL chính thức**: [https://www.w3.org/TR/scroll-animations-1/](https://www.w3.org/TR/scroll-animations-1/)
- **URL hỗ trợ**: [https://drafts.csswg.org/scroll-animations-1/](https://drafts.csswg.org/scroll-animations-1/), [https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_scroll-driven_animations](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_scroll-driven_animations)
- **Tóm tắt cốt lõi**: W3C Scroll-driven Animations Level 1 decouples animation timelines from wall-clock time and drives them based on scroll progression, executing entirely on the compositor thread.
- **Phân tích kỹ thuật & Kiến trúc Super App**:
  [SPEC FACT] Under W3C Scroll-driven Animations Level 1 §1, standard web animations progress proportionally to wall-clock time (monotonic clock timestamps). Prior to this specification, synchronizing visual animations with user scrolling required listening to JavaScript 'scroll' events on the main execution thread, manually calculating offsets via element.scrollTop, and applying style changes via requestAnimationFrame(). This legacy pattern incurred severe performance penalties: main-thread execution delays caused dropped frames, layout thrashing, and uncoordinated visual stuttering ('scroll jank') because scrolling and scripting ran asynchronously. Scroll-driven Animations introduce animation timelines whose current time is determined directly by scroll progression (0% to 100%) rather than elapsed seconds. Modern rendering engines (Blink, WebKit) execute these scroll-driven timelines entirely on the compositor thread or GPU rasterizer, completely bypassing the JavaScript main thread and ensuring silky 60fps/120fps animations even when the main thread is blocked by heavy script execution. [SUPERAPP ARCHITECTURE] Super-app mini-apps host complex e-commerce catalogs, long news feeds, and detailed transactional forms. Demanding smooth scrolling while updating sticky promotional banners, header shrinkage, and navigation bar transparency often caused severe Interaction to Next Paint (INP) degradations under the legacy pattern. The super-app standard specifies native Scroll-driven Animations for all scroll-linked UI transitions, completely eliminating JavaScript scroll listener overhead and guaranteeing stutter-free compositor scrolling across low-end mobile devices.

---
### [scroll_animations_072_02] ScrollTimeline Interface & CSS scroll-timeline: Scroller Axis Binding & Progression Mapping
- **Chủ đề**: `scrolltimeline_interface_and_css_properties`
- **Phân loại**: High-Performance Rendering & Animation | **Mức độ chứng cứ**: specification
- **Tiêu chuẩn tham chiếu**: W3C Scroll-driven Animations Level 1 §2 The ScrollTimeline Interface
- **URL chính thức**: [https://developer.mozilla.org/en-US/docs/Web/API/ScrollTimeline](https://developer.mozilla.org/en-US/docs/Web/API/ScrollTimeline)
- **URL hỗ trợ**: [https://www.w3.org/TR/scroll-animations-1/](https://www.w3.org/TR/scroll-animations-1/), [https://drafts.csswg.org/scroll-animations-1/](https://drafts.csswg.org/scroll-animations-1/)
- **Tóm tắt cốt lõi**: ScrollTimeline maps an animation's timeline directly to the scroll offset of a scroll container, configurable declaratively in CSS via scroll-timeline or imperatively via new ScrollTimeline().
- **Phân tích kỹ thuật & Kiến trúc Super App**:
  [SPEC FACT] W3C Scroll-driven Animations §2 defines the 'ScrollTimeline' interface (derived from AnimationTimeline) and associated CSS properties: 'scroll-timeline-name' (declaring a custom dashed-ident timeline name) and 'scroll-timeline-axis' (specifying 'block', 'inline', 'x', or 'y', defaulting to 'block'). The shorthand 'scroll-timeline' configures both. Declaratively, 'animation-timeline: --my-scroll-timeline' attaches an animation to the scroller. In JavaScript, an imperative timeline is created using 'new ScrollTimeline({ source: scrollContainerElement, axis: "block" })'. The timeline's 'currentTime' represents a CSSUnitValue expressed as a percentage ('0%' when scrolled to the start boundary, progressing linearly to '100%' at max scroll). If the scroller container has no overflow or cannot scroll, the timeline enters an inactive phase, and currentTime evaluates to null. [SUPERAPP ARCHITECTURE] In super-app mini-app storefronts, articles, and long transaction summaries, persistent progress bars at the top of the viewport indicate reading or checkout completion. Under the super-app framework, progress bars are bound to a root ScrollTimeline with keyframes '@keyframes progress { from { transform: scaleX(0); } to { transform: scaleX(1); } }' and 'animation-timeline: --page-scroll'. Because the transform calculation is handled by the GPU compositor, reading progress updates continuously without waking the CPU or executing a single line of JavaScript.

---
### [scroll_animations_072_03] ViewTimeline Interface & CSS view-timeline: Subject Element Viewport Intersection Progress
- **Chủ đề**: `viewtimeline_interface_and_subject_visibility_progress`
- **Phân loại**: High-Performance Rendering & Animation | **Mức độ chứng cứ**: specification
- **Tiêu chuẩn tham chiếu**: W3C Scroll-driven Animations Level 1 §3 The ViewTimeline Interface
- **URL chính thức**: [https://developer.mozilla.org/en-US/docs/Web/API/ViewTimeline](https://developer.mozilla.org/en-US/docs/Web/API/ViewTimeline)
- **URL hỗ trợ**: [https://www.w3.org/TR/scroll-animations-1/](https://www.w3.org/TR/scroll-animations-1/), [https://drafts.csswg.org/scroll-animations-1/](https://drafts.csswg.org/scroll-animations-1/)
- **Tóm tắt cốt lõi**: ViewTimeline tracks the relative visibility progression of a specific subject element as it traverses the scrollport of its nearest scrollable ancestor container.
- **Phân tích kỹ thuật & Kiến trúc Super App**:
  [SPEC FACT] W3C Scroll-driven Animations §3 specifies the 'ViewTimeline' interface and its declarative CSS properties: 'view-timeline-name', 'view-timeline-axis', and 'view-timeline-inset'. While ScrollTimeline monitors the absolute scroll position of the container itself, ViewTimeline monitors a specific 'subject' element as it intersects and traverses the container's visible scrollport. The constructor 'new ViewTimeline({ subject: element, axis: "block", inset: [CSS.px(0), CSS.px(100)] })' configures an imperative view timeline. The 'inset' property allows developers to adjust the active viewport boundaries, creating virtual trigger zones similar to IntersectionObserver rootMargin. The subject's 'startOffset' corresponds to the moment the subject's leading edge touches the scrollport's entering edge, and 'endOffset' corresponds to its trailing edge completely leaving the opposite boundary. [SUPERAPP ARCHITECTURE] Super-app mini-apps rely heavily on scroll-triggered entrance animations (e.g., cards fading in, badges rotating into place, statistics counters animating) to create engaging micro-interactions. Traditionally, mini-apps used IntersectionObserver or scroll polling to detect element visibility, followed by triggering CSS animation classes. This caused noticeable activation lag when scrolling rapidly. By applying 'view-timeline: --card-view block' and linking entrance keyframes directly to '--card-view', mini-app UI components smoothly animate in lockstep with touch velocity on the native compositor layer.

---
### [scroll_animations_072_04] Animation Range & Timeline Offsets: entry, exit, cover, contain Phase Control
- **Chủ đề**: `animation_range_and_timeline_offsets`
- **Phân loại**: High-Performance Rendering & Animation | **Mức độ chứng cứ**: specification
- **Tiêu chuẩn tham chiếu**: W3C Scroll-driven Animations Level 1 §4 Timeline Ranges & animation-range
- **URL chính thức**: [https://drafts.csswg.org/scroll-animations-1/](https://drafts.csswg.org/scroll-animations-1/)
- **URL hỗ trợ**: [https://www.w3.org/TR/scroll-animations-1/](https://www.w3.org/TR/scroll-animations-1/), [https://developer.mozilla.org/en-US/docs/Web/CSS/animation-range](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-range)
- **Tóm tắt cốt lõi**: The animation-range property defines precise activation intervals (entry, exit, cover, contain, entry-crossing, exit-crossing) along a scroll or view timeline.
- **Phân tích kỹ thuật & Kiến trúc Super App**:
  [SPEC FACT] Under W3C Scroll-driven Animations §4, an animation attached to a timeline does not necessarily need to span the entire 0% to 100% scroll duration. The 'animation-range' shorthand (and 'animation-range-start' / 'animation-range-end') defines the active progression window using named timeline ranges: (1) 'cover' (the default, spanning from 0% when the subject first enters the scrollport to 100% when it fully exits); (2) 'contain' (from when the subject is first fully inside the scrollport to when it first starts to leave); (3) 'entry' (from when the subject first touches the scrollport entry boundary to when it is fully inside); (4) 'exit' (from when the subject first starts to leave the scrollport to when it has completely exited); (5) 'entry-crossing' and 'exit-crossing'. Offsets can be combined with percentage or length adjustments, such as 'animation-range: entry 25% exit 75%'. In JavaScript, Animation objects configured with a ViewTimeline accept range definitions via the 'rangeStart' and 'rangeEnd' dictionary properties. [SUPERAPP ARCHITECTURE] In mini-app product detail pages, hero images must smoothly dissolve as they scroll out of view, while sticky 'Add to Cart' floating bars should animate in only after the main purchase button exits the viewport. Mini-app developers configure 'animation-range: exit 0% exit 100%' for hero fade-outs and 'animation-range: entry 100% cover 100%' for sticky CTA emergence. This fine-grained range control eliminates fragile scroll offset hardcoding and guarantees consistent responsive behavior across varying phone screen aspect ratios.

---
### [scroll_animations_072_05] Super-App Navigation Header Morphing, Parallax Transitions & Host Capsule Synchronization
- **Chủ đề**: `superapp_navigation_morphing_and_host_capsule_harmony`
- **Phân loại**: High-Performance Rendering & Animation | **Mức độ chứng cứ**: specification
- **Tiêu chuẩn tham chiếu**: W3C Scroll-driven Animations Level 1 §5 Integration with CSS Animations & Transforms
- **URL chính thức**: [https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_scroll-driven_animations](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_scroll-driven_animations)
- **URL hỗ trợ**: [https://www.w3.org/TR/scroll-animations-1/](https://www.w3.org/TR/scroll-animations-1/), [https://drafts.csswg.org/scroll-animations-1/](https://drafts.csswg.org/scroll-animations-1/)
- **Tóm tắt cốt lõi**: Scroll-driven animations synchronize mini-app navigation headers and parallax hero banners with host capsule chrome without triggering main-thread layout thrashing.
- **Phân tích kỹ thuật & Kiến trúc Super App**:
  [SPEC FACT] W3C Scroll-driven Animations integrate seamlessly with CSS transforms, opacity, and filter properties—properties that modern browser engines can mutate on the compositor without triggering DOM reflow or style recalculations on sibling elements. By combining 'scroll-timeline' on the root scroller with compositor-only properties, web applications achieve high-performance header shrinkage, background color cross-fading, and parallax background displacement. The browser compositing engine calculates intermediate transform matrices per vsync frame directly inside GPU shaders based on the hardware scroll position reported by the touchscreen digitizer. [SUPERAPP ARCHITECTURE] Super-app platforms enforce a permanent top-right security capsule (holding 'Close', 'Menu', and 'Security Verification' native buttons). When a mini-app implements an immersive hero banner with a collapsing transparent navigation bar, the mini-app header must dynamically transition from transparent to an opaque background with a subtle border as the user scrolls, while ensuring title text neatly avoids clipping under the host capsule. Under the super-app store standard, mini-apps implement header morphing via native CSS Scroll-driven Animations ('animation-timeline: scroll(root block)'). The host container injects CSS variables representing capsule dimensions, allowing mini-apps to create smooth, native-grade header transitions with zero bridge message overhead and flawless 120Hz responsiveness.

---
### [web_animations_072_01] W3C Web Animations API: Unified Imperative Animation Engine & Element.animate() Pipeline
- **Chủ đề**: `web_animations_api_architecture_and_element_animate`
- **Phân loại**: High-Performance Rendering & Animation | **Mức độ chứng cứ**: specification
- **Tiêu chuẩn tham chiếu**: W3C Web Animations §1 Introduction & §2 Architecture
- **URL chính thức**: [https://www.w3.org/TR/web-animations-1/](https://www.w3.org/TR/web-animations-1/)
- **URL hỗ trợ**: [https://drafts.csswg.org/web-animations-1/](https://drafts.csswg.org/web-animations-1/), [https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API)
- **Tóm tắt cốt lõi**: The W3C Web Animations API (WAAPI) provides a unified runtime model bridging declarative CSS transitions/animations with imperative JavaScript programmatic animation control.
- **Phân tích kỹ thuật & Kiến trúc Super App**:
  [SPEC FACT] W3C Web Animations §1-§2 unifies the conceptual animation models of CSS Transitions, CSS Animations, and SVG/SMIL into a single, standardized runtime architecture. The primary programmatic entry point is 'element.animate(keyframes, options)', which creates a new 'KeyframeEffect', binds it to the element, creates an 'Animation' object linked to the DocumentTimeline, and automatically calls play(). Keyframes are declared as an array of keyframe objects or an object of property-indexed arrays. Options accept timing properties: duration, delay, endDelay, iterations, iterationStart, direction, fill, easing, and composite. Crucially, animations created via Element.animate() execute using the same optimized underlying browser engine pipelines as CSS animations, running on the compositor thread whenever animated properties are restricted to transform, opacity, and filter. [SUPERAPP ARCHITECTURE] Super-app mini-apps incorporate dynamic UI interactions that cannot be predicted statically in CSS stylesheets (e.g., dynamic fly-to-cart particle effects, expanding modal sheets originating from variable touch coordinates, and physics-based fling gestures). In legacy webview architectures, developers relied on setInterval or requestAnimationFrame loops that frequently missed frames under CPU load. The super-app standard mandates WAAPI Element.animate() for all dynamic programmatic animations, allowing mini-apps to generate hardware-composited, sub-millisecond visual feedback without incurring main-thread script evaluation overhead during motion.

---
### [web_animations_072_02] Animation Interface: Deterministic Playback State Machine & Dynamic Timeline Arbitration
- **Chủ đề**: `animation_interface_playback_state_machine`
- **Phân loại**: High-Performance Rendering & Animation | **Mức độ chứng cứ**: specification
- **Tiêu chuẩn tham chiếu**: W3C Web Animations §3 The Animation Interface & §3.4 Playback States
- **URL chính thức**: [https://developer.mozilla.org/en-US/docs/Web/API/Animation](https://developer.mozilla.org/en-US/docs/Web/API/Animation)
- **URL hỗ trợ**: [https://www.w3.org/TR/web-animations-1/](https://www.w3.org/TR/web-animations-1/), [https://drafts.csswg.org/web-animations-1/](https://drafts.csswg.org/web-animations-1/)
- **Tóm tắt cốt lõi**: The Animation interface implements a formal playback state machine (idle, running, paused, finished) with precise temporal manipulation via play(), pause(), reverse(), finish(), and cancel().
- **Phân tích kỹ thuật & Kiến trúc Super App**:
  [SPEC FACT] Under W3C Web Animations §3, every active animation is governed by the 'Animation' interface. The interface exposes a strict playback state machine reflected in 'animation.playState': 'idle' (unattached or canceled), 'running' (advancing along timeline), 'paused' (frozen at current time), and 'finished' (reached end boundary). Key programmatic methods include 'play()', 'pause()', 'reverse()' (inverts playback direction smoothly), 'finish()' (jumps instantly to completion), and 'cancel()' (resets to idle and removes visual effects). The interface exposes 'currentTime' (allowing microsecond-level scrubbing and sync) and 'playbackRate' (enabling smooth speed-ramping, slow-motion, or negative playback rates). Modern revisions introduce 'pending' state flags ('pending' boolean indicating waiting for async compositor handshakes). [SUPERAPP ARCHITECTURE] In super-app multi-modal experiences, mini-apps orchestrate complex multi-step user onboarding, game micro-interactions, and multi-element transaction confirmations. Developers use Animation playback controls to pause or reverse animations mid-flight when a user abruptly dismisses an interactive sheet or cancels an ongoing gesture. The super-app container orchestrates global lifecycle events: when a mini-app is pushed to the background, the container suspends active animations by iterating over 'document.getAnimations()' and invoking pause(), preventing unnecessary GPU battery drain during background residence.

---
### [web_animations_072_03] KeyframeEffect & Composite Operations: Additive Keyframes and Dynamic Property Blending
- **Chủ đề**: `keyframeeffect_and_composite_operations`
- **Phân loại**: High-Performance Rendering & Animation | **Mức độ chứng cứ**: specification
- **Tiêu chuẩn tham chiếu**: W3C Web Animations §4 The KeyframeEffect Interface & §4.4 Composite Operations
- **URL chính thức**: [https://developer.mozilla.org/en-US/docs/Web/API/KeyframeEffect](https://developer.mozilla.org/en-US/docs/Web/API/KeyframeEffect)
- **URL hỗ trợ**: [https://www.w3.org/TR/web-animations-1/](https://www.w3.org/TR/web-animations-1/), [https://drafts.csswg.org/web-animations-1/](https://drafts.csswg.org/web-animations-1/)
- **Tóm tắt cốt lõi**: KeyframeEffect separates animation timing and target values from playback control, supporting advanced composite modes ('replace', 'add', 'accumulate') for layered motion blending.
- **Phân tích kỹ thuật & Kiến trúc Super App**:
  [SPEC FACT] W3C Web Animations §4 defines 'KeyframeEffect' (inheriting from AnimationEffect), decoupling the target element, keyframe values, and timing configuration from the Animation playback controller. A KeyframeEffect can be created standalone: 'new KeyframeEffect(target, keyframes, options)' and dynamically swapped between Animation controllers. Crucially, the specification standardizes 'composite' and per-keyframe 'composite' modes: (1) 'replace' (standard behavior, overwriting underlying style); (2) 'add' (additive blending, adding translation offsets or scaling factors onto existing animated values); (3) 'accumulate' (combines values across successive iterations). The 'iterationComposite' property allows animations to accumulate transformations across multiple loops (e.g., an icon walking across the screen step-by-step). [SUPERAPP ARCHITECTURE] In rich interactive mini-app components (such as bouncy spring list items, draggable chat bubbles, or live-streaming floating hearts), multiple concurrent gestures and state changes act simultaneously on the same DOM element. Under legacy CSS architectures, triggering a second animation canceled the first, resulting in harsh visual snapping. By leveraging KeyframeEffect with 'composite: "add"', mini-app components blend programmatic spring physics on top of ongoing idle breathing animations, delivering fluid, studio-grade mobile UX without glitching.

---
### [web_animations_072_04] Animation Lifecycle Promises & Events: Asynchronous Task Chaining & Zero-Jank Orchestration
- **Chủ đề**: `waapi_promises_and_lifecycle_event_orchestration`
- **Phân loại**: High-Performance Rendering & Animation | **Mức độ chứng cứ**: specification
- **Tiêu chuẩn tham chiếu**: W3C Web Animations §3.5 Asynchronous Animation Operations & Promises
- **URL chính thức**: [https://developer.mozilla.org/en-US/docs/Web/API/Animation/finished](https://developer.mozilla.org/en-US/docs/Web/API/Animation/finished)
- **URL hỗ trợ**: [https://www.w3.org/TR/web-animations-1/](https://www.w3.org/TR/web-animations-1/), [https://drafts.csswg.org/web-animations-1/](https://drafts.csswg.org/web-animations-1/)
- **Tóm tắt cốt lõi**: Animation exposes 'ready' and 'finished' promises alongside lifecycle event targets ('onfinish', 'oncancel', 'onremove'), enabling asynchronous animation sequencing.
- **Phân tích kỹ thuật & Kiến trúc Super App**:
  [SPEC FACT] W3C Web Animations §3.5 equips the Animation interface with native ECMAScript Promises to coordinate asynchronous rendering workflows: (1) 'animation.ready' resolves when the animation has been synchronized with the compositor hardware and is actively rendering its first frame; (2) 'animation.finished' resolves when the animation completes its active playback duration. If the animation is canceled, the finished promise rejects with an 'AbortError' DOMException. These promises allow idiomatic async/await sequencing: 'await anim1.finished; anim2.play();'. In addition to promises, Animation inherits from EventTarget, dispatching standard 'finish', 'cancel', and 'remove' events. [SUPERAPP ARCHITECTURE] Super-app checkout and authentication flows require strict deterministic sequencing: e.g., show biometric passkey prompt -> play success checkmark animation -> reveal digital receipt. Using setTimeout() or transitionend listeners in legacy webviews was inherently unreliable, as transitionend failed to fire if elements were hidden or if properties did not transition. The super-app SDK utilizes 'await animation.finished' within transaction pipelines, guaranteeing that next-step UI views and financial confirmation callbacks execute only after transition animations have completely settled on screen.

---
### [web_animations_072_05] Animation.commitStyles() & replaceState: Preventing CSSOM Memory Leaks in Long-Running Apps
- **Chủ đề**: `animation_commitstyles_and_memory_leak_mitigation`
- **Phân loại**: High-Performance Rendering & Animation | **Mức độ chứng cứ**: specification
- **Tiêu chuẩn tham chiếu**: W3C Web Animations §3.6 Replacing Animations & commitStyles()
- **URL chính thức**: [https://drafts.csswg.org/web-animations-1/](https://drafts.csswg.org/web-animations-1/)
- **URL hỗ trợ**: [https://www.w3.org/TR/web-animations-1/](https://www.w3.org/TR/web-animations-1/), [https://developer.mozilla.org/en-US/docs/Web/API/Animation/commitStyles](https://developer.mozilla.org/en-US/docs/Web/API/Animation/commitStyles)
- **Tóm tắt cốt lõi**: Animation.commitStyles() writes end-state animated values back to element inline styles, while replaceState and remove() prevent unbound memory leaks from accumulated finished animations.
- **Phân tích kỹ thuật & Kiến trúc Super App**:
  [SPEC FACT] Under W3C Web Animations §3.6, animations configured with 'fill: "forwards"' remain active indefinitely in the browser's animation stack to hold their final styling values. In long-running Single Page Applications, repeatedly creating animations with fill: forwards causes an unbound accumulation of animation objects in memory, creating significant memory leaks and steadily slowing down style recalculation pipelines. To solve this, WAAPI specifies 'animation.commitStyles()', which takes the current animated computed values and writes them explicitly into the element's 'attributeStyleMap' or inline 'style' attribute. Once committed, the animation can be safely canceled or removed via 'animation.cancel()'. Additionally, browsers implement automated animation removal: when a new animation completely replaces all properties of an older finished animation, the old animation's 'replaceState' transitions to 'removed', and a 'remove' event is fired. [SUPERAPP ARCHITECTURE] Super-app mini-apps are designed as persistent single-page applications that may stay loaded across hours of user interaction (e.g., navigation apps, food delivery tracking, loyalty reward hubs). If animations configured with fill: forwards are not pruned, the webview memory footprint balloons, triggering low-memory termination from the mobile OS (Android onTrimMemory / iOS jetsam). The super-app store review linter enforces strict memory hygiene: all programmatic animations must either avoid fill: forwards or invoke 'commitStyles()' and 'cancel()' upon completion, ensuring zero memory growth in long-running container sessions.

---

## 4. Khung Kiến trúc Điều phối Hiển thị & Chuyển động trong Super App SDK

### 4.1. Chuẩn hóa Pipeline Thao tác Style với CSS Typed OM
Trong kiến trúc mini-app, mọi thao tác cập nhật vị trí theo cử chỉ cảm ứng (touch tracking, drag-and-drop, card swiping) bắt buộc phải sử dụng `element.attributeStyleMap` thay cho `element.style`:

```javascript
// Chuẩn Super App: Thao tác typed không sinh chuỗi rác, không kích hoạt CSS parser
const card = document.getElementById('swipe-card');

function onTouchMove(deltaX, deltaY, scale) {
  // Biểu diễn transform đa thành phần có kiểu
  const transform = new CSSTransformValue([
    new CSSTranslate(CSS.px(deltaX), CSS.px(deltaY)),
    new CSSScale(CSS.number(scale), CSS.number(scale))
  ]);

  // Ghi trực tiếp vào attributeStyleMap của phần tử
  card.attributeStyleMap.set('transform', transform);

  // Điều chỉnh độ trong suốt mượt mà
  const opacity = CSS.number(Math.max(0, 1 - Math.abs(deltaX) / 300));
  card.attributeStyleMap.set('opacity', opacity);
}
```

### 4.2. Header Morphing & Parallax với Compositor Scroll-driven Animations
Khi người dùng cuộn danh sách hàng hóa trong mini-app, thanh tiêu đề phải co lại và chuyển từ trong suốt sang nền mờ để bảo vệ tầm nhìn nút Capsule của Super App:

```css
/* Container cuộn gốc của Mini-App */
.scroll-container {
  overflow-y: scroll;
  scroll-timeline-name: --page-scroll;
  scroll-timeline-axis: block;
}

/* Thanh Header co giãn đồng bộ trên Compositor Thread */
.mini-app-header {
  position: sticky;
  top: 0;
  height: 64px;
  animation-name: morph-header;
  animation-timing-function: linear;
  animation-fill-mode: both;
  animation-timeline: --page-scroll;
  animation-range: 0px 120px; /* Hoàn tất chuyển đổi sau 120px cuộn */
}

@keyframes morph-header {
  from {
    background-color: rgba(255, 255, 255, 0);
    backdrop-filter: blur(0px);
    box-shadow: 0 0 0 rgba(0, 0, 0, 0);
  }
  to {
    background-color: rgba(255, 255, 255, 0.95);
    backdrop-filter: blur(12px);
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  }
}
```

### 4.3. Điều phối Quy trình Thanh toán với Web Animations API Promises & commitStyles()
Để ngăn ngừa tình trạng nhảy giao diện (UI flickers) hoặc rò rỉ bộ nhớ khi thực hiện chuỗi xác thực sinh trắc học và hiển thị biên lai:

```javascript
async function executeCheckoutAnimationPipeline(sheetElement, badgeElement) {
  // 1. Hạ Bottom Sheet xác thực thanh toán
  const dismissSheet = sheetElement.animate(
    [
      { transform: 'translateY(0%)', opacity: 1 },
      { transform: 'translateY(100%)', opacity: 0 }
    ],
    { duration: 250, easing: 'cubic-bezier(0.32, 0.72, 0, 1)' }
  );
  await dismissSheet.finished;
  sheetElement.style.display = 'none';

  // 2. Hiển thị huy hiệu thanh toán thành công
  badgeElement.style.display = 'block';
  const popBadge = badgeElement.animate(
    [
      { transform: 'scale(0.5)', opacity: 0 },
      { transform: 'scale(1.05)', opacity: 1, offset: 0.7 },
      { transform: 'scale(1)', opacity: 1 }
    ],
    { duration: 400, easing: 'ease-out', fill: 'forwards' }
  );

  // Đợi animation settled trên màn hình
  await popBadge.finished;

  // 3. Chuẩn hóa chống rò rỉ bộ nhớ CSSOM: Ghi giá trị tĩnh và hủy animation object
  popBadge.commitStyles();
  popBadge.cancel();
}
```

---

## 5. Quy chuẩn Kiểm duyệt (Store Review & Linter Gates) cho Mini-App

1. **Gate ANIM-01: Cấm Lắng nghe Sự kiện Scroll Main-Thread cho Mục đích Animation**
   - Mini-app gửi lên store sẽ bị từ chối nếu phát hiện listener `window.addEventListener('scroll', ...)` có thao tác style trực tiếp trên DOM mà không thông qua passive scroll hoặc CSS Scroll-driven Animations.
2. **Gate ANIM-02: Bắt buộc Quản lý Bộ nhớ đối với `fill: forwards`**
   - Quét mã nguồn tĩnh (AST linting): Mọi lệnh gọi `element.animate()` có cấu hình `fill: 'forwards'` phải có cơ chế gọi `commitStyles()` kèm `cancel()`, hoặc phải gán vòng đời rõ ràng để tránh rò rỉ animation stack trong tiến trình webview.
3. **Gate ANIM-03: Kiểm soát An toàn CSS Custom Property Injection**
   - Không cho phép mini-app ghi đè các biến theme hệ thống bằng cách chèn chuỗi không qua kiểm dịch. Mọi token theme phải được đọc và thiết lập qua `attributeStyleMap` với kiểu `CSSUnparsedValue` hoặc `CSSKeywordValue`.
