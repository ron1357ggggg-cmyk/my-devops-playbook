# Environment Bootstrap Checklist

這份清單用來檢查新電腦、新專案或遠端主機是否已具備 AI 輔助開發的最低作戰能力，橫跨本章節（AI CLI 安裝）到 [01. AI Permission Governance](../01-ai-permission-governance/ai-general-rules.md)、[02. Project Entry & Remote Dev](../02-project-entry-and-remote-dev/github-project-discovery.md) 的檢查項目。

## 1. 基礎工具

- [ ] Git 已安裝
- [ ] GitHub 帳號可登入
- [ ] GitHub CLI `gh` 可用，或至少可用瀏覽器 / git 操作 repo
- [ ] Node.js / npm 可用
- [ ] 終端機可正常複製貼上
- [ ] Shell 環境明確：CMD / PowerShell / Bash / Zsh

## 2. GitHub / SSH

- [ ] SSH key 已建立
- [ ] 公鑰已加入 GitHub
- [ ] 可成功測試 GitHub SSH 連線
- [ ] 可 clone private / public repo
- [ ] 知道 repo clone 到本機哪個根目錄

```bash
ssh -T git@github.com
```

## 3. AI CLI

- [ ] Codex CLI 可啟動
- [ ] Claude CLI 可啟動
- [ ] AI CLI 可以讀取目前專案資料夾
- [ ] AI CLI 可以自由讀取/修改檔案；commit / push 是否自動做，依當下 SKILL.md 規定
- [ ] AI CLI 權限模式已理解

```bash
codex
claude
```

## 4. 專案入口

- [ ] 已知道 GitHub repo 名稱
- [ ] 已知道本機專案路徑
- [ ] 已確認目前 terminal 位於正確專案根目錄
- [ ] 已確認 `git status` 乾淨或知道目前異動內容

```bash
pwd
git status
git remote -v
```

## 5. 遠端長任務

- [ ] 遠端主機可 SSH 連入
- [ ] tmux 已安裝
- [ ] 長任務會在 tmux session 內執行
- [ ] 中斷連線後可重新 attach 回 session

```bash
tmux new -s ai-work
# detach: Ctrl+b, then d
tmux attach -t ai-work
```

## 6. 安全與治理

- [ ] 不把 `.env`、token、私鑰、公司內部機敏資訊 commit 到 repo
- [ ] AI 修改前先閱讀專案結構
- [ ] AI 修改後必須說明驗證方式
- [ ] AI 不得未測試就宣稱完成
- [ ] commit / push 是否自動做，依當下 SKILL.md 規定；沒有規定就先問過我

## 7. 開工判斷

如果以上條件不完整，先補環境，不要急著叫 AI 寫 code。

AI 很擅長解單點問題，但不會自動替你建立整套作業治理。因此這份 checklist 是每次開工前的第一道品質閘門。