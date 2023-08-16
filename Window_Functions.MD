<h1 align="center">Advanced Analysis With Window Functions</h1>

## Overview of the Section
* **[Window Functions With OVER](#Window-Functions-With-OVER)**
* **[PARTITION BY](#PARTITION-BY)**
* **[ROW_NUMBER](#ROW_NUMBER)**
* **[RANK and DENSE_RANK](#RANK-and-DENSE_RANK)**
* **[LEAD and LAG](#LEAD-and-LAG)**

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


#### Exercise 2

* Enhance your query from Exercise 1 by adding a derived column called

* "AvgPriceByCategory " that returns the average ListPrice for the product category in each given row.

#### Solution Exercise 2
```
SELECT 
ProductName = A.Name,
A.ListPrice,
ProductSubcategory = B.Name,
ProductCategory = C.Name,
AvgPriceByCategory = AVG(A.ListPrice) OVER(PARTITION BY C.Name)

FROM AdventureWorks2022.Production.Product A
JOIN AdventureWorks2022.Production.ProductSubcategory B
ON A.ProductSubcategoryID = B.ProductSubcategoryID
JOIN AdventureWorks2022.Production.ProductCategory C
ON B.ProductCategoryID = C.ProductCategoryID
```
#### Results:
![Section_01](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_01_E_06.JPG)

#### Exercise 3

* Enhance your query from Exercise 2 by adding a derived column called

* "AvgPriceByCategoryAndSubcategory" that returns the average ListPrice for the product category AND subcategory in each given row.w.

#### Solution Exercise 3
```
SELECT 
ProductName = A.Name,
A.ListPrice,
ProductSubcategory = B.Name,
ProductCategory = C.Name,
AvgPriceByCategory = AVG(A.ListPrice) OVER(PARTITION BY C.Name),
AvgPriceByCategoryAndSubcategory = AVG(A.ListPrice) OVER(PARTITION BY CName, B.Name)

FROM AdventureWorks2022.Production.Product A
JOIN AdventureWorks2022.Production.ProductSubcategory B
ON A.ProductSubcategoryID = B.ProductSubcategoryID
JOIN AdventureWorks2022.Production.ProductCategory C
ON B.ProductCategoryID = C.ProductCategoryID
```
#### Results:
![Section_01](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_01_E_07.JPG)

#### Exercise 4

* Enhance your query from Exercise 3 by adding a derived column called

* "ProductVsCategoryDelta" that returns the result of the following calculation:

* A product's list price, MINUS the average ListPrice for that product’s category.

#### Solution Exercise 4
```
SELECT 
ProductName = A.Name,
A.ListPrice,
ProductSubcategory = B.Name,
ProductCategory = C.Name,
AvgPriceByCategory = AVG(A.ListPrice) OVER(PARTITION BY C.Name),
AvgPriceByCategoryAndSubcategory = AVG(A.ListPrice) OVER(PARTITION BY CName, B.Name),
ProductVsCategoryDelta = A.ListPrice - AVG(A.ListPrice) OVER(PARTITION BY CName)

FROM AdventureWorks2022.Production.Product A
JOIN AdventureWorks2022.Production.ProductSubcategory B
ON A.ProductSubcategoryID = B.ProductSubcategoryID
JOIN AdventureWorks2022.Production.ProductCategory C
ON B.ProductCategoryID = C.ProductCategoryID
```
#### Results:
![Section_01](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_01_E_08.JPG)

#
## ROW_NUMBER

The ROW_NUMBER() is a window function that assigns a sequential integer to each row within the partition of a result set. The row number starts with 1 for the first row in each partition.

The following shows the syntax of the ROW_NUMBER() function:

```
ROW_NUMBER() OVER (
[PARTITION BY partition_expression, ... ]
ORDER BY sort_expression [ASC | DESC], ...
)
```

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

#### Exercise 2

* Enhance your query from Exercise 1 by adding a derived column called
* "Price Rank " that ranks all records in the dataset by ListPrice, in descending order. That is to say, the product with the most expensive price should have a rank of 1, and the product with the least expensive price should have a rank equal to the number of records in the dataset.

#### Solution Exercise 2
```
SELECT 
ProductName = A.Name,
A.ListPrice,
ProductSubcategory = B.Name,
ProductCategory = C.Name,
[Price Rank] = ROW_NUMBER() OVER(ORDER BY A.ListPrice DESC)


FROM AdventureWorks2022.Production.Product A
JOIN AdventureWorks2022.Production.ProductSubcategory B
ON A.ProductSubcategoryID = B.ProductSubcategoryID
JOIN AdventureWorks2022.Production.ProductCategory C
ON B.ProductCategoryID = C.ProductCategoryI
```
#### Results:
![Section_01](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_01_E_09.JPG)

#### Exercise 3

* Enhance your query from Exercise 2 by adding a derived column called
* "Category Price Rank" that ranks all products by ListPrice – within each category - in descending order. In other words, every product within a given category should be ranked relative to other products in the same category.

#### Solution Exercise 3
```
SELECT 
ProductName = A.Name,
A.ListPrice,
 ProductSubcategory = B.Name,
ProductCategory = C.Name,
[Price Rank] = ROW_NUMBER() OVER(ORDER BY A.ListPrice DESC),
[Category Price Rank] = ROW_NUMBER() OVER(PARTITION BY C.Name ORDER BY AListPrice DESC)

FROM AdventureWorks2022.Production.Product A
JOIN AdventureWorks2022.Production.ProductSubcategory B
ON A.ProductSubcategoryID = B.ProductSubcategoryID
JOIN AdventureWorks2022.Production.ProductCategory C
ON B.ProductCategoryID = C.ProductCategoryID
```
#### Results:
![Section_01](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_01_E_10.JPG)

#### Exercise 4

* Enhance your query from Exercise 3 by adding a derived column called

* "Top 5 Price In Category" that returns the string “Yes” if a product has one of the top 5 list prices in its product category, and “No” if it does not. You can try incorporating your logic from Exercise 3 into a CASE statement to make this work.

#### Solution Exercise 4
```
SELECT 
ProductName = A.Name,
A.ListPrice,
ProductSubcategory = B.Name,
ProductCategory = C.Name,
[Price Rank] = ROW_NUMBER() OVER(ORDER BY A.ListPrice DESC),
[Category Price Rank] = ROW_NUMBER() OVER(PARTITION BY C.Name ORDER BY AListPrice DESC),
[Top 5 Price In Category] = 
CASE 
WHEN ROW_NUMBER() OVER(PARTITION BY C.Name ORDER BY A.ListPrice DESC<= 5 THEN 'Yes'
ELSE 'No'
END

FROM AdventureWorks2022.Production.Product A
JOIN AdventureWorks2022.Production.ProductSubcategory B
ON A.ProductSubcategoryID = B.ProductSubcategoryID
JOIN AdventureWorks2022.Production.ProductCategory C
ON B.ProductCategoryID = C.ProductCategoryID
```
#### Results:
![Section_01](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_01_E_11.JPG)

#
## RANK and DENSE_RANK

**What are the RANK and DENSE_RANK SQL Functions?**

The RANK and DENSE_RANK functions allow you to calculate the rank, or the position in a list, of a value in a group of values. It returns a number value.

**What’s the Difference between RANK and DENSE_RANK and ROW_NUMBER?**

The RANK and DENSE_RANK functions are slightly different from each other as well as the ROW_NUMBER function:

RANK numbers are skipped so there may be a gap in rankings, and may not be unique.
DENSE_RANK numbers are not skipped so there will not be a gap in rankings, and may not be unique.
ROW_NUMBER is generated once for each row so there are no duplicates or gaps.
For example:

RANK: a list of results could use the RANK function and show values of 1, 2, 2, 4, and 5. The number 3 is skipped because the rank of 2 is tied.
DENSE_RANK: a list of results could use the DENSE_RANK function and show values of 1, 2, 2, 3, and 4. The number 3 is still used, even if rank of 2 is tied.
ROW_NUMBER: a list of results could use the ROW_NUMBER function and show values of 1, 2, 3, 4, and 5. All of the numbers are unique.

**Syntax**

The syntax for the SQL RANK function is:

```RANK () OVER ( [query_partition_clause] order_by_clause )```

The syntax for the SQL DENSE_RANK function is similar:

```DENSE_RANK () OVER ( [query_partition_clause] order_by_clause )```

You can also use these functions as an aggregate function. You provide an input value, and the function returns the rank of where that value would be.

Here’s the syntax for both functions:
```
RANK (search_value) WITHIN GROUP ( [query_partition_clause] order_by_clause );
DENSE_RANK (search_value) WITHIN GROUP ( [query_partition_clause] order_by_clause );
```

#### Exercise 1

Using your solution query to Exercise 4 from the ROW_NUMBER exercises as a staring point, add a derived column called “Category Price Rank With Rank” that uses the RANK function to rank all products by ListPrice – within each category - in descending order. 

Observe the differences between the “Category Price Rank” and “Category Price Rank With Rank” fields.

#### Solution Exercise 1
```
SELECT 
ProductName = A.Name,
A.ListPrice,
ProductSubcategory = B.Name,
ProductCategory = C.Name,
[Price Rank] = ROW_NUMBER() OVER(ORDER BY A.ListPrice DESC),
[Category Price Rank] = ROW_NUMBER() OVER(PARTITION BY C.Name ORDER BY AListPrice DESC),
[Category Price Rank With Rank] = RANK() OVER(PARTITION BY C.Name ORDER BYA.ListPrice DESC),
[Top 5 Price In Category] = 
CASE 
WHEN ROW_NUMBER() OVER(PARTITION BY C.Name ORDER BY A.ListPrice DESC) <= 5 THEN 'Yes'
ELSE 'No'
END

FROM AdventureWorks2022.Production.Product A
JOIN AdventureWorks2022.Production.ProductSubcategory B
ON A.ProductSubcategoryID = B.ProductSubcategoryID
JOIN AdventureWorks2022.Production.ProductCategory C
ON B.ProductCategoryID = C.ProductCategoryID
```
#### Results:
![Section_01](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_01_E_12.JPG)

#### Exercise 2

Modify your query from Exercise 2 by adding a derived column called "Category Price Rank With Dense Rank" that that uses the DENSE_RANK function to rank all products by ListPrice – within each category - in descending order.

Observe the differences among the “Category Price Rank”, “Category Price Rank With Rank”, and “Category Price Rank With Dense Rank” fields.

#### Solution Exercise 2
```
SELECT 
ProductName = A.Name,
A.ListPrice,
ProductSubcategory = B.Name,
ProductCategory = C.Name,
[Price Rank] = ROW_NUMBER() OVER(ORDER BY A.ListPrice DESC),
[Category Price Rank] = ROW_NUMBER() OVER(PARTITION BY C.Name ORDER BY AListPrice DESC),
[Category Price Rank With Rank] = RANK() OVER(PARTITION BY C.Name ORDER BYA.ListPrice DESC),
[Category Price Rank With Dense Rank] = DENSE_RANK() OVER(PARTITION BY CName ORDER BY A.ListPrice DESC),
[Top 5 Price In Category] = 
CASE 
WHEN ROW_NUMBER() OVER(PARTITION BY C.Name ORDER BY A.ListPrice DESC<= 5 THEN 'Yes'
ELSE 'No'
END

FROM AdventureWorks2022.Production.Product A
JOIN AdventureWorks2022.Production.ProductSubcategory B
ON A.ProductSubcategoryID = B.ProductSubcategoryID
JOIN AdventureWorks2022.Production.ProductCategory C
ON B.ProductCategoryID = C.ProductCategoryID
```
#### Results:
![Section_01](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_01_E_13.JPG)

#### Exercise 3

Examine the code you wrote to define the “Top 5 Price In Category” field back in the ROW_NUMBER exercises. 

Now that you understand the differences among ROW_NUMBER, RANK, and DENSE_RANK, consider which of these functions would be most appropriate to return a true top 5 products by price, assuming we want to see the top 5 distinct prices AND we want “ties” (by price) to all share the same rank.

#### Solution Exercise 3
```
SELECT 
ProductName = A.Name,
A.ListPrice,
ProductSubcategory = B.Name,
ProductCategory = C.Name,
[Price Rank] = ROW_NUMBER() OVER(ORDER BY A.ListPrice DESC),
[Category Price Rank] = ROW_NUMBER() OVER(PARTITION BY C.Name ORDER BY AListPrice DESC),
[Category Price Rank With Rank] = RANK() OVER(PARTITION BY C.Name ORDER BYA.ListPrice DESC),
[Category Price Rank With Dense Rank] = DENSE_RANK() OVER(PARTITION BY CName ORDER BY A.ListPrice DESC),
[Top 5 Price In Category] = 
CASE 
WHEN DENSE_RANK() OVER(PARTITION BY C.Name ORDER BY A.ListPrice DESC<= 5 THEN 'Yes'
ELSE 'No'
END

FROM AdventureWorks2022.Production.Product A
JOIN AdventureWorks2022.Production.ProductSubcategory B
ON A.ProductSubcategoryID = B.ProductSubcategoryID
JOIN AdventureWorks2022.Production.ProductCategory C
ON B.ProductCategoryID = C.ProductCategoryID
```
#### Results:
![Section_01](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_01_E_14.JPG)

#
## LEAD and LAG

**LEAD** and **LAG** let us grab values from subsequent or previous records relative to the position of the "current" record in our data.

They can be useful any time we want to compare a value in a given column to the next or previous value in the same column but side by side, in the same row.

This is a very common problem in real-world analytics scenarios.

#### Exercise 1

**Create a query with the following columns:**

* “PurchaseOrderID” from the Purchasing.PurchaseOrderHeader table
* “OrderDate” from the Purchasing.PurchaseOrderHeader table
* “TotalDue” from the Purchasing.PurchaseOrderHeader table
* “Name” from the Purchasing.Vendor table, which can be aliased as “VendorName”*

Join Purchasing.Vendor to Purchasing.PurchaseOrderHeader on BusinessEntityID = VendorID

**Apply the following criteria to the query:**

* Order must have taken place on or after 2013
* TotalDue must be greater than $500

#### Solution Exercise 1
```
SELECT 
PurchaseOrderID
,OrderDate
,TotalDue
,VendorName = B.Name

FROM AdventureWorks2022.Purchasing.PurchaseOrderHeader A
JOIN AdventureWorks2022.Purchasing.Vendor B
ON A.VendorID = B.BusinessEntityID

WHERE YEAR(A.OrderDate) >= 2013
AND A.TotalDue > 500
```
#### Results:
![Section_01](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_01_E_15.JPG)

#### Exercise 2

**Modify your query from Exercise 1 by adding a derived column called**

"PrevOrderFromVendorAmt", that returns the “previous” TotalDue value (relative to the current row) within the group of all orders with the same vendor ID. We are defining “previous” based on order date.


#### Solution Exercise 2
```
SELECT 
PurchaseOrderID
,OrderDate
,TotalDue
,VendorName = B.Name
,PrevOrderFromVendorAmt = LAG(A.TotalDue) OVER(PARTITION BY A.VendorIORDER BY A.OrderDate)

FROM AdventureWorks2022.Purchasing.PurchaseOrderHeader A
JOIN AdventureWorks2022.Purchasing.Vendor B
ON A.VendorID = B.BusinessEntityID

WHERE YEAR(A.OrderDate) >= 2013
AND A.TotalDue > 500

ORDER BY 
A.VendorID,
A.OrderDate
```
#### Results:
![Section_01](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_01_E_16.JPG)

#### Exercise 3

**Modify your query from Exercise 2 by adding a derived column called**

"NextOrderByEmployeeVendor", that returns the “next” vendor name (the “name” field from Purchasing.Vendor) within the group of all orders that have the same EmployeeID value in Purchasing.PurchaseOrderHeader. 

Similar to the last exercise, we are defining “next” based on order date.

#### Solution Exercise 3
```
SELECT 
PurchaseOrderID
,OrderDate
,TotalDue
,VendorName = B.Name
,PrevOrderFromVendorAmt = LAG(A.TotalDue) OVER(PARTITION BY A.VendorIORDER BY A.OrderDate)
,NextOrderByEmployeeVendor = LEAD(B.Name) OVER(PARTITION BY EmployeeID ORDER BY A.OrderDate)

FROM AdventureWorks2022.Purchasing.PurchaseOrderHeader A
JOIN AdventureWorks2022.Purchasing.Vendor B
ON A.VendorID = B.BusinessEntityID

WHERE YEAR(A.OrderDate) >= 2013
AND A.TotalDue > 500

ORDER BY 
A.EmployeeID,
A.OrderDate
```
#### Results:
![Section_01](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_01_E_17.JPG)

#### Exercise 4

Modify your query from Exercise 3 by adding a derived column called "Next2OrderByEmployeeVendor" that returns, within the group of all orders that have the same EmployeeID, the vendor name offset TWO orders into the “future” relative to the order in the current row. 

The code should be very similar to Exercise 3, but with an extra argument passed to the Window Function used.

#### Solution Exercise 4
```
SELECT 
PurchaseOrderID
,OrderDate
,TotalDue
,VendorName = B.Name
,PrevOrderFromVendorAmt = LAG(A.TotalDue) OVER(PARTITION BY A.VendorIORDER BY A.OrderDate)
,NextOrderByEmployeeVendor = LEAD(B.Name) OVER(PARTITION BY EmployeeID ORDER BY A.OrderDate)
,Next2OrderByEmployeeVendor = LEAD(B.Name,2) OVER(PARTITION BY EmployeeID ORDER BY A.OrderDate)

FROM AdventureWorks2022.Purchasing.PurchaseOrderHeader A
JOIN AdventureWorks2022.Purchasing.Vendor B
ON A.VendorID = B.BusinessEntityID

WHERE YEAR(A.OrderDate) >= 2013
AND A.TotalDue > 500

ORDER BY 
A.EmployeeID,
A.OrderDa
```
#### Results:
![Section_01](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_01_E_18.JPG)