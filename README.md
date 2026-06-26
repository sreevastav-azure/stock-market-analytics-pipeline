# 📈 Stock Market Analytics Pipeline

An end-to-end Azure Data Engineering pipeline that ingests real-time stock market data from the Alpha Vantage API, processes it through a Bronze → Silver → Gold medallion architecture, and surfaces analytics-ready gold tables in Databricks Unity Catalog.

---

## 🏗️ Architecture

```
Alpha Vantage API (AAPL)
        │
        ▼
┌─────────────────────┐
│   Azure Data Factory │  ← HTTP Linked Service
│   PL_StockMarket_    │  ← Scheduled daily at 6:00 AM IST
│   Ingestion          │
└────────┬────────────┘
         │ Copy Activity
         ▼
┌─────────────────────┐
│   ADLS Gen2          │  ← Bronze Container
│   (itsmystorage)     │  ← Raw JSON files
│   /bronze/stock_     │
│   market/            │
└────────┬────────────┘
         │ Databricks Notebook 1
         ▼
┌─────────────────────┐
│   Databricks         │  ← Unity Catalog
│   silver_stock_      │  ← Flattened Delta Table
│   prices             │  ← 100 trading days
└────────┬────────────┘
         │ Databricks Notebook 2
         ▼
┌─────────────────────┐
│   Gold Layer         │  ← Unity Catalog
│   ├─ gold_daily_     │  ← Daily % returns
│   │  returns         │
│   ├─ gold_moving_    │  ← 7-day & 21-day MA
│   │  averages        │
│   └─ gold_volume_    │  ← Volume vs 7-day avg
│      analysis        │
└─────────────────────┘
```

---

## 🛠️ Tech Stack

| Component | Technology |
|---|---|
| Cloud Platform | Microsoft Azure |
| Orchestration | Azure Data Factory (ADF) |
| Storage | Azure Data Lake Storage Gen2 |
| Processing | Azure Databricks (PySpark) |
| Table Format | Delta Lake |
| Governance | Unity Catalog |
| Source Control | GitHub (ADF Git Integration) |
| Data Source | Alpha Vantage REST API |
| Language | Python / PySpark |

---

## 📁 Project Structure

```
stock-market-analytics-pipeline/
│
├── pipeline/
│   └── PL_StockMarket_Ingestion.json       # ADF pipeline definition
│
├── linkedService/
│   ├── LS_HTTP_AlphaVantage.json           # Alpha Vantage HTTP connection
│   ├── LS_ADLS_StockMarket.json            # ADLS Gen2 connection
│   └── LS_Databricks_StockMarket.json      # Databricks connection
│
├── dataset/
│   ├── DS_HTTP_AlphaVantage.json           # Source dataset
│   └── DS_ADLS_Bronze_Stock.json           # Sink dataset
│
├── notebooks/
│   ├── 01_bronze_to_silver_stock.py        # Bronze → Silver transformation
│   └── 02_silver_to_gold_stock.py          # Silver → Gold analytics
│
└── README.md
```

---

## 🥉 Bronze Layer

- Raw JSON ingested from Alpha Vantage `TIME_SERIES_DAILY` endpoint
- Stored in ADLS Gen2 bronze container: `/bronze/stock_market/`
- Filename pattern: `stock_AAPL_YYYY-MM-DD-HH-mm.json`
- Contains nested JSON with OHLCV data per trading date

---

## 🥈 Silver Layer

**Table:** `stock_market_catalog.silver.silver_stock_prices`

Flattened and cleaned Delta table with one row per trading day.

| Column | Type | Description |
|---|---|---|
| symbol | string | Stock ticker (AAPL) |
| trade_date | date | Trading date |
| open | double | Opening price |
| high | double | Highest price |
| low | double | Lowest price |
| close | double | Closing price |
| volume | long | Shares traded |

---

## 🥇 Gold Layer

### 1. `gold_daily_returns`
Daily closing price with percentage change from previous day.

| Column | Description |
|---|---|
| symbol | Stock ticker |
| trade_date | Trading date |
| open / high / low / close | OHLC prices |
| volume | Shares traded |
| daily_return_pct | % change from previous close |

### 2. `gold_moving_averages`
Rolling moving averages for trend analysis.

| Column | Description |
|---|---|
| symbol | Stock ticker |
| trade_date | Trading date |
| close | Closing price |
| ma_7 | 7-day moving average |
| ma_21 | 21-day moving average |

### 3. `gold_volume_analysis`
Volume trends compared to 7-day average.

| Column | Description |
|---|---|
| symbol | Stock ticker |
| trade_date | Trading date |
| volume | Actual volume |
| avg_volume_7d | 7-day average volume |
| volume_vs_avg | Volume as % of 7-day average |

---

## ⚙️ ADF Pipeline

**Pipeline:** `PL_StockMarket_Ingestion`

```
Copy_AlphaVantage_to_Bronze → Run_Bronze_to_Silver → Run_Silver_to_Gold
```

- **Trigger:** Daily schedule at 6:00 AM IST
- **Source:** Alpha Vantage HTTP API (TIME_SERIES_DAILY, compact output)
- **Sink:** ADLS Gen2 bronze container (dynamic filename with timestamp)
- **Git Integration:** ADF connected to this GitHub repo (`adf_publish` branch)

---

## 🚀 How to Run

### Prerequisites
- Azure subscription with ADF, ADLS Gen2, and Databricks workspace
- Alpha Vantage API key (free at [alphavantage.co](https://www.alphavantage.co))
- Unity Catalog enabled on Databricks

### Setup Steps
1. Clone this repo and connect ADF via Git configuration
2. Create ADLS Gen2 storage account with `bronze`, `silver`, `gold` containers
3. Create Unity Catalog schemas: `stock_market_catalog.bronze/silver/gold`
4. Configure linked services with your credentials
5. Import Databricks notebooks to your workspace
6. Publish the ADF pipeline and activate the daily trigger

---

## 📊 Sample Output

**Gold Daily Returns (sample)**
```
+------+----------+-------+-------+-------+------+--------+----------------+
|symbol|trade_date|   open|   high|    low| close|  volume|daily_return_pct|
+------+----------+-------+-------+-------+------+--------+----------------+
|  AAPL|2026-02-02| 260.03| 270.49|259.205|270.01|73913425|            NULL|
|  AAPL|2026-02-03|  269.2|271.875| 267.61|269.48|63799388|            -0.2|
|  AAPL|2026-02-04|272.285| 278.95|272.285|276.49|90545710|             2.6|
+------+----------+-------+-------+-------+------+--------+----------------+
```

---

## 👤 Author

**Vastav** | Azure Data Engineer  
GitHub: [@sreevastav-azure](https://github.com/sreevastav-azure)

---

## 🔗 Other Portfolio Projects

- [IPL Cricket Analytics Pipeline](https://github.com/sreevastav-azure/ipl-cricket-analytics)
- [E-Commerce Sales Analytics Pipeline](https://github.com/sreevastav-azure/ecommerce-sales-analytics)
