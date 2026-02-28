# Memo-08：用 Cursor Rules + Skills 完整實現飯店訂單系統

> 目標：將 `pure-claude-approach/` 的完整功能，以 **Cursor 的精神與特色**重新實現。

---

## 一、現況盤點

### 已完成（Cursor 版）

| 檔案 | 說明 |
|------|------|
| `.cursor/rules/hotel-orders.mdc` | 訂單結構契約（glob: `orders/**/*.md`） |
| `.cursor/skills/hotel-order-from-req/SKILL.md` | 訂房確認單 + 任務清單（僅訂房，無訂餐/場地） |
| `AGENTS.md` | 全域政策（唯讀、價格、房號、ID 格式） |

### 尚未實現（對照 Claude 版功能）

| Claude 版功能 | Cursor 版現況 | 缺口 |
|--------------|--------------|------|
| 訂房處理 | ✓ 部分完成 | 需補 templates/ 分離 |
| 訂餐處理（低消邏輯） | ✗ 無 | 需新增 DR 分支 |
| 場地處理（桌次演算法） | ✗ 無 | 需新增 VR 分支 |
| 桌牌 prompt 產出 | ✗ 無 | 需新增 skill |
| 查空房 | ✗ 無 | 需新增 skill |
| 月度營運報告 | ✗ 無 | 需新增 skill + template |
| 稽核驗證 | ✗ 無 | 改用 skill 內建自查清單 |
| 訂餐 master-data | ✗ 無 | 需新增 menu.yaml |
| 場地 master-data | ✗ 無 | 需新增 venues.yaml |

---

## 二、值得從 Claude 版借鏡的設計

### 借鏡 1：子目錄 CLAUDE.md → Cursor 的 glob rule

Claude 版在 `orders/CLAUDE.md` 和 `master-data/CLAUDE.md` 各放一份規則，
操作對應目錄時自動生效，**規則與資料同住一處**，非常清晰。

Cursor 版對應做法：為每個目錄建立專屬 glob rule：
- `hotel-orders.mdc`（`globs: orders/**`）→ 已有，補強內容
- `hotel-master-data.mdc`（`globs: master-data/**`）→ 新增

**借鏡重點**：`orders/CLAUDE.md` 裡有一份非常完整的「賓客名單約定」（三種 guest_list 模式說明），
這份說明應該直接搬進 `hotel-orders.mdc`，讓 AI 在開啟訂單檔時就自動知道名單規則。

### 借鏡 2：templates/ 模板分離

Claude 版每個 skill 都有獨立的 `templates/` 子目錄，
模板與流程邏輯完全分離，修改模板不影響流程，反之亦然。

Cursor 版目前的 `hotel-order-from-req/SKILL.md` 把模板直接寫在 SKILL.md 裡，
**應該拆出來**，建立 `templates/` 子目錄，對應六種產出（BR/DR/VR × 確認單/任務清單）。

### 借鏡 3：確認單的 YAML frontmatter

Claude 版的確認單有 frontmatter：
```yaml
---
order_id: {id}
generated_at: {產出時間 YYYY-MM-DD HH:mm}
---
```

這讓月報 skill 可以用程式化方式掃描確認單（從 frontmatter 取 order_id），
而不是靠 AI 猜測。**Cursor 版的月報 skill 也應該依賴這個結構**。

### 借鏡 4：場地確認單的「桌牌提示」

Claude 版場地確認單模板末尾有一段：
```
### 桌牌
1. 執行 `/generate-card-prompt {id}` → 產出 card-prompt.md
2. 將 prompt 檔交給 Cursor + Gemini 執行 → 圖片存入 orders/{id}-cards/
```

這是「多 skill 串接」的設計——確認單本身就告訴使用者下一步是什麼。
Cursor 版應該保留這個設計，只是把指令改成 `@hotel-generate-card-prompt`。

### 借鏡 5：report skill 的掃描邏輯

Claude 版 report skill 定義了非常精確的欄位提取規則：

