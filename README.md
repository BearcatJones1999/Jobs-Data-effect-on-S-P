# Payroll Surprise → S&P 500 Returns

A predictive regression of next-month S&P 500 returns on the *surprise* component of U.S. nonfarm payroll releases, where surprise is extracted from a real-time expanding-window ARIMA forecast (no look-ahead, no reliance on survey consensus data).

Built as a personal research project to practice the macro-event-to-equity-index workflow: building a model-based expectation series, isolating the unanticipated component of a macro release, and testing whether that surprise carries predictive information for forward equity returns.

## Research Question

Does the *unanticipated* portion of U.S. nonfarm payroll growth contain information about next-month S&P 500 returns? If the relationship is negative — i.e., positive payroll surprises predict lower forward equity returns — that's consistent with a discount-rate channel in which strong labor data raises expected policy tightening and compresses valuations ("good news is bad news").

## Data

- **S&P 500 Index (^GSPC)** — monthly adjusted close, Yahoo Finance
- **Nonfarm Payrolls (PAYEMS)** — monthly level, FRED (pulled directly via public CSV endpoint, no API key)
- **Sample**: January 2005 – February 2026, monthly frequency

## Methodology

### 1. Stationary transforms

- Equity returns: monthly log returns, `r_t = log(P_t) − log(P_{t−1})`
- Payroll change: monthly first difference of the payroll level, `ΔPay_t = Pay_t − Pay_{t−1}` (PAYEMS is in thousands of workers)

ADF tests confirm both transformed series are stationary over the sample.

### 2. Surprise construction (no look-ahead)

Survey-based consensus data is not used. Instead, the surprise is the one-step-ahead forecast error from an **expanding-window ARIMA(1,0,1) with constant** on payroll changes:

```
Surprise_t = ΔPay_t − E_{t−1}[ΔPay_t]
```

At each month *t*, the ARIMA model is refit on data through *t−1* only, and the next-period forecast is generated out of sample. A 36-month minimum training window protects against unstable early-sample estimates.

The surprise is then standardized using a 36-month rolling standard deviation so the regression coefficient is interpretable as the return impact of a +1σ payroll surprise.

### 3. Predictive regression

```
r_{t+1} = α + β · Surprise_t^(z) + γ_0 · r_t + γ_1 · r_{t−1} + ε_{t+1}
```

Surprise measured at *t*, return measured at *t+1* — surprise leads the equity return by one period to avoid contemporaneous endogeneity. Return lags are included to control for short-run momentum/reversal dynamics. Newey–West (HAC) standard errors with 6 lags handle residual autocorrelation and heteroskedasticity in monthly data.

A second ARDL-style specification using payroll *growth* (rather than the model-implied surprise) with distributed lags is included as a robustness check.

### 4. Diagnostics

- **ADF tests** on returns and payroll growth (stationarity)
- **Ljung–Box** on residuals at 6 and 12 lags (residual autocorrelation)
- **ARCH(12)** test (conditional heteroskedasticity)

## Results

### Primary specification (surprise-based, n = 191)

| Variable | Coefficient | HAC std err | z | p-value |
|---|---|---|---|---|
| const | 0.0120 | 0.003 | 3.90 | <0.001 |
| **pay_surprise_z** | **−0.0041** | **0.003** | **−1.48** | **0.139** |
| r_sp_L0 | −0.1272 | 0.072 | −1.77 | 0.076 |
| r_sp_L1 | −0.0937 | 0.071 | −1.33 | 0.184 |

- **R² = 0.042**, joint F-test p = 0.002
- A +1σ payroll surprise is associated with approximately **−0.41% next-month S&P log return**

### Interpretation

The coefficient sign is consistent with the discount-rate channel hypothesis — positive payroll surprises predict lower forward equity returns. However, the relationship is **not statistically significant at conventional thresholds** (p = 0.139). The joint F-test rejects the null that all coefficients are zero, but the surprise variable alone cannot be cleanly distinguished from zero at the monthly horizon.

