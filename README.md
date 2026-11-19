# Find Winning Products Fast 🔥

> **Created by [Full Codes](https://github.com/full-codes)** — https://github.com/full-codes

A local dashboard that scrapes **Etsy, AliExpress and Amazon** search results,
ranks products with a **Winning Score (0–100)** and shows the winners in a web UI.

## Quick start (no installation needed)

1. Double-click **`run.bat`**
2. Your browser opens at **http://127.0.0.1:8765**
3. Type a niche (e.g. *dog toys*, *macrame wall hanging*) and click **Find Winners**

Everything is self-contained — the Python runtime lives in `runtime\`,
so you can copy the whole folder to a USB stick and run it on any Windows x64 PC.

## What the Winning Score means

| Component   | Weight | What it measures |
|-------------|--------|------------------|
| Popularity  | 38%    | Review / order volume (log-scaled) |
| Quality     | 30%    | Average rating (3.5★ = 0, 5★ = 100) |
| Price zone  | 18%    | Gaussian sweet spot around ~$27 |
| Revenue     | 14%    | Estimated lifetime gross revenue (reviews × price) |

Scores ≥ 70 are marked **hot** (green), ≥ 40 **warm** (amber), lower ones grey.
"Est. revenue" uses a rough heuristic (1 review ≈ 8 sales × price) — treat it as a
directional signal, not an exact figure.

## Using the dashboard

- **Marketplace chips** — enable/disable Etsy, AliExpress, Amazon per search
- **Filters** — min/max price, minimum reviews, sorting
- **Status chips** — each marketplace reports `ok` / `cached` / `blocked` / `error`
  (hover for the reason — big sites sometimes serve bot-check pages)
- **Demo mode** — clearly-labelled sample data for testing the UI with zero scraping
- **Export CSV** — downloads the current result set
- Results are cached in `data\cache.json` for 30 minutes to avoid re-scraping

## Project layout

```
runtime\          portable Python 3.12 (embeddable, official python.org build)
app\
  main.py         FastAPI server + API endpoints
  scoring.py      winning-score engine
  cache.py        30-min file cache
  demo.py         sample data for Demo Mode
  scrapers\
    base.py       shared HTTP/parse helpers
    etsy.py       Etsy scraper (JSON-LD based)
    amazon.py     Amazon scraper (search-result DOM)
    aliexpress.py AliExpress scraper (embedded JSON blob)
web\
  index.html      dashboard UI
  app.js / style.css
run.bat           double-click launcher
data\cache.json   search cache (auto-created)
```

## API (for scripts / Excel / Power Query)

```
GET /api/search?q=dog+toys&markets=etsy,amazon&sort=score&min_price=10&max_price=60
GET /api/export?q=dog+toys&markets=all            (CSV download)
```

## Honest limitations

- **Etsy** is protected by DataDome bot protection, which reliably blocks direct scraping
  (even browser automation). The dashboard therefore uses the **official Etsy Open API v3**:
  1. Click **⚙ Settings** in the dashboard
  2. Register a free app at [etsy.com/developers/register](https://www.etsy.com/developers/register) (~2 min)
  3. Copy the **Keystring** and paste it into Settings → Save

  Until a key is set, Etsy shows a `🔑 needs key` chip and the other marketplaces keep working.
  Note: the Etsy API provides favorites (hearts) instead of reviews — the winning score uses
  favorites for the popularity component.
- **Amazon / AliExpress** scrape OK most of the time from a home connection, but both
  serve bot-check pages under load. When a chip turns orange/red, that marketplace
  was blocked for that request — other marketplaces keep working. The 30-minute
  cache keeps you under the radar on repeat searches.
- Sales/revenue numbers are heuristic estimates, not real marketplace data.
