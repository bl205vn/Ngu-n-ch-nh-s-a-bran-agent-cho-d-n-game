# STATE: Cheezy Savoround

**Last Updated:** 2026-05-29

> **AI CONTEXT:** This document tracks the current state, active decisions, and known issues. Update it when major architectural decisions are made.

---

## 1. Recent Decisions (ADR)

- **Theme Change:** Chuyển đổi chủ đề game từ "Hoa" (Bloom Sort 3D) sang "Pizza" (Cheezy Savoround). Các thuật ngữ: Chậu -> Đĩa đựng pizza, Cánh hoa -> Miếng pizza.
- **Workflow Architecture:** Sử dụng chuẩn Brain-agent B+ cho hệ thống quản lý AI. Gộp `spec.md` và `tasks.md` thành một file duy nhất `tasks.md` nằm trong `.specs/features/`.
- **Phase 1 - Import 3D Assets (2026-05-29):** Đã import thành công bộ asset 3D vào `Assets/Models/` gồm: Floor_1, SinglePlate, DoublePlate (x2), Pizza_1~6, Table, Tile, Lobby. Kèm theo Materials (5 file), Textures (plate01~06 + AO/Normal/Roughness maps), Shader (Shader_Dia.shadergraph), và Font (SUPER GIGGLE SDF). Prefabs đã tạo: Floor_1, PizzaPlate, Pizza_1, Plane.
- **Unified Cell Prefab (2026-05-29):** GridManager chỉ dùng 1 `_cellPrefab` duy nhất (Floor_1), áp dụng Checkerboard bằng `MaterialPropertyBlock` thay vì 2 prefab riêng. Giúp giảm quản lý asset và giữ GPU Instancing hoạt động tốt.
- **Phase 1 - Slices & Object Pooling (2026-05-30):** Tích hợp `ObjectPoolManager`. Xóa bỏ enum `PizzaType` cũ gán cứng trên đĩa, thay bằng `int TypeIndex` gán trên từng miếng bánh. Áp dụng chuẩn Data-Driven bằng thuật toán Roulette Wheel Selection để config tỉ lệ xuất hiện của miếng bánh qua JSON.

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
