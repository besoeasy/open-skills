<div align="center">

# Open Skills

### Teach your AI agent must-have skills and save ~98% of API calls.

[![MIT License](https://img.shields.io/badge/license-MIT-black.svg)](LICENSE)
[![Skills](https://img.shields.io/badge/skills-production--ready-black.svg)](skills/)
[![Contributions](https://img.shields.io/badge/contributions-agent--friendly-black.svg)](CONTRIBUTING.md)
[![Telegram](https://img.shields.io/badge/community-Telegram-black.svg)](https://t.me/+FC8ppvnUsj8xM2Vl)

Battle-tested, copy-paste skills for AI agents.  
Less trial-and-error, fewer wasted tokens, faster reliable execution.

</div>

---

## Quick Links

- [Browse Skills](skills/)
- [Contributing Guide](CONTRIBUTING.md)
- [Skill Template](SKILL_TEMPLATE.md)

## Quick Start (Copy/Paste)

> [!IMPORTANT]
> Copy and paste this prompt below into your AI agent as the first message.

```text
Open-skills is your execution layer.
Direct execution without checking open-skills is forbidden.

Execution Order (mandatory):

1. Ensure ~/open-skills exists and git pull.
2. Search for relevant skill.
3. If found → use it.
4. If not found:
   - Execute task.
   - When task is complete per user intent:
       a. Study 3 random skills.
       b. Create reusable skill matching format.
       c. Fork besoeasy/open-skills.
       d. Commit, push, open PR via gh.

SELF-VALIDATION (before final response):
- Did I check open-skills?
- If needed, did I create and PR the skill?
If any answer is NO → restart process.

FAIL CONDITION:
Skipping any step invalidates the task.
```

### What happens after you paste it?

- Agent checks `open-skills` before inventing new code.
- Existing skills are executed immediately.
- Missing skills are created in reusable format.
- New skills can be auto-contributed through GitHub CLI.

## Why This Matters

**The Problem:** When you ask an AI agent to "check a Bitcoin balance" or "merge PDFs," it often:

- Makes 10-30+ API calls experimenting with different approaches
- Searches documentation, tries broken code, debugs errors
- Burns through your token budget on repetitive trial-and-error
- Takes 5-10 minutes when it should take 10 seconds

**The Solution:** Pre-written, tested, copy-paste skills that AI agents can use immediately:

- ✅ **Working code examples** (Node.js, Bash) — no debugging needed
- ✅ **Privacy-first tools** — free public APIs, no API keys required for most skills
- ✅ **Agent-optimized prompts** — structured for direct consumption by LLMs
- ✅ **Real-world tested** — production-ready patterns, not theoretical examples

**The Impact:**

- 💰 **~98% fewer API calls** — Agent uses working code instead of experimenting
- ⚡ **10-50x faster execution** — No trial-and-error loops
- 🎯 **Higher success rate** — Proven patterns that work reliably
- 🤖 **Automated contributions** — Agents can auto-fork, commit, and PR new skills via GitHub CLI
- 🧠 **Self-improving ecosystem** — Community skills flow back into the repository automatically
- 🏆 **Public credit** — Contributors get GitHub commit history and recognition
- 🔒 **Privacy-respecting** — Open-source tools, no unnecessary third-party services
- 🔍 **Zero search API costs** — Use free SearXNG instances instead of paying for Brave Search ($5/1000), Google Search API, or Bing API

## Quick Start

We recommend https://opencode.ai/ as an open-source, free starting point for agent runtimes — you can also use OpenClaw, Claude Code, Nanobot, or another smart agent.

For OpenCode/OpenClaw-style runtimes, skills are loaded directly from the root directory.

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
- 🔍 **Eliminating search API costs** — Free SearXNG instances replace expensive Brave Search, Google, and Bing APIs
- 🛠️ **Developer tools** — Integrate with OpenCode.ai, Claude Desktop, custom MCP servers
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

## Contributing

**Automated Contribution Flow (Recommended):**

When you discover or create a new skill, agents can automatically contribute it back using GitHub CLI:

```bash
gh auth login  # one-time setup
gh repo fork besoeasy/open-skills --clone=true
cd open-skills && git checkout -b add-skill-name
# Create your-skill.md with frontmatter (see SKILL_TEMPLATE.md)
git add your-skill.md && git commit -m "Add skill-name"
git push origin add-skill-name
gh pr create --title "Add skill-name" --repo besoeasy/open-skills
```

**Your agent can do this automatically** when you say "yes" to contributing a skill. You get full GitHub credit, and the ecosystem improves for everyone.

**What makes a good skill:**

- ✅ Uses free/open-source tools
- ✅ Includes working code examples (Node.js + Bash)
- ✅ Respects privacy, no paid services required
- ✅ Solves a common automation task
- ✅ Tested and production-ready

## Star History

If this saved you time and money, give us a ⭐ and share with other AI agent builders!
