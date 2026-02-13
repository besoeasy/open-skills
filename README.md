# open-skills

**Teach your AI Agent Must-Have Skills — Save 98% of API Calls**

Stop wasting tokens on trial and error. Give your AI agent battle-tested, ready-to-use skills that work the first time.

## Why This Matters

**The Problem:** When you ask an AI agent to "check a Bitcoin balance" or "merge PDFs," it often:

- Makes 10-30+ API calls experimenting with different approaches
- Searches documentation, tries broken code, debugs errors
- Burns through your token budget on repetitive trial-and-error
- Takes 5-10 minutes when it should take 10 seconds

**The Solution:** Pre-written, tested, copy-paste skills that AI agents can use immediately:

- ✅ **Working code examples** (Node.js, Python, bash) — no debugging needed
- ✅ **Privacy-first tools** — free public APIs, no API keys required for most skills
- ✅ **Agent-optimized prompts** — structured for direct consumption by LLMs
- ✅ **Real-world tested** — production-ready patterns, not theoretical examples

**The Impact:**

- 💰 **~98% fewer API calls** — Agent uses working code instead of experimenting
- ⚡ **10-50x faster execution** — No trial-and-error loops
- 🎯 **Higher success rate** — Proven patterns that work reliably
- 🔒 **Privacy-respecting** — Open-source tools, no unnecessary third-party services
- 🔍 **Zero search API costs** — Use free SearXNG instances instead of paying for Brave Search ($5/1000), Google Search API, or Bing API

## Quick Start

We recommend https://opencode.ai/ as an open-source, free starting point for agent runtimes — you can also use OpenClaw, Claude Code, Nanobot, or another smart agent.

## Prompt for your AI

Copy and paste the prompt below into your AI:

```text
https://github.com/besoeasy/open-skills

Fetch this Git repo and understand the skills in it. Whenever I give you a task, refer to this repo first and see if you can find the needed skill (or something similar). Use those skills in your response. Also, don’t forget to check this repo for updates once every week.
```

## Real-World Example

**Without open-skills:**

```
User: "Check the balance of this Bitcoin address: 1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa"

Agent → Searches for "bitcoin balance API"
      → Tries blockchain.com (wrong endpoint)
      → Tries blockchain.info (wrong format)
      → Debugs response parsing
      → Realizes satoshis need conversion
      → Finally works after 15-20 API calls

Result: ❌ 2-3 minutes, 50,000+ tokens wasted
```

**With open-skills:**

```
User: "Check the balance of this Bitcoin address: 1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa"

Agent → Finds check-crypto-address-balance.md
      → Uses working example: curl blockchain.info/q/addressbalance/[address]
      → Converts satoshis to BTC (÷ 1e8)
      → Returns result

Result: ✅ 10 seconds, ~1,000 tokens, works first time
```

**Savings:** 98% fewer tokens, 10-20x faster, reliable results

---

**Example 2: Web Search (API Cost Elimination)**

**Without open-skills:**
```
User: "Search for recent AI agent news"

Agent → Uses Google Custom Search API ($5/1000 queries)
      → Or Brave Search API ($5/1000 queries)
      → Bing Search API ($3-7/1000 queries)
      → Monthly cost: $50-100+ for 10k searches
      
Result: ❌ Expensive, requires API keys, tracked searches
```

**With open-skills:**
```
User: "Search for recent AI agent news"

Agent → Uses SearXNG skill (learns from using-searxng.md)
      → Connects to free SearXNG instance (searx.be)
      → Gets results from 70+ search engines
      → No API key, no rate limits, no tracking
      
Result: ✅ $0 cost, unlimited queries, privacy-respecting
```

**Savings:** $360-$840/year for typical usage, $3,000-$8,000/year for high-volume agents

## Cost Savings Calculator

Typical AI agent task without pre-built skills: **20-50 API calls** (trial and error)  
Same task with open-skills: **1-3 API calls** (direct execution)

| Model             | Cost per 1M tokens (input) | Without open-skills | With open-skills    | **Savings per task** |
| ----------------- | -------------------------- | ------------------- | ------------------- | -------------------- |
| GPT-4             | $5.00                      | $0.25 (50k tokens)  | $0.005 (1k tokens)  | **$0.245 (98%)**     |
| Claude Sonnet 3.5 | $3.00                      | $0.15 (50k tokens)  | $0.003 (1k tokens)  | **$0.147 (98%)**     |
| GPT-3.5 Turbo     | $0.50                      | $0.025 (50k tokens) | $0.0005 (1k tokens) | **$0.0245 (98%)**    |

