<h1 align="center">Optimization</h1>

### Overview of the Section
* **[Optimizing With UPDATE](#Optimizing-With-UPDATE)**
    * **[Optimizing With UPDATE Exercise](#Optimizing-With-UPDATE-Exercise)**
* **[An Improved EXISTS With UPDATE](#An-Improved-EXISTS-With-UPDATE)**
    * **[An Improved EXISTS With UPDATE Exercise](#An-Improved-EXISTS-With-UPDATE-Exercise)**
* **[Introducing Indexes](#Introducing-Indexes)**
    * **[Optimizing With Indexes Example](#Optimizing-With-Indexes-Example)**
	* **[Optimizing With Indexes Exercise](#Optimizing-With-Indexes-Exercise)**
* **[Lookup Tables](#Lookup-Tables)**
	* **[Lookup Tables Exercise](#Lookup-Tables-Exercise)**
#
### Optimizing With UPDATE
What slows down queries the most?

_Generally speaking, **JOINs** against or between very large tables_

**What can we do about it?**

- Define a filtered ataset as early as possible in our process, so we can JOIN additional tables to a smaller core population.
- Avoid several JOINs in a single SELECT query, especially those involving large tables.
- Instead, use UPDATE statemenst to populate fields in a temp table, one source table in a time.
- Apply indexes to fields that will be used in JOINs. 


### Optimizing With UPDATE Exercise
**Step 1. Starter Code:**
```
SELECT 
	   A.SalesOrderID
	  ,A.OrderDate
      ,B.ProductID
      ,B.LineTotal
	  ,C.[Name] AS ProductName
	  ,D.[Name] AS ProductSubcategory
	  ,E.[Name] AS ProductCategory

FROM AdventureWorks2022.Sales.SalesOrderHeader A
	JOIN AdventureWorks2022.Sales.SalesOrderDetail B
		ON A.SalesOrderID = B.SalesOrderID
	JOIN AdventureWorks2022.Production.Product C
		ON B.ProductID = C.ProductID
	JOIN AdventureWorks2022.Production.ProductSubcategory D
		ON C.ProductSubcategoryID = D.ProductSubcategoryID
	JOIN AdventureWorks2022.Production.ProductCategory E
		ON D.ProductCategoryID = E.ProductCategoryID

WHERE YEAR(A.OrderDate) = 2012
```

#### Results:
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_40.JPG)

**[Back To The Top](#Overview-of-the-Section)**


### An Improved EXISTS With UPDATE


### An Improved EXISTS With UPDATE Exercise


#### Results:
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_35.JPG)

**[Back To The Top](#Overview-of-the-Section)**

### Introducing Indexes

**[Back To The Top](#Overview-of-the-Section)**

### Optimizing With Indexes Example

**[Back To The Top](#Overview-of-the-Section)**

### Optimizing With Indexes Exercise

**[Back To The Top](#Overview-of-the-Section)**


### Lookup Tables
**[Back To The Top](#Overview-of-the-Section)**
### Lookup Tables Exercise
**[Back To The Top](#Overview-of-the-Section)**