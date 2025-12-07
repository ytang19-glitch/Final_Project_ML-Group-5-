# Evaluation & Leakage Audits

## Setup
- **Time split:** last 20% as test (chronological)
- **Sequence length (DL):** 24
- **Metrics:** RMSE, MAE, MAPE, R²; plus rounded-level accuracy for AQI ∈ {1..5}

## Baseline sanity check
Mean-baseline should give **R² ~ 0** (or slightly negative). That confirms the metric pipeline is not broken.

## Leakage / “cheating” checks
### 1) A/B ablation (near-neighbor shortcuts)
Compare performance under:
- A: all features
- B: drop `PM2.5_lag1`
- C: drop `lag1/2/3` + `roll3/roll5`
- D: drop all `lag*` + rolling means

Interpretation:
- **Big R² drop** ⇒ model mainly uses near-neighbor shortcuts
- **No R² drop** ⇒ high score comes from other valid signals (seasonality/city/weather), or the task is easy

### 2) Forbidden feature audit (red flags)
- Any feature that directly includes future target
- Any derived variable that improperly uses the target definition

## Step 12: Seasonal & event-based insights
Goal: identify **dangerous months** and whether event windows (Diwali / crop burning season) align with spikes.

```python
# df_plot must contain parsed date, City, and aqi column
df_plot["_date"] = pd.to_datetime(df_plot[date_col], errors="coerce", dayfirst=True)
df_plot = df_plot.dropna(subset=["_date"]).copy()
df_plot["month"] = df_plot["_date"].dt.month

TH = 4  # "dangerous" AQI threshold (adjust as needed)
monthly = df_plot.groupby("month")["aqi"].agg(["count","mean","median"])
monthly["pct_high(>=4)"] = df_plot.assign(high=(df_plot["aqi"]>=TH)).groupby("month")["high"].mean()

# Event proxy: Oct–Nov often contains Diwali + crop burning windows (year-specific)
df_plot["is_oct_nov"] = df_plot["_date"].dt.month.isin([10,11])
event_stats = df_plot.groupby("is_oct_nov")["aqi"].mean()

print(monthly.sort_values("mean", ascending=False))
print("Mean AQI Oct-Nov vs other:", event_stats.to_dict())
```
