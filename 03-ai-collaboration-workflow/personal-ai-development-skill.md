# Personal AI Development Skill（跨專案通用版）

> 目標：把你在公司與個人 GitHub 專案中已經驗證過的 AI 使用方式，抽象成可重複使用的開發 Skill。  
> 核心原則：**抽方法，不抽業務；抽流程，不抽結論；抽習慣，不抽專案規則。**  
> 修正版重點：**AI 文件不是每個專案都必須存在；只能讀取存在的文件，缺少時使用 README / 專案結構 / package 或 solution 檔作為 fallback，不可硬套 AI 資料夾。**

---

## 0. 適用場景

本 Skill 適用於新專案啟動、既有專案接手、除錯、重構前盤點、API/資料流追蹤、文件建立、測試驗證與部署前檢查。

適用技術棧包含但不限於：

- .NET / C# / ASP.NET MVC / Web API / EF
- Java / Spring Boot
- Node.js / TypeScript / LINE Bot / Browser Automation
- Swift / SwiftUI / iOS / macOS
- Electron / Windows 小工具
- PowerShell / local scheduler
- SQL Server / SQLite / local file storage
- Web 前端 / React / Vue / HTML / CSS

---

## 1. 來源盤點

### 1.1 公司 AI 文件來源

本次整理參考了以下公司專案 AI 文件：

| 文件 | 抽象後可保留的能力 |
|---|---|
| `AGENTS.md` | 任務開始前先盤點、必讀文件、工作規則、安全規則、知識更新規則 |
| `AI_CONTEXT.md` | 專案快速地圖、架構摘要、常見修改位置、資料流入口 |
| `DISCOVERIES.md` | 可重複使用的分析結果、root cause、流程記憶、禁止記錄敏感資訊 |
| `DECISIONS.md` | 架構決策、分層邊界、API contract、狀態流程、避免大型重構 |
| `BUSINESS_RULES.md` | 業務規則集中管理、公司別規則隔離、流程狀態定義 |
| `COMMON_COMMANDS.md` | restore / build / test / run / search 指令集中化 |
| `PARAMETER_FLOW_TEST_RULES.md` | 修改後產出參數流向驗證報告的觸發條件與格式 |

### 1.2 GitHub 專案來源

本次查到的 GitHub 專案與抽象後的工程模式：

| Repository | 專案定位 | 可抽象為通用 Skill 的模式 |
|---|---|---|
| `ai-morning-briefing-ios` | SwiftUI 健康晨報 App，整合 HealthKit、恢復分數、新聞、本機通知 | 隱私優先、本機資料處理、MVP 邊界清楚、醫療/健康輸出需標示限制 |
| `chatgpt-line-bridge` | LINE Bot 透過本機 Chrome CDP 操作 ChatGPT Web，不使用 OpenAI API | 架構邊界不可擅自改寫、README 維護規則、不得繞過登入/驗證/反自動化 |
| `codex-rate-widget` | Windows Electron 小窗，讀本機 Codex session rate limit | 本機 session 不跨設備同步、工具型小專案需清楚標示資料來源 |
| `codex-usage-macos` | macOS 原生選單列工具，查 Codex usage | 主資料來源 + 備援來源設計、不讀 token / cookie / API key |
| `discord-room-monitor` | Windows 小浮窗，以截圖觀察 Discord 語音房間 | 不使用 self-bot、不讀帳號資料、以低侵入方式完成目的 |
| `EmailToExcel` | README 目前只有標題 | 專案文件不足時，AI 應先補文件，不可自行腦補需求 |
| `immersive-reader` | SwiftUI PDF/EPUB 閱讀器，Mock AI assistant | 使用者自有資料、不提交大型 fixture、不提交 API key |
| `linebot-server` | LINE AI Recovery Bot，以 Apple Health ZIP 與手動資料產生近況趨勢 | 指令路由、被動群組模式、健康資料結構化、Bot runtime 開關、敏感資料環境變數 |
| `YoutubeClock` | Windows YouTube Music Scheduler | 小型本機工具需明確入口檔、設定檔與系統排程名稱 |

