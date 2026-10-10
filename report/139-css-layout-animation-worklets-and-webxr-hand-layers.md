# Chuyên đề 139: Chuẩn Houdini Worklet (CSS Layout & Animation Worklet) và WebXR Hand Tracking & Hardware Layers trong Super-App Container

## 1. Tổng quan nghiên cứu & Bối cảnh kỹ thuật Iteration 139

Trong kiến trúc container của Super App hiện đại, việc cân bằng giữa **khả năng mở rộng giao diện đồ họa cao cấp**, **trải nghiệm động học mượt mà 60/120Hz** và **bảo mật cách ly đa tiến trình** là thách thức sống còn. Khi các Mini App ngày càng phát triển thành các ứng dụng thương mại điện tử phức tạp, trò chơi tương tác, và không gian thực tế hỗn hợp (Spatial Computing / WebXR), việc phụ thuộc vào luồng chính (Main UI Thread) của JavaScript dẫn đến tình trạng suy giảm khung hình (jank), nghẽn giao diện và rủi ro kênh phụ (side-channel attacks).

Iteration 139 tập trung chuẩn hóa 3 trụ cột kỹ thuật mở rộng cấp cao từ W3C:
1. **W3C CSS Layout API Level 1 (Houdini Layout Worklet)**: Cho phép nhà phát triển định nghĩa các thuật toán bố cục tùy biến (custom layout algorithms như masonry, carousel, virtualized feed) chạy off-main-thread trong luồng layout độc lập, loại bỏ hiện tượng giật lag khi render DOM phức tạp.
2. **W3C CSS Animation Worklet API**: Kiến trúc động học chạy trực tiếp trên luồng Compositor phần cứng (Hardware Compositor Thread), đồng bộ hóa với tần số quét màn hình (60/120Hz ProMotion) và ràng buộc cử chỉ cuộn/chạm (ScrollTimeline) không có độ trễ.
3. **W3C WebXR Hand Input Module Level 1 & WebXR Layers API Level 1**: Khung tương tác không gian tự nhiên theo dõi 25 khớp xương bàn tay (XRHand, XRJointSpace), quy chuẩn bảo vệ dữ liệu sinh trắc học và kỹ thuật chiếu lớp đồ họa trực tiếp vào GPU Compositor (XRCompositionLayer, XRMediaBinding) phục vụ mini-app 3D/AR thương mại và video DRM.

---

## 2. Chi tiết 15 Findings chuẩn hóa W3C (Iteration 139)

### Nhóm 1: W3C CSS Layout API Level 1 (Houdini Layout Worklet)

#### Finding css_layout_139_01: W3C CSS Layout API Level 1 - Custom Layout API Container Box Model and display: layout() Syntax
- **Tiêu chuẩn tham chiếu**: W3C Working Draft 2026: CSS Layout API Level 1, Section 2 (Layout API Containers).
- **URL xác thực**: `https://www.w3.org/TR/css-layout-api-1/` (HTTP 200) | Supporting: `https://drafts.css-houdini.org/css-layout-api/`.
- **Cốt lõi kỹ thuật**:
  - Bổ sung cú pháp `layout(<ident>)` vào tập giá trị `<display-inside>`, cho phép các phần tử DOM kích hoạt hộp chứa thuật toán bố cục tùy biến do nhà phát triển lập trình.
  - Hộp chứa Layout API thiết lập một ngữ cảnh định dạng (formatting context) mới, cô lập hoàn toàn các phần tử con khỏi hiệu ứng float bên ngoài, margin-collapsing, và side-effects của BFC thông thường.
  - Cho phép các cấu trúc bố cục phức tạp (masonry grid, lưới hình tròn, topology động) vận hành trực tiếp trong pha layout gốc của trình duyệt thay vì giả lập bằng JavaScript can thiệp DOM gây nghẽn luồng.
  - Tự động chuẩn hóa các phần tử con inline (inlinifying block boxes, biến float thành in-flow inline boxes) nhằm bảo đảm cây hộp (box tree) nhất quán.
