# Install Codex CLI

Codex CLI 是 OpenAI 的 terminal coding agent，適合在本機 repo 內協助讀檔、改檔、執行測試與審查程式。

官方 repo：`https://github.com/openai/codex`

## 1. macOS / Linux 安裝

官方 quickstart 建議：

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```

安裝後啟動：

```bash
codex
```

## 2. Windows PowerShell 安裝

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 | iex"
```

安裝後啟動：

```powershell
codex
```

## 3. npm 安裝方式

如果本機 Node.js / npm 已經穩定，也可以用：

```bash
npm install -g @openai/codex
```

確認：

```bash
codex --version
codex
```

## 4. Homebrew 安裝方式

macOS 也可用：

```bash
brew install --cask codex
```

## 5. 第一次登入

啟動後通常會要求登入：

```bash
codex
```

建議選擇 ChatGPT 帳號登入，讓 Codex 使用目前方案權限。

## 6. 在專案內啟動

不要在隨機資料夾啟動。先進入專案根目錄：

```bash
cd <PROJECT_PATH>
git status
codex
```

## 7. 驗收清單

- [ ] `codex` 指令可執行
- [ ] 已登入成功
- [ ] 可讀取目前專案檔案
- [ ] 可顯示或分析 `git diff`
- [ ] 修改前知道不得自動 commit / push

## 8. 常見問題

### 找不到 `codex`

檢查 PATH：

```bash
which codex
```

Windows：

```powershell
where.exe codex
```

### npm 安裝失敗

先確認 Node / npm：

```bash
node -v
npm -v
```

### 安裝來源安全提醒

只使用官方來源：

- `https://github.com/openai/codex`
- `https://chatgpt.com/codex/install.sh`
- `https://chatgpt.com/codex/install.ps1`
- npm package：`@openai/codex`

不要安裝名稱很像、但不是官方的套件。