# Cursor vs Claude Code 關於Skills相關機制對照整理

本文整理 Cursor 與 Claude Code 在「規則、指令、技能、擴充」上的差異與對應關係，並納入本專案（飯店訂單、pure-claude-approach）的實際用法。

---

## 一、Cursor 的 Context 與擴充結構

### 1.1 Rules（規則）

- **位置**：`.cursor/rules/`
- **格式**：`.mdc`（專用 frontmatter）或 `.md`
- **範例**：`react-patterns.mdc`、`api-guidelines.md`，可再分子目錄如 `frontend/components.md`

Frontmatter 範例（.mdc）：

```yaml
---
description: "API 開發規則"   # AI 判斷是否相關
globs: "src/api/**/*.ts"      # 編輯到匹配檔案時載入
alwaysApply: false            # 是否永遠載入
---
```

**觸發方式（4 種）**：`alwaysApply` 永遠載入、globs 匹配、description 讓 AI 判斷、手動 `@規則名` 引用。

### 1.2 Commands（指令）

- **位置**：`.cursor/commands/`
- **格式**：每支一個 `.md` 檔案，內容為給 AI 的步驟或檢查清單
- **觸發**：多半透過 `/` 選單選擇（例如 `/run-all-tests-and-fix`）

與 Skills 的關係：兩者都能用 `/` 觸發；Commands 偏「固定流程、清單式」，Skills 偏「有觸發條件與完整流程」。若只靠 Rules + Skills，很多情境下 Commands 的獨特性不明顯，可視需求選擇是否使用。

### 1.3 Skills（技能）

- **位置**：`.cursor/skills/`
- **結構**：每個 skill 一個目錄，內含 `SKILL.md`，可帶 `scripts/`、`references/`、`assets/`
- **觸發**：`/skill-name`（例如 `/hotel-report`），或自然語言符合 skill 描述時由 AI 觸發

**注意**：`@` 在 Cursor 裡是用來**提及檔案或程式**，不是用來叫 skill；叫 skill 用 `/` 或自然語言。

### 1.4 其他

- **其他**：Subagents、Semantic Search、@ Mentions（檔案引用）、MCP（外部服務整合）等。

```
.cursor/
├── rules/           # 規則（.mdc）
├── commands/        # 指令（.md）
└── skills/          # 技能（SKILL.md）
```

> **注**：Cursor 的 Plugin 系統仍在演進中，具體結構請以官方文件為準。

---

## 二、Claude Code 的擴充機制

參考：<https://code.claude.com/docs/en/memory>

### 2.1 擴充插入代理迴圈的不同部分

| 功能 | 作用 | 何時使用 | 範例 |
|------|------|----------|------|
| CLAUDE.md | 每次對話載入的持久上下文 | 專案約定、「始終執行 X」 | 使用 pnpm、提交前跑測試 |
| Skill | 可重複使用的知識與可呼叫工作流程 | 可重複任務、參考文件 | /review 程式碼審查、API 文件 skill |
| Subagent | 隔離執行、回傳摘要 | 上下文隔離、並行、專門工作者 | 讀多檔只回關鍵發現 |
| MCP | 連外部服務 | 外部資料或操作 | 查 DB、發 Slack、控瀏覽器 |
| Hook | 事件上跑的確定性腳本 | 可預測自動化、不經 LLM | 每次編輯後跑 ESLint |

### 2.2 CLAUDE.md 與 AGENTS.md

- **CLAUDE.md**：給 Claude 的「指令文件」— 定義規則、政策、上下文；啟動時自動載入。
- **AGENTS.md**：與 CLAUDE.md **完全等價**，是別名。Claude Code 會同時找兩者並載入，內容合併。歷史上前者叫 AGENTS.md，後來主推 CLAUDE.md，保留兩者為向後相容。**實際差異：零。**

### 2.3 CLAUDE.md 可放的位置（載入範圍由廣到窄）

| 位置 | 載入範圍 | 用途 |
|------|----------|------|
| `~/.claude/CLAUDE.md` | 所有專案 | 個人全域偏好（如「回覆用繁體中文」） |
| `/專案根/CLAUDE.md` 或 `/專案根/.claude/CLAUDE.md` | 整個專案 | 專案級規則（如飯店經理政策） |
| `/專案根/子目錄/CLAUDE.md` | 該子目錄 | 目錄級規則（如 orders 訂單格式契約） |

**載入邏輯**：從根往下繼承；子目錄的 CLAUDE.md **疊加**在父層之上，不覆蓋。例如 `master-data/CLAUDE.md` 可補充「此目錄唯讀」，不影響根層的團隊政策。

### 2.4 開放格式 AGENTS.md（agents.md）

