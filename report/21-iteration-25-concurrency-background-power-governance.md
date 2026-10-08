# Iteration 25: Concurrency Synchronization, Background Execution & Device Power Governance

**Evidence records added:** 15
**Topic coverage:** W3C Web Locks API, WHATWG HTML Web Messaging (MessageChannel/MessagePort), WICG Background Sync, WICG Periodic Background Sync, WICG Background Fetch, W3C Media Session API, W3C Screen Wake Lock API, and W3C Battery Status / WeChat Power Governance.
**Verification status:** 100% verified; 11 unique primary/official URLs tested HTTP 200; zero credential exposure.

---

## 1. Domain Overview & Normative Anchors

This iteration addresses critical runtime concurrency, long-lived/scheduled background synchronization, and host-mediated power and device resource governance for Super App MiniApp runtimes:

1. **W3C Web Locks API (`https://www.w3.org/TR/web-locks/`)**:
   - Asynchronous lock coordination across multiple execution contexts (Window, Service Worker, Shared Worker, MiniApp logic/render threads) belonging to the same storage origin.
   - Normative lock modes: `exclusive` (default, mutual exclusion) and `shared` (multiple concurrent readers).
   - Resource scheduling algorithm: non-blocking probing (`ifAvailable: true`), deterministic queue eviction/preemption (`steal: true`), preemption cancellation via `AbortSignal`, and introspective diagnostics via `navigator.locks.query()`.

2. **WHATWG HTML Web Messaging (`https://html.spec.whatwg.org/multipage/web-messaging.html`)**:
   - Cross-context communication via `window.postMessage()` and decoupled communication channels via `MessageChannel` / `MessagePort`.
   - Explicit target origin specification (`targetOrigin != '*'`) to prevent information leakage to unauthorized frames or malicious origins.
   - Transferable object semantics: transferring ownership of `MessagePort`, `ArrayBuffer`, or `OffscreenCanvas` across contexts with zero-copy and complete neutralization of the sending side.
   - Dual-thread message boundaries: isolating MiniApp service/logic workers from UI WebView render contexts through supervised message ports with structured error handling.

3. **WICG Background Sync & Periodic Sync (`https://wicg.github.io/background-sync/spec/`, `https://wicg.github.io/periodic-background-sync/`)**:
   - One-shot Background Sync (`SyncManager`): registering deferred work (`sync` event) triggered immediately upon device network reconnection, guaranteed by the host platform.
   - Exponential backoff retry policies and `lastChance` boolean flag indicating final retry attempt before failure discard.
   - Periodic Background Sync (`PeriodicSyncManager`): scheduled background wakeups with developer-suggested `minInterval` gated by host-defined minimum intervals, user engagement heuristics, battery charge levels, and network metered/unmetered states.

4. **WICG Background Fetch (`https://wicg.github.io/background-fetch/`)**:
   - Resilient background file downloads and batch uploads that persist beyond the lifecycle of the active MiniApp view or Service Worker.
   - Native OS progress notification integration and paused/abortable controls.
   - Cache API destination storage and upload payload re-transmission constraints.

5. **W3C Media Session & Screen Wake Lock (`https://www.w3.org/TR/mediasession/`, `https://www.w3.org/TR/screen-wake-lock/`, WeChat Mini Program Media APIs)**:
   - W3C Media Session API: exposing rich media metadata (title, artist, album, artwork) to OS-level system controls (lock screen, notification tray, wearable remotes) and binding action handlers (`play`, `pause`, `seekto`, `previoustrack`, `nexttrack`).
   - WeChat `wx.getBackgroundAudioManager`: background audio entitlement requires host manifest declaration (`requiredBackgroundModes: ["audio"]`), automatic foreground/background audio continuity, and host-managed audio focus arbitration.
   - W3C Screen Wake Lock API (`navigator.wakeLock.request('screen')`): acquiring sentinel locks to prevent system display sleep during critical interactive operations (e.g., ticket QR display, active checkout scan, video playback).
   - Automatic wake lock release upon page visibility change (`visibilitychange` -> `hidden`) to prevent unconstrained battery drain.
   - WeChat `wx.setKeepScreenOn`: explicit host bridge counterpart for screen wakefulness.

