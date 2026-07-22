# CLAUDE.md

Guidance for AI assistants (and humans) working in this repository.

## Project overview

**Terminal_Trading** is a single, self-contained front-end artifact called **VOLT TERMINAL** —
a *simulated* European electricity trading terminal. It renders live-updating market cards, a
price-curve chart, an order book, open positions with P&L, a trade log, cross-market spreads,
and session KPIs across four views: **Day-Ahead**, **Intraday**, **Balancing**, and
**Spread Analysis**.

Everything is **client-side simulation** — a mean-reverting random walk drives prices, and
"trades" only mutate in-memory JavaScript state. There is **no backend, no real market data, no
persistence, and no real orders**. It is a UI/demo piece, not a real trading system.

- **Stack:** plain HTML + inline `<style>` + inline vanilla JavaScript. No framework.
- **Only external dependency:** [Chart.js 4.4.1](https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js) loaded from a CDN.
- **No build system, package manager, tests, linter, or CI.**

## Repository layout

| Path | What it is |
|------|-----------|
| `global_electricity_trading_platform.html` | **The canonical, runnable app.** Full markup + CSS + JS in one file. |
| `README.md` | The **same app**, preceded only by a `# Terminal_Trading` heading so GitHub renders the live terminal on the repo home page. |
| `apps/` | An aspirational, framework-neutral **scaffold of placeholder READMEs only** — no real code, not wired to the app. See [The `apps/` scaffold](#the-apps-scaffold). |

### ⚠️ The two-copy rule

`README.md` and `global_electricity_trading_platform.html` contain the **identical** app.
The only difference is the leading `# Terminal_Trading` Markdown title in the README:

```sh
# This diff must stay empty:
diff <(tail -n +2 README.md) global_electricity_trading_platform.html
```

**Any change to the app must be made in both files, in the same commit.** Treat
`global_electricity_trading_platform.html` as the source of truth and mirror it into `README.md`
(keeping the README's title line).

## Running & developing

There is nothing to install or build. To run the app, **open
`global_electricity_trading_platform.html` in a browser**.

Two things to know:

1. **Network is required** — Chart.js loads from a CDN. Offline, everything works except the
   main price-curve chart.
2. **The app expects a host theme.** The CSS references custom properties that are **not defined
   anywhere in this repo** — e.g. `var(--color-text-primary)`, `var(--color-background-secondary)`,
   `var(--color-border-tertiary)`, `var(--font-mono)`, `var(--border-radius-md)`. These are meant
   to be supplied by an outer container (a design-system / Claude-artifact-style host). Opened as a
   bare file, chrome colors, fonts, and radii fall back to browser defaults and the UI looks
   partly unstyled — this is expected, not a bug. Only the **semantic up/down colors are hardcoded
   hex** (see [Conventions](#conventions)).

## App architecture

One file, three layers: a `<style>` block, an HTML skeleton whose elements carry stable `id`s,
and a `<script>` block of vanilla JS. All state lives in module-level globals; the DOM is updated
by re-rendering `innerHTML` strings on an interval.

### Configuration constants (top of `<script>`)

Static market definitions — the data model everything else keys off:

- `DA_MARKETS` — Day-Ahead venues (EPEX, OMIE, Nord Pool, N2EX).
- `IDA_MARKETS` — Intraday venues.
- `BAL_MARKETS` — Balancing products (aFRR, mFRR, FCR, RR, Imbalance).
- `SPREAD_PAIRS` — cross-market spread definitions (`{ name, a, b }`, where `a`/`b` are market ids).

Market shape: `{ id, name, region, base, vol, product? }` — `base` is the anchor price the random
walk reverts to, `vol` scales the noise/spread, `product` labels balancing items.

### Runtime state (globals)

- `markets` — live map keyed by market id: `{ ...def, price, prev, change }`.
- `priceHistory` — map of market id → array of past prices (feeds sparklines + the chart).
- `positions` — open positions `{ market, side, qty, entryPx, id }` (seeded with 3 demo rows).
- `tradeLog` — executed trades `{ time, side, qty, market, px, pnl }`.
- `currentTab` (`'da' | 'ida' | 'bal' | 'spread'`), `selectedMarket`, `mainChartInst`.
- Session stats: `sessionHigh`, `sessionLow`, `sessionVol`, `sessionVwapSum`, `sessionVwapVol`.

### Render pipeline

`fullRender()` is the heartbeat. Each tick it calls `tickPrices()` then every renderer:
`renderGrid`, `renderOrderBook`, `renderPositions`, `renderSpreads`, `renderKPIs`, `renderTicker`,
`updateChart`, and `updateClock`.

Boot sequence (bottom of `<script>`): `initMarkets()` is called for all three market lists, then a
`setTimeout(..., 100)` runs `initChart()` + `fullRender()` and starts the timers:

- `setInterval(fullRender, 1800)` — price tick + full UI refresh (~every 1.8 s).
- `setInterval(updateClock, 1000)` — UTC clock.

### Simulation model

- `rnd(base, vol)` — jittered value around `base`.
- `tickPrices()` — mean-reverting step per market: `drift = (base - price) * 0.03`,
  `noise = (Math.random() - 0.48) * vol * 0.12` (the `0.48` gives a slight upward bias), floored at 10.
- `executeTrade(side)` — reads the qty/limit inputs, applies random **slippage**
  (`(Math.random()*0.3 + 0.05) * ±1`), then appends to `positions` and `tradeLog` and re-renders.

### Tabs & charting

- `setTab(tab, el)` switches views; `getCurrentList()` maps `currentTab` → the right `*_MARKETS`
  array. The `spread` tab repurposes the chart title for spread analysis.
- `initChart()` / `updateChart()` own the Chart.js line chart (price series + dashed average).

### Inline event handlers

Buttons wire up via inline HTML attributes (`onclick="setTab('da',this)"`,
`onclick="executeTrade('BUY')"`). Because of this, **`setTab` and `executeTrade` must remain
functions on the global scope** — do not move them into a module/closure without also switching to
`addEventListener`.

## Conventions

- **Aesthetic:** dense, monospace, terminal-style. Keep the `var(--font-mono)` font and compact spacing.
- **Color semantics (hardcoded hex, keep consistent):**
  - Up / buy / bids → green `#3B6D11` (text), `#639922` (status/spark), `#97C459`/`#EAF3DE` (buy button).
  - Down / sell / asks → red `#A32D2D` (text), `#F09595`/`#FCEBEB` (sell button).
  - Accent / selected → blue `#185FA5` with `#E6F1FB` background; chart average line `#BA7517`.
- **Theming rule:** use the host **CSS custom properties** for chrome (text, background, border,
  radius, font). Only reach for hardcoded hex when the color is a semantic up/down/accent signal.
- **Units:** prices in **€/MWh**, volumes in **MWh**, clock in **UTC**.
- **JS style:** vanilla only; state in module-level globals; render by building `innerHTML`
  strings; every renderer null-guards its container first (`const el = ...; if(!el) return;`).
- **Naming:** renderers are `render*`; market ids are lowercase hyphenated (`epex-de`, `afrr-de`);
  DOM ids are lowercase hyphenated (`market-grid`, `order-book`, `kpi-vwap`).

## Extending the app

Common changes and where to make them (then **mirror into both copies**):

- **Add a market:** append to the relevant `DA_MARKETS` / `IDA_MARKETS` / `BAL_MARKETS` array with
  a unique `id`. It flows through automatically via `initMarkets`/`getCurrentList`/`renderGrid`.
- **Add a tab:** add a `.tab` button (with its `onclick="setTab('<key>',this)"`), a matching
  `*_MARKETS` list, and branches in `getCurrentList()` (and any tab-specific logic in `setTab`).
- **Add a spread:** add an entry to `SPREAD_PAIRS` referencing existing market ids.
- **Add a KPI:** add a `.kpi` node with an `id` in the SESSION KPIs block, then populate it in
  `renderKPIs()`.
- **Change tick cadence / simulation:** adjust the `setInterval(fullRender, 1800)` value or the
  drift/noise coefficients in `tickPrices()`.

## The `apps/` scaffold

`apps/` is a **placeholder directory structure** describing where a future modular refactor
*could* live — `pages/{home,about}`, `sections/{header,content,footer}`, `documentation/`, and
`assets/{images,styles}` (with `.gitkeep` files). Every file there is a short README or stub; it
contains **no real code and is not connected to the running terminal**. Ignore it when editing the
app unless the task is explicitly about building out that structure.

## Git workflow

- Default branch: **`main`**.
- Develop on feature branches; do not commit directly to `main`.
- Keep `README.md` and `global_electricity_trading_platform.html` **in sync within the same commit**
  (see [the two-copy rule](#-the-two-copy-rule)).
- Open a pull request only when explicitly asked.
