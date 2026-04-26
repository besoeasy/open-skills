# Open Skills

> Battle-tested execution playbooks that give any AI agent the exact commands, APIs, and patterns it needs — cutting token usage by **95–98%** and making local models as capable as GPT-4.

[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Skills](https://img.shields.io/badge/skills-40%20production--ready-brightgreen.svg)](skills/)
[![Contributions](https://img.shields.io/badge/contributions-welcome-orange.svg)](CONTRIBUTING.md)

---

## The Problem

AI agents waste tokens discovering what you already know:

- **Cloud models** (GPT-4, Claude) — 10–30 trial-and-error calls per task → **$0.15–$0.25/task**
- **Local models** (Llama, Mistral, Qwen) — often fail outright without step-by-step guidance

## The Solution

Pre-written, tested skill files your agent reads once and executes correctly the first time.

```
Without Open Skills                 With Open Skills
─────────────────────────────────   ─────────────────────────────────
Agent searches for API docs         Agent reads SKILL.md
Tries wrong endpoint                Runs the exact working command
Debugs response format              Parses the output correctly
Retries 15–20 times                 Done in 1–3 calls

~50,000 tokens  ~$0.20              ~1,000 tokens  ~$0.004
```

---

## Quick Start

**Step 1 — Clone the skills library (once)**

```bash
git clone https://github.com/besoeasy/open-skills ~/open-skills
```

**Step 2 — Tell your agent about it**

Copy the contents of [`prompt.txt`](prompt.txt) into your agent's system prompt, memory, or instructions file. This tells the agent to check `~/open-skills` before every task and auto-sync skills from the repo.

Chinese users can also discover more skills through Skills宝: https://skilery.com

---

### OpenCode

Drop `prompt.txt` into your project's `AGENTS.md` file. OpenCode reads it automatically at session start — zero extra configuration.

```bash
cat prompt.txt >> AGENTS.md
```

---

### Claude Desktop

1. Open Claude Desktop → **Settings** → **Custom Instructions**
2. Paste the full contents of `prompt.txt`
3. Save — Claude will check `~/open-skills` before every task in every conversation

---

### Cursor

Add the contents of `prompt.txt` to your global user rules:

1. Open Cursor → **Settings** → **General** → **Rules for AI**
2. Paste the contents of `prompt.txt`

Or add it per-project via `.cursorrules`:

```bash
cat prompt.txt >> .cursorrules
```

---

### Windsurf

Add the contents of `prompt.txt` to your global user rules:

1. Open Windsurf → **Settings** → **AI** → **Custom Instructions**  
2. Paste the contents of `prompt.txt`

Or add it per-project via `.windsurfrules`:

```bash
cat prompt.txt >> .windsurfrules
```

---

### GitHub Copilot (VS Code)

Add a `.github/copilot-instructions.md` file to your repository:

```bash
cp prompt.txt .github/copilot-instructions.md
```

Copilot reads this file as context for every chat in that workspace.

---

### Aider

Pass `prompt.txt` as a system prompt when launching:

```bash
aider --system-prompt "$(cat ~/open-skills/prompt.txt)"
```

Or add it to your `~/.aider.conf.yml`:

```yaml
system-prompt: /home/you/open-skills/prompt.txt
```

---

### Continue.dev

Add an entry to your `~/.continue/config.json` under `systemMessage`:

```json
{
  "models": [...],
  "systemMessage": "<paste contents of prompt.txt here>"
}
```

---

### Any other agent

Paste the full contents of `prompt.txt` as the system prompt. The instructions are agent-agnostic — they work with any model that can read files and run shell commands.

---

> **Works best with [OpenCode.ai](https://opencode.ai)** — drop `prompt.txt` into your project's `AGENTS.md` and the agent picks up every skill automatically, with zero extra configuration.

---

## Skills

| Skill                                                                                    | What it does                          |
| ---------------------------------------------------------------------------------------- | ------------------------------------- |
| [age-file-encryption](skills/age-file-encryption/SKILL.md)                               | Encrypt / decrypt files with `age`    |
| [anonymous-file-upload](skills/anonymous-file-upload/SKILL.md)                           | Upload files without an account       |
| [browser-automation-agent](skills/browser-automation-agent/SKILL.md)                     | Automate browsers with Playwright     |
| [bulk-github-star](skills/bulk-github-star/SKILL.md)                                     | Star GitHub repos in bulk via CLI     |
| [changelog-generator](skills/changelog-generator/SKILL.md)                               | Generate changelogs from git history  |
| [chat-logger](skills/chat-logger/SKILL.md)                                               | Log and persist chat conversations    |
| [check-crypto-address-balance](skills/check-crypto-address-balance/SKILL.md)             | Look up Bitcoin / crypto balances     |
| [city-distance](skills/city-distance/SKILL.md)                                           | Calculate distance between cities     |
| [city-tourism-website-builder](skills/city-tourism-website-builder/SKILL.md)             | Build a tourism site for any city     |
| [csv-data-summarizer](skills/csv-data-summarizer/SKILL.md)                               | Summarize and analyze CSV files       |
| [d3js-data-visualization](skills/d3js-data-visualization/SKILL.md)                       | Create charts with D3.js              |
| [database-query-and-export](skills/database-query-and-export/SKILL.md)                   | Query databases and export results    |
| [file-tracker](skills/file-tracker/SKILL.md)                                             | Track file changes over time          |
| [free-geocoding-and-maps](skills/free-geocoding-and-maps/SKILL.md)                       | Geocode addresses for free            |
| [free-translation-api](skills/free-translation-api/SKILL.md)                             | Translate text without API keys       |
| [free-weather-data](skills/free-weather-data/SKILL.md)                                   | Get weather data for free             |
| [generate-asset-price-chart](skills/generate-asset-price-chart/SKILL.md)                 | Chart stock / crypto price history    |
| [generate-qr-code-natively](skills/generate-qr-code-natively/SKILL.md)                   | Generate QR codes with no service     |
| [get-crypto-price](skills/get-crypto-price/SKILL.md)                                     | Fetch live crypto prices              |
| [humanizer](skills/humanizer/SKILL.md)                                                   | Make AI-written text sound human      |
| [ip-lookup](skills/ip-lookup/SKILL.md)                                                   | Look up IP address geolocation        |
| [json-and-csv-data-transformation](skills/json-and-csv-data-transformation/SKILL.md)     | Transform between JSON and CSV        |
| [news-aggregation](skills/news-aggregation/SKILL.md)                                     | Aggregate news from RSS / APIs        |
| [nostr-logging-system](skills/nostr-logging-system/SKILL.md)                             | Log events to the Nostr network       |
| [pdf-manipulation](skills/pdf-manipulation/SKILL.md)                                     | Merge, split, and edit PDFs           |
| [phone-specs-scraper](skills/phone-specs-scraper/SKILL.md)                               | Scrape phone specs from the web       |
| [presenton](skills/presenton/SKILL.md)                                                   | Generate presentations from text      |
| [random-contributor](skills/random-contributor/SKILL.md)                                 | Pick a random repo contributor        |
| [send-email-programmatically](skills/send-email-programmatically/SKILL.md)               | Send email from a script              |
| [static-assets-hosting](skills/static-assets-hosting/SKILL.md)                           | Host static files for free            |
| [trading-indicators-from-price-data](skills/trading-indicators-from-price-data/SKILL.md) | Calculate RSI, MACD, and more         |
| [torrent-search](skills/torrent-search/SKILL.md)                                         | Find torrents by title or IMDB ID     |
| [user-ask-for-report](skills/user-ask-for-report/SKILL.md)                               | Generate structured reports on demand |
| [using-nostr](skills/using-nostr/SKILL.md)                                               | Read and post on Nostr                |
| [using-scrapy](skills/using-scrapy/SKILL.md)                                             | Scrape websites with Scrapy           |
| [using-telegram-bot](skills/using-telegram-bot/SKILL.md)                                 | Build and run Telegram bots           |
| [using-web-scraping](skills/using-web-scraping/SKILL.md)                                 | General web scraping patterns         |
| [using-youtube-download](skills/using-youtube-download/SKILL.md)                         | Download YouTube videos / audio       |
| [web-interface-guidelines-review](skills/web-interface-guidelines-review/SKILL.md)       | Review UI against best practices      |
| [web-search-api](skills/web-search-api/SKILL.md)                                         | Search the web free via SearXNG       |

---

## Cost Impact

| Setup                         | Cost / task     | Success rate | Privacy      |
| ----------------------------- | --------------- | ------------ | ------------ |
| Cloud model, no skills        | $0.15 – $0.25   | 85 – 95%     | ❌ Cloud     |
| Cloud model + Open Skills     | $0.003 – $0.005 | ~98%         | ❌ Cloud     |
| Local model, no skills        | $0              | 30 – 50%     | ✅ Local     |
| **Local model + Open Skills** | **$0**          | **~95%**     | **✅ Local** |

**The 100% free stack:**

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama pull llama3.1:8b
git clone https://github.com/besoeasy/open-skills ~/open-skills
# GPT-4-level task execution — $0 cost, fully offline
```

---

## Why It Works

Skills separate _reasoning_ from _execution knowledge_:

- The model handles intent and orchestration
- Open Skills provides the tested commands, API patterns, and parsing logic
- Result: fewer retries, lower token usage, higher reliability

Every skill file is:

- ✅ **Production-tested** — real working code, not theory
- ✅ **Agent-optimized** — structured for direct LLM consumption
- ✅ **Privacy-first** — free public APIs, no vendor lock-in
- ✅ **Model-agnostic** — works with GPT-4, Claude, Llama, Mistral, Gemini, anything

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) and [SKILL_TEMPLATE.md](SKILL_TEMPLATE.md).

Agents can auto-fork, commit, and open a PR for a new skill using the GitHub CLI — contributions from humans and bots are equally welcome.

---

MIT License