6. **Device Battery & Power Governance (`https://www.w3.org/TR/battery-status/`, WeChat `wx.getBatteryInfo`)**:
   - Privacy and fingerprinting hazards of high-precision battery status reporting (`BatteryManager`: charging, chargingTime, dischargingTime, level).
   - W3C security mitigations: coarse quantization of battery percentage and rounded time thresholds to obstruct cross-origin user tracking and device fingerprinting.
   - WeChat `wx.getBatteryInfo`: host-mediated battery telemetry providing integer percentage and charging state without microsecond timing vectors.
   - Super App runtime power governor: adaptive execution throttling, background sync suspension, and high-drain API disabling when device enters low-power or battery-saver mode.

---

## 2. Evidence Records (Iteration 25)

### Record 1: W3C Web Locks API Scope & Storage Partitioning
- **ID:** `concurrency-web-locks-scope-001`
- **Topic:** `concurrency_control_locks`
- **Title:** Web Locks API mediates origin-scoped resource coordination across worker and window contexts
- **Standard Reference:** W3C Web Locks API §3.1–3.2, §4.1–4.2
- **Evidence Level:** `normative_standard`
- **Source URL:** https://www.w3.org/TR/web-locks/
- **[SPEC FACT]:** The Web Locks API provides `navigator.locks.request(name, [options], callback)` allowing script to asynchronously acquire a lock over an origin-scoped resource name. While held, no other script in the same storage origin can acquire an incompatible lock. Locks are released when the callback promise settles or when context terminates.
- **[SUPER-APP CONTROL]:** MiniApp runtime must partition lock namespaces per `miniAppId`. Host container uses exclusive locks to serialize local SQLite/OPFS schema migrations and key rotation, preventing race conditions between background service workers and active render views.

### Record 2: W3C Web Locks Arbitration Modes (`exclusive` vs `shared`)
- **ID:** `concurrency-web-locks-modes-002`
- **Topic:** `concurrency_control_locks`
- **Title:** Web Locks exclusive and shared modes establish reader-writer synchronization for MiniApp storage
- **Standard Reference:** W3C Web Locks API §3.2.1, §4.1
- **Evidence Level:** `normative_standard`
- **Source URL:** https://www.w3.org/TR/web-locks/
- **[SPEC FACT]:** Locks support `options.mode`: `"exclusive"` (default) and `"shared"`. Multiple execution contexts can hold `"shared"` locks concurrently, but only one context can hold an `"exclusive"` lock. Queued lock requests are serviced in FIFO order by default.
- **[SUPER-APP CONTROL]:** Enforce reader-writer locks across MiniApp data caches. Cached catalog lookups, user profile reads, and session checks acquire shared locks; transactional writes, token refreshes, and wallet balance adjustments acquire exclusive locks.

### Record 3: Web Locks Preemption, Timeouts & Diagnostic Query
- **ID:** `concurrency-web-locks-preemption-003`
- **Topic:** `concurrency_control_locks`
- **Title:** `ifAvailable`, `steal`, `AbortSignal`, and `query()` enable dead-lock prevention and recovery
- **Standard Reference:** W3C Web Locks API §3.2.1–3.2.2, §4.1
- **Evidence Level:** `normative_standard`
- **Source URL:** https://www.w3.org/TR/web-locks/
- **[SPEC FACT]:** `options.ifAvailable: true` returns `null` immediately if the lock cannot be granted without waiting. `options.steal: true` forcibly releases existing locks and grants the request (failing victim locks with `AbortError`). `options.signal` aborts pending requests. `navigator.locks.query()` returns a snapshot of held and pending locks.
- **[SUPER-APP CONTROL]:** Super App host monitors MiniApp lock queues. If an exclusive lock is held beyond the execution timeout (e.g., 5,000ms), the watchdog issues an abort via `AbortSignal`. Critical host maintenance tasks can invoke `steal: true` to prevent uncooperative MiniApps from blocking container state teardown.

### Record 4: WHATWG Web Messaging Origin Binding & Validation
- **ID:** `concurrency-web-messaging-origin-004`
- **Topic:** `cross_context_messaging`
- **Title:** `postMessage` origin binding prevents cross-context message interception and spoofing
- **Standard Reference:** WHATWG HTML Web Messaging §9.1–9.2
- **Evidence Level:** `normative_standard`
- **Source URL:** https://html.spec.whatwg.org/multipage/web-messaging.html
- **[SPEC FACT]:** `window.postMessage(message, targetOrigin, [transfer])` requires an explicit target origin URI reference. If `targetOrigin` is `'*'`, any origin can intercept the message. Receiving contexts must verify `event.origin` before trusting event payload or ports.
- **[SUPER-APP CONTROL]:** Disallow wildcard `targetOrigin: '*'` across all MiniApp bridge communications. MiniApp SDK runtime must validate `event.origin === hostSuperAppOrigin` for inbound events, and container bridge must verify MiniApp sender origin before dispatching privileged device or session APIs.

