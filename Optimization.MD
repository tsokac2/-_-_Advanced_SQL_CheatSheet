<h1 align="center">Optimization</h1>

### Overview of the Section
* **[Optimizing With UPDATE](#Optimizing-With-UPDATE)**
    * **[Optimizing With UPDATE Exercise 1](#Optimizing-With-UPDATE-Exercise-1)**
	* **[Optimizing With UPDATE Exercise 2](#Optimizing-With-UPDATE-Exercise-2)**
* **[An Improved EXISTS With UPDATE](#An-Improved-EXISTS-With-UPDATE)**
    * **[An Improved EXISTS With UPDATE Exercise](#An-Improved-EXISTS-With-UPDATE-Exercise)**
* **[When shouldyou use what technique](#When-should-you-use-what-technique)**
* **[Introducing Indexes](#Introducing-Indexes)**
	* **[What are indexes](#What-are-indexes)**
	* **[Clustered Indexes](#Clustered-Indexes)**
	* **[Clustered-Indexes Strategies](#Clustered-Indexes-Strategies)**
	* **[Non-clustered-Indexes](#Non-clustered-Indexes)**
	* **[Non-Clustered Indexes Strategies](#Non-Clustered-Indexes-Strategies)**
	* **[Indexes General Approach](#Indexes-General-Approach)**
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

EXISTS strengths and weaknesses

- EXISTS lets you check for matching records from the many sides of a relationship, without resulting in duplicated data from one side.
- EXISTS works fine in this scenario as long as you don't need any additional information about the match, other than that it exists.
- However, if you need to see any data points about the match, UPDATE can be a superior alternative to EXISTS.

### An Improved EXISTS With UPDATE Exercise

Re-write the query, using temp tables and UPDATEs instead of EXISTS.

In addition to the three columns in the original query, you should also include a fourth column called "RejectedQty", which has one value for rejected quantity from the Purchasing.PurchaseOrderDetail table.

**Starter Code Query**
```
SELECT
       A.PurchaseOrderID,
	   A.OrderDate,
	   A.TotalDue

FROM AdventureWorks2022.Purchasing.PurchaseOrderHeader A

WHERE EXISTS (
	SELECT
	1
	FROM AdventureWorks2022.Purchasing.PurchaseOrderDetail B
	WHERE A.PurchaseOrderID = B.PurchaseOrderID
		AND B.RejectedQty > 5
)

ORDER BY 1
```

**Re-write Step 1**
```
CREATE TABLE #Purchases
(
PurchaseOrderID INT,
OrderDate DATE,
TotalDue MONEY,
RejectedQty INT
)
```
**Re-write Step 2**
```
INSERT INTO #Purchases
(
PurchaseOrderID,
OrderDate,
TotalDue
)

SELECT
PurchaseOrderID,
OrderDate,
TotalDue

FROM AdventureWorks2022.Purchasing.PurchaseOrderHeader
```
**Re-write Step 3**
```
UPDATE A
	SET RejectedQty = B.RejectedQty

FROM #Purchases A
	JOIN AdventureWorks2022.Purchasing.PurchaseOrderDetail B
		ON A.PurchaseOrderID = B.PurchaseOrderID
WHERE B.RejectedQty > 5
```
**Re-write Step 4**
```
SELECT * FROM #Purchases WHERE RejectedQty IS NOT NULL
```
#### Results:
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_44.JPG)

**[Back To The Top](#Overview-of-the-Section)**

### When should you use what technique
- If you need to see all matches from the many sides of the relationship, use a JOIN.
- If you don't want to see all matches from the many sides AND don't care to see any information about those matches(other than their existence), EXISTS is fine.
- If you don't want to see all matches from the many sides, but would like some information about a (any) match that was returned, use UPDATE.

**[Back To The Top](#Overview-of-the-Section)**

### Introducing Indexes

#### What are indexes
- Indexes are database objects that can make queries against your tables faster. 
- They do this by sorting the data in the fields they apply to – either in the table itself or in a separate data structure. 
- This sorting allows the database engine to locate records within a table without having to search through the table row-by-row. 
- There are two types of indexes – clustered and non-clustered.

#### Clustered Indexes
- The rows of a table with a clustered index are physically sorted based on the field or fields the index is applied to.
- A table with a primary key is given a clustered index (based on the primary key field) by default.
- Most tables should have at least a clustered index, as queries against tables with a clustered index generally tend to be faster. 
- A table may only have one clustered index.

#### Clustered Indexes Strategies
- Apply a clustered index to whatever field – or fields - are most likely to be used in a join against the table.
- Ideally, this field (or combination of fields) should also be the one that most uniquely defines a record in the table.
- Whatever field would be a good candidate for a primary key of a table, is usually also a good candidate for a clustered index.

#### Non-clustered Indexes
- A table may have many non-clustered indexes.
- Non-clustered indexes do not physically sort the data in a table like clustered indexes do. 
- The sorted order of the field or fields non-clustered indexes apply to is stored in an external data structure, which works like a kind of phone book.

#### Non-Clustered Indexes Strategies
- If you will be joining your table on fields besides the one “covered” by the clustered index, consider non-clustered indexes on those fields.
- You can add as many non-clustered indexes as you like, but should be judicious in doing so.
- Fields covered by a non-clustered index should still have a high level of uniqueness.

#### Indexes: General Approach
- It’s how our table is utilized in joins that should drive our use and design of indexes.
- You should generally add a clustered index first, and then layer in non-clustered indexes as needed to “cover” additional fields used in joins against our table. 
Indexes take up memory in the database, so only add them when they are really needed. 
- They also make inserts to tables take longer, so you should generally add indexes after data has been inserted into the table.

**[Back To The Top](#Overview-of-the-Section)**

### Optimizing With Indexes Example

**[Back To The Top](#Overview-of-the-Section)**

### Optimizing With Indexes Exercise

**[Back To The Top](#Overview-of-the-Section)**


### Lookup Tables
**[Back To The Top](#Overview-of-the-Section)**
### Lookup Tables Exercise
**[Back To The Top](#Overview-of-the-Section)**