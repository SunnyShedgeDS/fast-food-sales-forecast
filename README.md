# 🍗 Fast Food Restaurant — Sales Forecasting

> **End-to-end machine learning pipeline to forecast daily restaurant sales one week ahead, enabling smarter stock ordering and staff rostering.**

---

## 📌 Project Overview

Restaurants waste thousands of pounds every year on overstocked ingredients and inefficient staffing — because they rely on gut instinct rather than data. This project builds a **7-day sales forecasting model** that combines historical sales, weather data, and calendar events (bank holidays, school holidays, special events) to predict daily revenue with ~96% accuracy.

Built as a prototype for a fast food restaurant using 3 years of daily sales data.

---

## 🎯 Business Problem

| Problem | Impact |
|---|---|
| Over-ordering stock | Food waste, margin loss |
| Under-rostering staff | Poor service, lost sales |
| No visibility of next week | Reactive, not proactive management |

**Solution:** A machine learning model that every Monday produces a day-by-day forecast for the coming week, giving management actionable numbers.

---

## 📊 Demo Output

```
================================================
  7-DAY SALES FORECAST
================================================
  Monday      £  3,842  ████████████████
  Tuesday     £  3,654  ███████████████
  Wednesday   £  4,021  █████████████████
  Thursday    £  4,388  ██████████████████
  Friday      £  5,210  ██████████████████████
  Saturday    £  6,105  █████████████████████████
  Sunday      £  5,487  ██████████████████████
------------------------------------------------
  WEEK TOTAL  £ 32,707
================================================

Model Accuracy: 96.1% | MAE: ±£155/day
```

---

## 🗂️ Data Sources

Three CSV files feed the pipeline:

| File | Description | Key Columns |
|---|---|---|
| `sales_data.csv` | Daily sales history (2–3 years) | `date`, `total_sales`, `transaction_count`, `dine_in_sales`, `takeaway_sales` |
| `calendar_events.csv` | Holidays & special events | `date`, `day`, `specials`, `holiday_for_adults`, `holiday_for_kids` |
| `weather_data.csv` | Daily weather conditions | `date`, `temperature`, `humidity`, `sky_status` |

> ⚠️ Real data is not included. The notebook generates realistic dummy data automatically in Cell 1.

---

## 🏗️ Pipeline Architecture

```
┌─────────────────┐   ┌──────────────────┐   ┌─────────────────┐
│  sales_data.csv │   │calendar_events.csv│   │ weather_data.csv│
└────────┬────────┘   └────────┬─────────┘   └────────┬────────┘
         │                     │                       │
         └──────────────┬──────┘───────────────────────┘
                        ▼
              ┌─────────────────┐
              │  Merge on date  │  Stage 1
              └────────┬────────┘
                       ▼
              ┌─────────────────┐
              │   Data Cleaning │  Stage 2 — outliers, nulls
              └────────┬────────┘
                       ▼
              ┌──────────────────────┐
              │  Feature Engineering │  Stage 3 — 18 features
              │  • Lag sales 1/7/14d │
              │  • Rolling avg 7/28d │
              │  • Day of week       │
              │  • Weather score     │
              │  • Holiday flags     │
              └────────┬─────────────┘
                       ▼
              ┌─────────────────┐
              │  XGBoost Model  │  Stage 4 — trained on 3 years
              └────────┬────────┘
                       ▼
              ┌─────────────────┐
              │   Validation    │  Stage 5 — MAE, MAPE on held-out data
              └────────┬────────┘
                       ▼
              ┌─────────────────────────────────┐
              │       7-Day Forecast             │  Stage 6
              │  Stock plan · Staff schedule     │
              └─────────────────────────────────┘
```

---

## ⚙️ Feature Engineering

18 features are engineered from the 3 raw data sources:

| Category | Features |
|---|---|
| **Time** | Day of week, month, week number, is_weekend |
| **Sales history** | Lag 1 day, lag 7 days, lag 14 days |
| **Trend** | 7-day rolling avg, 28-day rolling avg |
| **Weather** | Temperature, humidity, sky score (1–5) |
| **Calendar** | Bank holiday, school holiday, has special event, is Valentine's, is Christmas |

---

## 🤖 Model

| Parameter | Value | Reason |
|---|---|---|
| Algorithm | XGBoost | Best performance on tabular time-series data |
| `n_estimators` | 300 | Optimal from 200-combo grid search |
| `learning_rate` | 0.01 | Slow learning → better generalisation |
| `max_depth` | 5 | Balanced complexity |
| `subsample` | 0.9 | Reduces overfitting |
| `colsample_bytree` | 0.9 | Feature sampling per tree |
| `min_child_weight` | 3 | Prevents overfitting on rare events |
| `reg_alpha` | 0.1 | L1 regularisation |
| `reg_lambda` | 2.0 | L2 regularisation |

These hyperparameters were found via a 200-combination random grid search.

---

## 📈 Results

| Metric | Value |
|---|---|
| MAE (avg £ off per day) | £155 |
| MAPE (avg % off per day) | 3.9% |
| **Accuracy** | **96.1%** |
| Training data | 1,007 days |
| Test data | 61 days (Nov–Dec 2024) |

---

## 🗺️ Notebook Structure

| Cell | Description |
|---|---|
| 1 | Import libraries & plot styling |
| 2 | Load & merge all 3 CSV sources |
| 3 | Data cleaning (outliers, nulls) |
| 4 | Feature engineering (18 features) |
| 5 | XGBoost model training |
| 6 | Validation — actual vs predicted chart |
| 7 | Feature importance chart |
| 8 | **7-day forecast** (edit weather/events here) |
| 9 | Forecast chart with confidence band |
| 10 | Day-of-week sales pattern |
| 11 | 3-year monthly trend |
| 12 | Summary dashboard |

---

## 🚀 Quick Start

### 1. Clone the repo
```bash
git clone https://github.com/SunnyShedgeDS/fast-food-sales-forecast.git
cd fast-food-sales-forecast
```

### 2. Install dependencies
```bash
pip install -r requirements.txt
```

### 3. Open the notebook
```bash
jupyter notebook Fast_Food_Rest_forecast.ipynb
```

### 4. Run all cells
The notebook generates dummy data automatically. To use real data, replace the 3 CSVs in the same folder and skip Cell 1 (dummy data generation).

---

## 📦 Requirements

```
pandas
numpy
scikit-learn
xgboost
matplotlib
seaborn
jupyter
```

Install with:
```bash
pip install -r requirements.txt
```

---

## 🔮 Future Improvements

- [ ] Live weather API integration (Open-Meteo / Met Office)
- [ ] Hourly sales breakdown for more granular forecasting
- [ ] Brighton local events calendar feed
- [ ] Streamlit web dashboard for non-technical managers
- [ ] Automated weekly retraining pipeline
- [ ] Stock and staffing recommendation engine on top of forecast

---

## 👤 Author

**Sunny Shedge**  
M.Sc. Data Science — University of Sussex  
3+ years experience as Product Analyst  

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?logo=linkedin)](https://linkedin.com/in/sunnyshedge)
[![GitHub](https://img.shields.io/badge/GitHub-SunnyShedgeDS-black?logo=github)](https://github.com/SunnyShedgeDS)

---

## 📄 License

MIT License — free to use, adapt, and build on.
