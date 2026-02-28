---
name: process-dining-order
description: 從訂餐需求檔產生確認單與任務清單。當使用者說「處理訂餐」「訂餐確認單」時觸發。
argument-hint: "[order-id]"
user-invocable: true
---

# 從訂餐需求產生確認單與任務清單

## 輸入

讀取 `orders/$ARGUMENTS.req.md`，解析 YAML front matter（type, date, id, guest, related_order）與內文需求。

## 步驟

1. **讀取需求**：開啟需求檔，理解用餐日期、餐別、人數、品項偏好、預算、特殊需求（素食/過敏等）。
2. **載入主資料**：讀取 `master-data/menu.yaml`，取得：
   - `config.minimum_charge`（低消門檻）
   - `meals`（正餐/套餐）
   - `drinks`（飲料）
   - `snacks`（點心）
3. **匹配品項**：依需求內容匹配 menu.yaml 中的品項，計算數量與小計。
4. **低消檢查**（本 Skill 的核心邏輯）：
   - 對每位用餐者，計算該人的消費額（分攤正餐 + 個人單點）
   - 若消費額 < `config.minimum_charge`（300 元）：
     - 差額 = minimum_charge - 消費額
     - 從 drinks + snacks 中建議補足項目（優先選最接近差額的組合）
     - 在確認單中標記：「⚠ 低消不足，建議加點」
5. **預算比對**：合計金額 vs 客戶預算。
6. **產出**：依模板寫入確認單與任務清單。
   - 確認單模板：`.claude/skills/process-dining-order/templates/confirmation.md`
   - 任務清單模板：`.claude/skills/process-dining-order/templates/task-list.md`

## 低消計算範例

4 人晚餐，3 人吃火鍋套餐（DN-B 1800/2人份 → 每人 900），1 人只點紅茶（DK-03 80 元）：

| 用餐者 | 品項 | 小計 | 低消狀態 |
|--------|------|------|---------|
| A | 火鍋套餐（均攤）| 900 | ✓ |
| B | 火鍋套餐（均攤）| 900 | ✓ |
| C | 火鍋套餐（均攤）| 900 | ✓ |
| D | 紅茶 ×1 | 80 | ⚠ 不足 220 元 |
| | 【建議加點】起司蛋糕 + 綜合堅果 | +300 | → 380 ✓ |

## 特殊需求處理

- 素食：標記在確認單備註，提醒廚房準備替代品項
- 過敏：同上，並在任務清單加入過敏警示項目
- 若需求中未指定用餐者各自點什麼，則視為均分

## 品質檢查

產出前執行以下驗證：
- 所有品項 ID 存在於 menu.yaml
- 所有單價與 menu.yaml 一致
- 逐人低消檢查已完成
- 確認單與任務清單的 order_id 一致
- 若有 related_order，確認關聯訂單存在
