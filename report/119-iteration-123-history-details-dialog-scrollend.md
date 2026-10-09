# Chuyên đề Nghiên cứu Iteration 123: History API, ToggleEvent & Modal/Scrollend Sandboxing

## 1. Bối cảnh & Mục tiêu Kỹ thuật
Trong kiến trúc Super App và Mini App Container đa nền tảng, việc điều phối điều hướng trang đơn (SPA routing), quản lý trạng thái hiển thị thành phần (collapsible disclosure), kiểm soát vòng đời hộp thoại modal và đồng bộ hóa kết thúc cuộn trang (scroll settlement) đòi hỏi các chuẩn giao diện cấp thấp thống nhất. Iteration 123 tập trung chuẩn hóa 3 nhóm năng lực web runtime cốt lõi:
1. **HTML History API & SPA Session Navigation Sandboxing**: Kiểm soát điều hướng phiên, giới hạn kích thước clone trạng thái và ngăn chặn bẫy back-button (pushState/replaceState/popstate/hashchange).
2. **HTML Details & ToggleEvent Lifecycle State Sandboxing**: Chuẩn hóa accordion nhóm độc quyền qua thuộc tính `name` và giao diện sự kiện `ToggleEvent` (oldState/newState) để tối ưu hóa render và lazy-loading.
3. **HTML Dialog Dismissal & Scroll Termination Sandboxing**: Quản lý sự kiện hủy bỏ hộp thoại modal (cancel/close), chuyển giao kết quả có cấu trúc (`returnValue`), và bắt điểm kết thúc cuộn chính xác (`scrollend`) trên Element và Document.

## 2. Bảng Tổng hợp Findings Chuẩn hóa (Iteration 123)

