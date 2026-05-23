## **TÀI LIỆU YÊU CẦU KỸ THUẬT CHI TIẾT (PROJECT SPECIFICATIONS)**

## **DỰ ÁN 1: CORE GAMEPLAY BLOOM SORT 3D (TUẦN 1 - TUẦN 4)**

**Vị trí thực tập:** Unity Developer (Solo Project)

**Mục tiêu tài liệu:** Cung cấp đầy đủ các thông số kỹ thuật, quy chuẩn lập trình, yêu cầu trải nghiệm (UX) và tối ưu hóa hiệu suất thiết bị cho dự án **Bloom Sort 3D** .

## **PHẦN I: MÔ TẢ GAMEPLAY (GAME DESIGN DOCUMENT SƠ LƯỢC)**

## **1. Trạng thái bàn chơi (Board State)**

- **Lưới chơi (Grid):** Một hệ thống lưới kích thước 3x3 hoặc 4x4 (có thể cấu hình động qua JSON) nằm trong không gian 3D tùy vào mỗi level.

- **Khay chứa (Hold Slots):** Gồm 3 vị trí trống phía trước bàn chơi dùng để chứa các chậu hoa dự phòng được sinh ra ngẫu nhiên.

- **Chậu hoa 3D:** Mỗi chậu chứa tối đa 6 khe xếp cánh hoa. Các cánh hoa có thể có nhiều màu sắc khác nhau xếp chồng hoặc xen kẽ trên cùng một chậu.

## **2. Vòng lặp Core Gameplay**

- **Tương tác:** Người chơi chạm/nhấp vào chậu hoa ở Hold Slots, kéo (Drag) và thả (Drop) vào một ô lưới trống trên Grid.

- **Logic Merge & Bloom:**
  - Khi chậu hoa được đặt xuống, hệ thống quét các chậu lân cận theo 4 hướng (Trái, Phải, Trên, Dưới).

  - Nếu các cánh hoa ngoài cùng của các chậu cạnh nhau có **cùng màu** , tiến hành chuyển 6

  - cánh hoa (bay tự do) từ chậu này sang chậu kia cho đến khi một chậu gom đủ 6 cánh cùng màu ‘ Chậu đó "Nở hoa" (Bloom), kích hoạt hiệu ứng nổ và biến mất khỏi lưới.

  - Khi một chậu biến mất, ô đó trở thành ô trống. Việc này có thể kích hoạt chuỗi kiểm tra tiếp theo cho các chậu lân cận tạo thành hiệu ứng dây chuyền ( **Combo Cascade** ).

- **Game Over:** Xảy ra khi cả Grid và Hold Slots đều đầy chậu hoa và không còn bất kỳ lượt di chuyển hay merge hợp lệ nào có thể thực hiện được nữa.

## **3. Tính năng phụ bổ trợ (Meta Features)**

- **Shop System (Skins):** Người chơi dùng vàng tích lũy để mua các loại chậu hoa mới. Khi trang bị, mô hình 3D (Mesh) của chậu trên Grid sẽ tự động thay đổi theo skin đã chọn.

- **Daily Rewards:** Điểm danh nhận vàng mỗi 24 giờ. Hệ thống cần bảo mật thời gian để chống việc người chơi đổi giờ trên thiết bị để hack vàng.

- **Achievement System:** Hệ thống nhiệm vụ trọn đời (Nở 100 bông hoa, Đạt combo x5, Tích lũy 1000 vàng...) hoạt động theo mô hình hướng sự kiện (Event-driven).

## **PHẦN II: YÊU CẦU SƠ ĐỒ HOẠT ĐỘNG HỆ THỐNG (SYSTEM FLOW DIAGRAM)**

Trước khi bắt tay vào viết code thực tế, sinh viên bắt buộc phải phác thảo luồng vận hành của hệ thống game thông qua **01 Sơ đồ khối hoạt động (System Flow Diagram)** duy nhất:

## **1. Yêu cầu nội dung sơ đồ**

Sơ đồ cần mô tả một cách trực quan cách các khối chức năng giao tiếp với nhau khi có sự kiện diễn ra. Sinh viên phải thể hiện được luồng xử lý chính sau đây:

- **Khối đầu vào (Input):** Người chơi chạm/nhấc/thả chậu hoa > Chuyển thông tin tọa độ xuống bộ điều khiển.

- **Khối xử lý logic (Logic Core):** Kiểm tra vị trí thả > Cập nhật ma trận Grid > Quét 4 hướng tìm chậu trùng màu > Tính toán trừ/nhập cánh hoa và nổ hoa (Bloom).

- **Khối truyền tin (Event System):** Khi có sự kiện xảy ra (Đặt hoa thành công, Hoa nở, Đạt combo), khối này sẽ phát tín hiệu đi để các khối khác tự xử lý.

- **Khối đầu ra hiển thị (Visual/Output):** Hệ thống UI (Điểm số, Vàng), Hệ thống âm thanh (Sound FX), Hệ thống hiệu ứng (VFX Particle) nhận tín hiệu từ Khối truyền tin để hiển thị trực quan cho người chơi.

