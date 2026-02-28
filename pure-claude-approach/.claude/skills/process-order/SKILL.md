---
name: process-order
description: 從訂房需求檔產生確認單與任務清單。當使用者說「處理訂單」「產生確認單」「產生任務清單」時觸發。
argument-hint: "[order-id]"
user-invocable: true
---

# 從需求產生訂單確認單與任務清單

## 輸入

讀取 `orders/$ARGUMENTS.req.md`，解析 YAML front matter（type, date, id, guest）與內文需求。

## 步驟

1. **讀取需求**：開啟需求檔，理解客戶的入住日期、人數、預算、偏好。
2. **載入主資料**：依 `type` 載入對應檔案：
   - 訂房 → `master-data/room-types.yaml` + `master-data/room-list.yaml`
   - 訂餐 → `master-data/menu.yaml`（若有）
   - 場地 → `master-data/venues.yaml`（若有）
3. **計算與匹配**：
   - 依人數、預算、偏好匹配最適房型
   - 計算總價 = 房價 × 晚數 + 各加購項目小計
   - 若超出預算，列出替代方案
   - 從 room-list.yaml 篩選該房型可用房號
4. **產出**：依模板寫入確認單與任務清單。
   - 確認單模板：`.claude/skills/process-order/templates/confirmation.md`
   - 任務清單模板：`.claude/skills/process-order/templates/task-list.md`

## 品質檢查

產出前執行以下驗證：
- 所有金額可從 master-data 的單價反算
- 房型容量 ≥ 入住人數
- 建議房號存在於 room-list.yaml 且屬於該房型
- 確認單與任務清單的 order_id 一致
