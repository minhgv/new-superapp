# Milestone 91: W3C CSS Counter Styles, CSS Custom Highlight API & Touch Events Sandboxing

**Milestone/Iteration**: Iteration 91 (Milestone 91)
**Timestamp**: 2026-10-09
**Status**: Completed
**Domain Scope**: Custom Numbering Systems & Regional Localization, In-Page Text Highlight Sandboxing & Mobile Touch Gesture Sandboxing
**Total Canonical Findings Added**: 15 (Cumulative Store Findings: 1,316)

---

## Executive Overview

Iteration 91 establishes comprehensive, normative standards across three critical runtime architecture, internationalization, and mobile interaction domains essential for enterprise super app mini app stores:

1. **W3C CSS Counter Styles Level 3 & Localization Styling Architecture**:
   Standardizes the `@counter-style` at-rule for defining custom numbering systems and list markers declaratively within CSS stylesheets. Eliminates heavy, brittle JavaScript regex formatters and DOM-mutating localization scripts when rendering non-Latin or specialized numeral systems. Standardizes the algorithmic conversion taxonomy via the `system` descriptor (`numeric`, `alphabetic`, `symbolic`, `additive`, `cyclic`, `fixed`), supporting diverse global and regional numbering paradigms. Standardizes marker glyphs via `symbols` and weighted numerals via `additive-symbols` for Roman numerals and classical counting systems essential in financial, leasing, and statutory contract clauses. Standardizes resilient out-of-range fallback mechanisms via the `fallback` descriptor, ensuring seamless, accessible degradation to decimal numbers when list collections expand unexpectedly.

2. **W3C CSS Custom Highlight API Module Level 1 & Text Interaction Sandboxing**:
   Standardizes the `Highlight` interface and `HighlightRegistry` map dictionary (`CSS.highlights`), enabling mini apps to programmatically highlight and style arbitrary DOM `Range` objects without altering DOM structure or injecting thousands of artificial `<span>` or `<mark>` wrapper elements. Standardizes declarative styling sandboxing via the `::highlight()` pseudo-element, restricting styling properties strictly to text-level visual attributes (color, background-color, text-decoration) that execute purely during the composite paint phase without triggering costly layout reflows. Standardizes multi-layer highlight priority orchestration via `Highlight.priority`, deterministically resolving visual stacking conflicts between real-time catalog search filtering, collaborative user annotations, and super app host security disclosures.

3. **W3C Touch Events Community Group / Living Standard & Mobile Gesture Sandboxing**:
   Establishes normative standards for mobile multi-touch surface interactions and gesture sandboxing. Standardizes the `TouchEvent` interface encapsulating simultaneous contact points (`touches`, `targetTouches`, `changedTouches`), enforcing passive event listener decoupling (`{ passive: true }`) to eliminate main-thread compositor locking during touch scrolls. Standardizes touch contact geometry and pressure telemetry via the `Touch` interface (`force`, `radiusX`, `radiusY`, `rotationAngle`), providing high-fidelity, legally binding digital signature and e-KYC biometric touch capture. Standardizes contact initiation via `touchstart` to eliminate legacy 300ms mobile click delays, and high-frequency continuous gesture tracking via `touchmove` with `requestAnimationFrame` compositor synchronization.

---

## Topic 1: W3C CSS Counter Styles Level 3 & Localization Styling Architecture

