# Iteration 28: Push Notification Architecture, Host Notification Governance, and Window Controls Overlay

## Overview & Scope

Iteration 28 expands the standard across three critical client-host interface boundaries:
1. **Push Notification Architecture & End-to-End Encryption**: IETF RFC 8030 (Web Push Protocol), RFC 8291 (Message Encryption for Web Push via `aes128gcm`, ECDH P-256, HKDF-SHA-256), RFC 8292 (Voluntary Application Server Identification / VAPID via ES256 JWT), and W3C Push API (`PushManager`, `userVisibleOnly`, endpoint isolation).
2. **Host Notification Governance & OS Badging**: WHATWG Notifications API Living Standard (permissions, tag replacement, action buttons), W3C Badging API (`setAppBadge`, `clearAppBadge`, aggregated indicators), Android Notification Channels (`NotificationChannelGroup`, importance levels, per-mini-app isolation), Apple iOS `UserNotifications` (`UNNotificationCategory`, `UNNotificationAction`, foreground presentation), and super-app notification fatigue controls (frequency capping, quiet hours, anti-phishing branding).
3. **Window Chrome & Custom Navigation Governance**: W3C Web Application Manifest `display_override` fallback chain, WICG Window Controls Overlay (WCO `navigator.windowControlsOverlay`, CSS `env(titlebar-area-*)`, `app-region: drag`), WeChat Mini Program capsule button architecture (`wx.getMenuButtonBoundingClientRect`, `navigationStyle: custom`), unconditional native exit guarantees, and cross-platform safe area integration (`env(safe-area-inset-*)`).

---

## Detailed Findings Matrix

| Finding ID | Domain / Standard | Title | Normative Reference | Evidence Level | Super-App Architectural Control |
|---|---|---|---|---|---|
| `push_vapid_028_01` | Push Protocol Gateway | RFC 8030 Web Push Protocol Gateway & Headers | RFC 8030 §5, §5.2 TTL, §5.3 Urgency, §5.4 Topic | Normative Standard | Host push gateway mediates push delivery; enforces TTL clamps, maps Urgency headers, and replaces stale pending messages via Topic header. |
| `push_vapid_028_02` | Push Payload Encryption | RFC 8291 `aes128gcm` End-to-End Encryption | RFC 8291 §2–4, NIST P-256, RFC 8188 | Normative Standard | Generates ECDH P-256 key pair and 16-octet auth secret per mini-app; mandates single-record `aes128gcm` ciphertext <= 3993 bytes; push relays cannot inspect payload. |
| `push_vapid_028_03` | Push Publisher Auth | RFC 8292 VAPID Application Server Identification | RFC 8292 §2, §2.1, §2.2 | Normative Standard | Binds mini-app AppID to developer VAPID P-256 public key; validates ES256 JWT tokens and contact `sub` claims; enforces exp <= 12h and per-key rate quotas. |
| `push_vapid_028_04` | Push Subscription Lifecycle | W3C Push API PushManager Lifecycle | W3C Push API §6–8 | Normative Standard | Mandates `userVisibleOnly=true` to prevent silent tracking; generates tenant-scoped subscription endpoints; handles `pushsubscriptionchange` on token rotation. |
| `push_vapid_028_05` | Push Mediation & Policy | Super-App Push Gateway Mediation | MDN / Industry Super-App Practice | Platform Practice | Centralized push gateway maps mini-app pushes into native OS pushes (APNs/FCM); enforces daily message caps, quiet hours (22:00–07:00), and independent user opt-out. |
| `host_notif_028_01` | Notifications API | WHATWG Notifications API Standard | WHATWG Notifications API §2–3 | Normative Standard | Scopes notification tags as `${mini_app_id}:${tag}`; injects verified mini-app branding; strips unauthorized action types; routes action clicks back to container context. |
| `host_notif_028_02` | App Badging | W3C Badging API Application Indicators | W3C Badging API §4, §6 | Normative Standard | Implements two-level badging: virtual badge on in-app catalog/dock tiles, aggregated badge on host OS app icon; clamps counts to 99; clears on app activation. |
| `host_notif_028_03` | OS Channel Isolation | Android Notification Channels & Groups | Android Developer Guides (API 26+) | Normative Standard | Dynamically registers dedicated `NotificationChannel` per mini-app under a 'Mini Apps' `NotificationChannelGroup`; prevents whole-app muting if one mini-app spams. |
| `host_notif_028_04` | iOS Presentation | Apple UserNotifications UNNotificationCategory | Apple Developer Documentation | Normative Standard | Registers standardized `UNNotificationCategory` types; converts foreground notifications into in-app snackbars via `willPresent`; routes clicks via `didReceive`. |
| `host_notif_028_05` | Notification Anti-Abuse | Super-App Notification Fatigue & Phishing Defense | Super-App Architecture Practice | Platform Practice | Enforces template-based messaging (WeChat/Alipay model), anti-phishing `[Mini App: Name]` banners, independent opt-out settings, and strict promotional volume limits. |
| `display_chrome_028_01` | Manifest Window Mode | W3C Manifest `display_override` Fallback Chain | W3C Manifest §1.8, WICG Manifest Incubations | Normative Standard | Parses ordered list of display modes (`window-controls-overlay`, `standalone`, `minimal-ui`); gracefully falls back across Android, iOS, tablet, and desktop runtimes. |
| `display_chrome_028_02` | Desktop Titlebar | WICG Window Controls Overlay Geometry & CSS env() | WICG Window Controls Overlay §3–5 | Normative Standard | Exposes `navigator.windowControlsOverlay` and `ongeometrychange`; injects CSS `env(titlebar-area-*)` coordinates; handles `app-region: drag/no-drag` in window manager. |
| `display_chrome_028_03` | Mobile Capsule Button | WeChat Mini Program Capsule Button Architecture | WeChat Mini Program API (`wx.getMenuButtonBoundingClientRect`) | Vendor Specification | Overlays persistent native capsule button (Menu/Close); exposes synchronous bounding rect coordinates; prevents guest UI overlap with host control security zones. |
| `display_chrome_028_04` | Navigation Security | Custom Navigation Security & Exit Guarantees | Super-App Security Architecture | Platform Practice | Native close button executes outside WebView JS runtime guaranteeing instant exit from frozen apps; enforces immutable security origin cues and top-level dialog rendering. |
| `display_chrome_028_05` | Safe Area Insets | Cross-Platform Safe Area & Notch Inset Integration | CSS Values Level 4 / Apple UIKit / Android Insets | Normative Standard | Unifies CSS `env(safe-area-inset-*)` with titlebar geometry; handles dynamic islands and display cutouts; provides emulator presets and automated viewport layout testing. |

