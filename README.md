# UCITS-Style Fund Risk Monitoring

A daily risk monitoring workflow for a simulated €100m UCITS-style fund
holding 30 securities: market risk (VaR three ways), issuer concentration,
liquidity, and redemption stress, feeding a Green / Amber / Red exception
report and an Excel risk pack.

> **Simulated fund.** Holdings and liquidity assumptions are illustrative.
> Equity prices are real (3 years of daily data from Yahoo Finance).
> UCITS concentration rules are simplified, and liquidity thresholds are
> internal assumptions, not statutory limits.

## Quick start

```bash
pip install -r requirements.txt
jupyter notebook UCITS_Risk_Monitoring.ipynb
```

Run all cells. The notebook downloads market data, runs every check, writes
`output/UCITS_Risk_Monitoring.xlsx` and saves the charts to `images/`.

## Results (run of 13 Sep 2026)

| Check | Result | Status |
|---|---:|:---:|
| 95% 1-day VaR (historical) | €563,465 | GREEN (limit €2.0m) |
| 99% 1-day VaR (historical) | €886,936 | GREEN (limit €3.0m) |
| 95% VaR: parametric / Monte Carlo | €604,690 / €565,802 | |
| Issuers above 5% | 0 | GREEN |
| Aggregate of issuers above 5% (40% rule) | 0.00% | GREEN |
| Longest days-to-liquidate (normal / stressed) | 20 / 40 days | RED |
| 20% redemption (€20m), stressed liquidity | covered in 1 day | GREEN |
| **Exceptions** | **12: 7 RED, 5 AMBER** | **Overall: RED** |

The three VaR methods agree to within about 7% at 95%, which is a useful
cross-check: historical, parametric and Monte Carlo are independent
calculations from the same return history.

Every exception is liquidity. Corporate bonds take 20 days to exit at a 10%
participation rate (RED, limit 5 days), government bonds 5 days (AMBER,
warning at 3). The fund can meet a 20% redemption from its equity book
alone, so the liquidity flags are a positions problem, not a fund-level
redemption problem. That distinction is the point of running both tests.

## Charts

![Portfolio weights](images/portfolio_weights.png)
![Days to liquidate](images/days_to_liquidate.png)
![Normal vs stressed liquidity](images/normal_vs_stressed_liquidity.png)

## What the notebook does

1. **Portfolio**: 30 positions across equities, government bonds,
   corporate bonds, ETFs and gold, with weights against NAV.
2. **Market risk**: 1-day VaR at 95% and 99% by historical simulation,
   parametric (variance-covariance) and Monte Carlo (10,000 correlated
   draws), checked against internal limits with an amber band at 90%.
3. **Concentration**: issuer exposure against a simplified 5/10/40 rule:
   warning above 5%, hard limit 10%, and issuers above 5% capped at 40% in
   aggregate.
4. **Liquidity**: days to liquidate each position at 10% of average daily
   volume, bucketed from Very Liquid to Highly Illiquid.
5. **Liquidity stress**: the same calculation with volumes halved.
6. **Redemption stress**: how much of a 20% redemption the fund can raise in
   1, 3, 5 and 10 days, under normal and stressed liquidity.
7. **Exception engine**: every breach becomes a row with risk type, actual,
   threshold, severity and action. Any RED makes the fund status RED.
8. **Reporting**: an Excel pack (Portfolio, Concentration, Market_Returns,
   Redemption_Stress, Exceptions, VaR) and three charts.

## Limitations

Stated up front, because they change how the numbers should be read:

- **Holdings total €76m against a €100m NAV.** The €24m gap is unallocated
  (treat it as cash). The notebook warns about it but does not stop.
- **VaR covers the equity sleeve only.** Bonds, ETFs and gold have no return
  series here, so they contribute zero risk. Fund-level VaR is understated.
- **Liquidity volumes are assumptions, not market data.** Each position's ADV
  is a fixed multiple of its own size by asset class, so days to liquidate is
  the same for every holding in a class (for example, every corporate bond
  shows 20 days).
- **Concentration is simplified.** Government bonds are tested under 5/10/40,
  whereas UCITS allows up to 35% per sovereign issuer.
- 1-day horizon, no expected shortfall, no VaR backtesting.

## Tools

Python, pandas, NumPy, SciPy, Matplotlib, yfinance, openpyxl

## Author

Ajay Thakur, MSc Finance (DCU)
[LinkedIn](https://www.linkedin.com/in/ajayranotthakur)
