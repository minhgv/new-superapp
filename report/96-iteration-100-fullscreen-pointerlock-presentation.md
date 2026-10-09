# 96. Iteration 100: Fullscreen API, Pointer Lock 2.0 & Presentation API Multi-Screen Sandboxing

## 1. Executive Context & Scope
Milestone 100 marks a monumental checkpoint in the Super App Mini App Store Standard research program, reaching **1,451 validated, source-grounded findings** across platform architecture, security sandboxing, container runtime ergonomics, and hardware peripheral governance.

This iteration addresses critical hardware-software display and pointer interaction boundaries:
1. **Fullscreen API Living Standard & Screen Orientation API Governance**: Top-layer DOM immersion, user activation requirements, native back-navigation coordination, orientation locking (`ScreenOrientation.lock()`), and dynamic rotation telemetry (`ScreenOrientation.angle`).
2. **Pointer Lock 2.0 API & Raw Cursor Sandboxing**: Mouse cursor capture (`Element.requestPointerLock()`), modal reclamation (`Document.exitPointerLock()`), pointer lock state change events, and raw high-precision delta movement tracking (`MouseEvent.movementX/Y`) for WebGL/WebGPU games and 3D enterprise visualizers.
3. **Presentation API & Multi-Screen Display Orchestration**: Secondary screen projection (`PresentationRequest`), native display picker orchestration (`start()`), bi-directional message routing (`PresentationConnection`), auxiliary receiver runtimes (`PresentationReceiver`), and adaptive display discovery (`PresentationAvailability`).

---

## 2. Core Architectural Standards & Container Governance

### 2.1 Fullscreen API & Screen Orientation Governance
- **Top-Layer Elevation & User Activation**: Mini apps frequently invoke fullscreen modes for media players, presentations, and games. Under the Fullscreen API Living Standard, `Element.requestFullscreen()` elevates elements to the browser's top layer. Super app containers must enforce transient user activation (direct click/touch handler) and enforce `fullscreen` Permissions Policy permissions. Containers must prohibit mini apps from emulating system status bars or host payment sheets to prevent phishing.
- **Back-Button & Gesture Synchronization**: Calling `Document.exitFullscreen()` deterministicly releases top-layer immersion. Containers must bind native OS back navigation (Android back gesture / back button) to `exitFullscreen()` before popping the mini-app route, preventing user trapping inside rogue mini-app views.
- **Orientation Control**: Mini apps require landscape orientation for media and gaming. `ScreenOrientation.lock()` allows locking display orientation, but containers must automatically release locks (`unlock()`) when the mini app is backgrounded or destroyed. `ScreenOrientation.angle` provides real-time rotation telemetry for viewfinder transforms and canvas alignment.

### 2.2 Pointer Lock 2.0 API & Raw Motion Telemetry
- **Mouse Capture for 3D & Immersive Canvases**: Desktop and tablet super apps (e.g. iPadOS with trackpad / desktop webview containers) require raw mouse movement for CAD tools and 3D games. `Element.requestPointerLock()` captures mouse movement beyond display boundaries.
- **Escape Hatch & Modal Protection**: Containers must enforce the standard `Esc` key release mechanism and intercept native host dialogs (e.g., incoming payment sheet, phone call overlay) by triggering `Document.exitPointerLock()`.
- **State Telemetry**: Mini-app engines must listen to `pointerlockchange` events to pause simulation loops and show UI pause menus when lock is surrendered. High-precision deltas via `MouseEvent.movementX` and `movementY` deliver sub-pixel cursor updates directly from hardware packets without DOM reflow penalties.

### 2.3 Presentation API & Second-Screen Auxiliary Run-Times
- **Target Allowlisting**: Retail super apps (POS terminals, customer checkout displays, classroom tools) use `PresentationRequest` to render auxiliary views on connected HDMI screens, Chromecast, or AirPlay displays. Containers must enforce domain allowlists on presentation target URLs in `manifest.json`.
- **Native Picker Orchestration**: `PresentationRequest.start()` opens the native system screen picker sheet subject to transient user gesture constraints, preventing unauthorized projection in public environments.
- **Bi-Directional State & Messaging**: `PresentationConnection` maintains an active, low-latency communication pipe (`send()`, `onmessage`, `close()`) between the controlling handheld mini app and the presenting screen context. `PresentationReceiver` provides the secondary display sandbox with message listeners, while `PresentationAvailability` enables the mini app to dynamically display 'Cast to Screen' buttons only when external monitors are present.

---

## 3. Standardized Normative Control Matrix

