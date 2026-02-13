---
name: common-patterns
description: Reusable error handling, retry logic, rate limiting, and timeout patterns for building robust agent skills.
---

# Common Patterns for Robust Skills

Reusable code patterns for error handling, retries, rate limiting, and timeouts. Reference these patterns when building or improving skills to reduce duplication and increase reliability.

All examples use **Node.js** (with Bash where applicable). Agents can translate to other languages on demand.

## Error Handling Patterns

### Basic try-catch with fallback

**Node.js:**
```javascript
async function fetchWithFallback(primaryUrl, fallbackUrl) {
  try {
    const res = await fetch(primaryUrl);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (err) {
    console.warn(`Primary failed: ${err.message}, trying fallback...`);
    const res = await fetch(fallbackUrl);
    if (!res.ok) throw new Error(`Fallback also failed: HTTP ${res.status}`);
    return await res.json();
  }
}
```

**Bash:**
```bash
fetch_with_fallback() {
  local primary="$1"
  local fallback="$2"
  
  if ! response=$(curl -fsS "$primary" 2>/dev/null); then
    echo "Primary failed, trying fallback..." >&2
    response=$(curl -fsS "$fallback")
  fi
  
  echo "$response"
}
```

## Retry Logic with Exponential Backoff

### Retry with increasing delays

**Node.js:**
```javascript
async function retryWithBackoff(fn, maxRetries = 3, baseDelay = 1000) {
  let lastError;
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (err) {
      lastError = err;
      if (i < maxRetries - 1) {
        const delay = baseDelay * Math.pow(2, i);
        console.warn(`Attempt ${i + 1} failed: ${err.message}. Retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }
  
  throw new Error(`Failed after ${maxRetries} attempts: ${lastError.message}`);
}

// Usage:
// await retryWithBackoff(() => fetch('https://api.example.com/data'));
```

**Bash:**
```bash
retry_with_backoff() {
  local cmd="$1"
  local max_retries="${2:-3}"
  local base_delay="${3:-1}"
  
  for i in $(seq 1 "$max_retries"); do
    if eval "$cmd"; then
      return 0
    fi
    
    if [ "$i" -lt "$max_retries" ]; then
      local delay=$((base_delay * (2 ** (i - 1))))
      echo "Attempt $i failed. Retrying in ${delay}s..." >&2
      sleep "$delay"
    fi
  done
  
  echo "Failed after $max_retries attempts" >&2
  return 1
}

# Usage:
# retry_with_backoff "curl -fsS https://api.example.com/data"
```

## Rate Limiting

### Simple token bucket rate limiter

**Node.js:**
```javascript
class RateLimiter {
  constructor(maxRequests, windowMs) {
    this.maxRequests = maxRequests;
    this.windowMs = windowMs;
    this.requests = [];
  }
  
  async throttle() {
    const now = Date.now();
    this.requests = this.requests.filter(time => now - time < this.windowMs);
    
    if (this.requests.length >= this.maxRequests) {
      const oldestRequest = this.requests[0];
      const waitTime = this.windowMs - (now - oldestRequest);
      console.log(`Rate limit reached. Waiting ${waitTime}ms...`);
      await new Promise(resolve => setTimeout(resolve, waitTime + 100));
      return this.throttle();
    }
    
    this.requests.push(now);
  }
}

