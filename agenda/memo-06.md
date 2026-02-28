# memo-06：重構 pure-claude-approach — 從「換個格式寫規則」到「展示 Claude 的獨特理念」

## 問題診斷：現在的版本到底缺了什麼？

現在的 `pure-claude-approach` 本質上做了一件事：
**把 Cursor 的規則換個位置存，用純 Markdown 重寫了一遍。**

三層架構（CLAUDE.md → AGENTS.md → skills/）確實能跑，但它展示的只是
「Claude 也能做到 Cursor 做的事」——這恰好是最不 Claude 的定位。

真正的問題不是「沒有 `.claude/` 資料夾」，而是：

> **整個範例只展示了「規則遵循」這一個能力，
> 而 Claude Code 最獨特的能力——委派、分工、自主行動——完全沒有出現。**

### Claude Code 獨有的、其他工具做不到的事

| 能力 | 說明 | 現狀 |
|------|------|------|
| **子代理委派** | 把任務交給有特定角色、工具、權限邊界的子代理執行 | 未展示 |
| **信任邊界** | settings.json 定義誰能讀/寫什麼，把「建議不要改」變成「系統不讓改」 | 未展示 |
| **Skill 作為指令** | `/hotel-order` 就像 `/commit` 一樣，是一個可觸發、帶參數的動作 | 未展示 |
| **動態上下文** | Skill 載入時執行 shell 取得即時資料（如空房狀態） | 未展示 |
| **跨 session 記憶** | agent-memory/ 讓代理記住旅客偏好、歷史訂單 | 未展示 |
| **Hooks 生命週期** | 產出後自動觸發驗證腳本 | 未展示 |

而這些能力，恰好可以用飯店訂單這個小場景自然地展示。

---

## 核心理念轉變

### 從「規則書」到「團隊」

**現有版本的隱喻**：一本 SOP 手冊（CLAUDE.md + AGENTS.md + skills/）

**重構後的隱喻**：一個飯店前台團隊

```
                    ┌─────────────────────┐
                    │   飯店經理 (CLAUDE.md) │  ← 制定政策、分派任務
                    └────────┬────────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
     ┌────────────┐  ┌────────────┐  ┌────────────┐
     │  訂單處理員  │  │  資料驗證員  │  │  報表分析員  │
     │  (agent)   │  │  (agent)   │  │  (agent)   │
     └──────┬─────┘  └────────────┘  └────────────┘
            │
            ▼
     ┌────────────┐
     │  SOP 手冊   │  ← skill：怎麼處理訂單
     │  (skill)   │
     └────────────┘
```

這不只是換個檔案格式——是從 **「AI 讀規則做事」** 變成 **「AI 組建團隊做事」**。

**Cursor 的哲學**：寫好規則 → AI 自動遵守 → 產出結果
**Claude 的哲學**：定義角色 → 委派任務 → 各司其職 → 產出結果

---

## 重構方案

### 目標結構

```
pure-claude-approach/
│
├── CLAUDE.md                              ← 飯店經理：政策 + 團隊分工
│
├── .claude/                               ← 【新增】Claude 原生目錄
│   │
│   ├── settings.json                      ← 信任邊界：誰能改什麼
│   │
│   ├── skills/                            ← SOP 手冊（可用 /指令 觸發）
│   │   ├── process-order/
│   │   │   ├── SKILL.md                   ← 正規格式 + frontmatter
│   │   │   └── templates/
│   │   │       ├── confirmation.md        ← 確認單模板（從 SKILL.md 抽出）
│   │   │       └── task-list.md           ← 任務清單模板
│   │   └── check-availability/
│   │       └── SKILL.md                   ← 【新增】查空房（展示動態注入）
│   │
│   └── agents/                            ← 【新增】團隊成員
│       ├── order-clerk.md                 ← 訂單處理員
│       └── auditor.md                     ← 稽核員
│
├── master-data/
│   ├── CLAUDE.md                          ← 改名自 AGENTS.md
│   ├── room-types.yaml
│   └── room-list.yaml
│
├── orders/
│   ├── CLAUDE.md                          ← 改名自 AGENTS.md
│   └── ...
│
└── docs/
```

