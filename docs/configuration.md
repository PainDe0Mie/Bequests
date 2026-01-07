# Configuration Guide

Learn how to configure Bequests for optimal performance and stealth.

---

## Initialization

### Basic Initialization

```python
from bequests.bequests import Bequests

# Default configuration
bot = Bequests()
```

### Custom Initialization

```python
from bequests.bequests import Bequests

bot = Bequests(
    proxies=["http://proxy1:8080", "http://proxy2:8080"],
    use_tor=False,
    logged=True,
    auto_load=True,
    timeout=30
)
```

---

## Configuration Parameters

### Network Configuration

#### Proxies

Configure proxy servers for routing requests.

```python
# Single proxy
bot = Bequests(proxies=["http://proxy.example.com:8080"])

# Multiple proxies (auto-rotation on failures)
bot = Bequests(proxies=[
    "http://proxy1.example.com:8080",
    "http://proxy2.example.com:8080",
    "http://proxy3.example.com:8080"
])

# SOCKS5 proxy
bot = Bequests(proxies=["socks5://proxy.example.com:1080"])
```

#### TOR Network

Route requests through TOR for anonymity.

```python
# Enable TOR routing
bot = Bequests(use_tor=True)

# Toggle TOR dynamically
bot.toggle_tor(True)
bot.toggle_tor(False)
```

**Requirements:**

- TOR must be installed and running
- Default TOR port: `127.0.0.1:9050`

#### Timeout

Configure request timeout in seconds.

```python
# 60 second timeout
bot = Bequests(timeout=60)

# Per-request timeout override
response = bot.get("https://slow-site.com", timeout=120)
```

---

### Behavioral Configuration

#### Logging

Enable or disable console logging.

```python
# Disable logging
bot = Bequests(logged=False)

# Enable detailed file logging
bot = (Bequests(logged=True)
    .enable_debug_log("bequests_debug.log"))
```

**Log Levels:**

- `CONFIG` - Configuration changes
- `DEPLOY` - Request execution
- `BLOCKED` - WAF blocks detected
- `NOISE` - Traffic generation
- `STORAGE` - Cookie operations
- `ERROR` - Errors and failures

#### Auto-Load Cookies

Automatically load cookies on initialization.

```python
# Auto-load from vault
bot = Bequests(auto_load=True)

# Disable auto-load
bot = Bequests(auto_load=False)
bot.load_cookies()  # Manual load
```

---

## Method Chaining

All configuration methods return `self` for chaining:

```python
bot = (Bequests()
    .layers(MAX_LAYER)
    .set_speed(2, 5)
    .set_engine("chrome120")
    .imit_nav(True)
    .toggle_auto_rotate(True)
    .set_rate_limit(10, 60)
    .enable_cache(ttl=600))
```

---

## Protection Configuration

### Protection Layers

Configure stealth layers:

```python
from bequests.bequests import (
    Bequests,
    LOWER_LAYER,
    MEDIUM_LAYER,
    MAX_LAYER,
    NUCLEAR_LAYER
)

# Use preset
bot = Bequests().layers(MAX_LAYER)

# Custom layers
bot = Bequests().layers(['identity', 'jitter', 'header_order'])
```

**See [Protection Layers Guide](protection-layers.md) for details.**

### TLS Fingerprint

Select browser fingerprint:

```python
# Chrome 120 (Windows)
bot = Bequests().set_engine("chrome120")

# Safari 15.5 (macOS)
bot = Bequests().set_engine("safari15_5")
```

**Available Engines:**

| Engine | Browser | OS | Use Case |
|--------|---------|-----|----------|
| `chrome120` | Chrome 120 | Windows | Most common |
| `safari15_5` | Safari 15.5 | macOS | Apple ecosystem |

### Speed Configuration

Configure jitter delays:

```python
# Slow and stealthy (2-5 seconds)
bot.set_speed(2.0, 5.0)

# Balanced (1-3 seconds) - Default
bot.set_speed(1.0, 3.0)

# Fast (0.5-1 second)
bot.set_speed(0.5, 1.0)
```

**Recommendations:**

