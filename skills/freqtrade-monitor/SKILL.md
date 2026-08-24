---
name: freqtrade-monitor
description: "Monitor a running Freqtrade trading bot: process health, open trades, P&L, and live exchange balance. Use when: (1) User asks to check a freqtrade bot status, (2) Reporting active trades or balance, or (3) Diagnosing a bot that seems unresponsive."
---

# Freqtrade Monitor

Check a running Freqtrade bot's health, list its open trades and realized P&L from the SQLite trade database, and fetch the live exchange wallet balance — without restarting or interrupting the bot.

## Quick quality checklist

- `name` matches folder name exactly (kebab-case)
- All examples are tested and runnable
- Includes both Bash and Node.js examples
- Uses free/public tools first (or explains paid fallback)
- No secrets, API keys, or personal data in examples

## When to use

- Use case 1: "Check my freqtrade bot / is it still trading?"
- Use case 2: "What are my open positions and balance?"
- Use case 3: The bot's REST API is down and you need trade state anyway

## Required tools / APIs

- `ps`/`pgrep` (system default)
- Python 3 with `sqlite3` (standard library) for DB fallback
- `ccxt` (`pip install ccxt`) only if a live wallet balance is needed
- Bot's REST API enabled in config (optional but preferred)

Install options:

```bash
# Ubuntu/Debian
sudo apt-get install -y python3

# ccxt, only if fetching live balances
pip install ccxt
```

## Skills

### basic_usage

Shortest reliable path: confirm process, then query REST API.

```bash
# 1. Is the bot running?
pgrep -af "freqtrade trade"

# 2. Query the REST API (read creds/port from the bot's config.json)
curl -fsS --max-time 10 -u "$FT_USER:$FT_PASS" \
  "http://127.0.0.1:8080/api/v1/status" | jq '.[] | {pair, open_rate, profit_ratio}'

curl -fsS --max-time 10 -u "$FT_USER:$FT_PASS" \
  "http://127.0.0.1:8080/api/v1/balance" | jq '.currencies'
```

**Node.js:**

```javascript
async function botStatus(port = 8080, user, pass) {
  const auth = Buffer.from(`${user}:${pass}`).toString("base64");
  const res = await fetch(`http://127.0.0.1:${port}/api/v1/status`, {
    headers: { Authorization: `Basic ${auth}` },
  });
  if (!res.ok) throw new Error(`API returned ${res.status}`);
  return res.json();
}

// Usage:
// botStatus(8080, 'user', 'pass').then(t => console.log(t));
```

### robust_usage

Fallback path when the API is down (port conflict, crashed uvicorn): read the SQLite database directly, then fetch wallet balances via ccxt using the bot's own exchange keys.

```bash
# Open trades + summary straight from the trade DB (read-only)
python3 - <<'EOF'
import sqlite3, glob
db = sorted(glob.glob("**/tradesv3.*.sqlite", recursive=True))[0]
con = sqlite3.connect(f"file:{db}?mode=ro", uri=True)
for row in con.execute(
    "SELECT pair, stake_amount, amount, open_rate, open_date "
    "FROM trades WHERE is_open=1"
):
    print(row)
print(con.execute(
    "SELECT COUNT(*), ROUND(SUM(close_profit_abs),4) FROM trades WHERE is_open=0"
).fetchone())
EOF
```

```bash
# Live spot balance via ccxt (read-only private call; never print keys)
python3 - <<'EOF'
import json, ccxt
cfg = json.load(open("config.json"))["exchange"]
api = ccxt.binance({"apiKey": cfg["key"], "secret": cfg["secret"],
                    "enableRateLimit": True})
bal = api.fetch_balance()
print("USDT total:", bal.get("USDT", {}).get("total"))
EOF
```

**Node.js:**

```javascript
const ccxt = require("ccxt");

async function walletBalance(exchangeId, apiKey, secret) {
  const ex = new ccxt[exchangeId]({ apiKey, secret, enableRateLimit: true });
  const bal = await ex.fetchBalance();
  return Object.fromEntries(
    Object.entries(bal.total).filter(([, v]) => v > 0)
  );
}

// Usage:
// walletBalance('binance', key, secret).then(console.log);
```

## Output format

- `bot_status`: `running` or `stopped` (+ PID)
- `open_trades[]`: pair, entry rate, current rate, stake amount, unrealized P&L %
- `realized_pnl`: sum of `close_profit_abs` over closed trades (number)
- `balance`: free/used/total per currency, plus total equity estimate
- Error shape: `{ "error": "reason", "fix": "actionable next step" }`

## Rate limits / Best practices

- Read the SQLite DB with `mode=ro` URI so you never lock the live writer.
- Never restart the bot just to fix the REST API while it holds open positions.
- Cache ticker/balance results ~30s; Binance allows ~1200 weight/min.
- If two `freqtrade trade` processes appear, flag it: one will fail to bind the API port and duplicate-instance races can cause bad behavior.

## Agent prompt

```text
You have freqtrade-monitor capability. When a user asks to check their trading bot:

1. Verify the process exists with pgrep -af "freqtrade trade".
2. Try the REST API (/status, /balance) with credentials from config.json.
3. If the API is unreachable, fall back to reading tradesv3*.sqlite read-only.
4. For live balances, use ccxt fetch_balance() with the bot's exchange keys; never echo secrets.
5. Report bot status, open positions with P&L, realized P&L, and total equity.

Always prefer read-only inspection over restarting anything.
```

## Troubleshooting

**REST API connection refused**
- Symptom: `curl` fails on 127.0.0.1:<port> even though the bot runs.
- Solution: grep the log for "address already in use" — another instance raced for the port at startup. Use the DB fallback; restart later during a quiet period.

**Two bot processes visible**
- Symptom: pgrep shows multiple `freqtrade trade` PIDs.
- Solution: identify the newer one (likely crash-looping or blocked on the DB); stop the stray one before it double-trades.

**Trade row has amount 0.0 while open**
- Symptom: an "open" trade owns nothing on-exchange.
- Solution: check logs/orders — the entry order likely never filled or was cancelled; reconcile against `fetch_balance()`.

## See also

- [../get-crypto-price/SKILL.md](../get-crypto-price/SKILL.md) — price lookups for marking open positions to market
- [../database-query-and-export/SKILL.md](../database-query-and-export/SKILL.md) — general SQL querying/export patterns

---

## Notes

- Skill file path should be `skills/freqtrade-monitor/SKILL.md`
- Quote `description` when it includes `:` to avoid YAML parsing issues
- Keep examples copy-paste friendly and verify they run before submitting
- See [CONTRIBUTING.md](CONTRIBUTING.md) for full contribution standards