---

## 2. Skill 核心原則

### 2.1 專案隔離原則

AI 必須先判定目前專案類型，再決定可以套用哪些規則。

```text
通用方法可以跨專案使用。
技術棧經驗可以在同技術棧中參考。
商業邏輯、公司規則、API contract、資料表欄位不可跨專案套用。
```

#### 不可跨專案帶入的內容

- 公司業務系統的 `CompanyCode`、業務流程、API callback 規則
- LINE Bot 的 webhook / 指令路由規則
- iOS HealthKit 的健康資料欄位與隱私假設
- Codex usage tool 的 session log 路徑
- Windows / macOS 本機工具的 OS 行為假設
- 任何專案的 token、password、connection string、個資、測試資料

#### 可以跨專案抽象的內容

- 先讀文件再改程式
- 先盤點資料流再改欄位
- 修改前提出影響範圍與修改策略
- 小範圍、低風險、可驗證的改動
- 修改後更新文件
- 不記錄 secrets / 個資 / 單次 debug logs
- 將 root cause、流程圖、常見坑寫入可重複使用的 AI 記憶文件

---

## 3. 新專案啟動時的 AI 執行流程

### 3.1 第一階段：專案偵察

AI 接手新 repo 時，先執行：

```text
1. 讀 README.md
2. 檢查是否有 AGENTS.md
3. 檢查是否有 AI/AI_CONTEXT.md
4. 檢查 package / solution / project / build files
5. 判斷專案類型：前端、後端、App、Bot、本機工具、資料處理、排程工具
6. 判斷資料敏感度：個資、健康資料、公司資料、token、帳號資料、瀏覽器 session
7. 建立本專案的「不可跨專案假設清單」
```

### 3.2 第二階段：任務開始前回報

修改程式前，AI 需先回報：

```markdown
## 任務理解
- 我理解這次要做什麼。

## 預期影響範圍
- 可能影響哪些功能、API、畫面、資料流、設定或排程。

## 預計修改檔案
- 列出預計修改的檔案；不確定時列出預計檢查的檔案。

## 修改策略
- 採用最小改動。
- 保留既有架構。
- 不改 API contract，除非使用者明確要求。
- 修改後提供驗證方式。
```

### 3.3 第三階段：執行修改

執行修改時遵守：

- 不修改無關檔案。
- 不為了「看起來乾淨」重構整包。
- 不引入大型架構變更。
- 不擅自把同步流程改成非同步流程。
- 不擅自改 API response shape。
- 不擅自改 DB schema。
- 不擅自新增依賴套件，除非有明確效益與替代方案比較。
- 不提交或記錄秘密、cookie、token、API key、connection string。
- 不把單次測試資料寫進長期文件。

### 3.4 第四階段：完成後回報

完成後回報：

```markdown
## 完成項目
- 實際修改了什麼。

## 修改檔案
- 檔案清單與原因。

## 驗證方式
- 已執行的 build / test / manual check。
- 未執行的原因。

## 風險與限制
- 哪些外部服務、實機、權限、白名單、token、資料狀態仍需人工驗證。

## 文件更新
- README / AGENTS / AI_CONTEXT / DISCOVERIES / DECISIONS 是否需要更新。
```

---

## 4. 通用文件架構

### 4.0 文件成熟度分級

不是每個專案都需要完整 `AI/` 資料夾。AI 必須依照專案規模與現況選擇文件層級：

