# Chuyên Đề 98 (Iteration 102): Web Audio API Processing Graphs, Synthesis, Routing & Offline Rendering

## 1. Bối Cảnh & Mục Tiêu Chuẩn Hóa
Trong kiến trúc Super App hiện đại, mini app không chỉ là các trang hiển thị biểu mẫu hay tra cứu dữ liệu đơn giản mà ngày càng mở rộng sang các trải nghiệm tương tác đa phương tiện cao cấp như: âm thanh giao diện người dùng (tactile UI feedback), mini game casual/action, phòng livestream tương tác, lớp học trực tuyến, hội nghị âm thanh, và các ứng dụng biên tập âm thanh di động.

Để đảm bảo hiệu năng cao, độ trễ cực thấp (sub-millisecond scheduling) và bảo vệ phần cứng âm thanh di động (tránh giật lag, acoustic clipping, shock âm lượng đột ngột, và rò rỉ bộ nhớ DSP), chuẩn hóa **Web Audio API** toàn diện cho Super App Web Container là bắt buộc. Chuyên đề 98 hệ thống hóa 15 thành phần cốt lõi của Web Audio API thành 3 trụ cột kỹ thuật:
1. **Lọc, Phân Tích & Điều Khiển Động Học Âm Thanh (Filtering, Analysis & Dynamics)**: `BiquadFilterNode`, `AnalyserNode`, `GainNode`, `StereoPannerNode`, `WaveShaperNode`.
2. **Tổng Hợp Sóng, Điều Khiển Tham Số & Ma Trận Định Tuyến Kênh (Synthesis, Modulation & Channel Routing)**: `OscillatorNode`, `ConstantSourceNode`, `ChannelSplitterNode`, `ChannelMergerNode`, `DelayNode`.
3. **Quản Trị Bộ Nhớ Đệm Âm Thanh, Luồng Thời Gian Thực & Kết Xuất Ngoại Tuyến (Buffers, Streams & Offline Rendering)**: `AudioBuffer`, `AudioBufferSourceNode`, `MediaStreamAudioSourceNode`, `MediaStreamAudioDestinationNode`, `OfflineAudioContext`.

---

## 2. Bảng Ma Trận Quy Chuẩn Kỹ Thuật (15 Thành Phần Chuẩn Hóa)

