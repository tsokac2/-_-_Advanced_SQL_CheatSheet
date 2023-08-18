<h1 align="center">Subqueries</h1>

## Overview of the Section
* **[Introducing Subqueries](#Introducing-Subqueries)**
* **[SCALAR Subqueries](#SCALAR-Subqueries)**
* **[CORRELATED Subqueries](#CORRELATED-Subqueries)**
* **[EXISTS and NOT EXISTS Subqueries](#EXISTS-and-NOT-EXISTS-Subqueries)**
* **[FOR XML PATH With STUFF Function](#FOR-XML-PATH-With-STUFF-Function)**
* **[PIVOT - Part 1](#PIVOT---Part-1)**
* **[PIVOT - Part 2](#PIVOT---Part-2)**

## Introducing Subqueries

A subquery is a SQL query nested inside a larger query.

A subquery may occur in :
- A SELECT clause
- A FROM clause
- A WHERE clause

The subquery can be nested inside a SELECT, INSERT, UPDATE, or DELETE statement or inside another subquery.

A subquery is usually added within the WHERE Clause of another SQL SELECT statement.

You can use the comparison operators, such as >, <, or =. The comparison operator can also be a multiple-row operator, such as IN, ANY, or ALL.

A subquery is also called an inner query or inner select, while the statement containing a subquery is also called an outer query or outer select.
The inner query executes first before its parent query so that the results of an inner query can be passed to the outer query.

You can use a subquery in a SELECT, INSERT, DELETE, or UPDATE statement to perform the following tasks:

Compare an expression to the result of the query.
Determine if an expression is included in the results of the query.
Check whether the query selects any rows.

Multi-step SQL queries are **EXTREMELY** common in the real world od data analysis.

Subqueries are best for straightforward, two-step queries.

SQL gives us other techniques that are better-suted to complex, many-step analysis.

**Query example:** Selecting the most expensive item per order in a single query

```
SELECT
*
FROM
(
SELECT
SalesOrderID,
SalesOrderDetailID,
LineTotal,
LineTotalRanking = ROW_NUMBER() OVER(PARTITION BY SalesOrderID ORDER BY LineTotal DESC)

FROM AdventureWorks2022.Sales.SalesOrderDetail

) A

WHERE LineTotalRanking = 1
```
### Exercise 1

Write a query that displays the three most expensive orders, per vendor ID, from the Purchasing.PurchaseOrderHeader table. 

There should ONLY be three records per Vendor ID, even if some of the total amounts due are identical. "Most expensive" is defined by the amount in the "TotalDue" field.

_Include the following fields in your output:_

- PurchaseOrderID
- VendorID
- OrderDate
- TaxAmt
- Freight
- TotalDue

_Hints:_

- You will first need to define a field that assigns a unique rank to every purchase order, within each group of like vendor IDs.
- You'll probably want to use a Window Function with PARTITION BY and ORDER BY to do this.
- The last step will be to apply the appropriate criteria to the field you created with your Window Function.

```
SELECT
	PurchaseOrderID,
	VendorID,
	OrderDate,
	TaxAmt,
	Freight,
	TotalDue

FROM (
	SELECT 
		PurchaseOrderID,
		VendorID,
		OrderDate,
		TaxAmt,
		Freight,
		TotalDue,
		PurchaseOrderRank = ROW_NUMBER() OVER(PARTITION BY VendorID ORDER BY TotalDue DESC)
	FROM AdventureWorks2022.Purchasing.PurchaseOrderHeader
) X

WHERE PurchaseOrderRank <= 3
```
#### Results:

![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_01.JPG)


### Exercise 2

Modify your query from the first problem, such that the top three purchase order amounts are returned, regardless of how many records are returned per Vendor Id.

In other words, if there are multiple orders with the same total due amount, all should be returned as long as the total due amount for these orders is one of the top three.

Ultimately, you should see three distinct total due amounts (i.e., the top three) for each group of like Vendor Ids. However, there could be multiple records for each of these amounts.

_Hint:_ Think carefully about how the different ranking functions (ROW_NUMBER, RANK, and DENSE_RANK) work, and which one might be best suited to help you here.

```
SELECT
	PurchaseOrderID,
	VendorID,
	OrderDate,
	TaxAmt,
	Freight,
	TotalDue

FROM (
	SELECT 
		PurchaseOrderID,
		VendorID,
		OrderDate,
		TaxAmt,
		Freight,
		TotalDue,
		PurchaseOrderRank = DENSE_RANK() OVER(PARTITION BY VendorID ORDER BY TotalDue DESC)

	FROM AdventureWorks2022.Purchasing.PurchaseOrderHeader
) X

WHERE PurchaseOrderRank <= 3
```
#### Results:

![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_02.JPG)

**[Back To The Top](#Overview-of-the-Section)**

#
## SCALAR Subqueries

Subqueries that return a single value.

A scalar subquery expression is a subquery that returns exactly one column value from one row. 

The value of the scalar subquery expression is the value of the select list item of the subquery. 

If the subquery returns 0 rows, then the value of the scalar subquery expression is NULL

**Query example:**
```
SELECT 
 ProductID
,[Name]
,StandardCost
,ListPrice
,AvgListPrice = (SELECT AVG(ListPrice) FROM AdventureWorks202Production.Product) -- Subquerie
,AvgListPriceDiff = ListPrice - (SELECT AVG(ListPrice) FROAdventureWorks2022.Production.Product) -- Subquerie

FROM AdventureWorks2022.Production.Product

WHERE ListPrice > (SELECT AVG(ListPrice) FROM AdventureWorks2022.Production.Product)

ORDER BY ListPrice ASC
```
**Results:**

![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_03.JPG)

### Exercise 1

Create a query that displays all rows and the following columns from the AdventureWorks2022.HumanResources.Employee table:

- BusinessEntityID
- JobTitle
- VacationHours

Also include a derived column called "MaxVacationHours" that returns the maximum amount of vacation hours for any one employee, in any given row.
```
SELECT
 BusinessEntityID
,JobTitle
,VacationHours
,MaxVacationHours = (SELECT MAX(VacationHours) FROM AdventureWorks202HumanResources.Employee)

FROM AdventureWorks2022.HumanResources.Employee
```
#### Results:

![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_04.JPG)

### Exercise 2

Add a new derived field to your query from Exercise 1, which returns the percent an individual employees' vacation hours are, of the maximum vacation hours for any employee. 

For example, the record for the employee with the most vacation hours should have a value of 1.00, or 100%, in this column.

_Hints:_

- You can repurpose your logic from the "MaxVacationHours" for the denominator.
- Make sure you multiply at least one side of your equation by 1.0, to ensure the output will be a decimal.

```
SELECT
 BusinessEntityID
,JobTitle
,VacationHours
,MaxVacationHours = (SELECT MAX(VacationHours) FROM AdventureWorks202HumanResources.Employee)
,PercentOfMaxVacationHours = (VacationHours * 1.0) / (SELECT M(VacationHours)

FROM AdventureWorks2022.HumanResources.Employee)

FROM AdventureWorks2022.HumanResources.Employee

```
#### Results:

![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_05.JPG)

### Exercise 3

Refine your output with a criterion in the WHERE clause that filters out any employees whose vacation hours are less then 80% of the maximum amount of vacation hours for any one employee. 

In other words, return only employees who have at least 80% as much vacation time as the employee with the most vacation time.

_Hint:_ The query should return 60 rows.

```
SELECT
 BusinessEntityID
,JobTitle
,VacationHours
,MaxVacationHours = (SELECT MAX(VacationHours) FROM AdventureWorks2022HumanResources.Employee)
,PercentOfMaxVacationHours = (VacationHours * 1.0) / (SELECT M(VacationHours) FROM AdventureWorks2022.HumanResources.Employee)

FROM AdventureWorks2022.HumanResources.Employee

WHERE (VacationHours * 1.0) / (SELECT MAX(VacationHours) FROM AdventureWorks2022.HumanResources.Employee) >= 0.8
```
#### Results:

![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_06.JPG)

**[Back To The Top](#Overview-of-the-Section)**

#
## CORRELATED Subqueries

Correlated subqueries are subqueries that run once for each record in the main/outer query.

They then return a scalar output(i.e, a single value) for each of those records.

Correlated subqueries can be used in either the SELECT or WHERE clauses.

**Query example:**
```
SELECT 
 SalesOrderID
,OrderDate
,SubTotal
,TaxAmt
,Freight
,TotalDue
,MultiOrderCount = --correlated subquery
    (
        SELECT
        COUNT(*)
        FROM AdventureWorks2022.Sales.SalesOrderDetail B
        WHERE A.SalesOrderID = B.SalesOrderID
        AND B.OrderQty > 1
    )

FROM AdventureWorks2022.Sales.SalesOrderHeader A
```
**Results:**

![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_07.JPG)

### Exercise 1

Write a query that outputs all records from the Purchasing.PurchaseOrderHeader table. Include the following columns from the table:

- PurchaseOrderID
- VendorID
- OrderDate
- TotalDue

Add a derived column called **NonRejectedItems** which returns, for each purchase order ID in the query output, the number of line items from the Purchasing.

PurchaseOrderDetail table which did not have any rejections (i.e., RejectedQty = 0). Use a correlated subquery to do this.

```
SELECT 
 PurchaseOrderID
,VendorID
,OrderDate
,TotalDue
,NonRejectedItems = 
	(
		SELECT
		COUNT(*)
		FROM Purchasing.PurchaseOrderDetail B
		WHERE A.PurchaseOrderID = B.PurchaseOrderID
		AND B.RejectedQty = 0
	)

FROM AdventureWorks2022.Purchasing.PurchaseOrderHeader A
```
#### Results:

![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_08.JPG)

### Exercise 2

Modify your query to include a second derived field called **MostExpensiveItem.**

This field should return, for each purchase order ID, the UnitPrice of the most expensive item for that order in the Purchasing.PurchaseOrderDetail table.

Use a correlated subquery to do this as well.

_Hint:_ Think of the most appropriate aggregate function to use in the correlated subquery for this scenar

```
SELECT 
 PurchaseOrderID
,VendorID
,OrderDate
,TotalDue
,NonRejectedItems = 
	(
		SELECT
		COUNT(*)
		FROM Purchasing.PurchaseOrderDetail B
		WHERE A.PurchaseOrderID = B.PurchaseOrderID
		AND B.RejectedQty = 0
	)
	,MostExpensiveItem = 
	(
		SELECT
		MAX(B.UnitPrice)
		FROM Purchasing.PurchaseOrderDetail B
		WHERE A.PurchaseOrderID = B.PurchaseOrderID
	)

FROM AdventureWorks2022.Purchasing.PurchaseOrderHeader A

```
#### Results:

![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_09.JPG)

**[Back To The Top](#Overview-of-the-Section)**

#
## EXISTS and NOT EXISTS Subqueries

We use JOINs to add fields from different tables to our SELECT queries.

We sometimes also apply **criteria** to those fields inour WHERE clause.

But what if we ONLY need to bring in a field to use it in our WHERE, and don't need to see it in our query output?

That's where EXISTS comes in.

But if we can accomplis the same thing with a JOIN, why do we even need EXISTS?

For one-to-one relationships, there really isn't much of an advantage to using EXISTS.

However, EXISTS offers some powerful advantages when dealing with one-to-many relationships.

### Use EXISTS if...

- You want to apply criteria to fields from a secondary table but don't need to include those fields in your output.
- You want to apply criteria to fields from a secondary table while ensuring that multiple matches in the secondary table won't duplicate data from the primary tale in your output.
- You need to check a secondary table to make sure a match of some types does NOT exist.

**Query example: EXISTS**
```
SELECT
     A.SalesOrderID
    ,A.OrderDate
    ,A.TotalDue

FROM AdventureWorks2022.Sales.SalesOrderHeader A

WHERE EXISTS (
	SELECT
	1
	FROM AdventureWorks2022.Sales.SalesOrderDetail B
	WHERE A.SalesOrderID = B.SalesOrderID
		AND B.LineTotal > 10000
)

ORDER BY 1
```
**Results:**

![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_10.JPG)

**Query example: NOT EXISTS**
```
SELECT
     A.SalesOrderID
    ,A.OrderDate
    ,A.TotalDue

FROM AdventureWorks2022.Sales.SalesOrderHeader A

WHERE NOT EXISTS (
	SELECT
	1
	FROM AdventureWorks2022.Sales.SalesOrderDetail B
	WHERE A.SalesOrderID = B.SalesOrderID
		AND B.LineTotal > 10000
)

ORDER BY 1
```
**Results:**

![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_11.JPG)

### Exercise 1

Select all records from the Purchasing.PurchaseOrderHeader table such that there is at least one item in the order with an order quantity greater than 500. 

The individual items tied to an order can be found in the Purchasing.PurchaseOrderDetail table.

**Select the following columns:**

- PurchaseOrderID
- OrderDate
- SubTotal
- TaxAmt

Sort by purchase order ID.

```
SELECT
       A.PurchaseOrderID
      ,A.OrderDate
      ,A.SubTotal
	  ,A.TaxAmt

FROM AdventureWorks2022.Purchasing.PurchaseOrderHeader A

WHERE EXISTS (
	SELECT
	1
	FROM AdventureWorks2022.Purchasing.PurchaseOrderDetail B
	WHERE A.PurchaseOrderID = B.PurchaseOrderID
		AND B.OrderQty > 500
)

ORDER BY 1
```
#### Results:

![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_12.JPG)

### Exercise 2

Modify your query from Exercise 1 as follows:

Select all records from the Purchasing.PurchaseOrderHeader table such that there is at least one item in the order with an order quantity greater than 500, AND a unit price greater than $50.00.

Select ALL columns from the Purchasing.PurchaseOrderHeader table for display in your output.

Even if you have aliased this table to enable the use of a JOIN or EXISTS, you can still use the SELECT * shortcut to do this. Assuming you have aliased your table "A", simply use "SELECT A.*" to select all columns from that table.

```
SELECT
       A.*
FROM AdventureWorks2022.Purchasing.PurchaseOrderHeader A

WHERE EXISTS (
	SELECT
	1
	FROM AdventureWorks2022.Purchasing.PurchaseOrderDetail B
	WHERE A.PurchaseOrderID = B.PurchaseOrderID
		AND B.OrderQty > 500
		AND B.UnitPrice > 50
)

ORDER BY 1
```
#### Results:

![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_13.JPG)

### Exercise 3
Select all records from the Purchasing.PurchaseOrderHeader table such that NONE of the items within the order have a rejected quantity greater than 0.

Select ALL columns from the Purchasing.PurchaseOrderHeader table using the "SELECT *" shortcut.

```
SELECT
       A.*

FROM AdventureWorks2022.Purchasing.PurchaseOrderHeader A

WHERE NOT EXISTS (
	SELECT
	1
	FROM AdventureWorks2022.Purchasing.PurchaseOrderDetail B
	WHERE A.PurchaseOrderID = B.PurchaseOrderID
		AND B.RejectedQty > 0
)

ORDER BY 1
```
#### Results:

![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_14.JPG)

**[Back To The Top](#Overview-of-the-Section)**

#
## FOR XML PATH With STUFF Function

We get the requirement to display the data from the relational SQL table in various formats. 

Sometimes developers want to retrieve data in the XML format from the SQL tables holding relational data in regular data types. SQL Server supports XML data using the FOR XML clause. 

We can easily convert existing data into the XML format using this. We have the following modes available in the FOR XML clause. We can use the FOR XML clause to join or concatenate multiple columns into a single row output as well.

**CAST()** function converts a value (of any type) into a specified datatype.

**The STUFF()** function deletes a part of a string and then inserts another part into the string, starting at a specified position.

**Step 1**
```
SELECT
	COUNT(*)
	FROM AdventureWorks2022.Sales.SalesOrderDetail A
	WHERE A.SalesOrderID = 43659
```
![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_15.JPG)

**Step 2**
```
SELECT
	*
	FROM AdventureWorks2022.Sales.SalesOrderDetail A
	WHERE A.SalesOrderID = 43659
```
![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_16.JPG)

**Step 3**
```
SELECT
	LineTotal
	FROM AdventureWorks2022.Sales.SalesOrderDetail A
	WHERE A.SalesOrderID = 43659
```
![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_17.JPG)

**Step 4**
```
 SELECT
	LineTotal
	FROM AdventureWorks2022.Sales.SalesOrderDetail A
	WHERE A.SalesOrderID = 43659
	FOR XML PATH('')
```
![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_18.JPG)

**Step 5**
```
SELECT
  STUFF(
		  (
			  SELECT
			  ',' + CAST(CAST(LineTotal AS MONEY) AS VARCHAR)
			  FROM AdventureWorks2022.Sales.SalesOrderDetail A
			  WHERE A.SalesOrderID = 43659
			  FOR XML PATH('')
		  ),
		  1,1,'I''m stuffed! '
        )
```
![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_20.JPG)

**Step 6**
```
SELECT 
     SalesOrderID
    ,OrderDate
    ,SubTotal
    ,TaxAmt
    ,Freight
    ,TotalDue
	,LineTotals = 
	STUFF(
			(
				SELECT
				',' + CAST(CAST(LineTotal AS MONEY) AS VARCHAR)
				FROM AdventureWorks2022.Sales.SalesOrderDetail B
				WHERE A.SalesOrderID = B.SalesOrderID 
				FOR XML PATH('')
			  ),
			1,1,''
		  )

FROM AdventureWorks2022.Sales.SalesOrderHeader A
```
![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_21.JPG)


### Exercise 1

Create a query that displays all rows from the Production.ProductSubcategory table, and includes the following fields:

- The "Name" field from Production.ProductSubcategory, which should be aliased as "SubcategoryName".
- A derived field called "Products" which displays, for each Subcategory in Production.ProductSubcategory, a semicolon-separated list of all products from Production.Product contained within the given subcategory.

_Hint:_ 

Production.ProductSubcategory and Production.Product are related by the "ProductSubcategoryID" field.

```
SELECT 
     SubcategoryName = A.[Name]
	,Products =
		STUFF(
			(
				SELECT
				';' + B.Name
				FROM AdventureWorks2022.Production.Product B
				WHERE A.ProductSubcategoryID = B.ProductSubcategoryID
				FOR XML PATH('')
			),1,1,''
		)

  FROM AdventureWorks2022.Production.ProductSubcategory A
```

#### Results:

![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_22.JPG)

### Exercise 2

Modify the query from Exercise 1 such that only products with a ListPrice value greater than $50 are listed in the "Products" field.

**Hint:** Assuming you used a correlated subquery in Exercise 1, keep in mind that you can apply additional criteria to it, just as with any other correlated subquery.

**NOTE:** Your query should still include ALL product subcategories, but only list associated products greater than $50. But since there are certain product subcategories that don't have any associated products greater than $50, some rows in your query output may have a NULL value in the product field.

```
SELECT 
    SubcategoryName = A.[Name]
	,Products =
	STUFF(
		    (
				SELECT
					';' + B.Name

				FROM AdventureWorks2022.Production.Product B
				WHERE A.ProductSubcategoryID = B.ProductSubcategoryID
					AND B.ListPrice > 50
				FOR XML PATH('')
			),1,1,''
		)

 FROM AdventureWorks2022.Production.ProductSubcategory A
```
#### Results:

![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_23.JPG)

**[Back To The Top](#Overview-of-the-Section)**

#
## PIVOT - Part 1


#
## PIVOT - Part 2




### Exercise 1

Using PIVOT, write a query against the HumanResources.Employee table

That summarizes the average amount of vacation time for Sales Representatives, Buyers, and Janitors.

```
SELECT
*
FROM
(
SELECT 
JobTitle,
VacationHours

FROM AdventureWorks2022.HumanResources.Employee
) A

PIVOT(
AVG(VacationHours)
FOR JobTitle IN([Sales Representative],[Buyer],[Janitor])
) B
```

#### Results:

![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_24.JPG)

### Exercise 2

Modify your query from Exercise 1 such that the results are broken out by Gender. Alias the Gender field as "Employee Gender" in your output.

```
SELECT
[Employee Gender] = Gender,
[Sales Representative],
Buyer,
Janitor
FROM
(
SELECT 
JobTitle,
Gender,
VacationHours

FROM AdventureWorks2022.HumanResources.Employee
) A

PIVOT(
AVG(VacationHours)
FOR JobTitle IN([Sales Representative],[Buyer],[Janitor])
) B
```
#### Results:

![Section_02](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_02_E_25.JPG)

**[Back To The Top](#Overview-of-the-Section)**