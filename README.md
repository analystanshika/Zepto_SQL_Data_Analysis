# 🛒 Zepto E-commerce SQL Data Analyst Portfolio Project

This is a complete, real-world data analyst portfolio project based on an e-commerce inventory dataset scraped from **Zepto** – one of India’s fastest-growing quick-commerce startups. This project simulates real analyst workflows, from raw data exploration to business-focused data analysis.

This project is perfect for:
* 📊 Data Analyst aspirants who want to build a strong Portfolio Project for interviews and LinkedIn
* 📚 Anyone learning SQL hands-on
* 💼 Preparing for interviews in retail, e-commerce, or product analytics

---

## 📌 Project Overview

The goal is to simulate how actual data analysts in the e-commerce or retail industries work behind the scenes to use SQL to:

- [x] **Set up** a messy, real-world e-commerce inventory database
- [x] **Perform** Exploratory Data Analysis (EDA) to explore product categories, availability, and pricing inconsistencies
- [x] **Implement** Data Cleaning to handle null values, remove invalid entries, and convert pricing from paise to rupees
- [x] **Write** business-driven SQL queries to derive insights around pricing, inventory, stock availability, revenue and more

---

## 📁 Dataset Overview

The dataset was sourced from **Kaggle** and was originally scraped from Zepto's official product listings. It mimics what you'd typically encounter in a real-world e-commerce inventory system.

Each row represents a unique **SKU (Stock Keeping Unit)** for a product. Duplicate product names exist because the same product may appear multiple times in different package sizes, weights, discounts, or categories to improve visibility – exactly how real catalog data looks.

### 📄 Columns & Data Dictionary:
* `sku_id`: Unique identifier for each product entry (Synthetic Primary Key)
* `name`: Product name as it appears on the app
* `category`: Product category like Fruits, Snacks, Beverages, etc.
* `mrp`: Maximum Retail Price (originally in paise, converted to ₹)
* `discountPercent`: Discount applied on MRP
* `discountedSellingPrice`: Final price after discount (also converted to ₹)
* `availableQuantity`: Units available in inventory
* `weightInGms`: Product weight in grams
* `outOfStock`: Boolean flag indicating stock availability (`TRUE`/`FALSE`)
* `quantity`: Number of units per package (mixed with grams for loose produce)

---

## 🛠️ Project Workflow & Complete SQL Code

### 1. Database & Table Creation
We start by creating a SQL table with appropriate data types:

```sql
DROP TABLE IF EXISTS zepto;

CREATE TABLE zepto (
    sku_id SERIAL PRIMARY KEY,
    category VARCHAR(120),
    name VARCHAR(150) NOT NULL,
    mrp NUMERIC(8,2),
    discountPercent NUMERIC(5,2),
    availableQuantity INTEGER,
    discountedSellingPrice NUMERIC(8,2),
    weightInGms INTEGER,
    outOfStock BOOLEAN,
    quantity INTEGER
);
```

### 2. Data Import
Loaded the CSV dataset into PostgreSQL. If you are executing via terminal CLI, run the following code:

```sql
\copy zepto(category,name,mrp,discountPercent,availableQuantity,discountedSellingPrice,weightInGms,outOfStock,quantity) 
FROM 'data/zepto_v2.csv' 
WITH (FORMAT CSV, HEADER);
```
*Note: If you run into encoding errors (UTF-8 issues), resave your raw CSV file explicitly using the CSV UTF-8 format.*

### 3. 🔍 Data Exploration
Audit queries used to check structural metrics and identify potential data gaps:

```sql
-- Count total records loaded
SELECT COUNT(*) AS total_skus FROM zepto;

-- View a sample of the dataset
SELECT * FROM zepto LIMIT 10;

-- Audit missing/NULL values across core fields
SELECT 
    COUNT(*) - COUNT(sku_id) AS missing_ids,
    COUNT(*) - COUNT(name) AS missing_names,
    COUNT(*) - COUNT(mrp) AS missing_mrp,
    COUNT(*) - COUNT(category) AS missing_categories
FROM zepto;

-- Identify unique business categories
SELECT DISTINCT category FROM zepto ORDER BY category;

-- In-stock vs Out-of-stock product counts
SELECT outOfStock, COUNT(*) AS product_count 
FROM zepto 
GROUP BY outOfStock;
```