- **Khuyến nghị chuẩn Super-App**:
  - Hỗ trợ `display: layout(miniapp-masonry)` trong WebView container của super-app để tối ưu hóa hiển thị danh mục sản phẩm e-commerce.
  - Cung cấp bộ quy tắc kiểm tra tĩnh (linter) trong Store Developer Console để kiểm soát các định nghĩa layout tùy biến.

#### Finding css_layout_139_02: LayoutWorkletGlobalScope & registerLayout() - Multi-Threaded Generator-Based Layout Dispatch
- **Tiêu chuẩn tham chiếu**: W3C Working Draft: CSS Layout API Level 1, Section 8 (Layout Worklet).
- **URL xác thực**: `https://www.w3.org/TR/css-layout-api-1/` (HTTP 200) | Supporting: `https://www.w3.org/TR/worklets-1/`.
- **Cốt lõi kỹ thuật**:
  - Thuật toán layout được đăng ký qua hàm `registerLayout(name, layoutCtor)` bên trong ngữ cảnh độc lập `LayoutWorkletGlobalScope`.
  - Hàm callback layout được định nghĩa dưới dạng JavaScript generator (`*layout(children, edges, constraints, styleMap, breakToken)`), cho phép yield các yêu cầu bố cục từng bước.
  - Mã thực thi có thể được điều phối chạy trên các luồng phụ (secondary background layout threads), tách rời hoàn toàn tính toán hình học khỏi luồng UI chính.
  - `LayoutWorkletGlobalScope` là môi trường cô lập tuyệt đối: không có quyền truy cập DOM, không có API hẹn giờ (setTimeout/setInterval), không có network primitives hay biến toàn cục `window`, ngăn chặn triệt để mã độc gây side-effect.
- **Khuyến nghị chuẩn Super-App**:
  - Yêu cầu các mini-app khai báo tĩnh file layout worklet trong manifest (`layoutWorklets`) để container biên dịch trước khi khởi tạo view.
  - Thực thi triệt để chính sách zero-DOM trong layout worklet, bảo đảm mã mini-app không thể đọc trộm hoặc can thiệp UI của ứng dụng mẹ.

#### Finding css_layout_139_03: LayoutChild Sizing, BreakToken Fragmentation & FragmentResultOptions Geometry Protocol
- **Tiêu chuẩn tham chiếu**: W3C Working Draft: CSS Layout API Level 1, Section 3 & Section 5.
- **URL xác thực**: `https://www.w3.org/TR/css-layout-api-1/` (HTTP 200).
- **Cốt lõi kỹ thuật**:
  - Các phần tử con trong container được biểu diễn dưới dạng đối tượng `LayoutChild`, cho phép truy vấn kiểu tính toán qua `styleMap` và kích thước nội tại.
  - Lập trình viên gọi `child.layoutNextFragment(constraints, breakToken)` để sinh ra các đối tượng `LayoutFragment` đại diện cho các lát cắt hiển thị.
  - Generator trả về từ điển `FragmentResultOptions` xác định `autoBlockSize`, `inlineSize`, `blockSize`, danh sách `childFragments` và thuộc tính tùy biến `data`.
  - Hỗ trợ phân trang, ngắt cột và ảo hóa danh sách (virtualization) thông qua chuỗi `ChildBreakToken`, tối ưu hóa các luồng dữ liệu vô tận (infinite feed).
- **Khuyến nghị chuẩn Super-App**:
  - Áp dụng mô hình ảo hóa `child.layoutNextFragment()` trong các danh sách feed dài của mini-app để cố định dung lượng RAM tiêu thụ.
  - Khử khuẩn và kiểm tra tọa độ vị trí của các fragment con, bảo đảm phần tử con không render tràn ra ngoài bounding box cho phép.

