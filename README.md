# E-Commerce Sales Analysis — Final Project

**Author:** Sofía Guaragna
**Course:** Proyecto Final — EscueladeDatosVivos.ai
---

## Table of Contents

* [Project Overview](#project-overview)
* [Business Questions and Hypotheses](#business-questions-and-hypotheses)
* [KPIs Implemented](#kpis-implemented)
* [Bronze Layer — Source Data](#bronze-layer--source-data)
* [ER Diagram](#er-diagram)
* [Silver Layer — SQL Transformations](#silver-layer--sql-transformations)
* [Metrics Plan](#metrics-plan)
* [Power BI — ETL and Modeling](#power-bi--etl-and-modeling)
* [DAX Measures](#dax-measures)
* [Validation](#validation)
* [Draft Findings](#draft-findings)
* [Reproducibility Steps](#reproducibility-steps)
* [Deliverables Checklist](#deliverables-checklist)
* [Next Steps](#next-steps)

---

# Project Overview

This repository contains an end-to-end analytical project following the Bronze → Silver → Power BI structure required by the course.

The main objectives were:

* Building a curated analytical dataset from a public BigQuery dataset.
* Defining business rules, KPIs, and a metric plan.
* Designing a Power BI dashboard to drive insights for decision-making.
* Documenting the complete process in a reproducible manner.

---

# Business Questions and Hypotheses

The analysis focuses on the following hypotheses and guiding questions:

* How do sales evolve over time?
* What customer segments exist, and how do they behave?
* Are there recurring customers, and how frequently do they purchase?
* How efficient is the order and fulfillment process?
* Are sales impacted by special dates or seasonal patterns?
* What are the top-selling products and categories?
* What is the average product price and order value?
* How are customers distributed geographically?

---

# KPIs Implemented

* Total Orders
* Total Products Sold
* Average Ticket (Average Order Value)
* Returning Customers
* Percentage of Cancelled Orders

---

# Bronze Layer — Source Data

The Bronze layer corresponds to the raw public dataset:

**BigQuery public dataset:**
`bigquery-public-data.thelook_ecommerce`

Tables used:

* users
* orders
* order_items
* products
* inventory_items

These tables were used without modification and served as the source to generate the Silver layer.

---

# ER Diagram

The model includes the relationships defined between the Bronze tables. The following was one of the firsts versions of the diagram whit of all the data base used to have a general understanding
(raw)
https://dbdiagram.io/d/E-Commerce-raw-691a25666735e111700f8739
Then:
https://dbdiagram.io/d/E-Commerce-bronce-691a25666735e111700f8739

---

# Silver Layer — SQL Transformations

The following Code was used to create the silver layer: 
## SQL CODIFICATION TO CREATE SILVER LAYER
```
CREATE Schema If not exists `integradorsg.silver`
--------------------fact------------------------------
CREATE OR REPLACE TABLE `integradorsg.silver.fact` AS
SELECT order_id, user_id,status, created_at,
  EXTRACT(YEAR FROM created_at) AS year,
  EXTRACT(MONTH FROM created_at) AS month,
  EXTRACT(DAY FROM created_at) AS day
FROM `bigquery-public-data.thelook_ecommerce.orders` 
------------------dim1------------------------------------
CREATE OR REPLACE TABLE `integradorsg.silver.dim1` AS
SELECT id as User_id, gender, age, country, latitude, longitude
FROM `bigquery-public-data.thelook_ecommerce.users` 
------------------dim2------------------------------------
CREATE OR REPLACE TABLE `integradorsg.silver.dim2` AS
SELECT order_id, user_id, product_id sale_price 
FROM `bigquery-public-data.thelook_ecommerce.order_items` 
-------------------dim3------------------------------------
Select id as product_id, category, brand, department
From `bigquery-public-data.thelook_ecommerce.products` 
```


# Metrics Plan

| KPI                 | Business Definition            | Visual              |
| ------------------- | ------------------------------ | ------------------- |
| Total Orders        | Number of unique orders        | Card                |
| Total Products Sold | Items sold                     | Card / bar chart    |
| Average Ticket      | Average order value            | Card                |
| Returning Customers | Users with more than one order | Card / cohort table |
| % Cancelled Orders  | Share of orders cancelled      | Card                |

---

# Power BI — ETL and Modeling

## Import

* Connect Power BI Desktop to Google BigQuery.
* Load the Silver tables: fact, dim1, dim2, dim3.

## Transformations

* Convert `created_at` to DateTime.
* Hide technical fields not required in the report.

## Relationships

* fact.order_id → dim2.order_id
* fact.user_id → dim1.user_id
* dim2.product_id → dim3.product_id

## Suggested Report Pages

1. Overall KPIs and trends
2. Customer segmentation and behavior
3. Product performance
4. Order efficiency and cancellations
5. Geographic distribution

---

# DAX Measures

```dax
Total Orders = DISTINCTCOUNT(fact[order_id])

Total Products Sold = COUNTROWS(dim2)

Total Sales = SUM(dim2[sale_price])

Average Ticket =
DIVIDE([Total Sales], [Total Orders], 0)

Returning Customers =
VAR OrdersPerUser =
    SUMMARIZE(
        fact,
        fact[user_id],
        "OrdersCount",
            COUNTX(
                FILTER(fact, fact[user_id] = EARLIER(fact[user_id])),
                fact[order_id]
            )
    )
RETURN
    COUNTROWS(FILTER(OrdersPerUser, [OrdersCount] > 1))

Pct Cancelled =
DIVIDE(
    CALCULATE(DISTINCTCOUNT(fact[order_id]), fact[status] = "cancelled"),
    [Total Orders],
    0
)
```

---

# Validation

* Row counts between Bronze and Silver validated.
* Spot-checks for consistency of `order_id` across fact and dimensions.
* Validation of parsed dates.
* Latitude and longitude ranges verified.

---

# Draft Findings

Replace the placeholders with your actual dashboard results.

* Sales seasonality shows increased activity in **[Month X]** and **[Month Y]**.
* The top-performing product or category is **[Product/Category]**.
* Average ticket is **$[Value]**.
* Percentage of returning customers is **[Value%]**.
* Cancellation rate is **[Value%]**.
* Most customers are located in **[Region/City]**.

---

# Reproducibility Steps

## BigQuery

1. Create dataset `integradorsg.silver`.
2. Run the SQL scripts listed above to generate all Silver tables.

## Power BI

1. Connect to BigQuery and load Silver tables.
2. Apply transformations as described.
3. Create the relationships.
4. Add the DAX measures.
5. Build visualization pages.
6. Publish the report to Power BI Service.

---

# Deliverables Checklist

* [x] Bronze layer defined
* [x] Silver SQL transformations created
* [x] Power BI model built
* [x] KPIs defined and implemented
* [x] README documented
* [ ] Screenshots added
* [ ] Power BI service link inserted

---

# Next Steps

* Add screenshots from Power BI.
* Add the published Power BI link.
* Replace placeholder values in the Findings section.

---

**LISTO.**
Este archivo está 100% listo para copiar y pegar como `README.md` en GitHub. Si querés, te preparo una **versión aún más formal** o una **versión con tabla de contenidos automática generada por GitHub**.
