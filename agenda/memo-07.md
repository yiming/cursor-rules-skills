# memo-07：功能需求清單 — 訂房 + 訂餐

> 本 memo 只定需求，不做設計。設計與實作留待 memo-06 重構時一併完成。

---

## 一、訂房（已有，待補強）

### 現有功能（✓ 已完成）

- [x] 從 `.req.md` 產生確認單（價格明細、房型匹配、房號建議）
- [x] 從 `.req.md` 產生任務清單
- [x] 主資料：room-types.yaml（房型、價格、加購）
- [x] 主資料：room-list.yaml（房號、樓層）
- [x] 品質檢查：金額可追溯、房號存在、容量足夠

### 待新增功能

- [ ] **查空房**：指定日期區間，列出可用房間（需掃描現有訂單）
- [ ] **預算不足時的替代方案**：自動列出降級房型或減少加購的選項
- [ ] **旅客歷史**：回訪旅客時，顯示過去訂單偏好（agent-memory）
- [ ] **批次處理**：一次處理多張 `.req.md`

---

## 二、訂餐（新增）

### 場景描述

飯店提供房客餐飲預訂服務：
- 早餐、午餐、晚餐、下午茶、宴會餐
- 可單點或選套餐
- 可搭配訂房一起預訂，也可獨立預訂

### 需要的主資料

**`master-data/menu.yaml`** — 餐飲品項與價格

```yaml
# 結構草案

# === 營業規則 ===
config:
  minimum_charge: 300          # 每人低消（元）
  minimum_charge_note: 飲料、點心可計入低消

# === 正餐 / 套餐 ===
meals:
  - id: BF-A
    name: 中式早餐
    price: 350
    type: 早餐
    note: 稀飯、小菜、蛋

  - id: BF-B
    name: 西式早餐
    price: 400
    type: 早餐
    note: 麵包、沙拉、炒蛋、咖啡

  - id: DN-A
    name: 主廚套餐
    price: 1200
    type: 晚餐
    note: 前菜＋主菜＋甜點

  - id: DN-B
    name: 火鍋套餐（2人份）
    price: 1800
    type: 晚餐
    note: 含肉品拼盤、蔬菜盤、飲料

  - id: TEA-A
    name: 英式下午茶（2人份）
    price: 800
    type: 下午茶
    note: 三層點心架＋紅茶

# === 飲料 ===
drinks:
  - id: DK-01
    name: 美式咖啡
    price: 120
    type: 飲料

  - id: DK-02
    name: 拿鐵
    price: 150
    type: 飲料

  - id: DK-03
    name: 紅茶 / 綠茶
    price: 80
    type: 飲料

  - id: DK-04
    name: 柳橙汁
    price: 100
    type: 飲料

  - id: DK-05
    name: 生啤酒
    price: 180
    type: 飲料

# === 點心 ===
snacks:
  - id: SN-01
    name: 手工餅乾拼盤
    price: 150
    type: 點心

  - id: SN-02
    name: 水果盤
    price: 200
    type: 點心

  - id: SN-03
    name: 起司蛋糕（片）
    price: 180
    type: 點心

  - id: SN-04
    name: 綜合堅果
    price: 120
    type: 點心
```

### 訂餐需求檔格式

**`orders/DR250420-001.req.md`** — 範例

```yaml
---
type: 訂餐
date: 2025-04-20
id: DR250420-001
guest: 王大明
related_order: BR250315-001    # 可選：關聯的訂房單
---
```

```markdown
# 訂餐需求

- 日期：2025-04-20 晚餐
- 人數：4 位
- 需求：想吃火鍋，另外加兩杯啤酒
- 預算：3,000 元以內
- 備註：有一位吃素，需要素食選項
```

### 訂餐確認單應包含

| 欄位 | 說明 |
|------|------|
| 訂單編號 | DR + YYMMDD + 序號 |
| 訂餐人 | guest |
| 用餐日期 | date |
| 餐別 | 早餐/午餐/晚餐/下午茶 |
| 人數 | guests |
| 品項明細 | 逐項：品項 × 數量 = 小計 |
| 合計 | 總金額 |
| 預算比對 | 在預算內 / 超出 |
| 特殊需求 | 素食、過敏等 |
| 關聯訂房 | 若有 related_order |

### 訂餐任務清單應包含

