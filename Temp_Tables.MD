<h1 align="center">Temp Tables</h1>

## Overview of the Section
* **[Temp Tables PART 1](#Temp-Tables-PART-1)**
    * **[Creating temporary tables](#Creating-temporary-tables)**

* **[Temp Tables PART 2](#Temp-Tables-PART-2)**

### Temp Tables PART 1

A temporary SQL table, also known as a temp table, is a table that is created and used within the context of a specific session or transaction in a database management system. 

It is designed to store temporary data that is needed for a short duration and does not require a permanent storage solution.

Temporary tables are tables that exist temporarily on the SQL Server.

The temporary tables are useful for storing the immediate result sets that are accessed multiple times.

### Creating temporary tables
SQL Server provided two ways to create temporary tables via SELECT INTO and CREATE TABLE statements.

Create temporary tables using SELECT INTO statement
The first way to create a temporary table is to use the SELECT INTO statement as shown below:
```
SELECT 
    select_list
INTO 
    temporary_table
FROM 
    table_name
```
The name of the temporary table starts with a hash symbol (#). 

For example, the following statement creates a temporary table using the SELECT INTO statement:

```
SELECT
    product_name,
    list_price
INTO #trek_products --- temporary table
FROM
    production.products
WHERE
    brand_id = 9;
```

In this example, we created a temporary table named #trek_products with two columns derived from the select list of the SELECT statement. 

The statement created the temporary table and populated data from the production.products table into the temporary table.

**Query example:**
```
SELECT 
       OrderDate
	  ,OrderMonth = DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1)
      ,TotalDue
	  ,OrderRank = ROW_NUMBER() OVER(PARTITION BY DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1) ORDER BY TotalDue DESC)

INTO #Sales
FROM AdventureWorks2022.Sales.SalesOrderHeader


SELECT
	OrderMonth,
	Top10Total = SUM(TotalDue)

INTO #Top10Sales
FROM #Sales
WHERE OrderRank <= 10

GROUP BY OrderMonth


SELECT
	A.OrderMonth,
	A.Top10Total,
	PrevTop10Total = B.Top10Total

FROM #Top10Sales A
	LEFT JOIN #Top10Sales B
		ON A.OrderMonth = DATEADD(MONTH,1,B.OrderMonth)

ORDER BY 1

SELECT * FROM #Sales WHERE OrderRank <= 10

DROP TABLE #Sales
DROP TABLE #Top10Sales
```

#### Results:
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_35.JPG)

**[Back To The Top](#Overview-of-the-Section)**

### Temp Tables PART 2