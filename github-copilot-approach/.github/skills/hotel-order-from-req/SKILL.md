---
name: hotel-order-from-req
description: "Process hotel order request files (.req.md) to generate confirmation docs and task lists. Use when asked to handle orders, generate confirmation, or create task lists from req files. Supports BR (room booking), DR (dining), VR (venue) order types."
argument-hint: "訂單需求檔路徑，例：orders/BR250420-001.req.md"
---

# 依訂單需求產生確認單與任務清單

## 適用時機

- 使用者說「處理這張訂單」「產生確認單」「建立任務清單」
- 開啟或引用任何 `orders/*.req.md` 檔案時

## 執行流程

### 步驟 1：讀取需求

1. 讀取指定的 `*.req.md`
2. 解析 YAML front matter，取得 `type`、`date`、`id`、`guest`
3. 依 `type` 進入對應分支

### 步驟 2A：訂房（BR）

1. 載入 `master-data/room-types.yaml`、`master-data/room-list.yaml`
2. 比對需求（人數、預算、日期）→ 推薦房型
3. 計算費用（房費 × 晚數 + 選項費用）
4. 從 room-list 選可用房號
5. 依 [br-confirmation.md](./assets/br-confirmation.md) 模板產出確認單
6. 依 [br-task.md](./assets/br-task.md) 模板產出任務清單

### 步驟 2B：訂餐（DR）

1. 載入 `master-data/menu.yaml`
2. 確認人數是否達低消門檻
3. 依需求推薦菜色組合
4. 依 [dr-confirmation.md](./assets/dr-confirmation.md) 模板產出確認單
5. 依 [dr-task.md](./assets/dr-task.md) 模板產出任務清單

### 步驟 2C：場地（VR）

1. 載入 `master-data/venues.yaml`
2. 依人數、活動類型推薦場地
3. 計算桌次（圓桌 10 人／桌）、費用明細
4. 處理賓客名單（三種模式，詳見 [order-types.md](./references/order-types.md)）
5. 依 [vr-confirmation.md](./assets/vr-confirmation.md) 模板產出確認單
6. 依 [vr-task.md](./assets/vr-task.md) 模板產出任務清單

### 步驟 3：輸出

- 確認單：`orders/{id}-確認單.md`
- 任務清單：`orders/{id}-任務.md`
- 不得修改來源 `*.req.md`

## 自查清單

產出前確認：
- [ ] 金額與 master-data 一致
- [ ] 房號存在於 room-list.yaml（BR）
- [ ] 低消達標（DR）
- [ ] 桌次計算正確（VR）
- [ ] 確認單含 YAML front matter（`order_id`、`generated_at`）
