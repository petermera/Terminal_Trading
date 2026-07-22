# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

`Terminal_Trading` is a **single-file, front-end-only interactive trading terminal** called **VOLT TERMINAL** — a browser dashboard that *simulates* live European electricity (power) markets. There is no backend, no build step, no package manager, and no tests. Everything runs client-side from static HTML with an inline `<style>` block and an inline vanilla-JS `<script>`.

All prices are generated locally with `Math.random()` (mean-reverting drift around a `base` value). Nothing connects to a real exchange — the "LIVE SIM" badge is literal. Do not treat any number here as market data.

## Repository layout — two very different things live here

1. **The actual app** (root):
   - `global_electricity_trading_platform.html` — the canonical, self-contained application (~465 lines).
   - `README.md` — a `# Terminal_Trading` H1 heading followed by a **verbatim copy of the HTML file's body**. It exists so the app renders when the README is previewed as HTML.

2. **An unrelated scaffold** (`apps/`): a framework-neutral placeholder tree (`pages/`, `sections/`, `documentation/`, `assets/`) containing only README stubs and `.gitkeep` files. **It is not wired to the trading app** and shares no code with it — treat it as an aspirational template for a future multi-page site, not part of the running product. Don't assume the app imports anything from `apps/`.

### ⚠️ Critical convention: the app is duplicated

`README.md` (everything after its first `# Terminal_Trading` line) and `global_electricity_trading_platform.html` are **byte-identical**. Any change to the app must be applied to **both files** or they drift out of sync. Verify with:

```bash
diff <(tail -n +2 README.md) global_electricity_trading_platform.html   # must print nothing
```

When editing, change one file, then mirror the exact change into the other (remember `README.md` has the extra H1 title line at the top).

## Running / previewing

There is nothing to build or install. To view the app, open the HTML in a browser, e.g.:

```bash
python3 -m http.server 8000   # then open http://localhost:8000/global_electricity_trading_platform.html
```

**Theming caveat:** the CSS uses custom properties that are **not defined anywhere in this repo** — `var(--color-background-primary)`, `var(--color-text-secondary)`, `var(--color-border-tertiary)`, `var(--font-mono)`, `var(--border-radius-md)`, etc. These tokens are expected to be supplied by the **host environment** that embeds the page (e.g. a Claude Artifact or a design-system host). Opened as a bare file with no host, layout works but themed colors/fonts fall back to browser defaults and it looks unstyled. This is expected, not a bug. Only **semantic P&L colors are hardcoded** as hex literals: green (`#3B6D11`, `#639922`) for up/buy, red (`#A32D2D`) for down/sell, blue (`#185FA5`) for accents/selection.

The one external runtime dependency is **Chart.js 4.4.1**, loaded from the cdnjs CDN via `<script src=...>`, so previewing requires network access.

## Application architecture

All logic is one inline script. Understanding it means understanding four things: the market data definitions, the mutable state, the simulation tick, and the render pipeline.

### Market data (static definitions)
Three arrays define the tradable instruments, one per tab:
- `DA_MARKETS` — Day-Ahead (EPEX, OMIE, Nord Pool, N2EX…).
- `IDA_MARKETS` — Intraday.
- `BAL_MARKETS` — Balancing/reserve products (aFRR, mFRR, FCR, RR, Imbalance), which additionally carry a `product` field.

Each entry is `{ id, name, region, base, vol, product? }` where `base` is the anchor price the simulation reverts toward and `vol` scales the noise. `SPREAD_PAIRS` defines cross-market spreads (e.g. `DE–FR`) by referencing two market `id`s.

### Mutable state (module-level globals)
`markets{}` (live price/prev/change keyed by id), `priceHistory{}` (per-id array feeding sparklines and the main chart), `positions[]` (seeded with 3 open positions), `tradeLog[]`, session stats (`sessionHigh/Low/Vol/VwapSum/VwapVol`), plus `currentTab` and `selectedMarket`.

### Simulation loop
`initMarkets(list)` seeds prices and a 24-point history. `tickPrices()` advances every market by mean-reverting drift toward `base` plus random noise and records history. Two `setInterval`s drive the app after a `setTimeout` bootstrap: `fullRender()` every **1800 ms** and `updateClock()` every **1000 ms** (UTC).

### Render pipeline
`fullRender()` calls the pure-DOM render functions in order: `renderGrid` (market cards + inline SVG sparklines), `renderOrderBook` (synthetic bid/ask depth around mid), `renderPositions` (recomputes live P&L), `renderSpreads`, `renderKPIs`, `renderTicker`, `renderTradeLog`, and `updateChart` (pushes latest history into the Chart.js instance created once by `initChart`).

### Interaction handlers (called from inline `onclick`/`oninput`)
`setTab(tab, el)` switches the visible market set, `selectMarket(m)` changes the focused instrument (chart, order book, session stats reset), and `executeTrade(side)` reads the qty/limit inputs, applies simulated slippage, appends to `tradeLog` and `positions`, and updates KPIs. These are global functions invoked directly from HTML attributes — there is no framework, module system, or event-delegation layer.

## Conventions when editing the app

- **Vanilla JS only**, no framework, no ES modules, no bundler. New behavior is a global function wired via an inline HTML handler, matching the existing style.
- **Dense, single-statement functions** with 2-space indentation; template literals build HTML strings assigned to `innerHTML`.
- **Prefer host CSS variables** for anything visual; reserve raw hex for the semantic up/down/buy/sell/accent palette above.
- Units are **€/MWh**, times are **UTC**, and the aesthetic is monospace terminal — keep new UI consistent with that.
- After any app edit, re-run the `diff` check above so `README.md` and the HTML stay identical.

## Git workflow

Branch-based; changes land on `main` via pull request (the current tree was assembled through PR #1). Keep the README/HTML duplication in sync within the same commit so reviewers never see the two files disagree.