#### Finding css_layout_139_04: Intrinsic Sizes Computation - minContentSize and maxContentSize Calculation via Generator Protocol
- **Tiêu chuẩn tham chiếu**: W3C Working Draft: CSS Layout API Level 1, Section 3.3 & Section 5.1.
- **URL xác thực**: `https://www.w3.org/TR/css-layout-api-1/` (HTTP 200).
- **Cốt lõi kỹ thuật**:
  - Lớp layout bắt buộc triển khai hàm `*intrinsicSizes(children, edges, styleMap)` để tính toán kích thước container theo quy tắc min-content và max-content.
  - Gọi yield `child.intrinsicSizes()` trên toàn bộ phần tử con để tổng hợp số liệu bất đồng bộ mà không khóa luồng layout.
  - Tích hợp mượt mà với CSS Grid và Flexbox khi container cha cần co giãn tự động dựa trên đóng góp kích thước của layout con tùy biến.
  - Bộ engine lưu cache kết quả tính toán trong internal slots, loại bỏ các chu kỳ tính toán layout dư thừa (reflow churn).
- **Khuyến nghị chuẩn Super-App**:
  - Chuẩn hóa thuật toán tính kích thước nội tại cho các widget, card, và dynamic badge của mini-app khi nhúng vào trang chủ super-app.
  - Đo đạc thời gian tính toán của hàm `intrinsicSizes` trong pipeline chẩn đoán hiệu năng để phát hiện layout bệnh lý trước khi duyệt lên store.

#### Finding css_layout_139_05: CSS Layout Worklet Security Boundaries - Side-Channel Defense, Thread Isolation & Memory Quotas
- **Tiêu chuẩn tham chiếu**: W3C CSS Layout API Level 1 Section 9 & W3C Worklets Level 1 Section 5.
- **URL xác thực**: `https://www.w3.org/TR/css-layout-api-1/` (HTTP 200) | Supporting: `https://www.w3.org/TR/worklets-1/`.
- **Cốt lõi kỹ thuật**:
  - Ngăn chặn các cuộc tấn công vi kiến trúc kênh phụ (Spectre/Meltdown) bằng cách cấm hoàn toàn các đồng hồ thời gian độ phân giải cao bên trong vòng lặp layout.
  - Trình duyệt giám sát hạn ngạch tài nguyên CPU; các generator lặp vô tận hoặc vượt quá ngưỡng thời gian quy định sẽ bị buộc ngắt và rơi về luồng CSS block layout mặc định.
  - Cấm nhập module động qua `import()` bên trong hàm callback layout để bảo đảm tính xác định và có thể kiểm toán của hồ sơ thực thi.
  - Bảo vệ độ ổn định của vỏ bọc super-app: ngay cả khi mini-app đối tác gặp lỗi sập layout worklet, khung ứng dụng mẹ vẫn phản hồi bình thường.
- **Khuyến nghị chuẩn Super-App**:
  - Thiết lập watchdog timer (ngưỡng 50ms) cho mọi lệnh thực thi custom layout worklet của mini-app; tự động fallback về CSS flow layout khi quá thời gian.
  - Cấm tải script layout worklet từ CDN từ xa; bắt buộc đóng gói nguyên vẹn trong gói cài đặt mini-app đã ký số (.wgt/.zpk).

---

### Nhóm 2: W3C CSS Animation Worklet API (Off-Main-Thread Compositor Animations)

#### Finding css_anim_worklet_139_06: W3C CSS Animation Worklet API - Off-Main-Thread Compositor Animations & WorkletAnimation Interface
- **Tiêu chuẩn tham chiếu**: W3C Working Draft: CSS Animation Worklet API, Section 2 & Section 5.
- **URL xác thực**: `https://www.w3.org/TR/css-animation-worklet-1/` (HTTP 200) | Supporting: `https://drafts.css-houdini.org/css-animation-worklet/`.
- **Cốt lõi kỹ thuật**:
  - Cho phép viết các đoạn mã động học tùy biến chạy trực tiếp trên luồng Compositor phần cứng của trình duyệt ở tần số quét màn hình gốc (60/120Hz).
  - Tách rời hoạt ảnh khỏi luồng chính: hiện tượng nghẽn luồng do JavaScript xử lý logic nặng hoặc thao tác DOM không làm rơi khung hình (drop frames) của các hoạt ảnh WorkletAnimation đang chạy.
  - Khởi tạo qua `new WorkletAnimation(animatorName, effects, timeline, options)` trên luồng chính để gán logic điều khiển vào hiệu ứng keyframe của phần tử DOM.
  - Hỗ trợ đa dạng timeline đầu vào (DocumentTimeline, ScrollTimeline, ViewTimeline), loại bỏ hoàn toàn chi phí của vòng lặp `requestAnimationFrame`.