| ID Quy Chuẩn | Giao Diện Chuẩn | Chuẩn Tham Chiếu | Cơ Chế Kỹ Thuật Cốt Lõi | Giá Trị Đối Với Super App Container | Quy Tắc Kiểm Duyệt Store (Store Review) |
|---|---|---|---|---|---|
| `STANDARDS-AUDIO-BIQUAD-FILTER` | `BiquadFilterNode` | W3C Web Audio API §5.12 | Bộ lọc IIR đệ quy bậc hai với các chế độ lowpass, highpass, bandpass, notch, peaking; điều khiển tần số và hệ số cộng hưởng Q bằng SIMD. | Tạo bộ cân bằng âm thanh (EQ), hiệu ứng âm thanh môi trường khi mở menu hoặc tạm dừng game. | Hệ số cộng hưởng Q phải giới hạn ≤ 30.0 để tránh hiện tượng tự kích âm hoặc méo tiếng gây hư hỏng loa di động. |
| `STANDARDS-AUDIO-ANALYSER-NODE` | `AnalyserNode` | W3C Web Audio API §5.11 | Biến đổi Fourier nhanh (FFT) thời gian thực trích xuất phổ tần số và biên độ miền thời gian mà không làm biến dạng tín hiệu gốc. | Hiển thị sóng âm thanh trực quan (visualizer), cảm biến hoạt động giọng nói (VAD) trong mini app live/chat. | Giới hạn `fftSize` tối đa 2048 trên thiết bị di động tầm trung; chỉ trích xuất dữ liệu trong `requestAnimationFrame` khi mini app hiển thị. |
| `STANDARDS-AUDIO-GAIN-NODE` | `GainNode` | W3C Web Audio API §5.10 | Điều chỉnh hệ số khuếch đại biên độ tín hiệu âm thanh với khả năng làm mờ tuyến tính/hàm mũ thông qua `AudioParam`. | Điều khiển âm lượng tổng, tắt tiếng tức thì và hỗ trợ ducking âm lượng khi có cuộc gọi điện thoại đến. | Bắt buộc mọi luồng âm thanh phát ra phải đi qua Master GainNode trước khi vào `destination` để container quản lý mute/pause. |
| `STANDARDS-AUDIO-STEREO-PANNER` | `StereoPannerNode` | W3C Web Audio API §5.19 | Định vị trường âm thanh nổi (stereo) với thuật toán cân bằng năng lượng không đổi (equal-power panning) chi phí tính toán thấp. | Giả lập không gian âm thanh nổi cho mini game 2D và hiệu ứng chuyển trang UI mà không tốn CPU như HRTF 3D. | Ưu tiên dùng `StereoPannerNode` thay vì `PannerNode` 3D đầy đủ trừ khi ứng dụng thực sự cần tọa độ không gian 3 trục XYZ. |
| `STANDARDS-AUDIO-WAVESHAPER-NODE` | `WaveShaperNode` | W3C Web Audio API §5.15 | Áp dụng đường cong truyền phi tuyến tính (Float32Array curve) để tạo độ bão hòa âm thanh và méo tiếng hài hòa. | Giả lập nhạc cụ, hiệu ứng tiếng nổ/biến dạng âm thanh trong game và phòng thu âm mini. | Cấm cấp phát mảng đường cong kích thước lớn (>65536 mẫu) liên tục trong render loop nhằm tránh GC giật khung hình. |
| `STANDARDS-AUDIO-OSCILLATOR-NODE` | `OscillatorNode` | W3C Web Audio API §5.7 | Tạo sóng dao động tuần hoàn (sine, square, sawtooth, triangle) theo lịch vi giây mà không cần tải file âm thanh tĩnh. | Tạo âm báo giao diện (click, ping, alert) hoàn toàn bằng thuật toán, giảm kích thước bundle mini app dưới 50KB. | Khuyến khích mini app dùng sóng tổng hợp thay vì đóng gói hàng loạt file WAV/MP3 âm lượng ngắn gây tốn băng thông. |
| `STANDARDS-AUDIO-CONSTANT-SOURCE-NODE` | `ConstantSourceNode` | W3C Web Audio API §5.9 | Phát tín hiệu vô hướng không đổi liên tục để điều chế nhiều tham số `AudioParam` trong đồ thị âm thanh đồng thời. | Tạo bus điều khiển vĩ mô (master control bus), liên kết thanh trượt UI với nhiều tham số lọc và âm lượng mà không cần JS loop. | Mini app điều phối nhiều node âm thanh đồng bộ phải dùng `ConstantSourceNode` thay vì gọi JS loop lặp lại. |
| `STANDARDS-AUDIO-CHANNEL-SPLITTER-NODE` | `ChannelSplitterNode` | W3C Web Audio API §5.20 | Phân tách luồng âm thanh đa kênh thành các kênh đơn (mono) riêng biệt để xử lý DSP độc lập. | Hỗ trợ tách kênh âm thanh đa ngôn ngữ, xử lý âm thanh vòm hoặc tách giọng hát karaoke trong mini app. | Cấm phân tách vượt quá số kênh thực tế của luồng đầu vào để tránh chiếm dụng tài nguyên bus âm thanh vô ích. |
| `STANDARDS-AUDIO-CHANNEL-MERGER-NODE` | `ChannelMergerNode` | W3C Web Audio API §5.21 | Tổng hợp nhiều luồng âm thanh đơn thành một luồng đa kênh (stereo, 5.1 surround) hoàn chỉnh. | Ghép các hiệu ứng và giọng nói riêng biệt vào một luồng xuất đa kênh trước khi phát hoặc ghi âm. | Thiết lập số kênh (`channelCount`) phải tương thích với năng lực phần cứng thực tế của thiết bị máy chủ. |
| `STANDARDS-AUDIO-DELAY-NODE` | `DelayNode` | W3C Web Audio API §5.16 | Đường trễ tín hiệu số có thể điều chỉnh độ trễ động, phục vụ hiệu ứng tiếng vang và căn chỉnh lệch pha âm thanh-hình ảnh. | Căn chỉnh đồng bộ hình-tiếng (lip-sync) trong phát trực tiếp và tạo không gian phản xạ âm thanh phòng kín. | `maxDelayTime` phải khai báo rõ ràng và không được vượt quá 5.0 giây trên thiết bị di động để chống tràn bộ nhớ đệm RAM. |
| `STANDARDS-AUDIO-BUFFER` | `AudioBuffer` | W3C Web Audio API §5.1 | Vùng nhớ lưu trữ dữ liệu âm thanh PCM tuyến tính 32-bit float đa kênh với cơ chế chia sẻ bộ nhớ không sao chép. | Định dạng lưu trữ âm thanh giải mã tiêu chuẩn cho hiệu ứng âm thanh game và âm báo phản hồi nhanh. | Áp dụng trần dung lượng cache AudioBuffer tối đa 50MB cho mỗi mini app; file âm thanh dài > 30s phải dùng streaming qua `<audio>` hoặc MSE. |
| `STANDARDS-AUDIO-BUFFER-SOURCE-NODE` | `AudioBufferSourceNode` | W3C Web Audio API §5.6 | Phát âm thanh từ vùng đệm `AudioBuffer` với độ trễ cực thấp, hỗ trợ vòng lặp (loop) và thay đổi tốc độ/cao độ phát động. | Phát âm thanh tương tác tức thì khi chạm nút, bắn súng trong game hoặc lặp lại tiếng động nền không ngắt quãng. | Đối tượng chỉ sử dụng một lần (single-use); mini app phải khởi tạo node mới cho mỗi lần phát thay vì tái sử dụng node đã dừng. |
| `STANDARDS-AUDIO-MEDIASTREAM-SOURCE` | `MediaStreamAudioSourceNode` | W3C Web Audio API §5.4 | Đưa luồng âm thanh trực tiếp từ microphone (`getUserMedia`) hoặc WebRTC vào đồ thị xử lý Web Audio. | Lọc tiếng ồn trực tiếp, thêm hiệu ứng giọng nói và đo mức âm lượng micro thời gian thực trong cuộc gọi mini app. | Bắt buộc phải xin cấp quyền microphone rõ ràng và tự động hủy giải phóng track âm thanh khi kết thúc phiên. |
| `STANDARDS-AUDIO-MEDIASTREAM-DESTINATION` | `MediaStreamAudioDestinationNode` | W3C Web Audio API §5.5 | Xuất toàn bộ tín hiệu sau xử lý từ đồ thị âm thanh thành một luồng `MediaStream` chuẩn để ghi âm hoặc truyền WebRTC. | Trộn nhạc nền, âm thanh game và giọng nói microphone thành một luồng âm thanh sạch duy nhất để livestream. | Xử lý điều tiết bộ đệm khi luồng đích bị nghẽn (backpressure); giải phóng tài nguyên ngay khi kết thúc ghi âm. |
| `STANDARDS-AUDIO-OFFLINE-CONTEXT` | `OfflineAudioContext` | W3C Web Audio API §5.2 | Kết xuất đồ thị âm thanh nhanh hơn thời gian thực trong tiến trình nền không cần đầu ra phần cứng loa. | Xuất file âm thanh, tạo nhạc chuông, nén file podcast hoặc dựng sẵn hiệu ứng âm thanh phức tạp tiết kiệm pin. | Các tác vụ xuất âm thanh hàng loạt bắt buộc phải dùng `OfflineAudioContext`; thời gian kết xuất bị giới hạn tối đa 30s để tránh treo CPU. |

