---
order_id: {id}
generated_at: {產出時間 YYYY-MM-DD HH:mm}
---
# 任務清單 {id}

## 場地準備
- [ ] 確認 {date} {session.name} 場地 {venue.name} 可用
- [ ] 場地佈置安排（{seating_type}，{table_count} 桌）
- [ ] {逐項列出加購項目的準備工作}

## 餐飲準備
- [ ] 通知宴會廚房 {date} {session.name} 備餐（{banquet_menu.name}，{guest_count} 人份）
- [ ] 素食需求 {veg_count} 位（{素食者姓名列表}）
- [ ] {若有其他飲食需求} {需求描述}

## 桌次與名單
- [ ] 確認最終名單（目前 {guest_count} 位）
- [ ] 桌次安排確認（{table_count} 桌）
- [ ] {若需要桌牌} 產生桌牌 prompt → 交付圖像生成

## 客戶溝通
- [ ] 回覆客戶 {guest} 確認單（含費用明細與桌次安排）
- [ ] {若超出預算} 聯繫客戶確認預算調整
- [ ] {若有 related_order} 通知房務一併安排（關聯訂房 {related_order}）
- [ ] {若最低消費不足} 聯繫客戶確認加購項目

## 活動當日
- [ ] 活動前 2 小時 final check（場地、餐飲、設備）
- [ ] 活動當日迎賓準備
- [ ] {若為戶外場地} 確認天候狀況，備案準備
