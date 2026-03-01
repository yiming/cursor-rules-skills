# GitHub Copilot Agent 客製化機制說明

> 本文說明 GitHub Copilot（VS Code）如何透過各種客製化檔案支援 agent skills，
> 包含預設目錄結構、各機制的用途，以及與 Cursor、Claude Code 的對比。

---

## 一、整體架構

GitHub Copilot 的 agent 客製化機制圍繞 **`.github/`** 目錄展開，提供六種原語（primitive）：

| 原語 | 檔案格式 | 目錄 | 用途 |
|------|----------|------|------|
| Workspace Instructions | `copilot-instructions.md` 或 `AGENTS.md` | `.github/` 或根目錄 | 全專案通用規則，永遠載入 |
| File Instructions | `*.instructions.md` | `.github/instructions/` | 依檔案類型或任務觸發的規則 |
| Prompts | `*.prompt.md` | `.github/prompts/` | 單一任務的可重複提示模板 |
| Hooks | `*.json` | `.github/hooks/` | 在 agent 生命週期執行確定性 shell 指令 |
| Custom Agents | `*.agent.md` | `.github/agents/` | 有特定工具限制與角色的子 agent |
| Skills | `SKILL.md` | `.github/skills/<name>/` | 按需載入的多步驟工作流程（本文重點） |

預設**最小**目錄結構如下：

```
.github/
├── copilot-instructions.md   # 或根目錄的 AGENTS.md（擇一）
├── instructions/
│   └── *.instructions.md
├── prompts/
│   └── *.prompt.md
├── agents/
│   └── *.agent.md
├── hooks/
│   └── *.json
└── skills/
    └── <skill-name>/
        ├── SKILL.md          # 必要，名稱須與資料夾一致
        ├── scripts/          # 可選：可執行腳本
        ├── references/       # 可選：參考文件（按需載入）
        └── assets/           # 可選：模板、樣板
```

---

## 二、Skills（技能）

### 2.1 何時使用

Skills 適用於**可重複、按需觸發**的多步驟工作流程，且需要附帶資源（腳本、模板、參考文件）。

- 有多個明確步驟的流程 → Skills
- 單一任務、有輸入參數 → Prompts
- 需要 context 隔離或不同工具限制的子流程 → Custom Agents
- 適用於所有工作的通用規則 → Instructions

### 2.2 目錄結構

```
.github/skills/<skill-name>/
├── SKILL.md           # 必要
├── scripts/           # 可執行程式碼
├── references/        # 按需載入的參考文件
└── assets/            # 模板、樣板檔案
```

Skills 也可放在以下替代路徑（三者等價）：
- `.github/skills/<name>/`
- `.agents/skills/<name>/`
- `.claude/skills/<name>/`

個人層級則放在 `~/.copilot/skills/<name>/`（不需版控，跨專案共用）。

### 2.3 SKILL.md 格式

```yaml
---
name: skill-name              # 必填；1–64 字元；小寫英數 + 連字號；須與資料夾名稱一致
description: 'Use when...'   # 必填；最多 1024 字元；關鍵字豐富才能被 AI 自動發現
argument-hint: '...'         # 選填；在 slash 叫用時顯示的提示
user-invocable: true          # 選填；是否出現在 / 選單（預設 true）
disable-model-invocation: false # 選填；禁止 AI 自動觸發（預設 false）
---
```

**invocable / auto-load 組合說明**：

| 設定 | 出現在 `/` 選單 | AI 自動載入 |
|------|----------------|------------|
| 兩者皆省略（預設） | ✓ | ✓ |
| `user-invocable: false` | ✗ | ✓ |
| `disable-model-invocation: true` | ✓ | ✗ |
| 兩者皆設 | ✗ | ✗ |

### 2.4 觸發方式

1. **Slash 指令**：在 chat 輸入 `/skill-name`
2. **AI 自動觸發**：對話中的任務符合 `description` 的關鍵字時，AI 主動載入
3. **明確 @ 引用**：`@<skill-name>` 呼叫（需在 agent 模式下）

### 2.5 漸進式載入

AI 讀取 skill 採三階段，節省 context 消耗：

1. **發現**（~100 tokens）：讀取 `name` 與 `description`，判斷是否相關
2. **指令**（最多 5000 tokens）：相關時載入 `SKILL.md` 本文
3. **資源**：只在 SKILL.md 中有明確引用時才載入 `references/`、`scripts/` 等

> 建議 `SKILL.md` 本文保持在 **500 行以內**，超出的內容移到 `references/` 子目錄。

