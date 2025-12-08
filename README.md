# Final_Project_ML-Group-5- — AQI Forecasting (India, Multi-City)

This repo predicts short-term **Air Quality Index (AQI, 1–5)** for multiple Indian cities using public air-pollution + meteorology data.  
Our main setting is **3-day ahead forecasting** by shifting the target to **AQI + 3 days**, while using only **past and current** inputs (no future leakage).

---

## What this project does
- Predict short-term AQI (1–5) for multiple Indian cities.
- Forecast up to **3 days ahead** (configurable).
- Provide **early-warning** style outputs for the next **24–72 hours**.
- Compare classic ML vs deep learning, with **time-split** evaluation.
- Include **leakage checks** (sanity tests) to confirm the pipeline is safe.

---

## Data format
Input CSV should contain (names may vary slightly):
- `City` (categorical)
- `date` / `datetime` (timestamp)
- pollutants: `PM2.5`, `PM10`, `NO2`, `SO2`, `CO`, `O3`, `NH3` (as available)
- weather: temperature, humidity, wind speed, pressure, rainfall (as available)
- `aqi` (target; in our dataset typically discrete **1–5**)

Notes:
- If predicting **AQI**, using pollutant readings at the **same timestamp** is valid, but you must avoid **future rows**.
- If you switch the target to **PM2.5**, do **not** use any feature derived from PM2.5→AQI (that would be leakage).

---

## Pipeline summary (matches the notebook steps)
**Split**
- Time split only (no shuffle): last ~20% as test.

**Feature engineering (leakage-safe)**
- Per-city lag features using `shift(lag)` (e.g., lag 1/2/3/12/24).
- Rolling mean features using `shift(1).rolling(k)` (prevents using current/future).
- Time features (calendar features from `date` or cyclical time index per city).

**Models**
- Classic ML: Linear Regression, Random Forest, XGBoost (optional), LightGBM (optional).
- Deep learning: RNN / LSTM / GRU / Hybrid (optional BiLSTM).

**Metrics**
- RMSE, MAE, MAPE, R²
- For discrete AQI (1–5): also report **rounded-level accuracy** (round predictions to 1–5).

---

## Repository layout 
```
.
├─ README.md
├─ MODEL_CARD.md
├─ EVALUATION.md
├─ data/                    # optional: do NOT commit large/private datasets
├─ research/                # notebooks / experiments (not graded for code quality)
├─ src/
│  └─ aqi_forecast/         # reusable library code (features, training, eval)
└─ scripts/
   ├─ train.py
   ├─ evaluate.py
   ├─ ablation_near_neighbor.py
   └─ seasonal_event_insights.py
```

---

## How to run

### 1) Install
```bash
python -m venv .venv
# Windows: .venv\Scripts\activate
# Mac/Linux: source .venv/bin/activate
pip install -U pip
pip install -r requirements.txt
```

### 2) Train (example)
```bash
python scripts/train.py --data data/air_pollution_data.csv --target aqi --horizon_days 3
```

### 3) Evaluate (example)
```bash
python scripts/evaluate.py --data data/air_pollution_data.csv --target aqi --horizon_days 3
```

### 4) Seasonal + event-based insights (STEP 12 add-on)
Answers: which months are risky? do Diwali / crop burning windows show spikes?
```bash
python scripts/seasonal_event_insights.py --data data/air_pollution_data.csv --target aqi
```
### 5) Shap charts-Evaluate the safety of prediciton
---

## Expected outputs

- `model_comparison_results_full.csv` (metrics table)
- best-model forecast plot (True vs Pred)
- monthly risk plot + event vs non-event plots (STEP 12)

---

## Notes on interpretation
- AQI is discrete (1–5), so regression R² may be modest even when MAE/accuracy look okay.
- Always compare against a simple baseline (mean / last-value) to confirm improvement.
- If SHAP or feature importance shows any “future” or target-derived variables, treat it as a red flag.
