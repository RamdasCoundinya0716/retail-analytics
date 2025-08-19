# 🛍️ Retail Analytics Project Using Snowflake Schema on Databricks

## 📌 Project Overview

This project simulates a real-world retail analytics pipeline (inspired by companies like Nike) using a synthetic dataset. It follows the Snowflake schema approach and is built entirely on Databricks, using SQL and PySpark.

## 🧱 Tech Stack

* **Platform:** Databricks (Community Edition)
* **Languages:** PySpark, SQL
* **Storage:** Delta Tables on DBFS
* **Schema Type:** Snowflake Schema
* **Visualization:** Databricks SQL Dashboard

## 🗃️ Dataset Description

The project is based on a denormalized dataset `master_customer_data` containing:

| Column            | Description                               |
| ----------------- | ----------------------------------------- |
| customer\_id      | Unique ID for each customer               |
| region            | Region of customer                        |
| signup\_date      | Signup date of customer                   |
| order\_id         | Unique order transaction ID               |
| order\_date       | Date of order                             |
| product\_id       | Product identifier                        |
| product\_category | Category like Shoes, Apparel, Accessories |
| product\_price    | Price of the product                      |
| payment\_id       | Unique payment transaction ID             |
| payment\_method   | UPI, Card, COD                            |
| payment\_status   | Success, Failed, Pending, etc.            |

## ❄️ Snowflake Schema Design

* `customers (customer_id, region, signup_date)`
* `orders (order_id, customer_id, order_date, product_id, product_price)`
* `products (product_id, product_category, product_price)`
* `payments (payment_id, order_id, payment_method, payment_status)`

## ER Diagram
<img width="2076" height="1086" alt="image" src="https://github.com/user-attachments/assets/59a9a2ab-ebe3-4641-8616-1f9b537babec" />


## 🧽 Data Cleaning Steps

* Standardizing column formats and casing
* Replacing typos (e.g., 'sucess' → 'success')
* Handling missing values:

  * `signup_date` → filled with `order_date`
  * `product_category` → filled with 'Unknown'
  * `payment_status` → filled with 'Pending'
* Capping `product_price` between ₹500 and ₹100,000
* Filtering out rows with null or invalid foreign keys

## 🧠 Business Questions + SQL Queries

1. **Which region has the highest number of customers?**

```sql
SELECT region, COUNT(DISTINCT customer_id) AS total_customers
FROM retail_db.customers
GROUP BY region
ORDER BY total_customers DESC;
```

2. **Average Order Value (AOV) per region:**

```sql
SELECT c.region, ROUND(AVG(o.product_price), 2) AS avg_order_value
FROM retail_db.orders o
JOIN retail_db.customers c ON o.customer_id = c.customer_id
GROUP BY c.region;
```

3. **Top-selling product categories by revenue:**

```sql
SELECT product_category, SUM(product_price) AS total_revenue
FROM retail_db.products
GROUP BY product_category
ORDER BY total_revenue DESC;
```

4. **Payment method distribution:**

```sql
SELECT payment_method, COUNT(*) AS total_payments
FROM retail_db.payments
GROUP BY payment_method
ORDER BY total_payments DESC;
```

5. **High-value customers (top 5% by spend):**

```sql
WITH customer_spend AS (
  SELECT customer_id, SUM(product_price) AS total_spent
  FROM retail_db.orders
  GROUP BY customer_id
)
SELECT *
FROM customer_spend
WHERE total_spent > (
  SELECT PERCENTILE(total_spent, 0.95) FROM customer_spend
);
```

6. **Orders with 'Pending' payment status:**

```sql
SELECT p.order_id, p.payment_status
FROM retail_db.payments p
WHERE p.payment_status = 'pending';
```

7. **Monthly revenue trend (last 6 months):**

```sql
SELECT DATE_TRUNC('month', order_date) AS month, SUM(product_price) AS revenue
FROM retail_db.orders
WHERE order_date >= ADD_MONTHS(CURRENT_DATE(), -6)
GROUP BY month
ORDER BY month;
```

## 📊 Dashboard KPIs (Databricks SQL)

* Total Revenue
* Orders by Region
* Payment Method Split
* Category Revenue Leaderboard
* High-Value Customer Heatmap

## 🔗 Author

Ramdas Coudinya VK
[GitHub](https://github.com/RamdasCoundinya0716) | [LinkedIn](https://www.linkedin.com/in/ramdascoudinya)