| ID | Tiêu chuẩn / API | Danh mục | Mức độ bằng chứng | URL Tham chiếu |
|---|---|---|---|---|
| `STANDARDS-MDN-HISTORY-API` | **MDN History API: SPA Navigation Lifecycle & Session History Sandboxing** | Interoperability, Offline & Runtime Architecture | `official_documentation` | [https://developer.mozilla.org/en-US/docs/Web/API/History_API](https://developer.mozilla.org/en-US/docs/Web/API/History_API) |
| `STANDARDS-MDN-HISTORY-PUSHSTATE` | **MDN history.pushState(): Client-Side Route State Ingestion & History Stack Sandboxing** | Interoperability, Offline & Runtime Architecture | `official_documentation` | [https://developer.mozilla.org/en-US/docs/Web/API/History/pushState](https://developer.mozilla.org/en-US/docs/Web/API/History/pushState) |
| `STANDARDS-MDN-HISTORY-REPLACESTATE` | **MDN history.replaceState(): In-Place History Modification & Clean Route Sandboxing** | Interoperability, Offline & Runtime Architecture | `official_documentation` | [https://developer.mozilla.org/en-US/docs/Web/API/History/replaceState](https://developer.mozilla.org/en-US/docs/Web/API/History/replaceState) |
| `STANDARDS-MDN-WINDOW-POPSTATE-EVENT` | **MDN Window popstate Event: Session History Traversal & State Restoration** | Interoperability, Offline & Runtime Architecture | `official_documentation` | [https://developer.mozilla.org/en-US/docs/Web/API/Window/popstate_event](https://developer.mozilla.org/en-US/docs/Web/API/Window/popstate_event) |
| `STANDARDS-MDN-WINDOW-HASHCHANGE-EVENT` | **MDN Window hashchange Event: Fragment Identifier Mutation & Anchor Sandboxing** | Interoperability, Offline & Runtime Architecture | `official_documentation` | [https://developer.mozilla.org/en-US/docs/Web/API/Window/hashchange_event](https://developer.mozilla.org/en-US/docs/Web/API/Window/hashchange_event) |
| `STANDARDS-MDN-HTMLDETAILS-TOGGLE-EVENT` | **MDN HTMLDetailsElement toggle Event: Collapsible Content State Mutation Sandboxing** | Interoperability, Offline & Runtime Architecture | `official_documentation` | [https://developer.mozilla.org/en-US/docs/Web/API/HTMLDetailsElement/toggle_event](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDetailsElement/toggle_event) |
| `STANDARDS-MDN-HTMLDETAILS-NAME` | **MDN HTMLDetailsElement.name: Exclusive Accordion Grouping & Native Disclosure Sandboxing** | Interoperability, Offline & Runtime Architecture | `official_documentation` | [https://developer.mozilla.org/en-US/docs/Web/API/HTMLDetailsElement/name](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDetailsElement/name) |
| `STANDARDS-MDN-TOGGLEEVENT` | **MDN ToggleEvent Interface: Unified Declarative State Mutation Event Architecture** | Interoperability, Offline & Runtime Architecture | `official_documentation` | [https://developer.mozilla.org/en-US/docs/Web/API/ToggleEvent](https://developer.mozilla.org/en-US/docs/Web/API/ToggleEvent) |
| `STANDARDS-MDN-TOGGLEEVENT-OLDSTATE` | **MDN ToggleEvent.oldState: Prior Lifecycle State Introspection & Reversibility Auditing** | Interoperability, Offline & Runtime Architecture | `official_documentation` | [https://developer.mozilla.org/en-US/docs/Web/API/ToggleEvent/oldState](https://developer.mozilla.org/en-US/docs/Web/API/ToggleEvent/oldState) |
| `STANDARDS-MDN-TOGGLEEVENT-NEWSTATE` | **MDN ToggleEvent.newState: Target Lifecycle State Verification & Lazy-Loading Triggers** | Interoperability, Offline & Runtime Architecture | `official_documentation` | [https://developer.mozilla.org/en-US/docs/Web/API/ToggleEvent/newState](https://developer.mozilla.org/en-US/docs/Web/API/ToggleEvent/newState) |
| `STANDARDS-MDN-HTMLDIALOG-CANCEL-EVENT` | **MDN HTMLDialogElement cancel Event: Native Escape Interception & State Teardown Sandboxing** | Interoperability, Offline & Runtime Architecture | `official_documentation` | [https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/cancel_event](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/cancel_event) |
| `STANDARDS-MDN-HTMLDIALOG-CLOSE-EVENT` | **MDN HTMLDialogElement close Event: Modal Lifecycle Termination & Result Synchronization** | Interoperability, Offline & Runtime Architecture | `official_documentation` | [https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/close_event](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/close_event) |
| `STANDARDS-MDN-HTMLDIALOG-RETURNVALUE` | **MDN HTMLDialogElement.returnValue: Typed Modal Result Passing & Form Sandboxing** | Interoperability, Offline & Runtime Architecture | `official_documentation` | [https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/returnValue](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/returnValue) |
| `STANDARDS-MDN-ELEMENT-SCROLLEND-EVENT` | **MDN Element scrollend Event: Compositor Scroll Termination & Inertia Settlement Sandboxing** | Interoperability, Offline & Runtime Architecture | `official_documentation` | [https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollend_event](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollend_event) |
| `STANDARDS-MDN-DOCUMENT-SCROLLEND-EVENT` | **MDN Document scrollend Event: Viewport Scroll Cessation & Dynamic Chrome Synchronization** | Interoperability, Offline & Runtime Architecture | `official_documentation` | [https://developer.mozilla.org/en-US/docs/Web/API/Document/scrollend_event](https://developer.mozilla.org/en-US/docs/Web/API/Document/scrollend_event) |

---

## 3. Phân tích Kỹ thuật Chi tiết Từng Tiêu chuẩn

### 3.1. MDN History API: SPA Navigation Lifecycle & Session History Sandboxing (`STANDARDS-MDN-HISTORY-API`)
- **Mô tả ngắn**: Standardizes browser session history manipulation via the History interface, enabling single-page mini-app routing without full page reloads.
- **Danh mục**: `Interoperability, Offline & Runtime Architecture`
- **Tiêu chuẩn tham chiếu**: [https://developer.mozilla.org/en-US/docs/Web/API/History_API](https://developer.mozilla.org/en-US/docs/Web/API/History_API)
- **Chi tiết phân tích & Khuyến nghị Super App**:
  MDN History API specification defines session history manipulation: (1) Exposes window.history interface allowing web applications to navigate backward and forward through user session history; (2) Enables programmatic modification of the URL and session state without causing document reloads or destroying in-memory state; (3) Mandates same-origin security boundaries: attempts to push or replace URLs belonging to a different origin throw a SecurityError DOMException; (4) Super-app runtime architectures sandbox window.history within the mini-app root path, intercepting cross-subpath mutations to ensure navigation remains strictly constrained within registered manifest scope.

### 3.2. MDN history.pushState(): Client-Side Route State Ingestion & History Stack Sandboxing (`STANDARDS-MDN-HISTORY-PUSHSTATE`)
- **Mô tả ngắn**: Documents history.pushState() for adding new state entries to browser history, detailing serializable state object boundaries and URL origin restrictions.
- **Danh mục**: `Interoperability, Offline & Runtime Architecture`
- **Tiêu chuẩn tham chiếu**: [https://developer.mozilla.org/en-US/docs/Web/API/History/pushState](https://developer.mozilla.org/en-US/docs/Web/API/History/pushState)
- **Chi tiết phân tích & Khuyến nghị Super App**:
  MDN history.pushState() specification defines history entry creation: (1) Takes state, unused (title), and optional url parameters, pushing a new entry into the session history stack; (2) State parameter must be serializable via the Structured Clone Algorithm; browsers enforce size limits (typically 2 MiB in Chromium and 640 KiB in Firefox), throwing DataCloneError if serialization fails or size ceiling is breached; (3) Passing cross-origin URLs throws SecurityError; (4) Super-app containers wrap pushState to enforce synthetic history depth caps (e.g., maximum 50 entries) and synchronize route changes with native host navigation headers and custom back-button bars.

### 3.3. MDN history.replaceState(): In-Place History Modification & Clean Route Sandboxing (`STANDARDS-MDN-HISTORY-REPLACESTATE`)
- **Mô tả ngắn**: Standardizes in-place modification of current history entry via replaceState(), preventing redundant history stack bloat during transient query/filter mutations.
- **Danh mục**: `Interoperability, Offline & Runtime Architecture`
- **Tiêu chuẩn tham chiếu**: [https://developer.mozilla.org/en-US/docs/Web/API/History/replaceState](https://developer.mozilla.org/en-US/docs/Web/API/History/replaceState)
- **Chi tiết phân tích & Khuyến nghị Super App**:
  MDN history.replaceState() documentation defines in-place history mutation: (1) Replaces the current session history entry with updated state object and URL without altering stack length; (2) Useful for non-reversible state transitions—such as filtering product catalogs, updating search query parameters, or updating checkout step state; (3) Mitigates 'back-button traps' where excessive pushState calls prevent users from returning to the host super-app shell via hardware or gesture back navigation; (4) Super-app store review criteria flag excessive pushState spam in mini-apps and recommend replaceState for ephemeral state updates to preserve clean back navigation.

### 3.4. MDN Window popstate Event: Session History Traversal & State Restoration (`STANDARDS-MDN-WINDOW-POPSTATE-EVENT`)
- **Mô tả ngắn**: Documents the popstate DOM event dispatched on active window when active history entry changes between entries for the same document.
- **Danh mục**: `Interoperability, Offline & Runtime Architecture`
- **Tiêu chuẩn tham chiếu**: [https://developer.mozilla.org/en-US/docs/Web/API/Window/popstate_event](https://developer.mozilla.org/en-US/docs/Web/API/Window/popstate_event)
- **Chi tiết phân tích & Khuyến nghị Super App**:
  MDN Window popstate event specification defines history traversal: (1) Dispatched when user navigates session history (e.g., via browser Back/Forward buttons, history.back(), history.go()); (2) Event object exposes state property containing a copy of the structured clone state passed to pushState or replaceState; (3) Does not fire upon pushState() or replaceState() invocations, only upon actual history traversal; (4) Super-app host bridges listen to popstate to harmonize native host gestures with mini-app route unstacking, ensuring smooth page transitions and avoiding orphaned sub-view memory leaks.

### 3.5. MDN Window hashchange Event: Fragment Identifier Mutation & Anchor Sandboxing (`STANDARDS-MDN-WINDOW-HASHCHANGE-EVENT`)
- **Mô tả ngắn**: Standardizes the hashchange event fired when URL fragment identifier (#) changes, providing lightweight routing and deep anchor link navigation.
- **Danh mục**: `Interoperability, Offline & Runtime Architecture`
- **Tiêu chuẩn tham chiếu**: [https://developer.mozilla.org/en-US/docs/Web/API/Window/hashchange_event](https://developer.mozilla.org/en-US/docs/Web/API/Window/hashchange_event)
- **Chi tiết phân tích & Khuyến nghị Super App**:
  MDN Window hashchange event documentation specifies fragment tracking: (1) Fired synchronously when the URL fragment identifier changes (either programmatically or via anchor link clicks); (2) Exposes oldURL and newURL read-only properties containing complete URL strings before and after the transition; (3) Operates without server network requests or document teardowns; (4) Super-app security layers inspect hashchange events to sanitize target fragment IDs against DOM XSS vectors (e.g., location.hash sink vulnerabilities) and prevent clickjacking via unexpected internal anchor scrolling.

### 3.6. MDN HTMLDetailsElement toggle Event: Collapsible Content State Mutation Sandboxing (`STANDARDS-MDN-HTMLDETAILS-TOGGLE-EVENT`)
- **Mô tả ngắn**: Standardizes the toggle event dispatched on <details> elements upon expanding or collapsing, synchronizing disclosure widgets and layout reflows.
- **Danh mục**: `Interoperability, Offline & Runtime Architecture`
- **Tiêu chuẩn tham chiếu**: [https://developer.mozilla.org/en-US/docs/Web/API/HTMLDetailsElement/toggle_event](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDetailsElement/toggle_event)
- **Chi tiết phân tích & Khuyến nghị Super App**:
  MDN HTMLDetailsElement toggle event documentation specifies disclosure lifecycle: (1) Dispatched on <details> element whenever its open attribute changes state (either via user click on <summary> or programmatic mutation); (2) Fires asynchronously after the state mutation has settled in the DOM; (3) Exposes oldState ('open' or 'closed') and newState ('open' or 'closed') via the inheriting ToggleEvent interface; (4) Super-app responsive layouts monitor details toggle events to trigger dynamic virtual list height recalculations and lazy-load nested catalog assets only when sections expand.

### 3.7. MDN HTMLDetailsElement.name: Exclusive Accordion Grouping & Native Disclosure Sandboxing (`STANDARDS-MDN-HTMLDETAILS-NAME`)
- **Mô tả ngắn**: Documents HTMLDetailsElement.name property for exclusive accordion behavior, natively grouping multiple <details> elements without JavaScript orchestration.
- **Danh mục**: `Interoperability, Offline & Runtime Architecture`
- **Tiêu chuẩn tham chiếu**: [https://developer.mozilla.org/en-US/docs/Web/API/HTMLDetailsElement/name](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDetailsElement/name)
- **Chi tiết phân tích & Khuyến nghị Super App**:
  MDN HTMLDetailsElement.name documentation specifies native accordion grouping: (1) Reflects the HTML name attribute of the <details> element; (2) Multiple <details> elements sharing the same name string form an exclusive accordion group where opening one automatically closes any previously open sibling; (3) Operates entirely in browser native C++ layout code, eliminating complex custom JavaScript state synchronization logic; (4) Super-app design guidelines mandate using native details[name] for mobile FAQ sheets and settings accordions, reducing main-thread CPU overhead on budget mobile devices.

### 3.8. MDN ToggleEvent Interface: Unified Declarative State Mutation Event Architecture (`STANDARDS-MDN-TOGGLEEVENT`)
- **Mô tả ngắn**: Standardizes the ToggleEvent interface dispatched by collapsible and popover elements, exposing structured oldState and newState transition telemetry.
- **Danh mục**: `Interoperability, Offline & Runtime Architecture`
- **Tiêu chuẩn tham chiếu**: [https://developer.mozilla.org/en-US/docs/Web/API/ToggleEvent](https://developer.mozilla.org/en-US/docs/Web/API/ToggleEvent)
- **Chi tiết phân tích & Khuyến nghị Super App**:
  MDN ToggleEvent interface specification defines state transition data: (1) Represents events that notify when an element toggles between open/closed states (e.g., <details> elements, popovers, and dialogs); (2) Inherits from standard Event interface, carrying read-only oldState and newState string properties; (3) Guaranteed to dispatch consistently regardless of whether the toggle was initiated via declarative HTML attributes, CSS invokers, or imperative script calls; (4) Super-app analytics pipelines hook into ToggleEvent to capture standardized telemetry on user interaction flows and expandable UI section dwell times.

### 3.9. MDN ToggleEvent.oldState: Prior Lifecycle State Introspection & Reversibility Auditing (`STANDARDS-MDN-TOGGLEEVENT-OLDSTATE`)
- **Mô tả ngắn**: Documents ToggleEvent.oldState property indicating the prior state of a toggling UI element ('open' or 'closed'), supporting state rollbacks.
- **Danh mục**: `Interoperability, Offline & Runtime Architecture`
- **Tiêu chuẩn tham chiếu**: [https://developer.mozilla.org/en-US/docs/Web/API/ToggleEvent/oldState](https://developer.mozilla.org/en-US/docs/Web/API/ToggleEvent/oldState)
- **Chi tiết phân tích & Khuyến nghị Super App**:
  MDN ToggleEvent.oldState documentation specifies antecedent state introspection: (1) Read-only string property returning 'open' or 'closed' indicating the state before the toggle event occurred; (2) Enables deterministic state rollback routines if a subsequent async condition (e.g., authentication check, network confirmation) fails after user toggles an accordion or popover; (3) Facilitates accurate directional animations (expanding vs collapsing transitions) in hybrid animation controllers; (4) Super-app UI frameworks inspect oldState to ensure state transitions follow valid lifecycle paths and prevent corrupted double-open states.

### 3.10. MDN ToggleEvent.newState: Target Lifecycle State Verification & Lazy-Loading Triggers (`STANDARDS-MDN-TOGGLEEVENT-NEWSTATE`)
- **Mô tả ngắn**: Documents ToggleEvent.newState property indicating target state of a toggling UI element ('open' or 'closed'), gating deferred asset loading.
- **Danh mục**: `Interoperability, Offline & Runtime Architecture`
- **Tiêu chuẩn tham chiếu**: [https://developer.mozilla.org/en-US/docs/Web/API/ToggleEvent/newState](https://developer.mozilla.org/en-US/docs/Web/API/ToggleEvent/newState)
- **Chi tiết phân tích & Khuyến nghị Super App**:
  MDN ToggleEvent.newState specification defines succeeding state introspection: (1) Read-only string property returning 'open' or 'closed' indicating the state the element is transitioning into; (2) When newState === 'open', mini-app components trigger lazy-loading of embedded images, iframes, or heavy WebGL canvases; (3) When newState === 'closed', components pause background timers, pause embedded video streams, and de-allocate expensive visual resources; (4) Super-app resource governance frameworks mandate listening to newState to ensure memory-intensive resources are discarded when collapsible containers close.

### 3.11. MDN HTMLDialogElement cancel Event: Native Escape Interception & State Teardown Sandboxing (`STANDARDS-MDN-HTMLDIALOG-CANCEL-EVENT`)
- **Mô tả ngắn**: Standardizes the cancel event fired when user attempts to close a native modal <dialog> via the Escape key, allowing graceful validation or cancellation veto.
- **Danh mục**: `Interoperability, Offline & Runtime Architecture`
- **Tiêu chuẩn tham chiếu**: [https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/cancel_event](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/cancel_event)
- **Chi tiết phân tích & Khuyến nghị Super App**:
  MDN HTMLDialogElement cancel event documentation defines dismissal interception: (1) Dispatched directly to an open <dialog> element when user triggers a native dismiss gesture (such as pressing the Escape key on hardware keyboards or sending back key events); (2) Event is cancelable: calling event.preventDefault() suppresses dialog dismissal, allowing mini-apps to prompt 'Discard unsaved changes?' confirmations before closing; (3) Followed by the close event if not canceled; (4) Super-app host review policies require mini-apps to handle cancel events gracefully rather than unconditionally trapping users within non-dismissable full-screen modal overlays.

### 3.12. MDN HTMLDialogElement close Event: Modal Lifecycle Termination & Result Synchronization (`STANDARDS-MDN-HTMLDIALOG-CLOSE-EVENT`)
- **Mô tả ngắn**: Documents the close event fired when a native <dialog> element closes, providing deterministic synchronization of return values and DOM focus cleanup.
- **Danh mục**: `Interoperability, Offline & Runtime Architecture`
- **Tiêu chuẩn tham chiếu**: [https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/close_event](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/close_event)
- **Chi tiết phân tích & Khuyến nghị Super App**:
  MDN HTMLDialogElement close event specification defines modal termination callbacks: (1) Dispatched when a dialog closes—whether via dialog.close(), form submission with method='dialog', or keyboard escape; (2) Non-cancelable event dispatched after dialog has been removed from the top layer and its open attribute removed; (3) Enables mini-apps to read dialog.returnValue and process user selection or form input; (4) Automatically restores keyboard focus to previously focused element outside the dialog, maintaining compliance with WCAG 2.2 focus management accessibility standards across super-app mini-apps.

### 3.13. MDN HTMLDialogElement.returnValue: Typed Modal Result Passing & Form Sandboxing (`STANDARDS-MDN-HTMLDIALOG-RETURNVALUE`)
- **Mô tả ngắn**: Documents HTMLDialogElement.returnValue property storing return value of a closed dialog, enabling clean data passing between modal and caller contexts.
- **Danh mục**: `Interoperability, Offline & Runtime Architecture`
- **Tiêu chuẩn tham chiếu**: [https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/returnValue](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/returnValue)
- **Chi tiết phân tích & Khuyến nghị Super App**:
  MDN HTMLDialogElement.returnValue specification defines modal output propagation: (1) String property reflecting the value passed to dialog.close(returnValue) or the value of the submitting button in a <form method='dialog'>; (2) Persists after dialog closure, allowing parent view controllers to retrieve user choice synchronously during or after the close event; (3) Eliminates hacky global variable leakage or detached promise-based monkey-patching; (4) Super-app UI component libraries standardize on dialog.returnValue to cleanly pass selected payment methods, address tokens, or consent choices back to host orchestrators.

### 3.14. MDN Element scrollend Event: Compositor Scroll Termination & Inertia Settlement Sandboxing (`STANDARDS-MDN-ELEMENT-SCROLLEND-EVENT`)
- **Mô tả ngắn**: Standardizes the scrollend event fired on Elements when a scroll operation has completed, eliminating error-prone debounce timers for scroll settlement.
- **Danh mục**: `Interoperability, Offline & Runtime Architecture`
- **Tiêu chuẩn tham chiếu**: [https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollend_event](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollend_event)
- **Chi tiết phân tích & Khuyến nghị Super App**:
  MDN Element scrollend event specification defines scroll completion telemetry: (1) Dispatched on an Element when scrolling has completed—including programmatic scrolls, user touch flings, mousewheel inertia, and keyboard navigation; (2) Fires precisely when visual scroll offset has fully settled and all momentum has ceased; (3) Eliminates legacy patterns relying on setTimeout(..., 150) scroll listener debouncing which caused false-positive triggers and excessive CPU wakeups; (4) Super-app virtualized lists and carousel banners use element scrollend to safely initiate heavy DOM hydration and lazy image decodes only after kinetic movement stops.

### 3.15. MDN Document scrollend Event: Viewport Scroll Cessation & Dynamic Chrome Synchronization (`STANDARDS-MDN-DOCUMENT-SCROLLEND-EVENT`)
- **Mô tả ngắn**: Documents the scrollend event fired on Document when document-level viewport scrolling ceases, coordinating host header collapse and pull-to-refresh resets.
- **Danh mục**: `Interoperability, Offline & Runtime Architecture`
- **Tiêu chuẩn tham chiếu**: [https://developer.mozilla.org/en-US/docs/Web/API/Document/scrollend_event](https://developer.mozilla.org/en-US/docs/Web/API/Document/scrollend_event)
- **Chi tiết phân tích & Khuyến nghị Super App**:
  MDN Document scrollend event specification defines viewport-level scroll cessation: (1) Dispatched to Document when the scrolling of the main document root has concluded; (2) Bubbles up from scrolling containers to Document, allowing centralized observation of viewport motion across all nested sub-views; (3) Enables super-app host shells to synchronize native top bar collapse/expand animations without stuttering during active user gestures; (4) Integrates with analytics engines to measure true user read/dwell time across long-form catalog pages by tracking the delta between scroll start and document scrollend events.

---
## 4. Tác động Kiến trúc & Hướng dẫn Thực thi Super App Container
1. **Kiểm soát Điều hướng SPA & Chống Bẫy Lùi Trang (Back-button Traps)**:
   - Container SDK phải bọc `history.pushState` và `history.replaceState` để áp trần độ sâu lịch sử (khuyến nghị <= 50 mục). Các thao tác cập nhật bộ lọc hoặc phân trang danh mục phải ưu tiên sử dụng `replaceState` thay vì `pushState` liên tục, đảm bảo người dùng có thể quay lại trang chủ Super App chỉ bằng 1 cử chỉ back.
   - Mọi nỗ lực điều hướng ngoài phạm vi subpath đã đăng ký trong Manifest phải bị chặn ngay tại tầng runtime bằng SecurityError.
2. **Tối ưu hóa Hiệu năng & Accordion Khai báo với `<details name>` và `ToggleEvent`**:
   - Khuyến khích mini app sử dụng phần tử HTML native `<details name='faq-group'>` thay vì viết script JavaScript để tạo accordion độc quyền. Cơ chế native giảm thiểu 100% chi phí tính toán layout của script bên thứ ba.
   - Lắng nghe `ToggleEvent` với thuộc tính `newState === 'open'` để kích hoạt lazy-loading hình ảnh, và `newState === 'closed'` để giải phóng tài nguyên đồ họa nặng.
3. **Chuẩn hóa Hộp thoại Modal & Kết thúc Cuộn Mượt mà (`scrollend`)**:
   - Modal dialog native (`<dialog>`) bắt buộc phải xử lý sự kiện `cancel` khi người dùng nhấn phím Escape hoặc cử chỉ back. Không cho phép mini app khóa cứng người dùng trong modal mà không có nút hủy hợp lệ.
   - Thay thế hoàn toàn các giải pháp debounce hẹn giờ `setTimeout(..., 150)` bằng sự kiện native `scrollend` trên container và document. Việc này giúp loại bỏ hiện tượng giật khung hình (frame drops) và đảm bảo quá trình hydrate danh sách cuộn ảo chỉ diễn ra khi động lượng vật lý đã dừng hoàn toàn.
