# Supply Chain Risk & Delivery Optimization System

An end-to-end analytics project: ETL pipeline, diagnostic analysis,
machine learning, and prescriptive recommendations.

## 1. Business Problem

A global retail company is experiencing rising late deliveries, which reduce
customer satisfaction and increase operational costs. Management lacks
visibility into the root causes, cannot anticipate which orders are at risk,
and has no data-driven strategy to reduce delays.

### Objectives
1. **Descriptive:** Measure current delivery performance.
2. **Diagnostic:** Identify the root causes of delivery delays.
3. **Predictive:** Predict late-delivery risk before an order ships.
4. **Prescriptive:** Recommend actions with an estimated financial impact.
5. **Early Warning:** Detect emerging risks and anomalies proactively.

### Key Performance Indicators (KPIs)
| KPI | Definition |
|---|---|
| On-Time Delivery Rate | % of orders delivered within the scheduled time |
| Late Delivery % | % of orders delivered after the scheduled time |
| Average Delay (days) | Actual shipping days minus scheduled shipping days |
| Revenue at Risk | Total sales value of late orders |
| Profit Impact of Delays | Profit lost on late or problematic orders |

## 2. Data Pipeline (ETL)

The pipeline ingests raw supply chain data, cleans and validates it, and loads
it into a star-schema database for analytics and machine learning.

```
Raw CSV → Extract → Transform → Quality Checks → Load → SQL Database
```

### Extract
- Source: DataCo Smart Supply Chain dataset (CSV)
- Read using pandas (`encoding='latin-1'`); each run is logged.

### Transform
- Standardized column names to `snake_case`
- Converted order and shipping dates to datetime
- Handled null values and duplicate records
- Removed irrelevant and sensitive columns (emails, passwords, image URLs)
- Engineered features: `delay_days`, `is_late`, `order_month`

### Data Quality Checks
- Row count reconciliation between source and target
- No null primary keys
- No duplicate order IDs
- Dates within a valid range
- No negative sales values

### Load (Star Schema)
| Table | Type | Description |
|---|---|---|
| fact_orders | Fact | Sales, profit, quantity, delay, delivery status |
| dim_customer | Dimension | Customer details and segment |
| dim_product | Dimension | Product, category, department, price |
| dim_location | Dimension | Market, region, country |
| dim_shipping | Dimension | Shipping mode and scheduled days |
| dim_date | Dimension | Date, month, quarter, year, weekday |

## 3. Project Setup

### Prerequisites
- Python 3.10+
- GitHub Codespaces or a local environment

### Installation
```bash
git clone https://github.com/akuldatt/supply-chain-risk-system.git
cd supply-chain-risk-system
pip install -r requirements.txt
```

### Dataset
Download the DataCo Smart Supply Chain dataset from
[Kaggle](https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis)
and place the CSV files in `data/raw/`. Raw files are excluded from version
control via `.gitignore`.