// Usage:
// const limiter = new RateLimiter(5, 1000); // 5 requests per second
// await limiter.throttle();
// await fetch('https://api.example.com/data');
```

## Timeout Handling

### Request with timeout and graceful degradation

**Node.js:**
```javascript
async function fetchWithTimeout(url, timeoutMs = 10000) {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeoutMs);
  
  try {
    const res = await fetch(url, { signal: controller.signal });
    clearTimeout(timeoutId);
    
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (err) {
    clearTimeout(timeoutId);
    
    if (err.name === 'AbortError') {
      throw new Error(`Request timeout after ${timeoutMs}ms`);
    }
    throw err;
  }
}
```

**Bash:**
```bash
fetch_with_timeout() {
  local url="$1"
  local timeout="${2:-10}"
  
  if ! response=$(curl -fsS --max-time "$timeout" "$url" 2>&1); then
    if echo "$response" | grep -q "timeout"; then
      echo "Request timeout after ${timeout}s" >&2
      return 28  # curl timeout exit code
    fi
    echo "Request failed: $response" >&2
    return 1
  fi
  
  echo "$response"
}
```

## Circuit Breaker Pattern

### Prevent cascading failures

**Node.js:**
```javascript
class CircuitBreaker {
  constructor(threshold = 5, timeout = 60000) {
    this.failureThreshold = threshold;
    this.timeout = timeout;
    this.failures = 0;
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
    this.nextAttempt = Date.now();
  }
  
  async execute(fn) {
    if (this.state === 'OPEN') {
      if (Date.now() < this.nextAttempt) {
        throw new Error('Circuit breaker is OPEN');
      }
      this.state = 'HALF_OPEN';
    }
    
    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (err) {
      this.onFailure();
      throw err;
    }
  }
  
  onSuccess() {
    this.failures = 0;
    this.state = 'CLOSED';
  }
  
  onFailure() {
    this.failures++;
    if (this.failures >= this.failureThreshold) {
      this.state = 'OPEN';
      this.nextAttempt = Date.now() + this.timeout;
      console.warn(`Circuit breaker opened. Next attempt in ${this.timeout}ms`);
    }
  }
}

// Usage:
// const breaker = new CircuitBreaker(3, 30000);
// await breaker.execute(() => fetch('https://api.example.com/data'));
```

## JSON Parse with Validation

### Safe JSON parsing with schema validation

**Node.js:**
```javascript
function safeParse(text, requiredFields = []) {
  try {
    const data = JSON.parse(text);
    
    for (const field of requiredFields) {
      if (!(field in data)) {
        throw new Error(`Missing required field: ${field}`);
      }
    }
    
    return data;
  } catch (err) {
    if (err instanceof SyntaxError) {
      throw new Error(`Invalid JSON: ${err.message}`);
    }
    throw err;
  }
}

// Usage:
// const data = safeParse(responseText, ['status', 'result']);
```

## Multiple Endpoint Fallback Chain

### Try multiple API endpoints until one succeeds

**Node.js:**
```javascript
async function tryEndpoints(endpoints, options = {}) {
  const errors = [];
  
  for (const endpoint of endpoints) {
    try {
      console.log(`Trying ${endpoint}...`);
      const res = await fetch(endpoint, options);
      
      if (!res.ok) {
        throw new Error(`HTTP ${res.status}`);
      }
      
      return { endpoint, data: await res.json() };
    } catch (err) {
      errors.push({ endpoint, error: err.message });
      console.warn(`${endpoint} failed: ${err.message}`);
    }
  }
  
  throw new Error(`All endpoints failed:\n${errors.map(e => `  ${e.endpoint}: ${e.error}`).join('\n')}`);
}

// Usage:
// const result = await tryEndpoints([
//   'https://api1.example.com/data',
//   'https://api2.example.com/data',
//   'https://api3.example.com/data'
// ]);
```

## Usage in Skills

When writing skills, copy relevant patterns and adapt them to your use case:

1. **API calls with fallback**: Use `fetchWithFallback` for services with multiple endpoints
2. **Unstable APIs**: Wrap with `retryWithBackoff` 
3. **Rate-limited APIs**: Add `RateLimiter` before request loops
4. **Slow endpoints**: Use `fetchWithTimeout` to prevent hanging
5. **Critical services**: Wrap with `CircuitBreaker` to fail fast
6. **Multiple providers**: Use `tryEndpoints` to iterate through alternatives

## See also

- Individual skill files for domain-specific error scenarios
- [web-search-api.md](web-search-api.md#troubleshooting) — SearXNG instance fallback example
- [check-crypto-address-balance.md](check-crypto-address-balance.md#troubleshooting) — Multiple blockchain API fallbacks
