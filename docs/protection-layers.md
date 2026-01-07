# Protection Layers

Understanding Bequests' multi-layer protection system.

---

## Overview

Bequests uses a **layered protection system** to evade bot detection. Each layer adds specific anti-detection techniques, from basic identity masking to advanced behavioral simulation.

## Layer Components

### `identity` - User-Agent Masking

**What it does:** Sets a realistic User-Agent header matching the TLS fingerprint.

**When to use:** Always - this is the foundation of stealth.

```python
bot = Bequests().layers(['identity'])
```

**Headers Added:**

- `User-Agent` - Browser identification string

**Example User-Agents:**

- Chrome: `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36...`
- Safari: `Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15...`

---

### `ghost` - Silent Captcha Detection

**What it does:** Detects captcha challenges in response HTML without breaking flow.

**Detection Keywords:**

- `g-recaptcha` - Google reCAPTCHA
- `hcaptcha` - hCaptcha
- `turnstile` - Cloudflare Turnstile
- `cf-challenge` - Cloudflare challenge page
- `captcha-delivery` - Generic captcha

**When detected:** Increments `stats["captcha"]` counter and logs warning.

```python
bot = Bequests().layers(['identity', 'ghost'])
response = bot.get("https://protected-site.com")

stats = bot.get_stats()
print(f"Captchas encountered: {stats['captcha']}")
```

---

### `jitter` - Random Delays

**What it does:** Adds random delays between requests to mimic human behavior.

**Default Range:** 1.0 - 3.0 seconds

**Customize:**

```python
# Slow and stealthy
bot.set_speed(2.0, 5.0).layers(['identity', 'jitter'])

# Fast but still human-like
bot.set_speed(0.5, 1.5).layers(['identity', 'jitter'])
```

**Why it works:** Bots typically make requests instantly. Humans take time to read, scroll, and click.

---

### `header_order` - Browser-Specific Headers

**What it does:** Adds Chrome-specific security headers in the correct order.

**Chrome Headers Added:**

```
Sec-CH-UA: "Not_A Brand";v="8", "Chromium";v="120", "Google Chrome";v="120"
Sec-CH-UA-Mobile: ?0
Sec-CH-UA-Platform: "Windows" or "macOS"
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: cross-site
Sec-Fetch-User: ?1
```

**When to use:** For sites that check Chrome security headers (most modern sites).

```python
bot.set_engine("chrome120").layers(['identity', 'header_order'])
```

**Note:** Only active when engine is `chrome120`. Safari doesn't send these headers.

---

### `canvas` - Canvas Fingerprinting Headers

**What it does:** Adds device capability headers to mimic real browsers.

**Headers Added:**

```
Viewport-Width: 1920, 1366, or 1440 (random)
Device-Memory: 8, 16, or 32 GB (random)
DPR: 1, 1.5, or 2 (device pixel ratio, random)
```

**Why it works:** These headers prove you're a real browser with actual hardware.

```python
bot.layers(['identity', 'canvas'])
```

---

### `cookie_warmup` - TCP/TLS Handshake

**What it does:** Pre-visits `favicon.ico` to establish TCP/TLS connection before main request.

**Benefits:**

- Warms up the TLS session
- Establishes connection pooling
- Makes subsequent requests faster and more legitimate

```python
bot.layers(['identity', 'cookie_warmup'])
```

**Process:**

1. First request to target → pre-visits `/favicon.ico`
2. TCP/TLS handshake completed
3. Main request uses established connection

---

### `search_click` - Google Search Referrer

**What it does:** Makes it appear you clicked a Google search result.

**Modifications:**

1. Sets referrer to Google search: `https://www.google.com/search?q={domain}`
2. Adds Google tracking parameter: `ved=0ahUKEwi{random}`

```python
bot.layers(['identity', 'search_click'])
response = bot.get("https://example.com")

# Actual URL visited:
# https://example.com?ved=0ahUKEwi5234
# 
# Referrer header:
# https://www.google.com/search?q=example.com
```

