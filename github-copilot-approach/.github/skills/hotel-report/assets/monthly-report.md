# 月度營運報告模板

---

# {YYYY} 年 {MM} 月 營運報告

> 產出日期：{generated_at}｜資料來源：`orders/*-確認單.md`

---

## 一、本月訂單概況

| 類型 | 訂單數 | 總營收 | 平均單價 |
|------|--------|--------|---------|
| 訂房（BR） | {br_count} 筆 | {br_revenue} 元 | {br_avg} 元 |
| 訂餐（DR） | {dr_count} 筆 | {dr_revenue} 元 | {dr_avg} 元 |
| 場地（VR） | {vr_count} 筆 | {vr_revenue} 元 | {vr_avg} 元 |
| **合計** | **{total_count} 筆** | **{total_revenue} 元** | **{total_avg} 元** |

---

## 二、訂房入住率（BR）

| 指標 | 數值 |
|------|------|
| 本月可用間晚 | {available_room_nights} 間晚 |
| 已售間晚 | {sold_room_nights} 間晚 |
| 入住率 | {occupancy_rate}% |

---

## 三、營收結構

```mermaid
pie title 本月營收占比
    "訂房 BR" : {br_revenue}
    "訂餐 DR" : {dr_revenue}
    "場地 VR" : {vr_revenue}
```

---

## 四、每日訂單量

```mermaid
xychart-beta
    title "每日訂單量"
    x-axis [{日期列表}]
    y-axis "訂單數"
    bar [{每日訂單數列表}]
```

---

## 五、本月訂單清單

| 訂單 ID | 類型 | 日期 | 客戶 | 金額 |
|---------|------|------|------|------|
| {id} | BR/DR/VR | {date} | {guest} | {total} 元 |

---
> 由 `/hotel-report` 自動產出，存入 `analyse/{YYYY-MM}-報告.md`。