| 訂單類型 | 金額來源 | 日期來源 | 人數來源 |
|---------|---------|---------|---------|
| BR | 「合計」列 | 「入住日期」 | 「人數」 |
| DR | 品項明細「合計」列 | 「用餐日期」 | 「人數」 |
| VR | 費用明細「合計」列 | 「活動日期」 | 「人數」 |

這份規格讓 AI 知道確切要找哪個欄位，不會猜錯。**Cursor 版 report skill 應完整移植這份規格**。

### 借鏡 6：monthly-report 模板的 Mermaid 圖表

Claude 版月報模板有三種 Mermaid 圖表（pie + xychart-beta），
且有條件顯示邏輯（`<!-- 僅當有 BR 訂單時顯示 -->`）。
Cursor 的 IDE 可以**直接預覽 Mermaid**，這是 Cursor 版的額外優勢，應完整保留。

---

## 三、Cursor 特色可以加進去的設計

### 特色 1：開檔即上下文（glob rule 的主動性）

**Claude 版**：使用者要先說 `/process-order BR250315-001`，AI 才知道要做什麼。
**Cursor 版可以做到**：使用者只要開啟 `orders/BR250315-001.req.md`，
`hotel-orders.mdc` 就自動注入，AI 已知道這是訂單、知道規則、知道可以呼叫哪個 skill。

使用者甚至不需要說任何話，直接問「幫我處理這張」就夠了。

**設計**：在 `hotel-orders.mdc` 末尾加入：
```
## 當前檔案提示
若使用者開啟的是 *.req.md，可直接說「處理這張訂單」觸發 @hotel-order-from-req。
```

### 特色 2：@mention 帶出檔案選擇 UI

Claude 版：`/process-order BR250315-001`（要記 ID）
Cursor 版：`@hotel-order-from-req` → 然後 `@orders/` → 彈出檔案選擇清單

**設計**：在每個 skill 的 description 中加入「使用者可以 @mention 訂單檔案」的說明，
引導使用者用 `@` 選檔，而不是手打 ID。

### 特色 3：Cursor 內建 Gemini 完成 AI-to-AI 接力

Claude 版的 AI-to-AI 接力需要切換工具（Claude Code 產 prompt → 切換到 Cursor + Gemini 產圖）。
Cursor 版可以在**同一個 IDE 內完成**：

```
@hotel-generate-card-prompt 請為 @orders/VR250420-001-確認單.md 產生桌牌 prompt
→ 產出 orders/VR250420-001-card-prompt.md
→ 直接在 Cursor 中：請依照 @orders/VR250420-001-card-prompt.md 的指示，用 Gemini 批次生成桌牌圖片
```

**設計**：`hotel-generate-card-prompt` skill 末尾加入「產出後的下一步」說明，
引導使用者直接在 Cursor 中用 `@card-prompt.md` 觸發 Gemini 產圖。

### 特色 4：多 rule 疊加形成「情境感知」

當使用者在 `orders/` 目錄工作時，同時疊加：
- `hotel-always.mdc`（全域禁止規則）
- `hotel-orders.mdc`（訂單結構 + 名單規則）

當使用者在 `master-data/` 工作時：
- `hotel-always.mdc`（全域禁止規則）
- `hotel-master-data.mdc`（唯讀保護 + 檔案說明）

這比 Claude 版的單一 CLAUDE.md 更有**情境感知**——AI 根據你在哪個目錄，自動調整行為。

**設計**：`hotel-master-data.mdc` 不只說「唯讀」，還要說明每個 yaml 的結構，
讓 AI 在查看 master-data 時就知道欄位含義，不需要額外說明。

### 特色 5：rule 中嵌入 skill 觸發說明（降低使用門檻）

在 `hotel-orders.mdc` 中加入一個「快速操作表」：

```markdown
## 快速操作
| 想做什麼 | 說法 |
|---------|------|
| 處理訂單 | 「處理這張訂單」或 @hotel-order-from-req |
| 查空房 | 「查 4/20-4/22 空房」或 @hotel-check-availability |
| 產生桌牌 | 「產生桌牌 prompt」或 @hotel-generate-card-prompt |
| 月報 | 「產出 2025-04 月報」或 @hotel-report |
```

使用者不需要記任何指令，看到這張表就知道怎麼操作。

