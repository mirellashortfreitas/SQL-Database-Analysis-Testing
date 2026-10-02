# SQL-Database-Analysis-Testing

# SQL Database Analysis & Testing

## 📌 Project Overview

This project focuses on **SQL database querying, data analysis, and query validation** using a relational e-commerce database.

The project was developed as an academic database assignment and demonstrates the ability to retrieve, filter, join, aggregate, format, and validate data using SQL.

The database contains information related to:

- Shoppers
- Orders
- Products
- Sellers
- Product categories
- Ordered products

The project includes four main SQL queries designed to answer different business and data-analysis questions, together with a set of validation queries used to test the correctness of the results.

**Project type:** University Database / SQL Project  
**Role:** SQL Development, Data Analysis & Query Testing  
**Technology:** SQL  
**Database concepts:** Relational Databases, Joins, Aggregation, Filtering, Subqueries & Data Validation  
**Domain:** E-commerce / Retail Data

---

## 🎯 Project Objectives

The main objective of the project was to develop SQL queries capable of retrieving meaningful information from a relational database while ensuring that the results were correctly filtered, calculated, formatted, and validated.

The project focused on:

- Retrieving and filtering shopper information
- Analysing shopper order history
- Connecting products with sellers and orders
- Calculating quantities and total sales
- Comparing product performance against category averages
- Handling missing data
- Formatting dates and monetary values
- Testing SQL query results
- Validating filtering and aggregation logic

The project also provided an opportunity to apply a **quality-focused approach** to SQL development by creating separate validation queries for the main requirements.

---

# 🔎 Database Analysis

The project contains four primary SQL queries, each addressing a different data requirement.

The queries demonstrate the use of relational database concepts including:

- `SELECT`
- `WHERE`
- `ORDER BY`
- `INNER JOIN`
- `LEFT JOIN`
- `GROUP BY`
- `HAVING`
- `COUNT`
- `SUM`
- `AVG`
- `IFNULL`
- `PRINTF`
- `STRFTIME`
- Subqueries
- Parameters

---

# 👤 Query A — Shopper Information Analysis

The first query retrieves shopper information and presents it in a clear and formatted way.

The query displays:

- Shopper first name
- Shopper surname
- Email address
- Gender
- Date joined
- Current age

Missing gender values are replaced with **"Not known"**, while the join date is formatted as `DD-MM-YYYY`.

The query filters shoppers based on two conditions:

- Joined on or after **1 January 2020**
- OR gender is **Female (`F`)**

The results are then ordered by gender and age, with the oldest shoppers appearing first.

### SQL concepts demonstrated

```text
SELECT
WHERE
OR
IFNULL()
STRFTIME()
Calculated fields
ORDER BY
```

### Example

```sql
SELECT
    shopper_first_name AS "Shopper First Name",
    shopper_surname AS "Shopper Surname",
    shopper_email_address AS "Email Address",
    IFNULL(gender, 'Not known') AS "Gender",
    STRFTIME('%d-%m-%Y', date_joined) AS "Date Joined",
    (STRFTIME('%Y', 'now') - STRFTIME('%Y', date_of_birth)) -
    (STRFTIME('%m-%d', 'now') < STRFTIME('%m-%d', date_of_birth))
    AS "Current Age"
FROM shoppers
WHERE
    date_joined >= '2020-01-01'
    OR gender = 'F'
ORDER BY
    gender ASC,
    "Current Age" DESC;
```

This query demonstrates how SQL can combine raw database fields with calculated and formatted values to create a more useful dataset for analysis.

---

# 🛒 Query B — Shopper Order History

The second query retrieves the order history for a **specific shopper** using a parameter:

```text
:shopper_id
```

The query connects several related tables:

```text
shoppers
    ↓
shopper_orders
    ↓
ordered_products
    ↓
products
    ↓
sellers
```

This allows the query to display:

- Shopper name
- Order ID
- Order date
- Product description
- Seller name
- Quantity ordered
- Product price
- Order status

The order date is formatted as `DD-MM-YYYY`, while prices are formatted to two decimal places with the `£` symbol.

Results are filtered using the selected shopper ID and sorted from newest to oldest order.

