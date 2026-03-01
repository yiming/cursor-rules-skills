---
order_id: "{id}"
generated_at: "{YYYY-MM-DD HH:mm}"
---

# 訂房確認單

## 訂單摘要

| 欄位 | 內容 |
|------|------|
| 訂單編號 | {id} |
| 訂房人 | {guest} |
| 入住日期 | {check_in} |
| 退房日期 | {check_out} |
| 晚數 | {nights} 晚 |
| 房型 | {room_type_name}（{room_type_code}） |
| 建議房號 | {room_no} |
| 人數 | {guests} 人 |

## 費用明細

| 項目 | 單價 | 數量 | 小計 |
|------|------|------|------|
| 房費 | {price_per_night} 元/晚 | {nights} 晚 | {room_total} 元 |
| 早餐（選） | {breakfast_surcharge} 元/人/晚 | {pax}×{nights} | {breakfast_total} 元 |
| **合計** | | | **{total} 元** |

## 備註

{自由填寫的特殊需求或注意事項}

---
> 此確認單由 `/hotel-order-from-req` 自動產出，如有異動請聯絡訂房部。