- **Heavy Protection Sites:** 3-7 seconds
- **Normal Sites:** 1-3 seconds  
- **APIs/Light Protection:** 0.5-1 second

### Navigation Imitation

Enable behavioral simulation:

```python
# Enable imitation
bot.imit_nav(True)

# Disable imitation
bot.imit_nav(False)
```

**What it does:**

- Fetches `/favicon.ico`
- Randomly fetches `/robots.txt` or `/sitemap.xml`
- Adds realistic delays between assets

---

## Rate Limiting

### Configure Rate Limits

Limit request frequency:

```python
# 10 requests per 60 seconds (10/minute)
bot.set_rate_limit(10, 60)

# 1 request per second
bot.set_rate_limit(1, 1)

# 100 requests per hour
bot.set_rate_limit(100, 3600)
```

**How it works:**

- Tracks timestamps of recent requests
- Automatically waits when limit is reached
- Cleans old timestamps outside time window

**Example:**

```python
bot = Bequests().set_rate_limit(5, 60)

# First 5 requests: instant
# 6th request: waits until window resets
for i in range(10):
    response = bot.get(f"https://api.example.com/item/{i}")
```

---

## Retry Configuration

### Exponential Backoff

Configure retry strategy:

```python
bot.set_retry_config(
    max_retries=5,      # Maximum retry attempts
    backoff_factor=2,   # Exponential multiplier
    backoff_max=120     # Max wait time (seconds)
)
```

**Wait Time Calculation:**

```
wait_time = min(backoff_factor ^ retry_count, backoff_max)
```

**Examples:**

| Retry | backoff_factor=2 | backoff_max=60 | Wait Time |
|-------|------------------|----------------|-----------|
| 1 | 2^1 = 2 | 60 | 2s |
| 2 | 2^2 = 4 | 60 | 4s |
| 3 | 2^3 = 8 | 60 | 8s |
| 4 | 2^4 = 16 | 60 | 16s |
| 5 | 2^5 = 32 | 60 | 32s |
| 6 | 2^6 = 64 | 60 | 60s (capped) |

**Scenarios:**

```python
# Conservative: More retries, longer waits
bot.set_retry_config(max_retries=10, backoff_factor=2, backoff_max=300)

# Aggressive: Faster retries
bot.set_retry_config(max_retries=3, backoff_factor=1.5, backoff_max=30)

# Disable retries
bot.set_retry_config(max_retries=0)
```

### Auto Proxy Rotation

Enable proxy rotation on failures:

```python
# Enable auto-rotation
bot.toggle_auto_rotate(True)

# Disable auto-rotation
bot.toggle_auto_rotate(False)

# Manual rotation
bot.rotate_proxy()
```

**Behavior:**

When enabled and request fails (403/429):

1. Waits (exponential backoff)
2. **Rotates to next proxy**
3. Switches TLS engine
4. Escalates protection layers
5. Retries request

---

## Domain Filtering

### Whitelisting

Only allow specific domains:

```python
# Single domain
bot.whitelist_domains("api.example.com")

# Multiple domains
bot.whitelist_domains([
    "api.example.com",
    "cdn.example.com",
    "auth.example.com"
])
```

**Effect:** All requests to non-whitelisted domains return `None`.

### Blacklisting

Block specific domains:

```python
# Block tracking and ads
bot.blacklist_domains([
    "analytics.google.com",
    "ads.example.com",
    "tracker.example.com",
    "facebook.com",
    "doubleclick.net"
])
```

**Effect:** All requests to blacklisted domains return `None`.

### Combined Usage

```python
# Whitelist main domains, blacklist ads
bot = (Bequests()
    .whitelist_domains(["example.com", "api.example.com"])
    .blacklist_domains(["ads.example.com"]))

# Works
bot.get("https://api.example.com/data")

# Blocked by blacklist
bot.get("https://ads.example.com/track")

# Blocked by whitelist
bot.get("https://other-site.com")
```

---

## Caching

### Enable Response Caching

Cache GET request responses:

```python
# Enable with 5 minute TTL (default)
bot.enable_cache()

# Custom TTL
bot.enable_cache(ttl=3600)  # 1 hour
```

