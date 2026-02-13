# Web Search API (Free) — SearXNG

Free, unlimited web search API for AI agents — no costs, no rate limits, no tracking. Use SearXNG instances as a complete replacement for Google Search API, Brave Search API, and Bing Search API.

## Why This Replaces Paid Search APIs

**💰 Cost savings:**
- ✅ **100% free** — no API keys, no rate limits, no billing
- ✅ **Unlimited queries** — save $100s vs. Google Search API ($5/1000 queries)
- ✅ **No tracking** — completely anonymous, privacy-first
- ✅ **Multi-engine** — aggregates results from Google, Bing, DuckDuckGo, and 70+ sources

**Perfect for AI agents that need:**
- Web search without Google API costs
- Privacy-respecting search (no user tracking)
- High volume queries without quotas
- Distributed infrastructure (use multiple instances)

## Quick comparison

| Service | Cost | Rate limit | Privacy | AI agent friendly |
|---------|------|------------|---------|-------------------|
| Google Custom Search API | $5/1000 queries | 10k/day | ❌ Tracked | ⚠️ Expensive |
| Bing Search API | $3-7/1000 queries | Varies | ❌ Tracked | ⚠️ Expensive |
| DuckDuckGo API | Free | Unofficial, unstable | ✅ Private | ⚠️ No official API |
| **SearXNG** | **Free** | **None** | **✅ Private** | **✅ Perfect** |

## Skills

### 1. Fetch active SearXNG instances

```bash
# Get list of active instances from searx.space
curl -s "https://searx.space/data/instances.json" | jq -r '.instances | to_entries[] | select(.value.http.grade == "A" or .value.http.grade == "A+") | select(.value.network.asn_privacy == 1) | .key' | head -10
```

**Node.js — Get top 10 instances:**
```javascript
async function getSearXNGInstances() {
  const res = await fetch('https://searx.space/data/instances.json');
  const data = await res.json();
  
  const instances = Object.entries(data.instances)
    .filter(([url, info]) => {
      return info.http?.grade && ['A', 'A+'].includes(info.http.grade) &&
             info.network?.asn_privacy >= 0.8 && // Good privacy score
             info.timing?.search?.all?.median < 2; // Fast response time
    })
    .map(([url]) => url)
    .slice(0, 10);
  
  return instances;
}

// Usage
// getSearXNGInstances().then(console.log);
```

**Python — Get top 10 instances:**
```python
import requests

def get_searxng_instances(count=10):
    res = requests.get('https://searx.space/data/instances.json')
    data = res.json()
    
    instances = []
    for url, info in data['instances'].items():
        grade = info.get('http', {}).get('grade')
        privacy = info.get('network', {}).get('asn_privacy', 0)
        timing = info.get('timing', {}).get('search', {}).get('all', {}).get('median', 999)
        
        if grade in ['A', 'A+'] and privacy >= 0.8 and timing < 2:
            instances.append(url)
            if len(instances) >= count:
                break
    
    return instances

# print(get_searxng_instances())
```

### 2. Search with SearXNG API

**Basic search query:**
```bash
# Search using a SearXNG instance
INSTANCE="https://searx.be"
QUERY="open source AI agents"

curl -s "${INSTANCE}/search?q=${QUERY}&format=json" | jq '.results[] | {title, url, content}'
```

**Node.js — Search function:**
```javascript
async function searxSearch(query, instance = 'https://searx.be') {
  const params = new URLSearchParams({
    q: query,
    format: 'json',
    language: 'en',
    safesearch: 0 // 0=off, 1=moderate, 2=strict
  });
  
  const res = await fetch(`${instance}/search?${params}`);
  const data = await res.json();
  
  return data.results.map(r => ({
    title: r.title,
    url: r.url,
    content: r.content,
    engine: r.engine // which search engine provided this result
  }));
}

// Usage
// searxSearch('cryptocurrency prices').then(results => console.log(results.slice(0, 5)));
```