| 層級 | 適用專案 | 建議文件 | AI 行為 |
|---|---|---|---|
| Level 0：無 AI 文件 | 臨時工具、早期 prototype、只有 README 的 repo | `README.md` | 只讀現有 README、程式結構與 package/solution/build files；不可假設 AI 文件存在。 |
| Level 1：最小 AI 文件 | 小工具、單人專案、功能穩定但規模不大 | `AGENTS.md` | 用一份短文件定義工作規則與安全邊界。 |
| Level 2：輕量 AI 資料夾 | 中型專案、會持續維護、有多次 AI 協作 | `AGENTS.md`、`AI/AI_CONTEXT.md`、`AI/COMMON_COMMANDS.md`、`AI/DISCOVERIES.md` | 開始累積架構、指令與常見問題。 |
| Level 3：完整 AI 治理 | 公司專案、複雜業務流程、多系統整合、高風險資料流 | 完整 `AI/` 文件組 | 使用決策、業務規則、參數流向驗證、知識更新流程。 |

原則：**文件不存在不是錯誤；AI 不得因為找不到 `AI/AI_CONTEXT.md`、`DISCOVERIES.md` 或 `DECISIONS.md` 就停工。**  
正確做法是回退到 `README.md`、目錄結構、專案設定檔、package/build files 與程式入口，並在回覆中標示「目前缺少 AI 文件，因此本次判斷只根據現有文件與程式結構」。

新專案「建議」建立以下文件，但不是硬性要求：

```text
AGENTS.md
AI/
  AI_CONTEXT.md
  DISCOVERIES.md
  DECISIONS.md
  COMMON_COMMANDS.md
  PROJECT_RULES.md
  SECURITY_RULES.md
  TESTING_NOTES.md
```

### 4.1 `AGENTS.md`

用途：給 AI 的最高階工作指令，保持短而硬。

建議內容：

```markdown
# AI Agent Instructions

## Required Reading

Before making changes, read existing project guidance in this order:

1. `README.md` if present
2. `AGENTS.md` if present
3. `AI/AI_CONTEXT.md` if present
4. `AI/PROJECT_RULES.md` if present
5. `AI/COMMON_COMMANDS.md` if the task involves install/build/test/run/deploy and the file exists
6. `AI/DECISIONS.md` if present

If these files do not exist, inspect project structure and build files instead. Do not assume missing AI files exist.

## Before Editing

Before editing code, summarize:

- Task understanding
- Expected impact scope
- Files likely to change
- Modification strategy

## Work Rules

- Do not modify unrelated files.
- Do not change public API contracts unless explicitly requested.
- Do not introduce large refactors unless the task is architecture-level.
- Preserve the existing project style.
- Keep changes small, reversible, and testable.
- Do not commit secrets, tokens, cookies, passwords, connection strings, personal data, health data, or one-off debug logs.

## Knowledge Update

After analysis or code changes, decide whether useful reusable knowledge was discovered.

If yes, update:

- `AI/DISCOVERIES.md` for flow/root-cause findings.
- `AI/AI_CONTEXT.md` for stable architecture.
- `AI/PROJECT_RULES.md` for stable project-specific rules.
- `AI/DECISIONS.md` for design decisions.

If no update is needed, explain why.
```

### 4.2 `AI/AI_CONTEXT.md`

用途：專案地圖，讓 AI 快速理解。

建議內容：

```markdown
# AI Context

## Project Purpose

本專案負責什麼，非目標是什麼。

## Architecture

主要資料流與模組依賴。

## Main Entry Points

- UI entry
- API entry
- CLI entry
- Scheduler entry
- Background worker entry

## Common Change Areas

| Task | Files |
|---|---|

## Data Flow

```text
Input
  -> Controller / Handler / ViewModel
  -> Service
  -> Repository / Storage
  -> External system or UI response
```

## Sensitive Data

列出本專案不可記錄的資料類型。
```

### 4.3 `AI/DISCOVERIES.md`

用途：保存已分析過、未來會重複用到的結果。

範本：

```markdown
# Discoveries

## YYYY-MM-DD - Topic

- Feature:
- Related files:
- Flow summary:
- Root cause:
- Business / product rule:
- Risk:
- Verification:
```