**Why it works:** Most legitimate traffic comes from search engines. This makes you look like organic traffic.

---

### `nuclear` - Google Cache Routing

**What it does:** Routes ALL requests through Google's cache server.

**⚠️ Warning:** This is the most extreme protection layer. Use only when everything else fails.

**How it works:**

Original URL: `https://example.com/page`

Nuclear URL: `https://webcache.googleusercontent.com/search?q=cache:https://example.com/page`

```python
from bequests import Bequests, NUCLEAR_LAYER

bot = Bequests().layers(NUCLEAR_LAYER)
response = bot.get("https://blocked-site.com")
```

**Limitations:**

- May not work for dynamic content
- Slower response times
- Cached content may be outdated
- Some sites block Google cache access

**Best for:**

- Sites with aggressive WAF/bot protection
- Last resort when all proxies are blocked
- Reading static content

---

## Protection Presets

### `LOWER_LAYER` - Minimal Protection

```python
LOWER_LAYER = ['identity', 'jitter']
```

**Use when:**

- Target has minimal bot protection
- Speed is priority
- Testing or development

**Example:**

```python
bot = Bequests().layers(LOWER_LAYER)
```

---

### `MEDIUM_LAYER` - Balanced (Default)

```python
MEDIUM_LAYER = ['identity', 'ghost', 'jitter', 'header_order']
```

**Use when:**

- Most production scenarios
- Balanced speed and stealth
- Standard web scraping

**Example:**

```python
bot = Bequests()  # Uses MEDIUM_LAYER by default
```

---

### `MAX_LAYER` - Maximum Stealth

```python
MAX_LAYER = [
    'identity',
    'ghost',
    'jitter',
    'header_order',
    'canvas',
    'cookie_warmup',
    'search_click'
]
```

**Use when:**

- Target has strong bot protection
- Getting blocked with MEDIUM_LAYER
- Scraping protected APIs

**Example:**

```python
from bequests import Bequests, MAX_LAYER

bot = Bequests().layers(MAX_LAYER).set_speed(2, 5)
```

---

### `NUCLEAR_LAYER` - Extreme Bypass

```python
NUCLEAR_LAYER = MAX_LAYER + ['nuclear']
```

**Use when:**

- All other methods failed
- Blocked by Cloudflare/strong WAF
- Need to access cached version

**Example:**

```python
from bequests import Bequests, NUCLEAR_LAYER

bot = Bequests().layers(NUCLEAR_LAYER)
```

---

## Custom Layer Combinations

You can create custom combinations:

```python
# Fast but secure
bot.layers(['identity', 'header_order', 'canvas'])

# Stealthy without search click
bot.layers(['identity', 'jitter', 'header_order', 'cookie_warmup'])

# Only behavioral simulation
bot.layers(['jitter', 'cookie_warmup'])
```

---

## Layer Escalation Strategy

Bequests automatically escalates to NUCLEAR_LAYER when blocked:

```python
bot = Bequests().layers(MEDIUM_LAYER)

# If blocked (403/429), automatically:
# 1. Waits (exponential backoff)
# 2. Rotates proxy
# 3. Switches TLS engine
# 4. Escalates to NUCLEAR_LAYER
# 5. Retries request
```

**Disable auto-escalation:**

```python
bot.set_retry_config(max_retries=0)  # No retries
```

---

## Layer Performance Comparison

| Layer | Speed | Stealth | CPU Usage | Use Case |
|-------|-------|---------|-----------|----------|
| LOWER_LAYER | ⚡⚡⚡ | 🛡️ | Low | Testing, simple sites |
| MEDIUM_LAYER | ⚡⚡ | 🛡️🛡️🛡️ | Medium | Production default |
| MAX_LAYER | ⚡ | 🛡️🛡️🛡️🛡️🛡️ | High | Protected sites |
| NUCLEAR_LAYER | 🐌 | 🛡️🛡️🛡️🛡️🛡️🛡️ | Medium | Last resort |

---

## Behavioral Simulation

