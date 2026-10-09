# Iteration 96: W3C CSS Scroll Anchoring, Pointer Events Hit-Testing, Inline & Ruby Typography, and Backgrounds/Borders Governance

## Executive Overview
Milestone 96 deepens the super-app mini-app container standard across three foundational web platform, touch-interaction, multilingual typography, and visual boundary domains:
1. **W3C CSS Scroll Anchoring Module Level 1 & W3C Pointer Events Level 3 Hit Testing**: Standardizing `overflow-anchor` for Cumulative Layout Shift (CLS) elimination during dynamic ad, merchant card, and chat message insertion; `pointer-events: none` for secure, non-blocking host container HUD overlays and clickjacking prevention; `touch-action` for declarative gesture arbitration resolving conflicts with host edge-swipe navigation; `PointerEvent.pointerType` for hardware input disambiguation across touch, stylus pen, and mouse; and `PointerEvent.isPrimary` for multi-touch master pointer arbitration preventing duplicate double-tap checkout race conditions.
2. **W3C CSS Inline Layout Module Level 3 & W3C CSS Ruby Layout Module Level 1 Typographic Sandboxing**: Establishing normative internationalization requirements for editorial drop-caps and first-letter baselines (`initial-letter`) conforming to dynamic type; multi-script vertical baseline harmonization (`dominant-baseline`) across Latin, CJK, Indic, and Arabic alphabets; 1px optical baseline alignment (`alignment-baseline`) for status badges and security lock icons; phonetic pronunciation placement (`ruby-position: over/under`) for furigana and pinyin in East Asian legal contracts; and phonetic text distribution (`ruby-align`) preventing container horizontal overflow.
3. **W3C CSS Backgrounds and Borders Module Level 3 & CSS Fragmentation Module Level 3 Visual Framing Sandboxing**: Standardizing `background-clip` for painting area containment and gradient text masking; `border-image` 9-slice scalable decorative skinning for promotional coupons and mini-game HUDs; `border-image-slice` corner partition offsets and interior background fills; `border-radius` mathematical corner curvature harmonizing with host container design tokens; and `box-decoration-break: clone` for multiline inline badge border and padding preservation.

---

## Detailed Findings Analysis

### 1. W3C CSS Scroll Anchoring Module Level 1 & Pointer Events Level 3 Hit Testing

| Finding ID | Standard / Feature | Specification Anchor | Normative Container Requirement |
| :--- | :--- | :--- | :--- |
| `STANDARDS-W3C-CSS-OVERFLOW-ANCHOR` | CSS `overflow-anchor` Property | W3C CSS Scroll Anchoring Level 1 §3 | Mini-app scrollable containers that stream dynamic content must allow default `overflow-anchor: auto` or explicitly manage virtualized pinning to keep Cumulative Layout Shift (CLS) scores below 0.1. |
| `STANDARDS-W3C-CSS-POINTER-EVENTS` | CSS `pointer-events` Property | SVG / CSS Basic UI / Pointer Events | Mini apps using full-viewport transparent or semi-transparent decorative elements must declare `pointer-events: none` to prevent accidental click interception or malicious clickjacking of underlying UI. |
| `STANDARDS-W3C-POINTER-EVENTS-TOUCH-ACTION` | CSS `touch-action` Property | W3C Pointer Events Level 3 §5.2 | Interactive mini-app widgets like digital signature pads, mini-games, and image croppers must declare `touch-action: none` or `manipulation` to guarantee responsive touch tracking and prevent accidental container navigation. |
| `STANDARDS-W3C-POINTER-EVENTS-POINTER-TYPE` | `PointerEvent.pointerType` Property | W3C Pointer Events Level 3 §5.1.1 | Mini apps must not disable mouse or stylus interactions based on mobile UA strings; input handling must inspect `PointerEvent.pointerType` to provide adaptive hit testing across foldables, tablets, and peripheral-attached devices. |
| `STANDARDS-W3C-POINTER-EVENTS-IS-PRIMARY` | `PointerEvent.isPrimary` Property | W3C Pointer Events Level 3 §5.1.1 | Mini-app transaction confirmation buttons and checkout flows must verify `event.isPrimary === true` or enforce single-pointer event handling to prevent multi-tap financial race conditions. |

### 2. W3C CSS Inline Layout Module Level 3 & Ruby Layout Module Level 1 Typographic Sandboxing