### 4. 🧹 Data Cleaning
Queries executed to enforce strict data quality standards:

```sql
-- Remove bad rows where core price indexes are zero or missing
DELETE FROM zepto 
WHERE mrp <= 0 OR discountedSellingPrice <= 0;

-- Convert raw pricing inputs from paise to rupees for clean readability
UPDATE zepto 
SET mrp = mrp / 100.0, 
    discountedSellingPrice = discountedSellingPrice / 100.0;
```

### 5. 📊 Business Insights & Analysis Queries
Strategic commercial queries written to unlock operational value from the clean inventory tables:

```sql
-- Query 1: Top 10 items with the deepest price discounts
SELECT name, category, mrp, discountPercent, discountedSellingPrice
FROM zepto
ORDER BY discountPercent DESC
LIMIT 10;

-- Query 2: Lost Revenue Risk (High-MRP products currently out of stock)
SELECT name, category, mrp, outOfStock
FROM zepto
WHERE outOfStock = TRUE
ORDER BY mrp DESC;

-- Query 3: Financial Forecasting (Potential revenue metrics if current stock sells out)
SELECT 
    category,
    COUNT(sku_id) AS unique_sku_count,
    SUM(availableQuantity) AS total_items_in_stock,
    ROUND(SUM(availableQuantity * discountedSellingPrice), 2) AS potential_revenue_in_rupees
FROM zepto
GROUP BY category
ORDER BY potential_revenue_in_rupees DESC;

-- Query 4: Margin Risk Tracking (Premium items over ₹500 with minimal markdown incentives)
SELECT name, category, mrp, discountPercent, discountedSellingPrice
FROM zepto
WHERE mrp > 500.00 AND discountPercent < 5.00
ORDER BY mrp DESC;

-- Query 5: Promotional Performance (Top 5 categories with highest average discounts)
SELECT 
    category,
    ROUND(AVG(discountPercent), 2) AS avg_discount_percentage,
    ROUND(AVG(mrp - discountedSellingPrice), 2) AS avg_cash_saved_rupees
FROM zepto
GROUP BY category
ORDER BY avg_discount_percentage DESC
LIMIT 5;

-- Query 6: Unit Economics Index (Finding best value-for-money items per gram)
SELECT 
    name, 
    category, 
    weightInGms, 
    discountedSellingPrice,
    ROUND((discountedSellingPrice / NULLIF(weightInGms, 0)), 4) AS price_per_gram
FROM zepto
WHERE weightInGms IS NOT NULL AND weightInGms > 0
ORDER BY price_per_gram ASC
LIMIT 10;

-- Query 7: Assortment Segmentation (Categorizing items into logical weight tiers)
SELECT 
    sku_id,
    name,
    category,
    weightInGms,
    CASE 
        WHEN weightInGms <= 100 THEN 'Low Weight'
        WHEN weightInGms > 100 AND weightInGms <= 500 THEN 'Medium Weight'
        ELSE 'Bulk / Heavy Weight'
    END AS shipping_weight_segment
FROM zepto
ORDER BY weightInGms DESC;

-- Query 8: Logistics Planning (Total operational weight footprint in kgs per category)
SELECT 
    category,
    ROUND(SUM(availableQuantity * weightInGms) / 1000.0, 2) AS total_inventory_weight_kg
FROM zepto
WHERE weightInGms IS NOT NULL
GROUP BY category
ORDER BY total_inventory_weight_kg DESC;
```

---

## ⚙️ How to Use This Project

1. Clone this repository to your local computer.
2. Open **pgAdmin 4** or your preferred PostgreSQL server tool.
3. Run the DDL setup script inside the `1. Database & Table Creation` phase to initialize the table.
4. Import your local `zepto_v2.csv` data using the import wizard or copy command tool script.
5. Run the exploration, cleaning, and insights steps sequentially to generate your data summary reports!

---

## 📂 Repository Directory Structure

Here is how the project files are organized in the GitHub repository:

```text
Zepto_SQL_Data_Analysis/
│
├── data/
│   └── zepto_v2.csv                 <- Raw source dataset containing scraped SKU details
│
├── Zepto_SQL_Data_Analysis.sql      <- Production-ready SQL script with full query sequence
└── README.md                        <- Project documentation, dataset schema, and analytics guide
```