### 2.6 飯店訂單範例結構

以本專案為例，若移植到 GitHub Copilot 方式：

```
.github/
├── copilot-instructions.md        # 或根目錄 AGENTS.md（全域政策）
└── skills/
    ├── hotel-order-from-req/
    │   ├── SKILL.md               # 主流程：讀 req.md → 產確認單 + 任務清單
    │   ├── references/
    │   │   └── order-types.md     # BR / DR / VR 三種訂單分支說明
    │   └── assets/
    │       ├── br-confirmation.md # 訂房確認單模板
    │       ├── dr-confirmation.md # 訂餐確認單模板
    │       └── vr-confirmation.md # 場地確認單模板
    ├── hotel-check-availability/
    │   └── SKILL.md               # 查空房
    ├── hotel-generate-card-prompt/
    │   └── SKILL.md               # 產桌牌 prompt
    └── hotel-report/
        ├── SKILL.md               # 月度營運報告
        └── assets/
            └── monthly-report.md  # 報告模板
```

---

## 三、其他相關機制

### 3.1 Workspace Instructions（全域規則）

對應 Cursor 的 `alwaysApply` rules 或 Claude Code 的根層 `CLAUDE.md`。

- **位置**：`.github/copilot-instructions.md` 或根目錄 `AGENTS.md`（擇一）
- **載入時機**：每次對話永遠載入
- **適合放**：專案規範、訂單格式契約、禁止事項、命名慣例

