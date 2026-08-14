# PSX Hub — merged PSX research, portfolio & screener app

This is a single Flask web app that merges **PSX Toolkit** and **PSX 360**
into one site. It keeps PSX 360's full front-end (it was the more complete
of the two) and adds the two capabilities PSX Toolkit had that PSX 360
didn't: **Crypto Technicals** and **Forex Technicals** — RSI-divergence /
trend-structure screeners on live intraday data — plus **portfolio CSV
import/export**. Overlapping features (PSX screener, fundamentals, mutual
funds, crypto/forex prices, portfolio tracking) were kept in their PSX 360
form since that implementation was more complete and already had a
matching database and UI.

## What's inside

**Overview:** Dashboard, Portfolio (holdings, day-by-day P/L, CSV
import/export), Watchlist, Transactions, Stock Screener.

**Markets:** Market overview, full PSX stock directory, Mutual Funds
(MUFAP), Crypto prices, Forex rates, Commodities, Fear & Greed sentiment,
Sector rotation, **Forex Technicals** (new), **Crypto Technicals** (new).

**Research:** Pakistan macro indicators, News/announcements, Journal,
Tools (calculators), World Clock, Stock comparison.

**Under the hood:** Background bulk quote refresh so the screener doesn't
wait on PSX for every click, cached technical indicators, a service
worker for repeat-visit speed, and a `/healthz` endpoint for uptime
monitors.

### New: Forex & Crypto Technicals

Ported over from PSX Toolkit. Scans major forex pairs (plus gold and
silver) and the top ~15 cryptocurrencies by market cap for:
- RSI(14)
- Bullish / bearish RSI divergence
- Trend-structure classification (higher highs/lows, lower highs/lows, etc.)

on 30-minute and 1-hour bars (forex) or 30m/1h/4h bars (crypto), using
free data from Yahoo Finance (`yfinance`, no API key needed). A scan
takes a minute or two, so it runs in the background — the page polls for
progress, and the last completed scan is cached so reopening the tab
shows results instantly.

This is a technical screen, not investment advice.

### New: Portfolio CSV import/export

On the Portfolio page, **Export CSV** downloads your current holdings
(symbol, quantity, avg_cost, acquired_date). **Import CSV** accepts a
file in the same format — matching symbols are updated, new ones are
added, and any malformed rows are reported back rather than silently
skipped.

## Run it locally

```bash
pip install -r requirements.txt
python app.py
```

Then open `http://127.0.0.1:5000` in your browser. To reach it from your
phone on the same WiFi, use your computer's local IP instead of
`127.0.0.1` (e.g. `http://192.168.1.23:5000`) — check your OS's network
settings for that IP, or run `ipconfig` (Windows) / `ifconfig` (Mac) in a
terminal. Add it to your phone's home screen ("Add to Home Screen" in
Safari, or the ⋮ menu → "Install app" in Chrome) for an app-like icon.

## Deploy for free (Render)

Render's free tier can run this Flask app at no cost, from any device,
without keeping your own computer on.

1. Put this folder in a GitHub repository (GitHub Desktop or
   `git init && git add . && git commit -m "init" && git push` all work).
2. In Render, choose **New → Web Service** and connect that repository.
3. Render will pick up the included `render.yaml` automatically —
   otherwise set:
   - **Build command:** `pip install -r requirements.txt`
   - **Start command:** `gunicorn app:app --workers 1 --threads 8 --timeout 120 --keep-alive 5`
   - **Health check path:** `/healthz`
4. Deploy. Render gives you a public `https://your-app.onrender.com` URL
   you can open from any device — phone, tablet, laptop.

Full official docs: https://render.com/docs/deploy-flask

**Free-tier notes:**
- Free services spin down after 15 minutes of inactivity and take ~30–60
  seconds to wake back up on the next visit — normal for a free host.
- The filesystem is ephemeral, so the SQLite portfolio database
  (`portfolio.db`) can be reset on redeploy/restart. For data that must
  never disappear, point the app at an external database — this is
  beyond what's included here, but Render's docs cover free Postgres
  add-ons.
- Use **Export CSV** on the Portfolio page from time to time as a manual
  backup regardless of hosting.

## Optional live market-data key

The app already uses public/fallback sources for everything (PSX's own
site, CoinGecko, Frankfurter, MUFAP, Yahoo Finance). For live world
indices/commodities beyond those free sources, you can optionally set:

- `MARKET_DATA_API_KEY`
- `MARKET_DATA_PROVIDER` (defaults to `twelvedata`)

as environment variables. Never put API keys directly in source code —
set them as environment variables in Render's dashboard (or a local
`.env` you don't commit) instead.

## What was merged from where

| Feature | Source | Notes |
|---|---|---|
| Dashboard, Portfolio, Watchlist, Transactions | PSX 360 | Kept as-is — SQLite-backed, includes P/L history |
| Stock Screener, Fundamentals | PSX 360 | Kept as-is — has its own in-house technical-indicator engine with real progress tracking |
| Mutual Funds, Crypto, Forex, Commodities, Sentiment, Macro, News, Journal, Tools, World Clock | PSX 360 | Kept as-is |
| **Forex Technicals, Crypto Technicals** | **PSX Toolkit** | Newly wired into PSX 360's UI/routing; RSI-divergence math reused from `psx_screener.py` |
| **Portfolio CSV import/export** | **PSX Toolkit** (concept) | Rebuilt against PSX 360's SQLite schema |

PSX Toolkit's own PSX-stock screener, portfolio system and fundamentals
lookup were **not** duplicated — PSX 360 already covers that ground with
a more capable, database-backed implementation.

## Notes

- PSX / MUFAP / Yahoo Finance data availability can change independently
  of this app. Some fields are deliberately shown as unavailable rather
  than guessed.
- This is a personal research tool, not investment advice or a brokerage
  connection — nothing here places trades.
- Your data (portfolio, watchlist, transactions) lives in this app's own
  SQLite database; nothing is sent to a third party except requests to
  public data sources for prices.
