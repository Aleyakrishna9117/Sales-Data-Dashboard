# Sales-Data-Dashboard
# SalesData-Dashboard

### Dashboard Link : https://app.powerbi.com/groups/me/reports/53d1c326-fe7a-474a-82f8-96e962156e59/79ba81feb9111cef5348?experience=power-bi

## Problem Statement

This project focuses on creating a Power BI dashboard for a sales database with 10,000 records to help management track KPIs, clean data issues, and identify growth opportunities across products, customers, regions, and salespeople.

### Steps followed 

Step 1 : Create and load the Azure SQL sales table.
	
Azure SQL Database Script
Copy and run this entire script in Azure SQL:

-- ============================================
-- SALES DATABASE SETUP FOR POWER BI REPORT
-- 10,000 Records with intentional errors (FIXED)
-- ============================================

-- Drop existing table if exists
IF OBJECT_ID('dbo.SalesData', 'U') IS NOT NULL
    DROP TABLE dbo.SalesData;
GO

-- Create SalesData table
CREATE TABLE dbo.SalesData (
    SaleID INT PRIMARY KEY,
    SaleDate DATE,
    ProductCategory VARCHAR(50),
    ProductName VARCHAR(100),
    Region VARCHAR(50),
    Salesperson VARCHAR(100),
    Quantity VARCHAR(20),  -- VARCHAR to allow errors (should be INT)
    UnitPrice DECIMAL(10,2),
    Cost DECIMAL(10,2),
    CustomerID INT,
    CustomerAge VARCHAR(20),  -- VARCHAR to allow errors (should be INT)
    IsPrimeMember VARCHAR(3)
);
GO

-- Insert 10,000 records using INSERT...SELECT (NO WHILE LOOP - fixes subquery error)
SET NOCOUNT ON;

WITH Numbers AS (
    SELECT TOP 10000 
        ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS n
    FROM sys.objects o1
    CROSS JOIN sys.objects o2
)
INSERT INTO dbo.SalesData (
    SaleID, SaleDate, ProductCategory, ProductName, Region, Salesperson, 
    Quantity, UnitPrice, Cost, CustomerID, CustomerAge, IsPrimeMember
)
SELECT 
    n AS SaleID,
    DATEADD(DAY, ABS(CHECKSUM(NEWID())) % 730, '2024-01-01') AS SaleDate,
    CASE (ABS(CHECKSUM(NEWID())) % 5) 
        WHEN 0 THEN 'Electronics' 
        WHEN 1 THEN 'Clothing' 
        WHEN 2 THEN 'Home & Garden' 
        WHEN 3 THEN 'Sports' 
        ELSE 'Books' 
    END AS ProductCategory,
    CASE (ABS(CHECKSUM(NEWID())) % 5)
        WHEN 0 THEN CASE ABS(CHECKSUM(NEWID())) % 5 
            WHEN 0 THEN 'Laptop' WHEN 1 THEN 'Smartphone' WHEN 2 THEN 'Tablet'
            WHEN 3 THEN 'Headphones' ELSE 'Smart TV' END
        WHEN 1 THEN CASE ABS(CHECKSUM(NEWID())) % 5 
            WHEN 0 THEN 'T-Shirt' WHEN 1 THEN 'Jeans' WHEN 2 THEN 'Jacket'
            WHEN 3 THEN 'Dress' ELSE 'Shoes' END
        WHEN 2 THEN CASE ABS(CHECKSUM(NEWID())) % 5 
            WHEN 0 THEN 'Furniture' WHEN 1 THEN 'Lamp' WHEN 2 THEN 'Plant Pot'
            WHEN 3 THEN 'Rug' ELSE 'Kitchen Set' END
        WHEN 3 THEN CASE ABS(CHECKSUM(NEWID())) % 5 
            WHEN 0 THEN 'Yoga Mat' WHEN 1 THEN 'Dumbbell' WHEN 2 THEN 'Running Shoes'
            WHEN 3 THEN 'Bicycle' ELSE 'Tennis Racket' END
        ELSE CASE ABS(CHECKSUM(NEWID())) % 5 
            WHEN 0 THEN 'Fiction' WHEN 1 THEN 'Non-Fiction' WHEN 2 THEN 'Cookbook'
            WHEN 3 THEN 'Biography' ELSE 'Textbook' END
    END AS ProductName,
    CASE (ABS(CHECKSUM(NEWID())) % 5) 
        WHEN 0 THEN 'North' WHEN 1 THEN 'South' WHEN 2 THEN 'East' 
        WHEN 3 THEN 'West' ELSE 'Central' 
    END AS Region,
    CASE (ABS(CHECKSUM(NEWID())) % 8) 
        WHEN 0 THEN 'Alice Johnson' WHEN 1 THEN 'Bob Smith' WHEN 2 THEN 'Carol Davis'
        WHEN 3 THEN 'David Wilson' WHEN 4 THEN 'Emma Brown' WHEN 5 THEN 'Frank Miller'
        WHEN 6 THEN 'Grace Lee' ELSE 'Henry Taylor'
    END AS Salesperson,
    CAST((ABS(CHECKSUM(NEWID())) % 20 + 1) AS VARCHAR) AS Quantity,
    CAST(ABS(CHECKSUM(NEWID())) % 1990 + 10 AS DECIMAL(10,2)) AS UnitPrice,
    CAST((ABS(CHECKSUM(NEWID())) % 1990 + 10) * (0.4 + (ABS(CHECKSUM(NEWID())) % 40) / 100.0) AS DECIMAL(10,2)) AS Cost,
    ABS(CHECKSUM(NEWID())) % 9000 + 1000 AS CustomerID,
    CAST(ABS(CHECKSUM(NEWID())) % 58 + 18 AS VARCHAR) AS CustomerAge,
    CASE WHEN ABS(CHECKSUM(NEWID())) % 2 = 0 THEN 'YES' ELSE 'NO' END AS IsPrimeMember