---

## 3. Kiến Trúc Tích Hợp Web Audio Graph Trong Super App Web Container

```
+-----------------------------------------------------------------------------------+
|                        SUPER APP SECURE WEB CONTAINER                             |
|                                                                                   |
|  +--------------------+   +---------------------+   +--------------------------+  |
|  | Micro (Live Input) |   | Synthesis (Zero-IO) |   | Preloaded Audio Assets   |  |
|  | MediaStreamAudio-  |   | OscillatorNode      |   | AudioBuffer              |  |
|  | SourceNode         |   | ConstantSourceNode  |   | AudioBufferSourceNode    |  |
|  +---------+----------+   +----------+----------+   +------------+-------------+  |
|            |                         |                           |                |
|            \-------------------------+---------------------------/                |
|                                      | (Audio Signal Bus)                         |
|                                      v                                            |
|                  +---------------------------------------+                        |
|                  |      DSP Processing & Shaping         |                        |
|                  |  - BiquadFilterNode (EQ / Resonant)   |                        |
|                  |  - WaveShaperNode (Harmonic Sat)      |                        |
|                  |  - DelayNode (Acoustic Reflection)    |                        |
|                  +-------------------+-------------------+                        |
|                                      |                                            |
|                                      v                                            |
|                  +---------------------------------------+                        |
|                  |      Spatial & Channel Matrix         |                        |
|                  |  - StereoPannerNode (Equal-Power)     |                        |
|                  |  - ChannelSplitter / ChannelMerger    |                        |
|                  +-------------------+-------------------+                        |
|                                      |                                            |
|                                      v                                            |
|                  +---------------------------------------+                        |
|                  |      Telemetry & Master Protection    |                        |
|                  |  - AnalyserNode (Non-destructive FFT) |                        |
|                  |  - Master GainNode (Host Duck/Mute)   |                        |
|                  |  - DynamicsCompressor (Safety Limiter)|                        |
|                  +-------------------+-------------------+                        |
|                                      |                                            |
|                      /---------------+---------------\                            |
|                     |                                 |                           |
|                     v                                 v                           |
|        +-------------------------+       +-----------------------------+          |
|        | AudioDestinationNode    |       | MediaStreamAudioDestination |          |
|        | (Device Hardware Out)   |       | (WebRTC Egress / Recording) |          |
|        +-------------------------+       +-----------------------------+          |
+-----------------------------------------------------------------------------------+
```

