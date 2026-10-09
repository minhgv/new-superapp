# Milestone 92: WHATWG DOM TreeWalker & NodeFilter, W3C CSS Logical Properties & CSS Motion Path

**Milestone/Iteration**: Iteration 92 (Milestone 92)
**Timestamp**: 2026-10-09
**Status**: Completed
**Domain Scope**: DOM Traversal & Sanitization Filtering, Flow-Relative Internationalized Layout & Declarative Compositor Motion Paths
**Total Canonical Findings Added**: 15 (Cumulative Store Findings: 1,331)

---

## Executive Overview

Iteration 92 establishes comprehensive, normative standards across three critical runtime architecture, internationalization, and mobile interaction domains essential for enterprise super app mini app stores:

1. **WHATWG DOM Living Standard TreeWalker, NodeIterator & NodeFilter Architecture**:
   Standardizes the `TreeWalker` interface providing zero-allocation, live cursor traversal across hierarchical DOM parent, child, and sibling axes. Establishes programmatic factory construction via `Document.createTreeWalker()` with granular `whatToShow` bitmasks and custom `NodeFilter` callbacks to inspect DOM trees, scrub malicious executable script tags, verify accessibility trees, and sanitize rich text before rendering without creating detached subtrees. Standardizes the `NodeIterator` interface and `Document.createNodeIterator()` for flat, linear document-order iteration, eliminating memory spikes from `querySelectorAll('*')` NodeLists during catalog search indexing and text parsing. Standardizes `NodeFilter` bitmasks (`SHOW_ELEMENT`, `SHOW_TEXT`) and tri-state filtering returns (`FILTER_ACCEPT`, `FILTER_REJECT`, `FILTER_SKIP`), enabling security filters to immediately prune unverified third-party custom element subtrees.

2. **W3C CSS Logical Properties and Values Module Level 1 & Multilingual BiDi Flow Architecture**:
   Standardizes CSS Logical Properties and Values Level 1, establishing flow-relative equivalents to physical box model dimensions that map dynamically to block and inline axes based on `writing-mode`, `direction`, and `text-orientation`. Eliminates brittle, duplicate RTL stylesheets and runtime JavaScript layout flipping scripts across regional ASEAN and Middle East deployments. Standardizes `margin-inline` for flow-relative horizontal spacing and icon-to-text separation across LTR and RTL scripts; `padding-inline` for preserving WCAG 2.2 AA compliant 48x48px responsive touch targets and content boundaries; `inset-inline` for flow-relative modal, drawer sheet, and floating action button positioning; and `border-inline` for vertical dividers, callouts, and form validation accents that naturally flip orientation.

3. **W3C CSS Motion Path Module Level 1 & Declarative Trajectory Sandboxing**:
   Standardizes the CSS Motion Path Module Level 1, allowing mini app developers to animate graphical elements and UI components along arbitrary vector curves and geometric trajectories declaratively. Offloads CPU-intensive `requestAnimationFrame` and `setInterval` coordinate calculation loops directly to the browser GPU compositor thread. Standardizes `offset-path` supporting SVG `path()` strings and `ray()` polar coordinates; `offset-distance` for normalized linear progress interpolation from 0% to 100% on the compositor; `offset-rotate` with `auto` tangential orientation, eliminating complex trigonometric `Math.atan2()` calculations in JavaScript render loops; and `offset-anchor` for registering kinetic transform pivots to vector pathways.

---

## Topic 1: WHATWG DOM Living Standard TreeWalker, NodeIterator & NodeFilter Architecture

