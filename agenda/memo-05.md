# 比較與說明

- 請分別說明:
  - claude skill / cursor skill : 是不是一樣的東西?
  - claude AGENTS.md / cursor Rules : 有什麼區別? 
  - cursor rules有什麼獨特性, 是claude AGENTS.md 無法取代, 或絕對優秀的?
- 簡單比較: Cursor Rules/Skills, Claude Skills, Github Copilot Custom Instructions
- 是否: 目前市面上相關機制, 幾乎以Claude Skills為基礎, 基本上相容, 但作法略有不同


---

## 1. Claude Skill vs Cursor Skill：是不是一樣的東西？

**不是一樣的東西，但解決類似的問題。**


| 面向          | Cursor Skill                                 | Claude Code Skill                                      |
| ----------- | -------------------------------------------- | ------------------------------------------------------ |
| 存放位置        | `.cursor/skills/{name}/SKILL.md`             | 由 Claude Code 內建機制註冊（如 `/commit`），或透過 CLAUDE.md 中的指示模擬 |
| 觸發方式        | 在 AGENTS.md 或 Rules 中寫「請參照 skill 執行」，AI 自行讀取 | 使用者輸入 `/skill-name`，系統自動展開為完整 prompt                   |
| 本質          | 一份 Markdown SOP 文件，AI 讀取後照做                  | 一段預先定義的 prompt 模板，由系統注入對話                              |
| 自訂彈性        | 高——任何 .md 檔案都可以當 skill，自由定義步驟與模板             | 低——目前主要是官方內建 skill（如 commit、review-pr），使用者較難自訂         |
| 與 Rules 的關係 | Rules 負責「何時」觸發，Skill 負責「怎麼做」                 | 沒有獨立的 Rules 層；觸發邏輯寫在 CLAUDE.md 裡                       |


**結論**：概念相似（都是「可重複使用的標準作業流程」），但 Cursor Skill 是純文件驅動、使用者完全自訂；Claude Code Skill 是系統內建機制，自訂空間較小。

---

## 2. Claude AGENTS.md vs Cursor Rules：有什麼區別？


| 面向   | Claude AGENTS.md           | Cursor Rules (.mdc)                                      |
| ---- | -------------------------- | -------------------------------------------------------- |
| 檔案格式 | 純 Markdown，放在專案根目錄或子目錄     | Markdown + YAML front matter（`.mdc`），放在 `.cursor/rules/` |
| 作用範圍 | 該目錄及其子目錄下的所有操作             | 透過 `globs` 精確指定適用的檔案路徑模式                                 |
| 啟用條件 | 永遠啟用（只要在對應目錄工作）            | `alwaysApply: true/false` + glob 匹配，可做到條件式啟用             |
| 層級繼承 | 支援——子目錄的 AGENTS.md 會疊加父目錄的 | 不支援目錄繼承；每條 rule 獨立宣告自己的 globs                            |
| 典型用途 | 專案整體約定（編碼風格、架構規則、禁止事項）     | 針對特定檔案類型或目錄的精準規則                                         |


**結論**：兩者都是「給 AI 的專案級指令」，但 Cursor Rules 多了 **條件式觸發（globs）** 的能力，而 AGENTS.md 靠的是 **目錄層級繼承**。

---

## 3. Cursor Rules 有什麼獨特性是 AGENTS.md 無法取代的？

### (a) Glob 條件觸發 — 最大優勢

```yaml
globs: orders/**/*.md,master-data/**
alwaysApply: false
```

只有當使用者操作的檔案匹配 glob 時，規則才會被注入。AGENTS.md 做不到這種「按檔案路徑自動開關」的精準控制——它要嘛整個目錄啟用，要嘛不啟用。

### (b) Rules + Skills 的分層架構

Cursor 把「何時套用」（Rules）和「怎麼做」（Skills）拆開：

- Rule 說：「當碰到 `orders/*.req.md` 時，去執行 hotel-order-from-req skill」
- Skill 說：「讀需求 → 載入 master data → 計算 → 產出確認單與任務」

這種分層讓同一個 Skill 可以被多條 Rules 在不同情境下觸發，複用性更好。

AGENTS.md 是扁平結構，所有指令寫在同一份文件裡，沒有這種觸發/執行的分離機制。

### (c) 不影響 Claude Code 使用者

