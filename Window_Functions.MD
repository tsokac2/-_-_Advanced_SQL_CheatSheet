<h1 align="center">Advanced Analysis With Window Functions</h1>

## Overview of the Section
* **[Window Functions With OVER](#Window-Functions-With-OVER)**
* **[PARTITION BY](#PARTITION-BY)**

## Window Functions With OVER
Window functions allow you to include aggregate calculations in your queries, WITHOT otherwise changing the output in any way.

The aggregate calculations is simpe tacked on to the query as an additional column.

It is also possible to group these calculations, just like we can with aggregate queries, using PARTITION.

**Query example:**
```
SELECT BusinessEntityID
      ,Bonus * 100
      ,CommissionPct
      ,SalesYTD
	  ,SalesLastYear
      ,[Total YTD Sales] = SUM(SalesYTD) OVER()
      ,[Max YTD Sales] = MAX(SalesYTD) OVER()
      ,[% of Best Performer] = SalesYTD/MAX(SalesYTD) OVER() * 100

FROM AdventureWorks2022.Sales.SalesPerson
```
**Results:**
![Section_01](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet
/blob/main/public/img/Section_01_01.jpg)

## Window Functions With OVER - Exercises
#

## PARTITION BY

## PARTITION BY - Exercises
#