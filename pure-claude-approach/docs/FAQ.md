# FAQ

## Q: Cursor 的 .cursor 資料夾, 可以放哪些東西? 怎麼放? 放什麼? 怎麼用?

### 資料夾結構

```
.cursor/
├── rules/                    # 規則檔案（核心功能）
│   ├── general.mdc           # 支援 .mdc（推薦）或 .md
│   └── frontend/             # 可巢狀子資料夾
│       └── components.md
├── mcp.json                  # MCP 伺服器設定（專案層級）
└── skills/                   # 技能（遵循 Agent Skills 標準）
    └── skill-name/
        └── SKILL.md
```

### Rules 規則（`.cursor/rules/`）

支援 `.mdc`（Markdown + frontmatter，推薦）和 `.md` 兩種副檔名。

`.mdc` 檔案範例：

```markdown
---
description: "描述這個規則的用途與適用範圍"
alwaysApply: false
globs: ["src/**/*.tsx", "components/**/*.jsx"]
---

# 規則內容（Markdown）

這裡寫具體指示...
```

### 四種規則觸發模式

| 模式 | 說明 | frontmatter 設定 |
|------|------|-----------------|
| **Always Apply** | 每次對話都載入 | `alwaysApply: true` |
| **Apply Intelligently** | Agent 根據 `description` 自動判斷是否相關 | `alwaysApply: false`，有 `description` |
| **Apply to Specific Files** | 當開啟的檔案符合 glob 規則時觸發 | 設定 `globs` |
| **Apply Manually** | 使用者在對話中用 `@規則名` 手動引用 | 無特殊設定 |

### MCP 設定（`.cursor/mcp.json`）

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "@some/mcp-server"],
      "env": {
        "API_KEY": "your-key"
      }
    }
  }
}
```

也可放在全域位置 `~/.cursor/mcp.json`。

### 最佳實踐

- 每個規則檔建議**不超過 500 行**，引用檔案而非複製程式碼
- 多個聚焦的小規則優於一個大而全的規則
- 規則內容應包含：技術堆疊慣例、資料夾結構、命名規範、常見陷阱
- 使用 `globs` 精確限定規則的適用範圍，例如 `frontend/**/*.jsx`

---

## Q: Claude 有沒有具體的資料夾規範? 如 .claude?

有，Claude Code 有一套完整且明確的 `.claude` 資料夾規範，比 Cursor 更為豐富。

### 完整資料夾結構

```
.claude/
├── CLAUDE.md                 # 專案層級記憶/指示（每次對話自動載入）
├── settings.json             # 設定檔（權限、hooks 等）
├── settings.local.json       # 本地設定（不應提交到版控）
├── commands/                 # 舊版自訂斜線指令（仍支援，建議遷移到 skills）
│   └── review.md
├── skills/                   # 技能（推薦使用，取代 commands）
│   └── skill-name/
│       ├── SKILL.md          # 主要指示檔（必須）
│       ├── template.md       # 模板
│       ├── examples/         # 範例
│       │   └── sample.md
│       ├── references/       # 參考文件
│       │   └── api-docs.md
│       ├── scripts/          # 腳本
│       │   └── helper.py
│       └── assets/           # 素材
│           └── logo.png
├── agents/                   # 子代理定義
│   └── code-reviewer.md
├── agent-memory/             # 子代理持久記憶（project 範圍）
│   └── agent-name/
│       └── MEMORY.md
└── agent-memory-local/       # 子代理本地記憶（不提交版控）
    └── agent-name/
        └── MEMORY.md
```

全域層級（使用者目錄）：

```
~/.claude/
├── CLAUDE.md                 # 全域記憶（所有專案共用）
├── settings.json             # 全域設定
├── skills/                   # 個人技能（所有專案可用）
│   └── explain-code/
│       └── SKILL.md
├── agents/                   # 個人子代理
│   └── debugger.md
└── agent-memory/             # 子代理使用者層級記憶
    └── agent-name/
        └── MEMORY.md