- [ ] 確認廚房備料（含特殊需求）
- [ ] 確認用餐時段與座位
- [ ] 若有關聯訂房，通知房務一併安排
- [ ] 用餐前 2 小時 final check

### 訂餐 Skill

**`.claude/skills/process-dining-order/SKILL.md`**

流程與訂房類似，但多了低消檢查：
1. 讀取需求 `.req.md`
2. 載入 `master-data/menu.yaml`（含 config.minimum_charge）
3. 匹配品項、計算金額
4. 處理特殊需求（素食替換等）
5. **低消檢查**（見下方邏輯）
6. 產出確認單 + 任務清單
7. 品質檢查

### 低消邏輯

```
對每位用餐者：
  計算該人的消費額（分攤正餐 + 個人單點）
  若 消費額 < config.minimum_charge（300 元）：
    差額 = minimum_charge - 消費額
    從 drinks + snacks 中建議補足項目（優先選最接近差額的組合）
    在確認單中標記：「⚠ 低消不足，建議加點」
```

**範例情境**：4 人晚餐，3 人吃火鍋套餐（1800/2人份 → 每人 900），
1 人只點一杯紅茶（80 元）→ 低消不足 220 元 → 建議加點：
- 起司蛋糕 180 + 綜合堅果 120 = 300 → 補足後 380 元 ✓
- 或：水果盤 200 + 美式咖啡 120 = 320 → 補足後 400 元 ✓

確認單中應呈現：

```
| 用餐者 | 品項 | 小計 | 低消狀態 |
|--------|------|------|---------|
| 王大明 | 火鍋套餐（均攤）| 900 | ✓ |
| 李小華 | 火鍋套餐（均攤）| 900 | ✓ |
| 張三   | 火鍋套餐（均攤）| 900 | ✓ |
| 陳四   | 紅茶 ×1        |  80 | ⚠ 不足 220 元 |
|        | 【建議加點】起司蛋糕 + 綜合堅果 | +300 | → 380 ✓ |
```

### 訂餐的特殊考量（與訂房不同）

| 面向 | 訂房 | 訂餐 |
|------|------|------|
| 主資料 | room-types + room-list | menu（含 config） |
| 營業規則 | 無低消 | **有低消**（每人 300 元），不足需建議加點 |
| 資源衝突 | 房號可能被佔（需查空房） | 通常不衝突（食材） |
| 關聯性 | 獨立 | 可關聯訂房單 |
| 特殊需求 | 加床、安靜 | 素食、過敏、忌口 |
| 時段 | 日期區間（多晚） | 單一餐別 |
| 單位 | 每房/每晚 | 每份/每人 |
| 計算複雜度 | 房價 × 晚數 + 加購 | 品項加總 + 逐人低消檢查 + 建議補足 |

---

## 三、場地租借（新增）
```
產圖需求: 先不執行, 先完成其它功能
但產圖的PROMPT要先做出來

Q.
場地租借: 純場地, + buffet snack bar, or 桌菜(3種價位)
額外功能, 可提供出席者名單, 或各桌名單, 套印:桌牌或名牌,
呼叫: gemini(nano banana)畫圖為: jpg or png
要能指定風格的prompt
能做到嗎?

請說明呼叫GEMINI的方法(不透過程式),
因為我可以想像, cursor本來就有gemini模型,
但claude本身完全不會畫圖
claude code也只有claude模型
, 要怎麼不寫程式, 又可以使用到nano banana?

我只要最簡單最AI AGENT的方法, 不要跟我說寫PYTHON什麼的

---
A.
Claude Code 本身不會畫圖，但它可以呼叫工具。
MCP Server 就是一個「工具插座」——裝上去，Claude 就多了一個能力。

但是——誠實說
目前 Gemini 的官方 MCP Server 生態還在早期。實際可用的路徑有幾條：
....
最務實的最簡方案
如果 MCP 現成包還沒穩定，退一步用 Skill 的 !command`` ：
---

Q.
如果我有安裝gemili cli, 可不可以用命令列執行: gemini + prompt(名稱) => JPG?
也許你可以給我一行命令試試, 例:
古典歐式, 中式風格, 現代商務風 的桌牌

A. (FROM GEMINI)
gemini cli不含nano banana,
最可行的是 MCP, 且需要API KEY

Q. 替代方案:(no code, no mcp, no api)
我能否:
1. 建一個資料夾, 寫一個md
2. 包含:圖片創作需求, 10個名字
3. 用antigravity打開該MD檔, 選GEMINI模型
4. 依指示:產出10張圖檔

測試可行: image-test\card-prompt.md
產圖需求, 暫時, CLAUDE先產出, 該筆訂位的名牌清單PROMPT.MD
半人工以: Cursor(VSCode系), 使用Gemini 3模型產出圖片
```

