# Phase 3 完成報告：場地租借

## 一、目標 vs 達成

### 待建立的檔案

| 類別 | 計畫檔案 | 狀態 |
|------|---------|------|
| 主資料 | `master-data/venues.yaml` | ✓ 完成 |
| Skill | `process-venue-order/SKILL.md` + templates/ | ✓ 完成 |
| Skill | `generate-card-prompt/SKILL.md` | ✓ 完成 |
| Agent | `print-designer.md` | ✓ 完成 |
| 範例 | `VR250420-001.req.md` + 名單 yaml | ✓ 完成 |

### 展示的新能力

| 能力 | 計畫 | 狀態 | 實作方式 |
|------|------|------|---------|
| 多 Skill 串接 | process-venue-order → generate-card-prompt | ✓ | 確認單提示 → 使用者觸發第二個 Skill |
| 跨工具協作 | Claude 產 prompt → Gemini 產圖 | ✓ | card-prompt.md 作為 AI-to-AI 的介面檔 |
| 複雜資料結構 | 名單 yaml + 桌次安排 | ✓ | groups/priority/dietary 結構 + 排桌演算法 |

---

## 二、超出原計畫的額外成果

| 項目 | 說明 |
|------|------|
| 3 種名單模式 | 外部檔案 / embedded / 無名單，由 `guest_list` front matter 宣告 |
| 4 張範例訂單 | 婚宴 120 人、會議 35 人、生日派對 18 人、雙場地婚宴 55 人 |
| 雙場地訂單 | VR250425-001：露臺證婚 + 玫瑰廳晚宴，測試跨場地計價 |
| settings.json 更新 | 加入 `allow` 規則解決動態注入權限問題 |
| Full Walkthrough | `docs/07-full-walkthrough.md`：8 步驟完整操作指引 |
| 所有現有檔案同步更新 | CLAUDE.md、readme、order-clerk、auditor、orders/CLAUDE.md、master-data/CLAUDE.md |

---

## 三、尚未實際驗證的項目

以下項目架構已就緒，但尚未在 Claude Code 中端到端執行驗證：

| 項目 | 說明 | 驗證方式 |
|------|------|---------|
| `/process-venue-order` 端到端 | Skill 已寫好但未實際跑過一張完整訂單 | 執行 `/process-venue-order VR250421-001`（最簡單的會議案例） |
| 桌次安排演算法 | 演算法規則已定義在 SKILL.md，但 AI 能否正確執行待驗證 | 執行 VR250420-001（120 人）檢查桌次表人數 = 120 |
| `/generate-card-prompt` 端到端 | 需要先有確認單才能跑 | 先完成上一步，再執行 `/generate-card-prompt VR250420-001` |
| card-prompt → Gemini 產圖 | 需要 Cursor + Gemini 環境 | 將 prompt 檔貼入 Cursor 測試 |
| auditor 場地驗證 | 新增的 5 項場地驗證規則是否正確執行 | 觀察 auditor 驗證報告 |
| 雙場地計價 | VR250425-001 跨場地是否正確加總 | 執行後檢查費用明細 |
| check-availability 權限 | settings.json allow 規則是否生效 | 執行 `/check-availability 2025-04-20 2025-04-22` |

### 建議驗證順序

```
1. /check-availability 2025-04-20 2025-04-22      ← 測試 settings.json allow
2. /process-venue-order VR250421-001               ← 最簡單：會議，無名單
3. /process-venue-order VR250422-001               ← embedded 名單，小型
4. /process-venue-order VR250420-001               ← 最複雜：120 人婚宴
5. /generate-card-prompt VR250420-001              ← 多 Skill 串接
6. /process-venue-order VR250425-001               ← 邊界案例：雙場地
7. 將 card-prompt.md 交給 Cursor + Gemini          ← 跨工具協作
```

---

## 四、Phase 3 新增的系統能力總覽

Phase 1-2 展示了 9 大 Claude 原生機制。Phase 3 新增了 3 項：

| # | 能力 | Phase | 實作 |
|---|------|-------|------|
| 1 | 指令式 Skill | 1 | `/process-order` |
| 2 | 動態上下文注入 | 1 | `/check-availability` 的 `` !`cmd` `` |
| 3 | 子代理委派 | 1 | order-clerk, auditor |
| 4 | 角色權限分離 | 1 | agent 各有不同 tools |
| 5 | 模型分配 | 1 | sonnet(處理) + haiku(稽核) |
| 6 | 信任邊界 | 1 | settings.json deny + allow |
| 7 | 目錄級規則繼承 | 1 | 子目錄 CLAUDE.md |
| 8 | 模板分離 | 1 | skill 的 templates/ |
| 9 | 參數化 Skill | 1 | $ARGUMENTS, $1, $2 |
| **10** | **多 Skill 串接** | **3** | **process-venue-order → generate-card-prompt** |
| **11** | **跨工具協作（AI-to-AI）** | **3** | **Claude 產 prompt → Gemini 產圖** |
| **12** | **複雜資料結構 + 演算法** | **3** | **名單 YAML + 桌次安排** |

---

## 五、專案演進時間線（更新）

```
memo-01~03    探索期    「Rules/Skills 能做什麼？」
memo-04       驗證期    「Cursor vs Claude 都能跑」
memo-05       比較期    「兩者的差異在哪？」
memo-06       重構期    「展示 Claude 獨有能力」         ← Phase 1 ✓
memo-07       擴充期    「訂餐流程」                    ← Phase 2 ✓
memo-08       完成期    「場地 + AI 協作 + 複雜場景」    ← Phase 3 ✓（架構完成，待驗證）
```

從「能不能用」→「差在哪裡」→「獨特在哪裡」→「能做多複雜」，
三個 Phase 完整展示了 Claude Code 從簡單規則到複雜 AI 團隊協作的全部能力。
