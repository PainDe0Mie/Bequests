# Bequests Documentation

**Bequests** is an advanced Python HTTP client library designed for web scraping, bot detection evasion, and intelligent request handling. It provides sophisticated protection layers, WAF detection, behavioral simulation, and seamless integration with TOR and proxy networks.

## Features

- **Multi-Layer Protection System**: Configurable stealth layers including identity masking, traffic jittering, header ordering, and canvas fingerprinting
- **WAF Detection & Evasion**: Automatic detection of Cloudflare, Akamai, Imperva, AWS WAF, and DataDome with intelligent retry strategies
- **Browser Fingerprint Impersonation**: Chrome and Safari TLS/JA3 fingerprint emulation
- **Behavioral Simulation**: DOM loading simulation, cookie warmup, search referrer spoofing
- **Proxy & TOR Support**: Built-in proxy rotation and TOR network integration
- **Rate Limiting**: Configurable request rate limiting with automatic throttling
- **Response Caching**: TTL-based response caching for improved performance
- **Statistics & Monitoring**: Comprehensive request tracking and success rate analytics
- **Async Support**: Full async/await compatibility with AsyncBequests class
- **Hooks System**: Pre-request and post-response hook support for custom processing

---

## Installation

```bash
pip install bequests
```

```python
from bequests.bequests import Bequests, AsyncBequests
```

### Requirements

- Python 3.7+
- curl_cffi
- TOR (optional, for TOR network routing)

---

## Quick Start

### Basic Usage

```python
from bequests.bequests import Bequests

# Initialize client
bot = Bequests()

# Make a simple request
response = bot.get("https://example.com")

if response and response.status_code == 200:
    print(response.text)
```

### With Stealth Layers

```python
from bequests.bequests import Bequests, MAX_LAYER

bot = Bequests(logged=True)
bot.layers(MAX_LAYER).set_speed(2, 5)

response = bot.get("https://protected-site.com")
```

### Async Usage

```python
import asyncio
from bequests.bequests import AsyncBequests

async def scrape():
    bot = AsyncBequests()
    response = await bot.get("https://example.com")
    print(response.status_code)

asyncio.run(scrape())
```

---

## Configuration

### Protection Layers

Bequests provides predefined protection layer configurations:

```python
# Available layer presets
LOWER_LAYER = ['identity', 'jitter']
MEDIUM_LAYER = ['identity', 'ghost', 'jitter', 'header_order']
MAX_LAYER = ['identity', 'ghost', 'jitter', 'header_order', 'canvas', 'cookie_warmup', 'search_click']
NUCLEAR_LAYER = MAX_LAYER + ['nuclear']
```

**Layer Components:**

- `identity`: User-Agent masking
- `ghost`: Silent captcha detection
- `jitter`: Random delays between requests
- `header_order`: Browser-specific header ordering (Chrome Sec-CH-UA headers)
- `canvas`: Canvas fingerprinting headers (viewport, device memory, DPR)
- `cookie_warmup`: Pre-visit assets to establish TCP/TLS connection
- `search_click`: Simulate Google search referrer
- `nuclear`: Route requests through Google Cache

### Basic Configuration

```python
bot = Bequests(
    proxies=["http://proxy1:8080", "http://proxy2:8080"],  # Proxy list
    use_tor=False,           # Use TOR network
    logged=True,             # Enable logging
    auto_load=True,          # Auto-load cookies from vault
    timeout=30               # Request timeout in seconds
)
```

### Chainable Methods

All configuration methods return `self` for method chaining:

```python
bot = (Bequests()
    .layers(MAX_LAYER)
    .set_speed(1, 3)
    .set_engine("chrome120")
    .imit_nav(True)
    .toggle_tor(False)
    .set_rate_limit(10, 60))
```

---

## API Reference

### Core Methods

#### `Bequests(proxies=None, use_tor=False, logged=True, auto_load=True, hooks=None, response_hooks=None, timeout=30)`

Initialize a Bequests client.

