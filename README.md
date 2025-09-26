# PL/SQL Window Functions Analysis: TechRetail Pro

# NAMES:TETA KEVINE
# ID : 27973

## 📊 Project Overview

Objective: Demonstrate mastery of PL/SQL window functions by solving business problems using customers, product, and transaction tables

***

## Business Problem & Expected Outcome

| Section | Description |
|---|---|
| **Business Context** | TechRetail Pro E-commerce Technology Retailer's Business Intelligence Department |
| **Data Challenge** | The retailer struggles to efficiently segment customers, identify regional product performance, and analyze sales trends across quarters using basic SQL aggregates |
| **Expected Outcomes** | The analysis provides actionable insights to optimize marketing campaigns, inventory planning, and customer retention strategies through advanced window function analytics on transaction data |

***

## Technology Stack & Requirements

  
**Tables Used:** `customers`, `product`, `transaction`

### TABLE CREATION 

**CUSTOMER TABLE**

![Project Screenshot](customer%20sql.PNG)


![Project Screenshot](customer%20table.PNG)

**PRODUCT TABLE**

![Project Screenshot](product%20sql.PNG)


![Project Screenshot](product%20table.PNG)

**TRANSACION  TABLE**

![Project Screenshot](transaction%20sql.PNG)


![Project Screenshot](transaction%20table.PNG)
***

## Database Schema (ER Diagram)

**CUSTOMER RELATION**

![Project Screenshot](cust%20rel.PNG)

**PRO RELATION**

![Project Screenshot](pro%20rel.PNG)

**TRANS RELATION**

![Project Screenshot](trans%20rel.PNG)

***

## SQL Scripts & Analytical Queries

The core of this project is the implementation of exactly five analytical queries using PL/SQL window functions on our three-table schema:

### 1. Top 5 Products per Category/Quarter (Ranking)

**Function Used:** **`RANK()`**

```sql
SELECT category, quarter, product_name, total_rev, product_rank
FROM (
    SELECT 
        p.category,
        TO_CHAR(t.transaction_date, 'YYYY-Q') as quarter,
        p.product_name,
        SUM(t.transaction_amount) as total_rev,
        RANK() OVER (
            PARTITION BY p.category, TO_CHAR(t.transaction_date, 'YYYY-Q') 
            ORDER BY SUM(t.transaction_amount) DESC
        ) as product_rank
    FROM transaction t
    JOIN product p ON t.product_id = p.product_id
    GROUP BY p.category, TO_CHAR(t.transaction_date, 'YYYY-Q'), p.product_name
)
WHERE product_rank <= 5;

 ```


### 2. Running monthly sales totals

**Function Used:** **`SUM() OVER()`**

```sql
-- Running total by customer with customer details
SELECT 
    c.customer_name,
    c.region,
    t.transaction_date,
    p.product_name,
    t.transaction_amount,
    SUM(t.transaction_amount) OVER (
        PARTITION BY t.customer_id 
        ORDER BY t.transaction_date
        ROWS UNBOUNDED PRECEDING
    ) as running_total,
    COUNT(*) OVER (
        PARTITION BY t.customer_id 
        ORDER BY t.transaction_date
        ROWS UNBOUNDED PRECEDING
    ) as transaction_count
FROM transaction t
JOIN customers c ON t.customer_id = c.customer_id
JOIN product p ON t.product_id = p.product_id
ORDER BY c.customer_name, t.transaction_date;
   ```

### 3. MONTH-OVER-MONTH GROWTH

**Function Used:** **`LAG()/LEAD()`**

```sql

-- Month-over-month revenue growth analysis
WITH monthly_revenue AS (
    SELECT 
        TO_CHAR(transaction_date, 'YYYY-MM') as month,
        EXTRACT(YEAR FROM transaction_date) as year,
        EXTRACT(MONTH FROM transaction_date) as month_num,
        COUNT(*) as transaction_count,
        SUM(transaction_amount) as monthly_revenue,
        AVG(transaction_amount) as avg_transaction_value
    FROM transaction
    GROUP BY TO_CHAR(transaction_date, 'YYYY-MM'), 
             EXTRACT(YEAR FROM transaction_date), 
             EXTRACT(MONTH FROM transaction_date)
)
SELECT 
    month,
    transaction_count,
    monthly_revenue,
    avg_transaction_value,
    LAG(monthly_revenue, 1) OVER (ORDER BY year, month_num) as prev_month_revenue,
    CASE 
        WHEN LAG(monthly_revenue, 1) OVER (ORDER BY year, month_num) IS NOT NULL THEN
            ROUND(
                ((monthly_revenue - LAG(monthly_revenue, 1) OVER (ORDER BY year, month_num)) 
                 / LAG(monthly_revenue, 1) OVER (ORDER BY year, month_num)) * 100, 2
            )
        ELSE NULL
    END as growth_percentage,
    
    -- Using LEAD to show next month's projection (based on current growth)
    LEAD(monthly_revenue, 1) OVER (ORDER BY year, month_num) as next_month_revenue
FROM monthly_revenue
ORDER BY year, month_num;

 ```

### 5. 3-MONTH MOVING AVERAGE
**Function Used:** **`AVG() OVER()`**


```sql

-- 3-month moving average analysis
WITH daily_metrics AS (
    SELECT 
        TRUNC(transaction_date) as transaction_day,
        TO_CHAR(transaction_date, 'YYYY-MM') as transaction_month,
        COUNT(*) as daily_transactions,
        SUM(transaction_amount) as daily_revenue,
        AVG(transaction_amount) as avg_daily_transaction
    FROM transaction
    GROUP BY TRUNC(transaction_date), TO_CHAR(transaction_date, 'YYYY-MM')
),
monthly_metrics AS (
    SELECT 
        transaction_month,
        SUM(daily_transactions) as monthly_transactions,
        SUM(daily_revenue) as monthly_revenue,
        AVG(daily_revenue) as avg_daily_revenue
    FROM daily_metrics
    GROUP BY transaction_month
)
SELECT 
    transaction_month,
    monthly_transactions,
    monthly_revenue,
    avg_daily_revenue,
    ROUND(AVG(monthly_revenue) OVER (
        ORDER BY transaction_month
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ), 2) as three_month_moving_avg_revenue,
    
    ROUND(AVG(monthly_transactions) OVER (
        ORDER BY transaction_month
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ), 2) as three_month_moving_avg_transactions,
    
    -- Compare current month to moving average
    ROUND(((monthly_revenue - 
           AVG(monthly_revenue) OVER (
               ORDER BY transaction_month
               ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
           )) / 
           AVG(monthly_revenue) OVER (
               ORDER BY transaction_month
               ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
           )) * 100, 2) as variance_from_moving_avg
FROM monthly_metrics
ORDER BY transaction_month;
   ```




# 📈 Key Insights Discovered

**🏆 Top Products: iPhone 14 Pro consistently ranked #1 in smartphones category**

**📊 Customer Value: Premium customers showed 60% higher average transaction value**

**📈 Revenue Growth: Q2 2024 demonstrated 54% growth compared to Q1**

**👥 Customer Segments: Platinum customers (top 25%) contributed 45% of total revenue**

**📅 Trend Analysis: 3-month moving averages revealed stable growth patterns for forecasting**