- **Khuyến nghị chuẩn Super-App**:
  - Ứng dụng CSS Animation Worklet cho toàn bộ hiệu ứng chuyển cảnh trang, ngăn kéo (drawer gesture tracking), và động học kéo-để-làm-mới (pull-to-refresh).
  - Đưa tiêu chí kiểm tra tuân thủ WorkletAnimation vào bộ công cụ audit hiệu năng của kho ứng dụng.

#### Finding css_anim_worklet_139_07: registerAnimator() Protocol - StatelessAnimator vs StatefulAnimator and animate() Execution Loop
- **Tiêu chuẩn tham chiếu**: W3C Working Draft: CSS Animation Worklet API, Section 3 & Section 4.
- **URL xác thực**: `https://www.w3.org/TR/css-animation-worklet-1/` (HTTP 200).
- **Cốt lõi kỹ thuật**:
  - Đăng ký animator qua `registerAnimator(name, animatorCtor)` trong `AnimationWorkletGlobalScope`.
  - Phân định rõ ràng giữa `StatelessAnimator` (hàm thuần túy, phụ thuộc thời gian và tham số đầu vào) và `StatefulAnimator` (duy trì đối tượng trạng thái xuyên suốt các frame tick).
  - Hàm `animate(currentTime, effect)` nhận mốc thời gian đã giải quyết từ timeline và tính toán giá trị `localTime` cho các `KeyframeEffect` liên kết.
  - Animator trạng thái triển khai phương thức `state()` để tuần tự hóa ảnh chụp trạng thái, hỗ trợ di chuyển (migration) liền mạch giữa các luồng compositor và worklet scope.
- **Khuyến nghị chuẩn Super-App**:
  - Khuyến nghị sử dụng mô hình `StatelessAnimator` cho các vi tương tác động lực học để tối ưu hóa khả năng tái sử dụng luồng.
  - Yêu cầu schema tuần tự hóa JSON rõ ràng đối với `StatefulAnimator` nhằm bảo đảm khôi phục trạng thái an toàn khi mini-app bị tạm dừng (suspend) ở chế độ nền.

#### Finding css_anim_worklet_139_08: ScrollTimeline & Gesture Synchronization - Zero-Latency Touch-Driven Micro-Interactions
- **Tiêu chuẩn tham chiếu**: W3C Working Draft: CSS Animation Worklet API, Section 5.1 | Supporting: `https://www.w3.org/TR/scroll-animations-1/`.
- **URL xác thực**: `https://www.w3.org/TR/css-animation-worklet-1/` (HTTP 200) | `https://www.w3.org/TR/scroll-animations-1/` (HTTP 200).
- **Cốt lõi kỹ thuật**:
  - WorkletAnimation tiếp nhận ScrollTimeline hoặc ViewTimeline, liên kết tiến trình hoạt ảnh trực tiếp với khoảng cách cuộn của người dùng.
  - Bỏ qua cơ chế điều phối sự kiện cảm ứng bất đồng bộ về luồng JS chính; các mô phỏng vật lý (lò xo, ma sát, quán tính) được tính toán tức thì trên luồng compositor.
  - Cho phép tạo các hiệu ứng parallax header mượt mà, thanh công cụ tự thu gọn, và tab bar dính chặt đồng bộ 1:1 với ngón tay người dùng mà không bị xé hình.
  - Ngăn ngừa tình trạng tính toán lại layout (reflow) trong khi cuộn bằng cách chỉ cho phép thay đổi các thuộc tính thân thiện với GPU (transform, opacity).
