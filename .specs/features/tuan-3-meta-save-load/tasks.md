# FEATURE: Tuần 3 - Tích Hợp Meta Features & Save/Load Dữ Liệu

**Status:** 🟡 IN PROGRESS
**Thời gian:** 12/06 - 19/06/2026 (Thực tế)

## 1. Feature Specification

Tuần 3 tập trung vào việc xây dựng các hệ thống Meta Game xung quanh Core Loop đã hoàn thiện ở Tuần 2:

- **Save/Load System:** Lưu trữ dữ liệu người dùng (Vàng, Skin, Tiến độ) bằng JSON.
- **UI Shop:** Cửa hàng mua skin bằng vàng (đọc từ cấu hình JSON) và thay đổi ngoại hình đĩa/bánh.
- **Daily Rewards:** Điểm danh nhận thưởng hàng ngày, chống hack giờ bằng UTC timestamp.
- **Achievement System:** Hệ thống thành tựu hướng sự kiện (Event-Driven), độc lập hoàn toàn với logic Gameplay.

## 2. Technical Constraints

- **Save/Load:** Sử dụng `System.IO.File` và `JsonUtility` (hoặc NewtonSoft.Json nếu có) để ghi/đọc JSON tại `Application.persistentDataPath`.
- **Bảo mật Daily Reward:** Phải dùng thời gian UTC (`DateTime.UtcNow`) lưu xuống file, tuyệt đối không dùng giờ Local của máy tính người chơi.
- **Event-Driven Architecture (Achievement):** Các script xử lý Thành Tựu KHÔNG ĐƯỢC phép reference trực tiếp đến `GridManager` hay `PizzaPlate`. Bắt buộc phải lắng nghe qua `C# event` (Action).
- **Decoupling (Shop):** UI Shop không được can thiệp logic gameplay. Khi đổi skin, hệ thống lưu ID Skin vào SaveData, `PizzaPlate` tự động đọc ID đó khi khởi tạo.

## 3. Execution Plan (Tasks)

---

### PHASE 1: Hệ thống Save/Load JSON (Ngày 1 - 2)

- [x] **Task 1.1: Thiết kế Data Models**
  - Tạo class `PlayerData.cs`: chứa `int Gold`, `List<string> UnlockedSkins`, `string CurrentSkinId`, `long LastDailyRewardTime`, `Dictionary<string, int> AchievementProgress`.
  - Tạo class `GameSettings.cs` (nếu cần cho setting âm thanh/nhạc).
  - **File:** `Scripts/Data/PlayerData.cs`

- [x] **Task 1.2: Triển khai SaveLoadManager**
  - Tạo Singleton `SaveLoadManager` xử lý việc mã hóa (tùy chọn) và ghi file `playerdata.json` xuống đĩa.
  - Viết hàm `Save()`, `Load()`, `ResetData()`.
  - Gọi `Load()` ở Awake của GameManager hoặc Bootstrap scene.
  - **Hoàn thiện (14/06/2026):**
    - Bổ sung cơ chế lưu State Bàn cờ (Grid & Tray) vào `LevelProgressData`. Lưu cả điểm số tiến trình (LevelProgressUI).
    - Tích hợp vòng lặp Game Economy (Cộng vàng khi nổ đĩa thông qua event `OnPlateExploded`).
    - Tích hợp hook tự động save an toàn cho Mobile qua `OnApplicationPause` và `OnApplicationQuit` trên `GameStateManager`.
  - **Verification:** Chạy game, đặt đĩa, thay đổi vàng, vuốt tắt màn hình/tắt game mở lại -> Bàn cờ và số vàng được giữ nguyên.
  - **File:** `Scripts/Core/SaveLoadManager.cs`

---

### PHASE 2: Cấu hình và UI Shop (Ngày 3)

- [x] **Task 2.1: Tạo ScriptableObject hoặc JSON Cấu hình Shop**
  - Định nghĩa danh sách vật phẩm: ID, Tên, Giá vàng, Prefab Mesh (hoặc Material).
  - **File:** `Scripts/Data/ShopConfig.cs` (ScriptableObject)

