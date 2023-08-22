<h1 align="center">Common_Table_Expressions_(CETs)</h1>

## Overview of the Section
* **[Introducing Common Table Expression CTE](#Introducing-Common-Table-Expression-CTE)**
    * **[Subquery approach](#Subquery-approach)**
    * **[Refactored CTE approach](#Refactored-CTE-approach)**
    * **[CTEs Exercise](#CTEs-Exercise)**
* **[Recursive CTEs](#Recursive-CTEs)**
    * **[Recursive CTEs - Exercise 1](#Recursive-CTEs---Exercise-1)**  


## Introducing Common Table Expression CTE

The Common Table Expression (CTE) is a powerful construct in SQL that helps simplify a query. 

CTEs work as virtual tables (with records and columns), created during the execution of a query, used by the query, and eliminated after query execution. 

CTEs often act as a bridge to transform the data in source tables to the format expected by the query.

A common table expression, or CTE, is a temporary named result set created from a simple SELECT statement that can be used in a subsequent SELECT statement. 

Each SQL CTE is like a named query, whose result is stored in a virtual table (a CTE) to be referenced later in the main quer

**The problem**

- Identify the top 10 sales orders per month.
- Aggregate these into a sum total, by month.
- Compare each month's total to the previous months's, on the same row. 

### Subquery approach

```
SELECT
A.OrderMonth,
A.Top10Total,
PrevTop10Total = B.Top10Total

FROM (
SELECT
OrderMonth,
Top10Total = SUM(TotalDue)
FROM (
SELECT 
       OrderDate
	  ,OrderMonth = DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1)
      ,TotalDue
	  ,OrderRank = ROW_NUMBER() OVER(PARTITION BY DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1) ORDER BY TotalDue DESC)
FROM AdventureWorks2022.Sales.SalesOrderHeader
) X
WHERE OrderRank <= 10
GROUP BY OrderMonth
) A

LEFT JOIN (
SELECT
OrderMonth,
Top10Total = SUM(TotalDue)
FROM (
SELECT 
       OrderDate
	  ,OrderMonth = DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1)
      ,TotalDue
	  ,OrderRank = ROW_NUMBER() OVER(PARTITION BY DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1) ORDER BY TotalDue DESC)
FROM AdventureWorks2022.Sales.SalesOrderHeader
) X
WHERE OrderRank <= 10
GROUP BY OrderMonth
) B
ON A.OrderMonth = DATEADD(MONTH,1,B.OrderMonth)

ORDER BY 1
```
#### Results:

![Section_03](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_03_E_28.JPG)

**[Back To The Top](#Overview-of-the-Section)**

### Refactored CTE approach
```
WITH Sales AS
(
SELECT 
       OrderDate
	  ,OrderMonth = DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1)
      ,TotalDue
	  ,OrderRank = ROW_NUMBER() OVER(PARTITION BY DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1) ORDER BY TotalDue DESC)
FROM AdventureWorks2022.Sales.SalesOrderHeader
)

,Top10Sales AS
(
SELECT
OrderMonth,
Top10Total = SUM(TotalDue)
FROM Sales
WHERE OrderRank <= 10
GROUP BY OrderMonth
)

SELECT
A.OrderMonth,
A.Top10Total,
PrevTop10Total = B.Top10Total

FROM Top10Sales A
	LEFT JOIN Top10Sales B
		ON A.OrderMonth = DATEADD(MONTH,1,B.OrderMonth)

ORDER BY 1
```
#### Results:
![Section_03](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_03_E_29.JPG)

**[Back To The Top](#Overview-of-the-Section)**

### CTEs Exercise

For this exercise, assume the CEO of our fictional company decided that the top 10 orders per month are actually outliers that need to be clipped out of our data before doing meaningful analysis.

Further, she would like the sum of sales AND purchases (minus these "outliers") listed side by side, by month.

We've got a query that already does this, but it's messy and hard to read. Re-write it using a CTE so other analysts can read and understand the code.

#### Messy SQL Code:
```
SELECT
A.OrderMonth,
A.TotalSales,
B.TotalPurchases

FROM (
	SELECT
	OrderMonth,
	TotalSales = SUM(TotalDue)
	FROM (
		SELECT 
		   OrderDate
		  ,OrderMonth = DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1)
		  ,TotalDue
		  ,OrderRank = ROW_NUMBER() OVER(PARTITION BY DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1) ORDER BY TotalDue DESC)
		FROM AdventureWorks2022.Sales.SalesOrderHeader
		) S
	WHERE OrderRank > 10
	GROUP BY OrderMonth
) A

JOIN (
	SELECT
	OrderMonth,
	TotalPurchases = SUM(TotalDue)
	FROM (
		SELECT 
		   OrderDate
		  ,OrderMonth = DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1)
		  ,TotalDue
		  ,OrderRank = ROW_NUMBER() OVER(PARTITION BY DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1) ORDER BY TotalDue DESC)
		FROM AdventureWorks2022.Purchasing.PurchaseOrderHeader
		) P
	WHERE OrderRank > 10
	GROUP BY OrderMonth
) B	ON A.OrderMonth = B.OrderMonth

ORDER BY 1
```
#### Results:
![Section_03](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_03_E_30.JPG)

#### CTE Solution SQL Code:
```
WITH Sales AS
(
SELECT 
       OrderDate
	  ,OrderMonth = DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1)
      ,TotalDue
	  ,OrderRank = ROW_NUMBER() OVER(PARTITION BY DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1) ORDER BY TotalDue DESC)
FROM AdventureWorks2022.Sales.SalesOrderHeader
)

,SalesMinusTop10 AS
(
SELECT
OrderMonth,
TotalSales = SUM(TotalDue)
FROM Sales
WHERE OrderRank > 10
GROUP BY OrderMonth
)

,Purchases AS
(
SELECT 
       OrderDate
	  ,OrderMonth = DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1)
      ,TotalDue
	  ,OrderRank = ROW_NUMBER() OVER(PARTITION BY DATEFROMPARTS(YEAR(OrderDate),MONTH(OrderDate),1) ORDER BY TotalDue DESC)
FROM AdventureWorks2022.Purchasing.PurchaseOrderHeader
)

,PurchasesMinusTop10 AS
(
SELECT
OrderMonth,
TotalPurchases = SUM(TotalDue)
FROM Purchases
WHERE OrderRank > 10
GROUP BY OrderMonth
)

SELECT
A.OrderMonth,
A.TotalSales,
B.TotalPurchases

FROM SalesMinusTop10 A
	JOIN PurchasesMinusTop10 B
		ON A.OrderMonth = B.OrderMonth

ORDER BY 1
```
#### Results:
![Section_03](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_03_E_30.JPG)

**[Back To The Top](#Overview-of-the-Section)**
#
### Recursive CTEs

A recursive Common Table Expression (CTE) is a CTE that references itself. 
By doing so, the CTE repeatedly executes, returns subsets of data, until it returns the complete result set.

A recursive CTE is useful in querying hierarchical data such as organization charts where one employee reports to a manager or multi-level bill of materials when a product consists of many components, and each component itself also consists of many other components.

**Query example:** Example 1: Generate number series with recursive CTE
```
WITH NumberSeries AS
(
SELECT
 1 AS MyNumber

UNION  ALL

SELECT 
MyNumber + 1
FROM NumberSeries
WHERE MyNumber < 100
)

SELECT
MyNumber
FROM NumberSeries
```
#### Results:
![Section_03](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_03_E_31.JPG)

The DATEADD() function adds a time/date interval to a date and then returns the date.

**Query example:** Example 2: Generate number series with recursive CTE
```
WITH Dates AS
(
SELECT
 CAST('01-01-2021' AS DATE) AS MyDate

UNION ALL

SELECT
DATEADD(DAY, 1, MyDate)
FROM Dates
WHERE MyDate < CAST('12-31-2021' AS DATE)
)

SELECT
MyDate

FROM Dates
OPTION (MAXRECURSION 365)
```
#### Results:
![Section_03](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_03_E_32.JPG)

**[Back To The Top](#Overview-of-the-Section)**

### Recursive CTEs - Exercise 1

Use a recursive CTE to generate a list of all odd numbers between 1 and 100.
```
WITH Odds AS
(
SELECT
 1 AS MyOddNumber

UNION  ALL

SELECT 
MyOddNumber + 2
FROM Odds
WHERE MyOddNumber < 99
)

SELECT
MyOddNumber
FROM Odds
```
#### Results:
![Section_03](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_03_E_33.JPG)

### Recursive CTEs - Exercise 2

Use a recursive CTE to generate a date series of all FIRST days of the month (1/1/2021, 2/1/2021, etc.) 

From 1/1/2020 to 12/1/2029.

_Hints:_
- Use the DATEADD function strategically in your recursive member.
- You may also have to modify MAXRECURSION.

```
WITH Dates AS
(
SELECT
 CAST('01-01-2020' AS DATE) AS MyDate

UNION ALL

SELECT
DATEADD(MONTH, 1, MyDate)
FROM Dates
WHERE MyDate < CAST('12-01-2029' AS DATE)
)

SELECT
MyDate

FROM Dates
OPTION (MAXRECURSION 120)
```
#### Results:
![Section_03](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_03_E_34.JPG)

**[Back To The Top](#Overview-of-the-Section)**