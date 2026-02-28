# 訂單結構契約

本目錄存放訂單需求檔（`*.req.md`）與 AI 產出的確認單、任務清單。

## 路徑約定

| 檔案類型 | 路徑格式 |
|---------|---------|
| 需求檔 | `orders/{id}.req.md` |
| 確認單 | `orders/{id}-確認單.md` |
| 任務清單 | `orders/{id}-任務.md` |

## .req.md 必備 YAML front matter

```yaml
type: 訂房 | 訂餐 | 場地    # 必填
date: YYYY-MM-DD            # 必填（訂房=入住日，訂餐=用餐日）
id: BR250315-001            # 必填，與檔名一致
guest: 王大明                # 必填，訂房/訂餐人姓名
related_order: BR250315-001  # 可選：關聯的訂房單（訂餐/場地用）
guest_list: {file}.yaml | embedded  # 可選：賓客名單來源（場地用）
```

front matter 下方為自由格式需求描述。

## 禁止事項

- 不得覆蓋或修改 `*.req.md`（由 `.claude/settings.json` 強制執行）
- 金額必須與 master-data 一致，不得自行編造價格
- 建議房號必須存在於 `master-data/room-list.yaml`

## 處理訂單

| 訂單類型 | 觸發指令 |
|---------|---------|
| 訂房（BR） | `/process-order [order-id]` |
| 訂餐（DR） | `/process-dining-order [order-id]` |
| 場地（VR） | `/process-venue-order [order-id]` |

也可委派給 order-clerk 子代理處理。

## 賓客名單約定

場地訂單的 `guest_list` front matter 決定名單來源：

| guest_list 值 | 意義 | 適用場景 |
|--------------|------|---------|
| `{id}-名單.yaml` | 外部檔案，路徑為 `orders/{id}-名單.yaml` | 大型活動（20+ 人） |
| `embedded` | 名單內嵌於 req.md 內文的 YAML 區塊 | 小型活動（~20 人以內） |
| （未填） | 無名單，僅依人數處理 | 會議等不需排桌的場景 |

名單格式（無論外部或內嵌）：
- 以 `groups` 分組，每組有 `priority` 決定排桌順序
- 每位賓客：`name`（姓名）、`title`（稱謂）、`dietary`（飲食需求，null=一般）
