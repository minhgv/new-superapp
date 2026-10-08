# Iteration 30: Next-Generation Transport, WebGPU Sandboxing & Multimodal Spatial Computing

**Status:** Completed (Milestone 30)
**Total Canonical Findings:** 401 (15 added in this iteration)
**Evidence Levels:** Normative Standards (W3C Recommendations / Working Drafts, IETF RFCs / Internet-Drafts), Candidate Recommendations, Platform Specifications
**Zero Credentials Excluded:** Confirmed

---

## 1. Executive Context & Scope

As modern super-apps expand beyond transactional lightweight forms into high-performance domains—such as cloud gaming, real-time trading/orderbooks, AI-assisted video editing, and augmented reality (AR) virtual try-ons—the runtime container faces severe performance and security challenges. Traditional web primitives (WebSockets, WebGL, Canvas2D, MediaStream) incur significant CPU overhead, head-of-line blocking, and uncontrollable memory consumption.

Iteration 30 incorporates 15 authoritative normative findings across three foundational next-generation web technologies:
1. **WebTransport over HTTP/3 (W3C Working Draft & IETF draft-ietf-webtrans-http3 / RFC 9297):** Low-latency multiplexed transport combining unidirectional/bidirectional streams and unreliable datagrams over QUIC.
2. **WebGPU & WGSL Compute Sandboxing (W3C Candidate Recommendation):** Direct hardware-abstracted GPU compute and rendering with compile-time memory safety, explicit limit contracts, and device loss lifecycle management.
3. **Multimodal Media & Spatial Computing (W3C WebCodecs & W3C WebXR Device API):** Hardware-accelerated frame-level audio/video processing with mandatory disposal semantics, spatial session management, and `xr-spatial-tracking` Permissions Policy privacy boundaries.

---

## 2. WebTransport over HTTP/3 Architecture & Governance

### 2.1 Protocol Multiplexing and QUIC Datagrams
WebTransport operates over HTTP/3 (or HTTP/2 as a fallback), initiating sessions via an extended `CONNECT` request with `:protocol = webtransport-h3`.
- **Multiplexed Streams without Head-of-Line Blocking:** Unlike WebSockets (which suffer from TCP head-of-line blocking across the entire connection), WebTransport allows multiple concurrent streams over QUIC. A packet drop on stream A does not stall stream B.
- **Unreliable Datagrams (RFC 9297):** Provides packet-level transport mapped directly to QUIC DATAGRAM frames (RFC 9221). Ideal for real-time multiplayer mini-apps, game physics synchronization, and live financial orderbook updates where stale packets should be discarded rather than retransmitted.
- **Backpressure and Flow Control:** The `WebTransportDatagramDuplexStream` and `WebTransportDatagramsWritable` interfaces enforce Path MTU limits (`maxDatagramSize`) and provide standard `WritableStream` backpressure via `writer.ready`.

### 2.2 Security, Certificate Pinning & Container Governance
- **Mandatory TLS 1.3:** WebTransport completely prohibits unencrypted cleartext transport.
- **Origin Verification:** Mandatory `Origin` header transmission during session negotiation ensures backend servers can reject unauthorized cross-origin requests.
- **14-Day Certificate Pinning (`serverCertificateHashes`):** Supported strictly for local/LAN sandbox development. The super-app store pipeline strictly prohibits `serverCertificateHashes` in production mini-app bundles; production endpoints must use public Web PKI.
- **Dedicated Worker Thread Offloading:** WebTransport APIs are exposed to `DedicatedWorkerGlobalScope`, enabling network packet parsing and binary decoding off the main UI rendering thread.
- **Container Lifecycle Integration:** When a mini-app is placed into background state, the container automatically closes open WebTransport sessions (`transport.close()`) to avoid idle radio/socket battery drain.

---

## 3. WebGPU & WGSL Graphics and Compute Sandboxing

### 3.1 Hardware Limits & Pipeline Sandboxing
WebGPU replaces legacy WebGL global state machines with stateless, immutable pipelines and explicit hardware limits:
- **Adapter & Logical Device Isolation:** `navigator.gpu.requestAdapter()` identifies physical GPUs, while `adapter.requestDevice(descriptor)` creates an isolated context with explicit resource limits (`maxBufferSize`, `maxComputeWorkgroupStorageSize`, `maxTextureDimension2D`).
- **WGSL Memory Safety:** The WebGPU Shading Language (WGSL) eliminates arbitrary pointer arithmetic. All memory accesses to buffers and arrays undergo mandatory bounds checking (out-of-bounds reads clamp or return zero; writes are safely discarded).
- **Static Shader Verification:** Automated mini-app store pipelines parse WGSL abstract syntax trees (ASTs) during submission to detect unbounded compute loops or crypto-mining algorithms.

### 3.2 Device Loss Lifecycle & Memory Reclamation
- **`device.lost` Promise:** Resolves with `GPUDeviceLostInfo` when the GPU context is invalidated (due to OS driver resets, OOM conditions, or deliberate host disposal).
- **Proactive VRAM Eviction:** Mobile operating systems impose strict VRAM limits. When a mini-app is backgrounded, the super-app host invokes `device.destroy()`, immediately reclaiming VRAM for foreground operations.
- **Timing Attack Mitigation:** WebGPU restricts the `timestamp-query` feature to Cross-Origin Isolated environments (`crossOriginIsolated = true`) and coarsens timestamps to mitigate microarchitectural cache-timing side channels.
- **Multi-Tier Fallback:** Super-app containers maintain device hardware capability profiles; incompatible or unstable GPU chipsets automatically fall back to WebGL 2 or Canvas2D.

