# Chuyên đề 76: Chuẩn W3C CSS Scroll Snap & Overscroll Behavior Module Level 1, WHATWG HTML Canvas 2D Context & Path2D Sandboxing, và W3C/WHATWG ImageBitmap, OffscreenCanvas Transfer & Programmatic Scrolling Governance

## 1. Bối cảnh & Mục tiêu Kỹ thuật

Trong môi trường WebView đa ứng dụng của Super-App, việc đảm bảo tính mượt mà của cử chỉ vuốt cuộn (smooth gesture physics) và hiệu năng đồ họa 2D (canvas graphics throughput) là yếu tố sống còn để đạt trải nghiệm tương đương ứng dụng native. Iteration 80 tập trung giải quyết 3 thách thức kiến trúc:
1. **Kiểm soát cơ chế cuộn và phân lập cử chỉ (Scroll Snap & Overscroll Sandboxing):** Các carousel khám phá sản phẩm, slider câu chuyện và giao diện phân trang mini-app đòi hỏi các điểm dừng cuộn (snap points) phần cứng chính xác mà không cần phụ thuộc vào các thư viện JavaScript nặng nề. Đồng thời, hành vi cuộn quá đà (overscroll) của modal nội bộ mini-app phải được cô lập tuyệt đối, tránh kích hoạt nhầm cử chỉ pull-to-refresh hoặc vuốt back của vỏ container Super-App.
2. **Đóng gói và an toàn ngữ cảnh đồ họa 2D (Canvas 2D Context & Path2D Sandboxing):** Các mini-game, công cụ vẽ biểu đồ tài chính và tính năng chụp chữ ký điện tử cần môi trường render 2D tách biệt. Việc quản lý stack trạng thái (save/restore) ngăn ngừa nhiễm bẩn ma trận biến đổi, Path2D tăng tốc GPU cho các hình học vector tái sử dụng, trong khi cơ chế origin-taint bảo vệ dữ liệu pixel không bị đánh cắp bởi các mã theo dõi ngầm.
3. **Giải mã hình ảnh phi đồng bộ và chuyển giao đồ họa luồng nền (ImageBitmap & OffscreenCanvas Governance):** Giải mã các tệp ảnh lớn trên worker ngầm bằng `createImageBitmap()` và xuất hình trực tiếp vào DOM canvas qua ngữ cảnh zero-copy `bitmaprenderer` (`ImageBitmapRenderingContext`), kết hợp chuyển quyền kiểm soát canvas sang DedicatedWorker bằng `transferControlToOffscreen()` để bảo toàn tốc độ phản hồi 60fps/120fps cho luồng giao diện chính.

---

## 2. Trụ cột 1: Chuẩn W3C CSS Scroll Snap Module Level 1 & Overscroll Behavior Module Level 1

### 2.1 Khởi tạo Khung Cuộn Khóa Điểm (`scroll-snap-type`)
- **Đặc tả kỹ thuật:** Thuộc tính `scroll-snap-type` quy định mức độ bắt buộc áp dụng các điểm dừng cuộn trên container. Hỗ trợ trục cuộn (`x`, `y`, `block`, `inline`, `both`) và độ nghiêm ngặt (`mandatory`, `proximity`). Với `mandatory`, container luôn phải dừng tại một snap point khi kết thúc thao tác cuộn; với `proximity`, container chỉ khóa điểm khi cử chỉ dừng gần snap point.
- **Ứng dụng Super-App:** Các mini-app catalog sản phẩm và onboarding stories kích hoạt `scroll-snap-type: x mandatory` trên thanh trượt ngang. Thao tác lướt được xử lý trực tiếp trên luồng compositor của GPU mà không tốn tài nguyên CPU main-thread, loại bỏ hoàn toàn hiện tượng giật lag khi người dùng vuốt nhanh qua danh sách.

