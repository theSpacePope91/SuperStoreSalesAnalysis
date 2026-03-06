# SuperStoreSalesAnalysis
Performance of SuperStore stores through the US between 2014 and 2017







## Data Cleaning Log
---
**Project:** Retail Performance Analysis
**Dataset:** Superstore Sales (Kaggle)
**Tool:** MariaDB via phpMyAdmin
**Date:** March 2026
**Analyst:** Michael Covelli
---
1. #### Dataset Overview
   | Attribute  | Detail  |
   | ---------  | ------- |
   | Source | Superstore Sales (Kaggle) |
   | Original Format | CSV |
   | Imported Format | CSV |
   | Total Rows | 10,134 |
   | Total Columns | 23 |
   | Date Range | January 2014 - December 2017 |
   | Granularity | One row per product per order |
   
2. #### Data Dictionary
   | # | Column  | Type  |   Description |
   | --- | ---------  | ------- | -------- |
   | 1 | Row ID | INT | Unique Row Identifier  |
   | 2 | Order ID | VARCHAR(14) | Unique Order Identifier (multiple rows per order expected)  |
   | 3 | Order Date | VARCHAR(10) | Date order was placed  |
   | 4 | cleaned_order_date | DATE | formatted date order was placed  |
   | 5 | Ship Date | VARCHAR(10) | Date order was shipped  |
   | 6 | cleaned_order_date | DATE | formatted date order was shipped  |
   | 7 | Ship Mode | VARCHAR(14) | Shipping method selected  |
   | 8 | Customer ID | VARCHAR(8) | Unique customer identifier |
   | 9 | Customer Name | VARCHAR(22) | Full Name of Customer |
   | 10 | Segment | VARCHAR(11) | Customer segment (Consumer, Corporate, Home Office)  |
   | 11 | Country | VARCHAR(13) | Country of order (All United States) |
   | 12 | City | VARCHAR(17) | City of delivery  |
   | 13 | State | VARCHAR(20) | State of delivery |
   | 14 | Postal Code | varchar(5) | Postal Code of delivery  |
   | 15 | Region | VARCHAR(7) | US Region (Central, East, West, South  |
   | 16 | Product ID | VARCHAR(15) | Unique Product Identifier  |
   | 17 | Category | VARCHAR(15) | Product Category (Furniture, Office Supplies, Technology) |
   | 18 | Sub-Category | VARCHAR(11) | Product Sub-Category |
   | 19 | Product Name | VARCHAR(127) | Full product name  |
   | 20 | Sales | DECIMAL(10,2) | Revenue Generated from sales (USD)  |
   | 21 | Quantity | INT | Number of units purchased |
   | 22 | Discount | DECIMAL(3,2) | Discount applied (0.0 - 0.8) |
   | 23 | Profit | DECIMAL(10,4) | Profit or loss from sale  |