### Record 5: WHATWG MessageChannel & Transferable Objects
- **ID:** `concurrency-messagechannel-transfer-005`
- **Topic:** `cross_context_messaging`
- **Title:** `MessageChannel` and `MessagePort` transferability enforce zero-copy capability-scoped communication
- **Standard Reference:** WHATWG HTML Web Messaging §9.3–9.4
- **Evidence Level:** `normative_standard`
- **Source URL:** https://html.spec.whatwg.org/multipage/web-messaging.html
- **[SPEC FACT]:** `MessageChannel` creates two entangling ports (`port1`, `port2`). Ports are transferable objects: transferring a port detaches it from the sending execution context, granting exclusive communication rights to the receiver context without shared-memory race hazards.
- **[SUPER-APP CONTROL]:** MiniApp container architecture leverages dedicated `MessageChannel` instances between the native host supervisor and individual WebViews. Dedicated ports isolate high-frequency UI rendering events from sensitive backend RPC channels, preventing DOM script injection from observing inter-thread RPC packets.

### Record 6: WICG Background Sync Lifecycle & Tag Deduplication
- **ID:** `background-sync-registration-001`
- **Topic:** `background_synchronization`
- **Title:** Background Sync guarantees network-reconnection execution and tag-scoped deduplication
- **Standard Reference:** WICG Background Sync §5.1–5.3, §6.1
- **Evidence Level:** `normative_standard`
- **Source URL:** https://wicg.github.io/background-sync/spec/
- **[SPEC FACT]:** `syncManager.register(tag)` registers a background sync request. If the device is online, the `sync` event fires promptly in the Service Worker; if offline, execution is deferred until network connectivity is restored. Re-registering an identical `tag` while pending coalesces into the existing registration without duplicating work.
- **[SUPER-APP CONTROL]:** Super App offline transaction queue maps user checkout, voucher claims, and analytics pings to unique idempotent sync tags (e.g., `checkout-tx-94812`). The host ensures sync events are executed upon network recovery even if the user has navigated away from the MiniApp.

### Record 7: Background Sync Retry Policies & `lastChance` Boundary
- **ID:** `background-sync-retry-lastchance-002`
- **Topic:** `background_synchronization`
- **Title:** `lastChance` flag and backoff algorithms govern background sync failure and queue retirement
- **Standard Reference:** WICG Background Sync §5.3–5.4
- **Evidence Level:** `normative_standard`
- **Source URL:** https://wicg.github.io/background-sync/spec/
- **[SPEC FACT]:** When a `sync` event handler rejects, the user agent reschedules the sync with exponential backoff. The `event.lastChance` boolean property indicates whether the current dispatch is the final retry attempt before the user agent drops the sync registration.
- **[SUPER-APP CONTROL]:** MiniApp service workers must inspect `event.lastChance`. On final attempt (`lastChance == true`), if synchronization fails, the worker must write an audit record to persistent local storage and notify the user via local notification rather than silently dropping pending payments or data.

### Record 8: WICG Periodic Background Sync Gating & Minimum Interval
- **ID:** `background-periodic-sync-governance-003`
- **Topic:** `periodic_background_synchronization`
- **Title:** Periodic Sync requires host permission, minimum interval negotiation, and engagement scores
- **Standard Reference:** WICG Periodic Background Sync §4.1–4.3, §5.1
- **Evidence Level:** `normative_standard`
- **Source URL:** https://wicg.github.io/periodic-background-sync/
- **[SPEC FACT]:** `periodicSync.register(tag, { minInterval })` requests scheduled wakeups. The user agent is not obligated to wake the worker at `minInterval`; execution frequency is bounded by user engagement scores, background execution permissions, device power state, and unmetered network availability.
- **[SUPER-APP CONTROL]:** Super App platform restricts Periodic Sync capability to authorized MiniApp categories (e.g., financial news, travel flight updates). The host enforces a global minimum interval floor (>= 4 hours for standard tier, >= 1 hour for premium enterprise) to prevent background battery drain.

