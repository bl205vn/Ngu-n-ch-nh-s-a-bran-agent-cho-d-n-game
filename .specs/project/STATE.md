# STATE: Cheezy Savoround

**Last Updated:** 2026-05-29

> **AI CONTEXT:** This document tracks the current state, active decisions, and known issues. Update it when major architectural decisions are made.

---

## 1. Recent Decisions (ADR)

- **[2026-06-01] Minority Slice Swapping (Tráo đổi chống kẹt):** Khi đĩa giữa báo đầy (`IsFull()`) nhưng không tinh khiết (`!IsFullAndPure()`), hệ thống sẽ gọi `GridManager.TrySwapMinoritySlice()`. Thuật toán sẽ tìm miếng bánh thiểu số (ít số lượng nhất) ở đĩa giữa để đẩy ra lân cận, và hút về miếng bánh đa số từ lân cận. Cơ chế Swap 1-1 qua `BezierTween` giải quyết hoàn toàn tình trạng Deadlock (kẹt đĩa) kinh điển của dòng game Bloom Sort, giữ vững Zero GC.
- **[2026-06-01] Batch Refill — Sinh đĩa theo batch (3 đĩa/lượt):** Thay đổi flow sinh đĩa pizza từ "sinh từng cái" sang "sinh cả batch khi khay trống hết" (giống Candy Crush / Triple Tile). TrayManager theo dõi `_slotPlates[]`, lắng nghe `InputManager.OnPlatePlaced` để đánh dấu slot trống. Khi cả 3 slot trống + FSM về `PlayingState` (merge/bloom xong hẳn) → `RefillTray()` sinh 3 đĩa mới cùng lúc. GameStateManager bổ sung `OnStateChanged` event (Observer Pattern) để TrayManager lắng nghe.
- **[2026-05-31] Vẽ lại Sơ đồ Logic Game (Game Flow Diagram):** Cập nhật `ARCHITECTURE.md` với sơ đồ mới từ `docs/Ve_so_do_game_banh_pizza.png`. Thay đổi so với phiên bản cũ: (1) Bổ sung event **"Đặt đĩa thất bại"** — Snap sai cũng phát event để chạy hoạt ảnh phản hồi lỗi; (2) Đổi tên node "Vị trí thả" → **"Vị trí thả Snap"** chính xác hơn; (3) Node J từ hình chữ nhật thành **diamond** (điều kiện rõ ràng); (4) Bổ sung loop **O → F** (đĩa nổ → reset Grid) cho chain combo; (5) Tách **"Hoạt ảnh"** và **"VFX"** thành 2 output độc lập; (6) Mapping event → output được làm rõ chính xác per-event (Đặt đĩa thành công → UI sinh đĩa mới).
- **Phase 3 - Parallel Pull & Drop Animation (2026-05-31):** Nâng cấp thuật toán Merge để kéo bánh đồng thời từ 4 hướng (1 miếng/hướng/lượt) giống cơ chế Bloom Sort. Sửa lỗi teleport bằng cách gọi `SetParent(transform, true)` để giữ nguyên World Position trước khi bay Bezier. Thêm Coroutine `AnimateToCell` (Quadratic Bezier 3D) vào `PizzaPlate.cs` để tạo hiệu ứng vòng cung nhô cao 0.8 unit khi snap đĩa vào ô lưới. Mọi thứ vận hành mượt mà với FSM `CheckingComboState` và `AnimatingState`.


## 2. Blockers

- **Scripts/UI/ đang trống:** Chưa có script UI nào được tạo (HUD, Score). Tiến độ hiện tại cần chuyển sang Phase 2 (Tweening, Merge Logic) rồi mới ghép UI.

## 3. Lessons Learned

- **Auto-Scale cho TrayManager (2026-05-29):** Đã bổ sung hàm `FitPlateToSlot()` trong `TrayManager` để tự động scale prefab đĩa pizza vừa vặn với `_slotSpacing` của khay (giải quyết blocker kích thước).
- **Trả nợ kỹ thuật Tuần 1 (2026-05-29):** Phát hiện và ĐÃ FIX thành công 2 Critical (GC trong GetComponent runtime + new alloc trong gameplay loop), 4 Warning (magic numbers, public field, RaycastAll đã được update sang NonAlloc), 3 Suggestion (thư mục thiếu, naming, scene đã chuẩn Architecture). Hệ thống đã đạt "Zero GC Alloc".
- **Zero-GC Raycast Pattern:** Việc kết hợp `Physics.RaycastNonAlloc` với mảng tĩnh đệm và hàm `Array.Sort` qua một `struct IComparer<RaycastHit>` là mẫu chuẩn tối ưu bắt buộc tái sử dụng cho các tính năng raycast trong tương lai.
- **Data-Driven approach:** JSON config cho level hoạt động tốt, sẽ mở rộng thêm ma trận đĩa bánh cho Tuần 2.
- **MaterialPropertyBlock cho Checkerboard (2026-05-29):** Dùng MaterialPropertyBlock thay đổi `_Color`/`_BaseColor` trên mỗi Renderer thay vì tạo Material instance. Pattern này đã chứng minh giữ được GPU Instancing và Zero GC. Áp dụng lại cho mọi hệ thống cần thay đổi màu runtime.
- **SetParent Scale Compensation (2026-05-30):** Khi sinh Prefab con ghép vào Parent đã bị thay đổi Scale (ví dụ đĩa bị ép Scale), lệnh `SetParent` mặc định của Unity tự động thay đổi `localScale` để bù trừ. Cần dùng `SetParent(parent, false)` và tái lập `localScale = Vector3.one` để khắc phục lỗi con phóng to/nhỏ bất thường.
- **Data-Driven Randomization (2026-05-30):** Tránh Random mù quáng. Dùng mảng `sliceCountProbabilities` lưu tỉ lệ phần trăm và thuật toán Roulette Wheel Selection để điều khiển tỉ lệ sinh miếng bánh trực tiếp từ file JSON Level Data.
