## .cursor\rules\hotel-orders.mdc

為什麼上方有: 
- 下拉選項: 
    - always Apply
    - apply intelligently
    - apply to specific files
    - apply manually
- 副檔名: .mdc 是什麼意思?
- 如何調用這個rule? 有哪些機制?
- 能夠定義各個: 流程, 文件格式嗎?

## .cursor\skills\hotel-order-from-req\SKILL.md

- 為什麼這個文件跟RULES看起來是一樣的工作?
- 它們要怎麼協同工作?
- 如果是這樣各做各的, 並沒有起到相互搭配的效果

我感覺這個階段的成果, 沒有有效設計出一套有生產力的AI訂房RULE/SKILL,
我希望有更好的範例, 現在的東西實在無法說服我。
還是說, 我對:rules/skills有不切實際的期望與想像?

---

# 回答與改寫方案

## Q1：.mdc 與下拉選項

`.mdc` = **M**ark**d**own **C**onfiguration，Cursor 專用的 Rule 檔案格式。
上方下拉選項控制 **Rule 何時被自動注入 AI 的對話上下文**：

| 選項 | 對應 front matter | 行為 |
|------|-------------------|------|
| Always Apply | `alwaysApply: true` | 每次對話都注入，不管開什麼檔 |
| Apply to specific files | `alwaysApply: false` + `globs` | 對話涉及匹配檔案時才注入 |
| Apply intelligently | Cursor 自行判斷 | AI 根據語意決定是否相關 |
| Apply manually | 使用者手動 @ 引用 | 明確提及時才注入 |

目前設定 `globs: orders/*.req.md` 代表：只有操作訂單需求檔時，Rule 才會被自動注入。

**可以定義流程和格式嗎？** 可以，但 Rule 適合定義「結構契約」（什麼該有、什麼不准），
而具體的「工作流程 + 產出格式」應交給 Skill。

## Q2：Rule 與 Skill 為何看起來做一樣的事？

**這正是目前最大的問題。** 兩者內容高度重疊——都在描述「檔案放哪、叫什麼、產出什麼」。
你的直覺完全正確：這樣各做各的，沒有互補效果。

正確分工：

| | Rule（自動注入的背景知識） | Skill（主動調用的工作流程） |
|--|--|--|
| 何時作用 | AI 碰到匹配檔案時自動「知道規矩」 | 你說「產生確認單」時才執行 |
| 該寫什麼 | 結構契約：命名、欄位、路徑、禁止事項 | 執行步驟 + **產出模板** + 計算邏輯 + 品質檢查 |
| 不該寫什麼 | 不寫「怎麼做」 | 不重複「檔案放哪」（Rule 已管） |

## Q3：期望不切實際嗎？

不是。期望合理，是先前的設計沒切開職責。具體問題：

1. **Rule 太像說明書** — 列了結構但沒有強制約束（缺必填/禁止條款）
2. **Skill 太像 Rule 的重述** — 步驟只有泛泛三行，沒有具體指引
3. **缺少產出模板** — 最致命。AI 每次要「猜」確認單長什麼樣，品質不穩定
4. **兩者沒互補** — Rule 管「是什麼」，Skill 管「怎麼做到什麼程度」，目前混在一起

## 改寫方案

只改兩個檔：

### Rule → 精簡為純契約（~15 行）

砍掉所有產出步驟描述，只留：路徑約定、YAML 必填欄位、明確禁止事項。
讓 AI 碰到訂單檔時自動知道「規矩」，但不干涉「怎麼做」。

### Skill → 加入模板與計算邏輯（~65 行）

加入完整的**確認單模板**與**任務清單模板**——AI 按模板填值，不用每次猜格式。
加入計算公式（房價 × 晚數 + 加購）、品質檢查（金額可反算、房型容量 ≥ 人數）。

### 預期效果對比

| | 改寫前 | 改寫後 |
|--|--|--|
| 說「產生確認單」 | AI 自由發揮，每次格式不同 | AI 照模板填值，格式穩定一致 |
| 金額正確性 | 碰運氣 | Skill 定義計算公式 + 品質檢查 |
| Rule vs Skill | 重複說同樣的事 | Rule 管結構，Skill 管流程，不重疊 |
| 擴充到訂餐/場地 | 要重寫大段 | 在 Skill 加 type 分支 + 對應模板 |

---

## Cursor 與 Claude Code 相容性

### SKILL.md 格式差異與修正

兩者都用 `SKILL.md` + YAML frontmatter（name, description），核心格式相容。
但 Claude Code 有更嚴格的要求，已依此修正：

| 項目 | Claude Code 要求 | 修正內容 |
|------|------------------|----------|
| 觸發條件位置 | **必須全放 description**（body 觸發後才載入，放 body 無法幫助觸發） | 將「何時使用」內容搬進 description，刪除 body 中的該段落 |
| 寫作語態 | 祈使句（imperative form） | body 統一改為祈使句 |
| description 內容 | 必須包含 WHAT + WHEN + 具體觸發詞 | 加入三種觸發情境與關鍵詞 |

### Rule 機制差異

| Cursor | Claude Code |
|--------|-------------|
| `.cursor/rules/*.mdc`（含 globs 觸發） | **`AGENTS.md`**（放在專案根目錄或子目錄） |

`.mdc` 是 Cursor 專屬格式，Claude Code 不認。
已新增根目錄 `AGENTS.md`，內容與 Rule 的結構契約一致，讓 Claude Code 也能讀取。

### 現在的檔案對應

| 用途 | Cursor 讀取 | Claude Code 讀取 |
|------|------------|-----------------|
| 結構契約（自動注入） | `.cursor/rules/hotel-orders.mdc` | `AGENTS.md` |
| 工作流程（主動調用） | `.cursor/skills/hotel-order-from-req/SKILL.md` | 同一份 `SKILL.md`（透過 `AGENTS.md` 引用，不需複製） |

---

## 第一次測試前的修正

### 補上 `guest` 欄位

訂房需要真實姓名。已在 Rule、AGENTS、Skill 三處同步新增：
- YAML front matter 新增必填欄位 `guest`（訂房人姓名）
- 確認單模板加入「訂房人」列
- 任務清單的回覆項加入客戶姓名

### 測試訂單

| 檔案 | 訂房人 | 情境 | 預期房型 | 預期金額 |
|------|--------|------|----------|----------|
| `BR250420-001.req.md` | 陳美玲 | 2人2晚，預算 6500 | STD-D | 6400（預算內） |
| `BR250420-002.req.md` | 林志偉 | 4人3晚+停車+早餐，預算 16000 | FAM-Q | ≥16500（超預算） |

兩張單刻意設計不同：一張剛好在預算內，一張略超預算，用來驗證 Skill 的計算與預算比對。

詳細操作步驟見 **memo-04**。
