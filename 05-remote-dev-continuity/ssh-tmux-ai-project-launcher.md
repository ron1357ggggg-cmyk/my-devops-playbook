# SSH + tmux AI Project Launcher

這份文件記錄一套可複製到新電腦的 SSH / tmux / AI CLI 工作流。

目標：

- SSH 進主機後，用 `projects` 列出常用專案。
- 用 `proj <alias>` 進入專案 shell tmux session。
- 用 `proj <alias> claude` 進入該專案的 Claude 主 session。
- 用 `proj <alias> codex` 進入該專案的 Codex 主 session。
- Claude 與 Codex 必須是不同 tmux session。
- 每個 AI 都可以建立獨立 `NEW TASK` session。
- `Delete task` 只能刪 task session，不能刪主 AI session。

## 本次實作版本紀錄

這份 SOP 來自一次實際 SSH/tmux/Codex/Claude launcher 調整，最後確認採用以下行為：

1. 專案 shell、Claude main、Codex main 全部分成不同 tmux session。
2. Claude 與 Codex 不能共用同一個 tmux session，也不要只用同 session 的不同 window 區分。
3. `NEW TASK` 每次都要建立新的 tmux session，不可因為同名 task 已存在就 attach 舊 session。
4. 如果 task 名稱重複，自動加序號，例如 `test1`、`test1-2`、`test1-3`。
5. AI 子選單要列出已存在的 task，讓使用者可以直接選回舊 task。
6. AI 子選單順序固定為：`Main <ai>`、既有 task 清單、`NEW TASK`、`Delete task`。
7. `Delete task` 只能列出並刪除 `<alias>-<ai>-task-*`，不能列出或刪除 `<alias>-claude` / `<alias>-codex`。
8. 已在 tmux 內執行 `proj` 時，用 `tmux switch-client`；一般 SSH shell 才用 `tmux attach`。
9. 主 AI session 用 resume 指令保留長期上下文；`NEW TASK` 用 fresh 啟動指令建立新的 AI 任務。
10. `prompt_task_name()` 這類函式若用 command substitution 接回傳值，提示文字必須輸出到 stderr，stdout 只能輸出乾淨的 task slug。

## 1. 設計原則

不要把同一專案的 Claude 與 Codex 放在同一個 tmux session 裡。

建議 session 命名：

```txt
<alias>                 # 專案 shell session
<alias>-claude          # Claude 主 session，不能由 task delete 刪
<alias>-codex           # Codex 主 session，不能由 task delete 刪
<alias>-claude-task-xxx # Claude 單一任務 session
<alias>-codex-task-xxx  # Codex 單一任務 session
```

範例：

```txt
chatgpt-line-bridge
chatgpt-line-bridge-claude
chatgpt-line-bridge-codex
chatgpt-line-bridge-claude-task-fix-login
chatgpt-line-bridge-codex-task-review-diff
```

## 2. 使用方式

列出已知專案：

```bash
projects
```

互動式選專案、AI、task：

```bash
proj
```

直接進專案 shell：

```bash
proj chatgpt-line-bridge
```

直接進 Claude 主 session：

```bash
proj chatgpt-line-bridge claude
```

直接進 Codex 主 session：

```bash
proj chatgpt-line-bridge codex
```

建立新的 Claude task：

```bash
proj chatgpt-line-bridge claude new fix-startup
```

建立新的 Codex task：

```bash
proj chatgpt-line-bridge codex new review-auth-flow
```

刪除 Claude task：

```bash
proj chatgpt-line-bridge claude delete
```

刪除 Codex task：

```bash
proj chatgpt-line-bridge codex delete
```

刪除清單只能出現：

```txt
<alias>-claude-task-*
<alias>-codex-task-*
```

不能出現：

```txt
<alias>-claude
<alias>-codex
```

## 3. 安裝前提

需要：

```bash
tmux -V
command -v claude
command -v codex
```

若 `~/.local/bin` 不在 PATH，加入 shell 設定：

