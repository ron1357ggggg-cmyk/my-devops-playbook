# Write Outlook Calendar (AI Skill)

把聊天內容、截圖、預約單或零散筆記，轉成 Outlook / Microsoft 365 行事曆的私人事件。

## 檔案用途

| 檔案 | 用途 |
|---|---|
| [SKILL.md](SKILL.md) | Claude Code skill 定義本體：抽取事件欄位、隱私規則（不加與會者、不開 Teams 會議）、瀏覽器操作 Outlook 的步驟、多筆提醒的處理方式、寫入失敗時的 `.ics` 備援格式與驗證流程。純 ASCII 撰寫，方便複製到其他 AI 工具而不會有編碼問題。 |
| [agents/openai.yaml](agents/openai.yaml) | 同一顆 skill 給 OpenAI 系 agent（例如 Codex CLI / ChatGPT 自訂 agent）讀取的精簡 manifest（`display_name` / `short_description` / `default_prompt`），讓同一個 skill 可以跨 Claude 與 OpenAI 兩種 agent 框架使用。 |

## 使用情境

當使用者要求「幫我把這個加到行事曆」、「這張截圖的預約幫我寫進 Outlook」時觸發。偏好順序：

1. 有 Outlook / Microsoft Graph connector 就直接呼叫。
2. 沒有 connector，就用使用者已登入的瀏覽器操作 Outlook 網頁版行事曆。
3. 瀏覽器登入被卡住時，保留分頁讓使用者手動登入，並產生 `.ics` 備援檔。

詳細規則見 [SKILL.md](SKILL.md)。
