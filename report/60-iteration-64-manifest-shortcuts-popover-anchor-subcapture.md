# Chuyên đề 60: Chuẩn Hóa Khởi Chạy Nhanh (Manifest Shortcuts), Giao Diện Top-Layer & Neo Định Vị (Popover & CSS Anchor), và Điều Hướng Âm Thanh & Ghi Hình Phân Vùng DOM (Audio Output & Sub-Capture Isolation)

**Mã tài liệu:** `STD-SAMAS-ITER-064`
**Phiên bản chuẩn:** 1.0
**Ngày phê duyệt:** 09/10/2026
**Thuộc bộ chuẩn:** Super-App Mini-App Store Standard (SAMAS)
**Trạng thái:** Ban hành chính thức (Normative Standard)
**Số lượng findings bổ sung:** 15 canonical findings (`shortcuts_064_01` – `shortcuts_064_05`, `popover_064_01` – `popover_064_05`, `media_subcapture_064_01` – `media_subcapture_064_05`)
**Tổng số findings lũy kế:** 911 validated findings

---

## 1. Tổng Quan & Bối Cảnh Kỹ Thuật

Tại các nền tảng Super-App hiện đại quy mô hàng chục triệu người dùng, việc tối ưu hóa đường dẫn tương tác người dùng (user journey) đòi hỏi ba trụ cột công nghệ cốt lõi trên client runtime:
1. **Khởi chạy tức thời vào tính năng chuyên sâu (Deep-feature Fast Path):** Cho phép người dùng truy cập trực tiếp các tác vụ thường xuyên (như quét mã thanh toán, theo dõi đơn hàng, chuyển khoản nhanh) ngay từ menu ngữ cảnh ngoài màn hình chính của hệ điều hành (OS Launcher Quick Actions) mà không cần qua nhiều bước điều hướng trung gian.
2. **Kiến trúc giao diện lớp đỉnh không xung đột (Zero-Conflict Top-Layer UI & Declarative Anchoring):** Xóa bỏ hoàn toàn tình trạng tranh chấp thứ tự hiển thị CSS (`z-index` wars) và hiện tượng tràn khung chứa (`overflow: hidden` clipping) thông qua chuẩn native Popover API, kết hợp tính toán neo định vị giao diện (CSS Anchor Positioning) trực tiếp trên compositor thread thay thế các thư viện JavaScript nặng nề gây giật khung hình.
3. **Bảo mật luồng ngoại vi âm thanh & bảo vệ quyền riêng tư khi chia sẻ màn hình (Audio Routing Sandboxing & DOM Sub-Capture Isolation):** Cung cấp cơ chế điều hướng luồng âm thanh chuyên biệt (tai nghe Bluetooth, loa ngoài) có kiểm soát quyền truy cập chặt chẽ, đồng thời ứng dụng chuẩn W3C Region Capture & WICG Element Capture nhằm cô lập chính xác phân vùng tài liệu DOM khi phát trực tiếp hoặc họp video, ngăn chặn rò rỉ dữ liệu nhạy cảm (tin nhắn riêng tư, hộp thoại xác thực sinh trắc học, thanh điều hướng Super-App) vào luồng truyền thông mạng.

Chuyên đề này chuẩn hóa các yêu cầu kỹ thuật và chỉ tiêu kiểm định store cho ba phân hệ trên.

---

## 2. Chuẩn Khởi Chạy Nhanh Từ Màn Hình Chính (W3C Web App Manifest Shortcuts & OS Quick Actions)

