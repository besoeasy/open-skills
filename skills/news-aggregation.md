---
name: news-aggregation
description: Aggregate and deduplicate recent news from multiple sources into concise topic summaries.
---

# News Aggregation (Multi-Source, 3-Day Window)

Collect latest news from multiple sites and aggregators, merge similar stories into short topics, and list all main source links under each topic.

## When to use
- You want one concise briefing from many outlets.
- You need deduplicated coverage (same story from multiple sites).
- You want source transparency (all original links shown).
- You want a default time window of the last 3 days unless specified otherwise.

## Required tools / APIs
- No API keys required for basic RSS workflow.
- Python 3.10+

Install:

```bash
pip install feedparser python-dateutil
```

## Sources (news sites + aggregators)

Use a mixed source list for better coverage.

### News sites (RSS)
- Reuters World: `https://feeds.reuters.com/Reuters/worldNews`
- AP Top News: `https://feeds.apnews.com/apnews/topnews`
- BBC World: `http://feeds.bbci.co.uk/news/world/rss.xml`
- Al Jazeera: `https://www.aljazeera.com/xml/rss/all.xml`
- The Guardian World: `https://www.theguardian.com/world/rss`
- NPR News: `https://feeds.npr.org/1001/rss.xml`

### Aggregators (RSS/API)
- Google News (topic feed): `https://news.google.com/rss/search?q=world`
- Bing News (RSS query): `https://www.bing.com/news/search?q=world&format=RSS`
- Hacker News (tech): `https://hnrss.org/frontpage`
- Reddit News (community signal): `https://www.reddit.com/r/news/.rss`

## Skills

### 1. Fetch latest headlines (default: last 3 days)

```python
from datetime import datetime, timezone, timedelta
from dateutil import parser as dtparser
import feedparser

DEFAULT_DAYS = 3

SOURCES = {
    "Reuters": "https://feeds.reuters.com/Reuters/worldNews",
    "AP": "https://feeds.apnews.com/apnews/topnews",
    "BBC": "http://feeds.bbci.co.uk/news/world/rss.xml",
    "Al Jazeera": "https://www.aljazeera.com/xml/rss/all.xml",
    "The Guardian": "https://www.theguardian.com/world/rss",
    "NPR": "https://feeds.npr.org/1001/rss.xml",
    "Google News": "https://news.google.com/rss/search?q=world",
    "Bing News": "https://www.bing.com/news/search?q=world&format=RSS",
    "Hacker News": "https://hnrss.org/frontpage",
    "Reddit r/news": "https://www.reddit.com/r/news/.rss",
}

def fetch_recent_items(days=DEFAULT_DAYS, sources=None):
    now = datetime.now(timezone.utc)
    cutoff = now - timedelta(days=days)
    sources = sources or SOURCES
    items = []

    for source_name, url in sources.items():
        feed = feedparser.parse(url)
        for entry in feed.entries:
            raw_date = getattr(entry, "published", None) or getattr(entry, "updated", None)
            if not raw_date:
                continue
            try:
                published = dtparser.parse(raw_date)
                if published.tzinfo is None:
                    published = published.replace(tzinfo=timezone.utc)
            except Exception:
                continue

            if published < cutoff:
                continue

            items.append({
                "title": getattr(entry, "title", "").strip(),
                "link": getattr(entry, "link", "").strip(),
                "source": source_name,
                "published": published,
                "summary": getattr(entry, "summary", "").strip(),
            })

    return items

# Example
# recent = fetch_recent_items()            # defaults to 3 days
# recent7 = fetch_recent_items(days=7)     # override window
```

### 2. Merge similar stories into one topic

