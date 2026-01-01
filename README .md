## AQI Forecasting for Decision Support (India, Multi-City)

This project explores how **interpretable machine learning can support short-term environmental decision-making under noisy, real-world data conditions**.

Using public air pollution and meteorological data from multiple Indian cities, we develop a **leakage-safe, time-series forecasting pipeline** to predict Air Quality Index (AQI, levels 1–5) up to **3 days ahead**. The focus is not only on predictive accuracy, but on **robustness, explainability, and operational safety**, which are critical when deploying AI in real-world systems.

Rather than relying on black-box models or AQI conversion formulas, this project emphasizes **past-only feature engineering, transparent models, and SHAP-based interpretability** to understand *why* predictions are made and *when* they can be trusted.

---
## Why This Matters

In many production and environmental monitoring systems, data is noisy, incomplete, and influenced by physical processes. Overly complex models can easily overfit or leak future information, leading to misleading performance.

This project demonstrates how:
- Careful time-aware data handling prevents false confidence
- Interpretable models can support early-warning and planning decisions
- AI can **assist**, rather than replace, domain-driven decision-making

---

## What this project does
- Predict short-term AQI (1–5) for multiple cities.
- Forecast up to **3 days ahead** (configurable in the training script).
- Provide **early-warning style outputs** for the next 24–72 hours.
- Evaluate performance using **time-based splits** to avoid leakage.
- Include **SHAP explainability** to verify feature safety & model behavior.
- Provide a clean, reusable **Python library (`src/aqi_forecast/`)** for training & evaluation.

---

## Data format

The input CSV is expected to contain the following columns (names may vary slightly):

- `City` *(categorical)*  
- `date` / `datetime` *(timestamp)*  
- **Pollutant features:**
  - `PM2.5`, `PM10`, `NO2`, `SO2`, `CO`, `O3`
- **Weather features (if available):**
  - temperature, humidity, wind speed, pressure, rainfall
- `AQI` *(target: discrete levels 1–5)*  

**Notes**

- When predicting AQI, using pollutant measurements from the **same timestamp** is allowed.  
- We **do not** use any PM2.5 → AQI formula or AQI-from-PM features (to avoid label leakage).  
- All feature engineering uses **shifted and rolling past-only windows** (lags, rolling means) so the model never sees future information at training time.

---

## Pipeline summary 
The pipeline is designed to mirror real deployment constraints, ensuring that no future information is available at training or inference time.

### **Split**
- Time-ordered split only (no shuffle).  
- Last ~20% treated as test.

### **Feature engineering (strictly leakage-safe)**
- Per-city lag features: `lag1`, `lag2`, `lag3`, `lag12`, `lag24`
- Rolling means: `shift(1).rolling(k)` for windows `3` and `5`
- Cyclical time features:  
  - day-of-week (sin/cos)  
  - month (sin/cos)  
  - day-of-year (sin/cos)

### **Models**
- Classical ML:
  - Linear Regression (best model)：A linear regression model was selected as the final model due to its strong performance, stability across cities, and interpretability, which are essential for trustworthy deployment in real-world decision support systems.

- Explainability:
  - **SHAP** (bar + summary chart)

### **Metrics**
- RMSE  
- MAE  
- MAPE  
- R²  
- Rounded accuracy (if treating AQI as discrete 1–5)

---

## Repository layout

```text
.
├── README.md                         
├── requirements.txt                  
├── MODEL_CARD.md                      
├── EVALUATION.md                      
├── Technical_Description_of_the_Proposed_Approach.md
├── Exploratory_Data_Analysis.ipynb 
├── src/
│   └── aqi_forecast/
│       ├── __init__.py
│       ├── config.py
│       ├── features.py
│       ├── models.py
│       ├── metrics.py
│       ├── pipeline.py
│       ├── io.py
│       └── plots.py
├── scripts/
│   ├── train.py
│   ├── evaluate.py
│   └── explain_shap.py
├── RESULTS
│   ├── Best Model Forecast Plot
│   ├── Monthly Mean AQI Trend
│   └── SHAP value

                     
```

blib
Outputs saved to assets/:
## How to run

### 1) Install environment

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -U pip
pip install -r requirements.txt
pip install -e .
```

---

### 2) Train the model

```bash
python scripts/train.py \
    --data data/air_pollution_data.csv \
    --model linear \
    --horizon-days 3
```

**Outputs**

- Models/best_model.joblib  
- Validation/test metrics printed in terminal  

---

### 3) Evaluate (metrics + prediction plots)

```bash
python scripts/evaluate.py \
    --data data/air_pollution_data.csv \
    --model-path models/best_model.joblib \
    --plot-city Ahmedabad
```

**Outputs saved to `assets/`:**

- Best_model_prediction_plot.png  
- Monthly_mean_aqi.png  

---

### 4) SHAP explainability (feature safety)

```bash
python scripts/explain_shap.py \
    --data data/air_pollution_data.csv \
    --model-path models/best_model.joblib
```

**Outputs saved to `assets/`:**

- Best Model Forecast Plot
- Event vs Non-event mean AQl
- SHAP value

**Notes on interpretation:**

- AQI is discrete (1–5), so R² may appear modest even with low MAE.
- SHAP should be checked for leakage:
--Any feature relying on future values is a red flag.
- Time-based splitting is mandatory; shuffling inflates performance.

## Key Learnings

This project reinforced that in applied machine learning:
- Preventing data leakage is often more important than model complexity
- Simple, interpretable models can outperform complex ones when data is noisy
- Explainability tools such as SHAP are essential for validating feature safety
- AI systems are most effective when designed to support, not replace, human decision-making


**Contact:**

For questions regarding training scripts or modeling choices:
Tangyujie • ytang19@ualberta.ca Email / GitHub


---
