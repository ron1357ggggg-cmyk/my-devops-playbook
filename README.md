# My DevOps Playbook

這個 repo 用來整理我自己的開發環境、遠端操作、CLI 工具、AI 協作與疑難排解 SOP。

更精準地說，它是我的 **AI 輔助開發前置作業手冊**。

它不是用來記錄所有零碎 bug，而是沉澱「AI 開始協助開發之前，人類必須先準備好的基礎設施、權限、操作流程與限制邊界」。

核心流程只有一句話：**裝好兩個 AI → 開放全域權限 → 設定 SSH + tmux，分專案分任務。**

## 核心定位

- Codex CLI / Claude CLI 安裝與環境檢查
- AI 權限治理：全域最大權限，commit/push 交給 SKILL 決定
- 專案入口與遠端開發環境（本機路徑、GitHub、SSH / tmux、proj launcher、獨立任務）
- Claude / Codex 協作與交叉驗證工作流
- Apple 平台（Xcode / iPhone Wi-Fi）部署限制邊界

## 快速入口

### 00. AI CLI Installation

新環境的第一件事：先把兩個 AI CLI 都裝起來，再用清單確認整條後續流程（權限、SSH、專案入口）具備最低作戰能力。

| 文件 | 用途 |
|---|---|
| [Install Codex CLI](00-ai-cli-installation/install-codex-cli.md) | 安裝與啟動 OpenAI Codex CLI |
| [Install Claude CLI](00-ai-cli-installation/install-claude-cli.md) | 安裝與啟動 Claude Code / Claude CLI |
| [Environment Bootstrap Checklist](00-ai-cli-installation/environment-bootstrap-checklist.md) | 新電腦/新環境開工前檢查表，涵蓋基礎工具、SSH、AI CLI、專案入口、遠端長任務、安全治理 |

### 01. AI Permission Governance

裝完 AI 馬上做的第二件事：開放全域最大權限。哲學：讀檔/改檔/執行指令/測試預設全開，不逐項卡確認；真正需要收斂的是 commit / push 這類影響共享狀態的動作，這類授權寫在各自的 SKILL.md 裡，不在這裡做全域硬性規定。

| 文件 | 用途 |
|---|---|
| [AI General Rules](01-ai-permission-governance/ai-general-rules.md) | AI 協作基本憲法：全域最大權限 + commit/push 交給 SKILL 決定 |
| [AI CLI Permissions](01-ai-permission-governance/ai-cli-permissions.md) | 權限類型、風險分級、常見權限異常排查 |

### 02. Project Entry & Remote Dev

第三件事：設定 SSH + tmux，把工作拆成一個個獨立的專案/任務 session。這一章把本機專案入口跟遠端 SSH/tmux 開發環境放在一起，照實際依賴順序排列：本機入口 → SSH 基礎 → tmux 續命 → Windows/WSL 設定 → proj launcher 與獨立任務。

| 文件 | 用途 |
|---|---|
| [GitHub Project Discovery](02-project-entry-and-remote-dev/github-project-discovery.md) | 找 GitHub repo、clone 專案、確認本機路徑 |
| [Auto Enter Project Path](02-project-entry-and-remote-dev/auto-enter-project-path.md) | 本機：CLI 啟動後自動進入常用專案路徑 |
| [SSH Basic Operations](02-project-entry-and-remote-dev/ssh-basic-operations.md) | SSH 基礎操作、key、config、scp、遠端指令 |
| [SSH + tmux Persistent Session](02-project-entry-and-remote-dev/ssh-tmux-persistent-session.md) | 用 tmux 保持遠端 AI 長任務不中斷 |
| [Windows OpenSSH Server + WSL tmux Setup](02-project-entry-and-remote-dev/windows-openssh-wsl-setup.md) | Windows 11 安裝 OpenSSH Server、WSL Ubuntu + tmux，讓 Windows 也能當遠端 AI 節點 |
| [SSH + tmux AI Project Launcher](02-project-entry-and-remote-dev/ssh-tmux-ai-project-launcher.md) | 遠端：SSH 進主機後用 `projects` / `proj` 選專案、分開 Claude/Codex tmux session、建立與刪除獨立 AI task |