- [x] **Task 2.2: Thiết kế Visual/Layout UI (Lobby, Shop, HUD Ingame, Game Over)**
  - Chia 2 tầng Canvas (Camera cho nền lót sàn, Overlay cho Nút bấm/HUD) khắc phục lỗi đè 3D.
  - Setup UI Shop dạng Carousel (1 Khung Item dùng chung, đổi mảng dữ liệu qua Tab). Tối ưu Tab không dùng ảnh riêng mà dùng Color (Gray/White).
  - Setup Game Over Overlay bật `Raycast Target` làm Dimmer chặn tương tác 3D.

- [x] **Task 2.3: Lập trình Logic UI Shop & UIManager**
  - Code `ShopManager.cs`: Đọc `ShopConfig`, chuyển dữ liệu Carousel, tự sinh các Dots (Pagination).
  - Logic mua: Check vàng -> Trừ vàng -> Lưu `UnlockedSkins`.
  - **Hoàn thiện (14/06/2026):** Tích hợp Data-Driven triệt để qua ScriptableObject, hỗ trợ lưu Boosters tự động cấp phát bộ nhớ, và refactor Zero-GC toàn bộ text.
  - **File:** `Scripts/UI/ShopManager.cs`

- [x] **Task 2.4: Tích hợp Skin vào Gameplay**
  - Khi `TrayManager` hoặc `PizzaPlate` sinh đĩa, đọc `CurrentSkinId` từ `PlayerData` để áp dụng đúng Mesh/Material mới.
  - **Hoàn thiện (14/06/2026):** Triển khai Dynamic Skin Sync - tự động thay đổi ngoại hình toàn bộ đĩa trên Grid và Tray theo thời gian thực khi người chơi mua/trang bị trong Shop (sử dụng `GameEvents.OnSkinChanged`).
  - **File:** `Scripts/Gameplay/PizzaPlate.cs` (cập nhật)

---

### PHASE 3: Daily Rewards (Ngày 4 - 5)

- [x] **Task 3.1: Logic tính toán thời gian UTC & Chống Gian Lận (Anti-Cheat)**
  - Tạo class `DailyRewardManager`.
  - Viết hàm check: So sánh `DateTime.UtcNow` với `LastDailyRewardTime` (được lưu dưới dạng binary hoặc tick).
  - Nếu khoảng cách qua ngày mới (hoặc > 24h) -> Kích hoạt nhận quà.
  - **Hoàn thiện (15/06/2026):** Triển khai kiến trúc Anti-Cheat mạnh mẽ bằng cách kết hợp Server-Time (Ping Header Google) và Offline Offset Validation (tính độ trễ qua `TimeValidationData`). Phát hiện và phạt reset chuỗi khi cố tình đổi giờ điện thoại.

- [x] **Task 3.2: Giao diện nhận thưởng**
  - Hiển thị UI nút "Nhận Quà", bấm vào cộng vàng -> Gọi `SaveLoadManager.Save()`.
  - **Hoàn thiện (15/06/2026):** Xây dựng Data-Driven qua `DailyRewardConfig`, hỗ trợ mixed rewards (Chest) và cập nhật UI Zero-GC bằng TextMeshPro. Đã sửa lỗi hiển thị ngày kế tiếp.
  - **Verification:** Đã test tắt mạng, đổi giờ Local đi lùi -> Console văng log cảnh báo và không cấp quyền nhận quà.

---

### PHASE 4: Achievement System (Ngày 6 - 7)

- [x] **Task 4.1: Khai báo hệ thống Event chung**
  - Tạo class tĩnh `GameEvents.cs` chứa các Action: `OnPizzaMerged(int count)`, `OnPlateExploded(int totalSlices)`, `OnLevelCompleted()`.
  - Sửa `GridManager` để phát event này mỗi khi nổ đĩa/merge.

- [x] **Task 4.2: AchievementManager lắng nghe Event**
  - `AchievementManager` đăng ký lắng nghe các event trên trong `OnEnable/OnDisable`.
  - Cập nhật tiến độ `AchievementProgress` tương ứng -> Khi đạt mốc -> Mở khóa -> Phát popup -> Lưu Save file.
  - **Verification:** Thấy thông báo Popup "Thành tựu" hiện lên khi đủ điều kiện mà không dính líu code vào `GridManager`.

---

### PHASE 5: Tối Ưu Hiệu Năng & Fix Bug Logic (Dựa theo Performance Optimization Plan)