```bash
mkdir -p ~/.local/bin
grep -q 'HOME/.local/bin' ~/.bashrc || echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

## 4. `projects` 腳本模板

建立：

```bash
mkdir -p ~/.local/bin
nano ~/.local/bin/projects
chmod +x ~/.local/bin/projects
```

內容範例：

```bash
#!/usr/bin/env bash
cat <<'EOF'
chatgpt-line-bridge  /mnt/c/users/user/Documents/chatgpt-line-bridge
linebot-server       /mnt/c/users/user/Documents/Codex/2026-05-14/git/linebot-server
youtube-clock        /mnt/c/users/user/Documents/Codex/2026-05-14/git/YoutubeClock
EOF
```

新電腦要改成自己的專案 alias 與本機路徑。

## 5. `proj` 腳本模板

建立：

```bash
mkdir -p ~/.local/bin
nano ~/.local/bin/proj
chmod +x ~/.local/bin/proj
```

內容範例：

```bash
#!/usr/bin/env bash
set -euo pipefail

show_projects() {
  cat <<'EOF'
Select a project:
1) chatgpt-line-bridge
2) linebot-server
3) youtube-clock
EOF
}

name="${1:-}"
mode="${2:-}"
action="${3:-}"
task_name="${4:-}"
interactive=0

if [ -z "$name" ]; then
  interactive=1
  show_projects
  printf "Project number: "
  read -r name
fi

case "$name" in
  1|chatgpt-line-bridge)
    name="chatgpt-line-bridge"
    dir="/mnt/c/users/user/Documents/chatgpt-line-bridge"
    ;;
  2|linebot-server)
    name="linebot-server"
    dir="/mnt/c/users/user/Documents/Codex/2026-05-14/git/linebot-server"
    ;;
  3|youtube-clock)
    name="youtube-clock"
    dir="/mnt/c/users/user/Documents/Codex/2026-05-14/git/YoutubeClock"
    ;;
  ""|-h|--help|help)
    show_projects
    exit 0
    ;;
  *)
    echo "Unknown project: $name" >&2
    show_projects >&2
    exit 2
    ;;
esac

if [ ! -d "$dir" ]; then
  echo "Project directory not found: $dir" >&2
  exit 1
fi

attach_session() {
  local session="$1"

  if [ -n "${TMUX:-}" ]; then
    exec tmux switch-client -t "$session"
  fi

  exec tmux attach -t "$session"
}

ensure_session() {
  local session="$1"
  local window="$2"
  local command="$3"

  if ! tmux has-session -t "$session" 2>/dev/null; then
    tmux new-session -Ad -s "$session" -n "$window" -c "$dir" "$command"
  fi
}

next_task_session_name() {
  local base="$1"
  local candidate="$base"
  local index=2

  while tmux has-session -t "$candidate" 2>/dev/null; do
    candidate="${base}-${index}"
    index=$((index + 1))
  done

  printf '%s' "$candidate"
}

sanitize_task_name() {
  printf '%s' "$1" \
    | tr '[:upper:]' '[:lower:]' \
    | sed -E 's/[^a-z0-9._-]+/-/g; s/^-+//; s/-+$//'
}

prompt_task_name() {
  local raw=""
  local slug=""

  if [ -n "$task_name" ]; then
    raw="$task_name"
  else
    printf "Task name (letters, numbers, dash, underscore): " >&2
    read -r raw
  fi

  slug="$(sanitize_task_name "$raw")"
  if [ -z "$slug" ]; then
    echo "Task name must include at least one letter or number." >&2
    exit 2
  fi

  printf '%s' "$slug"
}

show_ai_menu() {
  local ai="$1"
  shift
  local sessions=("$@")
  local index=2
  local prefix="${name}-${ai}-task-"
  local label=""

  echo "Select ${ai} task:"
  echo "1) Main ${ai}"

  for session in "${sessions[@]}"; do
    label="${session#"$prefix"}"
    printf "%s) %s\n" "$index" "$label"
    index=$((index + 1))
  done

  printf "%s) NEW TASK\n" "$index"
  index=$((index + 1))
  printf "%s) Delete task\n" "$index"
}

list_task_sessions() {
  local ai="$1"
  local prefix="${name}-${ai}-task-"

  tmux list-sessions -F '#S' 2>/dev/null | while IFS= read -r session; do
    case "$session" in
      "$prefix"*) printf '%s\n' "$session" ;;
    esac
  done
}