### SQL concepts demonstrated

```text
INNER JOIN
Parameterized queries
WHERE
ORDER BY
Date formatting
Currency formatting
Table aliases
```

### Example

```sql
SELECT 
    s.shopper_first_name AS "Shopper First Name",
    s.shopper_surname AS "Shopper Surname",
    so.order_id AS "Order ID",
    STRFTIME('%d-%m-%Y', so.order_date) AS "Order Date",
    p.product_description AS "Product Description",
    se.seller_name AS "Seller Name",
    op.quantity AS "Quantity Ordered",
    '£' || PRINTF('%.2f', op.price) AS "Price",
    op.ordered_product_status AS "Order Status"
FROM shoppers AS s
INNER JOIN shopper_orders AS so
    ON s.shopper_id = so.shopper_id
INNER JOIN ordered_products AS op
    ON so.order_id = op.order_id
INNER JOIN products AS p
    ON op.product_id = p.product_id
INNER JOIN sellers AS se
    ON op.seller_id = se.seller_id
WHERE s.shopper_id = :shopper_id
ORDER BY so.order_date DESC;
```

The use of multiple joins demonstrates how information stored across related tables can be combined into a single meaningful result.

---

# 💰 Query C — Seller & Product Sales Analysis

The third query focuses on seller and product performance.

It retrieves:

- Seller account reference
- Seller name
- Product code
- Product description
- Number of orders
- Total quantity sold
- Total sales

A `LEFT JOIN` is used when connecting products with ordered products. This allows products with no sales to remain in the results.

Missing aggregate values are replaced with `0`.

Total sales are calculated using:

```text
quantity × price
```

and formatted as a monetary value with two decimal places.

The results are sorted by total quantity sold, from lowest to highest.

### SQL concepts demonstrated

```text
JOIN
LEFT JOIN
COUNT(DISTINCT)
SUM()
IFNULL()
GROUP BY
Calculated values
Currency formatting
ORDER BY
```

### Example

```sql
SELECT 
    s.seller_account_ref AS "Seller Account Ref",
    s.seller_name AS "Seller Name",
    p.product_code AS "Product Code",
    p.product_description AS "Product Description",
    IFNULL(COUNT(DISTINCT op.order_id), 0) AS "No. of Orders",
    IFNULL(SUM(op.quantity), 0) AS "Total Quantity Sold",
    '£' || PRINTF(
        '%.2f',
        IFNULL(SUM(op.quantity * op.price), 0)
    ) AS "Total Sales"
FROM product_sellers ps
JOIN sellers s
    ON ps.seller_id = s.seller_id
JOIN products p
    ON ps.product_id = p.product_id
LEFT JOIN ordered_products op
    ON ps.product_id = op.product_id
    AND ps.seller_id = op.seller_id
GROUP BY
    s.seller_account_ref,
    s.seller_name,
    p.product_code,
    p.product_description
ORDER BY
    "Total Quantity Sold" ASC;
```

This query demonstrates the use of SQL aggregation to transform transactional data into useful sales information.

---

# 📊 Query D — Product vs Category Performance

The fourth query compares the average quantity sold for individual products against the average quantity sold for their respective categories.

The query:

- Joins products with categories
- Excludes cancelled orders
- Includes products with no sales
- Calculates product-level averages
- Calculates category-level averages
- Compares the two averages
- Returns products whose average quantity is below their category average

The results are ordered by category and product description.

### SQL concepts demonstrated

```text
JOIN
LEFT JOIN
AVG()
IFNULL()
GROUP BY
HAVING
Subqueries
Conditional comparison
ORDER BY
```

### Example

