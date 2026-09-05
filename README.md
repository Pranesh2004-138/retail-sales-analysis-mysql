# Retail Sales Analysis – MySQL Project

## Project Overview

**Project Title:** Retail Sales Analysis

**Project Level:** Intermediate

**Database:** `sales_db`

**Table:** `sale`

**Database Technology:** MySQL

This project focuses on analyzing retail sales data using **MySQL**. The objective is to demonstrate practical SQL skills including database exploration, data cleaning, aggregation, filtering, date-based analysis, customer analysis, and business-oriented reporting.

The project uses SQL queries to identify sales trends, customer purchasing behavior, product category performance, high-value transactions, and order patterns based on different time shifts.

---

## Objectives

The main objectives of this project are:

1. **Explore the retail sales dataset** and understand its structure.
2. **Clean the dataset** by identifying and removing records containing missing values.
3. **Perform exploratory data analysis (EDA)** using SQL.
4. **Analyze sales performance** across different product categories.
5. **Analyze customer purchasing behavior**.
6. **Identify high-value sales transactions**.
7. **Analyze monthly sales trends** and identify the best-performing months.
8. **Identify top customers** based on total sales.
9. **Analyze customer distribution across product categories**.
10. **Analyze order volume by time of day**.

---

# Project Structure

The project is divided into the following stages:

### 1. Database & Data Exploration

* Connect to the `sales_db` database.
* Explore the sales table.
* Inspect the available columns and records.
* Fix the incorrectly imported transaction ID column name.

### 2. Data Cleaning

* Identify records containing NULL values.
* Remove incomplete records.
* Ensure the dataset is suitable for analysis.

### 3. Exploratory Data Analysis

* Analyze sales by date.
* Analyze sales by category.
* Analyze customers by gender.
* Analyze transaction values.
* Analyze monthly sales performance.

### 4. Business Analysis

* Identify high-value transactions.
* Calculate category-wise sales.
* Find top customers.
* Calculate unique customers by category.
* Analyze sales by customer gender and category.
* Analyze orders by time-of-day shift.

---

# 1. Database & Data Exploration

The first step was to inspect the sales data stored in the `sales_db` database.

```sql
SELECT * 
FROM sales_db.sale;
```

During the initial inspection, the transaction ID column contained an incorrectly imported column name. The column was renamed for easier analysis.

```sql
ALTER TABLE sales_db.sale
RENAME COLUMN ï»¿transactions_id TO transactions_id;
```

The table was then checked again:

```sql
SELECT *
FROM sales_db.sale;
```

---

# 2. Data Cleaning

Data cleaning is an important step before performing analysis.

The dataset was checked for missing values across important columns including:

* Transaction ID
* Sale Date
* Sale Time
* Customer ID
* Gender
* Age
* Category
* Quantity
* Price per Unit
* COGS
* Total Sale

### Identify NULL Records

```sql
SELECT *
FROM sales_db.sale
WHERE transactions_id IS NULL
   OR sale_date IS NULL
   OR sale_time IS NULL
   OR customer_id IS NULL
   OR gender IS NULL
   OR age IS NULL
   OR category IS NULL
   OR quantiy IS NULL
   OR price_per_unit IS NULL
   OR cogs IS NULL
   OR total_sale IS NULL;
```

### Remove Incomplete Records

```sql
DELETE FROM sales_db.sale
WHERE transactions_id IS NULL
   OR sale_date IS NULL
   OR sale_time IS NULL
   OR customer_id IS NULL
   OR gender IS NULL
   OR age IS NULL
   OR category IS NULL
   OR quantiy IS NULL
   OR price_per_unit IS NULL
   OR cogs IS NULL
   OR total_sale IS NULL;
```

This ensures that incomplete records do not affect the analysis.

---

# 3. Data Analysis & Business Questions

The following SQL queries were developed to answer important business questions from the retail sales dataset.

---

## 1. Retrieve all sales made on November 5, 2022

This query retrieves all transactions that occurred on a specific date.

```sql
SELECT *
FROM sales_db.sale
WHERE sale_date = '2022-11-05';
```

### Business Purpose

This can be used to investigate daily sales activity and understand transactions made on a particular date.

---

## 2. Find Clothing transactions with quantity greater than 4 in November 2022

```sql
SELECT *
FROM sales_db.sale
WHERE category = 'Clothing'
  AND DATE_FORMAT(sale_date, '%Y-%m') = '2022-11'
  AND quantiy > 4;
```

### Business Purpose

This helps identify bulk purchases in the Clothing category during November 2022.

---

## 3. Calculate total sales for each product category

```sql
SELECT
    category,
    SUM(total_sale) AS net_sales
FROM sales_db.sale
GROUP BY category;
```

### Business Purpose

This analysis helps determine which product categories generate the highest revenue.

---

## 4. Calculate the average customer age for the Beauty category

```sql
SELECT
    ROUND(AVG(age), 2) AS average_age
FROM sales_db.sale
WHERE category = 'Beauty';
```

### Business Purpose

This helps understand the average age of customers purchasing products from the Beauty category.

---

## 5. Find high-value transactions above 1000

```sql
SELECT *
FROM sales_db.sale
WHERE total_sale > 1000;
```

### Business Purpose

High-value transactions can be analyzed to understand premium purchases and identify customers making larger purchases.

---

## 6. Calculate the number of transactions by gender and category

```sql
SELECT
    category,
    gender,
    COUNT(*) AS transactions
FROM sales_db.sale
GROUP BY gender, category
ORDER BY category;
```

### Business Purpose

This analysis helps understand purchasing patterns across different customer genders and product categories.

---

## 7. Calculate average sales for each month

