# Claude / Codex Review Workflow

這份文件定義兩個 AI coding agent 的企業級分工流程。

核心概念：

```txt
Claude 負責主控與實作。
Codex 負責審查 git diff。
人類負責目標、邊界與驗收。
```

## 1. 為什麼要分工

單一 AI 容易出現：

- 修改範圍過大
- 自認完成但沒驗證
- 沒注意回歸風險
- 忽略安全或邊界條件
- 對自己產出的程式過度樂觀

用第二個 AI 做 diff review，可以建立基本制衡。

## 2. 角色定義

| 角色 | 責任 | 禁止事項 |
|---|---|---|
| Claude | 需求理解、修改程式、執行測試、整理交付 | 不得自動 commit / push |
| Codex | 審查 git diff、指出風險、提出修正建議 | 審查模式不得修改檔案 |
| Human | 定義任務、授權權限、判斷是否交付 | 不應跳過驗收 |

## 3. 標準流程

```txt
1. Human 下任務
2. Claude 讀取專案結構
3. Claude 提出理解與計畫
4. Claude 修改程式
5. Claude 執行測試或模擬驗證
6. Claude 產生 git diff
7. Codex 審查 git diff
8. Codex 輸出 AI_REVIEW.md
9. Claude 根據審查修正
10. 最多重複 3 輪
11. Claude 輸出 final delivery report
12. Human 決定是否 commit / push
```

## 4. Claude 主控 prompt

```txt
請讀取 AI_TASK.md 並開始執行。

你是主控 AI，負責實作與協調 Codex CLI 審查。

執行規則：
1. 你負責修改程式。
2. Codex 只負責審查，不准修改。
3. 每次完成一輪修改後，必須呼叫 Codex CLI 審查 git diff。
4. Codex 的審查結果寫入 AI_REVIEW.md。
5. 根據 AI_REVIEW.md 修正問題。
6. 最多進行 3 輪。
7. 不得自動 commit / push。
8. 修改後必須執行測試；若不能測試，必須說明原因。
9. 最後輸出完整交付報告。
```

## 5. Codex 審查 prompt

```txt
你現在是審查 AI，只能審查，不准修改檔案。

請審查目前 git diff。

審查重點：
1. 是否有破壞既有功能。
2. 是否有安全性或資料外洩風險。
3. 是否有未處理的錯誤流程。
4. 是否有 race condition、timeout、null、edge case。
5. 是否有過度設計或重複邏輯。
6. 是否缺少測試。
7. 是否符合需求。

請輸出到 AI_REVIEW.md：
- Blocker
- Major
- Minor
- 建議修正方向
- 是否建議進入下一輪
```

## 6. 建議檔案：AI_TASK.md

```md
# AI Task

## 任務目標

## 背景

## 驗收條件

## 限制

- 不得自動 commit / push
- 不得修改無關模組
- 不得上傳 secrets

## 測試要求

## 交付格式
```

## 7. 建議檔案：AI_REVIEW.md

```md
# AI Review

## Review Round 1

### Blocker

### Major

### Minor

### 建議修正

### 結論
```

## 8. 建議指令

Claude 修改後：

```bash
git status
git diff --stat
git diff
```

讓 Codex 審查：

```bash
git diff | codex "Review this diff. Do not modify files. Output risks and suggested fixes."
```

或在 Codex CLI 內下：

```txt
請審查目前 git diff，只審查，不修改檔案。
```

## 9. 退出條件

符合任一條件就停止自動循環，交給人類決策：

- 超過 3 輪仍有 blocker
- 測試環境缺失
- 權限不足
- 需求不明確
- 涉及 production / 金流 / 個資 / 安全敏感流程

## 10. 最終交付報告格式

```md
# Final Delivery Report

## 任務摘要

## 修改範圍

## 驗證結果

## Codex Review 結論

## 剩餘風險

## 建議下一步
```

## 11. 結論

Claude / Codex 交叉驗證的目的不是追求形式，而是建立「實作與審查分離」的治理模型。

這套流程特別適合：

- 大型重構
- API 串接
- 付款 / 保險 / 會員流程
- CLI 工具開發
- 需要降低回歸風險的修改