### 2.2 Căn chỉnh Điểm dừng Khối Con (`scroll-snap-align`)
- **Đặc tả kỹ thuật:** Thuộc tính `scroll-snap-align` chỉ định vị trí neo của vùng snap phần tử con (`snap area`) so với khung nhìn của container (`snapport`). Hỗ trợ các giá trị `none`, `start`, `end`, `center` cùng cú pháp hai trục độc lập.
- **Ứng dụng Super-App:** Trong các hàng hiển thị thẻ mini-app nổi bật, các phần tử con áp dụng `scroll-snap-align: center` hoặc `start`. Thiết lập này đảm bảo các thẻ dịch vụ luôn nằm chính giữa hoặc căn thẳng lề trái màn hình thiết bị sau khi cuộn, bất kể tỷ lệ khung hình hay độ phân giải của các dòng điện thoại khác nhau.

### 2.3 Khống chế Gia tốc Cử chỉ Vuốt Lướt (`scroll-snap-stop`)
- **Đặc tả kỹ thuật:** Thuộc tính `scroll-snap-stop` điều khiển việc bộ cuộn có được phép 'bỏ qua' các điểm dừng khi người dùng thực hiện cú vuốt mạnh (high-velocity flick). Giá trị bao gồm `normal` (cho phép lướt qua nhiều điểm dựa trên đà quán tính) và `always` (bắt buộc bộ cuộn phải dừng lại tại điểm neo ngay liền kề).
- **Ứng dụng Super-App:** Trong các luồng giao dịch nhạy cảm như wizard thanh toán nhiều bước hoặc màn hình điều khoản pháp lý, mini-app áp dụng `scroll-snap-stop: always`. Điều này ngăn ngừa người dùng vô tình vuốt lướt qua các bước xác nhận quan trọng, đảm bảo tính chặt chẽ của quy trình tuân thủ.

### 2.4 Phân lập Biên Cuộn và Ngăn Chặn Chaining (`overscroll-behavior`)
- **Đặc tả kỹ thuật:** Thuộc tính `overscroll-behavior` (`auto`, `contain`, `none`) quy định hành vi của trình duyệt khi cuộn chạm biên container. Giá trị `contain` loại bỏ hiện tượng lan truyền cuộn (scroll chaining) lên các phần tử cha nhưng giữ lại hiệu ứng nảy/cuộn cục bộ, trong khi `none` triệt tiêu hoàn toàn cả scroll chaining lẫn hiệu ứng nảy.
- **Ứng dụng Super-App:** Các modal popup và danh sách cuộn nội bộ của mini-app bắt buộc khai báo `overscroll-behavior-y: contain` hoặc `none`. Thiết lập này ngăn chặn triệt để hiện tượng cuộn hết nội dung modal bị lan truyền lên document cha, làm kích hoạt nhầm tính năng làm mới trang (pull-to-refresh) hoặc cử chỉ đóng app của vỏ bọc Super-App container.

### 2.5 Khoảng đệm Khung nhìn cho Thanh Điều hướng (`scroll-padding` & `scroll-margin`)
- **Đặc tả kỹ thuật:** Thuộc tính `scroll-padding` xác định khoảng cách thụt lùi của vùng hiển thị tối ưu (`snapport`), trong khi `scroll-margin` mở rộng hoặc thu hẹp vùng tính toán snap của từng phần tử mà không làm thay đổi kích thước hộp layout thực tế.
- **Ứng dụng Super-App:** Để tránh tình trạng phần tử sau khi snap bị che khuất bởi nút capsule hệ thống (WeChat-like header capsule) hoặc thanh navigation cố định của host, mini-app root container thiết lập `scroll-padding-top: env(safe-area-inset-top, 44px)`. Đảm bảo các tiêu đề và nút chức năng luôn hiển thị toàn vẹn và dễ bấm.

---

## 3. Trụ cột 2: Chuẩn W3C HTML Canvas 2D Context & Path2D Sandboxing

### 3.1 Ngăn xếp Trạng thái Đồ họa (`save()` & `restore()`)
- **Đặc tả kỹ thuật:** Giao diện `CanvasRenderingContext2D` duy trì một drawing state stack dạng LIFO. Trạng thái bao gồm ma trận biến đổi tọa độ, vùng cắt (clip path), màu nét/vùng tô (`strokeStyle`, `fillStyle`), độ trong suốt `globalAlpha`, và chế độ hòa trộn `globalCompositeOperation`. `save()` đẩy bản sao trạng thái vào stack, `restore()` khôi phục trạng thái gần nhất.
- **Ứng dụng Super-App:** Trong các mini-game hoặc module biểu đồ tích hợp nhiều plugin bên thứ ba vẽ chung trên một canvas, việc đóng gói các phép biến đổi ma trận trong cặp lệnh `ctx.save()` / `ctx.restore()` ngăn ngừa tình trạng plugin làm sai lệch hệ tọa độ hoặc chế độ blend màu của các thành phần đồ họa khác của mini-app.

