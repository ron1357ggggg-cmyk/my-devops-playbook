# My DevOps Playbook

這個 repo 用來整理我自己的開發環境、遠端操作、CLI 工具、AI 協作與疑難排解 SOP。

更精準地說，它是我的 **AI 輔助開發前置作業手冊**。

它不是用來記錄所有零碎 bug，而是沉澱「AI 開始協助開發之前，人類必須先準備好的基礎設施、權限、操作流程與限制邊界」。

## 核心定位

- Codex CLI / Claude CLI 安裝與啟動
- 開發環境啟動前檢查
- GitHub 專案入口與路徑管理（本機與遠端）
- AI 權限治理與通用規則
- Claude / Codex 交叉驗證工作流
- SSH / tmux 遠端不中斷工作流
- Apple 平台（Xcode / iPhone Wi-Fi）部署限制邊界

## 快速入口

### 00. AI CLI Installation

新環境的第一件事：先把 AI CLI 裝起來，後面所有流程都靠它執行。

| 文件 | 用途 |
|---|---|
| [Install Codex CLI](00-ai-cli-installation/install-codex-cli.md) | 安裝與啟動 OpenAI Codex CLI |
| [Install Claude CLI](00-ai-cli-installation/install-claude-cli.md) | 安裝與啟動 Claude Code / Claude CLI |

### 01. Bootstrap

| 文件 | 用途 |
|---|---|
| [Environment Bootstrap Checklist](01-bootstrap/environment-bootstrap-checklist.md) | 新電腦、新環境、新專案開工前檢查表 |
| [AI Task Prompt Template](01-bootstrap/ai-task-prompt-template.md) | 給 Claude / Codex / AI coding agent 的任務啟動模板 |

### 02. Project Entry

「AI 進新環境後，第一步永遠是先確認專案在哪」——不管是本機還是透過 SSH 連進遠端主機，邏輯都放在這一章，避免跟 05 的 SSH/tmux 基礎設施混在一起。

| 文件 | 用途 |
|---|---|
| [GitHub Project Discovery](02-project-entry/github-project-discovery.md) | 找 GitHub repo、clone 專案、確認本機路徑 |
| [Auto Enter Project Path](02-project-entry/auto-enter-project-path.md) | 本機：CLI 啟動後自動進入常用專案路徑 |
| [SSH + tmux AI Project Launcher](02-project-entry/ssh-tmux-ai-project-launcher.md) | 遠端：SSH 進主機後用 `projects` / `proj` 選專案、分開 Claude/Codex tmux session、建立與刪除 AI task |

### 03. AI Permission Governance

哲學：全域預設開放最大權限（讀檔/改檔/執行指令/測試），不逐項卡確認。真正需要收斂的是 commit / push 這類影響共享狀態的動作，這類授權寫在各自的 SKILL.md 裡，不在這裡做全域硬性規定。

| 文件 | 用途 |
|---|---|
| [AI CLI Permissions](03-ai-permission-governance/ai-cli-permissions.md) | 權限類型、風險分級、常見權限異常排查 |
| [AI General Rules](03-ai-permission-governance/ai-general-rules.md) | AI 協作基本憲法：全域最大權限 + commit/push 交給 SKILL 決定 |

### 04. AI Collaboration Workflow

| 文件 | 用途 |
|---|---|
| [Personal AI Development Skill](04-ai-collaboration-workflow/personal-ai-development-skill.md) | 跨專案通用的 AI 協作 Skill：專案偵察、任務前後回報格式、文件成熟度分級、技術棧模板、參數流向追蹤、跨專案污染防火牆，並附錄 Claude 主控 / Codex 審查交叉驗證流程 |

### 05. Remote Dev Continuity

只處理 SSH 連線與 tmux 續命本身；「進去之後要打開哪個專案」請看 02。

| 文件 | 用途 |
|---|---|
| [SSH Basic Operations](05-remote-dev-continuity/ssh-basic-operations.md) | SSH 基礎操作、key、config、scp、遠端指令 |
| [SSH + tmux Persistent Session](05-remote-dev-continuity/ssh-tmux-persistent-session.md) | 用 tmux 保持遠端 AI 長任務不中斷 |
| [Windows OpenSSH Server + WSL tmux Setup](05-remote-dev-continuity/windows-openssh-wsl-setup.md) | Windows 11 安裝 OpenSSH Server、WSL Ubuntu + tmux，讓 Windows 也能當遠端 AI 節點 |

### 06. Apple Platform Limitations

專屬 Apple 生態系（Xcode / iOS / macOS）的部署與能力邊界，其他平台的限制不放這裡。

| 文件 | 用途 |
|---|---|
| [Xcode Wireless Deploy Limitation](06-apple-platform-limitations/xcode-wireless-deploy-limitation.md) | Xcode 透過 Wi-Fi / Tailscale 部署 iPhone 的限制邊界 |

### 07. AI Skills

獨立於上面的通用流程之外，收放單一功能、可攜式的 AI skill。

| 文件 | 用途 |
|---|---|
| [Write Outlook Calendar](07-ai-skills/write-outlook-calendar/README.md) | 可攜式 AI skill：把聊天內容/截圖轉成 Outlook 私人行事曆事件，含 Claude Code（`SKILL.md`）與 OpenAI agent（`agents/openai.yaml`）兩種格式 |

## 目錄結構

```txt
my-devops-playbook/
├── 00-ai-cli-installation/
│   ├── install-codex-cli.md
│   └── install-claude-cli.md
├── 01-bootstrap/
│   ├── environment-bootstrap-checklist.md
│   └── ai-task-prompt-template.md
├── 02-project-entry/
│   ├── github-project-discovery.md
│   ├── auto-enter-project-path.md
│   └── ssh-tmux-ai-project-launcher.md
├── 03-ai-permission-governance/
│   ├── ai-cli-permissions.md
│   └── ai-general-rules.md
├── 04-ai-collaboration-workflow/
│   └── personal-ai-development-skill.md
├── 05-remote-dev-continuity/
│   ├── ssh-basic-operations.md
│   ├── ssh-tmux-persistent-session.md
│   └── windows-openssh-wsl-setup.md
├── 06-apple-platform-limitations/
│   └── xcode-wireless-deploy-limitation.md
└── 07-ai-skills/
    └── write-outlook-calendar/
        ├── README.md
        ├── SKILL.md
        └── agents/
            └── openai.yaml
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
5. 全域權限預設最大化，只有 commit/push 與破壞性操作需要額外把關（見 03）。

## 使用建議

新電腦或新環境時，先看：

- [Install Codex CLI](00-ai-cli-installation/install-codex-cli.md) / [Install Claude CLI](00-ai-cli-installation/install-claude-cli.md)
- [Environment Bootstrap Checklist](01-bootstrap/environment-bootstrap-checklist.md)

開始讓 AI 接手專案前，先看：

- [GitHub Project Discovery](02-project-entry/github-project-discovery.md)
- [AI General Rules](03-ai-permission-governance/ai-general-rules.md)
- [Personal AI Development Skill](04-ai-collaboration-workflow/personal-ai-development-skill.md)

遠端長任務前，先看：

- [SSH Basic Operations](05-remote-dev-continuity/ssh-basic-operations.md)
- [SSH + tmux Persistent Session](05-remote-dev-continuity/ssh-tmux-persistent-session.md)
- [SSH + tmux AI Project Launcher](02-project-entry/ssh-tmux-ai-project-launcher.md)

碰到平台限制或部署邊界時，先看：

- [Xcode Wireless Deploy Limitation](06-apple-platform-limitations/xcode-wireless-deploy-limitation.md)