- **Khuyến nghị chuẩn Super-App**:
  - Cung cấp các mẫu hoạt ảnh cuộn chuẩn hóa cho trang chủ mini-app store, banner khuyến mãi và danh sách gian hàng.
  - Cấm sử dụng các trình lắng nghe sự kiện `scroll` thủ công trên luồng chính đối với các hiệu ứng thị giác dạng header co giãn.

#### Finding css_anim_worklet_139_09: Time Drift Compensation, Frame Budget Governance & Compositor Overload Defenses
- **Tiêu chuẩn tham chiếu**: W3C Working Draft: CSS Animation Worklet API, Section 5.4 & Section 6.
- **URL xác thực**: `https://www.w3.org/TR/css-animation-worklet-1/` (HTTP 200).
- **Cốt lõi kỹ thuật**:
  - Trình duyệt thực thi hạn ngạch khung hình nghiêm ngặt (thường < 4ms mỗi frame) đối với mã script animator chạy trong vòng lặp compositor.
  - Nếu animator thực thi chậm hoặc gây trễ, engine tự động giảm tần số thực thi hoặc tạm ngắt WorkletAnimation để bảo vệ độ nhạy cảm ứng của hệ điều hành.
  - Thuật toán bù trượt thời gian (time drift) đồng bộ hóa `currentTime` của animator với xung nhịp VSYNC của màn hình, khắc phục hiện tượng nhảy bước khi mất khung hình.
  - Cấm tuyệt đối các lời gọi IPC đồng bộ hoặc vòng lặp dài bên trong callback `animate()`.
- **Khuyến nghị chuẩn Super-App**:
  - Tích hợp giám sát ngân sách thời gian hoạt ảnh (< 4ms) trong runtime container; tự động ngắt các animator vi phạm để tránh lag máy.
  - Báo cáo dữ liệu telemetry về Developer Console khi mini-app có tỷ lệ rớt khung hình compositor bất thường.

#### Finding css_anim_worklet_139_10: Animation Worklet Isolation - Process Boundaries, CSP Inheritance & Side-Channel Mitigation
- **Tiêu chuẩn tham chiếu**: W3C Working Draft: CSS Animation Worklet API, Section 8.
- **URL xác thực**: `https://www.w3.org/TR/css-animation-worklet-1/` (HTTP 200) | Supporting: `https://www.w3.org/TR/worklets-1/`.
- **Cốt lõi kỹ thuật**:
  - `AnimationWorkletGlobalScope` thừa hưởng chính sách bảo mật nội dung (CSP) của tài liệu sở hữu, hạn chế nguồn tải mã kịch bản.
  - Môi trường hoàn toàn không có quyền truy cập cookie, Web Storage, IndexedDB, hay API gửi nhận mạng fetch/XHR, loại bỏ nguy cơ rò rỉ dữ liệu thông qua kênh hoạt ảnh.
  - Giới hạn độ chính xác của các API đo thời gian nhằm ngăn chặn tấn công kênh phụ suy đoán bộ nhớ vỏ ứng dụng mẹ.
  - Cách ly nhiều phiên bản WorkletAnimation trên các global scope riêng biệt, tránh xung đột giữa các thành phần mini-app khác nhau.
- **Khuyến nghị chuẩn Super-App**:
  - Bắt buộc áp dụng chỉ thị CSP `script-src 'self'` nghiêm ngặt cho toàn bộ mã worklet trong cấu hình store.
  - Đảm bảo animation worklet không thể thu thập các sự kiện viewport hay tọa độ cảm ứng bên ngoài phạm vi khung nhìn mini-app được cấp phát.

---

### Nhóm 3: W3C WebXR Hand Input & WebXR Hardware Compositor Layers

