# Set 2

| S.No. | Question                                                                                                                                               |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.    | [What are aggregate functions in MySQL? Name a few](#question-1-what-are-aggregate-functions-in-mysql-name-a-few)                                      |
| 2.    | [How do you count the total number of records in a table?](#question-2-how-do-you-count-the-total-number-of-records-in-a-table)                        |
| 3.    | [What is the use of the `GROUP BY` clause?](#question-3-what-is-the-use-of-the-group-by-clause)                                                        |
| 4.    | [What is the use of the `HAVING` clause?](#question-4-what-is-the-use-of-the-having-clause)                                                            |
| 5.    | [Explain the difference between `INNER JOIN` and `LEFT JOIN`](#question-5-explain-the-difference-between-inner-join-and-left-join)                     |
| 6.    | [What is a `CROSS JOIN`?](#question-6-what-is-a-cross-join)                                                                                            |
| 7.    | [What is a `SELF JOIN`?](#question-7-what-is-a-self-join)                                                                                              |
| 8.    | [What is normalization? Why is it important?](#question-8-what-is-normalization-why-is-it-important)                                                   |
| 9.    | [What is denormalization?](#question-9-what-is-denormalization)                                                                                        |
| 10.   | [What is the difference between DDL, DML, and DCL statements?](#question-10-what-is-the-difference-between-ddl-dml-and-dcl-statements)                 |
| 11.   | [What are indexes in MySQL and why are they used?](#question-11-what-are-indexes-in-mysql-and-why-are-they-used)                                       |
| 12.   | [How do you add and remove an index in MySQL?](#question-12-how-do-you-add-and-remove-an-index-in-mysql)                                               |
| 13.   | [What is a `VIEW` in MySQL?](#question-13-what-is-a-view-in-mysql)                                                                                     |
| 14.   | [How do you create and delete a view?](#question-14-how-do-you-create-and-delete-a-view)                                                               |
| 15.   | [How can you import and export data in MySQL?](#question-15-how-can-you-import-and-export-data-in-mysql)                                               |
| 16.   | [Explain the concept of ACID properties in MySQL](#question-16-explain-the-concept-of-acid-properties-in-mysql)                                        |
| 17.   | [What is the difference between `MyISAM` and `InnoDB` storage engines?](#question-17-what-is-the-difference-between-myisam-and-innodb-storage-engines) |
| 18.   | [What is a transaction in MySQL?](#question-18-what-is-a-transaction-in-mysql)                                                                         |
| 19.   | [How do you start, commit, and rollback a transaction?](#question-19-how-do-you-start-commit-and-rollback-a-transaction)                               |
| 20.   | [What is a deadlock and how can you avoid it?](#question-20-what-is-a-deadlock-and-how-can-you-avoid-it)                                               |

## Question 1. What are aggregate functions in MySQL? Name a few

### What Are Aggregate Functions in MySQL?

**Aggregate functions** in MySQL are functions that perform a calculation on **multiple rows of data** and return **a single result value**.

They are commonly used with the `SELECT` statement to summarize data, such as calculating totals, averages, counts, minimums, and maximums.

Think of aggregate functions as tools that answer questions like:

- How many employees are there?
- What is the average salary?
- What is the highest salary?
- What is the total payroll cost?

---

## Common Aggregate Functions in MySQL

| Function           | Purpose                                                     |
| ------------------ | ----------------------------------------------------------- |
| `COUNT()`          | Counts rows                                                 |
| `SUM()`            | Calculates total value                                      |
| `AVG()`            | Calculates average value                                    |
| `MIN()`            | Finds the smallest value                                    |
| `MAX()`            | Finds the largest value                                     |
| `GROUP_CONCAT()`   | Concatenates values from multiple rows into a single string |
| `JSON_ARRAYAGG()`  | Aggregates values into a JSON array                         |
| `JSON_OBJECTAGG()` | Aggregates key-value pairs into a JSON object               |

According to the latest MySQL documentation, these functions are designed to operate on sets of rows and are frequently used with `GROUP BY` clauses.

---

## Sample Table

```sql
CREATE DATABASE interview_db;
USE interview_db;

CREATE TABLE employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    department VARCHAR(50),
    salary DECIMAL(10,2),
    hire_date DATE,
    manager_id INT
);

INSERT INTO employees (name, department, salary, hire_date, manager_id) VALUES
('Alice', 'Engineering', 75000, '2020-01-15', NULL),
('Bob', 'Engineering', 80000, '2019-03-10', 1),
('Charlie', 'HR', 60000, '2021-07-22', NULL);
```

---

# 1. COUNT()

Counts the number of rows.

### Example

```sql
SELECT COUNT(*) AS total_employees
FROM employees;
```

### Explanation

- `COUNT(*)` counts every row in the table.
- Returns:

```text
total_employees
---------------
3
```

### Variations

```sql
SELECT COUNT(manager_id)
FROM employees;
```

Counts only non-NULL values in `manager_id`.

---

# 2. SUM()

Calculates the total of numeric values.

### Example

```sql
SELECT SUM(salary) AS total_salary
FROM employees;
```

### Explanation

```text
75000 + 80000 + 60000 = 215000
```

Result:

```text
total_salary
------------
215000
```

---

# 3. AVG()

Calculates the average value.

### Example

```sql
SELECT AVG(salary) AS avg_salary
FROM employees;
```

### Explanation

```text
(75000 + 80000 + 60000) / 3
= 71666.67
```

Result:

```text
avg_salary
----------
71666.67
```

---

# 4. MIN()

Returns the smallest value.

### Example

```sql
SELECT MIN(salary) AS lowest_salary
FROM employees;
```

Result:

```text
lowest_salary
-------------
60000
```

Employee:

```text
Charlie
```

---

# 5. MAX()

Returns the largest value.

### Example

```sql
SELECT MAX(salary) AS highest_salary
FROM employees;
```

Result:

```text
highest_salary
--------------
80000
```

Employee:

```text
Bob
```

---

# Aggregate Functions with GROUP BY

Aggregate functions become even more powerful when combined with `GROUP BY`.

### Example: Department-wise Average Salary

```sql
SELECT
    department,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department;
```

### Execution

Engineering:

```text
(75000 + 80000) / 2 = 77500
```

HR:

```text
60000
```

Output:

```text
department   avg_salary
----------   ----------
Engineering  77500
HR           60000
```

---

# GROUP_CONCAT()

Combines multiple row values into a single string.

### Example

```sql
SELECT
    department,
    GROUP_CONCAT(name) AS employees
FROM employees
GROUP BY department;
```

Output:

```text
Engineering   Alice,Bob
HR            Charlie
```

Useful for reporting and dashboards.

---

# JSON Aggregation Functions (Modern MySQL)

### JSON_ARRAYAGG()

Creates a JSON array.

```sql
SELECT JSON_ARRAYAGG(name) AS employee_names
FROM employees;
```

Output:

```json
["Alice", "Bob", "Charlie"]
```

---

### JSON_OBJECTAGG()

Creates a JSON object.

```sql
SELECT JSON_OBJECTAGG(id, name)
FROM employees;
```

Output:

```json
{
  "1": "Alice",
  "2": "Bob",
  "3": "Charlie"
}
```

These are commonly used in modern APIs and microservices.

---

# Important Interview Points

### 1. Aggregate Functions Ignore NULL Values

Example:

```sql
SELECT AVG(manager_id)
FROM employees;
```

`NULL` values are ignored during calculation.

---

### 2. COUNT(\*) vs COUNT(column)

```sql
COUNT(*)        -- Counts all rows
COUNT(column)   -- Counts only non-NULL values
```

Example:

```sql
SELECT COUNT(*) FROM employees;
SELECT COUNT(manager_id) FROM employees;
```

Output:

```text
3
1
```

because only Bob has a non-NULL `manager_id`.

---

### 3. WHERE vs HAVING

`WHERE` filters rows before aggregation.

```sql
SELECT AVG(salary)
FROM employees
WHERE department = 'Engineering';
```

`HAVING` filters groups after aggregation.

```sql
SELECT department, AVG(salary)
FROM employees
GROUP BY department
HAVING AVG(salary) > 70000;
```

---

# Performance Considerations

### Use EXPLAIN

```sql
EXPLAIN
SELECT department, AVG(salary)
FROM employees
GROUP BY department;
```

This helps identify:

- Full table scans
- Index usage
- Temporary tables
- Sorting operations

### Optimization

Create indexes on grouped columns:

```sql
CREATE INDEX idx_department
ON employees(department);
```

This can improve `GROUP BY` performance significantly on large tables.

---

# Interview Answer (Short Version)

**Aggregate functions in MySQL perform calculations on multiple rows and return a single summarized value. Common aggregate functions include `COUNT()`, `SUM()`, `AVG()`, `MIN()`, and `MAX()`. They are often used with `GROUP BY` to generate summaries such as department-wise totals, averages, or counts. Modern MySQL also provides aggregation functions like `GROUP_CONCAT()`, `JSON_ARRAYAGG()`, and `JSON_OBJECTAGG()`.**

## Question 2. How do you count the total number of records in a table?

## Question 3. What is the use of the `GROUP BY` clause?

## Question 4. What is the use of the `HAVING` clause?

## Question 5. Explain the difference between `INNER JOIN` and `LEFT JOIN`

## Question 6. What is a `CROSS JOIN`?

## Question 7. What is a `SELF JOIN`?

## Question 8. What is normalization? Why is it important?

## Question 9. What is denormalization?

## Question 10. What is the difference between DDL, DML, and DCL statements?

## Question 11. What are indexes in MySQL and why are they used?

## Question 12. How do you add and remove an index in MySQL?

## Question 13. What is a `VIEW` in MySQL?

## Question 14. How do you create and delete a view?

## Question 15. How can you import and export data in MySQL?

## Question 16. Explain the concept of ACID properties in MySQL

## Question 17. What is the difference between `MyISAM` and `InnoDB` storage engines?

## Question 18. What is a transaction in MySQL?

## Question 19. How do you start, commit, and rollback a transaction?

## Question 20. What is a deadlock and how can you avoid it?
