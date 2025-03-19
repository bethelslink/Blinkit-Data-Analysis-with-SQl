# Blinkit Data Analysis with SQL

## Table of Contents  
- [Project Overview](#project-overview)  
- [Dataset](#dataset)  
- [Data Cleaning](#data-cleaning)  
- [Key Performance Indicators (KPIs)](#key-performance-indicators-kpis)  
- [Sales Analysis](#sales-analysis)  
  - [Total Sales by Fat Content](#total-sales-by-fat-content)  
  - [Total Sales by Item Type](#total-sales-by-item-type)  
  - [Fat Content by Outlet for Total Sales](#fat-content-by-outlet-for-total-sales)  
  - [Total Sales by Outlet Establishment Year](#total-sales-by-outlet-establishment-year)  
  - [Percentage of Sales by Outlet Size](#percentage-of-sales-by-outlet-size)  
  - [Sales by Outlet Location](#sales-by-outlet-location)  
  - [All Metrics by Outlet Type](#all-metrics-by-outlet-type)  
- [Results and Insights](#results-and-insights)  
- [Conclusion](#conclusion)  
    

## Project Overview  
The **Blinkit Data Analysis** project focuses on analyzing sales data to uncover key insights related to item fat content, outlet locations, sales distribution, and business performance. This project involves **data cleaning, transformation, and visualization** using **SQL**.  

## How to Load the Data 
To analyze the BlinkIT Grocery Data using PostgreSQL in pgAdmin, follow these steps:

 Open pgAdmin
Launch pgAdmin on your system.
Connect to your PostgreSQL server.
Create a New Database
In the pgAdmin browser, right-click on Databases → Create → Database.
Name it blinkit_db and click Save.
Create a Table for the Data
Before importing the CSV file, create a table that matches the dataset structure.
```sql
Copy code
CREATE TABLE blinkit_data (
    Item_Identifier VARCHAR(50),
    Item_Weight DECIMAL(10,2),
    Item_Fat_Content VARCHAR(20),
    Item_Visibility DECIMAL(10,4),
    Item_Type VARCHAR(50),
    Item_MRP DECIMAL(10,2),
    Outlet_Identifier VARCHAR(20),
    Outlet_Establishment_Year INT,
    Outlet_Size VARCHAR(20),
    Outlet_Location_Type VARCHAR(20),
    Outlet_Type VARCHAR(50),
    Total_Sales DECIMAL(10,2),
    Rating DECIMAL(10,2)
);
```
This creates a table with the same column names as in your CSV file.
 Import the CSV File into pgAdmin
Using pgAdmin UI
Right-click the blinkit_data table → Import/Export Data.
Choose Import.
Select the CSV file (BlinkIT Grocery Data.csv).
Set Delimiter to , (comma).
Enable Header.
Click OK to import the data.

## Dataset  
The dataset contains sales records of various items, including details like:  
- **Item Type**  
- **Item Fat Content**  
- **Outlet Location Type**  
- **Total Sales**  
- **Ratings**  
- **Outlet Establishment Year**  
- **Outlet Size**  

## Data Cleaning  
Data cleaning ensures accuracy and consistency. The **Item_Fat_Content** field had inconsistent values like `"LF", "low fat"`, which were standardized to `"Low Fat"`, and `"reg"` was changed to `"Regular"`.  

```sql
UPDATE blinkit_data  
SET Item_Fat_Content =   
    CASE  
        WHEN Item_Fat_Content IN ('LF', 'low fat') THEN 'Low Fat'  
        WHEN Item_Fat_Content = 'reg' THEN 'Regular'  
        ELSE Item_Fat_Content  
    END;
```
## Explanation:
Purpose: Standardizes different labels for the same fat content category.
How it Works:
"LF" and "low fat" are converted to "Low Fat".
"reg" is converted to "Regular".
Other values remain unchanged.
To verify changes:

```sql
Copy code
SELECT DISTINCT Item_Fat_Content FROM blinkit_data;
```
![Data Cleaning](data%20cleaning.png)


Purpose: Checks if the cleaning was applied correctly.

## Key Performance Indicators (KPIs)
## Total Sales
```sql
Copy code
SELECT CAST(SUM(Total_Sales) / 1000000.0 AS DECIMAL(10,2)) AS Total_Sales_Million  
FROM blinkit_data;
```
![TOTAL SALES](TOTAL%20SALES.png)

## Explanation:
Purpose: Calculates total sales in millions for easier interpretation.
How it Works:
SUM(Total_Sales): Computes the total sales.
CAST(... AS DECIMAL(10,2)): Converts the result to two decimal places.
/ 1000000.0: Converts to millions.

## Average Sales
```sql
Copy code
SELECT CAST(AVG(Total_Sales) AS INT) AS Avg_Sales  
FROM blinkit_data;
```
![AVERAGE SALES](AVERAGE%20SALES.png)

## Explanation:
Purpose: 
Computes the average sales per transaction.
How it Works:

AVG(Total_Sales): Calculates the average total sales.
CAST(... AS INT): Rounds the result to an integer.

## Number of Items Sold
```sql
Copy code
SELECT COUNT(*) AS No_of_Orders  
FROM blinkit_data;
```
![NO OF ITEMS](NO%20OF%20ITEMS.png)

## Explanation:
Purpose: Counts the total number of sales records (orders).
How it Works:
COUNT(*): Counts all rows in the dataset.

## Average Rating
```sql
Copy code
SELECT CAST(AVG(Rating) AS DECIMAL(10,1)) AS Avg_Rating  
FROM blinkit_data;
```
![AVG RATING](AVG%20RATING.png)

## Explanation:
Purpose: Computes the average rating of products.
How it Works:
AVG(Rating): Finds the average rating.
CAST(... AS DECIMAL(10,1)): Formats to one decimal place.

## Sales Analysis
## Total Sales by Fat Content
```sql
Copy code
SELECT Item_Fat_Content, CAST(SUM(Total_Sales) AS DECIMAL(10,2)) AS Total_Sales  
FROM blinkit_data  
GROUP BY Item_Fat_Content;
```
![Total Sales by Fat Content](Total%20Sales%20by%20Fat%20Content.png)

## Explanation:
Purpose: Determines the total sales for each fat content category.
How it Works:
GROUP BY Item_Fat_Content: Groups by fat content category.
SUM(Total_Sales): Computes total sales for each category.

## Total Sales by Item Type
```sql
Copy code
SELECT Item_Type, CAST(SUM(Total_Sales) AS DECIMAL(10,2)) AS Total_Sales  
FROM blinkit_data  
GROUP BY Item_Type  
ORDER BY Total_Sales DESC;
```
![Total Sales by Item Type](Total%20Sales%20by%20Item%20Type.png)

## Explanation:
Purpose: Identifies the best-selling item types.
How it Works:
GROUP BY Item_Type: Groups sales by item type.
ORDER BY Total_Sales DESC: Sorts results in descending order.

## Fat Content by Outlet for Total Sales
```sql
Copy code
SELECT Outlet_Location_Type,  
       ISNULL([Low Fat], 0) AS Low_Fat,  
       ISNULL([Regular], 0) AS Regular  
FROM  
(  
    SELECT Outlet_Location_Type, Item_Fat_Content,  
           CAST(SUM(Total_Sales) AS DECIMAL(10,2)) AS Total_Sales  
    FROM blinkit_data  
    GROUP BY Outlet_Location_Type, Item_Fat_Content  
) AS SourceTable  
PIVOT  
(  
    SUM(Total_Sales)  
    FOR Item_Fat_Content IN ([Low Fat], [Regular])  
) AS PivotTable  
ORDER BY Outlet_Location_Type;
```
![Fat Content by Outlet](Fat%20Content%20by%20Outlet%20for%20Total%20Sales.png)


## Explanation:

Purpose: This query summarizes total sales by outlet location and fat content type.
How it Works:
The subquery groups data by Outlet_Location_Type and Item_Fat_Content, summing total sales.
The PIVOT function converts Item_Fat_Content from rows into columns (Low Fat, Regular).
ISNULL([Low Fat], 0) replaces NULL values with 0 to avoid missing data.
The final result shows how different fat content products perform in each outlet location.


## Total Sales by Outlet Establishment Year
```sql
Copy code
SELECT Outlet_Establishment_Year, CAST(SUM(Total_Sales) AS DECIMAL(10,2)) AS Total_Sales  
FROM blinkit_data  
GROUP BY Outlet_Establishment_Year  
ORDER BY Outlet_Establishment_Year;
```
![Total Sales by Outlet Establishment](Total%20Sales%20by%20Outlet%20Establishment.png)

## Explanation:

Purpose: Identifies how much sales are generated by outlets based on their establishment year.
How it Works:
GROUP BY Outlet_Establishment_Year groups the total sales by the year the outlet was established.
SUM(Total_Sales) calculates total sales per year.
CAST(... AS DECIMAL(10,2)) ensures precision by formatting total sales to two decimal places.
ORDER BY Outlet_Establishment_Year sorts the results chronologically.
Business Insight: Helps understand whether newer outlets perform better than older ones.

## Percentage of Sales by Outlet Size
```sql
Copy code
SELECT  
    Outlet_Size,  
    CAST(SUM(Total_Sales) AS DECIMAL(10,2)) AS Total_Sales,  
    CAST((SUM(Total_Sales) * 100.0 / SUM(SUM(Total_Sales)) OVER()) AS DECIMAL(10,2)) AS Sales_Percentage  
FROM blinkit_data  
GROUP BY Outlet_Size  
ORDER BY Total_Sales DESC;
```
![Percentage of Sales by Outlet Size](Percentage%20of%20Sales%20by%20Outlet%20Size.png)

## Explanation:

Purpose: Calculates total sales and percentage of total sales by outlet size (Small, Medium, Large).
How it Works:
SUM(Total_Sales) calculates total sales for each outlet size.
SUM(SUM(Total_Sales)) OVER() computes the grand total across all outlet sizes.
(SUM(Total_Sales) * 100.0 / SUM(SUM(Total_Sales)) OVER()) calculates the percentage share of sales per outlet size.
ORDER BY Total_Sales DESC sorts results in descending order, showing the highest-selling outlet sizes first.
Business Insight: Helps identify which outlet size contributes the most to overall revenue

## Sales by Outlet Location
```sql
Copy code
SELECT Outlet_Location_Type, CAST(SUM(Total_Sales) AS DECIMAL(10,2)) AS Total_Sales  
FROM blinkit_data  
GROUP BY Outlet_Location_Type  
ORDER BY Total_Sales DESC;
```
![Sales by Outlet Location](Sales%20by%20Outlet%20Location.png)

## Explanation:

Purpose: Calculates total sales by outlet location type (e.g., Urban, Rural, Semi-Urban).
How it Works:
GROUP BY Outlet_Location_Type groups sales by each location type.
SUM(Total_Sales) calculates total sales per location.
CAST(... AS DECIMAL(10,2)) formats the sales data to two decimal places.
ORDER BY Total_Sales DESC sorts the results from highest to lowest sales.
Business Insight: Helps determine which locations perform best in sales.

## All Metrics by Outlet Type
```sql
Copy code
SELECT Outlet_Type,  
CAST(SUM(Total_Sales) AS DECIMAL(10,2)) AS Total_Sales,  
CAST(AVG(Total_Sales) AS DECIMAL(10,0)) AS Avg_Sales,  
COUNT(*) AS No_Of_Items,  
CAST(AVG(Rating) AS DECIMAL(10,2)) AS Avg_Rating,  
CAST(AVG(Item_Visibility) AS DECIMAL(10,2)) AS Item_Visibility  
FROM blinkit_data  
GROUP BY Outlet_Type  
ORDER BY Total_Sales DESC;
```
![All Metrics by Outlet Type](All%20Metrics%20by%20Outlet%20Type.png)

## Explanation:

Purpose: Computes multiple sales-related metrics for each outlet type (Supermarket Type1, Type2, Grocery Store).
How it Works:
SUM(Total_Sales): Calculates total sales per outlet type.
AVG(Total_Sales): Finds the average sales per outlet type.
COUNT(*): Counts the total number of items sold per outlet type.
AVG(Rating): Computes the average customer rating per outlet type.
AVG(Item_Visibility): Finds the average visibility of items in stores.
GROUP BY Outlet_Type: Groups the results by outlet type.
ORDER BY Total_Sales DESC: Sorts outlets based on total sales in descending order.
Business Insight: Provides a holistic view of outlet performance, including sales, customer ratings, and product visibility.

## Results and Insights
Low Fat products contributed significantly to total sales.
Items like frozen foods and dairy had the highest sales.
Larger outlets generated higher sales compared to smaller ones.
Newer outlets (established in recent years) had increasing sales trends.
Urban outlet locations outperformed rural areas.
## Conclusion
This analysis provided valuable insights into Blinkit's sales performance, outlet locations, and customer preferences. Data-driven decisions can be made to optimize stock, enhance marketing, and improve customer experience.