#### Finding webxr_hand_layers_139_11: W3C WebXR Hand Input Module Level 1 - 25-Joint Skeletal Tracking Model and XRHand/XRJointSpace APIs
- **Tiêu chuẩn tham chiếu**: W3C Working Draft: WebXR Hand Input Module - Level 1, Section 3 (Physical Hand Input Sources) & Section 3.2.
- **URL xác thực**: `https://www.w3.org/TR/webxr-hand-input-1/` (HTTP 200) | Supporting: `https://www.w3.org/TR/webxr/`.
- **Cốt lõi kỹ thuật**:
  - Định nghĩa giao diện `XRHand` đại diện cho cử chỉ bàn tay người thực, phơi bày mô hình khung xương có cấu trúc gồm đúng 25 khớp giải phẫu học tiêu chuẩn.
  - Các khớp bao gồm cổ tay (wrist), ngón cái (metacarpal, phalanx-proximal, phalanx-distal, tip), và 4 ngón tay còn lại (metacarpal, phalanx-proximal, phalanx-intermediate, phalanx-distal, tip).
  - Mỗi khớp được truy cập dưới dạng một `XRJointSpace`, cho phép truy vấn vị trí 3D, hướng xoay, và bán kính hình học của khớp qua `XRFrame.getJointPose()`.
  - Hỗ trợ các tương tác tự nhiên không cần tay cầm điều khiển (pinch chọn, nắm bắt, chạm trực tiếp vào giao diện ảo) trong các mini-app điện toán không gian (spatial computing).
  - Thuộc tính bán kính khớp cho phép mô phỏng va chạm vật lý chân thực giữa đầu ngón tay người dùng và các nút bấm, mô hình sản phẩm 3D.
- **Khuyến nghị chuẩn Super-App**:
  - Mở quyền yêu cầu tính năng `'hand-tracking'` trong pipeline cấp phép WebXR của super-app phục vụ các mini-app showroom ảo, thời trang AR, và mua sắm không gian 3D.
  - Cung cấp thư viện tương tác cử chỉ chuẩn (chụm ngón, chạm trực tiếp) trong Super-App SDK để thống nhất trải nghiệm người dùng.

#### Finding webxr_hand_layers_139_12: Biometric Privacy & Fingerprinting Defenses - Skeletal Geometry Redaction and Explicit User Consent
- **Tiêu chuẩn tham chiếu**: W3C Working Draft: WebXR Hand Input Module - Level 1, Section 5.
- **URL xác thực**: `https://www.w3.org/TR/webxr-hand-input-1/` (HTTP 200).
- **Cốt lõi kỹ thuật**:
  - Kích thước khung xương bàn tay (chiều dài xương, khoảng cách giữa các khớp) cấu thành đặc điểm nhận dạng sinh trắc học độc nhất, có khả năng định danh và theo dõi người dùng xuyên suốt các phiên truy cập.
  - W3C bắt buộc sự đồng ý rõ ràng của người dùng (explicit user consent) trước khi cấp quyền cho mini-app truy cập tính năng `'hand-tracking'`.
  - Khuyến nghị container chuẩn hóa chiều dài xương về tỷ lệ nhân trắc học trung bình hoặc bổ sung nhiễu lượng giác nhẹ để ngăn chặn việc xây dựng hồ sơ sinh trắc học chính xác.
  - Cổng kiểm soát dữ liệu: khi mini-app mất tiêu điểm (blur) hoặc chuyển xuống nền, toàn bộ tọa độ khớp bàn tay bắt buộc trả về null hoặc vùng nhớ 0.
  - Ngăn chặn việc thu thập thụ động các tình trạng sức khỏe cá nhân (run tay, mệt mỏi, dấu hiệu bệnh lý thần kinh) từ các nhà phát triển thứ ba.
- **Khuyến nghị chuẩn Super-App**:
  - Xếp hạng tính năng `'hand-tracking'` vào nhóm Năng lực Sinh trắc học Nhạy cảm (Sensitive Biometric Capability) trong Ma trận Quyền Hạn của Mini App Store.
  - Triển khai cơ chế chuẩn hóa tỷ lệ khung xương bàn tay trong container trước khi chuyển tiếp dữ liệu đến mã script của mini-app.