Enable `imit_nav(True)` for enhanced behavioral simulation:

```python
bot = Bequests().layers(MAX_LAYER).imit_nav(True)
```

**Additional Actions:**

1. Fetches `/favicon.ico`
2. Randomly fetches `/robots.txt` or `/sitemap.xml`
3. Adds human-like delays between asset requests

**Combined with cookie_warmup:**

```python
# Maximum behavioral realism
bot = (Bequests()
    .layers(MAX_LAYER)
    .imit_nav(True)
    .set_speed(2, 5))
```

---

## Layer Selection Guide

### Choose LOWER_LAYER when:

- ✅ Site has no bot protection
- ✅ Speed is critical
- ✅ Testing/development
- ❌ Getting blocked occasionally

### Choose MEDIUM_LAYER when:

- ✅ Most production scenarios
- ✅ Standard anti-bot measures
- ✅ Need balance of speed/stealth
- ❌ Consistent blocking

### Choose MAX_LAYER when:

- ✅ Site has Cloudflare/WAF
- ✅ MEDIUM_LAYER is blocked
- ✅ Scraping at scale
- ❌ Site routes to captcha page

### Choose NUCLEAR_LAYER when:

- ✅ Everything else failed
- ✅ Need cached content
- ✅ Last resort bypass
- ❌ Need real-time/dynamic content

---

## Troubleshooting Layers

### Still Getting Blocked?

**1. Increase layer protection:**

```python
# Progress: LOWER → MEDIUM → MAX → NUCLEAR
bot.layers(NUCLEAR_LAYER)
```

**2. Add behavioral simulation:**

```python
bot.imit_nav(True).set_speed(3, 7)
```

**3. Generate browsing history:**

```python
bot.generate_noise(count=5)
```

**4. Switch TLS engine:**

```python
bot.set_engine("safari15_5")  # Try Safari instead of Chrome
```

**5. Use proxies:**

```python
bot = Bequests(proxies=["http://proxy:8080"]).layers(MAX_LAYER)
```

### Too Slow?

**1. Reduce protection:**

```python
bot.layers(LOWER_LAYER)
```

**2. Disable behavioral simulation:**

```python
bot.imit_nav(False)
```

**3. Reduce jitter:**

```python
bot.set_speed(0.3, 0.8)
```

**4. Use async:**

```python
from bequests import AsyncBequests
bot = AsyncBequests().layers(MEDIUM_LAYER)
```

---

## Advanced: Layer Internals

### How Layers Are Applied

```python
# Internal flow:
if 'jitter' in active_layers:
    time.sleep(random.uniform(min_delay, max_delay))

if 'cookie_warmup' in active_layers:
    session.get(f"{domain}/favicon.ico")

if 'header_order' in active_layers and engine == "chrome120":
    headers.update(chrome_sec_headers)

if 'search_click' in active_layers:
    referrer = f"https://www.google.com/search?q={domain}"
    url += f"?ved=0ahUKEwi{random_int}"

if 'nuclear' in active_layers:
    url = f"https://webcache.googleusercontent.com/search?q=cache:{url}"
```

### Custom Layer Example

While you can't add custom layers, you can use hooks:

```python
def my_custom_protection(url, headers, kwargs):
    # Add custom logic
    headers["X-My-Custom-Header"] = "value"
    return url, headers, kwargs

bot = Bequests().layers(MAX_LAYER).add_hook(my_custom_protection)
```

---

## Summary

**Quick Reference:**

```python
from bequests import Bequests, LOWER_LAYER, MEDIUM_LAYER, MAX_LAYER, NUCLEAR_LAYER

# Simple site
bot = Bequests().layers(LOWER_LAYER)

# Default
bot = Bequests()  # MEDIUM_LAYER

# Protected site
bot = Bequests().layers(MAX_LAYER).imit_nav(True)

# Extreme bypass
bot = Bequests().layers(NUCLEAR_LAYER)
```

**Golden Rule:** Start with LOWER, escalate only when blocked.
