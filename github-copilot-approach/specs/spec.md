# UU大飯店訂單管理系統 — GitHub Copilot 版規格書

> 本文件為 GitHub Copilot Agent 版的實作規格。
> 參考來源：Cursor 版（根目錄）與 Claude Code 版（`pure-claude-approach/`）。

---

## 一、專案定位

以「UU大飯店」訂單管理為場景，展示 GitHub Copilot 的六種客製化原語（Instructions、Skills、Prompts、Agents、Hooks、AGENTS.md）如何協同運作。

**核心原則**：

- 無程式碼、無資料庫——全部以 `.md` / `.yaml` 管理
- Master Data 為唯一定價來源，金額必須可追溯
- `*.req.md` 為客戶原始需求，不得修改
- 三種訂單類型共用同一套命名規則與產出格式

---

## 二、訂單類型

| 代碼 | 類型 | 範例 ID | 說明 |
|------|------|---------|------|
| BR | 訂房 Booking Room | BR250315-001 | 住宿預訂 |
| DR | 訂餐 Dining Reservation | DR250420-001 | 餐飲預訂（含低消檢查） |
| VR | 場地 Venue Reservation | VR250420-001 | 場地租借（含桌次安排、桌牌產出） |

ID 格式：`{類型碼}{YYMMDD}-{三位序號}`

---

## 三、Master Data

共用根目錄 `master-data/`（與 Cursor 版、Claude Code 版共用同一份）。

| 檔案 | 內容 | 來源 |
|------|------|------|
| `room-types.yaml` | 3 種房型（STD-D / DEL-T / FAM-Q）、定價、含早餐等選項 | 共用 |
| `room-list.yaml` | 20 間實體房間（房號→房型→樓層→備註） | 共用 |
| `menu.yaml` | 餐飲品項（5 主餐 + 5 飲品 + 4 點心）、低消 NT$300/人 | 需複製自 `pure-claude-approach/master-data/` |
| `venues.yaml` | 3 場地（Grand Ballroom / Rose Hall / Sky Terrace）、4 種宴會餐標、7 項加購 | 需複製自 `pure-claude-approach/master-data/` |

---

## 四、目錄結構

```
github-copilot-approach/
├── AGENTS.md                          # 全域契約（三工具共用格式）
├── master-data/                       # ← 待建立（symlink 或複製）
│   ├── room-types.yaml
│   ├── room-list.yaml
│   ├── menu.yaml
│   └── venues.yaml
├── orders/                            # ← 待建立
│   └── (*.req.md → *-確認單.md + *-任務.md)
├── analyse/                           # ← 待建立（月度報告產出）
├── specs/
│   ├── about-skills.md                # 三工具機制比較（已完成）
│   ├── spec.md                        # 本文件
│   ├── plan.md                        # 實作計畫
│   └── task.md                        # 任務追蹤
└── .github/
    ├── copilot-instructions.md        # ← 待建立（可選，或沿用 AGENTS.md）
    ├── instructions/
    │   ├── hotel-orders.instructions.md       # ✅ 已完成
    │   └── hotel-master-data.instructions.md  # ✅ 已完成
    ├── prompts/
    │   └── check-availability.prompt.md       # ✅ 已完成
    ├── agents/
    │   └── hotel-validator.agent.md           # ✅ 檔案與內容皆完成
    ├── hooks/                         # ← 待建立（Copilot 獨有）
    │   └── protect-readonly.json
    └── skills/
        ├── hotel-order-from-req/
        │   ├── SKILL.md                       # ✅ 流程完成（但引用了尚不存在的 task 模板）
        │   ├── assets/
        │   │   ├── br-confirmation.md         # ✅ 模板完成（含欄位佔位符）
        │   │   ├── dr-confirmation.md         # ✅ 模板完成（含低消提示）
        │   │   ├── vr-confirmation.md         # ✅ 模板完成（含桌次、後續步驟）
        │   │   ├── br-task.md                 # ❌ 缺少（SKILL.md 已引用但檔案不存在）
        │   │   ├── dr-task.md                 # ❌ 缺少（SKILL.md 已引用但檔案不存在）
        │   │   └── vr-task.md                 # ❌ 缺少（SKILL.md 已引用但檔案不存在）
        │   └── references/
        │       └── order-types.md             # ✅ 完成（三類訂單欄位說明、賓客名單模式）
        ├── hotel-check-availability/
        │   └── SKILL.md                       # ✅ 完成
        ├── hotel-generate-card-prompt/
        │   └── SKILL.md                       # ✅ 完成
        └── hotel-report/
            ├── SKILL.md                       # ✅ 完成
            └── assets/
                └── monthly-report.md          # ✅ 模板完成（含 Mermaid 圖表佔位符）
```

---

## 五、GitHub Copilot 六大原語對應

### 5.1 AGENTS.md（全域規則）

- **檔案**：根目錄 `AGENTS.md`（✅ 已完成）
- **內容**：路徑約定、req.md 格式契約、禁止事項、Skills 入口列表
- **載入時機**：每次對話自動載入

