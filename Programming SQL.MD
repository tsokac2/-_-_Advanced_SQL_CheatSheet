<h1 align="center">Programming SQL</h1>

### Overview of the Section
* **[Variables](#Variables)**
    * **[Variables Exercise](#Variables-Exercise)**
	

#
### Variables

- A variable is a named placeholder for a value or set of values.
- A variable only needs to be defined once but can be reused many times.
- This allows us to consolidate complex logic into a variable, and then refer to the variable as many times as we need without repeating the logic. 

### Variables Exercise

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