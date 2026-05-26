# ✈️ SkyPredict — Flight Delay & Price Forecasting at Scale

## 📌 Overview

**SkyPredict** is a big data flight intelligence platform that predicts flight delays and ticket price ranges for US domestic routes , powered by **Apache Spark** on **~30M rows** of BTS On-Time Performance data (2019–2023), served through a live **Streamlit web app** with real-time weather integration.

Given an origin airport, destination airport, and travel date, SkyPredict tells you:
- 🟢/🟡/🔴 **Expected delay risk** (low / medium / high)
- 💰 **Price forecast** and seasonal price trends
- 📋 **Buy or don't buy** recommendation with justification
- 🏆 **Best and worst airports** — cheapest routes, highest-delay airports, times to avoid

---

## 📊 Model Performance

| Model | Algorithm | Metric | Result |
|-------|-----------|--------|--------|
| **Delay Predictor** | Random Forest Classifier | AUC | **0.8534** |
| **Price Forecaster** | Random Forest Regressor | RMSE | **$24.42** |

**Data scale:**
- Training set: **24.6M rows**
- Test set: **6.1M rows**
- Total processed: **~30.7M flight records**

---

## 🏗️ Architecture

```
Raw BTS CSVs (2019–2023, ~30M rows)
        ↓
Spark ETL (PySpark) — clean, normalize, partition by year/month
        ↓
bts_clean.parquet (columnar, partition-pruned)
        ↓
Feature Engineering — centrality scores, season, time of day, route stats
        ↓
bts_features.parquet
        ↓
  ┌─────────────────┐    ┌──────────────────────┐
  │ Delay Predictor │    │  Price Forecaster     │
  │ RF Classifier   │    │  RF Regressor         │
  │ AUC: 0.8534     │    │  RMSE: $24.42         │
  └────────┬────────┘    └──────────┬────────────┘
           └──────────┬─────────────┘
                      ↓
           Saved as PipelineModel (Parquet)
                      ↓
        Streamlit Web App + Open-Meteo Live Weather
                      ↓
         User query → delay risk + price + recommendation
```

---

## ✨ Features

### 📈 Historical Analytics
- Airport with the largest number of departure and arrival flights
- Busiest airports overall (departures + arrivals combined)
- Top 10 airports with most departure and arrival delays
- Monthly delay trends and seasonal price patterns
- Delay vs weather correlation analysis
- Airport-month delay heatmaps

### 🔍 Per-Itinerary Predictions
For any user-selected origin → destination → date:

| Output | Description |
|--------|-------------|
| **Expected Delays** | Risk level (low/medium/high) based on historical route patterns |
| **Price Fluctuation** | Seasonal fare behavior, typical price range |
| **Recommendations** | Best airports, cheaper alternatives, buffer time suggestions |
| **Buy or Don't Buy** | Clear recommendation with short justification |
| **Best Report** | Best airports, cheapest flights, best value routes |
| **Worst Report** | Highest-delay airports, expensive routes, times to avoid |

### 🌦️ Live Weather Integration
Real-time weather for origin and destination airports via **Open-Meteo API**, incorporated into delay risk scoring at query time.

---

## 🗂️ Data Sources

| Source | Dataset | Volume | Use |
|--------|---------|--------|-----|
| **BTS TranStats** | Reporting Carrier On-Time Performance, 2019–2023 | ~30M rows | Primary delay + airline data |
| **OpenSky Network (Zenodo)** | Daily flight lists | — | Route coverage cross-check |
| **Open-Meteo API** | Forecast & current weather | Live | Real-time weather at query time |

> All data is historical and real — no synthetic or randomly generated flight records.

---

## 🤖 ML Pipeline

### Delay Predictor (Binary Classifier)

**Label:** `is_delayed` = 1 if departure delay > 15 minutes, 0 otherwise

**Features:**
- Airline (`Reporting_Airline` → `airline_idx` via StringIndexer)
- Origin airport (`Origin` → `origin_idx`)
- Destination airport (`Dest` → `dest_idx`)
- Airport network centrality score (`origin_centrality`)
- Season (Summer / Winter / Spring / Fall)
- Distance, time of day

**Model:** `RandomForestClassifier` (Spark MLlib) in a `Pipeline` with StringIndexers + VectorAssembler

**Result: AUC = 0.8534**

---

### Price Forecaster (Regressor)

**Target:** `price_proxy` — derived from BTS distance, delay, and season:

```python
price_proxy = Distance * 0.12 + abs(DepDelay) * 0.5
            + 50 (Summer) / 30 (Winter) / 10 (otherwise)
```

> Note: BTS does not publish per-itinerary ticket prices. This proxy is derived from distance, delay, and seasonal signals — the closest available signal at this volume.

**Model:** `RandomForestRegressor` (Spark MLlib)

**Result: RMSE = $24.42**

---

## 🛠️ Tech Stack

![PySpark](https://img.shields.io/badge/Apache_Spark-E25A1C?style=flat&logo=apachespark&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-FF4B4B?style=flat&logo=streamlit&logoColor=white)

| Technology | Purpose |
|-----------|---------|
| **Apache Spark / PySpark 3.5** | Distributed ETL and ML training over ~30M BTS records |
| **Spark MLlib** | Random Forest classifier (delay) + Random Forest regressor (price) |
| **Apache Parquet** | Columnar storage for cleaned data and serialized PipelineModels |
| **Python 3.9–3.11** | Driver, ETL scripts, web app |
| **Streamlit** | Interactive prediction web interface |
| **Open-Meteo API** | Live weather for origin and destination airports |
| **Google Colab** | Training environment (8GB driver + executor memory) |


## 💾 Data Storage Design

The pipeline follows: **Raw CSV → Spark ETL → Parquet → Spark ML → PipelineModel**

- **Parquet** was chosen over CSV/JSON/databases for this workload because:
  - Columnar format enables faster analytical scans
  - Partition pruning by year/month means Spark only reads relevant data
  - Avoids impedance mismatch of pushing 30M analytical rows into a document store
  - PipelineModel directories are themselves Parquet (metadata JSON + fitted stage files)

- **No relational or document database** — for an analytical workload at this scale, Parquet on disk is the standard and most efficient choice.

---

## 🏷️ Topics

`apache-spark` `pyspark` `big-data` `flight-delay-prediction` `price-forecasting` `random-forest` `spark-mllib` `streamlit` `parquet` `etl` `open-meteo` `bts` `airline-data` `machine-learning` `python`
