---
name: hotel-order-from-req
description: 依訂單資料夾內的 *.req.md（如 BR250315-001.req.md）與 master-data 產生確認單與任務清單。用於處理飯店訂房/訂餐/場地需求、產生確認單或任務.md 時。
---

# 從需求產生訂單產出

## 何時使用

- 使用者要求「依這張訂單產生確認單／任務清單」、或開啟 `orders/**/*.req.md` 並要求產出時。
- 產出請寫入 **orders/**，檔名為 `<id>-確認單.md`、`<id>-任務.md`（例：`BR250315-001-確認單.md`、`BR250315-001-任務.md`），不覆蓋該 `*.req.md`。

## 步驟

1. **讀取需求**：讀取 `orders/` 下的 `*.req.md`（如 `BR250315-001.req.md`），取得 YAML front matter（type, date, id）與內文需求；id 即檔名主檔名。
2. **參照 master data**：依 `type` 參照對應檔案：
   - 訂房 → `master-data/room-types.yaml`（房型與價錢、加購選項）、`master-data/room-list.yaml`（房號清單：room_type、floor、number；排房／建議房號時使用）
   - 訂餐 → `master-data/menu.yaml`（若有）
   - 場地 → `master-data/venues.yaml`（若有）
3. **產出**：
   - **確認單**：寫入 `orders/<id>-確認單.md`，摘要訂單編號、類型、日期、對應 master data 項目（如房型、房價）、總價或備註，供客戶確認。
   - **任務**：寫入 `orders/<id>-任務.md`，列出後續待辦（如：確認房況、回覆客戶、登記入住等），可含簡易清單。

## 產出格式約定

- 皆為 Markdown，可含 YAML front matter（如 `order_id`, `generated_at`）。
- 數字與金額與 master data 一致；若需求中有預算，可在確認單中註明「客戶預算」與「建議方案」是否相符。