- [x] **Task 5.1: Sửa lỗi Logic Gameplay (Kích hoạt Thành Tựu)**
  - **Hoàn thiện (16/06/2026):**
    - Gắn `GameEvents.TriggerLevelCompleted()` vào `LevelProgressUI.LevelUp()` — gọi TRƯỚC `LoadNextLevel()` để achievement xử lý xong trước khi level mới bắt đầu.
    - Bổ sung `GameEvents.TriggerBoosterUsed()` vào `BoostButton.OnClickButton()` khi người chơi thực sự sử dụng boost (trừ số lượng, phát event, cập nhật UI).
    - Sửa lỗi `UnlockCakes` đếm sai: đổi `UnlockedSkins.Count` → `Count - 1` (trừ skin mặc định `plate_01`).
    - Xóa dead code `OnPizzaMerged` / `TriggerPizzaMerged()` khỏi `GameEvents.cs` (không ai subscribe/invoke).

- [x] **Task 5.2: Khử FindObjectsByType & Boxing (Hot-Path GC)**
  - **Hoàn thiện (16/06/2026):**
    - `GoldDisplay`: Tạo `private static List<GoldDisplay> _activeDisplays` thay vì quét cả Scene mỗi khi có thay đổi vàng.
    - `LevelProgressUI`: Đổi sang pattern Singleton `Instance` và áp dụng vào `UIManager` / `SaveLoadManager` để loại bỏ `FindFirstObjectByType`.
    - `InputManager`: Đổi `private struct HitDistanceComparer` thành `private class` để triệt tiêu Boxing alloc khi gọi `Array.Sort` trong update loop.

- [x] **Task 5.3: Tối ưu Cấp Phát Memory (Medium GC)**
  - **Hoàn thiện (16/06/2026):**
    - `GridManager`: Đưa `new Queue<GridCell>()` và `new HashSet<GridCell>()` trong hàm `CalculatePriorities` và `CalculateLocalPriorities` ra thành biến toàn cục (Cache).
    - Dùng `.Clear()` trước mỗi lần BFS chạy thay vì khởi tạo lại liên tục, xoá bỏ memory leak ngầm. Bảm bảo vòng lặp Cascade sạch bóng Zero-GC.

---

### PHASE 6: Implement Gameplay Booster System (Ngày 8)

- [x] **Task 6.1: Tạo BoosterManager & API hỗ trợ**
  - **Hoàn thiện (16/06/2026):**
    - Tạo `BoosterManager.cs` (Singleton) quản lý state machine của Booster (`Idle`, `MoveSource`, `TrashTarget`).
    - Viết logic 4 loại Booster:
      - **Cutter:** Tạo đĩa bánh mới ở ô trống kề bên đĩa thiếu, spawn lượng miếng tương ứng type đa số để gộp đủ 6 miếng (chuẩn Game Design Document).
      - **Sauce:** Tìm đĩa có nhiều miếng đa số nhất, xóa thiểu số và đổ thêm sốt (tạo miếng mới) cho đủ 6/6.
      - **Move:** Trực tiếp tích hợp vào hệ thống kéo-thả (Drag & Drop) của InputManager. Cho phép nhấc 1 đĩa lên và đặt vào ô trống bất kì, scale Ghost tự động khớp với GridCell.
      - **Trash:** Cho phép tap vào 1 đĩa bất kì để xóa ngay lập tức khỏi lưới, đưa vào ObjectPool và kích hoạt cascade.
    - Cập nhật `GridManager.cs`: Thêm `GetAllCells()`, `TriggerCascade(centerCell)`.

- [x] **Task 6.2: Tích hợp Booster Input vào Game Loop & UI**
  - **Hoàn thiện (16/06/2026):**
    - Sửa `InputManager.cs`: Ưu tiên bắt Raycast cho `BoosterManager` khi ở State `TrashTarget`. Kích hoạt luồng kéo thả đặc biệt cho State `MoveSource`.
    - Sửa `BoostButton.cs`: Xây dựng hệ thống Toggle UI (tự động làm tối các nút đang Active, sáng lại khi Cancel).
    - Logic Hủy (Cancel): Nhấn lại nút đang chọn để hủy, hoặc chọn nút Booster khác để hủy cái cũ và kích hoạt cái mới. Trả lại State Idle an toàn.
  - **Lưu ý thao tác Unity Editor:** Cần kéo thả Prefab (hoặc tạo GameObject) gắn script `BoosterManager` vào Scene.