```

### Q: 一共有哪些檔案格式?

| 檔案 | 格式 | 用途 |
|------|------|------|
| `CLAUDE.md` | Markdown | 專案記憶，每次對話自動載入 |
| `settings.json` | JSON | 權限、hooks、MCP 設定 |
| `settings.local.json` | JSON | 本地設定（不提交版控） |
| `skills/*/SKILL.md` | Markdown + YAML frontmatter | 技能定義 |
| `agents/*.md` | Markdown + YAML frontmatter | 子代理定義 |
| `commands/*.md` | Markdown | 舊版斜線指令 |
| `agent-memory/*/MEMORY.md` | Markdown | 子代理持久記憶 |

#### CLAUDE.md 放置位置與優先層級

| 位置 | 適用範圍 |
|------|---------|
| `~/.claude/CLAUDE.md` | 所有專案（個人偏好） |
| `專案根目錄/CLAUDE.md` | 當前專案 |
| `.claude/CLAUDE.md` | 當前專案（等同根目錄） |
| `子目錄/CLAUDE.md` | 編輯該子目錄檔案時載入 |

### Q: Skills 放哪裡? 格式要求?

#### 存放位置與優先級

| 位置 | 路徑 | 適用範圍 | 優先級 |
|------|------|---------|--------|
| 企業級 | 管理員設定 | 組織所有使用者 | 最高 |
| 個人級 | `~/.claude/skills/<name>/SKILL.md` | 所有專案 | 高 |
| 專案級 | `.claude/skills/<name>/SKILL.md` | 僅此專案 | 中 |
| 外掛級 | `<plugin>/skills/<name>/SKILL.md` | 外掛啟用處 | 低 |

#### SKILL.md 格式

```yaml
---
name: my-skill              # 斜線指令名稱（小寫、數字、連字號，最多 64 字元）
description: 技能描述        # Claude 用此判斷何時啟用（最多 200 字元）
argument-hint: "[arg]"      # 可選：自動完成提示
disable-model-invocation: false  # 可選：true = 只能手動 /name 觸發
user-invocable: true        # 可選：false = 從 / 選單隱藏
allowed-tools: Read, Grep   # 可選：限制可用工具
model: sonnet               # 可選：指定模型
context: fork               # 可選：在子代理中執行
agent: Explore              # 可選：指定子代理類型
---

# 技能指示內容（Markdown）

支援 !`command` 語法動態注入 shell 輸出
支援 $ARGUMENTS 變數替換
```

#### Frontmatter 欄位完整列表

| 欄位 | 必要 | 說明 |
|------|------|------|
| `name` | 否 | 顯示名稱，省略時用資料夾名 |
| `description` | 建議 | 技能用途描述，Claude 據此判斷自動載入 |
| `argument-hint` | 否 | 自動完成提示，如 `[filename]` |
| `disable-model-invocation` | 否 | `true` 時僅能手動觸發 |
| `user-invocable` | 否 | `false` 時從選單隱藏 |
| `allowed-tools` | 否 | 限制可用工具 |
| `model` | 否 | 指定使用的模型 |
| `context` | 否 | 設為 `fork` 在子代理中執行 |
| `agent` | 否 | 搭配 `context: fork` 指定子代理類型 |
| `hooks` | 否 | 技能生命週期鉤子 |

#### 字串替換變數

| 變數 | 說明 |
|------|------|
| `$ARGUMENTS` | 傳入的所有引數 |
| `$ARGUMENTS[N]` 或 `$N` | 按索引存取引數（從 0 開始） |
| `${CLAUDE_SESSION_ID}` | 當前 session ID |

#### 動態內容注入

使用 `` !`command` `` 語法可在技能載入前執行 shell 指令：

```yaml
---
name: pr-summary
description: 摘要 PR 變更
context: fork
---

PR diff: !`gh pr diff`
Changed files: !`gh pr diff --name-only`
```

---

## 附錄：Cursor 與 Claude Code 關鍵差異對照

| 面向 | Cursor `.cursor/` | Claude Code `.claude/` |
|------|-------------------|----------------------|
| 規則/記憶 | `rules/*.mdc` / `*.md` | `CLAUDE.md` |
| 技能 | `skills/*/SKILL.md` | `skills/*/SKILL.md`（功能更豐富） |
| 子代理 | 無原生支援 | `agents/*.md`（完整子代理系統） |
| MCP 設定 | `mcp.json` | `settings.json` 內設定 |
| 動態內容 | 無 | `` !`command` `` 語法 |
| 子代理記憶 | 無 | `agent-memory/` 持久化 |
| 觸發控制 | 4 種模式（Always/Intelligent/Glob/Manual） | `disable-model-invocation` + `user-invocable` |
