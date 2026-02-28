# 執行方法

在 Claude Code 中，先 `cd` 到 `pure-claude-approach/` 作為工作目錄，然後可選以下方式：
```
cd pure-claude-approach
```

---

## 方法一：/process-order 指令（推薦）

```
/process-order BR250420-001
/process-order BR250420-002
/process-order BR250505-001
```

## 方法二：自然語言

請處理 `orders/BR250420-001.req.md`，產生確認單與任務清單。

## 方法三：批次

請依序處理以下三張訂單，各產生確認單與任務清單：

- BR250420-001
- BR250420-002
- BR250505-001

---

## 查空房

```
/check-availability 2025-04-20 2025-04-22
```

---

> **重點**：`/process-order` 是 SKILL.md 中定義的 `user-invocable: true` 指令，在 Claude Code CLI 中直接打 `/process-order` 就會出現在選單裡，和 `/commit` 一樣的操作方式。