> `AGENTS.md` 是開放格式（[agents.md](https://agents.md/)），三種工具（Copilot、Cursor、Claude Code）都支援，適合多工具共用同一份根規則。

**monorepo 分層**：可在子目錄放 `AGENTS.md`，最近的優先：

```
/AGENTS.md                 # 全域預設
/orders/AGENTS.md          # 訂單子目錄規則（覆蓋根層）
/master-data/AGENTS.md     # master-data 子目錄規則（僅此目錄有效）
```

### 3.2 File Instructions（檔案型規則）

對應 Cursor 的 `globs` rules（非 alwaysApply）。

- **位置**：`.github/instructions/*.instructions.md`
- **frontmatter**：
  - `description`：讓 AI 按需發現（on-demand）
  - `applyTo`：glob 匹配到特定檔案時自動附加
- **適合放**：特定語言慣例、特定目錄的操作規則

範例：

```yaml
---
description: "Use when editing or creating hotel order files (.req.md, confirmation docs)"
applyTo: "orders/**/*.md"
---
# 訂單操作規則
- 不得覆蓋 *.req.md
- 金額必須比對 master-data/room-types.yaml
```

### 3.3 Prompts（提示模板）

- **位置**：`.github/prompts/*.prompt.md`
- **觸發**：`/` 選單選取，或 `Chat: Run Prompt...` 指令
- **適合放**：單一、明確的一次性任務（如「產生README」、「生成測試用例」）
- **與 Skills 的差異**：Skills = 多步驟 + 附帶資源；Prompts = 單一任務 + 參數化輸入

### 3.4 Custom Agents（自訂 Agent）

- **位置**：`.github/agents/*.agent.md`
- **特點**：可限制 tools 集合、設定特定模型、定義子 agent 角色
- **適合用於**：context 隔離（子 agent 只回傳摘要）、不同步驟需不同工具限制的流程

```yaml
---
description: "Use when: validating hotel order files for completeness and pricing accuracy"
tools: [read, search]    # 唯讀，不允許編輯
user-invocable: false    # 只作為子 agent 被調用
---
```

### 3.5 Hooks（生命週期鉤子）

- **位置**：`.github/hooks/*.json`
- **特點**：在 agent 生命週期（`PreToolUse` / `PostToolUse`）執行確定性 shell 指令
- **與 Instructions 的差異**：Instructions 是「引導」（非確定性）；Hooks 是「強制執行」（確定性）
- **適合用於**：每次儲存後自動 lint、阻擋高風險工具操作、強制 approval 流程

---

## 四、與 Cursor / Claude Code 的對比

| 面向 | GitHub Copilot | Cursor | Claude Code |
|------|---------------|--------|-------------|
| 全域規則 | `.github/copilot-instructions.md` 或根目錄 `AGENTS.md` | `.cursor/rules/*.mdc`（`alwaysApply`）+ `AGENTS.md` | `CLAUDE.md`（根或子目錄） |
| 條件規則（glob） | `.github/instructions/*.instructions.md`（`applyTo`） | `.cursor/rules/*.mdc`（`globs`） | `.claude/rules/*.md`（`paths`） |
| AI 自動判斷觸發 | ✓ `description`（instructions + skills） | ✓ `description`（rules） | ✗ 無此機制 |
| 目錄分層繼承 | ✓ `AGENTS.md` 子目錄覆蓋 | ✗ 扁平 `.cursor/rules/` | ✓ 子目錄 `CLAUDE.md` 疊加 |
| Skills | `.github/skills/<name>/SKILL.md`（`/` 或 AI 觸發） | `.cursor/skills/<name>/SKILL.md`（`/` 觸發） | `.claude/skills/<name>/SKILL.md` |
| 子 Agent | `.github/agents/*.agent.md` | ✗ 無原生支援 | `.claude/agents/*.md` |
| Hooks | `.github/hooks/*.json` | ✗ 無（可用 Commands 替代） | Hooks（shell 腳本） |
| Prompts（slash 指令） | `.github/prompts/*.prompt.md` | `.cursor/commands/*.md` | ✗（可用 skills 替代） |

**關鍵觀察**：
- GitHub Copilot 的 skills 路徑（`.github/skills/`）與 Claude Code（`.claude/skills/`）雖不同，但可同時共存於同一 repo，互不干擾。
- `AGENTS.md` 是三工具共同支援的開放格式，適合放跨工具共用的全域規則。
- GitHub Copilot 的 Custom Agents 比 Cursor 更接近 Claude Code 的 Subagents，提供 context 隔離與工具限制。

---

## 五、決策速查

```
需求                              → 建議機制
──────────────────────────────────────────────────────
專案通用規則（始終生效）           → copilot-instructions.md / AGENTS.md
特定檔案類型的規則                 → .instructions.md（applyTo）
依任務按需觸發的規則               → .instructions.md（description）
可重複的多步驟工作流程 + 附帶資源  → Skills（SKILL.md）
單一聚焦任務 + 參數化輸入          → Prompts（.prompt.md）
需要 context 隔離或工具限制        → Custom Agents（.agent.md）
確定性自動化（非 AI 決策）         → Hooks（.json）
```

---

## 六、GitHub Copilot 獨有機制

以下三項是目前 Cursor 與 Claude Code **均未提供**的原生功能：

### 6.1 Hooks（生命週期鉤子）

`.github/hooks/*.json`

Hooks 在 agent 工具呼叫的生命週期注入**確定性 shell 指令**，完全不經過 LLM 判斷。

```
PreToolUse  → 工具執行前（可阻擋、要求 approval）
PostToolUse → 工具執行後（可執行自動格式化、lint）
```

與 Instructions 的本質差異：

| | Instructions | Hooks |
|---|---|---|
| 執行方式 | LLM 解讀後「引導」行為 | shell 指令直接執行 |
| 確定性 | 非確定（LLM 可能忽略） | 確定（一定發生） |
| 可阻擋工具 | ✗ | ✓（PreToolUse 可 block） |
| 適合場景 | 風格偏好、操作慣例 | 強制審核、自動 lint、安全防護 |

**本專案可用範例**：當 agent 想修改 `*.req.md` 時，`PreToolUse` hook 直接攔截並拒絕，比在 Instructions 裡寫「不得覆蓋」更可靠。

### 6.2 Custom Agents（子 Agent 原語）

`.github/agents/*.agent.md`

Cursor 沒有對應的原生子 agent 原語（只能靠自然語言描述讓 AI 分工）；Claude Code 有 `.claude/agents/`，但機制相近。GitHub Copilot 的 Custom Agents 可精確控制：

- **工具白名單**：`tools: [read, search]` 讓稽核 agent 永遠唯讀
- **模型選擇**：對不同子任務指定不同模型（快速 / 強力）
- **隱藏子 agent**：`user-invocable: false` 只作為主 agent 的「工人」，不暴露給使用者
- **Handoffs**：明確定義從哪個 agent 交棒給哪個 agent

使用情境：主 skill（hotel-order-from-req）產出確認單後，自動把「稽核」這個動作委派給 `hotel-validator` agent，後者只能讀取、不能修改，確保驗證步驟不會意外改到檔案。

### 6.3 Prompts（`.prompt.md`）的進階特性

Cursor 的 Commands（`.cursor/commands/*.md`）和 GitHub Copilot 的 Prompts 表面相似，但 Copilot 多出幾項能力：

| 特性 | GitHub Copilot `.prompt.md` | Cursor Commands |
|------|----------------------------|-----------------|
| 指定模型 | ✓（`model: GPT-5 (copilot)`，支援 fallback 陣列） | ✗ |
| 指定 agent 模式 | ✓（`agent: ask / agent / plan / 自訂 agent`） | ✗ |
| 限定 tools | ✓（`tools: [read, search]`） | ✗ |
| 引用 MCP 工具 | ✓（`tools: [myserver/*]`） | ✗ |
| Context 引用語法 | `[file](./path)` + `#tool:name` | 自然語言描述 |

---

## 七、三工具深度比較

### 7.1 設計哲學

| | GitHub Copilot | Cursor | Claude Code |
|---|---|---|---|
| **核心理念** | 「每種原語一個資料夾」，概念分離最清晰 | 「一切皆 rule」，極簡化，統一在 `.cursor/rules/` | 「目錄即規則」，層級繼承，規則隨資料走 |
| **規則觸發** | 3 種：glob (`applyTo`) / AI 判斷 (`description`) / always | 4 種：always / glob / AI 判斷 / 手動 `@` | 2 種：always / glob (`paths`) |
| **AI 自動發現** | ✓ instructions + skills 的 `description` | ✓ rules 的 `description` | ✗ 無此機制 |
| **規則分層** | `AGENTS.md` 子目錄覆蓋（淺層繼承） | ✗ 扁平，無分層 | ✓ 子目錄 `CLAUDE.md` 深層疊加 |

### 7.2 Skills 比較

三工具的 Skills 概念最為接近，但仍有細節差異：

| | GitHub Copilot | Cursor | Claude Code |
|---|---|---|---|
| **路徑** | `.github/skills/<name>/` | `.cursor/skills/<name>/` | `.claude/skills/<name>/` |
| **觸發** | `/` slash + AI 自動 | `/` slash（主要）+ 自然語言 | `/` slash + 自然語言 |
| **SKILL.md 格式** | 相同（name / description / argument-hint） | 相同 | 相同 |
| **跨工具共用** | ✓（三路徑可同時存在於同一 repo） | ✓ | ✓ |
| **`disable-model-invocation`** | ✓ 可關閉 AI 自動觸發 | 不明確 | 不明確 |

> **實務建議**：同一 repo 若要同時支援三種工具，把 SKILL.md 放在 `.github/skills/` 即可，Claude Code 和 Cursor 都能讀到（或各自在 `.claude/` / `.cursor/` 放一份）。

### 7.3 「本專案」三種方式對照

| 機制 | pure-claude-approach | Cursor 版（原有） | github-copilot-approach（本資料夾） |
|------|---------------------|-------------------|-------------------------------------|
| 全域規則 | `CLAUDE.md`（根 + 子目錄繼承） | `.cursor/rules/*.mdc`（alwaysApply）+ `AGENTS.md` | `AGENTS.md`（根）+ `.github/copilot-instructions.md` |
| 目錄型規則 | `orders/CLAUDE.md`、`master-data/CLAUDE.md` | `hotel-orders.mdc`（globs）、`hotel-master-data.mdc` | `.github/instructions/*.instructions.md`（applyTo） |
| Skills 觸發 | `/hotel-order-from-req`（Claude 內建） | `/hotel-order-from-req` | `/hotel-order-from-req` |
| 稽核邏輯 | CLAUDE.md 規則 + Subagents | Skill 內建自查清單 | `.github/agents/hotel-validator.agent.md`（唯讀子 agent） |
| 強制攔截 | ✗（靠 LLM 遵守） | ✗（靠 LLM 遵守） | ✓ `.github/hooks/`（PreToolUse，可阻擋修改 req.md） |
| 模型指定 | 無（依工具） | 無 | ✓ `.prompt.md` 可指定模型與 fallback |

### 7.4 選擇建議

```
只用 Cursor             → .cursor/rules/ + .cursor/skills/（最精簡）
只用 Claude Code        → CLAUDE.md 分層 + .claude/skills/（最自然）
只用 GitHub Copilot     → .github/ 全套（功能最完整，含 Hooks + Agents）
三工具共用同一 repo      → AGENTS.md（根）共用規則
                          + 各自的 skills 路徑（可並存）
                          + .github/hooks/ 做強制防護（Copilot 執行，其他工具無效但無害）
```

---

## 參考資料

- [VS Code Agent Skills](https://code.visualstudio.com/docs/copilot/customization/agent-skills)
- [Custom Instructions](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)
- [Prompt Files](https://code.visualstudio.com/docs/copilot/customization/prompt-files)
- [Custom Agents](https://code.visualstudio.com/docs/copilot/customization/custom-agents)
- [開放格式 AGENTS.md](https://agents.md/)

