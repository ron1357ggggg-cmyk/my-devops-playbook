# SSH Basic Operations

SSH 是遠端開發的基礎通道。

它解決的不是單一指令問題，而是讓你可以從一台電腦穩定控制另一台電腦，並讓 AI CLI 在遠端主機上處理專案。

## 1. 基本連線

```bash
ssh <USER>@<HOST>
```

範例：

```bash
ssh hero@192.168.1.10
```

指定 port：

```bash
ssh -p <PORT> <USER>@<HOST>
```

## 2. 建立 SSH key

建議使用 ed25519：

```bash
ssh-keygen -t ed25519 -C "<EMAIL>"
```

預設會產生：

```txt
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

注意：

- `id_ed25519` 是私鑰，不要外流。
- `id_ed25519.pub` 是公鑰，可以放到 GitHub 或遠端主機。

## 3. 把公鑰放到遠端主機

macOS / Linux：

```bash
ssh-copy-id <USER>@<HOST>
```

如果沒有 `ssh-copy-id`，手動複製公鑰內容到遠端：

```bash
cat ~/.ssh/id_ed25519.pub
```

遠端主機放到：

```txt
~/.ssh/authorized_keys
```

## 4. 測試 GitHub SSH

```bash
ssh -T git@github.com
```

成功時通常會看到類似：

```txt
Hi <USERNAME>! You've successfully authenticated...
```

## 5. 常用 SSH config

建立或編輯：

```bash
nano ~/.ssh/config
```

範例：

```txt
Host my-mac
  HostName <HOST>
  User <USER>
  Port 22
  IdentityFile ~/.ssh/id_ed25519
```

之後可以簡化成：

```bash
ssh my-mac
```

## 6. 複製檔案

本機到遠端：

```bash
scp local-file.txt <USER>@<HOST>:/remote/path/
```

遠端到本機：

```bash
scp <USER>@<HOST>:/remote/path/file.txt ./
```

複製資料夾：

```bash
scp -r ./local-folder <USER>@<HOST>:/remote/path/
```

## 7. 遠端執行單一指令

```bash
ssh <USER>@<HOST> "pwd && git status"
```

範例：

```bash
ssh my-mac "cd ~/Projects/my-devops-playbook && git status"
```

## 8. 常見錯誤

### Permission denied publickey

可能原因：

- 公鑰沒放到遠端
- 使用錯誤私鑰
- 遠端 `authorized_keys` 權限錯誤
- SSH config 指到錯的 user / host

檢查：

```bash
ssh -v <USER>@<HOST>
```

### Connection timed out

可能原因：

- Host 不通
- Port 沒開
- 防火牆擋住
- 不在同網路或 VPN 沒連上

測試：

```bash
ping <HOST>
```

或：

```bash
nc -vz <HOST> 22
```

Windows 可用：

```powershell
Test-NetConnection <HOST> -Port 22
```

## 9. SSH 與 AI 工作流

建議遠端 AI 任務流程：

```txt
本機 terminal
  ↓ ssh
遠端主機
  ↓ tmux
專案資料夾
  ↓ claude / codex
AI 長任務
```

## 10. 安全原則

- 不要把私鑰 commit 到 repo。
- 不要把正式機帳號密碼寫在文件。
- Public repo 只放 placeholder。
- 公司環境資訊要抽象化。
- 高權限遠端操作要保留人工確認。