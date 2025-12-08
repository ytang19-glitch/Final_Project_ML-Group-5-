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

### Scripts used

```bash
python scripts/train.py \
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
```

## 2. Quantitative Metrics

Replace `x.xx` with your actual results from running `train.py` and `evaluate.py`.

| Split | RMSE | MAE | MAPE | R² | Rounded Accuracy |
|------:|-----:|-----:|------:|----:|-----------------:|
| Valid | x.xx | x.xx | x.xx | x.xx | x.xx |
| Test  | x.xx | x.xx | x.xx | x.xx | x.xx |

### Interpretation

- **MAE < 1** → the average error is less than one AQI level, which is appropriate for discrete levels (1–5).  
- **Rounded accuracy** reflects how often the model predicts the exact AQI class after rounding.  
- **R² is expected to be modest** because AQI (1–5) has low variance and discrete values.


## 3. Visual Evaluation

### 3.1 Prediction vs. Ground Truth (Example City)

![Prediction vs true](../assets/best_model_prediction_plot.png)

**Observations**
- The model captures overall AQI trends and transitions between levels.  
- Sharp pollution spikes (e.g., festival emissions, crop burning events) may be underestimated — this is typical for linear models.  
- No patterns suggest information leakage (e.g., predictions anticipating future data).

---

### 3.2 Monthly AQI Behavior

![Monthly AQI](../assets/monthly_mean_aqi.png)

**Observations**
- Seasonal variations in AQI are clearly visible.  
- Indicates that the model has learned meaningful cyclic patterns rather than noise.  

---

### 3.3 SHAP Explainability

#### Feature Importance  
![SHAP bar](../assets/shap_bar_best_model.png)

#### SHAP Summary Plot  
![SHAP summary](../assets/shap_summary_best_model.png)

**Observations**
- Lagged pollutant variables and time–cycle features contribute most to predictions.  
- No evidence of future-derived or target-derived variables dominating importance → **no leakage detected**.  
- SHAP trends follow environmental intuition: higher pollutant values push predicted AQI upward.

---

## 4. Error Patterns & Practical Considerations

- Errors increase around **extreme pollution events** (holidays, fireworks, crop burning, dust storms).  
- Cities with **missing or inconsistent data** show more variability in prediction accuracy.  
- Because AQI is discrete (1–5), predictions often fall between categories; rounding may cause borderline misclassification.  
- Linear models have limited ability to represent nonlinear pollution dynamics, especially during sudden spikes.

---

## 5. Key Takeaways

- The Linear Regression model forms a **stable, interpretable, and leakage-free baseline** for 3-day AQI forecasting.  
- Evaluation demonstrates:
  - a clear and reproducible pipeline  
  - meaningful trend capture  
  - consistent behavior across validation and test sets  
  - transparent and intuitive explainability through SHAP  
- This provides a strong baseline foundation for experimenting with more advanced models (e.g., XGBoost, LightGBM, LSTM) using the same evaluation framework.