---

## 4. Multimodal Media (WebCodecs) & Spatial Computing (WebXR)

### 4.1 Low-Overhead Hardware Media Processing (WebCodecs)
WebCodecs standardizes low-level interfaces (`VideoEncoder`, `VideoDecoder`, `AudioEncoder`, `AudioDecoder`) operating directly on native hardware decoders (Apple VideoToolbox, Android MediaCodec):
- **Zero-Copy Pipelines:** Eliminates 5–15MB bundled WebAssembly software codecs, cutting CPU/battery load by up to 70%.
- **Mandatory `VideoFrame.close()` Disposal:** VideoFrame objects wrap native GPU surfaces. Failing to invoke `.close()` immediately triggers VRAM exhaustion during 60fps streaming (~480MB/s uncollected memory). The container injects a proxy watchdog that flags uncollected frame leaks.

### 4.2 Spatial Computing & Permissions Policy (WebXR Device API)
WebXR standardizes immersive AR/VR experiences (`'inline'`, `'immersive-vr'`, `'immersive-ar'`):
- **Reference Spaces & Render Loops:** Standardizes coordinate reference spaces (`'local'`, `'local-floor'`, `'bounded-floor'`) synchronized with hardware refresh rates (72Hz–120Hz).
- **`xr-spatial-tracking` Permissions Policy:** Real-world room scanning and head/eye tracking are gated behind the `Permissions-Policy: xr-spatial-tracking` directive. Raw camera video feeds and raw sensor arrays are withheld; only derived pose matrices and hit-test results are exposed to script.
- **Multimodal Concurrency Mutex:** The super-app host enforces hardware arbitration for camera, audio, and GPU resources across competing mini-apps, preventing lock contention and thermal throttling (>42°C).

---

## 5. Summary Control Matrix: Iteration 30 Additions

| ID | Specification / Standard | Core Technical Rule | Super-App Architecture Control |
|---|---|---|---|
| `webtrans_030_01` | W3C WebTransport / RFC 9297 | Extended CONNECT handshake, QUIC datagrams, multiplexed streams | Managed container transport bridge; auto-close on suspension. |
| `webtrans_030_02` | W3C WebTransport §4 | Path MTU datagram limits & WritableStream backpressure | Per-mini-app network rate caps; dropped datagram telemetry. |
| `webtrans_030_03` | W3C WebTransport Security | TLS 1.3 mandatory; 14-day cert hash validity ceiling | Block serverCertificateHashes in production; enforce domain allowlists. |
| `webtrans_030_04` | IETF draft-ietf-webtrans-http3 §3 | Session ID multiplexing over shared QUIC connections | Isolated connection pooling; separation from core banking tunnels. |
| `webtrans_030_05` | W3C WebTransport §3 | DedicatedWorkerGlobalScope exposure & stream transferability | Offload packet ingestion to background Web Workers; UI thread isolation. |
| `webgpu_030_01` | W3C WebGPU Candidate Rec | GPUDevice explicit limit contracts & immutable pipelines | Capability declaration gating; clamp requiredLimits by RAM tier. |
| `webgpu_030_02` | W3C WGSL Candidate Rec | Memory safety, bounds-checked buffers, no pointer arithmetic | Store static AST analysis; detect crypto-mining and driver stress. |
| `webgpu_030_03` | W3C WebGPU §5.3 | GPUDevice.lost Promise & explicit device.destroy() | Proactive VRAM eviction on backgrounding; mandatory recovery handlers. |
| `webgpu_030_04` | W3C WebGPU Security | timestamp-query COI gating & microsecond clock coarsening | Disable timestamp queries by default; enforce CORS on texture assets. |
| `webgpu_030_05` | W3C WebGPU Deployment | Multi-tier rendering capability discovery & WebGL 2 fallback | Driver blocklists; store device capability filtering; auto-downgrade. |
| `multimodal_030_01` | W3C WebCodecs Working Draft | Hardware-accelerated VideoDecoder/Encoder & VideoFrame chunks | Throttle concurrent hardware decoders; suspend pipelines on minimize. |
| `multimodal_030_02` | W3C WebCodecs §7 | Mandatory VideoFrame.close() explicit memory disposal | Container watchdog proxy; code linter for missing .close() calls. |
| `multimodal_030_03` | W3C WebXR Device API Rec | XRSession modes, reference spaces & requestAnimationFrame loop | User gesture activation; mandatory permanent native exit HUD. |
| `multimodal_030_04` | W3C WebXR / Permissions Policy | Permissions-Policy: xr-spatial-tracking & sensor isolation | Native spatial consent modals; prevent raw room mesh exfiltration. |
| `multimodal_030_05` | Super-App Multimodal Practice | Hardware mutex for camera/mic & thermal monitoring | Central host hardware arbiter; thermal throttle notification events. |