### 各部件詳解：為什麼要這樣設計

#### 1. `.claude/settings.json` — 信任邊界

現狀：CLAUDE.md 寫「不得修改 *.req.md」，但這只是「建議」，AI 可以無視。

```json
{
  "permissions": {
    "deny": ["Edit(orders/*.req.md)", "Edit(master-data/*)"]
  }
}
```

重構後：系統層級的強制保護。這是「規則」和「權限」的本質差異。

> **展示的理念**：Claude Code 區分「指導」（CLAUDE.md）和「強制」（settings.json）。
> 規則告訴 AI 該做什麼；權限控制 AI 能做什麼。兩者缺一不可。

#### 2. `.claude/skills/process-order/SKILL.md` — Skill 作為指令

現狀：`skills/hotel-order-from-req.md` 是一份被 CLAUDE.md 引用的文件。

```yaml
---
name: process-order
description: 從訂房需求檔產生確認單與任務清單
argument-hint: "[order-id]"
user-invocable: true
---

讀取 orders/$ARGUMENTS.req.md ...
```

重構後：使用者可以打 `/process-order BR250315-001` 直接觸發。
不再需要 CLAUDE.md 的「派發表」——Skill 本身就是入口。

> **展示的理念**：Cursor 的 Skill 需要 Rule 來觸發（被動），
> Claude 的 Skill 本身就是一個可呼叫的指令（主動）。
> 這是「文件」和「指令」的差異。

#### 3. `.claude/skills/check-availability/SKILL.md` — 動態上下文

