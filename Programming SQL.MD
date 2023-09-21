<h1 align="center">Programming SQL</h1>

### Overview of the Section
* **[Variables](#Variables)**
    * **[Variables Exercise 1](#Variables-Exercise-1)**
	* **[Variables Exercise 2](#Variables-Exercise-2)**
* **[Introducing User Defined Functions](#Introducing-User-Defined-Functions)**
* **[Making Functions Flexible With Parameters](#Making-Functions-Flexible-With-Parameters)**
	* **[User Defined Functions Exercise 1](#User-Defined-Functions-Exercise-1)**
	* **[User Defined Functions Exercise 2](#User-Defined-Functions-Exercise-2)**
* **[Stored Procedures](#Stored-Procedures)**
	* **[Stored Procedures Exercise](#Stored-Procedures-Exercise)**
* **[Control Flow With IF Statements](#Control-Flow-With-IF-Statements)**
	* **[Control Flow With IF Statements Exercise](#Control-Flow-With-IF-Statements-Exercise)**
* **[Using Multiple IF Statements](#Using-Multiple-IF-Statements)**
	* **[Using Multiple IF Statements Exercise](#Using-Multiple-IF-Statements-Exercise)**
* **[Dynamic SQL Part 1](#Dynamic-SQL-Part-1)**
	* **[Dynamic SQL Exercise 1](#Dynamic-SQL-Exercise-1)**
	* **[Dynamic SQL Exercise 2](#Dynamic-SQL-Exercise-2)**

#
### Variables
- A variable is a named placeholder for a value or set of values.
- A variable only needs to be defined once but can be reused many times.
- This allows us to consolidate complex logic into a variable, and then refer to the variable as many times as we need without repeating the logic. 

### Variables Exercise 1

Refactor the provided code to utilize variables instead of embedded scalar subqueries.

**Scalar subquery code:**
```
SELECT
	   BusinessEntityID
      ,JobTitle
      ,VacationHours
	  ,MaxVacationHours = (SELECT MAX(VacationHours) FROM AdventureWorks2022.HumanResources.Employee)
	  ,PercentOfMaxVacationHours = (VacationHours * 1.0) / (SELECT MAX(VacationHours) FROM AdventureWorks2022.HumanResources.Employee)

FROM AdventureWorks2022.HumanResources.Employee

WHERE (VacationHours * 1.0) / (SELECT MAX(VacationHours) FROM AdventureWorks2022.HumanResources.Employee) >= 0.8
```
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_52.JPG)

**Refactored Solution**
```
DECLARE @MaxVacation FLOAT = (SELECT MAX(VacationHours) FROM AdventureWorks2022.HumanResources.Employee)

SELECT
	   [BusinessEntityID]
      ,[JobTitle]
      ,[VacationHours]
	  ,MaxVacationHours = @MaxVacation
	  ,PercentOfMaxVacationHours = VacationHours / @MaxVacation

FROM AdventureWorks2022.HumanResources.Employee

WHERE VacationHours / @MaxVacation >= 0.8
```
#### Results:
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_53.JPG)

**[Back To The Top](#Overview-of-the-Section)**

### Variables Exercise 2

Let's say your company pays once per month, on the 15th.
If it's already the 15th of the current month (or later), the previous pay period will run from the 15th of the previous month, to the 14th of the current month.

If on the other hand it's not yet the 15th of the current month, the previous pay period will run from the
15th two months ago to the 14th on the previous month.

Set up variables defining the beginning and end of the previous pay period in this scenario. Select the variables to ensure they are working properly.

_Hint:_ In addition to incorporating date logic, you will probably also need to use CASE statements in one of your variable definitions.
```
DECLARE @Today DATE = CAST(GETDATE() AS DATE)
SELECT @Today

DECLARE @Current14 DATE = DATEFROMPARTS(YEAR(@Today),MONTH(@Today),14)

DECLARE @PayPeriodEnd DATE = 
	CASE
		WHEN DAY(@Today) < 15 THEN DATEADD(MONTH,-1,@Current14)
		ELSE @Current14
	END

DECLARE @PayPeriodStart DATE = DATEADD(DAY,1,DATEADD(MONTH,-1,@PayPeriodEnd))


SELECT @PayPeriodStart
SELECT @PayPeriodEnd
```
#### Results:
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_54.JPG)

**[Back To The Top](#Overview-of-the-Section)**

#
### Introducing User Defined Functions

SQL Server allows us to create our methods called user defined functions. 

For example, if we want to perform some complex calculations, then we can place them in a separate method and store them in the database. Whenever we need the calculation, we can call it. 

**Code to create user defined function:**
```
CREATE FUNCTION dbo.ufnCurrentDate()

RETURNS DATE

AS

BEGIN

	RETURN CAST(GETDATE() AS DATE)

END
```

**Query that calls user defined function**
```
SELECT
	   SalesOrderID
      ,OrderDate
      ,DueDate
      ,ShipDate
	  ,Today = dbo.ufnCurrentDate()

FROM AdventureWorks2022.Sales.SalesOrderHeader A

WHERE YEAR(A.OrderDate) = 2011
```
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_55.JPG)

**[Back To The Top](#Overview-of-the-Section)**

#
### Making Functions Flexible With Parameters
Procedure:
- Specify a name for the function.
- Specify a name and data type for each input parameter.
- Specify the RETURNS keyword and the data type of the scalar return value.
- Specify the BEGIN keyword to introduce the function-body.
- Specify the function body.
- Specify the END keyword.

**Correlated Subquery Example:**
```
SELECT
	   SalesOrderID
      ,OrderDate
      ,DueDate
      ,ShipDate
	  ,ElapsedBusinessDays = (
		SELECT
		COUNT(*)
		FROM AdventureWorks2022.dbo.Calendar B
		WHERE B.DateValue BETWEEN A.OrderDate AND A.ShipDate
			AND B.WeekendFlag = 0
			AND B.HolidayFlag = 0
	  ) - 1

FROM AdventureWorks2022.Sales.SalesOrderHeader A

WHERE YEAR(A.OrderDate) = 2011
```
**Rewriting as a fucntion, with variables:**
```
CREATE FUNCTION dbo.ufnElapsedBusinessDays(@StartDate DATE, @EndDate DATE)

RETURNS INT
AS  
BEGIN
	RETURN 
		(
			SELECT
				COUNT(*)
			FROM AdventureWorks2022.dbo.Calendar

			WHERE DateValue BETWEEN @StartDate AND @EndDate
				AND WeekendFlag = 0
				AND HolidayFlag = 0
		)	- 1
END
```
**Using the function in a query**
```
SELECT
	   SalesOrderID
      ,OrderDate
      ,DueDate
      ,ShipDate
	  ,ElapsedBusinessDays = dbo.ufnElapsedBusinessDays(OrderDate,ShipDate)

FROM AdventureWorks2022.Sales.SalesOrderHeader

WHERE YEAR(OrderDate) = 2011
```
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_56.JPG)

### User Defined Functions Exercise 1
Create a user-defined function that returns the percent that one number is of another.

For example, if the first argument is 8 and the second argument is 10, the function should return the string "80.00%".

The function should solve the "integer division" problem by allowing you to divide an integer by another integer, and yet get an accurate decimal result.

_Hints:_

Remember that you can implicitly convert an integer to a decimal by multiplying it by 1.0.

You can format a decimal (say, 0.1) as a percent (10%) with the following code: FORMAT(0.1, 'P').

Remember that the the return value of the function should be a text string.

```
CREATE FUNCTION dbo.ufnIntegerPercent(@Numerator INT, @Denominator INT)
RETURNS VARCHAR(8)
AS   
BEGIN
	DECLARE @Decimal FLOAT  = (@Numerator * 1.0) / @Denominator

	RETURN FORMAT(@Decimal, 'P')
END
```
### User Defined Functions Exercise 2
Store the maximum amount of vacation time for any individual employee in a variable.

Then create a query that displays all rows and the following columns from the AdventureWorks2019.HumanResources.Employee table:

- BusinessEntityID
- JobTitle
- VacationHours

Then add a derived field called "PercentOfMaxVacation", which returns the percent an individual employees' vacation hours are of the maximum vacation hours for any employee.

For example, the record for the employee with the most vacation hours should have a value of 100.00%, in this column.

The derived field should make use of your user-defined function from the previous exercise, as well as your variable that stored the maximum vacation hours for any employee.

```
DECLARE @MaxVacationHours INT = (SELECT MAX(VacationHours) FROM AdventureWorks2022.HumanResources.Employee)

SELECT
	BusinessEntityID,
	JobTitle,
	VacationHours,
	PercentOfMaxVacation = dbo.ufnIntegerPercent(VacationHours, @MaxVacationHours)

FROM AdventureWorks2022.HumanResources.Employee
```
#### Results:
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_57.JPG)

**[Back To The Top](#Overview-of-the-Section)**

#
### Stored Procedures
A stored procedure is a prepared SQL code that you can save, so the code can be reused over and over again. 

So if you have an SQL query that you write over and over again, save it as a stored procedure, and then just call it to execute it.

### Stored Procedures Exercise
Create a stored procedure called "OrdersAboveThreshold" that pulls in all sales orders with a total due amount above a threshold specified in a parameter called "@Threshold". The value for threshold will be supplied by the caller of the stored procedure.

The proc should have two other parameters: "@StartYear" and "@EndYear" (both INT data types), also specified by the called of the procedure. All order dates returned by the proc should fall between these two years.

**Create the procedure:**
```
CREATE PROCEDURE dbo.OrdersAboveThreshold(@Threshold MONEY, @StartYear INT, @EndYear INT)

AS

BEGIN
	SELECT 
		A.SalesOrderID,
		A.OrderDate,
		A.TotalDue

	FROM  AdventureWorks2022.Sales.SalesOrderHeader A
		JOIN AdventureWorks2022.dbo.Calendar B
			ON A.OrderDate = B.DateValue

	WHERE A.TotalDue >= @Threshold
		AND B.YearNumber BETWEEN @StartYear AND @EndYear
END
```

**Execute the procedure:**
```
EXEC dbo.OrdersAboveThreshold 10000, 2011, 2013
```
#### Results:
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_58.JPG)

**[Back To The Top](#Overview-of-the-Section)**

#
### Control Flow With IF Statements

If statements (also called conditionals or selection) change the flow of control through the program so that some code is only run when something is true. In an if statement, if the condition is true then the next statement or a block of statements will execute.

**IF Statement - basic example**
```
DECLARE @MyInput INT = 1

IF @MyInput > 1
	BEGIN
		SELECT 'Hello World'
	END
ELSE
	BEGIN
		SELECT 'Farewell For Now!'
	END
```
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_59.JPG)

#### IF Statement with Stored procedure example

**Step 1 Create Basic (non-dynamic) stored procedure**
```
CREATE PROCEDURE dbo.OrdersReport

AS

BEGIN
	SELECT
		*
	FROM (
		SELECT 
			ProductName = B.[Name],
			LineTotalSum = SUM(A.LineTotal),
			LineTotalSumRank = DENSE_RANK() OVER(ORDER BY SUM(A.LineTotal) DESC)

		FROM AdventureWorks2022.Sales.SalesOrderDetail A
			JOIN AdventureWorks2022.Production.Product B
				ON A.ProductID = B.ProductID

		GROUP BY
			B.[Name]
		) X

	WHERE LineTotalSumRank <= 10
END
```
**Step 2 Execute stored procedure**
```
EXEC dbo.OrdersReport
```
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_60.JPG)

**Step 3 Alter _dbo.OrdersReport_ stored procedure example with IF Statements**
```
ALTER PROCEDURE dbo.OrdersReport(@TopN INT, @OrderType INT)

AS

BEGIN

	IF @OrderType = 1
		BEGIN
			SELECT
				*
			FROM (
				SELECT 
					ProductName = B.[Name],
					LineTotalSum = SUM(A.LineTotal),
					LineTotalSumRank = DENSE_RANK() OVER(ORDER BY SUM(A.LineTotal) DESC)

				FROM AdventureWorks2022.Sales.SalesOrderDetail A
					JOIN AdventureWorks2022.Production.Product B
						ON A.ProductID = B.ProductID

				GROUP BY
					B.[Name]
				) X

			WHERE LineTotalSumRank <= @TopN
		END
	ELSE
		BEGIN
			SELECT
				*
			FROM (
				SELECT 
					ProductName = B.[Name],
					LineTotalSum = SUM(A.LineTotal),
					LineTotalSumRank = DENSE_RANK() OVER(ORDER BY SUM(A.LineTotal) DESC)

				FROM AdventureWorks2022.Purchasing.PurchaseOrderDetail A
					JOIN AdventureWorks2022.Production.Product B
						ON A.ProductID = B.ProductID

				GROUP BY
					B.[Name]
				) X

			WHERE LineTotalSumRank <= @TopN
		END
END
```
**Step 4 Calling the modified stored procedure**
```
EXEC dbo.OrdersReport 20,1
```
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_61.JPG)

**[Back To The Top](#Overview-of-the-Section)**

### Control Flow With IF Statements Exercise

Modify the stored procedure you created for the stored procedures exercise (dbo.OrdersAboveThreshold) to include an additional parameter called "@OrderType" (data type INT).

If the user supplies a value of 1 to this parameter, your modified proc should return the same output as previously.

If however the user supplies a value of 2, your proc should return purchase orders instead of sales orders.

Use IF/ELSE blocks to accomplish this.

```
ALTER PROCEDURE dbo.OrdersAboveThreshold(@Threshold MONEY, @StartYear INT, @EndYear INT, @OrderType INT)

AS

BEGIN
	IF @OrderType = 1
		BEGIN
			SELECT 
				A.SalesOrderID,
				A.OrderDate,
				A.TotalDue

			FROM  AdventureWorks2022.Sales.SalesOrderHeader A
				JOIN AdventureWorks2022.dbo.Calendar B
					ON A.OrderDate = B.DateValue

			WHERE A.TotalDue >= @Threshold
				AND B.YearNumber BETWEEN @StartYear AND @EndYear

			ORDER BY A.TotalDue DESC
		END

	ELSE
			BEGIN
				SELECT 
					A.PurchaseOrderID,
					A.OrderDate,
					A.TotalDue

				FROM  AdventureWorks2022.Purchasing.PurchaseOrderHeader A
					JOIN AdventureWorks2022.dbo.Calendar B
						ON A.OrderDate = B.DateValue

				WHERE A.TotalDue >= @Threshold
					AND B.YearNumber BETWEEN @StartYear AND @EndYear

				ORDER BY A.TotalDue DESC
			END
END
```

**Call modified procedure**
```
EXEC dbo.OrdersAboveThreshold 10000, 2011, 2013, 1
```
#### Results:
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_62.JPG)

**[Back To The Top](#Overview-of-the-Section)**
#
### Using Multiple IF Statements
Multiple statements that are part of an IF block must be enclosed within BEGIN and END statements. Always try to indent your code, particularly if you have nested IF statements.

**Multiple IF statement example**
```
ALTER PROCEDURE dbo.OrdersReport(@TopN INT, @OrderType INT)

AS

BEGIN

	IF @OrderType = 1
		BEGIN
			SELECT
				*
			FROM (
				SELECT 
					ProductName = B.[Name],
					LineTotalSum = SUM(A.LineTotal),
					LineTotalSumRank = DENSE_RANK() OVER(ORDER BY SUM(A.LineTotal) DESC)

				FROM AdventureWorks2022.Sales.SalesOrderDetail A
					JOIN AdventureWorks2022.Production.Product B
						ON A.ProductID = B.ProductID

				GROUP BY
					B.[Name]
				) X

			WHERE LineTotalSumRank <= @TopN
		END
	IF @OrderType = 2
		BEGIN
				SELECT
					*
				FROM(
					SELECT 
						ProductName = B.[Name],
						LineTotalSum = SUM(A.LineTotal),
						LineTotalSumRank = DENSE_RANK() OVER(ORDER BY SUM(A.LineTotal) DESC)

					FROM AdventureWorks2022.Purchasing.PurchaseOrderDetail A
						JOIN AdventureWorks2022.Production.Product B
							ON A.ProductID = B.ProductID

					GROUP BY
						B.[Name]
					) X

				WHERE LineTotalSumRank <= @TopN
			END

	IF @OrderType = 3
		BEGIN				
			SELECT
				ProductID,
				LineTotal

			INTO #AllOrders

			FROM AdventureWorks2022.Sales.SalesOrderDetail

			INSERT INTO #AllOrders

			SELECT
				ProductID,
				LineTotal

			FROM AdventureWorks2022.Purchasing.PurchaseOrderDetail
					

			SELECT
				*
			FROM (
				SELECT 
					ProductName = B.[Name],
					LineTotalSum = SUM(A.LineTotal),
					LineTotalSumRank = DENSE_RANK() OVER(ORDER BY SUM(A.LineTotal) DESC)

				FROM #AllOrders A
					JOIN AdventureWorks2022.Production.Product B
						ON A.ProductID = B.ProductID

				GROUP BY
					B.[Name]
				) X

			WHERE LineTotalSumRank <= @TopN

			DROP TABLE #AllOrders
		END
END
```
**Call modified stored procedure**
```
EXEC dbo.OrdersReport 20,1
```
```
EXEC dbo.OrdersReport 15,2
```
```
EXEC dbo.OrdersReport 25,3
```
#### Results:
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_63.JPG)

### Using Multiple IF Statements Exercise

Modify your "dbo.OrdersAboveThreshold" stored procedure once again, such that if a user supplies a value of 3 to the @OrderType parameter, the proc should return all sales AND purchase orders above the specified threshold, with order dates between the specified years.

In this scenario, include an "OrderType" column to the procedure output. This column should have a value of "Sales" for records from the SalesOrderHeader table, and "Purchase" for records from the PurchaseOrderHeader table.

**Hints:**
- Convert your ELSE block to an IF block, so that you now have 3 independent IF blocks.
- Make sure that your IF criteria are all mutually exclusive.
- Use UNION ALL to "stack" the sales and purchase data.
- Alias SalesOrderId/PurchaseOrderID as "OrderID" in their respective UNION-ed queries.

```
ALTER PROCEDURE dbo.OrdersAboveThreshold(@Threshold MONEY, @StartYear INT, @EndYear INT, @OrderType INT)

AS

BEGIN
	IF @OrderType = 1
		BEGIN
			SELECT 
				A.SalesOrderID,
				A.OrderDate,
				A.TotalDue

			FROM  AdventureWorks2022.Sales.SalesOrderHeader A
				JOIN AdventureWorks2022.dbo.Calendar B
					ON A.OrderDate = B.DateValue

			WHERE A.TotalDue >= @Threshold
				AND B.YearNumber BETWEEN @StartYear AND @EndYear

			ORDER BY A.TotalDue DESC
		END

		IF @OrderType = 2
			BEGIN
				SELECT 
					A.PurchaseOrderID,
					A.OrderDate,
					A.TotalDue

				FROM  AdventureWorks2022.Purchasing.PurchaseOrderHeader A
					JOIN AdventureWorks2022.dbo.Calendar B
						ON A.OrderDate = B.DateValue

				WHERE A.TotalDue >= @Threshold
					AND B.YearNumber BETWEEN @StartYear AND @EndYear

				ORDER BY A.TotalDue DESC
			END

		IF @OrderType = 3
			BEGIN
				SELECT 
					OrderID = A.SalesOrderID,
					OrderType = 'Sales',
					A.OrderDate,
					A.TotalDue

				FROM  AdventureWorks2022.Sales.SalesOrderHeader A
					JOIN AdventureWorks2022.dbo.Calendar B
						ON A.OrderDate = B.DateValue

				WHERE A.TotalDue >= @Threshold
					AND B.YearNumber BETWEEN @StartYear AND @EndYear

				UNION ALL

				SELECT 
					OrderID = A.PurchaseOrderID,
					OrderType = 'Purchase',
					A.OrderDate,
					A.TotalDue

				FROM  AdventureWorks2022.Purchasing.PurchaseOrderHeader A
					JOIN AdventureWorks2022.dbo.Calendar B
						ON A.OrderDate = B.DateValue

				WHERE A.TotalDue >= @Threshold
					AND B.YearNumber BETWEEN @StartYear AND @EndYear

				ORDER BY TotalDue DESC
			END
END
```
#### Results:
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_64.JPG)

**[Back To The Top](#Overview-of-the-Section)**
#
### Dynamic SQL Part 1

Dynamic SQL is a programming technique that enables you to build SQL statements dynamically at runtime. 

You can create more general purpose, flexible applications by using dynamic SQL because the full text of a SQL statement may be unknown at compilation.

```
DECLARE @DynamicSQL VARCHAR(MAX)

SET @DynamicSQL = 'SELECT TOP 100 * FROM AdventureWorks2022.Production.Product'

SELECT @DynamicSQL

EXEC(@DynamicSQL)
```
![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_65.JPG)

**[Back To The Top](#Overview-of-the-Section)**

### Dynamic SQL Exercise 1

Create a stored procedure called "NameSearch" that allows users to search the Person.Person table for a pattern provided by the user.

The user should be able to search by either first name, last name, or middle name.

You can return all columns from the table; that is to say, feel free to user SELECT *.

The stored procedure should take two arguments:

@NameToSearch: The user will be expected to enter either "first", "middle", or "last". This way, they do not have to remember exact column names.

@SearchPattern: The user will provide a text string to search for.

A record should be returned if the specified name (first, middle, or last) includes the specified pattern anywhere within it. I.e., if the user tells us to search the FirstName field for the pattern "ravi", both the names "Ravi" and "Travis" should be returned.

**Hints:**

You will probably want to use LIKE with a wildcard in your WHERE clause.

To include single quotes in your dynamic SQL, try "escaping" them by typing four consecutive single quotes ('''').

Try creating a variable to hold the actual column name to search, and then set this variable using IF statements, based on the value passed into the "NameToSearch" parameter by the user. Then simply plug this variable into your dynamic SQL. This is easier than having to execute different queries depending on what was passed in.

```
CREATE PROCEDURE dbo.NameSearch(@NameToSearch VARCHAR(100),@SearchPattern VARCHAR(100))

AS

BEGIN
	DECLARE @NameColumn VARCHAR(100)
	DECLARE @DynamicSQL VARCHAR(MAX)

	IF LOWER(@NameToSearch) = 'First'
		SET @NameColumn = 'FirstName'

	IF LOWER(@NameToSearch) = 'Middle'
		SET @NameColumn = 'MiddleName'

	IF LOWER(@NameToSearch) = 'Last'
		SET @NameColumn = 'LastName'

	SET @DynamicSQL = 
	'SELECT
	*
	FROM AdventureWorks2022.Person.Person
	WHERE '

	SET @DynamicSQL = @DynamicSQL + @NameColumn

	SET @DynamicSQL = @DynamicSQL + ' LIKE ' + '''' + '%' + @SearchPattern + '%' + ''''

	EXEC(@DynamicSQL)
END
```
### Dynamic SQL Exercise 2

Modify your "NameSearch" procedure to accept a third argument - @MatchType, with an INT datatype -  that specifies the match type:
- 1 means "exact match"
- 2 means "begins with"
- 3 means "ends with"
- 4 means "contains"

**Hint:** 

Use a series of IF statements to build out your WHERE clause based on the @MatchType parameter, then append this to the rest of your dynamic SQL before executing.
```
ALTER PROCEDURE dbo.NameSearch(@NameToSearch VARCHAR(100),@SearchPattern VARCHAR(100), @MatchType INT)

AS

BEGIN
	DECLARE @NameColumn VARCHAR(100)
	DECLARE @DynamicWhere VARCHAR(MAX)
	DECLARE @DynamicSQL VARCHAR(MAX)

	IF LOWER(@NameToSearch) = 'First'
		SET @NameColumn = 'FirstName'

	IF LOWER(@NameToSearch) = 'Middle'
		SET @NameColumn = 'MiddleName'

	IF LOWER(@NameToSearch) = 'Last'
		SET @NameColumn = 'LastName'


	IF @MatchType = 1
		SET @DynamicWhere = ' = ' + '''' + @SearchPattern + ''''

	IF @MatchType = 2
		SET @DynamicWhere = ' LIKE ' + '''' + @SearchPattern + '%' + ''''

	IF @MatchType = 3
		SET @DynamicWhere = ' LIKE ' + '''' + '%' + @SearchPattern + ''''

	IF @MatchType = 4
		SET @DynamicWhere = ' LIKE ' + '''' + '%' + @SearchPattern + '%' + ''''


	SET @DynamicSQL = 
	'SELECT
	*
	FROM AdventureWorks2022.Person.Person
	WHERE '

	SET @DynamicSQL = @DynamicSQL + @NameColumn + @DynamicWhere

	SELECT  @DynamicSQL

	EXEC(@DynamicSQL)
END
```
**[Back To The Top](#Overview-of-the-Section)**

