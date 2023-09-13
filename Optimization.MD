<h1 align="center">Optimization</h1>

### Overview of the Section
* **[Optimizing With UPDATE](#Optimizing-With-UPDATE)**
    * **[Optimizing With UPDATE Exercise 1](#Optimizing-With-UPDATE-Exercise-1)**
	* **[Optimizing With UPDATE Exercise 2](#Optimizing-With-UPDATE-Exercise-2)**
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

**Starter Code - JOINs ainst or between very large tables**
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

### Optimizing With UPDATE Exercise 1
**Step 1 - Create filtered temp table of sales order header table WHERE year = 2012**
```
CREATE TABLE #Sales2012 
(
SalesOrderID INT,
OrderDate DATE
)
```
```
INSERT INTO #Sales2012
(
SalesOrderID,
OrderDate
)
```
```
SELECT
SalesOrderID,
OrderDate

FROM AdventureWorks2022.Sales.SalesOrderHeader

WHERE YEAR(OrderDate) = 2012
```
#### Results:
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_41.JPG)

**[Back To The Top](#Overview-of-the-Section)**

**Step 2 - Create new temp table after joining in SalesOrderDetail table**
```
CREATE TABLE #ProductsSold2012
(
SalesOrderID INT,
OrderDate DATE,
LineTotal MONEY,
ProductID INT,
ProductName VARCHAR(64),
ProductSubcategoryID INT,
ProductSubcategory VARCHAR(64),
ProductCategoryID INT,
ProductCategory VARCHAR(64)
)
```
```
INSERT INTO #ProductsSold2012
(
SalesOrderID,
OrderDate,
LineTotal,
ProductID
)

SELECT 
	   A.SalesOrderID
	  ,A.OrderDate
      ,B.LineTotal
      ,B.ProductID

FROM #Sales2012 A
	JOIN AdventureWorks2022.Sales.SalesOrderDetail B
		ON A.SalesOrderID = B.SalesOrderID
```
**Step 3 - Add product data with UPDATE**
```
UPDATE A
SET
ProductName = B.[Name],
ProductSubcategoryID = B.ProductSubcategoryID

FROM #ProductsSold2012 A
	JOIN AdventureWorks2022.Production.Product B
		ON A.ProductID = B.ProductID
```
**Step 4 - Add product subcategory with UPDATE**
```
UPDATE A
SET
ProductSubcategory= B.[Name],
ProductCategoryID = B.ProductCategoryID

FROM #ProductsSold2012 A
	JOIN AdventureWorks2022.Production.ProductSubcategory B
		ON A.ProductSubcategoryID = B.ProductSubcategoryID
```
**Step 5 - Add product subcategory with UPDATE**
```
UPDATE A
SET
ProductCategory= B.[Name]

FROM #ProductsSold2012 A
	JOIN AdventureWorks2022.Production.ProductCategory B
		ON A.ProductCategoryID = B.ProductCategoryID
```
```
SELECT * FROM #ProductsSold2012
```
#### Results:
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_42.JPG)

**[Back To The Top](#Overview-of-the-Section)**

### Optimizing With UPDATE Exercise 2
**Re-write an optimized version of the below query using temp tables and UPDATE statements:**
```
SELECT 
	   A.BusinessEntityID
      ,A.Title
      ,A.FirstName
      ,A.MiddleName
      ,A.LastName
	  ,B.PhoneNumber
	  ,PhoneNumberType = C.Name
	  ,D.EmailAddress

FROM AdventureWorks2022.Person.Person A
	LEFT JOIN AdventureWorks2022.Person.PersonPhone B
		ON A.BusinessEntityID = B.BusinessEntityID
	LEFT JOIN AdventureWorks2022.Person.PhoneNumberType C
		ON B.PhoneNumberTypeID = C.PhoneNumberTypeID
	LEFT JOIN AdventureWorks2022.Person.EmailAddress D
		ON A.BusinessEntityID = D.BusinessEntityID
```
**Rewrite:**
```
CREATE TABLE #PersonContactInfo
(
	   BusinessEntityID INT
      ,Title VARCHAR(8)
      ,FirstName VARCHAR(50)
      ,MiddleName VARCHAR(50)
      ,LastName VARCHAR(50)
	  ,PhoneNumber VARCHAR(25)
	  ,PhoneNumberTypeID VARCHAR(25)
	  ,PhoneNumberType VARCHAR(25)
	  ,EmailAddress VARCHAR(50)
)
```
```
INSERT INTO #PersonContactInfo
(
	   BusinessEntityID
      ,Title
      ,FirstName
      ,MiddleName
      ,LastName
)

SELECT
	   BusinessEntityID
      ,Title
      ,FirstName
      ,MiddleName
      ,LastName

FROM AdventureWorks2022.Person.Person
```
```
UPDATE A
SET
	PhoneNumber = B.PhoneNumber,
	PhoneNumberTypeID = B.PhoneNumberTypeID

FROM #PersonContactInfo A
	JOIN AdventureWorks2022.Person.PersonPhone B
		ON A.BusinessEntityID = B.BusinessEntityID
```
```
UPDATE A
SET	PhoneNumberType = B.Name

FROM #PersonContactInfo A
	JOIN AdventureWorks2022.Person.PhoneNumberType B
		ON A.PhoneNumberTypeID = B.PhoneNumberTypeID
```
```
UPDATE A
SET	EmailAddress = B.EmailAddress

FROM #PersonContactInfo A
	JOIN AdventureWorks2022.Person.EmailAddress B
		ON A.BusinessEntityID = B.BusinessEntityID
```
```
SELECT * FROM #PersonContactInfo
```
#### Results:
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_43.JPG)

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