### 3.2 Tối ưu Hóa và Tái sử dụng Hình học Vector với `Path2D`
- **Đặc tả kỹ thuật:** Đối tượng `Path2D` cho phép lưu trữ và biên dịch sẵn các đường dẫn vector 2D từ các lệnh vẽ hoặc chuỗi ký tự SVG path data (ví dụ: `new Path2D('M10 10 h 80 v 80 h -80 Z')`). Phương thức `ctx.fill(path)` và `ctx.stroke(path)` thực thi trực tiếp trên GPU.
- **Ứng dụng Super-App:** Các icon vector và bản đồ tuyến đường trong mini-app biên dịch sẵn hình học vào đối tượng `Path2D` khi khởi tạo. Việc tái sử dụng `Path2D` trong vòng lặp render 60/120fps loại bỏ hoàn toàn chi phí phân tích chuỗi SVG và tính toán phân mảnh đa giác (tessellation) trên CPU di động, tiết kiệm pin tối đa.

### 3.3 An ninh Pixel Canvas và Phòng thủ Taint (`getImageData`)
- **Đặc tả kỹ thuật:** Khi canvas vẽ hình ảnh hoặc video từ origin khác mà không có tiêu đề CORS hợp lệ, cờ `origin-clean` của canvas bị đặt vĩnh viễn thành `false` (bị vấy bẩn - tainted). Mọi nỗ lực gọi `ctx.getImageData()`, `canvas.toDataURL()`, hoặc `canvas.toBlob()` lập tức kích hoạt ngoại lệ `SecurityError` DOMException.
- **Ứng dụng Super-App:** Host container kiểm soát chặt chẽ cờ CORS khi mini-app nạp hình ảnh từ CDN. Đồng thời, lớp bảo mật container giám sát tần suất gọi `getImageData()` để phát hiện và ngăn chặn các kỹ thuật lấy dấu vân tay trình duyệt (canvas fingerprinting) hoặc trích xuất trái phép dữ liệu hình ảnh nhạy cảm (như chữ ký, ảnh chụp căn cước).

### 3.4 Xuất Dữ liệu Nhị phân Phi đồng bộ với `HTMLCanvasElement.toBlob()`
- **Đặc tả kỹ thuật:** Phương thức `toBlob()` mã hóa nội dung raster của canvas thành đối tượng `Blob` nhị phân thuần túy dưới tiến trình phi đồng bộ. Hỗ trợ định dạng `image/webp`, `image/png`, `image/jpeg` kèm tham số chất lượng nén (0.0 đến 1.0).
- **Ứng dụng Super-App:** Tác vụ chụp chữ ký khách hàng hoặc cắt ảnh đại diện trong mini-app sử dụng `canvas.toBlob(callback, 'image/webp', 0.8)`. Quá trình nén diễn ra ngầm không làm đơ giao diện người dùng, đồng thời giảm 33% dung lượng bộ nhớ so với việc chuyển đổi sang chuỗi base64 qua `toDataURL()`.

### 3.5 Cắt Ghép Sprite Nhanh qua `drawImage()` 9 Tham số
- **Đặc tả kỹ thuật:** Cú pháp 9 tham số `drawImage(image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight)` cho phép cắt một vùng chữ nhật từ ảnh nguồn và vẽ co giãn lên tọa độ đích của canvas với hiệu năng tăng tốc GPU.
- **Ứng dụng Super-App:** Kỹ thuật gom cụm tài nguyên đồ họa (texture atlas / sprite sheet) trong mini-game gom hàng chục icon và khung hoạt ảnh vào một tệp ảnh duy nhất. Sử dụng `drawImage` 9 tham số giúp hiển thị hoạt ảnh mượt mà mà chỉ tiêu tốn đúng một lượt tải mạng ban đầu.

---

## 4. Trụ cột 3: W3C/WHATWG ImageBitmap, OffscreenCanvas Transfer & Programmatic Scrolling Governance