---

## Architectural & Governance Deep-Dive

### 1. Unified Push Gateway & Message Encryption Architecture

Third-party mini-apps running inside a super-app cannot establish direct, unmediated communication channels with platform operating system push services (such as Apple APNs, Google FCM, Huawei Push Kit, or Xiaomi Cloud Push). Exposing raw device device push tokens to external mini-app servers introduces catastrophic security and privacy hazards:
- **Device Fingerprinting & Cross-App Tracking**: Correlation of persistent hardware push tokens across disparate mini-apps deanonymizes users.
- **Silent Background Wake-ups & Battery Depletion**: Uncontrolled push wakes allow rogue mini-apps to execute covert background analytics, drain battery, and consume mobile data.
- **Reputational Contagion**: Spamming or deceptive push campaigns by a single mini-app destroy the host super-app's OS-level notification standing, leading users to silence the super-app entirely.

To resolve these challenges, the standard establishes an **Asymmetric Push Mediation Gateway**:
```
+-----------------------------------------------------------------------------------+
| Mini-App Backend Server                                                           |
| 1. Generates ES256 VAPID JWT (RFC 8292) using MiniApp Private Key                 |
| 2. Encrypts payload with aes128gcm (RFC 8291) using Client ECDH Public Key & Salt  |
| 3. Sends HTTP POST to Virtual Gateway Endpoint with TTL & Urgency (RFC 8030)      |
+-----------------------------------------------------------------------------------+
                                         │ HTTP POST (TLS)
                                         ▼
+-----------------------------------------------------------------------------------+
| Super-App Central Push Gateway (Cloud Infrastructure)                             |
| 1. Authenticates VAPID JWT signature against Developer Store Public Key           |
| 2. Enforces Publisher Quotas, Frequency Caps, and User Opt-Out Matrix             |
| 3. Evaluates Quiet Hours (22:00-07:00) based on recipient timezone                |
| 4. Deduplicates or supersedes pending messages using the 'Topic' header           |
| 5. Wraps opaque ciphertext into Host Native Push Payload (APNs / FCM)             |
+-----------------------------------------------------------------------------------+
                                         │ Native OS Push Delivery
                                         ▼
+-----------------------------------------------------------------------------------+
| Client Device Operating System (iOS / Android / Desktop)                         |
| Delivers payload to Super-App Host Process                                        |
+-----------------------------------------------------------------------------------+
                                         │ Internal Container Routing
                                         ▼
+-----------------------------------------------------------------------------------+
| Super-App Native Container Runtime                                                |
| 1. Intercepts incoming OS push payload and identifies target Mini-App ID          |
| 2. Verifies mini-app notification permission is 'granted' in host settings        |
| 3. Dispatches payload into sandboxed ServiceWorker / MiniApp Background Worker     |
| 4. Decrypts aes128gcm payload using isolated local ECDH private key               |
| 5. Enforces userVisibleOnly rule: requires showNotification() within 10 seconds   |
+-----------------------------------------------------------------------------------+
```

