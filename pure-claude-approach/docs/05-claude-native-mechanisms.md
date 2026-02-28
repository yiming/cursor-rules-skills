# Claude Code 原生機制詳解

本文件深入說明本專案如何運用 Claude Code 的九大原生機制，以及每個機制的技術細節。

## 1. `.claude/settings.json` — 信任邊界

### 機制說明

`settings.json` 是 Claude Code 的系統層級配置，定義了工具的使用權限。與 CLAUDE.md 的文字指示不同，`settings.json` 的限制由系統強制執行。

### 本專案用法

```json
{
  "permissions": {
    "deny": ["Edit(orders/*.req.md)", "Edit(master-data/*)"]
  }
}
```

### 效果

- AI 嘗試修改 `*.req.md` → 系統直接拒絕，不是 AI 自己選擇不做
- AI 嘗試修改 `master-data/*` → 同上

### 與 Cursor 的差異

Cursor 只能在 `.mdc` 裡寫「不得修改」，但 AI 理論上可以忽略。Claude 的 `deny` 是系統層面的硬限制。

---

## 2. `.claude/skills/*/SKILL.md` — 指令式 Skill

### 機制說明

Claude Code 的 Skill 是帶有 YAML frontmatter 的 Markdown 檔案，放在 `.claude/skills/{name}/SKILL.md`。設定 `user-invocable: true` 後，使用者可以用 `/{name}` 直接觸發。

### 本專案用法

```yaml
---
name: process-order
description: 從訂房需求檔產生確認單與任務清單
argument-hint: "[order-id]"
user-invocable: true
---
```

觸發：`/process-order BR250315-001`

### Frontmatter 關鍵欄位

| 欄位 | 用途 |
|------|------|
| `name` | 指令名稱（`/name`） |
| `description` | Claude 判斷是否自動啟用的依據 |
| `argument-hint` | 自動完成提示 |
| `user-invocable` | 是否出現在 `/` 選單 |
| `model` | 指定執行模型 |
| `context: fork` | 在子代理中執行 |

---

## 3. 動態上下文注入（`` !`command` ``）

### 機制說明

SKILL.md 中可以使用 `` !`shell-command` `` 語法。Skill 載入時，系統會先執行 shell 指令，將輸出注入到 Skill 的上下文中。

### 本專案用法（check-availability）

```markdown
## 現有房間
!`cat master-data/room-list.yaml`

## 已被預訂的房間
!`grep -r "建議房號" orders/*-確認單.md 2>/dev/null || echo "目前無訂單"`
```

### 效果

Skill 載入時，AI 已經看到：
- 所有房間的即時清單
- 所有已預訂的房號

不需要額外指示 AI「先去讀某個檔案」。

### 與 Cursor 的差異

Cursor 的 SKILL.md 是純靜態文件，無法在載入時執行指令。

---

## 4. `.claude/agents/*.md` — 子代理定義

### 機制說明

Claude Code 支援在 `.claude/agents/` 目錄下定義子代理。每個子代理有自己的：
- 角色描述
- 可用工具（tools）
- 指定模型（model）
- 可用技能（skills）
- 執行限制（maxTurns）

### 本專案用法

**order-clerk.md**：
```yaml
---
name: order-clerk
tools: Read, Write, Glob, Grep
model: sonnet
skills:
  - process-order
---
```

**auditor.md**：
```yaml
---
name: auditor
tools: Read, Grep, Glob
model: haiku
maxTurns: 5
---
```

### 委派方式

主 Claude 可以說：「把這張訂單交給 order-clerk 處理」，order-clerk 在自己的 context window 裡獨立完成任務。

---

## 5. 模型分配

### 機制說明

不同子代理可以使用不同的模型，根據任務需求選擇最合適的。

### 本專案用法

| 子代理 | 模型 | 原因 |
|--------|------|------|
| order-clerk | sonnet | 需要理解需求、計算匹配、產出文件 |
| auditor | haiku | 只需比對數字、查表驗證 |

### 效益

- 成本優化：簡單驗證任務不需要用昂貴的大模型
- 速度優化：haiku 比 sonnet 快得多
- 能力匹配：每個任務用最適合的模型

---

## 6. 目錄級 CLAUDE.md 繼承

### 機制說明

Claude Code 會自動讀取當前操作目錄路徑上所有的 CLAUDE.md。根目錄的 CLAUDE.md 永遠生效，子目錄的 CLAUDE.md 在操作該目錄時額外生效。

### 本專案用法

```
CLAUDE.md                    ← 永遠生效（全域政策）
├── master-data/CLAUDE.md    ← 操作 master-data/ 時生效
└── orders/CLAUDE.md         ← 操作 orders/ 時生效
```

### 與 Cursor 的差異

Cursor 用 glob 模式精確匹配檔案：`globs: orders/**/*.md`
Claude 用目錄位置自然界定作用域，不需要額外配置。

---

## 7. 模板分離

### 機制說明

SKILL.md 可以引用同目錄下的其他檔案作為模板或參考資料。

### 本專案用法

```
.claude/skills/process-order/
├── SKILL.md              ← 流程邏輯
└── templates/
    ├── confirmation.md   ← 確認單模板
    └── task-list.md      ← 任務清單模板
```

### 效益

- 模板獨立維護，修改模板不影響流程
- 流程獨立維護，修改流程不影響模板
- 模板可以被其他 Skill 複用

---

## 8. 參數化 Skill

### 機制說明

Skill 支援 `$ARGUMENTS` 變數替換，以及按索引存取：`$1`、`$2`。

### 本專案用法

```
/process-order BR250315-001
→ $ARGUMENTS = "BR250315-001"
→ 讀取 orders/$ARGUMENTS.req.md

/check-availability 2025-04-20 2025-04-22
→ $1 = "2025-04-20"
→ $2 = "2025-04-22"
```

---

## 9. 角色權限分離

### 機制說明

每個子代理的 `tools` 欄位限制了它能使用的工具。結合 `settings.json` 的 deny 規則，形成多層防護。

### 本專案用法

```
order-clerk:  Read, Write, Glob, Grep  → 能產出檔案
auditor:      Read, Grep, Glob        → 只能讀取驗證

settings.json:
  deny Edit(orders/*.req.md)          → 所有人都不能改需求檔
  deny Edit(master-data/*)            → 所有人都不能改主資料
```

### 三層防護

1. **CLAUDE.md**（指導層）：告訴 AI 不該做什麼
2. **agents 的 tools**（角色層）：限制子代理能用什麼工具
3. **settings.json deny**（系統層）：強制禁止特定操作

從「建議」到「限制」到「禁止」，逐層加強。
