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

Input CSV should contain (names may vary slightly):

- `City` (categorical)
- `date` / `datetime` (timestamp)
- pollutant features:  
  `PM2.5`, `PM10`, `NO2`, `SO2`, `CO`, `O3`
- weather features (if available):  
  temperature, humidity, wind speed, pressure, rainfall
- `aqi` (target: discrete levels **1–5**)

**Notes**
- When predicting AQI, using pollutants from the **same timestamp** is allowed.  
- Do **not** use any PM2.5 → AQI formula or AQI-from-PM features (that is leakage).  
- All feature engineering is done using **shifted** and **rolling (past-only)** windows.

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
  - Linear Regression (final selected model)
  - Random Forest (optional alternatives)
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

.
├── README.md
├── docs/
│ ├── MODEL_CARD.md
│ └── EVALUATION.md
│
├── src/
│ └── aqi_forecast/
│ ├── config.py
│ ├── features.py
│ ├── io.py
│ ├── metrics.py
│ ├── models.py
│ ├── pipeline.py
│ └── plots.py
│
├── scripts/
│ ├── train.py
│ ├── evaluate.py
│ └── explain_shap.py
│
├── assets/ # generated plots
├── research/ # notebooks (not graded for code quality)
├── models/ # saved trained model(s)
└── requirements.txt

---

## How to run

### 1) Install environment

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -U pip
pip install -r requirements.txt
pip install -e .
2) Train the model (example)
bash
Copy code
python scripts/train.py \
    --data data/air_pollution_data.csv \
    --model linear \
    --horizon-days 3
Outputs:

bash
Copy code
models/best_model.joblib
validation/test metrics in terminal
3) Evaluate (generate metrics + prediction plots)
bash
Copy code
python scripts/evaluate.py \
    --data data/air_pollution_data.csv \
    --model-path models/best_model.joblib \
    --plot-city Ahmedabad
Outputs saved to assets/:

best_model_prediction_plot.png

monthly_mean_aqi.png

4) SHAP explainability (feature safety)
bash
Copy code
python scripts/explain_shap.py \
    --data data/air_pollution_data.csv \
    --model-path models/best_model.joblib
Outputs saved to assets/:

shap_bar_best_model.png

shap_summary_best_model.png

Expected outputs
Model metrics (validation & test)

True vs Predicted AQI plot

SHAP bar & summary charts

Monthly AQI risk curve

Final model card (in docs/)

Evaluation markdown summary (in docs/)

Notes on interpretation
AQI is discrete (1–5) → R² naturally tends to be moderate even if MAE is small.

Check SHAP:
if any feature derived from future or target leakage, treat as a red flag.

Time-split is critical: random shuffling inflates performance artificially.
