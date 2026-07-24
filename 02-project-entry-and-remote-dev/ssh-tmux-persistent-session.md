# SSH + tmux Persistent Session

這份文件處理遠端 AI 長任務最常見的問題：

```txt
SSH 斷線、terminal 關掉、遠端桌面切換、網路不穩，任務會不會中斷？
```

答案是：如果直接在 SSH 裡跑，可能會中斷；如果放在 tmux 裡跑，就可以大幅降低風險。

## 1. tmux 是什麼

tmux 是 terminal multiplexer，可以把 terminal session 留在遠端主機上。

你可以：

- 開一個長時間 session
- 中途 detach 離開
- 斷線後重新 attach 回去
- 讓 Claude / Codex 長任務不中斷

## 2. 安裝 tmux

macOS：

```bash
brew install tmux
```

Ubuntu / Debian：

```bash
sudo apt update
sudo apt install tmux
```

確認：

```bash
tmux -V
```

## 3. 建立 session

```bash
tmux new -s ai-work
```

建議命名：

```bash
tmux new -s claude-linebot
tmux new -s codex-review
tmux new -s deploy-check
```

## 4. 在 tmux 裡跑 AI CLI

```bash
cd <PROJECT_PATH>
git status
claude
```

或：

```bash
cd <PROJECT_PATH>
codex
```

## 5. Detach 離開但不中斷

按鍵：

```txt
Ctrl + b
放開
按 d
```

你會回到原本 shell，但 tmux 裡的任務還在。

## 6. 重新 attach

列出 session：

```bash
tmux ls
```

回到 session：

```bash
tmux attach -t ai-work
```

## 7. 關閉 session

在 tmux 裡正常結束程式後：

```bash
exit
```

或從外部 kill：

```bash
tmux kill-session -t ai-work
```

## 8. 常用指令速查

| 動作 | 指令 |
|---|---|
| 新增 session | `tmux new -s <NAME>` |
| 列出 session | `tmux ls` |
| 回到 session | `tmux attach -t <NAME>` |
| detach | `Ctrl+b` 再按 `d` |
| 關閉 session | `tmux kill-session -t <NAME>` |

## 9. 建議 AI 長任務流程

```bash
ssh <USER>@<HOST>
tmux new -s ai-work
cd <PROJECT_PATH>
git status
claude
```

中途要離開：

```txt
Ctrl+b, d
```

回來：

```bash
ssh <USER>@<HOST>
tmux attach -t ai-work
```

## 10. 為什麼對 AI 很重要

Claude / Codex 常常會：

- 掃描大型專案
- 執行測試
- 跑多輪修改
- 產生長輸出
- 等待人工確認

如果沒有 tmux，一斷線就可能中斷或失去上下文。

## 11. 建議治理策略

所有超過 5 分鐘的遠端任務，都應該放在 tmux。

尤其是：

- AI 大型重構
- build / test
- iOS build
- 後端服務啟動
- log 監控
- 長時間 Codex review

## 12. 結論

SSH 是入口，tmux 是續航。

只用 SSH 是遠端連線；SSH + tmux 才是穩定的遠端工作平台。