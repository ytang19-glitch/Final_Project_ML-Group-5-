
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