### 2. Host Notification Mediation, Channels & OS Badging

When a mini-app requests to display a notification via the WHATWG `Notifications API` (`Notification` or `ServiceWorkerRegistration.showNotification()`), the super-app runtime intercepts the call and applies multi-tenant governance controls:
1. **Dynamic Android Notification Channels**:
   - The host automatically registers a dedicated `NotificationChannel` per mini-app under an umbrella `NotificationChannelGroup` (e.g. `Group: Partner Mini Apps`, `Channel: Food Delivery Updates`).
   - If an individual mini-app produces unwanted notifications, the end user can silence or demote that specific channel in native Android system settings without muting other mini-apps or the host super-app.
   - The container cleans up channels for uninstalled or long-dormant mini-apps, avoiding OS channel bloat.
2. **iOS UserNotifications Category & Presentation Binding**:
   - The host registers standard `UNNotificationCategory` instances with predefined interaction buttons (View, Dismiss, Reply).
   - In `UNUserNotificationCenterDelegate.willPresent`, the container detects if the user is currently interacting with the mini-app: if active in foreground, the system banner is suppressed in favor of an unobtrusive in-app toast or banner.
   - User notification clicks invoke `didReceive`, which parses the custom payload, launches the target mini-app, and dispatches a synthetic `notificationclick` event.
3. **Two-Tier Application Badging**:
   - Mini-app calls to `navigator.setAppBadge(count)` update an in-memory badge ledger.
   - Tier 1: Virtual badge counters are rendered directly on the mini-app's icon in the super-app store listing, recent tasks shelf, and home screen shortcuts.
   - Tier 2: The super-app calculates the aggregate badge sum across all active mini-apps and synchronizes the count to the super-app's native OS dock/launcher icon via the platform badging API.
   - Large counts are clamped to `99+` to maintain visual clarity and reduce cognitive strain.

### 3. Window Controls Overlay & Navigation Chrome Security

Immersive, full-screen, and desktop-extended layouts are vital for complex mini-apps (dashboards, productivity tools, casual games). However, completely relinquishing window chrome to third-party code creates severe vulnerabilities:
- **Kiosk Locking / Trap Loops**: Malicious mini-apps intercepting back gestures and hiding window controls trap the user in an unclosable view.
- **Phishing & UI Spoofing**: Custom titlebars can be painted to mimic official banking security notices, system login sheets, or super-app authority badges.
- **Control Collisions**: Inattentive developers place interactive buttons directly beneath physical notches, status bar icons, or host menu buttons.

The standard resolves these risks by synthesizing **WICG Window Controls Overlay** with the **WeChat Capsule Button** pattern:
1. **Immutable Native Capsule Button**:
   - On mobile devices, a persistent native widget containing 'Menu/More' and 'Close/Exit' is permanently overlaid in the upper right quadrant of the viewport.
   - The exit action binds directly to native host activity lifecycle methods, completely bypassing the WebView JavaScript thread. Even if the JavaScript runtime freezes in an infinite loop, the user can terminate the mini-app instantly.
   - The host exposes `wx.getMenuButtonBoundingClientRect()` (or container equivalent) returning `{ top, bottom, left, right, width, height }`. Mini-apps use these coordinates to position custom headers without obscuring the host capsule.
2. **WICG Window Controls Overlay for Desktop & Foldables**:
   - Mini-apps declare `display_override: ["window-controls-overlay", "standalone", "minimal-ui"]`.
   - On desktop environments, `navigator.windowControlsOverlay.visible` returns `true`, and the host provides CSS variables `env(titlebar-area-x)`, `env(titlebar-area-y)`, `env(titlebar-area-width)`, and `env(titlebar-area-height)`.
   - Window drag behaviors are cleanly partitioned via CSS `app-region: drag` and `app-region: no-drag`, ensuring titlebar buttons remain clickable while empty titlebar space allows window repositioning.
3. **Safe Area & Notch Insets Integration**:
   - The host container enforces `viewport-fit=cover` and injects standard CSS `env(safe-area-inset-top)`, `env(safe-area-inset-right)`, `env(safe-area-inset-bottom)`, and `env(safe-area-inset-left)`.
   - Layout linters in the super-app developer IDE verify that interactive touch targets maintain a minimum 8px margin from capsule boundaries and safe area cutouts.

---

## Verification & Citation Status

All 15 findings added in Iteration 28 have been validated:
- Schema conformance: 100% of records contain required keys (`id`, `topic`, `title`, `evidence_level`, `url`, `supporting_urls`, `standard_reference`, `detail`, `superapp_applicability`).
- Zero credential exposure: Automated secret scanning confirmed no API keys, private keys, or tokens.
- Citation verification: All 20 unique primary and supporting URLs rechecked with HTTP 200 OK.
- Append-only integrity: Verified line count increase from 356 to 371 records in `state/findings.jsonl`.