### 4.1 Giải mã Hình ảnh Luồng nền với `createImageBitmap()`
- **Đặc tả kỹ thuật:** Phương thức toàn cục `createImageBitmap()` tạo đối tượng `ImageBitmap` từ đa dạng nguồn ảnh (Blob, ImageData, Canvas, Image). Việc giải mã nén điểm ảnh (decoding) và chuyển đổi không gian màu diễn ra phi đồng bộ ngoài luồng UI chính.
- **Ứng dụng Super-App:** Thư viện ảnh và danh sách sản phẩm thương mại điện tử giải mã các tệp ảnh nhiều megapixel bằng `createImageBitmap()` kèm tùy chọn `resizeWidth: 800` để thu nhỏ ảnh ngay từ tầng phần cứng. Main-thread hoàn toàn không bị chặn, bảo đảm chỉ số INP luôn dưới 50ms.

### 4.2 Trình chiếu Đồ họa Zero-Copy với `ImageBitmapRenderingContext`
- **Đặc tả kỹ thuật:** Ngữ cảnh `bitmaprenderer` chỉ cung cấp tính năng thay thế nội dung canvas bằng đối tượng `ImageBitmap` thông qua phương thức `transferFromImageBitmap()`. Thao tác này chuyển giao trực tiếp con trỏ bộ đệm GPU (pointer swap) mà không tốn công rasterization hay sao chép bộ nhớ.
- **Ứng dụng Super-App:** Các mini-app xử lý video thời gian thực hoặc bộ lọc camera AR nạp khung hình đã render từ Web Worker lên giao diện chính qua `transferFromImageBitmap()`. Cơ chế zero-copy triệt tiêu hiện tượng thắt cổ chai băng thông VRAM và hạ nhiệt độ chip di động.

### 4.3 Cách ly Đồ họa Nặng sang Worker với `transferControlToOffscreen()`
- **Đặc tả kỹ thuật:** Phương thức `transferControlToOffscreen()` chuyển quyền điều khiển canvas HTML sang một đối tượng `OffscreenCanvas` hoạt động độc lập trong DedicatedWorker. Sau khi chuyển quyền, mọi lệnh vẽ WebGL2/WebGPU/2D thực thi ngầm.
- **Ứng dụng Super-App:** Các mini-game 3D phức tạp hoặc bản đồ vệ tinh giao toàn bộ quyền vẽ cho DedicatedWorker. Nếu tác vụ đồ họa bị quá tải tụt khung hình, luồng giao diện chính của Super-App và thanh điều hướng hệ thống vẫn phản hồi tức thì với thao tác bấm của người dùng.

### 4.4 Căn chỉnh Khung nhìn Theo Ngữ cảnh với `Element.scrollIntoView()`
- **Đặc tả kỹ thuật:** Phương thức `scrollIntoView()` cuộn các container tổ tiên để đưa phần tử mục tiêu vào vùng nhìn thấy. Tùy chọn `scrollIntoViewOptions` cho phép cấu hình hiệu ứng mượt (`behavior: 'smooth'`) và vị trí căn trục khối/nội dòng (`block: 'center'`).
- **Ứng dụng Super-App:** Khi người dùng nhập sai thông tin trong form đăng ký, mini-app tự động gọi `element.scrollIntoView({ behavior: 'smooth', block: 'center' })` để cuộn nhẹ nhàng đến ô nhập liệu lỗi, tạo trải nghiệm điều hướng tự nhiên và thân thiện trên màn hình cảm ứng.

### 4.5 Điều khiển Cuộn Toàn cục và Tôn trọng Khuyết tật Vận động (`scrollTo` & `scroll-behavior`)
- **Đặc tả kỹ thuật:** Phương thức `scrollTo()` điều hướng container đến tọa độ pixel xác định. Thuộc tính CSS `scroll-behavior: smooth` tạo chuyển động cuộn mềm mại cho toàn trang. Tuy nhiên, hiệu ứng này bắt buộc phải hạ cấp về `instant` khi người dùng kích hoạt cấu hình giảm chuyển động (`@media (prefers-reduced-motion: reduce)`).
- **Ứng dụng Super-App:** Container Super-App tự động tiêm quy tắc ghi đè `scroll-behavior: auto !important` khi hệ điều hành người dùng bật tính năng giảm chuyển động, phòng ngừa triệu chứng chóng mặt, buồn nôn cho người khuyết tật tiền đình theo khuyến nghị WCAG 2.2 Level AA.

