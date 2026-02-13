---
name: get-crypto-price
description: Fetch current and historical crypto prices and compute ATH or ATL over common time windows.
---

# Get Crypto Price (minimal guide)

This short guide shows how to fetch current prices and at least 3 months of past price action using CoinGecko, Binance, and Coinbase public APIs. It also shows how to compute ATH (highest) and ATL (lowest) within time windows: 1 DAY, 1 WEEK, 1 MONTH.

---

## Quick notes
- Timestamps: many APIs return milliseconds since epoch (ms) or seconds (s). Convert consistently.
- Rate limits: respect exchange rate limits; cache responses when possible.
- Symbols: use canonical pair symbols (e.g., `BTCUSDT` on Binance, `bitcoin` on CoinGecko).

---

## 1) CoinGecko (recommended for simple historical ranges)

- Current price (curl):

```bash
curl "https://api.coingecko.com/api/v3/simple/price?ids=bitcoin&vs_currencies=usd"
```

- Last 90 days (price history):

```bash
curl "https://api.coingecko.com/api/v3/coins/bitcoin/market_chart?vs_currency=usd&days=90"
```

Response contains `prices` array: [[timestamp_ms, price], ...].

Python example: fetch 90 days and compute ATH/ATL for 1d/7d/30d windows.

```python
import requests
import time
from datetime import datetime, timedelta

def fetch_coingecko_prices(coin_id='bitcoin', vs='usd', days=90):
    url = f'https://api.coingecko.com/api/v3/coins/{coin_id}/market_chart'
    r = requests.get(url, params={'vs_currency': vs, 'days': days}, timeout=15)
    r.raise_for_status()
    return r.json()['prices']  # list of [ts_ms, price]

def max_min_in_window(prices, since_ms):
    window = [p for (ts, p) in prices if ts >= since_ms]
    if not window:
        return None, None
    return max(window), min(window)

prices = fetch_coingecko_prices('bitcoin', 'usd', 90)
now_ms = int(time.time() * 1000)

windows = {
    '1d': now_ms - 24*3600*1000,
    '1w': now_ms - 7*24*3600*1000,
    '1m': now_ms - 30*24*3600*1000,
}

for name, since in windows.items():
    ath, atl = max_min_in_window(prices, since)
    print(name, 'ATH:', ath, 'ATL:', atl)
```

Notes: CoinGecko returns sampled points (usually hourly) — good for these windows.

---

## 2) Binance (exchange-level data)

- Current price (curl):

```bash
curl "https://api.binance.com/api/v3/ticker/price?symbol=BTCUSDT"
```

- Historical klines (candles): use `klines` endpoint. Example: fetch daily candles for the last 1000 days or hourly for finer resolution.

```bash
# daily candles for BTCUSDT (limit up to 1000 rows)
curl "https://api.binance.com/api/v3/klines?symbol=BTCUSDT&interval=1d&limit=1000"
```

Each kline row: [openTime, open, high, low, close, ...] where openTime is ms.

Python example: fetch hourly klines for last 90 days and compute ATH/ATL windows.

```python
import requests
import time
from datetime import datetime, timedelta

def fetch_binance_klines(symbol='BTCUSDT', interval='1h', limit=1000):
    url = 'https://api.binance.com/api/v3/klines'
    r = requests.get(url, params={'symbol': symbol, 'interval': interval, 'limit': limit}, timeout=15)
    r.raise_for_status()
    return r.json()  # list of lists

# To cover ~90 days hourly: 24*90 = 2160 rows -> call twice with different startTimes or use 4h interval
klines = fetch_binance_klines('BTCUSDT', '1h', 1000)
# For more than 1000 rows you'd loop with startTime using ms timestamps.

# Convert to list of (ts_ms, high, low)
data = [(row[0], float(row[2]), float(row[3])) for row in klines]
now_ms = int(time.time() * 1000)

def ath_atl_from_klines(data, since_ms):
    highs = [h for (ts,h,l) in data if ts >= since_ms]
    lows = [l for (ts,h,l) in data if ts >= since_ms]
    if not highs:
        return None, None
    return max(highs), min(lows)

windows = {
    '1d': now_ms - 24*3600*1000,
    '1w': now_ms - 7*24*3600*1000,
    '1m': now_ms - 30*24*3600*1000,
}

for name, since in windows.items():
    ath, atl = ath_atl_from_klines(data, since)
    print(name, 'ATH:', ath, 'ATL:', atl)
```

Notes: Binance `limit` is 1000 max per request; for full 90 days hourly, page by startTime.

---

## 3) Coinbase (public example)

- Current spot price (curl):

```bash
curl "https://api.coinbase.com/v2/prices/BTC-USD/spot"
```

- Historical candles (Coinbase Exchange API):

```bash
curl "https://api.exchange.coinbase.com/products/BTC-USD/candles?granularity=3600&start=2025-11-01T00:00:00Z&end=2026-02-01T00:00:00Z"
```

Response: array of [time, low, high, open, close, volume]. Use similar filtering by timestamp to compute ATH/ATL.

---

## 4) Compute ATH / ATL for a timeframe (1 DAY, 1 WEEK, 1 MONTH)

General steps (applies to any data source that gives timestamped prices or OHLC candles):

1. Fetch historical points that cover at least the desired window (e.g., last 90 days).
2. Choose the window start timestamp (now - window_seconds).
3. Filter the points where timestamp >= window_start.
4. If you have OHLC candles, use `high` as candidate for ATH and `low` as candidate for ATL. If you only have sampled prices, use max/min of sampled values.

Example in plain Python (assuming `points` is list of (ts_ms, price) or candles):

```python
# points = [(ts_ms, price), ...]
since_ms = int(time.time() * 1000) - 24*3600*1000  # 1 day
window_prices = [p for (ts, p) in points if ts >= since_ms]
if window_prices:
    ath = max(window_prices)
    atl = min(window_prices)
else:
    ath = atl = None
```

If using OHLC candles:

```python
# candles = [(ts_ms, open, high, low, close), ...]
window = [c for c in candles if c[0] >= since_ms]
ath = max(c[2] for c in window)
atl = min(c[3] for c in window)
```

---

## 5) Practical tips
- For 3 months of past price action, fetch 90 days of data or page the exchange candle endpoints until you cover ~90 days.
- Use hourly or daily granularity depending on required resolution. For 1-day ATH/ATL, hourly or minute granularity is better.
- Convert times into UTC and use ms for consistency.
- Respect API rate limits and use caching for repeated queries.

---

## 6) Example workflow (summary)
1. Try CoinGecko `market_chart?days=90` for quick 90-day history.
2. Compute windows for 1d/7d/30d from that array and derive ATH/ATL.
3. For exchange-precise data or higher resolution, query Binance `klines` or Coinbase `candles` and repeat the same aggregation.

---

If you want, I can add ready-to-run scripts for specific coins (BTC, ETH) and automate paginated Binance fetches to guarantee 90 days of hourly data.
