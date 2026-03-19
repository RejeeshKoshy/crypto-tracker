# Market Pulse — Live Crypto Dashboard

> A real-time cryptocurrency tracking dashboard with watchlists, live price feeds, sparkline charts, and a detailed asset modal — built as a zero-dependency single-page app.

**[→ Live Demo](https://69bc335d8c45792fca96e0ff--eloquent-lamington-7253e9.netlify.app/)**

---

## Tech Stack

| Technology | What it does | Why chosen |
|---|---|---|
| **Vanilla JS (ES2022)** | Application logic | Zero build step, ships instantly, no framework overhead for a data-display app |
| **CoinGecko Public API** | Live price + sparkline data | Free tier, no API key required, returns sparkline history in one call |
| **CSS Custom Properties** | Theming (dark/light) | Native cascade variables give instant theme switching without JS repaints |
| **LocalStorage** | Watchlist + theme persistence | Synchronous, sufficient for this data shape — no IndexedDB complexity needed |
| **SVG (hand-rolled)** | 7-day sparkline charts | No chart library needed for simple polylines; keeps bundle at 0 KB |
| **Google Fonts** | Typography (Space Mono + Syne) | Loaded async, only weights used are requested |

**No build tools. No bundler. No npm. One HTML file.**

---

## Architecture

The script is organized into numbered, self-contained modules — each with a single responsibility:

```
CONFIG          → frozen constants (refresh interval, LS keys, debounce timing)
CoinGeckoService → all network calls (swap provider here without touching anything else)
Fmt             → pure formatting utilities (price, compact, percent, time)
Storage         → thin localStorage wrappers
state           → single source of truth object
DOM             → cached element references (queried once at startup)
Render          → pure HTML-string factories (no DOM access, no side-effects)
UI              → all DOM mutations (the only layer allowed to write to the DOM)
getFilteredCoins → derived/computed state (filter + search combined)
loadData()      → async fetch + state mutations
bindEvents()    → all event listeners registered once, delegation used throughout
startAutoRefresh() → setInterval wrapper
init()          → bootstrap entry point
```

---

## Features

- **Live Feed** — Top 20 coins by market cap, auto-refreshes every 60 seconds
- **Search & Filter** — Debounced (200 ms) search by name or ticker symbol
- **Watchlist** — Star/unstar assets; persists across browser sessions via LocalStorage
- **Detail Modal** — 24h high/low range bar, ATH, volume, circulating supply, market cap rank
- **Sparklines** — 7-day price trend rendered as inline SVG polylines
- **Dark/Light Mode** — CSS variable swap, preference saved to LocalStorage
- **Skeleton Loaders** — Shown on first load only; background refreshes are silent
- **Error Handling** — Rate limit (429), generic API failures, and offline detection all surface friendly banners; stale data is preserved

---

## Setup — Run Locally

No installation required. This is a static single-file app.

### Option A — Just open it

```bash
# Clone the repo
git clone https://github.com/your-username/market-pulse.git
cd market-pulse

# Open directly in your browser
open index.html          # macOS
xdg-open index.html      # Linux
start index.html         # Windows
```

### Option B — Local dev server (recommended, avoids CORS edge cases)

```bash
# Python 3 (ships with most systems)
python3 -m http.server 8080
# → http://localhost:8080

# OR Node.js
npx serve .
# → http://localhost:3000

# OR VS Code
# Install "Live Server" extension → right-click index.html → "Open with Live Server"
```

### Option C — Deploy to GitHub Pages (free hosting)

```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/YOUR_USERNAME/market-pulse.git
git push -u origin main

# In GitHub: Settings → Pages → Source: Deploy from branch → main → / (root) → Save
# Live at: https://YOUR_USERNAME.github.io/market-pulse
```

### Option D — Deploy to Netlify (drag-and-drop, no account setup)

1. Go to [netlify.com/drop](https://netlify.com/drop)
2. Drag the `market-pulse/` folder onto the page
3. Get an instant live URL

---

## API Notes

This app uses the **CoinGecko free public API** — no key required.

```
Endpoint: https://api.coingecko.com/api/v3/coins/markets
Rate limit: ~10–30 req/min on the free tier
```

If you hit the rate limit (HTTP 429), the app will display a clear error banner and preserve the last-fetched data. It will not retry automatically to avoid making the limit worse — use the Refresh button once the window resets (~1 minute).

---

## Trade-offs & What I'd Improve

### Shortcuts taken

| Shortcut | Reason |
|---|---|
| Single HTML file | Suits the scope; avoids build tooling overhead for a demo |
| `innerHTML` for rendering | Simpler than a virtual DOM for this data shape; mitigated by escaping user-controlled content |
| No pagination | CoinGecko free tier caps at 250 coins/call; top 20 is sufficient for the demo |
| Auto-refresh via `setInterval` | A `visibilitychange` listener would be more battery-friendly — skipped for scope |
| No service worker / offline cache | The error banner + stale data fallback covers the offline case adequately |

### Given more time, I would:

1. **Add a `visibilitychange` listener** — pause auto-refresh when the tab is hidden, resume when focused. Saves unnecessary API calls.
2. **Virtualize the list** — for 100+ coins, a windowed list (only rendering visible rows) would eliminate jank.
3. **Switch to WebSockets** — CoinGecko offers a paid WS feed; replacing the polling `setInterval` would give true real-time updates.
4. **Add a portfolio view** — let users input their holdings quantity and see live P&L calculated against current prices.
5. **Add unit tests** — `Fmt`, `Calc`, and `getFilteredCoins` are pure functions and trivially testable with Vitest or Jest with no DOM setup needed.
6. **Extract into modules** — split into `services/coingecko.js`, `utils/fmt.js`, `state.js`, `ui/render.js`, etc. for a real production codebase. The current single-file structure mimics this separation logically.
7. **Add a currency selector** — CoinGecko supports `vs_currency=inr|eur|gbp` etc. in the same endpoint.

---

## Browser Support

| Browser | Support |
|---|---|
| Chrome / Edge 90+ | ✅ Full |
| Firefox 90+ | ✅ Full |
| Safari 14+ | ✅ Full |
| Mobile (iOS/Android) | ✅ Responsive layout |

Uses: CSS Grid, CSS Custom Properties, `fetch`, `async/await`, optional chaining (`?.`), nullish coalescing (`??`), `Intl.NumberFormat`. No polyfills required for modern browsers.

---

## License

MIT — free to use, modify, and deploy.
