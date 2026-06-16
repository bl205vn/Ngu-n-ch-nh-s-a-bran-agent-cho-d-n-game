# FEATURE: Tuần 4 - Tối Ưu Hóa Render, Profiler & Đóng Gói (Nghiệm Thu)

**Status:** 🟡 IN PROGRESS
**Thời gian:** 16/06 - 22/06/2026 (Thực tế)

## 1. Feature Specification

Tuần 4 là chặng cuối cùng của dự án. Mục tiêu chính là đập tan các vấn đề về hiệu suất (đặc biệt là Render), dọn dẹp các bug còn tồn đọng, hoàn thiện tài liệu và Build ra sản phẩm cuối cùng chuẩn bị cho nghiệm thu.

Các mục tiêu quan trọng nhất:

- **Tối ưu Render (Draw Calls < 50):** Đây là tiêu chí trừ điểm rất nặng. Hiện tại Game đang báo 176 Batches.
- **Tách Canvas UI:** Tách riêng các HUD thay đổi liên tục khỏi Background để tránh Rebuild UI diện rộng.
- **Tắt Bóng (Shadows):** Tối ưu ánh sáng cho Mobile.
- **Profiler Hunting:** Quét lại toàn bộ chuỗi nổ liên hoàn để đảm bảo 100% Zero-GC.
- **Build & Đóng gói:** Xuất APK và Update README.

## 2. Technical Constraints

- **Mobile First:** Mọi cấu hình đồ hoạ phải hướng tới nền tảng Android. Tắt các hiệu ứng dư thừa như Realtime Shadow Cast trên các object nhỏ.
- **Zero-GC Compliance:** Mọi đoạn code tối ưu phải duy trì chuẩn Zero-GC đã thiết lập từ các tuần trước.

## 3. Execution Plan (Tasks)

---

### PHASE 1: Tối Ưu Draw Calls & Render (Ngày 1 - 2)

- [ ] **Task 1.1: Bật GPU Instancing (Thao tác trên Editor)**
  - Vật phẩm áp dụng: Material của đĩa bánh (`PizzaPlate`) và các miếng bánh (`PizzaSlice`).
  - **Cách làm:** Mở thư mục chứa Material, click chọn các Material của Đĩa và Miếng bánh -> Tích vào ô `Enable GPU Instancing` trong Inspector. (Cần shader chuẩn URP/Lit hoặc Unlit có hỗ trợ Instancing).

- [ ] **Task 1.2: Tối ưu Bóng râm (Shadows)**
  - Tắt đổ bóng cho các vật thể nhỏ: Chọn Prefab của `GridCell` và `PizzaSlice`, trong component `MeshRenderer`, đổi `Cast Shadows` thành `Off` và `Receive Shadows` thành `Off`. (Có thể giữ đổ bóng cho đĩa `PizzaPlate` hoặc mặt bàn nếu muốn hình ảnh có chiều sâu, nhưng tắt hết các thứ nhỏ nhặt).

- [ ] **Task 1.3: Static Batching cho Môi Trường**
  - Các vật thể như Mặt bàn lót, Background, và các thành phần tĩnh không di chuyển -> Chọn trên Hierarchy -> Đánh dấu tích vào ô `Static` (góc trên cùng bên phải Inspector). Unity sẽ gộp chúng thành 1 Batch.

- [ ] **Task 1.4: Tách Canvas UI Tĩnh và Động**
  - Kiểm tra Canvas của Scene: Tách riêng Canvas chứa Nút bấm tĩnh/Hình nền ra khỏi Canvas chứa Text thay đổi (Gold, Score, Thời gian đếm ngược, Combo). UI tĩnh sẽ không bị rebuild khi UI động thay đổi Text.

- [ ] **Task 1.5: Gộp Sprite Atlas**
  - Đóng gói toàn bộ hình ảnh UI 2D vào 1 (hoặc 2) `Sprite Atlas` để Unity có thể gộp tất cả ảnh 2D thành 1 Batch render duy nhất.

---

### PHASE 2: Quét Profiler & Clean Bugs (Ngày 3)

- [ ] **Task 2.1: Chạy Profiler và Deep Profiling**
  - Chơi game trong Editor với Unity Profiler (Ctrl + 7). Theo dõi tab `Memory -> GC Alloc`.
  - Đặt đĩa tạo chuỗi nổ lớn (Cascade) và check xem có dòng nào báo Alloc đỏ lên không.

- [ ] **Task 2.2: Dọn dẹp Null Reference và Warning**
  - Chơi thử với các Booster, ép lỗi để check Console. Sửa mọi Warning xuất hiện.

---

### PHASE 3: Đóng Gói (Build) và Báo Cáo (Ngày 4 - 5)

- [ ] **Task 3.1: Hoàn thiện File README.md**
  - Nhúng hình ảnh sơ đồ Game Flow (System Flow Diagram) vào README.
  - Cập nhật hướng dẫn cài đặt và các tính năng chính.

- [ ] **Task 3.2: Xuất APK và Quay Video**
  - Setup `Player Settings` (Icons, Tên App, Package Name).
  - Xuất APK chơi thử trên máy thật.
  - Quay video trải nghiệm Gameplay thực tế (chứng minh các logic Nổ dây chuyền, Booster, Store, Auto-save hoạt động mượt mà).

---

### PHASE 4: Nghiệm Thu (Ngày 6 - 7)

- [ ] **Task 4.1: Kiểm tra chéo với Tiêu Chí Của Công Ty**
  - Bám sát file `Yêu Cầu Kỹ Thuật Dự Án 1 (Bloom Sort 3D).md` và `Lộ Trình & Tiêu Chí Nghiệm Thu Dự Án 1.md`. Này là chỉ cần hoàn thiện readme.md trên github và quay video là xong.