## **2. Quy chuẩn nộp sơ đồ thiết kế**

- **Công cụ vẽ tự chọn:** Sinh viên được sử dụng Draw.io, Figma, Miro, hoặc vẽ tay thô sơ trên giấy nhưng phải rõ ràng, sạch đẹp, dễ đọc.

- **Quy chuẩn lưu trữ trên Git:**
  - Tạo thư mục /docs/ trong thư mục gốc của dự án trên Git.

  - Lưu ảnh sơ đồ (.png hoặc .jpg) vào thư mục trên.

  - Nhúng trực tiếp ảnh này vào tệp README.md tại thư mục gốc của Git bằng cú pháp: ![System Flow](docs/ten_anh_so_do.png).

## **PHẦN III: YÊU CẦU KỸ THUẬT (TECHNICAL REQUIREMENTS)**

## **1. Kiến trúc Code & Tư duy Hệ thống (Architecture)**

- **Coding Conventions:** Viết code sạch, đặt tên Class dạng PascalCase, biến/hàm dạng camelCase, biến private bắt đầu bằng dấu gạch dưới \_. Code phải có chú thích giải thích rõ ràng các thuật toán phức tạp.

- **Data-Driven Design:** Không viết chết (hardcode) các thông số. Toàn bộ thông tin cấu hình màn chơi, giá tiền vật phẩm shop, thuộc tính các loại hoa phải được đọc từ file cấu hình ngoài (JSON hoặc ScriptableObject).

- **Finite State Machine (FSM):** Quản lý luồng hoạt động của game chặt chẽ qua các trạng thái rõ ràng (ví dụ: PlayingState, AnimatingState, CheckingComboState, GameOverState). Nghiêm cấm sử dụng các cờ hiệu boolean (bool) lồng chéo nhau vô tội vạ.

- **Observer Pattern (Event System):** Sử dụng hệ thống Event để giảm thiểu sự phụ thuộc trực tiếp (Tight Coupling) giữa các Class.

## **2. Tương tác 3D & Trải nghiệm Người chơi (UX / Game Feel)**

- **Raycast & Snapping:** Sử dụng Physics.Raycast để phát hiện điểm chạm trên bàn chơi 3D. Khi kéo chậu hoa đến gần một ô lưới trống hợp lệ, chậu hoa phải tự động "hút" (Snap) vào tâm của ô đó để định hướng cho người chơi trước khi thả tay (hoặc có thể sử dụng cách khác tương đương).

- **Preview Ghost Mesh:** Hiển thị một mô hình chậu hoa bán trong suốt (Ghost Material) tại ô lưới dự kiến khi người chơi đang rê chậu hoa qua ô đó.

- **Tweening Animation:**
  - Khi nhấc chậu: Có hiệu ứng phóng to nhẹ và nâng cao lên khỏi mặt bàn.

  - Khi đặt xuống: Có hiệu ứng đàn hồi nhún nhảy (Squash and Stretch) tạo cảm giác có trọng lực.

  - Quỹ đạo cánh hoa: Cánh hoa di chuyển giữa các chậu theo đường cong Bezier hoặc nội suy Vector 3D (Slerp/Lerp), không được biến mất và xuất hiện đột ngột.

  - Khi đặt sai ô: Chậu tự động di chuyển ngược về khay chứa cũ (Tweenback) kèm hiệu ứng rung lắc (Shake) báo lỗi trực quan.

- **Combo Audio Pitch Shift:** Âm thanh nổ hoa/bay cánh hoa phải tăng tiến dần cao độ (Pitch) theo nốt nhạc khi người chơi đạt chuỗi combo liên hoàn để tăng độ kích thích thính giác.

## **3. Tối ưu hóa hiệu năng (Performance Optimization)**

- **Zero GC Alloc in Update:** Cấm tuyệt đối việc sử dụng GameObject.Find, GetComponent, hoặc tạo mới vùng nhớ (new List, new Dictionary, cộng chuỗi string) trong các hàm chạy liên tục mỗi khung hình (Update).

- **Object Pooling:** Xây dựng hệ thống tái sử dụng (Pool) cho các tài nguyên sinh ra liên tục: các cánh hoa, hiệu ứng Particle nổ hoa, Text hiển thị điểm số bay (Floating Text).

- **Draw Call / Batches Control:**
  - Gom toàn bộ ảnh UI phẳng vào **Sprite Atlas** .

  - Sử dụng **Static Batching** cho bàn chơi cố định và **GPU Instancing / Dynamic Batching** cho các mô hình cánh hoa/chậu hoa giống nhau xuất hiện liên tục.

  - Đảm bảo tổng số lượng Batches (Draw Calls) hiển thị trên cửa sổ Profiler khi chơi thực 50

  - tế dưới **Batches** .

- **UI Canvas Optimization:** Tách UI thành Canvas tĩnh và Canvas động để tránh Engine phải render lại (Rebuild) toàn bộ UI mỗi khi điểm số thay đổi.

