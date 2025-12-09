# AQI Forecasting (India, Multi-City) — Final Project (Group 5)

This repository predicts short-term **Air Quality Index (AQI, levels 1–5)** for multiple Indian cities using public air-pollution and meteorological data.

Our main configuration uses **3-day ahead forecasting**, created by shifting the target to **AQI(t + 3 days)** while strictly using **only past & present features** (no future leakage).

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

## Pipeline summary (matches our clean training code)

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
  - Linear Regression (best model)
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

- Shap_bar_best_model.png  
- Shap_summary_best_model.png

**Notes on interpretation:**

- AQI is discrete (1–5), so R² may appear modest even with low MAE.
- SHAP should be checked for leakage:
--Any feature relying on future values is a red flag.
- Time-based splitting is mandatory; shuffling inflates performance.

**Contact:**

For questions regarding training scripts or modeling choices:
Tangyujie • ytang19@ualberta.ca Email / GitHub


---
