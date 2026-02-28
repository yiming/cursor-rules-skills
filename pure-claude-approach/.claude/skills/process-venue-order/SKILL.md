---
name: process-venue-order
description: 從場地租借需求檔產生確認單與任務清單。當使用者說「處理場地」「場地確認單」「處理VR訂單」時觸發。
argument-hint: "[order-id]"
user-invocable: true
---

# 從場地租借需求產生確認單與任務清單

## 輸入

讀取 `orders/$ARGUMENTS.req.md`，解析 YAML front matter（type, date, id, guest, related_order, guest_list）與內文需求。

依 `guest_list` 欄位決定名單來源：
- **外部檔案**（值為 `{file}.yaml`）：讀取 `orders/{guest_list}`
- **內嵌**（值為 `embedded`）：從 req.md 內文的 YAML 區塊解析名單
- **未填**：無名單，僅依人數處理（不執行桌次安排）

## 步驟

1. **讀取需求**：開啟需求檔，理解活動類型、日期、場次、人數、座位安排、餐標、預算、特殊需求。
2. **載入主資料**：讀取 `master-data/venues.yaml`，取得：
   - `config`（minimum_venue_charge, table_capacity_round, vip_table_capacity, card_style_default）
   - `venues`（場地清單與各場次定義）
   - `banquet_menus`（宴會餐標）
   - `options`（加購選項）
3. **場地匹配**：
   - 依座位安排類型（宴會/劇院/教室）選用對應的 capacity 欄位
   - 篩選容量 >= 人數的場地
   - 若多個場地符合，列出比較表供選擇（優先推薦最經濟的）
   - 若無場地符合，明確告知並建議拆分場次或減少人數
4. **場地費計算**：
   - 單場：`venue.price_per_session`
   - 全日場：`venue.price_per_session × session.price_multiplier`
5. **餐飲計算**（場地＋餐飲混合計價的核心）：
   - 若需求包含宴會餐標：餐費 = `banquet_menu.price_per_person × 人數`
   - 素食人數統計（從名單的 dietary 欄位），素食者同餐標價格，備註通知廚房
   - 若無餐標需求：餐飲費 = 0
6. **桌次安排演算法**（見下方詳述）：
   - 讀取名單 YAML
   - 依 group.priority 順序安排桌次
   - 主桌使用 `vip_table_capacity`（8 人）
   - 一般桌使用 `table_capacity_round`（10 人）
   - 同 group 成員盡量同桌
   - 產出桌次表
7. **加購項目計算**：逐項比對需求中提到的加購，從 options 取得單價。
8. **合計與最低消費檢查**：
   - 合計 = 場地費 + 餐飲費 + 加購合計
   - 若合計 < `config.minimum_venue_charge`：標記「⚠ 未達場地最低消費」並建議加購補足
9. **預算比對**：合計 vs 客戶預算。
10. **產出**：依模板寫入確認單與任務清單。
    - 確認單模板：`.claude/skills/process-venue-order/templates/confirmation.md`
    - 任務清單模板：`.claude/skills/process-venue-order/templates/task-list.md`
11. **提示桌牌生成**：若需求中提到「桌牌」或「card」，在確認單末尾提示可執行 `/generate-card-prompt $ARGUMENTS`

## 桌次安排演算法

### 輸入
- 名單 YAML（groups → members，每組有 priority）
- `table_capacity_round`（預設 10）
- `vip_table_capacity`（預設 8）

### 演算法步驟

1. **主桌優先**：priority=1 的 group 安排到第 1 桌（主桌），上限 `vip_table_capacity` 人。
   若主桌 group 人數 > vip_table_capacity，溢出者安排到第 2 桌。

2. **依 priority 順序填桌**：
   - 取下一個 group
   - 若當前桌剩餘座位 >= group 剩餘人數 → 全部放入當前桌
   - 若當前桌剩餘座位 < group 剩餘人數但 >= 一半 → 拆分（盡量多放當前桌）
   - 若當前桌剩餘座位 < group 人數的一半 → 開新桌，整組放入新桌
   - 原則：**同組盡量同桌 > 桌次利用率**

3. **素食標記**：每桌統計素食人數，若同桌素食 >= 3 人則標記為「素食桌」（全桌出素食餐），否則個別標記。

4. **邊界情況**：
   - 人數不足 1 桌（如 3 人）：仍開 1 桌
   - 人數恰好整除：理想狀態，直接分配
   - 最後 1-2 人溢出：併入前一桌（上限 +2）而非獨開小桌

### 桌次表範例

| 桌次 | 桌名 | 人數 | 賓客 | 備註 |
|------|------|------|------|------|
| 1 | 主桌 | 8 | 張大偉（新郎）、林佳怡（新娘）… | VIP |
| 2 | 男方親友 A | 10 | 張小明（堂兄）、張美華（姑姑）… | 素食 ×1 |
| 3 | 男方親友 B | 10 | … | |

## 品質檢查

產出前執行以下驗證：
- 場地 ID 存在於 venues.yaml
- 場次 ID 存在於該場地的 sessions 中
- 場地容量 >= 實際人數
- 餐標 ID 存在於 banquet_menus
- 所有加購 ID 存在於 options
- 所有單價與 venues.yaml 一致
- 桌次總人數 = 名單總人數
- 素食人數統計一致（名單 vs 確認單）
- 最低消費檢查已完成
- 確認單與任務清單的 order_id 一致
- 若有 related_order，確認關聯訂單存在
