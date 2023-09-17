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
* **[Using Multiple IF Statements](#Using-Multiple-IF-Statements)**
	* **[Using Multiple IF Statements Exercise](#Using-Multiple-IF-Statements-Exercise)**
* **[Dynamic SQL Part 1](#Dynamic-SQL-Part-1)**
* **[Dynamic SQL Part 2](#Dynamic-SQL-Part-2)**
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

![Section_04](https://github.com/tsokac2/-_-_Advanced_SQL_CheatSheet/blob/main/img/Section_04_E_58.JPG)

**[Back To The Top](#Overview-of-the-Section)**

#
### Control Flow With IF Statements

### Control Flow With IF Statements Exercise

**[Back To The Top](#Overview-of-the-Section)**

#
### Using Multiple IF Statements

### Using Multiple IF Statements Exercise

**[Back To The Top](#Overview-of-the-Section)**

#
### Dynamic SQL Part 1

**[Back To The Top](#Overview-of-the-Section)**

#
### Dynamic SQL Part 2

### Dynamic SQL Exercise 1

### Dynamic SQL Exercise 2

**[Back To The Top](#Overview-of-the-Section)**

