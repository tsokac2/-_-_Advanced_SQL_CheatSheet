<h1 align="center">Advanced Analysis With Window Functions</h1>

## Overview of the Section
* **[Window Functions With OVER](#Window-Functions-With-OVER)**
* **[PARTITION BY](#PARTITION-BY)**

## OVER()
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

![Section_01](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_01.JPG)

## OVER() - Exercises

### Exercise 1

**Create a query with the following columns:**

* _FirstName_ and _LastName_, from the **Person.Person table**
* _JobTitle_, from the **HumanResources.Employee table**
* _Rate_, from the **HumanResources.EmployeePayHistory table**
* A derived column called "AverageRate" that returns the average of all values in the "Rate" column, in each row
* All the above tables can be joined on **BusinessEntityID**
* All the tables can be ***inner joined***, and you do not need to apply any criteria.

#### Solution Exercise 1
```
SELECT 
 P.FirstName
,P.LastName
,T.JobTitle
,R.Rate
,[Average Rate] = AVG(R.Rate) OVER()

FROM AdventureWorks2022.Person.Person P
	JOIN AdventureWorks2022.HumanResources.Employee T
	    ON P.BusinessEntityID = T.BusinessEntityID
	JOIN AdventureWorks2022.HumanResources.EmployeePayHistory R
	    ON R.BusinessEntityID = P.BusinessEntityID
```
#### Results:

![Section_01](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_01_E_01.JPG)

### Exercise 2
* Enhance your query from Exercise 1 by adding a derived column called
* "MaximumRate" that returns the largest of all values in the "Rate" column, in each row.

```
SELECT
 P.FirstName
,P.LastName
,T.JobTitle
,R.Rate
,[AverageRate] = AVG(R.Rate) OVER()
,[MaximumRate] = MAX(R.Rate) OVER()

FROM AdventureWorks2022.Person.Person P
JOIN AdventureWorks2022.HumanResources.Employee T
	ON P.BusinessEntityID = T.BusinessEntityID
JOIN AdventureWorks2022.HumanResources.EmployeePayHistory R
	ON P.BusinessEntityID = R.BusinessEntityID
```

#### Results:

![Section_01](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_01_E_02.JPG)

### Exercise 3
* Enhance your query from Exercise 2 by adding a derived column called "DiffFromAvgRate" that returns the result of the following calculation:
* An employees's pay rate, MINUS the average of all values in the "Rate" column.

```
SELECT
 P.FirstName
,P.LastName
,T.JobTitle
,R.Rate
,[AverageRate] = AVG(R.Rate) OVER()
,[MaximumRate] = MAX(R.Rate) OVER()
,[DiffFromAvgRate] = R.Rate - AVG(R.Rate) OVER()

FROM AdventureWorks2022.Person.Person P
JOIN AdventureWorks2022.HumanResources.Employee T
	ON P.BusinessEntityID = T.BusinessEntityID
JOIN AdventureWorks2022.HumanResources.EmployeePayHistory R
	ON P.BusinessEntityID = R.BusinessEntityID
```
#### Results:

![Section_01](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_01_E_03.JPG)

### Exercise 4
* Enhance your query from Exercise 3 by adding a derived column called "PercentofMaxRate" that returns the result of the following calculation:
* An employees's pay rate, DIVIDED BY the maximum of all values in the "Rate" column, times 100.

```
SELECT
 P.FirstName
,P.LastName
,T.JobTitle
,R.Rate
,[AverageRate] = AVG(R.Rate) OVER()
,[MaximumRate] = MAX(R.Rate) OVER()
,[DiffFromAvgRate] = R.Rate - AVG(R.Rate) OVER()
,[PercentofMaxRate] = (R.Rate / MAX(R.Rate) OVER()) * 100

FROM AdventureWorks2022.Person.Person P
JOIN AdventureWorks2022.HumanResources.Employee T
	ON P.BusinessEntityID = T.BusinessEntityID
JOIN AdventureWorks2022.HumanResources.EmployeePayHistory R
	ON P.BusinessEntityID = R.BusinessEntityID
```
#### Results:

![Section_01](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_01_E_04.JPG)
#
## PARTITION BY

Allows us to compute aggregate totals for groups within our data, while still retaining row-level detail.

Assigns each row of your query output to a group, without collapsing your data into fewer rows as with GROUP BY.

Instead of groups being assigned based on the distinct values of ALL the non-aggregated columns of our data, we need to specify the columns these groups will be based on. 

## PARTITION BY - Exercises

### Exercise 1

**Create a query with the following columns:**

* “Name” from the **Production.Product** table, which can be alised as “ProductName”

* “ListPrice” from the **Production.Product** table

* “Name” from the **Production. ProductSubcategory** table, which can be alised as “ProductSubcategory”*

* “Name” from the **Production.ProductCategory** table, which can be alised as “ProductCategory”**

* Join **Production.ProductSubcategory** to **Production.Product** on “ProductSubcategoryID”

* Join **Production.ProductCategory** to **ProductSubcategory** on “ProductCategoryID”

* All the tables can be inner joined, and you do not need to apply any criteria.


#### Solution Exercise 1
```
SELECT 
  ProductName = A.Name,
  A.ListPrice,
  ProductSubcategory = B.Name,
  ProductCategory = C.Name


FROM AdventureWorks2022.Production.Product A
  JOIN AdventureWorks2022.Production.ProductSubcategory B
    ON A.ProductSubcategoryID = B.ProductSubcategoryID
  JOIN AdventureWorks2022.Production.ProductCategory C
    ON B.ProductCategoryID = C.ProductCategoryID
```
#### Results:

![Section_01](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_01_E_05.JPG)


#