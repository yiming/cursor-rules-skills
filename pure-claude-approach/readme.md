# 純 Claude 版：飯店訂單管理系統

將原本以 Cursor Rules/Skills 實作的飯店訂單系統，完全改用 Claude Code 原生機制重寫。

## 設計哲學

> **Cursor：「寫好規則 → AI 遵守 → 產出結果」**
> **Claude：「定義角色 → 委派任務 → 各司其職 → 產出結果」**

本版本展示的不是「AI 讀規則做事」，而是 **「AI 組建團隊做事」**。

## 九大 Claude 獨有能力展示

| 能力 | 對應實作 | Cursor 對比 |
|------|---------|------------|
| 指令式 Skill | `/process-order [id]` | Cursor: 被動觸發 |
| 動態上下文注入 | `` !`command` `` in check-availability | Cursor: 無此能力 |
| 子代理委派 | order-clerk, auditor | Cursor: 無此概念 |
| 角色權限分離 | agent 各有不同 tools | Cursor: 無 |
| 模型分配 | 處理用 sonnet, 稽核用 haiku | Cursor: 無 |
| 信任邊界 | settings.json deny | Cursor: 無（只有建議） |
| 目錄級規則繼承 | 子目錄 CLAUDE.md | Cursor: glob 觸發 |
| 模板分離 | skill 的 templates/ 子目錄 | Cursor: 寫在 SKILL.md 裡 |
| 參數化 Skill | $ARGUMENTS, $1, $2 | Cursor: 無 |

## 目錄結構

```
pure-claude-approach/
│
├── CLAUDE.md                              ← 飯店經理：政策 + 團隊分工
│
├── .claude/                               ← Claude 原生目錄
│   ├── settings.json                      ← 信任邊界：誰能改什麼
│   ├── skills/                            ← SOP 手冊（可用 /指令 觸發）
│   │   ├── process-order/
│   │   │   ├── SKILL.md                   ← 正規格式 + frontmatter
│   │   │   └── templates/
│   │   │       ├── confirmation.md        ← 確認單模板
│   │   │       └── task-list.md           ← 任務清單模板
│   │   ├── process-dining-order/
│   │   │   ├── SKILL.md                   ← 訂餐流程（含低消邏輯）
│   │   │   └── templates/
│   │   │       ├── confirmation.md
│   │   │       └── task-list.md
│   │   ├── process-venue-order/
│   │   │   ├── SKILL.md                   ← 場地流程（含桌次安排）
│   │   │   └── templates/
│   │   │       ├── confirmation.md
│   │   │       └── task-list.md
│   │   ├── generate-card-prompt/
│   │   │   └── SKILL.md                   ← 桌牌 prompt（AI-to-AI 接力）
│   │   ├── check-availability/
│   │   │   └── SKILL.md                   ← 查空房（展示動態注入）
│   │   └── report/
│   │       ├── SKILL.md                   ← 月度營運報告（含 Mermaid）
│   │       └── templates/
│   │           └── monthly-report.md
│   └── agents/                            ← 團隊成員
│       ├── order-clerk.md                 ← 訂單處理員（sonnet）
│       ├── auditor.md                     ← 稽核員（haiku）
│       └── print-designer.md             ← 印刷設計師（sonnet）
│
├── master-data/
│   ├── CLAUDE.md                          ← 主資料保護規則
│   ├── room-types.yaml
│   ├── room-list.yaml
│   ├── menu.yaml                          ← 餐飲品項、價格、低消規則
│   └── venues.yaml                        ← 場地定義、宴會餐標、加購選項
│
├── orders/
│   ├── CLAUDE.md                          ← 訂單結構契約
│   └── ...
│
└── docs/
```

## 使用方式

### 處理訂房

```
/process-order BR250315-001
```

流程：order-clerk → 讀取需求 → room-types + room-list → 產出 → auditor 驗證

### 處理訂餐

```
/process-dining-order DR250420-001
```

流程：order-clerk → 讀取需求 → menu.yaml → 匹配品項 → **低消檢查** → 產出 → auditor 驗證

### 處理場地

```
/process-venue-order VR250420-001
```

流程：order-clerk → 讀取需求 + 名單 → venues.yaml → 場地匹配 → **混合計價** → **桌次安排** → 產出 → auditor 驗證

### 產生桌牌 Prompt

```
/generate-card-prompt VR250420-001
```

流程：print-designer → 讀取確認單桌次表 → 產出 card-prompt.md → 交 Cursor + Gemini 產圖

### 查空房

```
/check-availability 2025-04-20 2025-04-22
```

Skill 載入時動態掃描現有訂單 + master-data，列出可用房間。

### 產出月報

```
/report 2025-04
```

流程：掃描所有確認單 → 篩選月份 → 聚合統計 → 產出 `report-2025-04.md`（含 Mermaid 圖表：營收結構、房型分布、場地使用）

### 手動處理

也可以用自然語言：「請處理 orders/DR250420-001.req.md，產生確認單和任務」

## 功能複雜度梯度

```
訂房（簡單）        訂餐（中等）           場地（複雜）
──────────        ──────────           ──────────
房型匹配            品項匹配              場地 + 餐飲混合計價
房價 × 晚數         品項加總              名單管理（3 種模式）
                   + 逐人低消檢查        + 桌次安排演算法
                   + 不足時建議加點       + card-prompt 產出
                   + 關聯訂房單          + AI-to-AI 接力
```

## 與舊版的核心差異

| | 舊版（規則模式） | 新版（團隊模式） |
|---|---|---|
| 隱喻 | 一本 SOP 手冊 | 一個飯店前台團隊 |
| 觸發 | CLAUDE.md 派發表 | `/指令` 或委派子代理 |
| 驗證 | 無 | auditor 自動驗證 |
| 權限 | 文字建議 | settings.json 強制 |
| 模型 | 單一 | 多模型（sonnet + haiku） |

## 文件

| 文件 | 內容 |
|------|------|
| [01-規劃說明](docs/01-規劃說明.md) | 從 Cursor 版到 Claude 版的改寫策略 |
| [02-特性分析](docs/02-特性分析.md) | Claude 獨有機制的完整分析 |
| [03-系統架構](docs/03-系統架構.md) | 目錄結構、團隊架構、資料流 |
| [04-使用指引](docs/04-使用指引.md) | 操作步驟、進階操作、常見問題 |
| [05-claude-native-mechanisms](docs/05-claude-native-mechanisms.md) | 九大原生機制詳解 |
| [06-階段回顧](docs/06-階段回顧.md) | Phase 1-2 回顧與 Phase 3 展望 |
| [07-full-walkthrough](docs/07-full-walkthrough.md) | 從頭跑一遍：8 步驟完整操作指引 |
| [08-Phase3-完成報告](docs/08-Phase3-完成報告.md) | Phase 3 目標達成、待驗證項目、能力總覽 |
