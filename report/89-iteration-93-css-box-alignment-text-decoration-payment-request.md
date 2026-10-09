# Milestone 93: W3C CSS Box Alignment, Text Decoration & Payment Request API Orchestration

## 1. Executive Summary & Scope Overview

Milestone 93 expands the Super App Mini App Store Standard across three fundamental web platform specifications and container governance vectors:
1. **W3C CSS Box Alignment Module Level 3 & Multi-Layout Flow Sandboxing**: Unified declarative alignment model governing space distribution, two-axis container alignment via `place-content`, inline-axis item alignment via `justify-items`, block-axis distribution via `align-content`, and gutter isolation via `row-gap` preventing cross-widget layout bleeds.
2. **W3C CSS Text Decoration Module Level 3/4 & Typographic Emphasis Sandboxing**: Standardized visual text accents, declarative link lines and error styling via `text-decoration-line` and `text-decoration-style`, baseline clearance preserving Southeast Asian diacritics via `text-underline-offset`, descender clipping prevention via `text-decoration-skip-ink: auto`, and East Asian typographic emphasis via `text-emphasis`.
3. **W3C Payment Request API Standard & Transaction Orchestration Architecture**: Standardized web platform payment mediation layer, user-gesture-activated native payment sheet presentation via `PaymentRequest.show()`, strongly-typed ISO 4217 currency contracts via `PaymentRequest`, silent capability detection with anti-fingerprinting protection via `canMakePayment()`, tokenized settlement handoff with 30s timeout contracts via `PaymentResponse`, and redacted shipping address delivery via `PaymentAddress`.

All 15 findings are grounded in official W3C specifications and MDN technical standards, verified HTTP 200, strictly deduplicated, and free of sensitive credentials.

---

## 2. Technical Control Matrix & Canonical Findings

### 2.1 W3C CSS Box Alignment Module Level 3 & Multi-Layout Flow Sandboxing

| Finding ID | Standard Reference | Scope & Core Architecture | Super App Store & Container Control |
|---|---|---|---|
| `STANDARDS-W3C-CSS-BOX-ALIGNMENT-SPEC` | W3C CSS Box Alignment Module Level 3 | Unified flow-relative alignment model across Flexbox, Grid, and Block layouts. Positional, baseline, and distributed alignment without layout-specific hacks. | Replaces JS bounding-box calculation loops with native GPU-accelerated layout rules, preventing Cumulative Layout Shift (CLS) in embedded multi-tenant widgets. |
| `STANDARDS-W3C-CSS-PLACE-CONTENT-SHORTHAND` | W3C CSS Box Alignment Level 3 / MDN place-content | Two-axis container distribution shorthand configuring `align-content` (block axis) and `justify-content` (inline axis) simultaneously. | Standardizes modal sheets, action drawers, and empty-state card centering across diverse form factors (foldables, tablets, desktop). |
| `STANDARDS-W3C-CSS-JUSTIFY-ITEMS-PROPERTY` | W3C CSS Box Alignment Level 3 / MDN justify-items | Controls inline-axis default item alignment inside grid and block containers, establishing subtree defaults. | Establishes strict layout containment boundaries for merchant product cards, ensuring titles, badges, and pricing elements align consistently. |
| `STANDARDS-W3C-CSS-ALIGN-CONTENT-BLOCK-AXIS` | W3C CSS Box Alignment Level 3 / MDN align-content | Controls block-axis track distribution and multi-line flow spacing in grid, flex, and block containers. | Prevents checkout drawers and multi-line service category chips from crowding the system status bar or getting clipped by bottom navigation bars. |
| `STANDARDS-W3C-CSS-ROW-GAP-ISOLATION` | W3C CSS Box Alignment Level 3 / MDN row-gap | Establishes specified gutters between rows in grid, flex, and multi-column layouts without boundary margin collapse. | Guarantees vertical spatial isolation between independent mini app widgets in feed streams, eliminating negative margin compensation bugs. |

### 2.2 W3C CSS Text Decoration Module Level 3/4 & Typographic Emphasis Sandboxing

| Finding ID | Standard Reference | Scope & Core Architecture | Super App Store & Container Control |
|---|---|---|---|
| `STANDARDS-W3C-CSS-TEXT-DECORATION-SPEC` | W3C CSS Text Decoration Module Level 3 | Normative rules for text decoration lines (underlines, overlines, line-throughs) and visual emphasis marks across multilingual typography. | Renders accessible link affordances, strike-through promotional pricing, and security warning underlines natively across multi-brand mini apps. |
| `STANDARDS-W3C-CSS-TEXT-DECORATION-LINE-STYLE` | W3C CSS Text Decoration Level 3 / MDN text-decoration-line | Declarative decoration geometry and stroke styles (solid, double, dotted, dashed, wavy). | Enables KYC and form validation error feedback via wavy underlines conforming to WCAG 2.2 color-independent feedback guidelines without DOM reflows. |
| `STANDARDS-W3C-CSS-TEXT-UNDERLINE-OFFSET` | W3C CSS Text Decoration Level 4 / MDN text-underline-offset | Specifies distance offset of an underline from its typographic baseline position without altering line-height or box-model reflow. | Essential for Vietnamese (dấu nặng), Lao, Thai, and Arabic scripts, ensuring decorative underlines do not collide with or obscure phonetic diacritics. |
| `STANDARDS-W3C-CSS-TEXT-DECORATION-SKIP-INK` | W3C CSS Text Decoration Level 4 / MDN text-decoration-skip-ink | Automatically breaks underline strokes around glyph descenders and ascenders (e.g. 'p', 'g', 'y', 'j', multilingual scripts). | Retains text-decoration-skip-ink: auto as mandatory container default to preserve character legibility across third-party merchant storefronts. |
| `STANDARDS-W3C-CSS-TEXT-EMPHASIS-SHORTHAND` | W3C CSS Text Decoration Level 3 / MDN text-emphasis | Applies typographic emphasis marks (dots, circles, sesame marks) to characters in East Asian scripts (Japanese kenten, Chinese boushi). | Provides native East Asian typography support without injecting redundant span tags or pseudo-elements that inflate DOM memory budgets. |

