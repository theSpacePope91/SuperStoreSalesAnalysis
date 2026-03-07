# Super Store Sales Analysis by Michael Covelli
Performance of SuperStore stores through the US between 2014 and 2017  

### [Tableau Viz](https://public.tableau.com/views/SuperStoreSalesAnalysis-MichaelCovelli/OverallPerformanceDashboard?:language=en-US&publish=yes&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)

## Data Cleaning Log  
**Project:** Retail Performance Analysis  
**Dataset:** Superstore Sales (Kaggle)  
**Tool:** MariaDB via phpMyAdmin  
**Date:** March 2026  
**Analyst:** Michael Covelli  
  
1. ### Dataset Overview  
   | Attribute  | Detail  |
   | ---------  | ------- |
   | Source | Superstore Sales (Kaggle) |
   | Original Format | CSV |
   | Imported Format | CSV |
   | Total Rows | 10,134 |
   | Total Columns | 23 |
   | Date Range | January 2014 - December 2017 |
   | Granularity | One row per product per order |  
  
2. ### Data Dictionary  
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
  
3. ### Important Issues and Resolutions  
   | # | Issue  | Resolution  |   Outcome |
   | --- | ---------  | ------- | -------- |
   | 1 | UTF- 8 BOM encoding error in MySQL workbench wizard | Switched to phpMyAdmin import tab | Encoding handled correctly |
   | 2 | Column names not recognized on first import | reimported with "first row as column names" enabled | All column names correctly assigned |
   | 3 | Sales Column out of range (#1264) | Altered Sales column from DECIMAL(8,4) to DECIMAL(10,2) | Import complete without errors |
   | 4 | max_allowed_packet too small for full dataset | Increased max_allowed_packet to 64mB in phpMyAdmin | Full dataset import successful |  
  
4. ### Data Type Corrections  
   | Column | Original Type  | Corrected Type  |   Reason |
   | --- | ---------  | ------- | -------- |
   | Order Date | VARCHAR(10) | DATE | Required for ship date vs order date validation |
   | Ship Date | VARCHAR(10) | DATE | Required for time-based analysis and date comparison |
   | Sales | DECIMAL(8,4) | DECIMAL(10,2) | Original Percision too small for large order values |  
  
   #### Date Conversion Method  
  
   ``` {sql}
   ALTER TABLE sample___superstore ADD COLUMN cleaned_order_date DATE;
   ALTER TABLE sample___superstore ADD COLUMN cleaned_ship_date DATE;
   
   UPDATE sample___superstore
   SET cleaned_order_date = STR_TO_DATE(`Order Date`, '%Y/%m/%d');
   
   UPDATE sample___superstore
   SET cleaned_ship_date = STR_TO_DATE(`Ship Date`, '%Y/%m/%d');
   ```  
   **Format Used:** %c/%e/%Y — handles dates without leading zeros (e.g., 11/8/2016)  
   **Original columns retained for reference and verification**  
    
5. ### Data Verification Checks
     
   5.1 Order ID -> Customer ID integrity  
   **Question:** Does each Order ID map to exactly one Customer ID?  
   **Risk:** An oder belonging to multiple customers would indicate a data integrity issue  
    
   ``` {sql}
   SELECT `Order ID`
   FROM sample__superstore
   GROUP BY `Order ID`
   HAVING COUNT(DISTINCT `Customer ID`) > 1
   ```    
    
   **Result:** 0 violations - every Order ID corresponds to exactly 1 Customer ID  
     
   5.2 Invalid ship dates
     
   **Question:** Are there any orders whose ship dates precede Order Date?  
   **Risk:** A ship date before an order date is logically impossible and would indicate bad data  
    
   ``` {sql}
   SELECT `Order ID`, cleaned_order_date, cleaned_ship_date
   FROM sample__superstore
   WHERE cleaned_ship_date < cleaned_order_date
   ```  
   **Result:** 0 violations - all shipments occur on or after order date   
    
   5.3 White Space Validation
     
   **Question:** Do any text column contain leading or trailing whitespace?  
   **Risk:** Extra white space causes GROUP BY and JOIN mismatches in analysis  
    
   ``` {sql}
   SELECT `City` FROM sample___superstore WHERE `City` != TRIM(`City`);
   SELECT `State` FROM sample___superstore WHERE `State` != TRIM(`State`);
   SELECT `Customer Name` FROM sample___superstore WHERE `Customer Name` != TRIM(`Customer Name`);
   SELECT `Product Name` FROM sample___superstore WHERE `Product Name` != TRIM(`Product Name`);
   ```
   **Result:** 0 violations across all checked text columns.  
    
   5.4 Negative sales values
  
   **Question:** Are there any negative sales values?  
   **Risk:** Negative sales values would skew analysis results in this context  
  
   ``` {sql}
   SELECT COUNT(*) AS negative_sales_count
   FROM sample___superstore
   WHERE Sales < 0
   ```
     
   5.5 Null Value Check  
     
   **Question:** Are there any null values in Sales, Profit or date columns?  
   **Risk:** Null values in critical columns (Sales, Profit, dates) would cause incorrect aggregations  
   Conditional Formatting in Excel to highlight any blank columns  
   **Result:** 0 violations in critical columns  
  
7. ### Notable Data Observations  
      | Observation | Detail | Action Taken |
   | --- | ---------  | ------- | -------- |
   | Discount Values reach 0.80 (80%) | Found in Central region — order Row ID 15 shows 80% discount on a $68.81 item resulting in -$123.86 profit | Retained for analysis — flagged as key finding |
   | Multiple rows per Order ID | Expected behavior — one row per line item purchased | Documented as intentional design, not a data error |
   | Profit can be negative| 1,900 rows have negative profit values | Expected — discounts cause losses, retained for analysis |
  
8. ### Final Dataset Summary  
   | Metric | Value |
   | --- | ---------  |
   | Total Rows | 10134 |
   | Total Orders | 5009 |
   | Total Customers | 793 |
   | Total Products | 1862 |
   | Date Range | January 2014 - December 2017 |
   | Regions | 4 (Central, South, East, West) |
   | Categories | 3 (Furniture, Technology, Office Supplies) |
   | Sub-Categories | 17 |
   | Validation Checks Passed | 5/5 |
   | Import Issues resolved | 4/4 |  
  
9. ### Known Limitations  
   - Dataset covers 2014–2017 only — findings may not reflect current business conditions
   - Geographic data limited to United States only
   - Discount values as high as 80% may reflect aggressive pricing policies or potential data entry errors — retained for analysis purposes
   - No customer demographic data available for segmentation beyond Segment field