### 03. AI Collaboration Workflow

| 文件 | 用途 |
|---|---|
| [Personal AI Development Skill](03-ai-collaboration-workflow/personal-ai-development-skill.md) | 跨專案通用的 AI 協作 Skill：專案偵察、任務前後回報格式、文件成熟度分級、技術棧模板、參數流向追蹤、跨專案污染防火牆，並附錄 Claude 主控 / Codex 審查交叉驗證流程 |

### 04. Apple Platform Limitations

專屬 Apple 生態系（Xcode / iOS / macOS）的部署與能力邊界，其他平台的限制不放這裡。獨立於上面的通用流程之外。

| 文件 | 用途 |
|---|---|
| [Xcode Wireless Deploy Limitation](04-apple-platform-limitations/xcode-wireless-deploy-limitation.md) | Xcode 透過 Wi-Fi / Tailscale 部署 iPhone 的限制邊界 |

### 05. Write Outlook Calendar Skill

獨立於上面的通用流程之外，是單一功能、可攜式的 AI skill，跟這個 repo 其他章節沒有依賴關係。

| 文件 | 用途 |
|---|---|
| [Write Outlook Calendar](05-write-outlook-calendar/README.md) | 把聊天內容/截圖轉成 Outlook 私人行事曆事件，含 Claude Code（`SKILL.md`）與 OpenAI agent（`agents/openai.yaml`）兩種格式 |

## 目錄結構

```txt
my-devops-playbook/
├── 00-ai-cli-installation/
│   ├── install-codex-cli.md
│   ├── install-claude-cli.md
│   └── environment-bootstrap-checklist.md
├── 01-ai-permission-governance/
│   ├── ai-general-rules.md
│   └── ai-cli-permissions.md
├── 02-project-entry-and-remote-dev/
│   ├── github-project-discovery.md
│   ├── auto-enter-project-path.md
│   ├── ssh-basic-operations.md
│   ├── ssh-tmux-persistent-session.md
│   ├── windows-openssh-wsl-setup.md
│   └── ssh-tmux-ai-project-launcher.md
├── 03-ai-collaboration-workflow/
│   └── personal-ai-development-skill.md
├── 04-apple-platform-limitations/
│   └── xcode-wireless-deploy-limitation.md
└── 05-write-outlook-calendar/
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
5. 全域權限預設最大化，只有 commit/push 與破壞性操作需要額外把關（見 01）。

## 使用建議

新電腦或新環境時，依序看：

1. [Install Codex CLI](00-ai-cli-installation/install-codex-cli.md) / [Install Claude CLI](00-ai-cli-installation/install-claude-cli.md)
2. [Environment Bootstrap Checklist](00-ai-cli-installation/environment-bootstrap-checklist.md)
3. [AI General Rules](01-ai-permission-governance/ai-general-rules.md) —— 開放全域權限
4. [SSH Basic Operations](02-project-entry-and-remote-dev/ssh-basic-operations.md) → [SSH + tmux Persistent Session](02-project-entry-and-remote-dev/ssh-tmux-persistent-session.md) → [SSH + tmux AI Project Launcher](02-project-entry-and-remote-dev/ssh-tmux-ai-project-launcher.md) —— 分專案分任務

開始讓 AI 接手專案前，先看：

- [GitHub Project Discovery](02-project-entry-and-remote-dev/github-project-discovery.md)
- [Personal AI Development Skill](03-ai-collaboration-workflow/personal-ai-development-skill.md)

碰到平台限制或部署邊界時，先看：

- [Xcode Wireless Deploy Limitation](04-apple-platform-limitations/xcode-wireless-deploy-limitation.md)
