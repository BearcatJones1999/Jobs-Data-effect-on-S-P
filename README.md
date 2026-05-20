# Jobs Data → S&P 500 Event Study

An event-study analysis of how monthly U.S. employment releases move the S&P 500, 
decomposing index reactions into the *surprise* component (actual − consensus) 
versus the headline print.

Built as a personal research project to practice the macro-event-to-equity-index 
workflow: pulling release data, aligning to market timestamps, isolating the 
surprise component, and measuring conditional return distributions.

## Research Question

Does the S&P 500's reaction to the monthly jobs report scale with the *surprise* 
relative to consensus, or does the market respond primarily to the headline 
print regardless of expectations? Are reactions symmetric for upside vs. downside 
surprises?

## Data

- **Release**: [NFP / Unemployment Rate / both — TELL ME WHICH]
- **Consensus**: [Source — Bloomberg? Refinitiv? FRED ALFRED vintages?]
- **Equity data**: SPY [daily / intraday] returns from [data source]
- **Sample**: [START YEAR] – [END YEAR], [N] release events
- **Frequency**: First Friday of each month, 8:30 ET release

## Methodology

### 1. Surprise Construction
`surprise = (actual − consensus) / std(actual − consensus)` 
Standardized so coefficients are interpretable as return per 1σ surprise.

### 2. Event Windows
- **Same-day**: SPY close-to-close on release day
- [Optional: pre-release drift, post-release drift, 5-day window]

### 3. Regressions
- `SPY_return = α + β · surprise + ε` (OLS, HAC standard errors)
- [Any subsample analysis: pre/post-2020? high/low VIX regimes?]

## Key Findings

[FILL IN — needs your actual numbers. Template:]

- Coefficient on standardized NFP surprise: **β = [X]%** per 1σ surprise (t = [Y])
- R² = [Z] — the surprise component explains [Z]% of SPY's release-day variance
- [Asymmetry finding: do negative surprises move the index more than positive ones? 
   This is the classic "bad news is bad news in tightening regimes" result]
- [Regime finding: does the relationship flip sign during Fed easing vs. tightening cycles?]

## Caveats

- **Single release type** — extending to CPI, FOMC, JOLTS, and PCE would 
  test whether the surprise-response pattern generalizes
- **Daily resolution** — intraday data around 8:30 ET would more cleanly isolate 
  the release impact from later-session noise
- **No vol component** — equity-index reaction is one channel; the implied vol 
  surface and term structure response is the natural next layer
- **Consensus quality** — Bloomberg consensus surveys have known biases (herding, 
  late revisions); using ALFRED vintages would give a cleaner expectations measure
- **Regime dependence** — the surprise → return relationship is widely documented 
  to flip sign with Fed posture (good news is good news vs. good news is bad news); 
  pooling across regimes may obscure this

## Extensions

- Add CPI and FOMC surprise series
- Extend to NDX (more rate-sensitive) and sector ETFs (XLF should respond differently 
  than XLU under rate-news regimes)
- Layer in VIX and SPX implied vol term structure response
- Look at intraday timestamps around the 8:30 release using minute bars

## Stack

Python, pandas, statsmodels, FRED API, [Yahoo / Alpaca / other for SPY data]
