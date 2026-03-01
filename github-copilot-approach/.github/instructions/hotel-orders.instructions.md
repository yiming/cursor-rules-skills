---
description: "Use when editing, creating, or reviewing hotel order files (req.md, confirmation docs, task lists). Covers naming rules, required fields, and output format."
applyTo: "orders/**/*.md"
---

# 飯店訂單操作規則

## 命名規則

- 需求檔：`{id}.req.md`（唯讀，不得修改）
- 確認單：`{id}-確認單.md`
- 任務清單：`{id}-任務.md`
- ID 格式：`BR|DR|VR` + `YYMMDD` + `-` + 三位序號（例：`BR250420-001`）

## 確認單必備結構

1. YAML front matter（`order_id`、`generated_at`）
2. 訂單摘要表（id、guest、日期、金額合計）
3. 費用明細（依 master-data 計算，不得自行編造）
4. 備註或特殊需求

## 任務清單必備結構

1. 訂單狀態（待確認 / 已確認 / 取消）
2. 工作項目清單（依訂單類型展開）
3. 負責部門標注

## 禁止事項

- 不得覆蓋 `*.req.md`
- 金額須與 `master-data/` 下的 YAML 一致
- 房號須存在於 `master-data/room-list.yaml`