**Parameters:**
- `proxies` (list): List of proxy URLs
- `use_tor` (bool): Enable TOR routing
- `logged` (bool): Enable console logging
- `auto_load` (bool): Auto-load cookies on initialization
- `hooks` (list): Pre-request hook functions
- `response_hooks` (list): Post-response hook functions
- `timeout` (int): Default request timeout

### HTTP Methods

All standard HTTP methods are supported:

```python
bot.get(url, **kwargs)
bot.post(url, **kwargs)
bot.put(url, **kwargs)
bot.delete(url, **kwargs)
bot.head(url, **kwargs)
bot.options(url, **kwargs)
bot.patch(url, **kwargs)
```

**Common kwargs:**
- `headers` (dict): Custom headers
- `params` (dict): URL parameters
- `json` (dict): JSON request body
- `data` (dict/str): Form data
- `timeout` (int): Request timeout override
- `proxies` (dict): Proxy override

#### `smart_json(url, **kwargs)`

Fetch and parse JSON with error handling:

```python
data = bot.smart_json("https://api.example.com/data")
if data:
    print(data)
```

#### `download(url, filename=None, show_progress=True, **kwargs)`

Download files with progress tracking:

```python
success = bot.download(
    "https://example.com/file.pdf",
    filename="downloads/document.pdf",
    show_progress=True
)
```

### Configuration Methods

#### `layers(layers_list)`

Set active protection layers:

```python
bot.layers(['identity', 'jitter', 'header_order'])
```

#### `set_speed(min_delay, max_delay)`

Configure jitter delay range in seconds:

```python
bot.set_speed(1.0, 3.0)  # Random delay between 1-3 seconds
```

#### `set_engine(engine_name)`

Select TLS fingerprint:

```python
bot.set_engine("chrome120")  # or "safari15_5"
```

#### `imit_nav(status)`

Enable behavioral simulation (DOM loading, asset fetching):

```python
bot.imit_nav(True)
```

#### `set_rate_limit(requests, per_seconds)`

Configure rate limiting:

```python
bot.set_rate_limit(10, 60)  # 10 requests per 60 seconds
```

#### `set_retry_config(max_retries=3, backoff_factor=2, backoff_max=60)`

Configure exponential backoff retry strategy:

```python
bot.set_retry_config(max_retries=5, backoff_factor=2, backoff_max=120)
```

#### `whitelist_domains(domains)` / `blacklist_domains(domains)`

Domain filtering:

```python
bot.whitelist_domains(["example.com", "api.example.com"])
bot.blacklist_domains(["blocked-site.com"])
```

#### `enable_cache(ttl=300)`

Enable response caching:

```python
bot.enable_cache(ttl=600)  # Cache for 10 minutes
```

### Cookie Management

#### `save_cookies(path=None)` / `load_cookies(path=None)`

Persist cookies to disk:

```python
bot.save_cookies("my_cookies.json")
bot.load_cookies("my_cookies.json")
```

#### `set_cookies(cookie_dict)` / `get_cookies()`

Programmatic cookie management:

```python
bot.set_cookies({"session": "abc123"})
cookies = bot.get_cookies()
```

#### `clear_vault()`

Clear all cookies:

```python
bot.clear_vault()
```

### Proxy Management

#### `rotate_proxy()`

Manually rotate to next proxy:

```python
bot.rotate_proxy()
```

#### `toggle_auto_rotate(status)`

Enable/disable automatic proxy rotation on failures:

```python
bot.toggle_auto_rotate(True)
```

### Stealth & Intelligence

#### `check_stealth()`

Verify if fingerprint is detected:

```python
if bot.check_stealth():
    print("Stealth verified!")
```

#### `generate_noise(count=3)`

Generate browsing history on neutral sites:

```python
bot.generate_noise(count=5)
```

#### `reset_session()`

Create fresh TLS session:

```python
bot.reset_session()
```

### Statistics & Monitoring

#### `get_stats()`

Get session statistics:

```python
stats = bot.get_stats()
print(f"Success rate: {stats['success_rate']:.2f}%")
print(f"Total requests: {stats['requests']}")
```

#### `export_stats(filepath="bequests_stats.json")`

Export statistics to file:

```python
bot.export_stats("session_report.json")
```

#### `enable_debug_log(filepath="bequests_debug.log")`

Enable detailed file logging:

```python
bot.enable_debug_log("debug.log")
```

### Hooks System

#### `add_hook(func)` / `add_response_hook(func)`

Add custom processing functions:

```python
def my_hook(url, headers, kwargs):
    headers["X-Custom"] = "value"
    return url, headers, kwargs

def response_hook(response):
    print(f"Response: {response.status_code}")

bot.add_hook(my_hook).add_response_hook(response_hook)
```

---

## Advanced Usage

### Nuclear Layer (Google Cache Routing)

For maximum stealth, route requests through Google Cache:

```python
from bequests.bequests import Bequests, NUCLEAR_LAYER

bot = Bequests().layers(NUCLEAR_LAYER)
response = bot.get("https://blocked-site.com")
```

### Multi-Proxy Setup with Auto-Rotation

```python
proxies = [
    "http://proxy1.example.com:8080",
    "http://proxy2.example.com:8080",
    "http://proxy3.example.com:8080"
]

bot = Bequests(proxies=proxies).toggle_auto_rotate(True)
```

### TOR Network Integration

```python
bot = Bequests(use_tor=True)
response = bot.get("https://example.com")
```

**Requirements:** TOR must be running on `127.0.0.1:9050`

### Rate-Limited API Scraping

```python
bot = (Bequests()
    .set_rate_limit(5, 60)  # 5 requests per minute
    .enable_cache(ttl=600))  # Cache responses for 10 minutes

for i in range(20):
    data = bot.smart_json(f"https://api.example.com/data/{i}")
    print(data)
```

### Batch Async Requests

```python
import asyncio
from bequests.bequests import AsyncBequests

async def fetch_all(urls):
    bot = AsyncBequests()
    tasks = [bot.get(url) for url in urls]
    responses = await asyncio.gather(*tasks)
    return responses

urls = ["https://example.com/page1", "https://example.com/page2"]
responses = asyncio.run(fetch_all(urls))
```

### Custom Hook Example

```python
def add_auth_token(url, headers, kwargs):
    """Add authentication token to all requests"""
    headers["Authorization"] = "Bearer YOUR_TOKEN"
    return url, headers, kwargs

def log_response(response):
    """Log all response status codes"""
    print(f"[{response.status_code}] {response.url}")

bot = (Bequests()
    .add_hook(add_auth_token)
    .add_response_hook(log_response))
```

### Domain Filtering

```python
# Only allow requests to specific domains
bot = Bequests().whitelist_domains([
    "api.example.com",
    "cdn.example.com"
])

# Block specific domains
bot.blacklist_domains(["ads.example.com"])
```

---

## Best Practices

### 1. Start with Lower Protection

Begin with `LOWER_LAYER` and increase protection only if blocked:

```python
bot = Bequests().layers(LOWER_LAYER)
```

### 2. Respect Rate Limits

Always configure rate limiting to avoid overwhelming target servers:

```python
bot.set_rate_limit(10, 60)  # Conservative rate
```

### 3. Use Cookie Persistence

Save and load cookies to maintain session state:

```python
bot = Bequests(auto_load=True)
# ... make requests
bot.save_cookies()  # Automatic on successful requests
```

### 4. Enable Caching for Repeated Requests

```python
bot.enable_cache(ttl=300)  # Cache GET requests
```

### 5. Monitor Statistics

Regularly check success rates:

```python
stats = bot.get_stats()
if stats['success_rate'] < 80:
    bot.layers(MAX_LAYER)  # Increase protection
```

### 6. Handle Errors Gracefully

```python
response = bot.get("https://example.com")
if response is None:
    print("Request failed after retries")
elif response.status_code != 200:
    print(f"Error: {response.status_code}")
```

