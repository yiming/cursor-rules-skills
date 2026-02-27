# 飯店訂單結構契約

本專案以 .md / .yaml 管理飯店訂單，不使用程式或資料庫。
操作 `orders/` 或 `master-data/` 下的檔案時，遵守以下約定。

## 路徑約定

- master data → `master-data/{類型}.yaml`（如 room-types.yaml, room-list.yaml）
- 需求檔 → `orders/{id}.req.md`（檔名即訂單 id）
- 產出 → `orders/{id}-確認單.md`、`orders/{id}-任務.md`
- 報表 → `analyse/`

## .req.md 必備 YAML front matter

- `type`：訂房 | 訂餐 | 場地（必填）
- `date`：訂單日期 YYYY-MM-DD（必填；訂房=入住日、訂餐=用餐日、場地=活動日）
- `id`：與檔名一致，格式為「類型碼 + YYMMDD + 序號」（必填）
  - 類型碼：BR=訂房、DR=訂餐、VR=場地
  - 範例：`BR250315-001`（訂房、2025-03-15、第 1 單）
- `guest`：訂房人／聯絡人姓名（必填）

front matter 下方為自由格式需求描述。

## 禁止事項

- 不得覆蓋或修改 `*.req.md`
- 金額必須與 master-data 一致，不得自行編造價格
- 建議房號必須存在於 room-list.yaml

## Skills

當使用者要求「依訂單產生確認單」「產生任務清單」或處理 orders/*.req.md 時，
請參照 `.cursor/skills/hotel-order-from-req/SKILL.md` 的流程執行。
