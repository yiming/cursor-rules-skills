---
name: check-availability
description: 查詢指定日期的空房狀態。當使用者問「有沒有空房」「查空房」時觸發。
argument-hint: "[check-in] [check-out]"
user-invocable: true
---

# 查詢空房狀態

## 現有房間

!`cat master-data/room-list.yaml`

## 房型與價格

!`cat master-data/room-types.yaml`

## 已被預訂的房間（從現有確認單中掃描）

!`grep -r "建議房號" orders/*-確認單.md 2>/dev/null || echo "目前無已確認訂單"`

## 指示

請根據以上即時資料，列出 $1 至 $2 期間可用的房間：

1. 掃描所有確認單，找出該期間已被預訂的房號
2. 從房間清單中排除已預訂的房號
3. 按房型分組，列出可用房號、容量、每晚價格
4. 以表格呈現結果
