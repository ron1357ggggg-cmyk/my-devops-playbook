# Mac / Windows 遠端 SSH + tmux 設定 SOP

## 1. 目標架構

### 1.1 Mac 遠端開發架構

```text
另一台電腦 / 公司筆電
        ↓ SSH
你的 Mac
        ↓ tmux session
Claude / Codex / xcodebuild / 專案操作
```

### 1.2 Windows 遠端開發架構

```text
Mac / 另一台電腦 / 公司筆電
        ↓ SSH
Windows 11 Home / Pro
        ↓ WSL Ubuntu
        ↓ tmux session
Claude / Codex / npm / dotnet / git / 專案操作
```

核心價值：**SSH 是遠端入口，tmux 是任務續命層。SSH 斷線沒關係，tmux 裡面的工作還會繼續跑。**

這套流程適合：

- 遠端進 Mac 跑 Claude / Codex CLI
- 遠端進 Windows 跑 WSL / Linux CLI 工作流
- 長時間跑 `xcodebuild`
- 長時間跑 `npm run dev`、`dotnet run`、測試腳本
- 避免 SSH 斷線導致工作整個消失
- 把多台家用 Windows 建成可遠端維運節點

> 安全原則：建議使用 **Tailscale + SSH**，不要把 22 port 直接開到公網。

---

## 2. Mac 端一次性設定

### 2.1 開啟 Mac SSH

在 Mac 上開啟：

```text
系統設定 → 一般 → 共享 → 遠端登入 Remote Login → 開啟
```

或用指令：

```bash
sudo systemsetup -setremotelogin on
```

確認 SSH 有開：

```bash
sudo systemsetup -getremotelogin
```

看到：

```text
Remote Login: On
```

代表 SSH 已啟用。

---

### 2.2 確認 Mac 帳號名稱

在 Mac Terminal 輸入：

```bash
whoami
```

假設回傳：

```bash
hero
```

之後 SSH 連線格式就是：

```bash
ssh hero@你的MacIP
```

---

### 2.3 確認 Mac IP 或 Tailscale IP

如果使用 Tailscale，在 Mac 上查：

```bash
tailscale ip -4
```

會得到類似：

```text
100.xx.xx.xx
```

另一台電腦就用這個連：

```bash
ssh hero@100.xx.xx.xx
```

也可以用 Tailscale 裝置名稱：

```bash
ssh hero@你的MacTailscale名稱
```

---

## 3. Mac 安裝 tmux

在 Mac 上執行：

```bash
brew install tmux
```

確認安裝成功：

```bash
tmux -V
```

看到類似：

```text
tmux 3.x
```

代表完成。

---

## 4. Windows 11 安裝 OpenSSH Server

### 4.1 適用情境

Windows 11 Home 不能當原生 RDP 遠端桌面主機，但可以安裝 **OpenSSH Server**，讓其他電腦透過 SSH 進來操作命令列。

Windows 上的分工建議：

```text
OpenSSH Server = 讓外部電腦 SSH 進 Windows
WSL Ubuntu     = 提供 Linux CLI 執行環境
tmux           = 讓長任務不因 SSH 斷線而中斷
```

---

### 4.2 檢查 OpenSSH 狀態

用「系統管理員 PowerShell」執行：

```powershell
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
```

可能會看到：

```text
Name  : OpenSSH.Client~~~~0.0.1.0
State : Installed

Name  : OpenSSH.Server~~~~0.0.1.0
State : NotPresent
```

意思是：

```text
OpenSSH.Client = 這台 Windows 可以 SSH 連別人
OpenSSH.Server = 這台 Windows 還不能被別人 SSH 進來
```

---

### 4.3 安裝 OpenSSH Server

系統管理員 PowerShell：

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

安裝時如果顯示 `Operation Running`，而且進度條還有動，就先不要中斷。

建議等待區間：

```text
正常：幾分鐘內完成
偏慢：10～30 分鐘仍可能正常
異常：超過 30 分鐘完全沒動，再考慮中斷與重試
```

裝完後再次確認：

```powershell
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
```

確認看到：

```text
OpenSSH.Server~~~~0.0.1.0
State : Installed
```

---

### 4.4 啟動 SSH 服務

```powershell
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
Get-Service sshd
```

正常會看到：

```text
Status   Name
Running  sshd
```

`Set-Service -StartupType Automatic` 的目的，是讓 Windows 重開機後自動啟動 SSH Server。

---

### 4.5 確認防火牆規則

```powershell
Get-NetFirewallRule -Name "OpenSSH-Server-In-TCP" -ErrorAction SilentlyContinue
```

如果查不到規則，再補這段：

```powershell
New-NetFirewallRule -Name "OpenSSH-Server-In-TCP" -DisplayName "OpenSSH Server (sshd)" -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
```

> 注意：這只是開本機防火牆。不要再到路由器做 port forwarding 把 22 port 開到公網。

---

### 4.6 查 Windows 使用者名稱

