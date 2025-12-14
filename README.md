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
* [Power BI Modeling](#power-bi-modeling)
* [DAX Measures](#dax-measures)
* [Power BI PROCEDURE](#Power-BI-PROCEDURE)
* [Power BI final link](#Power-BI-final-link)


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
FINAL DIAGRAM
<img width="972" height="732" alt="image" src="https://github.com/user-attachments/assets/2638c1ce-9e8a-4943-aa22-8e86346732c7" />

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

| KPI                 | Business Definition            |
| ------------------- | ------------------------------ | 
| Total Orders        | Number of unique orders        | 
| Total Products Sold | Items sold                     | 
| Average Ticket      | Average order value            | 
| Returning Customers | Users with more than one order | 
| % Cancelled Orders  | Share of orders cancelled      | 

---

# Power BI Modeling

## Import

* Connect Power BI Desktop to Google BigQuery.
* Load the Silver tables: fact, dim1, dim2, dim3.

## Transformations

* Convert `created_at` to DateTime.
*
```
  Dim_Calendar = ADDCOLUMNS (
CALENDAR ( DATE( YEAR ( MIN ( 'fact'[created_at])), 01, 01), DATE( YEAR( MAX( 'fact'[created_at]) ), 12, 31 ) ),
"FechaSK", FORMAT ( [Date], "YYYYMMDD" ),
"#Año", YEAR ( [Date] ),
"#Trimestre", QUARTER ( [Date] ),
"#Mes", MONTH ( [Date] ),
"#Día", DAY ( [Date] ),
"Trimestre", "T" & FORMAT ( [Date], "Q" ),
"Mes", FORMAT ( [Date], "MMMM" ),
"MesCorto", FORMAT ( [Date], "MMM" ),
"#DíaSemana", WEEKDAY ( [Date],2 ),
"#SemanaAño", WEEKNUM ( [Date],2 ),
"CierreSemana", ( [Date] + 7 - WEEKDAY( [Date],2 ) ),
"Día", FORMAT ( [Date], "DDDD" ),
"DíaCorto", FORMAT ( [Date], "DDD" ),
"AñoTrimestre", FORMAT ( [Date], "YYYY" ) & "/T" & FORMAT ( [Date], "Q" ),
"Año#Mes", FORMAT ( [Date], "YYYY/MM" ),
"AñoMesCorto", FORMAT ( [Date], "YYYY/mmm" ),
"InicioMes", EOMONTH( [Date], -1) + 1,
"FinMes", EOMONTH( [Date], 0) )
```

# DAX Measures

```DAX MEASURES
% Cancelados = 
DIVIDE(
    [Órdenes Canceladas],
    [Total Órdenes],
    0
)
Órdenes Canceladas = 
CALCULATE(
    COUNTROWS(fact),
    fact[status] = "Cancelled"
)
Ticket Promedio = [Total ventas]/[Total Órdenes]

Total Órdenes = DISTINCTCOUNT(fact[order_id])

Total ventas = SUM(dim2[sale_price])

Órdenes por Cliente = 
CALCULATE(
    DISTINCTCOUNT('fact'[order_id]),
    ALLEXCEPT('fact', 'fact'[user_id])
)
Clientes Totales = DISTINCTCOUNT('fact'[user_id])
Clientes Recurrentes (%) = 
DIVIDE(
    [Clientes Recurrentes],
    [Clientes Totales]
)
Clientes Recurrentes = 
CALCULATE(
    DISTINCTCOUNT('fact'[user_id]),
    FILTER(
        VALUES('fact'[user_id]),
        CALCULATE(DISTINCTCOUNT('fact'[order_id])) > 1
    )
)


```

## Power BI PROCEDURE

1. Connect to BigQuery and load Silver tables.
2. Apply transformations as described.
3. Create the relationships.
4. Add the DAX measures.
5. Build visualization pages. Add filters and segments. Edit design
6. Publish the report to Power BI Service.

---
## Power BI final link and screens
https://drive.google.com/file/d/1oNxqDkRyUOiSOLJigAeq6tRCNc2qd_dG/view?usp=sharing

## First page - Index

<img width="1154" height="653" alt="image" src="https://github.com/user-attachments/assets/0cfbea90-28c9-4049-a605-f079b5fbed5c" />

## Second page - Sales analysis and metrics
<img width="1161" height="697" alt="image" src="https://github.com/user-attachments/assets/09fa3c47-363e-4073-a09c-818e07e268e3" />

## Third page - Segmentation and users
<img width="1170" height="656" alt="image" src="https://github.com/user-attachments/assets/82c82eff-9ad2-4763-9beb-3334bf429dc8" />

## Forth page - Users map distribution
<img width="1136" height="620" alt="image" src="https://github.com/user-attachments/assets/f6507a46-ec70-41fb-9c53-217813040c23" />