| ID | Standard / Feature | Normative Specification | Technical Mechanism & Container Relevance | Verified Citation |
|---|---|---|---|---|
| `STANDARDS-W3C-CSS-COUNTER-STYLE-AT-RULE` | W3C CSS `@counter-style` At-Rule | W3C CSS Counter Styles Level 3 - Sec 2 | Defines declarative custom counter styles accessible via `list-style-type` and `counter()`, eliminating brittle JavaScript DOM numbering mutations across multi-lingual stores. | [MDN CSS @counter-style](https://developer.mozilla.org/en-US/docs/Web/CSS/@counter-style) |
| `STANDARDS-W3C-CSS-COUNTER-STYLE-SYSTEM` | `system` Descriptor: Algorithmic Conversion Taxonomy | W3C CSS Counter Styles Level 3 - Sec 3 | Defines conversion algorithms (`numeric`, `alphabetic`, `symbolic`, `additive`, `cyclic`, `fixed`) executed in native layout C++ code without garbage collection stalls. | [MDN @counter-style/system](https://developer.mozilla.org/en-US/docs/Web/CSS/@counter-style/system) |
| `STANDARDS-W3C-CSS-COUNTER-STYLE-SYMBOLS` | `symbols` Descriptor: Glyphic Marker Definition | W3C CSS Counter Styles Level 3 - Sec 4 | Specifies Unicode symbols or string tokens for positional and cyclic systems, enabling brand-aligned icon markers and Southeast Asian/East Asian script numerals. | [MDN @counter-style/symbols](https://developer.mozilla.org/en-US/docs/Web/CSS/@counter-style/symbols) |
| `STANDARDS-W3C-CSS-COUNTER-STYLE-ADDITIVE-SYMBOLS` | `additive-symbols` Descriptor: Roman & Classical Numerals | W3C CSS Counter Styles Level 3 - Sec 4.2 | Maps weighted numeral tuples (<integer> <symbol>) ordered by descending weight. Essential for statutory legal clauses, leasing contracts, and formal disclosure documents. | [MDN additive-symbols](https://developer.mozilla.org/en-US/docs/Web/CSS/@counter-style/additive-symbols) |
| `STANDARDS-W3C-CSS-COUNTER-STYLE-FALLBACK` | `fallback` Descriptor: Resilient Out-of-Range Recovery | W3C CSS Counter Styles Level 3 - Sec 5 | Specifies a resilient secondary counter style (defaulting to decimal) applied when counter values exceed defined ranges, preventing layout breaks or NaN text. | [MDN fallback](https://developer.mozilla.org/en-US/docs/Web/CSS/@counter-style/fallback) |

---

## Topic 2: W3C CSS Custom Highlight API Module Level 1 & Text Interaction Sandboxing

| ID | Standard / Feature | Normative Specification | Technical Mechanism & Container Relevance | Verified Citation |
|---|---|---|---|---|
| `STANDARDS-W3C-CSS-CUSTOM-HIGHLIGHT-OBJECT` | `Highlight` Interface & Range Encapsulation | CSS Custom Highlight API Level 1 - Sec 2 | Styles arbitrary DOM Range objects without modifying DOM tree hierarchies or injecting wrapper elements, preserving 60fps responsiveness during in-page filtering. | [MDN Highlight API](https://developer.mozilla.org/en-US/docs/Web/API/Highlight) |
| `STANDARDS-W3C-CSS-CUSTOM-HIGHLIGHT-REGISTRY` | `HighlightRegistry` Interface & Map Coordination | CSS Custom Highlight API Level 1 - Sec 3 | Map-like dictionary (`CSS.highlights`) mapping named identifiers to Highlight instances, enabling host capsules and mini apps to maintain isolated highlight layers. | [MDN HighlightRegistry](https://developer.mozilla.org/en-US/docs/Web/API/HighlightRegistry) |
| `STANDARDS-W3C-CSS-CUSTOM-HIGHLIGHT-PSEUDO-ELEMENT` | `::highlight()` Pseudo-Element Declarative Sandbox | CSS Custom Highlight API Level 1 - Sec 4 | Declaratively styles custom highlights (color, background-color, text-decoration). Sandboxed to text-level visual properties, preventing accidental layout reflows. | [MDN ::highlight()](https://developer.mozilla.org/en-US/docs/Web/CSS/::highlight) |
| `STANDARDS-W3C-CSS-HIGHLIGHTS-STATIC-MAP` | `CSS.highlights` Static Entry Point & Feature Gating | CSS Custom Highlight API Level 1 - Sec 3.1 | Canonical global entry point and feature check (`'highlights' in CSS`), enabling mini apps to detect highlight capabilities and provide clean fallbacks on older WebViews. | [MDN CSS.highlights](https://developer.mozilla.org/en-US/docs/Web/API/CSS/highlights_static) |
| `STANDARDS-W3C-CSS-CUSTOM-HIGHLIGHT-PRIORITY` | `Highlight.priority` Multi-Layer Conflict Resolution | CSS Custom Highlight API Level 1 - Sec 2.1 | Integer priority property deterministically ordering visual stacking when multiple highlights overlap the same characters, resolving conflicts between search and annotations. | [MDN Highlight.priority](https://developer.mozilla.org/en-US/docs/Web/API/Highlight/priority) |

---

## Topic 3: W3C Touch Events Community Group / Living Standard & Mobile Gesture Sandboxing

| ID | Standard / Feature | Normative Specification | Technical Mechanism & Container Relevance | Verified Citation |
|---|---|---|---|---|
| `STANDARDS-W3C-TOUCH-EVENT-INTERFACE` | W3C `TouchEvent` Interface & Multi-Touch State | W3C Touch Events CG Report - Sec 5 | Encapsulates touch point collections (`touches`, `targetTouches`, `changedTouches`), enforcing passive listener decoupling to eliminate main-thread scroll blocking. | [MDN TouchEvent](https://developer.mozilla.org/en-US/docs/Web/API/TouchEvent) |
| `STANDARDS-W3C-TOUCH-INTERFACE-GEOMETRY` | W3C `Touch` Interface: Geometry & Force Telemetry | W3C Touch Events CG Report - Sec 3 | Exposes spatial coordinates, touch radius ellipses (`radiusX`/`radiusY`), and normalized touch pressure (`force`), critical for legally binding digital signatures and e-KYC. | [MDN Touch](https://developer.mozilla.org/en-US/docs/Web/API/Touch) |
| `STANDARDS-W3C-TOUCH-LIST-MANAGEMENT` | `TouchList` Interface: Multi-Touch Indexing & Scope | W3C Touch Events CG Report - Sec 4 | Indexed collection of Touch points; enables precise gesture isolation to `targetTouches`, preventing nested mini app components from hijacking host navigation gestures. | [MDN TouchList](https://developer.mozilla.org/en-US/docs/Web/API/TouchList) |
| `STANDARDS-W3C-TOUCHSTART-EVENT-LIFECYCLE` | Element `touchstart` Event & Fast-Click Architecture | W3C Touch Events CG Report - Sec 5.1 | Fires immediately upon physical surface contact prior to synthesized mouse events, eliminating legacy 300ms tap delays and ensuring sub-50ms perceived responsiveness. | [MDN touchstart event](https://developer.mozilla.org/en-US/docs/Web/API/Element/touchstart_event) |
| `STANDARDS-W3C-TOUCHMOVE-EVENT-THROTTLING` | Element `touchmove` Event & Compositor Decoupling | W3C Touch Events CG Report - Sec 5.2 | Continuous gesture tracking synchronized with 60Hz/120Hz display refresh rates, requiring requestAnimationFrame throttling to avoid layout thrashing during drag interactions. | [MDN touchmove event](https://developer.mozilla.org/en-US/docs/Web/API/Element/touchmove_event) |

---

## Technical Architecture & Super App Governance Integration

```
+-----------------------------------------------------------------------------------+
|                           Super App Native Container Host                         |
|  +-----------------------------------------------------------------------------+  |
|  |             Host Gesture & Internationalization Governance Engine            |  |
|  |   - Touch Interception: Scopes targetTouches, Prevents Edge-Swipe Hijack    |  |
|  |   - Highlight Registry Watchdog: Enforces Priority Layering & WCAG Contrast |  |
|  |   - Counter Style Fallback Validator: Ensures Decimal Fallbacks on Lists    |  |
|  +-----------------------------------------------------------------------------+  |
+------------------------------------------+----------------------------------------+
                                           |
                    Platform IPC & Rendering Pipeline
                                           |
+------------------------------------------v----------------------------------------+
|                      Guest Mini App WebView Execution Sandbox                     |
|                                                                                   |
|   +------------------------------------+   +----------------------------------+   |
|   |   Styling & Internationalization   |   |   DOM & Text Interaction Sandbx  |   |
|   | - @counter-style custom numbering  |   | - CSS.highlights Map Registry    |   |
|   | - system: additive / numeric / cyc |   | - Highlight(range1, range2...)   |   |
|   | - symbols: localized Asian glyphs  |   | - ::highlight(search-result)     |   |
|   | - additive-symbols: legal Roman    |   | - priority: 1..10 visual layers  |   |
|   | - fallback: decimal (graceful deg) |   | - Zero DOM wrapper elements      |   |
|   +------------------------------------+   +----------------------------------+   |
|                                                                                   |
|   +---------------------------------------------------------------------------+   |
|   |                     Mobile Touch Gesture Sandboxing                       |   |
|   |   - TouchEvent: touches / targetTouches / changedTouches                  |   |
|   |   - Touch: clientX, clientY, radiusX, radiusY, rotationAngle, force       |   |
|   |   - Passive touchstart / touchmove ({ passive: true }) for 120fps scroll  |   |
|   |   - Non-passive isolated capture for e-Signatures & Digital Contracts     |   |
|   +---------------------------------------------------------------------------+   |
+-----------------------------------------------------------------------------------+
```

---

## Store Review Gating & Verification Rules

1. **Custom Counter Style Localization Rule (`AUDIT-CSS-COUNTER-STYLE-FALLBACK`)**:
   - Automated static analysis scans CSS bundles for `@counter-style` declarations.
   - Any mini app declaring custom counter styles for multi-lingual or regional interfaces must specify an explicit, robust `fallback` descriptor (such as `decimal`).
   - Mini apps utilizing custom font glyphs in `symbols` must declare verifiable font fallbacks or standard Unicode points to eliminate unreadable tofu glyphs across host devices.

2. **DOM Highlight Injection & Reflow Prohibition (`AUDIT-HIGHLIGHT-ZERO-DOM-MUTATION`)**:
   - In-page document search, catalog hit highlighting, or live text annotation features must utilize the CSS Custom Highlight API (`CSS.highlights` and `::highlight()`) where supported.
   - Injecting dynamic `<span>` or `<mark>` tags across large document trees during user keystroke search is flagged during store review due to severe layout invalidation and DOM tree thrashing.
   - Custom highlight styles must verify WCAG 2.2 AA contrast ratios between foreground text and highlight background color tokens.

3. **Touch Listener Passive Execution & Gesture Sandboxing (`AUDIT-TOUCH-PASSIVE-LISTENER`)**:
   - All `touchstart` and `touchmove` listeners registered on root document, window, or body elements must be explicitly declared `{ passive: true }`.
   - Non-passive touch listeners calling `preventDefault()` are restricted exclusively to designated signature canvas elements or custom drawing nodes.
   - Gesture recognizers must isolate touch point tracking to `event.targetTouches` rather than global `event.touches`, preventing third-party mini app widgets from intercepting host navigation drawers or system edge-swipes.