### 2.1 Cấu Trúc Khai Báo `shortcuts` & Từ Điển `ShortcutInfo`
Theo chuẩn **W3C Web Application Manifest Working Draft**, thành phần `shortcuts` trong tệp kê khai ứng dụng (`app.manifest.json`) cho phép định nghĩa một danh sách các tác vụ trọng yếu:
* **`name` (Bắt buộc):** Chuỗi định danh hành động hiển thị trên menu ngữ cảnh launcher (giới hạn tối đa 20 ký tự).
* **`short_name` (Khuyến nghị):** Chuỗi thu gọn hiển thị trên các màn hình có không gian giới hạn (giới hạn tối đa 12 ký tự).
* **`description` (Bắt buộc theo chuẩn store):** Mô tả ngữ cảnh phục vụ công nghệ trợ năng và trình đọc màn hình Screen Reader (từ 10 đến 100 ký tự).
* **`url` (Bắt buộc):** Đường dẫn đích, bắt buộc phải thuộc phạm vi (`scope`) của Mini-App và sử dụng giao thức HTTPS hoặc lược đồ URI tùy chỉnh của Super-App.
* **`icons` (Bắt buộc):** Tập hợp tài nguyên hình ảnh định dạng PNG/SVG với kích thước chuẩn 96x96 px hoặc 192x192 px.

```json
{
  "name": "Food Express Mini App",
  "short_name": "FoodExpress",
  "start_url": "/index.html",
  "scope": "/",
  "shortcuts": [
    {
      "name": "Quét mã thanh toán",
      "short_name": "Quét mã",
      "description": "Mở camera quét mã VietQR thanh toán hóa đơn ăn uống",
      "url": "/scan?action=pay",
      "icons": [
        {
          "src": "/assets/icons/shortcut-scan.png",
          "sizes": "96x96 192x192",
          "type": "image/png",
          "purpose": "any monochrome"
        }
      ]
    },
    {
      "name": "Theo dõi đơn hàng",
      "short_name": "Đơn hàng",
      "description": "Kiểm tra tiến độ giao hàng của các đơn đang xử lý",
      "url": "/orders?filter=active",
      "icons": [
        {
          "src": "/assets/icons/shortcut-orders.png",
          "sizes": "96x96 192x192",
          "type": "image/png",
          "purpose": "any"
        }
      ]
    }
  ]
}
```

### 2.2 Quy Chuẩn Tài Nguyên Biểu Tượng & Thích Ứng Giao Diện OS (Monochrome Masking)
* **Kích thước & Dung lượng:** Mỗi Mini-App được khai báo tối đa 4 shortcuts. Tổng dung lượng toàn bộ biểu tượng shortcut trong gói không được vượt quá 256 KB.
* **Hỗ trợ Chủ đề Động (Material You & iOS Tinted Icons):** Biểu tượng khai báo thuộc tính `purpose: "monochrome"` để hệ điều hành tự động phủ màu theo bảng màu hệ thống người dùng (Android Dynamic Theming hoặc iOS Dark/Tinted Mode).
* **Bảo mật vị trí tài nguyên:** Tất cả tệp biểu tượng phải được đóng gói cục bộ bên trong gói Mini-App (`local package`), tuyệt đối cấm tải từ CDN ngoài nhằm ngăn ngừa rủi ro rò rỉ dữ liệu qua tiêu đề HTTP Referer hoặc trễ mạng khi hiển thị menu launcher.

### 2.3 Ánh Xạ Native Host Bridge (Android ShortcutManager & iOS UIApplicationShortcutItem)
* Khi người dùng ghim Mini-App ra màn hình chính hoặc đưa vào danh sách yêu thích, Native Container tự động đồng bộ danh mục shortcut vào hệ điều hành:
  * Trên Android: Container chuyển đổi sang `android.content.pm.ShortcutInfo.Builder` và đăng ký thông qua `ShortcutManager.setDynamicShortcuts()`.
  * Trên iOS: Container đăng ký danh sách `UIApplicationShortcutItem` tương ứng.
* **Cơ chế gọi lại Runtime:** Khi người dùng kích hoạt shortcut, Intent chuyển trực tiếp đến Activity/ViewController của Super-App. Gateway bảo mật xác minh tính hợp lệ của gói tin, khởi động sandbox Mini-App và phát sự kiện vòng đời `onShow({ path, query, scene: 1014, source: 'os_launcher_shortcut' })`.
* **Cập nhật động (Dynamic Runtime Shortcuts):** Cung cấp API cầu nối `my.setShortcuts()` cho phép Mini-App cập nhật shortcut theo ngữ cảnh (ví dụ: người nhận tiền quen thuộc, chuyến xe gần nhất), áp dụng hạn mức điều tiết (rate-limit) tối đa 5 lần cập nhật mỗi giờ.

