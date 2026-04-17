# Methodology Report: Target_E Forecasting

## 1. Discovered Relationships Between Variables

### Redundant Series (Variables Dropped/Merged)
- **Series_C ~ Series_A** (r = 0.972): Series_C is approximately equal to Series_A with minor noise (C ~ 0.99A - 0.36). They encode the same deterministic sinusoidal signal.
- **Series_D ~ -1.20*Series_B + 50.16** (r = -0.963): Series_D is an inverted, scaled, and shifted version of Series_B.

Since C and D carry no independent information, they were excluded from the final forecasting model to avoid multicollinearity.

### Deterministic Structure of Series_A
Series_A follows a nearly perfect sine wave with a linear trend (R^2 = 0.990):

**A(t) = 0.051t + 0.37 - 5.0 * sin(2*pi*t/7 - 0.45)**

The period of 7 days was confirmed by autocorrelation peaks at lags 7, 14, 21, and 28. The residual standard deviation after fitting is only 0.48, confirming the deterministic nature. This means future Series_A values can be extrapolated analytically with high confidence.

### Stochastic Series_B
Series_B follows an autoregressive AR(1) process with coefficient 0.71 and near-zero mean. It decays toward zero and has no periodic structure.

### Lag-3 Relationship: B(t-3) -> E(t)
The most critical finding is a **strong 3-day lagged cross-correlation** between Series_B and Target_E (r = 0.85). This means Series_B values from 3 days prior are highly predictive of today's Target_E. This lag provides genuine forecasting power: for the first 3 forecast days, actual observed B values are used directly.

### The Core Formula
Combining these insights, the discovered regression formula is:

**E(t) = 1.50 * B(t-3) + 0.73 * A(t) - 0.40 * A(t-1) - 0.07 * A(t-2) + 0.52**

This achieves in-sample R^2 = 0.948. The A-related terms (positive current value minus lagged values) capture the rate of change of the sinusoidal cycle, explaining why Target_E exhibits an approximate 7-day periodicity inherited from Series_A.

## 2. Justification for Final Model

The final predictions use a **weighted ensemble of two models**:

1. **Regression Model (weight 60%)**: Uses the explicit formula above with deterministic A extrapolation and AR(3) B forecasting. It has the best rolling 1-step MAE (0.81) and its structure is fully interpretable.

2. **VAR(7) Model (weight 40%)**: A Vector Autoregression with 7 lags on all 5 series, selected by AIC. It achieved rolling 1-step MAE of 0.91, capturing any residual multivariate dynamics not in the regression formula.

The ensemble averages both approaches, hedging against model-specific biases. Both models produce similar forecasts (correlation > 0.99 between them), indicating the discovered structure is robust.

### Why Not Univariate ARIMA?
Target_E alone shows weak autocorrelation and no clean periodic structure. Univariate models miss the critical 3-day lagged influence of Series_B (the stochastic driver) and the deterministic sinusoidal component from Series_A. Without these cross-series relationships, forecast accuracy degrades substantially.

## 3. Variables Dropped and Modifications

| Variable | Action | Reason |
|---|---|---|
| Series_C | Dropped from regression | Near-duplicate of Series_A (r = 0.97); adds multicollinearity, no new information |
| Series_D | Dropped from regression | Inverted copy of Series_B (r = -0.96); redundant predictor |
| Series_A | Used deterministically | Modeled as sine + trend (R^2 = 0.99) for analytical future extrapolation |
| Series_B | Lag-3 used, AR(3) forecast | Only B(t-3) enters the model; future B values forecast via autoregression |
| Target_E | No autoregressive term | Adding E(t-1) gave negligible R^2 improvement (+0.0003) and risks error accumulation in multi-step forecasts |

The VAR(7) model retains all 5 series as a robustness check, but the regression model's performance confirms that the effective dimensionality of this system is just two: a deterministic oscillator (A) and a stochastic signal (B with 3-day lag).
