# Mac 遠端 SSH + tmux 設定 SOP

## 1. 目標架構

```text
另一台電腦 / 公司筆電
        ↓ SSH
你的 Mac
        ↓ tmux session
Claude / Codex / xcodebuild / 專案操作
```

核心價值：**SSH 斷線沒關係，tmux 裡面的工作還會繼續跑。**

這套流程適合：

- 遠端進 Mac 跑 Claude / Codex CLI
- 長時間跑 `xcodebuild`
- 遠端處理 iOS / macOS 專案
- 避免 SSH 斷線導致工作整個消失

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

## 3. 安裝 tmux

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

## 4. 從另一台電腦 SSH 進 Mac

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

## 5. 建立 tmux 工作區

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

## 6. tmux 日常操作

### 6.1 離開但不中斷工作

按：

```text
Ctrl + b
```

放開後再按：

```text
d
```

這叫 `detach`。

意思是：**人離開，但工作還在 Mac 裡跑。**

---

### 6.2 下次重新接回

SSH 進 Mac 後：

```bash
tmux attach -t ios-dev
```

如果忘記 session 名稱：

```bash
tmux ls
```

會看到類似：

```text
ios-dev: 1 windows
```

再接回：

```bash
tmux attach -t ios-dev
```

---

### 6.3 如果 session 不存在

如果看到：

```text
no sessions
```

代表之前沒有開，或已經被關掉。

重新建立：

```bash
tmux new -s ios-dev
```

---

## 7. 常見問題與解法

### 問題 1：SSH / 遠端連線一斷，Claude、Codex、build 就消失

原因：

如果直接在 SSH 裡跑 Claude / Codex / xcodebuild，SSH 斷線時，Shell 會一起結束。

解法：

一定要先進 tmux：

```bash
tmux new -s ios-dev
```

或接回既有 session：

```bash
tmux attach -t ios-dev
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

### 問題 3：不知道自己目前是不是在遠端 Mac

登入後用這幾個確認：

```bash
whoami
hostname
pwd
uname -a
```

如果 `hostname` 是你的 Mac 名稱，就代表你現在操作的是遠端 Mac。

---

### 問題 4：tmux 指令找不到

錯誤訊息可能是：

```text
tmux: command not found
```

代表 Mac 還沒裝 tmux。

解法：

```bash
brew install tmux
```

如果連 `brew` 都沒有，就要先安裝 Homebrew。

---

### 問題 5：attach 失敗

常見錯誤：

```text
can't find session: ios-dev
```

解法：

先看目前有哪些 session：

```bash
tmux ls
```

如果真的沒有，就重開：

```bash
tmux new -s ios-dev
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

之後會把這台 Mac 記到 `known_hosts`，通常下次就不會再問。

---

### 問題 7：遠端操作時不確定專案在哪

可以先用常見位置找：

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

---

## 8. 建議固定流程

每次遠端操作 Mac 時，固定照這樣：

```bash
ssh 你的Mac帳號@你的MacTailscaleIP
tmux attach -t ios-dev
```

如果 attach 失敗：

```bash
tmux new -s ios-dev
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

## 9. 一句話記法

```text
先 SSH 到 Mac，再進 tmux，最後才開 Claude / Codex / build。
```

這樣就算公司網路斷線、遠端視窗關掉、筆電睡眠，Mac 上的任務也不會直接被砍掉。
