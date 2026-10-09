# Chuyên đề 61: Service Worker Navigation Preload, WebGL 2.0 Graphics Context Loss, & WebAssembly Component Model (WASI 0.2)

## 1. Tóm tắt điều hành & Bối cảnh kỹ thuật (Iteration 65)

Iteration 65 mở rộng khung chuẩn kỹ thuật cho Super-App Mini-App Store với 3 trụ cột then chốt về tối ưu hóa khởi động mạng song song, phục hồi tài nguyên đồ họa GPU chống sập ứng dụng di động, và cách ly mô-đun nhị phân theo kiến trúc Capability-based Security:

1. **Service Worker Navigation Preload (Tối ưu hóa khởi động song song & loại bỏ độ trễ khởi động Worker):**
   - Truyền thống, khi người dùng mở một mini-app hoặc thực hiện điều hướng mới, trình duyệt/WebView container phải khởi động tiến trình Service Worker trước khi Service Worker có thể phát lệnh fetch mạng, tạo ra độ trễ khởi động từ 50ms đến 250ms trên thiết bị di động (được gọi là worker boot latency bottleneck).
   - Chuẩn `NavigationPreloadManager` giải quyết triệt để rào cản này bằng cách cho phép browser engine bắn request mạng tải HTML shell đồng thời (in parallel) với quá trình khởi động tiến trình worker.
   - Hỗ trợ header `Service-Worker-Navigation-Preload` cho phép gateway trả về payload tối ưu siêu nhỏ, kết hợp với `event.preloadResponse` trong fetch event handler và cơ chế fallback về CacheStorage khi mạng lỗi.

2. **WebGL 2.0 & Phục hồi mất ngữ cảnh đồ họa GPU (Context Loss Resilience & State Recovery):**
   - Đồ họa 3D và mini-games trên Super-App phụ thuộc vào `WebGL2RenderingContext` (OpenGL ES 3.0) hỗ trợ Textures 3D, Uniform Buffer Objects (UBO), Transform Feedback, và Multiple Render Targets (MRT).
   - Trên thiết bị di động, hệ điều hành Android/iOS thường xuyên thu hồi tài nguyên GPU khi thiếu RAM, thiết bị chuyển sang chế độ Sleep hoặc GPU driver bị crash. Trình duyệt sẽ phát sự kiện `webglcontextlost`.
   - Chuẩn kỹ thuật bắt buộc mini-app phải gọi `event.preventDefault()` trong listener `webglcontextlost` để bảo toàn quyền khôi phục ngữ cảnh; đồng thời triển khai pipeline phục hồi trạng thái tự động khi nhận sự kiện `webglcontextrestored` (tái biên dịch shader, khởi tạo lại VBO/texture). Sử dụng extension `WEBGL_lose_context` cho automated testing và `gl.isContextLost()` để bảo vệ render loop.

3. **WebAssembly Component Model & WASI 0.2 (Capability-Based Sandboxing & Zero Ambient Authority):**
   - WebAssembly Component Model mở rộng các module Wasm truyền thống bằng giao diện định kiểu chặt chẽ (WIT - WebAssembly Interface Type IDL) và Canonical ABI, loại bỏ các lỗ hổng tràn bộ đệm tuyến tính (linear memory corruption) khi giao tiếp liên ngôn ngữ.
   - WASI 0.2 (WASI Preview 2) chuẩn hóa các giao diện hệ thống an toàn theo mô hình Capability-based: một guest component có quyền truy cập mặc định bằng zero (Zero Ambient Authority), chỉ có thể truy cập filesystem, clock, hay outbound HTTP khi container chủ liên kết và cấp quyền rõ ràng.
   - Kiểm tra xác thực tĩnh W3C Wasm Core validation giới hạn trần bộ nhớ tuyến tính (linear memory ceiling) từ 64MB đến 128MB, bảo vệ toàn vẹn cho ứng dụng cha đa bên (multi-tenant super app).

---

## 2. Bảng đối chiếu tiêu chuẩn & Bằng chứng xác thực (Citations & Evidence)