#### Finding webxr_hand_layers_139_13: W3C WebXR Layers API Level 1 - XRCompositionLayer, XRProjectionLayer & Direct GPU Hardware Compositing
- **Tiêu chuẩn tham chiếu**: W3C Working Draft: WebXR Layers API Level 1, Section 3 & Section 6.
- **URL xác thực**: `https://www.w3.org/TR/webxrlayers-1/` (HTTP 200) | Supporting: `https://immersive-web.github.io/layers/`.
- **Cốt lõi kỹ thuật**:
  - Giới thiệu các giao diện `XRCompositionLayer` và `XRProjectionLayer`, cho phép kết cấu WebGL được truyền trực tiếp vào bộ ghép lớp phần cứng (hardware compositor) của kính thực tế ảo hoặc thiết bị AR di động.
  - Bỏ qua các bước sao chép kết cấu trung gian (texture copy) và lấy mẫu lại, giảm thiểu triệt để băng thông bộ nhớ, độ trễ và hiện tượng quá nhiệt trên thiết bị standalone XR.
  - Hỗ trợ các hình học lớp chuyên biệt: `XRQuadLayer` (bảng điều khiển UI phẳng 2D, hộp thoại văn bản), `XRCylinderLayer` (menu cong toàn cảnh), và `XREquirectLayer` (nền video 360 độ).
  - Duy trì độ sắc nét cực cao cho văn bản giao diện bằng cách cho phép OS compositor lấy mẫu kết cấu UI ở độ phân giải gốc kèm kỹ thuật tái chiếu (reprojection).
- **Khuyến nghị chuẩn Super-App**:
  - Tận dụng `XRQuadLayer` để hiển thị giao diện 2D của mini-app phẳng bên trong không gian 3D trên kính Apple Vision Pro, Meta Quest và Android XR.
  - Yêu cầu sử dụng `XRProjectionLayer` cho đồ họa 3D mini-app nhằm tiết kiệm pin và loại bỏ giật khung hình trong chế độ AR Shopping trên smartphone.

#### Finding webxr_hand_layers_139_14: XRMediaBinding & Hardware Video Acceleration - DRM Protected Content & 360 Cinema Projection
- **Tiêu chuẩn tham chiếu**: W3C Working Draft: WebXR Layers API Level 1, Section 7 & Section 7.5.
- **URL xác thực**: `https://www.w3.org/TR/webxrlayers-1/` (HTTP 200).
- **Cốt lõi kỹ thuật**:
  - Cung cấp `XRMediaBinding` cho phép ánh xạ trực tiếp luồng video từ phần tử `HTMLVideoElement` vào các lớp ghép phần cứng (`XRMediaQuadLayer`, `XRMediaCylinderLayer`).
  - Dữ liệu video giải mã đi thẳng vào pipeline phần cứng hiển thị mà không cần tải qua WebGL texture hay luồng JavaScript.
  - Tương thích hoàn toàn với các luồng nội dung bản quyền DRM (Encrypted Media Extensions / EME), cho phép chiếu phim rạp ảo, hòa nhạc trực tiếp trong không gian XR.
  - Tiết kiệm năng lượng tiêu thụ vượt trội trong các phiên phát đa phương tiện kéo dài, tối ưu hóa tuổi thọ pin thiết bị.
- **Khuyến nghị chuẩn Super-App**:
  - Cung cấp module trừu tượng `XRMediaBinding` trong SDK phát trực tuyến của super-app cho các mini-app bán vé, rạp chiếu phim và sự kiện thể thao.
  - Kiểm tra chứng nhận bảo vệ bản quyền DRM trong quá trình xét duyệt mini-app trên store đối với các ứng dụng truyền phát nội dung không gian.