```yaml
---
name: check-availability
description: 查詢指定日期的空房狀態
argument-hint: "[check-in] [check-out]"
user-invocable: true
---

## 現有房間
!`cat master-data/room-list.yaml`

## 已被預訂的房間（從現有訂單中掃描）
!`grep -r "建議房號" orders/*-確認單.md 2>/dev/null || echo "目前無訂單"`

請根據以上資料，列出 $1 至 $2 期間可用的房間。
```

> **展示的理念**：Skill 不只是靜態 SOP，它可以在載入時動態取得即時資料。
> 這是 Cursor Skill 做不到的——Cursor 的 SKILL.md 是純文件，
> Claude 的 SKILL.md 可以是「帶有即時上下文的指令」。

#### 4. `.claude/agents/order-clerk.md` — 子代理委派

```yaml
---
name: order-clerk
description: 飯店訂單處理員，負責從需求產生確認單與任務清單
tools: Read, Write, Glob, Grep
model: sonnet
skills:
  - process-order
---

你是 UU 大飯店的訂單處理員。

## 職責
- 收到訂單需求後，依照 process-order skill 的流程產出確認單與任務清單
- 嚴格遵守價格真實原則，所有金額必須可追溯到 master-data

## 限制
- 不得修改 *.req.md
- 不得修改 master-data/
- 產出前必須完成品質檢查
```

> **展示的理念**：子代理 = 角色 + 工具 + 權限 + 技能。
> 這不是「規則」，這是「一個有能力邊界的專業人員」。
>
> 主 Claude 可以說「把這張訂單交給 order-clerk 處理」，
> order-clerk 在自己的 context window 裡獨立完成任務。
> 這是 **委派**，不是「讀規則照做」。

#### 5. `.claude/agents/auditor.md` — 分工與制衡

```yaml
---
name: auditor
description: 驗證訂單確認單的金額與房號是否正確
tools: Read, Grep, Glob
model: haiku
maxTurns: 5
---

你是稽核員。你 **只能讀取**，不能修改任何檔案。

收到確認單路徑後：
1. 比對金額是否與 master-data/room-types.yaml 一致
2. 比對房號是否存在於 master-data/room-list.yaml
3. 確認房型容量 ≥ 入住人數
4. 回報：通過 / 不通過（附具體差異）
```

> **展示的理念**：處理員和稽核員是不同角色，有不同權限。
> 處理員能寫，稽核員只能讀。這是「職責分離」。
>
> 更重要的是：稽核員用 haiku（快且便宜），處理員用 sonnet（聰明）。
> Claude Code 可以為不同任務選擇不同模型——這也是獨特能力。

#### 6. AGENTS.md → CLAUDE.md

純改名，內容不變。用正式的子目錄 CLAUDE.md 機制。

#### 7. 更新根 CLAUDE.md

從「規則手冊 + 派發表」改為「團隊政策 + 分工表」：

```markdown
## 團隊分工

| 任務 | 負責人 | 觸發方式 |
|------|--------|---------|
| 處理訂單需求 | order-clerk（子代理） | 主動委派，或 `/process-order` |
| 驗證確認單 | auditor（子代理） | 處理完成後自動委派 |
| 查詢空房 | `/check-availability` | 使用者直接呼叫 |
```

---

## 重構前後的「使用體驗」對比

### 處理一張訂單

**重構前**（規則模式）：
```
使用者：「依這張訂單產生確認單」
Claude：（讀 CLAUDE.md → 找到派發表 → 讀 skill → 照做）
結果：一份確認單 + 一份任務清單
```

**重構後**（團隊模式）：
```
使用者：/process-order BR250315-001
Claude：（觸發 skill，或委派給 order-clerk）
  → order-clerk 在獨立 context 中執行
  → 產出確認單 + 任務清單
  → 自動委派 auditor 驗證
  → auditor 回報：✓ 通過
結果：一份確認單 + 一份任務清單 + 驗證報告
```

差異不在結果，在過程：**有分工、有委派、有制衡**。

### 查空房

**重構前**：做不到（沒有這個功能）

**重構後**：
```
使用者：/check-availability 2025-04-20 2025-04-22
Claude：（動態掃描現有訂單 + master-data → 列出可用房間）
```

Skill 載入時就已經抓到即時資料，不需要額外指示。

---

## 這個小例子能展示的 Claude 理念清單

| 理念 | 對應實作 | 對比 Cursor |
|------|---------|------------|
| 指令式 Skill | `/process-order [id]` | Cursor: 被動觸發 |
| 動態上下文注入 | `!`command`` 語法 | Cursor: 無此能力 |
| 子代理委派 | order-clerk, auditor | Cursor: 無此概念 |
| 角色權限分離 | agent 各有不同 tools | Cursor: 無 |
| 模型分配 | 處理用 sonnet, 稽核用 haiku | Cursor: 無 |
| 信任邊界 | settings.json deny | Cursor: 無（只有建議） |
| 目錄級規則繼承 | 子目錄 CLAUDE.md | Cursor: glob 觸發 |
| 模板分離 | skill 的 templates/ 子目錄 | Cursor: 寫在 SKILL.md 裡 |
| 參數化 Skill | $ARGUMENTS, $1, $2 | Cursor: 無 |

用一個「飯店訂單」的小場景，9 個 Claude 獨有理念全部自然呈現。

---

## 執行順序

1. 建立 `.claude/` 骨架（settings.json）
2. 遷移 + 重寫 skill（process-order + 模板分離）
3. 新增 skill（check-availability，展示動態注入）
4. 新增 agents（order-clerk, auditor）
5. 改名 AGENTS.md → CLAUDE.md
6. 重寫根 CLAUDE.md（從規則手冊 → 團隊政策）
7. 更新 readme.md
8. 更新 docs/（重寫 02-特性分析、03-系統架構、04-使用指引）
9. 新增 docs/05-claude-native-mechanisms.md
10. 刪除舊 `skills/` 目錄
11. 實測：用 `/process-order` 跑一次完整流程
