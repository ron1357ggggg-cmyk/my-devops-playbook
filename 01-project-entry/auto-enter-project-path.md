# Auto Enter Project Path

這份文件的目標是降低 context switching：打開 terminal 後，可以快速進入常用專案，不必每次手動 `cd`。

## 1. 最簡單策略：固定專案根目錄

建議所有 repo 統一放在：

macOS / Linux：

```txt
~/Projects/
```

Windows：

```txt
C:\Projects\
```

範例：

```txt
C:\Projects\line-bot
C:\Projects\my-devops-playbook
C:\Projects\codex-rate-widget
```

## 2. macOS / Linux：建立 alias

編輯 `~/.zshrc` 或 `~/.bashrc`：

```bash
alias cproj='cd ~/Projects'
alias playbook='cd ~/Projects/my-devops-playbook'
alias linebot='cd ~/Projects/line-bot'
```

套用：

```bash
source ~/.zshrc
```

使用：

```bash
playbook
```

## 3. Windows PowerShell：建立 function

打開 PowerShell profile：

```powershell
notepad $PROFILE
```

加入：

```powershell
function cproj { Set-Location "C:\Projects" }
function playbook { Set-Location "C:\Projects\my-devops-playbook" }
function linebot { Set-Location "C:\Projects\line-bot" }
```

重新開 PowerShell 後使用：

```powershell
playbook
```

如果 `$PROFILE` 不存在：

```powershell
New-Item -ItemType File -Path $PROFILE -Force
notepad $PROFILE
```

## 4. Windows CMD：建立 bat 啟動器

建立：

```txt
C:\Tools\open-playbook.bat
```

內容：

```bat
@echo off
cd /d C:\Projects\my-devops-playbook
cmd
```

之後直接執行：

```cmd
C:\Tools\open-playbook.bat
```

## 5. 啟動 AI CLI 的 bat 範例

Codex：

```bat
@echo off
cd /d C:\Projects\my-devops-playbook
codex
```

Claude：

```bat
@echo off
cd /d C:\Projects\my-devops-playbook
claude
```

## 6. macOS 啟動 Claude / Codex 範例

```bash
#!/bin/bash
cd ~/Projects/my-devops-playbook || exit 1
claude
```

或：

```bash
#!/bin/bash
cd ~/Projects/my-devops-playbook || exit 1
codex
```

給執行權限：

```bash
chmod +x open-playbook-claude.sh
```

## 7. 交給 AI 前的確認指令

任何自動進入專案的 script 最後都可以加：

```bash
pwd
git status --short
git remote -v
```

這樣可以降低 AI 改錯 repo 的機率。

## 8. 建議治理策略

不要只依賴 AI 自己找路徑。

AI 可以搜尋檔案，但「哪個 repo 是正確作戰範圍」應該由人類定義。固定路徑、固定啟動器、固定命名，是低成本但高價值的工程治理。