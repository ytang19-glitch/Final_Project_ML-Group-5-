Model Card — Multi-City Short-Term AQI Forecasting (India)
Model details

⦁	Task: short-term forecasting of AQI (discrete 1–5) for multiple cities
⦁	Inputs: historical pollutant + meteorological variables, city label, time features, and lag/rolling statistics
⦁	Models: Linear Regression, Random Forest, (optional) XGBoost/LightGBM, and RNN/LSTM/GRU-based deep models
⦁	Output: AQI prediction (continuous), optionally rounded/clipped to {1,2,3,4,5}

Intended use
⦁	Primary: early warning / decision support (next 24–72h)
⦁	Not intended for: medical diagnosis, regulatory enforcement, or real-time emergency response without validation

Data & preprocessing
⦁	Split: chronological split (last 20% test), no shuffle
⦁	City-wise features: lags/rolls computed within each city
⦁	Leakage controls:
⦁	Do not compute AQI from PM2.5 as a feature when predicting PM2.5
⦁	For +3-day forecasting, shift the target forward and ensure features only use past inputs

Metrics
⦁	Regression: RMSE, MAE, MAPE, R²
⦁	Discrete AQI reporting: rounded-level MAE and accuracy after rounding/clipping to [1,5]

Quantitative results (example from experiments)
⦁	AQI has 5 unique levels: {1,2,3,4,5}
⦁	Baseline (predict train-mean): R² ≈ 0
⦁	Model example: R² ≈ 0.34, RMSE ≈ 0.97
⦁	Rounded-level accuracy: ≈ 0.50

Responsible AI / ethics
⦁	Report data sources, preprocessing, and leakage checks.
⦁	Performance may vary by city; include city-wise metrics.
⦁	Use feature-importance / SHAP to audit suspicious “future-like” features.
Limitations
⦁	AQI is discrete here; regression metrics alone can be misleading.
⦁	Deep models can struggle with spikes/heavy tails and may need log/robust losses.
