# E-Commerce Sales Analytics Pipeline

## Project Overview

An end-to-end, scalable sales analytics pipeline built with **PostgreSQL**, **Python (pandas)**, and **Streamlit**. This project demonstrates real-world data engineering and analytics skills by processing raw e-commerce transactional data, storing it in a normalized relational database, running complex SQL analytics, and visualizing insights through an interactive dashboard.

**Key Highlights:**
- Normalized PostgreSQL schema designed for scalability
- Python ETL pipeline for data cleaning and loading
- Complex SQL analytics using JOINs, CTEs, and Window Functions
- Customer segmentation using RFM (Recency, Frequency, Monetary) analysis
- Live Streamlit dashboard deployed via GitHub
- Handles 500K+ records efficiently
- 30+ meaningful Git commits showing real development progression

---

## Dataset

**Source:** Kaggle - "Online Retail" Dataset
**URL:** https://www.kaggle.com/datasets/carrie1/ecommerce-data

### Dataset Details
- **Records:** Approximately 541,909 rows
- **Time Period:** 1st Dec 2010 to 9th Dec 2011
- **Columns:** 8 fields
- **Format:** CSV

### Raw Columns
| Column | Description |
|--------|-------------|
| InvoiceNo | Unique invoice number (6-digit). Starts with 'C' for cancelled orders |
| StockCode | Product code (5-digit) |
| Description | Product name/description |
| Quantity | Quantity purchased per transaction |
| InvoiceDate | Date and time of purchase (MM/DD/YY HH:MM) |
| UnitPrice | Price per unit in Sterling |
| CustomerID | Unique customer identifier |
| Country | Customer's country |

### Why This Dataset?
- Real-world transactional data
- Large volume (500K+ rows) - demonstrates scalability
- Contains common data quality issues (missing values, duplicates, cancelled orders) - perfect for ETL practice
- Rich enough for sales analysis, customer segmentation, and time-series trends

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Database | PostgreSQL |
| ETL | Python, pandas, SQLAlchemy, psycopg2 |
| Analytics | SQL (JOINs, CTEs, Window Functions) |
| Dashboard | Streamlit, matplotlib, seaborn, plotly |
| Deployment | GitHub + Streamlit Cloud (free) |

---

## Project Architecture

```
Raw CSV Data (Kaggle Online Retail)
    |
    v
Python ETL Script (pandas)
    |-- Data Cleaning (handle nulls, duplicates, cancelled orders)
    |-- Data Transformation (normalize into tables)
    |-- Data Loading (SQLAlchemy -> PostgreSQL)
    |
    v
PostgreSQL Database (Normalized Schema)
    |-- customers
    |-- products
    |-- orders
    |-- order_items
    |
    v
SQL Analytics Layer
    |-- Monthly Sales Trends
    |-- Top Customers
    |-- Product Category Performance
    |-- Customer Segmentation (RFM Analysis)
    |-- Cohort Retention Analysis
    |
    v
Streamlit Dashboard (Interactive)
    |-- Sales Overview
    |-- Customer Insights
    |-- Product Analytics
    |-- RFM Segments
```

---

## Database Schema (Normalized)

### Entity Relationship Diagram

```
    customers ||--o{ orders : places
    orders ||--o{ order_items : contains
    products ||--o{ order_items : included_in
```

### Table: customers
```sql
CREATE TABLE customers (
    customer_id VARCHAR(20) PRIMARY KEY,
    country VARCHAR(50) NOT NULL,
    first_purchase_date DATE,
    last_purchase_date DATE,
    total_orders INT DEFAULT 0,
    total_spent DECIMAL(10,2) DEFAULT 0.00
);
```

### Table: products
```sql
CREATE TABLE products (
    stock_code VARCHAR(20) PRIMARY KEY,
    description VARCHAR(255),
    unit_price DECIMAL(10,2) NOT NULL,
    category VARCHAR(50)
);
```

