# Setup Guide

Step-by-step instructions to deploy the Financial Research Telegram Bot.

## Prerequisites

- An n8n instance (cloud at app.n8n.cloud or self-hosted)
- A Telegram account
- An Anthropic API key (claude.ai console)
- A Google account (for Drive integration)
- An html2pdf.app account (free tier)

## Step 1: Create a Telegram Bot

1. Open Telegram and search for **@BotFather**
2. Send `/newbot` and follow the prompts to name your bot
3. Copy the **HTTP API token** — you'll need this in n8n

## Step 2: Get API Keys

### Anthropic (Claude)
1. Go to [console.anthropic.com](https://console.anthropic.com)
2. Create an account and generate an API key
3. Add credits to your account (usage is pay-per-call, ~$0.01-0.03 per report)

### html2pdf.app
1. Go to [html2pdf.app](https://html2pdf.app)
2. Create a free account (100 conversions/month)
3. Copy your API key from the dashboard

## Step 3: Import the Workflow into n8n

1. Download `workflow.json` from this repository
2. In n8n, go to **Workflows** → click the **⋮** menu → **Import from File**
3. Select the `workflow.json` file
4. The workflow will load with all nodes pre-configured

## Step 4: Configure Credentials

After importing, you need to set up credentials for each external service:

### Telegram Bot
1. Click the **Telegram Trigger** node
2. Click **Create New Credential**
3. Paste your Bot API Token from BotFather
4. Save

### Anthropic API
1. Click the **LLM Summary** node (HTTP Request)
2. In the Headers section, replace the `x-api-key` value with your Anthropic API key

### html2pdf.app
1. Click the **Convert to PDF** node
2. In the JSON body, replace `YOUR_HTML2PDF_API_KEY` with your actual key

### Google Drive
1. Click the **Upload to Drive** node
2. Click **Create New Credential** → Google Drive OAuth2
3. Follow the OAuth flow to authorize your Google account
4. Select or create a destination folder (e.g., "Financial Bot Reports")

## Step 5: Activate the Workflow

1. Click **Publish** (or the toggle in the top-right corner)
2. The workflow is now live and listening for Telegram messages 24/7

## Step 6: Test

1. Open Telegram and find your bot
2. Send a ticker: `AAPL`
3. You should receive:
   - A text summary with AI analysis and news links
   - A PDF report attached
   - The PDF saved in your Google Drive folder

## Configuration Notes

### Custom Asset Mappings

To add more cryptocurrencies, indices, or commodities, edit the **Code in JavaScript** node (Input Parser). Add entries to the appropriate map:

```javascript
// Add a new crypto
'PEPE': 'PEPE-USD',

// Add a new index
'RUSSELL': '^RUT',

// Add a new commodity  
'PALLADIUM': 'PA=F',
```

### Changing the AI Model

In the **Build LLM Prompt** node, change the model in the `requestBody`:

```javascript
model: "claude-sonnet-4-20250514"  // Current
model: "claude-haiku-4-5-20251001" // Cheaper, faster alternative
```

### Adjusting the Report Style

Edit the **Generate HTML Report** node to modify colors, fonts, layout, or add additional sections to the PDF.

### Google Drive Folder

The Upload to Drive node stores files with the naming convention:
```
{TICKER}_Report_{YYYY-MM-DD}.pdf
```
Example: `AAPL_Report_2026-03-06.pdf`

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| Bot doesn't respond | Workflow not activated | Click Publish/toggle ON |
| Empty price data | Invalid ticker | Check Yahoo Finance supports the symbol |
| Currency shows wrong symbol | Ticker not in Yahoo format | Add mapping to Input Parser |
| PDF not generated | html2pdf.app quota exceeded | Check your monthly usage (100 free/month) |
| Google Drive upload fails | OAuth token expired | Re-authorize in n8n credentials |
| AI summary mentions wrong company | News search returned irrelevant articles | The prompt filters these; try a more specific ticker |
| Rate limit errors | Too many rapid requests | Yahoo Finance allows ~2000 req/day; add caching if needed |
