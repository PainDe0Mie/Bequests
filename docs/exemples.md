# Examples

Practical examples for common Bequests use cases.

---

## Basic Examples

### Simple GET Request

```python
from bequests.bequests import Bequests

bot = Bequests()
response = bot.get("https://httpbin.org/get")

if response and response.status_code == 200:
    print(response.text)
```

### POST with JSON

```python
from bequests.bequests import Bequests

bot = Bequests()

data = {
    "username": "user123",
    "email": "user@example.com"
}

response = bot.post("https://httpbin.org/post", json=data)
print(response.json())
```

### Download a File

```python
from bequests.bequests import Bequests

bot = Bequests()

success = bot.download(
    "https://example.com/document.pdf",
    filename="downloads/my_document.pdf"
)

if success:
    print("✓ Download complete!")
```

---

## Web Scraping

### Scrape Protected Website

```python
from bequests.bequests import Bequests, MAX_LAYER

bot = (Bequests()
    .layers(MAX_LAYER)
    .set_speed(2, 4)
    .imit_nav(True))

# Generate browsing history
bot.generate_noise(count=3)

# Scrape multiple pages
urls = [
    "https://example.com/page1",
    "https://example.com/page2",
    "https://example.com/page3"
]

for url in urls:
    response = bot.get(url)
    
    if response and response.status_code == 200:
        # Process HTML
        html = response.text
        print(f"✓ Scraped: {url}")
        
        # Extract data (use BeautifulSoup, lxml, etc.)
        # ...
    else:
        print(f"✗ Failed: {url}")

# Save cookies for next session
bot.save_cookies()
```

### Scrape with Cloudflare Bypass

```python
from bequests.bequests import Bequests, NUCLEAR_LAYER

bot = (Bequests()
    .layers(NUCLEAR_LAYER)
    .set_speed(3, 6)
    .set_retry_config(max_retries=5))

response = bot.get("https://cloudflare-protected-site.com")

if response:
    print("✓ Bypassed Cloudflare!")
    print(response.text)
```

---

## API Scraping

### Scrape Rate-Limited API

```python
from bequests.bequests import Bequests
import json

# Respect rate limit: 10 requests per minute
bot = (Bequests()
    .set_rate_limit(10, 60)
    .enable_cache(ttl=600))

results = []

for page in range(1, 11):
    data = bot.smart_json(f"https://api.example.com/items?page={page}")
    
    if data:
        results.extend(data["items"])
        print(f"✓ Page {page}: {len(data['items'])} items")
    else:
        print(f"✗ Failed on page {page}")
        break

# Save results
with open("results.json", "w") as f:
    json.dump(results, f, indent=2)

print(f"\n Total items: {len(results)}")
```

### API with Authentication

```python
from bequests.bequests import Bequests

def add_api_key(url, headers, kwargs):
    headers["X-API-Key"] = "your_api_key_here"
    return url, headers, kwargs

bot = (Bequests()
    .add_hook(add_api_key)
    .set_rate_limit(5, 60))

# All requests will include API key
data = bot.smart_json("https://api.example.com/protected/data")
print(data)
```

---

## Proxy Usage

### Rotate Through Multiple Proxies

```python
from bequests.bequests import Bequests

proxies = [
    "http://proxy1.example.com:8080",
    "http://proxy2.example.com:8080",
    "http://proxy3.example.com:8080",
    "http://proxy4.example.com:8080"
]

bot = (Bequests(proxies=proxies)
    .toggle_auto_rotate(True)
    .set_retry_config(max_retries=5))

urls = [f"https://httpbin.org/ip" for _ in range(10)]

for i, url in enumerate(urls):
    response = bot.get(url)
    
    if response:
        ip = response.json()["origin"]
        print(f"Request {i+1}: IP = {ip}")
    
    # Manually rotate every 3 requests
    if (i + 1) % 3 == 0:
        bot.rotate_proxy()
```

### Check Proxy IP

```python
from bequests.bequests import Bequests

bot = Bequests(proxies=["http://your-proxy:8080"])

response = bot.get("https://httpbin.org/ip")

if response:
    data = response.json()
    print(f"Your IP: {data['origin']}")
```