禁止放入：

```text
tokens
passwords
API keys
connection strings
cookies
personal data
health raw data
one-off test data
debug logs
```

### 4.4 `AI/DECISIONS.md`

用途：保存穩定的設計決策。

範本：

```markdown
# Design Decisions

## YYYY-MM-DD - Decision Name

- Decision:
- Reason:
- Trade-off:
- Applies to:
- Does not apply to:
```

### 4.5 `AI/COMMON_COMMANDS.md`

用途：集中 build / test / run / search 指令。

範本：

```markdown
# Common Commands

## Install

## Build

## Test

## Run

## Search

## Deploy / Package

## Notes

- Do not paste secrets or environment values into docs.
```

### 4.6 `AI/PROJECT_RULES.md`

用途：只放本專案專屬規則。

範本：

```markdown
# Project Rules

## Stable Rules

## Business Rules

## Runtime Rules

## External Integration Rules

## UI / UX Rules

## Data Rules

## Rules that must not be generalized to other projects
```

### 4.7 `AI/SECURITY_RULES.md`

用途：安全與隱私規則。

範本：

```markdown
# Security Rules

## Never Record

- API keys
- tokens
- passwords
- cookies
- connection strings
- personal data
- health raw data
- one-off test data
- debug logs

## External Access

- Do not bypass login, captcha, traffic verification, or anti-automation controls.
- Do not scrape private data without explicit user authorization.
- Prefer least-privilege data access.

## Local Data

- Prefer local processing when handling sensitive health or personal data.
- Do not upload raw sensitive data unless the project explicitly requires it and the user authorizes it.
```

---

## 5. 技術棧模板

### 5.1 .NET / Enterprise System

適合：公司內部系統、MVC / Web API / Batch / EF。

工作策略：

- 先確認分層：Controller / Logic / Access / DB。
- Business logic 不放 Controller。
- 不手改 EF generated files。
- API response shape 不可輕易改。
- 狀態欄位、log、batch retry filter 要一起看。
- 外部 integration 需區分「程式邏輯驗證」與「實際 TEST 環境驗證」。

### 5.2 Node.js / Bot / Browser Automation

適合：LINE Bot、Chrome CDP bridge、Webhook server。

工作策略：

- 先讀 README 與 `.env.example`。
- 明確區分正常聊天流、指令流、附件流、背景工作。
- 不繞過登入、captcha、反自動化或驗證頁。
- Bot 群組模式要預設低干擾。
- 回覆格式要緊湊、可讀、可追溯來源。
- 操作瀏覽器時要有 lock，避免多人同時操作同一頁。

### 5.3 Swift / iOS / macOS

適合：HealthKit App、閱讀器、macOS menubar tool。

工作策略：

- 先看 privacy boundary。
- 健康資料、書籍、閱讀紀錄優先本機處理。
- 不提交 user-owned content、API key、大型 fixture。
- Simulator 與實機能力不同，HealthKit、通知、背景任務需標示限制。
- UI 改動要兼顧深色模式、Dynamic Type、錯誤狀態、空資料狀態。

### 5.4 Electron / Windows 小工具

適合：Codex usage widget、Discord monitor。

工作策略：

- 本機工具要清楚資料來源。
- 不讀取不必要帳號資料。
- 若資料只來自本機 session，明確標示不跨設備同步。
- 小工具優先穩定、低資源、可關閉、可移除。
- OS 自動化要標明限制，避免假裝能取得不存在的事件。

### 5.5 PowerShell / Scheduler

適合：YouTube alarm、local automation。

工作策略：

- 明確列出 entry script、設定檔、Task Scheduler 名稱。
- 所有本機副作用要可追蹤。
- 避免隱性全域狀態。
- 提供 install / uninstall / repair 指令。

---

## 6. 參數流向與資料流追蹤方法