---

## 3. Kiến Trúc Giao Diện Lớp Đỉnh & Neo Định Vị Không Mã Lệnh (HTML Popover & CSS Anchor Positioning)

### 3.1 Thuộc Tính Toàn Cầu `popover` & Xếp Chồng Lớp Đỉnh (Top Layer Stacking)
* Thuộc tính `popover="auto|manual"` theo chuẩn **WHATWG HTML Living Standard** đưa phần tử trực tiếp vào "Lớp Đỉnh" (Top Layer) của trình duyệt webview:
  * Hoàn toàn độc lập khỏi thứ tự xếp chồng CSS (`stacking contexts`), triệt tiêu triệt để các quy tắc cực đoan `z-index: 999999`.
  * Không bao giờ bị cắt xén (clipping) bởi các khung chứa cha có thiết lập `overflow: hidden`.
  * Hỗ trợ phần tử giả `::backdrop` giúp hiển thị màn mờ phủ nền mà không cần chèn các thẻ div bao bọc phức tạp.
* Kích hoạt hoàn toàn không cần JavaScript thông qua các thuộc tính khai báo trên nút bấm: `popovertarget="menu-id"` và `popovertargetaction="toggle|show|hide"`.

### 3.2 Cơ Chế Tự Động Thu Gọn (Light Dismiss) & Quản Lý Tiêu Điểm Truy Cập
* **Thu gọn tự động (Light Dismiss):** Popover chế độ `auto` tự động đóng khi người dùng chạm vào vùng trống ngoài menu, nhấn phím `Escape`, hoặc mở một popover `auto` khác không cùng huyết thống.
* **Cây phả hệ Popover (Ancestor Trees):** Trình duyệt tự động duy trì chuỗi quan hệ cha-con đối với các menu lồng nhau (nested sub-menus); việc nhấp chọn trong menu con không làm đóng các menu cha phía trên.
* **Tiêu điểm tiếp cận (Focus Management):** Khi đóng popover, tiêu điểm bàn phím tự động hoàn trả về chính xác phần tử kích hoạt ban đầu, bảo toàn trạng thái cuộn trang và không làm mất con trỏ trợ năng.

### 3.3 Neo Định Vị CSS (CSS Anchor Positioning Level 1) & Fallback Tránh Va Chạm
* Loại bỏ hoàn toàn sự phụ thuộc vào các thư viện JavaScript định vị tính toán tọa độ (Popper.js, Floating UI) vốn tiêu tốn CPU trên luồng chính (Main Thread):
  * Phần tử mốc (Anchor) khai báo: `anchor-name: --anchor-btn;`.
  * Phần tử menu nổi khai báo: `position-anchor: --anchor-btn; position-area: bottom span-right;`.
  * Trình duyệt tính toán và cập nhật vị trí tức thời ngay trên luồng tổng hợp đồ họa (Compositor Thread) trong suốt quá trình cuộn trang.
* **Quy tắc `@position-try` chống tràn màn hình:** Khi menu nổi gặp mép màn hình hoặc bị đẩy bởi bàn phím ảo, thuộc tính `position-try-fallbacks: flip-block, flip-inline;` tự động đảo chiều hiển thị lên phía trên hoặc vào trong mà không gây hiện tượng co giật giao diện.

```css
/* Khai báo phần tử mốc */
.action-button {
  anchor-name: --order-action-btn;
}

/* Khai báo menu nổi lớp đỉnh neo theo mốc */
.action-popover {
  position: fixed;
  position-anchor: --order-action-btn;
  position-area: bottom span-right;
  position-try-fallbacks: flip-block, flip-inline;
  margin: 8px 0;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
}
```

### 3.4 Chính Sách Đóng Khung Bảo Vệ Giao Diện Host (Anti-Capsule Hijacking)
* **Quy tắc an toàn bất biến:** Thanh điều hướng hệ thống và nút con nhộng (Native Capsule Button chứa nút Đóng, Xem thêm thông tin và Chứng chỉ bảo mật) của Super-App bắt buộc phải được render ở một View Layer riêng biệt phía trên Webview.
* Phần tử lớp đỉnh (`top layer`) của Webview tuyệt đối không được phép chồng lấn lên vùng an toàn đỉnh (`env(safe-area-inset-top)`) hoặc chặn tương tác cảm ứng của nút điều khiển Super-App. Bộ kiểm duyệt động tự động quét và đình chỉ các Mini-App cố tình tạo popover trong suốt che lấp nút Thoát của hệ thống.

