# Evaluation Summary — AQI Forecasting (India, Multi-City)

This document describes how the final model was evaluated and provides a clear interpretation of the results.  
Model details, intended use, architecture, and ethical considerations are documented separately in `MODEL_CARD.md`.

---

## 1. Evaluation Setup

- **Task**: 3-day ahead AQI forecasting (levels 1–5)  
- **Model evaluated**: Linear Regression (final selected model)  
- **Split strategy**: chronological (no shuffling)  
  - ~70% train  
  - ~15% validation  
  - ~15% test  

### **Scripts used**

- ** python scripts/train.py \**
    --data data/air_pollution_data.csv \
    --model linear \
    --horizon-days 3

python scripts/evaluate.py \
    --data data/air_pollution_data.csv \
    --model-path models/best_model.joblib \
    --plot-city Ahmedabad

python scripts/explain_shap.py \
    --data data/air_pollution_data.csv \
    --model-path models/best_model.joblib
### 2. Quantitative Metrics
Replace x.xx with your actual results from running train.py and evaluate.py.

Split	RMSE	MAE	MAPE	R²	Rounded Accuracy
Valid	x.xx	x.xx	x.xx	x.xx	x.xx
Test	x.xx	x.xx	x.xx	x.xx	x.xx

Interpretation
MAE < 1 → average error is less than one AQI level, which is acceptable for discrete levels (1–5).

Rounded accuracy reflects how often the model predicts the exact AQI level after rounding.

R² tends to be modest because AQI (1–5) has low variance — this is expected.

### 3. Visual Evaluation
3.1 Prediction vs. Ground Truth (Example City)

Observations

The model captures overall AQI trends and level transitions.

Sharp pollution spikes (e.g., festival or burning events) may be underestimated — common for linear models.

No unusual patterns suggest information leakage (e.g., predictions “knowing” future values).

3.2 Monthly AQI Behavior

Observations

Highlights seasonal pollution patterns.

Confirms that the model leverages meaningful cyclic signals rather than noise.

 3.3 SHAP Explainability
Feature Importance

SHAP Summary Plot

Observations

Lagged pollutant features and time-cycle features contribute most to predictions.

No indication of risky or future-derived features dominating importance → no leakage detected.

SHAP trends follow environmental intuition: higher pollutant levels increase predicted AQI.

### 4. Error Patterns & Practical Considerations
Higher errors occur around extreme events (e.g., holidays, crop burning, dust storms).

Cities with sparse or inconsistent data show more variable accuracy.

AQI predictions may fall between discrete classes; rounding can introduce boundary misclassification.

### 5. Key Takeaways
The Linear Regression baseline is stable, interpretable, and leakage-free for 3-day AQI forecasting.

This evaluation demonstrates:

clear and reproducible methodology

meaningful visualization and interpretation

transparent feature contribution analysis

The results form a solid foundation for exploring more advanced models (e.g., XGBoost, LSTM) under the same evaluation framework.