### 2.3 W3C Payment Request API Standard & Transaction Orchestration Architecture

| Finding ID | Standard Reference | Scope & Core Architecture | Super App Store & Container Control |
|---|---|---|---|
| `STANDARDS-W3C-PAYMENT-REQUEST-SHOW-METHOD` | W3C Payment Request API / MDN PaymentRequest.show | Prompts user agent to present the native payment UI; returns Promise resolving to PaymentResponse upon user authorization. | Security boundary ensuring guest mini apps cannot inject spoofed checkout forms or hijack user payment consent; strictly requires transient user activation. |
| `STANDARDS-W3C-PAYMENT-REQUEST-CONSTRUCTOR` | W3C Payment Request API / MDN PaymentRequest | Accepts paymentMethods, details (PaymentDetailsInit), and options (PaymentOptions) with strict parameter validation. | Enforces strict ISO 4217 currency contracts, immutable transaction metadata snapshots, and super app payment method descriptors. |
| `STANDARDS-W3C-PAYMENT-REQUEST-CAN-MAKE-PAYMENT` | W3C Payment Request API / MDN PaymentRequest.canMakePayment | Asynchronously queries whether supported payment instruments exist, incorporating anti-fingerprinting safeguards. | Allows mini apps to conditionally display 'Pay with SuperApp Wallet' buttons without harvesting user financial app footprints; probes are rate-limited. |
| `STANDARDS-W3C-PAYMENT-RESPONSE-INTERFACE` | W3C Payment Request API / MDN PaymentResponse | Tokenized payment credential container providing transaction payloads and the complete() resolution lifecycle method. | Enforces a mandatory 30-second completion contract; uncompleted promises trigger automated timeout and native payment sheet dismissal. |
| `STANDARDS-W3C-PAYMENT-ADDRESS-INTERFACE` | W3C Payment Request API / MDN PaymentAddress | Standardized physical address container supporting redacted shipping delivery and PII privacy protection. | Implements privacy-by-design, providing delivery fee calculation while withholding full street address details until user authorizes the transaction. |

---

## 3. Super App Architecture & Store Review Implementation

### 3.1 Container Layout & Typographic Isolation Runtime Rules
1. **Layout Engine Frame Budget**: Mini app developers are prohibited from calculating layout alignment via continuous `getBoundingClientRect()` loops inside `requestAnimationFrame`. Containers must mandate declarative CSS Box Alignment (`place-content`, `justify-items`, `align-content`, `row-gap`).
2. **Multilingual Diacritic Guard**: In Vietnamese, Lao, and Thai localized interfaces, webview user agent style sheets must apply `text-underline-offset: 0.15em` and `text-decoration-skip-ink: auto` to prevent underline collisions with lower phonetic tone marks.
3. **WCAG 2.2 Color-Independent Validation**: Form fields in mini app checkout flows must use `text-decoration-style: wavy` or `aria-invalid="true"` in addition to color styling to indicate invalid entries.

### 3.2 Host-Mediated Payment Request Flow & Lifecycle State Machine
```
[ Guest Mini App ]               [ Super App Host Container ]            [ Host Payment Gateway / Wallet ]
        |                                     |                                         |
        |--- canMakePayment() --------------->|                                         |
        |<-- Promise<boolean> (Anti-leak) ----|                                         |
        |                                     |                                         |
        | [ User taps 'Pay Now' ]             |                                         |
        |--- show() with PaymentDetails ----->|                                         |
        |    (Transient Activation Required)  |--- Validate ISO 4217 & Total ---------->|
        |                                     |--- Present Native Payment Sheet ------->|
        |                                     |<-- User Biometric Authorization (PIN) --|
        |<-- Promise<PaymentResponse> --------|                                         |
        |    (Tokenized Payload Delivered)    |                                         |
        |                                     |                                         |
        |--- Complete Settlement Backend ---->|                                         |
        |--- paymentResponse.complete('success') -> Dismiss Native Payment Sheet ------->|
        |    (Enforced <= 30s Timeout)        |                                         |
```

---

## 4. Verification & Audit Trail
- **Findings Count**: 15 new canonical records appended to `state/findings.jsonl` (Milestone 93 total: 1346).
- **URL Verification**: 100% of cited specifications verified via live HTTP 200 checks (MDN & W3C Technical Architecture).
- **Deduplication**: 0 duplicate IDs, 0 duplicate URLs across all historical iterations.
- **Safety**: 0 credentials, secrets, or internal proprietary tokens present in any state or candidate artifacts.