---

## TOR Network

### Anonymous Scraping with TOR

```python
from bequests.bequests import Bequests, MAX_LAYER

bot = (Bequests(use_tor=True)
    .layers(MAX_LAYER)
    .set_speed(3, 6))

# Check TOR connection
response = bot.get("https://check.torproject.org")

if response and "Congratulations" in response.text:
    print("✓ Connected through TOR!")
    
    # Scrape anonymously
    response = bot.get("https://example.com/sensitive-data")
    print(response.text)
else:
    print("✗ TOR connection failed")
```

### Rotate TOR Circuit

```python
from bequests.bequests import Bequests
import time

bot = Bequests(use_tor=True)

for i in range(5):
    response = bot.get("https://httpbin.org/ip")
    
    if response:
        ip = response.json()["origin"]
        print(f"Request {i+1}: IP = {ip}")
    
    # Reset session to get new TOR circuit
    bot.reset_session()
    time.sleep(2)
```

---

## Cookie Management

### Login and Save Session

```python
from bequests.bequests import Bequests

bot = Bequests(auto_load=True)

# Login
login_data = {
    "username": "user@example.com",
    "password": "secure_password"
}

response = bot.post("https://example.com/login", json=login_data)

if response and response.status_code == 200:
    print("✓ Login successful!")
    
    # Cookies automatically saved
    bot.save_cookies("session.json")
    
    # Access protected page
    dashboard = bot.get("https://example.com/dashboard")
    print(dashboard.text)
```

### Resume Session

```python
from bequests.bequests import Bequests

# Load previous session
bot = Bequests()
bot.load_cookies("session.json")

# Access protected pages without logging in again
response = bot.get("https://example.com/dashboard")
print(response.text)
```

### Share Cookies Between Scripts

```python
# Script 1: Login and save
from bequests.bequests import Bequests

bot = Bequests()
bot.post("https://example.com/login", json={"user": "me", "pass": "secret"})
bot.save_cookies("shared_session.json")

# Script 2: Load and use
from bequests.bequests import Bequests

bot = Bequests()
bot.load_cookies("shared_session.json")
bot.get("https://example.com/protected-page")
```

---

## Async Examples

### Concurrent Requests

```python
import asyncio
from bequests.bequests import AsyncBequests

async def fetch_all(urls):
    bot = AsyncBequests()
    
    # Create tasks for all URLs
    tasks = [bot.get(url) for url in urls]
    
    # Execute concurrently
    responses = await asyncio.gather(*tasks)
    
    return responses

# Scrape 10 pages concurrently
urls = [f"https://example.com/page{i}" for i in range(1, 11)]
responses = asyncio.run(fetch_all(urls))

for i, resp in enumerate(responses):
    if resp and resp.status_code == 200:
        print(f"✓ Page {i+1} scraped")
```

### Async with Rate Limiting

```python
import asyncio
from bequests.bequests import AsyncBequests

async def scrape_api():
    bot = (AsyncBequests()
        .set_rate_limit(5, 60)
        .enable_cache(ttl=600))
    
    results = []
    
    for page in range(1, 21):
        data = await bot.smart_json(
            f"https://api.example.com/items?page={page}"
        )
        
        if data:
            results.extend(data["items"])
            print(f"✓ Page {page}")
    
    return results

results = asyncio.run(scrape_api())
print(f"Total items: {len(results)}")
```

### Async with Hooks

```python
import asyncio
from bequests.bequests import AsyncBequests

async def async_auth_hook(url, headers, kwargs):
    # Can be async
    headers["Authorization"] = "Bearer TOKEN"
    return url, headers, kwargs

def sync_log_hook(response):
    # Can be sync
    print(f"[{response.status_code}] {response.url}")

async def main():
    bot = (AsyncBequests()
        .add_hook(async_auth_hook)
        .add_response_hook(sync_log_hook))
    
    response = await bot.get("https://api.example.com/data")
    print(response.json())

asyncio.run(main())
```

---

## Monitoring & Statistics

### Track Success Rate

