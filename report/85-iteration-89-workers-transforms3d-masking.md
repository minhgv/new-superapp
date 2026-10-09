# Milestone 89: Web Workers Sandbox, CSS Transforms Level 2 & CSS Masking Architecture

**Milestone/Iteration**: Iteration 89 (Milestone 89)
**Timestamp**: 2026-10-09
**Status**: Completed
**Domain Scope**: Background Thread Concurrency, 3D Spatial Rendering & Vector Clipping Architecture
**Total Canonical Findings Added**: 15 (Cumulative Store Findings: 1,286)

---

## Executive Overview

Iteration 89 establishes comprehensive, normative standards across three critical runtime architecture and visual rendering domains essential for enterprise super app mini app stores:

1. **WHATWG Web Workers Living Standard DedicatedWorker Execution Sandboxing & Thread Lifecycle**:
   Standardizes off-main-thread background execution via the `Worker` constructor interface (`new Worker(scriptURL, options)`), decoupling CPU-intensive data parsing, cryptographic verification, and business rule evaluation from the primary UI rendering pipeline. Standardizes high-throughput inter-thread communication via `Worker.postMessage()` using structured cloning and zero-copy `Transferable` object memory detachment (`ArrayBuffer`, `MessagePort`, `ImageBitmap`). Formalizes the `DedicatedWorkerGlobalScope` execution boundary that completely isolates workers from the host DOM and window object. Enforces container Content Security Policy (CSP) and origin-sandboxed execution on sub-script imports via `WorkerGlobalScope.importScripts()`. Standardizes immediate, deterministic thread teardown and native resource reclamation via `Worker.terminate()` to eliminate background zombie threads and battery drain.

2. **W3C CSS Transforms Module Level 2 & 3D Spatial Rendering Architecture**:
   Establishes normative standards for 3D coordinate space modeling, perspective projection, and GPU-accelerated spatial rendering across mobile host WebViews. Standardizes the CSS Transforms Level 2 specification, defining 4x4 transformation matrices for 3D transformations (`translate3d`, `rotate3d`, `scale3d`). Standardizes spatial context hierarchies via `transform-style: preserve-3d` versus `flat`, preventing 3D scenes from bleeding into host navigation elements. Standardizes depth perception and viewing distance scaling via the `perspective` property. Establishes vanishing point coordinate alignment with host viewport centerlines via `perspective-origin`. Standardizes native GPU backface culling via `backface-visibility: hidden`, eliminating redundant pixel shading passes and visual artifacting during interactive flip-card transitions.

3. **W3C CSS Masking Module Level 1 & Visual Clipping Architecture**:
   Establishes the normative framework for vector clipping, alpha masking, and compound shape compositing directly in the GPU compositor pipeline via the W3C CSS Masking Module Level 1. Standardizes basic shape clipping boundaries and touch hit-testing confinement via `clip-path` (`circle()`, `ellipse()`, `polygon()`, `path()`, `inset()`), preventing out-of-bounds touches from hijacking super app gestures. Standardizes alpha and luminance gradient masking via `mask-image`, enabling smooth horizontal scroll fade-outs and feathered card transitions without downloading heavy raster PNG overlays. Standardizes proportional mask scaling across diverse mobile viewport dimensions via `mask-size`. Standardizes multi-layer Boolean clipping operations via Porter-Duff compositing operators (`add`, `subtract`, `intersect`, `exclude`) using `mask-composite`, replacing heavy JavaScript canvas loops with native hardware-accelerated clipping.

---

## Topic 1: WHATWG Web Workers Living Standard DedicatedWorker Execution Sandboxing & Thread Lifecycle

