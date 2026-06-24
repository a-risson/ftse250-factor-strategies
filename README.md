# Robustness of Simple Technical Trading Signals in FTSE 250 Mid-Cap Equities

### A Walk-Forward Backtesting Approach

A survivorship-bias-free, out-of-sample study of whether simple, interpretable long-only
trading signals retain economically meaningful predictive power in the FTSE 250 mid-cap
segment **after** realistic UK transaction-cost frictions (bid-ask spreads + 0.5% stamp duty).

> MBA dissertation — FinTech & Data Analytics, Anglia Ruskin University.
> Python · pandas · statsmodels · Bloomberg (`xbbg`) · AQR factor library.

---

## Research question

Do cross-sectional momentum, short-term reversal, and low-volatility signals — and a simple
combination of them — survive realistic trading costs in UK mid-caps, when evaluated on a
**pre-registered, untouched 2024–2025 holdout**?

## Headline results

Long-only top-quintile portfolios, equal-weighted, rebalanced monthly, net of Corwin–Schultz
spreads and 0.5% stamp duty on buys. Alphas from a Carhart four-factor regression on UK
(AQR) factors.

| Strategy | Net-of-cost outcome | Notes |
|---|---|---|
| **Momentum + Low-Vol composite** | **Carhart α ≈ 5.07% (t = 2.01)**, OOS Sharpe ≈ 1.04, max drawdown ≈ −5.8% | Strongest strategy overall |
| Low-volatility | Positive net alpha | Most factor-resistant solo signal |
| Momentum | Positive net alpha | Largely absorbed by the UMD momentum factor |
| Short-term reversal | **Negative** net-of-cost return; significantly negative CAPM α | Turnover-driven cost erosion is structural |
| Volume-confirmation filter | No improvement to any signal | Added turnover without added alpha |

**Takeaway:** in this mid-cap universe, the costs are the story. A low-turnover momentum +
low-volatility composite clears its frictions with room to spare; a high-turnover reversal
signal does not.

---

## Pipeline

The repository is a sequence of numbered notebooks. Each consumes the previous stage's output
and writes Parquet files to a (git-ignored) `data/` directory.

| Notebook | Stage | What it does |
|---|---|---|
| `01_membership_pull` | Universe | Pulls point-in-time FTSE 250 (`MCX Index`) constituents for every month-end, 2010–2025. Builds a **survivorship-bias-free** master list of every ticker ever in the index (including delisted names). |
| `02_price_pull` | Raw data | Chunked, resumable Bloomberg pull of daily OHLCV, shares outstanding, market cap, and total-return index for all constituents, plus the benchmark and SONIA risk-free rate. |
| `03_data_verification` | QA + split | Pivots to a wide panel, runs coverage and extreme-return checks, then splits off the 2024–2025 **holdout** and locks it with SHA-256 hashes. |
| `04_signal_computation` | Signals | Builds the four signals: 12-1 momentum, 1-month reversal, 90-day realised volatility, and a 20d/60d volume-confirmation filter. |
| `05_backtest_engine` | Backtest | Walk-forward, membership-aware quintile backtest across all strategies on the training period (2010–2023). |
| `06_holdout_test` | Out-of-sample | One-time unlock of the locked 2024–2025 holdout — the honest test of every strategy. |
| `07_combined_portfolio` | Composite | Winsorised cross-sectional Z-score composite of momentum + low-vol, with Corwin–Schultz spread and stamp-duty costs applied. |
| `08_robustness_checks` | Robustness | Sensitivity of results to specification choices. |
| `09_factor_attribution` | Attribution | CAPM, Fama–French 3-factor, and Carhart 4-factor regressions against the AQR UK factor library. |
| `10_reconciliation` | Validation | Reconciles the canonical membership-aware engine against a broader-universe alternative, decomposing any difference into gross-return, turnover, and cost components. |

---

## How the code maps to the dissertation

| Dissertation section | Notebook(s) |
|---|---|
| Ch. 3.3 — Data Sources & Universe Construction | `01_membership_pull`, `02_price_pull` |
| Ch. 3.3 — QA & holdout integrity (Appendix C lock certificate) | `03_data_verification` |
| Ch. 3.4 — Signal Construction | `04_signal_computation` |
| Ch. 3.5–3.8 — Portfolio formation, costs, backtest engine | `05_backtest_engine`, `07_combined_portfolio` |
| Ch. 3.7 / Ch. 4.6 — Walk-forward holdout (2024–2025) | `06_holdout_test` |
| Ch. 4.5 — Robustness | `08_robustness_checks` |
| Ch. 3.10 / Ch. 4.3 — Factor Attribution (CAPM, FF3, Carhart) | `09_factor_attribution` |
| Ch. 3.8 / Appendix B — Engine Reconciliation Grid | `10_reconciliation` |

## Methodology highlights

- **Survivorship-bias-free universe** — built from point-in-time index-membership snapshots,
  so delisted and demoted names are present exactly when they were index members.
- **Pre-registered holdout** — the 2024–2025 test set is split out and SHA-256-hashed *before*
  any modelling, so out-of-sample results cannot be retrofitted.
- **Realistic UK frictions** — Corwin–Schultz bid-ask spread estimator plus the 0.5% UK stamp
  duty applied to buys only.
- **Walk-forward design** — portfolios are formed on information available at time *t* and held
  forward; no look-ahead.
- **Membership-aware backtesting** — at each rebalance the eligible universe is exactly that
  month's index members, with an explicit reconciliation (NB10) quantifying how this affects
  inferred mid-cap alpha versus a broader-universe engine.

## Tech stack

`Python 3.13` · `pandas` · `numpy` · `statsmodels` (OLS with Newey–West HAC errors) ·
`scipy` · `matplotlib` · `xbbg` (Bloomberg) · AQR factor data.

---

## Reproducibility & data

This repository contains **code only**. The underlying market data is sourced from a Bloomberg
Terminal via `xbbg` and from the AQR factor library, and **cannot be redistributed** under their
licensing terms. The `data/` directory is intentionally git-ignored.

To reproduce the study you need:

1. A Bloomberg Terminal with API access (`xbbg` / `blpapi` installed and a running session).
2. The AQR factor files (freely available from AQR's website) placed in `data/raw/aqr/`.
3. To run the notebooks in numerical order — each writes the inputs the next one expects.

Bloomberg fields used: `PX_LAST`, `PX_OPEN`, `PX_HIGH`, `PX_LOW`, `PX_VOLUME`, `EQY_SH_OUT`,
`CUR_MKT_CAP`, `TOT_RETURN_INDEX_GROSS_DVDS`, `BID`, `ASK`, and `INDX_MWEIGHT_HIST` for index
membership.

## Limitations

Long-only specification; no market-impact cost beyond modelled spreads; a single historical
path; and a short (two-year) holdout window. Results should be read as evidence about this
universe and period, not as a deployable trading strategy.

---

## Author

**Antony Risson** — MBA (FinTech & Data Analytics), Anglia Ruskin University.
Targeting graduate quantitative research / analyst roles, London.
*References and full dissertation available on request.*
