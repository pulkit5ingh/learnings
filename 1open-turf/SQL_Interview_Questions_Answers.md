
# SQL Interview Questions and Answers

## 1. Find the second highest salary from the `Employee` table.

```sql
SELECT MAX(Salary) AS SecondHighestSalary
FROM Employee
WHERE Salary < (SELECT MAX(Salary) FROM Employee);
```

---

## 2. Find all employees whose salary is above the average salary.

```sql
SELECT * 
FROM Employee
WHERE Salary > (SELECT AVG(Salary) FROM Employee);
```

---

## 3. Find the duplicate records in a table.

```sql
SELECT ColumnName, COUNT(*)
FROM TableName
GROUP BY ColumnName
HAVING COUNT(*) > 1;
```

---

## 4. Delete duplicate records from a table.

```sql
DELETE FROM Employee
WHERE EmployeeID NOT IN (
    SELECT MIN(EmployeeID)
    FROM Employee
    GROUP BY FirstName, LastName
);
```

---

## 5. Retrieve the current date in SQL.

```sql
SELECT GETDATE() AS CurrentDate; -- SQL Server
-- Or
SELECT CURRENT_DATE; -- MySQL/PostgreSQL
```

---

## 6. Get the details of employees with the highest salary in each department.

```sql
SELECT Department, EmployeeName, Salary
FROM Employee e
WHERE Salary = (
    SELECT MAX(Salary)
    FROM Employee
    WHERE Department = e.Department
);
```

---

## 7. Find all employees who joined in the year 2023.

```sql
SELECT * 
FROM Employee
WHERE YEAR(JoiningDate) = 2023;
```

---

## 8. Fetch the common records between two tables.

```sql
SELECT * 
FROM TableA
INTERSECT
SELECT * 
FROM TableB;
```

---

## 9. Display employee count per department.

```sql
SELECT Department, COUNT(EmployeeID) AS EmployeeCount
FROM Employee
GROUP BY Department;
```

---

## 10. List all employees whose names start with 'A'.

```sql
SELECT * 
FROM Employee
WHERE EmployeeName LIKE 'A%';
```

---

## 11. Retrieve the employees who do not have a manager.

```sql
SELECT *
FROM Employee
WHERE ManagerID IS NULL;
```

---

## 12. Find the nth highest salary from the `Employee` table.

```sql
SELECT DISTINCT Salary
FROM Employee
ORDER BY Salary DESC
LIMIT 1 OFFSET n - 1; -- Replace 'n' with the desired rank
```

---

## 13. Find employees who have the same salary.

```sql
SELECT Salary, COUNT(EmployeeID)
FROM Employee
GROUP BY Salary
HAVING COUNT(EmployeeID) > 1;
```

---

## 14. Fetch employees whose salary is between 50,000 and 100,000.

```sql
SELECT * 
FROM Employee
WHERE Salary BETWEEN 50000 AND 100000;
```

---

## 15. Retrieve employees with salaries in descending order.

```sql
SELECT *
FROM Employee
ORDER BY Salary DESC;
```

---

## 16. Fetch the first record from the Employee table.

```sql
SELECT * 
FROM Employee
ORDER BY EmployeeID
LIMIT 1;
```

---

## 17. Get the last record from the Employee table.

```sql
SELECT * 
FROM Employee
ORDER BY EmployeeID DESC
LIMIT 1;
```

---

## 18. Fetch the employees who joined in the last 6 months.

```sql
SELECT *
FROM Employee
WHERE JoiningDate >= DATEADD(MONTH, -6, GETDATE()); -- SQL Server
-- Or
SELECT *
FROM Employee
WHERE JoiningDate >= CURRENT_DATE - INTERVAL '6 months'; -- PostgreSQL
```

---

## 19. Find all employees whose salary is greater than their manager's salary.

```sql
SELECT e.EmployeeID, e.EmployeeName, e.Salary
FROM Employee e
JOIN Employee m ON e.ManagerID = m.EmployeeID
WHERE e.Salary > m.Salary;
```

---

## 20. Count the total number of employees.

```sql
SELECT COUNT(*) AS TotalEmployees
FROM Employee;
```

---
