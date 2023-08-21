<h1 align="center">Common_Table_Expressions_(CETs)</h1>

## Overview of the Section
* **[Introducing Common Table Expression (CTE)](#Introducing-Common-Table-Expression-(CTE))**


## Introducing Common Table Expression (CTE)

The Common Table Expression (CTE) is a powerful construct in SQL that helps simplify a query. 

CTEs work as virtual tables (with records and columns), created during the execution of a query, used by the query, and eliminated after query execution. 

CTEs often act as a bridge to transform the data in source tables to the format expected by the query.

A common table expression, or CTE, is a temporary named result set created from a simple SELECT statement that can be used in a subsequent SELECT statement. 

Each SQL CTE is like a named query, whose result is stored in a virtual table (a CTE) to be referenced later in the main quer

**The problem**

- Identify the top 10 sales orders per month.
- Aggregate these into a sum total, by month.
- Compare each month's total to the previous months's, on the same row. 

## Subquery approach to "The Problem":

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



#
## SCALAR Subqueries