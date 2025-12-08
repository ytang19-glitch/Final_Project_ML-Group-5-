
# Model Card — AQI Forecasting (India, Multi-City)

This model card documents the **final selected model** used in our AQI forecasting pipeline.  
The model performs **3-day ahead AQI level prediction (1–5)** using multi-city pollution & weather data.

---

## 1. Model Details

| Field | Description |
|-------|-------------|
| **Model Name** | AQI 3-Day Ahead Forecast — Linear Regression |
| **Version** | v1.0 |
| **Model Type** | Regression (mapped to discrete AQI levels if needed) |
| **Developers** | *Tangyujie / Group 5* |
| **Release Date** | 2025-12-08 |
| **Framework** | scikit-learn Pipeline |

---

## 2. Intended Use

### Primary Use
- Predict AQI (1–5) for cities **3 days ahead**  
- Provide **early-warning** signals for pollution monitoring teams  
- Support environmental policy planning / dashboards  

### Intended Users
- Environmental agencies  
- Researchers  
- Public AQI dashboards  
- City risk-monitoring systems  


---

## 3. Data Description

### Dataset
- Multi-city Indian air quality dataset
- Columns include:
  - `date`, `City`
  - Pollutants: PM2.5, PM10, NO2, SO2, CO, O3
  - Weather: temp, humidity, wind speed, pressure, rainfall
  - `aqi` levels **1–5** (target)

---

## 4. Feature Engineering (Leakage-Safe)

Located in `src/aqi_forecast/features.py`.

| Feature Type | Description |
|--------------|-------------|
| **Target Shift** | AQI(t + 3 days), grouped by city |
| **Lag Features** | lag1, lag2, lag3, lag12, lag24 |
| **Rolling Means** | shift(1).rolling(3), shift(1).rolling(5) |
| **Cyclical Time Features** | sin/cos for month, day-of-week, day-of-year |
| **Categorical Encoding** | OneHotEncoder for `City` |

All rolling & lag features strictly use **past-only information**, ensuring no future leakage.

---

## 5. Model Architecture

### Final model
- **Linear Regression** inside a scikit-learn Pipeline:
Preprocessing:

Numeric imputation → StandardScaler

City → OneHotEncoder (ignore unseen)
Model:

LinearRegression()


### Alternative models tested
- Random Forest  
- Gradient Boosting (optional)  
- RNN / LSTM (research only, not final)  

Linear Regression was selected for:
- stability  
- interpretability  
- better leakage robustness  
- consistent performance across cities  

---

## 6. Training Procedure

### Split Strategy
- Time-based split (no shuffle)
- ~70% train  
- ~15% validation  
- ~15% test  

### Horizon
- 3-day ahead forecast

### Environment
- Python 3.9+
- scikit-learn
- numpy / pandas
- SHAP for explainability

---

## 7. Evaluation Metrics

Metrics used:
- RMSE  
- MAE  
- MAPE  
- R²  
- Rounded Accuracy (if rounding to AQI levels 1–5)


| Split | RMSE | MAE | MAPE | R² | Rounded Acc |
|-------|------|------|--------|------|------------|
| Valid | x.xx | x.xx | x.xx | x.xx | x.xx |
| Test  | x.xx | x.xx | x.xx | x.xx | x.xx |

---

## 8. Visual Evaluation

### Prediction vs True  
![Prediction](../assets/best_model_prediction_plot.png)

---

### Monthly Mean AQI Trend  
![Monthly](../assets/monthly_mean_aqi.png)

---

### SHAP Feature Importance  
![SHAP bar](../assets/shap_bar_best_model.png)

---

### SHAP Summary (global effects)  
![SHAP summary](../assets/shap_summary_best_model.png)

---

## 9. Ethical Considerations

- AQI data coverage across cities may be uneven → potential bias.  
- Predictions should not be used for **legal or medical** decisions.  
- Models must be updated as pollution patterns change.  
- Use caution when communicating AQI predictions to general public.  

---

## 10. Limitations

- AQI levels are discrete (1–5), so regression may smooth sharp peaks.  
- Sudden events (dust storms, crop burning, fireworks) may be hard to forecast.  
- Some cities with fewer records have lower performance.  

---

## 11. Recommendations

- Retrain model monthly or quarterly  
- Consider XGBoost / LightGBM for improved peak capturing  
- Deploy as nightly batch forecast  
- Use SHAP monitoring to detect data drift  

---

## 12. References

- Model Cards: https://modelcards.withgoogle.com/about  
- Mitchell et al. (2019): *Model Cards for Model Reporting*  
- Kaggle Model Card Templates  