```python
from bequests.bequests import Bequests, MAX_LAYER

bot = Bequests().layers(MAX_LAYER)

urls = [f"https://example.com/page{i}" for i in range(1, 101)]

for url in urls:
    response = bot.get(url)
    
    # Check progress every 10 requests
    if len(bot.request_history) % 10 == 0:
        stats = bot.get_stats()
        print(f"\nProgress: {stats['requests']} requests")
        print(f"Success Rate: {stats['success_rate']:.1f}%")
        print(f"Blocked: {stats['blocked']}")
        print(f"Captchas: {stats['captcha']}")

# Final report
bot.export_stats("scrape_report.json")
```

### Real-Time Monitoring

```python
from bequests.bequests import Bequests
import time

def log_response(response):
    stats = bot.get_stats()
    print(f"[{time.strftime('%H:%M:%S')}] "
          f"{response.status_code} | "
          f"Success: {stats['success_rate']:.1f}%")

bot = Bequests().add_response_hook(log_response)

urls = ["https://example.com/page1", "https://example.com/page2"]
for url in urls:
    bot.get(url)
```

---

## Domain Filtering

### Whitelist Specific Domains

```python
from bequests.bequests import Bequests

# Only allow API and CDN
bot = Bequests().whitelist_domains([
    "api.example.com",
    "cdn.example.com"
])

# This works
bot.get("https://api.example.com/data")

# This is blocked
bot.get("https://other-site.com")  # Returns None
```

### Block Tracking Domains

```python
from bequests.bequests import Bequests

# Block ads and tracking
bot = Bequests().blacklist_domains([
    "analytics.google.com",
    "ads.example.com",
    "tracker.example.com",
    "facebook.com"
])

# Main request goes through
response = bot.get("https://example.com")
```

---

## Advanced Examples

### Multi-Step Authentication

```python
from bequests.bequests import Bequests

bot = Bequests()

# Step 1: Get CSRF token
response = bot.get("https://example.com/login")
csrf_token = extract_csrf(response.text)  # Your extraction logic

# Step 2: Login with token
login_data = {
    "username": "user@example.com",
    "password": "password123",
    "csrf_token": csrf_token
}

response = bot.post("https://example.com/login", data=login_data)

# Step 3: Access protected resource
if response.status_code == 200:
    bot.save_cookies()
    data = bot.get("https://example.com/api/user/data")
    print(data.json())
```

### Retry with Different Strategies

```python
from bequests.bequests import Bequests, MAX_LAYER, NUCLEAR_LAYER

def smart_scrape(url, max_attempts=3):
    strategies = [
        ("Medium", None, False),
        ("Max", MAX_LAYER, True),
        ("Nuclear", NUCLEAR_LAYER, True)
    ]
    
    bot = Bequests()
    
    for i, (name, layers, imitation) in enumerate(strategies):
        if i >= max_attempts:
            break
            
        print(f"Attempt {i+1}: {name} protection")
        
        if layers:
            bot.layers(layers)
        if imitation:
            bot.imit_nav(True)
        
        response = bot.get(url)
        
        if response and response.status_code == 200:
            print(f"✓ Success with {name}!")
            return response.text
        
        print(f"✗ Failed with {name}")
    
    return None

html = smart_scrape("https://protected-site.com")
```

### Intelligent Pagination

```python
from bequests.bequests import Bequests

bot = Bequests().set_rate_limit(10, 60)

def scrape_all_pages(base_url):
    page = 1
    all_items = []
    
    while True:
        data = bot.smart_json(f"{base_url}?page={page}")
        
        if not data or not data.get("items"):
            break
        
        items = data["items"]
        all_items.extend(items)
        
        print(f"✓ Page {page}: {len(items)} items")
        
        # Check if last page
        if page >= data.get("total_pages", page):
            break
        
        page += 1
    
    return all_items

items = scrape_all_pages("https://api.example.com/items")
print(f"\nTotal items scraped: {len(items)}")
```

---

## Debugging Examples

### Enable Debug Logging

```python
from bequests.bequests import Bequests, MAX_LAYER

bot = (Bequests()
    .layers(MAX_LAYER)
    .enable_debug_log("debug.log"))

response = bot.get("https://example.com")

# Check debug.log for detailed info
```

