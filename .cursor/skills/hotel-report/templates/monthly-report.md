# UU大飯店 {YYYY}年{MM}月 營運報告

> 產出時間：{generated_at}
> 資料來源：`orders/*-確認單.md`（日期範圍：{YYYY}-{MM}-01 ～ {YYYY}-{MM}-{末日}）

---

## 一、營收總覽

| 指標 | 數值 |
|------|------|
| 總營收 | {total_revenue} 元 |
| 訂單數 | {order_count} 筆（訂房 {br_count} / 訂餐 {dr_count} / 場地 {vr_count}） |
| 平均客單價 | {avg_revenue} 元 |
| 總服務人次 | {total_guests} 人次 |
| 預算達標率 | {budget_rate}%（{budget_pass}/{order_count} 筆在預算內） |

---

## 二、營收結構

```mermaid
pie title 營收結構
    "訂房 ({br_revenue}元)" : {br_revenue}
    "訂餐 ({dr_revenue}元)" : {dr_revenue}
    "場地 ({vr_revenue}元)" : {vr_revenue}
```

---

## 三、訂單明細

| # | 訂單編號 | 類型 | 客戶 | 日期 | 人數 | 金額 | 預算 |
|---|---------|------|------|------|------|------|------|
| 1 | {id} | {type} | {guest} | {date} | {guests}位 | {total}元 | {budget_result} |
| ... | | | | | | | |
| **合計** | | **{order_count} 筆** | | | **{total_guests} 人次** | **{total_revenue} 元** | |

---

## 四、客房分析

<!-- 僅當有 BR 訂單時顯示本區塊，否則顯示「本月無訂房訂單」 -->

| 指標 | 數值 |
|------|------|
| 客房收入 | {br_revenue} 元 |
| 訂房筆數 | {br_count} 筆 |
| 總房晚數 | {br_nights} 晚 |
| 平均房價（ADR） | {adr} 元/晚 |
| 入住人次 | {br_guests} 人次 |

### 房型分布

```mermaid
pie title 房型分布
    "{room_type_1}" : {count_1}
    "{room_type_2}" : {count_2}
```

---

## 五、餐飲分析

<!-- 僅當有 DR 訂單時顯示本區塊，否則顯示「本月無訂餐訂單」 -->

| 指標 | 數值 |
|------|------|
| 餐飲收入 | {dr_revenue} 元 |
| 訂餐筆數 | {dr_count} 筆 |
| 用餐人次 | {dr_guests} 人次 |
| 平均每人消費 | {dr_avg_per_person} 元/人 |

---

## 六、場地分析

<!-- 僅當有 VR 訂單時顯示本區塊，否則顯示「本月無場地訂單」 -->

| 指標 | 數值 |
|------|------|
| 場地收入 | {vr_revenue} 元 |
| 場地筆數 | {vr_count} 筆 |
| 服務人次 | {vr_guests} 人次 |

### 場地使用次數

```mermaid
xychart-beta
    title "場地使用次數"
    x-axis [{venue_names}]
    y-axis "次數" 0 --> {max_venue_count}
    bar [{venue_counts}]
```

### 加購項目排行

| 排名 | 加購項目 | 次數 | 金額小計 |
|------|---------|------|---------|
| 1 | {option_name} | {option_count} | {option_total} 元 |

---

> 本報告由 `@hotel-report` 自動產出，資料來源為已完成的確認單。