### 場景描述

飯店提供會議室、宴會廳租借服務：
- 純場地租借（會議、研討會、記者會）
- 場地 + Buffet / Snack Bar
- 場地 + 桌菜宴席（3 種價位）
- 附加服務：出席者名單管理、桌次安排、桌牌/名牌套印（含 AI 生圖）

### 需要的主資料

**`master-data/venues.yaml`** — 場地與餐飲方案

```yaml
# === 場地 ===
venues:
  - id: MR-A
    name: 松柏廳（小型會議室）
    capacity: 20
    price_half_day: 5000
    price_full_day: 8000
    note: 含投影機、白板、Wi-Fi

  - id: MR-B
    name: 翠竹廳（中型會議室）
    capacity: 50
    price_half_day: 12000
    price_full_day: 20000
    note: 含音響、投影、直播設備

  - id: BQ-A
    name: 國際宴會廳
    capacity: 200
    price_half_day: 30000
    price_full_day: 50000
    note: 含舞台、燈光音響

# === 場地餐飲方案 ===
venue_catering:
  buffet:
    - id: BUF-A
      name: 輕食 Buffet
      price: 500
      unit: 每人
      note: 三明治、沙拉、飲料

    - id: BUF-B
      name: 豪華 Buffet
      price: 800
      unit: 每人
      note: 熱食、冷盤、甜點、飲料吧

  snack_bar:
    - id: SNK-A
      name: 茶歇 Snack Bar
      price: 300
      unit: 每人
      note: 咖啡、茶、餅乾、水果

  banquet:  # 桌菜（每桌 10 人）
    - id: BQT-A
      name: 金禧桌菜
      price: 8000
      unit: 每桌
      note: 8 菜 1 湯，經濟實惠

    - id: BQT-B
      name: 鑽禧桌菜
      price: 12000
      unit: 每桌
      note: 10 菜 1 湯，含龍蝦

    - id: BQT-C
      name: 皇家桌菜
      price: 18000
      unit: 每桌
      note: 12 菜 1 湯，含鮑魚、魚翅
```

### 訂場地需求檔格式

**`orders/VR250420-001.req.md`** — 範例

```yaml
---
type: 場地
date: 2025-04-20
id: VR250420-001
guest: 陳經理
company: ABC 科技股份有限公司
---
```

```markdown
# 場地租借需求

- 日期：2025-04-20（全天）
- 用途：年度經銷商大會
- 人數：80 人
- 場地需求：需要舞台、投影、音響
- 餐飲：中午 Buffet，晚上桌菜宴席
- 桌菜預算：每桌 12,000 左右，預估 8 桌
- 預算：總計 200,000 元以內
- 附加需求：
  - 提供出席者名單（80 人，含公司、職稱）
  - 晚宴需要桌次安排（8 桌，每桌 10 人）
  - 每桌需要桌牌
  - 每位出席者需要名牌
  - 桌牌/名牌風格：日式和風水墨畫風格，配色淡雅
```

### 場地確認單應包含

| 欄位 | 說明 |
|------|------|
| 訂單編號 | VR + YYMMDD + 序號 |
| 預訂人 / 公司 | guest / company |
| 活動日期 | date |
| 場地 | 場地名稱、時段（半天/全天） |
| 場地費用 | price |
| 餐飲方案 | Buffet / Snack Bar / 桌菜（品項 × 數量） |
| 餐飲費用 | 小計 |
| 合計 | 場地 + 餐飲 |
| 預算比對 | 在預算內 / 超出 |
| 附加服務 | 桌次安排、桌牌/名牌（含風格） |

### 場地任務清單應包含

- [ ] 確認場地檔期，鎖定日期
- [ ] 確認餐飲方案，通知廚房
- [ ] 收集出席者名單（姓名、公司、職稱）
- [ ] 安排桌次（產出桌次表）
- [ ] 產出桌牌（AI 生圖 + 套印）
- [ ] 產出名牌（AI 生圖 + 套印）
- [ ] 活動前一日場地布置 check
- [ ] 活動當日流程確認

