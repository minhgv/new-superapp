# Iteration 95: W3C CSS UI Module Level 4, Writing Modes BiDi & CSS Animations Governance

## Executive Overview
Milestone 95 deepens the super-app mini-app container specification across three foundational user-interface, typography, and rendering performance pillars:
1. **W3C CSS Basic User Interface Module Level 4 (CSS UI) & Interaction Sandboxing**: Standardizing `accent-color` form theming with automatic contrast enforcement, `appearance` native UI de-escalation preventing platform spoofing attacks, `caret-color` visibility across dark/light mode switches, `cursor` pointer affordance taxonomies mitigating clickjacking risks, and `user-select` text selection hygiene preventing accidental touch-drag highlights on mobile controls while keeping order confirmation and receipt identifiers copyable.
2. **W3C CSS Writing Modes Level 4 & BiDi Vertical Typography**: Establishing normative internationalization requirements for East Asian vertical layouts (`writing-mode: vertical-rl/vertical-lr`), Middle Eastern right-to-left UI flows (`direction: rtl`), cryptographic/financial text isolation against Trojan Source directional spoofing (`unicode-bidi: isolate`), CJK glyph rotation controls (`text-orientation: mixed/upright`), and Tate-chu-yoko horizontal number combinations in vertical flow (`text-combine-upright: digits 2`).
3. **W3C CSS Animations Module Level 1 & Keyframe Governance**: Governing declarative GPU-accelerated kinetic sequences (`@keyframes`), scoping rules preventing collisions between third-party mini apps and host super-app chrome (`animation-name`), temporal duration bounds mitigating excessive CPU/GPU battery drain on mobile hardware (`animation-duration`), ergonomic acceleration curves aligned with host platform physics (`animation-timing-function`), and declarative lifecycle pausing (`animation-play-state: paused`) during active payment authorization sheets or system modals.

---

## Detailed Findings Analysis

### 1. W3C CSS Basic User Interface Module Level 4 (CSS UI)

| Finding ID | Standard / Feature | Specification Anchor | Normative Container Requirement |
| :--- | :--- | :--- | :--- |
| `STANDARDS-W3C-CSS-ACCENT-COLOR` | CSS `accent-color` Property | W3C CSS UI Level 4 §3 | Mini apps adopting custom accent colors on native form controls (checkboxes, radio buttons, sliders, progress bars) must maintain a minimum 3:1 contrast ratio against container background surfaces to pass accessibility review. |
| `STANDARDS-W3C-CSS-APPEARANCE-CONTROL` | CSS `appearance` Property | W3C CSS UI Level 4 §2 | Mini-app form inputs and custom pickers must declare `appearance: none` when applying custom themes to ensure consistent touch target boundaries across heterogeneous mobile OS versions and prevent platform spoofing. |
| `STANDARDS-W3C-CSS-CARET-COLOR` | CSS `caret-color` Property | W3C CSS UI Level 4 §4 | Mini apps implementing custom dark themes must verify `caret-color` contrast against input background fields; transparent carets or zero-contrast carets are flagged as usability defects during automated review. |
| `STANDARDS-W3C-CSS-CURSOR-TAXONOMY` | CSS `cursor` Property | W3C CSS UI Level 4 §5 | Mini apps running on pointer-enabled super app environments are prohibited from declaring `cursor: none` or custom `url()` cursors outside bounded canvas gaming contexts to mitigate clickjacking. |
| `STANDARDS-W3C-CSS-USER-SELECT-HYGIENE` | CSS `user-select` Property | W3C CSS UI Level 4 §6 | Interactive mini-app controls (buttons, tabs, bottom sheets) should declare `user-select: none`, but order confirmation codes, receipt IDs, and customer service contact details must retain `user-select: text`. |

### 2. W3C CSS Writing Modes Level 4 & BiDi Vertical Typography