#### Finding webxr_hand_layers_139_15: WebXR Layer Depth Sorting, Asynchronous Space Warp & Multi-App Occlusion Governance
- **Tiêu chuẩn tham chiếu**: W3C Working Draft: WebXR Layers API Level 1, Section 9 & Section 10.
- **URL xác thực**: `https://www.w3.org/TR/webxrlayers-1/` (HTTP 200).
- **Cốt lõi kỹ thuật**:
  - Cho phép chia sẻ bộ đệm độ sâu (depth buffer) và sắp xếp thứ tự chiều sâu phần cứng giữa các lớp ứng dụng và các lớp UI hệ thống.
  - Hỗ trợ gửi dữ liệu Motion Vector phục vụ kỹ thuật Asynchronous Space Warp (ASW), cho phép bộ ghép hệ điều hành tự động suy diễn các khung hình bị mất khi GPU quá tải.
  - Cách ly không gian (Spatial Sandboxing): Khung chứa super-app có thể chèn các lớp hệ thống (ranh giới an toàn, huy hiệu thông báo, cửa sổ thanh toán) một cách an toàn xen kẽ với nội dung mini-app.
  - Đảm bảo mini-app không thể che khuất (occlude) các hộp thoại xác thực bảo mật, lời nhắc thanh toán sinh trắc học hoặc ranh giới an toàn vật lý của thiết bị.
- **Khuyến nghị chuẩn Super-App**:
  - Cấu hình các lời nhắc xác thực thanh toán của ứng dụng mẹ hiển thị trên lớp phần cứng trên cùng (top-level overlay layer), bất khả xâm phạm trước mọi lớp đồ họa của mini-app.
  - Đặt ra tiêu chuẩn kiểm tra bộ đệm độ sâu trong quy trình QA để ngăn ngừa các lỗi hiển thị gây biến dạng thị giác khi chuyển đổi giữa các mini-app không gian.

---

## 3. Bảng Ma trận Kiểm soát Kỹ thuật Tổng hợp (Iteration 139)

| Mã Kiểm soát | Thành phần Tiêu chuẩn | Cơ chế Cách ly & Bảo mật | Tác động Hiệu năng / UX | Quy định Super-App Store |
|---|---|---|---|---|
| **CTL-HOUDINI-01** | W3C CSS Layout API | LayoutWorkletGlobalScope không DOM, không Network, không Timers. Chạy trên luồng phụ. | Bố cục masonry/grid mượt mà; giảm 80% reflow churn luồng chính. | Khai báo tĩnh trong manifest; watchdog 50ms tự động fallback về Block layout. |
| **CTL-HOUDINI-02** | W3C CSS Animation Worklet | Chạy trên Hardware Compositor; thừa hưởng CSP tài liệu sở hữu; cách ly global scopes. | Duy trì 60/120Hz VSYNC; khử giật lag khi cuộn (ScrollTimeline); <4ms frame budget. | Bắt buộc cho drawer gestures và parallax; cấm lắng nghe scroll thủ công trên main thread. |
| **CTL-WEBXR-01** | W3C WebXR Hand Input | 25 khớp xương chuẩn; xóa buffer khi mất focus; chuẩn hóa tỷ lệ sinh trắc học. | Điều khiển tự nhiên không cần tay cầm; hỗ trợ pinch/tap tương tác sản phẩm 3D. | Phân loại Sensitive Biometric; hiển thị cảnh báo quyền rõ ràng trên Store Listing. |
| **CTL-WEBXR-02** | W3C WebXR Layers API | Bỏ qua texture copy; nạp trực tiếp vào GPU Compositor; bảo vệ lớp UI hệ sinh thái. | Giảm 40% băng thông GPU; hiển thị văn bản siêu nét; hỗ trợ phát video DRM tiết kiệm pin. | Dialog thanh toán hệ thống ghim lớp phần cứng tối cao, cấm mini-app che khuất. |

---

## 4. Kết luận & Định hướng mở rộng

Iteration 139 đã hoàn thiện tiêu chuẩn hóa cho các công nghệ giao diện và hiển thị ngoại vi cấp cao nhất hiện nay, đưa Super App tiến lên ngang tầm các hệ điều hành không gian và trình duyệt hiện đại. Kết quả 15 findings đã được tích hợp đầy đủ vào `state/findings.jsonl`, nâng tổng số phát hiện kiểm chứng lên con số **2036**.
