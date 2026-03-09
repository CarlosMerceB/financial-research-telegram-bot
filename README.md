# Financial Research Telegram Bot

An automated financial research assistant built with **n8n** that delivers AI-powered market reports directly to Telegram — with a PDF backup saved to Google Drive.

Send any ticker, company name, cryptocurrency, index, or commodity to the bot and receive a complete market briefing within seconds.

## What It Does

A user sends a message like `AAPL`, `BTC`, `GOLD`, or `IDR.MC` to the Telegram bot. The workflow then:

1. **Classifies the asset** — determines whether it's a stock, crypto, index, or commodity
2. **Fetches market data** — pulls 1 year of daily OHLCV via Yahoo Finance (current price, previous close, YTD/YoY growth)
3. **Fetches relevant news** — retrieves the 3 most recent headlines for the asset
4. **Generates an AI analysis** — sends all data to Claude (Anthropic) which produces a structured research flash note
5. **Creates a formatted PDF report** — converts the analysis into a professional HTML-to-PDF document
6. **Uploads to Google Drive** — stores the PDF in a designated folder for archival
7. **Delivers via Telegram** — sends the text summary + PDF attachment back to the user

## Sample Output

The Telegram message includes:

```
📊 Apple Inc. ($AAPL)

💰 Price: $227.48 USD
📉 Prev Close: $228.14
📈 Daily: -0.29% | YTD: -8.42% | YoY: 8.15%

🤖 AI Analysis:

SECTOR: Technology | Consumer Electronics

MARKET SNAPSHOT
Apple is trading at $227.48, down 0.29% on the session...
[structured analyst commentary continues]

📰 Latest News:

1. Apple's Services Revenue Hits Record... (link)
2. iPhone 16 Demand Exceeds Expectations... (link)
3. Apple Announces New AI Features... (link)

📎 Full PDF report attached & saved to Google Drive.
```

## Architecture

```
Telegram Trigger
    → Code: Parse & Classify Input (stock/crypto/index/commodity)
    → Switch: Route by Asset Type
    → HTTP: Yahoo Finance (1y daily prices)
    → Code: Calculate Metrics (price, YTD, YoY, daily change)
    → HTTP: Yahoo Finance Search (top 3 news articles)
    → Code: Merge Data (metrics + news)
    → Code: Build LLM Prompt (structured analyst prompt)
    → HTTP: Anthropic Claude API (AI summary)
    → Telegram: Send Text Summary
    → Code: Generate HTML Report
    → HTTP: html2pdf.app (HTML → PDF conversion)
        → Google Drive: Upload PDF
        → Telegram: Send PDF Document
```

**Total nodes:** 13
**External APIs:** Yahoo Finance (free, no key), Anthropic Claude, html2pdf.app, Google Drive

## Key Features

- **Multi-asset support** — Stocks (any ticker), 20+ cryptocurrencies, 12+ global indices, 15+ commodities
- **Dynamic currency handling** — reads currency from the API response and displays the correct symbol (€, $, £, ¥, etc.)
- **Official company names** — resolves user input to the full registered company name via Yahoo Finance metadata
- **AI-powered analysis** — Claude produces structured flash notes with Market Snapshot, News Impact, Risk Factors, and Outlook sections
- **Sector identification** — the AI identifies and displays the asset's sector and industry classification
- **News relevance filtering** — searches by company name (not ticker) to avoid irrelevant articles, with prompt-level filtering as a second layer
- **Professional PDF reports** — color-coded metrics, structured layout, clickable news links
- **Google Drive archival** — every report is automatically saved with a timestamped filename

## Supported Assets

| Type | Examples | Yahoo Finance Format |
|------|----------|---------------------|
| **Stocks** | AAPL, MSFT, TSLA, IDR.MC, SAN.MC | Direct ticker |
| **Crypto** | BTC, ETH, SOL, DOGE, ADA, LINK... | BTC-USD |
| **Indices** | S&P500, NASDAQ, DOW, IBEX, DAX, FTSE, NIKKEI... | ^GSPC |
| **Commodities** | GOLD, SILVER, OIL, BRENT, COPPER, WHEAT, COFFEE... | GC=F |

## Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Workflow Engine | n8n (cloud or self-hosted) | Orchestration & automation |
| Market Data | Yahoo Finance API | Prices, metadata, news (free, no key) |
| AI Summarization | Anthropic Claude API (Sonnet) | Research analysis generation |
| PDF Generation | html2pdf.app | HTML → PDF conversion |
| File Storage | Google Drive API | PDF archival |
| Delivery | Telegram Bot API | User interface & report delivery |
| Language | JavaScript (n8n Code nodes) | Data processing & transformation |

## Setup

See [SETUP.md](./SETUP.md) for detailed installation and configuration instructions.

## Architecture Details

See [ARCHITECTURE.md](./ARCHITECTURE.md) for the complete technical blueprint, API documentation, and edge case handling.

## Costs

At moderate personal usage (~100-200 queries/month):

| Service | Monthly Cost |
|---------|-------------|
| Yahoo Finance | Free |
| Anthropic Claude (Sonnet) | ~€2-5 |
| html2pdf.app | Free (100/month) |
| Google Drive | Free (15 GB) |
| Telegram Bot | Free |
| n8n Cloud | Free tier or ~€20 |
| **Total** | **~€2-25/month** |

## Future Enhancements

- [ ] Earnings call transcript integration (for equities)
- [ ] Caching layer to reduce redundant API calls
- [ ] User rate limiting per Telegram chat ID
- [ ] Invalid ticker validation with suggestions
- [ ] Multi-language report support
- [ ] Scheduled daily/weekly reports for watchlist tickers

## Author

**Carlos** — Account Manager at Minsait (Indra) | MBA in International Management
Building toward Business Controller / Financial Analyst / Data Analyst roles.

This project is part of a broader technical portfolio demonstrating financial data engineering, API integration, and AI-powered automation capabilities.

## License

This project is for educational and portfolio purposes. The n8n workflow JSON is provided as-is.