---

## 四、完整架構設計（Refined）

### 目錄結構（目標）

```
.cursor/
├── rules/
│   ├── hotel-always.mdc              ← 新增：全域政策（alwaysApply: true）
│   ├── hotel-orders.mdc              ← 升級：補強名單規則 + 快速操作表
│   └── hotel-master-data.mdc         ← 新增：master-data 結構說明 + 唯讀保護
└── skills/
    ├── hotel-order-from-req/         ← 升級：支援 BR/DR/VR 三種類型
    │   ├── SKILL.md                  ← 流程邏輯（分支處理）
    │   └── templates/
    │       ├── br-confirmation.md    ← 訂房確認單
    │       ├── br-task-list.md       ← 訂房任務清單
    │       ├── dr-confirmation.md    ← 訂餐確認單（含低消表格）
    │       ├── dr-task-list.md       ← 訂餐任務清單
    │       ├── vr-confirmation.md    ← 場地確認單（含桌次表 + 桌牌提示）
    │       └── vr-task-list.md       ← 場地任務清單
    ├── hotel-check-availability/     ← 新增
    │   └── SKILL.md
    ├── hotel-generate-card-prompt/   ← 新增（含 Cursor 內產圖說明）
    │   └── SKILL.md
    └── hotel-report/                 ← 新增
        ├── SKILL.md
        └── templates/
            └── monthly-report.md    ← 含 Mermaid 圖表（Cursor 可直接預覽）

master-data/                          ← 新增（從 pure-claude-approach 複製）
    ├── room-types.yaml
    ├── room-list.yaml
    ├── menu.yaml
    └── venues.yaml
```

### Rule 設計

#### `hotel-always.mdc`（alwaysApply: true）
全域政策，任何對話都注入。內容：
- 絕對禁止（不改 req.md、不改 master-data、不編造價格）
- 訂單 ID 格式（BR/DR/VR）
- 產出路徑規則

#### `hotel-orders.mdc`（globs: orders/\*\*）
訂單目錄規則，開啟訂單相關檔案時注入。內容：
- req.md 的 YAML frontmatter 規格
- 三種 guest_list 模式（外部檔案 / embedded / 無名單）← 從 Claude 版借鏡
- 快速操作表 ← Cursor 特色

#### `hotel-master-data.mdc`（globs: master-data/\*\*）
主資料保護規則，開啟 master-data 時注入。內容：
- 唯讀保護說明
- 各 yaml 的欄位結構說明（讓 AI 查表時不需要額外解釋）

---

## 五、Cursor vs Claude 功能對照（完整版）

| 功能 | Claude 版 | Cursor 版 | 差異 |
|------|----------|----------|------|
| 訂房/訂餐/場地處理 | 3 個獨立 skill | 1 個 skill（type 分支） | Cursor 合併，@mention 更直觀 |
| 查空房 | `!cmd` 動態注入 | skill 明確指示讀取檔案 | 效果相同，機制不同 |
| 月報 | `/report` skill | `@hotel-report` skill | 功能相同 |
| 桌牌 prompt | `/generate-card-prompt` | `@hotel-generate-card-prompt` | Cursor 版可在 IDE 直接產圖 |
| 稽核驗證 | auditor 子代理（haiku） | skill 末尾自查清單 | Cursor 無子代理，AI 自我驗證 |
| 全域政策 | CLAUDE.md | `hotel-always.mdc` | 機制不同，效果相近 |
| 目錄規則 | 子目錄 CLAUDE.md | glob rule（.mdc） | 效果相同 |
| 模板分離 | templates/ 子目錄 | templates/ 子目錄 | 完全相同 |
| 確認單 frontmatter | ✓ 有（供月報掃描） | ✓ 應保留（同樣理由） | 借鏡 |
| 桌牌提示（串接提示） | 確認單末尾提示 | 確認單末尾提示（改 @mention） | 借鏡 |
| 情境感知 | 目錄 CLAUDE.md | 多 rule 疊加 | Cursor 更精細 |
| 使用門檻 | 需記 `/指令` | 快速操作表 + @mention UI | Cursor 更低 |
| 強制唯讀 | settings.json deny（硬） | rule 明確禁止（軟） | Claude 更強，Cursor 靠 AI 遵守 |