**How it works:**

1. First request: Fetches from server, caches response
2. Subsequent requests (within TTL): Returns cached response
3. After TTL expires: Cache entry deleted, re-fetches from server

**Example:**

```python
bot = Bequests().enable_cache(ttl=600)

# First call: fetches from server
data1 = bot.smart_json("https://api.example.com/data")  # ~1s

# Second call: instant from cache
data2 = bot.smart_json("https://api.example.com/data")  # ~0ms
```

### Clear Cache

```python
# Clear all cached responses
bot.clear_cache()
```

**Use Cases:**

- ✅ APIs with stable data
- ✅ Repeated requests to same endpoint
- ✅ Development/testing
- ❌ Real-time data
- ❌ Authenticated requests with tokens

---

## Hooks System

### Pre-Request Hooks

Modify requests before execution:

```python
def add_auth_header(url, headers, kwargs):
    headers["Authorization"] = "Bearer TOKEN123"
    return url, headers, kwargs

def add_timestamp(url, headers, kwargs):
    import time
    headers["X-Timestamp"] = str(int(time.time()))
    return url, headers, kwargs

bot = (Bequests()
    .add_hook(add_auth_header)
    .add_hook(add_timestamp))
```

**Hook Signature:**

```python
def hook_function(url, headers, kwargs):
    # Modify parameters
    return url, headers, kwargs
```

### Post-Response Hooks

Process responses after execution:

```python
def log_response(response):
    print(f"[{response.status_code}] {response.url}")

def check_rate_limit(response):
    if "X-RateLimit-Remaining" in response.headers:
        remaining = response.headers["X-RateLimit-Remaining"]
        if int(remaining) < 10:
            print(f"⚠️  Warning: Only {remaining} requests remaining!")

bot = (Bequests()
    .add_response_hook(log_response)
    .add_response_hook(check_rate_limit))
```

**Hook Signature:**

```python
def hook_function(response):
    # Process response
    pass  # No return needed
```

---

## Cookie Configuration

### Vault File Location

Configure cookie storage file:

```python
bot = Bequests()
bot.vault_file = "custom_cookies.json"

# Save to custom location
bot.save_cookies()

# Load from custom location
bot.load_cookies()
```

### Auto-Save on Success

Configure automatic cookie saving:

```python
bot = Bequests()

# Enable auto-save (default)
bot.save_on_success = True

# Disable auto-save
bot.save_on_success = False
bot.save_cookies()  # Manual save
```

---

## Statistics Configuration

### Request History Size

Configure history buffer size:

```python
bot = Bequests()

# Store last 1000 requests
bot.max_history = 1000

# Disable history
bot.max_history = 0
```

### WAF Detection

Enable/disable WAF reconnaissance:

```python
bot = Bequests()

# Enable WAF detection (default)
bot.recon_waf_active = True

# Disable WAF detection
bot.recon_waf_active = False
```

### Captcha Detection

Enable/disable captcha detection:

```python
bot = Bequests()

# Enable captcha detection (default)
bot.detect_captcha_active = True

# Disable captcha detection
bot.detect_captcha_active = False
```

---

## Complete Configuration Example

### Production Configuration

```python
from bequests.bequests import Bequests, MAX_LAYER

def add_api_key(url, headers, kwargs):
    headers["X-API-Key"] = "YOUR_API_KEY"
    return url, headers, kwargs

def log_response(response):
    import time
    print(f"[{time.strftime('%H:%M:%S')}] "
          f"{response.status_code} - {response.url}")

bot = (Bequests(
    proxies=[
        "http://proxy1.example.com:8080",
        "http://proxy2.example.com:8080",
        "http://proxy3.example.com:8080"
    ],
    use_tor=False,
    logged=True,
    auto_load=True,
    timeout=60
)
    # Protection
    .layers(MAX_LAYER)
    .set_engine("chrome120")
    .set_speed(2, 4)
    .imit_nav(True)
    
    # Network
    .toggle_auto_rotate(True)
    .set_retry_config(max_retries=5, backoff_factor=2, backoff_max=120)
    
    # Performance
    .set_rate_limit(10, 60)
    .enable_cache(ttl=600)
    
    # Filtering
    .whitelist_domains(["api.example.com", "cdn.example.com"])
    
    # Hooks
    .add_hook(add_api_key)
    .add_response_hook(log_response)
    
    # Monitoring
    .enable_debug_log("production.log"))

# Generate browsing history
bot.generate_noise(count=5)

# Configure additional settings
bot.vault_file = "production_cookies.json"
bot.max_history = 500
```