```sql
SELECT
    YEAR(sale_date) AS years,
    MONTH(sale_date) AS months,
    ROUND(AVG(total_sale), 2) AS avg_sale
FROM sales_db.sale
GROUP BY YEAR(sale_date), MONTH(sale_date)
ORDER BY years ASC, avg_sale DESC;
```

### Business Purpose

Monthly analysis helps identify sales trends and seasonal changes in customer spending.

### Best-Selling Month for Each Year

To identify the month with the highest average sale in each year:

```sql
WITH monthly_sales AS
(
    SELECT
        YEAR(sale_date) AS sales_year,
        MONTH(sale_date) AS sales_month,
        AVG(total_sale) AS avg_sale
    FROM sales_db.sale
    GROUP BY
        YEAR(sale_date),
        MONTH(sale_date)
),
ranked_sales AS
(
    SELECT
        sales_year,
        sales_month,
        avg_sale,
        RANK() OVER (
            PARTITION BY sales_year
            ORDER BY avg_sale DESC
        ) AS sales_rank
    FROM monthly_sales
)
SELECT
    sales_year,
    sales_month,
    ROUND(avg_sale, 2) AS avg_sale
FROM ranked_sales
WHERE sales_rank = 1;
```

---

## 8. Find the Top 5 Customers Based on Total Sales

```sql
SELECT
    customer_id,
    SUM(total_sale) AS total_sales
FROM sales_db.sale
GROUP BY customer_id
ORDER BY total_sales DESC
LIMIT 5;
```

### Business Purpose

This identifies the highest-value customers based on their total spending.

These customers can be considered for loyalty programs, targeted promotions, and personalized marketing campaigns.

---

## 9. Find the number of unique customers in each category

```sql
SELECT
    category,
    COUNT(DISTINCT customer_id) AS unique_customers
FROM sales_db.sale
GROUP BY category;
```

### Business Purpose

This helps measure customer reach across different product categories and identify categories with a larger customer base.

---

## 10. Analyze orders by time-of-day shift

The sales data was divided into three shifts:

* **Morning:** Before 12 PM
* **Afternoon:** 12 PM – 5 PM
* **Evening:** After 5 PM

```sql
WITH hourstb AS
(
    SELECT *,
        CASE
            WHEN EXTRACT(HOUR FROM sale_time) < 12
                THEN 'Morning'
            WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17
                THEN 'Afternoon'
            ELSE 'Evening'
        END AS shift
    FROM sales_db.sale
)
SELECT
    shift,
    COUNT(transactions_id) AS total_orders
FROM hourstb
GROUP BY shift;
```

### Business Purpose

This analysis helps identify the busiest time periods and can support decisions related to staffing, promotions, and operational planning.

---

# Key Findings

The SQL analysis provides several useful business insights:

### 1. Category Performance

Category-level sales analysis helps identify the product categories contributing the most revenue.

### 2. Customer Insights

The project identifies the top five customers based on total sales and measures the number of unique customers purchasing from each category.

### 3. High-Value Transactions

Transactions above 1000 can be isolated to understand premium purchasing behavior.

### 4. Sales Trends

Monthly average sales analysis provides visibility into changes in customer spending over time and helps identify the strongest-performing month in each year.

### 5. Customer Demographics

The analysis examines customer age and gender across different product categories.

### 6. Time-Based Sales Patterns

Sales are classified into Morning, Afternoon, and Evening shifts to identify periods with higher transaction volumes.

---

# Reports & Analysis

The project can be used to generate the following reports:

### Sales Performance Report

Includes:

* Total sales by category
* Monthly average sales
* High-value transactions
* Best-performing months

### Customer Analysis Report

Includes:

* Top 5 customers
* Unique customers by category
* Average customer age
* Customer gender distribution

### Transaction Analysis Report

Includes:

* Number of transactions by category
* Transactions by gender
* Orders by time-of-day shift
* Daily transaction analysis

---

# Technologies Used

* **MySQL**
* **SQL**
* **MySQL Workbench**
* **GitHub**

---

# SQL Concepts Demonstrated

This project demonstrates practical usage of:

* `SELECT`
* `WHERE`
* `DELETE`
* `ALTER TABLE`
* `RENAME COLUMN`
* `GROUP BY`
* `ORDER BY`
* `LIMIT`
* `DISTINCT`
* `COUNT()`
* `SUM()`
* `AVG()`
* `ROUND()`
* `DATE_FORMAT()`
* `YEAR()`
* `MONTH()`
* `EXTRACT()`
* `CASE`
* `WITH` / CTE
* Window Functions
* `RANK()`
* Data Cleaning
* NULL handling
* Aggregation
* Business-oriented SQL analysis

---

# Skills Demonstrated

Through this project, I demonstrated the ability to:

* Clean and prepare raw datasets using SQL.
* Identify and remove incomplete records.
* Perform exploratory data analysis.
* Analyze sales performance using aggregation functions.
* Work with date and time data.
* Analyze customer purchasing behavior.
* Create business-focused SQL queries.
* Use CTEs and window functions for advanced analysis.
* Extract actionable insights from transactional data.
* Translate business questions into SQL queries.

---

# Conclusion

This Retail Sales Analysis project demonstrates how SQL can be used to transform raw transactional data into meaningful business insights.

The project covers the complete analytical workflow, starting from **data exploration and cleaning**, followed by **exploratory analysis**, **customer analysis**, **sales trend analysis**, and **business-focused reporting**.

By analyzing product categories, customer behavior, transaction values, monthly sales trends, and order timing, the project provides a practical foundation for using SQL in real-world **Data Analyst, MIS Analyst, and Data Management** roles.

The project demonstrates practical knowledge of **MySQL, data cleaning, data analysis, aggregation, date functions, CTEs, window functions, and business intelligence concepts**.
