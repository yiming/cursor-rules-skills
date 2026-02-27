# 工作順序規劃

先建 master data / orders 範例，還是先建 rules/skills boilerplate？

## 建議：**先最小 boilerplate，再最小資料 + 一筆訂單**


| 順序                                   | 做什麼                                                         | 理由                                    |
| ------------------------------------ | ----------------------------------------------------------- | ------------------------------------- |
| **1. 最小 Rule**                       | 訂單資料夾命名、`.req.md` 必備 YAML 欄位、master data 檔案位置與格式約定          | 先定義「契約」，後面產生的資料與 Skill 才有同一套結構，不會各寫各的 |
| **2. 最小 master data + 一筆 orders 範例** | 例如：一種房型 YAML、一筆訂房資料夾與一份 `*.req.md`（如 `BR250315-001.req.md`） | 讓 Rule 有具體可指的檔案；讓待會寫的 Skill 有真實輸入可測   |
| **3. 第一支 Skill**                     | 讀取 `.req.md` + 參照 master data，產出確認單/任務清單                    | 立刻驗證「Rule + 資料 → Skill 產出」是否一致        |
| **4. 再迭代**                           | 補欄位、補資料、擴充 Skill 或報表                                        | 依實際使用調整                               |


## 若反過來會怎樣？

- **先建很多 master data / orders，再寫 rules/skills**  
容易出現：每筆訂單結構略不同、YAML 鍵名不統一。之後 Rule 要寫「請依 … 格式」時，得先回頭改既有的檔，反而多一輪整理。
- **先寫完整 rules/skills，再建資料**  
Rule/Skill 容易寫得太抽象（沒有實際檔可看），實作時才發現「少一個欄位」「路徑不對」，又要改 Rule 與 Skill。

## 結論

**先建 rules/skills 的 minimum boilerplate（至少：訂單結構、.req.md 欄位、master data 位置），**  
**再建「剛好夠用」的 master data 與一筆 orders 範例。**  
這樣 rules/skills 有依據可參照，資料也有依據可遵守，第一支 Skill 馬上能跑一輪驗證。

---

# 執行成果

## 做了哪些事

依「先最小 boilerplate，再最小資料 + 一筆訂單」的建議，實際建了以下內容：

### 1. 最小 Rule（契約）

- **檔案**：`.cursor/rules/hotel-orders.mdc`
- **內容**：
  - 約定 master data 放在 `master-data/`；訂單為 `orders/` 下的 `*.req.md`（檔名即 id），  
  產出為 `orders/<id>-確認單.md`、`orders/<id>-任務.md`；報表可放在 `analyse/`。  
  需求檔須含 YAML front matter（type、date、id），不覆蓋需求檔。
- **globs**：`orders/**/*.md`, `master-data/**/*.yaml`, `master-data/**/*.csv`，  
僅在編輯這些檔案時套用，避免干擾其他檔案。

### 2. 最小 master data + 一筆訂單範例

- **master-data/room-types.yaml**：三種房型（標準雙人房 STD-D、豪華雙床房 DEL-T、家庭四人房 FAM-Q），  
含 id、name、capacity、price_per_night、note；加購選項：早餐（200/人/日）、車位（300/車/日）。
- **master-data/room-list.yaml**：房號清單共 20 間，欄位 room_type、floor、number。  
3F 301～308 為 STD-D（8 間），4F 401～406 為 DEL-T（6 間），5F 501～506 為 FAM-Q（6 間）。
- **orders/BR250315-001.req.md**：一筆訂房需求範例（2 人、2 晚、預算約 7000），  
含 YAML（type、date、id）與內文描述。產出為同目錄下 `BR250315-001-確認單.md`、`BR250315-001-任務.md`。
- **analyse/README.md**：說明此目錄用途（日報產出），之後可在此放依 orders 彙總的報表。

### 3. 第一支 Skill（從需求產生產出）

- **位置**：`.cursor/skills/hotel-order-from-req/SKILL.md`
- **用途**：當使用者要求「依這張訂單產生確認單／任務清單」或處理 `orders/*.req.md` 時，  
引導 AI 讀取該需求檔、參照 master data，在 `orders/` 產出 `<id>-確認單.md` 與 `<id>-任務.md`。
- **產出約定**：Markdown、數字與 master data 一致、可標註客戶預算與建議方案是否相符；  
排房時可自 `room-list.yaml` 篩選對應房型之房號。

---

## 為什麼這麼做


| 選擇                             | 理由                                                                                                   |
| ------------------------------ | ---------------------------------------------------------------------------------------------------- |
| **先 Rule 再資料**                 | 先訂好「訂單資料夾、.req.md 欄位、master data 位置」，之後新增的訂單與 master data 都照同一套結構，Skill 也有明確路徑可讀。                    |
| **Rule 用 globs 不 alwaysApply** | 只有編輯 orders / master-data 相關檔時才套用，其他檔案不受影響，避免規則過度干擾。                                                 |
| **只放兩種房型、一筆 BR001**            | 起頭時剛好夠驗證；後已擴充為三種房型、加購選項（早餐/車位）、房號清單 20 間，分檔為 `room-types.yaml`（房型與價錢）與 `room-list.yaml`（房號），方便排房與算價。 |
| **Skill 寫「步驟」不寫死格式**           | 確認單、任務的具體長相可依實際使用再調；Skill 先約定「讀需求→參照 master data→產出兩份 .md」，方便之後迭代。                                   |
| **analyse 先放 README**          | 第一階段先不產日報，但保留目錄與說明，第二、三階段要加「依 orders 彙總」時可直接沿用。                                                      |


---

## 下一步可驗證的事

- 在 Cursor 中開啟 `orders/BR250315-001.req.md`，要求 AI：  
「依這張需求產生確認單與任務清單」→ 應在 `orders/` 產出 `BR250315-001-確認單.md`、`BR250315-001-任務.md`。
- 檢查產出是否符合 Rule（YAML、路徑、不覆蓋需求檔）、金額是否與 room-types.yaml 一致；  
若要排房，確認單可列出該房型可用房號（來自 room-list.yaml）。
- 通過後再依 memo-01 的階段二、三擴充訂餐與場地的 master data 與 Skill。

