# UU大飯店訂單管理系統 — GitHub Copilot 版實作計畫

> 本文件說明從目前狀態到全功能可展示版的執行路徑。
> 撰寫日期：2026-03-01　對應規格：[spec.md](spec.md)

---

## 一、環境前置條件

| 項目 | 需求 | 確認方式 |
|------|------|---------|
| VS Code | ≥ 1.99 | `Help > About` |
| GitHub Copilot Chat | 支援 Agent Mode 的版本 | Extensions 面板確認版本 |
| Agent Mode 開啟 | `chat.agent.enabled: true` | settings.json |
| Skills 支援 | `.github/skills/` 能被 Copilot 掃描 | 在 Chat 輸入 `/hotel-` 看是否自動補全 |
| Hooks 支援 | `preToolUse` 事件可用 | 目前為實驗性功能，需確認版本 |

---

## 二、現況盤點（2026-03-01）

### ✅ 已完成
- `AGENTS.md`：全域契約
- `.github/instructions/hotel-orders.instructions.md`
- `.github/instructions/hotel-master-data.instructions.md`
- `.github/prompts/check-availability.prompt.md`
- `.github/agents/hotel-validator.agent.md`
- `.github/skills/hotel-check-availability/SKILL.md`
- `.github/skills/hotel-generate-card-prompt/SKILL.md`
- `.github/skills/hotel-report/SKILL.md` + `assets/monthly-report.md`
- `.github/skills/hotel-order-from-req/SKILL.md`
- `.github/skills/hotel-order-from-req/assets/`：`br-confirmation.md`、`dr-confirmation.md`、`vr-confirmation.md`
- `.github/skills/hotel-order-from-req/references/order-types.md`

### ❌ 待建立
| 項目 | 優先級 |
|------|--------|
| `master-data/`（4 份 YAML） | 🔴 高 |
| `orders/`（空目錄 + .gitkeep） | 🔴 高 |
| `analyse/`（空目錄 + .gitkeep） | 🔴 高 |
| `.github/skills/hotel-order-from-req/assets/br-task.md` | 🔴 高 |
| `.github/skills/hotel-order-from-req/assets/dr-task.md` | 🔴 高 |
| `.github/skills/hotel-order-from-req/assets/vr-task.md` | 🔴 高 |
| `.github/hooks/protect-readonly.json` | 🟠 中 |
| 測試訂單（各類型至少一筆 req.md） | 🟠 中 |

---

## 三、分階段實作計畫

### Phase 1 — 基礎建設（目錄與 Master Data）

**目標**：讓 Skills 能找到定價資料，orders/ 有地方放訂單。

**步驟**：
1. 從 `../../master-data/` 複製以下四份 YAML 至 `github-copilot-approach/master-data/`：
   - `room-types.yaml`
   - `room-list.yaml`
   - `menu.yaml`（若 `../../master-data/` 沒有，從 `pure-claude-approach/master-data/` 取）
   - `venues.yaml`（同上）
2. 建立 `orders/` 目錄，加入 `.gitkeep`
3. 建立 `analyse/` 目錄，加入 `.gitkeep`

**驗收標準**：
- 執行 `/hotel-check-availability 2025-04-20 ~ 2025-04-22` 時，Copilot 能正確讀到 room-list.yaml 並列出可用房間

---

### Phase 2 — 補完任務模板（Task Templates）

**目標**：`hotel-order-from-req` SKILL.md 引用的三份 task 模板補齊，避免 Skill 執行到一半失敗。

**步驟**：建立以下三份檔案：

#### `br-task.md`（訂房任務清單模板）
必備區段：
- 訂單狀態（待確認 / 已確認 / 取消）
- 房務準備（打掃確認、備品補充、特殊需求佈置）
- 前台作業（Check-in 資料建檔、停車位預留）
- 餐廳通知（含早餐是否預訂）
- 負責部門：前台 / 房務部

#### `dr-task.md`（訂餐任務清單模板）
必備區段：
- 訂單狀態
- 備餐準備（品項確認、備料、過敏原標注）
- 場地佈置（桌號安排、座位卡）
- 特殊飲食需求（素食、過敏）處理
- 負責部門：餐廳 / 廚房

#### `vr-task.md`（場地任務清單模板）
必備區段：
- 訂單狀態
- 場地佈置（桌次、音響、燈光、簽到桌）
- 餐飲備料（宴會菜單確認、素食桌標記）
- 桌牌製作提醒（引導執行 `/hotel-generate-card-prompt`）
- VIP 接待安排
- 負責部門：宴會廳 / 餐飲部 / 工程部

**驗收標準**：
- 對任一 req.md 執行 `/hotel-order-from-req`，可同時產出 `{id}-確認單.md` 與 `{id}-任務.md`

---

### Phase 3 — Hooks 防護（Copilot 獨有亮點）