---

## 4. Quy Trình Giám Sát Tài Nguyên & Kiểm Duyệt Tự Động (Automated Store Gate)
1. **Kiểm Soát Bộ Nhớ Đệm Âm Thanh (AudioBuffer Memory Bounds)**:
   - Các mini app nạp nhiều tài nguyên âm thanh giải mã trước phải tuân thủ trần dung lượng RAM là 50MB. Bộ phân tích tĩnh và động của Store Review sẽ đo lường tổng `byteLength` của tất cả các `AudioBuffer` đang hoạt động. Nếu vượt ngưỡng, mini app sẽ bị cảnh báo hoặc từ chối duyệt, yêu cầu chuyển các file âm thanh dài (>30s) sang định dạng phát luồng trực tiếp.
2. **Bảo Vệ Thính Giác & Thiết Bị Loa (Master Safety & Ducking Gate)**:
   - Tất cả các nút phát âm thanh của mini app bắt buộc phải được định tuyến qua một Master GainNode trước khi vào `AudioDestinationNode`. Super App Container có quyền can thiệp vào Master GainNode này để giảm 80% âm lượng (ducking) khi có thông báo hệ thống, hoặc tắt tiếng (mute) tức thì khi ứng dụng chuyển vào trạng thái nền (Page Visibility: hidden).
   - Bắt buộc kiểm tra tham số cộng hưởng `Q` của `BiquadFilterNode` và trần bộ đệm của `DelayNode` (`maxDelayTime <= 5.0s`).
3. **Kết Xuất Ngoại Tuyến Không Gây Treo Trình Duyệt (Offline Audio Worker Policy)**:
   - Các tác vụ tổng hợp hoặc chỉnh sửa file âm thanh bắt buộc phải sử dụng `OfflineAudioContext` và khuyến khích chạy bên trong Dedicated Web Worker để đảm bảo giao diện chính (UI Thread) duy trì ổn định 60/120fps.
