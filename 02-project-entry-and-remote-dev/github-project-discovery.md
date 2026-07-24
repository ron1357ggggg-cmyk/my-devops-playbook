# GitHub Project Discovery

這份文件處理一個核心問題：**AI 要開始工作之前，必須先知道專案在哪裡。**

很多問題不是 AI 不會寫，而是它根本不在正確 repo、正確 branch、正確資料夾。

## 1. 列出 GitHub 上的 repo

建議安裝 GitHub CLI 後使用：

```bash
gh repo list <GITHUB_USERNAME> --limit 50
```

範例：

```bash
gh repo list ron1357ggggg-cmyk --limit 50
```

只看 repo 名稱：

```bash
gh repo list <GITHUB_USERNAME> --limit 100 --json nameWithOwner,url,visibility
```

## 2. 搜尋 repo

```bash
gh search repos <keyword> --owner <GITHUB_USERNAME>
```

範例：

```bash
gh search repos linebot --owner ron1357ggggg-cmyk
```

## 3. Clone repo

使用 HTTPS：

```bash
git clone https://github.com/<OWNER>/<REPO>.git
```

使用 SSH：

```bash
git clone git@github.com:<OWNER>/<REPO>.git
```

範例：

```bash
git clone git@github.com:ron1357ggggg-cmyk/my-devops-playbook.git
```

## 4. 確認目前專案

進入專案後先做三件事：

```bash
pwd
git status
git remote -v
```

判斷標準：

- `pwd`：確認目前資料夾位置。
- `git status`：確認目前 branch 與異動。
- `git remote -v`：確認是不是正確 GitHub repo。

## 5. 找本機已 clone 的專案

macOS / Linux：

```bash
find ~ -maxdepth 4 -type d -name ".git" 2>/dev/null
```

Windows PowerShell：

```powershell
Get-ChildItem -Path $HOME -Directory -Recurse -Force -ErrorAction SilentlyContinue |
  Where-Object { $_.Name -eq ".git" } |
  Select-Object FullName
```

## 6. 交給 AI 前的最低資訊

每次叫 AI 接手專案，至少提供：

```txt
Repo：<OWNER>/<REPO>
本機路徑：<PROJECT_PATH>
目前目標：<TASK>
限制：不要自動 commit / push
驗證：修改後請執行測試或說明無法測試原因
```

## 7. 最常見失誤

| 問題 | 風險 |
|---|---|
| 沒確認 repo | AI 可能改錯專案 |
| 沒確認 branch | AI 可能在錯誤分支修改 |
| 沒看 `git status` | AI 可能覆蓋既有未提交變更 |
| 沒說不得 commit | AI 可能自動提交 |
| 沒說驗證方式 | AI 可能只改不測 |

## 8. 建議治理策略

把所有本機專案集中在固定根目錄，例如：

```txt
~/Projects/
C:\Projects\
```

這樣 AI、terminal、腳本、文件都能用一致路徑，降低切換成本。