delete_task_session() {
  local ai="$1"
  local sessions=()
  local choice=""
  local index=1

  mapfile -t sessions < <(list_task_sessions "$ai")

  if [ "${#sessions[@]}" -eq 0 ]; then
    echo "No ${ai} task sessions found for ${name}."
    exit 0
  fi

  echo "Delete which ${ai} task?"
  for session in "${sessions[@]}"; do
    printf "%s) %s\n" "$index" "$session"
    index=$((index + 1))
  done

  printf "Task number: "
  read -r choice

  if ! [[ "$choice" =~ ^[0-9]+$ ]] || [ "$choice" -lt 1 ] || [ "$choice" -gt "${#sessions[@]}" ]; then
    echo "Invalid task number: $choice" >&2
    exit 2
  fi

  tmux kill-session -t "${sessions[$((choice - 1))]}"
  echo "Deleted ${sessions[$((choice - 1))]}"
}

run_ai_mode() {
  local ai="$1"
  local main_command="$2"
  local task_command="$3"
  local main_session="${name}-${ai}"
  local task_session=""
  local task_slug=""
  local task_sessions=()
  local new_index=0
  local delete_index=0

  if [ "$interactive" -eq 1 ] && [ -z "$action" ]; then
    mapfile -t task_sessions < <(list_task_sessions "$ai")
    show_ai_menu "$ai" "${task_sessions[@]}"
    printf "Task option: "
    read -r action
  fi

  if [[ "$action" =~ ^[0-9]+$ ]]; then
    if [ "$action" -eq 1 ]; then
      action="main"
    else
      if [ "${#task_sessions[@]}" -eq 0 ]; then
        mapfile -t task_sessions < <(list_task_sessions "$ai")
      fi

      if [ "$action" -ge 2 ] && [ "$action" -le $((${#task_sessions[@]} + 1)) ]; then
        attach_session "${task_sessions[$((action - 2))]}"
      fi

      new_index=$((${#task_sessions[@]} + 2))
      delete_index=$((${#task_sessions[@]} + 3))

      if [ "$action" -eq "$new_index" ]; then
        action="new"
      elif [ "$action" -eq "$delete_index" ]; then
        action="delete"
      fi
    fi
  fi

  case "$action" in
    ""|1|main)
      ensure_session "$main_session" "$ai" "$main_command"
      attach_session "$main_session"
      ;;
    2|new|new-task|task)
      task_slug="$(prompt_task_name)"
      task_session="$(next_task_session_name "${name}-${ai}-task-${task_slug}")"
      tmux new-session -Ad -s "$task_session" -n "$ai" -c "$dir" "$task_command"
      attach_session "$task_session"
      ;;
    3|delete|delete-task|rm)
      delete_task_session "$ai"
      ;;
    -h|--help|help)
      mapfile -t task_sessions < <(list_task_sessions "$ai")
      show_ai_menu "$ai" "${task_sessions[@]}"
      ;;
    *)
      echo "Unknown task option: $action" >&2
      echo "Use: main, new, or delete" >&2
      exit 2
      ;;
  esac
}

if [ "$interactive" -eq 1 ] && [ -z "$mode" ]; then
  cat <<'EOF'
Select mode:
1) shell only
2) claude
3) codex
EOF
  printf "Mode number: "
  read -r mode
fi

case "$mode" in
  ""|1|shell)
    exec tmux new -As "$name" -c "$dir"
    ;;
  2|claude)
    run_ai_mode "claude" 'claude --permission-mode bypassPermissions -c' 'claude --permission-mode bypassPermissions'
    ;;
  3|codex)
    run_ai_mode "codex" 'codex resume --last --all' 'codex'
    ;;
  -h|--help|help)
    cat <<'EOF'
Usage:
  proj
  proj <project-number|project-alias>
  proj <project-number|project-alias> shell
  proj <project-number|project-alias> claude [main|new|delete] [task-name]
  proj <project-number|project-alias> codex [main|new|delete] [task-name]
EOF
    ;;
  *)
    echo "Unknown mode: $mode" >&2
    echo "Use: shell, claude, or codex" >&2
    exit 2
    ;;
esac
```

## 6. 全域 AI 規則模板

Codex 全域規則可放：

```txt
~/.codex/AGENTS.md
```

Claude 全域規則可放：

```txt
~/.claude/CLAUDE.md
```

建議加入：

```markdown
## Tmux project workflow

