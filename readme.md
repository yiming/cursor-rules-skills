# Cursor Rules + Skills 飯店訂單範例

用 Cursor 的 Rules 與 Skills，以 .md / .yaml 管理飯店訂單（需求→確認單→任務），不寫程式、不用資料庫。

## 資料夾

| 資料夾 | 用途 |
|--------|------|
| `.cursor/rules/` | **Rule（結構契約）**：定義路徑、命名、必填欄位、禁止事項。AI 碰到匹配檔案時自動注入。 |
| `.cursor/skills/` | **Skill（工作流程）**：定義執行步驟、產出模板、計算邏輯、品質檢查。使用者主動調用。 |
| `master-data/` | 房型、房號等主資料（YAML） |
| `orders/` | `{id}.req.md` 需求檔 → 產出 `{id}-確認單.md`、`{id}-任務.md` |
| `analyse/` | 日報與統計產出 |
| `agenda/` | 備忘與規劃紀錄 |

## 使用方式

1. 在 `orders/` 新增 `{id}.req.md`（id 格式：類型碼 + YYMMDD + 序號，如 `BR250315-001`）
2. 填寫 YAML front matter（type、date、id）與需求描述
3. 對 AI 說「依這張訂單產生確認單與任務清單」

Rule 自動提供結構約束，Skill 按模板產出標準化的確認單與任務清單。
