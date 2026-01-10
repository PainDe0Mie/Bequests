
<img width="200" height="200" alt="bequests-logo" src="https://github.com/user-attachments/assets/4c4d8742-e48b-4c1f-a49c-db0bac076ae5" />

# Bequests

**Honestly, we're all tired of spending 3 hours debugging headers only to get slapped with a 403.** Bequests was born from a simple fact: making HTTP requests that actually fly under the radar has become a chore. This library isn't just another wrapper—it's an engine that handles the "dirty" work (TLS Fingerprinting, header ordering, hardware simulation) so you can focus on your data.

---

## Why Bequests instead of Requests??

* **TLS Fingerprinting that actually works**: Powered by `curl_cffi`, we perfectly mimic real JA3/HTTP2 fingerprints from Chrome 120 and Safari 15.5.
* **Built-in Tor Support**: Native tunneling through the Tor network for maximum anonymity.
* **Layered Protection System**: No need to reinvent the wheel. Activate `MAX` or `NUCLEAR` layers depending on how tough the WAF is.
* **Passive Recon**: The engine recognizes signatures from Cloudflare, Akamai, or Datadome and adjusts its behavior automatically.
* **Hardware Simulation**: We inject real telemetry (Canvas, RAM, Viewport) to prove to the site that you're a human on a browser, not a script in a terminal.
* **Smart Features**: Life-savers like `smart_json` (doesn't crash on weird chars) and a `vault` system to keep your sessions alive.

---

## Quick Start

### Install Bequests

```cmd
pip install bequests
```

### Standard Stealth Request
```python
from bequests.bequests import Bequests, MAX_LAYER

# Fire up the engine
bot = Bequests(logged=True)

# Max stealth + human-like navigation (favicon, robots.txt, etc.)
bot.layers(MAX_LAYER).imit_nav(True)

# Fetch data, Bequests handles the bypass
resp = bot.get("https://target-website.com")
print(resp.text)
```

### TOR Integration

```python
# Just toggle use_tor to True (assumes Tor is running on default port 9050)
bot = Bequests(use_tor=True, logged=True)
resp = bot.get("https://check.torproject.org")
```

### Async Version

```python
import asyncio
from bequests.bequests import AsyncBequests

async def main():
    bot = AsyncBequests(logged=True)
    
    # Generate some noise to warm up cookies
    await bot.generate_noise(count=3)
    
    # Clean async request
    resp = await bot.get("https://api.example.com/data")
    data = await bot.smart_json(resp.url)
    print(data)

asyncio.run(main())
```

---

### Protection Layers Guide

| Layer | Level | Features Included |
| :--- | :--- | :--- |
| **LOWER** | 1 | Identity (UA) + Jitter (random delays) |
| **MEDIUM** | 2 | LOWER + Header ordering consistency (Client Hints) |
| **MAX** | 3 | MEDIUM + Canvas/Hardware simulation + Cookie Warmup |
| **NUCLEAR** | 4 | MAX + Automatic pivot through Google Cache if still blocked |

---

## ⚙️ Under the Hood

### Session Management (Vault)
Bequests manages a `bequests_vault.json` file. Your cookies are automatically saved after each success, allowing you to maintain "warm" sessions over the long term.

### Rate Limiting & Proxies
* **Auto-Rotation**: The engine can rotate your proxies automatically upon block detection.
* **Tor Tunnel**: Simple toggle for SOCKS5h routing.
* **Limits**: Use `bot.set_rate_limit(requests=10, per_seconds=60)` to stay under the server's radar.

### Performance Tracking
To know if your tunnel is efficient, use `bot.get_stats()` to monitor real success rates and the types of WAFs encountered.

---

## License
MIT. Use it wisely.

---
**Got an idea to improve the engine?** Feel free to open an Issue or suggest a custom Hook!