This is a defensible null result that's worth its own discussion. Two channels likely compete in the pooled sample:

- **Discount-rate channel** (negative): strong labor → tightening expectations → lower valuations. Dominates in tightening regimes (2022–2023).
- **Growth channel** (positive): strong labor → strong earnings expectations → higher valuations. Dominates in easing/ZIRP regimes (2009–2015).

Pooling across 21 years of regimes likely attenuates the surprise coefficient toward zero. A regime-conditional specification (interacted with Fed-funds direction or VIX regime) is the natural next step.

### Robustness: ARDL specification on payroll growth (n = 249)

A parallel ARDL-style regression on payroll log-growth with distributed lags finds:

| Variable | Coefficient | HAC p-value |
|---|---|---|
| g_pay_L0 | −0.114 | 0.501 |
| g_pay_L1 | 0.005 | 0.971 |
| **g_pay_L2** | **−0.313** | **0.003** |

- Cumulative payroll-growth effect (sum across lags): **−0.422**
- Joint F-test p < 0.001

The two-month-lagged payroll growth term is statistically significant and negative, suggesting the equity response to labor-market information loads at a delay — consistent with the time it takes the rate market to digest and reprice the trajectory of payroll growth before equities respond. This is consistent with the surprise specification's directional finding but operates through a different (longer-horizon, level-based) channel.

## Conceptual Framework

A negative coefficient on payroll surprises is consistent with the macro-finance **discount-rate channel**:

1. Stronger-than-expected payroll growth ⇒ inflation pressure
2. Inflation pressure ⇒ higher expected policy rates
3. Higher discount rates ⇒ lower equity valuations

This is the "good news is bad news" pattern that dominates tightening regimes. In easing regimes, the same surprise can flip sign as the growth channel dominates the rate channel — i.e., strong jobs data is read as a recovery signal rather than a tightening trigger. This regime dependence is the most likely explanation for the attenuated coefficient in the pooled monthly specification.

## Caveats and Limitations

1. **Model-implied surprise, not survey consensus**. The ARIMA(1,0,1) forecast is a reasonable benchmark for what was "expected," but professional consensus (Bloomberg, Refinitiv) reflects forward-looking information the ARIMA can't see (initial-claims trends, ADP, regional Fed surveys). Using a real consensus series would give a cleaner surprise measure.

2. **Monthly horizon dilutes the signal**. A daily or intraday event study around the release timestamp (8:30 ET on the first Friday) would isolate the unanticipated-news impact more cleanly. The 30-day forward window blends the release-day reaction with three to four weeks of unrelated price action.

3. **Pooled across regimes**. The pre-2008, ZIRP era, and 2022+ tightening cycle almost certainly have different surprise-response relationships. A regime-switching or interacted specification (e.g., surprise × Fed-funds-direction dummy) would better characterize when the discount-rate channel dominates.

4. **No vol-surface component**. The equity-index reaction is one channel; the implied volatility surface response (VIX, SPX vol term structure) around payroll releases is the natural next layer for an options-aware version of this work.

5. **Single release type**. Extending to CPI, FOMC, JOLTS, and PCE surprises would test whether the surprise-response pattern generalizes across macro releases.

## Extensions

- Replace ARIMA-based surprise with Bloomberg/Refinitiv consensus surprise
- Move to intraday SPX/SPY data with a tight window around the 8:30 ET release
- Add CPI, FOMC, and JOLTS to a multi-release surprise panel
- Layer in SPX implied volatility term structure response to surprises
- Regime-conditional specification (Fed easing vs. tightening, VIX regime)
- Sector-level decomposition (XLF vs. XLU vs. XLK should respond differently to rate-news surprises)

## Stack

Python, pandas, NumPy, statsmodels (SARIMAX, OLS with HAC, ADF, Ljung–Box, ARCH tests), yfinance, FRED public CSV endpoint.

---

*Educational research project. Not investment advice.*
