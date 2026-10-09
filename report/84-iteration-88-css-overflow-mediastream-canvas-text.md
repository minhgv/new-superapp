# Milestone 88: CSS Overflow & Scrollbars, MediaStreamTrack Governance & Canvas 2D Text Metrics

**Milestone/Iteration**: Iteration 88 (Milestone 88)
**Timestamp**: 2026-10-09
**Status**: Completed
**Domain Scope**: Layout Sandboxing, Media Capture Hardware Governance, and Off-Screen Typographic Rendering
**Total Canonical Findings Added**: 15 (Cumulative Store Findings: 1,271)

---

## Executive Overview

Iteration 88 establishes normative standards across three critical runtime capability pillars essential for high-performance, enterprise-grade mini app containers:

1. **W3C CSS Overflow Module Level 3/4 & Scrollbar Stabilization**: Standardizes declarative layout shift mitigation via `scrollbar-gutter: stable`, eliminating Cumulative Layout Shift (CLS) when dynamic data causes vertical scrollbars to emerge. Standardizes compact mobile viewport optimization via `scrollbar-width: thin | none`, host theme synchronization via `scrollbar-color`, visual ink and focus ring preservation via `overflow-clip-margin`, and strict two-axis container confinement via the `overflow` shorthand.
2. **W3C Media Capture and Streams Track Governance & Content Hint Optimization**: Establishes normative control over real-time camera and microphone pipelines via `MediaStreamTrack.contentHint`, enabling mini apps to signal encoder trade-offs (`motion` vs `detail`) to hardware encoders. Standardizes zero-renegotiation hardware adjustments via `MediaStreamTrack.applyConstraints()`, runtime parameter inspection via `MediaStreamTrack.getSettings()`, device envelope introspection via `MediaStreamTrack.getCapabilities()`, and deterministic multi-device capability negotiation via `MediaTrackConstraints`.
3. **W3C HTML Canvas 2D Context Text Metrics & Advanced Typography Sandboxing**: Standardizes zero-reflow off-screen text layout and measurement via `CanvasRenderingContext2D.measureText()`. Standardizes sub-pixel typographic bounding box alignment via the extended `TextMetrics` interface (`actualBoundingBoxAscent`/`Descent` and `fontBoundingBoxAscent`/`Descent`), cross-platform OpenType kerning via `fontKerning`, declarative spacing parity via `letterSpacing`, and runtime quality vs. framerate optimization via `textRendering`.

---

## Topic 1: W3C CSS Overflow Module Level 3/4 & Scrollbar Stabilization

