# Set 12

| S.No. | Question                                                                                                                                                                                                                                                               |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is the `IF EXISTS` clause used for?](#question-1-what-is-the-if-exists-clause-used-for)                                                                                                                                                                          |
| 2.    | [How do you insert data into all columns vs specific columns?](#question-2-how-do-you-insert-data-into-all-columns-vs-specific-columns)                                                                                                                                |
| 3.    | [What happens if you insert fewer values than columns in a table?](#question-3-what-happens-if-you-insert-fewer-values-than-columns-in-a-table)                                                                                                                        |
| 4.    | [How do you prevent duplicate data insertion?](#question-4-how-do-you-prevent-duplicate-data-insertion)                                                                                                                                                                |
| 5.    | [What happens when you try to insert a `NULL` into a `NOT NULL` column?](#question-5-what-happens-when-you-try-to-insert-a-null-into-a-not-null-column)                                                                                                                |
| 6.    | [What is the default value of an uninitialized column?](#question-6-what-is-the-default-value-of-an-uninitialized-column)                                                                                                                                              |
| 7.    | [How do you copy the data of one table into another using `INSERT INTO ... SELECT`?](#question-7-how-do-you-copy-the-data-of-one-table-into-another-using-insert-into--select)                                                                                         |
| 8.    | [What does the `REPLACE()` function do in MySQL?](#question-8-what-does-the-replace-function-do-in-mysql)                                                                                                                                                              |
| 9.    | [How do you remove spaces from both sides of a string?](#question-9-how-do-you-remove-spaces-from-both-sides-of-a-string)                                                                                                                                              |
| 10.   | [What is the difference between `CHAR_LENGTH()` and `OCTET_LENGTH()`?](#question-10-what-is-the-difference-between-char_length-and-octet_length)                                                                                                                       |
| 11.   | [What does the `LPAD()` and `RPAD()` function do?](#question-11-what-does-the-lpad-and-rpad-function-do)                                                                                                                                                               |
| 12.   | [How do you extract only the first three characters of a column?](#question-12-how-do-you-extract-only-the-first-three-characters-of-a-column)                                                                                                                         |
| 13.   | [How can you merge the first name and last name columns into one column?](#question-13-how-can-you-merge-the-first-name-and-last-name-columns-into-one-column)                                                                                                         |
| 14.   | [How can you find records where a column is not null?](#question-14-how-can-you-find-records-where-a-column-is-not-null)                                                                                                                                               |
| 15.   | [What does the `ISNULL()` function do?](#question-15-what-does-the-isnull-function-do)                                                                                                                                                                                 |
| 16.   | [How can you calculate the average salary of each department but only show departments having an average greater than 50000?](#question-16-how-can-you-calculate-the-average-salary-of-each-department-but-only-show-departments-having-an-average-greater-than-50000) |
| 17.   | [How can you count total employees who joined in each year?](#question-17-how-can-you-count-total-employees-who-joined-in-each-year)                                                                                                                                   |
| 18.   | [How can you calculate cumulative sales per month?](#question-18-how-can-you-calculate-cumulative-sales-per-month)                                                                                                                                                     |
| 19.   | [How do you check whether two tables have identical data?](#question-19-how-do-you-check-whether-two-tables-have-identical-data)                                                                                                                                       |
| 20.   | [How do you find missing records between two tables?](#question-20-how-do-you-find-missing-records-between-two-tables)                                                                                                                                                 |

## Question 1. What is the `IF EXISTS` clause used for?

## What is the `IF EXISTS` clause used for in MySQL?

### Definition

The **`IF EXISTS`** clause in **MySQL** is used to **prevent an error when attempting to remove or modify a database object that may not exist**.

Instead of throwing an error, MySQL checks whether the object exists:

- If it exists → the operation is performed.
- If it doesn't exist → MySQL skips the operation and usually returns a warning instead of an error.

This makes scripts **safe, reusable, and idempotent**, especially during deployments and database migrations.

---

## Why is `IF EXISTS` useful?

Imagine you're writing a deployment script:

```sql
DROP TABLE employees;
```

If the `employees` table doesn't exist, MySQL returns:

```text
ERROR 1051 (42S02): Unknown table 'employees'
```

Using `IF EXISTS`:

```sql
DROP TABLE IF EXISTS employees;
```

Now:

- Table exists → dropped successfully.
- Table doesn't exist → no error; execution continues.

This is particularly useful in:

- Deployment scripts
- Database migrations
- CI/CD pipelines
- Automated setup scripts
- Resetting development databases

---

# Sample Schema

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

INSERT INTO employees (name, department, salary, hire_date, manager_id)
VALUES
('Alice','Engineering',75000,'2020-01-15',NULL),
('Bob','Engineering',80000,'2019-03-10',1),
('Charlie','HR',60000,'2021-07-22',NULL);
```

---

# Example 1: Drop a Table Safely

```sql
DROP TABLE IF EXISTS employees;
```

### Line-by-line explanation

```sql
DROP TABLE
```

- Removes a table from the database.

```sql
IF EXISTS
```

- Checks whether the table exists before attempting to drop it.

```sql
employees;
```

- The table to be dropped.

If `employees` doesn't exist, the statement completes without raising an error.

---

# Example 2: Drop Multiple Tables

```sql
DROP TABLE IF EXISTS employees, departments, projects;
```

MySQL checks each table:

- Existing tables are dropped.
- Missing tables are ignored with warnings instead of stopping execution.

---

# Example 3: Drop a Database

```sql
DROP DATABASE IF EXISTS interview_db;
```

Without `IF EXISTS`:

```sql
DROP DATABASE interview_db;
```

If the database doesn't exist:

```text
ERROR 1008 (HY000)
```

Using `IF EXISTS` avoids this error.

---

# Example 4: Drop a View

```sql
DROP VIEW IF EXISTS employee_summary;
```

If the view exists, it's removed; otherwise, MySQL continues without failing.

---

# Example 5: Drop an Index

```sql
DROP INDEX idx_salary ON employees;
```

In recent MySQL versions, you can also write:

```sql
DROP INDEX IF EXISTS idx_salary ON employees;
```

This avoids an error if the index is already absent.

---

# Example 6: Drop a Stored Procedure

```sql
DROP PROCEDURE IF EXISTS GetEmployees;
```

Useful before recreating a procedure:

```sql
DROP PROCEDURE IF EXISTS GetEmployees;

DELIMITER //

CREATE PROCEDURE GetEmployees()
BEGIN
    SELECT * FROM employees;
END //

DELIMITER ;
```

This ensures the script can be run repeatedly without failing because the procedure already exists.

---

# Common statements that support `IF EXISTS`

| Statement        | Example                                          |
| ---------------- | ------------------------------------------------ |
| `DROP TABLE`     | `DROP TABLE IF EXISTS employees;`                |
| `DROP DATABASE`  | `DROP DATABASE IF EXISTS interview_db;`          |
| `DROP VIEW`      | `DROP VIEW IF EXISTS employee_summary;`          |
| `DROP INDEX`     | `DROP INDEX IF EXISTS idx_salary ON employees;`  |
| `DROP PROCEDURE` | `DROP PROCEDURE IF EXISTS GetEmployees;`         |
| `DROP FUNCTION`  | `DROP FUNCTION IF EXISTS CalculateBonus;`        |
| `DROP EVENT`     | `DROP EVENT IF EXISTS yearly_cleanup;`           |
| `DROP TRIGGER`   | `DROP TRIGGER IF EXISTS before_insert_employee;` |

---

# `IF EXISTS` vs `IF NOT EXISTS`

These clauses serve opposite purposes.

### `IF EXISTS`

Used when **dropping or altering** an object to avoid errors if it is missing.

```sql
DROP TABLE IF EXISTS employees;
```

### `IF NOT EXISTS`

Used when **creating** an object to avoid errors if it already exists.

```sql
CREATE TABLE IF NOT EXISTS employees (
    id INT PRIMARY KEY
);
```

---

# Real-world use case

Suppose your deployment script recreates a table:

```sql
DROP TABLE IF EXISTS employees;

CREATE TABLE employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100)
);
```

Whether the table already exists or not, the script executes successfully, making it safe to rerun during development, testing, or automated deployments.

---

# Performance considerations

- `IF EXISTS` performs a metadata check before executing the operation.
- The overhead is minimal and negligible compared to the benefits of preventing script failures.
- It is considered a best practice in migration and deployment scripts.

---

# Common pitfalls

1. **Not all statements support `IF EXISTS`**
   Always verify support for the specific statement in the latest MySQL documentation, as availability varies by object type and MySQL version.

2. **Warnings vs. errors**
   When an object doesn't exist, MySQL may issue a warning instead of an error. If your application treats warnings specially, be aware of this behavior.

3. **Doesn't suppress other errors**
   `IF EXISTS` only handles the case where the object is absent. Errors such as insufficient privileges, dependency conflicts, or syntax errors will still be reported.

---

# Interview Tips

- `IF EXISTS` is primarily used with `DROP` statements and certain other object-removal operations.
- It prevents scripts from failing when the target object does not exist.
- It is especially valuable in database migrations, deployment automation, and repeatable setup scripts.
- Pair it with `IF NOT EXISTS` for writing robust, rerunnable database initialization scripts.

## Question 2. How do you insert data into all columns vs specific columns?

## Question 3. What happens if you insert fewer values than columns in a table?

## Question 4. How do you prevent duplicate data insertion?

## Question 5. What happens when you try to insert a `NULL` into a `NOT NULL` column?

## Question 6. What is the default value of an uninitialized column?

## Question 7. How do you copy the data of one table into another using `INSERT INTO ... SELECT`?

## Question 8. What does the `REPLACE()` function do in MySQL?

## Question 9. How do you remove spaces from both sides of a string?

## Question 10. What is the difference between `CHAR_LENGTH()` and `OCTET_LENGTH()`?

## Question 11. What does the `LPAD()` and `RPAD()` function do?

## Question 12. How do you extract only the first three characters of a column?

## Question 13. How can you merge the first name and last name columns into one column?

## Question 14. How can you find records where a column is not null?

## Question 15. What does the `ISNULL()` function do?

## Question 16. How can you calculate the average salary of each department but only show departments having an average greater than 50000?

## Question 17. How can you count total employees who joined in each year?

## Question 18. How can you calculate cumulative sales per month?

## Question 19. How do you check whether two tables have identical data?

## Question 20. How do you find missing records between two tables?
