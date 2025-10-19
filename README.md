# Stock Portfolio Analysis

A compact Python toolkit to compute daily portfolio holdings and performance vs. a benchmark (SPY), using transaction-level data and market calendars. It handles FIFO sell adjustments, aligns holdings with daily adjusted close prices, and produces interactive Plotly charts for visual analysis.

---

## Table of contents

* [What it does](#what-it-does)
* [Why this is useful](#why-this-is-useful)
* [Tech & skills showcased](#tech--skills-showcased)
* [Repository layout](#repository-layout)
* [Quick start](#quick-start)
* [Input data format](#input-data-format)
* [How it works — function overview](#how-it-works---function-overview)
* [Example output / visuals](#example-output--visuals)
* [Notes, limitations & improvements](#notes-limitations--improvements)
* [Contribution & license](#contribution--license)

---

## What it does

This project:

* Loads transaction-level portfolio data (buys and FIFO sells).
* Builds a trading-day calendar for the NYSE and fills per-day holdings.
* Applies FIFO logic so sells are matched to prior buys and holdings are adjusted correctly.
* Fetches historical price data for tickers and a benchmark (SPY) via `yfinance`.
* Computes `modified cost per share`, per-ticker returns, benchmark-equivalent exposure, daily gain/loss comparisons, and several portfolio-level metrics.
* Produces interactive line charts using Plotly for visual comparison of ticker vs benchmark returns and absolute gains.

---

## Why this is useful

* Accurate day-by-day portfolio performance when you have transaction history (not just snapshots).
* Compares stock performance against a benchmark in both relative and absolute terms.
* Useful for portfolio attribution, backtesting simple allocation decisions, or explaining realized/unrealized gains with FIFO accounting.

---

## Tech & skills showcased

**Languages / Libraries**

* Python (3.x)
* `pandas` — advanced data wrangling / time-series manipulations
* `numpy` — numeric operations
* `yfinance` — retrieving historical market data
* `pandas_market_calendars` — accurate trading-day calendars
* `plotly` (offline / notebook mode) — interactive visualizations

**Concepts & techniques**

* Time-series alignment (prices ⇄ holdings snapshots)
* FIFO position matching for sells
* Adjusted cost-per-share calculations
* Benchmark-relative performance calculations
* Data pipeline design: ingest → transform → aggregate → visualize
* Reproducible analysis and modular function design

---

## Repository layout

```
.
├── README.md                # (this file)
├── stock_analysis.py        # main analysis script (the code you provided)
├── test_stock_transactions.csv  # example transactions CSV
└── requirements.txt         # recommended Python packages
```

> If you prefer a notebook, you can adapt the main script into a Jupyter / Colab notebook for step-by-step exploration.

---

## Quick start

1. Create a virtual environment and install dependencies:

```bash
python -m venv venv
source venv/bin/activate   # or `venv\Scripts\activate` on Windows
pip install -r requirements.txt
```

`requirements.txt` should at minimum contain:

```
pandas
numpy
yfinance
pandas_market_calendars
plotly
```

2. Place your transactions file in the repo (example: `test_stock_transactions.csv`).

3. Run the script:

```bash
python stock_analysis.py
```

This will:

* load `test_stock_transactions.csv`
* fetch price data
* compute per-day portfolio metrics
* produce Plotly charts (opened via `plotly.offline.plot`)

---

## Input data format

The code expects a CSV with at least the following columns (based on the script):

* `Symbol` — ticker symbol (e.g., `AAPL`)
* `Type` — transaction type. Examples used in the script: `Buy`, `Sell.FIFO`
* `Open date` — date of transaction (parseable by `pd.to_datetime`)
* `Qty` — quantity (integer or float)
* (Optionally other fields used by your workflow — script reads specific columns)

Make sure `Open date` is in a consistent format (ISO `YYYY-MM-DD` recommended).

---

## How it works — function overview

High-level description of main functions in the script:

* `create_market_cal(start, end)`
  Builds a list of NYSE trading days between `start` and `end` using `pandas_market_calendars`.

* `get_data(stocks, start, end)`
  Uses `yfinance` to fetch historical OHLC data for provided `stocks`. Returns a multi-index `DataFrame` keyed by ticker and date.

* `get_benchmark(benchmark, start, end)`
  Wrapper to fetch benchmark data (e.g., `['SPY']`) and format it for merges.

* `position_adjust(daily_positions, sale)`
  Internal helper that walks prior buy positions and reduces quantities according to a FIFO sale record.

* `portfolio_start_balance(portfolio, start_date)`
  Computes the portfolio positions that are "active" at the start date after accounting for prior sells (FIFO).

* `fifo(daily_positions, sales, date)`
  Applies FIFO sell adjustments for a specific date.

* `time_fill(portfolio, market_cal)`
  Produces a per-day list of holdings (snapshots) across the market calendar, applying sells on their dates.

* `modified_cost_per_share(portfolio, adj_close, start_date)`
  Matches each per-day holding snapshot to the ticker's adjusted close for that date and computes adjusted cost (symbol-level).

* `benchmark_portfolio_calcs(portfolio, benchmark)`
  Merges benchmark prices and attaches benchmark start/end date closes for comparisons.

* `portfolio_end_of_year_stats`, `portfolio_start_of_year_stats`
  Helpers to find start/end price points for each ticker for YTD calculations and benchmark equivalence.

* `calc_returns(portfolio)`
  Derives returns for tickers and benchmark, share values, gain/loss, and relative return comparisons.

* `per_day_portfolio_calcs(per_day_holdings, daily_benchmark, daily_adj_close, stocks_start)`
  End-to-end combination: concatenates per-day holdings, computes MCPS, merges benchmark, calculates returns and final combined DataFrame.

* `line_facets` / `line`
  Plotly helper functions to visualize per-symbol facets and aggregated lines.

---

## Example output / visuals

* Interactive line charts comparing:

  * Per-ticker return vs benchmark return (faceted by ticker).
  * Stock Gain/(Loss) vs Benchmark Gain/(Loss) aggregated across the portfolio.

Plotly outputs are interactive and open in the browser when the script runs. You can export those charts as static images or embed them in a notebook/dashboard.

---

## Notes, limitations & suggested improvements

**Current limitations**

* The script fetches price data at runtime via `yfinance`. For reproducible analysis, cache the prices to CSV or a local DB.
* No explicit error handling for missing tickers, incomplete transaction rows, or non-trading-day transactions.
* `plotly.offline.plot` usage is notebook-friendly but you may prefer static charts or interactive dashboards for production.

**Suggested improvements**

* Add unit tests for FIFO logic and `position_adjust` to ensure correctness for edge cases.
* Add CLI arguments (e.g., `--transactions PATH`, `--start YYYY-MM-DD`, `--end YYYY-MM-DD`) to make the script configurable.
* Add a Dockerfile or GitHub Actions workflow to run tests and produce cached price artifacts.
* Provide exports: per-day CSV, summary Excel, and a simple HTML report with embedded charts.
* Add handling for corporate actions (splits/dividends) more explicitly if needed.

---

## Contribution & license

Contributions welcome via issues and pull requests. Please open an issue to discuss major changes before submitting a PR.

Suggested license: `MIT` — include `LICENSE` file if you choose this.

---

## Contact

If you have questions, improvements, or example datasets you'd like to test, open an issue or reach out in this repository's discussions.

---

If you’d like, I can:

* Convert this README into a `README.md` file and add a `requirements.txt` tailored to the script.
* Refactor the script into a cleaner, testable module with a small CLI.
  Which would you prefer?

