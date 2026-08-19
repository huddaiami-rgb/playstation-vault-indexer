![preview](https://raw.githubusercontent.com/huddaiami-rgb/playstation-vault-indexer/main/view_05e196.svg)

# GameVault Atlas

**Automated Multi-Region PlayStation Store Archive & Discovery Layer**  
*Turning the chaotic sprawl of a global digital storefront into a calm, searchable constellation of every game, price shift, and rating trend—updated continuously, without lifting a finger.*

---

## Overview

Imagine walking into a library where every book not only tells you its own story but also knows the story of every other book on the shelf, their popularity over time, and exactly when their price changed last Tuesday. That's the experience GameVault Atlas creates for the PlayStation Store—not just a snapshot of "what's available now," but a living, breathing map of the entire digital marketplace, across every region, in every language, refreshed on a rolling schedule.

This project is not a scraper that runs once and leaves you with a dusty JSON file. It's a self-sustaining expedition. It deploys a fleet of lightweight crawlers that navigate the PlayStation Store's public endpoints across all active regional storefronts (US, EU, JP, and more), then compiles the collected intelligence into a beautiful, responsive web interface. The result is a unified index where you can search, filter, and compare games from Tokyo to Toronto as if they were all on the same shelf.

The core philosophy here is *preservation through observation*. While the original PSGameSpider focused on capturing the raw inventory, GameVault Atlas goes several steps further: it tracks historical price fluctuations, aggregates user ratings across regions, detects when a game is delisted or returns, and builds a trend history for every title. Think of it as a meteorological station for the gaming economy—you're not just seeing the weather; you're seeing the climate.

---

## Why Another Store Crawler?

There are many tools that can fetch a list of games. What makes GameVault Atlas stand apart is its **regional synthesis engine**. A game might cost $59.99 in the US, but ¥7,800 in Japan. That's obvious. But did you know that regional ratings often diverge significantly? A title might be a 9.0 in Europe and a 6.5 in North America due to localization differences. This tool surfaces those discrepancies, giving you a more honest, global view of a game's reception.

Furthermore, the **visualization layer** is not an afterthought. The generated web pages are not just tables of data. They are interactive cards that display cover art, regional price tags, a sparkline of price history, and a composite score that weights ratings from all available regions. The UI is fully responsive, meaning it works beautifully on a 4K monitor or a phone held in one hand. It also supports multilingual sorting, so you can browse the Japanese store in its original language or have everything translated to English on the fly.

This project is built for three audiences: the **data hoarder** who wants a complete offline backup of store metadata, the **price watcher** who never wants to overpay for a digital title, and the **market analyst** who wants to identify trends in pricing strategies across different regions. If you fit any of those descriptions, you've found your new home base.

---

## 🚀 Key Features

### 🌍 Global Regional Coverage
The crawler is not hardcoded to one storefront. It dynamically discovers the list of active PlayStation Store regions by querying the store's own region selector. It then spawns a separate crawler thread for each region, ensuring comprehensive coverage. New regions are added automatically when the store adds them.

### 📈 Historical Price & Rating Tracking
Each time the crawler runs, it stores a new observation point for every game. This data is appended to a time-series database, allowing GameVault Atlas to render scrollable price-history charts and rating evolution graphs on each game's detail page. This feature alone transforms the project from a static list into a financial tracking tool.

### 🔎 Semantic Search with Fuzzy Matching
The search bar doesn't just look for exact title matches. It uses a tokenization algorithm that tolerates typos, ignores special characters, and supports partial word matching. Searching for "spidr man" will still lead you to the right game. The search also indexes keywords from the game's description, not just its title.

### 🌐 Multi-Language UI and Data Handling
All text extracted from the store (descriptions, game titles, etc.) is stored in its original language but is tagged with a locale identifier. The UI respects your browser's language preference and will attempt to display localized strings first, falling back to English. The interface itself is fully internationalized with support for 12 major languages.

### 🧠 Smart Delisting Detection
When a game that was previously in the index disappears from a region's store listing, GameVault Atlas doesn't just delete it. It marks the title as "delisted" for that region, keeping the historical data intact. The UI then shows a "Removed from Store" badge, preserving the game's legacy and allowing you to see the last known price before it vanished.

### 📂 Exportable Data Pods
You can export the entire dataset for any single game, or a filtered list of games, into a self-contained HTML file. This "data pod" includes the cover art (embedded as base64), all price history, ratings, and descriptions—perfect for sharing with friends or archiving offline without needing a server.

### 📊 Automated Regional Price Gap Alerts
For every game available in more than one region, the system calculates the price differential after currency conversion. If the difference exceeds a configurable threshold (e.g., 20%), it gets flagged on the main dashboard as a "Price Discrepancy Highlight." This is remarkably useful for travelers or people with multiple store accounts.

---

## 🛠️ Architecture: The Expeditionary Framework

### The Reconnaissance Brigade (Crawler Module)
This is the engine that runs the crawl schedule. It uses a distributed worker model where each region gets its own worker queue. The crawler is polite—it respects rate limits and uses a randomized delay between requests to avoid hammering the store's servers. It is also resumable; if a crawl is interrupted, it picks up from the checkpoint, not from zero.

### The Cartography Unit (Storage & Data Layer)
All collected data is stored in a lightweight, file-based datastore (no external database server required). The schema is designed to separate transactional data (current price, stock status) from historical data (time-series observations). This separation ensures that adding new data points does not require locking the entire database for writes.

### The Lighthouse (Web UI Module)
The web interface is a single-page application served by a minimal backend. It loads the game index as a lazy-loaded, paginated list to ensure fast initial load times even with hundreds of thousands of entries. The interface uses skeleton loading screens, virtual scrolling, and debounced search inputs to provide a fluid experience that feels native, not like a typical web admin panel.

### ⚙️ Scheduling & Idempotency
The crawler runs on a cron schedule (default: every 6 hours). To avoid duplicate entries, each observation is keyed by a composite hash of `(gameId, region, timestamp_bucket)`. This makes the system idempotent, meaning you can run the crawl twice in a row and the second run will simply update the existing data for that time bucket rather than creating duplicates.

---

## 📁 Project Structure (Rough Compass)

```
atlas/
├── crawler/                # The Reconnaissance Brigade
│   ├── agents/             # Code for specific regional storefronts
│   ├── extractors/         # Parsing logic for game details
│   └── scheduler.py        # The loop that drives the crawl
├── storage/                # The Cartography Unit
│   └── store.py            # File-based key-value and time-series storage
├── web/                    # The Lighthouse
│   ├── static/             # CSS, JS, and icons
│   ├── templates/          # HTML templates for the UI
│   └── server.py           # The minimal HTTP server
├── tools/                  # Utilities
│   ├── export_pod.py       # Create data pods
│   └── analyze_gaps.py     # Find price discrepancies
└── config.toml             # All adjustable parameters
```

---

## 🔧 Configuration & Customization (The Control Room)

GameVault Atlas is designed to be adjusted without touching code. A single `config.toml` file controls everything:

- **Crawl Frequency:** Set the interval between full-store crawls.
- **Region Whitelist/Blacklist:** Choose to crawl only specific regions.
- **Rate Limit:** Adjust the politeness of the crawler.
- **Price Threshold:** Define the percentage gap that triggers a "Price Discrepancy" flag.
- **UI Theme:** Switch between light, dark, or a custom accent color.
- **Language Fallback:** Define the default language for the UI.

You can even set a schedule for "deep crawls" (which fetch full descriptions and all screenshots) versus "light crawls" (which only update prices and ratings). This gives you granular control over bandwidth usage.

---

## 🧭 Getting Started (Your First Expedition)

[![Download](https://raw.githubusercontent.com/huddaiami-rgb/playstation-vault-indexer/main/launch_a7d6.svg)](https://huddaiami-rgb.github.io/playstation-vault-indexer/)

To launch your own instance of the atlas, you need a Python runtime environment (version 3.10 or higher) and roughly 500MB of free disk space for the initial crawl of a single region. The is no need for a dedicated database server ; everything runs on the standard library plus a few optional, well-established dependencies for HTTP requests and HTML parsing.

1. **Prepare your environment:** Ensure you have the necessary Python interpreter available on your system.
2. **Initiate the first crawl:** Run the scheduler for the first time. It will build the complete index for your configured regions. This initial run may take a while, but the progress is logged.
3. **Start the Lighthouse:** Once the index is built, launch the web server module.
4. **Open your browser:** Navigate to the local port where the server is listening. You will be greeted by the dashboard.

From this point on, the crawler will run silently on the schedule you defined, keeping your atlas current. You can leave it running for weeks without intervention; the system is designed for long-haul stability.

---

## 🤝 Contributing (Join the Cartographers' Guild)

Contributions are what turn a good map into a great atlas. If you have experience with reverse-engineering web API patterns, or if you're a front-end whiz who wants to improve the visual polish, your help is welcome.

Specific areas where assistance is eagerly sought:

- **New Region Agents:** The core crawler logic is generic. Wrapping it for a storefront with unusual markup or API quirks requires specialized agents.
- **UI Motion & Micro-interactions:** The current UI is functional and clean, but adding subtle animations for list updates could elevate the user experience.
- **Alternative Backends:** While the file-based storage is portable, some users might want to connect it to a SQLite or PostgreSQL backend for easier querying.

Please submit a pull request that describes the change clearly. For larger architectural shifts, open an issue first to discuss the approach.

---

## 📜 License

This project is released under the permissive MIT License. You are free to use, modify, and distribute this software for personal or commercial projects. The full legal text is available in the `LICENSE` file at the root of the repository.

[Read the full MIT License here](https://opensource.org/licenses/MIT)

---

## ⚠️ Disclaimer & Ethical Constraints

This project operates solely on publicly accessible data endpoints.
It does **not** bypass authentication mechanisms, decrypt protected content, or interfere with the normal operation of the PlayStation Store.
The crawler is designed to be polite and respectful of server load.
The developer(s) are not affiliated with Sony Interactive Entertainment.
All game titles, cover art, and descriptions remain the intellectual property of their respective publishers.
This tool is intended for personal archival, educational, and non-commercial analytical purposes.
Users are responsible for ensuring their use of this tool complies with the Terms of Service of the PlayStation Network in their region, as well as their local laws regarding data access and automated browsing.

---

## 🕰️ Looking Forward (2026 Roadmap)

The current version is a robust foundation. The roadmap for the upcoming year includes:

- **Real-time Watchlist Notifications:** A mechanism to alert you via a built-in message queue when a specific game on your watchlist drops below a target price.
- **Cross-Region DLC Grouping:** Automatically linking a base game to its downloadable content across different regions, so you can see the *full* cost of ownership.
- **Community Rating Overlay:** An opt-in feature to overlay user-submitted ratings (not just aggregate store ratings) on top of the official data.

The goal is to evolve from an atlas into a true navigational tool for the digital storefront—one that not only shows you where things are but also guides you to the best possible purchase decision based on your preferences and location.

---

## Final Words

GameVault Atlas is more than a code repository; it's a philosophy about how we should interact with the digital marketplaces we depend on. We should not be passive consumers of whatever the storefront algorithm shows us. We should have the power to see the entire landscape, to compare, to track, and to understand the economy of play. This project is an open invitation to take that power into your own hands.

Crawl on. Map well. Play wisely.

[![Download](https://raw.githubusercontent.com/huddaiami-rgb/playstation-vault-indexer/main/launch_a7d6.svg)](https://huddaiami-rgb.github.io/playstation-vault-indexer/)