```sql
SELECT
    c.category_description AS "Category Description",
    p.product_code AS "Product Code",
    p.product_description AS "Product Description",

    PRINTF(
        '%.2f',
        IFNULL(AVG(op.quantity), 0)
    ) AS "Product Average Quantity",

    PRINTF(
        '%.2f',
        (
            SELECT
                IFNULL(AVG(op2.quantity), 0)
            FROM products p2
            LEFT JOIN ordered_products op2
                ON p2.product_id = op2.product_id
                AND op2.ordered_product_status <> 'Cancelled'
            WHERE p2.category_id = p.category_id
        )
    ) AS "Category Average Quantity"

FROM products p

JOIN categories c
    ON p.category_id = c.category_id

LEFT JOIN ordered_products op
    ON p.product_id = op.product_id
    AND op.ordered_product_status <> 'Cancelled'

GROUP BY
    c.category_description,
    p.category_id,
    p.product_id,
    p.product_code,
    p.product_description

HAVING
    IFNULL(AVG(op.quantity), 0) < (
        SELECT
            IFNULL(AVG(op2.quantity), 0)
        FROM products p2
        LEFT JOIN ordered_products op2
            ON p2.product_id = op2.product_id
            AND op2.ordered_product_status <> 'Cancelled'
        WHERE p2.category_id = p.category_id
    )

ORDER BY
    c.category_description,
    p.product_description;
```

This was one of the more advanced queries in the project because it required comparing an aggregated product-level result against another aggregated value calculated at category level.

---

# 🧪 Testing & Validation

An important part of the project was the creation of separate SQL tests to validate the main queries.

Rather than relying only on the output of the queries, validation procedures were created to check whether the expected filtering, calculations, formatting, and ordering were working correctly.

The report contains four validation tests.

---

## Test A — Filter Validation

The first test validates the filtering logic used in Query A.

It checks:

- Number of shoppers who joined from 1 January 2020 onwards
- Number of shoppers with gender `F`
- Number of shoppers satisfying either condition

The expected counts recorded in the report were:

```text
Joined after/on 2020-01-01: 5
Female shoppers:             8
Total matching filter:      12
```

This test helps verify that the `OR` condition in the main query behaves as intended.

---

## Test B — Parameter Validation

The second test validates the shopper ID parameter used by Query B.

The test verifies that:

- The shopper exists in the database
- The returned shopper ID matches the selected ID
- The shopper's first name and surname correspond to that ID

The test was demonstrated using shopper ID:

```text
10000
```

The main query was also tested with another parameter:

```text
10019
```

This demonstrates validation of parameterised filtering rather than relying on a single hard-coded scenario. 

---

## Test C — Sales Calculation Validation

The third test validates the calculation of total sales.

The test checks:

```text
SUM(quantity * price)
```

It also compares the raw calculated value with the formatted monetary value.

The result is formatted using:

```text
£
```

and two decimal places.

The validation query returns the first five records for comparison.

### Example

```sql
SELECT
    seller_id,
    product_id,
    SUM(quantity * price) AS raw_total_sales,
    '£' || PRINTF(
        '%.2f',
        SUM(quantity * price)
    ) AS formatted_total_sales
FROM ordered_products
GROUP BY seller_id, product_id
LIMIT 5;
```

This demonstrates an understanding that calculated values should be independently checked rather than assumed to be correct.

---

## Test D — Average & Sorting Validation

The fourth test validates Query D.

It checks:

- Product average quantity
- Category and product ordering
- Exclusion of cancelled orders
- The structure of the aggregated results

The validation query returns the first ten records for comparison.

This provides an additional layer of confidence that the final query is returning the intended dataset.

---

# 🔄 SQL Development & Validation Process

The overall workflow followed in the project can be represented as:

```text
Database Requirements
        ↓
Understand Relationships
        ↓
Write SQL Query
        ↓
Join Required Tables
        ↓
Filter & Transform Data
        ↓
Aggregate Results
        ↓
Format Output
        ↓
Validate Query Results
        ↓
Review & Confirm Results
```

This approach demonstrates that SQL development is not only about writing queries, but also about understanding requirements and validating whether the resulting data is correct.

---

# 🧩 QA & Software Testing Perspective

Although this project focuses on SQL and database analysis, it also demonstrates several skills relevant to **Quality Assurance and Software Testing**.

The validation procedures required the identification of expected behaviour and the creation of SQL checks to verify that behaviour.

### QA-related areas demonstrated

- Requirements interpretation
- Data validation
- Query validation
- Functional testing
- Boundary and filtering considerations
- Parameter testing
- Calculation validation
- Data accuracy checks
- Result comparison
- Handling missing values
- Handling cancelled records
- Verification of sorting
- Verification of formatting
- Test scenario development

