---
order_id: "{id}"
generated_at: "{YYYY-MM-DD HH:mm}"
---

# 場地確認單

## 訂單摘要

| 欄位 | 內容 |
|------|------|
| 訂單編號 | {id} |
| 主辦人 | {guest} |
| 活動日期 | {date} |
| 活動時間 | {start_time} ～ {end_time} |
| 活動類型 | {event_type}（婚宴 / 會議 / 派對） |
| 場地 | {venue_name}（{venue_code}） |
| 人數 | {guests} 人 |
| 桌次 | {tables} 桌（{pax_per_table} 人/桌） |

## 費用明細

| 項目 | 單價 | 數量 | 小計 |
|------|------|------|------|
| 場地費 | {venue_fee} 元/{unit} | {qty} | {subtotal_venue} 元 |
| 餐飲（桌菜） | {meal_per_table} 元/桌 | {tables} 桌 | {subtotal_meal} 元 |
| **合計** | | | **{total} 元** |

## 賓客名單

{guest_list_section}
<!-- 三種模式（詳見 references/order-types.md）：
     A. 外部 YAML：!include VR250420-001-名單.yaml
     B. 內嵌名單：直接展開表格
     C. 無名單：僅記人數 -->

## 後續步驟

1. 確認後執行 `/hotel-generate-card-prompt {id}` → 產出桌牌 prompt
2. 將 prompt 交給圖片生成工具 → 存入 `orders/{id}-cards/`

---
> 此確認單由 `/hotel-order-from-req` 自動產出。