Cursor Rules 放在 `.cursor/` 目錄下，只有 Cursor 會讀取。如果團隊同時有人用 Claude Code，這些 `.cursor/` 檔案不會干擾 Claude Code 的行為（反之亦然，AGENTS.md 也不會影響 Cursor）。兩者可以共存。

### 總結一句話

> Cursor Rules 的核心優勢是 **glob 條件觸發 + Rules/Skills 分層**，適合需要「不同檔案套用不同規則」的專案。AGENTS.md 更簡單直接，適合「整個專案統一規則」的場景。兩者可以在同一個 repo 中並存互補。

---

## 4. 三大機制簡單比較


| 面向       | Cursor Rules/Skills                                 | Claude Code (CLAUDE.md / Skills) | GitHub Copilot Custom Instructions |
| -------- | --------------------------------------------------- | -------------------------------- | ---------------------------------- |
| 設定檔      | `.cursor/rules/*.mdc` + `.cursor/skills/*/SKILL.md` | `CLAUDE.md` / `AGENTS.md`（任意目錄）  | `.github/copilot-instructions.md`  |
| 格式       | Markdown + YAML front matter                        | 純 Markdown                       | 純 Markdown                         |
| 觸發方式     | glob 匹配自動注入                                         | 目錄層級自動生效；Skill 靠 `/command` 手動觸發 | 全專案永遠生效，無條件式觸發                     |
| 自訂 Skill | 完全自訂（任何 .md 都行）                                     | 有限（主要是內建 skill，可在 CLAUDE.md 模擬）  | 不支援獨立的 skill 概念                    |
| 條件式規則    | `globs` + `alwaysApply`                             | 靠目錄放置位置控制                        | 不支援                                |
| 複雜度      | 高（最靈活）                                              | 中（簡單但有層級）                        | 低（一檔搞定）                            |
| 版控友善     | 是（`.cursor/` 目錄）                                    | 是（直接在 repo 根目錄）                  | 是（`.github/` 目錄）                   |


> **一句話**：Copilot 是「一份全域備忘錄」，Claude 是「按目錄分層的規則書」，Cursor 是「按檔案精準派發的 SOP 系統」。

---

## 5. 各家都支援 CLAUDE.md — 但 Skills 相容嗎？操作方式相同嗎？

### 現狀：CLAUDE.md 已成為事實標準

各主要 AI 編輯器目前都會讀取 `CLAUDE.md`：


| 工具             | 讀取 CLAUDE.md | 自己的指令檔                            |
| -------------- | ------------ | --------------------------------- |
| Claude Code    | 原生           | `CLAUDE.md` / `AGENTS.md`         |
| Cursor         | 是            | `.cursor/rules/*.mdc`             |
| Windsurf       | 是            | `.windsurfrules`                  |
| Cline          | 是            | `.clinerules`                     |
| GitHub Copilot | 否（有自己的）      | `.github/copilot-instructions.md` |


所以 `CLAUDE.md` 裡寫的規則（編碼風格、架構約定、禁止事項），確實可以跨工具共用。

### Skills 相容嗎？

**規則內容相容，但 Skill 機制不相容。**


| 層次                 | 相容性    | 說明                                                              |
| ------------------ | ------ | --------------------------------------------------------------- |
| 規則內容（CLAUDE.md 本文） | ✓ 相容   | 「不得修改 *.req.md」「金額必須與 master-data 一致」— 這些純文字指令，任何工具都能理解         |
| Skill 定義（SOP 步驟）   | △ 部分相容 | Skill 的 Markdown 內容本身是通用的，但各家存放位置和觸發方式不同                        |
| Skill 觸發機制         | ✗ 不相容  | Cursor 靠 `.mdc` 的 glob 自動觸發；Claude Code 靠 `/command`；其他工具沒有對應機制 |


舉例：本專案的 `.cursor/skills/hotel-order-from-req/SKILL.md`

- **在 Cursor**：由 `hotel-orders.mdc` 的 glob 自動觸發，使用者不需手動操作
- **在 Claude Code**：需要在 `AGENTS.md` 裡寫「請參照此檔案執行」，或使用者手動提示 AI 去讀取
- **在 Copilot**：完全無法自動觸發，使用者必須自己貼上或提及該 Skill 內容

### 操作方式相同嗎？

**不相同。** 三種典型操作模式：