| Finding ID | Standard / Feature | Specification Anchor | Normative Container Requirement |
| :--- | :--- | :--- | :--- |
| `STANDARDS-W3C-CSS-WRITING-MODE-ORIENTATION` | CSS `writing-mode` Property | W3C CSS Writing Modes Level 4 §3 | Mini apps deploying vertical writing modes (`vertical-rl`, `vertical-lr`) must utilize CSS logical properties (`margin-block`, `padding-inline`) to prevent container layout clipping on diverse device aspect ratios. |
| `STANDARDS-W3C-CSS-DIRECTION-BIDI-FLOW` | CSS `direction` Property | W3C CSS Writing Modes Level 4 §2 | Mini apps claiming RTL locale support must configure the HTML `dir` attribute or CSS `direction: rtl` at the root container to ensure mirrored navigation, layout symmetry, and back-button alignment. |
| `STANDARDS-W3C-CSS-UNICODE-BIDI-ISOLATION` | CSS `unicode-bidi` Property | W3C CSS Writing Modes Level 4 §2.2 | Dynamic strings in mini-app financial transactions, merchant names, and order references must be wrapped in elements with `unicode-bidi: isolate` (or HTML `<bdi>`) to prevent directional spillover and Trojan Source attacks. |
| `STANDARDS-W3C-CSS-TEXT-ORIENTATION-CJK` | CSS `text-orientation` Property | W3C CSS Writing Modes Level 4 §5 | Vertical mini-app UI components displaying international identifiers or Latin currencies must declare `text-orientation: mixed` to prevent unreadable rotated East Asian characters. |
| `STANDARDS-W3C-CSS-TEXT-COMBINE-UPRIGHT-TATE-CHU-YOKO` | CSS `text-combine-upright` Property | W3C CSS Writing Modes Level 4 §5.3 | Mini apps formatted in vertical writing modes must utilize `text-combine-upright: digits 2` for two-digit dates and hours to maintain consistent vertical column line height and spacing. |

### 3. W3C CSS Animations Module Level 1 & Keyframe Governance

| Finding ID | Standard / Feature | Specification Anchor | Normative Container Requirement |
| :--- | :--- | :--- | :--- |
| `STANDARDS-W3C-CSS-KEYFRAMES-SYNTAX` | CSS `@keyframes` Rule | W3C CSS Animations Level 1 §2 | Mini-app `@keyframes` rules must strictly animate compositor-friendly properties (`transform`, `opacity`, `filter`) rather than reflow-inducing geometry (`width`, `height`, `top`, `margin`). |
| `STANDARDS-W3C-CSS-ANIMATION-NAME-SCOPING` | CSS `animation-name` Property | W3C CSS Animations Level 1 §3.1 | Mini-app build pipelines should prefix custom animation identifiers or encapsulate them inside Shadow DOM / CSS Modules to avoid accidental naming collisions with host chrome styles. |
| `STANDARDS-W3C-CSS-ANIMATION-DURATION-BOUNDS` | CSS `animation-duration` Property | W3C CSS Animations Level 1 §3.2 | Micro-interaction animations in mini apps must not exceed a duration of 500ms; looping ambient animations must automatically pause when `prefers-reduced-motion: reduce` is requested. |
| `STANDARDS-W3C-CSS-ANIMATION-TIMING-EASING` | CSS `animation-timing-function` Property | W3C CSS Animations Level 1 §3.3 | Mini-app navigational transitions and bottom-sheet drawers should utilize standard ease-out or platform-aligned `cubic-bezier` curves rather than jarring linear motion. |
| `STANDARDS-W3C-CSS-ANIMATION-PLAY-STATE-GOVERNANCE` | CSS `animation-play-state` Property | W3C CSS Animations Level 1 §3.8 | The super app container may inject a container-wide style rule (`* { animation-play-state: paused !important; }`) during active modal payment authorization to eliminate competing GPU load and battery drain. |

---

## Store Review & Governance Checklist Updates

1. **Accessibility & Form Aesthetics Gate (CSS UI)**:
   - Automated color contrast checks on form controls with custom `accent-color`.
   - Rejection of custom cursor overrides (`cursor: url(...)`) outside verified game canvases.
   - Enforcement of selectable text on critical receipt and order identifier elements.
2. **Internationalization & BiDi Sanitization Gate**:
   - Verification that user/merchant input fields embed `unicode-bidi: isolate` or `<bdi>` tags to eliminate RTL directional hijacking.
   - Validation of vertical writing mode geometry using logical properties rather than fixed coordinate properties.
3. **Compositor Efficiency & Power Gate (Animations)**:
   - Static analysis of `@keyframes` blocks to reject continuous animations of layout/geometry properties.
   - Verified compliance with `prefers-reduced-motion` media queries.
   - Injection testing for host container `animation-play-state: paused` freeze signals.
