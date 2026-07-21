# Pairs Trading: Mean Reversion Strategy

A end-to-end quantitative research project that tests whether a pair of stocks exhibits mean-reverting behavior, builds three competing trading strategies around that relationship, and stress-tests all three using Monte Carlo simulation before drawing any conclusions.

## Overview

The core idea behind pairs trading is simple: find two related assets whose price relationship tends to stay stable over time. When that relationship drifts unusually far from normal, bet that it reverts — and profit from the correction, regardless of which asset actually moves.

This project doesn't assume that relationship exists — it tests for it rigorously, then builds and honestly evaluates three different ways of trading it.

## What's Inside

1. **Data & Spread Construction** — Downloads daily prices for two stocks, normalizes them to a common scale (both rebased to start at 100), and builds the spread between them — the actual object the strategy trades.

2. **Statistical Validation** — Four independent tests check whether the spread genuinely mean-reverts before any strategy gets built:
   - Correlation (are the two assets even related?)
   - Lambda / mean-reversion regression (does the spread get pulled back to normal?)
   - Half-life (how long does a deviation typically take to correct?)
   - Augmented Dickey-Fuller test (the formal, rigorous stationarity test)

3. **Train/Test Split** — All strategy parameters are learned from a training period only, then applied to an untouched test period, to simulate what a real trader could actually have known at the time.

4. **Three Trading Strategies**, all using the same core entry/exit logic (enter on an unusual deviation, exit on reversion or a stop loss), differing only in how "normal" volatility is measured:
   - **Frozen volatility** — one fixed number, calculated once from training
   - **Rolling volatility** — recalculated daily from the trailing 60 days
   - **GARCH-based volatility** — a fitted GARCH(1,1) model, walked forward day-by-day using only real, already-known data

5. **Monte Carlo Simulation** — Rather than trusting a single historical backtest, the project simulates 1,000 statistically realistic alternate versions of the test period and runs all three strategies on each one, to distinguish genuine, reproducible edge from luck.

## Requirements

```
yfinance
pandas
numpy
matplotlib
statsmodels
arch
```

Install with:
```bash
pip install yfinance pandas numpy matplotlib statsmodels arch
```

## Running the Project

The notebook runs top to bottom. Update the ticker symbols and date range in the data-loading cell to test a different pair:

```python
ticker_a = "AAPL"
ticker_b = "MSFT"
date_1 = "2020-01-01"
date_2 = "2024-01-01"
```

## How to Use

The fastest way to try this out — no local setup, no installs — is to run it directly in **Google Colab**:

Just click the badge above, hit **Run all** in Colab, and it'll install any missing packages and execute the full pipeline in your browser.

**To test your own pair of stocks:** once the notebook is open in Colab, find the data-loading cell and edit these variables, then re-run everything from that cell down:

```python
ticker_a = "AAPL"
ticker_b = "MSFT"
date_1 = "2020-01-01"
date_2 = "2024-01-01"
```

A good pair to test should have a real, structural reason to move together (same sector, similar business model, or a direct economic link) — not just a coincidental historical correlation.

**Prefer to run it locally instead?** Clone the repo, install the requirements below, and open the notebook in Jupyter or VS Code.

## Results Summary

| Strategy | Adapts to changing conditions? | Monte Carlo: % profitable across 1,000 simulations |
|---|---|---|
| Frozen volatility | No | Low — median outcome is often zero trades |
| Rolling volatility | Yes | Majority profitable — the strongest, most reproducible edge |
| GARCH-based volatility | Yes (models volatility, not direction) | Close to a coin flip |

The rolling-volatility strategy showed the most robust, reproducible edge — not just in the one historical period tested, but across the full distribution of simulated alternate histories. The GARCH-based approach correctly modeled volatility clustering but, as designed here (betting on shock reversal), showed no real directional edge — a legitimate, informative negative result rather than a coding failure.

## Limitations

- No transaction costs, slippage, or borrowing fees are modeled — real-world returns would be lower
- Only one pair was tested; results may not generalize to other pairs without re-running the full validation pipeline
- The underlying cointegration signal, while real, sits close to the edge of standard statistical significance (ADF test) — treat results with appropriate caution
- The GARCH strategy tests one specific design (shock-reversal); GARCH volatility could alternatively be used as a filter layered on top of the rolling strategy, which was not implemented here
- Stationarity 

## Structure

The project is organized as a single Jupyter notebook, walking through each step above in order, with statistical tests, three backtests, Monte Carlo simulation, and a final verdict combining every result into one conclusion.
