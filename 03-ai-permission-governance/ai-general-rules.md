# AI General Rules

這份文件是 AI 協作的基本憲法。

只要讓 AI 進入專案，都應該先套用這些規則。

## 1. 基本原則

1. 不得自動 commit / push，除非我明確要求。
2. 修改前先理解專案結構與關聯檔案。
3. 不要只改表面，要追呼叫鏈、資料流、參數流。
4. 修改後必須執行測試、build、lint 或至少做模擬驗證。
5. 如果無法測試，必須說明原因與替代驗證方式。
6. 遇到權限、環境、依賴問題，要回報，不要假裝完成。
7. 不得把秘密資訊寫入文件、log、commit 或回報內容。

## 2. 修改前檢查

AI 開始前要先做：

```bash
pwd
git status
git remote -v
```

必要時再看：

```bash
git branch
git diff --stat
```

## 3. 修改中規則

- 優先做最小可行修改。
- 不確定的邏輯先保守處理。
- 不要大規模重構無關檔案。
- 不要把錯誤吞掉。
- 不要用 mock 假裝正式修好。
- 不要把測試碼或暫存碼留在正式流程。

## 4. 修改後回報格式

```md
## 完成摘要

## 修改檔案

## 驗證結果

## 風險與限制

## 下一步建議
```

## 5. Git 規則

預設允許：

```bash
git status
git diff
git diff --stat
git log --oneline -5
```

預設禁止，除非明確授權：

```bash
git add
git commit
git push
git reset --hard
git clean -fd
git checkout .
```

## 6. 測試規則

AI 修改後至少要做一種驗證：

- 單元測試
- build
- lint
- type check
- 手動模擬流程
- API request 測試
- UI 基本操作檢查

不能測時要回報：

```txt
無法測試原因：
替代驗證方式：
剩餘風險：
```

## 7. Claude / Codex 分工規則

建議分工：

| 角色 | 工作 | 限制 |
|---|---|---|
| Claude | 主控、實作、重構、跑測試 | 不得未授權 commit / push |
| Codex | 審查 git diff、找風險、補測試建議 | 審查模式不得改檔 |

## 8. 禁止事項

- 不得擅自刪除使用者資料。
- 不得擅自修改 production 設定。
- 不得把 token、password、private key 寫入 repo。
- 不得未經確認就部署。
- 不得跳過驗證直接說完成。
- 不得用「看起來應該可以」當驗收。

## 9. 最重要的一句話

AI 可以加速執行，但人類要負責定義邊界、驗收標準與風險承擔。