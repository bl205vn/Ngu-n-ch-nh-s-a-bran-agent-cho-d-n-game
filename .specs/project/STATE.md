# STATE: Cheezy Savoround
**Last Updated:** 2026-05-29

> **AI CONTEXT:** This document tracks the current state, active decisions, and known issues. Update it when major architectural decisions are made.

---

## 1. Recent Decisions (ADR)
- **Theme Change:** Chuyển đổi chủ đề game từ "Hoa" (Bloom Sort 3D) sang "Pizza" (Cheezy Savoround). Các thuật ngữ: Chậu -> Đĩa đựng pizza, Cánh hoa -> Miếng pizza.
- **Workflow Architecture:** Sử dụng chuẩn Brain-agent B+ cho hệ thống quản lý AI. Gộp `spec.md` và `tasks.md` thành một file duy nhất `tasks.md` nằm trong `.specs/features/`.
- **Phase 1 - Import 3D Assets (2026-05-29):** Đã import thành công bộ asset 3D vào `Assets/Models/` gồm: Floor_1, SinglePlate, DoublePlate (x2), Pizza_1~6, Table, Tile, Lobby. Kèm theo Materials (5 file), Textures (plate01~06 + AO/Normal/Roughness maps), Shader (Shader_Dia.shadergraph), và Font (SUPER GIGGLE SDF). Prefabs đã tạo: Floor_1, PizzaPlate, Pizza_1, Plane.
- **Unified Cell Prefab (2026-05-29):** GridManager chỉ dùng 1 `_cellPrefab` duy nhất (Floor_1), áp dụng Checkerboard bằng `MaterialPropertyBlock` thay vì 2 prefab riêng. Giúp giảm quản lý asset và giữ GPU Instancing hoạt động tốt.

## 2. Blockers
- **Scripts/UI/ và Scripts/Utils/ đang trống:** Chưa có script nào trong 2 thư mục này. Cần implement khi bắt đầu làm UI (HUD, Score) và tiện ích chung (ObjectPool, Extensions).
- **TrayManager thiếu Auto-Scale:** Không có hàm `FitPlateToSlot()` tương đương `FitPrefabToCell()` của GridManager. Nếu thay đổi kích thước Prefab đĩa pizza, cần bổ sung logic auto-scale cho TrayManager.

## 3. Lessons Learned
- **Trả nợ kỹ thuật Tuần 1 (2026-05-29):** Phát hiện và ĐÃ FIX thành công 2 Critical (GC trong GetComponent runtime + new alloc trong gameplay loop), 4 Warning (magic numbers, public field, RaycastAll đã được update sang NonAlloc), 3 Suggestion (thư mục thiếu, naming, scene đã chuẩn Architecture). Hệ thống đã đạt "Zero GC Alloc".
- **Zero-GC Raycast Pattern:** Việc kết hợp `Physics.RaycastNonAlloc` với mảng tĩnh đệm và hàm `Array.Sort` qua một `struct IComparer<RaycastHit>` là mẫu chuẩn tối ưu bắt buộc tái sử dụng cho các tính năng raycast trong tương lai.
- **Data-Driven approach:** JSON config cho level hoạt động tốt, sẽ mở rộng thêm ma trận đĩa bánh cho Tuần 2.
- **MaterialPropertyBlock cho Checkerboard (2026-05-29):** Dùng MaterialPropertyBlock thay đổi `_Color`/`_BaseColor` trên mỗi Renderer thay vì tạo Material instance. Pattern này đã chứng minh giữ được GPU Instancing và Zero GC. Áp dụng lại cho mọi hệ thống cần thay đổi màu runtime.
- **Documentation Drift (2026-05-29):** ARCHITECTURE.md ghi nhận hàm `FitPlateToSlot()` ở TrayManager nhưng hàm này KHÔNG TỒN TẠI trong code thực tế. Bài học: luôn audit ARCHITECTURE.md sau mỗi sprint/phase để tránh phantom references.