### 5.2 File Instructions（條件規則）

| 檔案 | applyTo | 用途 | 狀態 |
|------|---------|------|------|
| `hotel-orders.instructions.md` | `orders/**/*.md` | 訂單操作規則、命名、必備結構 | ✅ |
| `hotel-master-data.instructions.md` | `master-data/**/*.yaml` | Master Data 欄位說明、唯讀政策 | ✅ |

### 5.3 Skills（多步驟工作流程）

| Skill | 觸發方式 | 輸入 | 輸出 | 狀態 |
|-------|---------|------|------|------|
| `/hotel-order-from-req` | Slash 或 AI 自動 | `orders/{id}.req.md` | `{id}-確認單.md` + `{id}-任務.md` | ✅ 骨架完成 |
| `/hotel-check-availability` | Slash 或 AI 自動 | 日期範圍 | 可用房間清單 | ✅ |
| `/hotel-generate-card-prompt` | Slash | VR 訂單 ID | `{id}-card-prompt.md` | ✅ |
| `/hotel-report` | Slash | 年月（YYYY-MM） | `analyse/{YYYY-MM}-報告.md` | ✅ |

### 5.4 Prompts（單一任務模板）

| Prompt | 用途 | 狀態 |
|--------|------|------|
| `check-availability.prompt.md` | 快速查空房（agent mode） | ✅ |

### 5.5 Custom Agents（子 Agent）

| Agent | 角色 | Tools 限制 | user-invocable | 狀態 |
|-------|------|-----------|----------------|------|
| `hotel-validator` | 唯讀稽核員 | `read, search` | false | ✅ |

### 5.6 Hooks（生命週期鉤子）— Copilot 獨有

| Hook | 時機 | 用途 | 狀態 |
|------|------|------|------|
| `protect-readonly.json` | PreToolUse | 攔截對 `*.req.md` 和 `master-data/*` 的寫入操作 | ❌ 待建立 |

---

## 六、各訂單處理流程規格

### 6.1 訂房（BR）

```
輸入：orders/BR{YYMMDD}-{seq}.req.md
  ↓
讀取 master-data/room-types.yaml + room-list.yaml
  ↓
匹配房型（依人數、預算、偏好）
  ↓
計算費用 = 房費/晚 × 晚數 + 選項（早餐附加費 × 人數 × 晚數 + 停車費 × 車數 × 晚數）
  ↓
從 room-list 選可用房號（須排除已訂房間）
  ↓
輸出：{id}-確認單.md + {id}-任務.md
  ↓
委派 hotel-validator 驗證（金額、房號、容量）
```

**自查要點**：
- 金額可追溯至 room-types.yaml 單價
- 建議房號存在於 room-list.yaml
- 人數不超過房型容量

### 6.2 訂餐（DR）

```
輸入：orders/DR{YYMMDD}-{seq}.req.md
  ↓
讀取 master-data/menu.yaml
  ↓
匹配品項（依人數、預算、偏好、飲食限制）
  ↓
低消檢查：總金額 ÷ 人數 ≥ NT$300/人
  ├─ 未達標 → 建議追加品項至達標
  └─ 已達標 → 繼續
  ↓
處理特殊需求（素食、過敏）
  ↓
輸出：{id}-確認單.md + {id}-任務.md
  ↓
委派 hotel-validator 驗證（單價、品項、低消）
```

**自查要點**：
- 品項單價與 menu.yaml 一致
- 低消 NT$300/人 達標
- 特殊飲食需求已標注

### 6.3 場地（VR）

```
輸入：orders/VR{YYMMDD}-{seq}.req.md + 賓客名單（三種模式）
  ↓
讀取 master-data/venues.yaml
  ↓
匹配場地（依人數、活動類型、座位形式）
  ↓
計算費用 = 場地費 + 宴會餐標 × 人數 + 加購項目
  ↓
最低消費檢查（NT$15,000）
  ↓
桌次安排演算法：
  ├─ VIP 桌（8 人/桌）：priority-1 群組
  ├─ 一般桌（10 人/桌）：同群組同桌、依序填滿
  └─ 素食標記：同桌素食 ≥ 3 人 → 整桌標記為素食桌
  ↓
輸出：{id}-確認單.md + {id}-任務.md + {id}-名單.yaml
  ↓
委派 hotel-validator 驗證
  ↓
提示使用者執行 /hotel-generate-card-prompt {id}
```

**賓客名單三種模式**：
1. 外部檔案引用（req.md 指定路徑）
2. 內嵌於 req.md 的 YAML 區塊
3. 無名單（僅依人數計算桌次，不產出桌牌）

**自查要點**：
- 場地容量足夠
- 每桌人數不超標（VIP ≤ 8, 一般 ≤ 10）
- 費用明細可追溯至 venues.yaml
- 最低消費達標

### 6.4 桌牌 Prompt 產出

