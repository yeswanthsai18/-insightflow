# InsightFlow: Supply Chain Analytics & Demand Forecasting

> Data-driven analytics solution to optimize inventory levels, supplier performance, and demand forecasting across multiple distribution centers.

![Python](https://img.shields.io/badge/Python-3.9-blue.svg)
![Snowflake](https://img.shields.io/badge/Snowflake-Data_Warehouse-29B5E8.svg)
![Airflow](https://img.shields.io/badge/Apache_Airflow-Orchestration-017CEE.svg)
![Tableau](https://img.shields.io/badge/Tableau-Visualization-E97627.svg)
![SQL](https://img.shields.io/badge/SQL-Advanced-grey.svg)
![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)

---

## Project Overview

InsightFlow is an end-to-end data analytics solution designed to solve critical challenges in supply chain management. By integrating disparate data from ERPs, logistics systems, and warehouse inventories, this project provides a unified source of truth for stakeholder analysis.

The core of the project is a robust ETL pipeline orchestrated by **Apache Airflow**, a modern data warehouse built in **Snowflake**, and a predictive forecasting engine using **Python** (ARIMA, Prophet, XGBoost). The final insights are delivered through an interactive **Tableau** dashboard, enabling data-driven decisions that directly impact the bottom line.

## Business Problem & Objectives

In a modern supply chain, data is often siloed, leading to costly inefficiencies. The primary business problems this project solves are:
* **Poor Demand Forecasting:** Leading to expensive overstocking of slow-moving items and stock-outs (backorders) of high-demand items.
* **Reactive Inventory Management:** Inability to proactively manage inventory levels across different distribution centers.
* **Lack of Supplier Visibility:** Difficulty in tracking supplier performance, leading to unreliable delivery schedules and high costs.

**Project Objectives:**
1.  **Forecast Demand:** Implement machine learning models to predict product demand with an accuracy of at least 85%.
2.  **Optimize Inventory:** Provide analytics to reduce overstocking by 15% and backorders by 10%.
3.  **Monitor KPIs:** Deliver a real-time Tableau dashboard to visualize key metrics like inventory turnover, order fulfillment rate, and supplier delivery delays.
4.  **Automate Data Flow:** Build a scalable and reliable automated ETL pipeline to ensure data is fresh and trustworthy.

## Data & Workflow Architecture

The project follows a modern data stack (ELT-first) approach, divided into four key stages:

### 1. Data Ingestion & Orchestration (Apache Airflow)
* **Sources:** Data is extracted from multiple sources:
    * **ERP Database:** (PostgreSQL) - `orders`, `products`, `customer` tables.
    * **Logistics API:** (JSON) - `shipment_status`, `delivery_times`.
    * **Warehouse CSVs:** (S3) - `daily_inventory_levels`, `stock_counts`.
* **Orchestration:** An **Apache Airflow** DAG (Directed Acyclic Graph) runs daily.
    * `extract_erp_data`: Python script queries the ERP database and saves data as Parquet.
    * `call_logistics_api`: Python script hits the logistics API and saves the JSON response.
    * `load_to_s3`: All extracted files (Parquet, JSON, CSV) are uploaded to an S3 "landing zone" bucket.
    * `s3_to_snowflake`: Airflow triggers a Snowflake `COPY INTO` command to load all raw data from S3 into the **BRONZE** layer tables.

### 2. Warehousing & Transformation (Snowflake)
Data is modeled in Snowflake using a **Medallion Architecture**:
* **BRONZE Layer (Raw):** Raw data is loaded directly from S3. Tables mirror the source (`raw_orders`, `raw_shipments`).
* **SILVER Layer (Cleansed):** SQL transformations are applied to the raw data to create cleaned, standardized, and integrated tables.
    * Type casting, deduplication, and handling NULL values.
    * Joining `orders` and `shipments` data.
    * Creating standardized dimension tables like `dim_product`, `dim_customer`, `dim_distribution_center`.
* **GOLD Layer (Aggregated):** Business-ready, aggregated tables used by BI tools and ML models.
    * `fct_daily_inventory`: A daily snapshot of inventory levels, turnover, and stock value.
    * `fct_supplier_performance`: Aggregated metrics for each supplier (e.g., `avg_delivery_delay_days`, `order_fulfillment_rate`).
    * `fct_demand_history`: A clean, aggregated time series of product sales, used for forecasting.

### 3. Predictive Forecasting (Python)
* A separate Python script (which could also be an Airflow task) runs weekly.
* It reads data from the `GOLD.FCT_DEMAND_HISTORY` table in Snowflake.
* It preprocesses the data for time series analysis (e.g., setting date index, handling seasonality).
* It trains, evaluates, and selects the best forecasting model (**ARIMA**, **Prophet**, and **XGBoost** are tested).
* The final forecast (next 90 days) is written back into a Snowflake table: `GOLD.FCT_DEMAND_FORECAST`.

### 4. Visualization (Tableau)
* **Tableau Desktop/Cloud** connects *live* to the **GOLD** layer tables in Snowflake.
* This ensures dashboards are always up-to-date with the latest processed data.
* Row-level security can be applied in Tableau (or via Snowflake) to restrict data access based on user roles (e.g., a distribution center manager only sees their own center's data).

## Technology Stack

* **Orchestration:** <img>: `Apache Airflow`
* **Data Lake / Landing:** <img>: `AWS S3`
* **Data Warehouse:** <img>: `Snowflake`
* **Data Transformation:** <img>: `Python (Pandas)`, `SQL (SnowSQL)`
* **Forecasting & ML:** <img>: `Python (Prophet, Statsmodels, XGBoost, Scikit-learn)`
* **Visualization:** <img>: `Tableau`

## Dashboard & Key Insights

The final Tableau dashboard consists of three main pages:

1.  **Demand Forecast Dashboard:**
    * Visualizes historical demand vs. forecasted demand (with 87% accuracy).
    * Allows users to filter by product category and distribution center.
    * Highlights products at high risk of stock-out or overstocking.

2.  **Inventory Optimization View:**
    * Tracks real-time KPIs: Inventory Turnover, Stock-to-Sales Ratio, and Days of Supply.
    * Provides actionable recommendations for re-stocking, helping to achieve a **15% reduction in overstocking**.
    * Identifies slow-moving vs. fast-moving products.

3.  **Supplier Performance Tracker:**
    * Monitors Order Fulfillment Rate (OFR) and On-Time In-Full (OTIF) delivery for all suppliers.
    * Tracks delivery delays and links them to potential backorder events, helping to reduce backorders by **10%**.

## How to Run

1.  **Clone the repository:**
    ```bash
    git clone [https://github.com/yeswanthsai18/insightflow.git](https://github.com/yeswanthsai18/insightflow.git)
    cd insightflow
    ```

2.  **Install dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

3.  **Configure Environment:**
    * Set up your Snowflake, AWS S3, and Airflow connections in a `.env` file (see `.env.example`).

4.  **Run ETL:**
    * Deploy the DAG file from the `/airflow` directory to your Airflow instance.
    * Trigger the `supply_chain_etl_dag` manually or wait for the daily schedule.

5.  **Run Forecasting:**
    ```bash
    python models/run_forecasting.py
    ```

6.  **View Dashboard:**
    * Open the `InsightFlow_Dashboard.twbx` file in Tableau Desktop or publish it to Tableau Cloud.
    * Connect the dashboard to your Snowflake `GOLD` schema.

## License
This project is licensed under the MIT License.
