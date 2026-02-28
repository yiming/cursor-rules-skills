---
name: hotel-order-from-req
description: 從 orders/{id}.req.md 讀取需求、參照 master-data 產生確認單與任務清單。用於：(1) 使用者要求「依這張訂單產生確認單」或「產生任務清單」時 (2) 處理 orders/ 下的 *.req.md 訂房/訂餐/場地需求時 (3) 需要計算房價、排房號、低消檢查、場地計價、桌次安排時。
---

# 從需求產生訂單產出

## 步驟 0：識別訂單類型

讀取 `orders/{id}.req.md`，解析 YAML front matter：
- `type`：訂房 | 訂餐 | 場地
- `date`、`id`、`guest`
- `related_order`（可選）
- `guest_list`（場地用，可選）

依 `type` 進入對應分支。

---

## 分支 A：訂房（type: 訂房，id 前綴 BR）

### A1. 載入主資料

- `master-data/room-types.yaml`（房型、容量、每晚價格、加購選項）
- `master-data/room-list.yaml`（房號清單）

### A2. 計算與匹配

1. 依人數、預算、偏好匹配最適房型（容量 ≥ 入住人數）
2. 計算總價 = `price_per_night × nights + 各加購小計`
3. 若超出預算，列出替代方案（降一級房型）
4. 從 room-list.yaml 篩選該房型可用房號（建議同樓層連號）

### A3. 產出

依 `templates/br-confirmation.md` 寫入 `orders/{id}-確認單.md`
依 `templates/br-task-list.md` 寫入 `orders/{id}-任務.md`

### A4. 自查清單

- [ ] 房型容量 ≥ 入住人數
- [ ] 建議房號存在於 room-list.yaml 且屬於該房型
- [ ] 所有金額可從 room-types.yaml 的單價反算
- [ ] 確認單與任務清單的 order_id 一致

---

## 分支 B：訂餐（type: 訂餐，id 前綴 DR）

### B1. 載入主資料

- `master-data/menu.yaml`（品項、價格、低消規則）
  - `config.minimum_charge`（每人低消門檻）
  - `meals`、`drinks`、`snacks`

### B2. 匹配品項

依需求內容匹配 menu.yaml 中的品項，計算數量與小計。
若需求未指定各人分配，視為均分。

### B3. 低消檢查（核心邏輯）

對每位用餐者計算消費額（分攤正餐 + 個人單點）：
- 若消費額 ≥ `config.minimum_charge`（300 元）→ ✓
- 若消費額 < 300 元：
  - 差額 = 300 - 消費額
  - 從 drinks + snacks 中建議補足（優先選最接近差額的組合）
  - 在確認單標記「⚠ 低消不足，建議加點」

**低消計算範例**：
4 人晚餐，3 人吃火鍋套餐（DN-B 1800/2人份 → 每人 900），1 人只點紅茶（DK-03 80 元）：

| 用餐者 | 品項 | 消費額 | 低消狀態 |
|--------|------|--------|---------|
| A | 火鍋套餐（均攤）| 900 元 | ✓ |
| B | 火鍋套餐（均攤）| 900 元 | ✓ |
| C | 火鍋套餐（均攤）| 900 元 | ✓ |
| D | 紅茶 ×1 | 80 元 | ⚠ 不足 220 元 → 建議加點起司蛋糕(180)+綜合堅果(120) |

### B4. 產出

依 `templates/dr-confirmation.md` 寫入 `orders/{id}-確認單.md`
依 `templates/dr-task-list.md` 寫入 `orders/{id}-任務.md`

### B5. 自查清單

- [ ] 所有品項 ID 存在於 menu.yaml
- [ ] 所有單價與 menu.yaml 一致
- [ ] 逐人低消檢查已完成
- [ ] 若有 related_order，確認關聯訂單存在
- [ ] 確認單與任務清單的 order_id 一致

---

## 分支 C：場地（type: 場地，id 前綴 VR）

### C1. 載入主資料

- `master-data/venues.yaml`（場地、場次、餐標、加購選項）
  - `config`（minimum_venue_charge、table_capacity_round、vip_table_capacity）
  - `venues`、`banquet_menus`、`options`

### C2. 讀取名單

依 `guest_list` front matter 決定名單來源：
- **外部檔案**（值為 `{id}-名單.yaml`）：讀取 `orders/{guest_list}`
- **內嵌**（值為 `embedded`）：從 req.md 內文的 YAML 區塊解析
- **未填**：無名單，僅依人數處理，跳過桌次安排

### C3. 場地匹配

1. 依座位安排類型選用容量欄位：宴會 → `capacity_banquet`、劇院 → `capacity_theater`、教室 → `capacity_classroom`
2. 篩選容量 ≥ 人數的場地
3. 若多個符合，列出比較表，優先推薦最經濟的

### C4. 費用計算

**場地費**：
- 單場：`venue.price_per_session`
- 全日場：`venue.price_per_session × session.price_multiplier`

**餐飲費**：
- 若有宴會餐標：`banquet_menu.price_per_person × 人數`
- 素食人數從名單的 `dietary` 欄位統計，同餐標價格，備註通知廚房

**加購費**：逐項從 options 取得單價

**合計與最低消費檢查**：
- 合計 = 場地費 + 餐飲費 + 加購合計
- 若合計 < `config.minimum_venue_charge`（15,000 元）：標記「⚠ 未達最低消費」並建議加購補足

### C5. 桌次安排演算法（有名單時執行）

**輸入**：名單 YAML（groups → members，每組有 priority）

**演算法**：
1. **主桌優先**：priority=1 的 group 安排到第 1 桌（主桌），上限 `vip_table_capacity`（8 人）。溢出者安排到第 2 桌。
2. **依 priority 順序填桌**：
   - 當前桌剩餘 ≥ group 人數 → 全部放入
   - 當前桌剩餘 ≥ group 人數一半 → 拆分（盡量多放當前桌）
   - 當前桌剩餘 < group 人數一半 → 開新桌，整組放入
   - 原則：**同組盡量同桌 > 桌次利用率**
3. **素食標記**：每桌素食 ≥ 3 人 → 標記「素食桌」（全桌出素食餐）；否則個別標記
4. **邊界**：最後 1-2 人溢出時，併入前一桌（上限 +2）而非獨開小桌

### C6. 產出

依 `templates/vr-confirmation.md` 寫入 `orders/{id}-確認單.md`
依 `templates/vr-task-list.md` 寫入 `orders/{id}-任務.md`

確認單末尾包含桌牌產出說明（Cursor 特色：直接在 IDE 用 Gemini 產圖）。

### C7. 自查清單

- [ ] 場地 ID 存在於 venues.yaml
- [ ] 場次 ID 存在於該場地的 sessions 中
- [ ] 場地容量 ≥ 實際人數
- [ ] 餐標 ID 存在於 banquet_menus（若有）
- [ ] 所有加購 ID 存在於 options
- [ ] 所有單價與 venues.yaml 一致
- [ ] 桌次總人數 = 名單總人數（若有名單）
- [ ] 素食人數統計一致
- [ ] 最低消費檢查已完成
- [ ] 確認單與任務清單的 order_id 一致