## **PHẦN IV: BẢNG CHECKLIST CHẤM ĐIỂM CHI TIẾT (TỔNG: 100 ĐIỂM)**

| **Hạng mục đánh**<br>**giá**                         | **Chỉ số kiểm tra chi**<br>**tiết**                                                                                                                                                                                                                                                                                                                                                                                | **Điểm tối đa** | **Điểm đạt được** |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------- | ----------------- |
| **1. Thiết kế Hệ**<br>**thống & Sơ đồ**<br>**(15đ)** | - File README.md<br>hiển thị đầy đủ, trực<br>quan**01 Sơ đồ**<br>**luồng hoạt động**<br>**hệ thống**dưới<br>dạng ảnh nhúng từ<br>thư mục /docs/ (5đ).<br>- Sơ đồ phân tích<br>đúng luồng đi của<br>dữ liệu từ Input<br>Core Logic<br>Output (UI, Sound,<br>Effects) một cách<br>hợp lý (5đ).<br>- Sơ đồ khớp chính<br>xác với cấu trúc<br>thiết kế logic code<br>trong thực tế ở tuần<br>cuối cùng (5đ).<br>‘<br>5 | **15**          |                   |
| **2. Core Gameplay**<br>**& Thuật toán (25đ)**       | - Khởi tạo Grid 3D<br>động từ cấu hình<br>JSON/ScriptableOb                                                                                                                                                                                                                                                                                                                                                        | **25**          |                   |

|     | ject (10đ).<br>- Thuật toán kiểm<br>tra lân cận 4 hướng<br>chính xác (10đ).<br>- Xử lý đúng logic<br>Merge, Bloom và<br>chuỗi combo dây<br>chuyền (5đ).<br>**3. Animation & UX**<br>**Game Feel (20đ)**<br>- Raycast mượt, có<br>Preview Ghost<br>Mesh và cơ chế<br>Snap tự động (5đ).<br>- Cánh hoa di<br>chuyển mượt theo<br>đường cong Bezier<br>(10đ).<br>- Có hiệu ứng<br>Squash/Stretch,<br>Shake báo lỗi và<br>Pitch Shif của âm<br>thanh (5đ).<br>**20**<br>**4. Tối ưu hóa hiệu**<br>**năng (15đ)**<br>- Không tạo rác<br>(Zero GC Alloc)<br>trong Update (5đ).<br>- Áp dụng Object<br>Pooling cho tất cả<br>các phần tử sinh ra<br>liên tục (5đ).<br>- Draw<br>Calls/Batches dưới<br>50 nhờ Atlas và<br>Batching 3D (5đ).<br>**15**<br>**5. Kiến trúc Code**<br>- Áp dụng đúng<br>**15** |
| --- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

| **& Độ sạch (15đ)**               | Coding Convention<br>và chia nhánh Git<br>chuẩn (5đ).<br>- Áp dụng đúng<br>State Patern cho<br>trạng thái game và<br>Event System<br>(Observer) để giảm<br>liên kết cứng (5đ).<br>- Đọc toàn bộ chỉ<br>số từ cấu hình<br>JSON/ScriptableOb<br>ject (5đ). |         |     |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- | --- |
| **6. Meta Features**<br>**(10đ)** | - Hệ thống Shop<br>đổi Skin đổi được<br>Mesh 3D và lưu trữ<br>dữ liệu chuẩn JSON<br>(4đ).<br>- Daily Rewards<br>chống hack thời<br>gian bằng UTC<br>Timestamp (3đ).<br>- Achievement tự<br>động hoạt động<br>theo cơ chế<br>Event-driven (3đ).           | **10**  |     |
| **TỔNG CỘNG**                     | **Đánh giá toàn diện**<br>**chất lượng kỹ**<br>**thuật của sản**<br>**phẩm.**                                                                                                                                                                            | **100** |     |

## **Thang Xếp Loại Đánh Giá Dự Án 1:**

- **Từ 85 - 100 điểm (Xuất sắc):** Thiết kế luồng hệ thống thông minh, code sạch, tối ưu hiệu suất xuất sắc. Game chạy mượt không một lỗi nhỏ.

- **Từ 70 - 84 điểm (Khá):** Sơ đồ đầy đủ và dễ hiểu, game chạy tốt, đầy đủ tính năng, giao

diện mượt. Code còn một vài chỗ nhỏ chưa tối ưu hoàn hảo nhưng đạt chất lượng tốt.

- **Từ 50 - 69 điểm (Trung bình):** Sơ đồ vẽ sơ sài hoặc sai logic hoạt động thực tế, code lộn xộn, lạm dụng hàm Update hoặc không tối ưu hiệu năng khiến game giật lag khi nổ combo lớn.

- **Dưới 50 điểm (Không Đạt):** Sơ đồ không có hoặc sai lệch hoàn toàn luồng game, game lỗi crash nghiêm trọng, không áp dụng các yêu cầu kỹ thuật bắt buộc, commit Git sơ sài hoặc copy code.