### 附加功能：出席者名單與桌次安排

#### 名單格式

**`orders/VR250420-001-名單.yaml`**

```yaml
attendees:
  - name: 張董事長
    company: XYZ 集團
    title: 董事長
    table: 1          # 桌次（由 AI 安排或手動指定）
    note: 主桌

  - name: 李總經理
    company: DEF 貿易
    title: 總經理
    table: 1

  - name: 王經理
    company: GHI 科技
    title: 業務經理
    table: 2
    note: 素食
```

#### 桌次安排邏輯

```
1. 讀取名單
2. 依桌菜方案決定每桌人數（預設 10 人/桌）
3. 若名單已指定 table → 尊重指定
4. 未指定者 → 依公司分組（同公司盡量同桌），再依職稱排序
5. 產出「桌次總表」：每桌的完整名單
6. 檢查：每桌人數 ≤ 上限、所有人都有桌次
```

### 附加功能：桌牌 / 名牌（Claude 產 Prompt → Cursor + Gemini 產圖）

#### 分工流程

```
Claude Code（自動）                    Cursor + Gemini（半人工）
─────────────────                    ──────────────────────
1. 讀取名單 + 風格需求
2. 產出 card-prompt.md
   （含風格指示 + 逐張圖片需求）
                          ──交棒──→  3. 使用者在 Cursor 開啟 card-prompt.md
                                     4. 選 Gemini 模型
                                     5. Gemini 依指示逐張產出圖片
                                     6. 存入 orders/{id}-prints/
```

**關鍵**：Claude 不畫圖，但 Claude 負責產出完整的 prompt 檔案。
使用者只需「開檔 → 選模型 → 執行」，不需要自己寫 prompt。

#### Claude 產出的 card-prompt.md 範例

**`orders/VR250420-001-card-prompt.md`**

```markdown
# 桌牌 / 名牌生成指示

## 風格設定

請依照以下風格生成所有圖片：
- 風格：日式和風水墨畫風格
- 配色：淡雅、米白底、墨色漸層
- 排版：橫式名牌（寬 90mm × 高 55mm，300dpi）
- 要求：背景有留白空間放置文字，文字直接渲染在圖中

---

## 桌牌（共 8 張）

每張桌牌尺寸：寬 150mm × 高 100mm，300dpi
文字置中，大字顯示桌次編號

### 桌牌 1
生成一張日式和風水墨畫風格的桌牌。
淡雅米白背景，墨色山水意境裝飾邊框。
正中央大字顯示「第 1 桌」，字體優雅。
儲存為：table-01.png

### 桌牌 2
（同上風格）正中央大字顯示「第 2 桌」
儲存為：table-02.png

...（依此類推至第 8 桌）

---

## 名牌（共 80 張，以下列前 5 張為例）

每張名牌尺寸：寬 90mm × 高 55mm，300dpi
上方大字：姓名，下方小字：公司 / 職稱

### 名牌 — 張董事長
日式和風水墨畫風格名牌。
上方大字：「張董事長」
下方小字：「XYZ 集團｜董事長」
右下角小字：「第 1 桌」
儲存為：badge-張董事長.png

### 名牌 — 李總經理
（同上風格）
上方大字：「李總經理」
下方小字：「DEF 貿易｜總經理」
右下角小字：「第 1 桌」
儲存為：badge-李總經理.png

### 名牌 — 王經理
（同上風格）
上方大字：「王經理」
下方小字：「GHI 科技｜業務經理」
右下角小字：「第 2 桌」
備註：素食（名牌左下角加綠色小圓點標記）
儲存為：badge-王經理.png

...（依名單逐一列出）
```

#### 產出 card-prompt.md 的 Skill

**`.claude/skills/generate-card-prompt/SKILL.md`**

```yaml
---
name: generate-card-prompt
description: 從場地訂單名單產出桌牌/名牌的圖片生成 prompt 檔
argument-hint: "[order-id]"
user-invocable: true
---
```

流程：
1. 讀取 `orders/{id}-名單.yaml`（含桌次安排）
2. 讀取 `orders/{id}.req.md` 中的風格描述
3. 產出 `orders/{id}-card-prompt.md`，包含：
   - 風格統一指示（從需求檔擷取）
   - 桌牌區段：每桌一個 prompt
   - 名牌區段：每人一個 prompt（含姓名、公司、職稱、桌次、特殊標記）
