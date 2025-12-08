# Model Card — Multi-City Short-Term AQI Forecasting (India)

## Model details
This project forecasts short-term **AQI levels (1–5)** for multiple Indian cities using public air-pollution and meteorological data.  
We train and compare several model families (linear, tree-based, and sequence models). The model outputs a **continuous AQI value**, which can be **rounded and clipped to {1,2,3,4,5}** for level reporting.

- **Task:** short-term multi-city AQI forecasting (classification-like target reported via regression)
- **Horizon:** 24–72 hours (configurable); supports “+3 days ahead” by shifting the target
- **Inputs:** historical pollutants + weather, city indicator, time features, and leakage-safe lag/rolling statistics
- **Models:** Linear Regression, Random Forest, (optional) XGBoost/LightGBM, and RNN/LSTM/GRU variants
- **Output:** AQI prediction (float), optionally post-processed to discrete levels

## Intended use
**Primary use:** early warning / decision support for the next 1–3 days (e.g., “higher risk in the next 24–72h”).  
**Not intended for:** medical decisions, regulatory enforcement, or real-time emergency response without additional validation and human oversight.

## Data and preprocessing
- **Data:** multi-city air quality and weather data (public source)
- **Target:** `aqi` (discrete 1–5 in this dataset)
- **Split:** chronological split (no shuffle); last ~20% used as test to mimic future forecasting
- **City-wise features:** lag/rolling features are computed *within each city* to avoid cross-city mixing
- **Time features:** derived from timestamps (calendar features) or city time index (cyclical encoding)

## Leakage controls and sanity checks
Because air-quality data is time-ordered, leakage can inflate scores if future information slips into features. We apply and verify:
- **No “AQI from PM2.5” features** when predicting PM2.5 (would be target leakage).
- **Forecasting by target shift:** for +3-day forecasting, the target is shifted forward (e.g., AQI at *t+3 days*), while features only use information available at or before *t*.
- **Leakage-safe rolling stats:** rolling means are computed with `shift(1)` before `rolling(k)` so the window never includes the current target time.
- **Near-neighbor ablation (A/B/C/D):** remove lag/rolling shortcut features (e.g., PM2.5_lag1, PM2.5_lag1/2/3 + rolling, all lags + rolling) and re-evaluate. Large drops would suggest the model is mostly “riding” near-neighbor signals.

## Metrics
- **Regression:** RMSE, MAE, MAPE, R²
- **Discrete reporting for AQI (1–5):**
  - Rounded-level MAE after rounding/clipping predictions to [1,5]
  - Rounded-level accuracy (exact level match)

## Quantitative results
Results vary by city/time period; the numbers below are from one experiment run.
- **AQI unique values:** {1,2,3,4,5}
- **Baseline (predict train mean):** R² ≈ 0
- **Model example:** RMSE ≈ 0.97, R² ≈ 0.34
- **After rounding/clipping:** accuracy ≈ 0.50

## Responsible AI and ethical considerations
- **Transparency:** document data sources, splits, feature engineering, and leakage checks.
- **City variability:** performance can differ across cities; report city-wise metrics where possible.
- **Auditability:** use feature importance / SHAP as a lightweight audit—any feature that looks like “future info” or a target-derived proxy is a red flag.
- **Risk communication:** treat outputs as probabilistic guidance, not definitive statements; avoid over-confidence in high-risk alerts.

## Limitations
- **Discrete target:** AQI is coarse (1–5), so regression metrics alone can be misleading; level-based accuracy is necessary.
- **Spikes / heavy tails:** sequence models may under-predict extremes and can be sensitive to heavy-tailed pollution distributions.
- **Generalization:** models trained on certain cities or periods may not transfer cleanly to unseen cities or unusual events without re-validation.

## Contact
Maintainer: [Name]  
Project: Final_Project_ML-Group-5-