| ID | Standard / Feature | Normative Specification | Technical Mechanism & Container Relevance | Verified Citation |
|---|---|---|---|---|
| `STANDARDS-WHATWG-DOM-TREEWALKER-INTERFACE` | WHATWG `TreeWalker` Interface | WHATWG DOM Living Standard - Sec 6 | Filtered hierarchical DOM tree traversal across parent, child, and sibling axes with zero heap allocations, enabling safe live inspection of dynamic templates. | [MDN TreeWalker](https://developer.mozilla.org/en-US/docs/Web/API/TreeWalker) |
| `STANDARDS-WHATWG-DOM-CREATE-TREEWALKER` | `Document.createTreeWalker()` Factory Method | WHATWG DOM Living Standard - Sec 6 | Constructs scoped TreeWalker instances bound to root subtrees with whatToShow bitmasks, used by super app security bridges to strip unauthorized script/iframe tags. | [MDN Document.createTreeWalker](https://developer.mozilla.org/en-US/docs/Web/API/Document/createTreeWalker) |
| `STANDARDS-WHATWG-DOM-NODEITERATOR-INTERFACE` | WHATWG `NodeIterator` Interface | WHATWG DOM Living Standard - Sec 6 | Linear document-order sequence traversal providing a flat view of DOM subtrees for efficient keyword search indexing and catalog text scanning. | [MDN NodeIterator](https://developer.mozilla.org/en-US/docs/Web/API/NodeIterator) |
| `STANDARDS-WHATWG-DOM-CREATE-NODEITERATOR` | `Document.createNodeIterator()` Factory Method | WHATWG DOM Living Standard - Sec 6 | Factory method initializing sequential DOM iterators for lightweight chunked node paging via requestIdleCallback, preventing main-thread UI stalls. | [MDN Document.createNodeIterator](https://developer.mozilla.org/en-US/docs/Web/API/Document/createNodeIterator) |
| `STANDARDS-WHATWG-DOM-NODEFILTER-ARCHITECTURE` | `NodeFilter` Interface & Callback Constants | WHATWG DOM Living Standard - Sec 6 | Defines whatToShow bitmasks and tri-state return codes (FILTER_ACCEPT, FILTER_REJECT, FILTER_SKIP), enabling instant pruning of unverified custom element subtrees. | [MDN NodeFilter](https://developer.mozilla.org/en-US/docs/Web/API/NodeFilter) |

---

## Topic 2: W3C CSS Logical Properties and Values Module Level 1 & Multilingual BiDi Flow Architecture

| ID | Standard / Feature | Normative Specification | Technical Mechanism & Container Relevance | Verified Citation |
|---|---|---|---|---|
| `STANDARDS-W3C-CSS-LOGICAL-PROPERTIES-VALUES` | W3C CSS Logical Properties & Values | W3C CSS Logical Properties Level 1 - Sec 1 | Replaces physical box dimensions with flow-relative block and inline properties, enabling a single universal stylesheet to support LTR, RTL, and vertical writing modes. | [MDN CSS Logical Properties](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_logical_properties_and_values) |
| `STANDARDS-W3C-CSS-MARGIN-INLINE` | CSS `margin-inline` Shorthand Property | W3C CSS Logical Properties Level 1 - Sec 4 | Flow-relative inline margin spacing (margin-inline-start/end) automating RTL layout flipping for navigation bars, breadcrumbs, and form field icons. | [MDN margin-inline](https://developer.mozilla.org/en-US/docs/Web/CSS/margin-inline) |
| `STANDARDS-W3C-CSS-PADDING-INLINE` | CSS `padding-inline` Shorthand Property | W3C CSS Logical Properties Level 1 - Sec 4 | Flow-relative internal padding sandboxing preserving WCAG 2.2 AA compliant 48x48px responsive touch targets across internationalized button and input components. | [MDN padding-inline](https://developer.mozilla.org/en-US/docs/Web/CSS/padding-inline) |
| `STANDARDS-W3C-CSS-INSET-INLINE` | CSS `inset-inline` Shorthand Property | W3C CSS Logical Properties Level 1 - Sec 5 | Flow-relative inline positioning for absolute/fixed/sticky elements (inset-inline-start/end), anchoring notification badges and FABs to trailing edges in LTR and RTL. | [MDN inset-inline](https://developer.mozilla.org/en-US/docs/Web/CSS/inset-inline) |
| `STANDARDS-W3C-CSS-BORDER-INLINE` | CSS `border-inline` Shorthand Property | W3C CSS Logical Properties Level 1 - Sec 4 | Flow-relative inline border styling (border-inline-start/end) for vertical callout borders, timeline step indicators, and form field validation rules. | [MDN border-inline](https://developer.mozilla.org/en-US/docs/Web/CSS/border-inline) |

---

## Topic 3: W3C CSS Motion Path Module Level 1 & Declarative Trajectory Sandboxing

| ID | Standard / Feature | Normative Specification | Technical Mechanism & Container Relevance | Verified Citation |
|---|---|---|---|---|
| `STANDARDS-W3C-CSS-MOTION-PATH-MODULE` | W3C CSS Motion Path Module Level 1 | W3C CSS Motion Path Level 1 - Sec 1 | Declarative trajectory animation along arbitrary paths, offloading route tracking and tutorial pointer animations directly to the GPU compositor thread. | [MDN CSS Motion Path](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_motion_path) |
| `STANDARDS-W3C-CSS-OFFSET-PATH` | CSS `offset-path` Property | W3C CSS Motion Path Level 1 - Sec 2 | Specifies vector trajectories via SVG path() strings, ray() polar coordinates, or basic shapes, pre-compiled into GPU vertex buffers for zero-overhead animation. | [MDN offset-path](https://developer.mozilla.org/en-US/docs/Web/CSS/offset-path) |
| `STANDARDS-W3C-CSS-OFFSET-DISTANCE` | CSS `offset-distance` Property | W3C CSS Motion Path Level 1 - Sec 4 | Normalized linear progress (0% to 100%) along the offset-path, providing single-property animation for transit routes and multi-step onboarding journeys. | [MDN offset-distance](https://developer.mozilla.org/en-US/docs/Web/CSS/offset-distance) |
| `STANDARDS-W3C-CSS-OFFSET-ROTATE` | CSS `offset-rotate` Property | W3C CSS Motion Path Level 1 - Sec 5 | Tangential orientation control with auto alignment, rotating icons and vehicle markers to match curve tangents without JavaScript Math.atan2() calculations. | [MDN offset-rotate](https://developer.mozilla.org/en-US/docs/Web/CSS/offset-rotate) |
| `STANDARDS-W3C-CSS-OFFSET-ANCHOR` | CSS `offset-anchor` Property | W3C CSS Motion Path Level 1 - Sec 3 | Kinematic transform pivot registration positioning visual focal points (e.g. map pin tips) precisely along motion path coordinates. | [MDN offset-anchor](https://developer.mozilla.org/en-US/docs/Web/CSS/offset-anchor) |

---

## Super App Store Review & Runtime Enforcement Matrix

| Domain | Mandatory Store Review Requirement | Automated Verification Method | Remediation Action on Violation |
|---|---|---|---|
| **DOM Traversal & Sanitization** | Mini apps parsing or sanitizing dynamic HTML content must use `TreeWalker` or Native Sanitizer APIs rather than unbounded recursive JavaScript traversals. | AST inspection checking for recursive DOM walk patterns; automated runtime audit of `createTreeWalker` invocations. | Reject build; require migration to `createTreeWalker` with explicit `NodeFilter.SHOW_ELEMENT` bitmasks. |
| **Multilingual BiDi Layout** | Mini apps certified for multi-region release must utilize CSS logical properties (`margin-inline`, `padding-inline`, `inset-inline`, `border-inline`) on directional components. | Linter rules flagging physical `margin-left`/`margin-right` on directional selectors; automated RTL screenshot diffing. | Flag during review; require refactoring physical offsets to flow-relative CSS logical properties. |
| **Declarative Motion Paths** | Route tracking, delivery map markers, and interactive animated pointers must use CSS Motion Path or WAAPI rather than `setInterval`/`requestAnimationFrame` coordinate loops. | Static analysis scanning for periodic `element.style.left/top` mutation loops; CPU profiling during route animations. | Require migration to `offset-path` and `offset-distance` to ensure smooth 60fps/120fps compositor offloading. |

---

## Verified Citation References

- WHATWG DOM Living Standard — Traversal (`TreeWalker`, `createTreeWalker`, `NodeIterator`, `createNodeIterator`, `NodeFilter`): [https://developer.mozilla.org/en-US/docs/Web/API/TreeWalker](https://developer.mozilla.org/en-US/docs/Web/API/TreeWalker)
- W3C CSS Logical Properties and Values Module Level 1 (`CSS_logical_properties_and_values`, `margin-inline`, `padding-inline`, `inset-inline`, `border-inline`): [https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_logical_properties_and_values](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_logical_properties_and_values)
- W3C CSS Motion Path Module Level 1 (`CSS_motion_path`, `offset-path`, `offset-distance`, `offset-rotate`, `offset-anchor`): [https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_motion_path](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_motion_path)