| # | Tiêu chuẩn / Giao thức | Lớp kiến trúc | URL nguồn chính thức (HTTP 200) | Evidence Level | Giá trị chuẩn hóa cho Super-App Mini-App Store |
|---|------------------------|---------------|----------------------------------|----------------|------------------------------------------------|
| 1 | NavigationPreloadManager | Mạng & Service Worker | `https://developer.mozilla.org/en-US/docs/Web/API/NavigationPreloadManager` | specification | Loại bỏ độ trễ 50-250ms khi khởi động Service Worker bằng cách fetch tài nguyên mạng song song. |
| 2 | NavigationPreloadManager.enable() | Mạng & Service Worker | `https://developer.mozilla.org/en-US/docs/Web/API/NavigationPreloadManager/enable` | specification | Kích hoạt preload trong pha `activate` của Service Worker; throw InvalidStateError nếu chưa sẵn sàng. |
| 3 | setHeaderValue() & Header | Mạng & HTTP Gateway | `https://developer.mozilla.org/en-US/docs/Web/API/NavigationPreloadManager/setHeaderValue` | specification | Tùy biến header `Service-Worker-Navigation-Preload` giúp edge gateway trả về shell payload siêu nhẹ. |
| 4 | FetchEvent.preloadResponse | Service Worker Cache | `https://developer.mozilla.org/en-US/docs/Web/API/FetchEvent/preloadResponse` | specification | Tiêu thụ response mạng preloaded với fallback tất định về CacheStorage/IndexedDB khi offline. |
| 5 | W3C Service Worker Spec | Chuẩn W3C Normative | `https://w3c.github.io/ServiceWorker/` | specification | Định nghĩa cấu trúc `NavigationPreloadManager { enabled, headerValue }` chuẩn hóa toàn cầu. |
| 6 | WebGL2RenderingContext | Đồ họa GPU di động | `https://developer.mozilla.org/en-US/docs/Web/API/WebGL2RenderingContext` | specification | Chuẩn hóa OpenGL ES 3.0 đồ họa 3D, UBO, MRT, và Transform Feedback trong WebView sandbox. |
| 7 | webglcontextlost Event | Khả năng tự phục hồi | `https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/webglcontextlost_event` | specification | Đón bắt thu hồi GPU driver; bắt buộc `event.preventDefault()` để duy trì quyền phục hồi ngữ cảnh. |
| 8 | webglcontextrestored Event | Tái tạo tài nguyên GPU | `https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/webglcontextrestored_event` | specification | Pipeline tái nạp shader, texture, buffer sau khi GPU khởi tạo lại mà không làm crash mini-app. |
| 9 | WEBGL_lose_context | Kiểm thử tự động Store | `https://developer.mozilla.org/en-US/docs/Web/API/WEBGL_lose_context` | specification | Giả lập mất và phục hồi GPU ngữ cảnh tự động trong CI/CD review gate của App Store. |
| 10| gl.isContextLost() | Render Loop Guard | `https://developer.mozilla.org/en-US/docs/Web/API/WebGLRenderingContext/isContextLost` | specification | Kiểm tra trạng thái ngữ cảnh trước mỗi frame `requestAnimationFrame()` chống tràn bus và ngốn pin. |
| 11| Wasm Component Model | Mô-đun hóa an toàn | `https://component-model.bytecodealliance.org/` | specification | WIT IDL và Canonical ABI ngăn chặn rò rỉ bộ nhớ con trỏ và xung đột linear memory đa ngôn ngữ. |
| 12| Bytecode Alliance Sandboxing | Zero Ambient Authority | `https://bytecodealliance.org/` | standard | Mô hình bảo mật dựa trên Capability: không có quyền ngầm định với I/O, disk, network hay clock. |
| 13| WASI 0.2 (Preview 2) | Chuẩn hệ thống Wasm | `https://github.com/WebAssembly/WASI` | specification | Chuẩn hóa các interface mô-đun: wasi:filesystem, wasi:http, wasi:clocks, wasi:io có kiểm soát. |
| 14| Canonical ABI Memory Safety | Toàn vẹn bộ nhớ nhị phân | `https://github.com/WebAssembly/component-model` | specification | 'canon lift' / 'canon lower' với realloc cô lập; bẫy Wasm Trap tức thì khi có vi phạm bộ nhớ. |
| 15| W3C WebAssembly Core Spec | Thẩm định nhị phân Store | `https://webassembly.github.io/spec/core/` | specification | Thẩm định tĩnh mã nhị phân bytecode, kiểu dữ liệu và áp trần bộ nhớ tuyến tính 64MB-128MB. |

---

## 3. Kiến trúc chi tiết & Hướng dẫn triển khai (Implementation Architecture)

### 3.1. Triển khai Service Worker Navigation Preload trong Mini-App Container

