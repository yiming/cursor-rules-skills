---
order_id: {id}
generated_at: {產出時間 YYYY-MM-DD HH:mm}
---
# [UU大飯店]場地租借確認單 {id}

| 項目 | 內容 |
|------|------|
| 訂單編號 | {id} |
| 訂租人 | {guest} |
| 活動類型 | {event_type} |
| 活動日期 | {date} |
| 場地 | {venue.name}（{venue.id}）{venue.floor}F |
| 場次 | {session.name}（{session.time}） |
| 座位安排 | {seating_type} |
| 人數 | {guest_count} 位 |
| 關聯訂房 | {related_order 或 無} |

## 費用明細

| 項目 | 單價 | 數量 | 小計 |
|------|------|------|------|
| 場地費：{venue.name} {session.name} | {price_per_session} | 1 | {venue_subtotal} 元 |
| 宴會餐標：{banquet_menu.name}（{banquet_menu.id}） | {price_per_person}/人 | {guest_count} 人 | {dining_subtotal} 元 |
| {option.name}（{option.id}） | {option.price} | {qty} | {option_subtotal} 元 |
| ... | ... | ... | ... |
| **合計** | | | **{total} 元** |

## 最低消費檢查

| 場地最低消費 | 實際合計 | 狀態 |
|-------------|---------|------|
| {minimum_venue_charge} 元 | {total} 元 | {✓ 已達 / ⚠ 不足 {diff} 元} |

{若不足}
### 建議加購補足

| 建議項目 | 金額 |
|---------|------|
| {suggested_option} | +{amount} 元 |

## 預算比對

| 客戶預算 | 合計 | 結果 |
|---------|------|------|
| {budget} 元 | {total} 元 | {在預算內 ✓ / 超出 X 元} |

## 桌次安排

| 桌次 | 桌名 | 人數 | 賓客 | 備註 |
|------|------|------|------|------|
| 1 | 主桌 | {count} | {names_with_titles} | VIP |
| 2 | {group_name} A | {count} | {names_with_titles} | {dietary_notes} |
| ... | ... | ... | ... | ... |
| **合計** | | **{total_guests} 位** | | 素食 {veg_count} 位 |

## 飲食需求統計

| 需求類型 | 人數 | 安排 |
|---------|------|------|
| 一般 | {normal_count} | 標準餐 |
| 素食 | {veg_count} | 通知廚房備素食餐 |

### 特殊需求

{活動相關備註、佈置需求等}

### 桌牌

{若需要桌牌}

1. 執行 `/generate-card-prompt {id}` → 產出 `orders/{id}-card-prompt.md`（圖像生成用的 prompt 檔）
2. 將 prompt 檔交給 Cursor + Gemini 執行 → 圖片存入 `orders/{id}-cards/`
