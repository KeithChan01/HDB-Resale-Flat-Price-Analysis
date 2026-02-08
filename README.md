# HDB Resale Flat Price Analysis & Investment ROI Forecasting

An end-to-end data analysis pipeline that identifies the best-performing HDB resale flat combinations (town, flat type, storey range) for investment, using historical transaction data from 1990 to present and ARIMA-based price forecasting with Monte Carlo simulations.

## Problem Statement

Buying an HDB resale flat is the single largest financial decision most Singaporeans will make. Yet most buyers rely on gut feel, agent recommendations, or anecdotal evidence when choosing a location and flat type. This project takes a quantitative, data-driven approach to answer:

> **Which HDB town, flat type, and storey range combinations have historically delivered the best risk-adjusted returns — and which are predicted to perform best over the next 10 years?**

## Data Source

All transaction data is sourced from [Data.gov.sg](https://data.gov.sg/), Singapore's open government data portal. The analysis combines five official HDB resale transaction datasets spanning from 1990 to the present:

| Dataset | Period |
|---|---|
| Resale Flat Prices (Based on Approval Date) | 1990–1999 |
| Resale Flat Prices (Based on Approval Date) | 2000–Feb 2012 |
| Resale Flat Prices (Based on Registration Date) | Mar 2012–Dec 2014 |
| Resale Flat Prices (Based on Registration Date) | Jan 2015–Dec 2016 |
| Resale Flat Prices (Based on Registration Date) | Jan 2017–Present |

Each record includes the transaction month, town, flat type, block, street name, storey range, floor area, flat model, lease commencement date, and resale price.

## Methodology

The analysis is structured in three phases:

### Phase 1 — Exploratory Analysis & Historical ROI Ranking

- Calculates **price per square metre** across all towns and years to normalise across flat sizes.
- Computes **Compound Annual Growth Rate (CAGR)** to measure long-term price appreciation.
- Uses **5-year rolling window Sharpe Ratios** to assess risk-adjusted returns, benchmarked against the Singapore Government Securities (SGS) risk-free rate of 2.7%.
- Incorporates a **remaining lease factor** (normalised 0–1) to account for lease decay on resale value.
- Combines these into a **weighted ROI score**: 70% CAGR + 20% Sharpe Ratio + 10% Lease Factor.
- Produces a **risk-return quadrant analysis** (volatility vs. CAGR) colour-coded by remaining lease.

### Phase 2 — Granular Drill-Down

Breaks down the top-performing towns by:

- **Flat type** (1-room through executive) — CAGR comparison across types.
- **Storey range** (01–05, 06–10, ..., 26+) — floor premium analysis.
- **Street name** — identifying specific streets with the highest historical appreciation.
- **Combined analysis** — ranking the best town × flat type × storey range combinations by ROI score, with a stacked bar chart showing CAGR, Sharpe, and lease contributions.

### Phase 3 — ARIMA Forecasting & Monte Carlo Simulation

- For each valid combination (≥10 years of data, ≥15 transactions), fits an **ARIMA model** with grid search over (p, d, q) parameters optimised by AIC.
- Validates each model using an **80/20 train-test split**, filtering out combinations with MAPE > 30%.
- Forecasts price per sqm **10 years into the future**, with sanity caps on CAGR (≤10%) and price growth (≤3×).
- Runs **1,000 Monte Carlo simulations** using historical volatility as the standard deviation to produce an **80% confidence interval** around the forecast.
- Outputs predicted ROI scores, a comparison of current vs. predicted prices, and a validation error distribution.

## Key Visualisations

| Chart | Purpose |
|---|---|
| Price appreciation over time (top 5 towns) | Long-term trend comparison |
| ROI score bar chart (top 5 towns) | Overall ranking snapshot |
| Risk-return quadrant plot | Identify ideal low-risk, high-return towns |
| ACF plots (national & top towns) | Justify time series approach |
| Flat type CAGR by town | Best flat type per location |
| Storey range price premium | Quantify floor premium |
| Top streets by CAGR per town | Street-level drill-down |
| Stacked ROI component breakdown | Decompose what drives each combination's score |
| Current vs. predicted price comparison | ARIMA forecast summary |
| Monte Carlo forecast with confidence bands | Uncertainty-aware projection |
| Validation MAPE histogram | Model reliability overview |
| MAPE vs. predicted CAGR scatter | Check for systematic prediction bias |

## Tech Stack

- **Python** — pandas, NumPy, matplotlib, seaborn
- **Time Series Modelling** — statsmodels (ARIMA, ADF test, ACF)
- **Machine Learning Validation** — scikit-learn (MSE)
- **Simulation** — NumPy-based Monte Carlo
- **Environment** — Jupyter Notebook with ipywidgets progress tracking

## Project Structure

```
HDB-Resale-Flat-Price-Analysis/
├── .gitignore
├── README.md
├── requirements.txt
├── 01_descriptive_analysis.ipynb                 # EDA, ROI scoring & drill-down analysis
├── 02_arima_forecasting.ipynb                    # ARIMA forecasting & Monte Carlo simulation
├── data/                                         # Raw datasets from Data.gov.sg
│   ├── Resale Flat Prices (...), 1990 - 1999.csv
│   ├── Resale Flat Prices (...), 2000 - Feb 2012.csv
│   ├── Resale Flat Prices (...), From Mar 2012 to Dec 2014.csv
│   ├── Resale Flat Prices (...), From Jan 2015 to Dec 2016.csv
│   └── Resale flat prices (...) from Jan-2017 onwards.csv
└── checkpoints/                                  # Intermediate data & ARIMA cache (gitignored, generated at runtime)
```

## Getting Started

### Prerequisites

```bash
pip install -r requirements.txt
```

### Running the Analysis

1. Clone the repository.
2. The five CSV datasets from [Data.gov.sg](https://data.gov.sg/collections/189/view) are included in the `data/` directory.
3. Run the notebooks **in order** in Jupyter:
   - **`01_descriptive_analysis.ipynb`** — loads and processes the raw data, computes historical ROI metrics, and exports the processed dataset for the next notebook.
   - **`02_arima_forecasting.ipynb`** — loads the processed data, fits ARIMA models, runs Monte Carlo simulations, and produces the final investment recommendations.

The ARIMA modelling step may take several minutes on first run. Results are checkpointed to `checkpoints/` so subsequent runs load cached predictions.

## Configuration

All analysis parameters are centralised in the `config` dictionary at the top of each notebook:

| Parameter | Default | Description |
|---|---|---|
| `risk_free_rate` | 2.7% | SGS bond yield benchmark |
| `cagr_weight` | 0.7 | CAGR weight in ROI score |
| `sharpe_weight` | 0.2 | Sharpe ratio weight in ROI score |
| `lease_weight` | 0.1 | Remaining lease weight in ROI score |
| `forecast_years` | 10 | ARIMA forecast horizon |
| `mape_threshold` | 30% | Maximum acceptable validation error |
| `max_reasonable_cagr` | 10% | Cap on predicted annual growth |

## Limitations & Assumptions

- **ARIMA is univariate** — the model only uses historical price data and does not account for external factors such as new MRT lines, government cooling measures (ABSD, TDSR), BTO supply changes, or macroeconomic shifts.
- **Aggregate pricing** — prices are averaged at the town × flat type × storey range level per year. Individual block-level variation is smoothed out.
- **Remaining lease is estimated** — calculated as `99 − (latest_year − lease_commence_date)`, which assumes the standard 99-year lease and uses the most recent year in the dataset as the reference.
- **Stationarity assumption** — ARIMA assumes the underlying process is (or can be made) stationary through differencing, which may not hold during structural market shifts.
- **Past performance ≠ future results** — this analysis is for educational and exploratory purposes. It should not be taken as financial advice.

## License

This project is for personal and educational use.