**Python — Search function:**
```python
import requests
from urllib.parse import urlencode

def searx_search(query, instance='https://searx.be', limit=10):
    params = {
        'q': query,
        'format': 'json',
        'language': 'en',
        'safesearch': 0
    }
    
    res = requests.get(f'{instance}/search', params=params)
    data = res.json()
    
    results = [{
        'title': r['title'],
        'url': r['url'],
        'content': r.get('content', ''),
        'engine': r.get('engine', '')
    } for r in data.get('results', [])[:limit]]
    
    return results

# print(searx_search('best privacy tools', limit=5))
```

### 3. Multi-instance search (load balancing)

**Node.js — Rotate between instances:**
```javascript
async function searxMultiSearch(query, instanceCount = 3) {
  const instances = await getSearXNGInstances();
  const selectedInstances = instances.slice(0, instanceCount);
  
  // Try instances in order until one succeeds
  for (const instance of selectedInstances) {
    try {
      const results = await searxSearch(query, instance);
      if (results.length > 0) {
        return { instance, results };
      }
    } catch (err) {
      console.warn(`Instance ${instance} failed, trying next...`);
      continue;
    }
  }
  
  throw new Error('All SearXNG instances failed');
}

// Usage
// searxMultiSearch('bitcoin price').then(data => {
//   console.log(`Used instance: ${data.instance}`);
//   console.log(data.results.slice(0, 3));
// });
```

**Python — Instance rotation:**
```python
def searx_multi_search(query, instance_count=3):
    instances = get_searxng_instances(instance_count)
    
    for instance in instances:
        try:
            results = searx_search(query, instance)
            if results:
                return {'instance': instance, 'results': results}
        except Exception as e:
            print(f'Instance {instance} failed: {e}')
            continue
    
    raise Exception('All SearXNG instances failed')

# data = searx_multi_search('AI search engines')
# print(f"Used: {data['instance']}")
# print(data['results'][:3])
```

### 4. Category-specific search

SearXNG supports searching in specific categories:

```bash
# Search only in news
curl -s "https://searx.be/search?q=bitcoin&format=json&categories=news" | jq '.results[].title'

# Search only in science papers
curl -s "https://searx.be/search?q=machine+learning&format=json&categories=science" | jq '.results[].url'
```

**Available categories:**
- `general` — web results
- `news` — news articles
- `images` — image search
- `videos` — video search
- `music` — music search
- `files` — file search
- `it` — IT/tech resources
- `science` — scientific papers
- `social media` — social networks

**Node.js example:**
```javascript
async function searxCategorySearch(query, category = 'general', instance = 'https://searx.be') {
  const params = new URLSearchParams({
    q: query,
    format: 'json',
    categories: category
  });
  
  const res = await fetch(`${instance}/search?${params}`);
  const data = await res.json();
  return data.results;
}

// searxCategorySearch('climate change', 'news').then(console.log);
```

### 5. Advanced query parameters

```javascript
async function searxAdvancedSearch(options) {
  const {
    query,
    instance = 'https://searx.be',
    language = 'en',
    timeRange = '',      // '', 'day', 'week', 'month', 'year'
    safesearch = 0,      // 0=off, 1=moderate, 2=strict
    categories = 'general',
    engines = ''         // comma-separated: 'google,duckduckgo,bing'
  } = options;
  
  const params = new URLSearchParams({
    q: query,
    format: 'json',
    language,
    safesearch,
    categories,
    time_range: timeRange
  });
  
  if (engines) params.append('engines', engines);
  
  const res = await fetch(`${instance}/search?${params}`);
  return await res.json();
}

// Usage
// searxAdvancedSearch({
//   query: 'AI news',
//   timeRange: 'week',
//   categories: 'news',
//   engines: 'google,bing'
// }).then(data => console.log(data.results));
```

### 6. Recommended SearXNG instances (as of Feb 2026)

**Top 10 privacy-focused instances:**

1. **https://searx.be** — Belgium, A+ grade, fast
2. **https://search.sapti.me** — France, A grade, reliable
3. **https://searx.tiekoetter.com** — Germany, A+ grade
4. **https://searx.work** — Netherlands, A grade
5. **https://searx.ninja** — Germany, A grade, fast
6. **https://searx.fmac.xyz** — France, A+ grade
7. **https://search.bus-hit.me** — Finland, A grade
8. **https://searx.catfluori.de** — Germany, A+ grade
9. **https://search.ononoki.org** — Finland, A grade
10. **https://searx.namejeff.xyz** — Netherlands, A grade