| ID | Standard / Feature | Normative Specification | Technical Mechanism & Container Relevance | Verified Citation |
|---|---|---|---|---|
| `STANDARDS-WHATWG-WORKER-INTERFACE-CONSTRUCTOR` | Worker Interface Constructor | HTML Living Standard - Sec 10 (Web Workers) | Instantiates a DedicatedWorker running an isolated event loop and separate heap. Offloads heavy parsing and cryptographic operations, preventing UI frame drops and ANR watchdogs. | [MDN Worker Constructor](https://developer.mozilla.org/en-US/docs/Web/API/Worker/Worker) |
| `STANDARDS-WHATWG-WORKER-POSTMESSAGE-COMMUNICATION` | Worker `postMessage()` & Memory Transfer | HTML Living Standard - Sec 10.2 (Dedicated Workers) | Serializes data via structured clone algorithm or transfers buffer ownership via Transferable objects (`ArrayBuffer`, `MessagePort`). Enables zero-copy, low-latency cross-thread communication. | [MDN Worker postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Worker/postMessage) |
| `STANDARDS-WHATWG-DEDICATED-WORKER-GLOBAL-SCOPE` | `DedicatedWorkerGlobalScope` Execution Context | HTML Living Standard - Sec 10.2 (Global Scope) | Dedicated worker execution context lacking direct access to DOM, `window`, and document cookies. Provides a strict architectural security boundary for untrusted or heavy compute. | [MDN DedicatedWorkerGlobalScope](https://developer.mozilla.org/en-US/docs/Web/API/DedicatedWorkerGlobalScope) |
| `STANDARDS-WHATWG-WORKER-IMPORTSCRIPTS-GOVERNANCE` | `WorkerGlobalScope.importScripts()` Governance | HTML Living Standard - Sec 10.3 (WorkerGlobalScope) | Synchronously fetches, compiles, and evaluates sub-scripts inside the worker context. Bound by worker CSP `script-src` directives and origin allowlists to prevent remote code injection. | [MDN importScripts](https://developer.mozilla.org/en-US/docs/Web/API/WorkerGlobalScope/importScripts) |
| `STANDARDS-WHATWG-WORKER-TERMINATE-LIFECYCLE` | `Worker.terminate()` Teardown & Reclamation | HTML Living Standard - Sec 10.2 (Worker Lifecycle) | Immediately aborts worker execution, flushes pending microtasks, and reclaims native thread and memory handles. Crucial for super app `onDestroy`/`onHide` lifecycle management. | [MDN Worker terminate](https://developer.mozilla.org/en-US/docs/Web/API/Worker/terminate) |

---

## Topic 2: W3C CSS Transforms Module Level 2 & 3D Spatial Rendering Architecture

| ID | Standard / Feature | Normative Specification | Technical Mechanism & Container Relevance | Verified Citation |
|---|---|---|---|---|
| `STANDARDS-W3C-CSS-TRANSFORMS-2-SPECIFICATION` | W3C CSS Transforms Level 2 Specification | W3C CSS Transforms Module Level 2 | Extends 2D transforms with 3D affine transformations (`translate3d`, `rotate3d`, `matrix3d`) and 4x4 matrix mathematics. Promotes visual elements to dedicated GPU compositor layers. | [W3C CSS Transforms 2](https://www.w3.org/TR/css-transforms-2/) |
| `STANDARDS-W3C-CSS-TRANSFORM-STYLE-3D-SANDBOXING` | CSS `transform-style` Spatial Hierarchies | W3C CSS Transforms Module Level 2 - Sec 3 | Governs whether child elements render in a shared 3D coordinate space (`preserve-3d`) or flatten into 2D plane (`flat`). Strictly contains 3D visual scenes within mini app boundaries. | [MDN transform-style](https://developer.mozilla.org/en-US/docs/Web/CSS/transform-style) |
| `STANDARDS-W3C-CSS-PERSPECTIVE-VIEWPORT-GEOMETRY` | CSS `perspective` Depth Perception | W3C CSS Transforms Module Level 2 - Sec 2 | Defines viewing distance between viewport and z=0 plane, applying foreshortening to 3D elements. Standardizes depth perception and spatial fidelity across varying screen densities. | [MDN perspective](https://developer.mozilla.org/en-US/docs/Web/CSS/perspective) |
| `STANDARDS-W3C-CSS-PERSPECTIVE-ORIGIN-VANISHING-POINT` | CSS `perspective-origin` Vanishing Point | W3C CSS Transforms Module Level 2 - Sec 2.1 | Aligns 3D projection vanishing point coordinates with host viewport axes (default 50% 50%). Adapts dynamically to portrait/landscape orientations without visual shearing. | [MDN perspective-origin](https://developer.mozilla.org/en-US/docs/Web/CSS/perspective-origin) |
| `STANDARDS-W3C-CSS-BACKFACE-VISIBILITY-OCCLUSION` | CSS `backface-visibility` Culling Sandboxing | W3C CSS Transforms Module Level 2 - Sec 4 | Controls rendering of element reverse face. Setting to `hidden` triggers native GPU backface culling, eliminating overdraw and visual bleed during interactive two-sided flip cards. | [MDN backface-visibility](https://developer.mozilla.org/en-US/docs/Web/CSS/backface-visibility) |

---

## Topic 3: W3C CSS Masking Module Level 1 & Visual Clipping Architecture

| ID | Standard / Feature | Normative Specification | Technical Mechanism & Container Relevance | Verified Citation |
|---|---|---|---|---|
| `STANDARDS-W3C-CSS-MASKING-1-SPECIFICATION` | W3C CSS Masking Level 1 Specification | W3C CSS Masking Module Level 1 | Standardizes vector clipping paths and raster/gradient alpha masking executed in the GPU compositor pipeline, replacing heavy bitmap cutouts with native mathematical compositing. | [W3C CSS Masking 1](https://www.w3.org/TR/css-masking-1/) |
| `STANDARDS-W3C-CSS-CLIP-PATH-VECTOR-SANDBOXING` | CSS `clip-path` Shape & Hit-Testing Sandboxing | W3C CSS Masking Module Level 1 - Sec 3 | Restricts visual painting and touch event hit-testing to explicit geometric paths (`circle`, `polygon`, `path`). Prevents invisible transparent element regions from intercepting touch gestures. | [MDN clip-path](https://developer.mozilla.org/en-US/docs/Web/CSS/clip-path) |
| `STANDARDS-W3C-CSS-MASK-IMAGE-ALPHA-LUMINANCE` | CSS `mask-image` Alpha & Gradient Masking | W3C CSS Masking Module Level 1 - Sec 4 | Modulates element opacity using CSS linear/radial gradients or external SVG masks. Enables smooth scroll fade-out edges and feathered vouchers without loading raster PNG assets. | [MDN mask-image](https://developer.mozilla.org/en-US/docs/Web/CSS/mask-image) |
| `STANDARDS-W3C-CSS-MASK-SIZE-SCALING-GEOMETRY` | CSS `mask-size` Proportional Scaling | W3C CSS Masking Module Level 1 - Sec 4.5 | Standardizes mask layer dimensions (`contain`, `cover`, explicit lengths). Ensures mask imagery scales proportionally across diverse smartphone aspect ratios without raster distortion. | [MDN mask-size](https://developer.mozilla.org/en-US/docs/Web/CSS/mask-size) |
| `STANDARDS-W3C-CSS-MASK-COMPOSITE-BOOLEAN-OPERATIONS` | CSS `mask-composite` Porter-Duff Compositing | W3C CSS Masking Module Level 1 - Sec 4.8 | Standardizes multi-layer Boolean mask combinations using Porter-Duff operators (`add`, `subtract`, `intersect`, `exclude`). Powers intricate ticket cutouts and punch-outs at 60fps. | [MDN mask-composite](https://developer.mozilla.org/en-US/docs/Web/CSS/mask-composite) |

---

## Technical Architecture & Super App Governance Integration

```
+-----------------------------------------------------------------------------------+
|                           Super App Native Container Host                         |
|  +-----------------------------------------------------------------------------+  |
|  |             Host Lifecycle Manager & Resource Watchdog                      |  |
|  |   - Monitors Worker CPU/Memory Consumption (60fps SLA enforcement)          |  |
|  |   - Dispatches Worker.terminate() on onDestroy/onHide & Low Memory Warnings |  |
|  |   - Governs CSP Level 3 Allowlist on importScripts() Sub-Resources          |  |
|  +-----------------------------------------------------------------------------+  |
+------------------------------------------+----------------------------------------+
                                           |
                                           v
+-----------------------------------------------------------------------------------+
|                        Mini App Sandboxed Runtime Context                         |
|                                                                                   |
|  +----------------------------------+   +--------------------------------------+  |
|  |      Main UI Rendering Thread    |   |    Dedicated Worker Thread (self)    |  |
|  |  - DOM Tree & CSSOM Stylesheet   |   |  - Isolated Event Loop & Separate    |  |
|  |  - 3D Transforms (preserve-3d)   |   |    Heap Memory Space                 |  |
|  |  - Perspective Viewport Scaling  |   |  - Heavy Business Logic & Tax Calc   |  |
|  |  - Vector clip-path Hit-Testing  |   |  - Cryptographic Signing & Hashing   |  |
|  |  - GPU Compositor Layering       |   |  - No DOM Access (Strict Security)   |  |
|  +-----------------+----------------+   +-------------------+------------------+  |
|                    |                                        |                     |
|                    +---- postMessage(data, [transfer]) ---->+                     |
|                    |<--- postMessage(result, [transfer]) ---+                     |
+-----------------------------------------------------------------------------------+
```

### Store Review Rules & Developer Verification Checklist

1. **Dedicated Worker Thread Governance**:
   - Mini apps performing CPU-intensive business logic, mathematical modeling, or cryptographic key generation exceeding 50ms of continuous execution must delegate execution to a `DedicatedWorker`.
   - Data payloads exceeding 1MB exchanged between the main UI thread and workers must utilize `Transferable` objects (`ArrayBuffer`, `MessagePort`, `ImageBitmap`) to avoid heap ballooning.
   - All instantiated workers must be explicitly terminated via `worker.terminate()` inside the mini app's teardown hooks (`onDestroy`, `onUnload`) to prevent memory leaks and zombie processes.
   - Calls to `WorkerGlobalScope.importScripts()` must only reference assets packaged within the signed mini app bundle or hosted on pre-approved enterprise CDNs matching the container's strict Content Security Policy.

2. **3D Transforms & Spatial Rendering**:
   - `transform-style: preserve-3d` must be restricted to specific interactive visual components (e.g. card carousels, 3D flips) and must never be applied to root document containers (`<html>`, `<body>`).
   - The `perspective` property must use values >= 300px to ensure realistic optical foreshortening and prevent extreme visual distortion on mobile screens.
   - Interactive two-sided card components must declare `backface-visibility: hidden` on both front and back faces to engage GPU backface culling and eliminate redundant fill rate consumption.

3. **Vector Clipping & CSS Masking**:
   - Interactive custom-shaped elements must declare `clip-path` corresponding to their visual silhouette so that touch hit-testing boundaries accurately align with rendered geometry.
   - Horizontal fading carousels and soft edge transitions must use CSS `mask-image` linear/radial gradients rather than absolute-positioned floating PNG overlay elements.
   - Compound ticket cutout shapes and notched cards must use native `mask-composite` operators (`add`, `subtract`, `intersect`, `exclude`) rather than custom JavaScript canvas pixel loops to guarantee 60fps scrolling.