---

## 5. Tổng kết Ma trận Tiêu chuẩn Kỹ thuật Iteration 80

| Nhóm Tiêu chuẩn | Thành phần Cốt lõi | Đặc tả Tiêu chuẩn | Chỉ số / Ràng buộc Kỹ thuật |
| :--- | :--- | :--- | :--- |
| **W3C CSS Scroll Snap 1** | `scroll-snap-type`, `mandatory` | W3C CSS Scroll Snap Level 1 | Khóa điểm cuộn phần cứng 120fps trên luồng GPU compositor cho carousel và feed. |
| **W3C CSS Scroll Snap 1** | `scroll-snap-align`, `start/center` | W3C CSS Scroll Snap Level 1 | Căn thẳng lề hoặc căn giữa thẻ sản phẩm đồng nhất trên mọi độ phân giải màn hình. |
| **W3C CSS Scroll Snap 1** | `scroll-snap-stop: always` | W3C CSS Scroll Snap Level 1 | Bẫy gia tốc vuốt quán tính, chống bỏ qua bước xác nhận trong wizard thanh toán. |
| **W3C CSS Overscroll 1** | `overscroll-behavior: contain/none` | W3C CSS Overscroll Behavior 1 | Cách ly biên cuộn modal, triệt tiêu scroll chaining kích hoạt nhầm pull-to-refresh. |
| **W3C CSS Scroll Snap 1** | `scroll-padding`, `safe-area` | W3C CSS Scroll Snap Level 1 | Thụt lùi khung nhìn snapport, chống che khuất nội dung bởi host header capsule. |
| **WHATWG Canvas 2D** | `ctx.save()`, `ctx.restore()` | WHATWG HTML Living Standard | Ngăn xếp trạng thái LIFO, cô lập ma trận biến đổi và chế độ hòa trộn của plugin. |
| **WHATWG Canvas 2D** | `Path2D`, `addPath()` | WHATWG HTML Living Standard | Biên dịch sẵn hình học vector SVG, loại bỏ chi phí phân tích chuỗi trong loop 60fps. |
| **WHATWG Canvas 2D** | Origin Taint, `getImageData()` | WHATWG HTML Living Standard | Khóa quyền đọc pixel canvas khi thiếu CORS, phòng chống canvas fingerprinting. |
| **WHATWG Canvas 2D** | `canvas.toBlob()`, `image/webp` | WHATWG HTML Living Standard | Mã hóa nhị phân phi đồng bộ ngoài main-thread, giảm 33% RAM so với base64. |
| **WHATWG Canvas 2D** | `drawImage()` 9 tham số | WHATWG HTML Living Standard | Cắt ghép sprite sheet atlas trực tiếp trong bộ nhớ VRAM với 1 request mạng duy nhất. |
| **WHATWG ImageBitmap** | `createImageBitmap()` | WHATWG HTML Living Standard | Giải mã ảnh lớn và downsampling phi đồng bộ trên luồng nền, giữ INP < 50ms. |
| **WHATWG Canvas Context** | `ImageBitmapRenderingContext` | WHATWG HTML Living Standard | Ngữ cảnh `bitmaprenderer` tráo con trỏ buffer GPU zero-copy cho video và AR filter. |
| **WHATWG OffscreenCanvas** | `transferControlToOffscreen()` | WHATWG HTML Living Standard | Chuyển quyền render canvas sang DedicatedWorker, bảo vệ main-thread khỏi lag đồ họa. |
| **W3C CSSOM View** | `scrollIntoView({ smooth, center })`| W3C CSSOM View Module | Định vị mượt mà ô nhập liệu lỗi trong biểu mẫu theo chuẩn công thái học di động. |
| **W3C CSSOM View & CSS** | `scrollTo()`, `scroll-behavior` | W3C CSSOM View / CSS Scroll | Cuộn trang tọa độ chính xác, tự động fallback về instant khi bật prefers-reduced-motion. |
