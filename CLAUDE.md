# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This is a university IT project: optimizing the dispatch (charge/discharge schedule) of a
Battery Energy Storage System (BESS) on the German day-ahead electricity market, using
**PyPSA** (Python for Power System Analysis), solved with the **HiGHS** solver.

The core goal (see `Projekt_Beschreibung.pdf` for the full assignment notes in German):
- Maximize revenue from arbitrage: charge the battery when day-ahead prices are low,
  discharge when prices are high (`Erlös = Entladeerlöse − Ladekosten`).
- Model this as a real PyPSA energy system, not just a hand-rolled price-rule script
  ("True PyPSA" requirement from the assignment).
- Use a **Rolling Horizon** optimization strategy: instead of optimizing the full year at
  once, optimize a rolling window (e.g. 24/48/72h), commit the first step's decision,
  shift the window forward, and re-optimize. This avoids assuming perfect foresight of
  future prices.
- Track State of Charge (SOC) explicitly, with constraints `0 ≤ SOC ≤ capacity`, charge/
  discharge efficiency losses, and defined start/end SOC conditions (so the optimizer
  can't drain the battery for free revenue at the end of the horizon).
- Recommended build-up: (1) simple single-day/week example, (2) full-year optimization
  with real day-ahead prices, (3) evaluate KPIs: revenue, charge cycles, full-load cycles,
  average spread, SOC profile over time.
- Possible extensions discussed: multi-market optimization (day-ahead + intraday), grid
  connection / charge / discharge power limits, and CO2-price scenarios (which tend to
  widen price spreads and increase BESS revenue).

### PyPSA component mapping for this project

| PyPSA component      | Meaning in this project              |
|-----------------------|--------------------------------------|
| `Bus`                 | Electrical node / market connection point |
| `StorageUnit`         | The BESS                              |
| `Link` / `Generator` / `Load` | Charging/discharging i.e. market interaction |
| Time series (`p_max_pu`, marginal costs, etc.) | Day-ahead price data |
| `optimize()`          | Revenue maximization (LP via HiGHS)   |

## Repository structure

- `daten/` — input data (day-ahead and historical electricity prices as CSV).
  - `daten/dayahead_2025.csv` — columns `timestamp` (15-min resolution, UTC), `dayahead €/MWh`.
  - `daten/strompreise_2024.csv` — columns unnamed index timestamp (hourly), `price_eur_mwh`.
- `notebooks/` — one Jupyter notebook per team member (`Linus.ipynb`, `Hannah.ipynb`,
  `Vanessa.ipynb`) for individual exploration/experimentation. Notebooks load CSVs via
  relative paths like `../daten/strompreise_2024.csv`, so they must be run with the
  notebook's directory (`notebooks/`) as the working directory.
- `ergebnisse/` — intended output location for results (plots, KPI tables, etc.);
  currently empty.

## Environment / dependencies

There is no requirements file or environment spec yet. Required Python packages used so
far: `pypsa`, `pandas` (and transitively a solver — the project plan specifies HiGHS,
typically available via `highspy` or PyPSA's bundled HiGHS support). `pypsa` is **not**
currently installed in the default environment — install it before running notebook code
that imports it (e.g. `pip install pypsa highspy`).
