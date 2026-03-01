---
description: "Check hotel room availability for a given date range"
argument-hint: "查詢日期範圍，例：2025-04-20 到 2025-04-22"
agent: "agent"
---

# 查詢空房

請依下列步驟查詢指定日期範圍的可用房間：

1. 讀取 `master-data/room-types.yaml`，取得所有房型定義
2. 讀取 `master-data/room-list.yaml`，取得所有實體房間
3. 掃描 `orders/*-確認單.md`，找出日期區間重疊的訂房確認單，標記對應房號為「已佔用」
4. 輸出下列格式的可用房間清單：

---

## 可用房間（{{start_date}} ～ {{end_date}}）

### {{房型名稱}}（{{房型代碼}}）｜容量：{{capacity}} 人｜每晚 {{price}} 元

| 房號 | 樓層 | 備註 | 狀態 |
|------|------|------|------|
| ...  | ...  | ...  | ✓ 可用 |
