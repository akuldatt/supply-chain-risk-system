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
- Implemented in `etl/extract.py` with centralized logging (`logs/etl.log`)

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

### Load
- Cleaned data loaded into SQLite (`data/processed/supply_chain.db`) as `fact_orders`
- Full pipeline orchestrated via `etl/run_pipeline.py` (single entry point)

### Data Grain
The dataset is at **order-item level**, not order level. `Order Id` repeats
across rows when an order contains multiple products; `Order Item Id` is the
true unique identifier and is used as the primary key for quality checks.


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
## 4. Data Understanding

**Dataset:** DataCo Smart Supply Chain, 180,519 orders and 53 columns.

### Key Findings
| Finding | Detail |
|---|---|
| Overall late-delivery rate | 54.83% (57.29% excluding canceled orders) |
| Delivery status split | Late 98,977 · Advance 41,592 · On time 32,196 · Canceled 7,754 |
| Average shipping time | 3.50 days actual vs 2.93 days scheduled |
| Shipping modes | Standard 107,752 · Second 35,216 · First 27,814 · Same Day 9,737 |

### Data Quality Issues
- `Product Description`: 100% null, dropped
- `Order Zipcode`: ~86% null, dropped
- `Customer Lname` (8) and `Customer Zipcode` (3): minor nulls, handled in transform
- Sensitive fields (email, password) are masked and will be removed
- Inconsistent labels (e.g. `EE. UU.`) will be standardized

### Data Leakage Risks (for ML)
`Late_delivery_risk` and `Days for shipping (real)` are only known after
delivery and are excluded from model features to prevent target leakage.

### Analytical Decision
Canceled orders are excluded when calculating late-delivery rate, since they
were never delivered.

### Dataset
Download the DataCo Smart Supply Chain dataset from
[Kaggle](https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis)
and place the CSV files in `data/raw/`. Raw files are excluded from version
control via `.gitignore`.

## 5. How to Run the Pipeline
```bash
python -m etl.run_pipeline
```
This runs extract, transform, quality checks, and load in sequence, and
writes logs to `logs/etl.log`.

## 6. Diagnostic Analysis

### Finding: Shipping Mode Delay (Root Cause)

Late delivery rate is inversely related to shipping speed promised: First
Class (100% late), Second Class (79.8%), Same Day (47.9%), Standard Class
(39.8%). Analysis of actual vs scheduled days reveals the true fulfillment
time is roughly constant (~4 days for Second/Standard Class, ~2 days for
First Class) regardless of the shipping mode selected. This indicates the
delay problem stems from unrealistic delivery promises rather than
inconsistent operational performance — a promise/SLA design issue, not a
capacity issue.

### Finding: Product Category Delay

Late delivery rate is uniform across the top 15 product categories by volume
(57%-60%), similar to the region finding. This rules out product-specific
causes (e.g., specific suppliers or item types) and further confirms that
**shipping mode / delivery promise design is the dominant root cause**, not
product or geography.

### Finding: Region-wise Delay

Late delivery rate is fairly uniform across regions (51%-60%), indicating the
delay problem is systemic rather than region-specific. This rules out simple
geography-based fixes and points toward operational factors (shipping mode,
processing time) as more likely root causes.

### Finding: No Meaningful Time-Based Pattern

Late delivery rate varies only slightly by calendar month (56.7%-58.0%) and
by weekday (56.6%-57.8%). A year-over-year heatmap shows no consistent
seasonal pattern — the month with the highest late rate changes every year
(Sep in 2015, Jun in 2016, Aug in 2017), indicating this variation is random
rather than a genuine seasonal effect. Weather data, which could explain
short-term spikes, is not available in this dataset (see Limitations below).

### Diagnostic Conclusion

Across four dimensions tested (region, product category, month, weekday),
only **shipping mode** shows a strong, consistent relationship with late
delivery (40% to 100% late rate). This confirms the root cause is structural
— unrealistic delivery-time promises by shipping mode — rather than
geographic, seasonal, or product-specific factors.

### Limitation: Weather Data Not Available

This dataset does not include weather information. Seasonal or
weather-related delay causes (e.g., rain, storms) cannot be tested directly.
Order volume by month was checked as an indirect proxy for demand-driven
delays but showed no strong relationship with late-delivery rate.
