### Keywords
* OFFSET: like page, start with 0 index

### Consideration for null case
* Soltuion-1: Take the result as a temp table
```
SELECT (SELECT DISTINCT Salary FROM Employee ORDER BY Salary DESC LIMIT 1 OFFSET 1) 
AS SecondHighestSalary;
```
* Soltuion-2: Use IFNULL funtion
```
SELECT IFNULL( 
    (SELECT DISTINCT Salary FROM Employee ORDER BY Salary DESC LIMIT 1 OFFSET 1),
    NULL) AS SecondHighestSalary
```
