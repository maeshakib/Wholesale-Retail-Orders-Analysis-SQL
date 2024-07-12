# Wholesale-Retail-Orders-Analysis-SQL

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