4. 提示使用者：「請在 Cursor 中開啟此檔案，選擇 Gemini 模型執行產圖」

#### 這個功能展示的 Claude 機制

| 機制 | 怎麼用到 |
|------|---------|
| **Skill 參數化** | `/generate-card-prompt VR250420-001` |
| **跨工具協作** | Claude 產 prompt → Cursor + Gemini 產圖 |
| **Skill 讀取多檔** | 名單 .yaml + 需求 .req.md → 組合產出 |
| **純 Markdown 產出** | 不需要 API key、不需要 Python、不需要 MCP |
| **AI-to-AI 接力** | Claude（文字推理）→ Gemini（圖片生成） |

### 場地的特殊考量（與訂房、訂餐不同）

| 面向 | 訂房 | 訂餐 | 場地 |
|------|------|------|------|
| 主資料 | room-types + room-list | menu | venues + venue_catering |
| 計算 | 房價×晚數 | 品項加總+低消 | 場地費+餐飲費（混合計價） |
| 關聯性 | 獨立 | 可關聯訂房 | 可關聯訂房+訂餐 |
| 附加產出 | 無 | 無 | 桌次表、card-prompt.md（供 Gemini 產圖） |
| 跨工具協作 | 無 | 無 | Claude 產 prompt → Cursor + Gemini 產圖 |
| 複雜度 | 低 | 中（低消） | 高（名單+桌次+prompt生成+跨工具） |

---

## 四、三種訂單共用的部分

### 共用規則（寫在根 CLAUDE.md）

- ID 格式：`類型碼 + YYMMDD + 序號`（BR=訂房, DR=訂餐, VR=場地）
- 唯讀原則：`*.req.md` 不可修改
- 價格真實：所有金額必須從 master-data 計算
- 產出路徑：`orders/{id}-確認單.md`、`orders/{id}-任務.md`

### 共用子代理

- **order-clerk**：根據 type 判斷載入哪個 skill（訂房/訂餐/場地）
- **auditor**：驗證金額可追溯、品項存在、數量合理
- **print-designer**（新增）：專責產出 card-prompt.md（供 Cursor + Gemini 產圖）

### 共用 Skill 結構

三種訂單的 Skill 流程高度相似（讀需求 → 載入主資料 → 匹配計算 → 產出 → 品檢），
差異在主資料來源、確認單欄位、附加產出。

### 複雜度遞增設計

這三種訂單剛好形成一個**由簡到繁的梯度**，適合展示 Claude 的不同能力層次：

```
訂房（簡單）     訂餐（中等）          場地（複雜）
─────────────────────────────────────────────────
基本 CRUD        + 低消商業邏輯         + 名單管理
房型匹配          + 逐人檢查/建議        + 桌次安排演算法
                                       + 跨工具協作（Claude→Gemini）
                                       + prompt 工程（產出 card-prompt.md）
                                       + 多 skill 串接

展示：            展示：                展示：
CLAUDE.md        config 驅動邏輯       AI-to-AI 接力、
skills 基本用法   確認單進階格式         子代理分工協作
agents 委派                           跨 skill 呼叫
```

---

## 五、執行計劃

### Phase 1：重構基礎（memo-06）
1. 建立 `.claude/` 結構（settings.json, agents）
2. 遷移訂房 skill 到正規位置 + frontmatter
3. AGENTS.md → CLAUDE.md
4. 更新根 CLAUDE.md、readme、docs

### Phase 2：訂餐
5. 新增 `master-data/menu.yaml`（含 config、飲料、點心）
6. 新增訂餐 skill（含低消邏輯）
7. 新增訂餐範例 req + 跑一次流程

### Phase 3：場地
8. 新增 `master-data/venues.yaml`
9. 新增場地 skill（場地+餐飲計算）
10. 新增名單管理 + 桌次安排邏輯
11. 新增 generate-card-prompt skill（產出 prompt MD 檔）
12. 新增場地範例 req + 跑一次完整流程
13. 測試：用 Cursor + Gemini 開啟 card-prompt.md 產圖

### 前置需求
- Phase 3 產圖需要：Cursor（或任何支援 Gemini 的 VSCode 系編輯器）
- 不需要 API key、不需要 Python、不需要 MCP
