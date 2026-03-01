---
name: hotel-report
description: "Generate monthly hotel operations report from order confirmation files. Use when asked for monthly report, revenue summary, occupancy stats, or order analytics."
argument-hint: "報告年月，例：2025-04"
---

# 月度營運報告

## 適用時機

- 使用者說「產生月報」「{月份} 營運報告」「訂單統計」

## 執行流程

1. 掃描 `orders/*-確認單.md`，篩選符合指定月份的確認單
2. 依訂單類型（BR/DR/VR）分別提取：

   | 類型 | 金額欄位 | 日期欄位 | 人數欄位 |
   |------|---------|---------|---------|
   | BR（訂房） | 合計列 | 入住日期 | 人數 |
   | DR（訂餐） | 品項明細合計 | 用餐日期 | 人數 |
   | VR（場地） | 費用明細合計 | 活動日期 | 人數 |

3. 計算彙總統計
4. 依 [monthly-report.md](./assets/monthly-report.md) 模板產出報告
5. 寫入 `analyse/{YYYY-MM}-報告.md`

## 輸出摘要欄位

- 訂單數量（依類型）
- 總營收（依類型 + 合計）
- 平均訂單金額
- 入住率（BR 專用：已訂間晚 / 總可用間晚）
- Mermaid 圓餅圖（營收占比）
- Mermaid 長條圖（每日訂單量）