---

## 4. Quản Trị Ngoại Vi Âm Thanh & Phân Vùng Ghi Hình DOM (Audio Output & Sub-Capture Isolation)

### 4.1 Điều Hướng Thiết Bị Phát Âm Thanh (W3C Audio Output Devices API)
* Cho phép các Mini-App chuyên dụng (họp trực tuyến, tổng đài chăm sóc khách hàng, khám bệnh từ xa) điều hướng luồng âm thanh trực tiếp đến thiết bị phần cứng mong muốn:
  * Phương thức `HTMLMediaElement.setSinkId(sinkId)` gán luồng âm thanh vào loa thoại (earpiece), loa ngoài (loudspeaker) hoặc tai nghe Bluetooth.
  * Quyền truy cập được kiểm soát bởi chỉ thị Permissions Policy: `speaker-selection`.
* **Bảo vệ chống định danh phần cứng (Anti-Fingerprinting):** Chuẩn hóa việc sử dụng `navigator.mediaDevices.selectAudioOutput()`. Trình duyệt hiển thị hộp thoại chọn thiết bị gốc của hệ thống thay vì cho phép script quét danh sách phần cứng; định danh `deviceId` trả về được băm kèm muối (salted) và phân vùng độc lập theo từng Mini-App origin.

### 4.2 Cắt Khung Hình Phát Theo Vùng DOM (W3C Region Capture & `CropTarget`)
* Khi Mini-App chia sẻ màn hình qua `getDisplayMedia()`, luồng video mặc định sẽ chứa toàn bộ khung nhìn webview.
* Giao diện `CropTarget` cho phép cô lập luồng video vào đúng khung bao chữ nhật của một phần tử DOM được chỉ định:
  ```javascript
  const targetElement = document.getElementById('presentation-slide');
  const cropTarget = await CropTarget.fromElement(targetElement);
  await screenVideoTrack.cropTo(cropTarget);
  ```
* Bất cứ khi nào phần tử mục tiêu thay đổi kích thước hoặc di chuyển, luồng video sẽ tự động bám theo mà không để lộ các phần tử xung quanh trên trang.

### 4.3 Cách Ly Vùng Ghi Hình Không Bị Che Khuất (WICG Element Capture & `RestrictionTarget`)
* Khắc phục nhược điểm của Region Capture (khi một thông báo popover hoặc tin nhắn chat nổi đè lên trên vùng cắt chữ nhật thì người xem video vẫn nhìn thấy nội dung đè đó):
* Chuẩn **WICG Element Capture** sử dụng `RestrictionTarget` để chỉ ghi nhận duy nhất cây con DOM mục tiêu:
  ```javascript
  const confidentialSpreadsheet = document.getElementById('data-sheet');
  const restrictionTarget = await RestrictionTarget.fromElement(confidentialSpreadsheet);
  await screenVideoTrack.restrictTo(restrictionTarget);
  ```
* Mọi phần tử anh em, hộp thoại thông báo đẩy, hoặc popover lớp đỉnh nổi phía trên phần tử mục tiêu sẽ **hoàn toàn bị loại bỏ khỏi các khung hình video được mã hóa**, ngăn chặn rò rỉ dữ liệu nhạy cảm của khách hàng trong suốt phiên làm việc trực tiếp.
* **Huy hiệu bảo vệ quyền riêng tư:** Super-App hiển thị dải thông báo thường trực trên màn hình khi tính năng ghi hình phân vùng đang hoạt động, kèm nút bấm ngắt kết nối khẩn cấp (1-tap kill switch).

---

## 5. Bảng Ma Trận Kiểm Soát Tiêu Chuẩn Store (Store Compliance Control Matrix)