### Test Stealth Level

```python
from bequests.bequests import Bequests, LOWER_LAYER, MEDIUM_LAYER, MAX_LAYER

def test_stealth(layer, name):
    bot = Bequests().layers(layer)
    
    if bot.check_stealth():
        print(f"✓ {name}: Stealth verified")
    else:
        print(f"✗ {name}: Suspicious fingerprint")

test_stealth(LOWER_LAYER, "Lower")
test_stealth(MEDIUM_LAYER, "Medium")
test_stealth(MAX_LAYER, "Max")
```

### Compare Engines

```python
from bequests.bequests import Bequests

def test_engine(engine_name):
    bot = Bequests().set_engine(engine_name)
    response = bot.get("https://httpbin.org/headers")
    
    if response:
        headers = response.json()["headers"]
        print(f"\n{engine_name.upper()}:")
        print(f"  User-Agent: {headers.get('User-Agent')}")
        print(f"  Status: {response.status_code}")

test_engine("chrome120")
test_engine("safari15_5")
```

---

## Production Examples

### Complete Scraping Pipeline

```python
from bequests.bequests import Bequests, MAX_LAYER
import json
import time

class Scraper:
    def __init__(self):
        self.bot = (Bequests()
            .layers(MAX_LAYER)
            .set_speed(2, 4)
            .set_rate_limit(10, 60)
            .enable_cache(ttl=600)
            .imit_nav(True))
        
        # Generate browsing history
        self.bot.generate_noise(count=5)
    
    def scrape(self, urls):
        results = []
        
        for url in urls:
            print(f"Scraping: {url}")
            
            response = self.bot.get(url)
            
            if response and response.status_code == 200:
                # Process data
                data = self.extract_data(response.text)
                results.append({
                    "url": url,
                    "data": data,
                    "timestamp": time.time()
                })
                print(f"✓ Success")
            else:
                print(f"✗ Failed")
        
        return results
    
    def extract_data(self, html):
        # Your extraction logic
        return {"title": "Extracted data"}
    
    def save_results(self, results, filename):
        with open(filename, "w") as f:
            json.dump(results, f, indent=2)
        
        # Save session
        self.bot.save_cookies()
        
        # Export stats
        self.bot.export_stats("stats.json")

# Usage
scraper = Scraper()
urls = ["https://example.com/page1", "https://example.com/page2"]
results = scraper.scrape(urls)
scraper.save_results(results, "results.json")
```

---

## Error Handling

### Robust Error Handling

```python
from bequests.bequests import Bequests

bot = Bequests()

try:
    response = bot.get("https://example.com")
    
    if response is None:
        print("Request failed after retries")
    elif response.status_code == 200:
        print("Success!")
        print(response.text)
    elif response.status_code == 403:
        print("Blocked by WAF")
    elif response.status_code == 429:
        print("Rate limited")
    else:
        print(f"HTTP Error: {response.status_code}")
        
except KeyboardInterrupt:
    print("\nInterrupted by user")
    bot.save_cookies()
except Exception as e:
    print(f"Unexpected error: {e}")
finally:
    # Always save state
    bot.save_cookies()
    stats = bot.get_stats()
    print(f"Final success rate: {stats['success_rate']:.1f}%")
```

---

## Performance Optimization

### Batch Requests with Caching

```python
from bequests.bequests import Bequests

bot = (Bequests()
    .enable_cache(ttl=3600)
    .set_rate_limit(20, 60))

# First pass: populate cache
urls = [f"https://api.example.com/item/{i}" for i in range(100)]

for url in urls:
    bot.get(url)

# Second pass: instant from cache
for url in urls:
    response = bot.get(url)  # Instant from cache
    # Process response...
```

---

## Summary

These examples cover:

- ✅ Basic HTTP operations
- ✅ Web scraping techniques
- ✅ API interaction
- ✅ Proxy and TOR usage
- ✅ Cookie management
- ✅ Async operations
- ✅ Monitoring and statistics
- ✅ Error handling
- ✅ Production-ready pipelines

Choose the examples that match your use case and adapt them to your needs!