---

## WAF Detection

Bequests automatically detects the following WAFs:

- **Cloudflare** (cf-ray header, Server: cloudflare)
- **Akamai** (x-akamai-edge, Server: akamai)
- **Imperva/Incapsula** (incapsula, visid_incap headers)
- **AWS WAF** (awselb, Server: aws)
- **DataDome** (403 status + perimetre keyword)

When a WAF is detected and a request is blocked, Bequests automatically:
1. Logs the WAF type
2. Implements exponential backoff
3. Rotates proxy (if available)
4. Switches TLS fingerprint
5. Escalates to NUCLEAR_LAYER

---

## Troubleshooting

### Issue: Requests Are Being Blocked

**Solution:**
1. Increase protection layers: `bot.layers(MAX_LAYER)`
2. Enable navigation imitation: `bot.imit_nav(True)`
3. Generate noise traffic: `bot.generate_noise(5)`
4. Try NUCLEAR_LAYER: `bot.layers(NUCLEAR_LAYER)`

### Issue: Too Slow

**Solution:**
1. Reduce jitter: `bot.set_speed(0.5, 1.0)`
2. Disable imitation mode: `bot.imit_nav(False)`
3. Use async for parallel requests
4. Enable caching: `bot.enable_cache()`

### Issue: TOR Not Working

**Solution:**
1. Verify TOR is running: `systemctl status tor`
2. Check TOR port: Default is `9050`
3. Test connection: `curl --socks5-hostname 127.0.0.1:9050 https://check.torproject.org`

### Issue: Proxy Rotation Not Working

**Solution:**
1. Enable auto-rotation: `bot.toggle_auto_rotate(True)`
2. Verify proxy format: `["http://ip:port", ...]`
3. Check proxy health manually

---

## AsyncBequests Reference

The `AsyncBequests` class provides the same API with async/await support:

```python
import asyncio
from bequests.bequests import AsyncBequests

async def main():
    bot = AsyncBequests()
    
    # All methods are async
    await bot.generate_noise(3)
    response = await bot.get("https://example.com")
    
    # Statistics
    stats = bot.get_stats()
    print(stats)
    
    # Save cookies (synchronous)
    bot.save_cookies()

asyncio.run(main())
```

**Key Differences:**
- All network methods are async (use `await`)
- Hooks can be async or sync functions
- Cookie save/load operations are synchronous
- Statistics methods are synchronous

---

## Performance Tips

### 1. Connection Pooling

Reuse the same Bequests instance across requests to benefit from connection pooling:

```python
bot = Bequests()
for url in urls:
    bot.get(url)  # Reuses connections
```

### 2. Async for I/O-Bound Tasks

Use `AsyncBequests` when making many requests:

```python
async def scrape_many():
    bot = AsyncBequests()
    tasks = [bot.get(url) for url in urls]
    return await asyncio.gather(*tasks)
```

### 3. Cache Aggressively

For APIs with stable data:

```python
bot.enable_cache(ttl=3600)  # 1 hour cache
```

### 4. Batch Similar Requests

Group requests to the same domain to maximize connection reuse.

---

## Security Considerations

### Ethical Usage

Bequests is designed for legitimate web scraping and testing purposes. Always:

- Respect `robots.txt`
- Honor rate limits
- Identify your bot in User-Agent when appropriate
- Obtain permission for private/authenticated content
- Comply with website Terms of Service

### Proxy Privacy

When using proxies:
- Use trusted proxy providers
- Avoid free proxies (security risk)
- Rotate proxies regularly
- Monitor for proxy abuse

### TOR Usage

When using TOR:
- TOR provides anonymity but is slower
- Some sites block TOR exit nodes
- Don't use TOR for high-bandwidth tasks
- Respect TOR network guidelines

---

## License

This documentation covers the Bequests library. Please refer to the project repository for licensing information.

---

## Support

For issues, questions, or contributions, please refer to the project's GitHub repository or contact the maintainers.
