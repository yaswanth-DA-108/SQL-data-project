# SQL-data-project
## 📘 Project Overview  A retail analytics project built with SQL Server, focused on extracting insights from sales data. It includes customer segmentation, performance tracking, and reusable reporting views to support business intelligence and decision-making.
# 🧠 Business Intelligence SQL Analysis

This project contains a comprehensive set of SQL queries designed to explore and analyze a retail sales database. It covers everything from basic data exploration to advanced business intelligence reporting.

---

## 📌 Author

**Yaswanth**  
Aspiring Data Analyst | SQL Enthusiast 

---

## 🛠️ Technologies Used

- SQL Server / T-SQL  
- Window Functions  
- Aggregate Functions  
- Date Functions  
- Views for Reporting  

---

## 📄 SQL Code

```sql
/* ============================
   DATABASE EXPLORATION
   ============================ */

-- List all tables in the database
SELECT * FROM INFORMATION_SCHEMA.TABLES;

-- List all columns in the 'dim_customers' table
SELECT * FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'dim_customers';


/* ============================
   DIMENSION EXPLORATION
   ============================ */

-- Get distinct countries where customers are located
SELECT DISTINCT country FROM gold.dim_customers;

-- Get all product categories, subcategories, and product names
SELECT DISTINCT category, subcategory, product_name 
FROM gold.dim_products
ORDER BY category, subcategory, product_name;


/* ============================
   DATE EXPLORATION
   ============================ */

-- Find the first and last order dates and calculate the number of years of sales
SELECT 
    MIN(order_date) AS First_OrderDate,
    MAX(order_date) AS Last_OrderDate,
    DATEDIFF(YEAR, MIN(order_date), MAX(order_date)) AS Available_Sales_Years
FROM gold.fact_sales;

-- Identify the youngest and oldest customers
WITH min_max AS (
    SELECT *, 
           MIN(birthdate) OVER () AS oldest, 
           MAX(birthdate) OVER () AS youngest
    FROM gold.dim_customers
)
SELECT CONCAT(first_name, ' ', last_name) AS Name, 
       DATEDIFF(YEAR, birthdate, GETDATE()) AS Age 
FROM min_max
WHERE birthdate = oldest OR birthdate = youngest;


/* ============================
   MEASURE EXPLORATION
   ============================ */

-- Total sales amount
SELECT SUM(sales_amount) AS total_sales FROM gold.fact_sales;

-- Total quantity of items sold
SELECT SUM(quantity) AS total_items_sold FROM gold.fact_sales;

-- Average selling price
SELECT AVG(price) AS avg_price FROM gold.fact_sales;

-- Total number of orders
SELECT COUNT(DISTINCT order_number) AS total_orders FROM gold.fact_sales;

-- Total number of products
SELECT COUNT(DISTINCT product_key) AS total_products FROM gold.dim_products;

-- Total number of customers
SELECT COUNT(DISTINCT customer_key) AS total_customers FROM gold.dim_customers;

-- Number of customers who placed orders
SELECT COUNT(DISTINCT customer_key) AS active_customers FROM gold.fact_sales;


/* ============================
   BUSINESS METRICS REPORT
   ============================ */

-- Consolidated business metrics in a single query
SELECT 'Total Sales' AS Metric_Name, SUM(sales_amount) AS Metric_Value FROM gold.fact_sales
UNION ALL
SELECT 'Total Items Sold', SUM(quantity) FROM gold.fact_sales
UNION ALL
SELECT 'Average Selling Price', AVG(price) FROM gold.fact_sales
UNION ALL
SELECT 'Total Orders', COUNT(DISTINCT order_number) FROM gold.fact_sales
UNION ALL
SELECT 'Total Products', COUNT(DISTINCT product_key) FROM gold.dim_products
UNION ALL
SELECT 'Total Customers', COUNT(DISTINCT customer_key) FROM gold.dim_customers
UNION ALL
SELECT 'Customers Who Placed Orders', COUNT(DISTINCT customer_key) FROM gold.fact_sales;


/* ============================
   MAGNITUDE ANALYSIS
   ============================ */

-- Number of customers by country
SELECT country, COUNT(customer_key) AS NumberOfCustomers
FROM gold.dim_customers
GROUP BY country
ORDER BY NumberOfCustomers DESC;

-- Number of customers by gender
SELECT gender, COUNT(customer_key) AS NumberOfCustomers
FROM gold.dim_customers
GROUP BY gender
ORDER BY NumberOfCustomers DESC;

-- Number of products by category
SELECT category, COUNT(*) AS Total_Products
FROM gold.dim_products
GROUP BY category
ORDER BY Total_Products DESC;

-- Average cost per category
SELECT category, AVG(cost) AS Avg_Cost
FROM gold.dim_products
WHERE category IS NOT NULL
GROUP BY category
ORDER BY Avg_Cost DESC;


/* ============================
   REVENUE ANALYSIS
   ============================ */

-- Revenue by country
SELECT c.country AS Country, SUM(s.sales_amount) AS TotalSales
FROM gold.dim_customers c
JOIN gold.fact_sales s ON c.customer_key = s.customer_key
GROUP BY c.country
ORDER BY TotalSales DESC;

-- Revenue by customer
SELECT c.customer_key AS CustomerKey, SUM(s.sales_amount) AS TotalSales
FROM gold.dim_customers c
JOIN gold.fact_sales s ON c.customer_key = s.customer_key
GROUP BY c.customer_key
ORDER BY TotalSales DESC;

-- Items sold by country
SELECT c.country AS Country, SUM(s.quantity) AS TotalItemsSold
FROM gold.dim_customers c
JOIN gold.fact_sales s ON c.customer_key = s.customer_key
GROUP BY c.country
ORDER BY TotalItemsSold DESC;


/* ============================
   RANKING ANALYSIS
   ============================ */

-- Top 5 products by revenue
WITH TopProducts AS (
    SELECT TOP 5 product_key, SUM(price) AS Revenue
    FROM gold.fact_sales
    GROUP BY product_key
    ORDER BY Revenue DESC
)
SELECT tp.*, p.product_name AS Product
FROM TopProducts tp
LEFT JOIN gold.dim_products p ON tp.product_key = p.product_key;

-- Bottom 5 products by revenue
WITH BottomProducts AS (
    SELECT TOP 5 product_key, SUM(price) AS Revenue
    FROM gold.fact_sales
    GROUP BY product_key
    ORDER BY Revenue ASC
)
SELECT bp.*, p.product_name AS Product
FROM BottomProducts bp
LEFT JOIN gold.dim_products p ON bp.product_key = p.product_key;

-- Top 10 customers by revenue
SELECT TOP 10 c.first_name, c.last_name, SUM(s.sales_amount) AS Revenue
FROM gold.fact_sales s
JOIN gold.dim_customers c ON c.customer_key = s.customer_key
GROUP BY c.first_name, c.last_name
ORDER BY Revenue DESC;


/* ============================
   TIME SERIES ANALYSIS
   ============================ */

-- Monthly sales performance
SELECT
    YEAR(order_date) AS order_year,
    MONTH(order_date) AS order_month,
    SUM(sales_amount) AS total_sales,
    COUNT(DISTINCT customer_key) AS total_customers,
    SUM(quantity) AS total_quantity
FROM gold.fact_sales
WHERE order_date IS NOT NULL
GROUP BY YEAR(order_date), MONTH(order_date)
ORDER BY YEAR(order_date), MONTH(order_date;


/* ============================
   CUMULATIVE ANALYSIS
   ============================ */

-- Running total and moving average of sales
SELECT
    order_date,
    total_sales,
    SUM(total_sales) OVER (ORDER BY order_date) AS running_total_sales,
    AVG(avg_price) OVER (ORDER BY order_date) AS moving_average_price
FROM (
    SELECT 
        DATETRUNC(month, order_date) AS order_date,
        SUM(sales_amount) AS total_sales,
        AVG(price) AS avg_price
    FROM gold.fact_sales
    WHERE order_date IS NOT NULL
    GROUP BY DATETRUNC(month, order_date)
) t;


/* ============================
   YEAR-OVER-YEAR PERFORMANCE
   ============================ */

-- Compare product revenue year-over-year and against average
WITH cteYOY AS (
    SELECT p.product_name, COALESCE(YEAR(s.order_date), YEAR(shipping_date)) AS year,
           SUM(s.sales_amount) AS Revenue
    FROM gold.dim_products p
    JOIN gold.fact_sales s ON p.product_key = s.product_key
    GROUP BY p.product_name, COALESCE(YEAR(s.order_date), YEAR(shipping_date))
)
SELECT *, 
       AVG(revenue) OVER(PARTITION BY product_name) AS average_sales,
       Revenue - AVG(revenue) OVER(PARTITION BY product_name) AS Difference_with_avgSales,
       CASE 
           WHEN Revenue - AVG(revenue) OVER(PARTITION BY product_name) > 0 THEN 'Above Average'
           WHEN Revenue - AVG(revenue) OVER(PARTITION BY product_name) < 0 THEN 'Below Average'
           ELSE 'Average'
       END AS performance_Average,
       LAG(revenue) OVER(PARTITION BY product_name ORDER BY year) AS last_year_sales,
       CASE 
           WHEN Revenue > LAG(revenue) OVER(PARTITION BY product_name ORDER BY year) THEN 'Increase'
           WHEN Revenue < LAG(revenue) OVER(PARTITION BY product_name ORDER BY year) THEN 'Decrease'
           ELSE 'No Change'
       END AS compared_to_last_year
FROM cteYOY;


/* ============================
   SEGMENTATION ANALYSIS
   ============================ */

-- Segment products by cost
SELECT segment, COUNT(*) AS number_of_products
FROM (
    SELECT *,
           CASE 
               WHEN cost > 1500 THEN 'Luxury'
               WHEN cost > 800 THEN 'Costly'
               WHEN cost > 500 THEN 'Budget'
               ELSE 'Cheap'
           END AS segment
    FROM gold.dim_products
) AS sub
GROUP BY segment;

-- Segment customers by spending and lifespan
WITH customer_spending AS (
    SELECT
        c.customer_key,
        SUM(f.sales_amount) AS total_spending,
        MIN(order_date) AS first_order,
        MAX(order_date) AS last_order,
        D


-- =============================================================================
-- Part-to-Whole Analysis: Category Contribution to Overall Sales
-- =============================================================================
SELECT category, Total_sales,
       SUM(Total_sales) OVER() AS Overall_Revenue,
       ROUND(CAST(Total_sales AS FLOAT) / SUM(Total_sales) OVER() * 100, 2) AS contribution
FROM (
    SELECT p.category, SUM(s.sales_amount) AS Total_sales
    FROM gold.dim_products p
    JOIN gold.fact_sales s ON p.product_key = s.product_key
    GROUP BY p.category
) a
ORDER BY contribution DESC;

-- =============================================================================
-- Customer Report View: gold.report_customers
-- =============================================================================
IF OBJECT_ID('gold.report_customers', 'V') IS NOT NULL
    DROP VIEW gold.report_customers;
GO

CREATE VIEW gold.report_customers AS
WITH base_query AS (
    SELECT
        f.order_number,
        f.product_key,
        f.order_date,
        f.sales_amount,
        f.quantity,
        c.customer_key,
        c.customer_number,
        CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
        DATEDIFF(YEAR, c.birthdate, GETDATE()) AS age
    FROM gold.fact_sales f
    LEFT JOIN gold.dim_customers c ON c.customer_key = f.customer_key
    WHERE order_date IS NOT NULL
),
customer_aggregation AS (
    SELECT 
        customer_key,
        customer_number,
        customer_name,
        age,
        COUNT(DISTINCT order_number) AS total_orders,
        SUM(sales_amount) AS total_sales,
        SUM(quantity) AS total_quantity,
        COUNT(DISTINCT product_key) AS total_products,
        MAX(order_date) AS last_order_date,
        DATEDIFF(MONTH, MIN(order_date), MAX(order_date)) AS lifespan
    FROM base_query
    GROUP BY customer_key, customer_number, customer_name, age
)
SELECT
    customer_key,
    customer_number,
    customer_name,
    age,
    CASE 
        WHEN age < 20 THEN 'Under 20'
        WHEN age BETWEEN 20 AND 29 THEN '20-29'
        WHEN age BETWEEN 30 AND 39 THEN '30-39'
        WHEN age BETWEEN 40 AND 49 THEN '40-49'
        ELSE '50 and above'
    END AS age_group,
    CASE 
        WHEN lifespan >= 12 AND total_sales > 5000 THEN 'VIP'
        WHEN lifespan >= 12 AND total_sales <= 5000 THEN 'Regular'
        ELSE 'New'
    END AS customer_segment,
    last_order_date,
    DATEDIFF(MONTH, last_order_date, GETDATE()) AS recency,
    total_orders,
    total_sales,
    total_quantity,
    total_products,
    lifespan,
    CASE WHEN total_sales = 0 THEN 0 ELSE total_sales / total_orders END AS avg_order_value,
    CASE WHEN lifespan = 0 THEN total_sales ELSE total_sales / lifespan END AS avg_monthly_spend
FROM customer_aggregation;

-- =============================================================================
-- Product Report View: gold.report_products
-- =============================================================================
IF OBJECT_ID('gold.report_products', 'V') IS NOT NULL
    DROP VIEW gold.report_products;
GO

CREATE VIEW gold.report_products AS
WITH base_query AS (
    SELECT
        f.order_number,
        f.order_date,
        f.customer_key,
        f.sales_amount,
        f.quantity,
        p.product_key,
        p.product_name,
        p.category,
        p.subcategory,
        p.cost
    FROM gold.fact_sales f
    LEFT JOIN gold.dim_products p ON f.product_key = p.product_key
    WHERE order_date IS NOT NULL
),
product_aggregations AS (
    SELECT
        product_key,
        product_name,
        category,
        subcategory,
        cost,
        DATEDIFF(MONTH, MIN(order_date), MAX(order_date)) AS lifespan,
        MAX(order_date) AS last_sale_date,
        COUNT(DISTINCT order_number) AS total_orders,
        COUNT(DISTINCT customer_key) AS total_customers,
        SUM(sales_amount) AS total_sales,
        SUM(quantity) AS total_quantity,
        ROUND(AVG(CAST(sales_amount AS FLOAT) / NULLIF(quantity, 0)), 1) AS avg_selling_price
    FROM base_query
    GROUP BY product_key, product_name, category, subcategory, cost
)
SELECT 
    product_key,
    product_name,
    category,
    subcategory,
    cost,
    last_sale_date,
    DATEDIFF(MONTH, last_sale_date, GETDATE()) AS recency_in_months,
    CASE
        WHEN total_sales > 50000 THEN 'High-Performer'
        WHEN total_sales >= 10000 THEN 'Mid-Range'
        ELSE 'Low-Performer'
    END AS product_segment,
    lifespan,
    total_orders,
    total_sales,
    total_quantity,
    total_customers,
    avg_selling_price,
    CASE WHEN total_orders = 0 THEN 0 ELSE total_sales / total_orders END AS avg_order_revenue,
    CASE WHEN lifespan = 0 THEN total_sales ELSE total_sales / lifespan END AS avg_monthly_revenue
FROM product_aggregations;