### Record 9: Periodic Sync Resource & Network Metering Restrictions
- **ID:** `background-periodic-sync-budget-004`
- **Topic:** `periodic_background_synchronization`
- **Title:** Power status, Data Saver, and system quotas constrain periodic background execution
- **Standard Reference:** WICG Periodic Background Sync §5.2–5.3
- **Evidence Level:** `normative_standard`
- **Source URL:** https://wicg.github.io/periodic-background-sync/
- **[SPEC FACT]:** User agents must withhold `periodicsync` events when the device is in power-saving mode, battery is low, or when Android/iOS Data Saver mode is active on metered cellular networks. Registrations are automatically revoked when site permissions are withdrawn.
- **[SUPER-APP CONTROL]:** Host container integrates periodic sync dispatch with OS battery and network listeners. When OS signals `PowerSaveMode` or roaming data connection, all periodic MiniApp background wakeups are paused until power or Wi-Fi connectivity is restored.

### Record 10: WICG Background Fetch for Large Resilient Transfers
- **ID:** `background-fetch-large-transfers-005`
- **Topic:** `background_fetch_operations`
- **Title:** Background Fetch decouples large asset downloads and uploads from active worker lifecycles
- **Standard Reference:** WICG Background Fetch §4.1–4.4, §5.1–5.3
- **Evidence Level:** `normative_standard`
- **Source URL:** https://wicg.github.io/background-fetch/
- **[SPEC FACT]:** `backgroundFetch.fetch(id, requests, options)` hands multi-part asset transfers over to the browser/system download manager. The operation continues even if the document and service worker terminate. System UI displays persistent download progress, total bytes, and pause/abort controls.
- **[SUPER-APP CONTROL]:** MiniApp package updates exceeding 15MB, offline document bundles, and media catalog downloads must use Background Fetch. The Super App renders unified download progress indicators in the host notification shade, preventing network fragmentation and memory crashes during large asset ingestion.

### Record 11: W3C Media Session API & System Control Binding
- **ID:** `media-session-controls-001`
- **Topic:** `media_session_controls`
- **Title:** Media Session API binds MiniApp playback state and metadata to OS notification and lockscreen controls
- **Standard Reference:** W3C Media Session §4.1–4.3, §5.1–5.2
- **Evidence Level:** `normative_standard`
- **Source URL:** https://www.w3.org/TR/mediasession/
- **[SPEC FACT]:** `navigator.mediaSession.metadata` accepts `MediaMetadata` (title, artist, album, artwork array). `navigator.mediaSession.setActionHandler(action, handler)` registers handlers for `play`, `pause`, `previoustrack`, `nexttrack`, `seekto`, and `stop`. The user agent routes OS audio events directly to the active session.
- **[SUPER-APP CONTROL]:** When a MiniApp plays governed audio/video, the container populates host system notification controllers with verified MiniApp title and publisher attribution, ensuring system-level media controls remain active when the super app is minimized.

### Record 12: WeChat Background Audio Management Alignment
- **ID:** `media-session-wechat-background-audio-002`
- **Topic:** `platform_background_audio`
- **Title:** WeChat BackgroundAudioManager demonstrates host-mediated background playback and audio focus
- **Standard Reference:** WeChat Mini Program Documentation (wx.getBackgroundAudioManager)
- **Evidence Level:** `official_platform_documentation`
- **Source URL:** https://developers.weixin.qq.com/miniprogram/dev/api/media/background-audio/wx.getBackgroundAudioManager.html
- **[SPEC FACT]:** In WeChat Mini Programs, background audio playback requires declaring `"requiredBackgroundModes": ["audio"]` in `app.json`. The singleton `wx.getBackgroundAudioManager()` automatically attaches to system notification controls and manages audio ducking and focus arbitration with other mini programs.
- **[SUPER-APP CONTROL]:** Require MiniApp store review gate for background audio capability. The host runtime enforces a single active background audio session across all running MiniApps: launching playback in MiniApp B immediately pauses and releases audio resources held by MiniApp A.

### Record 13: W3C Screen Wake Lock API & Display Sleep Prevention
- **ID:** `power-screen-wake-lock-003`
- **Topic:** `screen_wake_lock_governance`
- **Title:** Screen Wake Lock sentinel acquisition and automatic release mitigate battery drain
- **Standard Reference:** W3C Screen Wake Lock API §4.1–4.4, §5.1–5.3
- **Evidence Level:** `normative_standard`
- **Source URL:** https://www.w3.org/TR/screen-wake-lock/
- **[SPEC FACT]:** `navigator.wakeLock.request('screen')` returns a `WakeLockSentinel`. Sentinels keep the device screen illuminated. The lock is automatically released whenever the document becomes hidden (`document.visibilityState === 'hidden'`) or when the user navigates away, firing the `release` event.
- **[SUPER-APP CONTROL]:** MiniApps presenting dynamic QR barcodes (transit gates, retail payment POS) or turn-by-turn navigation acquire screen wake locks. The container automatically revokes sentinels when the MiniApp view is minimized or when idle user timeout (120s) expires.