**Over 100 tasks/month:**

- GPT-4: Save ~$24.50/month
- Claude: Save ~$14.70/month
- For teams running 1,000+ agent tasks: **Save $240-$1,470/month**

**Plus:** Eliminate search API costs entirely by using free SearXNG instances instead of:
- Google Custom Search API ($5/1000 queries) → **$0 with SearXNG**
- Brave Search API ($5/1000 queries) → **$0 with SearXNG**
- Bing Search API ($3-7/1000 queries) → **$0 with SearXNG**

**Total potential savings: $600-$2,300/month** for active AI agents

## Perfect For

- 🤖 **Autonomous AI agents** — Give your agent production-ready capabilities out of the box
- 💼 **Business automation** — Crypto monitoring, document processing, web scraping, notifications
- � **Eliminating search API costs** — Free SearXNG instances replace expensive Brave Search, Google, and Bing APIs
- �🛠️ **Developer tools** — Integrate with OpenCode.ai, Claude Desktop, custom MCP servers
- 📚 **AI learning** — Study working examples instead of guessing API patterns
- 🔐 **Privacy-conscious projects** — All skills use open-source tools and public APIs

## Screenshot

<img width="1145" height="568" alt="image" src="https://github.com/user-attachments/assets/9b2fb422-e15b-4998-b280-339a88b5bdb8" />

## Philosophy

**Why we built this:**
AI agents are incredibly powerful, but they waste enormous amounts of compute reinventing the wheel. Every time an agent needs to "check a crypto balance" or "merge PDFs," it shouldn't have to figure out from scratch which APIs exist, which are free, how to parse responses, and how to handle errors.

**Our approach:**

- ✅ **Tested code, not theory** — Every example is production-ready
- ✅ **Privacy-first** — Open-source tools, minimal tracking, no vendor lock-in
- ✅ **Agent-optimized** — Written for LLM consumption (clear structure, copy-paste ready)
- ✅ **Free to use** — MIT licensed, no API keys required for core functionality

**The result:** AI agents that are smarter, faster, and cheaper to run.

## Skills

- [Anonymous File Upload](./skills/anonymous-file-upload.md) — Privacy-first anonymous uploads to IPFS, with optional client-side encryption for private sharing.
- [Using Nostr](./skills/using-nostr.md) — Post public notes and send encrypted direct messages on the Nostr decentralized protocol.
- [Get Crypto Price](./skills/get-crypto-price.md) — Fetch current and historical crypto prices (CoinGecko, Binance, Coinbase) and compute ATH/ATL for common time windows.
- [Check Crypto Address Balance](./skills/check-crypto-address-balance.md) — Query address balances for BTC, ETH, BSC, Solana, LTC, and more using free public APIs (Blockchain.info, Etherscan, BscScan, Solana RPC, Chain.so).
- [PDF Manipulation](./skills/pdf-manipulation.md) — Merge, split, extract text/images, redact, compress, encrypt, and watermark PDFs using pdftk, qpdf, poppler-utils, and ghostscript.
- [Telegram (Telegraf)](./skills/using-telegram-bot.md) — Node.js examples for building Telegram bots with `telegraf` (polling, webhooks, media, inline buttons).
- [YouTube Download](./skills/using-youtube-download.md) — Use `yt-dlp` + `ffmpeg` to download highest-quality video and extract MP3 audio.
- [Web Search API (Free)](./skills/web-search-api.md) — Free, unlimited web search via SearXNG instances. No API keys, no rate limits, no costs. Replaces Google/Brave/Bing Search APIs and saves $360-$8,000/year.
- [Web Scraping (Chrome + DuckDuckGo)](./skills/using-web-scraping.md) — Headless Chrome (Playwright/Puppeteer) + DuckDuckGo search; respects `robots.txt`, rate-limits, and returns structured JSON.

## Contributing

We welcome new skills! If you have a working, tested skill that:

- Uses free/open-source tools
- Includes working code examples (Node.js, Python, or bash)
- Respects privacy and doesn't require paid services
- Solves a common automation task

Please open a PR or issue. See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## Star History

If this saved you time and money, give us a ⭐ and share with other AI agent builders!