[agents.md](https://agents.md/) 是社群推動的**開放格式**：在專案根（或子目錄）放 AGENTS.md，用標準 Markdown 寫給 coding agents 的說明（建置、測試、程式風格等），多種工具都可讀。Cursor 列在該站的支援清單中。與 Claude Code 的關係：**檔案層級相容** — 同一份根目錄或子目錄的 AGENTS.md，在 Claude Code 裡會被當成 CLAUDE.md 的別名載入；在 Cursor 中若產品有實作，會當成專案說明讀取。

---

## 三、Rules 對照：Cursor vs Claude Code

概念很像，但機制不同。

### 3.1 Cursor Rules

- **目錄**：`.cursor/rules/`（單一層，扁平）
- **格式**：`.mdc`（description、globs、alwaysApply）
- **觸發**：4 種 — alwaysApply、globs 匹配、description 讓 AI 判斷、手動 @ 引用

### 3.2 Claude Code Rules

- **主指令**：`CLAUDE.md` 或 `.claude/CLAUDE.md`（永遠載入）
- **條件規則**：`.claude/rules/*.md`，可用 frontmatter `paths`（glob）限定
- **觸發**：2 種 — 無 `paths` 則永遠載入；有 `paths` 則操作到匹配檔案時載入

`.claude/rules/` 範例：

```yaml
---
paths:
  - "src/api/**/*.ts"
---
# API Development Rules
- All API endpoints must include input validation
- Use the standard error response format
```

### 3.3 核心差異

| 項目 | Cursor Rules | Claude Code Rules |
|------|--------------|-------------------|
| 檔案格式 | .mdc（專用） | .md（標準 markdown） |
| 觸發模式 | 4 種（always / glob / AI 判斷 / 手動） | 2 種（always / glob） |
| AI 自動判斷 | ✓ description 讓 AI 決定是否載入 | ✗ 無此機制 |
| 放置位置 | .cursor/rules/ 單一層 | CLAUDE.md 可放任意子目錄 |
| 目錄繼承 | ✗ 無 | ✓ 子目錄 CLAUDE.md 自動疊加父層 |

### 3.4 最關鍵的差異

- **Cursor**：規則是「扁平」的 — 所有規則在 `.cursor/rules/` 下，靠 glob 與 description 決定範圍。
- **Claude Code**：規則是「分層」的 — 任意子目錄可放 CLAUDE.md，形成繼承鏈：

```
根/CLAUDE.md              → 全局政策（永遠載入）
根/.claude/rules/         → 條件規則（glob 觸發）
根/orders/CLAUDE.md       → 目錄級規則（進入該目錄時生效）
根/master-data/CLAUDE.md  → 另一目錄級規則
```

**本專案**（pure-claude-approach）未使用 `.claude/rules/`，改用**子目錄 CLAUDE.md** 做分層 — 因飯店規則天然按目錄分（orders、master-data 各一套）。若規則按「主題」分（code-style、testing、security），則 `.claude/rules/` 較適合。

---

## 四、本專案用法摘要

### 4.1 Cursor 側（飯店主線）

- **Rules**：`.cursor/rules/`（如 hotel-always.mdc、hotel-orders.mdc、hotel-master-data.mdc）+ 根目錄 `AGENTS.md`（路徑約定、.req.md 格式、禁止事項、Skills 入口）。
- **Skills**：`.cursor/skills/` — hotel-order-from-req、hotel-check-availability、hotel-generate-card-prompt、hotel-report 等；觸發用 `/skill-name` 或自然語言。
- **Commands**：未使用；若需要「一鍵處理本單」「一致性檢查」等固定流程，可考慮加 `.cursor/commands/`。

### 4.2 Claude Code 側（pure-claude-approach）

```
pure-claude-approach/
├── CLAUDE.md              # 根層：飯店經理政策 + 團隊分工表
├── master-data/
│   └── CLAUDE.md          # 此目錄唯讀
└── orders/
    └── CLAUDE.md          # 訂單格式契約 + 命名規則
```

即「目錄級規則繼承」：每一層 CLAUDE.md 各司其職，不需把全部規則塞在根層。

**雙工具共用策略**：若同一個 repo 同時用 Cursor 與 Claude Code，可把**通用政策**放在根目錄的 CLAUDE.md（或 AGENTS.md），Claude Code 專有的能力（Skills、Agents、Settings）放在 `.claude/`。這樣用 Cursor 開同專案時，根層的規則仍有機會被讀到並影響行為。

---

## 五、檔案與目錄支援對照與注意事項

### 5.1 支援矩陣（以官方文件為準）

| 檔案／目錄 | Cursor | Claude Code |
|------------|--------|-------------|
| AGENTS.md（根或子目錄） | ✓ 會讀取作為專案上下文（[agents.md](https://agents.md/) 列為支援；繼承行為與 Claude Code 不完全相同，以 Cursor 文件為準） | ✓ 支援（與 CLAUDE.md 別名，載入邏輯相同） |
| CLAUDE.md（根或子目錄） | ✓ 會讀取作為專案上下文（實測確認；繼承行為以官方文件為準） | ✓ 支援（主要指令檔） |
| .cursor/rules/*.mdc | ✓ 專屬 | ✗ 不讀 |
| .claude/rules/*.md | ✗ 不讀 | ✓ 專屬 |
| .claude/skills/ | ✗ 不讀 | ✓ 專屬 |
| .claude/agents/ | ✗ 不讀 | ✓ 專屬 |

### 5.2 「支援」與「有影響」的差別

- **官方支援**：產品文件明確說「會載入某檔案作為 project instructions / system context」。只有看到這類說明，才能說「某工具支援某檔案」。
- **有影響**：LLM 的產出取決於當下 context 裡有什麼。專案裡任何檔案只要被讀進 context（被 @、被搜尋、被工具掃到等），就可能影響回覆。因此「放了某個檔會不會有影響」幾乎永遠是「可能會有」，與產品是否宣稱「支援」是兩回事。

實測中 Cursor 確實會讀取根目錄的 CLAUDE.md／AGENTS.md 作為專案上下文。但其載入行為（如子目錄繼承鏈、是否疊加）與 Claude Code 不完全相同，細節請以 Cursor 官方文件為準。

---

## 六、參考連結

- Claude Code 記憶與規則：<https://code.claude.com/docs/en/memory>
- Path-specific rules：<https://code.claude.com/docs/en/memory#path-specific-rules>
- 開放格式 AGENTS.md：<https://agents.md/>