PowerShell：

```powershell
whoami
```

可能會看到：

```text
bankpro\ron_li
```

SSH 登入時通常先用後面的帳號：

```bash
ssh ron_li@Windows_IP
```

如果失敗，再試完整網域格式：

```bash
ssh "bankpro\ron_li@Windows_IP"
```

---

### 4.7 查 Windows IP

區網 IP：

```powershell
ipconfig
```

如果使用 Tailscale，查 Tailscale IP：

```powershell
tailscale ip -4
```

建議優先使用 Tailscale IP 或 Tailscale 裝置名稱連線。

---

### 4.8 從 Mac / 另一台電腦 SSH 進 Windows

```bash
ssh Windows帳號@Windows_IP
```

例如：

```bash
ssh ron_li@10.0.0.2
```

如果是 Tailscale：

```bash
ssh ron_li@100.xx.xx.xx
```

或：

```bash
ssh ron_li@win-main
```

第一次連線可能會問：

```text
Are you sure you want to continue connecting?
```

輸入：

```text
yes
```

---

## 5. Windows 安裝 WSL Ubuntu + tmux

### 5.1 為什麼 Windows 上建議透過 WSL 裝 tmux

Windows 可以裝 OpenSSH Server，但 `tmux` 不是 Windows 原生標準工具。

最穩的做法是：

```text
SSH 進 Windows
        ↓
進 WSL Ubuntu
        ↓
使用 tmux
```

這樣 Linux CLI、Claude / Codex、git、npm、腳本工具都會比較一致。

---

### 5.2 安裝 WSL Ubuntu

系統管理員 PowerShell：

```powershell
wsl --install -d Ubuntu
```

如果系統要求重開機，就先重開。

重開後，第一次打開 Ubuntu 會要求建立 Linux 使用者名稱與密碼。

---

### 5.3 在 Ubuntu 安裝 tmux 與常用工具

進入 Ubuntu 後：

```bash
sudo apt update
sudo apt install -y tmux git curl
```

確認 tmux：

```bash
tmux -V
```

看到類似：

```text
tmux 3.x
```

代表完成。

---

### 5.4 從 SSH 進 Windows 後切進 WSL

SSH 進 Windows 後，執行：

```powershell
wsl
```

進入 Ubuntu 後，再開 tmux：

```bash
tmux new -s dev
```

日常建議直接用這行：

```bash
tmux attach -t dev || tmux new -s dev
```

意思是：

```text
有 dev session → 直接接回
沒有 dev session → 建立新的 dev session
```

---

## 6. 從另一台電腦 SSH 進 Mac

在另一台電腦開 PowerShell、Terminal 或 Windows Terminal：

```bash
ssh 你的Mac帳號@你的MacTailscaleIP
```

例如：

```bash
ssh hero@100.xx.xx.xx
```

第一次連線可能會問：

```text
Are you sure you want to continue connecting?
```

輸入：

```text
yes
```

登入後，先確認現在真的在 Mac 裡：

```bash
whoami
hostname
pwd
uname -a
```

如果 `hostname` 是你的 Mac 名稱，代表現在操作的是遠端 Mac。

---

## 7. 建立 tmux 工作區

### 7.1 Mac 建立 tmux session

第一次建立 session：

```bash
tmux new -s ios-dev
```

建議固定 session 名稱，例如：

```text
ios-dev
```

進入 tmux 後，再啟動實際工作：

```bash
claude
```

或：

```bash
codex
```

或：

```bash
xcodebuild ...
```

---

### 7.2 Windows / WSL 建立 tmux session

SSH 進 Windows 後：

```powershell
wsl
```

進 Ubuntu 後：

```bash
tmux attach -t dev || tmux new -s dev
```

再啟動實際工作：

```bash
claude
```

或：

```bash
codex
```

或：

```bash
npm run dev
```

或：

```bash
dotnet run
```

---

## 8. tmux 日常操作

### 8.1 離開但不中斷工作

按：

```text
Ctrl + b
```

放開後再按：

```text
d
```

這叫 `detach`。

意思是：**人離開，但工作還在遠端電腦裡跑。**

---

### 8.2 下次重新接回

Mac：

```bash
tmux attach -t ios-dev
```

Windows / WSL：

```bash
tmux attach -t dev
```

如果忘記 session 名稱：

```bash
tmux ls
```

會看到類似：

```text
ios-dev: 1 windows
dev: 1 windows
```

---

### 8.3 如果 session 不存在

如果看到：

```text
no sessions
```

代表之前沒有開，或已經被關掉。

重新建立：

```bash
tmux new -s dev
```

或 Mac iOS 開發用：

```bash
tmux new -s ios-dev
```

---

### 8.4 關掉 tmux session

在 tmux 裡面直接輸入：

```bash
exit
```

或從外部指定關閉：

```bash
tmux kill-session -t dev
```

---

## 9. 多台 Windows 建議命名與連線策略

