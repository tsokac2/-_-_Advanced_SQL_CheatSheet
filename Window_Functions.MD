<h1 align="center">Advanced Analysis With Window Functions</h1>

## Overview of the Section
* **[Window Functions With OVER](#Window-Functions-With-OVER)**
* **[PARTITION BY](#PARTITION-BY)**

## Window Functions With OVER
Window functions allow you to include aggregate calculations in your queries, WITHOUT otherwise changing the output in any way.

The aggregate calculations are simply tacked on to the query as an additional column.

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

![Section_01](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_01_01.JPG)

## Window Functions With OVER - Exercises

### Exercise 1

**Create a query with the following columns:**

* FirstName and LastName, from the Person.Person table**
* JobTitle, from the HumanResources.Employee table**
* Rate, from the HumanResources.EmployeePayHistory table**
* A derived column called "AverageRate" that returns the average of all values in the "Rate" column, in each row
* *All the above tables can be joined on **BusinessEntityID***
* All the tables can be ***inner joined***, and you do not need to apply any criteria.

#### Solution Exercise 1
```
SELECT 
 B.FirstName,
 B.LastName,
 C.JobTitle,
 A.Rate,
 AverageRate = AVG(A.Rate) OVER()

FROM AdventureWorks2022.HumanResources.EmployeePayHistory A
	JOIN AdventureWorks2022.Person.Person B
		ON A.BusinessEntityID = B.BusinessEntityID
	JOIN AdventureWorks2022.HumanResources.Employee C
		ON A.BusinessEntityID = C.BusinessEntityID
```
**Results:**

![Section_01](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_01_02.JPG)



#

## PARTITION BY

## PARTITION BY - Exercises
#