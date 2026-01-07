# Welcome to Bequests

<div align="center">

**Advanced Python HTTP Client**

[![Python](https://img.shields.io/badge/python-3.7+-blue.svg)](https://www.python.org/downloads/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![GitHub Stars](https://img.shields.io/github/stars/PainDe0Mie/Bequests.svg)](https://github.com/PainDe0Mie/Bequests)

</div>

## What is Bequests???

**Bequests** is a sophisticated Python HTTP client library designed for advanced web scraping, bot detection evasion, and intelligent request handling. It provides multi-layer protection systems, WAF detection, behavioral simulation, and seamless integration with TOR and proxy networks.

## Key Features

-  **Multi-Layer Protection** - Configurable stealth layers (identity, jitter, header ordering, canvas fingerprinting)
-  **WAF Detection** - Automatic detection of Cloudflare, Akamai, Imperva, AWS WAF, DataDome
-  **Browser Impersonation** - Chrome and Safari TLS/JA3 fingerprint emulation
-  **Behavioral Simulation** - DOM loading, cookie warmup, search referrer spoofing
-  **Proxy & TOR Support** - Built-in proxy rotation and TOR network integration
-  **Async Support** - Full async/await compatibility
-  **Statistics & Monitoring** - Comprehensive request tracking and analytics
-  **Hooks System** - Pre-request and post-response hooks for custom processing
-  **Response Caching** - TTL-based caching for improved performance
-  **Rate Limiting** - Configurable rate limiting with automatic throttling

## Quick Example

```python
from bequests import Bequests, MAX_LAYER

# Initialize with maximum protection
bot = Bequests().layers(MAX_LAYER).set_speed(2, 5)

# Make a stealth request
response = bot.get("https://protected-site.com")

if response and response.status_code == 200:
    print(response.text)
    
# Save session cookies
bot.save_cookies()
```

## Why Bequests??

Traditional HTTP libraries like `requests` are easily detected by modern anti-bot systems. Bequests solves this by:

- **TLS Fingerprint Spoofing** - Uses curl-cffi to mimic real browser TLS fingerprints
- **Intelligent Retry Logic** - Exponential backoff with automatic proxy rotation
- **WAF Bypass Strategies** - Nuclear layer routes requests through Google Cache
- **Human-Like Behavior** - Random jitter, DOM simulation, cookie warmup
- **Session Persistence** - Maintains cookies and connection state across requests