### Record 14: WeChat Screen & Battery Power Host Management
- **ID:** `power-wechat-screen-battery-governance-004`
- **Topic:** `platform_power_governance`
- **Title:** WeChat `wx.setKeepScreenOn` and `wx.getBatteryInfo` exemplify host power governance
- **Standard Reference:** WeChat Mini Program APIs (wx.setKeepScreenOn, wx.getBatteryInfo)
- **Evidence Level:** `official_platform_documentation`
- **Source URL:** https://developers.weixin.qq.com/miniprogram/dev/api/device/screen/wx.setKeepScreenOn.html
- **Supporting URL:** https://developers.weixin.qq.com/miniprogram/dev/api/device/battery/wx.getBatteryInfo.html
- **[SPEC FACT]:** WeChat provides `wx.setKeepScreenOn({ keepScreenOn: boolean })` to maintain screen wakefulness only while the mini program is alive and in foreground. `wx.getBatteryInfo()` returns `level` (percentage 1-100) and `isCharging` via asynchronous or synchronous bridge calls.
- **[SUPER-APP CONTROL]:** Container bridge mediates screen wakefulness and battery status. If `keepScreenOn` is requested, the host applies an inactivity watchdog timer. Battery telemetry is cached and quantized to prevent polling loops from creating CPU wake locks.

### Record 15: Battery Status API Fingerprinting Risks & Super App Mitigations
- **ID:** `power-battery-status-privacy-005`
- **Topic:** `battery_status_privacy`
- **Title:** Battery Status API privacy risks mandate coarse granularity and low-power execution throttling
- **Standard Reference:** W3C Battery Status API §4.1–4.2, §6 (Security and Privacy)
- **Evidence Level:** `normative_standard`
- **Source URL:** https://www.w3.org/TR/battery-status/
- **[SPEC FACT]:** W3C Battery Status API specification highlights that high-frequency, high-precision battery discharge time and level readings can serve as an invasive fingerprinting vector to track users across origins. Mitigations include reducing precision, rounding discharge times, and omitting events.
- **[SUPER-APP CONTROL]:** Disallow raw `navigator.getBattery()` direct access in untrusted MiniApp WebViews. Provide host bridge proxy that rounds battery level to increments of 5% or 10%, omits charging time estimates, and automatically invokes energy-saver throttling (reducing animation frame rates and network poll cadence) when battery drops below 15%.

---

## 3. Architecture & Control Matrix Updates

| Domain | Control ID | Normative Anchor | Super App Enforcement Profile |
|---|---|---|---|
| Concurrency | `SYNC-LOCK-01` | W3C Web Locks | Origin-partitioned `LockManager`; exclusive locks for DB migrations, shared for reads |
| Concurrency | `SYNC-LOCK-02` | W3C Web Locks | Host watchdog aborts pending locks exceeding 5s; `steal: true` reserved for container teardown |
| Messaging | `MSG-ORIGIN-01` | WHATWG Web Messaging | Enforce explicit `targetOrigin` URI; reject wildcard `'*'`; verify `event.origin` on receipt |
| Messaging | `MSG-PORT-02` | WHATWG Web Messaging | Transfer dedicated `MessagePort` channels between supervisor and worker; zero shared memory |
| Background | `BG-SYNC-01` | WICG Background Sync | Idempotent sync tags for transactions; automatic replay on reconnection |
| Background | `BG-SYNC-02` | WICG Background Sync | Inspect `lastChance == true`; persist uncommitted state to audit log upon final failure |
| Background | `BG-PERIOD-01` | WICG Periodic Sync | Require store capability review; enforce >= 4h minimum interval; gate on engagement |
| Background | `BG-FETCH-01` | WICG Background Fetch | Mandatory for asset transfers > 15MB; host-rendered notification progress UI |
| Media | `MEDIA-SESS-01` | W3C Media Session | OS lockscreen/notification metadata binding; forward hardware media keys to active app |
| Media | `MEDIA-FOCUS-02` | WeChat Platform | Enforce single-app background audio focus; auto-pause prior app upon concurrent playback |
| Power | `PWR-WAKE-01` | W3C Screen Wake Lock | Auto-revoke screen wake lock on `visibilitychange === 'hidden'`; 120s inactivity watchdog |
| Power | `PWR-BATT-02` | W3C Battery Status | Quantize battery level to 5–10% bins; throttle animation/poll rates when battery < 15% |