| ID | Standard / Feature | Normative Specification | Technical Mechanism & Container Relevance | Verified Citation |
|---|---|---|---|---|
| `CSS-SCROLLBAR-GUTTER-LAYOUT-STABILIZATION` | CSS `scrollbar-gutter` | W3C CSS Overflow 4 / CSS Scrollbars 1 | Reserves dedicated gutter space (`stable`, `stable both-edges`) before scrollbars appear. Eliminates CLS jitter in dynamic card feeds and async chat lists. | [MDN scrollbar-gutter](https://developer.mozilla.org/en-US/docs/Web/CSS/scrollbar-gutter) |
| `CSS-SCROLLBAR-WIDTH-FORM-FACTOR-GOVERNANCE` | CSS `scrollbar-width` | W3C CSS Scrollbars Module Level 1 | Configures scrollbar footprint (`auto`, `thin`, `none`) without vendor prefixes. Maximizes mobile screen real estate in tight super app drawers and split views. | [MDN scrollbar-width](https://developer.mozilla.org/en-US/docs/Web/CSS/scrollbar-width) |
| `CSS-SCROLLBAR-COLOR-THEME-SANDBOXING` | CSS `scrollbar-color` | W3C CSS Scrollbars Module Level 1 | Sets `<thumb-color> <track-color>` directly. Harmonizes scrollbar palettes with super app light/dark theme tokens, eliminating thousands of lines of hacky pseudo-selectors. | [MDN scrollbar-color](https://developer.mozilla.org/en-US/docs/Web/CSS/scrollbar-color) |
| `CSS-OVERFLOW-CLIP-MARGIN-FOCUS-PRESERVATION` | CSS `overflow-clip-margin` | W3C CSS Overflow Module Level 3 | Expands clipping boundaries for `overflow: clip` containers by `<length>`. Permits accessibility focus rings and drop shadows to render without establishing unwanted scrollports. | [MDN overflow-clip-margin](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow-clip-margin) |
| `CSS-OVERFLOW-SHORTHAND-CONTAINER-ESTABLISHMENT` | CSS `overflow` Shorthand | W3C CSS Overflow Module Level 3 | Two-axis scroll and clipping configuration (`visible`, `hidden`, `clip`, `scroll`, `auto`). Acts as the fundamental barrier confining mini app DOM elements to host WebView bounds. | [MDN overflow](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow) |

---

## Topic 2: W3C Media Capture and Streams Track Governance & Content Hints

| ID | Standard / Feature | Normative Specification | Technical Mechanism & Container Relevance | Verified Citation |
|---|---|---|---|---|
| `MEDIASTREAMTRACK-CONTENT-HINT-OPTIMIZATION` | `MediaStreamTrack.contentHint` | W3C Media Capture and Streams | Informs hardware encoders whether to prioritize fluid framerate (`motion`) or maximum resolution sharpness (`detail`, `text`). Optimizes eKYC scanning vs live video calling. | [MDN MediaStreamTrack.contentHint](https://developer.mozilla.org/en-US/docs/Web/API/MediaStreamTrack/contentHint) |
| `MEDIASTREAMTRACK-APPLY-CONSTRAINTS-DYNAMIC-TUNING` | `MediaStreamTrack.applyConstraints()` | W3C Media Capture and Streams §4.3.3.7 | Dynamically reconfigures track resolution, framerate, and exposure at runtime without tearing down active media streams or triggering redundant permission prompts. | [MDN MediaStreamTrack.applyConstraints](https://developer.mozilla.org/en-US/docs/Web/API/MediaStreamTrack/applyConstraints) |
| `MEDIASTREAMTRACK-GET-SETTINGS-RUNTIME-AUDITING` | `MediaStreamTrack.getSettings()` | W3C Media Capture and Streams §4.3.3.6 | Synchronously inspects active hardware capture settings (`width`, `height`, `frameRate`, `facingMode`). Enables host diagnostic monitoring of camera resource consumption. | [MDN MediaStreamTrack.getSettings](https://developer.mozilla.org/en-US/docs/Web/API/MediaStreamTrack/getSettings) |
| `MEDIASTREAMTRACK-GET-CAPABILITIES-HARDWARE-SANDBOXING` | `MediaStreamTrack.getCapabilities()` | W3C Media Capture and Streams §4.3.3.5 | Returns hardware capability envelopes (`DoubleRange`, `LongRange`). Allows mini apps to inspect device limits before applying constraints, preventing driver crashes. | [MDN MediaStreamTrack.getCapabilities](https://developer.mozilla.org/en-US/docs/Web/API/MediaStreamTrack/getCapabilities) |
| `MEDIA-TRACK-CONSTRAINTS-SPECIFICATION-FRAMEWORK` | `MediaTrackConstraints` Dictionary | W3C Media Capture and Streams §4.3.4 | Standardizes mandatory `exact` vs preferential `ideal` constraints. Enforces graceful degradation on low-end mobile devices and fragmented Android camera hardware. | [MDN MediaTrackConstraints](https://developer.mozilla.org/en-US/docs/Web/API/MediaTrackConstraints) |

---

## Topic 3: W3C HTML Canvas 2D Context Text Metrics & Advanced Typography

| ID | Standard / Feature | Normative Specification | Technical Mechanism & Container Relevance | Verified Citation |
|---|---|---|---|---|
| `CANVAS-2D-MEASURETEXT-LAYOUT-SANDBOXING` | `CanvasRenderingContext2D.measureText()` | HTML Living Standard §4.12.5.1 | Synchronously measures text off-screen, returning `TextMetrics`. Enables zero-reflow canvas word wrapping and dynamic label sizing without injecting measuring DOM nodes. | [MDN CanvasRenderingContext2D.measureText](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/measureText) |
| `CANVAS-TEXTMETRICS-TYPOGRAPHIC-GEOMETRY` | `TextMetrics` Interface | HTML Living Standard §4.12.5.1 | Exposes sub-pixel typographic bounding boxes (`actualBoundingBoxAscent`/`Descent`, `fontBoundingBoxAscent`/`Descent`). Prevents diacritic clipping in Vietnamese/Thai/Arabic. | [MDN TextMetrics](https://developer.mozilla.org/en-US/docs/Web/API/TextMetrics) |
| `CANVAS-2D-FONT-KERNING-TYPOGRAPHY-CONTROL` | `CanvasRenderingContext2D.fontKerning` | HTML Living Standard §4.12.5.1 | Enables or disables OpenType font kerning (`auto`, `normal`, `none`). Balances typographic aesthetic precision against layout speed during high-throughput canvas rendering. | [MDN CanvasRenderingContext2D.fontKerning](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/fontKerning) |
| `CANVAS-2D-LETTER-SPACING-CANVAS-INTEGRATION` | `CanvasRenderingContext2D.letterSpacing` | HTML Living Standard §4.12.5.1 | Declaratively sets inter-character spacing via CSS length strings (`2px`, `0.1em`). Eliminates costly manual character loop blitting and garbage collection pressure. | [MDN CanvasRenderingContext2D.letterSpacing](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/letterSpacing) |
| `CANVAS-2D-TEXT-RENDERING-OPTIMIZATION` | `CanvasRenderingContext2D.textRendering` | HTML Living Standard §4.12.5.1 | Provides rasterization hints (`optimizeSpeed`, `optimizeLegibility`, `geometricPrecision`). Allows mini apps to conserve battery/GPU during animation and switch to legibility when idle. | [MDN CanvasRenderingContext2D.textRendering](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/textRendering) |

---

## Store Governance & Compliance Verification Rules

1. **Cumulative Layout Shift (CLS) Gating**: Mini apps with dynamic data feeds must declare `scrollbar-gutter: stable` or `scrollbar-gutter: stable both-edges` on scrolling containers. Automated store performance tests reject submissions exhibiting CLS > 0.1 caused by layout resizing upon scrollbar emergence.
2. **Accessible Scrollbar Indicators**: If a mini app specifies `scrollbar-width: none` to save visual space, the container must provide clear visual scroll affordances and ensure full keyboard scrollability (`Tab` and arrow keys) to maintain WCAG 2.2 AA compliance.
3. **Camera Constraint Degradation**: Store linters inspect mini app calls to `applyConstraints()` and `getUserMedia()`. Mini apps must use `ideal` constraints or wrap `exact` constraints with programmatic fallbacks, catching `OverconstrainedError` rejections gracefully.
4. **Content Hint Alignment**: Mini apps capturing identification documents, credit cards, or text OCR must set `track.contentHint = 'detail'`. Real-time video communication or game streaming mini apps must set `track.contentHint = 'motion'`.
5. **Canvas Typography Hygiene**: Mini apps rendering dynamic international text onto canvas surfaces must query `TextMetrics.actualBoundingBoxAscent` and `actualBoundingBoxDescent` to calculate vertical bounds, guaranteeing that diacritics and complex vowel marks are never visually truncated.
