# Retail Data Warehouse Analytics with DuckDB

![Python](https://img.shields.io/badge/Python-3.12-blue?logo=python&logoColor=white)
![DuckDB](https://img.shields.io/badge/DuckDB-1.5-yellow?logo=duckdb&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-3.0-150458?logo=pandas&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

---

## Business Context

Grocery retailers process millions of transactions daily. Understanding purchasing behavior at scale requires analytical engines that can query large datasets in seconds without the overhead of a full database server. This project builds a local data warehouse using DuckDB, an embedded analytical database that brings data warehouse performance to a Jupyter notebook.

Using 3.4 million grocery orders from 200,000+ customers across 50,000 products (Instacart dataset), we demonstrate a complete analytical pipeline: ETL (CSV to Parquet to DuckDB), star schema modeling, SQL-first analytics with window functions and CTEs, and a performance benchmark against pandas.

---

## Key Results

| Metric | Value |
|--------|-------|
| Orders analyzed | 3,421,083 |
| Product-order rows | 32,434,489 |
| Unique customers | 206,209 |
| Products in catalog | 49,688 |
| Star schema tables | 6 (3 fact + 3 dimension) |
| Schema creation time | 3.5 seconds from Parquet |
| CSV to Parquet compression | 4.9x on largest file |
| DuckDB vs pandas speedup | **3.9x average** (up to 5.4x on JOINs) |

### Key Business Insights

> **7 out of 21 departments generate 80% of all orders** (Pareto principle confirmed). Produce alone accounts for 29% of total volume. Banana is the single most ordered product with 472,565 orders and an 84.4% reorder rate.

> **Sunday and Monday are the busiest days**, not Saturday. Peak hour is Sunday 2 PM (~54,000 orders). The dead zone (1-5 AM) has less than 1,000 orders per hour. This pattern directly informs delivery fleet scheduling and server scaling.

> **58.97% of all product orders are reorders.** Dairy and eggs lead with a 66% reorder rate (weekly staples), while personal care trails at 31% (occasional purchases). Customer loyalty is driven by habitual repurchasing of staple products.

---

## Business Discoveries

- **Customer segmentation:** 54.3% of customers shop monthly, 33.7% biweekly, 12% weekly. The weekly segment (24,670 customers) represents the highest-value cohort despite being the smallest.
- **Power users:** Only 5.6% of customers (11,464) have 50+ orders, but they average 68.9 orders each, making them the core revenue drivers worth retaining.
- **Basket size:** Median of 8 products per order, mean of 10.1. The right-skewed distribution indicates most orders are small, targeted replenishment trips rather than large weekly hauls.
- **Order frequency:** Median gap between orders is 7 days (weekly cycle). 50.4% of all reorders happen within a week of the previous order.

---

## Technical Achievements

### ETL Pipeline
- CSV (680 MB total) converted to Parquet (145 MB) with 4.7x overall compression
- Parquet preserves data types, eliminating re-parsing overhead on every load
- Star schema (fact + dimension tables) created in DuckDB in 3.5 seconds

### SQL-First Analytics
All analysis performed in SQL through DuckDB. Techniques demonstrated:
- Window functions: `RANK() OVER(PARTITION BY ...)` for top-N per group
- CTEs: multi-step customer segmentation with readable query structure
- Running totals: `SUM() OVER(ORDER BY ...)` for Pareto (80/20) analysis
- Cross-tabulation: `CASE WHEN` pivot for day x hour heatmap
- Subqueries: scalar subqueries for dataset overview metrics

### Performance Benchmark (DuckDB vs pandas on 32M rows)
| Query Type | DuckDB | pandas | Speedup |
|------------|--------|--------|---------|
| GROUP BY (basket size) | 1.01s | 2.67s | 2.6x |
| JOIN + GROUP BY (top products) | 3.43s | 18.40s | **5.4x** |
| Window function (rank per dept) | 3.50s | 12.62s | 3.6x |

Average speedup: **3.9x**. JOIN operations showed the largest advantage because DuckDB's columnar engine handles multi-table joins natively, while pandas must materialize the full merged DataFrame in memory (990 MB).

---

## Project Structure

```
retail-data-warehouse-duckdb/
├── data/
│   ├── raw/                 <- Original CSVs from Kaggle (not tracked)
│   ├── parquet/             <- Converted Parquet files (generated)
│   ├── retail_warehouse.duckdb  <- Persistent DuckDB warehouse (generated)
│   └── DOWNLOAD_HERE.txt
├── notebooks/
│   └── retail_analytics.ipynb
├── outputs/
│   ├── order_timing_patterns.png
│   ├── basket_size_distribution.png
│   ├── top_20_products.png
│   ├── reorder_rate_by_department.png
│   ├── customer_segmentation.png
│   ├── department_pareto.png
│   ├── order_heatmap_hour_day.png
│   └── duckdb_vs_pandas_benchmark.png
├── .gitignore
├── pyproject.toml
├── uv.lock
└── README.md
```

---

## Technical Pipeline

1. Environment setup with uv (modern Python package manager) + DuckDB initialization with persistent database file
2. ETL: CSV to Parquet conversion with PyArrow (4.9x compression on largest file)
3. Star schema creation: 3 dimension tables (departments, aisles, products) + 3 fact tables (orders, order_products_prior, order_products_train)
4. Schema inspection and data dictionary
5. Business analytics via SQL: order timing patterns, basket size distribution, top products, reorder behavior
6. Advanced SQL: window functions (RANK per department), CTEs (customer segmentation), running totals (Pareto analysis), cross-tabulation (hour x day heatmap)
7. Performance benchmark: DuckDB vs pandas on identical queries across 32M rows
8. Documentation and visualization export

---

## Tech Stack

| Tool | Usage |
|------|-------|
| Python 3.12 | Core language |
| DuckDB 1.5 | Embedded analytical database (SQL engine) |
| PyArrow 24.0 | Parquet file I/O and columnar data format |
| pandas 3.0 | Visualization data preparation only |
| matplotlib / seaborn | Charts and heatmaps |
| uv (Astral) | Modern Python package manager |

---

## Dataset

- **Name:** Instacart Market Basket Analysis
- **Source:** [Kaggle](https://www.kaggle.com/datasets/yasserh/instacart-online-grocery-basket-analysis-dataset)
- **Volume:** 3.4M orders, 32M order-product rows, 200K+ users, 50K products
- **Structure:** 6 relational CSVs forming a natural star schema
- **License:** Non-commercial use

---

## How to Reproduce

```bash
# 1. Clone the repository
git clone https://github.com/Lucas-Coutinhob/retail-data-warehouse-duckdb.git
cd retail-data-warehouse-duckdb

# 2. Install uv (if not installed)
# Windows:
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
# Linux/Mac:
# curl -LsSf https://astral.sh/uv/install.sh | sh

# 3. Install dependencies
uv sync

# 4. Download the dataset
# Access: https://www.kaggle.com/datasets/yasserh/instacart-online-grocery-basket-analysis-dataset
# Save all CSVs in the data/raw/ folder

# 5. Run the notebook
uv run jupyter notebook notebooks/retail_analytics.ipynb
```

---

## Author

**Lucas Coutinho Boros**
Data Scientist in Training | Bachelor's in Data Science and AI - IESB

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Lucas%20Boros-blue?logo=linkedin)](https://www.linkedin.com/in/datalucasboros)
[![GitHub](https://img.shields.io/badge/GitHub-Lucas--Coutinhob-black?logo=github)](https://github.com/Lucas-Coutinhob)