遇到 API、欄位、資料保存、UI 顯示、callback、external integration 問題時，AI 應使用以下追蹤矩陣。

```markdown
| 節點 | 檔案 / 方法 | 輸入來源 | 轉換邏輯 | 輸出去向 | 風險 |
|---|---|---|---|---|---|
| UI | | | | | |
| Controller / Handler | | | | | |
| Service / Logic | | | | | |
| Repository / Storage | | | | | |
| External API / Bot / OS | | | | | |
| Response / UI Display | | | | | |
```

狀態標示：

```text
已確認
需人工驗證
不適用
有風險
未知
```

注意：此報告只驗證程式邏輯；外部服務、白名單、真實 token、實機權限、TEST DB 與 callback 時序仍需人工驗證。

---

## 7. 知識分類規則

每次分析後，AI 要判斷新知識放哪裡。

| 知識類型 | 文件 |
|---|---|
| 穩定架構 | `AI/AI_CONTEXT.md` |
| 專案專屬規則 | `AI/PROJECT_RULES.md` |
| 設計決策 | `AI/DECISIONS.md` |
| API / feature flow | `AI/DISCOVERIES.md` |
| Root cause | `AI/DISCOVERIES.md` |
| 指令與操作 | `AI/COMMON_COMMANDS.md` |
| 安全限制 | `AI/SECURITY_RULES.md` |

### 7.1 不要放進長期記憶的資料

- 一次性的錯誤 log
- 一次性的測試帳號
- 個資
- 健康原始資料
- token / password / cookie / API key
- connection string
- 保單號碼 / 身分證字號
- 臨時 debug payload
- 截圖中的私人資訊

---

## 8. 跨專案汙染防火牆

AI 在每次任務開始前要先判斷：

```text
我現在使用的是：
- 通用開發方法？
- 技術棧經驗？
- 本專案文件明確規則？
- 其他專案經驗？
```

如果是其他專案經驗，只能這樣講：

```text
這個模式在其他專案可作為參考，但我不會直接套用。需要先確認目前專案文件或程式碼是否也有相同約束。
```

不可這樣做：

```text
因為 A 專案有 CompanyCode，所以這個 LINE Bot 也要用 CompanyCode。
因為 Codex usage 工具讀 ~/.codex/sessions，所以 iOS App 也要讀本機 session。
因為 LINE Bot 有被動群組模式，所以所有 bot 都要預設不回覆。
```

---

## 9. 新專案開局 Prompt

建立新專案時，可直接給 AI：

```text
請依照 Personal AI Development Skill 幫我初始化本專案的 AI 協作規則。

要求：
1. 先閱讀 README、package/project/solution/build files。
2. 判斷專案類型、專案規模、敏感資料邊界與維護頻率。
3. 先建議文件成熟度：
   - Level 0：只維護 README
   - Level 1：只新增 AGENTS.md
   - Level 2：新增 AGENTS.md + 輕量 AI 資料夾
   - Level 3：完整 AI 文件治理
4. 不要一開始就強制建立完整 AI 資料夾；除非專案確實複雜或我明確要求。
5. 若適合建立文件，優先順序為：
   - AGENTS.md
   - AI/AI_CONTEXT.md
   - AI/COMMON_COMMANDS.md
   - AI/DISCOVERIES.md
   - AI/PROJECT_RULES.md
   - AI/DECISIONS.md
   - AI/SECURITY_RULES.md
   - AI/TESTING_NOTES.md
6. 只寫通用工程規則與本專案已確認事實。
7. 不要把其他專案的商業邏輯、公司規則、API 欄位、資料表欄位帶進來。
8. 若資訊不足，標示「待確認」，不要自行腦補。
9. 完成後列出：
   - 建議文件層級
   - 建立/修改檔案
   - 每份文件用途
   - 哪些內容是已確認
   - 哪些內容待確認
```

---

## 10. 最小可用版 AGENTS.md