### Table: orders
```sql
CREATE TABLE orders (
    invoice_no VARCHAR(20) PRIMARY KEY,
    customer_id VARCHAR(20) NOT NULL,
    invoice_date TIMESTAMP NOT NULL,
    total_amount DECIMAL(10,2) DEFAULT 0.00,
    country VARCHAR(50) NOT NULL,
    is_cancelled BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

### Table: order_items
```sql
CREATE TABLE order_items (
    item_id SERIAL PRIMARY KEY,
    invoice_no VARCHAR(20) NOT NULL,
    stock_code VARCHAR(20) NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    total_price DECIMAL(10,2) GENERATED ALWAYS AS (quantity * unit_price) STORED,
    FOREIGN KEY (invoice_no) REFERENCES orders(invoice_no),
    FOREIGN KEY (stock_code) REFERENCES products(stock_code)
);
```

---

## ETL Pipeline Plan

### Step 1: Data Extraction
- Load raw CSV from Kaggle
- Inspect shape, dtypes, missing values

### Step 2: Data Cleaning
| Issue | Action |
|-------|--------|
| Missing CustomerID | Remove rows (cannot analyze without customer) |
| Missing Description | Fill from matching StockCode, else drop |
| Negative Quantity | Separate as cancelled orders (InvoiceNo starts with 'C') |
| Zero/Negative UnitPrice | Remove invalid pricing rows |
| Duplicates | Drop exact duplicate rows |
| Date Format | Convert InvoiceDate to TIMESTAMP |

### Step 3: Data Transformation
- Create **customers** table: unique customers + country + first/last purchase
- Create **products** table: unique stock codes + descriptions + prices
- Create **orders** table: unique invoices + customer + date + total
- Create **order_items** table: line items with quantities and prices

### Step 4: Data Loading
- Connect to PostgreSQL using SQLAlchemy
- Load tables in order: customers -> products -> orders -> order_items
- Verify row counts match expectations

---

## SQL Analytics Queries Plan

### Query 1: Monthly Sales Trend
Total revenue per month using DATE_TRUNC.

### Query 2: Top 10 Customers by Revenue
JOIN customers + orders + order_items, GROUP BY customer, SUM(total).

### Query 3: Top Selling Products
GROUP BY product, SUM(quantity), ORDER BY desc.

### Query 4: Sales by Country
GROUP BY country, COUNT(DISTINCT customers), SUM(revenue).

### Query 5: Customer RFM Segmentation (CTE + Window Function)
CTE for Recency, Frequency, Monetary. NTILE() OVER() for scoring R, F, M into quintiles. Concatenate scores to create RFM segment.

### Query 6: Cohort Retention Analysis
CTE to find first purchase month per customer. CTE to find purchase months per customer. Calculate cohort index (months since first purchase). Pivot to show retention % per cohort.

### Query 7: Monthly New vs Returning Customers
Window function to identify first purchase date. Compare each order month vs first purchase month.

### Query 8: Average Order Value (AOV) Trend
Monthly AOV = total revenue / number of orders.

### Query 9: Cancelled Orders Analysis
Filter is_cancelled = TRUE. Calculate cancellation rate by country/product.

### Query 10: Customer Lifetime Value Segments
CTE to calculate total spend per customer. CASE statement to categorize: Low, Medium, High, VIP.

---

## Streamlit Dashboard Plan

### Page 1: Sales Overview
- Monthly revenue line chart
- Total orders per month
- Average order value trend
- Sales by country (bar chart / map)

### Page 2: Customer Insights
- Top 10 customers table
- New vs returning customers per month
- Customer distribution by country
- RFM segment pie chart

### Page 3: Product Analytics
- Top 10 best-selling products
- Revenue by product category
- Product performance heatmap

### Page 4: RFM Segmentation
- RFM segment distribution
- Segment characteristics table
- Recommended actions per segment

---

## Project File Structure

```
ecommerce-sales-analytics/
|
|-- data/
|   |-- raw/
|   |   |-- data.csv                 # Original Kaggle dataset
|   |-- cleaned/
|   |   |-- cleaned_data.csv         # After ETL cleaning
|
|-- sql/
|   |-- schema.sql                   # CREATE TABLE statements
|   |-- create_indexes.sql           # Performance indexes
|   |-- 01_monthly_sales.sql         # Monthly sales trend query
|   |-- 02_top_customers.sql         # Top customers query
|   |-- 03_top_products.sql          # Top products query
|   |-- 04_sales_by_country.sql      # Country-wise sales query
|   |-- 05_rfm_segmentation.sql      # RFM analysis query
|   |-- 06_cohort_retention.sql      # Cohort analysis query
|   |-- 07_new_vs_returning.sql      # New vs returning customers
|   |-- 08_aov_trend.sql             # Average order value trend
|   |-- 09_cancelled_orders.sql      # Cancellation analysis
|   |-- 10_clv_segments.sql          # Customer lifetime value
|
|-- etl/
|   |-- extract.py                   # Load raw CSV
|   |-- transform.py                 # Clean and normalize data
|   |-- load.py                      # Load into PostgreSQL
|   |-- main.py                      # Run full ETL pipeline
|   |-- config.py                    # Database credentials
|
|-- dashboard/
|   |-- app.py                       # Main Streamlit app
|   |-- pages/
|   |   |-- 01_sales_overview.py
|   |   |-- 02_customer_insights.py
|   |   |-- 03_product_analytics.py
|   |   |-- 04_rfm_segmentation.py
|   |-- utils.py                     # Database connection helper
|   |-- queries.py                   # SQL queries for dashboard
|
|-- notebooks/
|   |-- eda.ipynb                    # Exploratory data analysis
|   |-- data_cleaning.ipynb          # Cleaning experiments
|   |-- rfm_analysis.ipynb          # RFM analysis experiments
|
|-- tests/
|   |-- test_etl.py                  # ETL unit tests
|   |-- test_queries.py              # SQL query tests
|
|-- requirements.txt                 # Python dependencies
|-- .gitignore                       # Git ignore file
|-- README.md                        # This file
```

---

## Git Commit Plan (30+ Commits)

A repo with 30+ meaningful commits looks authentic and professional. Follow this plan exactly. Make **one commit per task**.

| Commit # | Commit Message | What to Commit |
|----------|----------------|----------------|
| 1 | `Initial commit - project setup` | README.md |
| 2 | `Add .gitignore file` | .gitignore |
| 3 | `Add requirements.txt` | requirements.txt |
| 4 | `Add raw dataset` | data/raw/data.csv |
| 5 | `Add EDA notebook` | notebooks/eda.ipynb |
| 6 | `Add database schema SQL` | sql/schema.sql |
| 7 | `Add database indexes` | sql/create_indexes.sql |
| 8 | `Add ETL config module` | etl/config.py |
| 9 | `Add data extraction script` | etl/extract.py |
| 10 | `Add data cleaning functions` | etl/transform.py |
| 11 | `Add data loading script` | etl/load.py |
| 12 | `Add ETL main pipeline` | etl/main.py |
| 13 | `Add monthly sales query` | sql/01_monthly_sales.sql |
| 14 | `Add top customers query` | sql/02_top_customers.sql |
| 15 | `Add top products query` | sql/03_top_products.sql |
| 16 | `Add sales by country query` | sql/04_sales_by_country.sql |
| 17 | `Add RFM segmentation query` | sql/05_rfm_segmentation.sql |
| 18 | `Add cohort retention query` | sql/06_cohort_retention.sql |
| 19 | `Add new vs returning customers query` | sql/07_new_vs_returning.sql |
| 20 | `Add AOV trend query` | sql/08_aov_trend.sql |
| 21 | `Add cancelled orders query` | sql/09_cancelled_orders.sql |
| 22 | `Add CLV segments query` | sql/10_clv_segments.sql |
| 23 | `Add dashboard database utils` | dashboard/utils.py |
| 24 | `Add dashboard queries module` | dashboard/queries.py |
| 25 | `Add main Streamlit app` | dashboard/app.py |
| 26 | `Add sales overview page` | dashboard/pages/01_sales_overview.py |
| 27 | `Add customer insights page` | dashboard/pages/02_customer_insights.py |
| 28 | `Add product analytics page` | dashboard/pages/03_product_analytics.py |
| 29 | `Add RFM segmentation page` | dashboard/pages/04_rfm_segmentation.py |
| 30 | `Update README with setup instructions` | README.md |
| 31 | `Add project screenshots` | images/ folder |
| 32 | `Final code review and cleanup` | All files |

### Git Commit Rules
- One logical change = one commit
- Commit messages should be descriptive (not "update" or "fix")
- Use present tense: "Add feature" not "Added feature"
- Commit regularly, do not wait until everything is done

---

## Setup Instructions

### Prerequisites
- Python 3.9+
- PostgreSQL 13+
- Git

### Step 1: Clone the Repository
```bash
git clone https://github.com/yourusername/ecommerce-sales-analytics.git
cd ecommerce-sales-analytics
```

### Step 2: Create Virtual Environment
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

### Step 3: Install Dependencies
```bash
pip install -r requirements.txt
```

### Step 4: Setup PostgreSQL Database
```bash
# Login to PostgreSQL
psql -U postgres

