# 💰 AI Finance Assistant Agent

An AI-powered personal finance assistant built with **n8n**, **Groq**, **Google Sheets**, and **Gmail** — with a custom "Finance Passbook" chat UI (HTML/CSS/JS).

Chat with it naturally to log expenses/income, get spending insights, and receive email confirmations — all backed by a Google Sheet as your ledger.

---

## 📁 Files in this project

| File | Purpose |
|---|---|
| `AI_Finance_Assistant_Agent.json` | n8n workflow — import this into n8n |
| `finance_passbook_chat.html` | Standalone chat frontend (single file, no build step) |

---

## ⚙️ How it works

```
User message
     │
     ▼
Chat Webhook (n8n)
     │
     ▼
Finance AI Agent  ──────►  Groq Chat Model (qwen/qwen3.6-27b)
     │                     Chat Memory (per-session)
     │
     ├──► Log Transaction (Sheets Tool)      → appends a row to your Google Sheet
     ├──► Read Transactions (Sheets Tool)     → reads all rows for analysis/totals
     └──► Send Finance Report (Gmail Tool)    → emails a confirmation or summary
     │
     ▼
Respond to Chat (back to the frontend)
```

The agent decides which tools to call based on what you say. By default it:
- Logs a transaction whenever you mention spending/earning money
- Automatically sends a short confirmation email after every logged transaction
- Reads and analyzes your sheet when you ask about totals, categories, or trends
- Sends a fuller summary email only when you explicitly ask for one

---

## 🚀 Setup

### 1. Google Sheet
Create a sheet with a tab named exactly **`Transactions`**, and add this header row:

| Date | Category | Amount | Note | Type |
|---|---|---|---|---|

> ⚠️ Without this header row, n8n can't detect columns and the tool nodes will show "No columns found."

### 2. Import the n8n workflow
1. n8n → **Import from File** → select `AI_Finance_Assistant_Agent.json`
2. Set up credentials on each node:
   - **Groq Chat Model** → Groq API credential
   - **Log Transaction** & **Read Transactions** (Sheets Tool nodes) → Google Sheets OAuth2 credential, then pick your actual spreadsheet + `Transactions` sheet from the dropdowns
   - **Send Finance Report** (Gmail Tool) → Gmail OAuth2 credential, and set the **To** field (Fixed mode) to your own email address
3. Click **Active** (top-right toggle) to turn the workflow on
4. Copy the **production webhook URL** from the Chat Webhook node — it will look like:
   ```
   http://localhost:5678/webhook/finance-assistant-chat
   ```

### 3. Frontend
Open `finance_passbook_chat.html` in a browser (or host it anywhere — it's a static file).
If your webhook URL is different from the default, update this line near the bottom of the file:

```js
const WEBHOOK_URL = "http://localhost:5678/webhook/finance-assistant-chat";
```

---

## 🧪 Testing

**Via curl** (useful before the frontend is wired up):
```bash
curl -X POST http://localhost:5678/webhook/finance-assistant-chat \
  -H "Content-Type: application/json" \
  -d '{"message": "Spent 500 on groceries today", "sessionId": "test-1"}'
```

**Via the frontend:** just open the HTML file and type a message, or use one of the quick-action buttons (Monthly total / By category / Email summary).

---

## 🩹 Troubleshooting

| Symptom | Fix |
|---|---|
| "No columns found in Google Sheets Tool" | Add the header row (Date, Category, Amount, Note, Type) to your sheet, then click **Retry** in the node |
| `400 tool call validation failed ... expected string, but got number` | Check the `$fromAI(...)` type argument on that field matches what the model actually sends — Amount uses `'number'`, the rest use `'string'` |
| `400 Parsing failed. The model generated output that could not be parsed` | Usually a model/tool-calling compatibility issue. Try lowering temperature to 0, or switch models (this workflow uses `qwen/qwen3.6-27b` for reliable tool calls) |
| `The OAuth client was deleted` | Your Google Cloud OAuth client was removed. Create a new one in Google Cloud Console (or just re-authorize if using n8n's built-in Google login) and update the credential in n8n |
| Frontend shows "Could not reach the ledger" | Make sure the workflow is **Active** in n8n and `WEBHOOK_URL` in the HTML file matches the production webhook URL exactly |

---

## 🛠️ Tech Stack
- **n8n** — workflow orchestration
- **Groq** (`qwen/qwen3.6-27b`) — LLM reasoning + tool calling
- **Google Sheets** — transaction ledger / data store
- **Gmail** — automated email confirmations & summaries
- **HTML/CSS/JS** — lightweight standalone chat frontend
