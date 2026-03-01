---
description: "Use when reading, referencing, or updating hotel master data files (room types, room list, menu, venues). Covers field meanings and read-only policy."
applyTo: "master-data/**/*.yaml"
---

# 飯店 Master Data 規則

## 目錄內容

| 檔案 | 說明 |
|------|------|
| `room-types.yaml` | 房型定義（代碼、名稱、容量、每晚定價） |
| `room-list.yaml` | 實體房間清單（房號、房型代碼、樓層、備註） |
| `menu.yaml` | 餐飲品項與定價（含低消門檻） |
| `venues.yaml` | 場地清單（場地代碼、容量、計費方式） |

## 欄位約定

### room-types.yaml
- `code`：房型代碼（英數，如 STD-D）
- `name`：房型中文名稱
- `capacity`：最大容納人數
- `price_per_night`：每晚定價（整數，新台幣）
- `breakfast_surcharge`：早餐附加費（每人每晚）

### room-list.yaml
- `room_no`：房號（唯一）
- `type_code`：對應 room-types.yaml 的 `code`
- `floor`：樓層
- `notes`：備註（如「景觀房」「無障礙」）

## 唯讀政策

- master-data 為基準資料，訂單金額必須依此計算
- 若需調整定價，修改 YAML 後所有後續訂單自動採用新價
- 不得在訂單檔中硬編碼金額
