---
name: hotel-check-availability
description: "Check hotel room availability for a given date range. Use when asked to check availability, find empty rooms, or list available rooms for specific dates."
argument-hint: "日期範圍，例：2025-04-20 到 2025-04-22"
---

# 查詢空房

## 適用時機

- 使用者說「查空房」「哪些房間可用」「{日期} 有哪些房」

## 執行流程

1. 讀取 `master-data/room-types.yaml`，取得房型定義與定價
2. 讀取 `master-data/room-list.yaml`，取得所有實體房號
3. 掃描 `orders/*-確認單.md`的 front matter，找出日期重疊的訂房記錄，標記對應房號為「已佔用」
4. 輸出可用清單（依房型分組）

## 輸出格式

```
## 可用房間（YYYY-MM-DD ～ YYYY-MM-DD，共 N 晚）

### {房型名稱}（{代碼}）｜容量：N 人｜每晚 N,NNN 元
| 房號 | 樓層 | 備註 | 狀態 |
|------|------|------|------|
| 301  | 3F   |      | ✓ 可用 |
| 302  | 3F   |      | ✗ 已訂（{訂單ID}） |

---
共可用 N 間（{房型1} N 間、{房型2} N 間）
```
