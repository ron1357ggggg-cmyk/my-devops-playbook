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

## 快速入口

### 00. Bootstrap

| 文件 | 用途 |
|---|---|
| [Environment Bootstrap Checklist](00-bootstrap/environment-bootstrap-checklist.md) | 新電腦、新環境、新專案開工前檢查表 |
| [AI Task Prompt Template](00-bootstrap/ai-task-prompt-template.md) | 給 Claude / Codex / AI coding agent 的任務啟動模板 |

### 01. Project Entry

| 文件 | 用途 |
|---|---|
| [GitHub Project Discovery](01-project-entry/github-project-discovery.md) | 找 GitHub repo、clone 專案、確認本機路徑 |
| [Auto Enter Project Path](01-project-entry/auto-enter-project-path.md) | CLI 啟動後自動進入常用專案路徑 |

### 02. AI CLI Installation

| 文件 | 用途 |
|---|---|
| [Install Codex CLI](02-ai-cli-installation/install-codex-cli.md) | 安裝與啟動 OpenAI Codex CLI |
| [Install Claude CLI](02-ai-cli-installation/install-claude-cli.md) | 安裝與啟動 Claude Code / Claude CLI |

### 03. AI Permission Governance

| 文件 | 用途 |
|---|---|
| [AI CLI Permissions](03-ai-permission-governance/ai-cli-permissions.md) | 管理 AI 讀檔、改檔、執行指令、Git 操作權限 |
| [AI General Rules](03-ai-permission-governance/ai-general-rules.md) | AI 協作基本憲法：不得自動 commit / push、修改後要驗證 |

### 04. AI Collaboration Workflow

| 文件 | 用途 |
|---|---|
| [Claude / Codex Review Workflow](04-ai-collaboration-workflow/claude-codex-review-workflow.md) | Claude 主控實作、Codex 審查 git diff 的交叉驗證流程 |

### 05. Remote Dev Continuity

| 文件 | 用途 |
|---|---|
| [SSH Basic Operations](05-remote-dev-continuity/ssh-basic-operations.md) | SSH 基礎操作、key、config、scp、遠端指令 |
| [SSH + tmux Persistent Session](05-remote-dev-continuity/ssh-tmux-persistent-session.md) | 用 tmux 保持遠端 AI 長任務不中斷 |
| [SSH + tmux AI Project Launcher](05-remote-dev-continuity/ssh-tmux-ai-project-launcher.md) | SSH 進主機後用 `projects` / `proj` 選專案、分開 Claude/Codex tmux session、建立與刪除 AI task |

### 06. Platform Limitations

| 文件 | 用途 |
|---|---|
| [Xcode Wireless Deploy Limitation](06-platform-limitations/xcode-wireless-deploy-limitation.md) | Xcode 透過 Wi-Fi / Tailscale 部署 iPhone 的限制邊界 |

### Legacy / Existing Docs

| 文件 | 用途 |
|---|---|
| [Mac SSH tmux Remote SOP](docs/mac-ssh-tmux-remote-sop.md) | 既有 Mac 遠端 SSH + tmux 設定 SOP |

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
│   ├── ssh-tmux-persistent-session.md
│   └── ssh-tmux-ai-project-launcher.md
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

## 使用建議

新環境或新電腦時，先看：

- [Environment Bootstrap Checklist](00-bootstrap/environment-bootstrap-checklist.md)
- [AI Task Prompt Template](00-bootstrap/ai-task-prompt-template.md)

開始讓 AI 接手專案前，先看：

- [GitHub Project Discovery](01-project-entry/github-project-discovery.md)
- [AI General Rules](03-ai-permission-governance/ai-general-rules.md)
- [Claude / Codex Review Workflow](04-ai-collaboration-workflow/claude-codex-review-workflow.md)

遠端長任務前，先看：

- [SSH Basic Operations](05-remote-dev-continuity/ssh-basic-operations.md)
- [SSH + tmux Persistent Session](05-remote-dev-continuity/ssh-tmux-persistent-session.md)

碰到平台限制或部署邊界時，先看：

- [Xcode Wireless Deploy Limitation](06-platform-limitations/xcode-wireless-deploy-limitation.md)