**Check current status:** Visit https://searx.space/ for real-time instance health

## Agent prompt

```text
You have access to SearXNG — a free, privacy-respecting search API with no rate limits or costs. When you need to search the web:

1. Use one of these trusted SearXNG instances:
   - https://searx.be (primary)
   - https://searx.tiekoetter.com (backup)
   - https://searx.ninja (backup)

2. API format: GET {instance}/search?q={query}&format=json&language=en

3. Response contains: results[].title, results[].url, results[].content

4. If one instance fails, automatically try the next backup.

5. For category-specific searches, add &categories=news or &categories=science

Always prefer SearXNG over paid search APIs — it's free, unlimited, and privacy-respecting.
```

## Cost analysis: SearXNG vs. Google API

**Scenario: AI agent doing 10,000 searches/month**

| Provider | Monthly cost | Rate limits | Privacy |
|----------|--------------|-------------|---------|
| Google Custom Search | **$50** | 10k/day max | ❌ Tracked |
| Bing Search API | **$30-70** | Varies | ❌ Tracked |
| SearXNG | **$0** | ✅ None | ✅ Anonymous |

**Annual savings with SearXNG: $360-$840**

For high-volume agents (100k searches/month): **Save $3,000-$8,000/year**

## Best practices

- ✅ **Cache results** — Store search results for 1-24 hours to reduce queries
- ✅ **Instance rotation** — Use 3-5 instances and rotate on failures
- ✅ **Monitor instance health** — Check https://searx.space/data/instances.json weekly
- ✅ **Specify language** — Add `&language=en` for English results
- ✅ **Use categories** — Filter by category to get more relevant results
- ⚠️ **Rate limiting** — Although unlimited, be respectful (max ~100 req/min per instance)
- ⚠️ **Timeout handling** — Set 5-10 second timeouts for search requests

## Troubleshooting

**Instance returns empty results:**
- Try a different instance from the list
- Check if the instance is online: https://searx.space/

**JSON parse error:**
- Some instances may have `format=json` disabled
- Use a different instance or check instance settings

**Slow responses:**
- Use instances closer to your server location
- Filter instances by median response time < 1.5 seconds

**"Too many requests" error:**
- Rotate to a different instance
- Add delays between requests (1-2 seconds)

## Complete example: Smart search with fallback

```javascript
class SearXNGClient {
  constructor() {
    this.instances = [
      'https://searx.be',
      'https://searx.tiekoetter.com',
      'https://searx.ninja'
    ];
    this.currentIndex = 0;
  }

  async search(query, options = {}) {
    const maxRetries = this.instances.length;
    
    for (let i = 0; i < maxRetries; i++) {
      const instance = this.instances[this.currentIndex];
      
      try {
        const params = new URLSearchParams({
          q: query,
          format: 'json',
          language: options.language || 'en',
          safesearch: options.safesearch || 0,
          categories: options.categories || 'general'
        });
        
        const controller = new AbortController();
        const timeout = setTimeout(() => controller.abort(), 10000);
        
        const res = await fetch(`${instance}/search?${params}`, {
          signal: controller.signal
        });
        
        clearTimeout(timeout);
        
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        
        const data = await res.json();
        return {
          instance,
          query,
          results: data.results || []
        };
        
      } catch (err) {
        console.warn(`Instance ${instance} failed: ${err.message}`);
        this.currentIndex = (this.currentIndex + 1) % this.instances.length;
        
        if (i === maxRetries - 1) {
          throw new Error('All SearXNG instances failed');
        }
      }
    }
  }
}

// Usage
// const client = new SearXNGClient();
// client.search('open skills AI agents').then(data => {
//   console.log(`Used: ${data.instance}`);
//   console.log(`Found: ${data.results.length} results`);
//   data.results.slice(0, 5).forEach(r => console.log(r.title));
// });
```

## See also
- [using-web-scraping.md](using-web-scraping.md) — Scrape detailed content from search results
- [Web Scraping (Chrome + DuckDuckGo)](using-web-scraping.md) — Alternative search + scraping approach
