# Volt Terminal

A visual prototype for exploring European electricity markets in one trading-style interface.

> **Important:** all prices, positions, orders, and activity in the current demo are simulated. This is a product and interface prototype, not a live market-data or execution system.

## What is here

The main prototype lives in [`global_electricity_trading_platform.html`](./global_electricity_trading_platform.html). It demonstrates:

- EPEX, Nord Pool, OMIE, and balancing-market views;
- market cards and price charts;
- an order book, positions, spreads, and a trade ticket;
- simulated price changes and trade logs;
- responsive, theme-aware visual components.

The `apps/` and `docs/` folders organize future application sections and beginner documentation.

## Newbie map

| Area | Plain-language meaning |
|---|---|
| `global_electricity_trading_platform.html` | The working visual prototype |
| `apps/` | Future application pages and sections |
| `docs/` | Explanations, decisions, and project guides |
| `README.md` | The front door to the project |

## Project family

- **This repository:** the main product and documentation home.
- [`trading-terminal`](https://github.com/petermera/trading-terminal): a smaller sibling for lightweight interface experiments.
- [`eu-grid-data-connectors`](https://github.com/petermera/eu-grid-data-connectors): the planned data-ingestion foundation.

Keep product work here. Keep reusable source connectors in the connector repository so the terminal does not become responsible for authentication, normalization, and raw-data collection.

## Data contract for future live feeds

Every displayed observation should carry enough context to be understood and reproduced:

- source and retrieval time;
- market, product, and bidding zone;
- delivery start and end;
- original timezone plus normalized UTC time;
- unit and currency;
- observed, forecast, or simulated status;
- publication or forecast issue time;
- data vintage and quality flags.

This prevents simulated data from being mistaken for observed prices and reduces timezone, daylight-saving, and look-ahead errors.

## Suggested next steps

The staged product direction and definition of done are explained in [`docs/PRODUCT_ROADMAP.md`](docs/PRODUCT_ROADMAP.md).

1. Wrap the prototype in a complete standalone page.
2. Define typed market and observation models.
3. Connect read-only normalized data from `eu-grid-data-connectors`.
4. Add clear simulated/live badges and stale-data warnings.
5. Add tests for timezone conversion, negative prices, and missing intervals.
