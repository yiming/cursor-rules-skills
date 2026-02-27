# Cursor Rules + Skills 練習
- 為了以最小最簡單的方式, 練習/驗證 rules/skills的成效.
- 先了解: cursor rules / skills的觀念, 實作方法
- 以一個非程式的簡單範例, 驗證rules/skills是可行, 有效的

# 範例
- 假設一家飯店, 要以rules/skills管理其客戶訂單
- 不靠任何程式, 資料庫, 能做到什麼程度?
- 資料格式: .md(report, doc), .yaml(master data, configuration), .csv(master data), 
- 儘量不要: .json, 因為不具備"可視覺化, 易讀性"

## master data
- 服務內容: 如: 
  - 訂房, 
  - 訂餐:訂位/訂餐, 
  - 場地:宴會/會議
- 有master data
  - 以yaml, csv儲存

## orders
- 分類訂單, 每張訂單一個資料夾
- 建立:客戶需求.req.md, 包含yaml meta data
- 依需求產生相關文件.md, 如:
  - req : 3人訂餐廳: 訂哪些? 預算? 飲食偏好
  - order : 產生建議菜單, 可人工編輯
  - task : 訂單狀態管理

## analyse
- 依據orders做日報表

# 評估與建議
- 以上想法的可行性? 請批評/建議
- 可以如何建立: rules, skills
- 如何在日常工作調用 rules, skills
- 如何: 調整 master-data, configuration 做為基本資料, 影響orders
- 如何: 調整/擴充: rules, skills?
- 如何: 統計orders資料?

---

## 這個例子適不適合作為 RULES/SKILLS 練習？

**優點**
- 情境具體（訂房/訂餐/場地），容易想像輸入與產出。
- 有 master data + orders + 報表，能練到「約定格式（Rule）」與「從 A 產生 B（Skill）」。
- 不用寫程式，符合「非程式、簡單驗證」的目標。

**代價與風險**
- 領域稍重：要先定義房型、菜單、價目等 master data，才能讓「建議菜單」這類 Skill 有意義。
- 一次涵蓋三種服務（訂房、訂餐、場地），第一次做容易失焦，較難「最小」驗證。

**結論**：採**三階段執行**（訂房 → 訂餐 → 場地），每階段都有明確的 master data 與產出，
便於對應並證明 RULES/SKILLS 的作用。

## 為何不選讀書筆記、會議記錄？

| 考量 | 讀書筆記 / 會議記錄 | 飯店訂單（三階段） |
|------|---------------------|---------------------|
| **預先準備資料** | 難有 master data（每次內容不同、難以標準化） | 房型、菜單、場地、價目等可事先建好 YAML/CSV |
| **效益驗證** | 效益評估不明確，沒有數字可對照 | 訂單數、房晚、桌數、營收等可統計，能驗證「改 Rule/Skill → 產出是否一致、報表是否正確」 |

因此以**訂房 → 訂餐 → 場地**三階段實作，用明確資料與數字證明 RULES 與 SKILLS 的成效。

---

# 下一步 / 實作檢查清單

## 共通（先做一次）
- [ ] 確認 Cursor rules / skills 的安裝位置與載入方式（`.cursor/rules/`、skills 目錄）
- [ ] 建立專案資料夾結構：`master-data/`、`orders/`、`analyse/`
- [ ] 訂定訂單通用 Rule：資料夾命名、`.req.md` 結構、YAML 欄位約定
- [ ] 記錄日常工作中如何觸發 rules/skills（快捷詞、@ 提及、Agent 預設指令）

## 第一階段：訂房
- [ ] master data：房型 YAML（或 CSV）、房價、可售數量
- [ ] Skill：從「訂房需求.req.md」參照 master data 產生確認單、任務清單
- [ ] 1～2 筆範例訂房跑一輪，檢查產出與數字是否正確
- [ ] analyse：訂房日報（房晚、入住數等）

## 第二階段：訂餐
- [ ] master data：菜單/價目、訂位規則（桌數、時段）
- [ ] Skill：從「訂餐需求.req.md」產生建議菜單、訂單、任務
- [ ] 範例訂單驗證，analyse：訂餐日報（桌數、人數、營收等）

## 第三階段：場地
- [ ] master data：場地類型、容納人數、檔期/價目
- [ ] Skill：從「場地需求.req.md」產生確認單、任務、合約要項
- [ ] 範例驗證，analyse：場地日報（場次、人數、營收等）