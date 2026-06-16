# ETF-Crypto Relative Value Research
Author: Keane Gan

This project tests whether crypto ETFs exhibit short-term relative-value dislocations versus their underlying cryptocurrencies.

Crypto trades continuously, while ETFs trade only during market hours. This difference can create temporary pricing gaps between an ETF and its underlying crypto asset. The notebook tests whether those beta-adjusted residual dislocations mean-revert and can be used to build a market-neutral or beta-adjusted trading signal.

## Research Question

Can ETF-versus-underlying residual dislocations predict future relative returns?

Specifically, after adjusting an ETF’s return for its exposure to the underlying cryptocurrency, does the remaining residual return contain mean-reversion information?

## Strategy Overview

For each ETF and underlying crypto pair, the notebook:

1. Downloads ETF, crypto, and benchmark prices using `yfinance`
2. Computes daily close-to-close returns
3. Aligns crypto returns to ETF trading days
4. Estimates rolling ETF beta to the underlying crypto
5. Computes beta-adjusted residual returns
6. Converts residual z-scores into contrarian alpha forecasts
7. Builds ETF and crypto alpha forecasts for a beta-adjusted pair trade
8. Uses CVXPY to optimize portfolio weights
9. Applies exposure, risk, and turnover constraints
10. Shifts weights forward to reduce lookahead bias
11. Models transaction costs using turnover
12. Runs train/test parameter selection and out-of-sample validation

## Universe

The analysis focuses on relatively pure single-underlying crypto ETF products.

The notebook excludes ETFs with additional return drivers such as:

- futures-based mandates
- staking mandates
- covered-call overlays
- option-income overlays
- boosted-income overlays
- mixed crypto baskets

This keeps the research focused on ETF-versus-underlying pricing behavior.

## Methodology

### Rolling Beta

For each ETF and crypto pair, the notebook estimates:

```text
ETF return ≈ beta × crypto return
```

### Residual Return

The beta-adjusted residual is:

```text
residual return = ETF return - beta × crypto return
```

A positive residual means the ETF outperformed the underlying crypto after beta adjustment.
A negative residual means it underperformed.

### Residual Z-Score

The notebook measures how unusual each residual is relative to recent history:

```text
z = (residual - rolling mean) / rolling standard deviation
```

### Signal

The strategy uses a contrarian signal:

```text
signal = -tanh(z / zscale)
```

If the ETF is rich versus crypto, the strategy leans short ETF and long crypto.
If the ETF is cheap versus crypto, the strategy leans long ETF and short crypto.

Small residual deviations can be ignored using an entry threshold:

```text
if abs(z) < entry_z:
    signal = 0
```

## Portfolio Optimization

The notebook uses CVXPY to choose daily portfolio weights.

The optimizer maximizes expected return while penalizing risk and turnover:


The optimizer solves:

$$
\max_w \quad \alpha^\top w - \lambda w^\top \Sigma w - \kappa \lVert w - w_{\text{prev}} \rVert_1
$$

where $\alpha$ is the expected return vector, $w$ is the portfolio weight vector, $\Sigma$ is the covariance matrix, $\lambda$ controls risk aversion, and $\kappa$ controls turnover penalty.


Subject to:

```text
sum(abs(weights)) <= 1.00
max(abs(single_asset_weight)) <= 0.35
```

These constraints limit gross exposure and single-asset concentration.

## Backtest Design

The backtest includes several controls to reduce unrealistic performance:

- ETF weekend prices are not forward-filled
- signals are shifted before returns are applied
- transaction costs are modeled using turnover
- parameters are selected only on the training sample
- one frozen-parameter out-of-sample test is run
- benchmark alpha, beta, and correlation diagnostics are included

## Results

The final frozen-parameter out-of-sample test produced:

- **24.43% total return**
- **36.26% annualized return**
- **15.32% annualized volatility**
- **2.37 Sharpe ratio**
- **OOS period:** 2025-09-30 to 2026-06-15
- **Transaction-cost assumption:** 20 bps

The strategy also showed low beta and low correlation to major crypto and equity benchmarks, suggesting that performance was not simply disguised long-crypto or long-equity exposure.

Benchmarks included:

- BTC
- ETH
- SOL
- XRP
- SPY
- equal-weight ETF benchmark

## Interpretation

The results suggest that ETF-versus-underlying residual dislocations may contain short-term mean-reversion information.

This should be viewed as exploratory quantitative research, not a finished trading system.

## Limitations

Important caveats include:

- short sample period due to recent spot crypto ETF launches
- high turnover
- transaction costs may be underestimated
- bid-ask spreads are not fully modeled
- borrow costs are not included
- market impact is not included
- execution timing is simplified
- `yfinance` data may differ from institutional pricing data
- ETF and crypto close-time differences may explain part of the signal
- performance may decay as markets become more efficient

## Possible Extensions

Future improvements could include:

- intraday ETF and crypto data
- ETF premium and discount data
- bid-ask spread estimates
- borrow-cost modeling
- market-impact modeling
- walk-forward parameter selection
- alternative covariance estimators
- live paper-trading simulation
- ETF creation and redemption flow data

## Files

```text
etf_crypto_relative_value_research.ipynb
```

Main notebook containing data download, signal construction, optimization, backtesting, diagnostics, and out-of-sample validation.

## Tools Used

- Python
- pandas
- NumPy
- yfinance
- statsmodels
- CVXPY
- matplotlib

## How to Run

Install the required packages:

```bash
pip install pandas numpy yfinance statsmodels cvxpy matplotlib
```

Open the notebook:

```bash
jupyter notebook etf_crypto_relative_value_research.ipynb
```

Run the notebook from top to bottom.

## Disclaimer

This project is for educational and research purposes only. It is not investment advice or a recommendation to trade any ETF, cryptocurrency, or financial instrument.

Backtested results are hypothetical and may not reflect live trading performance.
