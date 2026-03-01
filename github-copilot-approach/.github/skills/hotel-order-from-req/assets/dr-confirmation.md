---
order_id: "{id}"
generated_at: "{YYYY-MM-DD HH:mm}"
---

# 訂餐確認單

## 訂單摘要

| 欄位 | 內容 |
|------|------|
| 訂單編號 | {id} |
| 聯絡人 | {guest} |
| 用餐日期 | {date} |
| 用餐時間 | {time} |
| 人數 | {guests} 人 |
| 餐廳／包廂 | {venue} |

## 建議菜單

| 品項 | 單價 | 數量 | 小計 |
|------|------|------|------|
| {item_1} | {price_1} 元 | {qty_1} | {subtotal_1} 元 |
| {item_2} | {price_2} 元 | {qty_2} | {subtotal_2} 元 |
| **合計** | | | **{total} 元** |

> 低消門檻：{min_spend} 元（{guests} 人 × {min_per_person} 元/人）
> 目前合計：{total} 元 → {達標 | 未達標，差額 {gap} 元}

## 飲食限制與特殊需求

{自由填寫}

---
> 此確認單由 `/hotel-order-from-req` 自動產出。