| Mã kiểm soát | Hạng mục kiểm tra | Tiêu chuẩn tham chiếu | Cấp độ bắt buộc | Biện pháp kiểm định Store |
| :--- | :--- | :--- | :--- | :--- |
| **CTL-SHORTCUT-01** | Khai báo `shortcuts` trong Manifest | W3C App Manifest (§ shortcuts) | Bắt buộc (Max 4) | Phân tích tệp kê khai `app.manifest.json`, từ chối nếu có > 4 shortcuts hoặc URL trỏ ra ngoài phạm vi `scope`. |
| **CTL-SHORTCUT-02** | Quy chuẩn biểu tượng Shortcut | W3C Manifest Image Resource | Bắt buộc | Kiểm tra kích thước (96x96 px hoặc 192x192 px), tổng dung lượng gói <= 256 KB, nguồn tệp nội bộ. |
| **CTL-SHORTCUT-03** | Mô tả trợ năng Accessible Description | WCAG 2.2 / W3C Manifest | Bắt buộc | Kiểm tra thuộc tính `description` có độ dài từ 10 - 100 ký tự, không được để trống hoặc trùng lặp với `name`. |
| **CTL-POPOVER-01** | Sử dụng Popover Top-Layer | WHATWG HTML Popover API | Khuyến nghị cao | Linter phát hiện và cảnh báo các phần tử modal tùy biến lạm dụng `z-index > 10000`. |
| **CTL-POPOVER-02** | Thu gọn tự động (Light Dismiss) | WHATWG Popover auto | Bắt buộc | Kiểm tra hành vi phím `Escape` và chạm vùng ngoài phải đóng menu, khôi phục tiêu điểm đúng nút kích hoạt. |
| **CTL-POPOVER-03** | Chống chiếm quyền nút điều khiển Host | Host Security Sandbox Policy | Bắt buộc | Ngăn chặn việc hiển thị popover trong vùng an toàn đỉnh (`env(safe-area-inset-top)`) che lấp nút Thoát. |
| **CTL-ANCHOR-01** | Định vị CSS Anchor & Fallbacks | W3C CSS Anchor Positioning | Khuyến nghị | Kiểm tra khai báo `position-try-fallbacks` chống tràn màn hình và cấm gắn event listener cuộn trang để định vị. |
| **CTL-MEDIA-01** | Điều hướng âm thanh qua `setSinkId` | W3C Audio Output Devices | Phân quyền manifest | Yêu cầu khai báo quyền `speaker-selection`; tự động thu hồi khi Mini-App chuyển xuống chạy nền. |
| **CTL-MEDIA-02** | Chọn thiết bị phát có người dùng xác nhận | W3C selectAudioOutput API | Bắt buộc | Cấm quét định danh phần cứng; chỉ chấp nhận hộp thoại chọn thiết bị do trình duyệt trung gian điều khiển. |
| **CTL-MEDIA-03** | Cách ly chia sẻ màn hình với Element Capture | WICG Element Capture / Region Capture | Bắt buộc (Tài chính/Y tế) | Mini-App chia sẻ tài liệu nhạy cảm bắt buộc ứng dụng `restrictTo` hoặc `cropTo` để che giấu dữ liệu cá nhân. |

---

## 6. Lộ Trình Triển Khai Thực Thi & Kế Hoạch Vòng Kế Tiếp

1. **Hoàn thành chỉ tiêu vòng 64:** Bổ sung trọn vẹn 15 findings quy chuẩn vào kho tri thức chuẩn hóa (nâng tổng số lên 911 findings), cập nhật báo cáo kỹ thuật chuyên đề và bảng mục lục README.
2. **Kế hoạch vòng 65 (Cột mốc đồng bộ Public Repository):**
   * Hoàn thành vòng nghiên cứu 65, cán mốc 926+ validated findings.
   * Sao chép toàn bộ các tệp báo cáo chuyên đề mới nhất (`01` đến `60`), tệp tổng hợp `evidence/findings.jsonl` và `evidence/validation-summary.json` sang kho mã nguồn mở `/home/minhgv/code/github/new-superapp`.
   * Thực hiện quy trình quét mã độc/bí mật (`secret scan`), kiểm tra diff staging, tạo conventional commit và push chính thức lên nhánh `main`.
