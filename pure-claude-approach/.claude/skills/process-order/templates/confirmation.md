---
order_id: {id}
generated_at: {產出時間 YYYY-MM-DD HH:mm}
---
# [UU大飯店]訂房確認單 {id}

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
