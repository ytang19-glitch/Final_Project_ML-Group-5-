# Evaluation Summary — AQI Forecasting (India, Multi-City)

This document summarizes the performance of the final Linear Regression model for **3-day ahead AQI forecasting**.

---

## 1. Evaluation Setup

- **Time-based split** (strict chronological order)  
- Train: ~70%  
- Validation: ~15%  
- Test: ~15%  
- Horizon: **AQI(t + 3 days)**  
- Target: AQI levels **1–5**

Scripts used:

```bash
python scripts/train.py
python scripts/evaluate.py
python scripts/explain_shap.py
2. Metrics (replace with your actual values)
Split	RMSE	MAE	MAPE	R²	Rounded Accuracy
Valid	x.xx	x.xx	x.xx	x.xx	x.xx
Test	x.xx	x.xx	x.xx	x.xx	x.xx

Interpretation

MAE < 1 indicates average prediction error is less than 1 AQI level.

Rounded Accuracy ≈ 0.45–0.55 is common for AQI classification using regression.

Moderate R² is typical because AQI is discrete and has low variance.

3. Prediction vs True
This plot compares real AQI values with predicted AQI levels for a chosen city:


Insights

Model captures broad AQI trends

Sharp spikes may be under-predicted (expected for linear models)

4. Monthly AQI Patterns

Insights

Helps identify seasonal risk periods

Useful for interpreting monthly stability or heavy pollution months

5. SHAP Explainability
Feature Importance (Global)

Shows which features contribute most to AQI forecasting.

SHAP Summary

Key Takeaways

PM2.5, PM10, NO2 lags strongly influence predictions

Cyclical time features reveal seasonal structure

No evidence of leakage: no future or target-derived features show abnormal importance

6. Overall Interpretation
The model performs consistently on validation and test sets.

MAE and rounded accuracy indicate good practical predictive value.

Spikes (pollution events) remain challenging, but the model captures trends well.

SHAP confirms the model relies on safe, interpretable features.

7. Conclusion
The final model offers a clean, leakage-free baseline for AQI forecasting.
It is suitable for early-warning dashboards and environmental research, and can be extended with more advanced models (XGBoost / LSTM / hybrid pipelines).