For example, Query C calculates total sales using `SUM(quantity * price)`, and a separate validation query independently checks the calculation.

Similarly, Query D contains specific logic for excluding cancelled orders, and the validation process checks the resulting dataset and ordering.

This demonstrates a **quality-focused approach to database development**, where query results are tested against expected behaviour.

---

# 🛠️ Skills Demonstrated

## SQL

- SQL querying
- `SELECT`
- `WHERE`
- `ORDER BY`
- `GROUP BY`
- `HAVING`
- `INNER JOIN`
- `LEFT JOIN`
- `COUNT()`
- `COUNT(DISTINCT)`
- `SUM()`
- `AVG()`
- `IFNULL()`
- `PRINTF()`
- `STRFTIME()`
- Subqueries
- Parameterised queries
- Calculated fields
- Data filtering
- Data aggregation
- Data formatting

## Database

- Relational database concepts
- Table relationships
- Primary/foreign key relationships
- Multi-table queries
- Transactional data analysis
- Aggregated data
- Handling missing data
- Handling cancelled transactions

## QA / Testing

- Requirements analysis
- Test scenario creation
- SQL validation
- Data accuracy testing
- Functional validation
- Parameter validation
- Calculation verification
- Filtering verification
- Sorting verification
- Output formatting verification
- Edge-case awareness
- Result comparison

## Data Analysis

- Shopper analysis
- Order analysis
- Product performance analysis
- Seller performance analysis
- Sales calculations
- Average calculations
- Category-level comparisons
- Data interpretation

---

# 📚 Key SQL Techniques

The project demonstrates several SQL techniques that are particularly relevant to database and QA roles.

### Joining relational data

Multiple tables were joined to create meaningful datasets rather than analysing each table independently.

### Aggregation

Functions such as:

```sql
COUNT()
SUM()
AVG()
```

were used to transform transactional records into analytical results.

### Handling missing values

`IFNULL()` was used to ensure missing values could be represented appropriately, including:

```text
Not known
```

for missing gender information and:

```text
0
```

for missing sales/quantity values. 

### Data formatting

The project also demonstrates formatting of:

- Dates
- Currency
- Decimal values
- Display labels

This improves the readability and usability of query results.

---

# 🎓 Academic Project

**Project type:** University Database / SQL Project

The work demonstrates practical application of SQL database concepts through a series of data retrieval, analysis, and validation requirements.

The project documentation contains the complete SQL queries and testing procedures.

---

# 📄 Full Project Documentation

The complete academic report is available in this repository.

📄 **[View the Full Database Report](./documentation/Data-Base-Report.pdf)**

The full report contains:

- SQL queries
- Query explanations
- Database analysis
- Testing procedures
- Validation queries
- Query results and verification

> **Note:** Rename the uploaded PDF in your repository to `Data-Base-Report.pdf`, or update the link above to match the actual filename.

---

# 📁 Suggested Repository Structure

```text
SQL-Database-Analysis/
│
├── README.md
│
├── documentation/
│   └── Data-Base-Report.pdf
│
└── sql/
    ├── query-a-shopper-analysis.sql
    ├── query-b-order-history.sql
    ├── query-c-sales-analysis.sql
    ├── query-d-product-category-analysis.sql
    │
    └── tests/
        ├── test-a-filter-validation.sql
        ├── test-b-parameter-validation.sql
        ├── test-c-sales-validation.sql
        └── test-d-average-validation.sql
```

Organising the repository this way makes it easier for a recruiter or hiring manager to understand the project without having to read the entire academic report first.

---


# 💼 Portfolio Skills Summary

This project demonstrates practical experience in:

```text
SQL
Database Analysis
Data Validation
Relational Databases
Data Aggregation
Query Development
Functional Testing
QA Mindset
Requirements Analysis
Test Scenario Design
Data Accuracy
```

The combination of **SQL development and query validation** demonstrates an understanding of both retrieving data and checking whether the retrieved data satisfies the intended requirements.

---

## Disclaimer

This project was developed as part of a university academic assignment.

The database and queries are used for educational purposes and demonstrate SQL, database analysis, and validation techniques.
