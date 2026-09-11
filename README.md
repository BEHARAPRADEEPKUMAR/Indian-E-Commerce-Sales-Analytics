# 📖 Data Dictionary — Indian E-Commerce Sales Analytics

## 1. Overview

This data dictionary describes the datasets and tables used in the Indian E-Commerce Sales Analytics project.

The project contains three source datasets:

- Customers
- Products
- Sales

The data is processed through Bronze, Silver, and Gold layers using Databricks and PySpark.

---

# 2. Source Datasets

## 2.1 Customers Dataset

**File:** `customers.csv`

**Records:** 40,000

| Column | Description |
|---|---|
| Customer_ID | Unique identifier for each customer |
| Customer_Name | Customer name |
| Gender | Customer gender |
| Age | Customer age |
| Age_Group | Customer age category |
| Date_of_Birth | Customer date of birth |
| Email | Customer email address |
| Phone | Customer phone number |
| City | Customer city |
| State | Customer state |
| Pincode | Customer postal code |
| Registration_Date | Date when customer registered |
| Customer_Tier | Customer classification/tier |
| Total_Orders | Total delivered orders associated with the customer |
| Total_Spent | Total spending from delivered orders |

---

## 2.2 Products Dataset

**File:** `products.csv`

**Records:** 2,000

| Column | Description |
|---|---|
| Product_ID | Unique identifier for each product |
| Product_Name | Name of the product |
| Category | Product category |
| Brand | Product brand |
| Original_Price | Original product price |
| Discount_Percent | Discount percentage |
| Discount_Amount | Discount amount |
| Selling_Price | Final selling price after discount |
| Stock_Quantity | Available stock quantity |
| Weight_kg | Product weight in kilograms |
| Avg_Rating | Average product rating |
| Total_Reviews | Total number of product reviews |

---

## 2.3 Sales Dataset

**File:** `sales.csv`

**Records:** 250,000

| Column | Description |
|---|---|
| Order_ID | Unique identifier for each order |
| Customer_ID | Customer associated with the order |
| Product_ID | Product associated with the order |
| Order_Date | Date when the order was placed |
| Order_Time | Time when the order was placed |
| Delivery_Date | Date when the order was delivered |
| Quantity | Number of units ordered |
| Unit_Price | Price per unit |
| Order_Value | Value of the ordered items |
| Shipping_Cost | Shipping cost for the order |
| Coupon_Code | Coupon applied to the order |
| Coupon_Discount | Discount provided through the coupon |
| Total_Amount | Final transaction amount |
| Payment_Mode | Payment method used |
| Order_Status | Status of the order |
| Rating | Customer rating for the order |
| Review_Text | Customer review text |
| City | Customer/order city |
| State | Customer/order state |
| Customer_Age | Customer age at the time of the order |
| Customer_Age_Group | Customer age category |

---

# 3. Bronze Layer

The Bronze layer stores the ingested source data in Delta tables.

| Table | Source |
|---|---|
| bronze_customers | customers.csv |
| bronze_products | products.csv |
| bronze_sales | sales.csv |

Purpose:

- Raw data ingestion
- Preserve source information
- Store data in Delta format

---

# 4. Silver Layer

The Silver layer contains cleaned and validated datasets.

| Table | Description |
|---|---|
| silver_customers | Cleaned customer data |
| silver_products | Cleaned product data |
| silver_sales | Cleaned and validated sales data |

Main processing includes:

- Data type standardization
- Null validation
- Duplicate validation
- Date validation
- Numeric validation
- Data quality checks

---

# 5. Gold Layer

The Gold layer contains analytics-ready dimensional and fact tables.

## 5.1 dim_customer

Customer dimension containing customer-level information used for analytics.

## 5.2 dim_product

Product dimension containing product, category, brand, pricing, and rating information.

## 5.3 dim_date

Date dimension used for time-based analysis such as:

- Monthly revenue
- Quarterly revenue
- Revenue trends
- Seasonality

## 5.4 fact_sales

The main sales fact table.

### Grain

**One row per Order_ID**

Important measures and attributes include:

| Column | Description |
|---|---|
| Order_ID | Unique order identifier |
| Customer_ID | Customer identifier |
| Product_ID | Product identifier |
| Date_Key | Date dimension key |
| Quantity | Units ordered |
| Unit_Price | Price per unit |
| Order_Value | Order item value |
| Shipping_Cost | Shipping cost |
| Coupon_Discount | Coupon discount |
| Total_Amount | Final transaction amount |
| Payment_Mode | Payment method |
| Order_Status | Order status |
| Rating | Customer rating |
| Is_Delivered | Indicates whether the order was delivered |
| Realized_Revenue | Revenue recognized for delivered orders |

---

# 6. Gold Analytics Tables

The project creates business-ready analytics tables for 13 business questions.

| Analytics Table | Business Purpose |
|---|---|
| gold_total_sales | Overall sales KPIs |
| gold_monthly_revenue | Monthly revenue and MoM growth |
| gold_revenue_trend | Revenue trends, quarterly performance and anomalies |
| gold_order_status | Order delivery, cancellation and return analysis |
| gold_payment_method | Payment method sales performance |
| State-wise analytics | State-level sales performance |
| City-wise analytics | City and tier-level sales performance |
| Top customer analytics | Top spending customers |
| Best-selling product analytics | Top products by units and revenue |
| Category analytics | Category performance |
| Brand analytics | Brand performance |
| Rating analytics | Product ratings and rating-sales relationship |
| gold_coupon_performance | Coupon performance |

---

# 7. Important Business Rules

## Revenue Rule

For the main revenue KPIs, only **Delivered orders** are treated as realized revenue.

```text
Realized_Revenue =
    Total_Amount when Order_Status = Delivered
    0 otherwise
Failed Orders

For order-status analysis:

Failed Orders =
Cancelled + Returned
Coupon Standardization

Null or blank coupon codes are standardized to:

NO_COUPON
Negative Transaction Amounts

Five records contain negative Total_Amount values because the coupon discount exceeds the small transaction amount.

These records are retained and flagged for data-quality review.

8. Data Quality Summary

The final data quality validation produced:

Failed Checks   : 0
Warning Checks  : 1
Overall Status  : PASS

The warning is related to the five negative transaction amounts described above.

9. Data Relationships
dim_customer
      │
      │ Customer_ID
      ↓
fact_sales
      │
      │ Product_ID
      ↓
dim_product

fact_sales
      │
      │ Date_Key
      ↓
dim_date

The main relationships are:

Customer_ID → dim_customer.Customer_ID
Product_ID  → dim_product.Product_ID
Date_Key    → dim_date.Date_Key
10. Data Volumes
Layer	Table	Records
Bronze	bronze_customers	40,000
Bronze	bronze_products	2,000
Bronze	bronze_sales	250,000
Silver	silver_customers	40,000
Silver	silver_products	2,000
Silver	silver_sales	250,000
Gold	dim_customer	40,000
Gold	dim_product	2,000
Gold	dim_date	760
Gold	fact_sales	250,000
11. Data Limitations

Some business metrics require additional external data.

Market Penetration

True market penetration requires population or total addressable market data.

The project therefore uses state revenue/order contribution as a proxy.

Profit Margin

True profit margin cannot be calculated because COGS is not available.

Revenue per unit and discount metrics are used as proxies.

Inventory Turnover

True inventory turnover requires inventory history or average inventory data.

12. Technology
Python
PySpark
Databricks
Delta Lake
SQL
Databricks Workflows
GitHub