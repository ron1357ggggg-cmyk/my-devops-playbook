# Windows OpenSSH Server + WSL tmux Setup

把一台 Windows 11 機器變成可遠端 SSH 進來、並在 WSL Ubuntu 裡跑 tmux 續命長任務的節點。

分工模型：

```txt
OpenSSH Server = 讓外部電腦 SSH 進 Windows
WSL Ubuntu     = 提供 Linux CLI 執行環境
tmux           = 讓長任務不因 SSH 斷線而中斷
```

裝好之後接 [SSH Basic Operations](ssh-basic-operations.md) 建連線、[SSH + tmux Persistent Session](ssh-tmux-persistent-session.md) 續任務。

## 1. 檢查 OpenSSH 狀態

系統管理員 PowerShell：

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

`OpenSSH.Client` = 這台 Windows 可以 SSH 連別人；`OpenSSH.Server` = 這台 Windows 還不能被別人 SSH 進來。

## 2. 安裝 OpenSSH Server

系統管理員 PowerShell：

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

如果顯示 `Operation Running` 且進度條還有動，先不要中斷。

```text
正常：幾分鐘內完成
偏慢：10～30 分鐘仍可能正常
異常：超過 30 分鐘完全沒動，再考慮中斷與重試
```

裝完後再次確認狀態變成 `Installed`。

## 3. 啟動 SSH 服務

```powershell
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
Get-Service sshd
```

正常會看到 `Running`。`-StartupType Automatic` 讓 Windows 重開機後自動啟動 SSH Server。

## 4. 確認防火牆規則

```powershell
Get-NetFirewallRule -Name "OpenSSH-Server-In-TCP" -ErrorAction SilentlyContinue
```

查不到就補上：

```powershell
New-NetFirewallRule -Name "OpenSSH-Server-In-TCP" -DisplayName "OpenSSH Server (sshd)" -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
```

> 注意：這只是開本機防火牆。不要再到路由器做 port forwarding 把 22 port 開到公網，建議搭配 Tailscale 使用。

## 5. 查登入帳號與 IP

```powershell
whoami
```

如果回傳 `<DOMAIN>\<USER>` 網域格式，SSH 登入時先試短帳號：

```bash
ssh <USER>@<WINDOWS_IP>
```

失敗再試完整網域格式：

```bash
ssh "<DOMAIN>\<USER>@<WINDOWS_IP>"
```

查 IP：

```powershell
ipconfig
```

有 Tailscale 就優先用 Tailscale IP 或裝置名稱：

```powershell
tailscale ip -4
```

## 6. 安裝 WSL Ubuntu

Windows 可以裝 OpenSSH Server，但 `tmux` 不是 Windows 原生標準工具，最穩的做法是 SSH 進 Windows 後再進 WSL Ubuntu 用 tmux，這樣 Linux CLI、Claude / Codex、git、npm 都會比較一致。

系統管理員 PowerShell：

```powershell
wsl --install -d Ubuntu
```

如果要求重開機，先重開。重開後第一次打開 Ubuntu 會要求建立 Linux 使用者名稱與密碼。

## 7. 在 Ubuntu 安裝 tmux 與常用工具

```bash
sudo apt update
sudo apt install -y tmux git curl
tmux -V
```

## 8. 從 SSH 進 Windows 後切進 WSL + tmux

SSH 進 Windows 後：

```powershell
wsl
```

進入 Ubuntu 後接回或新建 session：

```bash
tmux attach -t dev || tmux new -s dev
```

## 9. 多台 Windows 的命名建議

如果有多台 Windows，建議固定命名而不是只靠 IP 記憶：

```text
win-main     主力 Windows
win-home-1   家裡第一台 Windows
win-home-2   家裡第二台 Windows
```

用 Tailscale 裝置名稱連線：

```bash
ssh <USER>@win-main
```

## 10. 常見問題：安裝卡很久

如果 `Add-WindowsCapability` 顯示 `Operation Running` 且進度條仍有動，先不要中斷。

超過 30 分鐘完全沒動，可以中斷後檢查狀態，若仍是 `NotPresent` 可改用 DISM：

```powershell
DISM /Online /Add-Capability /CapabilityName:OpenSSH.Server~~~~0.0.1.0
```

或重啟 Windows Update 相關服務後再試：

```powershell
Restart-Service wuauserv -Force
Restart-Service bits -Force
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```
