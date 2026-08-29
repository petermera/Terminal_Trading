# Volt Terminal Product Roadmap

- Status: proposed
- Last updated: 2026-08-29
- Audience: product owner, developers, and first-time contributors

## Product brief

Volt Terminal is the main user-facing electricity-market workspace. Its job is to help a user understand market conditions and the evidence behind them. The current experience is a simulation; live market data and trading actions are not implied.

## Who it serves

- an analyst scanning prices, load, generation, and grid conditions;
- a researcher checking the evidence behind a model output;
- a developer verifying that grid data arrives with correct time and source labels.

## Repository connections

| Repository | Relationship |
|---|---|
| `eu-grid-data-connectors` | authoritative normalized market observations |
| `ai-trading-agent` | research signals and explanations, never an unlabelled order instruction |
| `trading-terminal` | visual experiment sibling; successful ideas graduate here deliberately |

## Experience requirements

1. Every screen labels whether data is simulated, delayed, or live.
2. Every value exposes unit, delivery period, source, and freshness.
3. Missing or stale data is visible; it is never replaced with a convincing fake value.
4. Model outputs show the observation time, model version, and a plain-language explanation.
5. No order is placed without a separate, explicit execution design and user confirmation.

## Release stages

### Stage 1 — trustworthy simulation

- name the core screens and navigation;
- make simulated states and empty states explicit;
- add a beginner guide explaining every panel.

### Stage 2 — read-only market data

- consume the versioned contract from `eu-grid-data-connectors`;
- display provenance, time zone, freshness, and quality flags;
- test missing, revised, stale, and daylight-saving data.

### Stage 3 — research signals

- show selected `ai-trading-agent` outputs as research;
- expose assumptions and confidence limits;
- compare a signal with the source observations used to create it.

### Stage 4 — decision workspace

- add watchlists, annotations, and scenario comparison;
- keep any future execution capability outside scope until separately specified and reviewed.

## Definition of done for the next release

- A newcomer can distinguish the product from the visual experiment repo.
- Simulated data is clearly labelled on every relevant screen.
- One canonical grid-data fixture renders with source and timestamp metadata.
- Empty, stale, revised, and error states have visible behavior.
- Accessibility and responsive-layout checks pass for the main workflow.

## Risks and open questions

- A polished simulation can be mistaken for live or actionable data.
- European delivery periods are easy to display incorrectly around daylight-saving changes.
- Open question: which single user decision should define the first end-to-end screen?