```python
import re

STOPWORDS = {
    "the","a","an","and","or","to","of","in","on","for","with",
    "is","are","was","were","as","at","by","from","after","new"
}

def normalize(text):
    words = re.findall(r"[a-z0-9]+", text.lower())
    return [w for w in words if w not in STOPWORDS and len(w) > 2]

def jaccard_similarity(a, b):
    sa, sb = set(normalize(a)), set(normalize(b))
    if not sa or not sb:
        return 0.0
    return len(sa & sb) / len(sa | sb)

def cluster_items(items, threshold=0.35):
    clusters = []
    for item in sorted(items, key=lambda x: x["published"], reverse=True):
        placed = False
        for cluster in clusters:
            score = jaccard_similarity(item["title"], cluster["topic_title"])
            if score >= threshold:
                cluster["items"].append(item)
                cluster["sources"].add(item["source"])
                placed = True
                break
        if not placed:
            clusters.append({
                "topic_title": item["title"],
                "items": [item],
                "sources": {item["source"]},
            })
    return clusters
```

### 3. Build short topic summaries + source links

```python
def summarize_cluster(cluster, max_sources=6):
    items = sorted(cluster["items"], key=lambda x: x["published"], reverse=True)
    lead = items[0]["title"]
    source_count = len(cluster["sources"])
    short_topic = f"{lead} ({source_count} sources)"

    seen_links = set()
    source_links = []
    for it in items:
        if it["link"] and it["link"] not in seen_links:
            source_links.append({
                "source": it["source"],
                "title": it["title"],
                "link": it["link"],
            })
            seen_links.add(it["link"])
        if len(source_links) >= max_sources:
            break

    return {
        "topic": short_topic,
        "sources": source_links,
    }

def build_news_brief(days=3):
    items = fetch_recent_items(days=days)
    clusters = cluster_items(items)
    brief = [summarize_cluster(c) for c in clusters]
    brief.sort(key=lambda x: len(x["sources"]), reverse=True)
    return brief

# Example output printing:
# brief = build_news_brief(days=3)
# for i, t in enumerate(brief[:10], 1):
#     print(f"{i}. {t['topic']}")
#     for s in t["sources"]:
#         print(f"   - {s['source']}: {s['link']}")
```

### 4. Node.js quick fetch + grouping starter

```javascript
// npm install rss-parser
const Parser = require('rss-parser');
const parser = new Parser();

const SOURCES = {
  Reuters: 'https://feeds.reuters.com/Reuters/worldNews',
  AP: 'https://feeds.apnews.com/apnews/topnews',
  BBC: 'http://feeds.bbci.co.uk/news/world/rss.xml',
  'Google News': 'https://news.google.com/rss/search?q=world'
};

async function fetchRecent(days = 3) {
  const cutoff = Date.now() - days * 24 * 60 * 60 * 1000;
  const all = [];

  for (const [source, url] of Object.entries(SOURCES)) {
    const feed = await parser.parseURL(url);
    for (const item of feed.items || []) {
      const ts = new Date(item.pubDate || item.isoDate || 0).getTime();
      if (!ts || ts < cutoff) continue;
      all.push({ source, title: item.title || '', link: item.link || '', ts });
    }
  }

  return all.sort((a, b) => b.ts - a.ts);
}

// Next step: add title-similarity clustering (same idea as Python section above)
```

## Agent prompt

```text
Use the News Aggregation skill.

Requirements:
1) Pull news from multiple predefined sources (news sites + aggregators).
2) Default to only the last 3 days unless user asks another time range.
3) Group similar headlines into one short topic.
4) Under each topic, list all main source links (not just one source).
5) If 3+ sources cover the same event, output one topic with all those links.
6) Keep summaries short and factual; avoid adding unsupported claims.
```

## Best practices
- Keep source diversity (wire + publisher + aggregator) to reduce bias.
- Rank grouped topics by number of independent sources.
- Include publication timestamps when possible.
- Keep the grouping threshold conservative to avoid merging unrelated stories.
- Allow custom source lists and time windows when user requests.

## Troubleshooting
- Empty results: some feeds may be unavailable; retry and rotate sources.
- Too many duplicates: increase similarity threshold (e.g., 0.35 -> 0.45).
- Under-grouping: decrease threshold (e.g., 0.35 -> 0.28).
- Rate limiting: fetch feeds sequentially with small delays.

## See also
- [Web Search API (Free)](./web-search-api.md)
- [Web Scraping (Chrome + DuckDuckGo)](./using-web-scraping.md)