# UU大飯店訂單管理系統

本專案以 .md / .yaml 管理飯店訂單，不使用程式或資料庫。
由 AI 團隊協作完成訂單處理、驗證、查詢等任務。

## 設計哲學

> **從「AI 讀規則做事」到「AI 組建團隊做事」**

```
                    ┌─────────────────────┐
                    │   飯店經理 (CLAUDE.md) │  ← 制定政策、分派任務
                    └────────┬────────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
     ┌────────────┐  ┌────────────┐  ┌────────────┐
     │  訂單處理員  │  │   稽核員    │  │  查詢服務   │
     │ order-clerk │  │  auditor   │  │ /check-... │
     │  (sonnet)  │  │  (haiku)   │  │  (skill)   │
     └────────────┘  └────────────┘  └────────────┘
```

## 全域政策

1. **唯讀原則**：`*.req.md` 是客戶原始需求，任何情況下不得覆蓋或修改。（由 `.claude/settings.json` 強制執行）
2. **價格真實**：所有金額必須從 `master-data/` 的單價計算得出，不得自行編造。
3. **房號真實**：建議的房號必須存在於 `master-data/room-list.yaml`。
4. **ID 格式**：訂單 ID 格式為 `類型碼 + YYMMDD + 序號`（如 `BR250315-001`）。
   - BR = 訂房、DR = 訂餐、VR = 場地

## 團隊分工

| 任務 | 負責人 | 觸發方式 |
|------|--------|---------|
| 處理訂單需求 | order-clerk（子代理） | 主動委派，或 `/process-order [id]` |
| 驗證確認單 | auditor（子代理） | 處理完成後自動委派 |
| 查詢空房 | `/check-availability` | 使用者直接呼叫 |

## 信任邊界

`.claude/settings.json` 定義系統層級的強制保護：

- **禁止修改** `orders/*.req.md`（客戶需求唯讀）
- **禁止修改** `master-data/*`（價格來源不可竄改）

這不是「建議」——是系統強制的權限控制。

## 處理流程

```
使用者：/process-order BR250315-001
  │
  ├─→ order-clerk（子代理，sonnet）
  │     ├── 讀取需求 → 載入主資料 → 計算匹配 → 產出
  │     └── 寫入確認單 + 任務清單
  │
  └─→ auditor（子代理，haiku）
        ├── 比對金額、房號、容量
        └── 回報：✓ 通過 / ✗ 不通過
```

## 目錄結構

```
├── CLAUDE.md                              ← 你正在讀的：團隊政策 + 分工表
├── .claude/
│   ├── settings.json                      ← 信任邊界（強制權限）
│   ├── skills/
│   │   ├── process-order/                 ← /process-order 指令
│   │   │   ├── SKILL.md
│   │   │   └── templates/
│   │   │       ├── confirmation.md        ← 確認單模板
│   │   │       └── task-list.md           ← 任務清單模板
│   │   └── check-availability/            ← /check-availability 指令
│   │       └── SKILL.md                   ← 含動態上下文注入
│   └── agents/
│       ├── order-clerk.md                 ← 訂單處理員（sonnet, Read+Write）
│       └── auditor.md                     ← 稽核員（haiku, Read-only）
├── master-data/
│   ├── CLAUDE.md                          ← 主資料保護規則
│   ├── room-types.yaml
│   └── room-list.yaml
├── orders/
│   ├── CLAUDE.md                          ← 訂單結構契約
│   └── ...
└── docs/
```