**目標**：建立 `PreToolUse` Hook，攔截任何對 `*.req.md` 與 `master-data/*` 的寫入操作，確保唯讀保護是確定性的（不依賴 LLM 遵守指令）。

**步驟**：
1. 建立 `.github/hooks/protect-readonly.json`

```json
{
  "hooks": [
    {
      "event": "PreToolUse",
      "tool": ["write_file", "edit_file", "create_file"],
      "matcher": {
        "path_pattern": ["orders/*.req.md", "master-data/**/*.yaml"]
      },
      "action": "block",
      "message": "🚫 唯讀保護：req.md 與 master-data 不可修改。請建立新的 {id}-確認單.md 而不是修改原始需求檔。"
    }
  ]
}
```

> ⚠️ 注意：Hooks 格式依 Copilot 實際版本可能有差異，需參考最新官方文件確認 `matcher` 語法。

**驗收標準**：
- 在 Agent Mode 中要求 Copilot「修改 BR250315-001.req.md」，應觸發 Hook 並顯示攔截訊息，**而非**靠 LLM 自行決定拒絕

---

### Phase 4 — 測試訂單

**目標**：各類型各置一筆 req.md，用於端到端測試。

**步驟**：在 `orders/` 建立以下測試資料：

| 檔案 | 情境 |
|------|------|
| `BR260301-001.req.md` | 標準訂房，週末 2 晚，2 大人，需停車 |
| `DR260301-001.req.md` | 4 人商務午餐，1 人素食，預算 NT$2,000 |
| `VR260301-001.req.md` | 20 人小型婚宴，內嵌賓客名單 YAML |

> req.md 的 front matter 必須符合 AGENTS.md 契約（type / date / id / guest 四欄必填）。

**驗收標準**：
- 每份 req.md 的 front matter 正確
- 不包含任何定價資訊（須由 Skill 從 master-data 計算）

---

### Phase 5 — 端到端驗收

**目標**：在 VS Code + Copilot Agent Mode 中走完完整流程，驗證六大原語協同運作。

**Demo 路徑（建議順序）**：

```
步驟 1：查詢空房
  指令：/hotel-check-availability 2026-03-01 ~ 2026-03-03
  預期：列出依房型分組的可用房間清單

步驟 2：處理訂房
  指令：/hotel-order-from-req BR260301-001
  預期：產出 BR260301-001-確認單.md + BR260301-001-任務.md
         hotel-validator 自動驗證金額與房號

步驟 3：處理訂餐
  指令：/hotel-order-from-req DR260301-001
  預期：產出確認單 + 任務，低消檢查通過

步驟 4：處理場地
  指令：/hotel-order-from-req VR260301-001
  預期：產出確認單 + 任務 + 名單.yaml（含桌次安排）

步驟 5：桌牌 Prompt
  指令：/hotel-generate-card-prompt VR260301-001
  預期：產出 VR260301-001-card-prompt.md

步驟 6：測試 Hook 防護
  指令：要求 Copilot 修改 BR260301-001.req.md
  預期：Hook 攔截，顯示唯讀錯誤訊息

步驟 7：月度報告
  指令：/hotel-report 2026-03
  預期：產出 analyse/2026-03-報告.md（含 Mermaid 圖表）
```

**全流程驗收清單**：
- [ ] 6 個 Skills 全部可正常觸發
- [ ] 確認單金額可追溯至 master-data（每筆有單價來源說明）
- [ ] hotel-validator 成功執行稽核並回傳結果
- [ ] Hook 成功攔截 req.md 寫入嘗試
- [ ] 月度報告含正確數字與 Mermaid 圖表

---

## 四、已知風險與應對

| 風險 | 說明 | 應對 |
|------|------|------|
| Hooks 格式未穩定 | `PreToolUse` 為實驗性功能，JSON schema 可能隨版本變動 | 先建立骨架，實際測試時依官方文件調整 |
| Skills 未被自動發現 | Skills 的 description 寫法影響 AI 是否自動觸發 | 確保 SKILL.md 前幾行有清晰的 trigger description |
| master-data 路徑問題 | 如果改用 symlink，Git 跨平台可能有問題 | 直接複製檔案，不使用 symlink |
| hotel-validator 工具白名單 | 若 Copilot 版本不支援 tool 限制，Agent 可能仍有寫入能力 | 以 instructions 補強「此 agent 絕不寫入」的指令 |

---

## 五、里程碑摘要

```
Phase 1　基礎建設        ← 從這裡開始
Phase 2　任務模板補完
Phase 3　Hooks 防護      ← 核心展示點
Phase 4　測試訂單建立
Phase 5　端到端驗收      ← 完成
```

預計最少步驟數：以 Copilot 執行 Phase 1-2 約需 2-3 次對話；Phase 3-4 各需 1 次；Phase 5 為手動驗收。
