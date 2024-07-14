# Wholesale-Retail-Orders-Analysis-SQL
- orders: 185013 
- product-supplier: 5504

Open SQL Server Management Studio (SSMS) and connect to your SQL Server instance.
- Right-click on the database where you want to import the CSV file.
- Select Tasks > Import Data.
- This will open the SQL Server Import and Export Wizard.
**Choose the Data Source:**
- Data Source: Flat File Source
- Browse to select your CSV file.

**Choose the Destination:**
- Destination: SQL Server Native Client
- Specify the server name and database name.
**Select the destination table:**
- You can import the data into an existing table or create a new one.
**Run the Import:**
- Review the mappings between source and destination columns.
- Click "Next" and then "Finish" to run the import process.

```sql
/******Creating a new column named "Item Retail Value"******/
ALTER TABLE [wholesale_retail_orders].[dbo].[orders]
ADD [Item Retail Value] DECIMAL(10,2);

/******update data from query******/
UPDATE [wholesale_retail_orders].[dbo].[orders]
SET [Item Retail Value] = CAST([Total Retail Price for This Order] AS DECIMAL(10,2)) / CAST([Quantity Ordered] AS DECIMAL(10,2));
```

```sql
/****** see top 100 records ******/
SELECT TOP (100) [Customer ID]
      ,[Customer Status]
      ,[Date Order was placed]
      ,[Delivery Date]
      ,[Order ID]
      ,[Product ID]
      ,[Quantity Ordered]
      ,[Total Retail Price for This Order]
      ,[Cost Price Per Unit]
      ,[Item Retail Value]
  FROM [wholesale_retail_orders ].[dbo].[orders]

```

Grouping by Product - Cost Price per Unit (Wholesale) Mean and Item Retail Value (Retail) Mean
```sql
SELECT  [Product ID],

  COUNT([Product ID]) AS Product_Count,
    AVG(CAST([Cost Price Per Unit] AS DECIMAL(10, 2))) AS Avg_Cost_Price_Per_Unit,
    AVG(CAST([Item Retail Value] AS DECIMAL(10, 2))) AS Avg_Item_Retail_Value
  FROM [wholesale_retail_orders ].[dbo].[orders]
  group by [Product ID]
  order by [Product ID]
```

**Find top 10 highest reveue generating products**

```sql
select top 10 s.[Product Name],sum(cast ([Total Retail Price for This Order] as decimal(10,2))) as sales
from [wholesale_retail_orders ].[dbo].[orders] o join  [wholesale_retail_orders ].[dbo].[product-supplier] s
on o.[Product ID]=s.[Product ID]
group by s.[Product Name]
order by sales desc
```

#### adding new column for "Supplier Country" with named "Supplier Country Long"
```sql
--Step 1: Add the new column "Supplier Country Long"--
ALTER TABLE [wholesale_retail_orders ].[dbo].[product-supplier] 
ADD   [Supplier Country Long]  VARCHAR(50);

-- Step 2: Update the new column with the full country names
UPDATE [wholesale_retail_orders ].[dbo].[product-supplier]
SET [Supplier Country Long] = CASE
    WHEN "Supplier Country" = 'US' THEN 'United States'
    WHEN "Supplier Country" = 'BE' THEN 'Belgium'
    WHEN "Supplier Country" = 'FR' THEN 'France'
    WHEN "Supplier Country" = 'NO' THEN 'Norway'
    WHEN "Supplier Country" = 'AU' THEN 'Australia'
    WHEN "Supplier Country" = 'SE' THEN 'Sweden'
    WHEN "Supplier Country" = 'PT' THEN 'Portugal'
    WHEN "Supplier Country" = 'ES' THEN 'Spain'
    WHEN "Supplier Country" = 'NL' THEN 'Netherlands'
    WHEN "Supplier Country" = 'DE' THEN 'Germany'
    WHEN "Supplier Country" = 'DK' THEN 'Denmark'
    WHEN "Supplier Country" = 'CA' THEN 'Canada'
    WHEN "Supplier Country" = 'GB' THEN 'United Kingdom'
    ELSE 'Unknown'
END;
```