FROM Numbers;
GO

-- Add INTENTIONAL ERRORS after initial insert (for Power Query practice)

-- ERROR 1: Empty quantity (every 500th record) - 20 records
UPDATE dbo.SalesData 
SET Quantity = '' 
WHERE SaleID % 500 = 0;

-- ERROR 2: NULL dates (every 700th record) - ~14 records  
UPDATE dbo.SalesData 
SET SaleDate = NULL 
WHERE SaleID % 700 = 0;

-- ERROR 3: Negative prices (every 800th record) - ~12 records
UPDATE dbo.SalesData 
SET UnitPrice = -ABS(UnitPrice) 
WHERE SaleID % 800 = 0;

-- ERROR 4: Empty region (every 600th record) - ~16 records
UPDATE dbo.SalesData 
SET Region = '' 
WHERE SaleID % 600 = 0;

-- ERROR 5: Text in CustomerAge (every 900th record) - ~11 records
UPDATE dbo.SalesData 
SET CustomerAge = 'Unknown' 
WHERE SaleID % 900 = 0;

GO

-- Verify record count
SELECT COUNT(*) AS TotalRecords FROM dbo.SalesData;
GO

-- Preview sample data
SELECT TOP 20 * FROM dbo.SalesData ORDER BY SaleID;
GO

-- Show error counts for verification
SELECT 
    SUM(CASE WHEN Quantity = '' THEN 1 ELSE 0 END) AS EmptyQuantityCount,
    SUM(CASE WHEN SaleDate IS NULL THEN 1 ELSE 0 END) AS NullDateCount,
    SUM(CASE WHEN UnitPrice < 0 THEN 1 ELSE 0 END) AS NegativePriceCount,
    SUM(CASE WHEN Region = '' THEN 1 ELSE 0 END) AS EmptyRegionCount,
    SUM(CASE WHEN CustomerAge = 'Unknown' THEN 1 ELSE 0 END) AS UnknownAgeCount
FROM dbo.SalesData;
GO

Step 2 :Clean the intentional errors in Power Query.

Step 3 :Create KPI measures in DAX.

--First Page KPI Measures and Their DAX formulas

	## Total Revenue = SUMX(SalesData, SalesData[Quantity] * SalesData[UnitPrice])

	## Total Profit = SUMX(SalesData, (SalesData[UnitPrice] - SalesData[Cost]) * SalesData[Quantity])

	## Profit Margin % = DIVIDE([Total Profit], [Total Revenue], 0) * 100

	## Total Transactions = COUNTROWS(SalesData)

	## Average Order Value = DIVIDE([Total Revenue], [Total Transactions], 0)

Step 4 :Design Page 1 for sales performance.

--Second Page KPI Measures and Their DAX formulas

	## Prime Member Rate = DIVIDE(CALCULATE(COUNTROWS(SalesData), SalesData[IsPrimeMember] = "YES"), COUNTROWS(SalesData), 0) * 100

	## Top Category = TOPN(1, VALUES(SalesData[ProductCategory]), [Total Revenue], DESC)

	## Average Customer Age = AVERAGE(SalesData[CustomerAge])

	## Active Salespeople = DISTINCTCOUNT(SalesData[Salesperson])

	## Revenue by Region = SUMX(SalesData, SalesData[Quantity] * SalesData[UnitPrice])

Step 5 :Design Page 2 for customer and product analysis.

Step 6 :Format, test, and finalize the report.
           


        

           


        