| ID | Specification & Standard | Primitive / API | Container Sandboxing Requirement | Store Review Gate & Rule |
| :--- | :--- | :--- | :--- | :--- |
| `STANDARDS-ELEMENT-REQUESTFULLSCREEN` | Fullscreen API Living Standard | `Element.requestFullscreen()` | Top-layer DOM immersion; require transient user activation; enforce Permissions Policy `fullscreen`. | Mini apps must only invoke requestFullscreen() inside direct user gesture handlers; full-screen UI traps fail review. |
| `STANDARDS-DOCUMENT-EXITFULLSCREEN` | Fullscreen API Living Standard | `Document.exitFullscreen()` | Coordinate with native OS back button and swipe-to-dismiss gestures. | Mini apps must provide visible exit affordances or support system back events to exit fullscreen. |
| `STANDARDS-DOCUMENT-FULLSCREENELEMENT` | Fullscreen API Living Standard | `Document.fullscreenElement` | Synchronous top-layer inspection; scope across shadow boundaries. | Container SDK must verify fullscreen state before rendering floating system alerts or native action sheets. |
| `STANDARDS-SCREENORIENTATION-LOCK` | Screen Orientation API | `ScreenOrientation.lock()` | Restrict orientation locking to authorized manifest capabilities; auto-unlock on blur. | Mini apps must release locks upon view unmount and gracefully handle lock rejection. |
| `STANDARDS-SCREENORIENTATION-ANGLE` | Screen Orientation API | `ScreenOrientation.angle` | Passive display rotation telemetry synced with hardware v-sync. | Camera viewfinders and canvas games must read orientation angle to prevent inverted projections. |
| `STANDARDS-ELEMENT-REQUESTPOINTERLOCK` | Pointer Lock 2.0 | `Element.requestPointerLock()` | Capture cursor delta movement; enforce transient user activation and Esc key unlock. | Pointer lock requests must be user-initiated; apps must visually instruct users how to exit. |
| `STANDARDS-DOCUMENT-EXITPOINTERLOCK` | Pointer Lock 2.0 | `Document.exitPointerLock()` | Programmatic release invoked when host modal sheets or native menus display. | Apps must exit pointer lock immediately when opening pause menus or yielding focus. |
| `STANDARDS-DOCUMENT-POINTERLOCKELEMENT` | Pointer Lock 2.0 | `Document.pointerLockElement` | Real-time element inspection across container shadow DOM boundaries. | Container gesture routers must inspect pointerLockElement before dispatching overlay touch events. |
| `STANDARDS-DOCUMENT-POINTERLOCKCHANGE-EVENT` | Pointer Lock 2.0 | `pointerlockchange` event | Dispatched asynchronously upon lock acquisition or surrender. | Mini apps must bind to pointerlockchange to pause game loops and render menus upon lock loss. |
| `STANDARDS-MOUSEEVENT-MOVEMENTX-MOVEMENTY` | Pointer Lock 2.0 | `MouseEvent.movementX / Y` | Unbounded relative delta mouse movement bypassing display screen edges. | Mini apps must sanitize delta coordinates against numerical overflow in physics calculation pipelines. |
| `STANDARDS-PRESENTATION-REQUEST-CONSTRUCTOR` | Presentation API | `new PresentationRequest(urls)` | Multi-screen projection target URL validation against manifest allowlists. | All target presentation URLs must be declared in the mini-app security manifest. |
| `STANDARDS-PRESENTATION-REQUEST-START` | Presentation API | `PresentationRequest.start()` | Native UI display picker prompt requiring transient user activation. | Silent or background attempts to initiate presentation screens are strictly rejected. |
| `STANDARDS-PRESENTATION-CONNECTION` | Presentation API | `PresentationConnection` | Bi-directional message pipe and lifecycle state machine (`connected`, `closed`, `terminated`). | Mini apps must clean up message handlers upon connection termination to avoid memory leaks. |
| `STANDARDS-PRESENTATION-RECEIVER` | Presentation API | `PresentationReceiver` | Auxiliary display browsing context runtime with sanitized DOM rendering. | Mini-app receiver pages must sanitize incoming presentation messages to prevent auxiliary XSS. |
| `STANDARDS-PRESENTATION-AVAILABILITY` | Presentation API | `PresentationAvailability` | Dynamic screen availability polling throttled during background/power-save states. | Mini apps must conditionally render presentation buttons based on `availability.value`. |

---

## 4. Verification & Audit Trail
- **Total Validated Canonical Findings**: 1,451 findings.
- **New Canonical Findings in Iteration 100**: 15 findings across 15 unique, verified URLs (MDN Web Docs / W3C Working Drafts / WHATWG Standards).
- **Security & Hygiene Scan**: Clean; zero credentials, secrets, or confidential tokens.
