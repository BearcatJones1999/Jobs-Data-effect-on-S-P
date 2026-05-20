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
 
A second ARDL-style specification using payroll *growth* (rather than the model-implied surprise) with distributed lags is included as a comparison.
 
### 4. Diagnostics
 
- **ADF tests** on returns and payroll growth (stationarity)
- **Ljung–Box** on residuals at 6 and 12 lags (residual autocorrelation)
- **ARCH(12)** test (conditional heteroskedasticity)
## Key Findings
 
> The notebook produces full regression output, ADF stats, and diagnostic tests when run.  Headline results to be added after a final clean run.
 
**Expected sign**, from the discount-rate channel framing: β < 0. A positive payroll surprise (stronger-than-expected jobs growth) raises expected policy tightening, which compresses next-month equity valuations.
 
[INSERT after running the notebook]:
- Coefficient on standardized payroll surprise (β): _______
- HAC t-statistic: _______
- R²: _______
- Number of observations: _______
- Ljung–Box p-values (6, 12 lags): _______
- ARCH(12) p-value: _______
## Conceptual Framework
 
A negative coefficient on payroll surprises is consistent with the macro-finance **discount-rate channel**:
 
1. Stronger-than-expected payroll growth ⇒ inflation pressure
2. Inflation pressure ⇒ higher expected policy rates
3. Higher discount rates ⇒ lower equity valuations
This is the "good news is bad news" pattern that dominates tightening regimes (e.g., 2022–2023). In easing regimes (e.g., 2009–2015 ZIRP), the same surprise can flip sign as the growth channel dominates the rate channel — i.e., strong jobs data is read as a recovery signal rather than a tightening trigger. This regime dependence is a known limitation of the pooled specification used here.
 
## Caveats and Limitations
 
1. **Model-implied surprise, not survey consensus**. The ARIMA(1,0,1) forecast is a reasonable benchmark for what was "expected," but professional consensus (Bloomberg, Refinitiv) reflects forward-looking information the ARIMA can't see (initial-claims trends, ADP, regional Fed surveys). Using a real consensus series would give a cleaner surprise measure.
2. **Monthly horizon**. The next-month window dilutes the immediate market reaction to the release. A daily or intraday event study around the release timestamp (8:30 ET on the first Friday) would isolate the unanticipated-news impact more cleanly.
3. **Pooled across regimes**. Pre-/post-2008, ZIRP era, and 2022+ tightening cycle likely have different surprise-response relationships. A regime-switching or interacted specification (e.g., surprise × Fed-funds-direction dummy) would better characterize when the discount-rate channel dominates.
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
## Stack

Python, pandas, statsmodels, FRED API, [Yahoo / Alpaca / other for SPY data]
