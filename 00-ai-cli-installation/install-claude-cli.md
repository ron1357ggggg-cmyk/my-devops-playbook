# Install Claude CLI

Claude Code 是 Anthropic 的 agentic coding tool，可以在 terminal、IDE、desktop、browser 等環境中讀取 codebase、修改檔案、執行指令與協助開發。

官方文件：`https://code.claude.com/docs/en/overview`

## 1. macOS / Linux / WSL 安裝

官方建議 native install：

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

安裝後進入專案：

```bash
cd <PROJECT_PATH>
claude
```

## 2. Windows PowerShell 安裝

```powershell
irm https://claude.ai/install.ps1 | iex
```

安裝後啟動：

```powershell
claude
```

## 3. Windows CMD 安裝

```cmd
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

如果看到 `&&` 不是有效分隔符，通常代表你其實在 PowerShell，不是在 CMD。

## 4. Homebrew 安裝方式

macOS 可用：

```bash
brew install --cask claude-code
```

更新：

```bash
brew upgrade claude-code
```

## 5. WinGet 安裝方式

```powershell
winget install Anthropic.ClaudeCode
```

更新：

```powershell
winget upgrade Anthropic.ClaudeCode
```

## 6. 第一次啟動

```bash
cd <PROJECT_PATH>
claude
```

第一次通常會要求登入 Claude / Anthropic 帳號。

## 7. Windows 建議

官方文件建議 native Windows 環境安裝 Git for Windows，讓 Claude Code 可以使用 Bash tool。若沒安裝，Claude Code 會改用 PowerShell 作為 shell tool。

確認 Git：

```powershell
git --version
```

確認 shell：

```powershell
$PSVersionTable
```

CMD：

```cmd
ver
```

## 8. 驗收清單

- [ ] `claude` 指令可執行
- [ ] 已登入成功
- [ ] 可讀取目前專案資料夾
- [ ] 可正常貼上長 prompt
- [ ] 可執行 terminal command
- [ ] 權限模式已理解
- [ ] 不會未授權 commit / push

## 9. 常見問題

### 找不到 `claude`

macOS / Linux：

```bash
which claude
```

Windows：

```powershell
where.exe claude
```

### CMD / PowerShell 指令混用

PowerShell prompt 通常長這樣：

```txt
PS C:\Users\...
```

CMD prompt 通常長這樣：

```txt
C:\Users\...
```

安裝指令要對應正確 shell，否則會出現語法錯誤。

## 10. 治理建議

Claude 很適合主控大型修改，但仍要設下邊界：

- 不得自動 commit / push
- 修改後要執行測試
- 權限問題要明確回報
- 大型任務建議搭配 Codex 做 `git diff` 審查