# Create database
CREATE DATABASE ecommerce_analytics;

# Exit
\q

# Run schema
psql -U postgres -d ecommerce_analytics -f sql/schema.sql
```

### Step 5: Download Dataset
- Go to https://www.kaggle.com/datasets/carrie1/ecommerce-data
- Download `data.csv`
- Place in `data/raw/` folder

### Step 6: Run ETL Pipeline
```bash
python etl/main.py
```

### Step 7: Run Streamlit Dashboard
```bash
streamlit run dashboard/app.py
```

---

## Requirements.txt

```
pandas>=1.5.0
numpy>=1.21.0
matplotlib>=3.5.0
seaborn>=0.12.0
plotly>=5.0.0
streamlit>=1.20.0
SQLAlchemy>=1.4.0
psycopg2-binary>=2.9.0
python-dotenv>=0.20.0
```

---

## Key Skills Demonstrated

| Skill | Where It Shows |
|-------|----------------|
| Database Design | Normalized schema in PostgreSQL |
| SQL | Complex queries with JOINs, CTEs, Window Functions |
| Python ETL | pandas data cleaning and transformation |
| Data Analysis | RFM segmentation, cohort retention, trends |
| Visualization | Streamlit dashboard with interactive charts |
| Scalability | Modular code, indexes, production-ready structure |
| Version Control | 30+ meaningful Git commits |

---

## Future Enhancements (Optional)

- [ ] Apache Airflow for scheduled ETL runs
- [ ] Docker containerization
- [ ] Unit tests with pytest
- [ ] CI/CD with GitHub Actions
- [ ] Deploy dashboard to Streamlit Cloud
- [ ] Add predictive model (churn prediction)

---

## About This Project

This project was built as a portfolio piece to demonstrate end-to-end data analytics capabilities. It covers the full data pipeline from raw CSV ingestion to interactive dashboard visualization, using industry-standard tools and best practices.

**Author:** Your Name
**LinkedIn:** your-linkedin-url
**Email:** your-email@example.com