如果家裡有多台 Windows，建議固定命名，不要只靠 IP 記憶。

範例：

```text
win-main     主力 Windows
win-home-1   家裡第一台 Windows
win-home-2   家裡第二台 Windows
macbook      Mac
```

使用 Tailscale 時，可以用裝置名稱連線：

```bash
ssh ron_li@win-main
ssh ron_li@win-home-1
ssh ron_li@win-home-2
```

建議標準化成：

```text
Tailscale = 私有網路層
SSH       = 遠端入口層
WSL       = Linux 執行層
tmux      = 任務續命層
Git       = 專案同步層
Claude/Codex = AI 執行與審查層
```

---

## 10. 常見問題與解法

### 問題 1：SSH / 遠端連線一斷，Claude、Codex、build 就消失

原因：

如果直接在 SSH 裡跑 Claude / Codex / xcodebuild / npm / dotnet，SSH 斷線時，Shell 可能會一起結束。

解法：

一定要先進 tmux：

```bash
tmux new -s dev
```

或接回既有 session：

```bash
tmux attach -t dev
```

再啟動 Claude / Codex / build。

---

### 問題 2：以為開了 SSH / Tailscale 就等於同 Wi-Fi，可做所有 iOS 部署

實務結論：

Tailscale 可以讓你像在區網內一樣 SSH 進 Mac，但不是所有 iOS 部署限制都會消失。

比較正確的模型是：

```text
你遠端控制 Mac
Mac 負責 Xcode build / install / 操作 iPhone
```

所以 iPhone 還是要跟那台 Mac 有配對、接線，或符合 Xcode 無線部署條件。

---

### 問題 3：不知道自己目前是不是在遠端 Mac / Windows / WSL

通用確認：

```bash
whoami
hostname
pwd
```

Mac / Linux 可再加：

```bash
uname -a
```

Windows PowerShell 可用：

```powershell
whoami
hostname
Get-Location
```

如果已經進入 WSL Ubuntu，`uname -a` 會顯示 Linux 核心資訊。

---

### 問題 4：tmux 指令找不到

錯誤訊息可能是：

```text
tmux: command not found
```

Mac 解法：

```bash
brew install tmux
```

Windows / WSL 解法：

```bash
sudo apt update
sudo apt install -y tmux
```

---

### 問題 5：attach 失敗

常見錯誤：

```text
can't find session: dev
```

解法：

先看目前有哪些 session：

```bash
tmux ls
```

如果真的沒有，就重開：

```bash
tmux new -s dev
```

---

### 問題 6：SSH 第一次連線問 yes/no

第一次連線時，SSH 會要求確認主機 fingerprint：

```text
Are you sure you want to continue connecting?
```

輸入：

```text
yes
```

之後會把這台主機記到 `known_hosts`，通常下次就不會再問。

---

### 問題 7：遠端操作時不確定專案在哪

可以先用常見位置找：

Mac / Linux / WSL：

```bash
ls ~/Desktop
ls ~/Documents
ls ~/Developer
ls ~/Projects
```

也可以用 find 搜尋：

```bash
find ~ -maxdepth 4 -type d -name ".git" 2>/dev/null
```

看到 `.git` 的上一層通常就是專案根目錄。

Windows PowerShell：

```powershell
Get-ChildItem $HOME -Recurse -Directory -Filter .git -ErrorAction SilentlyContinue
```

---

### 問題 8：Windows 安裝 OpenSSH Server 卡很久

如果 `Add-WindowsCapability` 顯示 `Operation Running`，且進度條仍有動，先不要中斷。

如果超過 30 分鐘完全沒動，可以中斷後檢查狀態：

```powershell
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
```

若仍是 `NotPresent`，可改用 DISM：

```powershell
DISM /Online /Add-Capability /CapabilityName:OpenSSH.Server~~~~0.0.1.0
```

或重啟 Windows Update 相關服務後再試：

```powershell
Restart-Service wuauserv -Force
Restart-Service bits -Force
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

---

## 11. 建議固定流程

### 11.1 遠端操作 Mac

```bash
ssh 你的Mac帳號@你的MacTailscaleIP
tmux attach -t ios-dev || tmux new -s ios-dev
```

然後才開始做事：

```bash
cd 你的專案路徑
claude
```

或：

```bash
codex
```

或：

```bash
xcodebuild ...
```

---

### 11.2 遠端操作 Windows

```bash
ssh Windows帳號@WindowsTailscaleIP
wsl
tmux attach -t dev || tmux new -s dev
```

然後才開始做事：

```bash
cd 你的專案路徑
claude
```

或：

```bash
codex
```

或：

```bash
npm run dev
```

或：

```bash
dotnet run
```

---

## 12. 一句話記法

```text
先用 Tailscale 建私有網路，再 SSH 到目標機，進 tmux 後才開 Claude / Codex / build。
```

這樣就算公司網路斷線、遠端視窗關掉、筆電睡眠，遠端電腦上的任務也不會直接被砍掉。
