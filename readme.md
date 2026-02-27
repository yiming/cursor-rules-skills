# Cursor Rules + Skills 飯店訂單範例

用 Cursor 的 rules 與 skills，以 .md / .yaml 管理飯店訂單（需求→確認單→任務），不寫程式、不用資料庫。

## 資料夾

- **.cursor/rules**、**.cursor/skills**：規則與技能
- **master-data/**：房型、房號等主資料（YAML）
- **orders/**：`*.req.md` 需求檔，產出 `*-確認單.md`、`*-任務.md`
- **analyse/**：日報產出  
- **agenda/**：備忘

## 使用

在 Cursor 開啟專案，在 `orders/` 新增 `*.req.md`，對 AI 說「依這張訂單產生確認單／任務清單」即可產出。