| Finding ID | Standard / Feature | Specification Anchor | Normative Container Requirement |
| :--- | :--- | :--- | :--- |
| `STANDARDS-W3C-CSS-INLINE-INITIAL-LETTER` | CSS `initial-letter` Property | W3C CSS Inline Layout Level 3 §6 | Mini apps rendering editorial drop caps must use `initial-letter` or verified responsive CSS rather than fixed-pixel absolute positioning, ensuring compatibility with container user-defined text scaling (dynamic type). |
| `STANDARDS-W3C-CSS-INLINE-DOMINANT-BASELINE` | CSS `dominant-baseline` Property | W3C CSS Inline Layout Level 3 §4 | Multilingual mini apps rendering mixed-script badges or price tags must use standard baseline alignment properties to guarantee vertical alignment parity across CJK and Latin scripts. |
| `STANDARDS-W3C-CSS-INLINE-ALIGNMENT-BASELINE` | CSS `alignment-baseline` Property | W3C CSS Inline Layout Level 3 §4 | Inline status icons and badges in mini-app checkout headers must align to the textual baseline using standard alignment properties, avoiding negative margin workarounds that break across screen densities. |
| `STANDARDS-W3C-CSS-RUBY-POSITION` | CSS `ruby-position` Property | W3C CSS Ruby Layout Level 1 §4 | Mini apps targeting East Asian locales that provide phonetic annotations for legal names or contracts must utilize semantic `<ruby>` and `<rt>` markup with standard `ruby-position` styling rather than detached overlay divs. |
| `STANDARDS-W3C-CSS-RUBY-ALIGN` | CSS `ruby-align` Property | W3C CSS Ruby Layout Level 1 §5 | Phonetic annotations in mini apps must declare appropriate `ruby-align` rules to prevent clipping or visual overlaps in constrained UI surfaces such as bottom sheets and modal confirmation cards. |

### 3. W3C CSS Backgrounds and Borders Module Level 3 & Visual Framing Sandboxing

| Finding ID | Standard / Feature | Specification Anchor | Normative Container Requirement |
| :--- | :--- | :--- | :--- |
| `STANDARDS-W3C-CSS-BACKGROUND-CLIP` | CSS `background-clip` Property | W3C CSS Backgrounds and Borders Level 3 §3.7 | Mini apps adopting translucent borders or gradient typography must use standard `background-clip` properties rather than nested wrapper divs, preserving clean DOM hierarchy and low memory usage. |
| `STANDARDS-W3C-CSS-BORDER-IMAGE` | CSS `border-image` Shorthand | W3C CSS Backgrounds and Borders Level 3 §6 | Mini apps using custom decorative coupon or frame styling must implement CSS `border-image` rather than 9 separate DOM elements, reducing DOM tree depth and paint complexity. |
| `STANDARDS-W3C-CSS-BORDER-IMAGE-SLICE` | CSS `border-image-slice` Property | W3C CSS Backgrounds and Borders Level 3 §6.2 | Mini apps employing resizable card containers with custom art borders must specify valid `border-image-slice` parameters to prevent image distortion and layout reflows during viewport transitions. |
| `STANDARDS-W3C-CSS-BORDER-RADIUS` | CSS `border-radius` Property | W3C CSS Backgrounds and Borders Level 3 §5 | Mini-app modal sheets and floating cards must adopt container design token variables for `border-radius` to preserve platform consistency with host super-app chrome. |
| `STANDARDS-W3C-CSS-BOX-DECORATION-BREAK` | CSS `box-decoration-break` Property | W3C CSS Fragmentation Level 3 / CSS Backgrounds | Inline promotional tags and badges that may wrap across lines on narrow mobile viewports must declare `box-decoration-break: clone` to avoid raw un-padded cut-offs at line boundaries. |

---

## Store Review & Governance Checklist Updates

1. **UX Stability & Touch Arbitration Gate**:
   - Automated audit verifying that scrollable product catalogs and dynamic feeds do not disable `overflow-anchor` without providing a verified virtualized anchor implementation.
   - Validation that interactive custom components declare explicit `touch-action` values (`pan-y`, `manipulation`, `none`) to avoid gesture deadlocks with host container swipe-to-dismiss transitions.
   - Concurrency checking on checkout buttons requiring single-touch validation via `PointerEvent.isPrimary === true`.
2. **Multilingual Typography & Internationalization Gate**:
   - Review of East Asian financial and legal apps to verify standard semantic `<ruby>` elements with valid `ruby-position` and `ruby-align` properties.
   - Automated baseline inspection ensuring that verified merchant badges, security shields, and price currency symbols use `alignment-baseline` or `dominant-baseline` to eliminate optical misalignment across mixed scripts.
3. **Visual Geometry & Performance Gate**:
   - Verification that decorative promotional borders and custom receipt coupon frames leverage CSS `border-image` rather than DOM-heavy 9-slice wrapper hierarchies.
   - Static analysis verifying that inline highlighted badges and discount tags adopt `box-decoration-break: clone` for consistent visual padding upon text wrapping.
   - Token compliance audit ensuring that modal surfaces and card components bind to host container design token variables for `border-radius`.