| 模式       | 代表工具        | 使用者怎麼觸發 Skill                          |
| -------- | ----------- | -------------------------------------- |
| **自動派發** | Cursor      | 不需要——碰到匹配的檔案，Rules 自動注入對應 Skill        |
| **指令觸發** | Claude Code | 使用者輸入 `/skill-name`，或在 CLAUDE.md 中預先綁定 |
| **手動提示** | Copilot、其他  | 使用者自己在對話中說「請照 XX 流程做」                  |


### 結論

> **CLAUDE.md 的「規則」已是事實標準，各家基本相容。但「Skill」機制（自動觸發 SOP）尚未標準化——各家存放位置不同、觸發方式不同、無法直接互通。**
>
> 實務建議：
>
> - 共通規則 → 寫在 `CLAUDE.md`，所有工具都能讀
> - Skill 內容 → 寫成獨立 `.md` 檔案（如 `skills/xxx.md`），內容本身是通用的
> - 觸發機制 → 各工具各自設定（Cursor 用 `.mdc` glob、Claude 用 `AGENTS.md` 引用、Copilot 靠手動）

---

## 6. 設計哲學比較：Cursor vs Claude Code

| | Cursor | Claude Code |
|---|---|---|
| 設計哲學 | **隱式觸發、檔案驅動** — 規則根據你碰了哪些檔案自動套用，使用者不需要主動呼叫 | **顯式觸發、目錄/指令驅動** — CLAUDE.md 按目錄層級永遠生效，Skill 靠 `/command` 手動呼叫 |
| 一言以蔽之 | 自動、靈活 | 標準、可控 |

### 硬幣的另一面

- Cursor 的「自動靈活」也意味著 **不透明** — 使用者不一定知道當下哪些 rules 被觸發了
- Claude Code 的「標準可控」也意味著 **可預測** — CLAUDE.md 永遠生效，不會有「漏觸發」的問題

### 一句話總結

> **Cursor 追求「對的時機自動做對的事」；Claude Code 追求「規則永遠在、行為可預期」。**

---

## 7. 對新手友善度 & 跨工具實際相容性

### 對 Novice Users 來說

| | Cursor | Claude Code |
|---|---|---|
| 新手體驗 | **不用學、不用知道** — 團隊寫好 Rules/Skills 後，新人只要打開檔案，規則自動套用，完全無感 | **需要知道有什麼可用** — 新人必須知道「有哪些 `/command`」或「AGENTS.md 裡寫了什麼」，才會去使用 |
| 類比 | 像自動駕駛 — 上車就走，不用懂引擎 | 像手排車 — 功能強大，但你要知道怎麼換檔 |
| 適合場景 | 團隊有 SOP 專家維護 Rules，其他人只管做事 | 團隊成員都有一定技術素養，願意了解工具 |

> **結論**：如果團隊目標是「讓新人零學習成本就能遵守 SOP」，Cursor 的隱式觸發明顯更友善。

### 本專案的實證：用 Cursor 規範寫的，Claude 也能正確使用

本專案的規則體系完全是照 Cursor 的規範建立的：
- `.cursor/rules/hotel-orders.mdc` — Cursor 專用的條件式規則
- `.cursor/skills/hotel-order-from-req/SKILL.md` — Cursor 專用的 Skill

但 Claude Code 一樣能正確運作，原因是：

1. **AGENTS.md 做了橋接** — 專案根目錄的 `AGENTS.md` 把相同的規則內容複製了一份，並且明確寫了「請參照 `.cursor/skills/hotel-order-from-req/SKILL.md` 的流程執行」
2. **Skill 內容本身是通用的** — SKILL.md 裡的步驟（讀需求 → 載入 master data → 計算 → 產出）是純 Markdown 描述，不依賴任何 Cursor 專屬 API，Claude Code 讀了一樣能照做
3. **規則本質是「給 AI 看的文字」** — 不管哪個工具，底層都是把 Markdown 文字塞進 LLM 的 context，所以只要 AI 能讀到，就能遵守

### 所以跨工具的實務做法是

```
寫一次 Skill 內容（通用 .md）
    ├── Cursor：用 .mdc glob 自動觸發 ← 新手無感
    ├── Claude Code：用 AGENTS.md 引用 ← 自動生效，但要知道規則存在
    └── Copilot/其他：使用者手動提及 ← 最原始
```

> **一句話**：Skill 的「內容」天生跨工具相容，差別只在「誰幫你自動觸發」。Cursor 全自動，Claude 半自動（靠 AGENTS.md），其他靠手動。