以下是可直接放入新 repo 的精簡版。

```markdown
# AI Agent Instructions

## Core Principle

Use general engineering workflow across projects, but never import business logic, API contracts, database fields, company rules, personal data assumptions, or runtime assumptions from another project unless this repository explicitly confirms them.

## Required Reading

Before making changes, read:

1. `README.md`
2. `AI/AI_CONTEXT.md` if present
3. `AI/PROJECT_RULES.md` if present
4. `AI/DECISIONS.md` if present
5. `AI/COMMON_COMMANDS.md` if the task involves install/build/test/run/deploy

## Before Editing

Before editing code, summarize:

- Task understanding
- Expected impact scope
- Files likely to change
- Modification strategy

## Work Rules

- Make the smallest safe change.
- Do not modify unrelated files.
- Preserve existing architecture and style.
- Do not change public API contracts unless explicitly requested.
- Do not introduce large refactors unless the task is explicitly architecture-level.
- Do not commit secrets, tokens, cookies, passwords, connection strings, personal data, health data, or one-off debug logs.
- If information is missing, mark it as unknown instead of guessing.

## Knowledge Update

After analysis or changes, decide whether reusable knowledge was discovered.

If yes, update the appropriate file:

- `AI/DISCOVERIES.md` for flow analysis and root cause findings.
- `AI/AI_CONTEXT.md` for stable architecture.
- `AI/PROJECT_RULES.md` for stable project-specific rules.
- `AI/DECISIONS.md` for design decisions.
- `AI/COMMON_COMMANDS.md` for commands.

If no update is needed, explain why.
```

---

## 11. 我的建議落地策略

### 第一階段：建立全域 Skill

把本文件作為你的「個人 AI 開發 Skill」。不要放任何公司機密、token、個資或專案細節。

### 第二階段：每個 repo 放最小 AGENTS.md

所有新專案都放入「最小可用版 AGENTS.md」。  
專案越大，再逐步補 `AI/` 資料夾。

### 第三階段：公司專案與私人專案分層

公司專案：

```text
AGENTS.md
AI/AI_CONTEXT.md
AI/DISCOVERIES.md
AI/DECISIONS.md
AI/COMMON_COMMANDS.md
AI/PROJECT_RULES.md
AI/SECURITY_RULES.md
```

私人小工具：

```text
AGENTS.md
README.md
AI/AI_CONTEXT.md
AI/COMMON_COMMANDS.md
AI/DISCOVERIES.md
```

MVP / 小實驗：

```text
README.md
AGENTS.md
```

### 第四階段：每次任務後累積資產

每次 root cause、資料流、安裝指令、部署坑、API 規則被釐清後，都寫進對應文件。  
這會讓 AI 從「一次性問答」升級成「專案知識庫」。

---

## 12. 結論

這套 Skill 的定位不是讓 AI 幫你亂套模板，而是讓 AI 形成固定作業節奏：

```text
先讀文件
先判斷專案邊界
先說修改策略
小範圍修改
追資料流
保留 API contract
保護敏感資訊
完成後更新知識庫
永遠不跨專案複製業務邏輯
```

最終目標是讓你開新專案時，不用重新教 AI 一次你的工作方式；但每個專案仍然保留自己的商業邏輯與架構邊界。

---

## 13. 附錄：Claude 主控 / Codex 審查交叉驗證流程

這套流程是本 Skill 在「大型重構、API 串接、金流/會員/高風險業務流程」等高回歸風險任務下的具體落地方式：Claude 負責主控與實作，Codex 負責審查 git diff，人類負責目標、邊界與驗收。

### 13.1 為什麼要分工

單一 AI 容易出現：

- 修改範圍過大
- 自認完成但沒驗證
- 沒注意回歸風險
- 忽略安全或邊界條件
- 對自己產出的程式過度樂觀

用第二個 AI 做 diff review，可以建立基本制衡。

