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
  - **Verification:** Chạy game, thay đổi vàng, tắt game mở lại -> Số vàng được giữ nguyên.
  - **File:** `Scripts/Core/SaveLoadManager.cs`

---

### PHASE 2: Cấu hình và UI Shop (Ngày 3)

- [x] **Task 2.1: Tạo ScriptableObject hoặc JSON Cấu hình Shop**
  - Định nghĩa danh sách vật phẩm: ID, Tên, Giá vàng, Prefab Mesh (hoặc Material).
  - **File:** `Scripts/Data/ShopConfig.cs` (ScriptableObject)

- [ ] **Task 2.2: Lập trình UI Shop**
  - Màn hình Shop lấy dữ liệu từ `ShopConfig` để sinh các nút mua.
  - Logic mua: Check vàng -> Trừ vàng -> Thêm skin vào `UnlockedSkins` -> Lưu file.
  - **File:** `Scripts/UI/ShopManager.cs`

- [x] **Task 2.3: Tích hợp Skin vào Gameplay**
  - Khi `TrayManager` hoặc `PizzaPlate` sinh đĩa, đọc `CurrentSkinId` từ `PlayerData` để áp dụng đúng Mesh/Material mới.
  - **File:** `Scripts/Gameplay/PizzaPlate.cs` (cập nhật)

---

### PHASE 3: Daily Rewards (Ngày 4 - 5)

- [ ] **Task 3.1: Logic tính toán thời gian UTC**
  - Tạo class `DailyRewardManager`.
  - Viết hàm check: So sánh `DateTime.UtcNow` với `LastDailyRewardTime` (được lưu dưới dạng binary hoặc tick).
  - Nếu khoảng cách qua ngày mới (hoặc > 24h) -> Kích hoạt nhận quà.

- [ ] **Task 3.2: Giao diện nhận thưởng**
  - Hiển thị UI nút "Nhận Quà", bấm vào cộng vàng -> Gọi `SaveLoadManager.Save()`.
  - **Verification:** Tắt game, đổi giờ Local của Window -> Mở lại game KHÔNG nhận được quà (Vì tính bằng UTC).

---

### PHASE 4: Achievement System (Ngày 6 - 7)

- [ ] **Task 4.1: Khai báo hệ thống Event chung**
  - Tạo class tĩnh `GameEvents.cs` chứa các Action: `OnPizzaMerged(int count)`, `OnPlateExploded(int totalSlices)`, `OnLevelCompleted()`.
  - Sửa `GridManager` để phát event này mỗi khi nổ đĩa/merge.

- [ ] **Task 4.2: AchievementManager lắng nghe Event**
  - `AchievementManager` đăng ký lắng nghe các event trên trong `OnEnable/OnDisable`.
  - Cập nhật tiến độ `AchievementProgress` tương ứng -> Khi đạt mốc -> Mở khóa -> Phát popup -> Lưu Save file.
  - **Verification:** Thấy thông báo Popup "Thành tựu" hiện lên khi đủ điều kiện mà không dính líu code vào `GridManager`.
