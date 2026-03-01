---
order_id: "{id}"
generated_at: "{YYYY-MM-DD HH:mm}"
status: 待確認
---

# 場地任務清單

## 訂單狀態

| 欄位 | 內容 |
|------|------|
| 訂單編號 | {id} |
| 聯絡人 | {guest} |
| 活動日期 | {date} |
| 場地 | {venue_name}（{venue_id}） |
| 場次 | {session_name} |
| 人數 | {pax} 人 |
| 桌數 | {table_count} 桌（含 VIP 桌 {vip_table_count} 桌）|
| 訂單金額 | {total} 元 |
| 狀態 | ⬜ 待確認 |

## 宴會廳佈置

- [ ] 場地圖確認（桌次安排表已產出）
- [ ] 圓桌搭設（{table_count} 桌）
- [ ] VIP 主桌佈置（{vip_table_count} 桌，每桌 ≤ 8 人）
- [ ] 音響、麥克風測試
- [ ] 投影設備測試<!-- 若有簡報需求 -->
- [ ] 舞台搭設<!-- 若需要 -->
- [ ] 簽到桌設置（來賓入口處）
- [ ] 加購項目：
  {options_checklist}<!-- 格式：- [ ] {加購項目名稱}，來源：{option_id} -->

## 餐飲備料（宴會）

- [ ] 宴會餐標確認：{banquet_menu_name}（{banquet_menu_id}，NT${banquet_price_per_person}/人）
- [ ] 廚房出餐時間表確認（{pax} 人份，{table_count} 桌）
- [ ] 素食桌標記（{vegetarian_table_count} 桌）<!-- 若無可刪除 -->
- [ ] 素食菜單備妥<!-- 若無可刪除 -->

## 桌牌製作提醒

- [ ] **執行 `/hotel-generate-card-prompt {id}`** 產出桌牌圖像 Prompt
- [ ] 桌牌輸出（{table_count} 桌次牌 或 {pax} 張個人座位牌）
- [ ] 桌牌確認後交印刷廠製作（活動前 3 日）

## VIP 接待

- [ ] VIP 賓客清單確認
- [ ] 主桌（{vip_table_no}）預留名牌
- [ ] 貴賓停車位預留（opt-parking-vip）<!-- 若有 VIP 停車需求 -->
- [ ] VIP 迎賓禮品準備<!-- 若有 -->

## 前台通知

- [ ] 確認單寄送給客人
- [ ] 確認付款方式（訂金 / 全額預付 / 現場結帳）
- [ ] 若有外燴承辦商，確認協調時間

## 備註

{其他任務或特殊說明}

---
> 此任務清單由 `/hotel-order-from-req` 自動產出，請各部門依項目完成後勾選。
> 負責部門：**宴會廳** / **餐飲部** / **工程部** / **前台**