- Use `projects` to list known project aliases.
- Use `proj <alias>` to create or attach the project shell tmux session.
- Use `proj <alias> claude` to create or attach that project's Claude tmux session.
- Use `proj <alias> codex` to create or attach that project's Codex tmux session.
- Claude and Codex must use separate tmux sessions, for example `<alias>-claude` and `<alias>-codex`.
- After choosing Claude or Codex interactively, use `Main`, `NEW TASK`, or `Delete task`.
- `NEW TASK` creates a separate task session named `<alias>-<ai>-task-<task-name>`.
- `Delete task` may only delete `<alias>-<ai>-task-*` sessions. Never delete the main `<alias>-claude` or `<alias>-codex` sessions through the task-delete flow.
- Use `Ctrl+b d` to detach without stopping work.
```

## 7. 給 AI 在新電腦設定用的 prompt

可以把下面整段交給 Claude / Codex：

```txt
請幫我在這台電腦設定 SSH/tmux/AI project launcher 工作流。

需求：
1. 先檢查 `tmux -V`、`command -v claude`、`command -v codex`、`echo $PATH`。
2. 確保 `~/.local/bin` 存在且在 PATH。
3. 建立 `~/.local/bin/projects`，列出我提供的 project alias 與絕對路徑。
4. 建立 `~/.local/bin/proj`，支援：
   - `proj` 互動式選專案。
   - `proj <alias>` 進入 `<alias>` shell tmux session。
   - `proj <alias> claude` 進入 `<alias>-claude` 主 session。
   - `proj <alias> codex` 進入 `<alias>-codex` 主 session。
   - Claude/Codex 選單都要有 `Main`、`NEW TASK`、`Delete task`。
   - `NEW TASK` 建立 `<alias>-<ai>-task-<task-name>`。
   - `Delete task` 只能列出並刪除 `<alias>-<ai>-task-*`，不可列出或刪除 `<alias>-claude` / `<alias>-codex` 主 session。
   - 如果已經在 tmux 裡執行 `proj`，要用 `tmux switch-client`；如果在一般 SSH shell，才用 `tmux attach`。
5. 更新 `~/.codex/AGENTS.md` 與 `~/.claude/CLAUDE.md`，寫入上述 tmux 工作流規則。
6. 做驗證：
   - `bash -n ~/.local/bin/proj`
   - `projects`
   - `proj help`
   - 用互動輸入確認 Claude/Codex 會顯示 `NEW TASK` 與 `Delete task`
   - 建立一個臨時 `<alias>-codex-task-proj-validation` session，再用 delete flow 刪除，確認主 `<alias>-codex` 與 `<alias>-claude` 沒被刪。
7. 不要寫入任何密碼、token、私鑰。

我的 project list：
<在這裡貼上 alias 與 path>
```

## 8. 驗證指令

語法：

```bash
bash -n ~/.local/bin/proj
```

列專案：

```bash
projects
```

看 session：

```bash
tmux list-sessions
```

確認主 session 與 task session 分離：

```bash
proj <alias> claude
proj <alias> codex
proj <alias> claude new test-task
proj <alias> codex new review-task
tmux list-sessions
```

預期會看到：

```txt
<alias>-claude
<alias>-codex
<alias>-claude-task-test-task
<alias>-codex-task-review-task
```

刪除 task：

```bash
proj <alias> claude delete
proj <alias> codex delete
```

刪除後主 session 仍應存在：

```bash
tmux list-sessions
```

## 9. 常見錯誤

### 選 Claude / Codex 後看起來都進同一個 tmux

通常是把 Claude/Codex 做成同一個 tmux session 裡的不同 window，或用了不可靠的：

```bash
tmux attach -t "$session:$window"
```

建議改成不同 session：

```txt
<alias>-claude
<alias>-codex
```

### 在 tmux 裡跑 `proj` 又 attach 失敗

在 tmux 內不能再用一般 attach 流程，應該使用：

```bash
tmux switch-client -t "$session"
```

腳本應該檢查：

```bash
if [ -n "${TMUX:-}" ]; then
  exec tmux switch-client -t "$session"
fi
```

### Delete task 誤刪主 AI session

刪除流程只能匹配：

```txt
<alias>-<ai>-task-*
```

不要讓使用者直接從 `Delete task` 選到：

```txt
<alias>-claude
<alias>-codex
```

## 10. 結論

這套流程把三種狀態拆清楚：

1. 專案 shell：處理一般檢查與手動操作。
2. AI 主 session：保留 Claude/Codex 長期上下文。
3. AI task session：處理可刪除、可獨立中斷的新任務。

主 AI session 是長期工作台，task session 是一次性工作區。刪 task 不能動主 session。
