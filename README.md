# 🛒 Indian E-Commerce Sales Analytics

## 📌 Project Overview

Indian E-Commerce Sales Analytics is an end-to-end Data Engineering and Analytics project built using **Databricks and PySpark**.

The project processes customer, product, and sales data through a **Medallion Architecture (Bronze → Silver → Gold)** and produces business-ready analytics for understanding sales, customers, products, categories, brands, payments, orders, ratings, and coupons.

The pipeline is orchestrated using **Databricks Workflows** with parallel Silver processing, retry configuration, email notifications, and final data quality validation.

---

## 🎯 Business Objective

The objective of this project is to transform raw e-commerce data into reliable and analytics-ready datasets that help answer important business questions such as:

- How much revenue is being generated?
- How is revenue changing month over month?
- Which states and cities generate the most sales?
- Which customers spend the most?
- Which products and categories perform best?
- Which brands generate high sales volume?
- Which payment methods are most popular?
- How do ratings relate to sales?
- How effective are coupon codes?
- Where are order cancellations and returns increasing?

---

# 🏗️ Architecture

The project follows the **Medallion Architecture**.

```text
                    RAW CSV FILES
                         │
              ┌──────────┼──────────┐
              ↓          ↓          ↓
          Customers   Products    Sales
              │          │          │
              └──────────┼──────────┘
                         ↓
                    BRONZE LAYER
                         ↓
              ┌──────────┼──────────┐
              ↓          ↓          ↓
          Silver      Silver      Silver
        Customers    Products     Sales
              └──────────┼──────────┘
                         ↓
                     GOLD LAYER
                         ↓
              Dimensions + Fact Tables
                         ↓
                  GOLD ANALYTICS
                         ↓
                   DATA QUALITY
                         ↓
                      SUCCESS



🥉 Bronze Layer

The Bronze layer ingests the raw CSV files into Delta tables with minimal transformation.

Source Files
customers.csv
products.csv
sales.csv
Source Data Volume
Dataset	Records
Customers	40,000
Products	2,000
Sales	250,000
Bronze Tables
bronze_customers
bronze_products
bronze_sales
🥈 Silver Layer

The Silver layer performs data cleaning, standardization, validation, and preparation for analytical processing.

Silver Tables
silver_customers
silver_products
silver_sales
Silver Processing

The Silver layer includes:

Data type standardization
Null validation
Duplicate validation
Date validation
Numeric validation
Foreign key validation
Business rule validation
Data cleansing
Silver Processing Strategy

The three Silver notebooks run independently after Bronze ingestion.

                 bronze_ingestion
                 /       |       \
                ↓        ↓        ↓
      silver_customers  silver_products  silver_sales

This allows the independent Silver transformations to execute in parallel.

🥇 Gold Layer

The Gold layer contains business-ready dimensional and fact tables.

Gold Dimensions
dim_customer
dim_product
dim_date
Gold Fact
fact_sales
Fact Table Grain

The fact_sales table contains one row per Order_ID.

Important business fields include:

Order_ID
Customer_ID
Product_ID
Date_Key
Quantity
Unit_Price
Order_Value
Shipping_Cost
Coupon_Discount
Total_Amount
Payment_Mode
Order_Status
Rating
Is_Delivered
Realized_Revenue

For revenue analytics, delivered orders are treated as realized sales.

📊 Gold Analytics

The project answers 13 major business questions.

1. Total Sales

Measures:

Total Gross Revenue
Total Units Sold
Total Delivered Orders
Average Order Value (AOV)
2. Monthly Revenue

Analyzes:

Monthly Gross Revenue
Monthly Units Sold
Delivered Orders
AOV
Month-over-Month Revenue Growth
3. Revenue Trend & Seasonality

Analyzes:

Monthly revenue trends
Quarterly revenue
Quarter-over-quarter growth
Seasonal patterns
Revenue anomalies

Revenue anomalies are identified using statistical analysis such as Z-score.

4. Order Status Performance

Analyzes:

Delivered orders
Cancelled orders
Returned orders
Failed orders
Monthly failure rate

Failed orders are treated as:

Cancelled + Returned
5. Payment Method Performance

Analyzes sales distribution across:

UPI
Net Banking
Credit/Debit Card
Cash on Delivery

Metrics include:

Orders
Revenue
Units Sold
AOV
Revenue Share
6. State-wise Sales

Analyzes sales performance by Indian states.

The analysis identifies:

High-revenue states
Low-revenue states
Revenue contribution
State-level order performance

Note: True market penetration requires external population/TAM data. State revenue/order share is therefore used as a proxy.

7. City-wise Sales

Analyzes:

City-level order volume
Sales
Average basket size
Tier-1 / Tier-2 / Tier-3 performance
8. Top Customers

Identifies the top 50 customers based on spending.

Metrics include:

Total spending
Number of orders
Average order value
9. Best-Selling Products

Identifies top-performing products based on:

Units sold
Revenue

The analysis identifies the top 10 products.

10. Category Performance

Analyzes product categories based on:

Sales volume
Revenue
Units sold
Category performance
11. Brand Performance

Analyzes:

Brand revenue
Units sold
Revenue per unit
High-volume / low-value brands

Note: True profit margin cannot be calculated because the dataset does not contain COGS. Revenue per unit and discount metrics are used as proxies.

12. Product Ratings

Analyzes:

Average rating by category
Product ratings
Rating distribution
Relationship between ratings and sales

Correlation analysis is used to examine the relationship between ratings and sales.

13. Coupon Performance

Analyzes:

Coupon usage
Revenue
Orders
Units sold
Average discount
AOV
Revenue share

Coupon codes are standardized so missing/blank coupon values are treated as:

NO_COUPON

Coupon analysis is observational. A lower AOV for discounted orders should not automatically be interpreted as a causal effect of coupons.

🧪 Data Quality

A dedicated Data Quality notebook validates the pipeline output.

Data Quality Checks
Row count validation
Primary key validation
Foreign key validation
Date foreign key validation
Critical null checks
Delivery date validation
Quantity validation
Order status validation
Negative transaction amount detection
Revenue reconciliation
Delivered units reconciliation
Final Data Quality Result
=======================================================
Failed Checks   : 0
Warning Checks  : 1
Overall Status  : PASS
=======================================================

The warning is related to 5 negative Total_Amount records caused by coupon discounts exceeding the small transaction amount.

These records were retained and flagged instead of silently removing source data.

⚙️ Databricks Workflow

The entire pipeline is automated using Databricks Workflows.

Workflow Name
ecommerce_sales_analytics_pipeline
Tasks
01_bronze_ingestion
02_silver_customers
03_silver_products
04_silver_sales
05_gold_dimensions
06_gold_fact_sales
07_gold_analytics
08_data_quality
Workflow Dependency
                 ┌── silver_customers ──┐
                 │                      │
bronze_ingestion ├── silver_products ───┼──→ gold_dimensions
                 │                      │
                 └── silver_sales ──────┘
                                             ↓
                                      gold_fact_sales
                                             ↓
                                       gold_analytics
                                             ↓
                                        data_quality
🚀 Pipeline Optimization

The workflow uses parallel execution where possible.

The following tasks run independently after Bronze:

silver_customers
silver_products
silver_sales

This reduces unnecessary sequential execution and improves pipeline efficiency.

🔁 Retry Configuration

Each workflow task is configured with:

Maximum retries: 2
Retry interval: 5 minutes

This provides resilience against temporary infrastructure or execution failures.

📧 Email Notifications

The Databricks Job is configured to send email notifications for:

Pipeline Success
Pipeline Failure

This allows pipeline failures or successful executions to be monitored without manually checking the Job UI.

🛠️ Technology Stack
Technology	Purpose
Python	Programming and transformation logic
PySpark	Distributed data processing
Databricks	Data engineering platform
Delta Lake	Data storage
SQL	Data analysis and validation
Databricks Workflows	Pipeline orchestration
GitHub	Version control and project documentation
📁 Project Structure
Indian-E-Commerce-Sales-Analytics/
│
├── 01_bronze_ingestion
├── 02_silver_customers
├── 03_silver_products
├── 04_silver_sales
├── 05_gold_dimensions
├── 06_gold_fact_sales
├── 07_gold_analytics
├── 08_data_quality
│
├── workflows/
│   └── ecommerce_sales_analytics_pipeline
│
├── documentation/
│   ├── data_dictionary
│   └── architecture
│
└── README.md
📈 Key Project Outcomes

The project successfully demonstrates:

End-to-end data engineering
Medallion Architecture
PySpark transformations
Delta Lake tables
Dimensional modeling
Fact table design
Business analytics
Data quality validation
Parallel processing
Workflow orchestration
Retry handling
Email monitoring

The final pipeline successfully processes:

40,000 Customers
2,000 Products
250,000 Sales Orders

with:

0 Failed Data Quality Checks
1 Warning
Overall Data Quality: PASS
🔮 Future Enhancements

Possible future improvements include:

Incremental data ingestion
Change Data Capture (CDC)
Apache Airflow integration
Real-time Kafka streaming
Power BI / Tableau dashboard
Azure Data Lake Storage integration
Automated data quality alerts
External market penetration datasets
Inventory history for true inventory turnover
COGS data for true profit-margin analysis
👨‍💻 Author
Pradeep Kumar Behara

Aspiring Data Engineer & AI Enthusiast with experience in Python, SQL, PySpark, Databricks, ETL pipelines, and data analytics.

Technical Interests
Data Engineering
PySpark
Databricks
ETL / ELT
SQL
Cloud Data Platforms
Artificial Intelligence
Generative AI
⭐ Project Highlights
✔ 250K+ Sales Records
✔ 40K Customers
✔ 2K Products
✔ Medallion Architecture
✔ PySpark
✔ Databricks
✔ Delta Lake
✔ 13 Business Analytics
✔ Data Quality Framework
✔ Parallel Silver Processing
✔ Automated Databricks Workflow
✔ Retry Configuration
✔ Email Notifications


                   DATA QUALITY
                         ↓
                      SUCC
