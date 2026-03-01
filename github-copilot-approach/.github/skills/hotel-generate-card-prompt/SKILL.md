---
name: hotel-generate-card-prompt
description: "Generate AI image generation prompts for banquet table name cards. Use when asked to create table cards, card prompts, or seating card images for venue orders."
argument-hint: "場地訂單 ID，例：VR250420-001"
---

# 產生桌牌 Image Prompt

## 適用時機

- 使用者說「產生桌牌」「產桌牌 prompt」「建立桌次名牌」
- 場地確認單產出後，進行桌牌製作

## 執行流程

1. 讀取指定的 `orders/{id}-確認單.md`，取得：
   - 活動名稱、日期
   - 桌次清單與賓客名單（table 1 ~ N）
   - 主辦方偏好風格（若有）
2. 依每桌產出一段 AI 圖片生成 prompt
3. 將所有 prompt 寫入 `orders/{id}-card-prompt.md`

## 輸出格式

````markdown
# {id} 桌牌 Prompt

> 活動：{活動名稱}｜日期：{日期}

## 桌次 1

**賓客**：{姓名列表}

**Prompt**：
```
A elegant table name card for table 1. Event: "{活動名稱}" on {日期}.
Guests: {姓名列表}. Style: {風格描述}.
White background, gold typography, minimalist design.
```

---

## 桌次 2
...
````