```
輸入：orders/{VR-id}-確認單.md（含桌次表）
  ↓
讀取桌次與賓客名單
  ↓
依規模決定桌牌策略：
  ├─ ≤ 50 人：每人一張個人座位牌
  └─ > 50 人：每桌一張桌次牌
  ↓
選擇風格（中國水墨 / 現代簡約 / 歐式古典 / 花園自然）
  ↓
輸出：{id}-card-prompt.md
```

### 6.5 查詢空房

```
輸入：日期範圍（check-in ～ check-out）
  ↓
讀取 room-types.yaml + room-list.yaml
  ↓
掃描 orders/*-確認單.md，找日期重疊的訂房
  ↓
輸出：依房型分組的可用房間清單
```

### 6.6 月度營運報告

```
輸入：年月（YYYY-MM）
  ↓
掃描 orders/*-確認單.md，篩選該月
  ↓
依 BR/DR/VR 分類聚合：
  ├─ 訂單數量、總營收、平均訂單金額
  ├─ BR：入住率（已訂間晚 ÷ 總可用間晚）、ADR
  ├─ DR：人均消費
  └─ VR：場地使用率、加購排行
  ↓
產出含 Mermaid 圖表（圓餅圖 + 長條圖）的報告
  ↓
輸出：analyse/{YYYY-MM}-報告.md
```

---

## 七、產出檔案格式規範

### 7.1 確認單（`{id}-確認單.md`）

```yaml
---
order_id: BR250315-001
type: 訂房
generated_at: 2025-03-15T10:30:00+08:00
---
```

必備區段：
1. 訂單摘要表（ID、聯絡人、日期、金額合計）
2. 費用明細（逐項列出，含單價來源）
3. 特殊需求與備註

### 7.2 任務清單（`{id}-任務.md`）

必備區段：
1. 訂單狀態（待確認 / 已確認 / 取消）
2. 工作項目（依訂單類型展開，含 checkbox）
3. 負責部門標注

### 7.3 月度報告（`analyse/{YYYY-MM}-報告.md`）

必備區段：
1. 報告期間與訂單總覽
2. 分類統計（BR / DR / VR）
3. Mermaid 圓餅圖（營收占比）
4. Mermaid 長條圖（每日訂單量）
5. 經營提醒與建議

---

## 八、與其他版本的關鍵差異

| 面向 | Cursor 版 | Claude Code 版 | GitHub Copilot 版（本版） |
|------|-----------|----------------|--------------------------|
| 全域規則 | `.cursor/rules/*.mdc` | `CLAUDE.md` 分層繼承 | `AGENTS.md` + File Instructions |
| Skills 拆分 | 統一一個 skill | 拆成 3 個獨立 skill（BR/DR/VR） | 統一為 `hotel-order-from-req`（內部分支） |
| 稽核機制 | Skill 內建自查清單 | 獨立 auditor 子代理（haiku） | `hotel-validator` 子 Agent（唯讀） |
| 強制防護 | 無（靠 LLM 遵守） | `settings.json` deny 規則 | **Hooks**（PreToolUse 攔截，確定性防護） |
| AI 自動觸發 | 有（description） | 無此機制 | 有（description） |
| 模型指定 | 無 | 有（per agent） | 有（`.prompt.md` 可指定） |
| Prompt 模板 | `.cursor/commands/` | 用 skills 替代 | `.github/prompts/`（可指定 agent mode） |

### 本版重點展示的 Copilot 獨有能力

1. **Hooks 強制防護**：`PreToolUse` 鉤子攔截對唯讀檔案的寫入，比 Instructions 的「請勿修改」更可靠
2. **Custom Agent + tool 白名單**：`hotel-validator` 只有 `read, search` 權限，物理隔離寫入能力
3. **漸進式載入**：Skills 三階段載入（發現→指令→資源），節省 context 消耗
4. **Prompt agent mode**：`check-availability.prompt.md` 指定 `agent: "agent"` 模式執行

---

## 九、待完成項目

| # | 項目 | 優先級 | 說明 |
|---|------|--------|------|
| 1 | 建立 `master-data/` | 高 | 從 `pure-claude-approach/master-data/` 複製 4 份 YAML |
| 2 | 建立 `orders/` 目錄 | 高 | 空目錄，準備放測試訂單 |
| 3 | 建立 `analyse/` 目錄 | 高 | 空目錄，報告產出位置 |
| 4 | 建立 Hooks | 高 | `.github/hooks/protect-readonly.json`（Copilot 獨有亮點） |
| 5 | 補建 3 份 task 模板 | 高 | `br-task.md`、`dr-task.md`、`vr-task.md`（SKILL.md 已引用但檔案不存在） |
| 6 | 建立測試訂單 | 中 | 各類型至少一筆 `.req.md` 用於驗證 |
| 7 | 端到端測試 | 中 | 在 VS Code + Copilot Agent 中實際執行全流程 |
| 8 | 撰寫 plan.md / task.md | 低 | 記錄實作順序與進度 |