### 13.2 角色定義

| 角色 | 責任 | 禁止事項 |
|---|---|---|
| Claude | 需求理解、修改程式、執行測試、整理交付 | commit / push 是否自動做依 [AI General Rules](../01-ai-permission-governance/ai-general-rules.md) 的 SKILL 授權規則 |
| Codex | 審查 git diff、指出風險、提出修正建議 | 審查模式不得修改檔案 |
| Human | 定義任務、授權權限、判斷是否交付 | 不應跳過驗收 |

### 13.3 標準流程

```txt
1. Human 下任務
2. Claude 讀取專案結構
3. Claude 提出理解與計畫
4. Claude 修改程式
5. Claude 執行測試或模擬驗證
6. Claude 產生 git diff
7. Codex 審查 git diff
8. Codex 輸出 AI_REVIEW.md
9. Claude 根據審查修正
10. 最多重複 3 輪
11. Claude 輸出 final delivery report
12. Human 決定是否 commit / push
```

### 13.4 Claude 主控 prompt

```txt
請讀取 AI_TASK.md 並開始執行。

你是主控 AI，負責實作與協調 Codex CLI 審查。

執行規則：
1. 你負責修改程式。
2. Codex 只負責審查，不准修改。
3. 每次完成一輪修改後，必須呼叫 Codex CLI 審查 git diff。
4. Codex 的審查結果寫入 AI_REVIEW.md。
5. 根據 AI_REVIEW.md 修正問題。
6. 最多進行 3 輪。
7. commit / push 是否自動做，依照本次任務的 SKILL 授權規則；沒有規定就先問過我。
8. 修改後必須執行測試；若不能測試，必須說明原因。
9. 最後輸出完整交付報告。
```

### 13.5 Codex 審查 prompt

```txt
你現在是審查 AI，只能審查，不准修改檔案。

請審查目前 git diff。

審查重點：
1. 是否有破壞既有功能。
2. 是否有安全性或資料外洩風險。
3. 是否有未處理的錯誤流程。
4. 是否有 race condition、timeout、null、edge case。
5. 是否有過度設計或重複邏輯。
6. 是否缺少測試。
7. 是否符合需求。

請輸出到 AI_REVIEW.md：
- Blocker
- Major
- Minor
- 建議修正方向
- 是否建議進入下一輪
```

### 13.6 建議檔案：AI_TASK.md

```md
# AI Task

## 任務目標

## 背景

## 驗收條件

## 限制

- commit / push 依 SKILL 授權規則，沒有規定就先問過我
- 不得修改無關模組
- 不得上傳 secrets

## 測試要求

## 交付格式
```

### 13.7 建議檔案：AI_REVIEW.md

```md
# AI Review

## Review Round 1

### Blocker

### Major

### Minor

### 建議修正

### 結論
```

### 13.8 建議指令

Claude 修改後：

```bash
git status
git diff --stat
git diff
```

讓 Codex 審查：

```bash
git diff | codex "Review this diff. Do not modify files. Output risks and suggested fixes."
```

或在 Codex CLI 內下：

```txt
請審查目前 git diff，只審查，不修改檔案。
```

### 13.9 退出條件

符合任一條件就停止自動循環，交給人類決策：

- 超過 3 輪仍有 blocker
- 測試環境缺失
- 權限不足
- 需求不明確
- 涉及 production / 金流 / 個資 / 安全敏感流程

### 13.10 最終交付報告格式

```md
# Final Delivery Report

## 任務摘要

## 修改範圍

## 驗證結果

## Codex Review 結論

## 剩餘風險

## 建議下一步
```

### 13.11 結論

Claude / Codex 交叉驗證的目的不是追求形式，而是建立「實作與審查分離」的治理模型。

這套流程特別適合：

- 大型重構
- API 串接
- 金流 / 高風險業務 / 會員流程
- CLI 工具開發
- 需要降低回歸風險的修改
