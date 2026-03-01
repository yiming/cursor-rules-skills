# UU大飯店訂單管理系統 — 任務追蹤

> 最後更新：2026-03-01

---

## 建置進度

| Phase | 項目 | 狀態 |
|-------|------|------|
| 1 | `master-data/room-types.yaml` | ✅ 完成 |
| 1 | `master-data/room-list.yaml` | ✅ 完成 |
| 1 | `master-data/menu.yaml` | ✅ 完成 |
| 1 | `master-data/venues.yaml` | ✅ 完成 |
| 1 | `orders/` 目錄 | ✅ 完成 |
| 1 | `analyse/` 目錄 | ✅ 完成 |
| 2 | `.github/skills/hotel-order-from-req/assets/br-task.md` | ✅ 完成 |
| 2 | `.github/skills/hotel-order-from-req/assets/dr-task.md` | ✅ 完成 |
| 2 | `.github/skills/hotel-order-from-req/assets/vr-task.md` | ✅ 完成 |
| 3 | `.github/hooks/protect-readonly.json` | ✅ 完成 |
| 4 | `orders/BR260301-001.req.md`（測試：訂房） | ✅ 完成 |
| 4 | `orders/DR260301-001.req.md`（測試：訂餐） | ✅ 完成 |
| 4 | `orders/VR260301-001.req.md`（測試：場地+賓客名單） | ✅ 完成 |

---

## Phase 5 — 端到端驗收

在 VS Code + Copilot Agent Mode 中依序執行以下步驟：

### 查詢空房

- [ ] 執行 `/hotel-check-availability 2026-03-01 ~ 2026-03-03`
- [ ] 確認能列出依房型分組的可用房間清單

### 訂房流程（BR260301-001）

- [ ] 執行 `/hotel-order-from-req BR260301-001`
- [ ] 產出 `orders/BR260301-001-確認單.md`（含費用明細，金額可追溯 room-types.yaml）
- [ ] 產出 `orders/BR260301-001-任務.md`（含前台、房務、餐廳三部門任務）
- [ ] hotel-validator 回傳稽核通過訊息

### 訂餐流程（DR260301-001）

- [ ] 執行 `/hotel-order-from-req DR260301-001`
- [ ] 產出 `orders/DR260301-001-確認單.md`（低消檢查通過或給追加建議）
- [ ] 產出 `orders/DR260301-001-任務.md`（含素食標注、過敏原警示）
- [ ] hotel-validator 回傳稽核通過訊息

### 場地流程（VR260301-001）

- [ ] 執行 `/hotel-order-from-req VR260301-001`
- [ ] 產出 `orders/VR260301-001-確認單.md`（含桌次安排、費用明細）
- [ ] 產出 `orders/VR260301-001-任務.md`（含宴會廳、桌牌提醒、VIP接待）
- [ ] 產出 `orders/VR260301-001-名單.yaml`（含桌號分配）
- [ ] hotel-validator 回傳稽核通過訊息
- [ ] Copilot 提示執行 `/hotel-generate-card-prompt VR260301-001`

### 桌牌 Prompt

- [ ] 執行 `/hotel-generate-card-prompt VR260301-001`
- [ ] 產出 `orders/VR260301-001-card-prompt.md`

### Hook 防護測試

- [ ] 要求 Copilot 修改 `orders/BR260301-001.req.md`
- [ ] 確認 Hook 攔截並顯示唯讀錯誤訊息（而非 LLM 文字拒絕）
- [ ] 要求 Copilot 修改 `master-data/room-types.yaml`
- [ ] 確認 Hook 攔截

### 月度報告

- [ ] 執行 `/hotel-report 2026-03`
- [ ] 產出 `analyse/2026-03-報告.md`（含 Mermaid 圓餅圖與長條圖）

---

## 已知事項

- Hooks 格式（`protect-readonly.json`）為實驗性功能，實際觸發行為需依當時 Copilot 版本驗證
- 若 Hooks 不生效，唯讀保護仍由 `hotel-master-data.instructions.md` 和 `AGENTS.md` 的 LLM 指令作為後備
