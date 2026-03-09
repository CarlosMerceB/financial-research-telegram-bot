# Architecture

Technical blueprint for the Financial Research Telegram Bot.

## Workflow Diagram

```
┌──────────────┐
│   Telegram    │
│   Trigger     │
└──────┬───────┘
       │ message.text, chat.id
       ▼
┌──────────────┐
│  Input Parser │  Classifies asset type, maps to Yahoo Finance format
│  (Code)       │  Stock: pass-through | Crypto: BTC→BTC-USD | Index: SPX→^GSPC
└──────┬───────┘
       │ ticker, assetType, chatId
       ▼
┌──────────────┐
│   Switch      │  Routes by assetType (stock/crypto/index/commodity)
│   (Router)    │  All 4 outputs connect to the same downstream node
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  GET Stock    │  Yahoo Finance: /v8/finance/chart/{ticker}?range=1y&interval=1d
│  Prices (HTTP)│  Returns: timestamps[], OHLCV arrays, meta (currency, name)
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  Calculate    │  Extracts: currentPrice, prevClose, dailyChange, ytdGrowth, yoyGrowth
│  Metrics(Code)│  Reads: meta.longName (company name), meta.currency (dynamic)
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  GET News     │  Yahoo Finance: /v1/finance/search?q={companyName}+stock&newsCount=3
│  (HTTP)       │  Returns: news[] with title, publisher, link
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  Merge Data   │  Combines metrics + top 3 news articles into single payload
│  (Code)       │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  Build LLM    │  Constructs structured analyst prompt with all data
│  Prompt(Code) │  Includes: sector identification, news relevance filtering
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  LLM Summary  │  POST to Anthropic /v1/messages (Claude Sonnet)
│  (HTTP)       │  Returns: structured research flash note
└──────┬───────┘
       │
       ├────────────────────────┐
       ▼                        ▼
┌──────────────┐        ┌──────────────┐
│  Send Text    │        │  Generate    │
│  Summary      │        │  HTML Report │
│  (Telegram)   │        │  (Code)      │
└──────────────┘        └──────┬───────┘
                               │
                               ▼
                        ┌──────────────┐
                        │  Convert to   │
                        │  PDF (HTTP)   │
                        │  html2pdf.app │
                        └──────┬───────┘
                               │
                        ┌──────┴───────┐
                        ▼              ▼
                 ┌──────────┐  ┌──────────┐
                 │ Upload to │  │ Send PDF  │
                 │ Drive     │  │ (Telegram)│
                 └──────────┘  └──────────┘
```

## Node Details

### Node 1: Telegram Trigger
- **Listens for:** `message` updates
- **Outputs:** `message.text` (user input), `message.chat.id` (for reply routing)

### Node 2: Input Parser (Code — JavaScript)
Classifies the user input and converts it to Yahoo Finance-compatible format:

- **Stocks:** Any unrecognized input passes through as-is (e.g., `AAPL`, `IDR.MC`, `7203.T`)
- **Crypto:** Maps common names/tickers to `{SYMBOL}-USD` format (20+ supported)
- **Indices:** Maps names to `^{SYMBOL}` format (12+ global indices)
- **Commodities:** Maps names to `{SYMBOL}=F` format (15+ commodities)

### Node 3: Switch Router
Routes on `$json.assetType`. All 4 outputs connect to the same GET Stock Prices node (Yahoo Finance handles all asset types uniformly).

### Node 4: GET Stock Prices (HTTP Request)
```
GET https://query1.finance.yahoo.com/v8/finance/chart/{ticker}
    ?range=1y&interval=1d
```
- **No API key required**
- Returns `chart.result[0]` with:
  - `meta.currency` — dynamic currency code
  - `meta.longName` / `meta.shortName` — official company name
  - `timestamp[]` — array of Unix timestamps
  - `indicators.adjclose[0].adjclose[]` — adjusted close prices

### Node 5: Calculate Metrics (Code)
Processes raw Yahoo data into clean metrics:
- **Current Price:** Last element in the close array
- **Previous Close:** Second-to-last element
- **Daily Change:** `(current - prevClose) / prevClose * 100`
- **YTD Growth:** Finds first trading day of current year, calculates % change
- **YoY Growth:** Uses first data point (~365 days ago) as baseline
- **Currency Symbol:** Maps ISO code to display symbol (€, $, £, ¥, etc.)

### Node 6: GET News (HTTP Request)
```
GET https://query1.finance.yahoo.com/v1/finance/search
    ?q={companyName}+stock&newsCount=3&quotesCount=0
```
Searches by company name (not ticker) for better relevance.

### Node 7: Merge Data (Code)
Formats the top 3 news articles and combines with all metrics into a single JSON payload.

### Node 8: Build LLM Prompt (Code)
Constructs a structured prompt that instructs Claude to act as a senior research analyst. Key prompt elements:
- Asset profile with all market data
- News headlines for context
- Required output format: SECTOR → MARKET SNAPSHOT → NEWS IMPACT → RISK FACTORS → OUTLOOK
- Explicit instructions to ignore unrelated headlines
- Character limit enforcement (~1200 chars)

### Node 9: LLM Summary (HTTP Request)
```
POST https://api.anthropic.com/v1/messages
Headers: x-api-key, anthropic-version: 2023-06-01
Body: { model: "claude-sonnet-4-20250514", max_tokens: 1024, messages: [...] }
```

### Node 10: Send Text Summary (Telegram)
Sends the formatted text message with:
- Price data with dynamic currency symbols
- Full AI analysis
- Clickable news links (Telegram Markdown `[title](url)`)

### Node 11: Generate HTML Report (Code)
Builds a complete HTML document with:
- Color-coded metrics (green for positive, red for negative)
- Structured AI analysis with section headers
- Clickable news links
- Professional table layout
- Footer disclaimer

### Node 12: Convert to PDF (HTTP Request)
```
POST https://api.html2pdf.app/v1/generate
Body: { html: "...", apiKey: "...", format: "A4", margin: {...} }
Response Format: File (binary)
```

### Node 13a: Upload to Drive (Google Drive)
Uploads the PDF binary to a specified Google Drive folder with filename: `{TICKER}_Report_{YYYY-MM-DD}.pdf`

### Node 13b: Send PDF (Telegram)
Sends the PDF as a document attachment to the user's chat.

## API Rate Limits

| API | Free Tier Limit | Impact |
|-----|----------------|--------|
| Yahoo Finance (chart) | ~2,000 req/day (unofficial) | Main data source |
| Yahoo Finance (search) | ~2,000 req/day (unofficial) | News fetching |
| Anthropic Claude | Pay-per-token (~$0.003/1K input) | ~$0.01-0.03 per report |
| html2pdf.app | 100 conversions/month | PDF generation |
| Google Drive | 15 GB storage free | PDF storage |
| Telegram Bot | Unlimited messages | No limit |

## Known Limitations

1. **Yahoo Finance unofficial API** — no SLA, endpoints may change. The v8/chart endpoint has been stable for years but is not officially documented.
2. **No caching** — each request makes fresh API calls. Future enhancement could cache recent results.
3. **No ticker validation** — invalid tickers return empty data rather than a user-friendly error.
4. **News relevance** — searching by company name works well but may occasionally return tangential results for companies with common names.
5. **Crypto/commodity maps are manual** — assets not in the predefined maps default to stock treatment and may fail.
