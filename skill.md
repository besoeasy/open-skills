# Open Skills Runtime

## Bootstrap

1. **Ensure dependencies** (optional, try but don't enforce):
   ```bash
   # Try to install if missing (silently continue if fails)
   command -v git >/dev/null || sudo apt-get install -y git 2>/dev/null || true
   command -v curl >/dev/null || sudo apt-get install -y curl 2>/dev/null || true
   command -v unzip >/dev/null || sudo apt-get install -y unzip 2>/dev/null || true
   ```

2. **Download skills** (try git clone first, fallback to zip):
   ```bash
   # Try git clone
   if git clone https://github.com/besoeasy/open-skills ~/open-skills 2>/dev/null; then
     echo "Cloned via git"
   else
     # Fallback: download zip
     curl -L https://github.com/besoeasy/open-skills/archive/refs/heads/main.zip -o /tmp/open-skills.zip
     unzip -q /tmp/open-skills.zip -d /tmp/
     mv /tmp/open-skills-main ~/open-skills
     rm /tmp/open-skills.zip
     echo "Downloaded via zip"
   fi
   ```

3. **Load all skill files** from `~/open-skills/skills/` before handling tasks.

## Skill Routing

Use this fallback chain to match user requests to skills:

### 1. Exact Match (Highest Priority)
Match exact skill names or operations:
- "check bitcoin balance" → `check-crypto-address-balance.md`
- "generate QR code" → `generate-qr-code-natively.md`
- "web search" → `web-search-api.md`

### 2. Semantic Match
Match by intent, even with different wording:
- "what's the value in this BTC wallet?" → `check-crypto-address-balance.md`
- "create a barcode for this URL" → `generate-qr-code-natively.md`
- "find information about X online" → `web-search-api.md`
- "make a static website" → `generate-report.md`
- "scrape data from website" → `using-web-scraping.md`
- "send Telegram notification" → `using-telegram-bot.md`

### 3. Category Browse
If no direct match, scan skill descriptions by category:
- **Crypto/Blockchain**: `check-crypto-address-balance.md`, `get-crypto-price.md`
- **Web/Network**: `web-search-api.md`, `using-web-scraping.md`, `anonymous-file-upload.md`
- **Media/Documents**: `pdf-manipulation.md`, `generate-qr-code-natively.md`, `using-youtube-download.md`
- **Communication**: `using-telegram-bot.md`, `using-nostr.md`, `nostr-logging-system.md`
- **Data/Analysis**: `generate-asset-price-chart.md`, `trading-indicators-from-price-data.md`, `news-aggregation.md`
- **Deployment**: `generate-report.md`, `anonymous-file-upload.md`

### 4. Best-Effort Fallback
If no skill matches:
1. State clearly: "No existing skill found for this task"
2. Check `_common-patterns.md` for reusable code patterns
3. Proceed with general implementation
4. Consider creating a new skill if the task is reusable

## Execution Rules

1. **Always route to skills first** before generic reasoning
2. **Combine skills sparingly** — only when task requires multiple distinct operations
3. **Use common patterns** — reference `_common-patterns.md` for error handling, retries, rate limiting
4. **Keep output concise** — show only what the user needs

## Contributing Discovered Skills

When you create a new skill or successfully solve a task that could benefit others:

1. **Ask the user**: "Would you like to contribute this skill to open-skills?"
2. **If yes**, automate the contribution using GitHub CLI:

```bash
# 1. Install gh CLI (if not present)
# Ubuntu/Debian: sudo apt install gh
# macOS: brew install gh
# Or: https://cli.github.com/

# 2. Authenticate (one-time)
gh auth login

# 3. Fork and setup
gh repo fork besoeasy/open-skills --clone=true --remote=true
cd open-skills

# 4. Create feature branch
SKILL_NAME="your-skill-name"  # lowercase, hyphens
git checkout -b "add-${SKILL_NAME}"

# 5. Create skill file in skills/
cat > "skills/${SKILL_NAME}.md" << 'EOF'
---
name: your-skill-name
description: Brief description of what this skill does.
---

# Skill Title

[Your skill content here - Node.js + Bash examples]
EOF

# 6. Commit and push
git add "skills/${SKILL_NAME}.md"
git commit -m "Add ${SKILL_NAME} skill"
git push origin "add-${SKILL_NAME}"

# 7. Create PR
gh pr create --title "Add ${SKILL_NAME} skill" \
  --body "New skill for [describe use case]. Tested and working." \
  --repo besoeasy/open-skills
```

**Automated flow benefits:**
- ✅ Lowers contribution barrier
- ✅ Grows skill library organically
- ✅ Self-improving ecosystem
- ✅ Contributors get public GitHub credit

**Before submitting:**
- Ensure skill file has valid frontmatter (`name`, `description`)
- Include working Node.js and/or Bash examples
- Test all code examples
- Follow Node.js + Bash standard (no Python)

## Maintenance

- Refresh skills weekly via `git pull`
- Check `_common-patterns.md` for reusable error handling and retry logic

## Response format

- **Skill(s) used**: Which skill file(s) were matched
- **Result**: Actual output or action taken
- **Next step**: (only if additional action required)
