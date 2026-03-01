# 訂單類型說明（BR / DR / VR）

## BR — 訂房（Booking Room）

**ID 格式**：`BR` + YYMMDD + `-` + 三位序號（例：`BR250420-001`）

**req.md 關鍵欄位**：
- `type: 訂房`
- `check_in`、`check_out`（或由 `date` + `nights` 推算）
- `guests`：人數
- `budget`：預算（選填）
- `preferences`：偏好（景觀、高樓層等，選填）

**產出**：
- 確認單：使用 `assets/br-confirmation.md`
- 任務清單：使用 `assets/br-task.md`

---

## DR — 訂餐（Dining Reservation）

**ID 格式**：`DR` + YYMMDD + `-` + 序號（例：`DR250421-001`）

**req.md 關鍵欄位**：
- `type: 訂餐`
- `time`：用餐時間
- `guests`：人數
- `preferences`：口味偏好、忌口
- `related_order`：關聯訂房 ID（選填）

**低消邏輯**：依 `menu.yaml` 的 `min_per_person` × `guests` 計算門檻，
若推薦菜色合計未達低消，提示差額。

**產出**：
- 確認單：使用 `assets/dr-confirmation.md`

---

## VR — 場地（Venue Reservation）

**ID 格式**：`VR` + YYMMDD + `-` + 序號（例：`VR250420-001`）

**req.md 關鍵欄位**：
- `type: 場地`
- `event_type`：婚宴 / 會議 / 派對
- `guests`：人數
- `start_time`、`end_time`：活動時段
- `guest_list`：賓客名單（三種模式如下）

### 賓客名單三種模式

| 模式 | req.md 寫法 | 說明 |
|------|------------|------|
| A：外部 YAML | `guest_list: VR250420-001-名單.yaml` | 名單存在獨立 YAML 檔 |
| B：內嵌名單 | `guest_list: inline`，下方列清單 | 直接在 req.md 內列出 |
| C：無名單 | `guest_list: none` 或省略 | 僅記人數，不列名 |

**桌次計算**：`ceil(guests / 10)` 桌（預設圓桌 10 人/桌）

**產出**：
- 確認單：使用 `assets/vr-confirmation.md`
