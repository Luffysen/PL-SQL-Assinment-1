# PL-SQL-Assinment-1

# SQL JOINs & Window Functions Project

**Course:** INSY 8311 - Database Development with PL/SQL  
**Student:** Djamaladine Salim 
**Student ID:** 25962
**Group:** A
**Date:** 08/02/2026

---

## Table of Contents
1. [Business Problem](#business-problem)
2. [Success Criteria](#success-criteria)
3. [Database Schema](#database-schema)
4. [SQL JOINs Implementation](#sql-joins-implementation)
5. [Window Functions Implementation](#window-functions-implementation)
6. [Results Analysis](#results-analysis)
7. [Screenshots](#screenshots)
8. [References](#references)
9. [Academic Integrity Statement](#academic-integrity-statement)

---

## Business Problem

### Business Context
**Company:** TechMart Electronics Retail  
**Industry:** Consumer Electronics Retail  
**Department:** Sales Analytics & Business Intelligence

TechMart is a mid-sized electronics retailer operating across 4 major regions (North, South, East, West) with 15 physical stores and an e-commerce platform. The company sells smartphones, laptops, tablets, and accessories.

### Data Challenge
TechMart's management needs to analyze sales performance across regions and time periods to identify top-performing products, understand customer purchasing patterns, and segment customers for targeted marketing campaigns. Currently, sales data is scattered across multiple tables without proper analytical insights, making it difficult to make data-driven decisions for inventory management and marketing strategies.

### Expected Outcome
The analysis will provide actionable insights including: top 5 products per region ranked by revenue, running monthly sales totals for trend analysis, month-over-month growth rates to identify seasonal patterns, customer segmentation into quartiles based on purchase value, and 3-month moving averages to smooth out sales volatility and forecast demand.

---

## Success Criteria

| # | Goal | Window Function | Business Purpose |
|---|------|-----------------|------------------|
| 1 | Identify Top 5 products per region by revenue | `RANK()` | Prioritize inventory for high-performing products |
| 2 | Calculate running monthly sales totals | `SUM() OVER()` | Track cumulative performance against targets |
| 3 | Compute month-over-month sales growth | `LAG()` / `LEAD()` | Identify growth trends and seasonal patterns |
| 4 | Segment customers into 4 quartiles | `NTILE(4)` | Create targeted marketing campaigns |
| 5 | Calculate 3-month moving average sales | `AVG() OVER()` | Smooth volatility and forecast demand |

---

## Database Schema

### Entity Relationship Diagram

```
+----------------+       +----------------------+       +----------------+
|   customers    |       |  sales_transactions  |       |    products    |
+----------------+       +----------------------+       +----------------+
| PK customer_id |<-----| FK customer_id       |       | PK product_id  |
| first_name     |       | FK product_id       |------>| product_name   |
| last_name      |       | quantity            |       | category       |
| email          |       | unit_price          |       | unit_price     |
| region         |       | total_amount        |       | stock_quantity |
| reg_date       |       | transaction_date    |       +----------------+
| segment        |       | sales_channel       |
+----------------+       +----------------------+
```

### Table Descriptions

| Table | Description | Records |
|-------|-------------|---------|
| customers | Stores customer information with regional data | 20 |
| products | Product catalog with pricing and inventory | 15 |
| sales_transactions | Sales records linking customers and products | 60+ |

### Relationships
- **customers** (1) --- (*) **sales_transactions**
- **products** (1) --- (*) **sales_transactions**

---

## SQL JOINs Implementation

### 1. INNER JOIN - Transactions with Valid Customers and Products

```sql
SELECT 
    st.transaction_id,
    c.first_name || ' ' || c.last_name AS customer_name,
    c.region,
    p.product_name,
    p.category,
    st.quantity,
    st.total_amount,
    st.transaction_date
FROM sales_transactions st
INNER JOIN customers c ON st.customer_id = c.customer_id
INNER JOIN products p ON st.product_id = p.product_id
ORDER BY st.transaction_date DESC
LIMIT 15;
```

**Business Interpretation:** This query retrieves complete transaction records where both the customer and product information exists in the database. It helps ensure data integrity by showing only valid sales transactions with complete information.

**Screenshot:** See `screenshots/01_inner_join.png`

---

### 2. LEFT JOIN - Customers Who Never Made a Transaction

```sql
SELECT 
    c.customer_id,
    c.first_name || ' ' || c.last_name AS customer_name,
    c.email,
    c.region,
    c.registration_date,
    st.transaction_id,
    st.total_amount
FROM customers c
LEFT JOIN sales_transactions st ON c.customer_id = st.customer_id
WHERE st.transaction_id IS NULL
ORDER BY c.registration_date;
```

**Business Interpretation:** This query identifies inactive customers who registered but never made a purchase. These customers represent potential leads for targeted marketing campaigns to convert them into active buyers.

**Screenshot:** See `screenshots/02_left_join.png`

---

### 3. RIGHT JOIN - Products with No Sales Activity

```sql
SELECT 
    p.product_id,
    p.product_name,
    p.category,
    p.unit_price,
    p.stock_quantity,
    st.transaction_id,
    st.quantity AS sold_quantity
FROM sales_transactions st
RIGHT JOIN products p ON st.product_id = p.product_id
WHERE st.transaction_id IS NULL
ORDER BY p.category, p.product_name;
```

**Business Interpretation:** This query identifies products in inventory that have never been sold. These products may need promotional campaigns, price adjustments, or could be candidates for discontinuation to free up warehouse space and capital.

**Screenshot:** See `screenshots/03_right_join.png`

---

### 4. FULL OUTER JOIN - Complete Comparison

```sql
SELECT 
    c.customer_id,
    c.first_name || ' ' || c.last_name AS customer_name,
    c.region AS customer_region,
    p.product_id,
    p.product_name,
    p.category,
    st.transaction_id,
    st.total_amount,
    CASE 
        WHEN c.customer_id IS NULL THEN 'Product Not Purchased'
        WHEN p.product_id IS NULL THEN 'Customer No Purchase'
        WHEN st.transaction_id IS NULL THEN 'No Transaction Match'
        ELSE 'Valid Transaction'
    END AS match_status
FROM customers c
FULL OUTER JOIN sales_transactions st ON c.customer_id = st.customer_id
FULL OUTER JOIN products p ON st.product_id = p.product_id
ORDER BY match_status, c.customer_id, p.product_id
LIMIT 20;
```

**Business Interpretation:** This comprehensive query provides a complete view of the customer-product-transaction relationship, highlighting gaps in the data such as customers who haven't purchased and products that haven't been sold.

**Screenshot:** See `screenshots/04_full_outer_join.png`

---

### 5. SELF JOIN - Compare Customers in Same Region

```sql
SELECT 
    c1.customer_id AS customer1_id,
    c1.first_name || ' ' || c1.last_name AS customer1_name,
    c2.customer_id AS customer2_id,
    c2.first_name || ' ' || c2.last_name AS customer2_name,
    c1.region,
    c1.registration_date AS customer1_reg_date,
    c2.registration_date AS customer2_reg_date
FROM customers c1
INNER JOIN customers c2 ON c1.region = c2.region AND c1.customer_id < c2.customer_id
ORDER BY c1.region, c1.customer_id
LIMIT 15;
```

**Business Interpretation:** This query creates pairs of customers from the same region, useful for regional marketing analysis and identifying potential referral opportunities.

**Screenshot:** See `screenshots/05_self_join.png`

---

## Window Functions Implementation

### Category 1: Ranking Functions

#### ROW_NUMBER()
```sql
SELECT 
    customer_id,
    customer_name,
    region,
    total_spent,
    ROW_NUMBER() OVER (ORDER BY total_spent DESC) AS row_num
FROM (
    SELECT 
        c.customer_id,
        c.first_name || ' ' || c.last_name AS customer_name,
        c.region,
        SUM(st.total_amount) AS total_spent
    FROM customers c
    JOIN sales_transactions st ON c.customer_id = st.customer_id
    GROUP BY c.customer_id, c.first_name, c.last_name, c.region
) customer_spending;
```
**Screenshot:** See `screenshots/06_row_number.png`

#### RANK() - Top Products Per Region
```sql
WITH regional_product_sales AS (
    SELECT 
        c.region,
        p.product_name,
        p.category,
        SUM(st.total_amount) AS total_revenue,
        SUM(st.quantity) AS units_sold
    FROM sales_transactions st
    JOIN customers c ON st.customer_id = c.customer_id
    JOIN products p ON st.product_id = p.product_id
    GROUP BY c.region, p.product_name, p.category
)
SELECT 
    region,
    product_name,
    category,
    total_revenue,
    units_sold,
    RANK() OVER (PARTITION BY region ORDER BY total_revenue DESC) AS product_rank
FROM regional_product_sales
ORDER BY region, product_rank;
```
**Screenshot:** See `screenshots/07_rank.png`

#### DENSE_RANK()
```sql
SELECT 
    customer_id,
    customer_name,
    region,
    total_spent,
    DENSE_RANK() OVER (ORDER BY total_spent DESC) AS dense_rank_num
FROM (
    SELECT 
        c.customer_id,
        c.first_name || ' ' || c.last_name AS customer_name,
        c.region,
        SUM(st.total_amount) AS total_spent
    FROM customers c
    JOIN sales_transactions st ON c.customer_id = st.customer_id
    GROUP BY c.customer_id, c.first_name, c.last_name, c.region
) customer_spending;
```
**Screenshot:** See `screenshots/08_dense_rank.png`

#### PERCENT_RANK()
```sql
SELECT 
    customer_id,
    customer_name,
    region,
    total_spent,
    ROUND(PERCENT_RANK() OVER (ORDER BY total_spent DESC)::numeric, 4) AS percentile_rank
FROM (
    SELECT 
        c.customer_id,
        c.first_name || ' ' || c.last_name AS customer_name,
        c.region,
        SUM(st.total_amount) AS total_spent
    FROM customers c
    JOIN sales_transactions st ON c.customer_id = st.customer_id
    GROUP BY c.customer_id, c.first_name, c.last_name, c.region
) customer_spending;
```
**Screenshot:** See `screenshots/09_percent_rank.png`

---

### Category 2: Aggregate Window Functions

#### SUM() OVER() - Running Totals
```sql
WITH monthly_sales AS (
    SELECT 
        DATE_TRUNC('month', transaction_date) AS sales_month,
        SUM(total_amount) AS monthly_total
    FROM sales_transactions
    GROUP BY DATE_TRUNC('month', transaction_date)
)
SELECT 
    sales_month,
    monthly_total,
    SUM(monthly_total) OVER (ORDER BY sales_month ROWS UNBOUNDED PRECEDING) AS running_total
FROM monthly_sales
ORDER BY sales_month;
```
**Screenshot:** See `screenshots/10_sum_over.png`

#### AVG() OVER() - 3-Month Moving Average
```sql
WITH monthly_sales AS (
    SELECT 
        DATE_TRUNC('month', transaction_date) AS sales_month,
        SUM(total_amount) AS monthly_total
    FROM sales_transactions
    GROUP BY DATE_TRUNC('month', transaction_date)
)
SELECT 
    sales_month,
    monthly_total,
    ROUND(AVG(monthly_total) OVER (
        ORDER BY sales_month 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ), 2) AS three_month_moving_avg
FROM monthly_sales
ORDER BY sales_month;
```
**Screenshot:** See `screenshots/11_avg_over.png`

#### MIN() and MAX() OVER()
```sql
WITH daily_sales AS (
    SELECT 
        transaction_date,
        SUM(total_amount) AS daily_total
    FROM sales_transactions
    GROUP BY transaction_date
)
SELECT 
    transaction_date,
    daily_total,
    MIN(daily_total) OVER (ORDER BY transaction_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS week_low,
    MAX(daily_total) OVER (ORDER BY transaction_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS week_high
FROM daily_sales
ORDER BY transaction_date;
```
**Screenshot:** See `screenshots/12_min_max_over.png`

---

### Category 3: Navigation Functions

#### LAG() - Month-over-Month Growth
```sql
WITH monthly_sales AS (
    SELECT 
        DATE_TRUNC('month', transaction_date) AS sales_month,
        SUM(total_amount) AS monthly_total
    FROM sales_transactions
    GROUP BY DATE_TRUNC('month', transaction_date)
)
SELECT 
    sales_month,
    monthly_total,
    LAG(monthly_total) OVER (ORDER BY sales_month) AS previous_month,
    monthly_total - LAG(monthly_total) OVER (ORDER BY sales_month) AS month_difference,
    ROUND(
        ((monthly_total - LAG(monthly_total) OVER (ORDER BY sales_month)) / 
        LAG(monthly_total) OVER (ORDER BY sales_month)) * 100, 
        2
    ) AS growth_rate_percent
FROM monthly_sales
ORDER BY sales_month;
```
**Screenshot:** See `screenshots/13_lag.png`

#### LEAD() - Forward-Looking Analysis
```sql
WITH monthly_sales AS (
    SELECT 
        DATE_TRUNC('month', transaction_date) AS sales_month,
        SUM(total_amount) AS monthly_total
    FROM sales_transactions
    GROUP BY DATE_TRUNC('month', transaction_date)
)
SELECT 
    sales_month,
    monthly_total,
    LEAD(monthly_total) OVER (ORDER BY sales_month) AS next_month,
    CASE 
        WHEN LEAD(monthly_total) OVER (ORDER BY sales_month) > monthly_total THEN 'Increasing'
        WHEN LEAD(monthly_total) OVER (ORDER BY sales_month) < monthly_total THEN 'Decreasing'
        ELSE 'Stable'
    END AS trend_direction
FROM monthly_sales
ORDER BY sales_month;
```
**Screenshot:** See `screenshots/14_lead.png`

---

### Category 4: Distribution Functions

#### NTILE(4) - Customer Quartile Segmentation
```sql
WITH customer_totals AS (
    SELECT 
        c.customer_id,
        c.first_name || ' ' || c.last_name AS customer_name,
        c.region,
        c.email,
        SUM(st.total_amount) AS total_spent,
        COUNT(st.transaction_id) AS transaction_count
    FROM customers c
    JOIN sales_transactions st ON c.customer_id = st.customer_id
    GROUP BY c.customer_id, c.first_name, c.last_name, c.region, c.email
)
SELECT 
    customer_id,
    customer_name,
    region,
    total_spent,
    transaction_count,
    NTILE(4) OVER (ORDER BY total_spent DESC) AS spending_quartile,
    CASE 
        WHEN NTILE(4) OVER (ORDER BY total_spent DESC) = 1 THEN 'Premium'
        WHEN NTILE(4) OVER (ORDER BY total_spent DESC) = 2 THEN 'High Value'
        WHEN NTILE(4) OVER (ORDER BY total_spent DESC) = 3 THEN 'Medium Value'
        ELSE 'Low Value'
    END AS customer_segment
FROM customer_totals
ORDER BY spending_quartile, total_spent DESC;
```
**Screenshot:** See `screenshots/15_ntile.png`

#### CUME_DIST() - Cumulative Distribution
```sql
WITH customer_totals AS (
    SELECT 
        c.customer_id,
        c.first_name || ' ' || c.last_name AS customer_name,
        c.region,
        SUM(st.total_amount) AS total_spent
    FROM customers c
    JOIN sales_transactions st ON c.customer_id = st.customer_id
    GROUP BY c.customer_id, c.first_name, c.last_name, c.region
)
SELECT 
    customer_id,
    customer_name,
    region,
    total_spent,
    ROUND(CUME_DIST() OVER (ORDER BY total_spent DESC)::numeric, 4) AS cumulative_distribution,
    ROUND((1 - CUME_DIST() OVER (ORDER BY total_spent DESC))::numeric, 4) AS top_percentile
FROM customer_totals
ORDER BY total_spent DESC;
```
**Screenshot:** See `screenshots/16_cume_dist.png`

---

## Results Analysis

### Descriptive Analysis (What happened?)

1. **Sales Performance:** The analysis of 60+ transactions reveals that the North region generated the highest revenue, with iPhone 15 Pro and MacBook Pro being the top-selling products.

2. **Customer Distribution:** All 20 registered customers made at least one purchase, indicating effective customer acquisition and retention strategies.

3. **Product Performance:** Out of 15 products in the catalog, most have recorded sales, with smartphones and laptops generating the highest revenue per transaction.

### Diagnostic Analysis (Why did it happen?)

1. **Regional Variations:** The North region's higher performance can be attributed to higher average transaction values, particularly for premium products like MacBook Pro ($2,499.99 average price).

2. **Product Preferences:** Smartphones (iPhone 15 Pro, Samsung Galaxy S24) show consistent sales across all regions, indicating strong brand recognition and market demand.

3. **Seasonal Patterns:** The month-over-month analysis shows steady growth from January to July 2024, with no significant declines, suggesting effective marketing and stable market conditions.

### Prescriptive Analysis (What should be done next?)

1. **Inventory Management:** Increase stock levels for top-ranked products (iPhone 15 Pro, MacBook Pro) in high-performing regions to prevent stockouts.

2. **Marketing Strategy:** Develop targeted campaigns for "Low Value" customers (Quartile 4) to increase their purchase frequency and average order value.

3. **Regional Expansion:** Consider expanding product offerings in the South and West regions, which show growth potential but lower current penetration.

4. **Forecasting:** Use the 3-month moving average trends to predict Q3 and Q4 2024 sales and adjust procurement accordingly.

---

## Screenshots

All query results are documented in the `screenshots/` folder:

| Screenshot | Description |
|------------|-------------|
| 01_inner_join.png | INNER JOIN results showing valid transactions |
| 02_left_join.png | LEFT JOIN showing customers without transactions |
| 03_right_join.png | RIGHT JOIN showing unsold products |
| 04_full_outer_join.png | FULL OUTER JOIN complete comparison |
| 05_self_join.png | SELF JOIN regional customer pairs |
| 06_row_number.png | ROW_NUMBER() customer ranking |
| 07_rank.png | RANK() top products per region |
| 08_dense_rank.png | DENSE_RANK() customer tiers |
| 09_percent_rank.png | PERCENT_RANK() customer percentiles |
| 10_sum_over.png | SUM() OVER() running totals |
| 11_avg_over.png | AVG() OVER() moving averages |
| 12_min_max_over.png | MIN/MAX OVER() weekly highs and lows |
| 13_lag.png | LAG() month-over-month growth |
| 14_lead.png | LEAD() forward-looking analysis |
| 15_ntile.png | NTILE(4) customer segmentation |
| 16_cume_dist.png | CUME_DIST() cumulative distribution |

---

## References

1. PostgreSQL Documentation. (2024). *Window Functions*. Retrieved from https://www.postgresql.org/docs/current/tutorial-window.html

2. Oracle. (2024). *SQL Language Reference - Window Functions*. Retrieved from https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Window-Functions.html

3. W3Schools. (2024). *SQL JOINs Tutorial*. Retrieved from https://www.w3schools.com/sql/sql_join.asp

4. Mode Analytics. (2024). *SQL Window Functions*. Retrieved from https://mode.com/sql-tutorial/sql-window-functions/

5. PostgreSQL Tutorial. (2024). *PostgreSQL Window Function*. Retrieved from https://www.postgresqltutorial.com/postgresql-window-function/

---

## Academic Integrity Statement

**All sources were properly cited. Implementations and analysis represent original work. No AI-generated content was copied without attribution or adaptation.**

This project was completed independently as part of the INSY 8311 course requirements. All SQL queries were written and tested by the student, and all analysis represents original interpretation of the query results.

---

*"Whoever is faithful in very little is also faithful in much." — Luke 16:10*
