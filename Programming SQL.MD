<h1 align="center">Programming SQL</h1>

### Overview of the Section
* **[Variables](#Variables)**
    * **[Variables Exercise 1](#Variables-Exercise-1)**
	* **[Variables Exercise 2](#Variables-Exercise-2)**
	

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