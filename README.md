# 🍕 Domino's Pizza Store Analysis (SQL)

![SQL](https://img.shields.io/badge/SQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Level](https://img.shields.io/badge/Level-Beginner%20to%20Intermediate-orange?style=for-the-badge)

## 📌 Project Overview

**Project Title:** Domino's Pizza Store Analysis
**Level:** Beginner to Intermediate
**Database:** `p1_dominos_db`

This project demonstrates SQL techniques used by data analysts to explore, clean, and analyze pizza sales and customer data. The analysis focuses on understanding **order patterns, revenue, customer behavior, and menu performance** to support business decision-making.

---

## 🎯 Objectives

1. **Set up the database** – Create and populate a Domino's pizza database with orders, pizzas, and customers.
2. **Data cleaning** – Identify and remove null or inconsistent records.
3. **Exploratory Data Analysis (EDA)** – Understand customer behavior, order trends, and menu performance.
4. **Business analysis** – Answer stakeholder-driven questions to derive actionable insights.

---

## 🗂️ Database Structure

| Table | Description | Columns |
|---|---|---|
| `orders` | Order-level information | `order_id`, `custId`, `order_date`, `order_time` |
| `order_details` | Line items of each order | `order_detail_id`, `order_id`, `pizza_id`, `quantity` |
| `pizzas` | Pizza size and price info | `pizza_id`, `pizza_type_id`, `size`, `price` |
| `pizza_types` | Pizza type info | `pizza_type_id`, `name`, `category` |
| `customers` | Customer info | `custId`, `first_name`, `last_name` |

### Schema Setup

```sql
CREATE DATABASE p1_dominos_db;
USE p1_dominos_db;

CREATE TABLE customers (
    custId      INT PRIMARY KEY,
    first_name  VARCHAR(50),
    last_name   VARCHAR(50)
);

CREATE TABLE pizza_types (
    pizza_type_id VARCHAR(50) PRIMARY KEY,
    name          VARCHAR(100),
    category      VARCHAR(50)
);

CREATE TABLE pizzas (
    pizza_id      VARCHAR(50) PRIMARY KEY,
    pizza_type_id VARCHAR(50),
    size          VARCHAR(5),
    price         DECIMAL(6,2),
    FOREIGN KEY (pizza_type_id) REFERENCES pizza_types(pizza_type_id)
);

CREATE TABLE orders (
    order_id   INT PRIMARY KEY,
    custId     INT,
    order_date DATE,
    order_time TIME,
    FOREIGN KEY (custId) REFERENCES customers(custId)
);

CREATE TABLE order_details (
    order_detail_id INT PRIMARY KEY,
    order_id        INT,
    pizza_id        VARCHAR(50),
    quantity        INT,
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (pizza_id) REFERENCES pizzas(pizza_id)
);
```

---

## 🧹 Data Cleaning & Exploration

- Verify total records in each table
- Check for null or missing values in critical columns
- Remove incomplete or inconsistent records

```sql
-- Record counts
SELECT 'customers' AS table_name, COUNT(*) AS total_rows FROM customers
UNION ALL SELECT 'orders', COUNT(*) FROM orders
UNION ALL SELECT 'order_details', COUNT(*) FROM order_details
UNION ALL SELECT 'pizzas', COUNT(*) FROM pizzas
UNION ALL SELECT 'pizza_types', COUNT(*) FROM pizza_types;

-- Null checks on critical columns
SELECT * FROM orders
WHERE order_id IS NULL OR custId IS NULL OR order_date IS NULL OR order_time IS NULL;

SELECT * FROM order_details
WHERE order_id IS NULL OR pizza_id IS NULL OR quantity IS NULL OR quantity <= 0;

SELECT * FROM pizzas
WHERE pizza_id IS NULL OR size IS NULL OR price IS NULL OR price <= 0;

-- Remove incomplete / inconsistent records
DELETE FROM order_details
WHERE order_id IS NULL OR pizza_id IS NULL OR quantity IS NULL OR quantity <= 0;

DELETE FROM orders
WHERE custId IS NULL OR order_date IS NULL OR order_time IS NULL;
```

---

## 📊 Analysis & Queries

### 1. Orders Volume Analysis
Total unique orders, orders by month, day-of-week analysis, repeat customers, average orders per customer, cumulative order trend.

```sql
-- Total unique orders
SELECT COUNT(DISTINCT order_id) AS total_orders FROM orders;

-- Orders by month
SELECT DATE_FORMAT(order_date, '%Y-%m') AS month, COUNT(order_id) AS total_orders
FROM orders
GROUP BY month
ORDER BY month;

-- Orders by day of week
SELECT DAYNAME(order_date) AS day_of_week, COUNT(order_id) AS total_orders
FROM orders
GROUP BY day_of_week
ORDER BY total_orders DESC;

-- Repeat customers
SELECT custId, COUNT(order_id) AS total_orders
FROM orders
GROUP BY custId
HAVING COUNT(order_id) > 1;

-- Average orders per customer
SELECT ROUND(COUNT(order_id) / COUNT(DISTINCT custId), 2) AS avg_orders_per_customer
FROM orders;

-- Cumulative order trend
SELECT order_date,
       COUNT(order_id) AS daily_orders,
       SUM(COUNT(order_id)) OVER (ORDER BY order_date) AS cumulative_orders
FROM orders
GROUP BY order_date;
```

### 2. Total Revenue from Pizza Sales

```sql
SELECT ROUND(SUM(od.quantity * p.price), 2) AS total_revenue
FROM order_details od
JOIN pizzas p ON od.pizza_id = p.pizza_id;
```

### 3. Highest-Priced Pizza

```sql
SELECT pt.name, p.size, p.price
FROM pizzas p
JOIN pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
ORDER BY p.price DESC
LIMIT 1;
```

### 4. Most Common Pizza Size Ordered

```sql
SELECT p.size, SUM(od.quantity) AS total_ordered
FROM order_details od
JOIN pizzas p ON od.pizza_id = p.pizza_id
GROUP BY p.size
ORDER BY total_ordered DESC
LIMIT 1;
```

### 5. Top 5 Most Ordered Pizza Types

```sql
SELECT pt.name, SUM(od.quantity) AS total_quantity
FROM order_details od
JOIN pizzas p ON od.pizza_id = p.pizza_id
JOIN pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.name
ORDER BY total_quantity DESC
LIMIT 5;
```

### 6. Total Quantity by Pizza Category

```sql
SELECT pt.category, SUM(od.quantity) AS total_quantity
FROM order_details od
JOIN pizzas p ON od.pizza_id = p.pizza_id
JOIN pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.category
ORDER BY total_quantity DESC;
```

### 7. Orders by Hour of the Day

```sql
SELECT HOUR(order_time) AS order_hour, COUNT(order_id) AS total_orders
FROM orders
GROUP BY order_hour
ORDER BY order_hour;
```

### 8. Category-Wise Pizza Distribution

```sql
SELECT pt.category,
       SUM(od.quantity) AS total_quantity,
       ROUND(SUM(od.quantity) * 100.0 / SUM(SUM(od.quantity)) OVER (), 2) AS pct_share
FROM order_details od
JOIN pizzas p ON od.pizza_id = p.pizza_id
JOIN pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.category
ORDER BY pct_share DESC;
```

### 9. Average Pizzas Ordered per Day

```sql
SELECT ROUND(AVG(daily_qty), 2) AS avg_pizzas_per_day
FROM (
    SELECT o.order_date, SUM(od.quantity) AS daily_qty
    FROM orders o
    JOIN order_details od ON o.order_id = od.order_id
    GROUP BY o.order_date
) AS daily;
```

### 10. Top 3 Pizzas by Revenue

```sql
SELECT pt.name, ROUND(SUM(od.quantity * p.price), 2) AS revenue
FROM order_details od
JOIN pizzas p ON od.pizza_id = p.pizza_id
JOIN pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.name
ORDER BY revenue DESC
LIMIT 3;
```

### 11. Revenue Contribution per Pizza

```sql
SELECT pt.name,
       ROUND(SUM(od.quantity * p.price), 2) AS revenue,
       ROUND(SUM(od.quantity * p.price) * 100.0 / SUM(SUM(od.quantity * p.price)) OVER (), 2) AS pct_contribution
FROM order_details od
JOIN pizzas p ON od.pizza_id = p.pizza_id
JOIN pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.name
ORDER BY pct_contribution DESC;
```

### 12. Cumulative Revenue Over Time

```sql
SELECT month,
       monthly_revenue,
       ROUND(SUM(monthly_revenue) OVER (ORDER BY month), 2) AS cumulative_revenue
FROM (
    SELECT DATE_FORMAT(o.order_date, '%Y-%m') AS month,
           SUM(od.quantity * p.price) AS monthly_revenue
    FROM orders o
    JOIN order_details od ON o.order_id = od.order_id
    JOIN pizzas p ON od.pizza_id = p.pizza_id
    GROUP BY month
) AS m;
```

### 13. Top 3 Pizzas by Category (Revenue-Based)

```sql
SELECT category, name, revenue
FROM (
    SELECT pt.category, pt.name,
           ROUND(SUM(od.quantity * p.price), 2) AS revenue,
           RANK() OVER (PARTITION BY pt.category ORDER BY SUM(od.quantity * p.price) DESC) AS rnk
    FROM order_details od
    JOIN pizzas p ON od.pizza_id = p.pizza_id
    JOIN pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
    GROUP BY pt.category, pt.name
) ranked
WHERE rnk <= 3;
```

### 14. Top 10 Customers by Spending

```sql
SELECT c.custId, CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
       ROUND(SUM(od.quantity * p.price), 2) AS total_spent
FROM customers c
JOIN orders o ON c.custId = o.custId
JOIN order_details od ON o.order_id = od.order_id
JOIN pizzas p ON od.pizza_id = p.pizza_id
GROUP BY c.custId, customer_name
ORDER BY total_spent DESC
LIMIT 10;
```

### 15. Orders by Weekday

```sql
SELECT DAYNAME(order_date) AS weekday, COUNT(DISTINCT order_id) AS total_orders
FROM orders
GROUP BY weekday, DAYOFWEEK(order_date)
ORDER BY DAYOFWEEK(order_date);
```

### 16. Average Order Size

```sql
SELECT ROUND(SUM(quantity) / COUNT(DISTINCT order_id), 2) AS avg_pizzas_per_order
FROM order_details;
```

### 17. Seasonal Trends

```sql
SELECT MONTHNAME(o.order_date) AS month_name,
       SUM(od.quantity) AS pizzas_sold,
       ROUND(SUM(od.quantity * p.price), 2) AS revenue
FROM orders o
JOIN order_details od ON o.order_id = od.order_id
JOIN pizzas p ON od.pizza_id = p.pizza_id
GROUP BY month_name, MONTH(o.order_date)
ORDER BY MONTH(o.order_date);
```

### 18. Revenue by Pizza Size

```sql
SELECT p.size, ROUND(SUM(od.quantity * p.price), 2) AS revenue
FROM order_details od
JOIN pizzas p ON od.pizza_id = p.pizza_id
GROUP BY p.size
ORDER BY revenue DESC;
```

### 19. Customer Segmentation

```sql
-- Threshold (500) is adjustable based on your spend distribution
SELECT c.custId, CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
       ROUND(SUM(od.quantity * p.price), 2) AS total_spent,
       CASE WHEN SUM(od.quantity * p.price) >= 500 THEN 'High Value'
            ELSE 'Regular' END AS segment
FROM customers c
JOIN orders o ON c.custId = o.custId
JOIN order_details od ON o.order_id = od.order_id
JOIN pizzas p ON od.pizza_id = p.pizza_id
GROUP BY c.custId, customer_name;
```

### 20. Repeat Customer Rate

```sql
SELECT ROUND(
    SUM(CASE WHEN order_count > 1 THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2
) AS repeat_customer_rate_pct
FROM (
    SELECT custId, COUNT(order_id) AS order_count
    FROM orders
    GROUP BY custId
) t;
```

---

## 🔑 Key Findings

- **Customer Behavior:** High-value and repeat customers identified.
- **Order Trends:** Peak hours, weekends, and seasonal patterns discovered.
- **Menu Insights:** Top-selling pizzas, revenue contributors, and popular sizes identified.
- **Revenue Analysis:** Monthly revenue, cumulative trends, and category-wise contributions analyzed.
- **Operational Insights:** Average order size, daily pizzas, and staffing optimization recommendations provided.

---

## 🛠️ SQL Concepts Used

`JOIN` · `GROUP BY` · `HAVING` · `CASE WHEN` · Subqueries · Window Functions (`SUM() OVER`, `RANK() OVER`) · Date & Time Functions · Aggregations · Data Cleaning

---

## 🚀 How to Run


1. Create the database and tables using the schema above
2. Import the dataset (CSV files) into the respective tables.
3. Run the cleaning and analysis queries in MySQL Workbench (or any SQL client).

---

## 👤 Author

** Shubham Mishra**
📧 [shubhammisra01@gmail.com]

⭐ If you found this project helpful, consider giving it a star!
