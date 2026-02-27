---
name: hotel-order-from-req
description: 從 orders/{id}.req.md 讀取需求、參照 master-data 產生確認單與任務清單。用於：(1) 使用者要求「依這張訂單產生確認單」或「產生任務清單」時 (2) 處理 orders/ 下的 *.req.md 訂房/訂餐/場地需求時 (3) 需要計算房價、排房號、比對預算時。
---

# 從需求產生訂單產出

## 步驟

1. **讀取需求**：開啟 `orders/{id}.req.md`，解析 YAML front matter（type, date, id, guest）與內文需求。
2. **載入 master data**：依 `type` 載入對應檔案：
   - 訂房 → `master-data/room-types.yaml`（房型、價格、加購選項）+ `master-data/room-list.yaml`（房號清單）
   - 訂餐 → `master-data/menu.yaml`（若有）
   - 場地 → `master-data/venues.yaml`（若有）
3. **計算與匹配**：
   - 依人數、預算、偏好匹配最適房型
   - 計算總價 = 房價 × 晚數 + 各加購項目小計
   - 若超出預算，列出替代方案
   - 從 room-list.yaml 篩選該房型可用房號
4. **產出**：依下方模板寫入 `orders/{id}-確認單.md` 與 `orders/{id}-任務.md`。

## 確認單模板

```
---
order_id: {id}
generated_at: {產出時間 YYYY-MM-DD HH:mm}
---
# 訂房確認單 {id}

| 項目 | 內容 |
|------|------|
| 訂單編號 | {id} |
| 訂房人 | {guest} |
| 入住日期 | {check_in} |
| 退房日期 | {check_out}（共 {nights} 晚） |
| 人數 | {guests} 位 |
| 房型 | {room_type.name}（{room_type.id}） |
| 建議房號 | {從 room-list 篩選} |
| 房價 | {price_per_night} × {nights} 晚 = {room_subtotal} 元 |
| 加購項目 | {逐項：名稱 × 數量 × 天數 = 小計} |
| **合計** | **{total} 元** |
| 客戶預算 | {budget} 元 |
| 預算比對 | {在預算內 ✓ / 超出 X 元，建議替代方案} |

### 備註

{需求中的備註與偏好}
```

## 任務清單模板

```
---
order_id: {id}
generated_at: {產出時間 YYYY-MM-DD HH:mm}
---
# 任務清單 {id}

- [ ] 確認 {check_in}～{check_out} 房況，鎖定房號 {建議房號}
- [ ] 回覆客戶 {guest} 確認單（含價格明細）
- [ ] 安排加購項目（{列出加購項}）
- [ ] 入住前一日 final check
- [ ] 入住當日迎賓準備
```

## 品質檢查

產出前執行以下驗證：
- 確認所有金額可從 master-data 的單價反算
- 確認房型容量 ≥ 入住人數
- 確認建議房號存在於 room-list.yaml 且屬於該房型
- 確認確認單與任務清單的 order_id 一致
