
# Basic SQL Queries for Database Setup and Operations

## 1. Create a New Database
```sql
CREATE DATABASE CompanyDB;
```
This command creates a new database called `CompanyDB`.

---

## 2. Use the Database
```sql
USE CompanyDB;
```
This command selects `CompanyDB` as the active database for running further SQL commands.

---

## 3. Create Department Table
```sql
CREATE TABLE Department (
    DepartmentID INT PRIMARY KEY AUTO_INCREMENT,
    DepartmentName VARCHAR(50) NOT NULL
);
```
Defines the `Department` table with columns `DepartmentID` and `DepartmentName`, where `DepartmentID` is a unique identifier for each department.

---

## 4. Create Employee Table (One-to-Many with Department)
```sql
CREATE TABLE Employee (
    EmployeeID INT PRIMARY KEY AUTO_INCREMENT,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    Salary DECIMAL(10, 2),
    JoiningDate DATE,
    DepartmentID INT,
    FOREIGN KEY (DepartmentID) REFERENCES Department(DepartmentID)
);
```
Defines the `Employee` table and sets up a foreign key relationship with the `Department` table, establishing a one-to-many relationship.

---

## 5. Create Address Table (One-to-One with Employee)
```sql
CREATE TABLE Address (
    AddressID INT PRIMARY KEY AUTO_INCREMENT,
    EmployeeID INT UNIQUE,
    Street VARCHAR(100),
    City VARCHAR(50),
    State VARCHAR(50),
    ZipCode VARCHAR(10),
    FOREIGN KEY (EmployeeID) REFERENCES Employee(EmployeeID)
);
```
Defines the `Address` table and establishes a one-to-one relationship with `Employee`.

---

## 6. Create Project Table
```sql
CREATE TABLE Project (
    ProjectID INT PRIMARY KEY AUTO_INCREMENT,
    ProjectName VARCHAR(100) NOT NULL,
    Budget DECIMAL(15, 2) NOT NULL
);
```
Defines the `Project` table to store information about projects.

---

## 7. Create EmployeeProject Table (Many-to-Many Relationship between Employee and Project)
```sql
CREATE TABLE EmployeeProject (
    EmployeeID INT,
    ProjectID INT,
    PRIMARY KEY (EmployeeID, ProjectID),
    FOREIGN KEY (EmployeeID) REFERENCES Employee(EmployeeID),
    FOREIGN KEY (ProjectID) REFERENCES Project(ProjectID)
);
```
Defines the `EmployeeProject` table to link employees and projects in a many-to-many relationship.

---

## Insert Demo Data into Tables

### 8. Insert Multiple Records into Department Table
```sql
INSERT INTO Department (DepartmentName) VALUES 
('Engineering'),
('Human Resources'),
('Marketing');
```
Inserts sample departments.

---

### 9. Insert Multiple Records into Employee Table
```sql
INSERT INTO Employee (FirstName, LastName, Salary, JoiningDate, DepartmentID) VALUES 
('Alice', 'Johnson', 75000, '2022-03-01', 1),
('Bob', 'Smith', 60000, '2023-06-10', 2),
('Charlie', 'Brown', 90000, '2021-07-15', 1),
('Diana', 'Williams', 50000, '2022-11-20', 3);
```
Inserts sample employee records.

---

### 10. Insert Data into Address Table (One-to-One Relationship)
```sql
INSERT INTO Address (EmployeeID, Street, City, State, ZipCode) VALUES 
(1, '123 Main St', 'New York', 'NY', '10001'),
(2, '456 Maple Ave', 'Los Angeles', 'CA', '90001'),
(3, '789 Elm St', 'Chicago', 'IL', '60601'),
(4, '101 Oak St', 'Houston', 'TX', '77001');
```
Inserts addresses for employees.

---

### 11. Insert Data into Project Table
```sql
INSERT INTO Project (ProjectName, Budget) VALUES 
('Website Redesign', 50000),
('New Product Launch', 150000),
('Software Upgrade', 80000);
```
Inserts sample projects.

---

### 12. Insert Many-to-Many Records in EmployeeProject Table
```sql
INSERT INTO EmployeeProject (EmployeeID, ProjectID) VALUES 
(1, 1),
(1, 2),
(2, 3),
(3, 1);
```
Links employees to projects.

---

## Creating Indexes

### 13. Index on Employee’s DepartmentID
```sql
CREATE INDEX idx_department_id ON Employee(DepartmentID);
```

### 14. Index on Address's City
```sql
CREATE INDEX idx_city ON Address(City);
```

---

## SELECT (READ) Queries

### 15. Employees with Their Departments (One-to-Many)
```sql
SELECT e.EmployeeID, e.FirstName, e.LastName, d.DepartmentName
FROM Employee e
JOIN Department d ON e.DepartmentID = d.DepartmentID;
```

### 16. Employees with Their Addresses (One-to-One)
```sql
SELECT e.EmployeeID, e.FirstName, e.LastName, a.Street, a.City, a.State, a.ZipCode
FROM Employee e
JOIN Address a ON e.EmployeeID = a.EmployeeID;
```

### 17. Employees and Their Projects (Many-to-Many)
```sql
SELECT e.EmployeeID, e.FirstName, e.LastName, p.ProjectName
FROM Employee e
JOIN EmployeeProject ep ON e.EmployeeID = ep.EmployeeID
JOIN Project p ON ep.ProjectID = p.ProjectID;
```

---

## UPDATE Queries

### 18. Update Employee Salary
```sql
UPDATE Employee
SET Salary = 80000
WHERE FirstName = 'Bob' AND LastName = 'Smith';
```

### 19. Update Employee Department
```sql
UPDATE Employee
SET DepartmentID = 3
WHERE EmployeeID = 2;
```

### 20. Update Address Information
```sql
UPDATE Address
SET City = 'San Francisco', State = 'CA'
WHERE EmployeeID = 1;
```

---

## DELETE Example

### 21. Delete Employee (with Cascade Delete on Address if Enabled)
```sql
DELETE FROM Employee
WHERE EmployeeID = 4;
```

---

## Optional Database Cleanup
```sql
DROP TABLE IF EXISTS EmployeeProject;
DROP TABLE IF EXISTS Project;
DROP TABLE IF EXISTS Address;
DROP TABLE IF EXISTS Employee;
DROP TABLE IF EXISTS Department;
```

This script covers creating and managing a relational database schema with tables, relationships, indexes, and CRUD operations.
