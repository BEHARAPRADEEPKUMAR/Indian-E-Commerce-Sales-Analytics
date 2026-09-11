# Databricks Workflow — E-Commerce Sales Analytics Pipeline

## Workflow Name

`ecommerce_sales_analytics_pipeline`

## Purpose

This Databricks Workflow orchestrates the complete Indian E-Commerce Sales Analytics pipeline from Bronze ingestion through Silver transformation, Gold analytics, and final data quality validation.

---

## Workflow Architecture

```text
                    ┌── silver_customers ──┐
                    │                      │
bronze_ingestion ───┼── silver_products ───┼──→ gold_dimensions
                    │                      │
                    └── silver_sales ──────┘
                                             ↓
                                      gold_fact_sales
                                             ↓
                                       gold_analytics
                                             ↓
                                        data_quality
Tasks
Task	Notebook	Depends On
bronze_ingestion	01_bronze_ingestion	None
silver_customers	02_silver_customers	bronze_ingestion
silver_products	03_silver_products	bronze_ingestion
silver_sales	04_silver_sales	bronze_ingestion
gold_dimensions	05_gold_dimensions	silver_customers, silver_products, silver_sales
gold_fact_sales	06_gold_fact_sales	gold_dimensions
gold_analytics	07_gold_analytics	gold_fact_sales
data_quality	08_data_quality	gold_analytics
Parallel Processing

After the Bronze ingestion task completes, the following Silver tasks can execute independently:

silver_customers
silver_products
silver_sales

This allows parallel processing of independent datasets.

Retry Configuration

Each task is configured with:

Maximum retries: 2
Retry interval: 5 minutes
Retry on timeout: Disabled
Serverless auto-optimization: Enabled
Notifications

Email notifications are configured at the Job level for:

Job success
Job failure
Compute

The workflow uses Databricks Serverless compute.

Data Quality

The final task is data_quality.

The successful validation result was:

Failed Checks   : 0
Warning Checks  : 1
Overall Status  : PASS

The warning was related to five negative Total_Amount records caused by coupon discounts exceeding the transaction amount. These records were retained and flagged for review.

Execution Flow
1. Bronze ingestion
        ↓
2. Silver transformations
        ↓
3. Gold dimensions
        ↓
4. Gold fact
        ↓
5. Gold analytics
        ↓
6. Data quality
        ↓
7. Pipeline success