# 🐍 BeQuests

**BeQuests** is a powerful, stealth-focused HTTP engine for Python built on top of `curl_cffi`. It is designed to bypass modern web protections like Cloudflare, Akamai, and Datadome by mimicking real browser fingerprints and behavior.

---

## Key Features

* **Advanced Fingerprinting**: Uses real TLS/JA3 fingerprints from Chrome 120 and Safari 15.5.
* **Stealth Layers**: Multiple protection levels ranging from `LOWER` to `NUCLEAR` (routing via Google Cache).
* **Behavioral Simulation**: Mimics human-like behavior including random jitter, search engine referrals, and asset pre-loading.
* **Smart Vault**: Automatically manages and persists cookies across sessions in a local JSON vault.
* **Async Support**: Built-in `AsyncBequests` class for high-performance concurrent operations.
* **Network Flexibility**: Native support for Tor and automatic proxy rotation.

---

## Quick Start

### Installation
```bash
pip install bequests
```
