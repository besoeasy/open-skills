<div align="center">

# Open Skills

### Make local models as powerful as cloud models, or make cloud models 98% cheaper. 100% free & self-hostable.

[**MAIN INSTALLATION: USE THE WEBSITE QUICK START**](https://openskills.besoeasy.com/)

[![Use Open Skills Now](https://img.shields.io/badge/USE%20NOW-openskills.besoeasy.com-2ea44f?style=for-the-badge)](https://openskills.besoeasy.com/)

[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Skills](https://img.shields.io/badge/skills-production--ready-brightgreen.svg)](skills/)
[![Contributions](https://img.shields.io/badge/contributions-agent--friendly-orange.svg)](CONTRIBUTING.md)
[![Telegram](https://img.shields.io/badge/community-Telegram-26A5E4.svg)](https://t.me/+FC8ppvnUsj8xM2Vl)

Battle-tested, copy-paste skills for AI agents.  
**Two ways to win:**  
🏠 **Go 100% free:** Llama/Mistral + Open Skills = Cloud-model performance at $0 cost  
💰 **Save 98% on cloud:** GPT-4/Claude + Open Skills = $0.005/task instead of $0.25

</div>

---

## Quick Links

- [Browse Skills](skills/)
- [Contributing Guide](CONTRIBUTING.md)
- [Skill Template](SKILL_TEMPLATE.md)

## Why This Matters

**The Problem:** AI agents are expensive and cloud-dependent:

- **Cloud models (GPT-4, Claude):** Make 10-30+ API calls experimenting with each task → $0.25+ per simple task
- **Local models (Llama, Mistral):** Struggle to figure out APIs and tools without guidance → often fail or give up
- Both burn through tokens on trial-and-error, searching documentation, and debugging

**The Solution:** Pre-written, tested skills that work with ANY AI model:

- ✅ **Working code examples** (Node.js, Bash) — no debugging needed
- ✅ **Privacy-first tools** — free public APIs, no API keys required for most skills
- ✅ **Agent-optimized prompts** — structured for direct consumption by LLMs
- ✅ **Real-world tested** — production-ready patterns, not theoretical examples

**The Game-Changer:** 🚀 **Make local models as capable as cloud models**

Instead of expensive cloud models figuring things out from scratch, **give cheap local models the answers**:
- Llama 3.1 / Mistral / Qwen (free, local) + Open Skills → performs like GPT-4/Claude for practical tasks
- **Result: $0 cost, 100% self-hostable, complete privacy**

**The Impact:**

- 💰 **~98% cost reduction** — Local models with skills = $0 vs. GPT-4 at $0.25+ per task
- 🏠 **100% self-hostable** — Run Ollama + Open Skills entirely offline
- 🔒 **Complete privacy** — No data leaves your machine
- ⚡ **10-50x faster execution** — No trial-and-error loops
- 🎯 **Higher success rate** — Proven patterns that work reliably
- 🤖 **Automated contributions** — Agents can auto-fork, commit, and PR new skills via GitHub CLI
- 🧠 **Self-improving ecosystem** — Community skills flow back into the repository automatically
- 🏆 **Public credit** — Contributors get GitHub commit history and recognition
- 🔍 **Zero search API costs** — Use free SearXNG instances instead of paying for Brave Search ($5/1000), Google Search API, or Bing API


## Real-World Example

**Without open-skills (Cloud models like GPT-4/Claude):**

```
User: "Check the balance of this Bitcoin address: 1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa"

Cloud AI Agent → Searches for "bitcoin balance API"
                → Tries blockchain.com (wrong endpoint)
                → Tries blockchain.info (wrong format)
                → Debugs response parsing
                → Realizes satoshis need conversion
                → Finally works after 15-20 API calls

Result: ❌ 2-3 minutes, 50,000+ tokens, $0.15-$0.25 cost
```

**Without open-skills (Local models like Llama/Mistral):**

```
User: "Check the balance of this Bitcoin address: 1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa"

Local AI (Llama/Mistral) → Tries to search for API documentation
                         → Gets confused about endpoints
                         → Generates incorrect curl command
                         → Unable to parse response correctly
                         → Gives up or returns error

Result: ❌ Task fails, user frustrated
```

**With open-skills (ANY MODEL - GPT-4, Claude, Llama, Mistral, Gemini):**

```
User: "Check the balance of this Bitcoin address: 1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa"

Any AI Agent → Finds check-crypto-address-balance.md
             → Uses working example: curl blockchain.info/q/addressbalance/[address]
             → Converts satoshis to BTC (÷ 1e8)
             → Returns result

Result: ✅ 10 seconds, ~1,000 tokens, works first time
        ✅ Cloud models: $0.003-$0.005 (was $0.15-$0.25) — 95%+ savings
        ✅ Local models: $0.00 (free) — task actually succeeds
```

**Key insight:** Open Skills doesn't just make expensive models cheaper — **it makes cheap/free models actually work**.

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

Agent → Uses SearXNG skill (learns from [skills/web-search-api.md](skills/web-search-api.md))
      → Connects to free SearXNG instance (searx.be)
      → Gets results from 70+ search engines
      → No API key, no rate limits, no tracking

Result: ✅ $0 cost, unlimited queries, privacy-respecting
```

**Savings:** $360-$840/year for typical usage, $3,000-$8,000/year for high-volume agents

---

**Example 3: Trading Indicators (Quant Analysis in Seconds)**

**Without open-skills:**

```
User: "Calculate RSI, MACD, and top indicators from this OHLCV dataset"

Agent → Searches for indicator formulas one by one
      → Implements RSI, then debugs MACD math
      → Repeats for Bollinger, Stochastic, ATR, ADX, etc.
      → Fixes column mapping/warmup NaN issues
      → Ends up with inconsistent outputs after many iterations

Result: ❌ Slow, error-prone, heavy token/API usage
```

**With open-skills:**

```
User: "Calculate RSI, MACD, and top indicators from this OHLCV dataset"

Agent → Finds trading-indicators-from-price-data.md
      → Runs the ready Python workflow with pandas + pandas-ta
      → Computes 20 indicators (RSI, MACD, SMA/EMA, BB, Stoch, ATR, ADX, CCI, OBV, MFI, ROC)
      → Returns clean, structured output immediately

Result: ✅ Fast, consistent, production-ready calculations
```

**Savings:** Massive reduction in trial-and-error, faster indicator pipelines, and more reliable strategy signals

---

**Example 4: Hosted Report Website (Tailwind + Originless)**

**Without open-skills:**

```
User: "Create a beautiful white-themed report website from this content and host it instantly"

Agent → Experiments with random HTML/CSS templates
      → Tries multiple hosting providers and auth flows
      → Debugs upload endpoints and response formats
      → Rewrites password logic several times
      → Finally ships a fragile page after many retries

Result: ❌ Slow delivery, inconsistent styling, avoidable token/API waste
```

**With open-skills:**

```
User: "Create a beautiful white-themed report website from this content and host it instantly"

Agent → Finds generate-report-originless-site.md
      → Generates index.html with Tailwind CDN + subtle animations
      → Applies clean white-background report layout
      → Uploads to Originless (local/public endpoint)
      → Returns hosted URL/CID immediately
      → If requested, adds client-side password unlock for encrypted content

Result: ✅ Fast static site generation, instant decentralized hosting, predictable output
```

**Savings:** Fewer retries, faster publish time, and consistent website quality with account-free hosting

## Cost Savings Calculator

### For Cloud Models (Make them 98% cheaper)

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

---

### For Local Models (Make them actually work)

**The Real Game-Changer:** Open Skills makes local models competitive with GPT-4 for practical tasks.

| Model Stack | Cost | Success Rate | Speed | Privacy |
|-------------|------|--------------|-------|---------|
| **Cloud models without skills** | $0.15-$0.25/task | 85-95% | 2-3 min | ❌ Cloud |
| **Cloud models with skills** | $0.003-$0.005/task | 98% | 10 sec | ❌ Cloud |
| **Local models without skills** | $0 | 30-50% | Varies | ✅ Local |
| **🚀 Local models + Open Skills** | **$0** | **95%+** | **10 sec** | **✅ Local** |

**The 100% Free, Self-Hostable AI Agent Stack:**

```bash
# Install Ollama (free, local)
curl -fsSL https://ollama.com/install.sh | sh
ollama pull llama3.1:8b

# Clone Open Skills (free, open-source)
git clone https://github.com/besoeasy/open-skills ~/open-skills

# Result: GPT-4-level task execution at $0 cost
# - No API keys needed
# - No cloud dependency
# - Complete privacy
# - 100% self-hostable
```

**Monthly cost comparison:**
- **Cloud models (GPT-4/Claude) without skills:** $150-$1,470/month (1,000 tasks)
- **Cloud models with skills:** $3-$15/month (95%+ savings)
- **Local models (Llama/Mistral) + Open Skills:** **$0/month** (100% free, actually works)

---

**Plus:** Eliminate search API costs entirely by using free SearXNG instances instead of:

- Google Custom Search API ($5/1000 queries) → **$0 with SearXNG**
- Brave Search API ($5/1000 queries) → **$0 with SearXNG**
- Bing Search API ($3-7/1000 queries) → **$0 with SearXNG**

**Total potential savings: $600-$2,300/month** for active AI agents  
**Or go 100% free with local models + Open Skills: $0/month forever**

## Perfect For

- 🏠 **Self-hosted AI enthusiasts** — Run Llama/Mistral with Ollama + Open Skills for GPT-4-level capabilities at $0 cost
- 🤖 **Autonomous AI agents** — Give your agent production-ready capabilities out of the box
- 💼 **Business automation** — Crypto monitoring, document processing, web scraping, notifications
- 🔍 **Eliminating API costs** — Replace expensive search, translation, geocoding, and weather APIs with free alternatives
- 🛠️ **Developer tools** — Integrate with OpenCode.ai, Claude Desktop, Ollama, custom MCP servers
- 📚 **AI learning** — Study working examples instead of guessing API patterns
- 🔐 **Privacy-conscious projects** — All skills use open-source tools and public APIs, run entirely offline
- 💰 **Cost-sensitive teams** — Reduce AI agent costs by 98% or go completely free with local models


## Philosophy

**Why we built this:**

AI agents are incredibly powerful, but there's a massive gap:
- **Expensive cloud models (GPT-4, Claude, Gemini):** Smart enough to figure things out, but cost $0.15-$0.25+ per task
- **Free local models (Llama, Mistral, Qwen):** Can't figure things out reliably, so they fail or give up

**Open Skills bridges this gap** by providing the "figuring out" part:
- Instead of making models search, experiment, and debug → Give them working code
- Instead of requiring high intelligence → Provide pre-tested patterns
- Result: **Cheap models execute like expensive models**

**Our approach:**

- ✅ **Tested code, not theory** — Every example is production-ready
- ✅ **Privacy-first** — Open-source tools, minimal tracking, no vendor lock-in
- ✅ **Agent-optimized** — Written for LLM consumption (clear structure, copy-paste ready)
- ✅ **Free to use** — MIT licensed, no API keys required for core functionality
- ✅ **Model-agnostic** — Works with GPT-4, Claude, Gemini, Llama, Mistral, Qwen, any LLM

**The result:** AI agents that are smarter, faster, and cheaper to run — or **completely free** with local models.

