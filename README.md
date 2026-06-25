# My DevOps Playbook

這個 repo 用來整理我自己的開發環境、遠端操作、CLI 工具、AI 協作與疑難排解 SOP。

更精準地說，它是我的 **AI 輔助開發前置作業手冊**。

它不是用來記錄所有零碎 bug，而是沉澱「AI 開始協助開發之前，人類必須先準備好的基礎設施、權限、操作流程與限制邊界」。

## 核心定位

- 開發環境啟動前檢查
- GitHub 專案入口與路徑管理
- Codex CLI / Claude CLI 安裝與啟動
- AI 權限治理與通用規則
- Claude / Codex 交叉驗證工作流
- SSH / tmux 遠端不中斷工作流
- Xcode / iPhone Wi-Fi 部署限制邊界

## 目錄結構

```txt
my-devops-playbook/
├── 00-bootstrap/
│   ├── environment-bootstrap-checklist.md
│   └── ai-task-prompt-template.md
├── 01-project-entry/
│   ├── github-project-discovery.md
│   └── auto-enter-project-path.md
├── 02-ai-cli-installation/
│   ├── install-codex-cli.md
│   └── install-claude-cli.md
├── 03-ai-permission-governance/
│   ├── ai-cli-permissions.md
│   └── ai-general-rules.md
├── 04-ai-collaboration-workflow/
│   └── claude-codex-review-workflow.md
├── 05-remote-dev-continuity/
│   ├── ssh-basic-operations.md
│   └── ssh-tmux-persistent-session.md
├── 06-platform-limitations/
│   └── xcode-wireless-deploy-limitation.md
└── docs/
    └── mac-ssh-tmux-remote-sop.md
```

## 核心原則

所有文件都盡量維持三個方向：

1. **可複製執行**：指令可以直接拿去跑。
2. **可重建環境**：換一台設備也能照流程設定。
3. **保留踩坑紀錄**：不只寫成功流程，也記錄錯誤原因與解法。

額外治理原則：

1. 不放密碼、token、私鑰、公司內部機敏資訊。
2. 所有指令預設使用 placeholder，例如 `<USER>`、`<HOST>`、`<PROJECT_PATH>`。
3. 文件重點是建立可重複、可移交、可驗證的工作流。
4. AI 可以協助解單點問題，但這個 repo 要保存的是架構型知識。

## 快速使用

新環境或新電腦時，先看：

- `00-bootstrap/environment-bootstrap-checklist.md`
- `00-bootstrap/ai-task-prompt-template.md`

開始讓 AI 接手專案前，先看：

- `01-project-entry/github-project-discovery.md`
- `03-ai-permission-governance/ai-general-rules.md`
- `04-ai-collaboration-workflow/claude-codex-review-workflow.md`

遠端長任務前，先看：

- `05-remote-dev-continuity/ssh-basic-operations.md`
- `05-remote-dev-continuity/ssh-tmux-persistent-session.md`

碰到平台限制或部署邊界時，先看：

- `06-platform-limitations/xcode-wireless-deploy-limitation.md`