```javascript
// service-worker.js của Mini-App
self.addEventListener('activate', (event) => {
  event.waitUntil(async function() {
    if (self.registration.navigationPreload) {
      // 1. Kích hoạt tải trước điều hướng song song
      await self.registration.navigationPreload.enable();
      // 2. Gắn header nhận diện phiên bản shell siêu tối ưu
      await self.registration.navigationPreload.setHeaderValue('superapp-miniapp-v2-shell');
      console.log('[SW] Navigation Preload enabled successfully.');
    }
  }());
});

self.addEventListener('fetch', (event) => {
  // Chỉ áp dụng cho các request điều hướng trang HTML
  if (event.request.mode === 'navigate') {
    event.respondWith(async function() {
      try {
        // 1. Chờ kết quả tải trước song song từ mạng
        const preloadResponse = await event.preloadResponse;
        if (preloadResponse && preloadResponse.ok) {
          return preloadResponse;
        }
        // 2. Nếu preload chưa sẵn sàng hoặc không hỗ trợ, fetch từ network
        return await fetch(event.request);
      } catch (err) {
        // 3. Fallback tất định về CacheStorage ngoại tuyến khi mất sóng hoặc lỗi gateway
        console.warn('[SW] Navigation preload/fetch failed, falling back to cache:', err);
        const cache = await caches.open('miniapp-offline-v1');
        const cachedFallback = await cache.match('/offline-shell.html');
        return cachedFallback || new Response('Offline', { status: 503, headers: { 'Content-Type': 'text/plain' } });
      }
    }());
  }
});
```

### 3.2. Quản lý mất ngữ cảnh WebGL 2.0 & Tái thiết lập trạng thái đồ họa

```javascript
// canvas-render-engine.js
const canvas = document.getElementById('gl-canvas');
const gl = canvas.getContext('webgl2', { alpha: false, desynchronized: true });

let isContextDead = false;

// Đón bắt sự kiện mất ngữ cảnh GPU
canvas.addEventListener('webglcontextlost', (event) => {
  // BẮT BUỘC: Ngăn chặn browser engine hủy bỏ vĩnh viễn context
  event.preventDefault();
  isContextDead = true;
  console.warn('[WebGL] Context lost! Suspending render loop and preserving game state.');
  cancelAnimationFrame(renderLoopHandle);
}, false);

// Đón bắt sự kiện phục hồi ngữ cảnh GPU
canvas.addEventListener('webglcontextrestored', (event) => {
  console.log('[WebGL] Context restored! Re-initializing graphics pipeline...');
  isContextDead = false;
  // Tái tạo toàn bộ shaders, buffers, textures từ bộ nhớ tạm
  initShadersAndBuffers(gl);
  restoreAssetTextures(gl);
  // Khởi động lại render loop
  requestAnimationFrame(renderLoop);
}, false);

function renderLoop() {
  if (gl.isContextLost() || isContextDead) {
    return; // Dừng thực thi các draw calls khi context chưa sẵn sàng
  }
  // Thực hiện render bình thường
  gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);
  gl.drawElements(gl.TRIANGLES, indexCount, gl.UNSIGNED_SHORT, 0);
  renderLoopHandle = requestAnimationFrame(renderLoop);
}
```

### 3.3. Mô hình cách ly Capability-based với WASI 0.2 & Component Model

```wit
// miniapp-plugin.wit
package superapp:plugin@0.1.0;

interface processor {
    record ProcessConfig {
        quality: u32,
        watermark: string,
    }

    // Giao tiếp kiểu có cấu trúc an toàn tuyệt đối qua Canonical ABI
    process-image: func(input: list<u8>, config: ProcessConfig) -> result<list<u8>, string>;
}

world miniapp-sandbox {
    import wasi:clocks/monotonic-clock@0.2.0;
    import wasi:io/streams@0.2.0;
    // wasi:filesystem và wasi:http bị chặn mặc định trừ khi container cấp quyền
    export processor;
}
```

---

## 4. Tiêu chí kiểm định Store Review (QoS & Security Acceptance Gates)

1. **Gate NP-01 (Navigation Preload Compliance):**
   - Mini-app có kích thước bundle > 500KB hoặc phục vụ dữ liệu động bắt buộc phải kích hoạt `NavigationPreloadManager.enable()`.
   - Thời gian First Contentful Paint (FCP) trên mạng 3G/4G mô phỏng phải đạt dưới 1.200ms (giảm tối thiểu 150ms so với trường hợp không kích hoạt preload).

2. **Gate GL-02 (Context Loss Resilience Testing):**
   - Trong quá trình automated store review, test runner kích hoạt `WEBGL_lose_context.loseContext()`, giữ trạng thái trong 3.000ms, sau đó kích hoạt `restoreContext()`.
   - Mini-app phải phục hồi giao diện hoàn chỉnh trong vòng 1.500ms mà không sinh ra ngoại lệ JavaScript chưa được xử lý (`unhandledrejection` hoặc `uncaught error`).

3. **Gate WASM-03 (Binary Validation & Memory Quotas):**
   - File nhị phân WebAssembly (.wasm) tải lên phải vượt qua kiểm tra cấu trúc W3C Wasm Core Validation.
   - Dung lượng bộ nhớ tuyến tính khai báo tối đa (`initial` và `maximum` pages) không được vượt quá 128MB (2.048 Wasm pages). Mọi lệnh gọi hệ thống phải ánh xạ qua giao diện WASI 0.2 được cấp phép rõ ràng trong manifest.
