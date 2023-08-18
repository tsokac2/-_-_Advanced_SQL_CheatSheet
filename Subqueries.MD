<h1 align="center">Subqueries</h1>

## Overview of the Section
* **[Introducing Subqueries](#Introducing-Subqueries)**
* **[SCALAR Subqueries](#SCALAR-Subqueries)**


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