#### Find top 5 highest selling products in each country
```sql
with cteTop10ProductSales as (
select top 10 s.[Supplier Country Long], s.[Product Name],sum(cast ([Total Retail Price for This Order] as decimal(10,2))) as sales
from [wholesale_retail_orders ].[dbo].[orders] o join  [wholesale_retail_orders ].[dbo].[product-supplier] s
on o.[Product ID]=s.[Product ID]
group by s.[Supplier Country Long],s.[Product Name]
order by sales desc

)
select * from (
select *
, row_number() over(partition by [Supplier Country Long] order by sales desc) as rn
from cteTop10ProductSales) A
where rn<=5
order by [Supplier Country Long]

```


#### Find month over month growth comparison from 2017 to 2021 sales eg : jan 2017 vs jan 2018
```sql
with cteSalesOrders as(
SELECT 
    YEAR(CONVERT(DATE, [Delivery Date], 120)) AS OrderYear,
    MONTH(CONVERT(DATE, [Delivery Date], 120)) AS OrderMonth,
   sum(cast ([Total Retail Price for This Order] as decimal(10,2))) as sales
FROM 
    [wholesale_retail_orders ].[dbo].[orders]
GROUP BY 
    YEAR(CONVERT(DATE, [Delivery Date], 120)),
    MONTH(CONVERT(DATE, [Delivery Date], 120))
	
)
select OrderMonth,
SUM(case when OrderYear=2017 then sales else 0 end) as sales_2017,
SUM(case when OrderYear=2018 then sales else 0 end) as sales_2018
,SUM(case when OrderYear=2019 then sales else 0 end) as sales_2019
,SUM(case when OrderYear=2020 then sales else 0 end) as sales_2020
,SUM(case when OrderYear=2021 then sales else 0 end) as sales_2021

from cteSalesOrders 
group by OrderMonth 
order by OrderMonth

```



#### Foreach category which month had highest sales 
```sql
with cteSalesOrders as(
SELECT 
[Product Category],
      FORMAT(CONVERT(DATE, [Delivery Date], 120), 'yyyyMM') AS OrderYearMonth,
   sum(cast ([Total Retail Price for This Order] as decimal(10,2))) as sales
FROM 
      [wholesale_retail_orders ].[dbo].[orders] o join  [wholesale_retail_orders ].[dbo].[product-supplier] s
on o.[Product ID]=s.[Product ID]
GROUP BY 
   [Product Category],
 FORMAT(CONVERT(DATE, [Delivery Date], 120), 'yyyyMM')
	
)
select * from 
(
select * , row_number() over(partition by [Product Category] order by sales desc) as rn
from cteSalesOrders
) a
where rn=1
```
#### Category had highest growth by profit in 2023 compare to 2022


```sql
with cteSalesOrders as(
SELECT 
   [Product Category], YEAR(CONVERT(DATE, [Delivery Date], 120)) AS OrderYear,
    
   sum(cast ([Total Retail Price for This Order] as decimal(10,2))) as sales
FROM 
   
	  [wholesale_retail_orders ].[dbo].[orders] o join  [wholesale_retail_orders ].[dbo].[product-supplier] s
on o.[Product ID]=s.[Product ID]
GROUP BY 
   [Product Category], YEAR(CONVERT(DATE, [Delivery Date], 120)) 
    
	
),
cte as(
select [Product Category],
 
SUM(case when OrderYear=2020 then sales else 0 end) as sales_2020
,SUM(case when OrderYear=2021 then sales else 0 end) as sales_2021

from cteSalesOrders 
group by [Product Category] 
 )
 select top 1 *
,
(
cast(sales_2021 as decimal(10,2))-cast(sales_2020 as decimal(10,2))
) [dd]
from  cte
order by (cast(sales_2021 as decimal(10,2))-cast(sales_2020 as decimal(10,2))) desc
```