---

## 六、實作計畫（Refined）

### Phase A：基礎建設（先做，其他都依賴這個）

**A1. 複製 master-data**
- 從 `pure-claude-approach/master-data/` 複製 `menu.yaml`、`venues.yaml` 到根目錄 `master-data/`
- `room-types.yaml`、`room-list.yaml` 已有，確認路徑正確

**A2. 新增 `hotel-always.mdc`**
- alwaysApply: true
- 全域禁止規則 + ID 格式 + 產出路徑

**A3. 升級 `hotel-orders.mdc`**
- 補入三種 guest_list 模式說明（從 Claude 版 orders/CLAUDE.md 借鏡）
- 加入快速操作表（Cursor 特色）

**A4. 新增 `hotel-master-data.mdc`**
- globs: master-data/**
- 唯讀保護 + 各 yaml 欄位說明

### Phase B：升級核心 Skill

**B1. 升級 `hotel-order-from-req` SKILL.md**
- 加入 DR（訂餐）分支：低消邏輯（從 Claude 版 process-dining-order/SKILL.md 移植）
- 加入 VR（場地）分支：桌次安排演算法（從 Claude 版 process-venue-order/SKILL.md 移植）
- 加入自查清單（取代 auditor）
- 加入 @mention 觸發說明

**B2. 新增 templates/**
- `br-confirmation.md`（從 Claude 版移植，加 frontmatter）
- `br-task-list.md`（從 Claude 版移植，加 frontmatter）
- `dr-confirmation.md`（從 Claude 版移植）
- `dr-task-list.md`（從 Claude 版移植）
- `vr-confirmation.md`（從 Claude 版移植，末尾改為 @mention 桌牌提示）
- `vr-task-list.md`（從 Claude 版移植）

### Phase C：新增 Skills

**C1. `hotel-check-availability`**
- 明確指示讀取 room-list.yaml、room-types.yaml
- 掃描 orders/*-確認單.md 找已預訂房號
- 以表格呈現

**C2. `hotel-generate-card-prompt`**
- 從確認單桌次表提取名單
- 風格規範（四種風格對照表，從 Claude 版移植）
- 人數 ≤50 → 座位卡，>50 → 桌牌
- **Cursor 特色**：末尾加入「產出後直接在 Cursor 用 @card-prompt.md + Gemini 產圖」說明

**C3. `hotel-report`**
- 完整移植 Claude 版的欄位提取規格（BR/DR/VR 各欄位來源）
- 移植 monthly-report.md 模板（含三種 Mermaid 圖表）
- 條件顯示邏輯（無 BR/DR/VR 訂單時的處理）

### Phase D：整合驗證

- 端到端測試：BR → DR → VR → 桌牌 → 月報
- 確認 rule 疊加效果（orders/ 目錄下同時觸發 hotel-always + hotel-orders）
- 確認 Mermaid 在 Cursor 中可預覽

---

## 七、設計亮點總結

### 從 Claude 版借鏡的
1. **templates/ 模板分離**：修改模板不影響流程邏輯
2. **確認單 YAML frontmatter**：讓月報 skill 可程式化掃描
3. **場地確認單末尾的串接提示**：告訴使用者下一步
4. **report skill 的精確欄位規格**：AI 不猜，直接找對欄位
5. **guest_list 三種模式**：外部檔案 / embedded / 無名單

### Cursor 特色加值的
1. **開檔即知曉**：glob rule 自動注入，不需要說「這是訂單」
2. **快速操作表**：rule 裡嵌入操作說明，使用者零學習成本
3. **@mention + 檔案選擇 UI**：不用記 ID，@orders/ 彈出清單
4. **AI-to-AI 接力不換工具**：card-prompt → Gemini 產圖在同一 IDE
5. **Mermaid 即時預覽**：月報圖表不只是文字，是可視化的
6. **多 rule 情境感知**：在 orders/ 和 master-data/ 各有專屬上下文

---

*Phase A → B → C → D 依序執行，A 完成後其他 Phase 可並行。*