---

## Configuration Best Practices

### 1. Start Conservative

```python
# Begin with minimal protection
bot = Bequests().layers(LOWER_LAYER)

# Escalate only if blocked
if getting_blocked:
    bot.layers(MEDIUM_LAYER)
```

### 2. Match Speed to Target

```python
# Fast APIs
bot.set_speed(0.3, 0.8)

# Normal sites
bot.set_speed(1, 3)

# Heavily protected
bot.set_speed(3, 7)
```

### 3. Respect Rate Limits

```python
# Always set rate limits
bot.set_rate_limit(10, 60)  # Conservative

# Monitor API response headers
def check_limits(response):
    if "X-RateLimit-Remaining" in response.headers:
        print(f"Remaining: {response.headers['X-RateLimit-Remaining']}")

bot.add_response_hook(check_limits)
```

### 4. Use Appropriate Caching

```python
# Cache stable data
bot.enable_cache(ttl=3600)  # 1 hour

# Don't cache dynamic data
bot.enable_cache(ttl=60)  # 1 minute
```

### 5. Monitor Performance

```python
# Check stats regularly
stats = bot.get_stats()

if stats['success_rate'] < 80:
    bot.layers(MAX_LAYER)
    bot.set_speed(3, 6)
```

---

## Configuration Presets

### Quick Start Preset

```python
bot = Bequests()  # Uses MEDIUM_LAYER by default
```

### Stealth Preset

```python
bot = (Bequests()
    .layers(MAX_LAYER)
    .set_speed(3, 6)
    .imit_nav(True)
    .set_retry_config(max_retries=5))
```

### Speed Preset

```python
bot = (Bequests()
    .layers(LOWER_LAYER)
    .set_speed(0.5, 1)
    .enable_cache(ttl=3600))
```

### API Preset

```python
bot = (Bequests()
    .set_rate_limit(10, 60)
    .enable_cache(ttl=600)
    .set_retry_config(max_retries=3))
```

### Anonymous Preset

```python
bot = (Bequests(use_tor=True)
    .layers(MAX_LAYER)
    .set_speed(4, 8)
    .imit_nav(True))
```

---

## Troubleshooting Configuration

### Requests Too Slow

**Problem:** Scraping takes too long

**Solutions:**

1. Reduce jitter: `bot.set_speed(0.5, 1)`
2. Lower protection: `bot.layers(LOWER_LAYER)`
3. Disable imitation: `bot.imit_nav(False)`
4. Use async: `AsyncBequests()`

### Getting Blocked

**Problem:** Consistent 403/429 errors

**Solutions:**

1. Increase protection: `bot.layers(MAX_LAYER)`
2. Add delay: `bot.set_speed(3, 7)`
3. Enable imitation: `bot.imit_nav(True)`
4. Generate noise: `bot.generate_noise(5)`
5. Try TOR: `bot.toggle_tor(True)`

### High Memory Usage

**Problem:** Memory consumption growing

**Solutions:**

1. Reduce history: `bot.max_history = 100`
2. Clear cache: `bot.clear_cache()`
3. Disable caching: Don't call `enable_cache()`

---

## Summary

**Essential Configuration:**

```python
from bequests.bequests import Bequests, MEDIUM_LAYER

bot = (Bequests()
    .layers(MEDIUM_LAYER)       # Protection
    .set_speed(1, 3)             # Human-like delays
    .set_rate_limit(10, 60)      # Respect limits
    .enable_cache(ttl=600))      # Performance
```

**Advanced Configuration:** See [API Reference](../api-reference.md)
