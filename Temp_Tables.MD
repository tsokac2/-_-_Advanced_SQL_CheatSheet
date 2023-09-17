<h1 align="center">Temp Tables</h1>

### Overview of the Section
* **[Temp Tables PART 1](#Temp-Tables-PART-1)**
    * **[Creating temporary tables](#Creating-temporary-tables)**
    * **[Temp Tables Exercise](#Temp-Tables-Exercise)**
* **[CREATE and INSERT](#CREATE-and-INSERT)**
    * **[CREATE and INSERT Exercise](#CREATE-and-INSERT-Exercise)**
* **[TRUNCATE](#TRUNCATE)**
    * **[TRUNCATE Exercise](#TRUNCATE-Exercise)**
* **[UPDATE](#UPDATE)**
* **[CASE](#CASE)**
    * **[UPDATE and CASE Exercise](#UPDATE-and-CASE-Exercise)**
* **[DELETE](#DELETE)**
* **[Use Temp Tables when](#Use-Temp-Tables-when)**
#
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

### Temp Tables Exercise

Refactor your solution to the exercise from the section on CTEs (average sales/purchases minus top 10) using temp tables in place of CTEs.

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
TotalSales = SUM(TotalDue)
INTO #SalesMinusTop10
FROM #Sales
WHERE OrderRank > 10
GROUP BY OrderMonth

SELECT 
       OrderDate
	  ,OrderMonth = DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1)
      ,TotalDue
	  ,OrderRank = ROW_NUMBER() OVER(PARTITION BY DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1) ORDER BY TotalDue DESC)
INTO #Purchases
FROM AdventureWorks2022.Purchasing.PurchaseOrderHeader

SELECT
OrderMonth,
TotalPurchases = SUM(TotalDue)
INTO #PurchasesMinusTop10
FROM #Purchases
WHERE OrderRank > 10
GROUP BY OrderMonth

SELECT
A.OrderMonth,
A.TotalSales,
B.TotalPurchases

FROM #SalesMinusTop10 A
	JOIN #PurchasesMinusTop10 B
		ON A.OrderMonth = B.OrderMonth

ORDER BY 1

DROP TABLE #Sales
DROP TABLE #SalesMinusTop10
DROP TABLE #Purchases
DROP TABLE #PurchasesMinusTop10
```
#### Results:
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_36.JPG)

**[Back To The Top](#Overview-of-the-Section)**

#
### Use Temp Tables when

- You need to reference one of your virtual tables in multiple outputs.
- You need to join massive datasets in your virtual tables.
- You need a "script" instead of a query. 

#
### CREATE and INSERT

SELECT INTO statement is one of the easy ways to CREATE a new table and then copy the source table data into this newly created table. In other words, the SELECT INTO statement performs a combo task:

Creates a clone table of the source table with exactly the same column names and data types
Reads data from the source table
Inserts data into the newly created table

### CREATE and INSERT Exercise

Create Temp Table #Sales

```
CREATE TABLE #Sales
(
       OrderDate DATE
	  ,OrderMonth DATE
      ,TotalDue MONEY
	  ,OrderRank INT
)
```

Insert data in to temp #Sales table

```
INSERT INTO #Sales
(
       OrderDate
	  ,OrderMonth
      ,TotalDue
	  ,OrderRank
)
SELECT 
       OrderDate
	  ,OrderMonth = DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1)
      ,TotalDue
	  ,OrderRank = ROW_NUMBER() OVER(PARTITION BY DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1) ORDER BY TotalDue DESC)

FROM AdventureWorks2022.Sales.SalesOrderHeader
```
Test created temp table: #Sales

```
SELECT * FROM #Sales
```
#### Results:
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_37.JPG)

**[Back To The Top](#Overview-of-the-Section)**

#
### TRUNCATE

For clearing and reusing tables

TRUNCATE TABLE removes all rows from a table, but the table structure and its columns, constraints, indexes, and so on remain. 

To remove the table definition in addition to its data, use the DROP TABLE statement.

#### TRUNCATE Exercise

Leverage TRUNCATE to re-use temp tables in your solution to "CREATE and INSERT" exercise.

**Hints:**

Instead of joining two tables in your final SELECT (#AvgSalesMinusTop10 and #AvgPurchasesMinusTop10), you will most likely need to join a single consolidated query to itself.

The join will work much like before, but you will need to add a new wrinkle that filters each copy of the table based on whether it contains purchase or sales data.

For whatever copy of the table you put after the FROM, include the filtering criteria in the WHERE clause.

For the other copy of the table, apply the filtering criteria directly in the join.

These different "cuts" of the same table will accomplish the same thing as two distinct tables did previously.

In the SELECT clause of your final query, you will probably need to apply aliases to a couple of field names

to distinguish total sales from total purchases. Make sure you apply the appropriate alias to the field

from the appropriate copy of the table.

**Step 1** - Create Temp Table _#Orders_

```
CREATE TABLE #Orders
(
       OrderDate DATE
	  ,OrderMonth DATE
      ,TotalDue MONEY
	  ,OrderRank INT
)
```

**Step 2** - Insert sales data into Temp Table _#Orders_

```
INSERT INTO #Orders
(
       OrderDate
	  ,OrderMonth
      ,TotalDue
	  ,OrderRank
)
SELECT 
       OrderDate
	  ,OrderMonth = DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1)
      ,TotalDue
	  ,OrderRank = ROW_NUMBER() OVER(PARTITION BY DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1) ORDER BY TotalDue DESC)

FROM AdventureWorks2022.Sales.SalesOrderHeader
```
**Step 3** - Create Temp Table _#OrdersMinusTop10_

```
CREATE TABLE #OrdersMinusTop10
(
OrderMonth DATE,
OrderType VARCHAR(32),
TotalDue MONEY
)
```
**Step 4** - Insert sales data into Temp Table _#OrdersMinusTop10_

```
INSERT INTO #OrdersMinusTop10
(
OrderMonth,
OrderType,
TotalDue
)
SELECT
OrderMonth,
OrderType = 'Sales',
TotalDue = SUM(TotalDue)
FROM #Orders
WHERE OrderRank > 10
GROUP BY OrderMonth
```
**Step 5** - Empty out #Orders Temp Table

```TRUNCATE TABLE #Orders```

**Step 6** - Insert purchase data in to Temp Table #Orders

```
INSERT INTO #Orders
(
       OrderDate
	  ,OrderMonth
      ,TotalDue
	  ,OrderRank
)
SELECT 
       OrderDate
	  ,OrderMonth = DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1)
      ,TotalDue
	  ,OrderRank = ROW_NUMBER() OVER(PARTITION BY DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1) ORDER BY TotalDue DESC)

FROM AdventureWorks2022.Purchasing.PurchaseOrderHeader --purchase data
```
**Step 7** - Insert purchase data in to Temp Table #OrdersMinusTop10

```
INSERT INTO #OrdersMinusTop10
(
OrderMonth,
OrderType,
TotalDue
)
SELECT
OrderMonth,
OrderType = 'Purchase',
TotalDue = SUM(TotalDue)
FROM #Orders
WHERE OrderRank > 10
GROUP BY OrderMonth
```
**Step 8**  - Final SELECT statement

```
SELECT
A.OrderMonth,
TotalSales = A.TotalDue,
TotalPurchases = B.TotalDue

FROM #OrdersMinusTop10 A
	JOIN #OrdersMinusTop10 B
		ON A.OrderMonth = B.OrderMonth
			AND B.OrderType = 'Purchase'

WHERE A.OrderType = 'Sales'

ORDER BY 1

DROP TABLE #Orders
DROP TABLE #OrdersMinusTop10

```
#### Results:
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_38.JPG)

**[Back To The Top](#Overview-of-the-Section)**

#
### UPDATE

For modifying tables - no SELECT is required.

To store data in tables, we use databases. 
After the data is written to such tables, it is common that you need to update specific fields at some point throughout the data’s lifespan. 

To do this, we can utilize the UPDATE statement in the SQL language. An Update query is a form of action query that makes several updates to multiple records at once.

### CASE
Evaluates a list of conditions and returns one of multiple possible result expressions.

The CASE expression has two formats:

- The simple CASE expression compares an expression to a set of simple expressions to determine the result.

- The searched CASE expression evaluates a set of Boolean expressions to determine the result.

Both formats support an optional ELSE argument.

CASE can be used in any statement or clause that allows a valid expression. For example, you can use CASE in statements such as SELECT, UPDATE, DELETE and SET, and in clauses such as <select_list>, IN, WHERE, ORDER BY, and HAVING.

#### UPDATE and CASE Exercise
Update the value in the "OrderSubcategory" field as follows:

The value in the field should consist of the following string values concatenated together in this order:

- The value in the "OrderCategory" field
- A space
- A hyphen
- Another space
- The value in the "OrderAmtBucket" field

**Step 1**
```
CREATE TABLE #SalesOrders
(
 SalesOrderID INT,
 OrderDate DATE,
 TaxAmt MONEY,
 Freight MONEY,
 TotalDue MONEY,
 TaxFreightPercent FLOAT,
 TaxFreightBucket VARCHAR(32),
 OrderAmtBucket VARCHAR(32),
 OrderCategory VARCHAR(32),
 OrderSubcategory VARCHAR(32)
)
```
**Step 2**
```
INSERT INTO #SalesOrders
(
 SalesOrderID,
 OrderDate,
 TaxAmt,
 Freight,
 TotalDue,
 OrderCategory
)
SELECT
 SalesOrderID,
 OrderDate,
 TaxAmt,
 Freight,
 TotalDue,
 OrderCategory = 'Non-holiday Order'

FROM [AdventureWorks2022].[Sales].[SalesOrderHeader]

WHERE YEAR(OrderDate) = 2013
```
**Step 3**
```
UPDATE #SalesOrders
SET 
TaxFreightPercent = (TaxAmt + Freight)/TotalDue,
OrderAmtBucket = 
	CASE
		WHEN TotalDue < 100 THEN 'Small'
		WHEN TotalDue < 1000 THEN 'Medium'
		ELSE 'Large'
	END
```
**Step 4**
```
UPDATE #SalesOrders
SET TaxFreightBucket = 
	CASE
		WHEN TaxFreightPercent < 0.1 THEN 'Small'
		WHEN TaxFreightPercent < 0.2 THEN 'Medium'
		ELSE 'Large'
	END
```
**Step 5**
```
UPDATE #SalesOrders
SET  OrderCategory = 'Holiday'
FROM #SalesOrders
WHERE DATEPART(quarter,OrderDate) = 4
```
**Step 6**
```
UPDATE #SalesOrders
SET OrderSubcategory = OrderCategory + ' - ' + OrderAmtBucket
```

**Step 7**
```
SELECT * FROM #SalesOrders
```
#### Results:
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_39.JPG)

**[Back To The Top](#Overview-of-the-Section)**

#
### DELETE

Removing rows with precision.

#### Selecting from temp table with WHERE clause

**Step 1**
```
SELECT 
       OrderDate
	  ,OrderMonth = DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1)
      ,TotalDue
	  ,OrderRank = ROW_NUMBER() OVER(PARTITION BY DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1) ORDER BY TotalDue DESC)

INTO #Sales

FROM AdventureWorks2022.Sales.SalesOrderHeader
```
**Step 2**
```
SELECT
*
FROM #Sales
WHERE OrderRank <= 10
```
```
DROP TABLE #Sales
```
#### Deleting all records from temp table
```
SELECT 
       OrderDate
	  ,OrderMonth = DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1)
      ,TotalDue
	  ,OrderRank = ROW_NUMBER() OVER(PARTITION BY DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1) ORDER BY TotalDue DESC)

INTO #Sales

FROM AdventureWorks2022.Sales.SalesOrderHeader
```
```
DELETE FROM #Sales 
```
**[Back To The Top](#Overview-of-the-Section)**
